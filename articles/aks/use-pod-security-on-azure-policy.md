---
title: Sécuriser les pods avec Azure Policy dans Azure Kubernetes Service (AKS)
description: Découvrez comment sécuriser les pods avec Azure Policy dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 09/22/2020
ms.openlocfilehash: 8e437095b3d527647a453ba89adaa2ab62672177
ms.sourcegitcommit: 99955130348f9d2db7d4fb5032fad89dad3185e7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93348523"
---
# <a name="secure-pods-with-azure-policy"></a>Sécuriser les pods avec Azure Policy

Pour améliorer la sécurité de votre cluster AKS, vous pouvez contrôler quelles fonctions sont accordées aux pods et si quelque chose va à l'encontre de la politique de l'entreprise. Cet accès est défini par le biais de stratégies intégrées fournies par le module complémentaire [Azure Policy pour AKS][kubernetes-policy-reference]. Le contrôle supplémentaire sur les aspects de sécurité de la spécification de votre pod, comme les privilèges racine, permet une adhésion de sécurité plus stricte et une meilleure visibilité sur ce qui est déployé dans votre cluster. Si un pod ne remplit pas les conditions spécifiées dans la stratégie, Azure Policy peut empêcher le pod de démarrer ou signaler une violation. Cet article explique comment utiliser Azure Policy pour limiter le déploiement de pods dans AKS.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

### <a name="install-the-azure-policy-add-on-for-aks"></a>Installer le module complémentaire Azure Policy pour AKS

Pour sécuriser les pods AKS via Azure Policy, vous devez installer le module complémentaire Azure Policy pour AKS sur un cluster AKS. Pour installer le module complémentaire Azure Policy, [procédez comme suit](../governance/policy/concepts/policy-for-kubernetes.md#install-azure-policy-add-on-for-aks).

Ce document suppose que vous disposez des éléments suivants, qui sont déployés dans la procédure pas à pas liée ci-dessus.

* Inscrit le `Microsoft.ContainerService` et `Microsoft.PolicyInsights` fournisseurs de ressources à l’aide de `az provider register`
* Azure CLI 2.12 ou version ultérieure
* Un cluster AKS sur une version 1.15 ou ultérieure installée avec le module complémentaire Azure Policy

## <a name="overview-of-securing-pods-with-azure-policy-for-aks"></a>Présentation de la sécurisation des pods avec Azure Policy pour AKS

>[!NOTE]
> Ce document explique en détail comment utiliser Azure Policy pour sécuriser les pods, qui sont les successeurs de la fonctionnalité de stratégie de sécurité des pods [ Kubernetes dans la version préliminaire](use-pod-security-policies.md).
> **La stratégie de sécurité des pods (version préliminaire) et le module complémentaire Azure Policy pour AKS ne peuvent pas être activés simultanément.**
> 
> Si vous installez le module complémentaire Azure Policy dans un cluster avec la stratégie de sécurité des pods activée, [procédez comme suit pour désactiver la stratégie de sécurité des pods](use-pod-security-policies.md#enable-pod-security-policy-on-an-aks-cluster).

Dans un cluster AKS, un contrôleur d’admission est utilisé pour intercepter les requêtes envoyées au serveur d’API lorsqu’une ressource doit être créée. Le contrôleur d’admission peut ensuite *valider* la requête de ressource par rapport à un ensemble de règles à l’endroit où elle doit être créée.

Auparavant, la fonctionnalité [Stratégie de sécurité des pods (version préliminaire)](use-pod-security-policies.md) était activée via le projet Kubernetes pour limiter les pods pouvant être déployés.

Via le module complémentaire Azure Policy, un cluster AKS peut utiliser des politiques Azure intégrées qui sécurisent des pods et d’autres ressources Kubernetes similaires à la stratégie de sécurité des pods utilisée auparavant. Le module complémentaire Azure Policy pour AKS installe une instance gérée de [Gatekeeper](https://github.com/open-policy-agent/gatekeeper), un contrôleur d’admission de validation. Azure Policy pour Kubernetes est intégré à l’agent open source Open Policy qui s’appuie sur le [langage de stratégie Rego](../governance/policy/concepts/policy-for-kubernetes.md#policy-language).

Ce document explique en détail comment utiliser Azure Policy pour sécuriser des pods dans un cluster AKS et comment migrer à partir de stratégies de sécurité Pod (version préliminaire).

## <a name="limitations"></a>Limites

Les limitations générales suivantes s’appliquent au module complémentaire Azure Policy pour les clusters Kubernetes :

- Le module complémentaire Azure Policy pour Kubernetes est pris en charge sur la version **1.14** ou ultérieure de Kubernetes.
- Le module complémentaire Azure Policy pour Kubernetes peut uniquement être déployé dans des pools de nœuds Linux
- Seules les définitions de stratégie intégrées sont prises en charge
- Nombre maximal d’enregistrements non conformes par stratégie par cluster : **500**
- Nombre maximal d’enregistrements non conformes par abonnement : **1 million**
- Les installations de Gatekeeper en dehors du module complémentaire Azure Policy ne sont pas prises en charge. Désinstallez tous les composants installés par une installation antérieure de Gatekeeper avant d’activer le module complémentaire Azure Policy.
- Les [raisons de non-conformité](../governance/policy/how-to/determine-non-compliance.md#compliance-reasons) ne sont pas disponibles pour le [mode Fournisseur de ressources](../governance/policy/concepts/definition-structure.md#resource-provider-modes).

Les limitations suivantes s’appliquent uniquement au module complémentaire Azure Policy pour AKS :

- La [stratégie de sécurité des pods AKS (préversion)](use-pod-security-policies.md) et le module complémentaire Azure Policy pour AKS ne peuvent pas être activés simultanément. 
- Espaces de noms automatiquement exclus par le module complémentaire Azure Policy à des fins d’évaluation : _kube-system_ , _gatekeeper-system_ et _aks-periscope_.

### <a name="recommendations"></a>Recommandations

Voici les recommandations générales relatives à l’utilisation du module complémentaire Azure Policy :

- Le module complémentaire Azure Policy requiert 3 composants Gatekeeper pour s’exécuter : 1 pod d’audit et 2 réplicas de pod de webhook. Ces composants consomment davantage de ressources, car le nombre de ressources Kubernetes et d’attributions de stratégie augmente dans le cluster, ce qui nécessite des opérations d’audit et de mise en œuvre.

  - Pour moins de 500 pods dans un seul cluster avec un maximum de 20 contraintes : 2 processeurs virtuels et 350 Mo de mémoire par composant.
  - Pour plus de 500 pods dans un seul cluster avec un maximum de 40 contraintes : 3 processeurs virtuels et 600 Mo de mémoire par composant.

La recommandation suivante s’applique uniquement à AKS et au module complémentaire Azure Policy :

- Utilisez le pool de nœuds système avec la teinte `CriticalAddonsOnly` pour planifier des pods Gatekeeper. Pour plus d’informations, consultez [Utilisation de pools de nœuds système](use-system-pools.md#system-and-user-node-pools).
- Sécurisez le trafic sortant de vos clusters AKS. Pour plus d’informations, consultez [Contrôle du trafic de sortie pour les nœuds de cluster](limit-egress-traffic.md).
- Si `aad-pod-identity` est activé dans le cluster, les pods NMI (Node Managed Identity) modifient les tables d’adresses IP des nœuds pour intercepter les appels vers le point de terminaison Azure Instance Metadata. Cette configuration signifie que toutes les requêtes adressées au point de terminaison Metadata sont interceptées par NMI, même si le pod n’utilise pas `aad-pod-identity`. La CRD AzurePodIdentityException peut être configurée de manière à informer `aad-pod-identity` que toutes les requêtes adressées au point de terminaison Metadata depuis un pod correspondant aux étiquettes définies dans la CRD doivent être envoyées par proxy sans aucun traitement dans NMI. Les pods système qui disposent de l'étiquette `kubernetes.azure.com/managedby: aks` dans l’espace de noms _kube-system_ doivent être exclus de `aad-pod-identity` en configurant la CRD AzurePodIdentityException. Pour plus d’informations, consultez [Désactiver aad-pod-identity pour un pod ou une application spécifique](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).
  Pour configurer une exception, installez le fichier [YAML mic-exception](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml).

Le module complémentaire Azure Policy requiert des ressources de processeur et de mémoire pour fonctionner. Ces exigences augmentent à mesure que la taille d’un cluster augmente. Consultez les [recommandations d’Azure Policy][policy-recommendations] pour une générale relative à l’utilisation du module complémentaire Azure Policy.

## <a name="azure-policies-to-secure-kubernetes-pods"></a>Stratégies Azure pour sécuriser les pods Kubernetes

Après l’installation du module complémentaire Azure Policy, aucune stratégie n’est appliquée par défaut.

Il existe 11 stratégies Azure intégrées et deux initiatives intégrées qui sécurisent spécifiquement les pods dans un cluster AKS.
Chaque stratégie peut être personnalisée à l’aide d’un effet. La liste complète des stratégies [AKS et leurs effets pris en charge sont répertoriées ici][policy-samples]. Découvrez plus en détail les effets [Azure Policy](../governance/policy/concepts/effects.md).

Les stratégies Azure peuvent être appliquées au niveau du groupe d’administration, de l’abonnement ou des groupes de ressources. Lorsque vous affectez une stratégie au niveau du groupe de ressources, assurez-vous que le groupe de ressources du cluster AKS cible est sélectionné dans l’étendue de la stratégie. Chaque cluster de l’étendue attribuée avec le module complémentaire Azure Policy installé se situe dans l’étendue de la stratégie.

Si vous utilisez une [stratégie de sécurité des pods (préversion)](use-pod-security-policies.md), découvrez comment [migrer vers Azure Policy et apprenez-en plus sur les autres différences de comportement](#migrate-from-kubernetes-pod-security-policy-to-azure-policy).

### <a name="built-in-policy-initiatives"></a>Initiatives de stratégie intégrées

Une initiative dans Azure Policy est une collection de définitions de stratégie qui sont spécialement conçues pour atteindre un objectif global particulier. L’utilisation d’initiatives peut simplifier la gestion et l’affectation de stratégies entre les clusters AKS. Une initiative existe sous la forme d’un objet unique, en savoir plus sur les initiatives [Azure Policy](../governance/policy/overview.md#initiative-definition).

Azure Policy pour Kubernetes offre deux initiatives intégrées qui sécurisent les pods, la [ligne de base](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) et la [restreinte](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00).

Les deux initiatives intégrées sont générées à partir des définitions utilisées dans la [stratégie de sécurité des pods à partir de Kubernetes](https://github.com/kubernetes/website/blob/master/content/en/examples/policy/baseline-psp.yaml).

|[Contrôle de la sécurité des pods](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Lien de définition d’Azure Policy| [Initiative de la ligne de base](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) | [Initiative restreinte](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) |
|---|---|---|---|
|Interdire l’exécution des conteneurs privilégiés|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F95edb821-ddaf-4404-9732-666045e056b4)| Oui | Oui
|Interdire l’utilisation partagée des espaces de noms d’hôtes|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F47a1ee2f-2a2a-4576-bf2a-e0e36709c2b8)| Oui | Oui
|Restreindre toute utilisation des ports et des réseaux hôtes|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F82985f06-dc18-4a48-bc1c-b9f4f0098cfe)| Oui | Oui
|Restreindre toute utilisation du système de fichiers hôte|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F098fc59e-46c7-4d99-9b16-64990e543d75)| Oui | Oui
|Restreindre les capacités Linux au-delà [de l’ensemble par défaut](https://docs.docker.com/engine/reference/run/#runtime-privilege-and-linux-capabilities)|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fc26596ff-4d70-4e6a-9a30-c2506bd2f80c) | Oui | Oui
|Restreindre l’utilisation des types de volumes définis|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F16697877-1118-4fb1-9b65-9898ec2509ec)| - | Oui, les types de volumes autorisés sont `configMap`, `emptyDir`, `projected`, `downwardAPI` et `persistentVolumeClaim`|
|Élévation des privilèges à la racine|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F1c6e92c9-99f0-4e55-9cf2-0c234dc48f99) | - | Oui |
|Restreindre les ID d’utilisateur et de groupe du conteneur|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | Oui|
|Restreindre l’allocation d’un FSGroup qui possède les volumes du pod|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff06ddb64-5fa3-4b77-b166-acb36f7f6042) | - | Oui, les règles autorisées sont `runAsUser: mustRunAsNonRoot`, `supplementalGroup: mustRunAs 1:65536`, `fsGroup: mustRunAs 1:65535` et `runAsGroup: mustRunAs 1:65535`.  |
|Nécessite un profil seccomp|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F975ce327-682c-4f2e-aa46-b9598289b86c) | - | Oui, les profils autorisés sont *`docker/default` ou `runtime/default` |

\*docker/default est déconseillé dans Kubernetes depuis la version 1.11

### <a name="additional-optional-policies"></a>Autres stratégies facultatives

Des stratégies Azure supplémentaires peuvent être appliquées séparément en dehors de l’application d’une initiative. Envisagez de les ajouter en plus des initiatives si les initiatives intégrées ne répondent pas à vos besoins.

|[Contrôle de la sécurité des pods](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)| Lien de définition d’Azure Policy| Appliquer en plus de l’initiative de la ligne de base | Appliquer en plus de l’initiative restreinte |
|---|---|---|---|
|Définir le profil AppArmor utilisé par les conteneurs|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F511f5417-5d12-434d-ab2e-816901e72a5e) | Facultatif | Facultatif |
|Autoriser les montages qui ne sont pas en lecture seule|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Fdf49d893-a74c-421d-bc95-c663042e5b80) | Facultatif | Facultatif |
|Restreindre à des pilotes FlexVolume spécifiques|[Cloud public](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2Ff4a8fce0-2dd5-4c21-9a36-8f0ec809d663) | Facultatif : à utiliser si vous souhaitez restreindre uniquement les pilotes FlexVolume, mais pas les autres définis par « Restreindre l’utilisation des types de volumes définis » | Non applicable : l’initiative restreinte comprend « Restreindre l’utilisation des types de volumes définis », ce qui interdit tous les pilotes FlexVolume |

### <a name="unsupported-built-in-policies-for-managed-aks-clusters"></a>Stratégies intégrées non prises en charge pour les clusters AKS gérés

> [!NOTE]
> Les trois stratégies suivantes **ne sont pas prises en charge dans AKS** en raison des aspects de personnalisation gérés et sécurisés par AKS en tant que service géré. Ces stratégies sont générées spécifiquement pour les clusters connectés Azure Arc avec des plans de contrôle non gérés.

|[Contrôle de la sécurité des pods](https://kubernetes.io/docs/concepts/policy/pod-security-policy/#what-is-a-pod-security-policy)|
|---|
|Définir le contexte SELinux personnalisé d’un conteneur|
|Restreindre le profil sysctl utilisé par les conteneurs|
|Les types de montage du processeurroc par défaut sont définis pour réduire la surface d'attaque|

<!---
# Removing until custom initiatives are supported the week after preview

#### Custom initiative

If the built-in initiatives to address pod security do not match your requirements, you can choose your own policies to exist in a custom initiative. Read more about [building custom initatives in Azure Policy](../governance/policy/tutorials/create-and-manage#create-and-assign-an-initiative-definition).

--->

### <a name="namespace-exclusion"></a>Exclusion d’espace de noms

> [!WARNING]
> Les pods dans les espaces de noms d’administrateur tels que kube-system doivent s’exécuter pour qu’un cluster reste intègre, et la suppression d’un espace de noms requis de la liste des espaces de noms exclus par défaut peut déclencher des violations de stratégie en raison d’un pod système requis.

AKS requiert que les pods système s’exécutent sur un cluster pour fournir des services essentiels, tels que la résolution DNS. Les stratégies qui limitent la fonctionnalité des pods peuvent avoir un impact sur la capacité de stabilité du pod système. Par conséquent, les espaces de noms suivants sont **exclus de l’évaluation de la stratégie pendant les demandes d’admission lors de la création, de la mise à jour et de l’audit de stratégie**. Cela oblige les nouveaux déploiements vers ces espaces de noms à être exclus des stratégies Azure.

1. kube-system
1. gatekeeper-system
1. azure-arc
1. aks-periscope

Les espaces de noms personnalisés supplémentaires peuvent être exclus de l’évaluation lors de la création, de la mise à jour et de l’audit. Ces exclusions doivent être utilisées si vous avez des pods spécialisés qui s’exécutent dans un espace de noms approuvé et que vous souhaitez éviter de déclencher des violations d’audit.

## <a name="apply-the-baseline-initiative"></a>Appliquer l’initiative de la ligne de base

> [!TIP]
> La valeur par défaut de toutes les stratégies est un effet d’audit. Les effets peuvent être mis à jour pour refuser systématiquement via Azure Policy.

Pour appliquer l’initiative de base, nous pouvons attribuer via le Portail Azure.
<!--
1. Navigate to the Policy service in Azure portal
1. In the left pane of the Azure Policy page, select **Definitions**
1. Search for "Baseline Profile" on the search pane to the right of the page
1. Select `Kubernetes Pod Security Standards Baseline Profile for Linux-based workloads` from the `Kubernetes` category
-->
1. Suivez [ce lien vers le](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) Portail Azure pour passer en revue l’initiative de la ligne de base de sécurité des pods
1. Définir **l’Étendue** au niveau de l’abonnement ou uniquement au groupe de ressources contenant les clusters AKS avec le module complémentaire Azure Policy activé
1. Sélectionnez la page **Paramètres** et mettez à jour  **l’Effet** de `audit` à `deny` pour bloquer les nouveaux déploiements enfreignant l’initiative de base
1. Ajoutez des espaces de noms supplémentaires à exclure de l’évaluation lors de la création, de la mise à jour et de l’audit, [certains espaces de noms sont exclus obligatoirement de l’évaluation de la stratégie.](#namespace-exclusion)
![effet de mise à jour](media/use-pod-security-on-azure-policy/update-effect.png)
1. Cliquez sur **Vérifier + Créer** pour envoyer les stratégies

Vérifiez que les stratégies sont appliquées à votre cluster en exécutant `kubectl get constrainttemplates`.

> [!NOTE]
> Les stratégies peuvent prendre [jusqu’à 20 minutes pour synchroniser ](../governance/policy/concepts/policy-for-kubernetes.md#assign-a-built-in-policy-definition) dans chaque cluster.

La sortie doit ressembler à ce qui suit :

```console
$ kubectl get constrainttemplate
NAME                                     AGE
k8sazureallowedcapabilities              30m
k8sazureblockhostnamespace               30m
k8sazurecontainernoprivilege             30m
k8sazurehostfilesystem                   30m
k8sazurehostnetworkingports              30m
```

## <a name="validate-rejection-of-a-privileged-pod"></a>Valider le rejet d’un pod privilégié

Nous allons commencer par tester ce qui se passe lorsque vous planifiez un pod avec le contexte de sécurité `privileged: true`. Ce contexte de sécurité fait remonter les privilèges du pod. L’initiative de base n’autorise pas les pods privilégiés, de sorte que la requête est refusée, entraînant le rejet du déploiement.

Créez un fichier nommé `nginx-privileged.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
      securityContext:
        privileged: true
```

Créez le pod à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f nginx-privileged.yaml
```

LComme prévu, la planification de la commande échoue, comme indiqué dans l’exemple de sortie suivant :

```console
$ kubectl apply -f privileged.yaml

Error from server ([denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}): error when creating "privileged.yaml": admission webhook "validation.gatekeeper.sh" denied the request: [denied by azurepolicy-container-no-privilege-00edd87bf80f443fa51d10910255adbc4013d590bec3d290b4f48725d4dfbdf9] Privileged container is not allowed: nginx-privileged, securityContext: {"privileged": true}
```

Le pod n'atteint pas la phase de planification ; il n'existe aucune ressource à supprimer avant de poursuivre.

## <a name="test-creation-of-an-unprivileged-pod"></a>Tester la création d’un pod non privilégié

Dans l’exemple précédent, l’image de conteneur tentait automatiquement d’utiliser la racine pour lier NGINX au port 80. Cette requête a été refusée par l’initiative de la stratégie de base, de sorte que le pod ne démarre pas. Essayons maintenant d’exécuter ce même pod NGINX sans la requête d’accès privilégié.

Créez un fichier nommé `nginx-unprivileged.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
```

Créez le pod à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f nginx-unprivileged.yaml
```

Le pod est planifié avec succès. Lorsque vous vérifiez l’état du pod à l’aide de la commande [kubectl get pods][kubectl-get], le pod est *en cours d’exécution*  :

```console
$ kubectl get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          18s
```

Cet exemple montre l’initiative de base qui affecte uniquement les déploiements violant les stratégies de la collection. Les déploiements autorisés continuent à fonctionner.

Supprimez le pod non privilégié NGINX à l’aide de la commande [kubectl delete][kubectl-delete] et spécifiez le nom de votre manifeste YAML :

```console
kubectl delete -f nginx-unprivileged.yaml
```

## <a name="disable-policies-and-the-azure-policy-add-on"></a>Désactivez les stratégies et le module complémentaire Azure Policy

Pour supprimer l’initiative de la ligne de base :

1. Accéder au volet de stratégie sur le Portail Azure
1. Sélectionnez **Attributions** dans le volet gauche
1. Cliquez sur le bouton « ... » en regard du profil de la ligne de base
1. Sélectionner « Supprimer l’attribution »

![Supprimer l'attribution](media/use-pod-security-on-azure-policy/delete-assignment.png)

Pour désactiver le module complémentaire Azure Policy, utilisez la commande [az aks disable-addons][az-aks-disable-addons].

```azurecli
az aks disable-addons --addons azure-policy --name MyAKSCluster --resource-group MyResourceGroup
```

Découvrez comment supprimer le module complémentaire [Azure Policy du Portail Azure](../governance/policy/concepts/policy-for-kubernetes.md#remove-the-add-on-from-aks).

## <a name="migrate-from-kubernetes-pod-security-policy-to-azure-policy"></a>Migrer à partir de la stratégie de sécurité des pods Kubernetes vers Azure Policy

Pour migrer à partir d’une stratégie de sécurité des pods, vous devez effectuer les actions suivantes sur un cluster.

1. [Désactivez la stratégie de sécurité des pods](use-pod-security-policies.md#clean-up-resources) sur le cluster
1. Installez le [module complémentaire Azure Policy][kubernetes-policy-reference]
1. Activez les stratégies Azure souhaitées à partir des [stratégies intégrées disponibles][policy-samples]

Vous trouverez ci-dessous un résumé des changements de comportement entre la stratégie de sécurité des pods et Azure Policy.

|Scénario| Stratégie de sécurité des pods | Azure Policy |
|---|---|---|
|Installation|Activez la fonctionnalité de stratégie de sécurité des pods |Installez le module complémentaire Azure Policy
|Déployer des stratégies| Déployer la ressource de stratégie de sécurité des pods| Affectez des stratégies Azure à l’étendue de l’abonnement ou du groupe de ressources. Le module complémentaire Azure Policy est requis pour les applications de ressources Kubernetes.
| Stratégies par défaut | Lorsque la stratégie de sécurité des pods est activée dans AKS, les stratégies privilégiées et non restreintes par défaut sont appliquées. | Après l’activation du module complémentaire Azure Policy, aucune stratégie n’est appliquée par défaut. Vous devez explicitement activer les stratégies dans Azure Policy.
| Qui peut créer et attribuer une stratégie | L’administrateur de cluster crée une ressource de stratégie de sécurité des pods | Les utilisateurs doivent avoir un rôle minimal d’autorisations « propriétaire » ou « contributeur de stratégie de ressource » sur le groupe de ressources du cluster AKS. - Via l’API, les utilisateurs peuvent affecter des stratégies dans l’étendue de la ressource de cluster AKS. Les utilisateurs doivent avoir un rôle minimal d’autorisations « propriétaire » ou « contributeur de stratégie de ressource » sur le groupe de ressources du cluster AKS. - Dans le portail Azure, les stratégies peuvent être appliquées au niveau du groupe d’administration, de l’abonnement ou du groupe de ressources.
| Autorisation des stratégies| Les utilisateurs et les comptes de service requièrent des autorisations explicites pour utiliser des stratégies de sécurité des pods. | Aucune affectation supplémentaire n’est requise pour autoriser des stratégies. Une fois les stratégies affectées dans Azure, tous les utilisateurs du cluster peuvent utiliser ces stratégies.
| Applicabilité de la stratégie | L’utilisateur administrateur ignore la mise en œuvre des stratégies de sécurité des pods. | Tous les utilisateurs (administrateur & non-administrateur) voient les mêmes stratégies. Il n’existe aucune casse particulière basée sur les utilisateurs. L’application de stratégie peut être exclue au niveau de l’espace de noms.
| Étendue de la stratégie | Les stratégies de sécurité des pods n'ont pas d'espace de noms | Les modèles de contrainte utilisés par Azure Policy n’ont pas d’espaces de noms.
| Action Refus/Audit/Mutation | Les stratégies de sécurité des pods prennent uniquement en charge les actions de refus. La mutation peut être effectuée avec des valeurs par défaut sur les demandes de création. La validation peut être effectuée pendant les demandes de mise à jour.| Azure Policy prend en charge les actions d’audit et de refus. La mutation n’est pas encore prise en charge, mais planifiée.
| Conformité à la stratégie de sécurité des pods | Il n’existe aucune visibilité sur la conformité des pods qui existaient avant l’activation de la stratégie de sécurité des pods. Les pods non conformes créés après l’activation des stratégies de sécurité des pods sont refusés. | Les pods non conformes qui existaient avant l’application des stratégies Azure s’affichent dans les violations de stratégie. Les pods non conformes créées après l’activation des stratégies Azure sont refusées si les stratégies sont définies avec un effet de refus.
| Comment afficher les stratégies sur le cluster | `kubectl get psp` | `kubectl get constrainttemplate` -Toutes les stratégies sont retournées.
| Norme de stratégie de sécurité des pods - Privilégiée | Une ressource de stratégie de sécurité des pods privilégiée est créée par défaut lors de l’activation de la fonctionnalité. | Le mode Privilégié n’implique aucune restriction, par conséquent, il équivaut à ne pas avoir d’affectation Azure Policy.
| [Norme de stratégie de sécurité des pods - Ligne de base/Par défaut](https://kubernetes.io/docs/concepts/security/pod-security-standards/#baseline-default) | L’utilisateur installe une ressource de ligne de base de stratégie de sécurité des pods. | Azure Policy fournit une [initiative de ligne de base intégrée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2Fa8640138-9b0a-4a28-b8cb-1666c838647d) qui correspond à la stratégie de sécurité des pods de base.
| [Norme de stratégie de sécurité des pods - Restreinte](https://kubernetes.io/docs/concepts/security/pod-security-standards/#restricted) | L’utilisateur installe une ressource de ligne de base pour la stratégie de sécurité des pods. | Azure Policy fournit une [initiative de ligne de base intégrée](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicySetDefinitions%2F42b8ef37-b724-4e24-bbc8-7a7708edfe00) qui correspond à la stratégie de sécurité des pods restreinte.

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment appliquer une stratégie Azure qui limite le déploiement des pods privilégiée pour empêcher l’utilisation de l’accès privilégié. De nombreuses stratégies peuvent être appliquées, telles que les stratégies qui limitent l’utilisation de volumes. Pour plus d’informations sur les options disponibles, consultez les [documents de référence Azure Policy pour Kubernetes][kubernetes-policy-reference].

Pour plus d’informations sur la limitation du trafic réseau des pods, consultez [Sécuriser le trafic entre les pods avec des stratégies réseau dans AKS][network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[aad-pod-identity]: https://github.com/Azure/aad-pod-identity
[aad-pod-identity-exception]: https://azure.github.io/aad-pod-identity/docs/configure/application_exception

<!-- LINKS - internal -->
[policy-recommendations]: ../governance/policy/concepts/policy-for-kubernetes.md
[policy-limitations]: ../governance/policy/concepts/policy-for-kubernetes.md?#limitations
[kubernetes-policy-reference]: ../governance/policy/concepts/policy-for-kubernetes.md
[policy-samples]: ./policy-reference.md#microsoftcontainerservice
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[network-policies]: use-network-policies.md
[az-feature-register]: /cli/azure/feature#az-feature-register
[az-feature-list]: /cli/azure/feature#az-feature-list
[az-provider-register]: /cli/azure/provider#az-provider-register
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[az-aks-update]: /cli/azure/ext/aks-preview/aks#ext-aks-preview-az-aks-update
[az-extension-add]: /cli/azure/extension#az-extension-add
[aks-support-policies]: support-policies.md
[aks-faq]: faq.md
[az-extension-add]: /cli/azure/extension#az-extension-add
[az-extension-update]: /cli/azure/extension#az-extension-update
[az-aks-disable-addons]: /cli/azure/aks#az-aks-disable-addons
[az-policy-assignment-delete]: /cli/azure/policy/assignment#az-policy-assignment-delete
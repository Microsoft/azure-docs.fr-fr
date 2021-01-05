---
title: Utiliser des stratégies de sécurité des pods dans Azure Kubernetes Service (AKS)
description: Apprendre à contrôler les admissions pod à l’aide de PodSecurityPolicy dans Azure Kubernetes Service (AKS)
services: container-service
ms.topic: article
ms.date: 07/21/2020
ms.openlocfilehash: 77c618429503caf9aa7bb6abda109504bbf68d71
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695994"
---
# <a name="preview---secure-your-cluster-using-pod-security-policies-in-azure-kubernetes-service-aks"></a>Aperçu - Sécuriser votre cluster à l’aide de stratégies de sécurité des pods dans Azure Kubernetes Service (AKS)

> [!WARNING]
> **La fonctionnalité décrite dans ce document, Stratégie de sécurité des pods (préversion), sera bientôt dépréciée et ne sera plus disponible après le 31 mai 2021** ; elle sera remplacée par [Azure Policy pour AKS](use-pod-security-on-azure-policy.md). La date de dépréciation a été repoussée (il s’agit initialement du 15 octobre 2020).
>
> Une fois que la stratégie de sécurité des pods (préversion) sera déconseillée, vous devrez désactiver la fonctionnalité sur tous les clusters existants à l’aide de la fonctionnalité déconseillée pour effectuer les futures mises à niveau de cluster et continuer à bénéficier du support Azure.
>
> Il est vivement recommandé de commencer à tester des scénarios avec Azure Policy pour AKS, qui propose des stratégies intégrées permettant de sécuriser les pods et des initiatives intégrées qui correspondent aux stratégies de sécurité des pods. Cliquez ici pour en savoir plus sur la [migration vers Azure Policy à partir de la stratégie de sécurité des pods (préversion)](use-pod-security-on-azure-policy.md#migrate-from-kubernetes-pod-security-policy-to-azure-policy).

Pour améliorer la sécurité de votre cluster AKS, vous pouvez limiter les pods pouvant être planifiés. Les pods qui demandent des ressources non autorisées ne sont pas exécutés dans le cluster AKS. Vous définissez cet accès à l’aide de stratégies de sécurité des pods. Cet article explique comment utiliser des stratégies de sécurité des pods pour limiter le déploiement de pods dans AKS.

[!INCLUDE [preview features callout](./includes/preview/preview-callout.md)]

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

L’interface de ligne de commande Azure (Azure CLI) version 2.0.61 ou une version ultérieure doit avoir été installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

### <a name="install-aks-preview-cli-extension"></a>Installer l’extension CLI de préversion d’aks

Pour utiliser des stratégies de sécurité des pods, vous aurez besoin de l’extension de CLI *aks-preview* version 0.4.1 ou version ultérieure. Installez l’extension Azure CLI *aks-preview* à l’aide de la commande [az extension add][az-extension-add], puis recherchez toutes les mises à jour disponibles à l’aide de la commande [az extension update][az-extension-update] :

```azurecli-interactive
# Install the aks-preview extension
az extension add --name aks-preview

# Update the extension to make sure you have the latest version installed
az extension update --name aks-preview
```

### <a name="register-pod-security-policy-feature-provider"></a>Inscrire le fournisseur de fonctionnalités de stratégie de sécurité des pods

**Ce document et cette fonctionnalité seront déconseillés à partir du 15 octobre 2020.**

Pour créer ou mettre à jour un cluster AKS afin d’utiliser des stratégies de sécurité des pods, commencez par activer un indicateur de fonctionnalité sur votre abonnement. Pour enregistrer l’indicateur de fonctionnalité *PodSecurityPolicyPreview*, utilisez la commande [az feature register][az-feature-register], comme indiqué dans l’exemple suivant :

```azurecli-interactive
az feature register --name PodSecurityPolicyPreview --namespace Microsoft.ContainerService
```

Quelques minutes sont nécessaires pour que l’état s’affiche *Registered* (Inscrit). Vous pouvez vérifier l’état de l’enregistrement à l’aide de la commande [az feature list][az-feature-list] :

```azurecli-interactive
az feature list -o table --query "[?contains(name, 'Microsoft.ContainerService/PodSecurityPolicyPreview')].{Name:name,State:properties.state}"
```

Lorsque vous êtes prêt, actualisez l’inscription du fournisseur de ressources *Microsoft.ContainerService* à l’aide de la commande [az provider register][az-provider-register] :

```azurecli-interactive
az provider register --namespace Microsoft.ContainerService
```

## <a name="overview-of-pod-security-policies"></a>Présentation des stratégies de sécurité des pods

Dans un cluster Kubernetes, un contrôleur d’admission est utilisé pour intercepter les requêtes envoyées au serveur d’API lorsqu’une ressource doit être créée. Le contrôleur d’admission peut ensuite *valider* la requête de ressource par rapport à un ensemble de règles, ou *muter* la ressource à modifier les paramètres de déploiement.

*PodSecurityPolicy* est un contrôleur d’admission qui confirme si une spécification de pod répond à vos besoins définis. Ces exigences peuvent limiter l’utilisation de conteneurs privilégiés, l’accès à certains types de stockage, ou l’utilisateur/groupe sous lequel le conteneur peut s’exécuter. Lorsque vous tentez de déployer une ressource où les spécifications de pod ne répondent pas aux exigences décrites dans la stratégie de sécurité des pods, la requête est refusée. Ce contrôle sur les pods pouvant être planifiés dans le cluster AKS empêche d’éventuelles vulnérabilités de sécurité ou escalades de privilèges.

Lorsque vous activez la stratégie de sécurité des pods dans un cluster AKS, certaines stratégies par défaut sont appliquées. Ces stratégies par défaut fournissent une expérience prête à l’emploi pour définir les pods à planifier. Toutefois, les utilisateurs de cluster peuvent rencontrer des problèmes de déploiement de pods jusqu'à ce que vous définissez vos propres stratégies. L'approche recommandée consiste à :

* Créer un cluster AKS
* Définir vos propres stratégies de sécurité des pods
* Activer la fonctionnalité de stratégie de sécurité des pods

Pour montrer comment les stratégies par défaut limitent les déploiements de pods, dans cet article, nous allons tout d’abord activer la fonctionnalité de stratégies de sécurité des pods, puis créer une stratégie personnalisée.

## <a name="enable-pod-security-policy-on-an-aks-cluster"></a>Activer la stratégie de sécurité des pods sur un cluster AKS

Vous pouvez activer ou désactiver la stratégie de sécurité des pods à l’aide de la commande [az aks update][az-aks-update]. L’exemple suivant active la stratégie de sécurité des pods sur le nom de cluster *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup*.

> [!NOTE]
> Pour une utilisation en conditions réelles, n’activez pas la stratégie de sécurité des pods tant que vous n’avez pas défini vos propres stratégies personnalisées. Dans cet article, vous activez la stratégie de sécurité des pods en tant que première étape pour voir comment les stratégies par défaut limitent les déploiements de pods.

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --enable-pod-security-policy
```

## <a name="default-aks-policies"></a>Stratégies AKS par défaut

Lorsque vous activez la stratégie de sécurité des pods, AKS crée une seule stratégie par défaut nommée *privileged*. Ne modifiez pas et ne supprimez pas cette stratégie par défaut. Créez plutôt vos propres stratégies qui définissent les paramètres souhaités pour le contrôle. Nous allons examiner ces stratégies par défaut et observer leur impact sur les déploiements de pods.

Pour afficher les stratégies disponibles, utilisez la commande [kubectl get psp][kubectl-get], comme indiqué dans l’exemple suivant

```console
$ kubectl get psp

NAME         PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged   true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *     configMap,emptyDir,projected,secret,downwardAPI,persistentVolumeClaim
```

La stratégie de sécurité des pods *privileged* est appliquée à tout utilisateur authentifié dans le cluster AKS. Cette affectation est contrôlée par ClusterRoles et ClusterRoleBindings. Utilisez la commande [kubectl get rolebindings][kubectl-get] et recherchez la liaison *default:privileged* dans l’espace de noms *kube-system* :

```console
kubectl get rolebindings default:privileged -n kube-system -o yaml
```

Comme indiqué dans la sortie condensée suivante, le ClusterRole *psp:privileged* est attribué à tous les utilisateurs *system:authenticated*. Cette capacité offre un niveau de base pour les privilèges sans avoir à définir vos propres stratégies.

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  [...]
  name: default:privileged
  [...]
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp:privileged
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:masters
```

Il est important de comprendre comment ces stratégies par défaut interagissent avec les requêtes des utilisateurs pour planifier les pods avant de commencer à créer vos propres stratégies de sécurité des pods. Dans les sections suivantes, nous allons planifier des pods pour voir ces stratégies par défaut en action.

## <a name="create-a-test-user-in-an-aks-cluster"></a>Créer un utilisateur de test dans un cluster AKS

Par défaut, lorsque vous utilisez la commande [az aks get-credentials][az-aks-get-credentials], les informations d'identification *administrateur* du cluster AKS sont ajoutées à votre configuration `kubectl`. L’utilisateur administrateur ignore la mise en œuvre des stratégies de sécurité des pods. Si vous utilisez l’intégration Azure Active Directory pour vos clusters AKS, vous pouvez vous connecter avec les informations d’identification d’un utilisateur non administrateur pour voir la mise en œuvre des stratégies en action. Dans cet article, nous allons créer un compte d’utilisateur de test dans le cluster AKS que vous pouvez utiliser.

Créer un espace de noms exemple nommé *psp-aks* pour les ressources de test à l’aide de la commande [kubectl create namespace][kubectl-create]. Ensuite, créez un compte de service nommé *nonadmin-user* à l’aide de la commande [kubectl create serviceaccount][kubectl-create] :

```console
kubectl create namespace psp-aks
kubectl create serviceaccount --namespace psp-aks nonadmin-user
```

Créez un RoleBinding pour *nonadmin-user* afin d’effectuer des actions de base dans l’espace de noms à l’aide de la commande  [kubectl create rolebinding][kubectl-create] :

```console
kubectl create rolebinding \
    --namespace psp-aks \
    psp-aks-editor \
    --clusterrole=edit \
    --serviceaccount=psp-aks:nonadmin-user
```

### <a name="create-alias-commands-for-admin-and-non-admin-user"></a>Créer des commandes d’alias pour l’utilisateur administrateur et non-administrateur

Pour mettre en évidence la différence entre l’utilisateur administrateur régulier lors de l’utilisation de `kubectl` et l’utilisateur non administrateur créé lors des étapes précédentes, créez deux alias de ligne de commande :

* L’alias **kubectl-admin** est destiné à l’utilisateur administrateur régulier et limité à l’espace de noms *psp-aks*.
* L’alias **kubectl-nonadminuser** est destiné à *nonadmin-user* créé à l’étape précédente et limité à l’espace de noms *psp-aks*.

Créez ces deux alias comme indiqué dans les commandes suivantes :

```console
alias kubectl-admin='kubectl --namespace psp-aks'
alias kubectl-nonadminuser='kubectl --as=system:serviceaccount:psp-aks:nonadmin-user --namespace psp-aks'
```

## <a name="test-the-creation-of-a-privileged-pod"></a>Tester la création d’un pod privilégié

Nous allons commencer par tester ce qui se passe lorsque vous planifiez un pod avec le contexte de sécurité `privileged: true`. Ce contexte de sécurité fait remonter les privilèges du pod. Dans la section précédente qui montrait les stratégies de sécurité des pods par défaut d’AKS, la stratégie *privilege* devait refuser cette requête.

Créez un fichier nommé `nginx-privileged.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-privileged
spec:
  containers:
    - name: nginx-privileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        privileged: true
```

Créez le pod à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl-nonadminuser apply -f nginx-privileged.yaml
```

La planification de la commande échoue, comme indiqué dans l’exemple de sortie suivant :

```console
$ kubectl-nonadminuser apply -f nginx-privileged.yaml

Error from server (Forbidden): error when creating "nginx-privileged.yaml": pods "nginx-privileged" is forbidden: unable to validate against any pod security policy: []
```

Le pod n'atteint pas la phase de planification ; il n'existe aucune ressource à supprimer avant de poursuivre.

## <a name="test-creation-of-an-unprivileged-pod"></a>Tester la création d’un pod non privilégié

Dans l’exemple précédent, la spécification de pod a demandé une élévation des privilèges. Cette requête est refusée par la stratégie de sécurité des pods *privilege* par défaut. Il est donc impossible de planifier le pod. Essayons maintenant d’exécuter ce même pod NGINX sans la requête d’élévation des privilèges.

Créez un fichier nommé `nginx-unprivileged.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
```

Créez le pod à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

La planification de la commande échoue, comme indiqué dans l’exemple de sortie suivant :

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged.yaml": pods "nginx-unprivileged" is forbidden: unable to validate against any pod security policy: []
```

Le pod n'atteint pas la phase de planification ; il n'existe aucune ressource à supprimer avant de poursuivre.

## <a name="test-creation-of-a-pod-with-a-specific-user-context"></a>Tester la création d’un pod avec un contexte utilisateur spécifique

Dans l’exemple précédent, l’image de conteneur tentait automatiquement d’utiliser la racine pour lier NGINX au port 80. Cette requête était refusée par la stratégie de sécurité des pods *privilege* par défaut. Il était donc impossible de planifier le pod. Nous allons essayer maintenant d’exécuter ce même pod NGINX avec un contexte utilisateur spécifique, tel que `runAsUser: 2000`.

Créez un fichier nommé `nginx-unprivileged-nonroot.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-unprivileged-nonroot
spec:
  containers:
    - name: nginx-unprivileged
      image: mcr.microsoft.com/oss/nginx/nginx:1.14.2-alpine
      securityContext:
        runAsUser: 2000
```

Créez le pod à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml
```

La planification de la commande échoue, comme indiqué dans l’exemple de sortie suivant :

```console
$ kubectl-nonadminuser apply -f nginx-unprivileged-nonroot.yaml

Error from server (Forbidden): error when creating "nginx-unprivileged-nonroot.yaml": pods "nginx-unprivileged-nonroot" is forbidden: unable to validate against any pod security policy: []
```

Le pod n'atteint pas la phase de planification ; il n'existe aucune ressource à supprimer avant de poursuivre.

## <a name="create-a-custom-pod-security-policy"></a>Créer une stratégie de sécurité des pods personnalisée

Maintenant que vous avez vu le comportement des stratégies de sécurité des pods par défaut, donnons les moyens à *nonadmin-user* de planifier correctement des pods.

Nous allons créer une stratégie pour rejeter les pods qui demandent un accès privilégié. Les autres options, telles que *runAsUser* ou les *volumes* autorisés, ne sont pas explicitement limitées. Ce type de stratégie refuse une requête d’accès privilégié, mais elle permet néanmoins au cluster d’exécuter les pods demandés.

Créez un fichier nommé `psp-deny-privileged.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: psp-deny-privileged
spec:
  privileged: false
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
  - '*'
```

Créez la stratégie à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f psp-deny-privileged.yaml
```

Pour afficher les stratégies disponibles, utilisez la commande [kubectl get psp][kubectl-get], comme indiqué dans l’exemple suivant. Comparez la stratégie *psp-deny-privileged* à la stratégie *privilege* par défaut appliquée dans les exemples précédents de création de pod. Seule l’utilisation de l’escalade *PRIV* est refusée par votre stratégie. Il n’existe aucune restriction sur l’utilisateur ou le groupe pour la stratégie *psp-deny-privileged*.

```console
$ kubectl get psp

NAME                  PRIV    CAPS   SELINUX    RUNASUSER          FSGROUP     SUPGROUP    READONLYROOTFS   VOLUMES
privileged            true    *      RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *
psp-deny-privileged   false          RunAsAny   RunAsAny           RunAsAny    RunAsAny    false            *          
```

## <a name="allow-user-account-to-use-the-custom-pod-security-policy"></a>Autoriser le compte d’utilisateur à utiliser la stratégie de sécurité des pods personnalisée

À l’étape précédente, vous avez créé une stratégie de sécurité des pods qui refuse les pods demandant un accès privilégié. Pour permettre l’utilisation de la stratégie, vous créez un *rôle* ou un *ClusterRole*. Ensuite, vous associez un de ces rôles à l’aide de *RoleBinding* ou *ClusterRoleBinding*.

Pour cet exemple, créez un ClusterRole qui vous permet d’*utiliser* la stratégie *psp-deny-privileged* créée à l’étape précédente. Créez un fichier nommé `psp-deny-privileged-clusterrole.yaml` et collez le manifeste YAML suivant :

```yaml
kind: ClusterRole
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: psp-deny-privileged-clusterrole
rules:
- apiGroups:
  - extensions
  resources:
  - podsecuritypolicies
  resourceNames:
  - psp-deny-privileged
  verbs:
  - use
```

Créez le ClusterRole à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f psp-deny-privileged-clusterrole.yaml
```

Créez maintenant un ClusterRoleBinding pour utiliser le ClusterRole créé à l’étape précédente. Créez un fichier nommé `psp-deny-privileged-clusterrolebinding.yaml` et collez le manifeste YAML suivant :

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: psp-deny-privileged-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: psp-deny-privileged-clusterrole
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: system:serviceaccounts
```

Créez un ClusterRoleBinding à l’aide de la commande [kubectl apply][kubectl-apply] et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f psp-deny-privileged-clusterrolebinding.yaml
```

> [!NOTE]
> À la première étape de cet article, la fonctionnalité de stratégie de sécurité des pods a été activée sur le cluster AKS. La pratique recommandée consistait à activer uniquement la fonctionnalité de stratégie de sécurité des pods après avoir défini vos propres stratégies. Il s’agit du stade où vous devez activer la fonctionnalité de stratégie de sécurité des pods. Au moins une stratégie personnalisée a été définie, et les comptes d’utilisateur ont été associés à ces stratégies. Désormais, vous pouvez activer en toute sécurité la fonctionnalité de stratégie de sécurité des pods et réduire les problèmes causés par les stratégies par défaut.

## <a name="test-the-creation-of-an-unprivileged-pod-again"></a>Retester la création d’un pod non privilégié

Avec votre stratégie de sécurité des pods personnalisée appliquée et une liaison permettant au compte d’utilisateur d’utiliser la stratégie, nous allons essayer à nouveau de créer un pod non privilégié. Utilisez le même manifeste `nginx-privileged.yaml` pour créer le pod à l’aide de la commande [kubectl apply][kubectl-apply] :

```console
kubectl-nonadminuser apply -f nginx-unprivileged.yaml
```

Le pod est planifié avec succès. Lorsque vous vérifiez l’état du pod à l’aide de la commande [kubectl get pods][kubectl-get], le pod est *en cours d’exécution* :

```
$ kubectl-nonadminuser get pods

NAME                 READY   STATUS    RESTARTS   AGE
nginx-unprivileged   1/1     Running   0          7m14s
```

Cet exemple montre comment vous pouvez créer des stratégies de sécurité des pods personnalisées pour définir l’accès au cluster AKS pour différents utilisateurs ou groupes. Les stratégies AKS par défaut fournissent un contrôle strict quant aux pods autorisés à exécuter. Par conséquent, créez vos propres stratégies personnalisées, puis définissez correctement les restrictions dont vous avez besoin.

Supprimez le pod non privilégié NGINX à l’aide de la commande [kubectl delete][kubectl-delete] et spécifiez le nom de votre manifeste YAML :

```console
kubectl-nonadminuser delete -f nginx-unprivileged.yaml
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour désactiver la stratégie de sécurité des pods, utilisez la commande [az aks update][az-aks-update] à nouveau. L’exemple suivant désactive la stratégie de sécurité des pods sur le nom de cluster *myAKSCluster* dans le groupe de ressources nommé *myResourceGroup* :

```azurecli-interactive
az aks update \
    --resource-group myResourceGroup \
    --name myAKSCluster \
    --disable-pod-security-policy
```

Ensuite, supprimez ClusterRole et ClusterRoleBinding :

```console
kubectl delete -f psp-deny-privileged-clusterrolebinding.yaml
kubectl delete -f psp-deny-privileged-clusterrole.yaml
```

Supprimez la stratégie de sécurité en utilisant la commande [kubectl delete][kubectl-delete] et spécifiez le nom de votre manifeste YAML :

```console
kubectl delete -f psp-deny-privileged.yaml
```

Enfin, supprimez l’espace de noms *psp-aks* :

```console
kubectl delete namespace psp-aks
```

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a montré comment créer une stratégie de sécurité des pods pour empêcher l’utilisation de l’accès privilégié. Il existe un grand nombre de fonctionnalités qu’une stratégie peut appliquer, comme le type de volume ou l’utilisateur RunAs. Pour plus d’informations sur les options disponibles, consultez les [documents de référence sur les stratégies de sécurité des pods Kubernetes][kubernetes-policy-reference].

Pour plus d’informations sur la limitation du trafic réseau des pods, consultez [Sécuriser le trafic entre les pods avec des stratégies réseau dans AKS][network-policies].

<!-- LINKS - external -->
[kubectl-apply]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply
[kubectl-delete]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#delete
[kubectl-get]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get
[kubectl-create]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#create
[kubectl-describe]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#describe
[kubectl-logs]: https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#logs
[terms-of-use]: https://azure.microsoft.com/support/legal/preview-supplemental-terms/
[kubernetes-policy-reference]: https://kubernetes.io/docs/concepts/policy/pod-security-policy/#policy-reference

<!-- LINKS - internal -->
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

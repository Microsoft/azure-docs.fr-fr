---
title: Découvrir Azure Policy pour Kubernetes
description: Découvrez comment Azure Policy utilise Rego et Open Policy Agent pour gérer des clusters exécutant Kubernetes dans Azure ou localement.
ms.date: 12/01/2020
ms.topic: conceptual
ms.openlocfilehash: e2b9253d8ce60d5dc77d406e3c9d0469539f2c77
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96511329"
---
# <a name="understand-azure-policy-for-kubernetes-clusters"></a>Comprendre Azure Policy pour les clusters Kubernetes

Azure Policy étend [Gatekeeper](https://github.com/open-policy-agent/gatekeeper) v3, un _webhook contrôleur d’admission_ pour [Open Policy Agent](https://www.openpolicyagent.org/) (OPA), afin d’appliquer des mises en œuvre et protections à grande échelle à vos clusters d’une manière centralisée et cohérente. Azure Policy vous permet de gérer vos clusters Kubernetes et de générer des rapports sur leur état de conformité à partir d’un seul emplacement. Le module complémentaire implémente les fonctions suivantes :

- Il vérifie auprès du service Azure Policy les affectations de stratégies au cluster.
- Il déploie des définitions de stratégies dans le cluster en tant que ressources personnalisées [modèle de contrainte](https://github.com/open-policy-agent/gatekeeper#constraint-templates) et [contrainte](https://github.com/open-policy-agent/gatekeeper#constraints).
- Il fournit des rapports d’audit et de conformité au service Azure Policy.

Azure Policy pour Kubernetes prend en charge les environnements de cluster suivants :

- [Azure Kubernetes Service (AKS)](../../../aks/intro-kubernetes.md)
- [Kubernetes compatible avec Azure Arc](../../../azure-arc/kubernetes/overview.md)
- [Moteur AKS](https://github.com/Azure/aks-engine/blob/master/docs/README.md)

> [!IMPORTANT]
> Les modules complémentaires pour le moteur AKS et les instances Kubernetes compatibles avec Arc sont en **préversion**. Azure Policy pour Kubernetes prend uniquement en charge les pools de nœuds Linux et les définitions de stratégie intégrées. Les définitions de stratégie intégrée se trouvent dans la catégorie **Kubernetes**. Les définitions de stratégie limitées en version préversion avec les effets **EnforceOPAConstraint** et **EnforceRegoPolicy**, ainsi que la catégorie **Service Kubernetes** associée sont _déconseillées_. À la place, utilisez les effets _audit_ et _deny_ avec le mode Fournisseur de ressources `Microsoft.Kubernetes.Data`.

## <a name="overview"></a>Vue d’ensemble

Pour activer et utiliser Azure Policy avec votre cluster Kubernetes, procédez comme suit :

1. Configurez votre cluster Kubernetes et installez le module complémentaire :
   - [Azure Kubernetes Service (AKS)](#install-azure-policy-add-on-for-aks)
   - [Kubernetes compatible avec Azure Arc](#install-azure-policy-add-on-for-azure-arc-enabled-kubernetes)
   - [Moteur AKS](#install-azure-policy-add-on-for-aks-engine)

   > [!NOTE]
   > Pour les problèmes courants liés à l’installation, consultez [Résolution des problèmes : module complémentaire Azure Policy](../troubleshoot/general.md#add-on-for-kubernetes-installation-errors).

1. [Comprendre le langage d’Azure Policy pour Kubernetes](#policy-language)

1. [Affecter une définition intégrée à votre cluster Kubernetes](#assign-a-built-in-policy-definition)

1. [Attendre la validation](#policy-evaluation)

## <a name="limitations"></a>Limites

Les limitations générales suivantes s’appliquent au module complémentaire Azure Policy pour les clusters Kubernetes :

- Le module complémentaire Azure Policy pour Kubernetes est pris en charge sur la version **1.14** ou ultérieure de Kubernetes.
- Le module complémentaire Azure Policy pour Kubernetes peut uniquement être déployé dans des pools de nœuds Linux
- Seules les définitions de stratégie intégrées sont prises en charge
- Nombre maximal d’enregistrements non conformes par stratégie par cluster : **500**
- Nombre maximal d’enregistrements non conformes par abonnement : **1 million**
- Les installations de Gatekeeper en dehors du module complémentaire Azure Policy ne sont pas prises en charge. Désinstallez tous les composants installés par une installation antérieure de Gatekeeper avant d’activer le module complémentaire Azure Policy.
- Les [Raisons de non-conformité](../how-to/determine-non-compliance.md#compliance-reasons) ne sont pas disponibles pour le `Microsoft.Kubernetes.Data`
  [mode Fournisseur de ressources](./definition-structure.md#resource-provider-modes). Utilisez [Détails du composant](../how-to/determine-non-compliance.md#component-details-for-resource-provider-modes).
- Les [Exemptions](./exemption-structure.md) ne sont pas prises en charge pour les [modes Fournisseur de ressources](./definition-structure.md#resource-provider-modes).

Les limitations suivantes s’appliquent uniquement au module complémentaire Azure Policy pour AKS :

- La [stratégie de sécurité des pods AKS](../../../aks/use-pod-security-policies.md) et le module complémentaire Azure Policy pour AKS ne peuvent pas être activés simultanément. Pour plus d’informations, consultez [Limitation de la sécurité des pods AKS](../../../aks/use-pod-security-on-azure-policy.md#limitations).
- Espaces de noms automatiquement exclus par le module complémentaire Azure Policy à des fins d’évaluation : _kube-system_, _gatekeeper-system_ et _aks-periscope_.

## <a name="recommendations"></a>Recommandations

Voici les recommandations générales relatives à l’utilisation du module complémentaire Azure Policy :

- Le module complémentaire Azure Policy requiert 3 composants Gatekeeper pour s’exécuter : 1 pod d’audit et 2 réplicas de pod de webhook. Ces composants consomment davantage de ressources, car le nombre de ressources Kubernetes et d’attributions de stratégie augmente dans le cluster, ce qui nécessite des opérations d’audit et de mise en œuvre.

  - Pour moins de 500 pods dans un seul cluster avec un maximum de 20 contraintes : 2 processeurs virtuels et 350 Mo de mémoire par composant.
  - Pour plus de 500 pods dans un seul cluster avec un maximum de 40 contraintes : 3 processeurs virtuels et 600 Mo de mémoire par composant.

- Les pods Windows [ne prennent pas en charge les contextes de sécurité](https://kubernetes.io/docs/concepts/security/pod-security-standards/#what-profiles-should-i-apply-to-my-windows-pods).
  Ainsi, certaines des définitions Azure Policy, telles que la désactivation des privilèges racine, ne peuvent pas être réaffectées dans des pods Windows et s’appliquent uniquement à des pods Linux.

La recommandation suivante s’applique uniquement à AKS et au module complémentaire Azure Policy :

- Utilisez le pool de nœuds système avec la teinte `CriticalAddonsOnly` pour planifier des pods Gatekeeper. Pour plus d’informations, consultez [Utilisation de pools de nœuds système](../../../aks/use-system-pools.md#system-and-user-node-pools).
- Sécurisez le trafic sortant de vos clusters AKS. Pour plus d’informations, consultez [Contrôle du trafic de sortie pour les nœuds de cluster](../../../aks/limit-egress-traffic.md).
- Si `aad-pod-identity` est activé dans le cluster, les pods NMI (Node Managed Identity) modifient les tables d’adresses IP des nœuds pour intercepter les appels vers le point de terminaison Azure Instance Metadata. Cette configuration signifie que toutes les requêtes adressées au point de terminaison Metadata sont interceptées par NMI, même si le pod n’utilise pas `aad-pod-identity`. La CRD AzurePodIdentityException peut être configurée de manière à informer `aad-pod-identity` que toutes les requêtes adressées au point de terminaison Metadata depuis un pod correspondant aux étiquettes définies dans la CRD doivent être envoyées par proxy sans aucun traitement dans NMI. Les pods système qui disposent de l’étiquette `kubernetes.azure.com/managedby: aks` dans l’espace de noms _kube-system_ doivent être exclus de `aad-pod-identity` en configurant la CRD AzurePodIdentityException. Pour plus d’informations, consultez [Désactiver aad-pod-identity pour un pod ou une application spécifique](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).
  Pour configurer une exception, installez le fichier [YAML mic-exception](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml).

## <a name="install-azure-policy-add-on-for-aks"></a>Installer un module complémentaire Azure Policy pour AKS

Avant d’installer le module complémentaire Azure Policy ou d’activer des fonctionnalités du service, votre abonnement doit activer les fournisseurs de ressources **Microsoft.ContainerService** and **Microsoft.PolicyInsights**.

1. La version 2.12.0 ou ultérieure d’Azure CLI doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Inscrivez les fournisseurs de ressources et les fonctionnalités en préversion.

   - Portail Azure :

     Inscrivez les fournisseurs de ressources **Microsoft.ContainerService** et **Microsoft.PolicyInsights**. Pour connaître les étapes, consultez [Types et fournisseurs de ressources](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).

   - Azure CLI :

     ```azurecli-interactive
     # Log in first with az login if you're not using Cloud Shell

     # Provider register: Register the Azure Kubernetes Service provider
     az provider register --namespace Microsoft.ContainerService

     # Provider register: Register the Azure Policy provider
     az provider register --namespace Microsoft.PolicyInsights
     ```

1. Si des définitions de stratégies en préversion étaient installées, supprimez le module complémentaire à l’aide du bouton **Désactiver** sur votre cluster AKS dans la page **Stratégies**.

1. Le cluster AKS doit être de version _1.14_ ou ultérieure. Utilisez le script suivant pour valider la version de votre cluster AKS :

   ```azurecli-interactive
   # Log in first with az login if you're not using Cloud Shell

   # Look for the value in kubernetesVersion
   az aks list
   ```

1. Installez la version _2.12.0_ ou une version ultérieure d’Azure CLI. Pour plus d’informations, consultez la rubrique [Installation de l’interface de ligne de commande Azure (CLI)](/cli/azure/install-azure-cli).

Une fois les étapes préalables ci-dessus accomplies, installez le module complémentaire Azure Policy dans le cluster AKS que vous souhaitez gérer.

- Portail Azure

  1. Lancez le service AKS dans le portail Azure en sélectionnant **Tous les services**, puis en recherchant et en sélectionnant **Services Kubernetes**.

  1. Sélectionnez l’un de vos clusters AKS.

  1. Sélectionnez **Stratégies** sur le côté gauche de la page du service Kubernetes.

  1. Dans la page principale, sélectionnez le bouton **Activer un module complémentaire**.

     <a name="migrate-from-v1"></a>
     > [!NOTE]
     > Si le bouton **Désactiver un module complémentaire** est activé et qu’un message d’avertissement de migration v2 s’affiche, cela signifie que le module complémentaire v1 est installé et doit être supprimé avant d’attribuer des définitions de stratégie v2. Le module complémentaire v1 _obsolète_ sera automatiquement remplacé par le module complémentaire v2 à compter du 24 août
     > 2020. Les nouvelles versions v2 des définitions de stratégie doivent ensuite être affectées. Pour effectuer la mise à niveau maintenant, procédez comme suit :
     >
     > 1. Vérifiez que le module complémentaire v1 est installé sur votre cluster AKS en visitant la page **Stratégies** sur votre cluster AKS et que le cluster AKS affiche le message « The current cluster uses Azure Policy add-on v1... » (Le cluster actuel utilise le module complémentaire v1 Azure Policy).
     > 1. [Supprimez le module complémentaire](#remove-the-add-on-from-aks).
     > 1. Cliquez sur le bouton **Activer l’extension** pour installer la version v2 du module complémentaire.
     > 1. [Affecter des versions v2 de vos définitions de stratégie intégrées v1](#assign-a-built-in-policy-definition)

- Azure CLI

  ```azurecli-interactive
  # Log in first with az login if you're not using Cloud Shell

  az aks enable-addons --addons azure-policy --name MyAKSCluster --resource-group MyResourceGroup
  ```

Pour vérifier que l’installation du module complémentaire a réussi et que les pods _azure-policy_ et _gatekeeper_ sont en cours d’exécution, exécutez la commande suivante :

```bash
# azure-policy pod is installed in kube-system namespace
kubectl get pods -n kube-system

# gatekeeper pod is installed in gatekeeper-system namespace
kubectl get pods -n gatekeeper-system
```

Enfin, vérifiez que le module complémentaire le plus récent est installé en exécutant cette commande Azure CLI, en remplaçant `<rg>` par le nom de votre groupe de ressources et `<cluster-name>` par le nom de votre cluster AKS : `az aks show --query addonProfiles.azurepolicy -g <rg> -n <cluster-name>`. Le résultat doit ressembler à la sortie suivante et la valeur de **config.version** doit être `v2` :

```output
"addonProfiles": {
    "azurepolicy": {
        "config": {
            "version": "v2"
        },
        "enabled": true,
        "identity": null
    },
}
```

## <a name="install-azure-policy-add-on-for-azure-arc-enabled-kubernetes-preview"></a><a name="install-azure-policy-add-on-for-azure-arc-enabled-kubernetes"></a>Installer le module complémentaire Azure Policy pour Kubernetes compatible avec Azure Arc (préversion)

Avant d’installer le module complémentaire Azure Policy ou d’activer des fonctionnalités du service, vous devez activer le fournisseur de ressources **Microsoft.PolicyInsights** et créer une attribution de rôle pour le principal de service du cluster dans votre abonnement.

1. La version 2.12.0 ou ultérieure d’Azure CLI doit être installée et configurée. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Pour activer le fournisseur de ressources, suivez les étapes décrites dans [Fournisseurs de ressources et types](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal) ou exécutez la commande Azure CLI ou Azure PowerShell :

   - Azure CLI

     ```azurecli-interactive
     # Log in first with az login if you're not using Cloud Shell

     # Provider register: Register the Azure Policy provider
     az provider register --namespace 'Microsoft.PolicyInsights'
     ```

   - Azure PowerShell

     ```azurepowershell-interactive
     # Log in first with Connect-AzAccount if you're not using Cloud Shell

     # Provider register: Register the Azure Policy provider
     Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
     ```

1. Le cluster Kubernetes doit être de la version _1.14_ ou ultérieure.

1. Installez [Helm 3](https://v3.helm.sh/docs/intro/install/).

1. Votre cluster Kubernetes activé pour Azure Arc. Pour plus d’informations, consultez [Intégration d’un cluster Kubernetes à Azure Arc](../../../azure-arc/kubernetes/connect-cluster.md).

1. Munissez-vous de l’ID de ressource Azure complet du cluster Kubernetes compatible avec Azure Arc.

1. Ouvrez des ports pour le module complémentaire. Le module complémentaire Azure Policy utilise ces domaines et ports pour extraire des définitions et affectations de stratégie, et rendre compte de la conformité du cluster à Azure Policy.

   |Domain |Port |
   |---|---|
   |`gov-prod-policy-data.trafficmanager.net` |`443` |
   |`raw.githubusercontent.com` |`443` |
   |`login.windows.net` |`443` |
   |`dc.services.visualstudio.com` |`443` |

1. Attribuez le rôle « Policy Insights Data Writer (préversion) » au cluster Kubernetes compatible avec Azure Arc. Remplacez `<subscriptionId>` par votre ID d’abonnement, `<rg>` par le groupe de ressources du cluster Kubernetes compatible avec Azure Arc, et `<clusterName>` par le nom du cluster Kubernetes compatible avec Azure Arc. Effectuez le suivi des valeurs retournées pour _appId_, _password_ et _tenant_ pour les étapes d’installation.

   - Azure CLI

     ```azurecli-interactive
     az ad sp create-for-rbac --role "Policy Insights Data Writer (Preview)" --scopes "/subscriptions/<subscriptionId>/resourceGroups/<rg>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"
     ```

   - Azure PowerShell

     ```azurepowershell-interactive
     $sp = New-AzADServicePrincipal -Role "Policy Insights Data Writer (Preview)" -Scope "/subscriptions/<subscriptionId>/resourceGroups/<rg>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>"

     @{ appId=$sp.ApplicationId;password=[System.Runtime.InteropServices.Marshal]::PtrToStringAuto([System.Runtime.InteropServices.Marshal]::SecureStringToBSTR($sp.Secret));tenant=(Get-AzContext).Tenant.Id } | ConvertTo-Json
     ```

   Exemple de sortie des commandes ci-dessus :

   ```json
   {
       "appId": "aaaaaaaa-aaaa-aaaa-aaaa-aaaaaaaaaaaa",
       "password": "bbbbbbbb-bbbb-bbbb-bbbb-bbbbbbbbbbbb",
       "tenant": "cccccccc-cccc-cccc-cccc-cccccccccccc"
   }
   ```

Une fois les étapes préalables ci-dessus accomplies, installez le module complémentaire Azure Policy dans votre cluster Kubernetes compatible avec Azure Arc :

1. Ajoutez le référentiel du module complémentaire Azure Policy à Helm :

   ```bash
   helm repo add azure-policy https://raw.githubusercontent.com/Azure/azure-policy/master/extensions/policy-addon-kubernetes/helm-charts
   ```

1. Installez le module complémentaire Azure Policy à l’aide d’un graphique Helm :

   ```bash
   # In below command, replace the following values with those gathered above.
   #    <AzureArcClusterResourceId> with your Azure Arc enabled Kubernetes cluster resource Id. For example: /subscriptions/<subscriptionId>/resourceGroups/<rg>/providers/Microsoft.Kubernetes/connectedClusters/<clusterName>
   #    <ServicePrincipalAppId> with app Id of the service principal created during prerequisites.
   #    <ServicePrincipalPassword> with password of the service principal created during prerequisites.
   #    <ServicePrincipalTenantId> with tenant of the service principal created during prerequisites.
   helm install azure-policy-addon azure-policy/azure-policy-addon-arc-clusters \
       --set azurepolicy.env.resourceid=<AzureArcClusterResourceId> \
       --set azurepolicy.env.clientid=<ServicePrincipalAppId> \
       --set azurepolicy.env.clientsecret=<ServicePrincipalPassword> \
       --set azurepolicy.env.tenantid=<ServicePrincipalTenantId>
   ```

   Pour plus d’informations sur les composants installés par le module complémentaire Helm Chart, voir la [Définition du module complémentaire Helm Chart Azure Policy](https://github.com/Azure/azure-policy/tree/master/extensions/policy-addon-kubernetes/helm-charts/azure-policy-addon-arc-clusters) sur GitHub.

Pour vérifier que l’installation du module complémentaire a réussi et que les pods _azure-policy_ et _gatekeeper_ sont en cours d’exécution, exécutez la commande suivante :

```bash
# azure-policy pod is installed in kube-system namespace
kubectl get pods -n kube-system

# gatekeeper pod is installed in gatekeeper-system namespace
kubectl get pods -n gatekeeper-system
```

## <a name="install-azure-policy-add-on-for-aks-engine-preview"></a><a name="install-azure-policy-add-on-for-aks-engine"></a>Installer le module complémentaire Azure Policy pour le moteur AKS (préversion)

Avant d’installer le module complémentaire Azure Policy ou d’activer des fonctionnalités du service, vous devez activer le fournisseur de ressources **Microsoft.PolicyInsights** et créer une attribution de rôle pour le principal de service du cluster dans votre abonnement.

1. Azure CLI 2.0.62 ou version ultérieure doit être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

1. Pour activer le fournisseur de ressources, suivez les étapes décrites dans [Fournisseurs de ressources et types](../../../azure-resource-manager/management/resource-providers-and-types.md#azure-portal) ou exécutez la commande Azure CLI ou Azure PowerShell :

   - Azure CLI

     ```azurecli-interactive
     # Log in first with az login if you're not using Cloud Shell

     # Provider register: Register the Azure Policy provider
     az provider register --namespace 'Microsoft.PolicyInsights'
     ```

   - Azure PowerShell

     ```azurepowershell-interactive
     # Log in first with Connect-AzAccount if you're not using Cloud Shell

     # Provider register: Register the Azure Policy provider
     Register-AzResourceProvider -ProviderNamespace 'Microsoft.PolicyInsights'
     ```

1. Créez une attribution de rôle pour le principal de service du cluster.

   - Si vous ne connaissez pas l’ID de l’application du principal de service du cluster, cherchez-le avec la commande suivante.

     ```bash
     # Get the kube-apiserver pod name
     kubectl get pods -n kube-system

     # Find the aadClientID value
     kubectl exec <kube-apiserver pod name> -n kube-system cat /etc/kubernetes/azure.json
     ```

   - Affectez l’attribution de rôle « Policy Insights Data Writer (Preview) » à l’ID de l’application du principal de service du cluster (valeur _aadClientID_ de l’étape précédente) avec Azure CLI. Remplacez `<subscriptionId>` par votre ID d’abonnement et `<aks engine cluster resource group>` par le groupe de ressources dans lequel se trouve le cluster Kubernetes autogéré AKS Engine.

     ```azurecli-interactive
     az role assignment create --assignee <cluster service principal app ID> --scope "/subscriptions/<subscriptionId>/resourceGroups/<aks engine cluster resource group>" --role "Policy Insights Data Writer (Preview)"
     ```

Une fois les étapes préalables ci-dessus accomplies, installez le module complémentaire Azure Policy. L’installation peut être effectuée pendant le cycle de création ou de mise à jour d’un moteur AKS Engine ou comme action indépendante sur un cluster existant.

- Effectuer l’installation pendant le cycle de création ou de mise à jour

  Pour activer le module complémentaire Azure Policy lors de la création d’un cluster autogéré ou comme mise à jour d’un cluster existant, ajoutez la définition du cluster de propriétés [addons](https://github.com/Azure/aks-engine/tree/master/examples/addons/azure-policy) pour AKS Engine.

  ```json
  "addons": [{
      "name": "azure-policy",
      "enabled": true
  }]
  ```

  Pour plus d’informations à ce sujet, voir le guide externe [Définition du cluster AKS Engine](https://github.com/Azure/aks-engine/blob/master/docs/topics/clusterdefinitions.md).

- Installer un cluster existant avec Helm Charts

  Suivez les étapes ci-dessous pour préparer le cluster et installer le module complémentaire :

  1. Installez [Helm 3](https://v3.helm.sh/docs/intro/install/).

  1. Ajoutez le référentiel Azure Policy à Helm.

     ```bash
     helm repo add azure-policy https://raw.githubusercontent.com/Azure/azure-policy/master/extensions/policy-addon-kubernetes/helm-charts
     ```

     Pour plus d’informations, voir [Chart Helm – Guide de démarrage rapide](https://helm.sh/docs/using_helm/#quickstart-guide).

  1. Installez le module complémentaire avec un chart Helm. Remplacez `<subscriptionId>` par votre ID d’abonnement et `<aks engine cluster resource group>` par le groupe de ressources dans lequel se trouve le cluster Kubernetes autogéré AKS Engine.

     ```bash
     helm install azure-policy-addon azure-policy/azure-policy-addon-aks-engine --set azurepolicy.env.resourceid="/subscriptions/<subscriptionId>/resourceGroups/<aks engine cluster resource group>"
     ```

     Pour plus d’informations sur les composants installés par le module complémentaire Helm Chart, voir la [Définition du module complémentaire Helm Chart Azure Policy](https://github.com/Azure/azure-policy/tree/master/extensions/policy-addon-kubernetes/helm-charts/azure-policy-addon-aks-engine) sur GitHub.

     > [!NOTE]
     > En raison de la relation entre le module complémentaire Azure Policy et l’ID du groupe de ressources, Azure Policy ne prend en charge qu’un seul cluster AKS Engine par groupe de ressources.

Pour vérifier que l’installation du module complémentaire a réussi et que les pods _azure-policy_ et _gatekeeper_ sont en cours d’exécution, exécutez la commande suivante :

```bash
# azure-policy pod is installed in kube-system namespace
kubectl get pods -n kube-system

# gatekeeper pod is installed in gatekeeper-system namespace
kubectl get pods -n gatekeeper-system
```

## <a name="policy-language"></a>Langage de stratégie

La structure du langage Azure Policy pour la gestion de Kubernetes suit celle des définitions de stratégies existantes. Avec un [mode Fournisseur de ressources](./definition-structure.md#resource-provider-modes) de `Microsoft.Kubernetes.Data`, les effets [audit](./effects.md#audit) et [deny](./effects.md#deny) sont utilisés pour gérer vos clusters Kubernetes. Les effets _audit_ et _deny_ doivent fournir des **informations** sur les propriétés spécifiques à l’utilisation d’[OPA Constraint Framework](https://github.com/open-policy-agent/frameworks/tree/master/constraint) et de Gatekeeper v3.

Au sein des propriétés _details.constraintTemplate_ et _details.constraint_ de la définition de stratégie, Azure Policy transmet les URI de ces [CustomResourceDefinitions](https://github.com/open-policy-agent/gatekeeper#constraint-templates) (CRD) au module complémentaire. Rego est le langage pris en charge par OPA et Gatekeeper pour valider une requête au cluster Kubernetes. Grâce à la prise en charge d’une norme existante pour la gestion de Kubernetes, Azure Policy permet de réutiliser des règles existantes et de les jumeler avec Azure Policy afin de bénéficier d’une expérience de rapports de conformité du cloud unifiée. Pour plus d’informations, consultez [What is Rego?](https://www.openpolicyagent.org/docs/latest/policy-language/#what-is-rego).

## <a name="assign-a-built-in-policy-definition"></a>Affecter une définition de stratégie intégrée

Pour que vous puissiez affecter une définition de stratégie à votre cluster Kubernetes, il faut que les opérations appropriées d’attribution de stratégie de contrôle d’accès Azure en fonction du rôle (Azure RBAC) vous aient été attribuées. Les rôles Azure intégrés **Contributeur de la stratégie de ressource** et **Propriétaire** incluent ces opérations. Pour plus d’informations, consultez [Autorisations Azure RBAC dans Azure Policy](../overview.md#azure-rbac-permissions-in-azure-policy).

Recherchez les définitions de stratégie intégrées pour la gestion de votre cluster à l’aide du portail Azure en procédant comme suit :

1. Démarrez le service Azure Policy dans le portail Azure. Sélectionnez **Tous les services** dans le volet gauche, puis recherchez et sélectionnez **Stratégie**.

1. Dans le volet gauche de la page Azure Policy, sélectionnez **Définitions**.

1. Dans la zone de liste déroulante Catégorie, utilisez **Sélectionner tout** pour effacer le filtre, puis sélectionnez **Kubernetes**.

1. Sélectionnez la définition de stratégie, puis sélectionnez le bouton **Affecter**.

1. Définissez **Étendue** sur le groupe de gestion, l’abonnement ou le groupe de ressources du cluster Kubernetes auquel s’appliquera l’affectation de stratégie.

   > [!NOTE]
   > Lors de l’affectation de la définition d’Azure Policy pour Kubernetes, l’**Étendue** doit inclure la ressource de cluster. Pour un cluster de moteur AKS, l’**Étendue** doit être le groupe de ressources du cluster.

1. Donnez un **Nom** et une **Description** à l’affectation de stratégie, qui permettront de l’identifier facilement.

1. Définissez l’[application de stratégie](./assignment-structure.md#enforcement-mode) sur l’une des valeurs ci-dessous.

   - **Activé** - Appliquer la stratégie sur le cluster. Les demandes d’admission Kubernetes avec des violations sont refusées.

   - **Désactivé** - Ne pas appliquer la stratégie sur le cluster. Les demandes d’admission Kubernetes avec des violations ne sont pas refusées. Les résultats de l’évaluation de la conformité sont toujours disponibles. Lors du déploiement de nouvelles définitions de stratégies sur Des clusters en cours d’exécution, l’option _Désactivé_ est utile pour tester la définition de stratégie, étant donné que les demandes d’admission avec des violations ne sont pas refusées.

1. Sélectionnez **Suivant**.

1. Définir les **valeurs de paramètres**

   - Pour exclure les espaces de noms Kubernetes de l’évaluation de stratégie, répertoriez les espaces de noms dans le paramètre **Exclusions d’espaces de noms**. Il est recommandé d’exclure _Kube-System_, _Gatekeeper-System_ et _Azure-arc_.

1. Sélectionnez **Revoir + créer**.

Vous pouvez également utiliser le démarrage rapide [Attribuer une stratégie – Portail](../assign-policy-portal.md) pour rechercher et attribuer une stratégie Kubernetes. Recherchez une définition de stratégie Kubernetes au lieu de l’exemple « audit vms ».

> [!IMPORTANT]
> Les définitions de stratégie intégrées sont disponibles pour les clusters Kubernetes dans la catégorie **Kubernetes**. Pour obtenir la liste des définitions de stratégies intégrées, consultez [Exemples Kubernetes](../samples/built-in-policies.md#kubernetes).

## <a name="policy-evaluation"></a>Évaluation de la stratégie

Le module complémentaire contacte le service Azure Policy toutes les quinze minutes pour vérifier si des modifications ont été apportées aux affectations de stratégie.
Pendant ce cycle d’actualisation, le module complémentaire recherche d’éventuelles modifications, qui déclenchent la création, la mise à jour ou la suppression des modèles de contrainte et des contraintes.

Dans un cluster Kubernetes, si un espace de noms a l’une des étiquettes suivantes, les demandes d’admission avec violations sont refusées. Les résultats de l’évaluation de la conformité sont toujours disponibles.

- `control-plane`
- `admission.policy.azure.com/ignore`

> [!NOTE]
> Un administrateur de cluster peut être autorisé à créer et mettre à jour des modèles et ressources de contrainte installés par le module complémentaire Azure Policy, ces scénarios ne sont pas pris en charge, car les mises à jour manuelles sont remplacées. Gatekeeper continue d’évaluer les stratégies qui existaient avant l’installation du module complémentaire et l’affectation de définitions de stratégie Azure Policy.

Toutes les quinze minutes, le module complémentaire demande une analyse complète du cluster. Après la collecte des détails de l’analyse complète et les évaluations en temps réel faites par Gatekeeper des tentatives de modification du cluster, le module complémentaire renvoie les résultats à Azure Policy pour les inclure dans les [détails de conformité](../how-to/get-compliance-data.md) comme toute affectation Azure Policy. Seuls les résultats des affectations de stratégie actives sont renvoyés au cours du cycle d’audit. Les résultats d’audit peuvent également être considérés comme des [violations](https://github.com/open-policy-agent/gatekeeper#audit) indiquées dans le champ d’état de la contrainte non réussie. Pour plus d’informations sur les ressources _Non conformes_, consultez [Détails du composant pour les modes Fournisseur de ressources](../how-to/determine-non-compliance.md#component-details-for-resource-provider-modes).

> [!NOTE]
> Chaque rapport de conformité dans Azure Policy pour vos clusters Kubernetes inclut toutes les violations des 45 dernières minutes. L’horodateur indique quand une violation s’est produite.

Quelques autres considérations :

- Si l’abonnement au cluster est inscrit auprès d’Azure Security Center, les stratégies Kubernetes Azure Security Center sont automatiquement appliquées au cluster.

- Lorsqu’une stratégie de refus est appliquée sur un cluster avec des ressources Kubernetes existantes, toute ressource préexistante non conforme à la nouvelle stratégie continue de s’exécuter. Si la ressource non conforme est replanifiée sur un autre nœud, Gatekeeperbloque la création de la ressource.

- Si un cluster comporte une stratégie de refus qui valide les ressources, l’utilisateur ne verra pas de message de refus lors de la création d’un déploiement. Prenons l’exemple d’un déploiement Kubernetes contenant des jeux de réplicas et des pods. Lorsqu’un utilisateur exécute `kubectl describe deployment $MY_DEPLOYMENT`, il ne renvoie pas de message de refus dans le cadre des événements. Toutefois, `kubectl describe replicasets.apps $MY_DEPLOYMENT` retourne les événements associés au rejet.

## <a name="logging"></a>Journalisation

En tant que contrôleur/conteneur Kubernetes, les pods _azure-policy_ et _gatekeeper_ conservent les journaux dans le cluster Kubernetes. Les journaux peuvent être exposés dans la page **Insights** du cluster Kubernetes.
Pour plus d’informations, consultez [Superviser les performances de votre cluster Kubernetes avec Azure Monitor pour conteneurs](../../../azure-monitor/insights/container-insights-analyze.md).

Pour afficher les journaux du module complémentaire, utilisez `kubectl` :

```bash
# Get the azure-policy pod name installed in kube-system namespace
kubectl logs <azure-policy pod name> -n kube-system

# Get the gatekeeper pod name installed in gatekeeper-system namespace
kubectl logs <gatekeeper pod name> -n gatekeeper-system
```

Pour plus d’informations, consultez [Déboguer Gatekeeper](https://github.com/open-policy-agent/gatekeeper#debugging) dans la documentation de Gatekeeper.

## <a name="troubleshooting-the-add-on"></a>Résolution des problèmes liés au module complémentaire

Pour plus d’informations sur la résolution des problèmes liés au module complémentaire pour Kubernetes, consultez la section [Kubernetes](/azure/governance/policy/troubleshoot/general#add-on-for-kubernetes-general-errors) de l’article sur la résolution des problèmes liés à Azure Policy.

## <a name="remove-the-add-on"></a>Supprimer le module complémentaire

### <a name="remove-the-add-on-from-aks"></a>Supprimer le module complémentaire d’AKS

Pour supprimer le module complémentaire Azure Policy de votre cluster AKS, utilisez le Portail Azure ou Azure CLI :

- Portail Azure

  1. Lancez le service AKS dans le portail Azure en sélectionnant **Tous les services**, puis en recherchant et en sélectionnant **Services Kubernetes**.

  1. Sélectionnez le cluster AKS dans lequel vous souhaitez désactiver le module complémentaire Azure Policy.

  1. Sélectionnez **Stratégies** sur le côté gauche de la page du service Kubernetes.

  1. Dans la page principale, sélectionnez le bouton **Désactiver un module complémentaire**.

- Azure CLI

  ```azurecli-interactive
  # Log in first with az login if you're not using Cloud Shell

  az aks disable-addons --addons azure-policy --name MyAKSCluster --resource-group MyResourceGroup
  ```

### <a name="remove-the-add-on-from-azure-arc-enabled-kubernetes"></a>Supprimer le module complémentaire de Kubernetes compatible avec Azure Arc

Pour supprimer le module complémentaire Azure Policy et Gatekeeper de votre cluster Kubernetes compatible avec Azure Arc, exécutez la commande Helm suivante :

```bash
helm uninstall azure-policy-addon
```

### <a name="remove-the-add-on-from-aks-engine"></a>Supprimer le module complémentaire du moteur AKS

Pour supprimer le module complémentaire Azure Policy et Gatekeeper de votre cluster AKS Engine, utilisez la méthode correspondant à la façon dont le module a été installé :

- S’il a été installé en définissant la propriété **addons** dans la définition du cluster pour AKS Engine :

  Redéployez la définition du cluster sur AKS Engine après avoir modifié la propriété **addons** pour _azure-policy_ en lui donnant la valeur false :


  ```json
  "addons": [{
      "name": "azure-policy",
      "enabled": false
  }]
  ```

  Pour plus d’informations, consultez [Moteur AKS – Désactiver le module complémentaire Azure Policy](https://github.com/Azure/aks-engine/blob/master/examples/addons/azure-policy/README.md#disable-azure-policy-add-on).

- S’il a été installé avec des graphiques Helm, exécutez la commande Helm suivante :

  ```bash
  helm uninstall azure-policy-addon
  ```

## <a name="diagnostic-data-collected-by-azure-policy-add-on"></a>Données de diagnostic collectées par le module complémentaire Azure Policy

Le module complémentaire Azure Policy pour Kubernetes collecte une quantité limitée de données de diagnostics de cluster. Ces données de diagnostic sont des données techniques vitales concernant les logiciels et le niveau de performance. Elles sont utilisées de différentes manières :

- Tenir à jour le module complémentaire Azure Policy
- Maintenir la sécurité, la fiabilité et le niveau de performance du module complémentaire Azure Policy
- Améliorer le module complémentaire Azure Policy à travers l’analyse globale de son utilisation

Les informations collectées par le module complémentaire ne sont pas des données personnelles. Les détails suivants sont actuellement recueillis :

- Version de l’agent du module complémentaire Azure Policy
- Type de cluster
- Région du cluster
- Groupe de ressources de cluster
- ID de la ressource de cluster
- ID de l’abonnement du cluster
- Système d’exploitation du cluster (exemple : Linux)
- Ville du cluster (exemple : Seattle)
- État ou province du cluster (exemple : Washington)
- Pays ou région du cluster (exemple : États-Unis)
- Exceptions/erreurs rencontrées par le module complémentaire Azure Policy lors de l’installation de l’agent concernant l’évaluation de la stratégie
- Nombre de définitions de stratégies Gatekeeper non installées par le module complémentaire Azure Policy

## <a name="next-steps"></a>Étapes suivantes

- Consultez des exemples à la page [Exemples Azure Policy](../samples/index.md).
- Consultez la page [Structure de définition Azure Policy](definition-structure.md).
- Consultez la page [Compréhension des effets de Policy](effects.md).
- Découvrez comment [créer des stratégies par programmation](../how-to/programmatically-create.md).
- Découvrez comment [obtenir des données de conformité](../how-to/get-compliance-data.md).
- Découvrez comment [corriger des ressources non conformes](../how-to/remediate-resources.md).
- Pour en savoir plus sur les groupes d’administration, consultez [Organiser vos ressources avec des groupes d’administration Azure](../../management-groups/overview.md).

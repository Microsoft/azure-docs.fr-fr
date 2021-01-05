---
title: Utiliser les identités managées dans Azure Kubernetes Service
description: Découvrez comment utiliser les identités managées dans Azure Kubernetes Service (AKS).
services: container-service
ms.topic: article
ms.date: 12/16/2020
ms.openlocfilehash: 948a189e1c6e03efca046b6d43dddcaf3d141957
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97607284"
---
# <a name="use-managed-identities-in-azure-kubernetes-service"></a>Utiliser les identités managées dans Azure Kubernetes Service

Actuellement, un cluster Azure Kubernetes Service ou AKS (plus précisément, le fournisseur cloud Kubernetes) nécessite une identité pour créer des ressources supplémentaires telles que des équilibreurs de charge et des disques managés dans Azure. Cette identité peut être une *identité gérée* ou un *principal de service*. Si vous utilisez un [principal de service](kubernetes-service-principal.md), vous devez en fournir un ; sinon, AKS en crée un en votre nom. Si vous utilisez une identité managée, elle est automatiquement créée pour vous par AKS. Les clusters utilisant des principaux de service finissent par atteindre un état dans lequel le principal de service doit être renouvelé pour que le cluster continue de fonctionner. La gestion des principaux de service ajoute de la complexité : c’est pourquoi il est plus facile d’utiliser à la place des identités managées. Les mêmes exigences d’autorisation s’appliquent aux principaux de service et aux identités managées.

Les *identités managées* correspondent essentiellement à un wrapper autour des principaux de service, ce qui simplifie leur gestion. La rotation des informations d’identification pour MI se produit automatiquement tous les 46 jours selon la valeur par défaut dans Azure Active Directory. AKS utilise aussi bien les identités managées affectées par le système que les types d’identités managées affectées par l’utilisateur. Ces identités sont actuellement immuables. Pour en savoir plus, découvrez les [identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="before-you-begin"></a>Avant de commencer

La ressource suivante doit être installée :

- Azure CLI 2.15.1 ou version ultérieure

## <a name="limitations"></a>Limites

* Pendant les opérations de **mise à niveau** du cluster, l’identité managée est temporairement indisponible.
* Le déplacement de locataires ou la migration de clusters avec des identités managées ne sont pas pris en charge.
* Si `aad-pod-identity` est activé dans le cluster, les pods NMI (Node Managed Identity) modifient les tables d’adresses IP des nœuds pour intercepter les appels vers le point de terminaison Azure Instance Metadata. Cette configuration signifie que toutes les requêtes adressées au point de terminaison Metadata sont interceptées par NMI, même si le pod n’utilise pas `aad-pod-identity`. La CRD AzurePodIdentityException peut être configurée de manière à informer `aad-pod-identity` que toutes les requêtes adressées au point de terminaison Metadata depuis un pod correspondant aux étiquettes définies dans la CRD doivent être envoyées par proxy sans aucun traitement dans NMI. Les pods système qui disposent de l’étiquette `kubernetes.azure.com/managedby: aks` dans l’espace de noms _kube-system_ doivent être exclus de `aad-pod-identity` en configurant la CRD AzurePodIdentityException. Pour plus d’informations, consultez [Désactiver aad-pod-identity pour un pod ou une application spécifique](https://azure.github.io/aad-pod-identity/docs/configure/application_exception).
  Pour configurer une exception, installez le fichier [YAML mic-exception](https://github.com/Azure/aad-pod-identity/blob/master/deploy/infra/mic-exception.yaml).

## <a name="summary-of-managed-identities"></a>Résumé des identités managées

AKS utilise plusieurs identités managées pour les services intégrés et les modules complémentaires.

| Identité                       | Nom    | Cas d’utilisation | Autorisations par défaut | Apportez votre propre identité
|----------------------------|-----------|----------|
| Plan de contrôle | non visible | Utilisé par les composants du plan de contrôle AKS pour gérer les ressources de cluster, notamment les équilibreurs de charge d’entrée et les IP publiques gérées par AKS, ainsi que les opérations de mise à l’échelle automatique | Rôle Contributeur pour le groupe de ressources du nœud | supported
| Kubelet | Nom du cluster AKS - agentpool | Authentification avec Azure Container Registry (ACR) | NA (pour kubernetes v1.15+) | Non prise en charge pour le moment
| Composant additionnel | AzureNPM | Aucune identité requise | N/D | Non
| Composant additionnel | Analyse du réseau AzureCNI | Aucune identité requise | N/D | Non
| Composant additionnel | azure-policy (gatekeeper) | Aucune identité requise | N/D | Non
| Composant additionnel | azure-policy | Aucune identité requise | N/D | Non
| Composant additionnel | Calico | Aucune identité requise | N/D | Non
| Composant additionnel | tableau de bord | Aucune identité requise | N/D | Non
| Composant additionnel | HTTPApplicationRouting | Gère les ressources réseau requises | Rôle Lecteur pour le groupe de ressources du nœud, rôle Contributeur pour la zone DNS | Non
| Composant additionnel | Passerelle d'application d’entrée | Gère les ressources réseau requises| Rôle Contributeur pour le groupe de ressources du nœud | Non
| Composant additionnel | omsagent | Utilisé pour envoyer des métriques AKS à Azure Monitor | Rôle Éditeur des métriques de surveillance | Non
| Composant additionnel | Nœud virtuel (ACIConnector) | Gère les ressources réseau requises pour Azure Container Instances (ACI) | Rôle Contributeur pour le groupe de ressources du nœud | Non
| Projet OSS | aad-pod-identity | Permet aux applications d'accéder aux ressources cloud en toute sécurité avec Azure Active Directory (AAD) | N/D | Étapes à suivre pour octroyer une autorisation disponibles à l'adresse https://github.com/Azure/aad-pod-identity#role-assignment.

## <a name="create-an-aks-cluster-with-managed-identities"></a>Créer un cluster AKS avec des identités managées

Vous pouvez désormais créer un cluster AKS avec des identités managées à l’aide des commandes CLI suivantes.

Commencez par créer un groupe de ressources Azure :

```azurecli-interactive
# Create an Azure resource group
az group create --name myResourceGroup --location westus2
```

Créez ensuite un cluster AKS :

```azurecli-interactive
az aks create -g myResourceGroup -n myManagedCluster --enable-managed-identity
```

Un cluster créé à l’aide d’identités managées contient les informations de profil du principal de service suivantes :

```output
"servicePrincipalProfile": {
    "clientId": "msi"
  }
```

Utilisez la commande suivante pour interroger l’objectid de votre identité managée de plan de contrôle :

```azurecli-interactive
az aks show -g myResourceGroup -n myManagedCluster --query "identity"
```

Le résultat doit avoir l’aspect suivant :

```output
{
  "principalId": "<object_id>",   
  "tenantId": "<tenant_id>",      
  "type": "SystemAssigned"                                 
}
```

Une fois le cluster créé, vous pouvez déployer vos charges de travail d’application sur le nouveau cluster et interagir avec celui-ci comme vous le faisiez avec les clusters AKS basés sur le principal de service.

> [!NOTE]
> Pour créer et utiliser votre propre réseau virtuel, une adresse IP statique ou un disque Azure attaché où les ressources se trouvent en dehors du groupe de ressources du nœud Worker, utilisez le PrincipalID du cluster Identité managée affectée par le système pour effectuer une attribution de rôle. Pour plus d’informations sur l’attribution de rôle, consultez [Déléguer l’accès à d’autres ressources Azure](kubernetes-service-principal.md#delegate-access-to-other-azure-resources).
>
> L'octroi d'une autorisation à une identité managée en cluster utilisée par le fournisseur Azure Cloud peut prendre jusqu'à 60 minutes.

Obtenez enfin les informations d’identification pour accéder au cluster :

```azurecli-interactive
az aks get-credentials --resource-group myResourceGroup --name myManagedCluster
```
## <a name="update-an-aks-cluster-to-managed-identities-preview"></a>Mettre à jour un cluster AKS vers des identités managées (préversion)

Vous pouvez désormais mettre à jour un cluster AKS actuellement utilisé avec des principaux de service pour travailler avec des identités managées à l’aide des commandes CLI suivantes.

Tout d’abord, enregistrez l’indicateur de fonctionnalité pour l’identité affectée par le système :

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n MigrateToMSIClusterPreview
```

Mettez à jour l’identité affectée par le système :

```azurecli-interactive
az aks update -g <RGName> -n <AKSName> --enable-managed-identity
```

Enregistrez l’indicateur de fonctionnalité pour l’identité affectée par l’utilisateur :

```azurecli-interactive
az feature register --namespace Microsoft.ContainerService -n UserAssignedIdentityPreview
```

Mettez à jour l’identité affectée par l’utilisateur :

```azurecli-interactive
az aks update -g <RGName> -n <AKSName> --enable-managed-identity --assign-identity <UserAssignedIdentityResourceID> 
```
> [!NOTE]
> Une fois les identités affectées par le système ou par l’utilisateur mises à jour avec l’identité managée, effectuez une opération `az nodepool upgrade --node-image-only` sur vos nœuds pour terminer la mise à jour de l’identité managée.

## <a name="bring-your-own-control-plane-mi"></a>Apporter votre propre instance gérée de plan de contrôle
Une identité de plan de contrôle personnalisé permet d’accorder l’accès à l’identité existante avant la création du cluster. Cette fonctionnalité permet des scénarios tels que l’utilisation d’un réseau virtuel personnalisé ou du paramètre outboundType d’UDR avec une identité managée pré-créée.

Vous devez avoir installé Azure CLI 2.15.1 ou une version ultérieure.

### <a name="limitations"></a>Limites
* Azure Government n’est pas pris en charge.
* Azure China 21Vianet n’est pas pris en charge.

Si vous n’avez pas encore d’identité managée, vous devez en créer une, par exemple à l’aide de la commande CLI [az identity][az-identity-create].

```azurecli-interactive
az identity create --name myIdentity --resource-group myResourceGroup
```
Le résultat doit avoir l’aspect suivant :

```output
{                                                                                                                                                                                 
  "clientId": "<client-id>",
  "clientSecretUrl": "<clientSecretUrl>",
  "id": "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity", 
  "location": "westus2",
  "name": "myIdentity",
  "principalId": "<principalId>",
  "resourceGroup": "myResourceGroup",                       
  "tags": {},
  "tenantId": "<tenant-id>>",
  "type": "Microsoft.ManagedIdentity/userAssignedIdentities"
}
```

Si votre identité managée fait partie de votre abonnement, vous pouvez utiliser la commande CLI [az identity][az-identity-list] pour l’interroger.  

```azurecli-interactive
az identity list --query "[].{Name:name, Id:id, Location:location}" -o table
```

Vous pouvez maintenant utiliser la commande suivante pour créer votre cluster avec l’identité existante :

```azurecli-interactive
az aks create \
    --resource-group myResourceGroup \
    --name myManagedCluster \
    --network-plugin azure \
    --vnet-subnet-id <subnet-id> \
    --docker-bridge-address 172.17.0.1/16 \
    --dns-service-ip 10.2.0.10 \
    --service-cidr 10.2.0.0/24 \
    --enable-managed-identity \
    --assign-identity <identity-id> \
```

Un cluster créé à l’aide de vos propres identités managées contient les informations de profil userAssignedIdentities :

```output
 "identity": {
   "principalId": null,
   "tenantId": null,
   "type": "UserAssigned",
   "userAssignedIdentities": {
     "/subscriptions/<subscriptionid>/resourcegroups/myResourceGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/myIdentity": {
       "clientId": "<client-id>",
       "principalId": "<principal-id>"
     }
   }
 },
```

## <a name="next-steps"></a>Étapes suivantes
* Utilisez des [modèles ARM (Azure Resource Manager)][aks-arm-template] pour créer des clusters avec gestion des identités.

<!-- LINKS - external -->
[aks-arm-template]: /azure/templates/microsoft.containerservice/managedclusters
[az-identity-create]: /cli/azure/identity?view=azure-cli-latest#az-identity-create&preserve-view=true
[az-identity-list]: /cli/azure/identity?view=azure-cli-latest#az-identity-list&preserve-view=true

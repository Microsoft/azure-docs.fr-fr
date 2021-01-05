---
title: Déployer un cluster Service Fabric managé (préversion)
description: Dans ce tutoriel, vous allez déployer un cluster Service Fabric managé à des fins de test.
ms.topic: tutorial
ms.date: 08/27/2020
ms.custom: references_regions
ms.openlocfilehash: 0265a1393c697cbd767de13df1064a5eea957380
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92316170"
---
# <a name="tutorial-deploy-a-service-fabric-managed-cluster-preview"></a>Tutoriel : Déployer un cluster Service Fabric managé (préversion)

Dans cette série de tutoriels, nous allons aborder les points suivants :

> [!div class="checklist"]
> * Comment déployer un cluster Service Fabric managé 
> * [Comment effectuer le scale-out d’un cluster Service Fabric managé](tutorial-managed-cluster-scale.md)
> * [Comment ajouter et supprimer des nœuds dans un cluster Service Fabric managé](tutorial-managed-cluster-add-remove-node-type.md)
> * [Comment déployer une application sur un cluster Service Fabric managé](tutorial-managed-cluster-deploy-app.md)

Cette partie de la série explique comment :

> [!div class="checklist"]
> * Se connecter au compte Azure
> * Créer un groupe de ressources
> * Déployer un cluster Service Fabric managé
> * Ajouter un type de nœud principal au cluster

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas encore d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Installez le [Kit de développement logiciel (SDK) Service Fabric et le module PowerShell](service-fabric-get-started.md).

* Installez [Azure PowerShell 4.7.0](/powershell/azure/release-notes-azureps?preserve-view=true&view=azps-4.7.0#azservicefabric) (ou ultérieur).

## <a name="connect-to-your-azure-account"></a>Se connecter au compte Azure

Remplacez `<your-subscription>` par la chaîne de connexion de votre compte Azure et connectez-vous :

```powershell
Login-AzAccount
Set-AzContext -SubscriptionId <your-subscription>

```

## <a name="create-a-new-resource-group"></a>Créer un groupe de ressources

Ensuite, créez le groupe de ressources pour le cluster Service Fabric managé, en remplaçant `<your-rg>` et `<location>` par le nom du groupe et l’emplacement souhaités.

> [!NOTE]
> Les régions prises en charge pour la préversion publique incluent `centraluseuap`, `eastus2euap`, `eastasia`, `northeurope`, `westcentralus` et `eastus2`.

```powershell
$resourceGroup = "myResourceGroup"
$location = "EastUS2"

New-AzResourceGroup -Name $resourceGroup -Location $location
```

## <a name="deploy-a-service-fabric-managed-cluster"></a>Déployer un cluster Service Fabric managé

### <a name="create-a-service-fabric-managed-cluster"></a>Créer un cluster managé Service Fabric

Dans cette étape, vous allez créer un cluster Service Fabric managé en utilisant la commande PowerShell New-AzServiceFabricManagedCluster. L’exemple suivant crée un cluster nommé myCluster dans le groupe de ressources nommé myResourceGroup. Ce groupe de ressources a été créé à l’étape précédente dans la région eastus2.

Pour cette étape, fournissez vos propres valeurs pour les paramètres suivants :

* **Nom du cluster** : Entrez un nom unique pour votre cluster, par exemple *mysfcluster*.
* **Mot de passe administrateur** : Entrez un mot de passe pour l’administrateur à utiliser pour le protocole RDP sur les machines virtuelles sous-jacentes dans le cluster.
* **Empreinte de certificat du client** : Indiquez l’empreinte du certificat client que vous souhaitez utiliser pour accéder à votre cluster. Si vous n’avez pas de certificat, suivez [Définir et récupérer un certificat](../key-vault/certificates/quick-create-portal.md) pour créer un certificat auto-signé.
* **Référence SKU du cluster** : Spécifiez le [type de cluster Service Fabric managé](overview-managed-cluster.md#service-fabric-managed-cluster-skus) à déployer. Les clusters avec une référence SKU *De base* sont destinés seulement aux déploiements de test et ne permettent pas l’ajout ou la suppression d’un type de nœud.

```powershell
$clusterName = "<unique cluster name>"
$password = "Password4321!@#" | ConvertTo-SecureString -AsPlainText -Force
$clientThumbprint = "<certificate thumbprint>"
$clusterSku = "Standard"

New-AzServiceFabricManagedCluster -ResourceGroupName $resourceGroup -Location $location -ClusterName $clusterName -ClientCertThumbprint $clientThumbprint -ClientCertIsAdmin -AdminPassword $password -Sku $clusterSKU -Verbose
```

### <a name="add-a-primary-node-type-to-the-service-fabric-managed-cluster"></a>Ajouter un type de nœud principal au cluster Service Fabric managé

Dans cette étape, vous allez ajouter un type de nœud principal au cluster que vous venez de créer. Chaque cluster Service Fabric doit avoir au moins un type de nœud principal.

Pour cette étape, fournissez vos propres valeurs pour les paramètres suivants :

* **Nom du type de nœud** : Entrez un nom unique pour le type de nœud à ajouter à votre cluster, par exemple « NT1 ».

> [!NOTE]
> Si le type de nœud à ajouter est le premier ou le seul type de nœud dans le cluster, la propriété Principal doit être utilisée.

```powershell
$nodeType1Name = "NT1"

New-AzServiceFabricManagedNodeType -ResourceGroupName $resourceGroup -ClusterName $clusterName -Name $nodeType1Name -Primary -InstanceCount 5
```

Cette commande peut prendre plusieurs minutes.

## <a name="validate-the-deployment"></a>Valider le déploiement

### <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Une fois le déploiement terminé, recherchez la valeur Service Fabric Explorer dans la page de vue d’ensemble de la ressource de cluster Service Fabric managé dans le portail. Quand un certificat vous est demandé, utilisez le certificat pour lequel l’empreinte numérique du client a été fournie dans la commande PowerShell.

## <a name="next-steps"></a>Étapes suivantes

Dans cette étape, nous avons créé et déployé notre premier cluster Service Fabric managé. Pour en savoir plus sur la mise à l’échelle d’un cluster, consultez :

> [!div class="nextstepaction"]
> [Effectuer le scale-out d’un cluster Service Fabric managé](tutorial-managed-cluster-scale.md)
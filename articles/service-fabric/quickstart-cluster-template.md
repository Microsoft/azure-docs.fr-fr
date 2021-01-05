---
title: Créer un cluster Service Fabric avec un modèle Azure Resource Manager
description: Dans ce guide de démarrage rapide, vous allez créer un cluster de test Azure Resource Manager à l’aide d’un modèle Azure Resource Manager.
author: erikadoyle
ms.service: service-fabric
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: edoyle
ms.date: 07/29/2020
ms.openlocfilehash: 359b527733ee8eebf7e1e7d12c40a0c74ec1c9bd
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87460301"
---
# <a name="quickstart-create-a-service-fabric-cluster-using-arm-template"></a>Démarrage rapide : Créer un cluster Service Fabric à l’aide d’un modèle Resource Manager

Azure Service Fabric est une plateforme de systèmes distribués qui facilite le packaging, le déploiement et la gestion de conteneurs et de microservices évolutifs et fiables. Un *cluster* Service Fabric est un ensemble de machines virtuelles connectées au réseau, sur lequel vos microservices sont déployés et gérés. Cet article explique comment déployer un cluster de test Service Fabric dans Azure à l’aide d’un modèle Resource Manager (Azure Resource Manager).

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Ce cluster Windows à cinq nœuds est sécurisé avec un certificat auto-signé et, par conséquent, uniquement destiné à des fins pédagogiques (plutôt que pour des charges de travail de production). Nous allons utiliser Azure PowerShell pour déployer le modèle. Outre Azure PowerShell, vous pouvez également utiliser le portail Azure, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-portal.md).

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fservice-fabric-secure-cluster-5-node-1-nodetype%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

### <a name="install-service-fabric-sdk-and-powershell-modules"></a>Installer le SDK Service Fabric et des modules PowerShell

Pour suivre ce guide de démarrage rapide, vous devez effectuer les opérations suivantes :

* Installez le [Kit de développement logiciel (SDK) Service Fabric et le module PowerShell](service-fabric-get-started.md).

* Installez [Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="download-the-sample-template-and-certificate-helper-script"></a>Télécharger l’exemple de modèle et le script d’assistance de certificat

Clonez ou téléchargez le dépôt [Modèles de démarrage rapide Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates). Vous pouvez également copier localement les fichiers suivants à partir du dossier *service-fabric-secure-cluster-5-node-1-nodetype* :

* [New-ServiceFabricClusterCertificate.ps1](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/service-fabric-secure-cluster-5-node-1-nodetype/scripts/New-ServiceFabricClusterCertificate.ps1)
* [azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/service-fabric-secure-cluster-5-node-1-nodetype/azuredeploy.json)
* [azuredeploy.parameters.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/service-fabric-secure-cluster-5-node-1-nodetype/azuredeploy.parameters.json)

### <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure, puis désignez l’abonnement à utiliser pour la création de votre cluster Service Fabric.

```powershell
# Sign in to your Azure account
Login-AzAccount -SubscriptionId "<subscription ID>"
```

### <a name="create-a-self-signed-certificate-stored-in-key-vault"></a>Créer un certificat auto-signé dans Key Vault

Service Fabric utilise des certificats X.509 pour [sécuriser un cluster](./service-fabric-cluster-security.md) et fournir des fonctionnalités de sécurité d’application, ainsi que [Key Vault](../key-vault/general/overview.md) pour gérer ces certificats. La création réussie d’un cluster exige un certificat de cluster pour permettre la communication nœud à nœud. Dans le cadre de la création d’un cluster de test dans ce guide de démarrage rapide, nous allons créer un certificat auto-signé pour l’authentification du cluster. Les charges de travail de production exigent des certificats créés à l’aide d’un service de certificats Windows Server correctement configuré ou obtenu auprès d’une autorité de certification approuvée.

```powershell
# Designate unique (within cloudapp.azure.com) names for your resources
$resourceGroupName = "SFQuickstartRG"
$keyVaultName = "SFQuickstartKV"

# Create a new resource group for your Key Vault and Service Fabric cluster
New-AzResourceGroup -Name $resourceGroupName -Location SouthCentralUS

# Create a Key Vault enabled for deployment
New-AzKeyVault -VaultName $keyVaultName -ResourceGroupName $resourceGroupName -Location SouthCentralUS -EnabledForDeployment

# Generate a certificate and upload it to Key Vault
.\scripts\New-ServiceFabricClusterCertificate.ps1
```

Le script vous invite à entrer les informations suivantes (veillez à remplacer les exemples de valeurs *CertDNSName* et *KeyVaultName* ci-dessous) :

* **Mot de passe :** Password!1
* **CertDNSName :** *sfquickstart*.southcentralus.cloudapp.azure.com
* **KeyVaultName :** *SFQuickstartKV*
* **KeyVaultSecretName :** clustercert

Une fois que vous avez terminé, le script fournit les valeurs de paramètres nécessaires pour le déploiement du modèle. Veillez à les stocker dans les variables suivantes, car elles seront nécessaires pour déployer votre modèle de cluster :

```powershell
$sourceVaultId = "<Source Vault Resource Id>"
$certUrlValue = "<Certificate URL>"
$certThumbprint = "<Certificate Thumbprint>"
```

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/service-fabric-secure-cluster-5-node-1-nodetype/). Le modèle utilisé pour cet article est trop long pour être affiché ici. Pour afficher le modèle, consultez le fichier [azuredeploy.json](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/service-fabric-secure-cluster-5-node-1-nodetype/azuredeploy.json).

Plusieurs ressources Azure ont été définies dans le modèle :

* [Microsoft.Storage/storageAccounts](/azure/templates/microsoft.storage/storageaccounts)
* [Microsoft.Network/virtualNetworks](/azure/templates/microsoft.network/virtualnetworks)
* [Microsoft.Network/publicIPAddresses](/azure/templates/microsoft.network/publicipaddresses)
* [Microsoft.Network/loadBalancers](/azure/templates/microsoft.network/loadbalancers)
* [Microsoft.Compute/virtualMachineScaleSets](/azure/templates/microsoft.compute/virtualmachinescalesets)
* [Microsoft.ServiceFabric/clusters](/azure/templates/microsoft.servicefabric/clusters)

Pour rechercher d’autres modèles liés à Azure Service Fabric, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?sort=Popular&term=service+fabric).

### <a name="customize-the-parameters-file"></a>Personnaliser le fichier de paramètres

Ouvrez *azuredeploy.parameters.json*, puis modifiez les valeurs des paramètres afin que :

* **clusterName** corresponde à la valeur que vous avez fournie pour *CertDNSName* lors de la création de votre certificat de cluster
* **adminUserName** soit une valeur autre que le jeton *GEN-UNIQUE* par défaut
* **adminPassword** soit une valeur autre que le jeton *GEN-PASSWORD* par défaut
* **certificateThumbprint**, **sourceVaultResourceId** et **certificateUrlValue** soient tous une chaîne vide (`""`)

Par exemple :

```json
{
  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentParameters.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "clusterName": {
      "value": "sfquickstart"
    },
    "adminUsername": {
      "value": "testadm"
    },
    "adminPassword": {
      "value": "Password#1234"
    },
    "certificateThumbprint": {
      "value": ""
    },
    "sourceVaultResourceId": {
      "value": ""
    },
    "certificateUrlValue": {
      "value": ""
    }
  }
}
```

## <a name="deploy-the-template"></a>Déployer le modèle

Stockez le chemin de votre modèle Resource Manager et de vos fichiers de paramètres dans des variables, puis déployez le modèle.

```powershell
$templateFilePath = "<full path to azuredeploy.json>"
$parameterFilePath = "<full path to azuredeploy.parameters.json>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile $templateFilePath `
    -TemplateParameterFile $parameterFilePath `
    -CertificateThumbprint $certThumbprint `
    -CertificateUrlValue $certUrlValue `
    -SourceVaultResourceId $sourceVaultId `
    -Verbose
```

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Une fois le déploiement terminé, recherchez la valeur `managementEndpoint` dans la sortie, puis ouvrez l’adresse dans un navigateur web pour voir votre cluster dans [Service Fabric Explorer](./service-fabric-visualizing-your-cluster.md).

![Service Fabric Explorer montrant le nouveau cluster](./media/quickstart-cluster-template/service-fabric-explorer.png)

Vous pouvez également trouver le point de terminaison Service Fabric Explorer dans votre panneau de ressources Service Fabric Explore dans le portail Azure.

![Panneau de ressources Service Fabric présentant le point de terminaison Service Fabric Explorer](./media/quickstart-cluster-template/service-fabric-explorer-endpoint-azure-portal.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

```powershell
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

Ensuite, supprimez le certificat du cluster de votre magasin local. Listez les certificats installés pour trouver l’empreinte numérique de votre cluster :

```powershell
Get-ChildItem Cert:\CurrentUser\My\
```

Supprimez ensuite le certificat :

```powershell
Get-ChildItem Cert:\CurrentUser\My\{THUMBPRINT} | Remove-Item
```

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment créer un modèle de cluster Azure Service Fabric personnalisé, consultez :

> [!div class="nextstepaction"]
> [Créer un modèle Resource Manager pour un cluster Service Fabric](service-fabric-cluster-creation-create-template.md)

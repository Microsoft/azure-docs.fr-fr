---
title: 'Démarrage rapide : Créer un service Azure SignalR - Modèle ARM'
description: Dans ce guide de démarrage rapide, vous allez découvrir comment créer un service Azure SignalR à l’aide d’un modèle Azure Resource Manager (modèle ARM).
author: sffamily
ms.service: signalr
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.author: zhshang
ms.date: 10/02/2020
ms.openlocfilehash: 4ab029048b37a4dcb44ef405249dcb9e20de70cf
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94841613"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-signalr-service"></a>Démarrage rapide : Utiliser un modèle ARM pour déployer Azure SignalR Service

Ce guide de démarrage rapide décrit comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer un service Azure SignalR. Vous pouvez déployer Azure SignalR Service par le biais du portail Azure, de PowerShell ou de l’interface CLI.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure une fois que vous êtes connecté.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Bouton permettant de déployer Azure SignalR Service sur Azure à l’aide d’un modèle ARM dans le portail Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/azure-portal)

Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).
* Si vous souhaitez exécuter le code localement, [Azure PowerShell](/powershell/azure/install-az-ps).

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).
* Si vous voulez exécuter le code localement :
    * Un shell Bash (comme Git Bash, qui est inclus dans [Git pour Windows](https://gitforwindows.org)).
    * [Azure CLI](/cli/azure/install-azure-cli).

---

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-signalr/).

:::code language="json" source="~/quickstart-templates/101-signalr/azuredeploy.json":::

Le modèle définit une ressource Azure :

* [**Microsoft.SignalRService/SignalR**](/azure/templates/microsoft.signalrservice/signalr)

## <a name="deploy-the-template"></a>Déployer le modèle

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sélectionnez le lien suivant afin de déployer Azure SignalR Service en utilisant le modèle ARM dans le portail Azure :

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Bouton permettant de déployer Azure SignalR Service sur Azure à l’aide du modèle ARM dans le portail Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-signalr%2fazuredeploy.json)

Dans la page **Déployer un service Azure SignalR** :

1. Si vous le souhaitez, remplacez la valeur par défaut de l’option **Abonnement**.

2. Pour **Groupe de ressources**, sélectionnez **Créer**, entrez un nom pour le nouveau groupe de ressources, puis sélectionnez **OK**.

3. Si vous avez créé un groupe de ressources, sélectionnez une **Région** pour le groupe de ressources.

4. Si vous le souhaitez, entrez un nouveau **Nom** et la **Localisation** (par exemple **eastus2**) du service Azure SignalR. Si vous ne spécifiez pas de nom, il est généré automatiquement. La localisation du service Azure SignalR peut être identique ou différente de la région du groupe de ressources. Si vous ne spécifiez pas de localisation, elle est définie sur la même région que le groupe de ressources.

5. Choisissez le **Niveau tarifaire** (**Free_F1** ou **Standard_S1**), entrez la **Capacité** (nombre d’unités SignalR), puis choisissez le **Mode de service** **Par défaut** (nécessite un serveur hub), **Serverless** (n’autorise pas de connexion serveur) ou **Classique** (routage vers le serveur hub uniquement si le hub dispose d’une connexion au serveur). Indiquez ensuite si vous souhaitez **Activer les journaux de connectivité** ou **Activer les journaux de messagerie**.

    > [!NOTE]
    > Pour le niveau tarifaire **Free_F1**, la capacité est limitée à 1 unité.

    :::image type="content" source="./media/signalr-quickstart-azure-signalr-service-arm-template/deploy-azure-signalr-service-arm-template-portal.png" alt-text="Capture d’écran du modèle ARM utilisé pour créer un service Azure SignalR dans le portail Azure":::

6. Sélectionnez **Revoir + créer**.

7. Lisez les conditions générales, puis sélectionnez **Créer**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Si vous voulez exécuter les scripts PowerShell localement, entrez d’abord `Connect-AzAccount` pour configurer vos informations d’identification Azure.

Utilisez le code suivant pour déployer Azure SignalR Service en utilisant le modèle ARM. Le code vous invite à entrer les éléments suivants :

* Nom et région du nouveau service Azure SignalR
* Nom et région d’un nouveau groupe de ressources
* Niveau tarifaire Azure (**Free_F1** ou **Standard_S1**)
* Capacité d’unités SignalR (1, 2, 5, 10, 20, 50 ou 100)
  > [!NOTE]
  > Pour le niveau tarifaire **Free_F1**, la capacité est limitée à 1 unité.
* Mode de service : **Par défaut**, pour exiger un serveur hub, **Serverless** pour interdire toute connexion serveur, ou **Classique**, pour effectuer un routage vers un serveur hub uniquement si le hub dispose d’une connexion au serveur
* S’il faut activer les journaux pour la connectivité ou la messagerie (**true** ou **false**)

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure SignalR Service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

$priceTier = Read-Host -Prompt "Enter the pricing tier (Free_F1 or Standard_S1)"
$unitCapacity = Read-Host -Prompt "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100)"
$servicingMode = Read-Host -Prompt "Enter the service mode (Default, Serverless, or Classic)"
$enableConnectionLogs = Read-Host -Prompt "Specify whether to enable connectivity logs (true or false)"
$enableMessageLogs = Read-Host -Prompt "Specify whether to enable messaging logs (true or false)"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion

$paramObjHashTable = @{
    name = $serviceName
    location = $serviceLocation
    pricingTier = $priceTier
    capacity = [int]$unitCapacity
    serviceMode = $servicingMode
    enableConnectivityLogs = $enableConnectionLogs
    enableMessagingLogs = $enableMessageLogs
}

Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure SignalR Service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateParameterObject $paramObjHashTable `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Utilisez le code suivant pour déployer Azure SignalR Service en utilisant le modèle ARM. Le code vous invite à entrer les éléments suivants :

* Nom et région du nouveau service Azure SignalR
* Nom et région d’un nouveau groupe de ressources
* Niveau tarifaire Azure (**Free_F1** ou **Standard_S1**)
* Capacité d’unités SignalR (1, 2, 5, 10, 20, 50 ou 100)
    > [!NOTE]
    > Pour le niveau tarifaire **Free_F1**, la capacité est limitée à 1 unité.
* Mode de service : **Par défaut**, pour exiger un serveur hub, **Serverless** pour interdire toute connexion serveur, ou **Classique**, pour effectuer un routage vers un serveur hub uniquement si le hub dispose d’une connexion au serveur
* S’il faut activer les journaux pour la connectivité ou la messagerie (**true** ou **false**)

```azurecli-interactive
read -p "Enter a name for the new Azure SignalR Service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
read -p "Enter the pricing tier (Free_F1 or Standard_S1): " priceTier &&
read -p "Enter the number of SignalR units (1, 2, 5, 10, 20, 50, or 100): " unitCapacity &&
read -p "Enter the service mode (Default, Serverless, or Classic): " servicingMode &&
read -p "Specify whether to enable connectivity logs (true or false): " enableConnectionLogs &&
read -p "Specify whether to enable messaging logs (true or false): " enableMessageLogs &&
params='name='$serviceName' location='$serviceLocation' pricingTier='$priceTier' capacity='$unitCapacity' serviceMode='$servicingMode' enableConnectivityLogs='$enableConnectionLogs' enableMessagingLogs='$enableMessageLogs &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure SignalR Service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-signalr/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> Le déploiement peut prendre quelques minutes. Notez les noms du service Azure SignalR et du groupe de ressources, que vous utiliserez ultérieurement pour examiner les ressources déployées.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

# <a name="portal"></a>[Portail](#tab/azure-portal)

Suivez ces étapes pour obtenir une vue d’ensemble de votre nouveau service Azure SignalR :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **SignalR**.

2. Dans la liste SignalR, sélectionnez votre nouveau service. La page **Vue d’ensemble** pour le nouveau service Azure SignalR apparaît.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Exécutez le code interactif suivant pour visualiser les détails de votre service Azure SignalR. Vous devrez entrer le nom du nouveau service et du groupe de ressources.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure SignalR Service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzSignalR -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Exécutez le code interactif suivant pour visualiser les détails de votre service Azure SignalR. Vous devrez entrer le nom du nouveau service et du groupe de ressources.

```azurecli-interactive
read -p "Enter the name of your Azure SignalR Service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az signalr show --resource-group $resourceGroupName --name $serviceName" &&
az signalr show --resource-group $resourceGroupName --name $serviceName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Groupes de ressources**.

2. Dans la liste des groupes de ressources, choisissez le nom de votre groupe de ressources.

3. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.

4. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the name of the resource group to delete"
Write-Verbose "Remove-AzResourceGroup -Name $resourceGroupName" -Verbose
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
read -p "Enter the name of the resource group to delete: " resourceGroupName &&
echo "DELETE A RESOURCE GROUP (AND ITS RESOURCES):  az group delete --name $resourceGroupName" &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel pas à pas expliquant comment créer un modèle ARM, consultez :

> [!div class="nextstepaction"]
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

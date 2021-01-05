---
title: 'Démarrage rapide : Déployer l’API Azure pour FHIR en utilisant un modèle ARM'
description: Dans ce guide de démarrage rapide, vous allez découvrir comment déployer l’API Azure pour Fast Healthcare Interoperability Resources (FHIR®) en utilisant un modèle Azure Resource Manager (modèle ARM).
author: matjazl
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: quickstart
ms.custom: subject-armqs, devx-track-azurecli
ms.author: cavoeg
ms.date: 10/06/2020
ms.openlocfilehash: 0cc31c26a71775dfdf3e9ee94852fda8af2c0cfc
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844537"
---
# <a name="quickstart-use-an-arm-template-to-deploy-azure-api-for-fhir"></a>Démarrage rapide : Utiliser un modèle ARM pour déployer l’API Azure pour FHIR

Dans ce guide de démarrage rapide, vous allez découvrir comment utiliser un modèle Azure Resource Manager (modèle ARM) pour déployer l’API Azure pour Fast Healthcare Interoperability Resources (FHIR®). Vous pouvez déployer l’API Azure pour FHIR via le portail Azure, PowerShell ou l’interface CLI.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure une fois que vous êtes connecté.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer sur Azure un service API Azure pour FHIR en utilisant un modèle ARM dans le portail Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-api-for-fhir%2fazuredeploy.json)

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

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azure-api-for-fhir/).

:::code language="json" source="~/quickstart-templates/101-azure-api-for-fhir/azuredeploy.json":::

Le modèle définit une ressource Azure :

* **Microsoft.HealthcareApis/services**

<!--

Replace the line above with the following line once https://docs.microsoft.com/azure/templates/microsoft.healthcareapis/services goes live:

* [**Microsoft.HealthcareApis/services**](/azure/templates/microsoft.healthcareapis/services)

-->

## <a name="deploy-the-template"></a>Déployer le modèle

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sélectionnez le lien suivant afin de déployer l’API Azure pour FHIR en utilisant le modèle ARM dans le portail Azure :

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer sur Azure un service API Azure pour FHIR en utilisant le modèle ARM dans le portail Azure.":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-azure-api-for-fhir%2fazuredeploy.json)

Dans la page **Déployer l’API Azure pour FHIR** :

1. Si vous le souhaitez, remplacez la valeur par défaut de **Abonnement** par un autre abonnement.

2. Pour **Groupe de ressources**, sélectionnez **Créer**, entrez un nom pour le nouveau groupe de ressources, puis sélectionnez **OK**.

3. Si vous avez créé un groupe de ressources, sélectionnez une **Région** pour le groupe de ressources.

4. Entrez un nouveau **Nom du service** et choisissez l’**Emplacement** de l’API Azure pour FHIR. L’emplacement peut être identique ou différent de la région du groupe de ressources.

    :::image type="content" source="./media/fhir-paas-arm-template-quickstart/deploy-azure-api-fhir.png" alt-text="Déployez l’API Azure pour FHIR en utilisant le modèle ARM dans le portail Azure.":::

5. Sélectionnez **Revoir + créer**.

6. Lisez les conditions générales, puis sélectionnez **Créer**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

> [!NOTE]
> Si vous voulez exécuter les scripts PowerShell localement, entrez d’abord `Connect-AzAccount` pour configurer vos informations d’identification Azure.

Si le fournisseur de ressources `Microsoft.HealthcareApis` n’est pas déjà inscrit pour votre abonnement, vous pouvez l’inscrire avec le code interactif suivant. Pour exécuter le code dans Azure Cloud Shell, sélectionnez **Essayer** dans le coin supérieur de n’importe quel bloc de code.

```azurepowershell-interactive
Register-AzResourceProvider -ProviderNamespace Microsoft.HealthcareApis
```

Utilisez le code suivant pour déployer le service API Azure pour FHIR en utilisant le modèle ARM. Le code vous invite à entrer le nom du nouveau service FHIR, le nom d’un nouveau groupe de ressources et les emplacements de chacun d’eux.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter a name for the new Azure API for FHIR service"
$serviceLocation = Read-Host -Prompt "Enter an Azure region (for example, westus2) for the service"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group to contain the service"
$resourceGroupRegion = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"

Write-Verbose "New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion" -Verbose
New-AzResourceGroup -Name $resourceGroupName -Location $resourceGroupRegion
Write-Verbose "Run New-AzResourceGroupDeployment to create an Azure API for FHIR service using an ARM template" -Verbose
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-api-for-fhir/azuredeploy.json `
    -serviceName $serviceName `
    -location $serviceLocation
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Si le fournisseur de ressources `Microsoft.HealthcareApis` n’est pas déjà inscrit pour votre abonnement, vous pouvez l’inscrire avec le code interactif suivant. Pour exécuter le code dans Azure Cloud Shell, sélectionnez **Essayer** dans le coin supérieur de n’importe quel bloc de code.

```azurecli-interactive
az extension add --name healthcareapis
```

Utilisez le code suivant pour déployer le service API Azure pour FHIR en utilisant le modèle ARM. Le code vous invite à entrer le nom du nouveau service FHIR, le nom d’un nouveau groupe de ressources et les emplacements de chacun d’eux.

```azurecli-interactive
read -p "Enter a name for the new Azure API for FHIR service: " serviceName &&
read -p "Enter an Azure region (for example, westus2) for the service: " serviceLocation &&
read -p "Enter a name for the new resource group to contain the service: " resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group: " resourceGroupRegion &&
params='serviceName='$serviceName' location='$serviceLocation &&
echo "CREATE RESOURCE GROUP:  az group create --name $resourceGroupName --location $resourceGroupRegion" &&
az group create --name $resourceGroupName --location $resourceGroupRegion &&
echo "RUN az deployment group create, which creates an Azure API for FHIR service using an ARM template" &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-api-for-fhir/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

> [!NOTE]
> Le déploiement prend quelques minutes. Notez les noms du service API Azure pour FHIR et du groupe de ressources, que vous allez utiliser pour examiner ultérieurement les ressources déployées.

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

# <a name="portal"></a>[Portail](#tab/azure-portal)

Suivez ces étapes pour obtenir une vue d’ensemble de votre nouveau service API Azure pour FHIR :

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **API Azure pour FHIR**.

2. Dans la liste FHIR, sélectionnez votre nouveau service. La page **Vue d’ensemble** pour le nouveau service API Azure pour FHIR apparaît.

3. Pour vérifier que le compte de l’API FHIR est provisionné, sélectionnez le lien en regard de **Point de terminaison de métadonnées FHIR** pour récupérer la déclaration de capacité de l’API FHIR. Le format du lien est `https://<service-name>.azurehealthcareapis.com/metadata`. Si le compte est provisionné, un grand fichier JSON est affiché.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Exécutez le code interactif suivant pour visualiser les détails de votre service API Azure pour FHIR. Vous devrez entrer le nom du nouveau service et du groupe de ressources.

```azurepowershell-interactive
$serviceName = Read-Host -Prompt "Enter the name of your Azure API for FHIR service"
$resourceGroupName = Read-Host -Prompt "Enter the resource group name"
Write-Verbose "Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName" -Verbose
Get-AzHealthcareApisService -ResourceGroupName $resourceGroupName -Name $serviceName
Read-Host "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned"

$requestUri="https://" + $serviceName + ".azurehealthcareapis.com/metadata"
$metadata = Invoke-WebRequest -Uri $requestUri
$metadata.RawContent
Read-Host "Press [ENTER] to continue"
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Exécutez le code interactif suivant pour visualiser les détails de votre service API Azure pour FHIR. Vous devrez entrer le nom du nouveau service et du groupe de ressources.

```azurecli-interactive
read -p "Enter the name of your Azure API for FHIR service: " serviceName &&
read -p "Enter the resource group name: " resourceGroupName &&
echo "SHOW SERVICE DETAILS:  az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName" &&
az healthcareapis service show --resource-group $resourceGroupName --resource-name $serviceName &&
read -p "Press [ENTER] to fetch the FHIR API capability statement, which shows that the new service has been provisioned: " &&
requestUrl='https://'$serviceName'.azurehealthcareapis.com/metadata' &&
curl --url $requestUrl &&
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

Pour obtenir un tutoriel pas à pas expliquant comment créer un modèle ARM, consultez le [tutoriel sur la création et le déploiement de votre premier modèle ARM](../azure-resource-manager/templates/template-tutorial-create-first-template.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez déployé l’API Azure pour FHIR sur votre abonnement. Pour définir des paramètres supplémentaires dans votre API Azure pour FHIR, consultez le guide pratique relatif aux paramètres supplémentaires. Si vous êtes prêt à commencer à utiliser l’API Azure pour FHIR, découvrez plus en détail comment inscrire des applications.

>[!div class="nextstepaction"]
>[Paramètres supplémentaires dans l’API Azure pour FHIR](azure-api-for-fhir-additional-settings.md)

>[!div class="nextstepaction"]
>[Présentation de l’inscription d’applications](fhir-app-registration.md)

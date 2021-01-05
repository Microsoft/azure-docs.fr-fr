---
title: Créer des modèles d’application logique pour le déploiement
description: Découvrez comment créer des modèles Azure Resource Manager pour automatiser le déploiement dans Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 07/26/2019
ms.openlocfilehash: 4535e6bf11f8c2abf20b1b323925c3fc3299d362
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90971784"
---
# <a name="create-azure-resource-manager-templates-to-automate-deployment-for-azure-logic-apps"></a>Créez des modèles Azure Resource Manager afin d’automatiser le déploiement pour le service Azure Logic Apps

Pour vous aider à automatiser la création et le déploiement de votre application logique, cet article décrit les méthodes permettant de créer un [modèle de Azure Resource Manager](../azure-resource-manager/management/overview.md) pour votre application logique. Pour obtenir une vue d’ensemble de la structure et de la syntaxe d’un modèle qui comprend votre définition de flux de travail et d’autres ressources nécessaires pour le déploiement, consultez [Vue d’ensemble : Automatiser le déploiement pour les applications logiques avec des modèles Azure Resource Manager](logic-apps-azure-resource-manager-templates-overview.md).

Azure Logic Apps fournit un [modèle Azure Resource Manager prédéfini pour les applications logiques](https://github.com/Azure/azure-quickstart-templates/blob/master/101-logic-app-create/azuredeploy.json). Ce modèle vous permet non seulement de créer des applications logiques, mais aussi de définir les ressources et les paramètres à utiliser pour les déployer. Vous pouvez utiliser ce modèle pour vos propres scénarios professionnels, ou le personnaliser selon vos besoins.

> [!IMPORTANT]
> Assurez-vous que les connexions dans votre modèle utilisent les mêmes groupe de ressources et même emplacement Azure que votre application logique.

Pour plus d’informations sur les modèles Azure Resource Manager, consultez ces articles :

* [Structure et syntaxe du modèle Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Création de modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md)
* [Développer des modèles Azure Resource Manager pour la cohérence du cloud](../azure-resource-manager/templates/templates-cloud-consistency.md)

<a name="visual-studio"></a>

## <a name="create-templates-with-visual-studio"></a>Créer des modèles avec Visual Studio

La manière la plus simple de créer des modèles d’application logique paramétrés valides prêts pour le déploiement consiste à utiliser Visual Studio (version Community Edition gratuite ou version ultérieure) et les outils Azure Logic Apps pour Visual Studio. Vous pouvez ensuite [créer votre application logique dans Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) ou [rechercher et télécharger une application logique depuis le portail Azure dans Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md).

En téléchargeant votre application logique, vous recevez un modèle qui comprend les définitions de votre application logique et d’autres ressources telles que les connexions. Le modèle *paramètre* également, ou définit des paramètres pour, les valeurs utilisées pour déployer votre application logique et d’autres ressources. Pour fournir des valeurs pour ces paramètres, vous pouvez utiliser un fichier de paramètres séparé. De cette façon, vous pouvez modifier plus facilement ces valeurs en fonction de vos besoins en matière de déploiement. Pour plus d’informations, consultez les rubriques suivantes :

* [Créer des applications logiques avec Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md)
* [Gérer des applications logiques avec Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)

<a name="azure-powershell"></a>

## <a name="create-templates-with-azure-powershell"></a>Créer des modèles avec Azure Powershell

Vous pouvez également créer des modèles Resource Manager en utilisant Azure PowerShell avec le [module LogicAppTemplate](https://github.com/jeffhollan/LogicAppTemplateCreator). Ce module open source évalue d’abord votre application logique et toutes les connexions que celle-ci utilise. Il génère ensuite les ressources de modèle avec les paramètres nécessaires pour le déploiement.

Par exemple, supposons que votre application logique reçoive un message d’une file d’attente Azure Service Bus et qu’elle ajoute des données à une base de données Azure SQL. Le module conserve toute la logique d’orchestration et paramètre les chaînes de connexion SQL et Service Bus afin que vous puissiez fournir et modifier ces valeurs en fonction de vos besoins en matière de déploiement.

Ces exemples montrent comment créer et déployer des applications logiques à l’aide de modèles Azure Resource Manager, d’Azure Pipelines dans Azure DevOps et d’Azure PowerShell :

* [Exemple : se connecter à des files d’attente Azure Service Bus à partir d’Azure Logic Apps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-service-bus-queues-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : se connecter à des comptes de stockage Azure à partir d’Azure Logic Apps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-azure-storage-accounts-from-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : configurer une action Function App pour Azure Logic Apps](/samples/azure-samples/azure-logic-apps-deployment-samples/set-up-an-azure-function-app-action-for-azure-logic-apps-and-deploy-with-azure-devops-pipelines/)
* [Exemple : se connecter à un compte d’intégration à partir d’Azure Logic Apps](/samples/azure-samples/azure-logic-apps-deployment-samples/connect-to-an-integration-account-from-azure-logic-apps-and-deploy-by-using-azure-devops-pipelines/)

### <a name="install-powershell-modules"></a>Installer des modules PowerShell

1. Installez [Azure PowerShell](/powershell/azure/install-az-ps) si ce n’est pas déjà fait.

1. La méthode la plus simple pour installer le module LogicAppTemplate à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/LogicAppTemplate) est d’utiliser cette commande :

   ```powershell
   Install-Module -Name LogicAppTemplate
   ```

   Pour effectuer une mise à jour vers la dernière version, exécutez la commande suivante :

   ```powershell
   Update-Module -Name LogicAppTemplate
   ```

Ou, pour procéder à l’installation manuelle, suivez les étapes décrites dans GitHub pour le [Créateur du modèle d’application logique](https://github.com/jeffhollan/LogicAppTemplateCreator).

### <a name="install-azure-resource-manager-client"></a>Installer le client Azure Resource Manager

Pour que le module LogicAppTemplate fonctionne avec un jeton d’accès d’abonnement et un locataire Azure, installez l’[outil client Azure Resource Manager](https://github.com/projectkudu/ARMClient), qui est un outil en ligne de commande simple qui appelle l’API Azure Resource Manager.

Quand vous exécutez la commande `Get-LogicAppTemplate` avec cet outil, la commande obtient d’abord un jeton d’accès via l’outil ARMClient, dirige le jeton vers le script PowerShell et crée le modèle en tant que fichier JSON. Pour plus d’informations sur cet outil, consultez cet [article sur l’outil client Azure Resource Manager](https://blog.davidebbo.com/2015/01/azure-resource-manager-client.html).

### <a name="generate-template-with-powershell"></a>Générer un modèle avec PowerShell

Pour générer votre modèle après l’installation du module LogicAppTemplate et [Azure CLI](/cli/azure/), exécutez la commande PowerShell :

```powershell
$parameters = @{
    Token = (az account get-access-token | ConvertFrom-Json).accessToken
    LogicApp = '<logic-app-name>'
    ResourceGroup = '<Azure-resource-group-name>'
    SubscriptionId = $SubscriptionId
    Verbose = $true
}

Get-LogicAppTemplate @parameters | Out-File C:\template.json
```

Pour suivre la recommandation relative à la redirection dans un jeton à partir de l’[outil client Azure Resource Manager](https://github.com/projectkudu/ARMClient), exécutez cette commande à l’emplacement dans lequel `$SubscriptionId` est votre ID d’abonnement Azure :

```powershell
$parameters = @{
    LogicApp = '<logic-app-name>'
    ResourceGroup = '<Azure-resource-group-name>'
    SubscriptionId = $SubscriptionId
    Verbose = $true
}

armclient token $SubscriptionId | Get-LogicAppTemplate @parameters | Out-File C:\template.json
```

Après l’extraction, vous pouvez créer un fichier de paramètres à partir de votre modèle en exécutant la commande suivante :

```powershell
Get-ParameterTemplate -TemplateFile $filename | Out-File '<parameters-file-name>.json'
```

Pour l’extraction avec des références Azure Key Vault (statique uniquement), exécutez la commande suivante :

```powershell
Get-ParameterTemplate -TemplateFile $filename -KeyVault Static | Out-File $fileNameParameter
```

| Paramètres | Obligatoire | Description |
|------------|----------|-------------|
| TemplateFile | Oui | Le chemin d'accès à votre fichier de modèle |
| KeyVault | Non | Une énumération qui décrit comment gérer les valeurs de coffre de clés possibles. Par défaut, il s’agit de `None`. |
||||

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer des modèles d’application logique](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md)

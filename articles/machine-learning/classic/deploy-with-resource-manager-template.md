---
title: 'ML Studio (classique) : Déploiement d’espaces de travail avec Azure Resource Manager – Azure'
description: Comment déployer un espace de travail pour Azure Machine Learning Studio (classique) à l’aide d’un modèle Azure Resource Manager
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18, devx-track-azurepowershell
ms.date: 02/05/2018
ms.openlocfilehash: f860a52691d4d7fe72cbd7c8989277b224214064
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322775"
---
# <a name="deploy-azure-machine-learning-studio-classic-workspace-using-azure-resource-manager"></a>Déployer un espace de travail Azure Machine Learning Studio (classique) à l’aide d’Azure Resource Manager

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)

Les modèles de déploiement Azure Resource Manager vous font gagner du temps en vous offrant une méthode évolutive pour déployer des composants interconnectés avec un mécanisme de validation et de nouvelle tentative. Pour configurer des espaces de travail Azure Machine Learning Studio (classique), par exemple, vous devez d’abord configurer un compte de stockage Azure, puis déployer votre espace de travail. Imaginez effectuer cette opération manuellement pour des centaines d’espaces de travail. Une alternative plus simple consiste à utiliser un modèle Azure Resource Manager pour déployer un espace de travail Studio (classique) et toutes ses dépendances. Cet article vous accompagne tout au long de cette procédure pas à pas. Pour une intéressante présentation d’Azure Resource Manager, consultez [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

## <a name="step-by-step-create-a-machine-learning-workspace"></a>Pas à pas : créer un espace de travail Machine Learning
Nous créerons un groupe de ressources Azure, puis déploierons un nouveau compte de stockage Azure et un nouvel espace de travail Azure Machine Learning Studio (classique) à l’aide d’un modèle Resource Manager. Une fois le déploiement terminé, nous imprimerons des informations importantes sur les espaces de travail créés (clé primaire, workspaceID et URL de l’espace de travail).

### <a name="create-an-azure-resource-manager-template"></a>Créer un modèle Azure Resource Manager

Un espace de travail Machine Learning requiert un compte de stockage Azure pour stocker le jeu de données lié.
Le modèle suivant utilise le nom du groupe de ressources pour générer le nom du compte de stockage et celui de l’espace de travail.  Il utilise également le nom du compte de stockage comme propriété lors de la création de l’espace de travail.

```json
{
    "contentVersion": "1.0.0.0",
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "variables": {
        "namePrefix": "[resourceGroup().name]",
        "location": "[resourceGroup().location]",
        "mlVersion": "2016-04-01",
        "stgVersion": "2015-06-15",
        "storageAccountName": "[concat(variables('namePrefix'),'stg')]",
        "mlWorkspaceName": "[concat(variables('namePrefix'),'mlwk')]",
        "mlResourceId": "[resourceId('Microsoft.MachineLearning/workspaces', variables('mlWorkspaceName'))]",
        "stgResourceId": "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]",
        "storageAccountType": "Standard_LRS"
    },
    "resources": [
        {
            "apiVersion": "[variables('stgVersion')]",
            "name": "[variables('storageAccountName')]",
            "type": "Microsoft.Storage/storageAccounts",
            "location": "[variables('location')]",
            "properties": {
                "accountType": "[variables('storageAccountType')]"
            }
        },
        {
            "apiVersion": "[variables('mlVersion')]",
            "type": "Microsoft.MachineLearning/workspaces",
            "name": "[variables('mlWorkspaceName')]",
            "location": "[variables('location')]",
            "dependsOn": ["[variables('stgResourceId')]"],
            "properties": {
                "UserStorageAccountId": "[variables('stgResourceId')]"
            }
        }
    ],
    "outputs": {
        "mlWorkspaceObject": {"type": "object", "value": "[reference(variables('mlResourceId'), variables('mlVersion'))]"},
        "mlWorkspaceToken": {"type": "string", "value": "[listWorkspaceKeys(variables('mlResourceId'), variables('mlVersion')).primaryToken]"},
        "mlWorkspaceWorkspaceID": {"type": "string", "value": "[reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId]"},
        "mlWorkspaceWorkspaceLink": {"type": "string", "value": "[concat('https://studio.azureml.net/Home/ViewWorkspace/', reference(variables('mlResourceId'), variables('mlVersion')).WorkspaceId)]"}
    }
}

```
Enregistrez ce modèle en tant que fichier mlworkspace.json sous C:\temp\.

### <a name="deploy-the-resource-group-based-on-the-template"></a>Déployer le groupe de ressources à partir du modèle

* Ouvrez PowerShell
* Installez les modules d’Azure Resource Manager et d’Azure Service Management

```powershell
# Install the Azure Resource Manager modules from the PowerShell Gallery (press "A")
Install-Module Az -Scope CurrentUser

# Install the Azure Service Management modules from the PowerShell Gallery (press "A")
Install-Module Azure -Scope CurrentUser
```

   Ces étapes de téléchargement et d’installation des modules sont nécessaires pour effectuer les étapes restantes. Cette opération ne doit être effectuée qu’une fois dans l’environnement dans lequel vous exécutez les commandes PowerShell.

* Authentifiez-vous sur Azure

```powershell
# Authenticate (enter your credentials in the pop-up window)
Connect-AzAccount
```
Cette étape doit être répétée pour chaque session. Une fois que vous êtes authentifié, vos informations d’abonnement s’affichent.

![Compte Azure.](./media/deploy-with-resource-manager-template/azuresubscription.png)

Maintenant que nous avons accès à Azure, nous pouvons créer le groupe de ressources.

* Créer un groupe de ressources

```powershell
$rg = New-AzResourceGroup -Name "uniquenamerequired523" -Location "South Central US"
$rg
```

Vérifiez que le groupe de ressources est correctement configuré. **ProvisioningState** doit avoir la valeur « Réussi ».
Le nom du groupe de ressources est utilisé par le modèle pour générer le nom du compte de stockage. Le nom du compte de stockage doit comprendre entre 3 et 24 caractères, uniquement des lettres en minuscules et des nombres.

![Groupe de ressources](./media/deploy-with-resource-manager-template/resourcegroupprovisioning.png)

* À l’aide du déploiement du groupe de ressources, déployez un nouvel espace de travail Machine Learning.

```powershell
# Create a Resource Group, TemplateFile is the location of the JSON template.
$rgd = New-AzResourceGroupDeployment -Name "demo" -TemplateFile "C:\temp\mlworkspace.json" -ResourceGroupName $rg.ResourceGroupName
```

Une fois le déploiement terminé, il est simple d’accéder aux propriétés de l’espace de travail que vous avez déployé. Par exemple, vous pouvez accéder au jeton de clé primaire.

```powershell
# Access Azure Machine Learning Studio Workspace Token after its deployment.
$rgd.Outputs.mlWorkspaceToken.Value
```

Une autre méthode pour récupérer des jetons de l’espace de travail existant consiste à utiliser la commande Invoke-AzResourceAction. Par exemple, vous pouvez répertorier les jetons principaux et secondaires de tous les espaces de travail.

```powershell
# List the primary and secondary tokens of all workspaces
Get-AzResource |? { $_.ResourceType -Like "*MachineLearning/workspaces*"} |ForEach-Object { Invoke-AzResourceAction -ResourceId $_.ResourceId -Action listworkspacekeys -Force}
```
Après la configuration de l’espace de travail, vous pouvez également automatiser de nombreuses tâches Azure Machine Learning Studio (classique) à l’aide du [Module PowerShell pour Azure Machine Learning Studio (classique)](https://aka.ms/amlps).

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus, consultez [Création de modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).
* Parcourez le [Référentiel de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates).
* Regardez cette vidéo sur [Azure Resource Manager](https://channel9.msdn.com/Events/Ignite/2015/C9-39).
* Consultez l'[aide de référence sur les modèles Resource Manager](/azure/templates/microsoft.machinelearning/allversions).

<!--Link references-->
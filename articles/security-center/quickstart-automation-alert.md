---
title: Créer une automatisation de la sécurité pour des alertes de sécurité spécifiques en utilisant un modèle Azure Resource Manager (modèle ARM)
description: Découvrez comment créer une automatisation Azure Security Center pour déclencher une application logique, qui est déclenchée par des alertes Security Center spécifiques en utilisant un modèle Azure Resource Manager (modèle ARM).
services: azure-resource-manager
author: memildin
ms.service: azure-resource-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: memildin
ms.date: 08/20/2020
ms.openlocfilehash: e53a5c4840cee0651090bec0b68cc53e13102299
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705487"
---
# <a name="quickstart-create-an-automatic-response-to-a-specific-security-alert-using-an-arm-template"></a>Démarrage rapide : Créer une réponse automatique à une alerte de sécurité spécifique en utilisant un modèle ARM

Ce guide de démarrage rapide explique comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une automatisation du workflow qui déclenche une application logique lorsque des alertes de sécurité particulières sont reçues par Azure Security Center.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Pour obtenir la liste des rôles et des autorisations nécessaires à l’utilisation de la fonctionnalité d’automatisation du workflow d’Azure Security Center, consultez [Automatisation du workflow](workflow-automation.md).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-securitycenter-create-automation-for-alertnamecontains/).

:::code language="json" source="~/quickstart-templates/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json":::

### <a name="relevant-resources"></a>Ressources appropriées

- [**Microsoft.Security/automations**](/azure/templates/microsoft.security/automations) : l’automatisation qui déclenche l’application logique, à la réception d’une alerte Azure Security Center contenant une chaîne spécifique.
- [**Microsoft.Logic/workflows**](/azure/templates/microsoft.logic/workflows) : une application logique vide déclenchable.

Pour d’autres modèles de démarrage rapide Security Center, consultez ces [modèles ajoutés par la communauté](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Security&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

- **PowerShell** :

  ```azurepowershell-interactive
  New-AzResourceGroup -Name <resource-group-name> -Location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  New-AzResourceGroupDeployment -ResourceGroupName <resource-group-name> -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
  ```

- **CLI** :

  ```azurecli-interactive
  az group create --name <resource-group-name> --location <resource-group-location> #use this command when you need to create a new resource group for your deployment
  az deployment group create --resource-group <my-resource-group> --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-securitycenter-create-automation-for-alertnamecontains/azuredeploy.json
  ```

- **Portail** :

  [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-securitycenter-create-automation-for-alertnamecontains%2fazuredeploy.json)

  Pour plus d’informations sur cette option de déploiement, consultez [Utiliser un bouton de déploiement pour étendre des modèles à partir du dépôt GitHub](../azure-resource-manager/templates/deploy-to-azure-button.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Utilisez le portail Azure pour vérifier que l’automatisation du workflow a été déployée.

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez **Security Center**.
1. Dans la barre de menus supérieure, sélectionnez l’icône de filtre, puis sélectionnez l’abonnement spécifique sur lequel vous avez déployé la nouvelle automatisation du workflow.
1. Dans la barre latérale de Security Center, ouvrez **Automation du workflow** et recherchez votre nouvelle automatisation.
    :::image type="content" source="./media/quickstart-automation-alert/validating-template-run.png" alt-text="Liste des automatisations configurées" lightbox="./media/quickstart-automation-alert/validating-template-run.png":::
    >[!TIP]
    > Si vous comptez de nombreuses automatisations de workflow dans votre abonnement, utilisez l’option **filtrer par nom**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez l’automatisation du workflow par le biais du portail Azure.

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez **Security Center**.
1. Dans la barre de menus supérieure, sélectionnez l’icône de filtre, puis sélectionnez l’abonnement spécifique sur lequel vous avez déployé la nouvelle automatisation du workflow.
1. Dans la barre latérale de Security Center, ouvrez **Automation du workflow** et recherchez l’automatisation à supprimer.
    :::image type="content" source="./media/quickstart-automation-alert/deleting-workflow-automation.png" alt-text="Étapes de suppression d’une automatisation de workflow" lightbox="./media/quickstart-automation-alert/deleting-workflow-automation.png":::
1. Activez la case à cocher de l’élément à supprimer.
1. Dans la barre d’outils, sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel pas à pas vous guidant tout au long du processus de création d’un modèle, consultez :

> [!div class="nextstepaction"]
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

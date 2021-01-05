---
title: Créer votre compte Azure Maps à l’aide d’un modèle ARM | Microsoft Azure Maps
description: Découvrez comment créer un compte Azure Maps à l’aide d’un modèle Azure Resource Manager (ARM).
author: philmea
ms.author: philmea
ms.date: 10/20/2020
ms.topic: how-to
ms.service: azure-maps
ms.openlocfilehash: c715c0639e962f76f669515c1d2c826c8cf6cc9e
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92524975"
---
# <a name="create-your-azure-maps-account-using-an-arm-template"></a>Créer votre compte Azure Maps à l’aide d’un modèle ARM

Vous pouvez créer votre compte Azure Maps à l’aide d’un modèle Azure Resource Manager (ARM). Une fois que vous avez un compte, vous pouvez implémenter les API dans votre site web ou application mobile.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-maps-create/).

:::code language="json" source="~/quickstart-templates/101-maps-create/azuredeploy.json":::

La ressource du compte Azure Maps est définie dans ce modèle :

* [**Microsoft.Maps/accounts**](/azure/templates/microsoft.maps/accounts) : créer un compte Azure Maps.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée un compte Azure Maps.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-maps-create%2Fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes.

    ![Portail de déploiement du modèle ARM](./media/how-to-create-template/create-account-using-template-portal.png)

    Sauf si elle est spécifiée, utilisez la valeur par défaut pour créer votre compte Azure Maps.

    * **Abonnement** : sélectionnez un abonnement Azure.
    * **Groupe de ressources**  : sélectionnez **Créer** , entrez un nom unique pour le groupe de ressources, puis cliquez sur **OK** .
    * **Emplacement** : sélectionnez un emplacement. Par exemple, **USA Ouest 2** .
    * **Nom du compte**  : entrez un nom pour votre compte Azure Maps, qui doit être globalement unique.
    * **Niveau tarifaire**  : sélectionnez le niveau tarifaire approprié ; la valeur par défaut du modèle est S0.

3. Sélectionnez **Revoir + créer** . 
4. Confirmez vos paramètres sur la page de vérification, puis cliquez sur **Créer** . Une fois votre Azure Maps correctement déployé, vous recevez une notification :

    ![Notification du portail de déploiement du modèle ARM](./media/how-to-create-template/resource-manager-template-portal-deployment-notification.png)

Le portail Azure est utilisé pour déployer votre modèle. Vous pouvez également utiliser Azure PowerShell, Azure CLI ou l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Vous pouvez utiliser le portail Azure pour vérifier votre compte Azure Maps et visualiser vos clés. Vous pouvez également utiliser le script Azure CLI suivant pour répertorier vos clés de compte.

```azurecli-interactive
az maps account keys list --name MyMapsAccount --resource-group MyResourceGroup
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également le compte Azure Maps. Supprimer le groupe de ressources en utilisant Azure CLI :

```azurecli-interactive
az group delete --name MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Maps et Azure Resource Manager, passez aux articles ci-dessous.

- Créer une [application de démonstration](quick-demo-map-app.md) Azure Maps
- En savoir plus sur les [modèles ARM](../azure-resource-manager/templates/overview.md)
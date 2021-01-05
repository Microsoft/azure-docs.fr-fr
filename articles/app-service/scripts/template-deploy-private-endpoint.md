---
title: Utiliser un modèle Azure Resource Manager et déployer un point de terminaison privé pour une application web
description: Apprenez à utiliser un modèle ARM afin de déployer un point de terminaison privé pour votre application web.
author: ericgre
ms.assetid: 49e460d0-7759-4ceb-b5a4-f1357e4fde56
ms.topic: sample
ms.date: 07/08/2020
ms.author: ericg
ms.service: app-service
ms.workload: web
ms.openlocfilehash: acf7198477ce0c55f9321fd3f8e57294a60c0961
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89652023"
---
# <a name="create-an-app-service-app-and-deploy-a-private-endpoint-by-using-an-azure-resource-manager-template"></a>Créer une application App Service et déployer un point de terminaison privé en utilisant un modèle Azure Resource Manager

Dans ce guide de démarrage rapide, vous allez utiliser un modèle Azure Resource Manager (ARM) pour créer une application web et l’exposer avec un point de terminaison privé.

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

## <a name="prerequisite"></a>Configuration requise

Vous devez avoir un compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-private-endpoint"></a>Créer un Private Endpoint

Ce modèle crée un point de terminaison privé pour une application web Azure.

### <a name="review-the-template"></a>Vérifier le modèle

:::code language="json" source="~/quickstart-templates/101-private-endpoint-webapp/azuredeploy.json" :::

### <a name="deploy-the-template"></a>Déployer le modèle

Voici comment déployer le modèle Azure Resource Manager sur Azure :

1. Pour vous connecter à Azure et ouvrir le modèle, sélectionnez ce lien :  [Déploiement vers Azure](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-private-endpoint-webapp%2Fazuredeploy.json). Le modèle crée le réseau virtuel, l’application web, le point de terminaison privé et la zone DNS privée.
2. Sélectionnez ou créez votre groupe de ressources.
3. Entrez le nom de votre application web, du plan App Service et du point de terminaison privé.
5. Lisez la politique des conditions générales. Si vous acceptez, sélectionnez **J’accepte les conditions générales mentionnées ci-dessus** > **Acheter**. Le déploiement peut prendre plusieurs minutes.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec le point de terminaison privé, supprimez le groupe de ressources. Cette action supprime le point de terminaison privé et toutes les ressources qui y sont associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

- Vous trouverez d’autres modèles Azure Resource Manager pour les applications web Azure App Service dans les [exemples de modèle ARM](../samples-resource-manager-templates.md).

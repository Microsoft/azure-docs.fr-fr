---
title: Créer une ressource Azure Cognitive Services avec des modèles ARM | Microsoft Docs
description: Créez une ressource Azure cognitive Services avec un modèle Resource Manager (ARM).
keywords: services cognitifs, solutions cognitives, intelligence cognitive, intelligence artificielle cognitive
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: quickstart
ms.date: 09/14/2020
ms.author: aahi
ms.custom: subject-armqs, devx-track-azurecli
ms.openlocfilehash: 4afaad77fdff7a1ba80a385e9f2c3d8ddef8ab96
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842105"
---
# <a name="quickstart-create-a-cognitive-services-resource-using-an-arm-template"></a>Démarrage rapide : Créer une ressource Cognitive Services avec un modèle ARM

Ce guide de démarrage rapide décrit comment utiliser un modèle Azure Resource Manager (ARM) pour créer des services cognitifs.

La solution Azure Cognitive Services correspond à des services cloud avec des API REST et des kits SDK de bibliothèque de client destinés à aider les développeurs à intégrer une intelligence cognitive dans des applications sans connaissances ni compétences directes en intelligence artificielle (IA) ou en science des données. Azure Cognitive Services permet aux développeurs d’ajouter facilement des fonctionnalités cognitives dans leurs applications à l’aide de solutions cognitives capables de voir, entendre, parler, comprendre et même commencer à raisonner.

Créez une ressource à l’aide d’un modèle Azure Resource Manager (ARM). Avec cette ressource multiservice :

* Accédez à plusieurs services cognitifs Azure avec une seule clé et un seul point de terminaison.
* Centralisez la facturation des services que vous utilisez.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer votre service cognitif sur Azure](../media/template-deployments/deploy-to-azure.svg "Déployer votre service cognitif sur Azure")](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, [créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-cognitive-services-universalkey/).

:::code language="json" source="~/quickstart-templates/101-cognitive-services-universalkey/azuredeploy.json":::

Une seule ressource Azure est définie dans le modèle :
* [Microsoft.CognitiveServices/accounts](/azure/templates/microsoft.cognitiveservices/accounts) : crée une ressource Cognitive Services.

## <a name="deploy-the-template"></a>Déployer le modèle

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Cliquez sur le bouton **Déployer dans Azure**.

    [![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cognitive-services-universalkey%2Fazuredeploy.json)

2. Saisissez les valeurs suivantes.

    |Valeur  |Description  |
    |---------|---------|
    | **Abonnement** | Sélectionnez un abonnement Azure. |
    | **Groupe de ressources** | Sélectionnez **Créer**, entrez un nom unique pour le groupe de ressources, puis cliquez sur **OK**. |
    | **Région** | Sélectionnez une région.  Par exemple, **USA Est** |
    | **Nom du service cognitif** | Remplacez la valeur par le nom unique de votre groupe de ressources. Vous aurez besoin de ce nom dans la section suivante quand vous validerez le déploiement. |
    | **Lieu** | Remplacez la valeur par la région utilisée ci-dessus. |
    | **Sku** | Le [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/) applicable à votre ressource. |

    :::image type="content" source="media/arm-template/universal-key-portal-template.png" alt-text="Écran Création de ressources":::.

3. Sélectionnez **Vérifier + créer**, puis **Créer**. Une fois le déploiement de la ressource terminé, le bouton **Accéder à la ressource** est mis en surbrillance.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

> [!NOTE]
> `az deployment group` create requiert Azure CLI version 2.6 ou ultérieure. Pour afficher la version, tapez `az --version`. Pour plus d’informations, consultez la [documentation](/cli/azure/deployment/group).

Exécutez le script suivant à l’aide de l’interface de ligne de commande Azure CLI [sur votre machine locale](/cli/azure/install-azure-cli?view=azure-cli-latest), ou du bouton **Essayer** dans un navigateur. Entrez un nom et une localisation (par exemple, `centralus`) pour un nouveau groupe de ressources où le modèle ARM déploiera une ressource Cognitive Services. Notez le nom que vous choisissez, car vous en aurez besoin plus tard pour valider le déploiement.


```azurecli-interactive
read -p "Enter a name for your new resource group:" resourceGroupName &&
read -p "Enter the location (i.e. centralus):" location &&
templateUri="https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-cognitive-services-universalkey/azuredeploy.json" &&
az group create --name $resourceGroupName --location "$location" &&
az deployment group create --resource-group $resourceGroupName --template-uri  $templateUri &&
echo "Press [ENTER] to continue ..." &&
read
```

---

[!INCLUDE [Register Azure resource for subscription](./includes/register-resource-subscription.md)]


## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

# <a name="portal"></a>[Portail](#tab/portal)

Une fois votre déploiement terminé, vous pouvez cliquer sur le bouton **Accéder à la ressource** pour voir votre nouvelle ressource. Vous pouvez également rechercher le groupe de ressources :

1. en sélectionnant **Groupes de ressources** dans le volet de navigation de gauche ;
2. en sélectionnant le nom du groupe de ressources.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

Avec Azure CLI, exécutez le script suivant, puis entrez le nom du groupe de ressources que vous avez créé précédemment.

```azurecli-interactive
echo "Enter the resource group where the Cognitive Services resource exists:" &&
read resourceGroupName &&
az cognitiveservices account list -g $resourceGroupName
```

---


## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez nettoyer et supprimer un abonnement Cognitive Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources supprime également toutes les autres ressources se trouvant dans le groupe.

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

1. Sur le portail Azure, développez le menu de gauche pour ouvrir le menu des services, et sélectionnez **Groupes de ressources** pour afficher la liste de vos groupes de ressources.
2. Localisez le groupe de ressources contenant la ressource à supprimer.
3. Cliquez avec le bouton droit sur la liste des groupes de ressources. Sélectionnez **Supprimer le groupe de ressources** et confirmez.

# <a name="azure-cli"></a>[Azure CLI](#tab/CLI)

Avec Azure CLI, exécutez le script suivant, puis entrez le nom du groupe de ressources que vous avez créé précédemment.

```azurecli-interactive
echo "Enter the resource group name, for deletion:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

---

## <a name="next-steps"></a>Étapes suivantes

* [Authentifier des requêtes auprès d’Azure Cognitive Services](authentication.md)
* [Qu’est-ce qu’Azure Cognitive Services ?](./what-are-cognitive-services.md)
* [Prise en charge en langage naturel](language-support.md)
* [Prise en charge des conteneurs Docker](cognitive-services-container-support.md)

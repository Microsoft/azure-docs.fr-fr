---
title: Utiliser un modèle Azure Resource Manager pour déployer un service
titleSuffix: Azure Cognitive Search
description: Vous pouvez déployer une instance de service Recherche cognitive Azure rapidement à l’aide du modèle Azure Resource Manager.
manager: nitinme
author: HeidiSteen
ms.author: heidist
ms.service: cognitive-search
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 11/17/2020
ms.openlocfilehash: 12aa7f123ec5e417d710636f2afb4c60231cefc1
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94746548"
---
# <a name="quickstart-deploy-cognitive-search-using-an-arm-template"></a>Démarrage rapide : Déployer Recherche cognitive à l’aide d’un modèle Resource Manager

Cet article vous guide tout au long du processus d’utilisation d’un modèle Resource Manager (Azure Resource Manager) pour déployer une ressource Recherche cognitive Azure dans le portail Azure.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-search-create%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azure-search-create/).

:::code language="json"source="~/quickstart-templates/101-azure-search-create/azuredeploy.json":::

La ressource Azure définie dans ce modèle :

- [Microsoft.Search/searchServices](/azure/templates/Microsoft.Search/searchServices) : créer un service Recherche cognitive Azure

## <a name="deploy-the-template"></a>Déployer le modèle

Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée une ressource Recherche cognitive Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fazure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-search-create%2Fazuredeploy.json)

Le portail affiche un formulaire qui vous permet de spécifier facilement des valeurs de paramètre. Certains paramètres sont préremplis avec les valeurs par défaut du modèle. Vous devrez spécifier votre abonnement, le groupe de ressources, l’emplacement et le nom du service. Si vous souhaitez utiliser Cognitive Services dans un pipeline d’[enrichissement par IA](cognitive-search-concept-intro.md), par exemple pour analyser le texte de fichiers image binaires, choisissez un emplacement qui offre à la fois le service Recherche cognitive et Cognitive Services. Les deux services doivent se trouver dans la même région pour les charges de travail d’enrichissement par IA. Quand vous avez rempli le formulaire, vous devez accepter les conditions générales, puis sélectionner le bouton Acheter pour terminer votre déploiement.

> [!div class="mx-imgBorder"]
> ![Portail Azure - affichage du modèle](./media/search-get-started-arm/arm-portalscrnsht.png)

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

Quand votre déploiement est terminé, vous pouvez accéder au nouveau groupe de ressources et au nouveau service de recherche dans le portail.

## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres guides de démarrage rapide et tutoriels sur le service Recherche cognitive reposent sur ce guide de démarrage rapide. Si vous prévoyez d’utiliser les guides de démarrage rapide et tutoriels suivants, il peut être utile de conserver ces ressources. Si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources. Dans ce cas, le service Recherche cognitive et les ressources associées seront également supprimés.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un service Recherche cognitive en utilisant un modèle Resource Manager, puis vous avez validé le déploiement. Pour plus d’informations sur le service Recherche cognitive et Azure Resource Manager, consultez les articles ci-dessous.

- Consultez la [vue d’ensemble de Recherche cognitive Azure](search-what-is-azure-search.md).
- [Créer un index](search-get-started-portal.md) pour votre service de recherche.
- [Créer une application de démonstration](search-create-app-portal.md) à l’aide de l’Assistant du portail.
- [Créer un ensemble de compétences](cognitive-search-quickstart-blob.md) pour extraire des informations de vos données.

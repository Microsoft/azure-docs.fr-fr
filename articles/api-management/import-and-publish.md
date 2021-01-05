---
title: Tutoriel - Importer et publier votre première API dans Gestion des API Azure
description: Dans ce tutoriel, vous importez une API de la spécification OpenAPI dans Gestion des API Azure, puis vous testez votre API dans le portail Azure.
author: mikebudzynski
ms.service: api-management
ms.custom: mvc
ms.topic: tutorial
ms.date: 09/30/2020
ms.author: apimpm
ms.openlocfilehash: 9ff64f57e61002101b4e2c560bdcd91863cc461e
ms.sourcegitcommit: d479ad7ae4b6c2c416049cb0e0221ce15470acf6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/01/2020
ms.locfileid: "91626951"
---
# <a name="tutorial-import-and-publish-your-first-api"></a>Tutoriel : Importer et publier votre première API

Ce tutoriel montre comment importer une API back-end de la spécification OpenAPI au format JSON dans Gestion des API Azure. Microsoft fournit l’API back-end utilisée dans cet exemple et l’héberge sur Azure à l’adresse [https://conferenceapi.azurewebsites.net?format=json](https://conferenceapi.azurewebsites.net?format=json).

Une fois que vous avez importé l’API back-end dans Gestion des API, votre API du service Gestion des API devient une façade pour l’API back-end. Vous pouvez personnaliser la façade selon vos besoins dans Gestion des API sans toucher à l’API back-end. Pour plus d’informations, consultez [Transformer et protéger votre API](transform-api.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Importer une API dans Gestion des API
> * Tester l’API dans le portail Azure

Après l’importation, vous pouvez gérer l’API dans le portail Azure.

:::image type="content" source="media/import-and-publish/created-api.png" alt-text="Nouvelle API dans Gestion des API":::

## <a name="prerequisites"></a>Prérequis

- Comprendre la [terminologie relative au service Gestion des API Azure](api-management-terminology.md)
- [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-backend-api"></a>Importer et publier une API back-end

Cette section montre comment importer et publier une API back-end de la spécification OpenAPI.

1. Dans le volet de navigation gauche de votre instance du service Gestion des API, sélectionnez **API**.
1. Sélectionnez la vignette **OpenAPI**.
1. Dans la fenêtre **Créer à partir de la spécification OpenAPI**, sélectionnez **Complet**.
1. Entrez les valeurs du tableau suivant. Sélectionnez ensuite **Créer** pour créer votre API.

   Vous pouvez définir les valeurs de l’API au moment de la création, ou un peu plus tard en accédant à l’onglet **Paramètres**.

   :::image type="content" source="media/import-and-publish/create-api.png" alt-text="Nouvelle API dans Gestion des API":::


   |Paramètre|Valeur|Description|
   |-------|-----|-----------|
   |**Spécification OpenAPI**|*https:\//conferenceapi.azurewebsites.net?format=json*|Service qui implémente l’API. Gestion des API transmet les demandes à cette adresse.|
   |**Nom complet**|Une fois que vous avez entré l’URL de service précédente, le service Gestion des API remplit ce champ en fonction du contenu JSON.|Nom affiché dans le [portail des développeurs](api-management-howto-developer-portal.md).|
   |**Nom**|Une fois que vous avez entré l’URL de service précédente, le service Gestion des API remplit ce champ en fonction du contenu JSON.|Nom unique de l’API.|
   |**Description**|Une fois que vous avez entré l’URL de service précédente, le service Gestion des API remplit ce champ en fonction du contenu JSON.|Description facultative de l’API.|
   |**Modèle d’URL**|**HTTPS**|Protocoles pouvant accéder à l’API.|
   |**Suffixe de l’URL de l’API**|*conference*|Suffixe ajouté à l’URL de base du service Gestion des API. Le service Gestion des API distingue les API par leur suffixe. Ainsi, ce dernier doit être unique pour chaque API d’un éditeur donné.|
   |**Balises**| |Étiquettes permettant d’organiser les API pour la recherche, le regroupement ou le filtrage.|
   |**Produits**|**Illimité**|Association d’une ou de plusieurs API. Chaque instance du service Gestion des API est fournie avec deux exemples de produits : **Starter** et **Unlimited**. Vous publiez une API en l’associant à un produit, **Unlimited** (Illimité) dans cet exemple.<br/><br/> Vous pouvez inclure plusieurs API dans un produit et les proposer aux développeurs via le portail des développeurs. Pour ajouter cette API à un autre produit, tapez ou sélectionnez le nom du produit. Répétez cette étape pour ajouter l’API à plusieurs produits. Vous pouvez également ajouter des API aux produits plus tard à partir de la page **Paramètres**.<br/><br/>  Pour plus d’informations sur les produits, consultez [Créer et publier un produit](api-management-howto-add-products.md).|
   |**Passerelles**|**Managé**|La ou les passerelles API qui exposent l’API. Ce champ est disponible uniquement dans les services des niveaux **Développeur** et **Premium**.<br/><br/>**Managée** indique la passerelle intégrée au service Gestion des API et hébergée par Microsoft dans Azure. Les [passerelles auto-hébergées](self-hosted-gateway-overview.md) sont disponibles seulement dans les niveaux de service Premium et Développeur. Vous pouvez les déployer localement ou dans d’autres clouds.<br/><br/> Si aucune passerelle n’est sélectionnée, l’API n’est pas disponible et vos demandes d’API n’aboutissent pas.|
   |**Créer une version pour cette API ?**|Sélectionner ou désélectionner|Pour plus d’informations, consultez [Publier plusieurs versions d’une API de manière prévisible](api-management-get-started-publish-versions.md).|

   > [!NOTE]
   > Pour publier l’API à l’adresse des consommateurs d’API, vous devez l’associer à un produit.

2. Sélectionnez **Create** (Créer).

Si vous rencontrez des problèmes au moment de l’importation d’une définition d’API, consultez la [liste des problèmes et restrictions connus](api-management-api-import-restrictions.md).

## <a name="test-the-new-api-in-the-azure-portal"></a>Tester la nouvelle API dans le Portail Azure

Vous pouvez appeler les opérations d’API directement à partir du portail Azure, qui offre un moyen pratique de voir et de tester les opérations.

1. Dans le volet de navigation gauche de votre instance du service Gestion des API, sélectionnez **API** > **Demo Conference API**.
1. Sélectionnez l’onglet **Test**, puis sélectionnez **GetSpeakers**. La page affiche les **paramètres de requête** et les **en-têtes**, le cas échéant. La valeur de **Ocp-Apim-Subscription-Key** est indiquée automatiquement pour la clé d’abonnement associée à cette API.
1. Sélectionnez **Envoyer**.

   :::image type="content" source="media/import-and-publish/01-import-first-api-01.png" alt-text="Nouvelle API dans Gestion des API":::

   Le back-end répond avec **200 OK** et des données.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Importer votre première API
> * Tester l’API dans le portail Azure

Passez au tutoriel suivant pour découvrir comment créer et publier un produit :

> [!div class="nextstepaction"]
> [Créer et publier un produit](api-management-howto-add-products.md)

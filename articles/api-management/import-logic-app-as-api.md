---
title: Importer une application logique en tant qu’API avec le portail Azure | Microsoft Docs
description: Cet article vous montre comment utiliser le service Gestion des API (APIM) pour importer une application logique en tant qu’API.
services: api-management
documentationcenter: ''
author: vladvino
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 04/22/2020
ms.author: apimpm
ms.openlocfilehash: 0dd460bce01b86c72d0b8fd70351fdcc85c69bd4
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93146958"
---
# <a name="import-a-logic-app-as-an-api"></a>Importer une application logique en tant qu’API

Cet article explique comment importer une application logique en tant qu’API, et tester l’API importée.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
>
> -   Importer une application logique en tant qu’API
> -   Tester l’API dans le portail Azure
> -   Tester l’API dans le portail des développeurs

## <a name="prerequisites"></a>Prérequis

-   Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)
-   Assurez-vous que votre abonnement contient une application logique qui expose un point de terminaison HTTP. Pour plus d’informations, consultez l’article [Appeler, déclencher ou imbriquer des workflows via des points de terminaison HTTP dans des applications logiques](../logic-apps/logic-apps-http-endpoint.md).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="import-and-publish-a-back-end-api"></a><a name="create-api"> </a>Importer et publier une API back-end

1. À partir du portail Azure, accédez à votre service Gestion des API, puis sélectionnez **API** dans le menu.
2. Sélectionnez **Application logique** dans la liste **Ajouter une nouvelle API**.

    ![Application logique](./media/import-logic-app-as-api/logic-app-api.png)

3. Appuyez sur **Parcourir** pour afficher la liste des applications logiques avec déclencheur HTTP de votre abonnement. (Notez que les applications logiques sans déclencheur HTTP n’apparaissent pas dans la liste.)
4. Sélectionnez l’application. Gestion des API recherche le swagger associé à l’application sélectionnée, l’extrait et l’importe.
5. Ajoutez un suffixe d’URL d’API. Le suffixe est un nom qui identifie cette API spécifique dans cette instance Gestion des API. Il doit être unique dans cette instance Gestion des API.
6. Publiez l’API en l’associant à un produit. Dans ce cas, le produit « _Illimité_ » est utilisé. Si vous souhaitez que l’API soit publiée et mise à la disposition des développeurs, ajoutez-la à un produit. Vous pouvez effectuer cette opération durant la création de l’API ou ultérieurement.

    Les produits sont des associations d’une ou de plusieurs API. Vous pouvez inclure un certain nombre d’API et les proposer aux développeurs dans le portail des développeurs. Les développeurs doivent s’abonner à un produit pour obtenir l’accès à l’API. Quand ils s’abonnent à un produit, ils obtiennent une clé d’abonnement qui est valable pour toutes les API de ce produit. Si vous avez créé l’instance Gestion des API, vous êtes abonné à chaque produit par défaut, car vous êtes déjà administrateur.

    Par défaut, chaque instance Gestion des API est fournie avec deux exemples de produits :

    - **Starter**
    - **Illimité**

7. Entrez d’autres paramètres d’API. Vous pouvez définir les valeurs lors de la création, ou les configurer ultérieurement en accédant à l’onglet **Paramètres**. Les paramètres sont expliqués dans le tutoriel [Importer et publier votre première API](import-and-publish.md#import-and-publish-a-backend-api).
8. Sélectionnez **Create** (Créer).

## <a name="test-the-api-in-the-azure-portal"></a>Tester l’API dans le portail Azure

Les opérations peuvent être directement appelées depuis le portail Azure, qui permet d’afficher et de tester les opérations d’une API.

1. Sélectionnez l’API que vous avez créée à l’étape précédente.
2. Appuyez sur l’onglet **Test**.
3. Sélectionnez une opération.

    La page affiche des champs pour les paramètres de requête et des champs pour les en-têtes. L’un des en-têtes est « Ocp-Apim-Subscription-Key », pour la clé d’abonnement du produit qui est associé à cette API. Si vous avez créé l’instance Gestion des API, la clé est renseignée automatiquement, car vous êtes déjà administrateur.

4. Appuyez sur **Envoyer**.

    Le serveur principal répond avec **200 OK** et certaines données.

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

>[!NOTE]
>Chaque application logique a une opération **manual-invoke**. Si vous souhaitez que votre API soit composée de plusieurs applications logiques afin d’éviter les conflits, vous devez renommer la fonction.

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>
> [Transformer et protéger une API publiée](transform-api.md)

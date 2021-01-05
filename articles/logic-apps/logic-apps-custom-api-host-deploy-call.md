---
title: Appeler des API web et API REST personnalisées
description: Appeler vos propres API web et API REST à partir d’Azure Logic Apps
services: logic-apps
ms.suite: integration
ms.reviewer: jonfan, logicappspm
ms.topic: article
ms.date: 05/13/2020
ms.openlocfilehash: 7b4d00e8c0366d10fddafa66db699c1a59fd9ad7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83659775"
---
# <a name="deploy-and-call-custom-apis-from-workflows-in-azure-logic-apps"></a>Déployer et appeler des API personnalisées à partir de workflows dans Azure Logic Apps

Après avoir [créé vos propres API](./logic-apps-create-api-app.md) en vue de les utiliser dans vos workflows d’application logique, vous devez déployer ces API avant de pouvoir les appeler. Vous pouvez déployer vos API en tant [qu’applications web](../app-service/overview.md). Toutefois, envisagez de les déployer sous la forme [d’applications API](../app-service/app-service-web-tutorial-rest-api.md) pour simplifier votre travail lorsque vous générez, hébergez et consommez des API dans le cloud et en local. Vous n’avez pas besoin de modifier le code dans vos API. Il vous suffit de déployer votre code dans une application API. Vous pouvez héberger vos API sur [Azure App Service](../app-service/overview.md), une offre PaaS (platform as a service) qui propose un hébergement d’API simple et hautement évolutif.

Même si vous pouvez appeler n’importe quelle API à partir d’une application logique, vous bénéficierez d’une expérience optimale si vous ajoutez des [métadonnées Swagger](https://swagger.io/specification/) qui décrivent les opérations et les paramètres de votre API. Ce document Swagger facilite l’intégration de votre API aux applications logiques et améliore son fonctionnement avec ces dernières.

## <a name="deploy-your-api-as-a-web-app-or-api-app"></a>Déployer votre API en tant qu’application web ou application API

Pour pouvoir appeler votre API personnalisée à partir d’une application logique, déployez votre API en tant qu’application web ou application API sur Azure App Service. Pour que votre document Swagger puisse être lu par le concepteur Logic Apps, définissez les propriétés de définition d’API et activez le [partage de ressources Cross-Origin (CORS)](../app-service/overview.md) pour votre application web ou application API.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez votre application web ou votre application API.

2. Dans le menu de l’application qui s’affiche, sous **API**, choisissez **Définition de l’API**. Sous **Emplacement de la définition de l’API**, spécifiez l’URL de votre fichier swagger.json.

   En règle générale, l’URL se présente sous le format suivant :`https://{name}.azurewebsites.net/swagger/docs/v1)`

   ![Lien vers le document Swagger pour votre API personnalisée](./media/logic-apps-custom-api-deploy-call/custom-api-swagger-url.png)

3. Sous **API**, choisissez **CORS**. Dans le cadre de la stratégie CORS, sous **Origines autorisées**, spécifiez **'*'** (tout autoriser).

   Ce paramètre autorise les demandes à partir du concepteur d’application logique.

   ![Autoriser les demandes du concepteur d’application logique à votre API personnalisée](./media/logic-apps-custom-api-deploy-call/custom-api-cors.png)

Pour plus d’informations, consultez [Héberger une API RESTful avec CORS dans Azure App Service](../app-service/app-service-web-tutorial-rest-api.md).

## <a name="call-your-custom-api-from-logic-app-workflows"></a>Appeler votre API personnalisée à partir de flux de travail d’application logique

Après avoir configuré les propriétés de définition d’API et le partage CORS, vous devriez être en mesure d’inclure les déclencheurs et actions de votre API personnalisée dans votre workflow d’application logique. 

*  Pour visualiser les sites web disposant d’URL OpenAPI, vous pouvez parcourir vos sites web d’abonnement dans le Concepteur Logic Apps.

*  Pour afficher les actions et entrées disponibles en pointant sur un document Swagger, utilisez [l’action HTTP + Swagger](../connectors/connectors-native-http-swagger.md).

*  Pour appeler une API, y compris celles qui n’ont pas ou n’exposent pas de document Swagger, vous pouvez toujours créer une requête avec l’[action HTTP](../connectors/connectors-native-http.md).

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble des connecteurs personnalisés](../logic-apps/custom-connector-overview.md)

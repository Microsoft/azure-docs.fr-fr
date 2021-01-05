---
title: Métadonnées OpenAPI dans Azure Functions
description: Vue d’ensemble de la prise en charge d’OpenAPI dans les fonctions Azure
ms.topic: conceptual
ms.date: 03/23/2017
ms.openlocfilehash: 499d4f685e3802fcc37c8a3050ae367207f192d2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000201"
---
# <a name="openapi-20-metadata-support-in-azure-functions-preview"></a>Prise en charge des métadonnées OpenAPI 2.0 dans Azure Functions (préversion)
La prise en charge des métadonnées OpenAPI 2.0 (anciennement Swagger) dans Azure Functions est une fonctionnalité en préversion que vous pouvez utiliser pour écrire une définition OpenAPI 2.0 dans une application de fonction. Vous pouvez ensuite héberger ce fichier à l’aide de l’application de fonction.

> [!IMPORTANT]
> La fonctionnalité en préversion OpenAPI est disponible aujourd’hui seulement dans le runtime 1.x. Vous pouvez trouver plus d’informations sur la création d’une application de fonction 1.x [ici](./functions-versions.md#creating-1x-apps).

Les [métadonnées OpenAPI](https://swagger.io/) permettent à une grande variété d’autres logiciels de consommer une fonction hébergeant une API REST. Ces logiciels incluent les offres Microsoft comme PowerApps et la [fonctionnalité API Apps d’Azure App Service](../app-service/overview.md), des outils de développeurs tiers comme [Postman](https://www.getpostman.com/docs/importing_swagger) et [beaucoup d’autres packages](https://swagger.io/tools/).

[!INCLUDE [intro](../../includes/functions-bindings-intro.md)]

>[!TIP]
>Nous vous conseillons de commencer par le [didacticiel de mise en route](./functions-openapi-definition.md) , puis de revenir à ce document pour en savoir plus sur les fonctionnalités spécifiques.

## <a name="enable-openapi-definition-support"></a><a name="enable"></a>Activer la prise en charge de la définition OpenAPI
Vous pouvez configurer tous les paramètres OpenAPI dans la page **Définition d’API**, dans les **Fonctionnalités de la plateforme** de votre application de fonction.

> [!NOTE]
> La fonctionnalité Définition de l’API de fonction n’est pas prise en charge pour le runtime bêta.

Pour activer la génération d’une définition OpenAPI hébergée et d’une définition de démarrage rapide, définissez **Source de définition d’API** sur **Fonction (préversion)** . **L’URL externe** permet à votre fonction d’utiliser une définition OpenAPI qui est hébergée ailleurs.

## <a name="generate-a-swagger-skeleton-from-your-functions-metadata"></a><a name="generate-definition"></a>Générer une structure Swagger à partir des métadonnées de votre fonction
Un modèle peut vous aider à commencer à écrire votre première définition OpenAPI. La fonctionnalité de modèle de définition crée une définition OpenAPI partielle à l’aide de toutes les métadonnées dans le fichier function.json, pour chacune de vos fonctions de déclencheur HTTP. Vous devrez renseigner plus d’informations sur votre API à partir de la [spécification OpenAPI](https://swagger.io/specification/), comme les modèles de requête et de réponse.

Pour des instructions détaillées, consultez le [didacticiel de prise en main](./functions-openapi-definition.md).

### <a name="available-templates"></a><a name="templates"></a>Modèles disponibles

|Nom| Description |
|:-----|:-----|
|Définition générée|Une définition OpenAPI avec la quantité maximale d’informations pouvant être déduites à partir de métadonnées existantes de la fonction.|

### <a name="included-metadata-in-the-generated-definition"></a><a name="quickstart-details"></a>Métadonnées incluses dans la définition générée

Le tableau suivant représente les paramètres du portail Azure et les données correspondantes dans function.json telles qu’elles sont mappées sur la structure Swagger générée.

|Swagger.json|Interface utilisateur du portail|Function.json|
|:----|:-----|:-----|
|[Hôte](https://swagger.io/specification/#fixed-fields-15)|**Paramètres de l’application de fonctions** > **Paramètres App Service** > **Vue d’ensemble** > **URL**|*Non présent*
|[Chemins d’accès](https://swagger.io/specification/#paths-object-29)|**Intégrer** > **Méthodes HTTP sélectionnées**|Liaisons : Routage
|[Élément du chemin d’accès](https://swagger.io/specification/#path-item-object-32)|**Intégrer** > **Modèle de routage**|Liaisons : Méthodes
|[Sécurité](https://swagger.io/specification/#security-scheme-object-112)|**Clés**|*Non présent*|
|operationID*|**Itinéraire + verbes autorisés**|Itinéraire + verbes autorisés|

\*L’ID de l’opération est nécessaire uniquement pour l’intégration avec PowerApps et Flow.
> [!NOTE]
> L’extension x-ms-summary fournit un nom d’affichage dans Logic Apps, Flow et PowerApps.
>
> Consultez [Personnaliser votre définition Swagger pour PowerApps](/connectors/custom-connectors/openapi-extensions) pour en savoir plus.

## <a name="use-cicd-to-set-an-api-definition"></a><a name="CICD"></a>Définir une définition d’API avec CI/CD

 Vous devez activer l’hébergement de la définition d’API dans le portail avant d’activer le contrôle de code source pour modifier votre définition d’API à partir du contrôle de code source. Suivez ces instructions :

1. Accédez à **Définition d’API (préversion)** dans vos paramètres d’application de fonction.
   1. Définissez **Source de définition d’API** sur **Fonction**.
   1. Cliquez sur **Générer le modèle de définition d’API**, puis sur **Enregistrer** pour créer une définition de modèle à modifier ultérieurement.
   1. Notez votre URL de définition d’API et votre clé.
1. [Configurez l’intégration continue/le déploiement continu (CI/CD)](./functions-continuous-deployment.md#requirements-for-continuous-deployment).
2. Modifiez swagger.json dans le contrôle de code source à l’adresse \site\wwwroot\.azurefunctions\swagger\swagger.json.

Désormais, les modifications apportées à swagger.json dans votre référentiel sont hébergées par votre application de fonction au niveau de l’URL de définition d’API et de la clé que vous avez notées à l’étape 1.c.

## <a name="next-steps"></a>Étapes suivantes
* [Didacticiel de prise en main](./functions-openapi-definition.md). Essayez notre procédure détaillée pour voir une définition OpenAPI en action.
* [Référentiel GitHub Azure Functions](https://github.com/Azure/Azure-Functions/). Consultez le référentiel Functions pour nous donner votre avis sur la préversion du support de définition d’API. Créez un ticket GitHub pour les mises à jour que vous souhaitez.
* [Référence du développeur Azure Functions](functions-reference.md). Apprenez-en davantage sur le codage de fonctions et la définition de déclencheurs et de liaisons.

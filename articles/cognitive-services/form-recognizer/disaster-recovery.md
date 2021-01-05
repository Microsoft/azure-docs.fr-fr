---
title: Guide de reprise d’activité pour Azure Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser l’API de copie de modèle pour sauvegarder vos ressources Form Recognizer.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 05/27/2020
ms.author: pafarley
ms.openlocfilehash: 747ceb0106f437f9e2442c2b8c68c0b73a9107a6
ms.sourcegitcommit: 02ed9acd4390b86c8432cad29075e2204f6b1bc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97808252"
---
# <a name="back-up-and-recover-your-form-recognizer-models"></a>Sauvegarder et récupérer vos modèles Form Recognizer

Quand vous créez une ressource Form Recognizer dans le portail Azure, vous spécifiez une région. Ensuite, votre ressource et toutes ses opérations restent associées à cette région de serveur Azure. Il est rare, mais pas impossible, de rencontrer un problème réseau qui touche une région entière. Si votre solution doit toujours être disponible, vous devez la concevoir pour qu’elle bascule vers une autre région ou qu’elle répartisse la charge de travail entre deux régions ou plus. Les deux approches nécessitent au moins deux ressources Form Recognizer dans différentes régions et la possibilité de synchroniser des modèles personnalisés entre les régions.

L’API de copie autorise ce scénario en vous permettant de copier des modèles personnalisés d’un compte Form Recognizer vers d’autres, qui peuvent exister dans n’importe quelle région géographique prise en charge. Ce guide vous montre comment utiliser l’API REST de copie avec cURL. Vous pouvez également utiliser un service de requête HTTP tel que Postman pour émettre les demandes.

## <a name="business-scenarios"></a>Scénarios d’entreprise

Si votre application ou votre entreprise dépend de l’utilisation d’un modèle personnalisé Form Recognizer, nous vous recommandons de copier votre modèle vers un autre compte Form Recognizer dans une autre région. En cas de panne régionale, vous pouvez alors accéder à votre modèle dans la région où il a été copié.

##  <a name="prerequisites"></a>Prérequis

1. Deux ressources Azure Form Recognizer dans différentes régions Azure. Si vous ne les avez pas, accédez au portail Azure et <a href="https://ms.portal.azure.com/#create/Microsoft.CognitiveServicesFormRecognizer" title="créez une ressource Form Recognizer" target="_blank">Créer une ressource Form Recognizer <span class="docon docon-navigate-external x-hidden-focus"></span></a>.
1. La clé d’abonnement, l’URL de point de terminaison et l’ID d’abonnement de votre ressource Form Recognizer. Ces valeurs se trouvent sous l’onglet **Vue d’ensemble** de la ressource sur le portail Azure.


## <a name="copy-api-overview"></a>Vue d’ensemble de l’API de copie

Le processus de copie d’un modèle personnalisé se compose des étapes suivantes :

1. Tout d’abord, vous émettez une demande d’autorisation de copie vers la ressource cible&mdash;autrement dit, la ressource qui recevra le modèle copié. Vous obtenez l’URL du modèle cible qui vient d’être créé, qui recevra les données copiées.
1. Ensuite, vous envoyez la demande de copie à la ressource source&mdash;la ressource qui contient le modèle à copier. Vous obtenez une URL que vous pouvez interroger pour suivre la progression de l’opération.
1. Vous utilisez vos informations d’identification de ressource source pour interroger l’URL de progression jusqu’à ce que l’opération soit couronnée de succès. Vous pouvez également interroger le nouvel ID de modèle dans la ressource cible pour obtenir l’état du nouveau modèle.

> [!CAUTION]
> L’API de copie ne prend actuellement pas en charge les ID de modèle pour les [modèles personnalisés composés](https://westcentralus.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2-1-preview-2/operations/Compose). La composition de modèles est une fonctionnalité d’évaluation dans la préversion v2.1-preview.2. 

## <a name="generate-copy-authorization-request"></a>Générer la demande d’autorisation de copie

La requête HTTP suivante obtient l’autorisation de copie de votre ressource cible. Vous devez entrer le point de terminaison et la clé de votre ressource cible en tant qu’en-têtes.

```
POST https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Vous obtenez une réponse `201\Created` dont le corps contient une valeur `modelId`. Cette chaîne est l’ID du modèle (vide) récemment créé. L’`accessToken` est nécessaire pour que l’API copie les données dans cette ressource, tandis que la valeur `expirationDateTimeTicks` représente l’expiration du jeton. Enregistrez ces trois valeurs dans un emplacement sécurisé.

```
HTTP/1.1 201 Created
Location: https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d
{"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
```

## <a name="start-copy-operation"></a>Démarrer l’opération de copie

La requête HTTP suivante démarre l’opération de copie sur la ressource source. Vous devez entrer le point de terminaison et la clé de votre ressource source en tant qu’en-têtes. Notez que l’URL de la demande contient l’ID du modèle source que vous souhaitez copier.

```
POST https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/<your model ID>/copy HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Le corps de la demande doit avoir le format suivant. Vous devez entrer l’ID de ressource et le nom de la région de votre ressource cible. Vous pouvez trouver votre ID de ressource sous l’onglet **Propriétés** de votre ressource dans le portail Azure, et le nom de la région sous l’onglet **Clés et points de terminaison**. Vous aurez également besoin de l’ID de modèle, du jeton d’accès et de la valeur d’expiration que vous avez copiés à l’étape précédente.

```json
{
   "targetResourceId": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}",  
   "targetResourceRegion": "{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}",
   "copyAuthorization": {"modelId":"<your model ID>","accessToken":"<your access token>","expirationDateTimeTicks":637233481531659440}
}
```

> [!NOTE]
> L’API Copy prend en charge de manière transparente la fonctionnalité [AEK/CMK](https://msazure.visualstudio.com/Cognitive%20Services/_wiki/wikis/Cognitive%20Services.wiki/52146/Customer-Managed-Keys). L’opération ne nécessite aucun traitement spécial, mais notez que si vous copiez une ressource non chiffrée dans une ressource chiffrée, vous devez inclure l’en-tête de demande `x-ms-forms-copy-degrade: true`. Si cet en-tête n’est pas inclus, l’opération de copie échoue et retourne une erreur `DataProtectionTransformServiceError`.

Vous obtenez une réponse `202\Accepted` avec un en-tête Operation-Location. Cette valeur est l’URL que vous utiliserez pour suivre la progression de l’opération. Copiez-la dans un emplacement temporaire pour l’étape suivante.

```
HTTP/1.1 202 Accepted
Operation-Location: https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1
```

### <a name="common-errors"></a>Erreurs courantes

|Error|Résolution|
|:--|:--|
| 400 / Requête incorrecte avec `"code:" "1002"` | Indique une erreur de validation ou un format de requête de copie incorrect. Les problèmes courants sont les suivants : a) charge utile `copyAuthorization` non valide ou modifiée. b) valeur expirée pour le jeton `expirationDateTimeTicks` (la charge utile `copyAuhtorization` est valide pendant 24 heures). c) valeur `targetResourceRegion` non valide ou non prise en charge. d) chaîne `targetResourceId` non valide ou incorrecte.
|

## <a name="track-copy-progress"></a>Suivre la progression de la copie

Suivez votre progression en interrogeant l’API **Obtenir le résultat de la copie du modèle** sur le point de terminaison de la ressource source.

```
GET https://{SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/eccc3f13-8289-4020-ba16-9f1d1374e96f/copyresults/02989ba8-1296-499f-aaf4-55cfff41b8f1 HTTP/1.1
Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

La réponse varie selon l’état de l’opération. Recherchez le champ `"status"` dans le corps JSON. Si vous automatisez cet appel d’API dans un script, nous vous recommandons d’interroger l’opération une fois par seconde.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"status":"succeeded","createdDateTime":"2020-04-23T18:18:01.0275043Z","lastUpdatedDateTime":"2020-04-23T18:18:01.0275048Z","copyResult":{}}
```

### <a name="common-errors"></a>Erreurs courantes

|Error|Résolution|
|:--|:--|
|"errors":[{"code":"AuthorizationError",<br>"message":"Authorization failure due to <br>missing or invalid authorization claims."}]   | Cette erreur s’affiche si le contenu ou la charge utile `copyAuthorization` varie par rapport à la valeur retournée par l’API `copyAuthorization`. Vérifiez que la charge utile est exactement identique à celle retournée par l’appel `copyAuthorization` précédent.|
|"errors":[{"code":"AuthorizationError",<br>"message":"Could not retrieve authorization <br>metadata. If this issue persists use a different <br>target model to copy into."}] | Indique que la charge utile `copyAuthorization` est réutilisée avec une demande de copie. Une demande de copie qui aboutit n’autorisera aucune autre demande utilisant la même charge utile `copyAuthorization`. Si une erreur se produit (comme celles indiquées ci-dessous) et que vous relancez la copie avec la même charge utile d’autorisation, cette erreur est générée. La solution consiste à générer une nouvelle charge utile `copyAuthorization` puis à émettre à nouveau la demande de copie.|
|"errors":[{"code":"DataProtectionTransformServiceError",<br>"message":"Data transfer request is not allowed <br>as it downgrades to a less secure data protection scheme. Refer documentation or contact your service administrator <br>for details."}]    | Se produit lors de la copie d’une ressource `AEK` activée vers une ressource `AEK` non activée. Pour autoriser la copie d’un modèle chiffré vers la cible en tant que modèle non chiffré, spécifiez l’en-tête `x-ms-forms-copy-degrade: true` dans la demande de copie.|
|"errors":[{"code":"ResourceResolverError",<br>"message":"Could not fetch information for Cognitive resource with Id '...'. Ensure the resource is valid and exists in the specified region 'westus2'.."}] | Indique que la ressource Azure spécifiée par `targetResourceId` n’est pas une ressource Cognitive valide ou n’existe pas. Vérifiez et réexécutez la requête de copie pour résoudre ce problème.|


### <a name="optional-track-the-target-model-id"></a>[Facultatif] Suivre l’ID de modèle cible 

Vous pouvez également utiliser l’API **Obtenir un modèle personnalisé** pour suivre l’état de l’opération en interrogeant le modèle cible. Appelez cette API à l’aide de l’ID de modèle cible que vous avez copié à la première étape.

```
GET https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/33f4d42c-cd2f-4e74-b990-a1aeafab5a5d HTTP/1.1
Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}
```

Dans le corps de la réponse, vous verrez des informations sur le modèle. Vérifiez l’état du modèle dans le champ `"status"`.

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
{"modelInfo":{"modelId":"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d","status":"ready","createdDateTime":"2020-02-26T16:59:28Z","lastUpdatedDateTime":"2020-02-26T16:59:34Z"},"trainResult":{"trainingDocuments":[{"documentName":"0.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"1.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"2.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"3.pdf","pages":1,"errors":[],"status":"succeeded"},{"documentName":"4.pdf","pages":1,"errors":[],"status":"succeeded"}],"errors":[]}}
```

## <a name="curl-sample-code"></a>Exemple de code cURL

Les extraits de code suivants utilisent cURL pour effectuer les appels d’API décrits dans les étapes ci-dessus. Vous devez toujours renseigner les ID de modèle et les informations d’abonnement propres à vos ressources.

### <a name="generate-copy-authorization-request"></a>Générer la demande d’autorisation de copie

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" 
```

### <a name="start-copy-operation"></a>Démarrer l’opération de copie

```bash
curl -i -X POST "https://{TARGET_FORM_RECOGNIZER_RESOURCE_ENDPOINT}/formrecognizer/v2.0/custom/models/copyAuthorization" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {TARGET_FORM_RECOGNIZER_RESOURCE_API_KEY}" --data-ascii "{ \"targetResourceId\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_ID}\",   \"targetResourceRegion\": \"{TARGET_AZURE_FORM_RECOGNIZER_RESOURCE_REGION_NAME}\", \"copyAuthorization\": "{\"modelId\":\"33f4d42c-cd2f-4e74-b990-a1aeafab5a5d\",\"accessToken\":\"1855fe23-5ffc-427b-aab2-e5196641502f\",\"expirationDateTimeTicks\":637233481531659440}"}"
```

### <a name="track-copy-progress"></a>Suivre la progression de la copie

```bash
curl -i GET "https://<SOURCE_FORM_RECOGNIZER_RESOURCE_ENDPOINT>/formrecognizer/v2.0/custom/models/{SOURCE_MODELID}/copyResults/{RESULT_ID}" -H "Content-Type: application/json" -H "Ocp-Apim-Subscription-Key: {SOURCE_FORM_RECOGNIZER_RESOURCE_API_KEY}"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à utiliser l’API de copie pour sauvegarder vos modèles personnalisés dans une ressource Form Recognizer secondaire. Explorez à présent la documentation de référence sur les API pour voir ce que vous pouvez faire d’autre avec Form Recognizer.
* [Documentation de référence sur l’API REST](https://westus2.dev.cognitive.microsoft.com/docs/services/form-recognizer-api-v2/operations/AnalyzeWithCustomForm)

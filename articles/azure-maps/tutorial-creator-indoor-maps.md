---
title: 'Tutoriel : Utiliser Microsoft Azure Maps Creator (préversion) pour créer des cartes d’intérieur'
description: Tutoriel sur la façon d’utiliser Microsoft Azure Maps Creator (préversion) pour créer des cartes d’intérieur
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: tutorial
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: eab8a2729209bb0023662b652f862b4fa678470e
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905721"
---
# <a name="tutorial-use-creator-preview-to-create-indoor-maps"></a>Tutoriel : Utiliser Creator (préversion) pour créer des cartes d’intérieur

> [!IMPORTANT]
> Les services d’Azure Maps Creator sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).



Ce didacticiel montre comment créer des cartes d’intérieur. Ce tutoriel explique comment utiliser l’API pour effectuer les opérations suivantes :

> [!div class="checklist"]
> * Télécharger votre package de dessin de carte d’intérieur
> * Convertir votre package de dessin en données cartographiques
> * Créer un jeu de données à partir de vos données cartographiques
> * Créer un tileset à partir des données de votre jeu de données
> * Interroger l’API de service de caractéristique web Azure Maps pour découvrir les caractéristiques de votre carte
> * Créer un stateset de caractéristique à l’aide des caractéristiques de votre carte et des données de votre jeu de données
> * Mettre à jour votre stateset de caractéristique

## <a name="prerequisites"></a>Prérequis

Pour créer des cartes d’intérieur, vous devez effectuer les opérations suivantes :

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
3. [Créer une ressource Creator (préversion)](how-to-manage-creator.md)
4. Téléchargez l’[exemple de package de dessin](https://github.com/Azure-Samples/am-creator-indoor-data-examples/blob/master/Sample%20-%20Contoso%20Drawing%20Package.zip).

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

>[!IMPORTANT]
> Les url d’API dans ce document peuvent devoir être ajustées en fonction de l'emplacement de votre ressource du Créateur. Pour plus d’informations, consultez [l’accès aux services du Créateur](how-to-manage-creator.md#access-to-creator-services).

## <a name="upload-a-drawing-package"></a>Charger un package de dessin

Utilisez l’[API de chargement de données](/rest/api/maps/data/uploadpreview) pour charger le package de dessin dans les ressources Azure Maps.

L’API de chargement de données est une transaction de longue durée qui implémente le modèle défini ici. Une fois l’opération terminée, nous allons utiliser l’`udid` pour accéder au package chargé afin de le convertir. Suivez les étapes ci-dessous pour obtenir l’`udid`.

1. Ouvrez l’application Postman. En haut de l'application Postman, sélectionnez **New** (Nouveau). Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Collection**.  Nommez la collection puis sélectionnez le bouton **Create** (Créer).

2. Pour créer la demande, sélectionnez **New** à nouveau. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Request name** (Nom de demande) pour la demande. Sélectionnez la collection que vous avez créée à l’étape précédente, puis sélectionnez **Enregistrer**.

3. Sélectionnez la méthode HTTP **POST** sous l’onglet du générateur, puis entrez l’URL suivante pour charger le package de dessin dans le service Azure Maps. Pour cette requête et d’autres requêtes mentionnées dans cet article, remplacez `{Azure-Maps-Primary-Subscription-key}` par votre clé d’abonnement principale.

    ```http
    https://atlas.microsoft.com/mapData/upload?api-version=1.0&dataFormat=zip&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. Sous l’onglet **En-têtes**, spécifiez une valeur pour la clé `Content-Type`. Le package de dessin étant un dossier compressé, utilisez la valeur `application/octet-stream`. Sous l’onglet **Corps**, sélectionnez **binaire**. Cliquez sur **Sélectionner un fichier**, puis choisissez un package de dessin.

     ![data-management](./media/tutorial-creator-indoor-maps/enter-content-type-dialog.png)

5. Cliquez sur le bouton bleu **Envoyer**, puis attendez que la requête soit traitée. Une fois la requête terminée, accédez à l’onglet **En-têtes** de la réponse. Copiez la valeur de la **Emplacement**, qui est l’URL `status URL`.

6. Pour vérifier l’état de l’appel d’API, créez une requête HTTP **GET** sur `status URL`. Vous devez ajouter votre clé d’abonnement principale à l’URL pour l’authentification. La requête **GET** doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/mapData/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

7. Lorsque la requête HTTP **GET** se termine avec succès, un `resourceLocation`est retourné. Le `resourceLocation` contient également la valeur unique `udid` pour les données chargées. Vous pouvez éventuellement utiliser l’URL `resourceLocation` pour récupérer les métadonnées de cette ressource à l’étape suivante.

    ```json
    {
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0"
    }
    ```

8. Pour récupérer les métadonnées de contenu, créez une requête HTTP **GET** sur l’URL `resourceLocation` que vous avez récupérée à l’étape 7. Vous devez ajouter votre clé d’abonnement principale à l’URL pour l’authentification. La requête **GET** doit ressembler à l’URL suivante :

    ```http
   https://atlas.microsoft.com/mapData/metadata/{udid}?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

9. Lorsque la requête HTTP **GET** se termine avec succès, le corps de la réponse contient un `udid` spécifié au `resourceLocation` de l’étape 7, l’emplacement où accéder au contenu et le télécharger à l’avenir, ainsi que d’autres métadonnées relatives au contenu, telles que la date de création/mise à jour, la taille, etc. Voici un exemple de réponse globale :

    ```json
    {
        "udid": "{udid}",
        "location": "https://atlas.microsoft.com/mapData/{udid}?api-version=1.0",
        "created": "2020-02-03T02:32:25.0509366+00:00",
        "updated": "2020-02-11T06:12:13.0309351+00:00",
        "sizeInBytes": 766,
        "uploadStatus": "Completed"
    }
    ```

## <a name="convert-a-drawing-package"></a>Convertir un package de dessin

 À présent que le package de dessin est chargé, nous allons utiliser `udid` pour le convertir en données cartographiques. L’API de conversion utilise une transaction de longue durée qui implémente le modèle défini [ici](creator-long-running-operation.md). Une fois l’opération terminée, nous allons utiliser la valeur `conversionId` pour accéder aux données converties. Suivez les étapes ci-dessous pour obtenir la valeur `conversionId`.

1. Sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Sélectionnez la méthode HTTP **POST** sous l’onglet du générateur, puis entrez l’URL suivante pour convertir votre package de dessin chargé en données cartographiques. Utilisez l’`udid` pour le package chargé.

    ```http
    https://atlas.microsoft.com/conversion/convert?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0&udid={udid}&inputType=DWG
    ```

    >[!IMPORTANT]
    > Les url d’API dans ce document peuvent devoir être ajustées en fonction de l'emplacement de votre ressource du Créateur. Pour plus d’informations, consultez [Accéder aux services de Creator (préversion)](how-to-manage-creator.md#access-to-creator-services).

3. Cliquez sur le bouton **Envoyer**, puis attendez que la requête soit traitée. Une fois la requête terminée, accédez à l’onglet **En-têtes** de la réponse, puis recherchez la clé **Emplacement**. Copiez la valeur de la clé **Emplacement** qui est `status URL` pour la demande de conversion. Vous en aurez besoin lors de l'étape suivante.

    :::image type="content" source="./media/tutorial-creator-indoor-maps/copy-location-uri-dialog.png" border="true" alt-text="Copier la valeur de la clé Emplacement":::

4. Démarrez une nouvelle méthode HTTP **GET** sous l’onglet du générateur. Ajoutez votre clé d’abonnement principale Azure Maps à `status URL`. Envoyez une requête **GET** à la `status URL` que vous avez copiée à l'étape 3. La `status URL` est semblable à l'URL suivante :

    ```http
    https://atlas.microsoft.com/conversion/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

    Si le processus de conversion n’est pas encore terminé, il se peut que vous voyiez quelque chose ressemblant la réponse JSON suivante :

    ```json
    {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Running"
    }
    ```

5. Une fois la requête correctement exécutée, un message d’état de réussite apparaît dans le corps de la réponse.  Copiez la valeur `conversionId` à partir de l’URL `resourceLocation` pour le package converti. La valeur `conversionId` est utilisée par une autre API pour accéder aux données cartographiques converties.

    ```json
   {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
        "properties": {}
    }
    ```

>[!NOTE]
>L’application Postman ne prend pas en charge en mode natif les requêtes HTTP de longue durée. Vous constaterez donc peut-être un long délai lors de la création d’une requête **GET** à l’URL d’état.  Attendez environ trente secondes et essayez de cliquer de nouveau sur le bouton **Envoyer** jusqu’à ce que la réponse indique une réussite ou un échec.

L’exemple de package de dessin doit être converti sans erreur ou avertissement. Toutefois, si vous recevez des erreurs ou des avertissements de votre propre package de dessin, la réponse JSON vous donne un lien pointant vers le [visualiseur d’erreur de dessin](drawing-error-visualizer.md). Le visualiseur d’erreur de dessin vous permet d’examiner les détails des erreurs et avertissements. Pour recevoir des recommandations sur la résolution des erreurs et avertissements de conversion, consultez [Erreurs et avertissements liés à la conversion de dessin](drawing-conversion-error-codes.md).

```json
{
    "operationId": "<operationId>",
    "created": "2020-04-22T19:39:54.9518496+00:00",
    "status": "Failed",
    "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
    "properties": {
        "diagnosticPackageLocation": "https://atlas.microsoft.com/mapData/ce61c3c1-faa8-75b7-349f-d863f6523748?api-version=1.0"
    }
}
```

## <a name="create-a-dataset"></a>Créer un jeu de données

Le jeu de données est une collection de caractéristiques cartographiques, telles que des bâtiments, des niveaux et des salles. Pour créer un jeu de données, utilisez l’[API de création de jeu de données](/rest/api/maps/dataset/createpreview). L’API de création jeu de données prend la valeur `conversionId` pour le package de dessin converti et retourne une valeur `datasetId` du jeu de données créé. Les étapes ci-dessous vous montrent comment créer un jeu de données.

1. Dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Adressez une requête **POST** à l’[API de création de jeu de données](/rest/api/maps/dataset/createpreview) pour créer un jeu de données. Avant de soumettre la requête, ajoutez à la fois votre clé d’abonnement et la valeur `conversionId` avec la valeur `conversionId` obtenue pendant le processus de conversion à l’étape 5.  La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/dataset/create?api-version=1.0&conversionID={conversionId}&type=facility&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Obtenez la valeur `statusURL` de la clé **Emplacement** des **En-têtes** de réponse.

4. Effectuez une requête **GET** à l’URL `statusURL` pour obtenir la valeur `datasetId`. Ajoutez votre clé d’abonnement principale Azure Maps pour l’authentification. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/dataset/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

5. Lorsque la requête HTTP **GET** se termine correctement, l’en-tête de réponse contient la valeur `datasetId` pour le jeu de données créé. Copiez la valeur `datasetId`. Vous devez utiliser la valeur `datasetId` pour créer un tileset.

    ```json
    {
        "operationId": "<operationId>",
        "created": "2020-04-22T19:52:38.9352189+00:00",
        "status": "Succeeded",
        "resourceLocation": "https://azure.microsoft.com/dataset/{datasetiId}?api-version=1.0"
     }
    ```

## <a name="create-a-tileset"></a>Créer un tileset

Un tileset est un ensemble de vignettes vectorielles qui s’affichent sur la carte. Des tilesets sont créés à partir de jeux de données existants. Toutefois, un tileset est indépendant du jeu de données source. Si le jeu de données est supprimé, le tileset continue d’exister. Pour créer un tileset, suivez les étapes ci-dessous :

1. Dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Effectuez une requête **POST** sous l’onglet du générateur. L’URL de la requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/tileset/create/vector?api-version=1.0&datasetID={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Effectuez une requête **GET** à l’URL `statusURL` pour le tileset. Ajoutez votre clé d’abonnement principale Azure Maps pour l’authentification. La requête doit ressembler à l’URL suivante :

   ```http
    https://atlas.microsoft.com/tileset/operations/<operationId>?api-version=1.0&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

4. Lorsque la requête HTTP **GET** se termine correctement, l’en-tête de réponse contient la valeur `tilesetId` pour le tileset créé. Copiez la valeur `tilesetId`.

    ```json
    {
        "operationId": "<operationId>",
        "createdDateTime": "3/11/2020 8:45:13 PM +00:00",
        "status": "Succeeded",
        "resourceLocation": "https://atlas.microsoft.com/tileset/{tilesetId}?api-version=1.0"
    }
    ```

## <a name="query-datasets-with-wfs-api"></a>Interroger des jeux de données avec l’API de service de caractéristique web

 Des jeux de données peuvent être interrogés à l’aide de l’[API de service de caractéristique web](/rest/api/maps/wfs). L’API de service de caractéristique web vous permet d’interroger des collections de caractéristiques, une collection spécifique ou une caractéristique spécifique avec un **ID** de caractéristique. L’**ID** de caractéristique identifie de façon unique la caractéristique à l’intérieur du jeu de données. Il est utilisé, par exemple, pour identifier l’état de caractéristique à mettre à jour dans un stateset donné.

1. Dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Effectuez une requête **GET** pour afficher la liste des collections dans votre jeu de données. Remplacez la valeur `<dataset-id>` par vôtre valeur `datasetId`. Utilisez votre clé primaire Azure Maps au lieu de l’espace réservé. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

3. Le corps de la réponse est fourni au format GeoJSON, et contient toutes les collections dans le jeu de données. Par souci de simplicité, l’exemple fourni ici affiche uniquement la collection `unit`. Pour voir un exemple contenant tous les collections, consultez [API Describe Collections du service de caractéristique web](/rest/api/maps/wfs/collectiondescriptionpreview). Pour en savoir plus sur une collection, vous pouvez cliquer sur l’une des URL à l’intérieur de l’élément `link`.

    ```json
    {
    "collections": [
        {
            "name": "unit",
            "description": "A physical and non-overlapping area which might be occupied and traversed by a navigating agent. Can be a hallway, a room, a courtyard, etc. It is surrounded by physical obstruction (wall), unless the is_open_area attribute is equal to true, and one must add openings where the obstruction shouldn't be there. If is_open_area attribute is equal to true, all the sides are assumed open to the surroundings and walls are to be added where needed. Walls for open areas are represented as a line_element or area_element with is_obstruction equal to true.",
            "links": [
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/definition?api-version=1.0",
                    "rel": "describedBy",
                    "title": "Metadata catalogue for unit"
                },
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?api-version=1.0",
                    "rel": "data",
                    "title": "unit"
                }
                {
                    "href": "https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit?api-version=1.0",
                    "rel": "self",
                    "title": "Metadata catalogue for unit"
                }
            ]
        },
    ```

4. Effectuez une requête **GET** pour les collections de caractéristiques de l’`unit`.  Remplacez la valeur `{datasetId}` par vôtre valeur `datasetId`. Utilisez votre clé primaire Azure Maps au lieu de l’espace réservé. Le corps de la réponse contient toutes les caractéristiques de la collection `unit`. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/wfs/datasets/{datasetId}/collections/unit/items?subscription-key={Azure-Maps-Primary-Subscription-key}&api-version=1.0
    ```

5. Copiez l’`id` de caractéristique pour une caractéristique d’unité qui a des propriétés de style modifiables de façon dynamique.  Étant donné que l’état d’occupation et la température de l’unité peuvent être mis à jour de façon dynamique, nous allons utiliser cet `id` de caractéristique dans la section suivante. Dans l’exemple suivant, l’`id` de caractéristique est « UNIT26 ». Nous allons faire référence aux propriétés de style de cette caractéristique en tant qu’états. et nous allons utiliser la caractéristique pour créer un stateset.

     ```json
    {
        "type": "FeatureCollection",
        "features": [
            {
                "type": "Feature",
                "geometry": {
                    "type": "Polygon",
                    "coordinates": ["..."]
                },
                "properties": {
                    "original_id": "b7410920-8cb0-490b-ab23-b489fd35aed0",
                    "category_id": "CTG8",
                    "is_open_area": true,
                    "navigable_by": [
                        "pedestrian"
                    ],
                    "route_through_behavior": "allowed",
                    "level_id": "LVL14",
                    "occupants": [],
                    "address_id": "DIR1",
                    "name": "157"
                },
                "id": "UNIT26",
                "featureType": ""
            }, {"..."}
        ]
    }
    ```

## <a name="create-a-feature-stateset"></a>Créer un stateset de caractéristique

1. Dans l’application Postman, sélectionnez **Nouveau**. Dans la fenêtre **Create New** (Créer nouveau), sélectionnez **Request** (Demande). Entrez un **Nom de demande**, puis sélectionnez une collection. Cliquez sur **Enregistrer**.

2. Adressez une requête **POST** à l’[API de création de stateset](/rest/api/maps/featurestate/createstatesetpreview). Utilisez l’ID `datasetId` du jeu de données contenant l’état que vous souhaitez modifier. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/featureState/stateset?api-version=1.0&datasetId={datasetId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

3. Dans les **En-têtes** de la requête **POST**, définissez `Content-Type` sur `application/json`. Dans le **Corps**, fournissez les styles ci-dessous pour refléter les modifications apportées aux *états* `occupied` et `temperature`. Quand vous avez terminé, cliquez sur **Envoyer**.

    ```json
    {
       "styles":[
          {
             "keyname":"occupied",
             "type":"boolean",
             "rules":[
                {
                   "true":"#FF0000",
                   "false":"#00FF00"
                }
             ]
          },
          {
             "keyname":"temperature",
             "type":"number",
             "rules":[
                {
                   "range":{
                      "exclusiveMaximum":66
                   },
                   "color":"#00204e"
                },
                {
                   "range":{
                      "minimum":66,
                      "exclusiveMaximum":70
                   },
                   "color":"#0278da"
                },
                {
                   "range":{
                      "minimum":70,
                      "exclusiveMaximum":74
                   },
                   "color":"#187d1d"
                },
                {
                   "range":{
                      "minimum":74,
                      "exclusiveMaximum":78
                   },
                   "color":"#fef200"
                },
                {
                   "range":{
                      "minimum":78,
                      "exclusiveMaximum":82
                   },
                   "color":"#fe8c01"
                },
                {
                   "range":{
                      "minimum":82
                   },
                   "color":"#e71123"
                }
             ]
          }
       ]
    }
    ```

4. Copiez la l’ID `statesetId` à partir du corps de la réponse.

5. Créez une requête **POST** pour mettre à jour l’état : Transmettez la valeur statesetId et l’`ID` de caractéristique avec votre clé d’abonnement Azure Maps. La requête doit ressembler à l’URL suivante :

    ```http
    https://atlas.microsoft.com/featureState/state?api-version=1.0&statesetID={statesetId}&featureID={featureId}&subscription-key={Azure-Maps-Primary-Subscription-key}
    ```

6. Dans les **En-têtes** de la requête **POST**, définissez `Content-Type` sur `application/json`. Dans le **CORPS** de la requête **POST**, copiez et collez le code JSON de l’exemple ci-dessous.

    ```json
    {
        "states": [
            {
                "keyName": "occupied",
                "value": true,
                "eventTimestamp": "2019-11-14T17:10:20"
            }
        ]
    }
    ```

    >[!NOTE]
    > La mise à jour n’est enregistrée que si l’horodatage de la publication est postérieur à celui de la requête précédente. Nous pouvons transmettre n’importe quelle valeur keyname configurée lors de la création.

7. Une fois la mise à jour réussie, vous recevez un code d’état HTTP `200 OK`. Si vous avez une [application de style dynamique implémentée](indoor-map-dynamic-styling.md) pour une carte d’intérieur, la mise à jour s’affiche dans votre carte rendue à l’horodatage spécifié.

L’[API d’obtention d’états de caractéristique](/rest/api/maps/featurestate/getstatespreview) vous permet de récupérer l’état d’une caractéristique à l’aide de son `ID` de caractéristique. Vous pouvez également supprimer le stateset et ses ressources à l’aide de l’[API de suppression d’état de caractéristique](/rest/api/maps/featurestate/deletestatesetpreview).

Pour en savoir plus sur les différents services Azure Maps Creator (préversion) abordés dans cet article, consultez [Creator pour cartes d’intérieur](creator-indoor-maps.md).

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment utiliser le module de cartes d’intérieur, consultez

> [!div class="nextstepaction"]
> [Utiliser le module Cartes d’intérieur](how-to-use-indoor-module.md)
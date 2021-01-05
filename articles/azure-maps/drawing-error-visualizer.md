---
title: Utiliser le Visualiseur d’erreurs de dessin Azure Maps
description: Cet article explique comment visualiser les avertissements et les erreurs retournés par l’API de conversion Creator (préversion).
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: db88e347e12783205ea8c31fed0bb374fccb4736
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903579"
---
# <a name="using-the-azure-maps-drawing-error-visualizer-with-creator-preview"></a>Utilisation du Visualiseur d’erreurs de dessin Azure Maps avec Creator (préversion)

> [!IMPORTANT]
> Les services Azure Maps Creator sont disponibles en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).


Le Visualiseur d’erreurs de dessin est une application web autonome qui affiche les [Avertissements et erreurs du package de dessin](drawing-conversion-error-codes.md) détectés pendant le processus de conversion. L’application web du Visualiseur d’erreurs se compose d’une page statique que vous pouvez utiliser sans vous connecter à Internet.  Vous pouvez utiliser le Visualiseur d’erreurs pour corriger les erreurs et les avertissements conformément aux [Exigences du package de dessin](drawing-requirements.md). L’[API de conversion Azure Maps](/rest/api/maps/conversion) retourne une réponse avec un lien vers le Visualiseur d’erreurs uniquement quand une erreur est détectée.

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir télécharger le Visualiseur d’erreurs de dessin, vous devez :

1. [Créer un compte Azure Maps](quick-demo-map-app.md#create-an-azure-maps-account)
2. [Obtenir une clé d’abonnement principale](quick-demo-map-app.md#get-the-primary-key-for-your-account), également appelée clé primaire ou clé d’abonnement.
3. [Créer une ressource Creator (préversion)](how-to-manage-creator.md)

Ce tutoriel utilise l’application [Postman](https://www.postman.com/), mais vous pouvez choisir un autre environnement de développement d’API.

## <a name="download"></a>Téléchargement

1. Chargez votre package de dessin dans le service Azure Maps Creator (préversion) afin d’obtenir un `udid` pour le package chargé. Pour connaître les étapes de chargement d’un package, consultez [Charger un package de dessin](tutorial-creator-indoor-maps.md#upload-a-drawing-package).

2. À présent que le package de dessin est chargé, nous allons utiliser `udid` pour le convertir en données cartographiques. Pour connaître les étapes de conversion d’un package, consultez [Convertir un package de dessin](tutorial-creator-indoor-maps.md#convert-a-drawing-package).

    >[!NOTE]
    >Si le processus de conversion réussit, vous ne recevez pas de lien vers l’outil Visualiseur d’erreurs.

3. Sous l’onglet **En-têtes** de la réponse dans l’application Postman, recherchez la propriété `diagnosticPackageLocation` retournée par l’API de conversion. La réponse doit ressembler au code JSON suivant :

    ```json
    {
        "operationId": "77dc9262-d3b8-4e32-b65d-74d785b53504",
        "created": "2020-04-22T19:39:54.9518496+00:00",
        "status": "Failed",
        "resourceLocation": "https://atlas.microsoft.com/conversion/{conversionId}?api-version=1.0",
        "properties": {
            "diagnosticPackageLocation": "https://atlas.microsoft.com/mapData/ce61c3c1-faa8-75b7-349f-d863f6523748?api-version=1.0"
        }
    }
    ```

4. Téléchargez le Visualiseur d’erreurs du package de dessin en effectuant une requête `HTTP-GET` sur l’URL `diagnosticPackageLocation`.

## <a name="setup"></a>Programme d’installation

Dans le package compressé téléchargé à partir du lien `diagnosticPackageLocation`, vous trouverez deux fichiers.

* _VisualizationTool.zip_ : contient le code source, le média et la page web pour le Visualiseur d’erreurs de dessin.
* _ConversionWarningsAndErrors.json_ : contient une liste mise en forme d’avertissements, d’erreurs et de détails supplémentaires utilisés par le Visualiseur d’erreurs de dessin.

Décompressez le dossier _VisualizationTool.zip_. Il contient les fichiers suivants :

* Dossier _assets_ : contient des images et des fichiers multimédias.
* Dossier _static_ : code source.
* Fichier _index.html_ : l’application web.

Ouvrez le fichier _index.html_ dans l’un des navigateurs ci-dessous avec le numéro de version respectif. Vous pouvez utiliser une autre version si celle-ci offre un comportement identique à celui de la version indiquée.

* Microsoft Edge 80
* Safari 13
* Chrome 80
* Firefox 74

## <a name="using-the-drawing-error-visualizer-tool"></a>Utilisation de l’outil Visualiseur d’erreurs de dessin

Après le lancement de l’outil Visualiseur d’erreurs de dessin, la page de chargement s’affiche. La page de chargement contient une zone glisser-déposer. La zone glisser-déposer fonctionne également comme bouton qui lance une boîte de dialogue de l’Explorateur de fichiers.

:::image type="content" source="./media/drawing-errors-visualizer/start-page.png" alt-text="Application Visualiseur d’erreurs de dessin – Page de démarrage":::

Le fichier _ConversionWarningsAndErrors.json_ a été placé à la racine du répertoire téléchargé. Pour charger le fichier _ConversionWarningsAndErrors.json_, vous pouvez le glisser-déposer sur la zone ou cliquer sur la zone, rechercher le fichier dans la boîte de dialogue de l’Explorateur de fichiers, puis le charger.

:::image type="content" source="./media/drawing-errors-visualizer/loading-data.gif" alt-text="Application Visualiseur d’erreurs de dessin – Glisser-déposer pour charger des données":::

Une fois le fichier _ConversionWarningsAndErrors.json_ chargé, vous voyez s’afficher la liste de vos avertissements et erreurs de package de dessin. Chaque erreur ou avertissement est spécifié par sa couche, son niveau et un message détaillé. Pour voir des informations détaillées sur une erreur ou un avertissement, cliquez sur le lien **Détails**. Une section inflexible s’affiche alors en dessous de la liste. Vous pouvez maintenant accéder à chaque erreur pour en savoir plus sur la manière de la résoudre.

:::image type="content" source="./media/drawing-errors-visualizer/errors.png" alt-text="Application Visualiseur d’erreurs de dessin – Erreurs et avertissements":::

## <a name="next-steps"></a>Étapes suivantes

Une fois que votre [Package de dessin répond aux exigences](/rest/api/maps/conversion), vous pouvez utiliser le [service Jeu de données Azure Maps](drawing-requirements.md) pour convertir le package en un jeu de données. Ensuite, vous pouvez utiliser le module web Cartes d’intérieur pour développer votre application. Pour en savoir plus, consultez les articles suivants :

> [!div class="nextstepaction"]
> [Codes d’erreur de conversion de dessin](drawing-conversion-error-codes.md)

> [!div class="nextstepaction"]
> [Creator (préversion) pour cartes d’intérieur](creator-indoor-maps.md)

> [!div class="nextstepaction"]
> [Utiliser le module Cartes d’intérieur](how-to-use-indoor-module.md)

> [!div class="nextstepaction"]
> [Implémenter des styles dynamiques pour les cartes d’intérieur](indoor-map-dynamic-styling.md)
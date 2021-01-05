---
title: Erreurs et avertissements liés à la conversion de dessin d’Azure Maps
description: Découvrez les erreurs et avertissements liés à la conversion que vous pouvez rencontrer quand vous utilisez le service de conversion d’Azure Maps. Lisez les recommandations relatives à la résolution des erreurs et avertissements, avec quelques exemples.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philMea
ms.openlocfilehash: cecc19f0984ce1801d50e5cbda73e98a01e2825b
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96906214"
---
# <a name="drawing-conversion-errors-and-warnings"></a>Erreurs et avertissements liés à la conversion de dessin

Le [service de conversion d’Azure Maps](/rest/api/maps/conversion) vous permet de convertir les packages de dessin chargés en données cartographiques. Les packages de dessin doivent respecter les [exigences du package de dessin](drawing-requirements.md). Si une ou plusieurs exigences ne sont pas satisfaites, le service de conversion renverra des erreurs ou avertissements. Cet article répertorie les codes d’erreur et d’avertissement liés à la conversion, ainsi que des recommandations sur la façon de les résoudre. Il fournit également des exemples de dessins qui peuvent provoquer le renvoi de ces codes par le service de conversion.

Le service de conversion fonctionnera s’il y a des avertissements de conversion. Toutefois, il est recommandé d’examiner et de résoudre tous les avertissements. Un avertissement signifie qu’une partie de la conversion a été ignorée ou corrigée automatiquement. L’échec de la résolution des avertissements pourrait entraîner des erreurs dans les derniers processus.

## <a name="general-warnings"></a>Avertissements généraux

### <a name="geometrywarning"></a>**Avertissement geometryWarning**

#### <a name="description-for-geometrywarning"></a>*Description de l’avertissement geometryWarning*

Un avertissement **geometryWarning** se produit quand le dessin contient une entité non valide. Une entité non valide est une entité qui n’est pas conforme aux contraintes géométriques. Des entités non valides sont, par exemple, un polygone avec auto-intersection ou une polyligne non fermée dans un calque qui ne prend en charge qu’une géométrie fermée.

Le service de conversion ne peut pas créer une caractéristique de carte à partir d’une entité non valide et l’ignore.

#### <a name="examples-for-geometrywarning"></a>*Exemples d’avertissement geometryWarning*

* Les deux images ci-dessous présentent des exemples de polygones avec auto-intersection.

     ![Exemple de polygone avec auto-intersection, exemple un.](./media/drawing-conversion-error-codes/geometry-warning-1.png)

     ![Exemple de polygone avec auto-intersection, exemple deux.](./media/drawing-conversion-error-codes/geometry-warning-2.png)

* Vous trouverez ci-dessous une image illustrant une polyligne non fermée. Supposons que le calque ne prend en charge qu’une géométrie fermée.

    ![Exemple de polyligne non fermée](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-geometrywarning"></a>*Comment corriger un avertissement geometryWarning*

Inspectez l’avertissement **geometryWarning** pour chaque entité afin de vérifier qu’elle respecte les contraintes géométriques.

### <a name="unexpectedgeometryinlayer"></a>**Avertissement unexpectedGeometryInLayer**

#### <a name="description-for-unexpectedgeometryinlayer"></a>*Description de l’avertissement unexpectedGeometryInLayer*

Un avertissement **unexpectedGeometryInLayer** se produit quand le dessin contient une géométrie incompatible avec le type de géométrie attendu pour un calque donné. Quand le service de conversion retourne un avertissement **unexpectedGeometryInLayer**, il ignore cette géométrie.

#### <a name="example-for-unexpectedgeometryinlayer"></a>*Exemple d’avertissement unexpectedGeometryInLayer*

L’image ci-dessous montre une polyligne non fermée. Supposons que le calque ne prend en charge qu’une géométrie fermée.

![Exemple de polyligne non fermée](./media/drawing-conversion-error-codes/geometry-warning-3.png)

#### <a name="how-to-fix-unexpectedgeometryinlayer"></a>*Comment corriger un avertissement unexpectedGeometryInLayer*

Inspectez chaque avertissement **unexpectedGeometryInLayer** et déplacez la géométrie incompatible vers un calque compatible. Si la géométrie n’est compatible avec aucun autre calque, il faut la supprimer.

### <a name="unsupportedfeaturerepresentation"></a>**Avertissement unsupportedFeatureRepresentation**

#### <a name="description-for-unsupportedfeaturerepresentation"></a>*Description de l’avertissement unsupportedFeatureRepresentation*

L’avertissement **unsupportedFeatureRepresentation** se produit quand le dessin contient un type d’entité non pris en charge.

#### <a name="example-for-unsupportedfeaturerepresentation"></a>*Exemples d’avertissement unsupportedFeatureRepresentation*

L’image ci-dessous montre un type d’entité non pris en charge, tel qu’un objet texte multiligne sur un calque d’étiquette.
  
![Exemple d’objet texte multiligne sur un calque d’étiquette](./media/drawing-conversion-error-codes/multi-line.png)

#### <a name="how-to-fix-unsupportedfeaturerepresentation"></a>*Comment corriger un avertissement unsupportedFeatureRepresentation*

Assurez-vous que vos fichiers DWG ne contiennent que des types d’entités pris en charge. Les types pris en charge sont répertoriés dans la [section Exigences des fichiers de dessin de l’article Exigences du package de dessin](drawing-requirements.md#drawing-package-requirements).

### <a name="automaticrepairperformed"></a>**Avertissement automaticRepairPerformed**

#### <a name="description-for-automaticrepairperformed"></a>*Description de l’avertissement automaticRepairPerformed*

L’avertissement **automaticRepairPerformed** se produit quand le service de conversion répare automatiquement une géométrie non valide.

#### <a name="examples-for-automaticrepairperformed"></a>*Exemples d’avertissement automaticRepairPerformed*

* L’image suivante montre comment le service de conversion a réparé un polygone avec auto-intersection dans une géométrie valide.

    ![Exemple de polygone avec auto-intersection réparé](./media/drawing-conversion-error-codes/automatic-repair-1.png)

* L’image ci-dessous montre comment le service de conversion a aligné les premier et dernier sommets d’une polyligne non fermée pour créer une polyligne fermée, où les premier et dernier sommets étaient séparés de moins de 1 mm.  

    ![Exemple de polyligne alignée](./media/drawing-conversion-error-codes/automatic-repair-2.png)

* L’image ci-dessous montre comment, dans une couche qui prend en charge uniquement des polylignes fermées, le service de conversion a réparé plusieurs polylignes non fermées. Afin d’éviter d’ignorer les polylignes non fermées, le service les combine en une seule polyligne fermée.

    ![Exemple de polylignes non fermées combinées en une seule polyligne fermée](./media/drawing-conversion-error-codes/automatic-repair-3.png)

#### <a name="how-to-fix-automaticrepairperformed"></a>*Comment corriger un avertissement automaticRepairPerformed*

Pour corriger un avertissement **automaticRepairPerformed**, effectuez les actions suivantes :

1. Inspectez la géométrie de chaque avertissement et le texte d’avertissement spécifique.
2. Déterminez si la réparation automatisée est correcte.
3. Si la réparation est correcte, continuez. Sinon, accédez au fichier de conception et résolvez l’avertissement manuellement.

>[!TIP]
>Pour supprimer un avertissement à l’avenir, apportez des modifications au dessin d’origine afin que celui-ci corresponde au dessin réparé.

## <a name="manifest-warnings"></a>Avertissements de manifeste

### <a name="redundantattribution"></a>**Avertissement redundantAttribution**

#### <a name="description-for-redundantattribution"></a>*Description de l’avertissement redundantAttribution*

L’avertissement **redundantAttribution** se produit quand le manifeste contient des propriétés d’objet redondantes ou conflictuelles.

#### <a name="examples-for-redundantattribution"></a>*Exemples d’avertissements redundantAttribution*

* L’extrait de code JSON ci-dessous contient au moins deux objets `unitProperties` avec le même `name`.

    ```json
    "unitProperties": [
        {
            "unitName": "L1-100",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        },
        {
            "unitName": "L1-101",
            "categoryName": "room.office"
        }
    ]
    ```

* Dans l’extrait de code JSON ci-dessous, au moins deux objets `zoneProperties` ont le même `name`.

    ```json
     "zoneProperties": [
        {
            "zoneName": "Assembly Area 1",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        },
        {
            "zoneName": "Assembly Area 2",
            "categoryName": "zone.assembly"
        }
    ```

#### <a name="how-to-fix-redundantattribution"></a>*Comment corriger un avertissement redundantAttribution*

Pour corriger un avertissement **redundantAttribution*, supprimez les propriétés d’objet redondantes ou conflictuelles.

### <a name="manifestwarning"></a>**Avertissement manifestWarning**

#### <a name="description-for-manifestwarning"></a>*Description de l’avertissement manifestWarning*

Un avertissement **manifestWarning** se produit quand le manifeste contient des objets unitProperties ou zoneProperties qui ne sont pas utilisés lors de la conversion.

#### <a name="examples-for-manifestwarning"></a>*Exemples d’avertissements manifestWarning*

* Le manifeste contient un objet `unitProperties` avec un `unitName` qui n’a pas d’étiquette correspondante dans un calque `unitLabel`.

* Le manifeste contient un objet `zoneProperties` avec un `zoneName` qui n’a pas d’étiquette correspondante dans un calque `zoneLabel`.

#### <a name="how-to-fix-manifestwarning"></a>*Comment corriger un avertissement manifestWarning*

Pour corriger un avertissement **manifestWarning**, supprimez l’objet `unitProperties` ou `zoneProperties` inutilisé du manifeste, ou ajoutez une étiquette d’unité/de zone au dessin afin que l’objet propriétés soit utilisé lors de la conversion.

## <a name="wall-warnings"></a>Avertissements de mur

### <a name="walloutsidelevel"></a>**Avertissement wallOutsideLevel**

#### <a name="description-for-walloutsidelevel"></a>*Description de l’avertissement wallOutsideLevel*

L’avertissement **wallOutsideLevel** se produit quand le dessin contient une géométrie de mur en dehors des limites d’un plan de niveau.

#### <a name="example-for-walloutsidelevel"></a>*Exemple d’avertissement wallOutsideLevel*

* L’image ci-dessous montre un mur intérieur, en rouge, en dehors de la limite de niveau jaune.

    ![Exemple de mur intérieur en dehors de la limite de niveau](./media/drawing-conversion-error-codes/wall-outside-level.png)

* L’image suivante montre un mur extérieur, en rouge, en dehors de la limite de niveau jaune.

    ![Exemple de mur extérieur en dehors de la limite de niveau](./media/drawing-conversion-error-codes/wall-outside-level-exterior.png)

#### <a name="how-to-fix-walloutsidelevel"></a>*Comment corriger un avertissement wallOutsideLevel*

Pour corriger un avertissement **wallOutsideLevel**, développez la géométrie du niveau pour inclure tous les murs. Ou modifiez les limites de mur pour qu’elles s’inscrivent à l’intérieur de la limite de niveau.

## <a name="unit-warnings"></a>Avertissements d’unité

### <a name="unitoutsidelevel"></a>**Avertissement unitOutsideLevel**

#### <a name="description-for-unitoutsidelevel"></a>*Description de l’avertissement unitOutsideLevel*

Un avertissement **unitOutsideLevel** se produit quand le dessin contient une géométrie d’unité en dehors des limites du plan de niveau.

#### <a name="example-for-unitoutsidelevel"></a>*Exemple d’avertissement unitOutsideLevel*

 Dans l’image suivante, la géométrie d’unité, en rouge, dépasse les limites de la limite de niveau jaune.

 ![Exemple d’unité dépassant la limite de niveau](./media/drawing-conversion-error-codes/unit-outside-level.png)

#### <a name="how-to-fix-unitoutsidelevel"></a>*Comment corriger un avertissement unitOutsideLevel*

Pour corriger un avertissement **unitOutsideLevel**, développez la limite du niveau pour inclure toutes les unités. Ou modifiez la géométrie d’unité pour qu’elle s’inscrive à l’intérieur de la limite de niveau.

### <a name="partiallyoverlappingunit"></a>**Avertissement partiallyOverlappingUnit**

#### <a name="description-for-partiallyoverlappingunit"></a>*Description de l’avertissement partiallyOverlappingUnit*

Un avertissement **partiallyOverlappingUnit** se produit quand le dessin contient une géométrie d’unité chevauchant partiellement une autre géométrie d’unité. Le service de conversion ignore les unités qui se chevauchent.

#### <a name="example-scenarios-partiallyoverlappingunit"></a>*Exemples de scénarios partiallyOverlappingUnit*

Dans l’image suivante, l’unité qui chevauche est en surbrillance en rouge. `UNIT110` et `HALLWAY` sont ignorés.

![Exemple d’unités qui se chevauchent](./media/drawing-conversion-error-codes/partially-overlapping-unit.png)

#### <a name="how-to-fix-partiallyoverlappingunit"></a>*Comment corriger un avertissement partiallyOverlappingUnit*

Pour corriger un avertissement **partiallyOverlappingUnit**, redessinez chaque unité qui chevauche partiellement afin qu’elle ne chevauche aucune autre unité.

## <a name="door-warnings"></a>Avertissements de porte

### <a name="dooroutsidelevel"></a>**Avertissement doorOutsideLevel**

#### <a name="description-for-dooroutsidelevel"></a>*Description de l’avertissement doorOutsideLevel*

Un avertissement **doorOutsideLevel** se produit quand le dessin contient une géométrie de porte en dehors des limites de la géométrie de niveau.

#### <a name="example-for-dooroutsidelevel"></a>*Exemple d’avertissement doorOutsideLevel*

Dans l’image suivante, la géométrie de porte, en surbrillance en rouge, chevauche la limite de niveau jaune.

![Exemple de porte chevauchant une limite de niveau](./media/drawing-conversion-error-codes/door-outside-level.png)

#### <a name="how-to-fix-dooroutsidelevel"></a>*Comment corriger un avertissement doorOutsideLevel*

Pour corriger un avertissement **doorOutsideLevel**, redessinez la géométrie de porte afin qu’elle se trouve à l’intérieur des limites du niveau.

## <a name="zone-warnings"></a>Avertissements de zone

### <a name="zonewarning"></a>**Avertissement zoneWarning**

#### <a name="description-for-zonewarning"></a>*Description de l’avertissement zoneWarning*

Un avertissement **zoneWarning** se produit quand une zone ne contient pas d’étiquette. Le service de conversion ignore une zone qui n’est pas étiquetée.

#### <a name="example-for-zonewarning"></a>*Exemple d’avertissement zoneWarning*

L’image suivante montre une zone qui ne contient pas d’étiquette.

![Exemple de zone ne contenant pas d’étiquette](./media/drawing-conversion-error-codes/zone-warning.png)

#### <a name="how-to-fix-zonewarning"></a>*Comment corriger un avertissement zoneWarning*

Pour corriger un avertissement **zoneWarning**, vérifiez que chaque zone n’a qu’une seule étiquette.

## <a name="label-warnings"></a>Avertissements d’étiquette

### <a name="labelwarning"></a>*Avertissement labelWarning*

#### <a name="description-for-labelwarning"></a>*Description de l’avertissement labelWarning*

L’avertissement **labelWarning** se produit quand le dessin contient une caractéristique d’étiquettes ambiguë ou contradictoire.

Un avertissement **labelWarning** se produit pour une ou plusieurs des raisons suivantes :

* Une étiquette d’unité n’est dans aucune unité.
* Une étiquette de zone n’est dans aucune zone.
* Une étiquette de zone se trouve dans au moins deux zones.

#### <a name="example-for-labelwarning"></a>*Exemple d’avertissement labelWarning*

L’image suivante montre une étiquette se trouvant dans deux zones.

![Exemple d’étiquette se trouvant dans deux zones ](./media/drawing-conversion-error-codes/label-warning.png)

#### <a name="how-to-fix-labelwarning"></a>*Comment corriger un avertissement labelWarning*

Pour corriger un avertissement **labelWarning**, vérifiez les points suivants :

* Toutes les étiquettes d’unité se trouvent dans des unités.
* Toutes les étiquettes de zone se trouvent dans des zones.
* Toutes les étiquettes de zone se trouvent dans une seule zone.

## <a name="drawing-package-errors"></a>Erreurs du package de dessin

### <a name="invalidarchiveformat"></a>**Erreur invalidArchiveFormat**

#### <a name="description-for-invalidarchiveformat"></a>*Description de l’erreur invalidArchiveFormat*

Une erreur **invalidArchiveFormat** se produit quand le package de dessin est dans un format d’archive non valide tel que GZIP ou 7-zip. Seul le format d’archive ZIP est pris en charge.

Une erreur **invalidArchiveFormat** se produit également si l’archive ZIP est vide.

#### <a name="how-to-fix-invalidarchiveformat"></a>*Comment corriger une erreur invalidArchiveFormat*

Pour corriger une erreur **invalidArchiveFormat**, vérifiez les points suivants :

* Le nom de votre fichier d’archive se termine par _.zip_.
* Votre archive ZIP contient des données.
* Vous pouvez ouvrir votre archive ZIP.

### <a name="invaliduserdata"></a>**Erreur invalidUserData**

#### <a name="description-for-invaliduserdata"></a>*Description de l’erreur invalidUserData*

Une erreur **invalidUserData** se produit lorsque le service de conversion ne peut pas lire un objet données utilisateur à partir du stockage.

#### <a name="example-scenario-for-invaliduserdata"></a>*Exemple de scénario pour invalidUserData*

Vous avez tenté de charger un package de dessin avec un paramètre `udid` incorrect.

#### <a name="how-to-fix-invaliduserdata"></a>*Comment corriger une erreur invalidUserData*

Pour corriger une erreur **invalidUserData**, vérifiez les points suivants :

* Vous avez fourni un `udid` correct pour le package chargé.
* Azure Maps Creator (préversion) a été activé pour le compte Azure Maps que vous avez utilisé dans le but de charger le package de dessin.
* La demande d’API adressée au service de conversion contient la clé d’abonnement au compte Azure Maps que vous avez utilisé pour charger le package de dessin.

### <a name="dwgerror"></a>**Erreur dwgError**

#### <a name="description-for-dwgerror"></a>*Description de l’erreur dwgError*

Une erreur **dwgError** se produit quand le package de dessin contient un problème avec un ou plusieurs fichiers DWG dans l’archive ZIP chargée.

L’erreur **dwgError** se quand le package de dessin contient un fichier DWG qui ne peut pas être ouvert parce qu’il est non valide ou endommagé.

* Un fichier DWG n’est pas un dessin au format de fichier DWG AutoCAD valide.
* Un fichier DWG est endommagé.
* Un fichier DWG répertorié dans le fichier _manifest.json_ est manquant dans l’archive ZIP.

#### <a name="how-to-fix-dwgerror"></a>*Comment corriger une erreur dwgError*

Pour corriger une erreur **dwgError**, inspectez votre fichier _manifest.json_ pour vérifier les points suivants :

* Tous les fichiers DWG dans votre archive ZIP sont des dessins au format DWG AutoCAD valides. Ouvrez-les dans AutoCAD. Supprimez ou corrigez tous les dessins non valides.
* La liste des fichiers DWG dans le fichier _manifest.json_ correspond aux fichiers DWG dans l’archive ZIP.

## <a name="manifest-errors"></a>Erreurs de manifeste

### <a name="invalidjsonformat"></a>**Erreur invalidJsonFormat**

#### <a name="description-for-invalidjsonformat"></a>Description de l’erreur invalidJsonFormat

Une erreur **invalidJsonFormat** se produit lorsque le fichier _manifest.json_ n’est pas lisible.

Le fichier _manifest.json_ n’est pas lisible en raison d’erreurs de syntaxe ou de format JSON. Pour en savoir plus sur le format et la syntaxe JSON, consultez [Format d’échange de données JSON (JavaScript Object Notation)](https://tools.ietf.org/html/rfc7159)

#### <a name="how-to-fix-invalidjsonformat"></a>*Comment corriger une erreur invalidJsonFormat*

Pour corriger une erreur **invalidJsonFormat**, utilisez un linter JSON pour détecter et résoudre toute erreur JSON.

### <a name="missingrequiredfield"></a>**Erreur missingRequiredField**

#### <a name="description-for-missingrequiredfield"></a>*Description de l’erreur missingRequiredField*

Une erreur **missingRequiredField** se produit quand des données obligatoires sont absentes du fichier _manifest.json_.

#### <a name="how-to-fix-missingrequiredfield"></a>*Comment corriger une erreur missingRequiredField*

Pour corriger une erreur **missingRequiredField**, vérifiez que le manifeste contient toutes les propriétés requises. Pour obtenir la liste complète des objets de manifeste requis, consultez la [section de manifeste dans les exigences du package de dessin](drawing-requirements.md#manifest-file-requirements)  

### <a name="missingmanifest"></a>**Erreur missingManifest**

#### <a name="description-for-missingmanifest"></a>*Description de l’erreur missingManifest*

L’erreur **missingManifest** se produit quand le fichier _manifest.json_ est manquant dans l’archive ZIP.

L’erreur **missingManifest** se produit pour une ou plusieurs des raisons suivantes :

* Le fichier _manifest.json_ est mal orthographié.
* Le fichier _manifest.json_ est manquant.
* Le fichier _manifest.json_ ne figure pas dans le répertoire racine de l’archive ZIP.

#### <a name="how-to-fix-missingmanifest"></a>*Comment corriger l’erreur missingManifest*

Pour corriger une erreur **missingManifest**, vérifiez que l’archive ZIP contient un fichier nommé _manifest.json_ au niveau racine.

### <a name="conflict"></a>**Erreur conflict**

#### <a name="description-for-conflict"></a>*Description de l’erreur conflict*

L’erreur **conflict** se produit quand le fichier _manifest.json_ contient des informations conflictuelles.

#### <a name="example-scenario-for-conflict"></a>*Example de scénario pour l’erreur conflict*

Le service de conversion retourne une erreur **conflict** quand plusieurs niveaux sont définis avec la même valeur ordinale de niveau. L’extrait de code JSON suivant montre deux niveaux définis avec la même valeur ordinale.

```JSON
"buildingLevels":
{
    "levels": [
        {
            "levelName": "Ground",
            "ordinal": 0,
            "filename": "./Level_0.dwg"
        },
        {
            "levelName": "Parking",
            "ordinal": 0,
            "filename": "./Level_P.dwg"
        }
    ]
}
```

#### <a name="how-to-fix-conflict"></a>*Comment corriger une erreur conflict*

Pour corriger une erreur **conflict**, inspectez votre fichier _manifest.json_ pour en supprimer toutes les informations conflictuelles.

### <a name="invalidgeoreference"></a>**Erreur invalidGeoreference**

#### <a name="description-for-invalidgeoreference"></a>*Description de l’erreur invalidGeoreference*

L’erreur **invalidGeoreference** se produit quand un fichier _manifest.json_ contient une référence géographique non valide.

L’erreur **invalidGeoreference** se produit pour une ou plusieurs des raisons suivantes :

* L’utilisateur fait une géoréférence à une valeur de latitude ou de longitude hors limites.
* L’utilisateur fait référence à une valeur de rotation hors limites.

#### <a name="example-scenario-for-invalidgeoreference"></a>*Exemple de scénario d’erreur invalidGeoreference*

Dans l’extrait de code JSON ci-dessous, la latitude est supérieure à la limite supérieure.

```json
"georeference"
{
    "lat": 88.0,
    "lon": -122.132600,
    "angle": 0
},
```

#### <a name="how-to-fix-invalidgeoreference"></a>*Comment corriger une erreur invalidGeoreference*

Pour corriger une erreur **invalidGeoreference**, vérifiez que les valeurs géoréférencées s’inscrivent dans la plage.

>[!IMPORTANT]
>Dans géoJSON, l’ordre des coordonnées est longitude et latitude. Si vous n’utilisez pas le bon ordre, vous pouvez faire référence accidentellement à une valeur de latitude ou de longitude hors limites.

## <a name="wall-errors"></a>Erreurs de mur

### <a name="wallerror"></a>**Erreur wallError**

#### <a name="description-for-wallerror"></a>*Description de l’erreur wallError*

L’erreur **wallError** se produit quand le dessin contient une erreur lors d’une tentative de création de caractéristique de mur.

#### <a name="example-scenario-for-wallerror"></a>*Exemple de scénario d’erreur wallError*

L’image suivante montre une caractéristique de mur qui ne chevauche aucune unité.

![Exemple de caractéristique de mur qui ne chevauche aucune unité](./media/drawing-conversion-error-codes/wall-error.png)

#### <a name="how-to-fix-wallerror"></a>*Comment corriger une erreur wallError*

Pour corriger une erreur **wallError**, redessinez le mur afin qu’il chevauche au moins une unité. Vous pouvez aussi créer une unité qui chevauche le mur.

## <a name="vertical-penetration-errors"></a>Erreurs de pénétration verticale

### <a name="verticalpenetrationerror"></a>**Erreurs verticalPenetrationError**

#### <a name="description-for-verticalpenetrationerror"></a>*Description de l’erreur verticalPenetrationError*

L’erreur **verticalPenetrationError** se produit quand le dessin contient une caractéristique de pénétration verticale ambiguë.

L’erreur **verticalPenetrationError** se produit pour une ou plusieurs des raisons suivantes :

* Le dessin contient une zone de pénétration verticale sans zone de pénétration verticale qui la chevauche au-dessus ou en dessous.
* Le package de dessin contient un niveau avec au moins deux caractéristiques de pénétration verticale sur celui-ci, qui chevauchent toutes deux une caractéristique de pénétration verticale sur un autre niveau situé directement au-dessus ou en dessous.

#### <a name="example-scenario-for-verticalpenetrationerror"></a>*Exemple de scénario d’erreur verticalPenetrationError*

L’image ci-dessous montre une zone de pénétration verticale sans zone de pénétration verticale qui la chevauche aux niveaux situés au-dessus ou en dessous.

![Exemple de pénétration verticale 1](./media/drawing-conversion-error-codes/vrt-2.png)

L’illustration suivante montre une zone de pénétration verticale qui chevauche plus d’une zone de pénétration verticale sur un niveau adjacent.

![Exemple de pénétration verticale 2](./media/drawing-conversion-error-codes/vrt-1.png)

#### <a name="how-to-fix-verticalpenetrationerror"></a>Comment corriger une erreur verticalPenetrationError

Pour corriger une erreur **verticalPenetrationError**, lisez l’article [Exigences du package de dessin](drawing-requirements.md) pour savoir comment utiliser une caractéristique de pénétration verticale.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Comment utiliser le visualiseur d’erreurs de dessin Azure Maps](drawing-error-visualizer.md)

> [!div class="nextstepaction"]
> [Creator (préversion) pour cartes d’intérieur](creator-indoor-maps.md)
---
title: Concepts de reconnaissance faciale
titleSuffix: Azure Cognitive Services
description: Cet article explique les concepts des opérations de reconnaissance faciale Vérifier, Rechercher semblables, Grouper et Identifier, ainsi que des structures de données sous-jacentes.
services: cognitive-services
author: PatrickFarley
manager: nitime
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/23/2019
ms.author: pafarley
ms.openlocfilehash: 00dadf8a91b7ed01ab9f91933d296744305a95af
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92518804"
---
# <a name="face-recognition-concepts"></a>Concepts de reconnaissance faciale

Cet article explique les concepts des opérations de reconnaissance faciale Vérifier, Rechercher semblables, Grouper et Identifier, ainsi que des structures de données sous-jacentes. Globalement, la reconnaissance décrit le travail de comparaison de deux visages différents pour déterminer s’ils sont similaires ou s’ils appartiennent à la même personne.

## <a name="recognition-related-data-structures"></a>Structures de données liées à la reconnaissance

Les opérations de reconnaissance utilisent principalement les structures de données suivantes. Ces objets sont stockés dans le cloud et peuvent être référencés par leurs chaînes d’ID. Les chaînes d’ID sont toujours uniques dans un abonnement. Les champs de noms peuvent être dupliqués.

|Name|Description|
|:--|:--|
|DetectedFace| Cette représentation de visage unique est récupérée par l’opération de [détection des visages](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md). Son ID expire 24 heures après sa création.|
|PersistedFace| Lorsque des objets DetectedFace sont ajoutés à un groupe, tel que FaceList ou Person, ils deviennent des objets PersistedFace. Ils peuvent être [récupérés](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524c) à n’importe quel moment et n’expirent pas.|
|[FaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b) ou [LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc)| Cette structure de données est une liste assortie d’objets PersistedFace. Un objet FaceList a un ID unique, une chaîne de nom et éventuellement une chaîne de données utilisateur.|
|[Person](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523c)| Cette structure de données est une liste d’objets PersistedFace qui appartiennent à la même personne. Elle possède un ID unique, une chaîne de nom et éventuellement une chaîne de données utilisateur.|
|[PersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) ou [LargePersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d)| Cette structure de données est une liste assortie d’objets Person. Elle possède un ID unique, une chaîne de nom et éventuellement une chaîne de données utilisateur. Un PersonGroup doit être [entraîné](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395249) avant de pouvoir être utilisé dans les opérations de reconnaissance.|

## <a name="recognition-operations"></a>Opérations de reconnaissance

Cette section décrit en détail comment les quatre opérations de reconnaissance utilisent les structures de données précédemment décrites. Pour obtenir une description générale de chaque opération de reconnaissance, consultez [Vue d’ensemble](../Overview.md).

### <a name="verify"></a>Vérifier

L’opération [Vérifier](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a) accepte un ID de visage issu de DetectedFace ou de PersistedFace et un autre ID de visage ou un objet Person, et détermine s’ils appartiennent à la même personne. Si vous transmettez un objet Person, vous pouvez éventuellement transmettre un PersonGroup auquel cet objet Person appartient pour améliorer les performances.

### <a name="find-similar"></a>Rechercher semblables

L’opération [Rechercher semblables](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) accepte un ID de visage issu de DetectedFace ou de PersistedFace et un FaceList ou un tableau d’autres ID de visages. Avec un FaceList, elle retourne un plus petit FaceList de visages semblables au visage donné. Avec un tableau d’ID de visages, elle retourne de même un tableau plus petit.

### <a name="group"></a>Groupe

L’opération [Grouper](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238) accepte un tableau d’ID de visages assortis issus de DetectedFace ou de PersistedFace, et retourne les mêmes ID regroupés en plusieurs tableaux plus petits. Chaque tableau de « groupes » contient les ID de visages qui apparaissent semblables. Un tableau unique « messyGroup » contient des ID de visages pour lesquels aucune ressemblance n’a été trouvée.

### <a name="identify"></a>Identifier

L’opération [Identity](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) prend un ou plusieurs ID de visage à partir de DetectedFace ou de PersistedFace ainsi qu’un groupe de personnes et retourne une liste d’objets Person auxquels chaque face peut appartenir. Les objets Person retournés sont empaquetés en tant qu’objets de candidats et comportent une valeur de niveau de confiance de prédiction.

## <a name="input-data"></a>Données d’entrée

Utilisez les conseils suivants pour vous assurer que vos images d’entrée fournissent les résultats de reconnaissance les plus précis :

* Les formats d’image d’entrée pris en charge sont JPEG, PNG, GIF (la première image) et BMP.
* La taille du fichier image ne doit pas dépasser 6 Mo.
* Lorsque vous créez des objets Person, utilisez des photos présentant différents types d’éclairage et d’angle.
* Certains visages peuvent ne pas être reconnus en raison de défis techniques, tels que :
  * Des images avec un éclairage extrême, par exemple, un contre-jour marqué.
  * Des obstacles cachant un œil ou les deux yeux.
  * Des différences de type de cheveux ou de pilosité faciale.
  * Des changements d’apparence du visage dus à l’âge.
  * Une très grande expressivité du visage.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec les concepts de reconnaissance faciale, écrivez un script pour identifier les visages par rapport à un objet PersonGroup formé.

* [Démarrage rapide : Bibliothèque de client Visage](../Quickstarts/client-libraries.md)
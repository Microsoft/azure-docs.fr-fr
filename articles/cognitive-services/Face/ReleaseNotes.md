---
title: Nouveautés du service Visage
titleSuffix: Azure Cognitive Services
description: Les notes de publication du service Visage comportent un historique des modifications apportées aux différentes versions.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 12/11/2020
ms.author: pafarley
ms.openlocfilehash: 8a409d66065a7ccd2b34365bd565202f207ac0e8
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97560744"
---
# <a name="whats-new-in-face-service"></a>Nouveautés du service Visage

Le service Visage Azure est mis à jour régulièrement. Lisez cet article pour vous tenir informé des améliorations de fonctionnalités, des correctifs et des mises à jour de la documentation.

## <a name="november-2020"></a>Novembre 2020
* Publication d’un exemple d’application d’inscription à Visage pour montrer les meilleures pratiques pour l’établissement d’un consentement explicite et créer des systèmes de reconnaissance faciale de haute précision via des inscriptions de haute qualité. L’exemple open source se trouve dans le guide [Créer une application d’inscription](build-enrollment-app.md) et sur [GitHub](https://github.com/Azure-Samples/cognitive-services-FaceAPIEnrollmentSample), prêt à être déployé ou personnalisé par des développeurs.  

## <a name="june-2019"></a>Juin 2019

* Ajout d’un nouveau modèle de détection des visages avec une meilleure précision sur les visages de petite taille, de profil, masqués et flous. Utilisez-le par le biais de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) et [LargePersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42) en spécifiant le nom du nouveau modèle de détection des visages `detection_02` dans le paramètre `detectionModel`. Vous trouverez des détails supplémentaires dans [Guide pratique pour spécifier un modèle de détection](Face-API-How-to-Topics/specify-detection-model.md).

## <a name="april-2019"></a>Avril 2019

* Amélioration de la précision globale des attributs `age` et `headPose`. L’attribut `headPose` a également été mis à jour ; la valeur `pitch` est désormais activée. Utilisez ces attributs en les spécifiant dans le paramètre `returnFaceAttributes` de [Face - Detect paramètre ](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) `returnFaceAttributes`. 

* Amélioration de la rapidité de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) et [LargePersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42).

## <a name="march-2019"></a>Mars 2019

* Ajout d’un nouveau modèle de reconnaissance faciale avec une meilleure précision. Utilisez-le par le biais de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039524b), [LargeFaceList - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc), [PersonGroup - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395244) et [LargePersonGroup - Create](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d) en spécifiant le nom du nouveau modèle de reconnaissance faciale `recognition_02` dans le paramètre `recognitionModel`. Vous trouverez des détails supplémentaires dans [Guide pratique pour spécifier un modèle de reconnaissance](Face-API-How-to-Topics/specify-recognition-model.md).

## <a name="january-2019"></a>Janvier 2019

* Ajout de la fonctionnalité de capture instantanée pour prendre en charge la migration des données entre des abonnements : [Capture instantanée](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/snapshot-get). Vous trouverez des détails supplémentaires dans [Migrer vos données de visage vers un autre abonnement API Visage](Face-API-How-to-Topics/how-to-migrate-face-data.md).

## <a name="october-2018"></a>Octobre 2018

* Description détaillée de `status`, `createdDateTime`, `lastActionDateTime` et `lastSuccessfulTrainingDateTime` dans [PersonGroup - Obtenir l’état de l’entrainement](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395247), [LargePersonGroup - Obtenir l’état de l’entrainement](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae32c6ac60f11b48b5aa5) et [ LargeFaceList - Obtenir l’état de l’entrainement](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a1582f8d2de3616c086f2cf).

## <a name="may-2018"></a>Mai 2018

* Nous avons considérablement amélioré l’attribut `gender` ainsi que nous avons amélioré les attributs `age`, `glasses`, `facialHair`, `hair` et `makeup`. Utilisez-les via le paramètre `returnFaceAttributes` de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236). 

* Nous avons augmenté la taille maximale des fichiers d’image d’entrée la faisant passer de 4 à 6 Mo dans [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250), [LargeFaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a158c10d2de3616c086f2d3), [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) et [LargePersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599adf2a3a7b9412a4d53f42).

## <a name="march-2018"></a>Mars 2018

* Nous avons ajouté un conteneur à l’échelle un million : [LargeFaceList](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/5a157b68d2de3616c086f2cc) et [LargePersonGroup](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599acdee6ac60f11b48b5a9d). Vous trouverez davantage d’informations dans la section [Comment utiliser la fonctionnalité à grande échelle](Face-API-How-to-Topics/how-to-use-large-scale.md).

* Nous avons fait passer la plage de valeurs du paramètre `maxNumOfCandidatesReturned` de [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) de [1, 5] à [1, 100], sa valeur par défaut étant 10.

## <a name="may-2017"></a>Mai 2017

* Nous avons ajouté les attributs `hair`, `makeup`, `accessory`, `occlusion`, `blur`, `exposure` et `noise` dans le paramètre `returnFaceAttributes` de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

* Les paramètres PersonGroup et [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239) acceptent désormais 10 000 personnes.

* Nous prenons en charge la pagination dans [PersonGroup Person - List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395241) avec comme paramètres facultatifs `start` et `top`.

* Nous prenons en charge l’accès concurrentiel pour ajouter et supprimer des visages concernant les listes FaceLists et les personnes dans PersonGroup.

## <a name="march-2017"></a>Mars 2017
* Nous avons ajouté l’attribut `emotion` dans le paramètre `returnFaceAttributes` de [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

* Nous avons résolu le problème qui entraînait une absence de détection et le renvoi d’un rectangle depuis [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) en tant que `targetFace` dans [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) et [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b).

* Nous avons résolu le problème affectant la taille du visage détectable pour garantir qu’elle se trouvait bien entre 36 x 36 et 4 096 x 4096 pixels.

## <a name="november-2016"></a>Novembre 2016
* Nous avons ajouté un abonnement Stockage Visage standard qui permet de stocker des visages persistants supplémentaires lorsque vous utilisez [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b) ou [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) pour l’identification ou la mise en correspondance des similitudes. Les images stockées sont facturées au tarif de 0,5 $ tous les 1 000 visages (tarif journalier au prorata). Les abonnements de niveau gratuit sont toujours limités à un total de 1 000 personnes.

## <a name="october-2016"></a>Octobre 2016
* Nous avons modifié le message d’erreur lorsque plusieurs visages apparaissent dans targetFace. Auparavant, il indiquait « There are more than one face in the image ». Maintenant, il indique « There is more than one face in the image » dans [FaceList - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395250) et [PersonGroup Person - Add Face](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523b).

## <a name="july-2016"></a>Juillet 2016
* Nous prenons en charge l’authentification d’objet Face to Person dans [Face - Verify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a).

* Nous avons ajouté un paramètre `mode` facultatif qui permet de sélectionner deux modes de fonctionnement : `matchPerson` et `matchFace` dans [Face - Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237), le paramètre par défaut étant `matchPerson`.

* Nous avons ajouté un paramètre `confidenceThreshold` facultatif qui permet à l’utilisateur de définir le seuil indiquant si un visage appartient à l’objet Person dans [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).

* Nous avons ajouté les paramètres `start` et `top` facultatifs dans [PersonGroup - List](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395248) pour permettre à l’utilisateur d’indiquer le point de départ et le nombre total de PersonGroups dans la liste.

## <a name="v10-changes-from-v0"></a>Changements apportés entre la version 0 et la version 1.0

* Nous avons mis à jour le point de terminaison de la racine du service le faisant passer de ```https://westus.api.cognitive.microsoft.com/face/v0/``` à ```https://westus.api.cognitive.microsoft.com/face/v1.0/```. Nous avons apporté des modifications à : [Face - Detect](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236), [Face - Identify](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239), [Face - Find Similar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) et [Face - Group](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).

* Nous avons mis à jour la taille minimale pour la détection des visages la faisant passer à 36 x 36 pixels. Les visages dont la taille est inférieure à 36 x 36 pixels ne seront pas détectés.

* Nous avons abandonné les données PersonGroup et Person qui étaient disponibles dans la version 0 de Visage. Ces données ne sont pas accessibles dans la version 1.0 de Visage.

* Nous avons abandonné le point de terminaison de la version 0 de l’API Visage au 30 juin 2016.

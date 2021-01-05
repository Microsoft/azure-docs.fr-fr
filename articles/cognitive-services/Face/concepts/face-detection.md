---
title: Concepts de détection et d’attributs de visage
titleSuffix: Azure Cognitive Services
description: La détection des visages est l’action visant à localiser des visages humains dans une image et à retourner, si vous le souhaitez, différents types de données liées aux visages.
services: cognitive-services
author: PatrickFarley
manager: nitime
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 04/26/2019
ms.author: pafarley
ms.openlocfilehash: 92b19941f34b9bf5656c9beb188a68d2cf01f674
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92504127"
---
# <a name="face-detection-and-attributes"></a>Détection et attributs de visage

Cet article explique les concepts de détection des visages et des données d’attribut de visage. La détection des visages est l’action visant à localiser des visages humains dans une image et à retourner, si vous le souhaitez, différents types de données liées aux visages.

Vous utilisez l’opération [Visage - Détecter](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236) pour détecter les visages dans une image. Au minimum, chaque visage détecté correspond à un champ faceRectangle dans la réponse. Cet ensemble de coordonnées de pixels left (gauche), top (haut), width (largeur) et height (hauteur) marque le visage localisé. Ces coordonnées vous permettent d’obtenir l’emplacement du visage et sa taille. Dans la réponse de l’API, les visages sont répertoriés par ordre de taille, du plus grand au plus petit.

## <a name="face-id"></a>ID du visage

L’ID de visage est une chaîne d’identification unique pour chaque visage détecté dans une image. Vous pouvez demander un ID de visage dans votre appel d’API [Visage - Détecter](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

## <a name="face-landmarks"></a>Points de repère du visage

Les points de repère de visage sont un ensemble de points faciles à trouver sur un visage, tels que les pupilles ou la pointe du nez. Il existe 27 points de repères prédéfinis par défaut. La figure suivante montre l’ensemble des 27 points :

![Schéma de visage avec les 27 points de repère étiquetés](../Images/landmarks.1.jpg)

Les coordonnées des points sont retournées en unités de pixels.

## <a name="attributes"></a>Attributs

Les attributs sont un ensemble de fonctionnalités qui peuvent éventuellement être détectées par l’API [Visage - Détecter](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236). Les attributs suivants peuvent être détectés :

* **age** . L’âge estimé en années d’un visage particulier.
* **blur** . L’aspect flouté du visage dans l’image. Cet attribut retourne une valeur comprise entre 0 et 1, ainsi qu’une évaluation informelle low (faible), medium (moyenne) ou high (élevée).
* **emotion** . Une liste d’émotions avec leur confiance de détection pour le visage donné. Les scores de confiance sont normalisés et la somme des scores de toutes les émotions est égale à 1. Les émotions détectées sont le bonheur, la tristesse, la neutralité, la colère, le mépris, le dégoût, la surprise et la peur.
* **exposure** . L’exposition du visage dans l’image. Cet attribut retourne une valeur comprise entre 0 et 1, ainsi qu’une évaluation informelle underExposure (sous-exposition), goodExposure (bonne exposition) ou overExposure (surexposition).
* **facialHair** . La présence estimée de pilosité faciale et sa longueur pour le visage donné.
* **gender** . Le sexe estimé de la personne détentrice du visage donné. Les valeurs possibles sont male (homme), female (femme) et genderless (sexe indéterminé).
* **glasses** . Indique si le visage donné porte des lunettes. Les valeurs possibles sont NoGlasses (pas de lunettes), ReadingGlasses (lunettes de lecture), Sunglasses (lunettes de soleil) et Swimming Goggles (lunettes de natation).
* **hair** . Type de cheveux du visage. Cet attribut indique si les cheveux sont visibles, si une calvitie est détectée et les couleurs de cheveux détectées.
* **headPose** . Orientation du visage dans l’espace 3D. Cet attribut est défini par des angles de roulis, de tangage et de lacet, mesurés en degrés Les plages de valeurs vont de -90 degrés à 90 degrés, - 90 degrés à 90 degrés, et -90 degrés à 90 degrés, respectivement. Consultez le diagramme suivant pour les mappages des angles :

    ![Tête avec les axes d’inclinaison longitudinale (pitch), d’inclinaison latérale (roll) et de lacet (yaw) étiquetés](../Images/headpose.1.jpg)
* **makeup** . Indique si le visage comporte du maquillage. Cet attribut retourne une valeur booléenne pour eyeMakeup (maquillage des yeux) et lipMakeup (maquillage des lèvres).
* **noise** . Le bruit visuel détecté dans l’image du visage. Cet attribut retourne une valeur comprise entre 0 et 1, ainsi qu’une évaluation informelle low (faible), medium (moyenne) ou high (élevée).
* **occlusion** . Indique si des objets obstruent des parties du visage. Cet attribut retourne une valeur booléenne pour eyeOccluded (obstruction des yeux), foreheadOccluded (obstruction du front) et mouthOccluded (obstruction de la bouche).
* **smile** . L’expression de sourire du visage donné. Cette valeur est comprise entre 0 pour aucun sourire et 1 pour un sourire clair.

> [!IMPORTANT]
> Les attributs de visage sont prévus via l’utilisation d’algorithmes statistiques. Ils peuvent ne pas toujours être exacts. Soyez prudent lorsque vous prenez des décisions basées sur les données d’attribut.

## <a name="input-data"></a>Données d’entrée

Utilisez les conseils suivants pour vous assurer que vos images d’entrée fournissent les résultats de détection les plus précis :

* Les formats d’image d’entrée pris en charge incluent JPEG, PNG, GIF pour la première image et BMP.
* La taille du fichier image ne doit pas dépasser 6 Mo.
* La plage des tailles de visages détectables s’étend de 36 x 36 à 4096 x 4096 pixels. Les visages en dehors de cette plage ne sont pas détectés.
* Certains visages peuvent ne pas être détectés en raison de défis techniques. Un angle extrême du visage (posture de la tête) ou l’obstruction du visage (des objets tels que des lunettes de soleil ou les mains obstruent une partie du visage) peuvent affecter la détection. Les visages frontaux et quasi-frontaux fournissent les meilleurs résultats.

Si vous détectez les visages à partir d’un flux vidéo, vous pouvez éventuellement améliorer les performances en ajustant certains paramètres de votre caméra vidéo :

* **Lissage**  : De nombreuses caméras vidéo appliquent un effet de lissage. Vous devez désactiver cette option si vous le pouvez, car elle crée un effet de flou entre les images et réduit la clarté.
* **Vitesse d’obturation**  : Une plus grande vitesse d’obturation réduit l’ampleur des mouvements entre les images et rend chaque image plus claire. Nous vous recommandons une vitesse d’obturation de 1/60 seconde ou plus.
* **Angle d’obturation**  : Certaines caméras spécifient un angle d’obturation à la place d’une vitesse d’obturation. Vous devez utiliser un angle d’obturation inférieur si possible. Cela génère des images vidéo plus claires.

    >[!NOTE]
    > Une caméra avec un angle d’obturation inférieur recevra moins de lumière pour chaque image et l’image sera plus sombre. Vous devrez déterminer le niveau approprié à utiliser.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous êtes familiarisé avec les concepts de détection des visages, découvrez comment écrire un script pour détecter les visages dans une image donnée.

* [Détecter des visages sur une image](../Face-API-How-to-Topics/HowtoDetectFacesinImage.md)
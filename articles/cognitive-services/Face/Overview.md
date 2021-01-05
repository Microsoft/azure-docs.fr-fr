---
title: Présentation du service Visage Azure
titleSuffix: Azure Cognitive Services
description: Le service Visage Azure fournit les algorithmes d’intelligence artificielle que vous utilisez pour détecter, reconnaître et analyser les visages humains dans des images.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: overview
ms.date: 11/23/2020
ms.author: pafarley
ms.custom: cog-serv-seo-aug-2020
keywords: reconnaissance faciale, logiciel de reconnaissance faciale, analyse faciale, correspondance de visage, application de reconnaissance faciale, recherche de visage par image, recherche de reconnaissance faciale
ms.openlocfilehash: 600ca48cc19ee8723b423e484ec96736a55ae7fc
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95532254"
---
# <a name="what-is-the-azure-face-service"></a>Présentation du service Visage Azure

> [!WARNING]
> Le 11 juin 2020, Microsoft a annoncé qu’il ne vendra pas de technologie de reconnaissance faciale aux services de police des États-Unis tant qu’un règlement strict fondé sur les droits de l’homme n’aura pas été promulgué. Par conséquent, les clients ne doivent pas utiliser les fonctionnalités de reconnaissance faciale ni les fonctionnalités comprises dans les services Azure comme Visage ou Video Indexer, s’ils représentent, ou autorisent l’utilisation de ces services par ou pour, un service de police des États-Unis.

[!INCLUDE [TLS 1.2 enforcement](../../../includes/cognitive-services-tls-announcement.md)]

Le service Visage Azure fournit les algorithmes d’intelligence artificielle qui détectent, reconnaissent et analysent les visages humains dans des images. Les logiciels de reconnaissance faciale sont importants dans de nombreux domaines, notamment la sécurité, l’interface utilisateur naturelle, l’analyse et la gestion du contenu de l’image, les applications mobiles et la robotique.

Le service Visage fournit plusieurs fonctions d’analyse faciale différentes, chacune décrite dans les sections suivantes.

## <a name="face-detection"></a>Détection des visages

Le service Visage détecte les visages humains dans une image et retourne les coordonnées du rectangle de leur emplacement. La détection des visages peut également permettre d’extraire un ensemble d’attributs associés au visage tels que la posture de la tête, le sexe, l’âge, l’émotion, la pilosité du visage et les lunettes.

> [!NOTE]
> La fonctionnalité de détection des visages est également disponible par le biais du [service Vision par ordinateur](../computer-vision/overview.md). En revanche, si vous voulez effectuer d’autres opérations avec des données faciales, préférez plutôt utiliser ce service.

![Une image d’une femme et d’un homme, avec des rectangles dessinés autour de leur visage, et un âge et un sexe affichés](./Images/Face.detection.jpg)

Pour plus d’informations sur la détection de visage, consultez l’article sur les concepts de la [détection des visages](concepts/face-detection.md). Consultez également la documentation de référence de l’[API de détection](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395236).

## <a name="face-verification"></a>Vérification faciale

L’API de vérification effectue une authentification en comparant deux visages détectés ou un visage détecté et un objet Personne. En pratique, il évalue si deux visages appartiennent à la même personne. Cette capacité peut être utile dans le domaine de la sécurité. Pour plus d’informations, consultez le guide relatif aux concepts de la [reconnaissance faciale](concepts/face-recognition.md) ou la documentation de référence de l’[API de vérification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a).

## <a name="find-similar-faces"></a>Rechercher des visages semblables

L’API de recherche de visages semblables permet de mettre en correspondance un visage cible avec un ensemble de visages candidats, en cherchant à obtenir un plus petit ensemble de visages ressemblant au visage cible. Cette API s’avère utile pour effectuer une recherche de visage par image. 

Deux modes de travail, **matchPerson** et **matchFace**, sont pris en charge. Le mode **matchPerson** renvoie des visages semblables après avoir filtré les résultats pour la même personne à l’aide de l’[API de vérification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f3039523a). Le mode **matchFace** ignore le filtre de correspondance de personne. Il renvoie la liste des visages candidats semblables susceptibles ou non d’appartenir à la même personne.

L’exemple suivant montre le visage cible :

![Une femme qui sourit](./Images/FaceFindSimilar.QueryFace.jpg)

Et ces images sont les visages candidats :

![Cinq images de personnes qui sourient. Les images a et b correspondent à la même personne.](./Images/FaceFindSimilar.Candidates.jpg)

Pour rechercher quatre visages semblabes, le mode **matchPerson** renvoie a et b, qui représentent le même visage que la cible. Le mode **matchFace** retourne a, b, c et d, soit exactement quatre candidats, même si certains ne sont pas la même personne que la cible ou s’ils présentent une faible similarité. Pour plus d’informations, consultez le guide relatif aux concepts de la [reconnaissance faciale](concepts/face-recognition.md) ou la documentation de référence de l’[API de recherche de visages semblables](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237).

## <a name="face-grouping"></a>Regroupement de visages

L’API de regroupement divise un ensemble de visages inconnus en plusieurs groupes en fonction de leurs similarités. Chaque groupe est un sous-ensemble distinct de l’ensemble de visages d’origine. Tous les visages d’un groupe sont susceptibles d’appartenir à la même personne. Une seule personne peut correspondre à plusieurs groupes différents. Les groupes sont différenciés par un autre facteur, comme l’expression. Pour plus d’informations, consultez le guide relatif aux concepts de la [reconnaissance faciale](concepts/face-recognition.md) ou la documentation de référence de l’[API de groupe](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395238).

## <a name="person-identification"></a>Identification de personnes

L’API d’identification permet d’identifier un visage détecté en le comparant à ceux d’une base de données de personnes (recherche de reconnaissance faciale). Cette fonctionnalité peut être utile pour baliser automatiquement les images dans les logiciels de gestion de photos. Vous créez la base de données à l’avance,qui pourra ensuite être modifiée au fil du temps.

L’image suivante est un exemple de base de données nommée `"myfriends"`. Chaque groupe peut contenir jusqu’à 1 million d’objets Personne différents. Chaque objet Personne peut avoir jusqu’à 248 visages enregistrés.

![Une grille avec 3 colonnes pour les différentes personnes, chacune avec 3 lignes d’images de visages](./Images/person.group.clare.jpg)

Après avoir créé et entraîné une base de données, vous pouvez effectuer l’identification par rapport au groupe avec un nouveau visage détecté. Si le visage est identifié en tant qu’une personne du groupe, l’objet Personne est renvoyé.

Pour plus d’informations sur l’identification de personnes, consultez le guide relatif aux concepts de la [reconnaissance faciale](concepts/face-recognition.md) ou la documentation de référence de l’[API d’identification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239).

## <a name="deploy-on-premises-using-docker-containers"></a>Déployer localement en utilisant des conteneurs Docker

[Utilisez le conteneur Visage (préversion)](face-how-to-install-containers.md) pour déployer des fonctionnalités d’API localement. Ce conteneur Docker vous donne la possibilité de rapprocher le service plus près de vos données, ce qui peut être souhaitable pour des raisons de conformité, de sécurité ou opérationnelles.

## <a name="sample-apps"></a>Exemples d’application

Les exemples d’applications suivants montrent différentes façons d’utiliser le service Visage :

- [API Visage : bibliothèque de client Windows et exemple](https://github.com/Microsoft/Cognitive-Face-Windows) est une application WPF qui présente plusieurs scénarios de détection, d’analyse et d’identification de visages.
- L’[application FamilyNotes UWP](https://github.com/Microsoft/Windows-appsample-familynotes) est une application Universal Windows Platform (UWP) qui utilise l’identification des visages, ainsi que la reconnaissance vocale, Cortana, l’écriture manuscrite et la caméra dans un scénario de partage de notes en famille.

## <a name="data-privacy-and-security"></a>Sécurité et confidentialité des données

Comme avec toutes les ressources Cognitive Services, les développeurs utilisant le service Visage doivent connaître les politiques de Microsoft relatives aux données client. Pour en savoir plus, consultez la [page Cognitive Services](https://www.microsoft.com/trustcenter/cloudservices/cognitiveservices) dans le Centre de gestion de la confidentialité Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Suivez un guide de démarrage rapide pour coder les composants de base d’une application de reconnaissance faciale dans le langage de votre choix.

- [Démarrage rapide : Bibliothèque de client](quickstarts/client-libraries.md).
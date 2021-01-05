---
title: Qu’est-ce que le Lecteur immersif ?
titleSuffix: Azure Cognitive Services
description: Le Lecteur immersif est un outil conçu pour aider les personnes ayant des besoins en apprentissage différents, qui viennent d’apprendre à lire ou qui apprennent une langue avec une compréhension à la lecture.
services: cognitive-services
author: metanMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: overview
ms.date: 01/4/2020
ms.author: metang
ms.custom: cog-serv-seo-aug-2020
keywords: lecteurs, personnes apprenant une langue, afficher des images, améliorer la lecture, lire du contenu, traduire
ms.openlocfilehash: 0661bd5f917a2bf071f11393ad8ac9af53ddfe97
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92631237"
---
# <a name="what-is-immersive-reader"></a>Qu’est-ce que le lecteur immersif ?

Le [Lecteur immersif](https://www.onenote.com/learningtools) est un outil qui permet d’implémenter des techniques éprouvées pour améliorer la compréhension dans le cadre de l’apprentissage de la lecture, l’apprentissage d’une langue et pour les personnes avec des difficultés d’apprentissage, comme la dyslexie. Avec la bibliothèque de client Lecteur immersif, vous pouvez tirer parti de la même technologie que celle utilisée dans Microsoft Word et Microsoft One Note pour améliorer vos applications web. 

## <a name="use-immersive-reader-to-improve-reading-accessibility"></a>Utiliser le Lecteur immersif pour améliorer l’accessibilité de la lecture 

Le Lecteur immersif est conçu pour rendre la lecture plus facile et plus accessible pour tout le monde. Examinons quelques-unes des principales fonctionnalités du Lecteur immersif.

### <a name="isolate-content-for-improved-readability"></a>Isoler du contenu pour améliorer la lisibilité

Le Lecteur immersif isole du contenu pour améliorer la lisibilité. 

  ![Isoler du contenu pour améliorer la lisibilité avec le Lecteur immersif](./media/immersive-reader.png)

### <a name="display-pictures-for-common-words"></a>Afficher des images pour des mots courants

Pour les termes couramment utilisés, le Lecteur immersif va afficher une image.

  ![Dictionnaire d’images avec le Lecteur immersif](./media/picture-dictionary.png)

### <a name="highlight-parts-of-speech"></a>Mettre en évidence des parties d’un discours

Le Lecteur immersif peut être utilisé pour aider les apprenants à comprendre les parties d’un discours et la grammaire en mettant en évidence les verbes, les noms, les pronoms, etc.

  ![Montrer des parties d’un discours avec le Lecteur immersif](./media/parts-of-speech.png)

### <a name="read-content-aloud"></a>Lire du contenu à voix haute

La synthèse vocale est intégrée au service Lecteur immersif, qui permet à vos lecteurs de sélectionner du texte à lire à voix haute. 

  ![Lire le texte à voix haute avec le Lecteur immersif](./media/read-aloud.png)

### <a name="translate-content-in-real-time"></a>Traduire du contenu en temps réel

Le Lecteur immersif peut traduire en temps réel du texte en de nombreuses langues. C’est utile pour améliorer la compréhension des lecteurs qui apprennent une nouvelle langue.

  ![Traduire du texte avec le Lecteur immersif](./media/translation.png)

### <a name="split-words-into-syllables"></a>Décomposer des mots en syllabes

Avec le Lecteur immersif, vous pouvez décomposer des mots en syllabes pour améliorer la lisibilité ou pour prononcer de nouveaux mots en détachant les syllabes.

  ![Décomposer les mots en syllabes avec le Lecteur immersif](./media/syllabification.png)

## <a name="how-does-immersive-reader-work"></a>Comment fonctionne le lecteur immersif ?

Le Lecteur immersif est une application web autonome. Quand elle est appelée en utilisant la bibliothèque de client Lecteur immersif, elle s’affiche par-dessus votre application web existante dans un `iframe`. Quand votre application web appelle le service Lecteur immersif, vous spécifiez le contenu pour montrer le lecteur. La bibliothèque de client Lecteur immersif gère la création et le style du `iframe`, et la communication avec le service back-end du Lecteur immersif. Le service Lecteur immersif traite le contenu pour des parties du discours, la synthèse vocale, la traduction, etc.

## <a name="get-started-with-immersive-reader"></a>Bien démarrer avec le Lecteur immersif

La bibliothèque de client Lecteur immersif est disponible en C#, JavaScript, Java (Android), Kotlin (Android) et Swift (iOS). Commencez avec :

* [Démarrage rapide : Utiliser la bibliothèque de client Lecteur immersif](quickstarts/client-libraries.md)

## <a name="next-steps"></a>Étapes suivantes

Prise en main du lecteur immersif :

* Lire les [Informations de référence sur la bibliothèque de client Lecteur immersif](./reference.md)
* Explorer la [bibliothèque de client Lecteur immersif sur GitHub](https://github.com/microsoft/immersive-reader-sdk)

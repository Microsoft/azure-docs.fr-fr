---
title: L’Interface de ligne de commande Azure Speech
titleSuffix: Azure Cognitive Services
description: L’interface de ligne de commande Speech est un outil en ligne de commande qui vous aide à utiliser le service Speech sans écrire de code. L’interface de ligne commande Speech requiert une configuration minimale et il est facile de commencer immédiatement à expérimenter les fonctionnalités clés du service Speech pour déterminer si vos cas d’utilisation peuvent être atteints.
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/14/2020
ms.author: trbye
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6b852186834fba858e8a049a8230b38f3d69164d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88067424"
---
# <a name="what-is-the-speech-cli"></a>Qu’est-ce que l’interface CLI Speech ?

L’interface de ligne de commande Speech est un outil en ligne de commande qui vous aide à utiliser le service Speech sans écrire de code. L’interface de ligne de commande Speech requiert une configuration minimale et il est facile de commencer immédiatement à expérimenter les fonctionnalités clés du service Speech pour déterminer si vos cas d’utilisation peuvent être atteints. En quelques minutes, vous pouvez exécuter des flux de travail de test simples comme la reconnaissance vocale par lot à partir d’un répertoire de fichiers ou la conversion de texte par synthèse vocale sur une collection de chaînes à partir d’un fichier. Au-delà des flux de travail simples, l’interface de ligne de commande Speech est prête pour la production et peut être mise à l’échelle pour exécuter des processus plus volumineux à l’aide de `.bat` automatisés ou de scripts d’interpréteurs de commandes.

La plupart des fonctionnalités principales du Kit de développement logiciel (SDK) Speech sont disponibles dans l’interface de ligne de commande Speech, et certaines fonctionnalités avancées et personnalisations sont simplifiées dans l’interface de ligne de commande Speech. Tenez compte des conseils suivants pour décider quand utiliser l’interface de ligne de commande Speech ou le kit de développement logiciel (SDK) Speech.

Utilisez l’interface CLI Speech lorsque :
* vous souhaitez expérimenter les fonctionnalités du service Speech avec une installation minimale et sans code
* vous avez des exigences relativement simples pour une application de production qui utilise le service Speech

Utilisez le kit de développement logiciel (SDK) Speech lorsque :
* vous souhaitez intégrer la fonctionnalité de service Speech dans une plateforme ou un langage spécifique (par exemple C# , Python, C++ )
* vous avez des exigences complexes qui peuvent nécessiter des demandes de service avancées ou développer un comportement personnalisé incluant la diffusion en continu des réponses

## <a name="core-features"></a>Fonctionnalités de base

* Reconnaissance vocale : convertissez la parole en texte à partir de fichiers audio ou directement à partir d’un microphone ou transcrivez une conversation enregistrée.

* Synthèse vocale : convertissez la synthèse vocale en entrée à partir de fichiers texte ou en entrée directement à partir de la ligne de commande. Personnalisez les caractéristiques de la sortie vocale à l’aide de [configurations SSML](speech-synthesis-markup.md) et [les voix standard ou neuronales](speech-synthesis-markup.md#standard-neural-and-custom-voices).

* Traduction vocale : traduisez de l’audio dans une langue source en texte ou audio dans une langue cible.

* Exécuter sur des ressources de calcul Azure : envoyer des commandes CLI Speech à exécuter sur une ressource de calcul distante Azure à l’aide de `spx webjob`.

## <a name="get-started"></a>Bien démarrer

Pour commencer à utiliser l’interface de ligne commande Speech, consultez [l’article de base](spx-basics.md). Cet article explique comment exécuter certaines commandes de base et affiche également des commandes légèrement plus avancées pour l’exécution d’opérations par lot pour la reconnaissance vocale et la synthèse vocale. Après avoir lu l’article de base, vous devez disposer d’une connaissance suffisante de la syntaxe pour commencer à écrire des commandes personnalisées ou automatiser des opérations de service Speech simples.

## <a name="next-steps"></a>Étapes suivantes

- [Bases de l’interface CLI Speech](spx-basics.md)
- Si votre cas d’utilisation est plus complexe, [obtenez le kit de développement logiciel (SDK) Speech](speech-sdk.md)

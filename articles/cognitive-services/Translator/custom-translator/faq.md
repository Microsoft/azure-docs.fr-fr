---
title: Forum aux questions - Custom Translator
titleSuffix: Azure Cognitive Services
description: Cet article contient les réponses aux questions fréquentes sur Azure Cognitive Services Custom Translator.
author: swmachan
manager: nitinme
ms.service: cognitive-services
ms.subservice: translator-text
ms.date: 08/17/2020
ms.author: swmachan
ms.topic: reference
ms.openlocfilehash: 0f3d8131635fae55fd8dc2fd2649ae5dda1e62ae
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368234"
---
# <a name="custom-translator-frequently-asked-questions"></a>Questions fréquentes sur Custom Translator

Cet article contient des réponses aux questions fréquemment posées sur [Custom Translator](https://portal.customtranslator.azure.ai).

## <a name="what-are-the-current-restrictions-in-custom-translator"></a>Quelles sont les restrictions actuelles dans Custom Translator ?

Il existe des restrictions et des limites concernant la taille du fichier, l’apprentissage du modèle et le déploiement du modèle. N’oubliez pas ces restrictions lorsque vous configurez votre outil de formation pour générer un modèle dans Custom Translator.

- Les fichiers envoyés doivent être inférieurs à 100 Mo.
- Les données monolingues ne sont pas prises en charge.

## <a name="when-should-i-request-deployment-for-a-translation-system-that-has-been-trained"></a>Quand dois-je demander le déploiement d’un système de traduction qui a été formé ?

La création du système de traduction optimal pour votre projet peut nécessiter plusieurs formations. Voulez pouvez essayer d’utiliser plus de données d’apprentissage ou de données filtrées plus attentivement, si le score BLEU et/ou les résultats des tests ne sont pas satisfaisants. La conception votre jeu de paramétrage et de votre jeu de test doit s’effectuer de manière stricte et prudente, pour être entièrement représentative de la terminologie et du style de documents que vous souhaitez traduire. Vous pouvez procéder à la composition de vos données d’apprentissage de manière plus libre et faire des essais avec différentes options. Demandez un déploiement système lorsque vous êtes satisfait des traductions dans vos résultats de test système, que vous n’avez plus aucune donnée à ajouter à la formation pour améliorer votre système formé, et que vous souhaitez accéder au modèle formé via les API.

## <a name="how-many-trained-systems-can-be-deployed-in-a-project"></a>Combien de systèmes formés peuvent être déployés dans un projet ?

Un seul système formé peut être déployé par projet. La création d’un système de traduction approprié pour votre projet peut nécessiter plusieurs formations, et nous vous encourageons à demander le déploiement d’une formation qui vous donne les meilleurs résultats. Vous pouvez déterminer la qualité de la formation grâce au score BLEU (plus il est élevé, mieux c’est) et en contactant des réviseurs avant de décider que la qualité des traductions est adéquate pour le déploiement.

## <a name="when-can-i-expect-my-trainings-to-be-deployed"></a>Quand mes formations seront-elles déployées ?

En général, le déploiement prend moins d’une heure.

## <a name="how-do-you-access-a-deployed-system"></a>Comment accéder à un système déployé ?

Les systèmes déployés sont accessibles via l’API de traduction de texte Translator Text V3 de Microsoft en spécifiant le CategoryID. Vous trouverez plus d’informations sur l’API de traduction de texte Translator Text dans la page web [Informations de référence sur les API](../reference/v3-0-reference.md).

## <a name="how-do-i-skip-alignment-and-sentence-breaking-if-my-data-is-already-sentence-aligned"></a>Comment ignorer l’alignement et la rupture de phrase si mes données sont déjà alignées par phrase ?

Custom Translator ignore l’alignement de phrase et la rupture de phrase pour les fichiers TMX et les fichiers texte avec l’extension `.align`. Les fichiers `.align` offrent aux utilisateurs la possibilité d’utiliser le processus d’alignement et de découpage des phrases de Custom Translator pour les fichiers parfaitement alignés qui ne réclament aucun traitement supplémentaire. Nous vous recommandons d’utiliser l’extension `.align` uniquement pour les fichiers qui sont parfaitement alignés.

Si le nombre de phrases extraites ne correspond pas aux deux fichiers portant le même nom de base, Custom Translator exécute quand même l’outil d’alignement de phrase sur les fichiers `.align`.

## <a name="i-tried-uploading-my-tmx-but-it-says-document-processing-failed"></a>J’ai essayé de charger mon TMX, mais je reçois le message « Échec de traitement du document »


Vérifiez que le TMX est conforme à la spécification TMX 1.4b sous <https://www.gala-global.org/tmx-14b>.
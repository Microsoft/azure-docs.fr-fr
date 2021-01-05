---
title: Mots clés personnalisés - Service Speech
titleSuffix: Azure Cognitive Services
description: Vue d’ensemble des fonctionnalités, caractéristiques et restrictions des mots clés personnalisés à l’aide du SDK Speech.
services: cognitive-services
author: hasyashah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/06/2020
ms.author: hasshah
ms.custom: devx-track-csharp
ms.openlocfilehash: b2763372cc63637f73610e1936e3ea54a4524e7d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91356602"
---
# <a name="what-is-a-keyword"></a>Qu’est-ce qu’un mot clé ?

Un mot clé est un mot ou une expression courte qui permet d’activer la fonction vocale de votre produit. Par exemple, « Hey Cortana » est le mot clé de l’assistant Cortana. Avec l’activation de la fonction vocale, vos utilisateurs peuvent commencer à interagir avec votre produit en mode mains libres, simplement en prononçant le mot clé. Comme votre produit écoute en permanence le mot clé, toutes les données audio sont traitées localement sur l’appareil de l’utilisateur jusqu’à ce qu’une détection se produise afin de garantir que ses données restent aussi privées que possible.

## <a name="core-features-of-custom-keyword"></a>Fonctionnalités principales d’un mot clé personnalisé

Avec les fonctionnalités de personnalisation, de performances et d’intégration des mots clés personnalisés, vous pouvez adapter l’activation de la fonction vocale pour mieux répondre à l’objectif de votre produit et aux besoins de vos utilisateurs.

| Fonctionnalité | Description |
|----------|----------|
| Personnalisation des mots clés | Extension de votre marque, un mot clé renforce la relation que vous avez tissée avec vos clients. Le portail des mots clés personnalisés sur Speech Studio vous permet de spécifier n’importe quel mot ou n’importe quelle expression courte qui représente le mieux votre marque. Vous pouvez personnaliser davantage votre mot clé en choisissant les prononciations appropriées, qui seront respectées par le modèle de mot clé généré.
| Vérification du mot clé | Quand le niveau de confiance du mot clé détecté localement est élevé, le contenu audio est envoyé au cloud pour vérifier plus en profondeur qu’un utilisateur a prononcé le mot clé. La vérification du mot clé fournit une couche de sécurité supplémentaire en réduisant l’impact d’une mauvaise détection locale incorrecte et en protégeant la confidentialité des utilisateurs.
| Intégration de l’assistant vocal et du SDK Speech | Les mots clés générés à partir de l’option de mot clé personnalisé sur Speech Studio peuvent être facilement intégrés à votre appareil ou application par le biais du SDK Speech. Pointez simplement le SDK vers le modèle de mot clé fourni par Speech Studio : votre produit sera activé par la voix, avec un renforcement par une vérification du mot clé. Vous pouvez effectuer des expériences commandées par la voix sur votre produit en générant votre propre [assistant vocal](voice-assistants.md).

## <a name="get-started-with-custom-keywords"></a>Bien démarrer avec les mots clés personnalisés

* Consultez [notions de base sur les mots clés personnalisés](custom-keyword-basics.md) pour les modèles d’utilisation et de conception de base.
* [Activer votre produit par la voix avec le SDK Speech, à l'aide de C#](tutorial-voice-enable-your-bot-speech-sdk.md)

## <a name="choose-an-effective-keyword"></a>Choisir un mot clé efficace

Il est essentiel de créer un mot clé efficace pour que votre appareil réponde bien avec logique et précision. Pour différencier votre appareil et renforcer votre marque, vous pouvez personnaliser votre mot clé. Vous devez respecter les consignes suivantes lors du choix d’un mot clé :

> [!div class="checklist"]
> * Votre mot clé doit être un mot ou une expression en anglais.
> * Le/la prononcer ne doit pas prendre plus de deux secondes.
> * Il est préférable d’utiliser un mot de quatre à sept syllabes. Par exemple, « Hey, Computer » est un bon mot clé, ce qui n’est pas le cas du mot « Hey » seul.
> * Les mots clés doivent suivre les règles courantes de prononciation de la langue anglaise.
> * Un mot unique ou même inventé qui suit les règles courantes de prononciation de la langue anglaise peut réduire les faux positifs. Par exemple, « computerama » pourrait être un bon mot clé.
> * Ne choisissez pas un mot courant, comme « eat » et « go » qui sont prononcés très fréquemment dans les conversations. Ces faux déclencheurs activeraient à tort votre appareil.
> * Évitez d’utiliser un mot clé qui a plusieurs prononciations. Sinon, les utilisateurs devront connaître la « bonne » prononciation pour obtenir une réponse de leur appareil. Par exemple, « 509 » peut se prononcer « five zero nine », « five oh nine » ou « five hundred and nine ». « R.E.I. » peut être prononcé « r-e-i » ou « ray ». « Live » peut être prononcé « /līv/ » ou « /liv/ ».
> * N’utilisez pas de caractères spéciaux, de symboles ou de chiffres. Par exemple, « Go# » et « 20+ cats » peuvent être des mots clés problématiques. Cependant, « go sharp » ou « twenty plus cats » peuvent fonctionner. Pour rendre évidente la bonne prononciation, vous pouvez utiliser les symboles de votre marque et vous servir du marketing et de la documentation.

> [!NOTE]
> Si vous choisissez un mot commercial comme mot clé, vous devez être propriétaire de cette marque commerciale ou avoir l’autorisation de son propriétaire pour utiliser le mot. Microsoft n’est pas responsable des problèmes juridiques que pourrait provoquer votre choix de mot clé.

## <a name="see-samples-on-github"></a>Voir des exemples sur GitHub

* [Reconnaître les mots clés avec le SDK Speech, sur la plateforme Windows universelle à l'aide de C#](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/csharp/uwp/keyword-recognizer)
* [Reconnaître les mots clés avec le SDK Speech, sur Android à l'aide de Java](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/quickstart/java/android/keyword-recognizer)

## <a name="next-steps"></a>Étapes suivantes

* [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
* [Obtenir le kit SDK Speech](speech-sdk.md)
* [En savoir plus sur les assistants vocaux](voice-assistants.md)

---
title: Conversation multi-appareil (préversion) - Service Speech
titleSuffix: Azure Cognitive Services
description: ''
services: cognitive-services
author: trevorbye
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 03/11/2020
ms.author: trbye
ms.openlocfilehash: 3c2dbaf8f65208126c52617d4b89d901b3670dac
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025172"
---
# <a name="what-is-multi-device-conversation-preview"></a>Qu'est-ce que la conversation multi-appareil (préversion) ?

La **conversation multi-appareil** permet de lancer facilement une conversation vocale ou textuelle entre plusieurs clients et de coordonner les messages envoyés.

Grâce à la **conversation multi-appareil**, vous pouvez :

- Connecter plusieurs clients à une même conversation, et gérer l'envoi et la réception des messages échangés entre eux
- Transcrire facilement les messages audio de chaque client et envoyer la transcription aux autres participants, avec traduction facultative
- Envoyer facilement des messages texte entre les clients, avec traduction facultative

Vous pouvez créer une fonctionnalité ou une solution compatible avec différents types d'appareils. Chaque appareil peut indépendamment envoyer des messages (transcriptions audio ou messages instantanés) à tous les autres appareils.

Contrairement à la [**transcription de conversation**](conversation-transcription.md) qui fonctionne sur un seul appareil avec un réseau de microphones multicanaux, la **conversation multi-appareil** fonctionne avec plusieurs appareils, chacun doté d'un seul microphone.

>[!IMPORTANT]
> La conversation multi-appareil ne prend **pas** en charge l'envoi de fichiers audio entre clients. Seules la transcription et/ou la traduction sont prises en charge.

## <a name="key-features"></a>Fonctionnalités clés

- **Transcription en temps réel** : chaque participant reçoit une transcription de la conversation, ce qui lui permet de suivre le texte en temps réel ou de le conserver pour l'utiliser ultérieurement.
- **Traduction en temps réel** : grâce à la [prise en charge de plus de 60 langues](language-support.md#text-languages) pour la traduction de texte, les utilisateurs peuvent traduire la conversation dans la ou les langues de leur choix.
- **Transcriptions lisibles** : grâce à la ponctuation et aux coupures de phrases, la transcription et la traduction sont faciles à suivre.
- **Entrée vocale ou entrée de texte** : chaque utilisateur peut parler ou taper sur son appareil, selon les fonctionnalités de prise en charge linguistique activées pour la langue choisie par le participant. Consultez [Langues prises en charge](language-support.md#speech-to-text).
- **Relais des messages** : le service de conversation multi-appareil distribue les messages envoyés par un client à tous les autres, dans la ou les langues de leur choix.
- **Identification des messages** : chaque message que les utilisateurs reçoivent dans le cadre de la conversation est étiqueté avec le surnom de celui qui l'a envoyé.

## <a name="use-cases"></a>Cas d'utilisation

### <a name="lightweight-conversations"></a>Conversations légères

Il est très facile de créer et de rejoindre une conversation. Un utilisateur « hôte » lance une conversation, ce qui génère un code de conversation aléatoire de cinq lettres et un code QR. Tous les autres utilisateurs peuvent rejoindre la conversation en saisissant le code de celle-ci ou en scannant son code QR. 

Dans la mesure où les utilisateurs se connectent via le code de conversation et ne sont pas tenus de partager leurs coordonnées, la création de conversations est simple et rapide.

### <a name="inclusive-meetings"></a>Réunions inclusives

La transcription et la traduction en temps réel peuvent rendre les conversations accessibles aux personnes qui parlent des langues différentes et/ou aux personnes sourdes ou malentendantes. Chaque personne peut également participer activement à la conversation, en s'exprimant dans la langue de son choix ou en envoyant des messages instantanés.

### <a name="presentations"></a>Présentations

Vous pouvez également afficher des sous-titres lors des présentations et des conférences, à la fois à l'écran et sur les appareils des membres du public. Une fois que les membres du public ont rejoint la présentation à l'aide du code de conversation, la transcription peut apparaître sur leur appareil, dans la langue de leur choix.

> [!NOTE]
> Pour consulter un exemple, reportez-vous à [Traducteur de présentations](https://www.microsoft.com/translator/apps/presentation-translator/), un complément PowerPoint qui utilise le service de conversation multi-appareil. Vous pouvez le télécharger [ici](https://download.cnet.com/s/powerpoint-add-in/).

## <a name="how-it-works"></a>Fonctionnement

Tous les clients utiliseront le kit de développement logiciel (SDK) Speech pour créer ou rejoindre une conversation. Le kit SDK Speech interagit avec le service de conversation multi-appareil, qui gère la durée de vie d'une conversation, ainsi que la liste des participants, la ou les langues choisies par chaque client et les messages envoyés.  

Chaque client peut envoyer des messages audio ou des messages instantanés. Le service utilisera la reconnaissance vocale pour convertir les messages audio en texte et envoyer des messages instantanés en l'état. Si les clients choisissent des langues différentes, le service traduira tous les messages dans la ou les langues qu'ils ont spécifiées.

![Diagramme de présentation d'une conversation multi-appareil](media/scenarios/multi-device-conversation.png)

## <a name="overview-of-conversation-host-and-participant"></a>Présentation de la conversation, de l'hôte et du participant

Une **conversation** est une session lancée par un utilisateur pour permettre aux autres participants de la rejoindre. Tous les clients se connectent à la conversation à l'aide d'un **code de conversation** en cinq lettres.

Chaque conversation crée des métadonnées qui incluent ce qui suit :
-    Horodatages de début et de fin de conversation
-    Liste de tous les participants - celle-ci comprend le surnom choisi par chaque utilisateur ainsi que sa langue principale pour l'entrée vocale ou l'entrée de texte


Deux types d'utilisateurs participent à une conversation : l'**hôte** et les **participants**.

L'**hôte** est l'utilisateur qui lance la conversation et qui joue le rôle d'administrateur de celle-ci.
- Il ne peut y avoir qu'un seul hôte par conversation.
- L'hôte doit être connecté à la conversation pendant toute la durée de celle-ci. Si l'hôte quitte la conversation, elle prend fin pour tous les autres participants.
- L'hôte dispose de quelques commandes supplémentaires pour gérer la conversation : 
    - Verrouiller la conversation : empêche d'autres participants de rejoindre la conversation
    - Désactiver le micro de tous les participants : empêche les autres participants d'envoyer des messages dans le cadre de la conversation, qu'ils soient transcrits à partir de messages vocaux ou de messages instantanés
    - Désactiver le micro de participants individuels
    - Réactiver le micro de tous les participants
    - Réactiver le micro de participants individuels

Un **participant** est un utilisateur qui rejoint une conversation.
- Un participant peut quitter et rejoindre la même conversation à tout moment, sans y mettre fin pour les autres participants.
- Les participants ne peuvent pas verrouiller la conversation ou désactiver/réactiver le micro des autres participants.

> [!NOTE]
> Chaque conversation peut accueillir un maximum de 100 participants, dont 10 peuvent parler simultanément.

## <a name="language-support"></a>Support multilingue

Lorsqu'il crée ou rejoint une conversation, chaque utilisateur doit choisir une **langue principale** : la langue dans laquelle il parlera, enverra des messages instantanés et verra les messages des autres utilisateurs.

Il existe deux types de langues : **Reconnaissance vocale** et **Texte uniquement** :
- Si l'utilisateur choisit une langue de type **Reconnaissance vocale** comme langue principale, il pourra utiliser à la fois l'entrée vocale et l'entrée de texte dans le cadre de la conversation.

- Si l'utilisateur choisit une langue de type **Texte uniquement**, il pourra seulement utiliser l'entrée de texte et envoyer des messages instantanés dans le cadre de la conversation. Les langues de type Texte uniquement sont les langues prises en charge pour la traduction de texte, mais pas pour la reconnaissance vocale. Pour connaître les langues disponibles, consultez la page [Langues prises en charge](./language-support.md).

Outre sa langue principale, chaque participant peut également spécifier des langues supplémentaires pour traduire la conversation.

Vous trouverez ci-dessous un récapitulatif de ce que l'utilisateur peut faire dans le cadre d'une conversation multi-appareil en fonction de la langue principale qu'il a choisie.


| Ce que l'utilisateur peut faire dans le cadre de la conversation | Reconnaissance vocale | Texte uniquement |
|-----------------------------------|----------------|------|
| Utiliser l'entrée vocale | ✔️ | ❌ |
| Envoyer des messages instantanés | ✔️ | ✔️ |
| Traduire la conversation | ✔️ | ✔️ |

> [!NOTE]
> Les langues compatibles avec la reconnaissance vocale et la traduction de texte sont répertoriées dans [Langues prises en charge](./language-support.md).



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Traduire les conversations en temps réel](quickstarts/multi-device-conversation.md)
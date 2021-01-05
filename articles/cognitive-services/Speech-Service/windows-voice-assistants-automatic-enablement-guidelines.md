---
title: Assistants vocaux Windows - Directives en matière de confidentialité
titleSuffix: Azure Cognitive Services
description: Instructions relatives à l'activation vocale d'un agent vocal par défaut.
services: cognitive-services
author: cfogg6
manager: trrwilson
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: travisw
ms.openlocfilehash: a9fe083818c5850ad05556b18911c19fe19e0e72
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987397"
---
# <a name="privacy-guidelines-for-voice-assistants-on-windows"></a>Assistants vocaux Windows - Directives en matière de confidentialité

Il est important de fournir aux utilisateurs des informations claires sur la façon dont leurs données vocales sont collectées et utilisées, de même qu'il est important qu'ils aient le contrôle sur le mode de collecte de ces données. Ces facettes essentielles de la confidentialité (*divulgation* et *consentement*) sont particulièrement importantes pour les assistants vocaux intégrés à Windows compte tenu de la nature de leur utilisation, basée sur l'écoute permanente.

Les développeurs qui créent des assistants vocaux sous Windows doivent inclure dans leurs applications des éléments d'interface utilisateur clairs qui reflètent les capacités d'écoute de l'assistant.

> [!NOTE]
> En l'absence de divulgation et de consentement appropriés, y compris après les mises à jour de l'application, l'activation vocale de l'assistant peut devenir indisponible jusqu'à ce que les problèmes de confidentialité soient résolus.

## <a name="minimum-requirements-for-feature-inclusion"></a>Exigences minimales en matière d'inclusion de fonctionnalités

Les utilisateurs de Windows peuvent voir et contrôler la disponibilité de leurs applications d'assistant sous **`Settings > Privacy > Voice activation`** .

 > [!div class="mx-imgBorder"]
 > [![La capture d’écran montre des options permettant de contrôler la disponibilité de Cortana. ](media/voice-assistants/windows_voice_assistant/privacy-app-listing.png "Entrée de paramètre de confidentialité d'activation vocale Windows pour une application d'assistant")](media/voice-assistants/windows_voice_assistant/privacy-app-listing.png#lightbox)

Pour être éligible et être inclus dans cette liste, contactez Microsoft à l’adresse winvoiceassistants@microsoft.com. Par défaut, les utilisateurs doivent activer explicitement l’activation vocale pour un nouvel assistant dans **`Settings > Privacy > Voice Activation`** , auquel une application peut effectuer une liaison de protocole avec `ms-settings:privacy-voiceactivation`. Une application autorisée s’affiche dans la liste une fois qu’elle a été exécutée et a utilisé les API `Windows.ApplicationModel.ConversationalAgent`. Ses paramètres d’activation vocale pourront être modifiés une fois que l’application aura obtenu le consentement relatif au microphone de la part de l’utilisateur.

Étant donné que les paramètres de confidentialité de Windows incluent des informations sur le fonctionnement de l’activation vocale et sur le fait qu’elle dispose d’une interface utilisateur standard pour le contrôle de l’autorisation, la divulgation et le consentement sont tous deux satisfaits. L’assistant restera dans cette liste autorisée tant qu’il ne fera pas ce qui suit :

* Induire en erreur ou désinformer l’utilisateur sur l’activation vocale ou la gestion des données vocales par l’assistant
* Interférer indûment avec un autre assistant
* Rompre toute autre stratégie Microsoft pertinente

Si l’une des actions ci-dessus est découverte, Microsoft peut supprimer un assistant de la liste autorisée jusqu’à ce que les problèmes soient résolus.

> [!NOTE]
> Dans tous les cas, l'autorisation d'activation vocale nécessite une autorisation d'accès à un microphone. Si l'application d'assistant n'a pas accès à un microphone, elle n'est pas éligible à l'activation vocale et apparaît comme désactivée dans les paramètres de confidentialité de l'activation vocale.

## <a name="additional-requirements-for-inclusion-in-microphone-consent"></a>Exigences supplémentaires pour l'inclusion dans le consentement relatif au microphone

Les créateurs d’assistants qui souhaitent faciliter l’accès à l’activation vocale à leurs utilisateurs doivent satisfaire des exigences supplémentaires pour se conformer pleinement aux conditions de divulgation et de consentement sans avoir à visiter de nouveau la page des paramètres. Une fois approuvée, l’activation vocale est immédiatement disponible dès qu’un utilisateur accorde l’autorisation relative au microphone à l’application d’assistant. Pour ce faire, une application d’assistant doit faire ce qui suit **avant** de demander le consentement relatif au microphone (par exemple, à l’aide de l’API `AppCapability.RequestAccessAsync`) :

1. Indiquer clairement à l’utilisateur que l’application attend qu’il prononce un mot clé, *même lorsque l’application n’est pas en cours d’exécution*, et souhaite obtenir son consentement
1. Inclure des informations pertinentes sur les stratégies d’utilisation et de confidentialité des données, comme un lien vers une déclaration de confidentialité officielle
1. Éviter toute formulation directive ou suggestive (par exemple, « Cliquez sur Oui à l’invite suivante ») dans le flux d’expérience divulguant le comportement de capture audio

Si une application respecte toutes les exigences ci-dessus, elle est éligible pour activer la fonctionnalité d’activation vocale ainsi que le consentement relatif au microphone. Pour plus d’informations et pour consulter une expérience de première utilisation, contactez winvoiceassistants@microsoft.com.

> [!NOTE]
> L'activation vocale par-dessus le verrouillage n'est pas éligible à l'activation automatique avec accès au microphone et oblige toujours l'utilisateur à se rendre sur la page de confidentialité de l'activation vocale pour activer l'accès par-dessus le verrouillage d'un assistant.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les meilleures pratiques relatives aux assistants vocaux Windows](windows-voice-assistants-best-practices.md)
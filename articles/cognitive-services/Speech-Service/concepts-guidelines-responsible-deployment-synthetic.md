---
title: Instructions pour le déploiement responsable de la technologie des voix de synthèse
titleSuffix: Azure Cognitive Services
description: Instructions de conception générale de Microsoft pour l’utilisation de la technologie des voix de synthèse. Elles ont été développées dans des études menées par Microsoft auprès de voix professionnelles, de consommateurs, ainsi que de personnes ayant des troubles de la parole pour guider le développement responsable des voix de synthèse.
services: cognitive-services
author: benoah
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 10/11/2019
ms.author: benoah
ms.openlocfilehash: 7d80ffb575c6aa15695279584b58288cbc16be43
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024974"
---
# <a name="guidelines-for-responsible-deployment-of-synthetic-voice-technology"></a>Instructions pour le déploiement responsable de la technologie des voix de synthèse
Voici les instructions de conception générale de Microsoft pour l’utilisation de la technologie des voix de synthèse. Elles ont été développées dans des études menées par Microsoft auprès de voix professionnelles, de consommateurs, ainsi que de personnes ayant des troubles de la parole pour guider le développement responsable des voix de synthèse.

## <a name="general-considerations"></a>Considérations d’ordre général
Pour le déploiement de la technologie des voix de synthèse, les instructions suivantes s’appliquent dans la plupart des scénarios.

### <a name="disclose-when-the-voice-is-synthetic"></a>Révéler qu’il s’agit d’une voix de synthèse
Le fait de révéler qu’une voix est générée par ordinateur permet de réduire les déceptions éventuelles, mais aussi d’augmenter la confiance en l’organisation qui fournit la voix. En savoir plus sur [comment le révéler](concepts-disclosure-guidelines.md).

### <a name="select-appropriate-voice-types-for-your-scenario"></a>Sélectionner les types de voix appropriés pour votre scénario
Examinez attentivement le contexte d’utilisation et les risques potentiels associés à l’utilisation de voix de synthèse. Par exemple, des voix de synthèse très fidèles ne sont pas forcément appropriées pour les scénarios à haut risque, tels que les messages personnels, les transactions financières ou les situations complexes pour lesquels une certaine empathie ou humanité est nécessaire. Les utilisateurs peuvent également avoir des attentes différentes pour les types de voix. Par exemple, lorsqu’ils écoutent les dernières actualités sensibles lues par une voix de synthèse, certains utilisateurs préféreront un ton plus humain et plus empathique, tandis que d’autres aimeraient une voix plus monotone et neutre. Pensez à tester votre application pour mieux comprendre les préférences des utilisateurs.

### <a name="be-transparent-about-capabilities-and-limitations"></a>Faire preuve de transparence au sujet des capacités et des limitations
Les utilisateurs auront vraisemblablement des attentes plus élevées s’ils interagissent avec des agents vocaux de synthèse. Par conséquent, lorsque les capacités du système sont en deçà de ces attentes, une baisse de confiance peut être observée et conduire à des expériences désagréables voire négatives.

### <a name="provide-optional-human-support"></a>Proposer une assistance humaine facultative
Dans certains scénarios transactionnels et ambigus (par exemple, un centre d’appel), les utilisateurs ne font pas toujours confiance à un agent virtuel pour répondre correctement à leurs demandes. Une assistance humaine peut être nécessaire dans ces situations, quels que soient le degré de réalisme de la voix ou les capacités du système.

## <a name="considerations-for-voice-talent"></a>Considérations relatives aux voix professionnelles
Lorsque vous travaillez avec des voix professionnelles (p. ex. des acteurs) pour créer des voix de synthèse, l’instruction suivante s’applique.

### <a name="obtain-meaningful-consent-from-voice-talent"></a>Obtenir le consentement explicite des voix professionnelles
Les voix professionnelles s’attendent à avoir un contrôle sur leur police vocale (comment et où elle sera utilisée) et à être rémunérées chaque fois qu’elle est utilisée. Les propriétaires du système doivent donc obtenir une autorisation écrite explicite de la part des voix professionnelles et établir un contrat clair sur les cas d’utilisation, la durée d’utilisation, la rémunération, etc. Certaines voix professionnelles n’ont pas conscience des utilisations potentiellement malveillantes de la technologie ; les propriétaires du système doivent donc les informer à ce sujet. Pour plus d’informations sur le talent vocal et le consentement, lisez notre [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent).


## <a name="considerations-for-those-with-speech-disorders"></a>Considérations relatives aux personnes ayant des troubles de la parole
Lorsque vous travaillez avec des personnes ayant des troubles de la parole, pour créer ou déployer une technologie de voix de synthèse, les instructions suivantes s’appliquent.

### <a name="provide-guidelines-to-establish-contracts"></a>Fournir des instructions pour établir des contrats
Fournir des instructions pour établir des contrats avec les personnes qui utilisent les voix de synthèse pour s’exprimer. Le contrat doit indiquer les parties propriétaires de la voix, la durée d’utilisation, les critères de transfert de propriété, les procédures pour supprimer la police vocale et comment empêcher les accès non autorisés. En outre, permettez le transfert du contrat de propriété de la police vocale après le décès aux membres de la famille si la personne concernée en a donné l’autorisation.

### <a name="account-for-inconsistencies-in-speech-patterns"></a>Tenir compte des incohérences dans les modèles vocaux
Pour les personnes ayant des troubles de la parole et qui enregistrent leurs propres polices vocales, les incohérences dans leur modèle de discours (déformation ou incapacité à prononcer certains mots) peuvent compliquer le processus d’enregistrement. Dans ce cas, la technologie de voix de synthèse et des sessions d’enregistrement doivent les aider (autrement dit, proposer des pauses et d’autres sessions d’enregistrement).

### <a name="allow-modification-over-time"></a>Pouvoir appliquer des modifications ultérieurement
Les personnes ayant des troubles de la parole souhaitent modifier leur voix de synthèse pour qu’elle tienne compte de leur âge (par exemple, un enfant qui devient adolescent). Les utilisateurs peuvent également avoir des préférences stylistiques évoluant au fil du temps et peuvent souhaiter modifier la tonalité, l’accent ou d’autres caractéristiques vocales.


## <a name="reference-docs"></a>Documents de référence

* [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent)
* [Vue d’ensemble du contingentement](concepts-gating-overview.md)
* [Comment divulguer](concepts-disclosure-guidelines.md)
* [Modèles de conception de révélation](concepts-disclosure-patterns.md)

## <a name="next-steps"></a>Étapes suivantes

* [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent)
* [Comment divulguer](concepts-disclosure-guidelines.md)
* [Modèles de conception de révélation](concepts-disclosure-patterns.md)
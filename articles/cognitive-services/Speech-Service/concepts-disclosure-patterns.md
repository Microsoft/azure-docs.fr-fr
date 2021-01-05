---
title: Modèles de conception de divulgation
titleSuffix: Azure Cognitive Services
description: Modèles de conception et bonnes pratiques pour la divulgation.
services: cognitive-services
author: sharonlo101
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: shlo
ms.openlocfilehash: 348277d42c543bc45cf98bfeaf7ea8313afe8f6f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021981"
---
# <a name="disclosure-design-patterns"></a>Modèles de conception de divulgation
Maintenant que vous avez déterminé le [niveau de divulgation](concepts-disclosure-guidelines.md#disclosure-assessment) adéquat pour votre expérience de voix de synthèse, le moment est venu d’explorer les modèles de conception potentiels.
## <a name="overview"></a>Vue d’ensemble
Vous pouvez appliquer un large éventail de modèles de conception de divulgation à votre expérience de voix de synthèse. Si le résultat de votre évaluation est une « divulgation élevée », nous vous recommandons une [**divulgation explicite**](#explicit-disclosure), ce qui implique de communiquer franchement les origines de la voix de synthèse. Une [**divulgation implicite**](#implicit-disclosure) comprend des signaux et des modèles d’interaction qui profitent aux expériences vocales, que les niveaux de divulgation exigés soient élevés ou faibles.
![Éventail des modèles de divulgation](media/responsible-ai/disclosure-patterns/affordances.png)






| Modèles de divulgation explicite                                                                                                                                                                                    | Modèles de divulgation implicite                                                                 |
|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
|[Présentation transparente](#transparent-introduction)<br> [Présentation transparente verbale](#verbal-transparent-introduction)<br>  [Signature explicite](#explicit-byline)<br>  [Personnalisation et réglage](#customization-and-calibration)<br> [Divulgation parentale](#parental-disclosure)<br> [Offrir des possibilités d’en savoir plus sur la façon dont la voix a été fabriquée](#providing-opportunities-to-learn-more-about-how-the-voice-was-made) | [Divulgation des fonctions](#capability-disclosure)<br>[Signaux et commentaires implicites](#implicit-cues--feedback)<br> [Transparence conversationnelle](#conversational-transparency) |



Utilisez le graphique suivant pour vous référer directement aux modèles applicables à votre voix de synthèse. Certaines autres conditions indiquées dans ce graphique peuvent également s’appliquer à votre scénario :<br/>



| Si votre expérience de voix de synthèse… | Recommandations | Modèles de conception |
| --- | --- | --- |
| Nécessite une divulgation élevée  | Utilisez au moins un modèle explicite et des signaux implicites en amont pour aider les utilisateurs à élaborer des associations. |[Divulgation explicite](#explicit-disclosure)<br>[Divulgation implicite](#implicit-disclosure)  |
| Nécessite une divulgation faible | La divulgation peut être minime voire inutile, mais peut profiter à certains modèles implicites. | [Divulgation des fonctions](#capability-disclosure)<br>[Transparence conversationnelle](#conversational-transparency)  |
| A un haut niveau d’engagement | Voyez à long terme et offrez plusieurs points d’entrée à la divulgation tout au long du parcours utilisateur. Il est vivement recommandé d’avoir une expérience d’intégration. | [Présentation transparente](#transparent-introduction)<br>[Personnalisation et réglage](#customization-and-calibration)<br>[Divulgation des fonctions](#capability-disclosure) |
| Inclut les enfants comme public concerné principal | Ciblez les parents en tant que public de divulgation principal et assurez-vous qu’ils peuvent communiquer efficacement la divulgation aux enfants.  | [Divulgation parentale](#parental-disclosure)<br>[Présentation transparente verbale](#verbal-transparent-introduction)<br> [Divulgation implicite](#implicit-disclosure)<br> [Transparence conversationnelle](#conversational-transparency)  |
| Inclut des utilisateurs non-voyants ou malvoyants en tant que public concerné principal  | Incluez tous les utilisateurs et assurez-vous que toutes les formes de divulgation visuelle sont associées à du texte ou à des effets sonores alternatifs. Respectez les normes d’accessibilité en termes de rapport de contraste et de taille d’affichage. Utilisez des signaux auditifs pour communiquer la divulgation.  | [Présentation transparente verbale](#verbal-transparent-introduction) <br>[Signaux auditifs](#implicit-cues--feedback)<br>[Signaux tactiles](#implicit-cues--feedback)<br>[Transparence conversationnelle](#conversational-transparency)<br>[Normes d’accessibilité](https://www.microsoft.com/accessibility) |
| N’inclut pas d’écran, ni d’appareil ou utilise la voix comme mode d’interaction principal ou exclusif | Utilisez des signaux auditifs pour communiquer la divulgation. | [Présentation transparente verbale](#verbal-transparent-introduction) <br> [Signaux auditifs](#implicit-cues--feedback)  |
| Inclut éventuellement plusieurs utilisateurs/personnes qui écoutent (par exemple, un assistant personnel)  | N’oubliez pas les divers contextes utilisateur et niveaux de compréhension et proposez plusieurs possibilités de divulgation dans le parcours utilisateur.  | [Présentation transparente (retour utilisateur)](#transparent-introduction)<br> [Offrir des possibilités d’en savoir plus sur la façon dont la voix a été fabriquée](#providing-opportunities-to-learn-more-about-how-the-voice-was-made)<br> [Transparence conversationnelle](#conversational-transparency)  |



## <a name="explicit-disclosure"></a>Divulgation explicite
Si votre expérience de voix de synthèse nécessite une divulgation élevée, il est préférable d’utiliser au moins l’un des modèles explicites suivants pour indiquer clairement la nature synthétique.
### <a name="transparent-introduction"></a>Présentation transparente

Avant que l’expérience vocale ne démarre, présentez l’assistant numérique en étant complètement transparent sur les origines de sa voix et ses fonctions. Le meilleur moment pour utiliser ce modèle est pendant l’intégration d’un nouvel utilisateur ou la présentation de nouvelles fonctionnalités à un utilisateur déjà connu. L’implémentation de signaux implicites pendant une présentation aide les utilisateurs à élaborer un modèle mental sur la nature synthétique de l’agent numérique.

#### <a name="first-time-user-experience"></a>Première expérience utilisateur

![Présentation transparente à la première expérience](media/responsible-ai/disclosure-patterns/transparent-intro-first.png) <br>
*La voix de synthèse est présentée pendant l’intégration d’un nouvel utilisateur.*

Recommandations
- Expliquer que la voix est artificielle (par exemple, &quot;numérique&quot;)
- Décrire ce que l’agent est capable de faire
- Indiquer explicitement les origines de la voix
- Offrir un point d’entrée pour en savoir plus sur la voix de synthèse

#### <a name="returning-user-experience"></a>Expérience utilisateur réitérée

Si un utilisateur ignore l’expérience d’intégration, continuez à proposer des points d’entrée vers l’expérience de présentation transparente jusqu’à ce que l’utilisateur déclenche la voix pour la première fois.
<br/>

![Présentation transparente pendant une expérience utilisateur réitérée](media/responsible-ai/disclosure-patterns/transparent-intro-return.png)<br/>
*Proposez un point d’entrée cohérent vers l’expérience de voix de synthèse. Permettez à l’utilisateur de revenir à l’expérience d’intégration quand il déclenche la voix pour la première fois à tout moment du parcours utilisateur.*


### <a name="verbal-transparent-introduction"></a>Présentation transparente verbale

Une invite orale indiquant les origines de la voix de l’assistant numérique est suffisamment explicite pour la divulgation. Ce modèle convient davantage aux scénarios de divulgation élevée dans lesquels la voix est le seul mode d’interaction disponible.
<br/>

![Présentation transparente verbale](media/responsible-ai/disclosure-patterns/spoken-prompt-1.png)
<br/>*Utilisez une présentation transparente quand l’expérience utilisateur comprend des moments où vous pouvez peut-être déjà présenter ou prêter la voix d’une personne.*


![Présentation transparente verbale à la première personne](media/responsible-ai/disclosure-patterns/spoken-prompt-2.png)<br/>
*Pour plus de transparence, l’acteur peut divulguer les origines de la voix de synthèse à la première personne.*

### <a name="explicit-byline"></a>Signature explicite

Utilisez ce modèle si l’utilisateur va interagir avec un lecteur audio ou un composant interactif pour déclencher la voix.


![Signature explicite dans un scénario de médias d’actualités](media/responsible-ai/disclosure-patterns/explicit-byline.png) <br/>
*Une signature explicite est l’attribution de l’origine de la voix.*

Recommandations

- Offrir un point d’entrée pour en savoir plus sur la voix de synthèse

### <a name="customization-and-calibration"></a>Personnalisation et réglage

Laissez les utilisateurs contrôler la façon dont l’assistant numérique leur répond (par exemple, le son de la voix).  Quand un utilisateur interagit avec un système selon ses propres conditions et avec des objectifs spécifiques à l’esprit, par définition, il a déjà compris qu’il ne s’agit pas d’une personne réelle.

#### <a name="user-control"></a>Contrôle utilisateur

Proposez des choix qui exercent un impact significatif et notable sur l’expérience de voix de synthèse.

![Préférences utilisateur](media/responsible-ai/disclosure-patterns/customization-user-control.png)<br/>
*Les préférences utilisateur permettent aux utilisateurs de personnaliser et d’améliorer leur expérience.*

Recommandations

- Permettre aux utilisateurs de personnaliser la voix (par exemple, de sélectionner la langue et le type de voix)
- Proposer aux utilisateurs un moyen d’apprendre au système à répondre à sa voix unique (par exemple, un réglage de la voix, des commandes personnalisées)
- Optimiser les interactions contextuelles ou générées par l’utilisateur (par exemple, des rappels)

#### <a name="persona-customization"></a>Personnalisation du personnage

Proposer des moyens de personnaliser la voix de l’assistant numérique. Si la voix se base sur celle d’une célébrité ou d’une personne très connue, envisagez d’utiliser à la fois des présentations visuelles et orales de la voix quand les utilisateurs en demandent un aperçu.

![Personnalisation de la voix](media/responsible-ai/disclosure-patterns/customization-voice-type.png)<br/>
*Offrir la possibilité de choisir parmi un ensemble de voix vous aide à communiquer la nature artificielle de la voix.*

Recommandations
- Permettre aux utilisateurs d’écouter un échantillon de chaque voix
- Utiliser une présentation authentique pour chaque voix
- Offrir des points d’entrée pour en savoir plus sur la voix de synthèse

### <a name="parental-disclosure"></a>Divulgation parentale

En plus d’une conformité à la loi COPPA, proposez la divulgation aux parents si votre public concerné principal correspond aux jeunes enfants et que votre niveau d’exposition est élevé. Pour les usages sensibles, envisagez de contingenter l’expérience tant qu’un adulte n’a pas reconnu l’utilisation de la voix de synthèse. Encouragez les parents à communiquer le message à leurs enfants.

![Divulgation pour les parents](media/responsible-ai/disclosure-patterns/parental-disclosure.png)<br/>
*Une présentation transparente optimisée pour les parents permet de garantir qu’un adulte a pris conscience de la nature synthétique de la voix avant qu’un enfant n’interagisse avec elle.*

Recommandations

- Cibler les parents en tant que public principal pour la divulgation
- Encourager les parents à communiquer la divulgation à leurs enfants
- Offrir des points d’entrée pour en savoir plus sur la voix de synthèse
- Contingenter l’expérience en posant aux parents une simple question de &quot;sécurité&quot; pour montrer qu’ils ont lu la divulgation

### <a name="providing-opportunities-to-learn-more-about-how-the-voice-was-made"></a>Offrir des possibilités d’en savoir plus sur la façon dont la voix a été fabriquée

Offrez des points d’entrée contextuels vers une page, une fenêtre contextuelle ou un site externe qui fournit des informations supplémentaires sur la technologie de la voix de synthèse. Par exemple, vous pouvez faire apparaître un lien qui permet d’en savoir plus pendant l’intégration ou quand l’utilisateur demande plus d’informations au cours d’une conversation.

![Point d’entrée pour en savoir plus](media/responsible-ai/disclosure-patterns/learn-more-entry-point.png)<br/>
*Exemple de point d’entrée pour offrir la possibilité d’en savoir plus sur la voix synthétisée.*

Une fois qu’un utilisateur a demandé plus d’informations sur la voix de synthèse, l’objectif principal vise à l’informer sur les origines de cette voix et d’être transparent sur la technologie utilisée.

![Fournir aux utilisateurs plus d’informations sur la voix de synthèse](media/responsible-ai/disclosure-patterns/learn-more.png)<br/>
*Des informations supplémentaires peuvent être proposées sur un site d’aide externe.*

Recommandations

- Simplifier les concepts complexes et éviter d’utiliser des jargons techniques et juridiques
- Ne pas dissimuler ce contenu dans des déclarations de confidentialité et des conditions d’utilisation
- Favoriser la concision du contenu et utiliser des illustrations imagées dès que possible

## <a name="implicit-disclosure"></a>Divulgation implicite

La cohérence est la clé d’une divulgation implicite tout au long du parcours utilisateur. Une utilisation cohérente de signaux visuels et auditifs sur tous les appareils et dans tous les modes d’interaction peut faciliter l’élaboration d’associations entre les modèles implicites et la divulgation explicite.

![Cohérence des signaux implicites](media/responsible-ai/disclosure-patterns/consistency.png)

### <a name="implicit-cues--feedback"></a>Signaux implicites et commentaires

L’anthropomorphisme peut se manifester de différentes façons, de la représentation visuelle réelle de l’agent à la voix, aux sons, aux voyants lumineux, aux mouvements, voire même aux vibrations d’un appareil. Quand vous définissez votre personnage, exploitez des signaux implicites et des modèles de commentaires, au lieu de viser un avatar à l’apparence très humaine. Ainsi, vous réduisez la nécessité d’une divulgation plus explicite.

![Signaux visuels et commentaires](media/responsible-ai/disclosure-patterns/visual-affordances.png)<br/>
*Ces signaux permettent de donner à l’agent un aspect anthropomorphe mais sans exagération. Ils peuvent également devenir des mécanismes de divulgation à part entière s’ils sont utilisés de manière cohérente dans le temps.*

Tenez compte des différents modes d’interaction de votre expérience pour incorporer les types de signaux suivants :

| Signaux visuels                                                                                                                                                               | Signaux auditifs                                                      | Signaux tactiles |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------|-------------|
|  Avatar <br>Signaux en temps réel réactifs (par exemple, des animations)<br> Signaux hors écran (par exemple, voyants lumineux et motifs sur un appareil)<br>  | Sonicon (par exemple, un son bref distinctif, une série de notes de musique) | Vibration   |

### <a name="capability-disclosure"></a>Divulgation des fonctions

La divulgation peut être réalisée implicitement en définissant des attentes précises quant aux capacités de l’assistant numérique. Fournissez des exemples de commandes afin que les utilisateurs puissent apprendre à interagir avec l’assistant numérique et proposez une aide contextuelle pour en savoir plus sur la voix de synthèse dès les premières phases de l’expérience.

![Exemple de réponses par défaut à une conversation que vous pouvez rédiger.](media/responsible-ai/disclosure-patterns/capability-disclosure.png)<br/>

### <a name="conversational-transparency"></a>Transparence conversationnelle

Quand les conversations glissent sur des sujets inattendus, envisagez de créer des réponses par défaut permettant de redéfinir les attentes, de renforcer la transparence et de réorienter les utilisateurs. Il existe aussi des possibilités d’utiliser la divulgation explicite dans une conversation.

![Gestion des sujets inattendus](media/responsible-ai/disclosure-patterns/conversational-transparency-1.png)<br/>

<br/>
Les questions d’ordre &quot;personnel&quot; ou inappropriées posées à l’agent sont une bonne occasion de rappeler aux utilisateurs la nature synthétique de l’agent et de les inviter à échanger des propos appropriés avec lui ou à adresser ces questions à une personne réelle.

![Gestion des questions inappropriées](media/responsible-ai/disclosure-patterns/conversational-transparency-2.png)<br/>

## <a name="when-to-disclose"></a>Quand divulguer

Il existe de nombreuses opportunités de divulgation tout au long du parcours utilisateur. Concevez-les pour la première utilisation, la deuxième utilisation, la nième utilisation, etc., mais profitez également des moments de &quot;défaillance&quot; pour souligner la transparence (comme lorsque le système fait une erreur ou lorsque l’utilisateur découvre une limite des capacités de l’agent).

![Opportunités de divulgation tout au long d’un parcours utilisateur](media/responsible-ai/disclosure-patterns/touchpoints.png)<br/>

Exemple de parcours utilisateur d’un assistant numérique standard mettant en évidence diverses opportunités de divulgation.

### <a name="up-front"></a>En amont

Le moment idéal pour la divulgation est la première fois qu’une personne interagit avec la voix de syntèse.  Dans un scénario d’assistant vocal personnel, ce moment a lieu pendant l’intégration ou la première fois que l’utilisateur découvre l’expérience. Dans d’autres scénarios, il peut s’agir de la première fois qu’une voix de synthèse lit du contenu sur un site web ou de la première fois qu’un utilisateur interagit avec un personnage virtuel.

- [Présentation transparente](#transparent-introduction)
- [Divulgation des fonctions](#capability-disclosure)
- [Personnalisation et réglage](#customization-and-calibration)
- [Signaux implicites](#implicit-cues--feedback)

### <a name="upon-request"></a>Sur demande

Les utilisateurs doivent être en mesure d’accéder facilement à des informations supplémentaires, de contrôler leurs préférences et de recevoir des communications transparentes à tout moment pendant le parcours utilisateur, sur leur demande.

- [Offrir des possibilités d’en savoir plus sur la façon dont la voix a été fabriquée](#providing-opportunities-to-learn-more-about-how-the-voice-was-made)
- [Personnalisation et réglage](#customization-and-calibration)
- [Transparence conversationnelle](#conversational-transparency)

### <a name="continuously"></a>En continu

Utilisez les modèles de conception implicites qui améliorent l’expérience utilisateur en continu.

- [Divulgation des fonctions](#capability-disclosure)
- [Signaux implicites](#implicit-cues--feedback)

### <a name="when-the-system-fails"></a>En cas de défaillance du système

Utilisez la divulgation comme une opportunité d’échec élégant.

- [Transparence conversationnelle](#conversational-transparency)
- [Offrir des possibilités d’en savoir plus sur la façon dont la voix a été fabriquée](#providing-opportunities-to-learn-more-about-how-the-voice-was-made)
- [Procédure de transfert aux humains](#conversational-transparency)



## <a name="additional-resources"></a>Ressources supplémentaires
- [Instructions pour Microsoft Bot](https://www.microsoft.com/research/uploads/prod/2018/11/Bot_Guidelines_Nov_2018.pdf)
- [Instructions pour la conception de Cortana](/cortana/voice-commands/voicecommand-design-guidelines)
- [Instructions pour la conception de Microsoft Windows UWP Speech](/windows/uwp/design/input/speech-interactions)
- [Instructions pour les commandes vocales de Microsoft Windows Mixed Reality](/windows/mixed-reality/voice-design#top-things-users-should-know-about-speech-in-mixed-reality)

## <a name="reference-docs"></a>Documents de référence

* [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent)
* [Instructions pour le déploiement responsable de la technologie des voix de synthèse](concepts-guidelines-responsible-deployment-synthetic.md)
* [Vue d’ensemble du contingentement](concepts-gating-overview.md)
* [Comment divulguer](concepts-disclosure-guidelines.md)

## <a name="next-steps"></a>Étapes suivantes

* [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent)
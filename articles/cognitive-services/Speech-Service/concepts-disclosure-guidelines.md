---
title: Guide de conception de la divulgation
titleSuffix: Azure Cognitive Services
description: Introduction au guide de conception de la divulgation et évaluation du niveau de divulgation.
services: cognitive-services
author: sharonlo101
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 12/03/2019
ms.author: shlo
ms.openlocfilehash: 2276e66fa47b1c125a8c30f3c73a7c948f65a35d
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021930"
---
# <a name="disclosure-design-guidelines"></a>Guide de conception de la divulgation
Apprenez à nouer et à entretenir une relation de confiance avec vos clients en étant transparent sur la nature synthétique de votre expérience vocale.

## <a name="what-is-disclosure"></a>Qu’est-ce que la divulgation ?

La divulgation est un moyen d’informer les utilisateurs qu’ils interagissent ou écoutent une voix fabriquée par synthèse.

## <a name="why-is-disclosure-necessary"></a>Pourquoi la divulgation est-elle nécessaire ?

La nécessité de divulguer les origines synthétiques d’une voix générée par ordinateur est relativement nouvelle. Par le passé, les voix générées par ordinateur l’étaient de toute évidence : personne n’aurait pu les confondre avec celles de personnes réelles. De nos jours, en revanche, le réalisme des voix de synthèse ne cesse de s’améliorer et il devient de plus en plus difficile de faire la différence entre voix humaines et voix de synthèse.

## <a name="goals"></a>Objectifs
Voici les principes à ne pas oublier quand vous concevez des expériences de voix de synthèse :

**Renforcer la confiance**
<br>Concevez dans l’intention de ne pas réussir le Turing Test sans dégrader l’expérience. Préparez les utilisateurs au fait qu’ils interagissent avec une voix de synthèse tout en leur permettant de s’engager de façon continue dans l’expérience.

**Adapter au contexte d’utilisation**
<br>Sachez quand, où et comment vos utilisateurs vont interagir avec la voix de synthèse pour offrir le type de divulgation approprié au bon moment.

**Définir des attentes claires**
<br>Permettez aux utilisateurs de découvrir et comprendre facilement les capacités de l’agent. Offrez des opportunités d’en savoir plus sur la technologie de la voix de synthèse sur simple demande.

**Exploiter les défaillances**
<br>Utilisez les moments de défaillance pour renforcer les capacités de l’agent.

## <a name="how-to-use-this-guide"></a>Comment utiliser ce guide

Ce guide vous aide à déterminer quels modèles de divulgation sont les plus adaptés à votre expérience de voix de synthèse. Nous proposons ensuite des exemples montrant comment et quand les utiliser. Chacun de ces modèles est conçu pour maximiser la transparence auprès des utilisateurs sur la synthèse vocale tout en restant fidèle à une conception axée sur l’humain.

Étant donné le large éventail des directives pour la conception des expériences vocales, nous nous concentrons ici particulièrement sur les points suivants :

1. [**Évaluation de la divulgation**](#disclosure-assessment) : Processus permettant de déterminer le type de divulgation recommandé pour votre expérience de voix de synthèse

2. [**Comment révéler**](concepts-disclosure-patterns.md) : Exemples de modèles de divulgation applicables à votre expérience de voix de synthèse

3. [**Quand divulguer**](concepts-disclosure-patterns.md#when-to-disclose) : Meilleurs moments pour la divulgation tout au long du parcours utilisateur

## <a name="disclosure-assessment"></a>Évaluation de la divulgation
Tenez compte des attentes de vos utilisateurs liées à l’interaction et au contexte dans lequel ils vont être confrontés à la voix. Si le contexte indique clairement qu’une voix de synthèse est &quot;en train de parler&quot;, la divulgation peut être minime, momentanée voire même inutile. Les principaux types de contexte qui affectent la divulgation incluent le type de personnage, le type de scénario et le niveau d’exposition. Tenir compte des types de personnes qui écoutent s’avère également utile.

### <a name="understand-context"></a>Comprendre le contexte

Utilisez cette feuille pour déterminer le contexte de votre expérience de voix de synthèse. Vous allez l’appliquer à l’étape suivante pour déterminer votre niveau de divulgation.

|                                    | Contexte d’utilisation                                                                                                                                                                                                                                                                                                                                                       | Risques potentiels et difficultés                                                                                                                                                                                                                                                                                                                                                                       |
|------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **1. Type de personnage**               | **Si l’un des critères suivants est rempli, votre personnage relève de la catégorie « Personnage quasi humain » :**<br><br><ul><li> Le personnage incarne un vrai humain, qu’il s’agisse d’une représentation fictive ou non (par exemple, photographie ou rendu généré par ordinateur d’une personne réelle).<br><br><li> La voix de synthèse se base sur la voix d’une personne réelle largement reconnaissable (par exemple, une célébrité, une personnalité politique). | Plus vous donnez des représentations quasi humaines à votre personnage, plus l’utilisateur est enclin à l’associer à une personne réelle ou à croire que le contenu est énoncé par une personne réelle plutôt que par un ordinateur. </ul>                                                                                                                                                                      |
| **2. Type de scénario**            | **Si l’un des critères suivants est rempli, votre expérience vocale relève de la catégorie « Sensible » :**<br><br><ul><li> Obtient ou affiche des informations personnelles de l’utilisateur <br><br> <li> Diffuse des informations urgentes/actualités brûlantes (par exemple, un état d’alerte)<br><br><li> A pour but d’aider les utilisateurs à communiquer entre eux (par exemple, lire des e-mails/textes personnels)<br><br> <li> Fournit une assistance médicale </ul>            | L’emploi d’une voix de synthèse peut ne pas sembler approprié ou ne pas inspirer confiance aux utilisateurs quand les sujets sont liés à des questions sensibles, personnelles ou urgentes. Les utilisateurs peuvent aussi attendre le même niveau d’empathie et d’écoute que celles de véritables personnes humaines. |
| **3. Niveau d’exposition** |**Votre expérience vocale relève très probablement de la catégorie « Niveau d’exposition élevé » dans les cas suivants :** <br><br><ul><li>L’utilisateur va entendre la voix de synthèse ou interagir avec elle souvent ou pendant une longue période. </ul>                                                                                                                                                                             | L’importance de la transparence et de la création d’une relation de confiance avec les utilisateurs est encore plus élevée quand il s’agit d’établir des relations durables.                                                                                                                                                                                                                                                                      |

### <a name="determine-disclosure-level"></a>Déterminer le niveau de divulgation

Utilisez le diagramme suivant pour déterminer si votre expérience de voix de synthèse nécessite une divulgation élevée ou faible en fonction de votre contexte d’utilisation.

  ![Diagramme d’évaluation de la divulgation](media/responsible-ai/disclosure-guidelines/flowchart.png)

## <a name="reference-docs"></a>Documents de référence

* [Divulgation d’un talent vocal](/legal/cognitive-services/speech-service/disclosure-voice-talent)
* [Instructions pour le déploiement responsable de la technologie des voix de synthèse](concepts-guidelines-responsible-deployment-synthetic.md)
* [Vue d’ensemble du contingentement](concepts-gating-overview.md)

## <a name="next-steps"></a>Étapes suivantes

* [Modèles de conception de divulgation](concepts-disclosure-patterns.md)
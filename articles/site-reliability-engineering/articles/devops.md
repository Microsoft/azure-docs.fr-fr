---
title: 'Questions fréquentes : SRE et DevOps | Microsoft Docs'
titleSuffix: Azure
description: 'Questions fréquentes : Comprendre la relation entre SRE et DevOps'
author: dnblankedelman
manager: ScottCa
ms.service: site-reliability-engineering
ms.topic: article
ms.date: 09/14/2020
ms.author: dnb
ms.openlocfilehash: 62872443c386c45d9821053dc901ad8e0055b74f
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674266"
---
# <a name="frequently-asked-questions-whats-the-relationship-between-sre-and-devops"></a>Forum Aux Questions : Quelle est la relation entre SRE et DevOps ?

Plusieurs questions courantes s’interrogent sur la nature de la relation entre l’ingénierie de fiabilité de site (SRE) et DevOps, notamment « En quoi sont-ils identiques ? En quoi sont-ils différents ? Pouvons-nous avoir les deux dans notre organisation ? ». Cet article tente de partager certaines des réponses proposées par les communautés SRE et DevOps qui nous aident à comprendre cette relation.

## <a name="how-are-they-the-same"></a>En quoi sont-ils identiques  ?

SRE et DevOps sont des pratiques opérationnelles modernes qui ont été créées et développées en réponse aux défis suivants, entre autres :

- la complexité croissante de nos environnements de production et processus de développement
- l’importance toujours plus grande du fonctionnement continu de ces environnements pour l’entreprise
- l’impossibilité d’adapter la force de travail de façon linéaire par rapport à la taille de ces environnements
- le besoin d’agir plus rapidement tout en conservant la stabilité opérationnelle

Ces deux pratiques opérationnelles attirent l’attention sur des sujets essentiels pour gérer ces défis, par exemple, la supervision/l’observabilité, l’automatisation, la documentation et les outils de développement de logiciels collaboratifs.

SRE et DevOps partagent une grande partie des outils et des domaines de travail. Comme indiqué dans l’ouvrage _The Site Reliability Workbook_ , « SRE croit aux mêmes choses que DevOps, mais pour des raisons légèrement différentes ».

## <a name="three-different-ways-to-compare-the-two-operations-practices"></a>Trois façons différentes de comparer les deux pratiques opérationnelles

Les similarités entre SRE et DevOps sont évidentes. Ce qui est vraiment intéressant, c’est la façon dont elles diffèrent ou divergent. Ici, nous proposons trois axes de réflexion sur la nature de leur relation. Vous pouvez ne pas être d’accord avec ces réponses, mais chacune est un bon point de départ pour la discussion.

### <a name="class-sre-implements-interface-devops"></a>« la classe SRE implémente l’interface DevOps »

L’ouvrage _The Site Reliability Workbook_ (mentionné dans notre [liste d’ouvrages](../resources/books.md)) aborde SRE et DevOps dans son premier chapitre. Le sous-titre de ce chapitre est « la classe SRE implémente l’interface DevOps ». Cette phrase suggère (sous la forme une expression destinée aux développeurs) que SRE peut être considéré comme une implémentation spécifique de la philosophie DevOps. Comme décrit dans le chapitre, « DevOps est relativement silencieux quant à la façon dont les opérations sont exécutées dans le détail », tandis que SRE est considérablement plus normatif dans ses pratiques. Par conséquent, pour tenter de répondre à la question sur la nature de la relation entre les deux, on peut dire que SRE est l’une des nombreuses implémentations possibles de DevOps.

### <a name="sre-is-to-reliability-as-devops-is-to-delivery"></a>SRE est à la fiabilité ce que DevOps est à la distribution

Cette comparaison est un peu floue, car il existe plusieurs définitions pour SRE et DevOps, mais elle est toujours potentiellement utile. Elle commence par la question « si vous deviez résumer chaque pratique opérationnelle en un ou deux mots reflétant sa principale préoccupation, quels seraient-ils ? »

D’après la définition de SRE du [hub d’ingénierie de fiabilité de site](../index.yml) :

> L’ingénierie de fiabilité de site (SRE) est une discipline d’ingénierie qui aide les organisations à atteindre durablement le niveau de fiabilité approprié dans leurs systèmes, services et produits.

Le mot résumant SRE est donc « fiabilité ». Il s’agit aussi du deuxième mot du nom, ce choix paraît donc évident.

D’après la définition de DevOps du [Centre de ressources Azure DevOps](/azure/devops/learn/) :

> DevOps rassemble des personnes, des processus et des produits pour permettre la distribution continue de valeur ajoutée aux clients finaux.

Le mot qui résume DevOps est donc « distribution ».

Par conséquent, « SRE est à la fiabilité ce que DevOps est à la distribution ».

### <a name="direction-of-attention"></a>Sens de l’attention

Cette réponse est tirée ou légèrement paraphrasée d’une contribution de Thomas Limoncelli dans l’ouvrage _Seeking SRE_ , mentionné dans notre [liste d’ouvrages](../resources/books.md). Ce dernier note que les ingénieurs DevOps se focalisent en grande partie sur le pipeline du cycle de vie du développement de logiciels avec des responsabilités occasionnelles d’opérations de production, tandis que les ingénieurs SRE se focalisent sur les opérations de production avec des responsabilités occasionnelles de pipeline.

Plus important encore, il dessine un diagramme qui commence par le processus de développement logiciel d’un côté et le travail des opérations de production de l’autre. Les deux sont connectés par le pipeline habituel, qui consiste à prendre le code d’un développeur, le conduire à travers le nombre de tests et de phases souhaités, puis le passer en production.

Limoncelli note que les ingénieurs DevOps démarrent dans l’environnement de développement et automatisent les étapes vers la production. Une fois terminé, ils reviennent en arrière pour optimiser les goulots d’étranglement.

Les ingénieurs SRE, quant à eux, se concentrent sur les opérations de production et étudient en détail le pipeline pour pouvoir améliorer le résultat final (ils travaillent dans le sens inverse, en somme).

C’est donc parce qu’ils travaillent en sens inverse que les ingénieurs DevOps et SRE se distinguent.

## <a name="coexistence-in-the-same-organization"></a>Coexistence au sein de la même organisation

La dernière question à laquelle nous voulons répondre est « Est-il possible d’avoir à la fois des ingénieurs SRE et DevOps dans la même organisation ? »

La réponse à cette question est un « Oui ! » catégorique.

Nous espérons que les réponses précédentes vous permettent de comprendre dans quelle mesure ces deux pratiques opérationnelles sont similaires et, quand ce n’est pas le cas, comment elles peuvent être complémentaires. Les organisations qui ont une pratique DevOps établie peuvent expérimenter les pratiques SRE à petite échelle (par exemple, en essayant des opérations SLI et SLO) sans avoir à créer de postes ou d’équipes d’ingénierie SRE. C’est un modèle d’adoption de l’ingénierie SRE assez courant.

## <a name="next-steps"></a>Étapes suivantes

Vous souhaitez en savoir plus sur l’ingénierie de fiabilité de site ou sur DevOps ? Consultez notre [hub d’ingénierie de fiabilité de site](../index.yml) et le [centre de ressources Azure DevOps](/azure/devops/learn/).
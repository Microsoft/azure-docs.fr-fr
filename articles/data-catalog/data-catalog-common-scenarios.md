---
title: Scénarios courants avec Azure Data Catalog
description: Vue d’ensemble des scénarios courants pour Azure Data Catalog, notamment l’inscription et la découverte des sources de données de grande valeur, l’activation du décisionnel libre-service et la capture de connaissances existantes sur les sources de données et les processus.
author: JasonWHowell
ms.author: jasonh
ms.service: data-catalog
ms.topic: conceptual
ms.date: 08/01/2019
ms.openlocfilehash: aeae505b510f563a6640726c384ea358983eb24f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "68736455"
---
# <a name="azure-data-catalog-common-scenarios"></a>Scénarios courants avec Azure Data Catalog
Cet article présente des scénarios courants dans lesquels Azure Data Catalog peut aider votre organisation à mieux exploiter ses sources de données existantes.

## <a name="scenario-1-registration-of-central-data-sources"></a>Scénario 1 : Enregistrement des sources de données centrales
Les organisations ont souvent de nombreuses sources de données de grande valeur. Celles-ci incluent les systèmes d’entreprise OLTP, les entrepôts de données et les bases de données de décisionnel/analyse. Le nombre de systèmes augmente et le chevauchement entre les systèmes s’accentue au fur et à mesure de l’évolution des besoins de l’entreprise ou de l’évolution de l’entreprise elle-même, par exemple par le biais de fusions et d’acquisitions.

Il peut être difficile pour les membres de l’organisation de savoir où rechercher les données parmi ces sources de données. Des questions telles que celles-ci se posent régulièrement :

* Parmi les trois systèmes de RH utilisés par l’entreprise, lequel dois-je utiliser pour créer ce type de rapport ?
* Où puis-je obtenir les chiffres des ventes certifiés pour l’exercice qui vient de se clôturer ?
* À qui dois-je m’adresser ou quel processus dois-je suivre pour accéder à l’entrepôt de données ?
* Je ne sais pas si ces chiffres sont corrects. Qui pourrait m’expliquer comment ces données sont censées être utilisées avant de partager ce tableau de bord avec mon équipe ?

Azure Data Catalog peut fournir des réponses à ces questions et à de nombreuses autres. Les sources de données centrales, de grande valeur, gérées par informatique et utilisées à l’échelle de l’organisation sont souvent le point de départ logique pour remplir le catalogue. Bien que n’importe quel utilisateur puisse inscrire une source de données, il sera plus facile d’encourager l’adoption et l’utilisation du système si vous commencez par remplir le catalogue avec les sources de données qui sont susceptibles d’être utiles au plus grand nombre d’utilisateurs. 

Si vous n’avez jamais utilisé Azure Data Catalog auparavant, l’identification et l’enregistrement des principales sources de données utilisées par de nombreuses équipes consommatrices de données peuvent représenter votre premier pas vers la réussite.

Ce scénario permet également d’annoter les sources de données à valeur élevée pour les rendre plus faciles à comprendre et plus faciles d’accès. Un aspect clé de cet effort consiste à inclure des informations sur la façon dont les utilisateurs peuvent demander l'accès à la source de données. Avec Azure Data Catalog, vous pouvez fournir l’adresse e-mail de l’équipe ou de l’utilisateur chargé de contrôler l’accès aux sources de données, les liens vers des outils existants ou une documentation, ou du texte libre qui décrit le processus de demande d’accès. Ces informations aident les membres qui découvrent des sources de données enregistrées, mais qui n’ont pas encore les autorisations pour accéder aux données, à demander facilement l’accès à l’aide de processus définis et contrôlés par les propriétaires des sources de données.

## <a name="scenario-2-self-service-business-intelligence"></a>Scénario 2 : Décisionnel libre-service
Bien que les solutions traditionnelles d’analyse décisionnelle des entreprises continuent à être un élément important du paysage de données de nombreuses organisations, le décisionnel libre-service gagne chaque jour en importance du fait de l’intensification du rythme des opérations. Le décisionnel libre-service permet aux professionnels de l’information et aux analystes de créer leurs propres rapports, classeurs et tableaux de bord sans dépendre d’une équipe informatique centrale, et sans être limités par l’emploi du temps ou la disponibilité de l’équipe informatique.

Dans les scénarios de décisionnel libre-service, les utilisateurs combinent fréquemment des données provenant de plusieurs sources, dont un grand nombre n’ont peut-être jamais été utilisées auparavant pour le décisionnel et l’analyse. Bien qu’il arrive que certaines de ces sources soient déjà connues, il peut être difficile de savoir comment procéder pour localiser et évaluer les sources potentielles de données pour une tâche donnée.

Généralement, ce processus est exécuté manuellement : les analystes utilisent leurs connexions réseau homologues pour identifier d’autres personnes travaillant avec les données recherchées. Une fois qu’une source de données a été trouvée et utilisée, le processus se répète pour chaque initiative ultérieure de décisionnel libre-service, avec plusieurs utilisateurs effectuant un processus manuel de découverte redondant.

Dans Azure Data Catalog, votre organisation peut rompre ce cycle d’efforts. Après avoir découvert une source de données selon un processus traditionnel, un analyste peut l’enregistrer pour la rendre plus facilement identifiable ultérieurement par d’autres utilisateurs. L’analyste peut annoter les ressources de données enregistrées pour leur conférer davantage de valeur ; il n’est pas obligatoire que l’annotation soit effectuée au moment de l’enregistrement. Les utilisateurs peuvent apporter leur contribution au fil du temps, en fonction de leur emploi du temps, renforçant ainsi progressivement la valeur des sources de données enregistrées dans le catalogue.

Cette croissance organique du contenu du catalogue est un complément naturel de l’enregistrement initial des sources de données centrales. Le préremplissage du catalogue avec des données utiles à un grand nombre d’utilisateurs pourra initialement inciter ces derniers à découvrir et utiliser le catalogue. De plus, si les utilisateurs sont autorisés à enregistrer et à annoter des sources supplémentaires, cela contribue à consolider leur engagement et celui d’autres membres de l’organisation.

Il est important de noter que, bien que ce scénario se concentre spécifiquement sur le décisionnel libre-service, les mêmes schémas et les mêmes défis s’appliquent également aux projets décisionnels d’entreprise de grande échelle. Grâce à Data Catalog, votre organisation peut améliorer les efforts qui impliquent un processus manuel de découverte de source de données.

## <a name="scenario-3-capturing-tribal-knowledge"></a>Scénario 3 : Capture des connaissances tribales
Comment savoir quelles données sont nécessaires pour effectuer votre travail et où les trouver ?

Si vous occupez le même poste depuis un certain temps, vous connaissez probablement la réponse. Vous avez suivi un processus d’apprentissage progressif, et au fil du temps vous avez appris quelles sources de données sont essentielles à la réalisation de vos tâches quotidiennes.

Quand un nouvel employé intègre votre équipe, comment sait-il de quelles données il aura besoin pour effectuer son travail et où les trouver ?

Il est fort probable que ce nouvel employé se tournera vers vous pour obtenir des réponses à ces questions.

Ce transfert continu de connaissances tribales fait partie du processus de découverte des sources de données dans toutes les entreprises, quelle que soit leur taille. Les membres les plus anciens et les plus expérimentés de l’équipe ont accumulé des connaissances au fil des années, tandis que les membres ayant rejoint l’équipe plus récemment ont appris à s’adresser à eux lorsqu’ils ont des questions. Souvent, seules quelques personnes connaissent les informations essentielles, et lorsque ces personnes sont en vacances ou quittent l’équipe, l’organisation en pâtit.

Les experts en données font normalement l’effort de documenter leurs connaissances en les partageant par e-mail ou par le biais de documents Word sur un site d’équipe SharePoint. Bien que cette approche puisse se révéler utile, elle introduit également un nouveau problème de découverte : comment savoir quelle documentation existe et où la trouver...

Avec Azure Data Catalog, votre organisation dispose d’un emplacement unique et centralisé pour le stockage et le partage de ces connaissances tribales, et pour leur mise à disposition sans difficulté. Dans Data Catalog, vos experts en données peuvent annoter directement les ressources de données et fournir des liens vers la documentation existante. Quand les membres de l’organisation utilisent le catalogue pour découvrir une source de données, ils trouvent non seulement la source proprement dite, mais également la base de connaissances qui existait précédemment uniquement dans l’esprit des experts de votre organisation.

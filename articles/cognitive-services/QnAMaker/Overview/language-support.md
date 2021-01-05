---
title: Prise en charge linguistique - QnA Maker
titleSuffix: Azure Cognitive Services
description: Liste de cultures et de langues naturelles prises en charge par QnA Maker pour votre base de connaissances. Ne mélangez pas différentes langues dans une même base de connaissances.
services: cognitive-services
manager: nitinme
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 11/09/2019
ms.openlocfilehash: 1edd5ffc2578a27a53c7e9a46a4a5f1cf61331ff
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605057"
---
# <a name="language-support-for-a-qna-maker-resource-and-knowledge-bases"></a>Prise en charge linguistique pour une ressource QnA Maker et bases de connaissances

Cet article décrit les options de prise en charge linguistique pour les ressources QnA Maker et les bases de connaissances. 

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

La langue du service est sélectionnée lorsque vous créez la première base de connaissances dans la ressource. Toutes les bases de connaissances supplémentaires de la ressource doivent être dans la même langue. 

La langue détermine la pertinence des résultats que QnA Maker fournit en réponse aux requêtes de l’utilisateur. La ressource QnA Maker et toutes les bases de connaissances à l’intérieur de celle-ci prennent en charge une seule langue. L’unilinguisme est nécessaire pour produire des résultats de réponse optimaux à une requête.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

Dans QnA Maker managé, vous pouvez choisir de créer des paramètres de langue au niveau de la base de connaissances individuelle. Ce paramètre peut être activé uniquement avec la base de connaissances du service. Une fois vous avez défini les paramètres de langue, vous ne pouvez pas les changer pour le service. 

Si vous choisissez des paramètres de langue spécifiques selon la base de connaissances, vous êtes autorisé à créer des bases de connaissances de différentes langues dans le service lui-même. 

---

## <a name="single-language-per-resource"></a>Une seule langue par ressource

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

Tenez compte des éléments suivants :

* Un service QnA Maker et toutes ses bases de connaissances ne prennent en charge qu’une seule langue.
* La langue est définie explicitement lors de la création de la première base de connaissances du service.
* La langue est déterminée à partir des fichiers et URL ajoutés lors de la création de la base de connaissances.
* Il n’est pas possible de changer la langue pour d’autres bases de connaissances du service.
* Les services Recherche cognitive (ranker n° 1) et QnA Maker (ranker n° 2) utilisent la langue pour générer la meilleure réponse à une requête.

# <a name="qnamaker-managed-preview"></a>[QnA Maker managé (préversion)](#tab/v2)
![Paramètre de langue dans QnA Maker managé](../media/language-support/language-setting-managed.png)

Si vous **ne cochez pas la case pour activer le paramètre de langue par base de connaissances**, prenez en compte les points suivants : 
* Un service QnA Maker et toutes ses bases de connaissances ne prennent en charge qu’une seule langue.
* La langue est définie explicitement lors de la création de la première base de connaissances du service.
* La langue est déterminée à partir des fichiers et URL ajoutés lors de la création de la base de connaissances.
* Il n’est pas possible de modifier la langue pour d’autres bases de connaissances du service.
* Les services Recherche cognitive (ranker #1) et QnA Maker (ranker #2) utilisent la langue pour générer la meilleure réponse à une requête.

---

## <a name="supporting-multiple-languages-in-one-qna-maker-resource"></a>Prise en charge de plusieurs langues dans une seule ressource QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)
Cette fonctionnalité n’est pas prise en charge dans notre version stable en disponibilité générale (GA). Consultez QnA Maker managé pour tester cette fonctionnalité. 

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
* Quand vous créez la première base de connaissances dans votre service, vous pouvez choisir d’activer le paramètre de langue par base de connaissances. Cochez la case pour créer des bases de connaissances appartenant à des langues différentes au sein d’un même service.
* L’option du paramètre de langue n’est pas modifiable pour le service, une fois la première base de connaissances créée.
* Si vous activez des paramètres de langue propres à chaque base de connaissances, au lieu d’avoir un seul index de test pour le service, vous aurez un index de test par base de connaissances. 

![Paramètre de langue dans QnA Maker managé](../media/language-support/language-setting-managed.png)

---

## <a name="supporting-multiple-languages-in-one-knowledge-base"></a>Prise en charge de plusieurs langues dans une seule base de connaissances

Pour prendre en charge un système de base de connaissances comprenant plusieurs langues, vous pouvez :

* Utiliser le [service Translator](../../translator/translator-info-overview.md) pour traduire une question dans une langue individuelle avant de l’envoyer à votre base de connaissances. Cela vous permet de vous concentrer sur la qualité d’une seule langue et la qualité des autres questions et réponses.
* Créez une ressource QnA Maker, ainsi qu’une base de connaissances à l’intérieur de cette ressource, pour chaque langue. Cela vous permet de gérer d’autres questions et du texte de réponse plus nuancé pour chaque langue. Vous bénéficiez ainsi d’une flexibilité sensiblement accrue, mais devez exposer des frais de maintenance bien plus élevés lorsque les questions ou réponses changent dans toutes les langues.


## <a name="languages-supported"></a>Langues prises en charge

La liste suivante contient les langues prises en charge pour une ressource QnA Maker. 

| Langage |
|--|
| Arabe |
| Arménien |
| Bangla |
| Basque |
| Bulgare |
| Catalan |
| Chinois simplifié |
| Chinois traditionnel |
| Croate |
| Tchèque |
| Danois |
| Néerlandais |
| Anglais |
| Estonien |
| Finnois |
| Français |
| Galicien |
| Allemand |
| Grec |
| Goudjrati |
| Hébreu |
| Hindi |
| Hongrois |
| Islandais |
| Indonésien |
| Irlandais |
| Italien |
| Japonais |
| Kannada |
| Coréen |
| Letton |
| Lituanien |
| Malayalam |
| Malais |
| Norvégien |
| Polonais |
| Portugais |
| Pendjabi |
| Roumain |
| Russe |
| Serbe (cyrillique) |
| Serbe (latin) |
| Slovaque |
| Slovène |
| Espagnol |
| Suédois |
| Tamoul |
| Télougou |
| Thaï |
| Turc |
| Ukrainien |
| Ourdou |
| Vietnamien |

## <a name="query-matching-and-relevance"></a>Correspondance et pertinence des requêtes
QnA Maker dépend des [analyseurs de langue Recherche cognitive Azure](/rest/api/searchservice/language-support) pour fournir des résultats.

Même si les fonctionnalités de Recherche cognitive Azure sont à égalité pour les langues prises en charge, QnA Maker possède un outil de classement supplémentaire qui se trouve au-dessus des résultats de recherche Azure. Dans ce modèle d’outil de classement, nous utilisons une sémantique spéciale et des fonctionnalités basées sur des mots dans les langues suivantes.

|Langues avec outil de classement supplémentaire|
|--|
|Chinois|
|Tchèque|
|Néerlandais|
|Anglais|
|Français|
|Allemand|
|Hongrois|
|Italien|
|Japonais|
|Coréen|
|Polonais|
|Portugais|
|Espagnol|
|Suédois|

Ce classement supplémentaire est un travail interne de l’outil de classement de QnA Maker.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Sélection de la langue](../index.yml)
---
title: Concevoir une base de connaissances - Concepts de QnA Maker
description: Découvrez comment concevoir une base de connaissances - QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 01/27/2020
ms.openlocfilehash: c21c82b5fd024598da30f4ac7b1ed01e64561e3b
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000111"
---
# <a name="question-and-answer-pair-concepts"></a>Concepts des paires question/réponse

Une base de connaissances est composée de paires question/réponse (Q/R).  Chaque paire a une réponse et contient toutes les informations associées à cette _réponse_. Une réponse peut un peu s’apparenter à une ligne de base de données ou à une instance de structure de données.

## <a name="question-and-answer-pairs"></a>Paires de questions-réponses

Les paramètres **obligatoires** dans une paire question/réponse (Q/R) sont les suivants :

* Une **question** : texte de la requête utilisateur, utilisé pour le machine learning de QnA Maker, afin de s’aligner avec le texte de la question de l’utilisateur ayant une formulation différente mais la même réponse.
* La **réponse** : la réponse de la paire est ce qui est retourné quand une correspondance est établie entre une requête utilisateur et la question associée.

Chaque paire est représenté par un **ID**.

Les paramètres **facultatifs** pour une paire sont les suivants :

* **Autres formes de la question** : aident QnA Maker à retourner la bonne réponse pour une variété plus large de formulations de questions.
* **Métadonnées** : les métadonnées sont des étiquettes associées à une paire QnA. Elles sont représentées sous forme de paires clé-valeur. Les balises de métadonnées servent à filtrer les paires QnA et à limiter le jeu sur lequel la mise en correspondance de requête est effectuée.
* **Invites multitours** : utilisées pour continuer une conversation multitour.

![ID de base de connaissances QnA Maker](../media/qnamaker-concepts-knowledgebase/knowledgebase.png)

## <a name="editorially-add-to-knowledge-base"></a>Ajouter à la base de connaissances de manière éditoriale

En l'absence de contenu préexistant pour alimenter la base de connaissances, vous pouvez ajouter des paires QnA en tant que rédacteur sur le portail QnA Maker. Découvrez comment mettre à jour votre base de connaissances [ici](../How-To/edit-knowledge-base.md).

## <a name="editing-your-knowledge-base-locally"></a>Modification de votre base de connaissances en local

Une fois qu’une base de connaissances est créée, il est recommandé d’apporter des modifications au texte associé dans le [portail QnA Maker](https://qnamaker.ai), plutôt que d’exporter et de réimporter via des fichiers locaux. Toutefois, vous pouvez être amené à modifier une base de connaissances localement.

Exportez la base de connaissances à partir de la page **Paramètres**, puis modifiez la base de connaissances avec Microsoft Excel. Si vous choisissez d’utiliser une autre application pour modifier votre fichier exporté, celle-ci peut introduire des erreurs de syntaxe, car elle ne prend pas totalement en charge le format TSV. Les fichiers TSV de Microsoft Excel n’entraînent généralement pas d’erreurs de mise en forme.

Une fois que vous avez terminé vos modifications, réimportez le fichier TSV à partir de la page **Paramètres**. Cela remplace complètement la base de connaissances actuelle par la base de connaissances importée.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Cycle de vie d’une base de connaissances dans QnA Maker](./development-lifecycle-knowledge-base.md)
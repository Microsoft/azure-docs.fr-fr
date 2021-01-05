---
title: Types d’URL pris en charge pour l’importation – QnA Maker
description: Découvrez comment les types d’URL sont utilisés pour importer et créer des paires de Q/R.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 01/02/2020
ms.openlocfilehash: 8bf50c1ea81cdf5246c47646d1a55926fe7d58d6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91776695"
---
# <a name="urls-supported-for-importing-documents"></a>URL prises en charge pour l’importation de documents

Découvrez comment les types d’URL sont utilisés pour importer et créer des paires de Q/R.

## <a name="faq-urls"></a>URL de FAQ

QnA Maker peut prendre en charge 3 différentes formes de pages web de questions fréquentes (FAQ), à savoir :

* Pages de FAQ simples
* Pages de FAQ avec des liens
* Pages de FAQ avec une page d’accueil Rubriques

### <a name="plain-faq-pages"></a>Pages de FAQ simples

Il s’agit du type le plus courant de page de FAQ, dans lequel les réponses suivent immédiatement les questions dans la même page.

Voici un exemple d’une page de FAQ simple :

![Exemple de page de FAQ simple pour une base de connaissances](./media/qnamaker-concepts-datasources/plain-faq.png)


### <a name="faq-pages-with-links"></a>Pages de FAQ avec des liens

Dans ce type de page de FAQ, les questions sont regroupées puis associées à des réponses qui se trouvent soit dans différentes sections de la même page, soit dans différentes pages.

Voici un exemple de page de FAQ avec des liens dans des sections qui se trouvent dans la même page :

 ![Exemple de page de FAQ avec liens de sections pour une base de connaissances](./media/qnamaker-concepts-datasources/sectionlink-faq.png)


### <a name="parent-topics-page-links-to-child-answers-pages"></a>Liens de la page Rubriques parente vers les pages de réponses enfants

Ce type de FAQ comporte une page Rubriques sur laquelle chaque rubrique est liée à un ensemble de questions et de réponses correspondant sur une autre page. QnA Maker analyse toutes les pages associées pour extraire les questions et réponses correspondantes.

Voici un exemple de page Rubriques comportant des liens vers des sections de FAQ sur d’autres pages.

 ![Exemple de page de FAQ avec liens profonds pour une base de connaissances](./media/qnamaker-concepts-datasources/topics-faq.png)

## <a name="support-urls"></a>URL de support technique

QnA Maker peut traiter des pages web de support semi-structurées, telles que des articles web décrivant comment effectuer une tâche donnée, comment diagnostiquer et résoudre un problème donné, ainsi quelles sont les meilleures pratiques pour un processus donné. L’extraction fonctionne de façon optimale sur du contenu ayant une structure claire avec des titres hiérarchiques.

> [!NOTE]
> L’extraction pour les articles de support est une nouvelle fonctionnalité à un stade de développement encore précoce. Il convient idéalement pour les pages simples, qui sont bien structurées et qui ne contiennent pas d’en-têtes ou pieds de page complexes.

![QnA Maker prend en charge l’extraction à partir de pages web semi-structurées où une structure claire est présentée avec des titres hiérarchiques](./media/qnamaker-concepts-datasources/support-web-pages-with-heirarchical-structure.png)
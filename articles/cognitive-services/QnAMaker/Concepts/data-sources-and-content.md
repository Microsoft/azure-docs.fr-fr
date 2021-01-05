---
title: Sources de données et types de contenu – QnA Maker
description: Découvrez comment importer des paires de questions et de réponses à partir de sources de données et de types de contenu pris en charge, qui incluent de nombreux documents structurés standard tels que les fichiers PDF, DOCX et TXT, dans QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 10/13/2020
ms.openlocfilehash: a6488d2c7611513528ff02d4e620124763912730
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353253"
---
# <a name="importing-from-data-sources"></a>Importation depuis des sources de données

Une base de connaissances est constituée de paires question/réponse apportées par des URL et des fichiers publics.

## <a name="data-source-locations"></a>Emplacements des sources de données

Le contenu est placé dans une base de connaissances à partir d’une source de données. Les emplacements des sources de données sont des **URL ou fichiers publics**, qui ne requièrent pas d’authentification.

Les [fichiers SharePoint](../how-to/add-sharepoint-datasources.md), sécurisés par une authentification, sont l’exception. Les ressources SharePoint doivent être des fichiers, et non des pages web. Si l’URL se termine par une extension web, telle que .ASPX, elle n’est pas importée dans QnA Maker à partir de SharePoint.

## <a name="chit-chat-content"></a>Contenu des bavardages

L’ensemble de contenu des bavardages est proposé sous la forme d’une source complète de données de contenu dans plusieurs langues et styles de conversation. Cela peut constituer un point de départ pour la personnalité de votre bot, et vous permettra d’économiser le temps et les efforts qui seraient nécessaires à leur écriture à partir de zéro. Découvrez [comment ajouter ce contenu de bavardages](../how-to/chit-chat-knowledge-base.md) à votre base de connaissances.

## <a name="structured-data-format-through-import"></a>Format de données structurées via l’importation

L’importation d’une base de connaissances remplace le contenu de la base de connaissances existante. L’importation requiert un fichier `.tsv` structuré qui contient des questions et leur réponse. Ces informations aident QnA Maker à regrouper les paires de question-réponse et à les attribuer à une source de données en particulier.

| Question  | Réponse  | Source| Métadonnées (1 clé : 1 valeur) |
|-----------|---------|----|---------------------|
| Question1 | Réponse1 | Url1 | <code>Key1:Value1 &#124; Key2:Value2</code> |
| Question2 | Réponse2 | Éditorial|    `Key:Value`       |

## <a name="structured-multi-turn-format-through-import"></a>Format multitour structuré via l’importation

Vous pouvez créer les conversations multitours dans un format de fichier `.tsv`. Le format vous permet de créer les conversations multitours en analysant les journaux de conversation précédents (avec d’autres processus, sans utiliser QnA Maker), puis de créer le fichier `.tsv` par le biais de l’automatisation. Importez le fichier pour remplacer la base de connaissances existante.

> [!div class="mx-imgBorder"]
> ![Modèle conceptuel de trois niveaux de question multitour](../media/qnamaker-concepts-knowledgebase/nested-multi-turn.png)

La colonne d’un `.tsv` multitour, spécifique au multitour, est **Invites**. Un exemple de `.tsv`, affiché dans Excel, illustre les informations à inclure pour définir les enfants multitours :

```JSON
[
    {"displayOrder":0,"qnaId":2,"displayText":"Level 2 Question A"},
    {"displayOrder":0,"qnaId":3,"displayText":"Level 2 - Question B"}
]
```

Le **displayOrder** est numérique et le **displayText** est un texte qui ne doit pas inclure de Markdown.

> [!div class="mx-imgBorder"]
> ![Exemple de questions multitours comme indiqué dans Excel](../media/qnamaker-concepts-knowledgebase/multi-turn-tsv-columns-excel-example.png)

## <a name="export-as-example"></a>Exporter en tant qu’exemple

Si vous ne savez pas comment représenter votre paire QnA dans le fichier `.tsv` :
* Utilisez cet [exemple téléchargeable à partir de GitHub](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx?raw=true)
* Ou créez la paire dans le portail QnA Maker, enregistrez, puis exportez la base de connaissances pour obtenir un exemple illustrant la façon de représenter la paire.

## <a name="content-types-of-documents-you-can-add-to-a-knowledge-base"></a>Types de contenu de documents que vous pouvez ajouter à une base de connaissances
Les types de contenu incluent bon nombre de documents structurés standard tels que PDF, DOC et TXT.

### <a name="file-and-url-data-types"></a>Types de données de fichier et d’URL

Le tableau ci-dessous récapitule les types de contenu et formats de fichiers pris en charge par QnA Maker.

|Type de source|Type de contenu| Exemples|
|--|--|--|
|URL|FAQ<br> (plates, avec des sections ou une page d’accueil de rubriques)<br>Pages de support <br> (Articles sur les procédures d’une seule page, articles sur la résolution des problèmes, etc.)|[FAQ brut](../troubleshooting.md), <br>[Forum aux questions avec des liens](https://www.microsoft.com/en-us/software-download/faq),<br> [Forum aux questions avec une page d’accueil contenant des rubriques](https://www.microsoft.com/Licensing/servicecenter/Help/Faq.aspx)<br>[Article de support technique](./best-practices.md)|
|PDF / DOC|FAQs,<br> Manuel de produit,<br> Brochures,<br> Article,<br> Stratégie de prospectus,<br> Guide de support,<br> Questions et réponses structurées,<br> , etc.|**Sans fonctionnalité multi-tour**<br>[Structured QnA.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/structured.docx),<br> [Sample Product Manual.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/product-manual.pdf),<br> [Sample semi-structured.docx](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/semi-structured.docx),<br> [Sample white paper.pdf](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/white-paper.pdf),<br><br>**Fonctionnalité multi-tour** :<br>[Surface Pro (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)<br>[Contoso Benefits (docx)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.docx)<br>[Contoso Benefits (pdf)](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-ContosoBenefits.pdf)|
|*Excel|Fichier QnA structuré<br> (y compris la prise en charge de RTF et HTML)|**Sans fonctionnalité multi-tour** :<br>[Sample QnA FAQ.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/QnA%20Maker%20Sample%20FAQ.xlsx)<br><br>**Fonctionnalité multi-tour** :<br>[Structured simple FAQ.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Structured-multi-turn-format.xlsx)<br>[Surface laptop FAQ.xls](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Multiturn-Surface-Pro.xlsx)|
|*TXT/TSV|Fichier QnA structuré|[Sample chit-chat.tsv](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/Scenario_Responses_Friendly.tsv)|

Si vous avez besoin d’une authentification pour votre source de données, envisagez les méthodes suivantes afin d'obtenir ce contenu dans QnA Maker :

* Télécharger le fichier manuellement et l’importer dans QnA Maker
* Ajouter le fichier à partir d’un [emplacement SharePoint](../how-to/add-sharepoint-datasources.md) authentifié

### <a name="url-content"></a>Contenu d'URL

Deux types de documents peuvent être importés via **URL** dans QnA Maker :

* URL de FAQ
* URL de support technique

Chaque type indique un format attendu.

### <a name="file-based-content"></a>Contenu basé sur des fichiers

Vous pouvez ajouter des fichiers à une base de connaissances à partir d’une source publique ou de votre système de fichiers local, dans le portail [QnA Maker](https://www.qnamaker.ai).

### <a name="content-format-guidelines"></a>Instructions relatives au format de contenu

En savoir plus sur les [directives relatives au format](../reference-document-format-guidelines.md) des différents fichiers.

## <a name="next-steps"></a>Étapes suivantes

Comprendre quelles informations sont stockées dans une paire [question/réponse (Q/R)](question-answer-set.md).
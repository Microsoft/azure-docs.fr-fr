---
title: Format Markdown - QnA Maker
description: Voici la liste des formats Markdown que vous pouvez utiliser dans le texte de réponse de QnA Maker.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: reference
ms.date: 03/19/2020
ms.openlocfilehash: f49e05578f55a38845acbd4010f928fb17c51606
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96352267"
---
# <a name="markdown-format-supported-in-qna-maker-answer-text"></a>Format Markdown pris en charge dans le texte de réponse QnA Maker

QnA Maker stocke le texte de réponse au format Markdown. Il existe plusieurs saveurs de Markdown. Pour vérifier que le texte de réponse est retourné et correctement affiché, utilisez cette référence.

Utilisez le tutoriel **[CommonMark](https://commonmark.org/help/tutorial/index.html)** pour valider votre Markdown. Ce tutoriel a une fonctionnalité **Essayer** pour la validation rapide de copier/coller.

## <a name="when-to-use-rich-text-editing-versus-markdown"></a>Choisir entre la modification de texte enrichi et Markdown

[Modifier le texte enrichi](How-To/edit-knowledge-base.md#add-an-editorial-qna-set) de réponses vous permet, en tant qu’auteur, d’utiliser une barre d’outils de mise en forme pour sélectionner le texte et le mettre en forme rapidement.

Markdown est un outil plus approprié quand vous devez générer automatiquement du contenu pour créer des bases de connaissances à importer dans le cadre d’un pipeline CI/CD ou pour des [tests par lots](./index.yml).

## <a name="supported-markdown-format"></a>Format Markdown pris en charge

Voici la liste des formats Markdown que vous pouvez utiliser dans le texte de réponse de QnA Maker.

|Objectif|Format|Exemple de Markdown|Rendu<br>tel qu’il est affiché dans le bot de chat|
|--|--|--|--|
Nouvelle ligne entre 2 phrases.|`\n\n`|`How can I create a bot with \n\n QnA Maker?`|![nouvelle ligne entre 2 phrases](./media/qnamaker-concepts-datasources/format-newline.png)|
|En-têtes de H1 à H6. Le nombre de `#` dénote de quel en-tête il s’agit. 1 `#` correspond à H1.|`\n# text \n## text \n### text \n####text \n#####text` |`## Creating a bot \n ...text.... \n### Important news\n ...text... \n### Related Information\n ....text...`<br><br>`\n# my h1 \n## my h2\n### my h3 \n#### my h4 \n##### my h5`|![format avec en-têtes Markdown](./media/qnamaker-concepts-datasources/format-headers.png)<br>![format avec en-têtes Markdown H1 à H5](./media/qnamaker-concepts-datasources/format-h1-h5.png)|
|Italique |`*text*`|`How do I create a bot with *QnA Maker*?`|![format en italique](./media/qnamaker-concepts-datasources/format-italics.png)|
|Fort (gras)|`**text**`|`How do I create a bot with **QnA Maker**?`|![format avec un marquage renforcé pour le gras](./media/qnamaker-concepts-datasources/format-strong.png)|
|URL du lien|`[text](https://www.my.com)`|`How do I create a bot with [QnA Maker](https://www.qnamaker.ai)?`|![format pour l’URL (lien hypertexte)](./media/qnamaker-concepts-datasources/format-url.png)|
|*URL pour l’image publique|`![text](https://www.my.com/image.png)`|`How can I create a bot with ![QnAMaker](https://review.docs.microsoft.com/azure/cognitive-services/qnamaker/media/qnamaker-how-to-key-management/qnamaker-resource-list.png)`|![format de l’URL de l’image publique ](./media/qnamaker-concepts-datasources/format-image-url.png)|
|Barré|`~~text~~`|`some ~~questoins~~ questions need to be asked`|![format barré](./media/qnamaker-concepts-datasources/format-strikethrough.png)|
|Gras et italique|`***text**_`|`How can I create a _*_QnA Maker_*_ bot?`|![format pour le gras et l’italique](./media/qnamaker-concepts-datasources/format-bold-italics.png)|
|URL en gras pour le lien|`[_*text**](https://www.my.com)`|`How do I create a bot with [**QnA Maker**](https://www.qnamaker.ai)?`|![format de l’URL en gras](./media/qnamaker-concepts-datasources/format-bold-url.png)|
|URL en italiques pour le lien|`[*text*](https://www.my.com)`|`How do I create a bot with [*QnA Maker*](https://www.qnamaker.ai)?`|![format de l’URL en italique](./media/qnamaker-concepts-datasources/format-url-italics.png)|
|Ajouter un caractère d’échappement devant les symboles Markdown|`\*text\*`|`How do I create a bot with \*QnA Maker\*?`|![Format d’un caractère d’échappement devant les symboles Markdown](./media/qnamaker-concepts-datasources/format-escape-markdown-symbols.png)|
|Liste triée|`\n 1. item1 \n 1. item2`|`This is an ordered list: \n 1. List item 1 \n 1. List item 2`<br>L’exemple précédent utilise la numérotation automatique intégrée au Markdown.<br>`This is an ordered list: \n 1. List item 1 \n 2. List item 2`<br>L’exemple précédent utilise la numérotation explicite.|![format de la liste triée](./media/qnamaker-concepts-datasources/format-ordered-list.png)|
|Liste non triée|`\n * item1 \n * item2`<br>or<br>`\n - item1 \n - item2`|`This is an unordered list: \n * List item 1 \n * List item 2`|![format de la liste triée](./media/qnamaker-concepts-datasources/format-unordered-list.png)|
|Listes imbriquées|`\n * Parent1 \n\t * Child1 \n\t * Child2 \n * Parent2`<br><br>`\n * Parent1 \n\t 1. Child1 \n\t * Child2 \n 1. Parent2`<br><br>Vous pouvez imbriquer des listes triées et non triées ensemble. L’onglet, `\t`, indique le niveau de mise en retrait de l’élément enfant.|`This is an unordered list: \n * List item 1 \n\t * Child1 \n\t * Child2 \n * List item 2`<br><br>`This is an ordered nested list: \n 1. Parent1 \n\t 1. Child1 \n\t 1. Child2 \n 1. Parent2`|![format de la liste non triée imbriquée](./media/qnamaker-concepts-datasources/format-nested-unordered-list.png)<br>![format de la liste triée imbriquée](./media/qnamaker-concepts-datasources/format-nested-ordered-list.png)|

*QnA Maker ne traite pas du tout l’image. C’est le rôle de l’application cliente d’effectuer le rendu de l’image.

Si vous souhaitez ajouter du contenu à l’aide des API de mise à jour ou de remplacement de la base de connaissances, et si le contenu ou le fichier contiennent des balises HTML, vous pouvez conserver le code HTML de votre fichier en veillant à ce que l’ouverture et la fermeture des balises soient converties au format encodé.

| Conserver le code HTML  | Représentation de la requête d’API  | Représentation dans la base de connaissances |
|-----------|---------|-------------------------|
| Oui | \&lt;br\&gt; | &lt;br&gt; |
| Oui | \&lt;h3\&gt;header\&lt;/h3\&gt; | &lt;h3&gt;header&lt;/h3&gt; |

En outre, CR LF (\r\n) est converti en \n dans la base de connaissances. LF (\n) est conservé tel quel. Si vous souhaitez échapper une séquence d’échappement telle que \t ou \n, vous pouvez utiliser la barre oblique inverse, par exemple : « \\\\r\\\\n » et « \\\\t ».

## <a name="next-steps"></a>Étapes suivantes

Passez en revue les [formats de fichiers](reference-tsv-format-batch-testing.md) des tests par lots.
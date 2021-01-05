---
title: Lancer le Lecteur immersif avec du contenu HTML
titleSuffix: Azure Cognitive Services
description: Cet article vous indique comment lancer le Lecteur immersif avec du contenu HTML.
author: metanMSFT
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: conceptual
ms.date: 01/14/2020
ms.author: metang
ms.custom: devx-track-js
ms.openlocfilehash: d028e9ef2087b866b3c168483d55556e38f9a7cd
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92636559"
---
# <a name="how-to-launch-the-immersive-reader-with-html-content"></a>Guide de lancement du Lecteur immersif avec du contenu HTML

Cet article explique comment lancer le Lecteur immersif avec du contenu HTML.

## <a name="prepare-the-html-content"></a>Préparer le contenu HTML

Placez le contenu que vous souhaitez afficher dans le Lecteur immersif à l’intérieur d’un élément conteneur. Assurez-vous que l’élément conteneur a un `id` unique. Le Lecteur immersif fournit la prise en charge des éléments HTML de base. Consultez la [référence](./reference.md#html-support) pour plus d’informations.

```html
<div id='immersive-reader-content'>
    <b>Bold</b>
    <i>Italic</i>
    <u>Underline</u>
    <strike>Strikethrough</strike>
    <code>Code</code>
    <sup>Superscript</sup>
    <sub>Subscript</sub>
    <ul><li>Unordered lists</li></ul>
    <ol><li>Ordered lists</li></ol>
</div>
```

## <a name="get-the-html-content-in-javascript"></a>Recevoir le contenu HTML dans JavaScript

Utilisez l’`id` de l’élément conteneur pour récupérer le contenu HTML dans votre code JavaScript.

```javascript
const htmlContent = document.getElementById('immersive-reader-content').innerHTML;
```

## <a name="launch-the-immersive-reader-with-your-html-content"></a>Lancer le Lecteur immersif avec votre contenu HTML

Lors de l’appel de `ImmersiveReader.launchAsync`, définissez la propriété `mimeType` du bloc sur `text/html` pour activer l’affichage HTML.

```javascript
const data = {
    chunks: [{
        content: htmlContent,
        mimeType: 'text/html'
    }]
};

ImmersiveReader.launchAsync(YOUR_TOKEN, YOUR_SUBDOMAIN, data, YOUR_OPTIONS);
```

## <a name="next-steps"></a>Étapes suivantes

* Explorer le [Guide de référence du SDK du Lecteur immersif](./reference.md)
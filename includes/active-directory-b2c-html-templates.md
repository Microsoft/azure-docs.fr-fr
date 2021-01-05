---
author: msmimart
ms.service: active-directory-b2c
ms.subservice: B2C
ms.topic: include
ms.date: 02/12/2020
ms.author: mimart
ms.openlocfilehash: d43b879057001d62ea72bd2e011ad52957d47470
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95990879"
---
## <a name="sample-templates"></a>Exemples de modèles
Vous trouverez ici des exemples de modèles pour la personnalisation de l’interface utilisateur :

```bash
git clone https://github.com/Azure-Samples/Azure-AD-B2C-page-templates
```

Ce projet contient les modèles suivants :
- [Ocean Blue](https://github.com/Azure-Samples/Azure-AD-B2C-page-templates/tree/master/ocean_blue) (Bleu océan)
- [Slate Gray](https://github.com/Azure-Samples/Azure-AD-B2C-page-templates/tree/master/slate_gray) (Gris ardoise)

Pour utiliser l’exemple :

1. Clonez le référentiel sur votre ordinateur local. Choisissez un dossier de modèles `/ocean_blue` ou `/slate_gray`.
1. Chargez tous les fichiers du dossier de modèles et du dossier `/assets` dans le stockage Blob, comme décrit dans les sections précédentes.
1. Ensuite, ouvrez chaque fichier `\*.html` dans la racine de `/ocean_blue` ou `/slate_gray`, remplacez toutes les instances d’URL relatives par les URL des fichiers css, images et de polices que vous avez chargés à l’étape 2. Par exemple :
    ```html
    <link href="./css/assets.css" rel="stylesheet" type="text/css" />
    ```

    À
    ```html
    <link href="https://your-storage-account.blob.core.windows.net/your-container/css/assets.css" rel="stylesheet" type="text/css" />
    ```
1. Enregistrez les fichiers `\*.html` et chargez-les dans le stockage Blob.
1. Modifiez à présent la stratégie, en la faisant pointer vers votre fichier HTML, comme indiqué précédemment.
1. Si vous constatez qu’il manque des polices, des images ou des feuilles de style CSS, vérifiez vos références dans la stratégie d’extensions et les fichiers \*.html.

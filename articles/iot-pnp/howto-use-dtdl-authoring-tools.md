---
title: Utiliser un outil pour créer et valider des modèles DTDL | Microsoft Docs
description: Installez l’éditeur DTDL pour Visual Studio Code ou Visual Studio 2019 et utilisez-le pour créer des modèles IoT Plug-and-Play.
author: dominicbetts
ms.author: dobett
ms.date: 09/14/2020
ms.topic: how-to
ms.service: iot-pnp
services: iot-pnp
ms.openlocfilehash: 40d1ae4da07e159c24970c065d1c39e22b89a29a
ms.sourcegitcommit: 58f12c358a1358aa363ec1792f97dae4ac96cc4b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93280195"
---
# <a name="install-and-use-the-dtdl-authoring-tools"></a>Installer et utiliser les outils de création DTDL

Les modèles [DTDL (Digital Twins Definition Language)](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md) sont des fichiers JSON. Vous pouvez utiliser une extension pour Visual Studio Code ou Visual Studio 2019 pour créer et valider ces fichiers de modèle.

## <a name="install-and-use-the-vs-code-extension"></a>Installer et utiliser l’extension VS Code

L’extension DTDL pour VS Code ajoute les fonctionnalités de création DTDL suivantes :

- Validation de la syntaxe DTDL v2.
- IntelliSense, y compris l’autocomplétion, pour vous aider dans la syntaxe du langage.
- Possibilité de créer des interfaces à partir de la palette de commandes.

Pour installer l’extension DTDL, accédez à l’[éditeur DTDL pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=vsciot-vscode.vscode-dtdl). Vous pouvez également rechercher **DTDL** dans la vue Extensions de VS Code.

Une fois l’extension installée, utilisez-la pour vous aider à créer des fichiers de modèle DTDL dans VS code :

- L’extension fournit la validation de la syntaxe dans les fichiers de modèle DTDL, en mettant en surbrillance les erreurs comme indiqué sur la capture d’écran suivante :

    :::image type="content" source="media/howto-use-dtdl-authoring-tools/model-validation.png" alt-text="Validation de modèle dans VS Code":::

- Utilisez IntelliSense et l’autocomplétion lorsque vous modifiez des modèles DTDL :

    :::image type="content" source="media/howto-use-dtdl-authoring-tools/model-intellisense.png" alt-text="Utiliser IntelliSense pour les modèles DTDL dans VS Code":::

- Créez une nouvelle interface DTDL. La commande **DTDL: Create Interface** crée un fichier JSON avec une nouvelle interface. L’interface comprend des exemples de définitions de télémétrie, de propriété et de commande.

## <a name="install-and-use-the-visual-studio-extension"></a>Installer et utiliser l’extension Visual Studio

L’extension DTDL pour Visual Studio 2019 ajoute les fonctionnalités de création DTDL suivantes :

- Validation de la syntaxe DTDL v2.
- IntelliSense, y compris l’autocomplétion, pour vous aider dans la syntaxe du langage.

Pour installer l’extension DTDL, accédez à [Prise en charge du langage DTDL pour VS 2019](https://marketplace.visualstudio.com/items?itemName=vsc-iot.vs16dtdllanguagesupport). Vous pouvez également rechercher **DTDL** dans **Gérer les extensions** dans Visual Studio.

Une fois l’extension installée, utilisez-la pour vous aider à créer des fichiers de modèle DTDL dans Visual Studio :

- L’extension fournit la validation de la syntaxe dans les fichiers de modèle DTDL, en mettant en surbrillance les erreurs comme indiqué sur la capture d’écran suivante :

    :::image type="content" source="media/howto-use-dtdl-authoring-tools/model-validation-2.png" alt-text="Validation de modèles dans Visual Studio":::

- Utilisez IntelliSense et l’autocomplétion lorsque vous modifiez des modèles DTDL :

    :::image type="content" source="media/howto-use-dtdl-authoring-tools/model-intellisense-2.png" alt-text="Utiliser IntelliSense pour les modèles DTDL dans Visual Studio":::

## <a name="next-steps"></a>Étapes suivantes

Dans cet article de procédure, vous avez appris à utiliser les extensions DTDL pour Visual Studio Code et Visual Studio 2019 pour créer et valider des fichiers de modèle DTDL. À présent, nous vous invitons à découvrir comment utiliser l’[explorateur Azure IoT avec vos modèles et appareils](./howto-use-iot-explorer.md).

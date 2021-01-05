---
title: 'Tutoriel : Démarrer le Lecteur immersif à l’aide de l’exemple de code Swift iOS'
titleSuffix: Azure Cognitive Services
description: Dans ce tutoriel, vous allez configurer et exécuter un exemple d’application Swift qui démarre le Lecteur immersif.
services: cognitive-services
author: dylankil
manager: guillasi
ms.service: cognitive-services
ms.subservice: immersive-reader
ms.topic: tutorial
ms.date: 06/10/2020
ms.author: dylankil
ms.openlocfilehash: ce9d3af8f7517e2acae5264197b842d47085279c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88516365"
---
# <a name="tutorial-start-the-immersive-reader-using-the-swift-ios-code-sample"></a>Tutoriel : Démarrer le Lecteur immersif à l’aide de l’exemple de code Swift iOS

Dans la [présentation](./overview.md), vous avez appris ce qu’est le Lecteur immersif et comment il implémente des techniques éprouvées pour améliorer la compréhension de la lecture pour les apprenants en langue, les lecteurs émergents et les étudiants présentant des difficultés d’apprentissage. Ce tutoriel explique comment créer une application iOS qui démarre le Lecteur immersif. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Configurer et exécuter une application Swift pour iOS à l’aide d’un exemple de projet
> * Obtenez un jeton d’accès.
> * Démarrer le Lecteur immersif avec un exemple de contenu

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

* Une ressource Lecteur immersif configurée pour l’authentification Azure Active Directory. Suivez [ces instructions](./how-to-create-immersive-reader.md) pour la configurer. Vous aurez besoin de certaines des valeurs créées ici quand vous configurerez les propriétés d’environnement. Enregistrez la sortie de votre session dans un fichier texte pour référence ultérieure.
* [macOS](https://www.apple.com/macos).
* [Git](https://git-scm.com/).
* [Kit SDK Lecteur immersif](https://github.com/microsoft/immersive-reader-sdk).
* [Xcode](https://apps.apple.com/us/app/xcode/id497799835?mt=12).

## <a name="configure-authentication-credentials"></a>Configurer les informations d’identification d’authentification

1. Ouvrez Xcode, puis ouvrez **immersive-reader-sdk/js/samples/ios/quickstart-swift/quickstart-swift.xcodeproj**.
1. Dans le menu supérieur, sélectionnez **Product** > **Scheme** > **Edit Scheme**.
1. Dans la vue **Run**, cliquez sur l’onglet **Arguments**.
1. Dans la section **Environment Variables**, ajoutez les noms et valeurs suivants. Spécifiez les valeurs fournies au moment de la création de votre ressource Lecteur immersif.

    ```text
    TENANT_ID=<YOUR_TENANT_ID>
    CLIENT_ID=<YOUR_CLIENT_ID>
    CLIENT_SECRET<YOUR_CLIENT_SECRET>
    SUBDOMAIN=<YOUR_SUBDOMAIN>
    ```

Ne validez pas cette modification dans le contrôle de code source, car elle contient des secrets qui ne doivent pas être rendus publics.

## <a name="start-the-immersive-reader-with-sample-content"></a>Démarrer le Lecteur immersif avec un exemple de contenu

Dans Xcode, appuyez sur **Ctrl+R** pour exécuter le projet.

## <a name="next-steps"></a>Étapes suivantes

* Explorer le [SDK Lecteur Immersif](https://github.com/microsoft/immersive-reader-sdk) et la [référence du SDK Lecteur immersif](./reference.md)
* Voir des exemples de code sur [GitHub](https://github.com/microsoft/immersive-reader-sdk/tree/master/js/samples/)

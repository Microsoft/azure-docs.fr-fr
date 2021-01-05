---
title: Comment fonctionne le débogage à distance de votre code avec Azure Dev Spaces
services: azure-dev-spaces
ms.date: 03/24/2020
ms.topic: conceptual
description: Décrit les processus de débogage à distance sur Azure Kubernetes Service avec Azure Dev Spaces
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: 0487b80d23974a66bafe93ee1fbdf9b796d0ab53
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91975040"
---
# <a name="how-remote-debugging-your-code-with-azure-dev-spaces-works"></a>Comment fonctionne le débogage à distance de votre code avec Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Azure Dev Spaces vous offre plusieurs façons de parcourir et de déboguer rapidement des applications Kubernetes et de collaborer avec votre équipe sur un cluster Azure Kubernetes Service (AKS). Une fois que votre projet s’exécute dans un espace de développement, Azure Dev Spaces offre un moyen d’attacher et de déboguer une application en cours d’exécution dans AKS.

Cet article décrit le fonctionnement du débogage à distance avec Dev Spaces.

## <a name="debug-your-code"></a>Déboguer votre code

Pour les applications Java, .NET Core et Node.js, vous pouvez déboguer votre application exécutée directement dans votre espace de développement en utilisant Visual Studio Code ou Visual Studio. Visual Studio Code et Visual Studio fournissent des outils pour se connecter à votre espace de développement, lancer votre application et joindre un débogueur. Après avoir lancé `azds prep`, vous pouvez ouvrir votre projet dans Visual Studio Code ou Visual Studio. Visual Studio Code ou Visual Studio généreront leurs propres fichiers de configuration pour la connexion, une opération distincte de l’exécution de `azds prep`. Depuis Visual Studio Code ou Visual Studio, vous pouvez définir des points d'arrêt et lancer votre application dans votre espace de développement.

![Débogage de votre code](media/get-started-node/debug-configuration-nodejs2.png)

Lorsque vous lancez votre application en utilisant Visual Studio Code ou Visual Studio pour le débogage, ces programmes gèrent le lancement et la connexion à votre espace de développement de la même manière qu'en exécutant `azds up`. Par ailleurs, les outils côté client de Visual Studio Code et Visual Studio fournissent chacun un paramètre supplémentaire avec des informations spécifiques pour le débogage. Ce paramètre contient le nom de l'image du débogueur, l'emplacement du débogueur dans cette image, et l'emplacement de destination dans le conteneur de l'application pour monter le dossier du débogueur.

L'image du débogueur est automatiquement déterminée par les outils côté client. Ces outils adoptent une méthode similaire à celle utilisée lors de l'exécution du Dockerfile et du graphique Helm Chart générés lors de l'exécution de `azds prep`. Une fois le débogueur monté dans l'image de l'application, il est lancé en utilisant `azds exec`.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](how-dev-spaces-works.md)

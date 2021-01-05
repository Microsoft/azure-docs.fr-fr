---
title: Fonctionnement de Visual Studio Code avec Azure Dev Spaces
services: azure-dev-spaces
ms.date: 07/08/2019
ms.topic: conceptual
description: Découvrez comment Visual Studio Code et Azure Dev Spaces vous aident à déboguer et à itérer rapidement vos applications Kubernetes
keywords: Azure Dev Spaces, Dev Spaces, Docker, Kubernetes, Azure, AKS, Azure Kubernetes Service, conteneurs
ms.openlocfilehash: edfcb8107280bb86144b798da2d5b1c16371528e
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91960743"
---
# <a name="how-visual-studio-code-works-with-azure-dev-spaces"></a>Fonctionnement de Visual Studio Code avec Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../includes/dev-spaces-deprecation.md)]

Vous pouvez utiliser Visual Studio Code et l’[extension Azure Dev Spaces][azds-extension] pour préparer, exécuter et déboguer vos services avec Azure Dev Spaces. Avec Visual Studio Code et l’extension Azure Dev Spaces, vous pouvez :

* Générez des ressources pour l’exécution et le débogage de services dans AKS
* Exécutez vos services Java, Node.js et .NET Core dans un espace de développement
* Déboguez directement vos services Java, Node.js et .NET Core en cours d’exécution dans un espace de développement

## <a name="generate-assets"></a>Générer des ressources

Visual Studio Code et l’extension Azure Dev Spaces génèrent les ressources suivantes pour votre projet :

* Dockerfile pour les applications Java à l’aide de Maven, des applications Node.js et des applications .NET Core
* Graphiques Helm pour presque n’importe quel langage avec un Dockerfile
* Un fichier `azds.yaml`, c’est-à-dire le [fichier de configuration Azure Dev Spaces][azds-yaml] pour votre projet
* Un `.vscode` dossier avec la configuration de lancement de Visual Studio Code de votre projet pour les applications Java à l’aide de Maven, des applications Node.js et des applications .NET Core

Le Dockerfile, le graphique Helm, et les fichiers `azds.yaml` sont les mêmes ressources générées pendant l’exécution de `azds prep`. Ces fichiers peuvent également être utilisés en dehors du code de Visual Studio pour exécuter votre projet dans AKS, telles que l’exécution de `azds up`. Le dossier `.vscode` est utilisé uniquement par le code de Visual Studio pour exécuter votre projet dans AKS à partir de Visual Studio Code.

## <a name="run-your-service-in-aks"></a>Exécutez votre service dans AKS

Une fois que vous générez les ressources pour votre projet, vous pouvez exécuter vos services Java, Node.js et .NET Core dans un espace de développement existant à partir de Visual Studio Code. Dans la page *Déboguer* de Visual Studio Code, vous pouvez appeler la configuration de lancement à partir du répertoire `.vscode` pour exécuter votre projet.

Vous devez créer votre cluster AKS et activer Azure Dev Spaces dans votre cluster en dehors de Visual Studio Code. Vous pouvez réutiliser les Dockerfiles existant, les graphiques Helm, et les fichiers `azds.yaml` créés en dehors de Visual Studio Code, tels que les ressources générées en exécutant `azds prep`. Si vous réutilisez les ressources générées en dehors de Visual Studio Code, vous devez toujours avoir un répertoire `.vscode`. Ce répertoire `.vscode` peut être régénérée par Visual Studio code et l’extension Azure Dev Spaces et ne remplacera pas vos ressources existantes.

Pour les projets .NET Core, vous devez avoir l’[extension C#][csharp-extension] installée pour exécuter votre service .NET à partir de Visual Studio Code. Par ailleurs, pour les projets Java utilisant Maven, l’[extension débogueur Java pour Azure Dev Spaces][java-extension] doit être installée et [Maven doit être installé et configuré][maven] pour que vous puissiez exécuter votre service Java à partir de Visual Studio Code.

## <a name="debug-your-service-in-aks"></a>Déboguez votre service dans AKS

Une fois que vous lancez votre projet, vous pouvez déboguer vos services Java, Node.js et .NET Core en cours d’exécution dans un espace de développement directement à partir de Visual Studio Code. La configuration de lancement dans le répertoire `.vscode` fournit les informations de débogage supplémentaires pour un service en cours d’exécution avec débogage activé dans un espace de développement. Visual Studio Code se lie également au processus de débogage dans le conteneur en cours d’exécution dans vos espaces de développement, ce qui vous permet de définir des points d’arrêt, inspecter des variables et effectuer d’autres opérations de débogage.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](how-dev-spaces-works.md)

[azds-extension]: https://marketplace.visualstudio.com/items?itemName=azuredevspaces.azds
[azds-yaml]: how-dev-spaces-works-prep.md#prepare-your-code
[csharp-extension]: https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp
[java-extension]: https://marketplace.visualstudio.com/items?itemName=vscjava.vscode-java-debugger-azds
[maven]: https://maven.apache.org

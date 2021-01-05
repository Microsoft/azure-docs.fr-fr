---
title: Développer et exécuter des fonctions Azure Functions en local
description: Apprenez à coder et à tester des fonctions Azure Functions sur votre ordinateur local avant de les exécuter dans Azure Functions.
ms.topic: conceptual
ms.date: 09/04/2018
ms.openlocfilehash: 9c37d51abcc8d612b777b845515cf07666369d4f
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96168132"
---
# <a name="code-and-test-azure-functions-locally"></a>Coder et tester Azure Functions localement

Bien qu’il soit possible de développer et tester Azure Functions dans le [Azure portal], de nombreux développeurs préfèrent une expérience de développement local. Functions facilite l’utilisation de votre éditeur de code et de vos outils de développement préférés pour développer et tester des fonctions sur votre ordinateur local. Vos fonctions locales peuvent être connectées aux services Azure actifs, et vous pouvez les déboguer sur votre ordinateur local à l’aide du runtime Functions complet.

## <a name="local-development-environments"></a>Environnements de développement local

La façon dont vous développez des fonctions sur votre ordinateur local dépend de votre [langage](supported-languages.md) et de vos préférences d’outils. Les environnements du tableau suivant prennent en charge le développement local :

|Environnement                              |Languages         |Description|
|-----------------------------------------|------------|---|
|[Visual Studio Code](functions-develop-vs-code.md)| [C# (bibliothèque de classes)](functions-dotnet-class-library.md), [Script C# (.csx)](functions-reference-csharp.md), [JavaScript](functions-reference-node.md), [PowerShell](./create-first-function-vs-code-powershell.md), [Python](functions-reference-python.md) | L’[extension Azure Functions pour VS Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) ajoute la prise en charge de Functions à VS Code. Requiert les outils de base. Prend en charge le développement sur Linux, MacOS et Windows, lorsque vous utilisez la version 2.x des outils de base. Pour en savoir plus, voir [Créer votre première fonction à l’aide de Visual Studio Code](./create-first-function-vs-code-csharp.md). |
| [Invite de commandes terminal](functions-run-local.md) | [C# (bibliothèque de classes)](functions-dotnet-class-library.md), [Script C# (.csx)](functions-reference-csharp.md), [JavaScript](functions-reference-node.md), [PowerShell](functions-reference-powershell.md), [Python](functions-reference-python.md) | [Azure Functions Core Tools] fournit le runtime et les modèles principaux de création de fonctions, ce qui permet le développement local. La version 2.x prend en charge le développement sur Linux, MacOS et Windows. Tous les environnements s’appuient sur les outils de base pour le runtime Functions local. |
| [Visual Studio 2019](functions-develop-vs.md) | [C# (bibliothèque de classes)](functions-dotnet-class-library.md) | Les outils Azure Functions sont inclus dans la charge de travail de **développement Azure** de [Visual Studio 2019](https://www.visualstudio.com/vs/) et des versions ultérieures. Vous permet de compiler les fonctions dans une bibliothèque de classes et de publier le fichier .dll dans Azure. Inclut les principaux outils pour effectuer des test en local. Pour en savoir plus, voir [Développer Azure Functions à l’aide de Visual Studio](functions-develop-vs.md). |
| [Maven](./create-first-function-cli-java.md) (divers) | [Java](functions-reference-java.md) | S’intègre avec les outils principaux pour permettre le développement de fonctions Java. La version 2.x prend en charge le développement sur Linux, MacOS et Windows. Pour en savoir plus, consultez [Créer votre première fonction dans Azure avec Java et Maven](./create-first-function-cli-java.md). Prend également en charge le développement en utilisant [Eclipse](functions-create-maven-eclipse.md) et [IntelliJ IDEA](functions-create-maven-intellij.md) |

[!INCLUDE [Don't mix development environments](../../includes/functions-mixed-dev-environments.md)]

Chacun de ces environnements de développement local vous permet de créer des projets d’application de fonction et d’utiliser des modèles Functions prédéfinis pour créer de nouvelles fonctions. Ils intègrent les outils principaux afin que vous puissiez tester et déboguer vos fonctions selon le runtime Functions réel sur votre propre ordinateur comme vous le feriez pour toute autre application. Vous pouvez également publier votre projet d’application de fonction à partir d’un de ces environnements sur Azure.

## <a name="next-steps"></a>Étapes suivantes

+ Pour en savoir plus sur le développement local de fonctions C# précompilées à l’aide de Visual Studio 2019, consultez [Développer Azure Functions à l’aide de Visual Studio](functions-develop-vs.md).
+ Pour en savoir plus sur le développement local de fonctions à l’aide de VS Code sur un ordinateur Mac, Linux ou Windows, voir [Déployer Azure Functions à partir de Visual Studio Code](/azure/developer/javascript/tutorial-vscode-serverless-node-01).
+ Pour en savoir plus sur le développement de fonctions à partir de l’invite de commandes ou d’un terminal, consultez [Coder et tester Azure Functions localement](functions-run-local.md).

<!-- LINKS -->

[Azure Functions Core Tools]: https://www.npmjs.com/package/azure-functions-core-tools
[Azure portal]: https://portal.azure.com
[Node.js]: https://docs.npmjs.com/getting-started/installing-node#osx-or-windows
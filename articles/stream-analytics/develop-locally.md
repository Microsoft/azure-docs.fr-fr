---
title: Développer et déboguer des travaux Azure Stream Analytics localement
description: Découvrez comment développer et tester des travaux Azure Stream Analytics sur votre ordinateur local avant de les exécuter dans le portail Azure.
ms.author: mamccrea
author: mamccrea
ms.topic: conceptual
ms.date: 03/31/2020
ms.service: stream-analytics
ms.openlocfilehash: 797577ce976f71f41ece0d103bd0145775d48b56
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90885587"
---
# <a name="develop-and-debug-azure-stream-analytics-jobs-locally"></a>Développer et déboguer des travaux Azure Stream Analytics localement

Bien qu’il soit possible de créer et de tester des travaux Azure Stream Analytics dans le portail Azure, de nombreux développeurs préfèrent une expérience de développement local. Stream Analytics facilite l’utilisation de votre éditeur de code et de vos outils de développement préférés pour créer et tester des travaux avec des flux d’événements en direct depuis Azure Event Hub, IoT Hub et Stockage Blob à l’aide d’un runtime local à nœud unique entièrement fonctionnel. Vous pouvez également soumettre des travaux à Azure directement depuis votre environnement de développement local.

## <a name="local-development-environments"></a>Environnements de développement local

La manière dont vous développez des travaux Stream Analytics sur votre ordinateur local dépend de vos préférences en matière d’outils et de la disponibilité des fonctionnalités. Pour connaître les fonctionnalités prises en charge pour chaque environnement de développement, consultez [Comparaison des fonctionnalités Azure Stream Analytics](feature-comparison.md).

Les environnements du tableau suivant prennent en charge le développement local :

|Environnement                              |Description    |
|-----------------------------------------|------------|
|[Visual Studio Code](visual-studio-code-explore-jobs.md)| [L’extension Azure Stream Analytics Tools](https://marketplace.visualstudio.com/items?itemName=ms-bigdatatools.vscode-asa) pour Visual Studio Code vous permet de créer, de gérer et de tester votre travail Stream Analytics localement et dans le cloud à l’aide d’IntelliSense et du contrôle de code source natif. Prend en charge le développement sur Linux, macOS et Windows. Pour plus d'informations, consultez [Créer une tâche Azure Stream Analytics dans Visual Studio Code](quick-create-visual-studio-code.md). L’extension prend également en charge [Visual Studio Codespaces](https://visualstudio.microsoft.com/services/visual-studio-codespaces/), un environnement de développement hébergé dans le cloud.|
|[Visual Studio 2019](stream-analytics-tools-for-visual-studio-install.md) |Stream Analytics Tools fait partie des charges de travail de développement Azure et Stockage et traitement des données dans Visual Studio. Vous pouvez utiliser Visual Studio pour écrire des fonctions et des désérialiseurs C# définis par l'utilisateur. Pour plus d'informations, consultez [Créer une tâche Azure Stream Analytics à l’aide de Visual Studio](stream-analytics-quick-create-vs.md).|
|[Invite de commandes terminal](stream-analytics-tools-for-visual-studio-cicd.md)|Le package NuGet CI/CD Azure Stream Analytics fournit des outils pour la génération de projets Visual Studio testés localement sur un ordinateur arbitraire. Le package npm CI/CD Azure Stream Analytics fournit des outils pour la génération de projets Visual Studio Code (qui génèrent un modèle Azure Resource Manager) sur un ordinateur arbitraire.|

## <a name="next-steps"></a>Étapes suivantes

* [Tester des requêtes Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code](visual-studio-code-local-run.md)
* [Tester des requêtes Stream Analytics localement par rapport à une entrée de stream en direct à l'aide de Visual Studio Code](visual-studio-code-local-run-live-input.md)
* [Tester des requêtes Stream Analytics localement avec Visual Studio](stream-analytics-vs-tools-local-run.md)
* [Tester des données actives localement à l’aide d’Azure Stream Analytics Tools pour Visual Studio](stream-analytics-live-data-local-testing.md)

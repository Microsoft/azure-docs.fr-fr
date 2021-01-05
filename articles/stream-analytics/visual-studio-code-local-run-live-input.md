---
title: Tester des requêtes Azure Stream Analytics localement par rapport à une entrée de stream en direct à l'aide de Visual Studio Code
description: Cet article explique comment tester des requêtes localement par rapport à une entrée de stream en direct à l'aide d'Azure Stream Analytics Tools pour Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/14/2019
ms.topic: how-to
ms.openlocfilehash: dd4966ee75e9dc0ff401823e4291f8d299c8893c
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93122892"
---
# <a name="test-stream-analytics-queries-locally-against-live-stream-input-by-using-visual-studio-code"></a>Tester des requêtes Stream Analytics localement par rapport à une entrée de stream en direct à l'aide de Visual Studio Code

Vous pouvez utiliser Azure Stream Analytics Tools pour Visual Studio Code afin de tester vos travaux Stream Analytics localement par rapport à une entrée de stream en direct. L'entrée peut provenir d'une source comme Azure Event Hubs ou Azure IoT Hub. Les résultats sont envoyés sous forme de fichiers JSON vers un dossier de votre projet nommé **LocalRunOutputs**.

## <a name="prerequisites"></a>Prérequis

* Installez le [kit SDK .NET Core](https://dotnet.microsoft.com/download) et redémarrez Visual Studio Code.

* Utilisez ce [guide de démarrage rapide](quick-create-visual-studio-code.md) pour apprendre à créer un travail Stream Analytics à l'aide de Visual Studio Code.

## <a name="define-a-live-stream-input"></a>Définir une entrée de stream en direct

1. Cliquez avec le bouton droit sur le dossier **Entrées** de votre projet Stream Analytics. Sélectionnez ensuite **ASA : Ajouter une entrée** dans le menu contextuel.

   ![Ajouter une entrée à partir du dossier Entrées](./media/quick-create-visual-studio-code/add-input-from-inputs-folder.png)

   Vous pouvez également sélectionner **Ctrl+Maj+P** pour ouvrir la palette de commandes, puis entrer **ASA : Ajouter une entrée**.

   ![Ajouter une entrée Stream Analytics dans Visual Studio Code](./media/quick-create-visual-studio-code/add-input.png)

2. Dans la liste déroulante, sélectionnez un type de source d'entrée.

   ![Sélectionner IoT Hub comme option d'entrée](./media/quick-create-visual-studio-code/iot-hub.png)

3. Si vous avez ajouté l'entrée à partir de la palette de commandes, choisissez le script de requête Stream Analytics qui utilisera l'entrée. Celui-ci doit être automatiquement renseigné avec le chemin du fichier **myASAproj.asaql**.

   ![Sélectionner un script Stream Analytics dans Visual Studio Code](./media/quick-create-visual-studio-code/asa-script.png)

4. Choisissez **Sélectionner parmi vos abonnements Azure** dans le menu déroulant.

    ![Sélectionner parmi des abonnements](./media/quick-create-visual-studio-code/add-input-select-subscription.png)

5. Configurez le fichier JSON qui vient d'être généré. Vous pouvez vous aider de la fonctionnalité CodeLens pour entrer une chaîne, opérer une sélection dans une liste déroulante ou modifier le texte directement dans le fichier. La capture d’écran suivante montre **Sélectionner parmi vos abonnements** à titre d’exemple.

   ![Configurer l’entrée dans Visual Studio Code](./media/quick-create-visual-studio-code/configure-input.png)

## <a name="preview-input"></a>Afficher un aperçu de l'entrée

Pour afficher un aperçu des données d'entrée, sélectionnez **Aperçu des données** dans le fichier de configuration des entrées en direct, sur la ligne supérieure. Certaines données d'entrée proviennent d'un hub IoT et sont affichées dans la fenêtre d'aperçu. L'affichage de l'aperçu peut prendre quelques secondes.

 ![Aperçu de l'entrée en direct](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="run-queries-locally"></a>Exécuter des requêtes localement

Revenez à votre éditeur de requête, puis sélectionnez **Exécuter localement**. Sélectionnez ensuite **Utiliser l'entrée en direct** dans la liste déroulante.

![Sélectionner « Exécuter localement » dans l'éditeur de requêtes](./media/vscode-local-run/run-locally.png)

![Sélectionner « Utiliser l'entrée en direct »](./media/vscode-local-run-live-input/run-locally-use-live-input.png)

Le résultat apparaît dans la fenêtre de droite et est actualisé toutes les 3 secondes. Vous pouvez sélectionner **Exécuter** pour effectuer un nouveau test. Vous pouvez également sélectionner **Ouvrir dans le dossier** pour afficher les fichiers de résultats dans l'Explorateur de fichiers et les ouvrir avec Visual Studio Code ou un outil comme Excel. Notez que les fichiers de résultats sont uniquement disponibles au format JSON.

L'heure par défaut à laquelle le travail doit commencer à créer la sortie est définie sur **Maintenant**. Vous pouvez personnaliser cette heure en sélectionnant le bouton **Heure de début de la sortie** dans la fenêtre des résultats.

![Afficher le résultat de l'exécution locale](./media/vscode-local-run-live-input/vscode-livetesting.gif)

## <a name="next-steps"></a>Étapes suivantes

* [Explorer des travaux Azure Stream Analytics avec Visual Studio Code (préversion)](visual-studio-code-explore-jobs.md)

* [Configurer des pipelines CI/CD à l'aide du package npm](./cicd-overview.md)
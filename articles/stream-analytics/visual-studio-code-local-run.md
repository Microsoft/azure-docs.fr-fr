---
title: Tester un travail Azure Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code
description: Cet article décrit comment tester des requêtes localement avec des exemples de données à l’aide d’Azure Stream Analytics Tools pour Visual Studio Code.
ms.service: stream-analytics
author: su-jie
ms.author: sujie
ms.date: 11/10/2019
ms.topic: how-to
ms.openlocfilehash: 0122e67714efb201584a52f1cb763528dbb0b925
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90903801"
---
# <a name="test-stream-analytics-queries-locally-with-sample-data-using-visual-studio-code"></a>Tester des requêtes Stream Analytics localement avec des exemples de données à l’aide de Visual Studio Code

Vous pouvez utiliser Azure Stream Analytics Tools pour Visual Studio Code afin de tester vos travaux Stream Analytics localement à l’aide d’exemples de données. Vous pouvez trouver les résultats de la requête dans des fichiers JSON dans le dossier **LocalRunOutputs** de votre projet.

## <a name="prerequisites"></a>Prérequis

* Installez le [SDK .NET Core](https://dotnet.microsoft.com/download) et redémarrez Visual Studio Code.

* Utilisez ce [démarrage rapide](quick-create-visual-studio-code.md) pour apprendre à créer un travail Stream Analytics à l’aide de Visual Studio Code.

## <a name="prepare-sample-data"></a>Préparer l’exemple de données

Vous devez d’abord préparer des exemples de fichiers de données d’entrée. Si vous avez déjà des exemples de fichiers de données sur votre ordinateur, vous pouvez ignorer cette étape et passer à l’étape suivante.

1. Cliquez sur **Aperçu des données** dans votre fichier de configuration d’entrée à partir de la ligne supérieure. Certaines données d’entrée sont extraites d’IoT Hub et affichées dans la fenêtre d’aperçu. Notez que cette opération peut durer un moment.

2. Une fois les données affichées, cliquez sur **Enregistrer sous** pour les enregistrer dans un fichier local.

 ![Aperçu de l’entrée en direct](./media/quick-create-visual-studio-code/preview-live-input.png)

## <a name="define-a-local-input"></a>Définir une entrée locale

1. Cliquez sur **input.json** sous le dossier Entrées dans votre projet Stream Analytics. Sélectionnez ensuite **Ajouter une entrée locale** à partir de la ligne supérieure.

    ![Ajouter une entrée locale à partir d’un projet](./media/quick-create-visual-studio-code/add-input-from-project.png)

    Vous pouvez également utiliser **Ctrl+Maj+P** pour ouvrir la palette de commandes, puis entrer **ASA : Ajouter une entrée**.

   ![Ajouter une entrée Stream Analytics dans VS Code](./media/quick-create-visual-studio-code/add-input.png)

2. Sélectionnez **Entrée locale**.

    ![Ajouter une entrée locale ASA dans Visual Studio Code](./media/vscode-local-run/add-local-input.png)

3. Sélectionnez **+ New Local Input (Nouvelle entrée locale)** .

    ![Ajouter une entrée locale ASA dans Visual Studio Code](./media/vscode-local-run/add-new-local-input.png)

4. Entrez le même alias d’entrée que pour votre requête.

    ![Ajouter un alias d’entrée locale ASA](./media/vscode-local-run/new-local-input-alias.png)

5. Dans le fichier **LocalInput_Input.json**, entrez le chemin d’accès du fichier où se trouve votre fichier de données local.

    ![Entrez le chemin d’accès du fichier local dans Visual Studio](./media/vscode-local-run/local-file-path.png)

6. Sélectionnez **Aperçu des données** pour afficher un aperçu des données d’entrée. Le type de sérialisation de vos données est automatiquement détecté (JSON ou CSV). Utilisez le sélecteur pour afficher vos données au format **Tableau** ou **Brut**. Voici un exemple de données au **format tableau** :

     ![Prévisualiser les données locales dans un format de table](./media/vscode-local-run/local-file-preview-table.png)

    Voici un exemple de données au **format brut** :

    ![Prévisualiser les données locales dans un format brut](./media/vscode-local-run/local-file-preview-raw.png)

## <a name="run-queries-locally"></a>Exécuter des requêtes localement

Revenez à votre éditeur de requête, puis sélectionnez **Exécuter localement**. Sélectionnez ensuite **Utiliser l’entrée locale** dans la liste déroulante.

![Dans l’éditeur de requête, sélectionnez Exécuter localement](./media/vscode-local-run/run-locally.png)

![Utiliser l’entrée locale](./media/vscode-local-run/run-locally-use-local-input.png)

Le résultat s’affiche dans la fenêtre de droite. Vous pouvez cliquer sur **Exécuter** pour effectuer un nouveau test. Vous pouvez également sélectionner **Ouvrir dans le dossier** pour afficher les fichiers de résultats dans l’Explorateur de fichiers et les ouvrir avec d’autres outils. Notez que les fichiers de résultats sont uniquement disponibles au format JSON.

![Afficher le résultat de l’exécution locale](./media/vscode-local-run/run-locally-result.png)

## <a name="next-steps"></a>Étapes suivantes

* [Tester des travaux Azure Stream Analytics localement avec une entrée en direct à l’aide de Visual Studio Code](visual-studio-code-local-run-live-input.md)

* [Explorer des travaux Azure Stream Analytics avec Visual Studio Code (préversion)](visual-studio-code-explore-jobs.md)

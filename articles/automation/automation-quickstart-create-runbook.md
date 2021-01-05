---
title: Démarrage rapide Azure - Créer un runbook Azure Automation | Microsoft Docs
description: Cet article vous aide à créer un runbook Azure Automation.
services: automation
ms.date: 02/05/2019
ms.topic: quickstart
ms.subservice: process-automation
ms.custom: mvc
ms.openlocfilehash: 948ca820347c7cdcd560ade46e850f66b25bc88e
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90987296"
---
# <a name="create-an-azure-automation-runbook"></a>Créer un runbook Azure Automation

Les runbooks Azure Automation peuvent être créés via Azure. Cette méthode fournit une interface utilisateur basée sur un navigateur pour créer des runbooks Automation. Dans ce démarrage rapide, vous passez au travers de la création, de la modification, du test et de la publication d’un runbook Automation PowerShell.

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure sur https://portal.azure.com.

## <a name="create-the-runbook"></a>Créer le runbook

Tout d’abord, créez un runbook. L’exemple de runbook créé dans ce démarrage rapide génère `Hello World` par défaut.

1. Ouvrez votre compte Automation.

1. Cliquez sur **Runbooks** sous **Automatisation de processus**. La liste des runbooks s’affiche.

1. Cliquez sur **Créer un runbook** en haut de la liste.

1. Entrez `Hello-World` comme nom du runbook dans le champ **Nom**, puis sélectionnez **PowerShell** pour le champ **Type de runbook**. 

   ![Saisir les informations relatives à votre runbook Automation dans la page](./media/automation-quickstart-create-runbook/automation-create-runbook-configure.png)

1. Cliquez sur **Créer**. Le runbook est créé et la page Modifier un runbook PowerShell s’ouvre.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-edit-runbook-empty.png" alt-text="Capture d’écran de la page Modifier le runbook PowerShell.":::

1. Saisissez ou bien copiez et collez le code suivant dans le volet d’édition. Il crée un paramètre d’entrée facultatif nommé `Name` avec `World` comme valeur par défaut et il génère une chaîne qui utilise cette valeur d’entrée :

   ```powershell-interactive
   param
   (
       [Parameter(Mandatory=$false)]
       [String] $Name = "World"
   )

   "Hello $Name!"
   ```

1. Cliquez sur **Enregistrer** pour enregistrer un brouillon du runbook.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-edit-runbook.png" alt-text="Capture d’écran de la page Modifier le runbook PowerShell avec un exemple de code dans la fenêtre de droite.":::

## <a name="test-the-runbook"></a>Tester le Runbook

Une fois le runbook créé, vous devez le tester pour vérifier qu’il fonctionne.

1. Cliquez sur **Volet de test** pour ouvrir le volet de test.

1. Saisissez une valeur comme **nom**, puis cliquez sur **Démarrer**. Le travail de test démarre et l’état du travail ainsi que la sortie s’affichent.

    :::image type="content" source="./media/automation-quickstart-create-runbook/automation-test-runbook.png" alt-text="Capture d’écran du volet Test avec un exemple de valeur dans le champ Nom.":::

1. Fermez le volet de test en cliquant sur le symbole **X** dans le coin supérieur droit. Sélectionnez **OK** dans le menu contextuel qui s’affiche.

1. Dans la page Modifier un runbook PowerShell, cliquez sur **Publier** pour publier le runbook en tant que version officielle du runbook dans le compte.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-job.png" alt-text="Capture d’écran de la page Modifier le runbook PowerShell avec le bouton Publier sélectionné.":::

## <a name="run-the-runbook"></a>Exécuter le runbook

Une fois le runbook publié, la page de vue d’ensemble apparaît.

1. Dans la page de vue d’ensemble du runbook, cliquez sur **Démarrer** pour ouvrir la page de configuration Démarrer le runbook pour ce runbook.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-start.png" alt-text="Capture d’écran de la page Démarrer la configuration d’un runbook.":::

1. Laissez le **nom** vide afin que la valeur par défaut soit utilisée, puis cliquez sur **OK**. Le travail de runbook est soumis et la page du travail s’affiche.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-job-page.png" alt-text="Capture d’écran de la page Travail montrant le bouton Sortie sélectionné.":::

1. Lorsque l’état du travail est `Running` ou `Completed`, cliquez sur **Sortie** pour ouvrir le volet Sortie et afficher la sortie du runbook.

   :::image type="content" source="./media/automation-quickstart-create-runbook/automation-hello-world-runbook-job-output.png" alt-text="Capture d’écran du volet Sortie montrant la sortie du runbook.":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsqu’il n’est plus nécessaire, supprimez le runbook. Pour le faire, sélectionnez le runbook dans la liste de runbooks, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez créé, modifié, testé et publié un runbook et vous avez démarré un travail de runbook. Pour en savoir plus sur les runbooks Automation, consultez l’article traitant des différents types de runbooks que vous pouvez créer et utiliser dans Automation.

> [!div class="nextstepaction"]
> [Types de Runbooks Azure Automation](./automation-runbook-types.md)

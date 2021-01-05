---
title: Modifier des runbooks textuels dans Azure Automation
description: Cet article explique comment utiliser l’éditeur de texte d’Azure Automation pour travailler avec runbooks PowerShell et des runbooks de Flux de travail PowerShell.
services: automation
ms.service: automation
ms.subservice: process-automation
author: mgoedtel
ms.author: magoedte
ms.date: 08/01/2018
ms.topic: conceptual
manager: carmonm
ms.openlocfilehash: c51ef23e27cd63d3706c104d1e39a14bf61c258e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86186348"
---
# <a name="edit-textual-runbooks-in-azure-automation"></a>Modifier des runbooks textuels dans Azure Automation

L’éditeur de texte dans Azure Automation permet de modifier des [Runbooks PowerShell](automation-runbook-types.md#powershell-runbooks) et des [Runbooks de flux de travail PowerShell](automation-runbook-types.md#powershell-workflow-runbooks). Cet éditeur offre les caractéristiques standard des autres éditeurs de code, par exemple IntelliSense. Il utilise également un code couleur assorti de fonctionnalités spéciales supplémentaires pour vous aider à accéder à des ressources communes aux runbooks. 

L’éditeur de texte inclut une fonctionnalité permettant d’insérer du code pour les cmdlets, les ressources et les runbooks enfants dans un runbook. Plutôt que de taper le code vous-même, vous pouvez sélectionner le code approprié dans la liste des ressources disponibles, et l’éditeur l’insère dans le runbook.

Dans Azure Automation, chaque runbook existe en deux versions : un brouillon et une version publiée. Vous devez modifier le brouillon du runbook, puis le publier afin qu’il puisse être exécuté. La version publiée ne peut pas être modifiée. Pour plus d’informations, consultez [Publier un runbook](manage-runbooks.md#publish-a-runbook).

Cet article fournit des instructions détaillées pour effectuer différentes fonctions avec cet éditeur. Elles ne s’appliquent pas aux [runbooks graphiques](automation-runbook-types.md#graphical-runbooks). Pour utiliser ces runbooks, consultez [Création de graphiques dans Azure Automation](automation-graphical-authoring-intro.md).

## <a name="edit-a-runbook-with-the-azure-portal"></a>Pour modifier un runbook via le portail Azure

1. Dans le portail Azure, sélectionnez votre compte Automation.
2. Sous **AUTOMATISATION DE PROCESSUS**, cliquez sur **Runbooks** pour ouvrir la liste des runbooks.
3. Choisissez le runbook à modifier, puis cliquez sur **Modifier**.
4. Modifiez le runbook.
5. Cliquez sur **Enregistrer** lorsque vos modifications sont terminées.
6. Cliquez sur **Publier** si vous souhaitez que le dernier brouillon du runbook soit publié.

### <a name="insert-a-cmdlet-into-a-runbook"></a>Insérer une applet de commande dans un runbook

1. Dans le canevas de l’éditeur de texte, placez le curseur à l’emplacement où vous souhaitez placer l’applet de commande.
2. Développez le nœud **Applets de commande** dans le contrôle Bibliothèque.
3. Développez le module contenant l’applet de commande à utiliser.
4. Cliquez avec le bouton droit sur le nom de la cmdlet à insérer, puis sélectionnez **Ajouter au canevas**. Si plusieurs jeux de paramètres sont définis pour l’applet de commande, l’éditeur ajoute le jeu par défaut. Vous pouvez également développer l’applet de commande pour sélectionner un jeu de paramètres différent.
5. Notez que le code de l’applet de commande est inséré avec sa liste entière de paramètres.
6. Indiquez une valeur appropriée à la place de la valeur entourée par des crochets (<>) pour tous les paramètres requis. Supprimez les paramètres dont vous n’avez pas besoin.

### <a name="insert-code-for-a-child-runbook-into-a-runbook"></a>Insérer du code pour un runbook enfant dans un runbook

1. Dans le canevas de l’éditeur de texte, placez le curseur à l’emplacement où vous souhaitez placer le code pour le [runbook enfant](automation-child-runbooks.md).
2. Développez le nœud **Runbooks** dans le contrôle Bibliothèque.
3. Cliquez avec le bouton droit sur le runbook à insérer, puis sélectionnez **Ajouter au canevas**.
4. Le code du runbook enfant est inséré avec des espaces réservés pour les paramètres des runbooks.
5. Remplacez les espaces réservés par des valeurs appropriées pour chaque paramètre.

### <a name="insert-an-asset-into-a-runbook"></a>Insérer une ressource dans un runbook

1. Dans le contrôle Canevas de l’éditeur de texte, positionnez le curseur à l’emplacement où vous souhaitez placer le code pour le runbook enfant.
2. Développez le nœud **Ressources** dans le contrôle Bibliothèque.
3. Développez le nœud du type de ressource souhaité.
4. Cliquez avec le bouton droit sur le nom de la ressource à insérer, puis sélectionnez **Ajouter au canevas**. Pour les [ressources variables](./shared-resources/variables.md), sélectionnez **Ajouter « Obtenir la variable » au canevas** ou **Ajouter « Définir la variable » au canevas** selon que vous souhaitez obtenir ou définir la variable.
5. Notez que le code de la ressource est inséré dans le runbook.

## <a name="edit-an-azure-automation-runbook-using-windows-powershell"></a>Modifier un runbook Azure Automation à l’aide de Windows PowerShell

Pour modifier un runbook avec Windows PowerShell, utilisez l’éditeur de votre choix et enregistrez le runbook dans un fichier **.ps1**. Vous pouvez utiliser l’applet de commande [Export-AzAutomationRunbook](/powershell/module/Az.Automation/Export-AzAutomationRunbook) pour récupérer le contenu du runbook. Vous pouvez utiliser l’applet de commande [Import-AzAutomationRunbook](/powershell/module/Az.Automation/import-azautomationrunbook) pour remplacer le brouillon de runbook existant par le runbook modifié.

### <a name="retrieve-the-contents-of-a-runbook-using-windows-powershell"></a>Récupérer le contenu d’un runbook à l’aide de Windows PowerShell

Les exemples de commandes suivants montrent comment récupérer le script d’un runbook et l’enregistrer dans un fichier de script. Dans cet exemple, le brouillon est récupéré. Il est également possible de récupérer la version publiée du runbook, bien que cette version ne puisse pas être modifiée.

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Export-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -OutputFolder $scriptFolder -Slot Draft
```

### <a name="change-the-contents-of-a-runbook-using-windows-powershell"></a>Modifier le contenu d’un runbook à l’aide de Windows PowerShell

Les exemples de commandes suivants montrent comment remplacer le contenu existant d’un runbook par le contenu d’un fichier de script. 

```powershell-interactive
$resourceGroupName = "MyResourceGroup"
$automationAccountName = "MyAutomatonAccount"
$runbookName = "Hello-World"
$scriptFolder = "c:\runbooks"

Import-AzAutomationRunbook -Path "$scriptfolder\Hello-World.ps1" -Name $runbookName -Type PowerShell -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName -Force
Publish-AzAutomationRunbook -Name $runbookName -AutomationAccountName $automationAccountName -ResourceGroupName $resourceGroupName
```

## <a name="next-steps"></a>Étapes suivantes

* [Gérer les runbooks dans Azure Automation](manage-runbooks.md).
* [Apprentissage du flux de travail PowerShell](automation-powershell-workflow.md).
* [Création graphique dans Azure Automation](automation-graphical-authoring-intro.md).
* [Certificats](./shared-resources/certificates.md) :
* [Connexions](automation-connections.md).
* [Informations d’identification](./shared-resources/credentials.md).
* [Planifications](./shared-resources/schedules.md).
* [Variables](./shared-resources/variables.md).
* [Référence sur les cmdlets PowerShell](/powershell/module/az.automation/?view=azps-3.7.0#automation).

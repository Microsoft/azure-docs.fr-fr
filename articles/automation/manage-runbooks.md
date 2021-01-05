---
title: Gérer les runbooks dans Azure Automation
description: Cet article explique comment gérer des runbooks dans Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 10/23/2020
ms.topic: conceptual
ms.openlocfilehash: ed5eda668f6bd52ba144aa664119ab613fdb7742
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183581"
---
# <a name="manage-runbooks-in-azure-automation"></a>Gérer les runbooks dans Azure Automation

Vous pouvez ajouter un runbook à Azure Automation en en créant un ou en en important un existant à partir d’un fichier ou de la [galerie de runbooks](automation-runbook-gallery.md). Cet article fournit des informations sur la gestion d’un runbook importé à partir d’un fichier. Vous trouverez toutes les informations sur l’accès aux runbooks et modules communautaires dans [Galeries de runbooks et de modules pour Azure Automation](automation-runbook-gallery.md).

## <a name="create-a-runbook"></a>Créer un runbook

Créez un runbook dans Azure Automation à l’aide du portail Azure ou de Windows PowerShell. Une fois le runbook créé, vous pouvez le modifier à l'aide des informations contenues dans :

* [Modifier un runbook textuel dans Azure Automation](automation-edit-textual-runbook.md)
* [Découvrir les principaux concepts de workflow Windows PowerShell pour les runbooks Automation](automation-powershell-workflow.md)
* [Gérer des packages Python 2 dans Azure Automation](python-packages.md)

### <a name="create-a-runbook-in-the-azure-portal"></a>Créer un runbook dans le portail Azure

1. Dans le portail Azure, ouvrez votre compte Automation.
2. À partir du hub, sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.
3. Cliquez sur **Créer un runbook**.
4. Entrez un nom pour le runbook et sélectionnez son [type](automation-runbook-types.md). Le nom du runbook doit commencer par une lettre et peut contenir des lettres, des chiffres, des traits de soulignement et des tirets.
5. Cliquez sur **Créer** pour créer le runbook et ouvrez l’éditeur.

### <a name="create-a-runbook-with-powershell"></a>Créer un Runbook avec PowerShell

Utilisez l’applet de commande [New-AzAutomationRunbook](/powershell/module/az.automation/new-azautomationrunbook) pour créer un runbook vide. Utilisez le paramètre `Type` pour spécifier l’un des types de runbook définis pour `New-AzAutomationRunbook`.

L’exemple suivant montre comment créer un runbook vide.

```azurepowershell-interactive
New-AzAutomationRunbook -AutomationAccountName MyAccount `
-Name NewRunbook -ResourceGroupName MyResourceGroup -Type PowerShell
```

## <a name="import-a-runbook"></a>Importer un runbook

Vous pouvez importer un script PowerShell ou un workflow PowerShell ( **.ps1**), un runbook graphique ( **.graphrunbook**) ou un script Python 2 ( **.py**) pour créer votre propre runbook. Vous devez spécifier le [type de runbook](automation-runbook-types.md) qui sera créé à lors de l’importation et ce, en tenant compte des considérations suivantes.

* Vous pouvez importer un fichier **.ps1** qui ne contient pas de workflow dans un [runbook PowerShell](automation-runbook-types.md#powershell-runbooks) ou un [runbook PowerShell Workflow](automation-runbook-types.md#powershell-workflow-runbooks). Si vous l’importez dans un runbook PowerShell Workflow, il est converti en workflow. Dans ce cas, les commentaires sont inclus dans le runbook pour décrire les modifications apportées.

* Vous ne pouvez importer un fichier **.ps1** contenant un workflow PowerShell que dans un [runbook PowerShell Workflow](automation-runbook-types.md#powershell-workflow-runbooks). Si le fichier contient plusieurs workflows PowerShell, l’importation échoue. Vous devez enregistrer chaque workflow dans son propre fichier, puis les importer séparément.

* N'importez pas de fichier **.ps1** contenant un workflow PowerShell dans un [runbook PowerShell](automation-runbook-types.md#powershell-runbooks) car le moteur de script PowerShell ne peut pas le reconnaître.

* Importez un fichier **.graphrunbook** que dans un nouveau [runbook graphique](automation-runbook-types.md#graphical-runbooks).

### <a name="import-a-runbook-from-the-azure-portal"></a>Importer un runbook à partir du portail Azure

Vous pouvez utiliser la procédure suivante pour importer un fichier de script dans Azure Automation.

> [!NOTE]
> Vous pouvez uniquement importer un fichier **.ps1** dans un runbook PowerShell Workflow à l’aide du portail.

1. Dans le portail Azure, ouvrez votre compte Automation.
2. Sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.
3. Cliquez sur **Importer un runbook**.
4. Cliquez sur **Fichier runbook** et sélectionnez le fichier à importer.
5. Si le champ **Nom** est activé, vous avez la possibilité de modifier le nom du runbook. Celui-ci doit commencer par une lettre et peut contenir des lettres, des chiffres, des traits de soulignement et des tirets.
6. Le [type de runbook](automation-runbook-types.md) est sélectionné automatiquement, mais vous pouvez le modifier après avoir pris en compte les restrictions applicables.
7. Cliquez sur **Créer**. Le nouveau runbook apparaît dans la liste des runbooks pour le compte Automation.
8. Vous devez [Publier le runbook](#publish-a-runbook) avant de pouvoir l’exécuter.

> [!NOTE]
> Après avoir importé un runbook graphique, vous pouvez en convertir le type. Toutefois, vous ne pouvez pas convertir un runbook graphique en runbook textuel.

### <a name="import-a-runbook-with-windows-powershell"></a>Importer un runbook avec Windows PowerShell

Utilisez l’applet de commande [Import-AzAutomationRunbook](/powershell/module/az.automation/import-azautomationrunbook) pour importer un fichier de script en tant que brouillon de runbook. Si le runbook existe déjà, l’importation échoue, sauf si vous utilisez le paramètre `Force` avec l’applet de commande.

L’exemple suivant montre comment importer un fichier de script dans un runbook.

```azurepowershell-interactive
$automationAccountName =  "AutomationAccount"
$runbookName = "Sample_TestRunbook"
$scriptPath = "C:\Runbooks\Sample_TestRunbook.ps1"
$RGName = "ResourceGroup"

Import-AzAutomationRunbook -Name $runbookName -Path $scriptPath `
-ResourceGroupName $RGName -AutomationAccountName $automationAccountName `
-Type PowerShellWorkflow
```

## <a name="handle-resources"></a>Gérer les ressources

Si votre runbook crée une [ressource](automation-runbook-execution.md#resources), le script doit vérifier qu’elle n’existe pas déjà avant d’essayer de la créer. Voici un exemple simple.

```powershell
$vmName = "WindowsVM1"
$resourceGroupName = "myResourceGroup"
$myCred = Get-AutomationPSCredential "MyCredential"
$vmExists = Get-AzResource -Name $vmName -ResourceGroupName $resourceGroupName

if(!$vmExists)
    {
    Write-Output "VM $vmName does not exist, creating"
    New-AzVM -Name $vmName -ResourceGroupName $resourceGroupName -Credential $myCred
    }
else
    {
    Write-Output "VM $vmName already exists, skipping"
    }
```

## <a name="retrieve-details-from-activity-log"></a>Récupérer les détails du journal d’activité

Vous pouvez récupérer les détails d’un runbook, comme le nom de la personne ou le compte qui a démarré le runbook, à partir du [journal d’activité](automation-runbook-execution.md#activity-logging) du compte Automation. L’exemple PowerShell suivant indique le dernier utilisateur à avoir exécuté le runbook spécifié.

```powershell-interactive
$SubID = "00000000-0000-0000-0000-000000000000"
$AutomationResourceGroupName = "MyResourceGroup"
$AutomationAccountName = "MyAutomationAccount"
$RunbookName = "MyRunbook"
$StartTime = (Get-Date).AddDays(-1)
$JobActivityLogs = Get-AzLog -ResourceGroupName $AutomationResourceGroupName -StartTime $StartTime `
                                | Where-Object {$_.Authorization.Action -eq "Microsoft.Automation/automationAccounts/jobs/write"}

$JobInfo = @{}
foreach ($log in $JobActivityLogs)
{
    # Get job resource
    $JobResource = Get-AzResource -ResourceId $log.ResourceId

    if ($JobInfo[$log.SubmissionTimestamp] -eq $null -and $JobResource.Properties.runbook.name -eq $RunbookName)
    {
        # Get runbook
        $Runbook = Get-AzAutomationJob -ResourceGroupName $AutomationResourceGroupName -AutomationAccountName $AutomationAccountName `
                                            -Id $JobResource.Properties.jobId | ? {$_.RunbookName -eq $RunbookName}

        # Add job information to hashtable
        $JobInfo.Add($log.SubmissionTimestamp, @($Runbook.RunbookName,$Log.Caller, $JobResource.Properties.jobId))
    }
}
$JobInfo.GetEnumerator() | sort key -Descending | Select-Object -First 1
```

## <a name="track-progress"></a>Suivre la progression

Il est recommandé de créer des runbooks de nature modulaire, avec une logique pouvant être réutilisée et redémarrée facilement. Le suivi de la progression d’un runbook garantit que la logique du runbook s’exécute correctement en cas de problème.

Vous pouvez suivre la progression d’un runbook en utilisant une source externe, notamment un compte de stockage, une base de données ou des fichiers partagés. Créez une logique dans votre runbook qui vérifie dans un premier temps l’état de la dernière action effectuée. Ensuite, selon les résultats de la vérification, la logique peut ignorer ou poursuivre certaines tâches du runbook.

## <a name="prevent-concurrent-jobs"></a>Prévention des travaux simultanés

Certains runbooks peuvent se comporter bizarrement quand ils exécutent plusieurs tâches en même temps. Dans ce cas, il est important qu’un runbook implémente une logique qui puisse déterminer si une tâche est déjà en cours d’exécution. Voici un exemple simple.

```powershell
# Authenticate to Azure
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationId $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint

$AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

# Check for already running or new runbooks
$runbookName = "<RunbookName>"
$rgName = "<ResourceGroupName>"
$aaName = "<AutomationAccountName>"
$jobs = Get-AzAutomationJob -ResourceGroupName $rgName -AutomationAccountName $aaName -RunbookName $runbookName -AzContext $AzureContext

# Check to see if it is already running
$runningCount = ($jobs | ? {$_.Status -eq "Running"}).count

If (($jobs.status -contains "Running" -And $runningCount -gt 1 ) -Or ($jobs.Status -eq "New")) {
    # Exit code
    Write-Output "Runbook is already running"
    Exit 1
} else {
    # Insert Your code here
}
```

## <a name="handle-transient-errors-in-a-time-dependent-script"></a>Gérer les erreurs temporaires dans un script dépendant de l’heure

Vos runbooks doivent être robustes et capables de gérer les [erreurs](automation-runbook-execution.md#errors) temporaires, notamment les erreurs temporaires qui peuvent entraîner leur redémarrage ou une défaillance. Si un runbook rencontre une défaillance, Azure Automation effectue un nouvel essai.

Si votre runbook s’exécute normalement dans les limites d’une contrainte de temps, faites en sorte que le script implémente la logique pour vérifier la durée d’exécution. Cette vérification contrôle la bonne exécution de certaines opérations comme le démarrage, l’arrêt ou le scale-out, uniquement à des moment précis.

> [!NOTE]
> L’heure locale du processus de bac à sable Azure est définie sur le temps universel coordonné (UTC). Les calculs de date et d’heure dans vos runbooks doivent prendre cet élément en considération.

## <a name="work-with-multiple-subscriptions"></a>Utilisation de plusieurs abonnements

Votre runbook doit pouvoir utiliser des [abonnements](automation-runbook-execution.md#subscriptions). Par exemple, pour gérer plusieurs abonnements, le runbook utilise la cmdlet [Disable-AzContextAutosave](/powershell/module/Az.Accounts/Disable-AzContextAutosave). Cette cmdlet permet de s'assurer que le contexte d’authentification n’est pas récupéré à partir d’un autre runbook s'exécutant dans le même bac à sable. Le runbook utilise également l’applet de commande `Get-AzContext` pour récupérer le contexte de la session active et l’assigner à la variable `$AzureContext`.

```powershell
Disable-AzContextAutosave -Scope Process

$Conn = Get-AutomationConnection -Name AzureRunAsConnection
$AzureContext = Connect-AzAccount -ServicePrincipal `
-Tenant $Conn.TenantID `
-ApplicationId $Conn.ApplicationID `
-CertificateThumbprint $Conn.CertificateThumbprint `
-Subscription $Conn.SubscriptionId

$ChildRunbookName = 'ChildRunbookDemo'
$AutomationAccountName = 'myAutomationAccount'
$ResourceGroupName = 'myResourceGroup'

Start-AzAutomationRunbook `
-ResourceGroupName $ResourceGroupName `
-AutomationAccountName $AutomationAccountName `
-Name $ChildRunbookName `
-DefaultProfile $AzureContext
```

## <a name="work-with-a-custom-script"></a>Utilisation d’un script personnalisé

> [!NOTE]
> Normalement, vous ne pouvez pas exécuter des scripts personnalisés et des runbooks sur l’hôte sur lequel est installé un agent Log Analytics.

Pour utiliser un script personnalisé :

1. Créez un compte Automation et obtenez un [rôle Contributeur](automation-role-based-access-control.md).
2. [Liez le compte à l’espace de travail Azure](../security-center/security-center-enable-data-collection.md).
3. Activez [Runbook Worker hybride](automation-hybrid-runbook-worker.md), [Update Management](./update-management/overview.md) ou une autre fonctionnalité Automation. 
4. Sur un ordinateur Linux, vous avez besoin d’autorisations élevées. Connectez-vous pour [désactiver les vérifications de signature](automation-linux-hrw-install.md#turn-off-signature-validation).

## <a name="test-a-runbook"></a>Tester un runbook

Lorsque vous testez un runbook, la [version Brouillon](#publish-a-runbook) est exécutée et toutes les actions qu’il effectue sont finalisées. Aucun historique des tâches n’est créé, mais les flux [Résultat](automation-runbook-output-and-messages.md#use-the-output-stream) et [Avertissement et Erreur](automation-runbook-output-and-messages.md#working-with-message-streams) s’affichent dans le volet de sortie du test. Les messages destinés au [flux de commentaires](automation-runbook-output-and-messages.md#write-output-to-verbose-stream) ne sont affichés dans le volet des résultats que si la variable [VerbosePreference](automation-runbook-output-and-messages.md#work-with-preference-variables) est définie sur `Continue`.

Bien que la version brouillon soit exécutée, le runbook s'exécute normalement et effectue des actions sur des ressources dans l’environnement. Pour cette raison, vous devez tester les runbooks uniquement sur des ressources hors production.

La procédure de test de chaque [type de runbook](automation-runbook-types.md) est identique. Il n’y a aucune différence de test entre l’éditeur de texte et l’éditeur graphique du portail Azure.

1. Ouvrez la version Brouillon du runbook dans l’[éditeur de texte](automation-edit-textual-runbook.md) ou l’[éditeur graphique](automation-graphical-authoring-intro.md).
1. Cliquez sur **Tester** pour ouvrir la page de test.
1. Si le runbook a des paramètres, ils figurent dans le volet gauche dans lequel vous pouvez fournir des valeurs à utiliser pour le test.
1. Si vous souhaitez exécuter le test sur un [runbook Worker hybride](automation-hybrid-runbook-worker.md), définissez les **Paramètres d’exécution** sur **Worker hybride** et sélectionnez le nom du groupe cible.  Dans le cas contraire, conservez la valeur par défaut **Azure** pour exécuter le test dans le cloud.
1. Cliquez sur **Démarrer** pour commencer le test.
1. Vous pouvez utiliser les boutons situés sous le volet de sortie pour arrêter ou suspendre un runbook [PowerShell Workflow](automation-runbook-types.md#powershell-workflow-runbooks) ou [graphique](automation-runbook-types.md#graphical-runbooks) pendant son test. Lorsque vous interrompez le runbook, il termine l’activité en cours avant de s’interrompre. Lorsque le runbook est suspendu, vous pouvez l’arrêter ou le redémarrer.
1. Inspectez la sortie du runbook dans le volet de sortie.

## <a name="publish-a-runbook"></a>Publier un runbook

Lorsque vous créez ou importez un runbook, vous devez le publier avant de pouvoir l’exécuter. Dans Azure Automation, chaque runbook a une version brouillon et une version publiée. Seule la version publiée est disponible à l'exécution, et seul le brouillon peut être modifié. La version publiée n'est pas affectée par les modifications apportées à la version Bouillon. Quand la version brouillon doit être rendue disponible, vous la publiez, ce qui remplace la version publiée actuelle par la version brouillon.

### <a name="publish-a-runbook-in-the-azure-portal"></a>Publier un runbook dans le portail Azure

1. Ouvrez le runbook dans le portail Azure.
2. Cliquez sur **Modifier**.
3. Cliquez sur **Publier**, puis sur **Oui** en réponse au message de vérification.

### <a name="publish-a-runbook-using-powershell"></a>Publier un runbook à l’aide de PowerShell

Utilisez l’applet de commande [Publish-AzAutomationRunbook](/powershell/module/Az.Automation/Publish-AzAutomationRunbook) pour publier votre runbook. 

```azurepowershell-interactive
$automationAccountName =  "AutomationAccount"
$runbookName = "Sample_TestRunbook"
$RGName = "ResourceGroup"

Publish-AzAutomationRunbook -AutomationAccountName $automationAccountName `
-Name $runbookName -ResourceGroupName $RGName
```

## <a name="schedule-a-runbook-in-the-azure-portal"></a>Planifier un runbook dans le portail Azure

Une fois votre runbook publié, vous pouvez le planifier en vue de l'utiliser :

1. Ouvrez le runbook dans le portail Azure.
2. Sélectionnez **Planifications** sous **Ressources**.
3. Sélectionnez **Ajouter une planification**.
4. Dans le volet Planifier le runbook, sélectionnez **Associer une planification à votre runbook**.
5. Choisissez **Créer une planification** dans le volet Planification.
6. Entrez un nom, une description et d'autres paramètres dans le volet Nouvelle planification.
7. Une fois la planification créée, mettez-la en surbrillance et cliquez sur **OK**. Elle doit maintenant être associée à votre runbook.
8. Accédez à votre boîte aux lettres et recherchez-y un e-mail qui vous informe de l'état du runbook.

## <a name="obtain-job-statuses"></a>Obtenir les états des tâches

### <a name="view-statuses-in-the-azure-portal"></a>Afficher les états sur le portail Azure

Les détails de la gestion des tâches dans Azure Automation apparaissent dans la section [Tâches](automation-runbook-execution.md#jobs). Lorsque vous êtes prêt à afficher vos tâches runbook, utilisez le portail Azure et accédez à votre compte Automation. À droite, vous pouvez voir un résumé de toutes les tâches runbook dans la section **Statistiques des tâches**.

![Vignette Statistiques des tâches](./media/manage-runbooks/automation-account-job-status-summary.png)

Le résumé affiche un nombre et une représentation graphique de l’état de chaque tâche exécutée.

Lorsque vous cliquez sur la vignette, la page Tâches s’affiche avec un récapitulatif de toutes les tâches exécutées. Cette page indique l’état, le nom du runbook, l’heure de début et l’heure de fin de chaque tâche.

:::image type="content" source="./media/manage-runbooks/automation-account-jobs-status-blade.png" alt-text="Capture d’écran de la page Tâches.":::

Vous pouvez filtrer la liste des tâches en sélectionnant **Filtrer les tâches**. Filtrez en fonction d’un runbook spécifique, d’un état de tâche ou d’un choix dans la liste déroulante, puis indiquez un intervalle de temps pour la recherche.

![Filtrer en fonction de l’état des tâches](./media/manage-runbooks/automation-account-jobs-filter.png)

Vous pouvez aussi afficher un résumé détaillé des tâches d’un runbook en sélectionnant le runbook dans la page Runbooks de votre compte Automation, puis en sélectionnant **Tâches**. Cette action affiche la page Tâches. De là, vous pouvez cliquer sur un enregistrement de tâche pour en afficher les détails et la sortie.

:::image type="content" source="./media/manage-runbooks/automation-runbook-job-summary-blade.png" alt-text="Capture d’écran de la page Tâches avec le bouton Erreurs mis en surbrillance.":::

### <a name="retrieve-job-statuses-using-powershell"></a>Récupérer des états de tâches avec PowerShell

Utilisez l’applet de commande [Get-AzureAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob) pour récupérer les tâches créées pour un runbook ainsi que les détails d’une tâche particulière. Si vous démarrez un runbook à l’aide de `Start-AzAutomationRunbook`, la tâche obtenue est retournée. Utilisez [AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput) pour récupérer la sortie de la tâche.

Les exemples suivants obtiennent la dernière tâche d’un exemple de runbook et affichent son état, les valeurs définies pour les paramètres du runbook et la sortie de la tâche.

```azurepowershell-interactive
$job = (Get-AzAutomationJob –AutomationAccountName "MyAutomationAccount" `
–RunbookName "Test-Runbook" -ResourceGroupName "ResourceGroup01" | sort LastModifiedDate –desc)[0]
$job.Status
$job.JobParameters
Get-AzAutomationJobOutput -ResourceGroupName "ResourceGroup01" `
–AutomationAccountName "MyAutomationAcct" -Id $job.JobId –Stream Output
```

L’exemple suivant récupère la sortie d’une tâche spécifique et retourne chaque enregistrement. Si une [exception](automation-runbook-execution.md#exceptions) se produit pour l’un des enregistrements, le script écrit l’exception à la place de la valeur. Ce comportement est utile, car les exceptions peuvent fournir des informations supplémentaires qui ne sont pas nécessairement journalisées au moment de la sortie.

```azurepowershell-interactive
$output = Get-AzAutomationJobOutput -AutomationAccountName <AutomationAccountName> -Id <jobID> -ResourceGroupName <ResourceGroupName> -Stream "Any"
foreach($item in $output)
{
    $fullRecord = Get-AzAutomationJobOutputRecord -AutomationAccountName <AutomationAccountName> -ResourceGroupName <ResourceGroupName> -JobId <jobID> -Id $item.StreamRecordId
    if ($fullRecord.Type -eq "Error")
    {
        $fullRecord.Value.Exception
    }
    else
    {
    $fullRecord.Value
    }
}
```

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la gestion des runbooks, voir [Exécution d'un Runbook dans Azure Automation](automation-runbook-execution.md).
* Pour préparer un runbook PowerShell, consultez [Modifier des runbooks textuels dans Azure Automation](automation-edit-textual-runbook.md).
* Pour résoudre les problèmes liés à l’exécution de runbooks, voir [Résoudre les erreurs avec les runbooks](troubleshoot/runbooks.md).
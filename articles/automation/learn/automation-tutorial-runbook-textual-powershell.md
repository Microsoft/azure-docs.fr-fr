---
title: Créer un runbook PowerShell dans Azure Automation
description: Cet article vous apprend à créer, tester et publier un runbook PowerShell simple.
keywords: azure powershell, didacticiel sur le script powershell, automatisation powershell
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: a1b0dff9421f493958554c659043c49ff2874379
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87014998"
---
# <a name="tutorial-create-a-powershell-runbook"></a>Tutoriel : Créer un runbook PowerShell

Ce didacticiel vous guide dans la création d’un [Runbook PowerShell](../automation-runbook-types.md#powershell-runbooks) dans Azure Automation. Les Runbooks PowerShell sont basés sur Windows PowerShell. Vous pouvez modifier directement le code du runbook à l’aide de l’éditeur de texte du portail Azure.

> [!div class="checklist"]
> * Créer un runbook PowerShell simple
> * Tester et publier le runbook
> * Exécuter le travail du runbook et en suivre l’état
> * Mettre à jour le runbook pour démarrer une machine virtuelle Azure avec des paramètres de runbook

## <a name="prerequisites"></a>Prérequis

Pour réaliser ce didacticiel, vous avez besoin des éléments suivants :

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [compte Automation](../automation-quickstart-create-account.md) pour le stockage du Runbook et l’authentification auprès des ressources Azure. Ce compte doit avoir l’autorisation de démarrer et d’arrêter la machine virtuelle.
* Une machine virtuelle Azure. Comme vous allez devoir l’arrêter et la démarrer, il ne doit pas s’agir d’une machine virtuelle de production.
* Si nécessaire, [importez des modules Azure](../shared-resources/modules.md) ou [mettez à jour les modules](../automation-update-azure-modules.md) en fonction des applets de commande que vous utilisez.

## <a name="differences-from-powershell-workflow-runbooks"></a>Différences entre les runbooks PowerShell et les runbooks PowerShell Workflow

Les runbooks PowerShell ont les mêmes cycle de vie, fonctionnalités et mode de gestion que ceux de PowerShell Workflow. Toutefois, il existe certaines différences et limitations.

| Caractéristique  | Runbooks PowerShell | Runbooks PowerShell Workflow |
| ------ | ----- | ----- |
| Vitesse | S’exécutent rapidement, car ils n’utilisent pas d’étape de compilation. | S’exécutent moins rapidement. |
| Points de contrôle | Ne prennent pas en charge les points de contrôle. Les runbooks PowerShell peuvent uniquement reprendre les opérations depuis le début. | Utilisent des points de contrôle qui permettent à un classeur de reprendre une opération à partir de n’importe quel stade. |
| Exécution des commandes | Prennent uniquement en charge l’exécution en série. | Prennent en charge à la fois l’exécution en série et l’exécution parallèle.|
| Instance d'exécution | Une instance d’exécution exécute tout ce qui se trouve dans un script. | Une instance d’exécution distincte peut être utilisée pour une activité, une commande ou un bloc de script. |

En plus de ces différences, les runbooks PowerShell présentent des [différences syntaxiques](/previous-versions/technet-magazine/dn151046(v=msdn.10)) par rapport aux runbooks PowerShell Workflow.

## <a name="step-1---create-runbook"></a>Étape 1 - Création d’un Runbook

Commencez par créer un runbook simple qui renvoie le texte `Hello World`.

1. Dans le portail Azure, ouvrez votre compte Automation.

2. Sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.

3. Créez un runbook en sélectionnant **Créer un runbook**.

4. Nommez le Runbook **MyFirstRunbook-PowerShell**.

5. Dans ce cas, vous allez créer un [runbook PowerShell](../automation-runbook-types.md#powershell-runbooks). Pour **Type de runbook**, sélectionnez **PowerShell**.

6. Cliquez sur **Créer** pour créer le Runbook et ouvrez l’éditeur textuel.

## <a name="step-2---add-code-to-the-runbook"></a>Étape 2 - Ajouter du code au Runbook

Vous pouvez soit taper du code directement dans le Runbook, soit sélectionner des applets de commande, des Runbooks et des ressources à partir du contrôle Bibliothèque et les ajouter au Runbook avec tous les paramètres associés. Pour ce tutoriel, vous allez taper le code directement dans le runbook.

1. Votre runbook est actuellement vide. Tapez `Write-Output "Hello World"` dans le corps du script.

   ![Hello World](../media/automation-tutorial-runbook-textual-powershell/automation-helloworld.png)

2. Enregistrez le Runbook en cliquant sur **Enregistrer**.

## <a name="step-3---test-the-runbook"></a><a name="step-3---test-the-runbook"> </a> Étape 3 : Test du runbook

Avant de publier le runbook pour le rendre disponible en production, vous devez le tester pour vous assurer qu’il fonctionne correctement. Le test d’un runbook exécute sa version Brouillon et vous permet d’afficher sa sortie de manière interactive.

1. Cliquez sur **Volet de test** pour ouvrir le volet de test.

2. Cliquez sur **Démarrer** pour démarrer le test. Ce doit être la seule option activée.

3. Un [travail de runbook](../automation-runbook-execution.md) est créé, dont l’état apparaît dans le volet.

   L’état initial du travail est Mis en file d’attente pour indiquer qu’il attend qu’un Runbook Worker devienne disponible dans le cloud. L’état passe ensuite à Démarrage quand un Worker revendique le travail. Enfin, l’état passe à Exécution lorsque le runbook commence à s’exécuter.

4. Une fois le travail du runbook terminé, le volet Test affiche sa sortie. Dans ce cas, vous voyez `Hello World`.

   ![Sortie du volet de test](../media/automation-tutorial-runbook-textual-powershell/automation-testpane-output.png)

5. Fermez le volet de test pour revenir au canevas.

## <a name="step-4---publish-and-start-the-runbook"></a>Étape 4 : Publication et démarrage du Runbook

Le runbook que vous avez créé est toujours en mode Brouillon. Il doit être publié avant de pouvoir l’exécuter en production. Lorsque vous publiez un Runbook, vous écrasez la version publiée existante par la version brouillon. Dans ce cas, vous n’avez pas encore de version publiée car vous venez de créer le runbook.

1. Cliquez sur **Publier** pour publier le Runbook, puis sur **Oui** quand vous y êtes invité.

2. Faites défiler la page vers la gauche pour afficher le runbook sur la page Runbooks, et notez que la valeur **État de création** indique **Publié**.

3. Faites défiler la page vers la droite pour voir la page pour **MyFirstRunbook-PowerShell**.
   
   Les options en haut vous permettent de démarrer le runbook, de planifier une heure de démarrage ou de créer un [webhook](../automation-webhooks.md) pour démarrer le runbook via un appel HTTP.

4. Cliquez sur **Démarrer**, puis sur **Oui** quand vous y êtes invité pour démarrer le runbook. 

5. Un volet Travail s’ouvre pour le travail du runbook que vous avez créé. Même si vous pouvez fermer ce volet, laissez-le ouvert pour suivre la progression du travail. L’état du travail est affiché dans **Résumé du travail**, et les états possibles sont ceux décrits pour les tests du runbook.

   ![Résumé des tâches](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-jobsummary.png)

6. Quand l’état du runbook passe à Terminé, cliquez sur **Sortie** pour ouvrir la page Sortie, où vous voyez `Hello World`.

   ![Sortie de la tâche](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-outputtile.png)

7. Fermez la page Sortie.

8. Cliquez sur **Tous les journaux d’activité** pour ouvrir le volet Flux de la tâche du Runbook. Vous ne devriez voir que `Hello World` dans le flux de sortie.

    Notez que le volet Flux peut afficher d’autres flux pour un travail de runbook, tels que les flux de détails et d’erreurs, si le runbook écrit dans ceux-ci.

   ![Tous les journaux d’activité](../media/automation-tutorial-runbook-textual-powershell/job-pane-status-blade-alllogstile.png)

9. Fermez les volets Flux et Travail pour revenir à la page MyFirstRunbook-PowerShell.

10. Sous **Détails**, cliquez sur **Travaux** pour ouvrir la page Travaux pour ce runbook. Cette page répertorie tous les travaux créés par votre runbook. Vous ne devriez en voir qu’un seul, car vous n’avez exécuté le travail qu’une seule fois.

   ![Liste des tâches](../media/automation-tutorial-runbook-textual-powershell/runbook-control-job-tile.png)

11. Cliquez sur le nom du travail pour ouvrir le volet Travail que vous avez vu au démarrage du runbook. Utilisez ce volet pour voir les détails de tout travail créé pour le runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Étape 5 : Ajout d’une authentification pour gérer les ressources Azure

Vous avez testé et publié votre runbook, mais jusqu’à présent, il ne fait rien d’utile. Vous souhaitez qu’il gère les ressources Azure. Pour ce faire, le runbook doit pouvoir s’authentifier à l’aide du compte d’identification qui a été créé automatiquement lors de la création de votre compte Automation.

Comme le montre l’exemple ci-dessous, la connexion d’identification est établie avec l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-3.5.0). Si vous gérez des ressources associées à plusieurs abonnements, utilisez le paramètre `AzContext` avec [Get-AzContext](/powershell/module/Az.Accounts/Get-AzContext?view=azps-3.5.0).

> [!NOTE]
> Pour les runbooks PowerShell, `Add-AzAccount` et `Add-AzureRMAccount` sont des alias de `Connect-AzAccount`. Vous pouvez utiliser ces cmdlets ou [mettre à jour vos modules](../automation-update-azure-modules.md) dans votre compte Automation vers les dernières versions. Il est possible que vous deviez mettre à jour vos modules, même si vous venez de créer un compte Automation.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   # Wrap authentication in retry logic for transient network failures
   $logonAttempt = 0
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   $AzureContext = Get-AzSubscription -SubscriptionId $connection.SubscriptionID

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $AzureContext
   ```

1. Ouvrez l’éditeur de texte en cliquant sur **Modifier** dans la page MyFirstRunbook-PowerShell.

2. Vous n’avez plus besoin de la ligne `Write-Output`. Vous pouvez donc la supprimer.

3. Entrez ou copiez-collez le code suivant, qui gère l’authentification auprès de votre compte d’authentification Automation.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection

   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

4. Cliquez sur le volet de **Test** afin de tester le runbook.

5. Cliquez sur **Démarrer** pour démarrer le test. Une fois le test terminé, une sortie semblable à celle illustrée ci-dessous devrait afficher des informations de base sur votre compte. Cette sortie confirme que le compte d’identification est valide.

   ![Authenticate](../media/automation-tutorial-runbook-textual-powershell/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Étape 6 - Ajouter du code pour démarrer une machine virtuelle

À présent que votre runbook s’authentifie auprès de votre abonnement Azure, vous pouvez gérer les ressources. Ajoutons une commande pour démarrer une machine virtuelle. Vous pouvez choisir n’importe quelle machine virtuelle dans votre abonnement Azure et coder pour l’instant ce nom en dur dans le runbook.

1. Pour votre script de runbook, ajoutez l’applet de commande [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM?view=azps-3.5.0) pour démarrer la machine virtuelle. Comme indiqué ci-dessous, l’applet de commande démarre une machine virtuelle nommée `VMName` et avec un groupe de ressources nommé `ResourceGroupName`.

   ```powershell
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName'
   ```

2. Enregistrez le runbook, puis cliquez sur le volet de **Test** pour le tester.

3. Cliquez sur **Démarrer** pour commencer le test. Une fois le test terminé, vérifiez que la machine virtuelle a démarré.

## <a name="step-7---add-an-input-parameter"></a>Étape 7 - Ajouter un paramètre d’entrée

Votre runbook démarre actuellement la machine virtuelle que vous avez codée en dur dans le runbook. Le runbook est plus pratique si vous spécifiez le nom de la machine virtuelle lors du démarrage du runbook. Ajoutons des paramètres d’entrée au runbook pour fournir cette fonctionnalité.

1. Dans l’éditeur de texte, modifiez l’applet de commande `Start-AzVM` afin d’utiliser des variables pour les paramètres `VMName` et `ResourceGroupName`. 

   ```powershell
   Param(
    [string]$VMName,
    [string]$ResourceGroupName
   )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $connection = Get-AutomationConnection -Name AzureRunAsConnection
   while(!($connectionResult) -and ($logonAttempt -le 10))
   {
       $LogonAttempt++
       # Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                               -ServicePrincipal `
                               -Tenant $connection.TenantID `
                               -ApplicationId $connection.ApplicationID `
                               -CertificateThumbprint $connection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }

   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   ```

2. Enregistrez le Runbook et ouvrez le volet de test. Vous pouvez désormais fournir des valeurs pour les deux variables d’entrée que vous utilisez dans le test.

3. Fermez le volet de test.

4. Cliquez sur **Publier** pour publier la nouvelle version du Runbook.

5. Arrêtez la machine virtuelle que vous avez démarrée précédemment.

6. Cliquez sur **Démarrer** pour démarrer le Runbook. 

7. Tapez les valeurs de **VMNAME** et **RESOURCEGROUPNAME** pour la machine virtuelle que vous allez démarrer, puis cliquez sur **OK**.

    ![Paramètre de réussite](../media/automation-tutorial-runbook-textual-powershell/automation-pass-params.png)

8. Une fois l’exécution du runbook terminée, vérifiez que la machine virtuelle a été démarrée.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur PowerShell, notamment le langage de référence et les modules d’apprentissage, consultez la [Documentation PowerShell](/powershell/scripting/overview).
* Pour obtenir des informations de référence sur les cmdlets PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
* Pour démarrer avec les runbooks graphiques, consultez [Créer un runbook graphique](automation-tutorial-runbook-graphical.md).
* Pour démarrer avec les runbooks de workflow PowerShell, consultez [Créer un runbook de workflow PowerShell](automation-tutorial-runbook-textual.md).
* Pour en savoir plus sur les types de runbook, leurs avantages et leurs limites, consultez [Types de runbooks Azure Automation](../automation-runbook-types.md).
* Pour plus d’informations sur la fonctionnalité de prise en charge de script PowerShell, consultez [Prise en charge de script PowerShell natif dans Azure Automation](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).

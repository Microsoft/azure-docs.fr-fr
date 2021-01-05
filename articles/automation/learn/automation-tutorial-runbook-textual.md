---
title: Créer un runbook de workflow PowerShell dans Azure Automation
description: Cet article vous apprend à créer, tester et publier un runbook de workflow PowerShell simple.
services: automation
ms.subservice: process-automation
ms.date: 04/19/2020
ms.topic: tutorial
ms.openlocfilehash: 613ce41820dc5f388af441836bdb4a93e43e5d07
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90987640"
---
# <a name="tutorial-create-a-powershell-workflow-runbook"></a>Tutoriel : Créer un runbook de workflow PowerShell

Ce didacticiel décrit la création d’un [Runbook PowerShell Workflow](../automation-runbook-types.md#powershell-workflow-runbooks) dans Azure Automation. Les runbooks de workflow PowerShell sont des runbooks texte basés sur un workflow Windows PowerShell. Vous pouvez créer et modifier le code du runbook à l’aide de l’éditeur de texte du portail Azure. 

> [!div class="checklist"]
> * Créer un runbook de workflow PowerShell simple
> * Tester et publier le runbook
> * Exécuter le travail du runbook et en suivre l’état
> * Mettre à jour le runbook pour démarrer une machine virtuelle Azure avec des paramètres de runbook

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [compte Automation](../index.yml) pour le stockage du Runbook et l’authentification auprès des ressources Azure. Ce compte doit avoir l’autorisation de démarrer et d’arrêter la machine virtuelle.
* Une machine virtuelle Azure. Comme vous allez devoir l’arrêter et la démarrer, il ne doit pas s’agir d’une machine virtuelle de production.

## <a name="step-1---create-new-runbook"></a>Étape 1 - Création d’un Runbook

Commencez par créer un runbook simple qui renvoie le texte `Hello World`.

1. Dans le portail Azure, ouvrez votre compte Automation.

   La page du compte Automation vous offre un aperçu rapide des ressources de ce compte. Vous devriez déjà posséder certains éléments. La plupart de ces ressources sont les modules automatiquement inclus dans un nouveau compte Automation. Vous devez également disposer de la ressource Informations d’identification associée à votre abonnement.
 
2. Sélectionnez **Runbooks** sous **Automatisation de processus** pour ouvrir la liste des runbooks.

3. Créez un runbook en sélectionnant **Créer un runbook**.

4. Nommez le runbook **MyFirstRunbook-Workflow**.

5. Dans ce cas, vous allez créer un [runbook Workflow PowerShell](../automation-runbook-types.md#powershell-workflow-runbooks). Sélectionnez **PowerShell Workflow** comme **Type de runbook**.

6. Cliquez sur **Créer** pour créer le Runbook et ouvrez l’éditeur textuel.

## <a name="step-2---add-code-to-the-runbook"></a>Étape 2 - Ajouter du code au Runbook

Vous pouvez soit taper du code directement dans le runbook, soit sélectionner des cmdlets, des runbooks et des ressources à partir du contrôle Bibliothèque, puis les ajouter au runbook avec tous les paramètres associés. Pour ce tutoriel, vous entrez le code directement dans le runbook.

1. Pour l’instant, votre runbook est vide, à l’exception du mot clé `Workflow` obligatoire, du nom de votre runbook et des accolades qui entourent la totalité du workflow.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   }
   ```

2. Tapez `Write-Output "Hello World"` entre les accolades.

   ```powershell-interactive
   Workflow MyFirstRunbook-Workflow
   {
   Write-Output "Hello World"
   }
   ```

3. Enregistrez le Runbook en cliquant sur **Enregistrer**.

## <a name="step-3---test-the-runbook"></a>Étape 3 : Test du Runbook

Avant de publier le runbook pour le rendre disponible en production, vous devez le tester pour vous assurer qu’il fonctionne correctement. Le test d’un runbook exécute sa version Brouillon et vous permet d’afficher sa sortie de manière interactive.

1. Sélectionnez l’option **Volet de test**.

2. Cliquez sur **Démarrer** pour démarrer le test en testant la seule option activée.

3. Un [travail de runbook](../automation-runbook-execution.md) est créé, dont l’état apparaît dans le volet.

   L’état initial du travail est Mis en file d’attente pour indiquer qu’il attend qu’un Runbook Worker devienne disponible dans le cloud. L’état passe ensuite à Démarrage quand un Worker revendique le travail. Enfin, l’état passe à Exécution lorsque le runbook commence à s’exécuter.

4. Une fois le travail du runbook terminé, le volet Test affiche sa sortie. Dans ce cas, vous voyez `Hello World`.

   ![Hello World](../media/automation-tutorial-runbook-textual/test-output-hello-world.png)

5. Fermez le volet de test pour revenir au canevas.

## <a name="step-4---publish-and-start-the-runbook"></a>Étape 4 : Publication et démarrage du Runbook

Le runbook que vous avez créé est toujours en mode brouillon. Vous devez le publier pour pouvoir l’exécuter en production. Lorsque vous publiez un Runbook, vous écrasez la version publiée existante par la version brouillon. Dans ce cas, vous n’avez pas encore de version publiée car vous venez de créer le runbook.

1. Cliquez sur **Publier** pour publier le Runbook, puis sur **Oui** quand vous y êtes invité.

2. Faites défiler la page vers la gauche pour afficher le runbook sur la page **Runbooks**, et notez que le champ **État de création** indique **Publié**.

3. Faites défiler la page vers la droite pour afficher **MyFirstRunbook-Workflow**.

   Les options en haut vous permettent de démarrer le runbook, de planifier une heure de démarrage ou de créer un [webhook](../automation-webhooks.md) pour démarrer le runbook via un appel HTTP.

4. Cliquez sur **Démarrer**, puis sur **Oui** quand vous y êtes invité pour démarrer le runbook.

   ![Démarrer un Runbook](../media/automation-tutorial-runbook-textual/automation-runbook-controls-start.png)

5. Un volet Travail s’ouvre pour le travail du runbook que vous avez créé. Dans ce cas, laissez le volet ouvert pour pouvoir suivre la progression du travail.

6. Notez que l’état du travail est affiché dans **Résumé du travail**. Cet état correspond aux états que vous avez vus lors du test du runbook.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-jobsummary.png" alt-text="Capture d’écran du volet Tâche du runbook montrant la section Résumé du travail en évidence.":::

7. Lorsque l’état du runbook est Terminé, cliquez sur **Sortie**. La page Sortie s’ouvre, dans laquelle vous pouvez voir votre message `Hello World`.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-outputtile.png" alt-text="Capture d’écran du volet Tâche du runbook montrant la section Résumé du travail en évidence.":::

8. Fermez la page Sortie.

9. Cliquez sur **Tous les journaux d’activité** pour ouvrir le volet Flux de la tâche du Runbook. Vous ne devriez voir que `Hello World` dans le flux de sortie. Notez que le volet Flux peut afficher d’autres flux pour une tâche de runbook, tels que des flux de détails et d’erreurs, si le runbook écrit dans ceux-ci.

   :::image type="content" source="../media/automation-tutorial-runbook-textual/job-pane-status-blade-alllogstile.png" alt-text="Capture d’écran du volet Tâche du runbook montrant la section Résumé du travail en évidence.":::

10. Fermez les volets du flux et de la tâche pour revenir à la page MyFirstRunbook.

11. Cliquez sur **Travaux** sous **Ressources** pour ouvrir la page Travaux de ce runbook. Cette page répertorie tous les travaux créés par votre runbook. Vous ne devriez en voir qu’un seul, car vous n’avez exécuté le travail qu’une seule fois.

   ![travaux](../media/automation-tutorial-runbook-textual/runbook-control-job-tile.png)

12. Cliquez sur le nom du travail pour ouvrir le volet Travail que vous avez vu au démarrage du runbook. Utilisez ce volet pour voir les détails de tout travail créé pour le runbook.

## <a name="step-5---add-authentication-to-manage-azure-resources"></a>Étape 5 : Ajout d’une authentification pour gérer les ressources Azure

Vous avez testé et publié votre runbook, mais jusqu’à présent, il ne fait rien d’utile. Vous souhaitez qu’il gère les ressources Azure. Il ne peut le faire que s’il s’authentifie à l’aide des informations d’identification de l’abonnement. L’authentification utilise la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-3.7.0).

>[!NOTE]
>Pour les runbooks PowerShell, `Add-AzAccount` et `Add-AzureRMAccount` sont des alias de `Connect-AzAccount`. Vous pouvez utiliser ces cmdlets ou [mettre à jour vos modules](../automation-update-azure-modules.md) dans votre compte Automation vers les dernières versions. Il est possible que vous deviez mettre à jour vos modules, même si vous venez de créer un compte Automation.

1. Accédez à la page MyFirstRunbook-Workflow et ouvrez l’éditeur de texte en cliquant sur **Modifier**.

2. Supprimez la ligne `Write-Output`.

3. Positionnez le curseur sur une ligne vide entre les accolades.

4. Entrez ou copiez-collez le code suivant, qui gère l’authentification auprès de votre compte d’authentification Automation.

   ```powershell-interactive
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID `
   -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Select-AzSubscription -SubscriptionId $Conn.SubscriptionID
   ```

5. Cliquez sur le volet de **Test** afin de tester le runbook.

6. Cliquez sur **Démarrer** pour démarrer le test. Une fois le test terminé, une sortie semblable à celle illustrée ci-dessous devrait afficher des informations de base sur votre compte. Cette action confirme la validité des informations d’identification.

   ![Authenticate](../media/automation-tutorial-runbook-textual/runbook-auth-output.png)

## <a name="step-6---add-code-to-start-a-virtual-machine"></a>Étape 6 - Ajouter du code pour démarrer une machine virtuelle

À présent que votre runbook s’authentifie auprès de l’abonnement Azure, vous pouvez gérer les ressources. Ajoutons une commande pour démarrer une machine virtuelle. Vous pouvez choisir n’importe quelle machine virtuelle de votre abonnement Azure. Pour l’instant, vous allez coder ce nom en dur dans le runbook. Si vous gérez des ressources associées à plusieurs abonnements, utilisez le paramètre `AzContext` avec l’applet de commande [Get-AzContext](/powershell/module/az.accounts/get-azcontext).

1. Indiquez le nom et le nom du groupe de ressources de la machine virtuelle à démarrer en entrant un appel à la cmdlet [Start-AzVM](/powershell/module/Az.Compute/Start-AzVM?view=azps-3.5.0) comme illustré ci-dessous. 

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
   # Ensures that you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $AzureContext = Get-AzSubscription -SubscriptionId $Conn.SubscriptionID

   Start-AzVM -Name 'VMName' -ResourceGroupName 'ResourceGroupName' -AzContext $AzureContext
   }
   ```

2. Enregistrez le runbook, puis cliquez sur le volet de **Test** pour le tester.

3. Cliquez sur **Démarrer** pour démarrer le test. Une fois l’opération terminée, vérifiez que la machine virtuelle a démarré.

## <a name="step-7---add-an-input-parameter-to-the-runbook"></a>Étape 7 : Ajout d’un paramètre d'entrée au Runbook

Votre runbook démarre actuellement la machine virtuelle que vous avez codée en dur dans le runbook. Elle sera plus utile si vous pouvez la spécifier au démarrage du runbook. Ajoutons des paramètres d’entrée au runbook pour fournir cette fonctionnalité.

1. Ajoutez des variables aux paramètres `VMName` et `ResourceGroupName` du runbook, puis utilisez les variables avec l’applet de commande `Start-AzVM`, comme indiqué ci-dessous.

   ```powershell-interactive
   workflow MyFirstRunbook-Workflow
   {
    Param(
     [string]$VMName,
     [string]$ResourceGroupName
    )
   # Ensures you do not inherit an AzContext in your runbook
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint
   Start-AzVM -Name $VMName -ResourceGroupName $ResourceGroupName
   }
   ```

2. Enregistrez le Runbook et ouvrez le volet de test. Vous pouvez désormais fournir des valeurs pour les deux variables d’entrée se trouvant dans le test.

3. Fermez le volet de test.

4. Cliquez sur **Publier** pour publier la nouvelle version du Runbook.

5. Arrêtez la machine virtuelle que vous avez démarrée.

6. Cliquez sur **Démarrer** pour démarrer le Runbook. 

7. Tapez les valeurs de **VMNAME** et **RESOURCEGROUPNAME** pour la machine virtuelle que vous allez démarrer.

   ![Démarrer le Runbook](../media/automation-tutorial-runbook-textual/automation-pass-params.png)

8. Une fois l’exécution du runbook terminée, vérifiez que la machine virtuelle a démarré.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur PowerShell, notamment le langage de référence et les modules d’apprentissage, consultez la [Documentation PowerShell](/powershell/scripting/overview).
* Pour obtenir des informations de référence sur les cmdlets PowerShell, consultez [Az.Automation](/powershell/module/az.automation/?view=azps-3.7.0#automation).
* Pour démarrer avec les runbooks graphiques, consultez [Créer un runbook graphique](automation-tutorial-runbook-graphical.md).
* Pour démarrer avec les runbooks PowerShell, consultez [Créer un runbook PowerShell](automation-tutorial-runbook-textual-powershell.md).
* Pour plus d’informations sur les types de runbooks, sur leurs avantages et sur leurs limites, voir [Types de runbooks Azure Automation](../automation-runbook-types.md).
* Pour plus d’informations sur les fonctionnalités de prise en charge de script PowerShell, voir [Prise en charge de script PowerShell natif dans Azure Automation](https://azure.microsoft.com/blog/announcing-powershell-script-support-azure-automation-2/).

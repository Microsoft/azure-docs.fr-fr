---
title: Résoudre des problèmes de runbook Azure Automation
description: Cet article explique comment dépanner et résoudre des problèmes liés aux runbooks Azure Automation.
services: automation
ms.date: 11/03/2020
ms.topic: conceptual
ms.service: automation
ms.custom: has-adal-ref
ms.openlocfilehash: c7ab093f601ebcd33d184b9a9008f9de447534a1
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368081"
---
# <a name="troubleshoot-runbook-issues"></a>Résoudre les problèmes de runbook

 Cet article décrit décrit des problèmes de runbook qui peuvent se produire, et explique comment les résoudre. Pour obtenir des informations générales sur les runbooks, consultez [Exécution d’un Runbook dans Azure Automation](../automation-runbook-execution.md).

## <a name="diagnose-runbook-issues"></a>Diagnostiquer des problèmes de runbook

Lorsque des erreurs surviennent pendant l’exécution de runbooks dans Azure Automation, vous pouvez utiliser les étapes suivantes pour diagnostiquer les problèmes :

1. Vérifiez que le script de votre runbook s’exécute correctement sur votre machine locale.

    Pour obtenir des modules de référence et d’entraînement du langage, reportez-vous à la [Documentation de PowerShell](/powershell/scripting/overview) ou à la [Documentation de Python](https://docs.python.org/3/). L’exécution en local de votre script peut détecter et résoudre les erreurs courantes, telles que :

      * Modules manquants
      * Erreurs de syntaxe
      * Erreurs logiques

1. Examinez les [flux d’erreurs](../automation-runbook-output-and-messages.md#runbook-output) des runbooks.

    Regardez si ces flux contiennent des messages particuliers, et comparez-les aux erreurs documentées dans cet article.

1. Vérifiez que vos nœuds et votre espace de travail Automation disposent des modules nécessaires.

    Si votre runbook importe des modules, vérifiez qu’ils sont disponibles pour votre compte Automation en suivant les étapes décrites dans [Importer des modules](../shared-resources/modules.md#import-modules). Mettez à jour vos modules PowerShell vers la dernière version disponible en suivant les instructions fournies dans [Mettre à jour des modules Azure PowerShell dans Azure Automation](../automation-update-azure-modules.md). Pour plus d’informations sur la résolution des problèmes, consultez [Résoudre les problèmes liés aux modules](shared-resources.md#modules).

1. En cas d’échec inattendu ou d’interruption du runbook :

    * [Renouvelez le certificat](../manage-runas-account.md#cert-renewal) si le compte d’identification a expiré.
    * [Renouvelez le webhook](../automation-webhooks.md#renew-a-webhook) si vous essayez d’utiliser un webhook expiré pour démarrer le runbook.
    * [Vérifiez les états des travaux](../automation-runbook-execution.md#job-statuses) pour déterminer les états de runbook actuels et des causes possibles du problème.
    * [Ajoutez une sortie supplémentaire](../automation-runbook-output-and-messages.md#working-with-message-streams) au runbook pour identifier ce qui se passe avant l'interruption du runbook.
    * [Gérez les exceptions](../automation-runbook-execution.md#exceptions) qui sont levées par votre travail.

1. Effectuez cette étape si le travail ou l’environnement du runbook sur Runbook Worker hybride ne répond pas.

    Si vous exécutez vos runbooks sur un Runbook Worker hybride plutôt que dans Azure Automation, vous devez peut-être [résoudre les problèmes du worker hybride](hybrid-runbook-worker.md).

## <a name="scenario-runbook-fails-with-a-no-permission-or-forbidden-403-error"></a><a name="runbook-fails-no-permission"></a>Scénario : Un runbook échoue avec une erreur 403 Aucune autorisation ou Interdit

### <a name="issue"></a>Problème

Votre runbook échoue avec une erreur 403 Aucune autorisation ou Interdit, ou une erreur équivalente.

### <a name="cause"></a>Cause

Les comptes d’identification n’ont peut-être pas les mêmes autorisations sur les ressources Azure que votre compte Automation actuel. 

### <a name="resolution"></a>Résolution

Vérifiez que votre compte d’identification dispose des [autorisations nécessaires pour accéder aux ressources](../../role-based-access-control/role-assignments-portal.md) utilisées dans votre script.

## <a name="scenario-sign-in-to-azure-account-failed"></a><a name="sign-in-failed"></a>Scénario : Échec de la connexion au compte Azure

### <a name="issue"></a>Problème

L’une des erreurs suivantes s’affiche quand vous utilisez l’applet de commande `Connect-AzAccount` :

```error
Unknown_user_type: Unknown User Type
```

```error
No certificate was found in the certificate store with thumbprint
```

### <a name="cause"></a>Cause

Ces erreurs se produisent quand le nom de la ressource d’informations d’identification n’est pas valide. Elles peuvent également se produire si le nom d’utilisateur et le mot de passe que vous avez utilisés pour configurer la ressource d’informations d’identification Automation ne sont pas valides.

### <a name="resolution"></a>Résolution

Pour identifiez le problème, suivez ces étapes :

1. Vérifiez qu’il n’existe aucun caractère spécial (notamment le caractère `\@`) dans le nom de la ressource d’informations d’identification Automation que vous utilisez pour vous connecter à Azure.
1. Vérifiez que vous pouvez utiliser le nom d’utilisateur et le mot de passe stockés dans les informations d’identification Azure Automation dans votre éditeur d’ISE PowerShell local. Exécutez les applets de commande suivantes dans PowerShell ISE.

   ```powershell
   $Cred = Get-Credential
   #Using Azure Service Management
   Add-AzureAccount –Credential $Cred
   #Using Azure Resource Manager
   Connect-AzAccount –Credential $Cred
   ```

1. Si l’authentification échoue localement, cela signifie que vous n’avez pas correctement configuré vos informations d’identification Azure Active Directory (Azure AD). Pour configurer correctement le compte Azure AD, consultez l’article [S’authentifier sur Azure à l’aide d’Azure Active Directory](../automation-use-azure-ad.md).

1. Si cette erreur semble temporaire, essayez d’ajouter la logique de nouvelle tentative à votre routine d’authentification pour renforcer l’authentification.

   ```powershell
   # Get the connection "AzureRunAsConnection"
   $connectionName = "AzureRunAsConnection"
   $servicePrincipalConnection = Get-AutomationConnection -Name $connectionName

   $logonAttempt = 0
   $logonResult = $False

   while(!($connectionResult) -And ($logonAttempt -le 10))
   {
       $LogonAttempt++
       #Logging in to Azure...
       $connectionResult = Connect-AzAccount `
                              -ServicePrincipal `
                              -Tenant $servicePrincipalConnection.TenantId `
                              -ApplicationId $servicePrincipalConnection.ApplicationId `
                              -CertificateThumbprint $servicePrincipalConnection.CertificateThumbprint

       Start-Sleep -Seconds 30
   }
   ```

## <a name="scenario-run-login-azurermaccount-to-log-in"></a><a name="login-azurerm"></a>Scénario : Exécuter Login-AzureRMAccount pour se connecter

### <a name="issue"></a>Problème

Lors de l’exécution d’un runbook, vous recevez l’erreur suivante :

```error
Run Login-AzureRMAccount to login.
```

### <a name="cause"></a>Cause

Cette erreur peut se produire lorsque vous n’utilisez pas un compte d’identification ou quand le compte d’identification a expiré. Pour plus d’informations, consultez [Gérer les comptes d’identification Azure Automation](../manage-runas-account.md).

Cette erreur a deux causes principales :

* Il existe plusieurs versions du module AzureRM ou Az.
* Vous essayez d’accéder à des ressources dans un abonnement distinct.

### <a name="resolution"></a>Résolution

Si vous recevez cette erreur après avoir mis à jour un module AzureRM ou Az, mettez à jour tous vos modules vers la même version.

Si vous essayez d’accéder à des ressources dans un autre abonnement, suivez ces étapes pour configurer les autorisations :

1. Accédez au compte d’identification Automation, puis copiez l’ **ID d’application** et l’ **Empreinte**.

    ![Copier l’ID d’application et l’empreinte](../media/troubleshoot-runbooks/collect-app-id.png)

1. Accédez au **Contrôle d’accès** de l’abonnement dans lequel le compte Automation n’est *pas* hébergé, puis ajoutez une nouvelle attribution de rôle.

    ![Contrôle d’accès](../media/troubleshoot-runbooks/access-control.png)

1. Ajoutez l’ **ID d’application** collecté précédemment. Sélectionnez des autorisations **Contributeur**.

    ![Ajouter une attribution de rôle](../media/troubleshoot-runbooks/add-role-assignment.png)

1. Copiez le nom de l’abonnement.

1. Vous pouvez maintenant utiliser le code de runbook suivant pour tester les autorisations de votre compte Automation sur l’autre abonnement. Remplacez `"\<CertificateThumbprint\>"` par la valeur copiée à l’étape 1. Remplacez `"\<SubscriptionName\>"` par la valeur copiée à l’étape 4.

    ```powershell
    $Conn = Get-AutomationConnection -Name AzureRunAsConnection
    Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint "<CertificateThumbprint>"
    #Select the subscription you want to work with
    Select-AzSubscription -SubscriptionName '<YourSubscriptionNameGoesHere>'

    #Test and get outputs of the subscriptions you granted access.
    $subscriptions = Get-AzSubscription
    foreach($subscription in $subscriptions)
    {
        Set-AzContext $subscription
        Write-Output $subscription.Name
    }
    ```

## <a name="scenario-unable-to-find-the-azure-subscription"></a><a name="unable-to-find-subscription"></a>Scénario : Impossible de trouver l’abonnement Azure

### <a name="issue"></a>Problème

L’erreur suivante s’affiche quand vous utilisez l’applet de commande `Select-AzureSubscription`, `Select-AzureRMSubscription` ou `Select-AzSubscription` :

```error
The subscription named <subscription name> cannot be found.
```

### <a name="error"></a>Error

Cette erreur peut se produire si :

* Le nom de l’abonnement n’est pas valide.
* L’utilisateur Azure AD qui tente d’obtenir les détails de l’abonnement n’est pas configuré comme administrateur de l’abonnement.
* L’applet de commande n’est pas disponible.

### <a name="resolution"></a>Résolution

Suivez ces étapes pour déterminer si vous vous êtes correctement authentifié auprès d’Azure, et si vous avez accès à l’abonnement que vous souhaitez sélectionner :

1. Pour vérifier qu’il fonctionne de façon autonome, testez votre script en dehors d’Azure Automation.
1. Vérifiez que votre script exécute la cmdlet [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) avant d’exécuter la cmdlet `Select-*`.
1. Ajoutez `Disable-AzContextAutosave –Scope Process` au début de votre runbook. Cette applet de commande permet de s'assurer que les informations d’identification s’appliquent uniquement à l’exécution du runbook actuel.
1. Si le message d’erreur persiste, modifiez votre code en ajoutant le paramètre `AzContext` pour `Connect-AzAccount`, puis exécutez le code.

   ```powershell
   Disable-AzContextAutosave –Scope Process

   $Conn = Get-AutomationConnection -Name AzureRunAsConnection
   Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -ApplicationId $Conn.ApplicationID -CertificateThumbprint $Conn.CertificateThumbprint

   $context = Get-AzContext

   Get-AzVM -ResourceGroupName myResourceGroup -AzContext $context
    ```

## <a name="scenario-runbooks-fail-when-dealing-with-multiple-subscriptions"></a><a name="runbook-auth-failure"></a>Scénario : Les runbooks échouent lors du traitement de plusieurs abonnements

### <a name="issue"></a>Problème

Lors de l’exécution de runbooks, le runbook ne parvient pas à gérer les ressources Azure.

### <a name="cause"></a>Cause

Le runbook n’utilise pas le contexte approprié lors de l’exécution.

### <a name="resolution"></a>Résolution

Le contexte de l’abonnement peut être perdu lorsqu’un runbook appelle plusieurs runbooks. Pour garantir que le contexte d’abonnement sera passé aux runbooks, indiquez au client runbook de passer le contexte à l’applet de commande `Start-AzureRmAutomationRunbook` dans le paramètre `AzureRmContext`. Utilisez l’applet de commande `Disable-AzureRmContextAutosave` avec le paramètre `Scope` défini sur `Process` pour faire en sorte que les informations d’identification spécifiées soient utilisées uniquement pour le runbook actuel. Pour plus d'informations, consultez [Abonnements](../automation-runbook-execution.md#subscriptions).

```azurepowershell-interactive
# Ensures that any credentials apply only to the execution of this runbook
Disable-AzContextAutosave –Scope Process

# Connect to Azure with Run As account
$ServicePrincipalConnection = Get-AutomationConnection -Name 'AzureRunAsConnection'

Connect-AzAccount `
    -ServicePrincipal `
    -Tenant $ServicePrincipalConnection.TenantId `
    -ApplicationId $ServicePrincipalConnection.ApplicationId `
    -CertificateThumbprint $ServicePrincipalConnection.CertificateThumbprint

$AzContext = Select-AzSubscription -SubscriptionId $ServicePrincipalConnection.SubscriptionID

$params = @{"VMName"="MyVM";"RepeatCount"=2;"Restart"=$true}

Start-AzAutomationRunbook `
    –AutomationAccountName 'MyAutomationAccount' `
    –Name 'Test-ChildRunbook' `
    -ResourceGroupName 'LabRG' `
    -AzContext $AzContext `
    –Parameters $params –wait
```

## <a name="scenario-authentication-to-azure-fails-because-multifactor-authentication-is-enabled"></a><a name="auth-failed-mfa"></a>Scénario : L’authentification auprès d’Azure échoue parce que l’authentification multifacteur est activée

### <a name="issue"></a>Problème

Vous recevez l’erreur suivante lors de l’authentification auprès d’Azure avec vos nom d’utilisateur et mot de passe Azure :

```error
Add-AzureAccount: AADSTS50079: Strong authentication enrollment (proof-up) is required
```

### <a name="cause"></a>Cause

Si l’authentification multifacteur est configurée sur votre compte Azure, vous ne pouvez pas vous authentifier auprès d’Azure avec un utilisateur Azure Active Directory. Au lieu de cela, vous devez utiliser un certificat ou un principal de service pour l’authentification.

### <a name="resolution"></a>Résolution

Pour utiliser un compte d’identification Classic avec les cmdlets du modèle de déploiement Azure Classic, consultez [Créer un compte d’identification Classic pour gérer les services Azure](../automation-create-standalone-account.md#create-a-classic-run-as-account). Pour utiliser un principal de service avec des applets de commande Azure Resource Manager, reportez-vous à [Création du principal du service à l’aide du portail Azure](../../active-directory/develop/howto-create-service-principal-portal.md) et [Authentification d’un principal du service à l’aide d’Azure Resource Manager](../../active-directory/develop/howto-authenticate-service-principal-powershell.md).

## <a name="scenario-runbook-fails-with-a-task-was-canceled-error-message"></a><a name="task-was-cancelled"></a>Scénario : Le runbook échoue avec le message d’erreur « Une tâche a été annulée »

### <a name="issue"></a>Problème

Votre runbook échoue avec une erreur similaire à l’exemple suivant :

```error
Exception: A task was canceled.
```

### <a name="cause"></a>Cause

Cette erreur peut être due à l’utilisation de modules Azure obsolètes.

### <a name="resolution"></a>Résolution

Vous pouvez corriger cette erreur en mettant à jour vos modules Azure avec la toute dernière version :

1. Dans votre compte Automation, sélectionnez **Modules** , puis **Mettre à jour les modules Azure**.
1. La mise à jour prend environ 15 minutes. Une fois l’opération terminée, réexécutez le runbook qui a échoué.

Pour en savoir plus sur la mise à jour de vos modules, consultez [Mettre à jour des modules Azure dans Azure Automation](../automation-update-azure-modules.md).

## <a name="scenario-term-not-recognized-as-the-name-of-a-cmdlet-function-or-script"></a><a name="not-recognized-as-cmdlet"></a>Scénario : Terme non reconnu comme nom d’une applet de commande, d’une fonction or d’un script

### <a name="issue"></a>Problème

Votre runbook échoue avec une erreur similaire à l’exemple suivant :

```error
The term 'Connect-AzAccount' is not recognized as the name of a cmdlet, function, script file, or operable program. Check the spelling of the name, or if the path was included verify that the path is correct and try again.
```

### <a name="cause"></a>Cause

Cette erreur peut se produire pour les raisons suivantes :

* Le module qui contient l’applet de commande n’est pas importé dans le compte Automation.
* Le module qui contient l’applet de commande est importé, mais il est obsolète.

### <a name="resolution"></a>Résolution

Effectuez l’une des tâches suivantes pour corriger cette erreur :

* Pour un module Azure, consultez [Guide de mise à jour des modules Azure PowerShell dans Azure Automation](../automation-update-azure-modules.md) afin de savoir comment mettre à jour vos modules dans votre compte Automation.
* Pour un module non-Azure, vérifiez que le module a été importé dans votre compte Automation.

## <a name="scenario-cmdlet-fails-in-pnp-powershell-runbook-on-azure-automation"></a>Scénario : La cmdlet échoue dans le runbook PnP PowerShell sur Azure Automation

### <a name="issue"></a>Problème

Lorsqu’un runbook écrit directement un objet généré par PowerShell PnP dans la sortie Azure Automation, la sortie de la cmdlet ne peut pas être renvoyée à Automation.

### <a name="cause"></a>Cause

Ce problème est généralement dû au fait qu’Azure Automation traite les runbooks qui appellent des applets de commande PowerShell PnP, par exemple `add-pnplistitem`, sans intercepter les objets de retour.

### <a name="resolution"></a>Résolution

Modifiez vos scripts pour attribuer des valeurs retournées à des variables, afin que les applets de commande n’essaient pas d’écrire des objets entiers dans la sortie standard. Un script peut rediriger le flux de sortie vers une applet de commande, comme indiqué ici.

```azurecli
  $null = add-pnplistitem
```

Si votre script analyse la sortie de la cmdlet, le script doit stocker la sortie dans une variable et manipuler la variable au lieu de simplement diffuser la sortie.

```azurecli
$SomeVariable = add-pnplistitem ....
if ($SomeVariable.someproperty -eq ....
```

## <a name="scenario-cmdlet-not-recognized-when-executing-a-runbook"></a><a name="cmdlet-not-recognized"></a>Scénario : La cmdlet n’est pas reconnue lors de l’exécution d’un runbook

### <a name="issue"></a>Problème

Le travail de votre runbook échoue avec l’erreur :

```error
<cmdlet name>: The term <cmdlet name> is not recognized as the name of a cmdlet, function, script file, or operable program.
```

### <a name="cause"></a>Cause

Cette erreur survient quand le moteur PowerShell ne trouve pas la cmdlet que vous utilisez dans votre runbook. Il est possible que le module contenant l’applet de commande ne soit pas présent dans le compte, qu’il existe un conflit de nom avec un nom de runbook ou que l’applet de commande existe déjà dans un autre module et qu’Automation ne puisse pas résoudre le nom.

### <a name="resolution"></a>Résolution

Utilisez l’une des solutions suivantes pour résoudre le problème :

* Vérifiez que vous avez correctement entré le nom de l’applet de commande.
* Vérifiez que l’applet de commande existe dans votre compte Automation et qu’il n’existe aucun conflit. Pour vérifier si l’applet de commande est présente, ouvrez un runbook en mode édition et recherchez l’applet de commande dans la bibliothèque, ou exécutez `Get-Command <CommandName>`. Après avoir vérifié que l’applet de commande est disponible pour le compte, et qu’il n’existe aucun conflit de nom avec d’autres applets de commande ou runbooks, ajoutez l’applet de commande dans le canevas. Veillez à utiliser un jeu de paramètres valide dans votre runbook.
* Si vous rencontrez un conflit de noms et si l’applet de commande est disponible dans deux modules différents, vous pouvez résoudre ce problème en utilisant le nom complet de l’applet de commande. Par exemple, vous pouvez utiliser `ModuleName\CmdletName`.
* Si vous exécutez le runbook localement dans un groupe de workers hybrides, vérifiez que le module et l’applet de commande sont installés sur la machine qui héberge le worker hybride.

## <a name="scenario-incorrect-object-reference-on-call-to-add-azaccount"></a><a name="object-reference-not-set"></a>Scénario : Référence d’objet incorrecte lors de l’appel à Add-AzAccount

### <a name="issue"></a>Problème

Vous recevez cette erreur lorsque vous utilisez `Add-AzAccount`, qui est un alias pour l’applet de commande `Connect-AzAccount` :

```error
Add-AzAccount : Object reference not set to an instance of an object
```

### <a name="cause"></a>Cause

Cette erreur peut se produire si le runbook n’effectue pas les étapes appropriées avant d’appeler `Add-AzAccount` pour ajouter le compte Automation. Un exemple d’étape nécessaire est la connexion avec un compte d’identification. Pour connaître les opérations correctes à utiliser dans votre runbook, consultez [Exécution d’un runbook dans Azure Automation](../automation-runbook-execution.md).

## <a name="scenario-object-reference-not-set-to-an-instance-of-an-object"></a><a name="child-runbook-object"></a>Scénario : La référence d’objet n’a pas la valeur d’une instance d’un objet

### <a name="issue"></a>Problème

Vous recevez l’erreur suivante lorsque vous appelez un runbook enfant avec le paramètre `Wait`, et que le flux de sortie contient un objet :

```error
Object reference not set to an instance of an object
```

### <a name="cause"></a>Cause

Si le flux contient des objets, `Start-AzAutomationRunbook` ne gère pas correctement le flux de sortie.

### <a name="resolution"></a>Résolution

Implémentez une logique d’interrogation et utilisez l’applet de commande [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput) pour récupérer la sortie. Un exemple de cette logique est défini ici :

```powershell
$automationAccountName = "ContosoAutomationAccount"
$runbookName = "ChildRunbookExample"
$resourceGroupName = "ContosoRG"

function IsJobTerminalState([string] $status) {
    return $status -eq "Completed" -or $status -eq "Failed" -or $status -eq "Stopped" -or $status -eq "Suspended"
}

$job = Start-AzAutomationRunbook -AutomationAccountName $automationAccountName -Name $runbookName -ResourceGroupName $resourceGroupName
$pollingSeconds = 5
$maxTimeout = 10800
$waitTime = 0
while($false -eq (IsJobTerminalState $job.Status) -and $waitTime -lt $maxTimeout) {
   Start-Sleep -Seconds $pollingSeconds
   $waitTime += $pollingSeconds
   $job = $job | Get-AzAutomationJob
}

$job | Get-AzAutomationJobOutput | Get-AzAutomationJobOutputRecord | Select-Object -ExpandProperty Value
```

## <a name="scenario-runbook-fails-because-of-deserialized-object"></a><a name="fails-deserialized-object"></a>Scénario : Le runbook échoue à cause d’un objet désérialisé

### <a name="issue"></a>Problème

Votre runbook échoue avec l’erreur :

```error
Cannot bind parameter <ParameterName>.

Cannot convert the <ParameterType> value of type Deserialized <ParameterType> to type <ParameterType>.
```

### <a name="cause"></a>Cause

Si votre runbook est un workflow PowerShell, il stocke les objets complexes dans un format désérialisé afin de conserver l’état du runbook si le workflow est suspendu.

### <a name="resolution"></a>Résolution

Utilisez une des solutions suivantes pour résoudre ce problème :

* Si vous redirigez des objets complexes d’une applet de commande vers une autre, wrappez ces deux applets de commande dans une activité `InlineScript`.
* Transmettez le nom ou la valeur dont vous avez besoin depuis l’objet complexe au lieu de transmettre la totalité de l’objet.
* Utilisez un runbook PowerShell au lieu d’un runbook Workflow PowerShell.

## <a name="scenario-400-bad-request-status-when-calling-a-webhook"></a><a name="expired webhook"></a>Scénario : État 400 Requête incorrecte lors de l’appel d’un webhook

### <a name="issue"></a>Problème

Lorsque vous essayez d’appeler un webhook pour un runbook Azure Automation, vous recevez l’erreur suivante :

```error
400 Bad Request : This webhook has expired or is disabled
```

### <a name="cause"></a>Cause

Le webhook que vous tentez d’appeler est désactivé ou a expiré. 

### <a name="resolution"></a>Résolution

Si le webhook est désactivé, vous pouvez le réactiver via le portail Azure. Si le webhook a expiré, vous devez le supprimer puis le recréer. Vous pouvez uniquement [renouveler un webhook](../automation-webhooks.md#renew-a-webhook) s’il n’a pas déjà expiré. 

## <a name="scenario-429-the-request-rate-is-currently-too-large"></a><a name="429"></a>Scénario : 429 : Le taux de requêtes est actuellement trop important

### <a name="issue"></a>Problème

Vous recevez le message d’erreur suivant lors de l’exécution de l’applet de commande `Get-AzAutomationJobOutput` :

```error
429: The request rate is currently too large. Please try again
```

### <a name="cause"></a>Cause

Cette erreur peut se produire lors de la récupération d’une sortie de tâche à partir d’un runbook comportant de nombreux [flux détaillés](../automation-runbook-output-and-messages.md#write-output-to-verbose-stream).

### <a name="resolution"></a>Résolution

Effectuez l’une des actions suivantes pour corriger cette erreur :

* Modifiez le runbook et réduisez le nombre de flux de tâches émis.
* Réduisez le nombre de flux à récupérer lors de l’exécution de la cmdlet. Pour cela, vous pouvez définir la valeur du paramètre `Stream` de la cmdlet [Get-AzAutomationJobOutput](/powershell/module/Az.Automation/Get-AzAutomationJobOutput) de manière à récupérer uniquement les flux de sortie. 

## <a name="scenario-runbook-job-fails-because-allocated-quota-was-exceeded"></a><a name="quota-exceeded"></a>Scénario : Le travail du runbook échoue en raison du dépassement du quota alloué

### <a name="issue"></a>Problème

Le travail de votre runbook échoue avec l’erreur :

```error
The quota for the monthly total job run time has been reached for this subscription
```

### <a name="cause"></a>Cause

Cette erreur se produit quand l’exécution du travail dépasse le quota gratuit de 500 minutes pour votre compte. Ce quota s’applique à tous les types de tâches d’exécution de travail, notamment le test d’un travail, le démarrage d’un travail à partir du portail, l’exécution d’un travail à l’aide de webhooks ou la planification d’un travail à exécuter en utilisant le portail Azure ou dans votre centre de données. Pour en savoir plus sur la tarification d’Automation, consultez [Tarification d’Automation](https://azure.microsoft.com/pricing/details/automation/).

### <a name="resolution"></a>Résolution

Si vous voulez utiliser plus de 500 minutes de traitement par mois, passez d’un abonnement de niveau Gratuit à un abonnement de niveau De base :

1. Connectez-vous à votre abonnement Azure.
1. Sélectionnez le compte Automation à mettre à niveau.
1. Sélectionnez **Paramètres** , puis **Tarifs**.
1. Sélectionnez **Activer** au bas de la page pour mettre à niveau votre compte vers le niveau De base.

## <a name="scenario-runbook-output-stream-greater-than-1-mb"></a><a name="output-stream-greater-1mb"></a>Scénario : Flux de sortie du runbook supérieur à 1 Mo

### <a name="issue"></a>Problème

Votre runbook s’exécutant dans le bac à sable (sandbox) Azure échoue avec l’erreur suivante :

```error
The runbook job failed due to a job stream being larger than 1MB, this is the limit supported by an Azure Automation sandbox.
```

### <a name="cause"></a>Cause

Cette erreur se produit parce que votre runbook a tenté d’écrire trop de données d’exception dans le flux de sortie.

### <a name="resolution"></a>Résolution

Une limite de 1 Mo s’applique au flux de sortie des travaux. Vérifiez que votre runbook englobe des appels à un fichier exécutable ou à un sous-processus à l’aide des blocs `try` et `catch`. Si les opérations lèvent une exception, faites en sorte que le code écrive le message de l’exception dans une variable Automation. Cette technique empêche l’écriture du message dans le flux de sortie du travail. Pour les travaux Runbook Workers hybrides exécutés, c’est le flux de sortie tronqué à 1 Mo qui s’affiche, sans message d’erreur.

## <a name="scenario-runbook-job-start-attempted-three-times-but-fails-to-start-each-time"></a><a name="job-attempted-3-times"></a>Scénario : Le démarrage du travail du runbook a été tenté à trois reprises, mais a échoué

### <a name="issue"></a>Problème

Votre runbook échoue avec l’erreur suivante :

```error
The job was tried three times but it failed
```

### <a name="cause"></a>Cause

Cette erreur se produit en raison de l’un des problèmes suivants :

* **Limite de mémoire.** Un travail peut échouer s’il utilise plus de 400 Mo de mémoire. Les limites documentées sur la mémoire allouée à un bac à sable sont indiquées dans [Limites du service Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits). 

* **Sockets réseau.** Les bacs à sable Azure sont limités à 1 000 sockets réseau simultanés. Pour plus d’informations, consultez [Limites du service Automation](../../azure-resource-manager/management/azure-subscription-service-limits.md#automation-limits).

* **Module incompatible.** Les dépendances du module sont peut-être incorrectes. Dans ce cas, votre runbook retourne généralement un message `Command not found` ou `Cannot bind parameter`.

* **Aucune authentification auprès d’Active Directory pour le bac à sable.** Votre runbook a tenté d’appeler un fichier exécutable ou un sous-processus qui s’exécute dans un bac à sable Azure. La configuration de runbooks pour s’authentifier auprès d’Azure AD en utilisant la Bibliothèque d’authentification Azure Active Directory (ADAL) n’est pas prise en charge.

### <a name="resolution"></a>Résolution

* **Limite de mémoire, sockets réseau.** Les méthodes suggérées pour respecter les limites de mémoire consistent à diviser la charge de travail entre plusieurs runbooks, à traiter moins de données en mémoire, à éviter d’écrire une sortie inutile depuis vos runbooks et à prendre en compte le nombre de points de contrôle écrits dans vos runbooks de workflow PowerShell. Utilisez la méthode de vidage, telle que `$myVar.clear`, pour effacer les variables et utilisez `[GC]::Collect` pour exécuter immédiatement un nettoyage de la mémoire. Cela permet de réduire l’empreinte mémoire de votre runbook pendant l’exécution.

* **Module incompatible.** Mettez à jour vos modules Azure en suivant les étapes décrites dans [Guide pratique pour mettre à jour des modules Azure PowerShell dans Azure Automation](../automation-update-azure-modules.md).

* **Aucune authentification auprès d’Active Directory pour le bac à sable.** Lorsque vous vous authentifiez auprès d’Azure AD avec un runbook, vérifiez que le module Azure AD est disponible dans votre compte Automation. Veillez à accorder au compte d’identification les autorisations nécessaires pour effectuer les tâches que le runbook automatise.

  Si votre runbook ne peut pas appeler un exécutable ou un sous-processus s’exécutant dans un bac à sable Azure, utilisez le runbook sur un [Runbook Worker hybride](../automation-hrw-run-runbooks.md). Les Workers hybrides ne sont pas restreints par les limites de mémoire et réseau associées aux bacs à sable Azure.

## <a name="scenario-powershell-job-fails-with-cannot-invoke-method-error-message"></a><a name="cannot-invoke-method"></a>Scénario : Le travail PowerShell échoue avec le message d’erreur « Impossible d’appeler la méthode »

### <a name="issue"></a>Problème

Vous recevez le message d’erreur suivant lorsque vous démarrez un travail PowerShell dans un runbook qui s’exécute dans Azure :

```error
Exception was thrown - Cannot invoke method. Method invocation is supported only on core types in this language mode.
```

### <a name="cause"></a>Cause

Cette erreur peut indiquer que l’exécution de runbooks dans un bac à sable (sandbox) Azure n’est pas possible dans le [mode Langage complet](/powershell/module/microsoft.powershell.core/about/about_language_modes).

### <a name="resolution"></a>Résolution

Il existe deux façons de résoudre cette erreur :

* Au lieu d’utiliser [Start-Job](/powershell/module/microsoft.powershell.core/start-job), utilisez [Start-AzAutomationRunbook](/powershell/module/az.automation/start-azautomationrunbook) pour démarrer le runbook.
* Exécutez le runbook sur un Runbook Worker hybride.

Pour plus d’informations sur ce comportement et d’autres comportements des runbooks Azure Automation, consultez [Exécution d’un Runbook dans Azure Automation](../automation-runbook-execution.md).

## <a name="scenario-a-long-running-runbook-fails-to-complete"></a><a name="long-running-runbook"></a>Scénario : Échec de l’exécution d’un long runbook

### <a name="issue"></a>Problème

Votre runbook affiche l’état Stopped (Arrêté) au bout de trois heures d’exécution. Il est également possible que vous obteniez cette erreur :

```error
The job was evicted and subsequently reached a Stopped state. The job cannot continue running.
```

Il s’agit du comportement par défaut dans les bacs à sable Azure en raison de la supervision de la [répartition de charge équilibrée](../automation-runbook-execution.md#fair-share) des processus au sein d’Azure Automation. Si un processus s’exécute pendant plus de trois heures, la répartition de charge équilibrée arrête automatiquement un runbook. L’état d’un runbook qui dépasse la limite de temps de la répartition de charge équilibrée diffère selon le type de runbook. Les runbooks PowerShell et Python sont mis à l’état Stopped (Arrêté). Les runbooks PowerShell Workflow sont mis à l’état Failed (Échec).

### <a name="cause"></a>Cause

Le runbook s’est exécuté au-delà de la limite de trois heures autorisée par la répartition de charge équilibrée dans un bac à sable Azure.

### <a name="resolution"></a>Résolution

Une solution recommandée consiste à exécuter le runbook sur un [Runbook Worker hybride](../automation-hrw-run-runbooks.md). Les Workers hybrides ne sont pas restreints par la limite d’exécution des runbooks de trois heures, qui s’applique aux bacs à sable Azure. Les runbooks qui s’exécutent sur les runbooks Workers hybrides doivent néanmoins être développés pour prendre en charge les comportements de redémarrage en cas de problème inattendu avec l’infrastructure locale.

Une autre solution consiste à optimiser le runbook en créant des [runbooks enfants](../automation-child-runbooks.md). Si votre runbook exécute une boucle via la même fonction sur plusieurs ressources, comme une opération de base de données sur diverses bases de données, vous pouvez déplacer cette fonction vers un runbook enfant. Chaque runbook enfant s’exécute en parallèle dans un processus distinct. Ce comportement réduit la quantité totale de temps pour l’exécution du runbook parent.

Applets de commande PowerShell prenant en charge le scénario avec des runbooks enfants :

* [Start-AzAutomationRunbook](/powershell/module/Az.Automation/Start-AzAutomationRunbook). Cette applet de commande vous permet de démarrer un runbook et de lui transmettre des paramètres.
* [Get-AzAutomationJob](/powershell/module/Az.Automation/Get-AzAutomationJob). Si des opérations doivent être effectuées à la fin de l’exécution du runbook enfant, cette applet de commande vous permet de vérifier l’état du travail de chaque enfant.

## <a name="scenario-error-in-job-streams-about-the-get_serializationsettings-method"></a><a name="get-serializationsettings"></a>Scénario : Une erreur se produit dans les flux de travaux en lien avec la méthode get_SerializationSettings

### <a name="issue"></a>Problème

Vous voyez l’erreur suivante dans votre flux de travail concernant un runbook :

```error
Connect-AzAccount : Method 'get_SerializationSettings' in type
'Microsoft.Azure.Management.Internal.Resources.ResourceManagementClient' from assembly
'Microsoft.Azure.Commands.ResourceManager.Common, Version=4.0.0.0, Culture=neutral, PublicKeyToken=31bf3856ad364e35'
does not have an implementation.
At line:16 char:1
+ Connect-AzAccount -ServicePrincipal -Tenant $Conn.TenantID -Appl ...
+ ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
    + CategoryInfo          : NotSpecified: (:) [Connect-AzAccount], TypeLoadException
    + FullyQualifiedErrorId : System.TypeLoadException,Microsoft.Azure.Commands.Profile.ConnectAzAccountCommand
```

### <a name="cause"></a>Cause

Cette erreur est probablement due à l’utilisation d’une migration incomplète d’AzureRM vers les modules Az dans votre runbook. Cette situation peut amener Azure Automation à démarrer un travail de runbook en utilisant uniquement des modules AzureRM, puis à démarrer un autre travail en utilisant uniquement des modules Az, ce qui aboutit à un incident de bac à sable (sandbox).

### <a name="resolution"></a>Résolution

Nous vous déconseillons d’utiliser des cmdlets Az et AzureRM dans le même runbook. Pour en savoir plus sur l’utilisation correcte de ces modules, consultez [Migrer vers AZ modules](../shared-resources/modules.md#migrate-to-az-modules).

## <a name="scenario-access-denied-when-using-azure-sandbox-for-runbook-or-application"></a><a name="access-denied-azure-sandbox"></a>Scénario : Accès refusé lors de l’utilisation d’un bac à sable (sandbox) Azure pour un runbook ou une application

### <a name="issue"></a>Problème

Lorsque votre runbook ou votre application tente de s’exécuter dans un bac à sable (sandbox) Azure, l’environnement refuse l’accès.

### <a name="cause"></a>Cause

Ce problème peut se produire, car les bacs à sable (sandboxes) Azure empêchent l’accès à tous les serveurs COM hors processus. Par exemple, un runbook ou une application sandbox ne peut pas appeler Windows Management Instrumentation (WMI) ni le service Windows Installer (msiserver.exe).

### <a name="resolution"></a>Résolution

Pour plus d’informations sur l’utilisation de bacs à sable (sandbox) Azure, consultez [Environnement d’exécution de runbook](../automation-runbook-execution.md#runbook-execution-environment).

## <a name="scenario-invalid-forbidden-status-code-when-using-key-vault-inside-a-runbook"></a>Scénario : Code d’état Interdit non valide lors de l’utilisation de Key Vault à l’intérieur d’un runbook

### <a name="issue"></a>Problème

Lorsque vous tentez d’accéder à Azure Key Vault à l’aide d’un runbook Azure Automation, vous obtenez l’erreur suivante :

```error
Operation returned an invalid status code 'Forbidden'
```

### <a name="cause"></a>Cause

Les causes possibles de ce problème sont :

* N’utilise pas de compte d’identification.
* Autorisations insuffisantes.

### <a name="resolution"></a>Résolution

#### <a name="not-using-a-run-as-account"></a>N’utilise pas un compte d’identification.

Suivez les instruction de l’[Étape 5 : Ajouter l’authentification pour gérer les ressources Azure](../learn/automation-tutorial-runbook-textual-powershell.md#step-5---add-authentication-to-manage-azure-resources) pour vous vérifier que vous utilisez un compte d’identification pour accéder à Key Vault.

#### <a name="insufficient-permissions"></a>Autorisations insuffisantes

[Ajoutez des autorisations à Key Vault](../manage-runas-account.md#add-permissions-to-key-vault) pour vous assurer que votre compte d’identification dispose d’autorisations suffisantes pour accéder à Key Vault.

## <a name="recommended-documents"></a>Documents recommandés

* [Exécution d’un runbook dans Azure Automation](../automation-runbook-execution.md)
* [Démarrage d’un Runbook dans Azure Automation](../start-runbooks.md)

## <a name="next-steps"></a>Étapes suivantes

Si votre problème ne figure pas dans cet article, ou si vous ne parvenez pas à le résoudre, utilisez un des canaux suivants pour obtenir de l’assistance :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Connectez-vous à [@AzureSupport](https://twitter.com/azuresupport), le compte Microsoft Azure officiel pour améliorer l’expérience client. Le Support Azure vous met en relation avec la communauté Azure pour obtenir des réponses, un support technique et des experts.
* Si vous avez besoin de plus d’aide, vous pouvez signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.

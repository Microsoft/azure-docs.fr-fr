---
title: Résoudre les problèmes de ressources partagées Azure Automation
description: Cet article explique comment résoudre des problèmes liés aux ressources partagées Azure Automation.
services: automation
author: mgoedtel
ms.author: magoedte
ms.date: 03/12/2019
ms.topic: conceptual
ms.service: automation
manager: carmonm
ms.openlocfilehash: c6bdc09d37cf29458346eaea360b4cd9e0d1226f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86187164"
---
# <a name="troubleshoot-shared-resource-issues"></a>Résoudre les problèmes liés aux ressources partagées

Cet article aborde des problèmes que vous pourriez rencontrer lors de l’utilisation des [ressources partagées](../automation-intro.md#shared-resources) dans Azure Automation.

## <a name="modules"></a>Modules

### <a name="scenario-a-module-is-stuck-during-import"></a><a name="module-stuck-importing"></a>Scénario : Module bloqué pendant l’importation

#### <a name="issue"></a>Problème

Un module est bloqué à l’état *Importation* quand vous importez ou mettez à jour vos modules Azure Automation.

#### <a name="cause"></a>Cause

Comme l’importation de modules PowerShell est un processus complexe en plusieurs étapes, un module peut ne pas être importé correctement et être bloqué dans un état transitoire. Pour plus d’informations sur le processus d’importation, consultez [Importation d’un module PowerShell](/powershell/scripting/developer/module/importing-a-powershell-module#the-importing-process).

#### <a name="resolution"></a>Résolution

Pour résoudre ce problème, vous devez supprimer le module bloqué en utilisant l’applet de commande [Remove-AzAutomationModule](/powershell/module/Az.Automation/Remove-AzAutomationModule?view=azps-3.7.0). Vous pouvez ensuite réessayer d’importer le module.

```azurepowershell-interactive
Remove-AzAutomationModule -Name ModuleName -ResourceGroupName ExampleResourceGroup -AutomationAccountName ExampleAutomationAccount -Force
```

### <a name="scenario-azurerm-modules-are-stuck-during-import-after-an-update-attempt"></a><a name="update-azure-modules-importing"></a>Scénario : Les modules AzureRM sont bloqués pendant l’importation après une tentative de mise à jour

#### <a name="issue"></a>Problème

Une bannière contenant le message suivant reste dans votre compte après la tentative de mise à jour de vos modules AzureRM :

```error
Azure modules are being updated
```

#### <a name="cause"></a>Cause

Il existe un problème connu lié à la mise à jour des modules AzureRM dans un compte Automation. Plus précisément, le problème se produit quand les modules se trouvent dans un groupe de ressources dont le nom numérique commence par 0.

#### <a name="resolution"></a>Résolution

Pour mettre à jour vos modules AzureRM dans votre compte Automation, celui-ci doit appartenir à un groupe de ressources portant un nom alphanumérique. Pour le moment, les groupes de ressources dont les noms numériques commencent par 0 ne peuvent pas mettre à jour les modules AzureRM.

### <a name="scenario-module-fails-to-import-or-cmdlets-cant-be-executed-after-importing"></a><a name="module-fails-to-import"></a>Scénario : Le module ne parvient pas à terminer l’importation ou il est impossible d’exécuter des cmdlets après l’importation

#### <a name="issue"></a>Problème

Un module ne parvient pas à importer ou réussit l’importation, mais aucune applet de commande n’est extraite.

#### <a name="cause"></a>Cause

Voici quelques raisons courantes pour lesquelles l’importation d’un module dans Azure Automation peut échouer :

* La structure ne correspond pas à celle dont Automation a besoin.
* Le module dépend d’un autre module qui n’a pas été déployé sur votre compte Automation.
* Le module n’a pas de dépendances dans le dossier.
* L’applet de commande [New-AzAutomationModule](/powershell/module/Az.Automation/New-AzAutomationModule?view=azps-3.7.0) est utilisée pour charger le module, et vous n’avez pas spécifié le chemin de stockage complet ou vous n’avez pas chargé le module en utilisant une URL accessible publiquement.

#### <a name="resolution"></a>Résolution

Utilisez l’une de ces solutions pour corriger le problème :

* Assurez-vous que le module respecte le format suivant : NomModule.zip -> NomModule ou Numéro de Version -> (NomModule.psm1, NomModule.psd1).
* Ouvrez le fichier **.psd1** et regardez si le module possède des dépendances. Si c’est le cas, téléchargez ces modules dans le compte Automation.
* Assurez-vous que les fichiers **.dll** référencés sont présents dans le dossier du module.

### <a name="scenario-update-azuremoduleps1-suspends-when-updating-modules"></a><a name="all-modules-suspended"></a>Scénario : L’exécution d’Update-AzureModule.ps1 est suspendue lors de la mise à jour des modules

#### <a name="issue"></a>Problème

Lorsque vous utilisez le runbook [Update-AzureModule.ps1](https://github.com/azureautomation/runbooks/blob/master/Utility/ARM/Update-AzureModule.ps1) pour mettre à jour vos modules Azure, le processus de mise à jour du module est interrompu.

#### <a name="cause"></a>Cause

Pour ce runbook, le paramètre par défaut permettant de déterminer le nombre de modules mis à jour simultanément est de 10. Le processus de mise à jour est sujet aux erreurs lorsque les modules mis à jour en même temps sont en nombre excessif.

#### <a name="resolution"></a>Résolution

Il n’est pas courant que tous les modules AzureRM ou Az soient requis dans le même compte Automation. Vous ne devez importer que les modules dont vous avez besoin.

> [!NOTE]
> Évitez d’importer la totalité du module `Az.Automation` ou `AzureRM.Automation`, qui importe tous les modules qu’il contient.

Si le processus de mise à jour est interrompu, ajoutez le paramètre `SimultaneousModuleImportJobCount` au script **Update-AzureModules.ps1** et indiquez une valeur inférieure à la valeur par défaut, à savoir 10. Si vous implémentez cette logique, essayez de commencer avec la valeur 3 ou 5. `SimultaneousModuleImportJobCount` est un paramètre du runbook système **Update-AutomationAzureModulesForAccount** qui est utilisé pour mettre à jour les modules Azure. Si vous effectuez ce réglage, le processus de mise à jour est plus long, mais il a plus de chances d’aboutir. L’exemple suivant montre le paramètre et où le mettre dans le runbook :

 ```powershell
         $Body = @"
            {
               "properties":{
               "runbook":{
                   "name":"Update-AutomationAzureModulesForAccount"
               },
               "parameters":{
                    ...
                    "SimultaneousModuleImportJobCount":"3",
                    ... 
               }
              }
           }
"@
```

## <a name="run-as-accounts"></a>Compte d’identification

### <a name="scenario-youre-unable-to-create-or-update-a-run-as-account"></a><a name="unable-create-update"></a>Scénario : Vous ne pouvez pas créer ou mettre à jour un compte d’identification

#### <a name="issue"></a>Problème

Quand vous tentez de créer ou de mettre à jour un compte d’identification, vous recevez une erreur similaire à celle-ci :

```error
You do not have permissions to create…
```

#### <a name="cause"></a>Cause

Vous ne disposez pas des autorisations nécessaires pour créer ou mettre à jour le compte d’identification, ou la ressource est verrouillée au niveau du groupe de ressources.

#### <a name="resolution"></a>Résolution

Pour créer ou mettre à jour d’un compte d’identification, vous devez disposer des [autorisations](../manage-runas-account.md#permissions) appropriées pour les diverses ressources utilisées par le compte d’identification. 

Si le problème est dû à un verrou, vérifiez que le verrou peut être supprimé. Accédez ensuite à la ressource verrouillée dans le portail Azure, cliquez avec le bouton droit sur le verrou, puis sélectionnez **Supprimer**.

### <a name="scenario-you-receive-the-error-unable-to-find-an-entry-point-named-getperadapterinfo-in-dll-iplpapidll-when-executing-a-runbook"></a><a name="iphelper"></a>Scénario : Vous recevez l’erreur « Impossible de trouver le point d’entrée "GetPerAdapterInfo" dans la DLL "iplpapi.dll" » lors de l’exécution d’un runbook

#### <a name="issue"></a>Problème

Lors de l’exécution d’un runbook, vous recevez l’exception suivante :

```error
Unable to find an entry point named 'GetPerAdapterInfo' in DLL 'iplpapi.dll'
```

#### <a name="cause"></a>Cause

Cette erreur est probablement due à un [Compte d’identification](../manage-runas-account.md) mal configuré.

#### <a name="resolution"></a>Résolution

Assurez-vous que votre compte d’identification est correctement configuré. Vérifiez ensuite que votre runbook contient le code approprié pour vous authentifier auprès d’Azure. L’exemple suivant montre un extrait de code permettant de s’authentifier auprès d’Azure dans un runbook à l’aide d’un compte d’identification.

```powershell
$connection = Get-AutomationConnection -Name AzureRunAsConnection
Connect-AzAccount -ServicePrincipal -Tenant $connection.TenantID `
-ApplicationID $connection.ApplicationID -CertificateThumbprint $connection.CertificateThumbprint
```

## <a name="next-steps"></a>Étapes suivantes

Si cet article ne vous permet pas de résoudre votre problème, utilisez l’un des canaux suivants pour obtenir une aide supplémentaire :

* Obtenez des réponses de la part d’experts Azure via les [Forums Azure](https://azure.microsoft.com/support/forums/).
* Contactez [@AzureSupport](https://twitter.com/azuresupport). Il s’agit du compte Microsoft Azure officiel qui permet à la communauté Azure d’accéder aux ressources dont elle a besoin : réponses, support et experts.
* Signaler un incident au support Azure Accédez au [site du support Azure](https://azure.microsoft.com/support/options/), puis sélectionnez **Obtenir de l’aide**.


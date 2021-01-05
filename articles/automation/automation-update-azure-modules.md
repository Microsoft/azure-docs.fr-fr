---
title: Mettre à jour des modules Azure PowerShell dans Azure Automation
description: Cet article explique comment mettre à jour des modules Azure PowerShell courants fournis par défaut dans Azure Automation.
services: automation
ms.subservice: process-automation
ms.date: 06/14/2019
ms.topic: conceptual
ms.openlocfilehash: 9cc5e5107784d447640ef0ae77d412885b1e5668
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86185770"
---
# <a name="update-azure-powershell-modules"></a>Mettre à jour les modules Azure PowerShell

Les modules PowerShell les plus courants sont fournis par défaut dans chaque compte Automation. Voir [Modules par défaut](shared-resources/modules.md#default-modules). À mesure que l’équipe Azure met à jour les modules Azure, des modifications peuvent se produire avec les cmdlets incluses. Ces modifications, par exemple, en renommant un paramètre ou en déconseillant entièrement une cmdlet, peuvent avoir un impact négatif sur vos runbooks. 

> [!NOTE]
> Vous ne pouvez pas supprimer des modules globaux, qui sont des modules prêts à l’emploi fournis par Automation.

## <a name="set-up-an-automation-account"></a>Configurer un compte Automation

Pour éviter toute incidence sur vos runbooks et sur les processus qu’ils automatisent, veillez à tester et valider les mises à jour. Si vous n’avez pas de compte Automation dédié destiné à cet usage, créez-en un pour pouvoir tester différents scénarios pendant le développement de vos runbooks. Ces tests doivent comprendre des changements itératifs, comme la mise à jour des modules PowerShell.

Assurez-vous que votre compte Automation dispose d’[informations d’identification de compte d’identification Azure](manage-runas-account.md).

Si vous développez vos scripts localement, il est recommandé de disposer localement des mêmes versions de module que celles disponibles dans votre compte Automation à des fins de test, pour être assuré d’obtenir les mêmes résultats. Une fois que les résultats sont validés et que vous avez appliqué les changements nécessaires, vous pouvez les passer en production.

> [!NOTE]
> Un nouveau compte Automation peut ne pas contenir les derniers modules.

## <a name="obtain-a-runbook-to-use-for-updates"></a>Obtenir un runbook à utiliser pour les mises à jour

Pour mettre à jour les modules Azure dans votre compte Automation, vous devez utiliser le runbook [Update-AutomationAzureModulesForAccount](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update) disponible en open source. Pour commencer à utiliser ce runbook pour mettre à jour vos modules Azure, téléchargez-le à partir du dépôt GitHub. Vous pouvez ensuite l’importer dans votre compte Automation ou l’exécuter en tant que script. Pour savoir comment importer un runbook dans votre compte Automation, consultez [Importer un runbook](manage-runbooks.md#import-a-runbook).

Le runbook **Update-AutomationAzureModulesForAccount** prend en charge la mise à jour des modules Azure, AzureRM et Az par défaut. Consultez le [fichier Lisez-moi du runbook Mettre à jour les modules Azure](https://github.com/microsoft/AzureAutomation-Account-Modules-Update/blob/master/README.md) pour plus d’informations sur la mise à jour des modules Az.Automation avec ce runbook. Il existe d’autres facteurs importants que vous devez prendre en compte lorsque vous utilisez les modules Az dans votre compte Automation. Pour en savoir plus, consultez [Gérer les modules dans Azure Automation](shared-resources/modules.md).

## <a name="use-update-runbook-code-as-a-regular-powershell-script"></a>Utiliser une code de runbook de mise à jour comme script PowerShell standard

Vous pouvez utiliser le code de runbook comme un script PowerShell ordinaire plutôt que comme runbook. Pour ce faire, commencez par vous connecte à Azure à l’aide de la cmdlet [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount?view=azps-3.7.0), puis transmettez `-Login $false` au script.

## <a name="use-the-update-runbook-on-sovereign-clouds"></a>Utiliser le runbook de mise à jour sur des clouds souverains

Pour utiliser ce runbook sur des clouds souverains, utilisez le paramètre `AzEnvironment` pour transmettre l’environnement approprié au runbook. Les valeurs acceptables sont AzureCloud (cloud public Azure), AzureChinaCloud, AzureGermanCloud et AzureUSGovernment. Ces valeurs peuvent être récupérées en utilisant `Get-AzEnvironment | select Name`. Si vous ne transmettez pas de valeur à cette cmdlet, la valeur par défaut du runbook est AzureCloud.

## <a name="use-the-update-runbook-to-update-a-specific-module-version"></a>Utiliser le runbook de mise à jour pour mettre à jour une version de module spécifique

Si vous souhaitez utiliser une version spécifique du module Azure PowerShell au lieu du module le plus récent disponible sur PowerShell Gallery, transmettez ces versions au paramètre facultatif `ModuleVersionOverrides` du runbook **Update-AutomationAzureModulesForAccount**. Pour obtenir des exemples, consultez le runbook [Update-AutomationAzureModulesForAccount.ps1](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update/blob/master/Update-AutomationAzureModulesForAccount.ps1). Les modules Azure PowerShell qui ne sont pas mentionnés dans le paramètre `ModuleVersionOverrides` sont mis à jour avec les dernières versions de module sur PowerShell Gallery. Si vous ne passez aucune valeur au paramètre `ModuleVersionOverrides`, tous les modules sont mis à jour avec les dernières versions de module sur PowerShell Gallery. Ce comportement est le même que celui du bouton **Mettre à jour les modules Azure** dans le portail Azure.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation des modules, consultez [Gérer les modules dans Azure Automation](shared-resources/modules.md).
* Pour plus d’informations sur le runbook de mise à jour, consultez [Runbook Mettre à jour les modules Azure](https://github.com/Microsoft/AzureAutomation-Account-Modules-Update).

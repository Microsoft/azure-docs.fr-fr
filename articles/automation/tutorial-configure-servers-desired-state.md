---
title: Configurer les machines à l’état souhaité dans Azure Automation
description: Cet article explique comment configurer des ordinateurs à un état souhaité à l’aide de la fonction State Configuration d’Azure Automation.
services: automation
ms.subservice: dsc
ms.topic: conceptual
ms.date: 04/15/2021
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d29c8ec4e0b992f38eec9e203ad6ad302f71308b
ms.sourcegitcommit: 2f322df43fb3854d07a69bcdf56c6b1f7e6f3333
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/27/2021
ms.locfileid: "108018490"
---
# <a name="configure-machines-to-a-desired-state"></a>Configurer les machines à l’état souhaité

Azure Automation State Configuration vous permet de spécifier des configurations pour vos serveurs et de vérifier que leur état ainsi défini perdure dans le temps.

> [!div class="checklist"]
> - Intégrer une machine virtuelle à gérer par Azure Automation DSC
> - Charger une configuration dans Azure Automation
> - Compiler une configuration dans une configuration de nœud
> - Attribuer une configuration de nœud à un nœud géré
> - Vérifier l’état de conformité d’un nœud géré

Dans ce tutoriel, nous utilisons une [configuration DSC](/powershell/scripting/dsc/configurations/configurations) simple qui permet de veiller à ce qu’IIS soit installé sur la machine virtuelle.

## <a name="prerequisites"></a>Prérequis

- Un compte Azure Automation. Pour en savoir plus sur un compte Automation et ses exigences, consultez [Vue d’ensemble de l’authentification du compte Automation](./automation-security-overview.md).
- Une machine virtuelle Azure Resource Manager (et non Classic) fonctionnant sur Windows Server 2008 R2 ou version ultérieure. Pour obtenir des instructions sur la création d’une machine virtuelle, consultez [Création d’une première machine virtuelle Windows sur le Portail Azure](../virtual-machines/windows/quick-create-portal.md).
- Module Azure PowerShell, version 3.6 ou ultérieure. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/azurerm/install-azurerm-ps).
- Des notions de base de la configuration de l’état souhaité. Pour plus d’informations sur la configuration DSC, consultez [Vue d’ensemble de la fonctionnalité Desired State Configuration de Windows PowerShell](/powershell/scripting/dsc/overview/overview).

## <a name="support-for-partial-configurations"></a>Prise en charge des configurations partielles

Azure Automation State Configuration prend en charge l’utilisation de [configurations partielles](/powershell/scripting/dsc/pull-server/partialconfigs). Dans ce scénario, DSC est configuré pour gérer plusieurs configurations de manière indépendante, et chaque configuration est récupérée à partir d’Azure Automation. Toutefois, on ne peut affecter qu’une seule configuration à un nœud par compte Automation. Par conséquent, si vous utilisez deux configurations pour un nœud, vous aurez besoin de deux comptes Automation.

Pour savoir comment inscrire une configuration partielle à partir d’un service Pull, consultez la documentation relative aux [configurations partielles](/powershell/scripting/dsc/pull-server/partialconfigs#partial-configurations-in-pull-mode).

Pour plus d’informations sur la collaboration possible entre équipes dans le but de gérer les serveurs avec la configuration en tant que code, consultez [Présentation du rôle de la configuration DSC dans un pipeline CI/CD](/powershell/scripting/dsc/overview/authoringadvanced).

## <a name="log-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure avec la cmdlet [Connect-AzAccount](/powershell/module/Az.Accounts/Connect-AzAccount) et suivez les instructions qui s’affichent à l’écran.

```powershell
Connect-AzAccount
```

## <a name="create-and-upload-a-configuration-to-azure-automation"></a>Créer et charger une configuration dans Azure Automation

Dans un éditeur de texte, tapez le code suivant et enregistrez-le localement sous le nom **TestConfig.ps1**.

```powershell
configuration TestConfig {
   Node WebServer {
      WindowsFeature IIS {
         Ensure               = 'Present'
         Name                 = 'Web-Server'
         IncludeAllSubFeature = $true
      }
   }
}
```

> [!NOTE]
> Dans les scénarios plus avancés où vous devez importer plusieurs modules qui fournissent des ressources DSC, vérifiez que chaque module a une ligne `Import-DscResource` unique dans votre configuration.

Appelez la cmdlet [Import-AzAutomationDscConfiguration](/powershell/module/Az.Automation/Import-AzAutomationDscConfiguration) pour charger la configuration dans votre compte Automation.

```powershell
 Import-AzAutomationDscConfiguration -SourcePath 'C:\DscConfigs\TestConfig.ps1' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Published
```

## <a name="compile-a-configuration-into-a-node-configuration"></a>Compiler une configuration dans une configuration de nœud

Une configuration DSC doit être compilée dans une configuration de nœud avant de pouvoir être affectée à un nœud. Consultez [Configurations DSC](/powershell/scripting/dsc/configurations/configurations).

Appelez la cmdlet [Start-AzAutomationDscCompilationJob](/powershell/module/Az.Automation/Start-AzAutomationDscCompilationJob) pour compiler la configuration `TestConfig` dans une configuration de nœud nommée `TestConfig.WebServer` au sein de votre compte Automation.

```powershell
Start-AzAutomationDscCompilationJob -ConfigurationName 'TestConfig' -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount'
```

## <a name="register-a-vm-to-be-managed-by-state-configuration"></a>Inscrire une machine virtuelle devant être gérée par State Configuration

Azure Automation State Configuration vous permet de gérer vos machines virtuelles Azure (par le biais des modèles de déploiement classique et Resource Manager), vos machines virtuelles locales, vos machines Linux, vos machines virtuelles AWS et vos ordinateurs physiques en local. Dans cette rubrique, nous allons voir comment inscrire uniquement des machines virtuelles Azure Resource Manager. Pour plus d’informations sur l’inscription d’autres types de machines, consultez [Intégration de machines pour la gestion avec Azure Automation State Configuration](automation-dsc-onboarding.md).

Appelez la cmdlet [Register-AzAutomationDscNode](/powershell/module/Az.Automation/Register-AzAutomationDscNode) pour inscrire votre machine virtuelle auprès d’Azure Automation State Configuration en tant que nœud géré. 

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm'
```

### <a name="specify-configuration-mode-settings"></a>Spécifier les paramètres du mode de configuration

Utilisez la cmdlet [Register-AzAutomationDscNode](/powershell/module/azurerm.automation/register-azurermautomationdscnode) pour inscrire une machine virtuelle en tant que nœud géré et indiquer des propriétés de configuration. Par exemple, vous pouvez spécifier que l’état de la machine ne doit être appliqué qu’une seule fois en choisissant `ApplyOnly` comme valeur de la propriété `ConfigurationMode`. State Configuration n’essaie pas d’appliquer la configuration après la vérification initiale.

```powershell
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationMode 'ApplyOnly'
```

Vous pouvez également préciser la fréquence à laquelle DSC vérifie l’état de configuration à l’aide de la propriété `ConfigurationModeFrequencyMins`. Pour plus d’informations sur les paramètres de configuration DSC, consultez [Configuration du Gestionnaire de configuration local](/powershell/scripting/dsc/managing-nodes/metaConfig).

```powershell
# Run a DSC check every 60 minutes
Register-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -AzureVMName 'DscVm' -ConfigurationModeFrequencyMins 60
```

## <a name="assign-a-node-configuration-to-a-managed-node"></a>Attribuer une configuration de nœud à un nœud géré

Nous pouvons désormais attribuer la configuration de nœud compilé à la machine virtuelle que nous souhaitons configurer.

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Assign the node configuration to the DSC node
Set-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeConfigurationName 'TestConfig.WebServer' -NodeId $node.Id
```

Cette opération attribue la configuration de nœud nommée `TestConfig.WebServer` au nœud DSC inscrit `DscVm`. Par défaut, le nœud DSC est vérifié pour la conformité avec la configuration de nœud toutes les 30 minutes. Pour obtenir des informations sur la modification de l’intervalle de vérification de conformité, consultez [Configuration du Gestionnaire de configuration local](/powershell/scripting/dsc/managing-nodes/metaConfig).

## <a name="check-the-compliance-status-of-a-managed-node"></a>Vérifier l’état de conformité d’un nœud géré

Pour obtenir des rapports sur l’état de conformité d’un nœud géré, utilisez la cmdlet [Get-AzAutomationDscNodeReport](/powershell/module/Az.Automation/Get-AzAutomationDscNodeReport).

```powershell
# Get the ID of the DSC node
$node = Get-AzAutomationDscNode -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -Name 'DscVm'

# Get an array of status reports for the DSC node
$reports = Get-AzAutomationDscNodeReport -ResourceGroupName 'MyResourceGroup' -AutomationAccountName 'myAutomationAccount' -NodeId $node.Id

# Display the most recent report
$reports[0]
```

## <a name="next-steps"></a>Étapes suivantes

- Pour commencer, consultez [Bien démarrer avec Azure Automation State Configuration](automation-dsc-getting-started.md).
- Pour savoir comment activer des nœuds, voir [Activer State Configuration](automation-dsc-onboarding.md).
- Pour découvrir comment compiler des configurations DSC pour pouvoir les affecter à des nœuds cibles, voir [Compiler des configurations DSC dans Azure Automation State Configuration](automation-dsc-compile.md).
- Pour voir un exemple d’utilisation d’Azure Automation State Configuration dans un pipeline de déploiement continu, consultez [Configurer un déploiement continu avec Chocolatey](automation-dsc-cd-chocolatey.md).
- Pour obtenir des informations sur les prix, consultez [Tarification d’Azure Automation State Configuration](https://azure.microsoft.com/pricing/details/automation/).
- Pour obtenir des informations de référence sur les applets de commande PowerShell, consultez [Az.Automation](/powershell/module/az.automation).

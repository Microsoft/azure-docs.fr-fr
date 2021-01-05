---
title: Déployer un Runbook Worker hybride Windows dans Azure Automation
description: Cet article décrit comment déployer un Runbook Worker hybride qui vous permet d’exécuter des runbooks sur des ordinateurs Windows de votre centre de données local ou de votre environnement cloud.
services: automation
ms.subservice: process-automation
ms.date: 11/24/2020
ms.topic: conceptual
ms.openlocfilehash: e77a90617d79dab8c71cdf0c7a6a4fb21e052fbd
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96182782"
---
# <a name="deploy-a-windows-hybrid-runbook-worker"></a>Déployer un Runbook Worker hybride Windows

Vous pouvez utiliser la fonctionnalité Runbook Worker hybride utilisateur d’Azure Automation pour exécuter des runbooks directement sur la machine Azure ou non-Azure, y compris les serveurs inscrits auprès de [serveurs avec Azure Arc](../azure-arc/servers/overview.md). Sur l’ordinateur ou le serveur qui héberge le rôle, vous pouvez exécuter les runbooks directement avec les ressources disponibles dans l’environnement pour gérer les ressources locales.

Azure Automation stocke et gère les runbooks, puis les remet à une ou plusieurs machines désignées. Cet article explique comment déployer un Runbook Worker hybride utilisateur sur une machine Windows, comment supprimer le Worker et comment supprimer un groupe de Runbook Workers hybrides.

Une fois le Runbook Worker déployé, consultez [Exécuter des runbooks sur un Runbook Worker hybride](automation-hrw-run-runbooks.md) pour apprendre à configurer vos Runbooks en vue d’automatiser les processus de votre centre de données local ou autre environnement cloud.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants.

### <a name="a-log-analytics-workspace"></a>Un espace de travail Log Analytics

Le rôle Runbook Worker hybride dépend d’un espace de travail Azure Monitor Log Analytics pour l’installation et la configuration du rôle. Vous pouvez le créer en utilisant [Azure Resource Manager](../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace), [PowerShell](../azure-monitor/scripts/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json) ou le [portail Azure](../azure-monitor/learn/quick-create-workspace.md).

Si vous n’avez pas d’espace de travail Azure Monitor Log Analytics, consultez le [guide de conception des journaux Azure Monitor](../azure-monitor/platform/design-logs-deployment.md) avant de créer l’espace de travail.

### <a name="log-analytics-agent"></a>Agent Log Analytics

Le rôle Runbook Worker hybride requiert l’[agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) pour le système d’exploitation Windows pris en charge. Pour les serveurs ou les ordinateurs hébergés en dehors d’Azure, vous pouvez installer l’agent Log Analytics à l’aide de [serveurs avec Azure Arc](../azure-arc/servers/overview.md).

### <a name="supported-windows-operating-system"></a>Système d’exploitation Windows pris en charge

La fonctionnalité de Runbook Worker hybride prend en charge les systèmes d’exploitation suivants :

* Windows Server 2019
* Windows Server 2016, versions 1709 et 1803
* Windows Server 2012, 2012 R2
* Windows Server 2008 SP2 (x64), 2008 R2
* Windows 10 entreprise (y compris multi-sessions) et Pro
* Windows 8 Entreprise et Professionnel
* Windows 7 SP1

### <a name="minimum-requirements"></a>Configuration minimale requise

Voici la configuration minimale requise pour un système Windows et un Runbook Worker hybride utilisateur :

* Windows PowerShell 5.1 ([télécharger WMF 5.1](https://www.microsoft.com/download/details.aspx?id=54616)). PowerShell Core n’est pas pris en charge.
* .NET Framework 4.6.2 ou version ultérieure
* 2 cœurs
* 4 Go de RAM
* Port 443 (sortie)

### <a name="network-configuration"></a>Configuration réseau

Pour connaître la configuration réseau requise pour le Runbook Worker hybride, consultez [Configuration de votre réseau](automation-hybrid-runbook-worker.md#network-planning).

### <a name="adding-a-machine-to-a-hybrid-runbook-worker-group"></a>Ajout d’un ordinateur à un groupe de Runbook Workers hybrides

Vous pouvez ajouter l’ordinateur Worker à un groupe de Runbook Workers hybrides dans l’un de vos comptes Automation. Pour les ordinateurs qui hébergent le Runbook Worker hybride système géré par Update Management, ils peuvent être ajoutés à un groupe de Runbook Workers hybrides. Toutefois, vous devez utiliser le même compte Automation pour Update Management et l’appartenance au groupe de Runbook Workers hybrides.

>[!NOTE]
>Azure Automation [Update Management](./update-management/overview.md) installe automatiquement le Runbook Worker hybride système sur un ordinateur Azure ou non Azure où la fonctionnalité Update Management est activée. Toutefois, ce Worker n’est inscrit auprès d’aucun des groupes de Runbook Workers hybrides figurant dans votre compte Automation. Pour exécuter vos runbooks sur ces ordinateurs, vous devez les ajouter à un groupe de Runbook Workers hybrides. Suivez l’étape 6 de la section [Déploiement manuel](#manual-deployment) pour l’ajouter à un groupe.

## <a name="enable-for-management-with-azure-automation-state-configuration"></a>Activer pour la gestion avec Azure Automation State Configuration

Pour plus d’informations sur l’activation d’ordinateurs en vue d’une gestion avec Azure Automation State Configuration, consultez [Activer la gestion des machines par Azure Automation State Configuration](automation-dsc-onboarding.md).

> [!NOTE]
> Pour gérer la configuration des machines qui prennent en charge le rôle Runbook Worker hybride avec Desired State Configuration (DSC), vous devez ajouter ces machines en tant que nœuds DSC.

## <a name="installation-options"></a>Options d'installation

Pour installer et configurer un Runbook Worker hybride utilisateur Windows, vous pouvez utiliser l’une des méthodes suivantes.

* Utilisez un script PowerShell fourni pour [automatiser](#automated-deployment) complètement le processus de configuration d’un ou plusieurs ordinateurs Windows. Il s’agit de la méthode recommandée pour les machines de votre centre de données ou d’un autre environnement cloud.
* Importez manuellement la solution Automation, installez l’agent Log Analytics pour Windows et configurez le rôle de travail sur l’ordinateur.

## <a name="automated-deployment"></a>Déploiement automatisé

La méthode de déploiement automatisée utilise le script PowerShell **New-OnPremiseHybridWorker.ps1** pour automatiser et configurer le rôle Runbook Worker hybride Windows. Il effectue les opérations suivantes :

* Installe les modules nécessaires
* Se connecte votre compte Azure
* Vérifie l’existence du groupe de ressources spécifié et du compte Automation
* Crée des références à des attributs de compte Automation
* Crée un espace de travail Azure Monitor Log Analytics s’il n’est pas spécifié
* Activer la solution Azure Automation dans l’espace de travail
* Télécharger et Installer l’agent Log Analytics pour Windows
* Inscrire la machine en tant que Runbook Worker hybride

Procédez comme suit pour installer le rôle sur votre ordinateur Windows à l’aide du script.

1. Téléchargez le script **New-OnPremiseHybridWorker.ps1** à partir de [PowerShell Gallery](https://www.powershellgallery.com/packages/New-OnPremiseHybridWorker). Une fois que vous avez téléchargé le script, copiez-le ou exécutez-le sur la machine cible. Le script **New-OnPremiseHybridWorker.ps1** utilise les paramètres suivants lors de l’exécution.

    | Paramètre | Statut | Description |
    | --------- | ------ | ----------- |
    | `AAResourceGroupName` | Obligatoire | nom du groupe de ressources associé à votre compte Automation. |
    | `AutomationAccountName` | Obligatoire | nom de votre compte Automation.
    | `Credential` | Facultatif | Informations d’identification à utiliser lors de la connexion à l’environnement Azure. |
    | `HybridGroupName` | Obligatoire | nom d’un groupe de Runbooks Workers hybrides que vous spécifiez comme cible pour les Runbooks prenant en charge ce scénario. |
    | `OMSResourceGroupName` | Facultatif | nom du groupe de ressources pour l’espace de travail Log Analytics. Si ce groupe de ressources n’est pas spécifié, la valeur de `AAResourceGroupName` est utilisée. |
    | `SubscriptionID` | Obligatoire | Identificateur de l’abonnement Azure associé à votre compte Automation. |
    | `TenantID` | Facultatif | Identificateur de l’organisation locataire associée à votre compte Automation. |
    | `WorkspaceName` | Facultatif | Nom de l’espace de travail Log Analytics. Si vous n’avez pas d’espace de travail Log Analytics, le script en crée un et le configure. |

2. Ouvrez une invite de commandes PowerShell 64 bits avec élévation de privilèges.

3. À partir de l’invite de commandes PowerShell, accédez au dossier contenant le script que vous avez téléchargé. Modifiez les valeurs des paramètres `AutomationAccountName`, `AAResourceGroupName`, `OMSResourceGroupName`, `HybridGroupName`, `SubscriptionID` et `WorkspaceName`. Ensuite, exécutez le script.

    Une fois le script exécuté, vous êtes invité à vous authentifier auprès d’Azure. Vous devez vous connecter avec un compte membre du rôle **Administrateurs des abonnements** et coadministrateur de l’abonnement.

    ```powershell-interactive
    $NewOnPremiseHybridWorkerParameters = @{
      AutomationAccountName = <nameOfAutomationAccount>
      AAResourceGroupName   = <nameOfResourceGroup>
      OMSResourceGroupName  = <nameOfResourceGroup>
      HybridGroupName       = <nameOfHRWGroup>
      SubscriptionID        = <subscriptionId>
      WorkspaceName         = <nameOfLogAnalyticsWorkspace>
    }
    .\New-OnPremiseHybridWorker.ps1 @NewOnPremiseHybridWorkerParameters
    ```

4. Vous êtes invité à accepter l’installation de NuGet et à vous authentifier avec vos informations d’identification Azure. Si vous ne disposez pas de la dernière version de NuGet, vous pouvez le télécharger à partir de la page répertoriant les [versions de distribution NuGet disponibles](https://www.nuget.org/downloads).

5. Vérifiez le déploiement une fois le script terminé. Dans la page **Groupes de Runbook Workers hybrides** de votre compte Automation, sous l’onglet **Groupe de Runbook Workers hybrides utilisateur**, le nouveau groupe et le nombre de membres sont affichés. S’il s’agit d’un groupe existant, le nombre de membres est incrémenté. Vous pouvez sélectionner ce groupe dans la liste de cette page, dans le menu de gauche, choisissez **Workers hybrides**. Dans la page **Workers hybrides**, vous pouvez voir chaque membre du groupe listé.

## <a name="manual-deployment"></a>Déploiement manuel

Pour installer et configurer un runbook Worker hybride pour Windows, effectuez les étapes suivantes.

1. Activez la solution Azure Automation dans votre espace de travail Log Analytics en exécutant la commande suivante dans une invite de commandes PowerShell avec élévation de privilèges ou dans Cloud Shell, dans le [portail Azure](https://portal.azure.com).

    ```powershell
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resourceGroupName> -WorkspaceName <workspaceName> -IntelligencePackName "AzureAutomation" -Enabled $true
    ```

2. Déployez l’agent Log Analytics sur la machine cible.

    * Pour les machines virtuelles Azure, installez l’agent Log Analytics pour Windows à l’aide de l’[extension de machine virtuelle pour Windows](../virtual-machines/extensions/oms-windows.md). L’extension installe l’agent Log Analytics sur les machines virtuelles Azure et inscrit les machines virtuelles dans un espace de travail Log Analytics existant. Vous pouvez utiliser un modèle Azure Resource Manager, PowerShell ou Azure Policy pour attribuer la stratégie intégrée [Déployer l’agent Log Analytics pour les machines virtuelles *Linux* ou *Windows*](../governance/policy/samples/built-in-policies.md#monitoring). Une fois l’agent installé, la machine peut être ajoutée à un groupe de Runbook Workers hybrides dans votre compte Automation.
    
    * Pour les ordinateurs non Azure, vous pouvez installer l’agent Log Analytics à l’aide de [serveurs avec Azure Arc](../azure-arc/servers/overview.md). Les serveurs avec Arc prennent en charge le déploiement de l’agent Log Analytics à l’aide des méthodes suivantes :
    
        - Utilisation de l’infrastructure des extensions de machine virtuelle.
        
            Cette fonctionnalité des serveurs activés pour Azure Arc vous permet de déployer l’extension de machine virtuelle de l’agent Log Analytics sur un serveur Windows et/ou Linux non-Azure. Les extensions de machine virtuelle peuvent être gérées à l’aide des méthodes suivantes sur vos machines ou serveurs hybrides gérés par des serveurs compatibles Arc :
        
            - [Portail Azure](../azure-arc/servers/manage-vm-extensions-portal.md)
            - [Azure CLI](../azure-arc/servers/manage-vm-extensions-cli.md)
            - [Azure PowerShell](../azure-arc/servers/manage-vm-extensions-powershell.md)
            - [Modèles Azure Resource Manager](../azure-arc/servers/manage-vm-extensions-template.md)
        
        - Utilisation d’Azure Policy.
        
            Avec cette approche, vous utilisez la stratégie intégrée [Déployer l’agent Log Analytics sur des machines Linux ou Windows Azure Arc](../governance/policy/samples/built-in-policies.md#monitoring) d’Azure Policy si l’agent Log Analytics est installé sur le serveur activé pour Arc. Si l’agent n’est pas installé, elle le déploie automatiquement en utilisant une tâche de remédiation. Autrement, si vous envisagez de superviser les machines avec Azure Monitor pour machines virtuelles, utilisez plutôt l’initiative [Activer Azure Monitor pour machines virtuelles](../governance/policy/samples/built-in-initiatives.md#monitoring) pour installer et configurer l’agent Log Analytics.

    Nous vous recommandons d’installer l’agent Log Analytics pour Windows ou Linux en utilisant Azure Policy.

3. Vérifiez que l’agent rend compte à l’espace de travail.

    L’agent Log Analytics pour Windows connecte les ordinateurs à un espace de travail Log Analytics Azure Monitor. Lorsque vous installez l’agent sur votre machine et que vous le connectez à votre espace de travail, il télécharge automatiquement les composants nécessaires au runbook Worker hybride.

    Lorsque l’agent s’est correctement connecté à votre espace de travail Log Analytics, après quelques minutes, vous pouvez exécuter la requête suivante pour vérifier qu’il envoie les données de pulsation à l’espace de travail.

    ```kusto
    Heartbeat 
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    Dans les résultats de la recherche, vous voyez en principe des enregistrements de pulsation pour la machine, ce qui indique que l’agent est connecté et qu’il rend compte au service. Par défaut, chaque agent transmet un enregistrement de pulsation à l’espace de travail qui lui est attribué. Utilisez les étapes suivantes pour effectuer l’installation et la configuration de l’agent.

4. Confirmez la version du Runbook Worker hybride sur l’ordinateur hébergeant l’agent Log Analytics, accédez à `C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\` et notez le sous-dossier **version**. Ce dossier apparaîtra sur l’ordinateur plusieurs minutes après l’activation de la solution dans l’espace de travail.

5. Installez l’environnement de runbook et connectez-vous à Azure Automation. Quand vous configurez un agent pour qu’il envoie des rapports à un espace de travail Log Analytics et importez la solution **Automation**, cette dernière envoie (push) le module PowerShell `HybridRegistration`. Ce module contient l’applet de commande `Add-HybridRunbookWorker`. Vous utilisez cette cmdlet pour installer l’environnement du Runbook sur la machine et l’inscrire auprès d’Azure Automation.

    Ouvrez une session PowerShell en mode administrateur et exécutez les commandes suivantes pour importer le module.

    ```powershell-interactive
    cd "C:\Program Files\Microsoft Monitoring Agent\Agent\AzureAutomation\<version>\HybridRegistration"
    Import-Module .\HybridRegistration.psd1
    ```

6. Exécutez l’applet de commande `Add-HybridRunbookWorker` en spécifiant les valeurs des paramètres `Url`, `Key` et `GroupName`.

    ```powershell-interactive
    Add-HybridRunbookWorker –GroupName <String> -Url <Url> -Key <String>
    ```

    Vous trouverez les informations requises pour les paramètres `Url` et `Key` dans la page **Clés** de votre compte Automation. Sélectionnez **Clés** sous la section **Paramètres du compte** sur le côté gauche de la page.

    ![Page Gérer les clés](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Pour le paramètre `Url`, copiez la valeur de l’**URL**.

    * Pour le paramètre `Key`, copiez la valeur de la clé d’accès principale (**PRIMARY ACCESS KEY**).

    * Pour le paramètre `GroupName`, utilisez le nom du groupe de Runbooks Workers hybrides. Si ce groupe existe déjà dans le compte Automation, la machine active y est ajoutée. Si le groupe n’existe pas, il est ajouté.

    * Si nécessaire, définissez le paramètre `Verbose` pour recevoir des détails sur l’installation.

7. Vérifiez le déploiement une fois la commande terminée. Dans la page **Groupes de Runbook Workers hybrides** de votre compte Automation, sous l’onglet **Groupe de Runbook Workers hybrides utilisateur**, le groupe nouveau ou existant et le nombre de membres sont affichés. S’il s’agit d’un groupe existant, le nombre de membres est incrémenté. Vous pouvez sélectionner le groupe dans la liste de cette page en choisissant **Workers hybrides** dans le menu de gauche. Dans la page **Workers hybrides**, vous pouvez voir chaque membre du groupe listé.

## <a name="install-powershell-modules"></a>Installer des modules PowerShell

Les Runbooks peuvent utiliser toutes les activités et applets de commande définies dans les modules installés dans votre environnement Azure Automation. Comme ces modules ne sont pas automatiquement déployés sur des ordinateurs locaux, vous devez les installer manuellement. L’exception est le module Azure. Le module est installé par défaut et permet d’accéder aux cmdlets de l’ensemble des services et activités Azure pour Azure Automation.

Comme l’objectif principal de Runbook Worker hybride est de gérer les ressources locales, vous devrez probablement installer les modules qui prennent en charge ces ressources, en particulier le module `PowerShellGet`. Pour plus d’informations sur l’installation des modules Windows PowerShell, consultez [Windows PowerShell](/powershell/scripting/developer/windows-powershell).

Les modules installés doivent se trouver dans un emplacement référencé par la variable d’environnement `PSModulePath` afin d’être importés automatiquement par le Worker hybride. Pour plus d’informations, consultez [Installer des modules dans PSModulePath](/powershell/scripting/developer/module/installing-a-powershell-module).

## <a name="remove-the-hybrid-runbook-worker"></a><a name="remove-windows-hybrid-runbook-worker"></a>Supprimer le Runbook Worker hybride

1. Dans le portail Azure, accédez à votre compte Automation.

2. Sous **Paramètres du compte**, sélectionnez **Clés**, puis notez les valeurs des champs **URL** et **Clé d’accès primaire**.

3. Ouvrez une session PowerShell en mode administrateur et exécutez la commande suivante avec votre URL et les valeurs de clé d'accès principal. Utilisez le paramètre `Verbose` pour afficher un journal détaillé du processus de suppression. Pour supprimer des ordinateurs obsolètes à partir de votre groupe Worker hybride, utilisez le paramètre facultatif `machineName`.

```powershell-interactive
Remove-HybridRunbookWorker -Url <URL> -Key <primaryAccessKey> -MachineName <computerName>
```

## <a name="remove-a-hybrid-worker-group"></a>Supprimer un groupe de Workers hybrides

Pour supprimer un groupe de Runbooks Workers hybrides, vous commencer par supprimer les Runbooks Workers hybrides de chaque ordinateur membre du groupe. Ensuite, suivez les étapes ci-dessous pour supprimer le groupe :

1. Dans le portail Azure, ouvrez le compte Automation.

2. Sélectionnez **Groupes Worker hybride** sous **Automatisation des processus**. Sélectionnez le groupe à supprimer. La page Propriétés du groupe s’affiche.

   ![Page Propriétés](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-group-properties.png)

3. Dans la page Propriétés du groupe sélectionné, cliquez sur **Supprimer**. Un message vous invite à confirmer cette action. Si vous êtes sûr de vouloir continuer, sélectionnez **Oui**.

   ![Message de confirmation](media/automation-hybrid-runbook-worker/automation-hybrid-runbook-worker-confirm-delete.png)

   Ce processus peut prendre plusieurs secondes. Vous pouvez suivre la progression sous **Notifications** dans le menu.

## <a name="next-steps"></a>Étapes suivantes

* Pour apprendre à configurer vos Runbooks afin d’automatiser les processus dans votre centre de données local ou un autre environnement cloud, consultez la rubrique [Exécuter des Runbooks sur un Runbook Worker hybride](automation-hrw-run-runbooks.md).

* Pour savoir comment résoudre les problèmes de vos Runbook Workers hybrides, consultez [Résoudre les problèmes liés à la fonctionnalité Runbook Worker hybride](troubleshoot/hybrid-runbook-worker.md#general).
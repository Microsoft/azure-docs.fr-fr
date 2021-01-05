---
title: Déployer un Runbook Worker hybride Linux dans Azure Automation
description: Cet article explique comment déployer un runbook Worker hybride Azure Automation qui vous permet d’exécuter des runbooks sur les machines Linux de votre centre de données local ou de votre environnement cloud.
services: automation
ms.subservice: process-automation
ms.date: 11/23/2020
ms.topic: conceptual
ms.openlocfilehash: 20683808c81b32560170b175edf1c37c332f47ad
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183615"
---
# <a name="deploy-a-linux-hybrid-runbook-worker"></a>Déployer un Runbook Worker hybride Linux

Vous pouvez utiliser la fonctionnalité Runbook Worker hybride utilisateur d’Azure Automation pour exécuter des runbooks directement sur la machine Azure ou non-Azure, y compris les serveurs inscrits auprès de [serveurs avec Azure Arc](../azure-arc/servers/overview.md). Sur l’ordinateur ou le serveur qui héberge le rôle, vous pouvez exécuter les runbooks directement avec les ressources disponibles dans l’environnement pour gérer ces ressources locales.

La fonctionnalité Runbook Worker hybride Linux exécute des Runbooks en tant qu’utilisateur spécial dont les privilèges peuvent être élevés pour exécuter des commandes nécessitant une élévation. Azure Automation stocke et gère les runbooks, puis les remet à une ou plusieurs machines désignées. Cet article explique comment installer le Runbook Worker hybride sur une machine Linux, comment supprimer le Worker et comment supprimer un groupe Runbook Worker hybride.

Une fois le Runbook Worker déployé, consultez [Exécuter des runbooks sur un Runbook Worker hybride](automation-hrw-run-runbooks.md) pour apprendre à configurer vos Runbooks en vue d’automatiser les processus de votre centre de données local ou autre environnement cloud.

## <a name="prerequisites"></a>Prérequis

Avant de commencer, vérifiez que vous disposez des éléments suivants.

### <a name="a-log-analytics-workspace"></a>Un espace de travail Log Analytics

Le rôle Runbook Worker hybride dépend d’un espace de travail Azure Monitor Log Analytics pour l’installation et la configuration du rôle. Vous pouvez le créer en utilisant [Azure Resource Manager](../azure-monitor/samples/resource-manager-workspace.md#create-a-log-analytics-workspace), [PowerShell](../azure-monitor/scripts/powershell-sample-create-workspace.md?toc=/powershell/module/toc.json) ou le [portail Azure](../azure-monitor/learn/quick-create-workspace.md).

Si vous n’avez pas d’espace de travail Azure Monitor Log Analytics, consultez le [guide de conception des journaux Azure Monitor](../azure-monitor/platform/design-logs-deployment.md) avant de créer l’espace de travail.

### <a name="log-analytics-agent"></a>Agent Log Analytics

Le rôle Runbook Worker hybride requiert l’[agent Log Analytics](../azure-monitor/platform/log-analytics-agent.md) pour le système d’exploitation Linux pris en charge. Pour les serveurs ou les ordinateurs hébergés en dehors d’Azure, vous pouvez installer l’agent Log Analytics à l’aide de [serveurs avec Azure Arc](../azure-arc/servers/overview.md).

>[!NOTE]
>Après l’installation de l’agent Log Analytics pour Linux, vous ne devez modifier ni les autorisations du dossier `sudoers.d` ni son appartenance. L’autorisation sudo est requise pour le compte **nxautomation**, qui est le contexte utilisateur sous lequel le Runbook Worker hybride s’exécute. Les autorisations ne doivent pas être supprimées. Limiter ce principe à certains dossiers ou à certaines commandes peut entraîner une modification critique.
>

### <a name="supported-linux-operating-systems"></a>Systèmes d’exploitation Linux pris en charge

La fonctionnalité Runbook Worker hybride prend en charge les distributions suivantes. Tous les systèmes d’exploitation sont censés être x64. L’architecture x86 n’est prise en charge pour aucun système d’exploitation.

* Amazon Linux 2012.09 à 2015.09
* CentOS Linux 5, 6 et 7
* Oracle Linux 5, 6 et 7
* Red Hat Enterprise Linux Server 5, 6 et 7
* Debian GNU/Linux 6, 7 et 8
* Ubuntu 12.04 LTS, 14.04 LTS, 16.04 LTS et 18.04 LTS
* SUSE Linux Enterprise Server 12

### <a name="minimum-requirements"></a>Configuration minimale requise

Voici la configuration minimale requise pour un système Linux et un Runbook Worker hybride utilisateur :

* 2 cœurs
* 4 Go de RAM
* Port 443 (sortie)

| **Package requis** | **Description** | **Version minimum**|
|--------------------- | --------------------- | -------------------|
|Glibc |Bibliothèque C de GNU| 2.5-12 |
|Openssl| Bibliothèques OpenSSL | 1.0 (TLS 1.1 et TLS 1.2 sont pris en charge)|
|Curl | Client web cURL | 7.15.5|
|Python-ctypes | Python 2.x est requis |
|PAM | Modules d’authentification enfichable|
| **Package facultatif** | **Description** | **Version minimum**|
| PowerShell Core | Pour exécuter des runbooks PowerShell, vous devez avoir installé PowerShell Core. Consultez la rubrique [Installation de PowerShell Core sous Linux](/powershell/scripting/install/installing-powershell-core-on-linux) pour connaître la procédure d’installation. | 6.0.0 |

### <a name="adding-a-machine-to-a-hybrid-runbook-worker-group"></a>Ajout d’un ordinateur à un groupe de Runbook Workers hybrides

Vous pouvez ajouter l’ordinateur Worker à un groupe de Runbook Workers hybrides dans l’un de vos comptes Automation. Pour les ordinateurs qui hébergent le Runbook Worker hybride système géré par Update Management, ils peuvent être ajoutés à un groupe de Runbook Workers hybrides. Toutefois, vous devez utiliser le même compte Automation pour Update Management et l’appartenance au groupe de Runbook Workers hybrides.

>[!NOTE]
>Azure Automation [Update Management](./update-management/overview.md) installe automatiquement le Runbook Worker hybride système sur un ordinateur Azure ou non Azure où la fonctionnalité Update Management est activée. Toutefois, ce Worker n’est inscrit auprès d’aucun des groupes de Runbook Workers hybrides figurant dans votre compte Automation. Pour exécuter vos runbooks sur ces ordinateurs, vous devez les ajouter à un groupe de Runbook Workers hybrides. Suivez l’étape 4 de la section [Installer un Runbook Worker hybride Linux](#install-a-linux-hybrid-runbook-worker) pour l’ajouter à un groupe.

## <a name="supported-linux-hardening"></a>Renforcement de la sécurité Linux pris en charge

Les éléments suivants ne sont pas encore pris en charge :

* CIS

## <a name="supported-runbook-types"></a>Types de Runbook pris en charge

Les runbooks Workers hybrides pour Linux prennent en charge un ensemble limité de types de runbooks dans Azure Automation. Ces types sont décrits dans le tableau suivant.

|Type de runbook | Prise en charge |
|-------------|-----------|
|Python 2 |Oui |
|PowerShell |Oui<sup>1</sup> |
|PowerShell Workflow |Non |
|Graphique |Non |
|Graphique workflow PowerShell |Non |

<sup>1</sup>Pour exécuter des runbooks PowerShell, vous devez avoir installé PowerShell Core sur la machine Linux. Consultez la rubrique [Installation de PowerShell Core sous Linux](/powershell/scripting/install/installing-powershell-core-on-linux) pour connaître la procédure d’installation.

### <a name="network-configuration"></a>Configuration réseau

Pour connaître la configuration réseau requise pour le Runbook Worker hybride, consultez [Configuration de votre réseau](automation-hybrid-runbook-worker.md#network-planning).

## <a name="install-a-linux-hybrid-runbook-worker"></a>Installer un Runbook Worker hybride Linux

Pour installer et configurer un runbook Worker hybride pour Linux, effectuez les étapes suivantes.

1. Activez la solution Azure Automation dans votre espace de travail Log Analytics en exécutant la commande suivante dans une invite de commandes PowerShell avec élévation de privilèges ou dans Cloud Shell, dans le [portail Azure](https://portal.azure.com) :

    ```powershell
    Set-AzOperationalInsightsIntelligencePack -ResourceGroupName <resourceGroupName> -WorkspaceName <workspaceName> -IntelligencePackName "AzureAutomation" -Enabled $true
    ```

2. Déployez l’agent Log Analytics sur la machine cible.

    * Pour les machines virtuelles Azure, installez l’agent Log Analytics pour Linux à l’aide de l’[extension de machine virtuelle pour Linux](../virtual-machines/extensions/oms-linux.md). L’extension installe l’agent Log Analytics sur les machines virtuelles Azure et inscrit les machines virtuelles dans un espace de travail Log Analytics existant. Vous pouvez utiliser un modèle Azure Resource Manager, l’interface de ligne de commande Azure ou Azure Policy pour attribuer la stratégie intégrée [Déployer l’agent Log Analytics pour les machines virtuelles *Linux* ou *Windows*](../governance/policy/samples/built-in-policies.md#monitoring). Une fois l’agent installé, la machine peut être ajoutée à un groupe de Runbook Workers hybrides dans votre compte Automation.

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

    > [!NOTE]
    > Pour gérer la configuration des machines qui prennent en charge le rôle Runbook Worker hybride avec Desired State Configuration (DSC), vous devez ajouter ces machines en tant que nœuds DSC.

    > [!NOTE]
    > Le [compte nxautomation](automation-runbook-execution.md#log-analytics-agent-for-linux) disposant des autorisations sudo correspondantes doit être présent lors de l’installation d’un Worker hybride Linux. Si vous essayez d’installer le Worker et que le compte n’est pas présent ou ne dispose pas des autorisations appropriées, l’installation échoue.

3. Vérifiez que l’agent rend compte à l’espace de travail.

    L’agent Log Analytics pour Linux connecte les machines à un espace de travail Azure Monitor Log Analytics. Lorsque vous installez l’agent sur votre machine et que vous le connectez à votre espace de travail, il télécharge automatiquement les composants nécessaires au runbook Worker hybride.

    Lorsque l’agent s’est correctement connecté à votre espace de travail Log Analytics, après quelques minutes, vous pouvez exécuter la requête suivante pour vérifier qu’il envoie les données de pulsation à l’espace de travail.

    ```kusto
    Heartbeat
    | where Category == "Direct Agent"
    | where TimeGenerated > ago(30m)
    ```

    Dans les résultats de la recherche, vous voyez en principe des enregistrements de pulsation pour la machine, ce qui indique que l’agent est connecté et qu’il rend compte au service. Par défaut, chaque agent transmet un enregistrement de pulsation à l’espace de travail qui lui est attribué.

4. Exécutez la commande suivante pour ajouter la machine à un groupe Runbook Worker hybride, en spécifiant les valeurs des paramètres `-w`, `-k`, `-g` et `-e`.

    Vous trouverez les informations requises pour les paramètres `-k` et `-e` dans la page **Clés** de votre compte Automation. Sélectionnez **Clés** sous la section **Paramètres du compte** sur le côté gauche de la page.

    ![Page Gérer les clés](media/automation-hybrid-runbook-worker/elements-panel-keys.png)

    * Pour le paramètre `-e`, copiez la valeur de l’**URL**.

    * Pour le paramètre `-k`, copiez la valeur de la clé d’accès principale (**PRIMARY ACCESS KEY**).

    * Pour le paramètre `-g`, spécifiez le nom du groupe Runbook Worker hybride auquel le nouveau runbook Worker hybride Linux doit être joint. Si ce groupe existe déjà dans le compte Automation, la machine active y est ajoutée. Si ce groupe n’existe pas, il est créé avec ce nom.

    * Pour le paramètre `-w`, spécifiez votre ID d’espace de travail Log Analytics.

   ```bash
   sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/onboarding.py --register -w <logAnalyticsworkspaceId> -k <automationSharedKey> -g <hybridGroupName> -e <automationEndpoint>
   ```

5. Vérifiez le déploiement une fois le script terminé. Dans la page **Groupes de Runbook Workers hybrides** de votre compte Automation, sous l’onglet **Groupe de Runbook Workers hybrides utilisateur**, le groupe nouveau ou existant et le nombre de membres sont affichés. S’il s’agit d’un groupe existant, le nombre de membres est incrémenté. Vous pouvez sélectionner le groupe dans la liste de cette page en choisissant **Workers hybrides** dans le menu de gauche. La page **Workers hybrides** indique chaque membre du groupe listé.

    > [!NOTE]
    > Si vous utilisez l'extension de machine virtuelle Log Analytics pour Linux sur une machine virtuelle Azure, nous vous recommandons de définir `autoUpgradeMinorVersion` sur la valeur `false`, car la mise à niveau automatique des versions peut provoquer des problèmes au niveau du runbook Worker hybride. Pour savoir comment mettre à niveau l'extension manuellement, consultez [Déploiement Azure CLI](../virtual-machines/extensions/oms-linux.md#azure-cli-deployment).

## <a name="turn-off-signature-validation"></a>Désactiver la validation de signature

Par défaut, les Runbooks Workers hybrides Linux nécessitent la validation de signature. Si vous exécutez un runbook non signé sur un Worker, l'erreur `Signature validation failed` s'affiche. Pour désactiver la validation de la signature, exécutez la commande suivante. Remplacez le deuxième paramètre par votre ID d’espace de travail Log Analytics.

 ```bash
 sudo python /opt/microsoft/omsconfig/modules/nxOMSAutomationWorker/DSCResources/MSFT_nxOMSAutomationWorkerResource/automationworker/scripts/require_runbook_signature.py --false <logAnalyticsworkspaceId>
 ```

## <a name="remove-the-hybrid-runbook-worker"></a><a name="remove-linux-hybrid-runbook-worker"></a>Supprimer le Runbook Worker hybride

Vous pouvez utiliser la commande `ls /var/opt/microsoft/omsagent` sur le Runbook Worker hybride pour obtenir l’ID d’espace de travail. Un dossier portant l'ID de l'espace de travail est créé.

```bash
sudo python onboarding.py --deregister --endpoint="<URL>" --key="<PrimaryAccessKey>" --groupname="Example" --workspaceid="<workspaceId>"
```

> [!NOTE]
> Ce script ne supprime pas l’agent Log Analytics pour Linux de la machine. Il supprime uniquement les fonctionnalités et la configuration du rôle Runbook Worker hybride.

## <a name="remove-a-hybrid-worker-group"></a>Supprimer un groupe de Workers hybrides

Pour supprimer un groupe Runbook Worker hybride sur des machines Linux, suivez les mêmes étapes que pour un groupe de Workers hybrides Windows. Consultez [Supprimer un groupe de Workers hybrides](automation-windows-hrw-install.md#remove-a-hybrid-worker-group).

## <a name="next-steps"></a>Étapes suivantes

* Pour apprendre à configurer vos Runbooks afin d’automatiser les processus dans votre centre de données local ou un autre environnement cloud, consultez la rubrique [Exécuter des Runbooks sur un Runbook Worker hybride](automation-hrw-run-runbooks.md).

* Pour savoir comment résoudre les problèmes de vos Runbook Workers hybrides, consultez [Résoudre les problèmes liés à la fonctionnalité Runbook Worker hybride - Linux](troubleshoot/hybrid-runbook-worker.md#linux).
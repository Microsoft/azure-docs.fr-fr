---
title: Résolution des problèmes de l’agent de mise à jour Windows dans Azure Automation
description: Cet article explique comment détecter et résoudre les problèmes liés à l’agent Windows Update dans Update Management.
services: automation
author: mgoedtel
ms.author: magoedte
ms.date: 01/16/2020
ms.topic: conceptual
ms.service: automation
ms.subservice: update-management
manager: carmonm
ms.openlocfilehash: 92020313fccf1b8be0add58a7bafab62b5daa4d5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86187130"
---
# <a name="troubleshoot-windows-update-agent-issues"></a>Résoudre les problèmes de l’agent de mise à jour Windows

De nombreuses raisons peuvent expliquer pourquoi votre machine n’apparaît pas comme étant prête (saine) pendant le déploiement d’Update Management. Vous pouvez vérifier l’intégrité d’un agent Runbook Worker hybride Windows pour déterminer le problème sous-jacent. Voici les trois états de préparation possibles d’une machine :

* Prêt : le runbook Worker hybride est déployé et a été affiché pour la dernière fois il y a moins d’une heure.
* Déconnecté : le runbook Worker hybride est déployé et a été affiché pour la dernière fois il y a plus d’une heure.
* Non configuré : le Runbook Worker hybride est introuvable, ou son déploiement n’est pas terminé.

> [!NOTE]
> L’état actuel d’une machine peut s’afficher avec un léger retard sur le Portail Azure.

Cet article explique comment exécuter l’utilitaire de résolution des problèmes pour les machines Azure à partir du portail Azure, et comment exécuter les machines non Azure dans un [scénario hors connexion](#troubleshoot-offline). 

> [!NOTE]
> Le script de l’utilitaire de résolution des problèmes comprend désormais des vérifications pour Windows Server Update Services (WSUS), ainsi que pour les clés de téléchargement automatique et d’installation. 

## <a name="start-the-troubleshooter"></a>Démarrer l’utilitaire de résolution des problèmes

Pour les machines Azure, vous pouvez ouvrir la page « Résoudre les problèmes de l’agent de mise à jour » en sélectionnant le lien **Résolution des problèmes** sous la colonne **Préparation de l’agent de mise à jour** dans le portail. Pour les machines non-Azure, le lien vous dirige vers cet article. Consultez [Résolution des problèmes hors connexion](#troubleshoot-offline) pour résoudre les problèmes d’une machine non Azure.

![Capture d’écran de la liste des machines virtuelles Update Management](../media/update-agent-issues/vm-list.png)

> [!NOTE]
> Pour vérifier l’intégrité du Runbook Worker hybride, la machine virtuelle doit être en cours d’exécution. Si la machine virtuelle n’est pas en cours d’exécution, un bouton **Démarrer la machine virtuelle** s’affiche.

Dans la page Résoudre les problèmes de l’agent de mise à jour, sélectionnez **Exécuter les vérifications** pour démarrer l’utilitaire de résolution des problèmes. L’utilitaire de résolution des problèmes utilise [Run Command](../../virtual-machines/windows/run-command.md) pour exécuter un script sur la machine et vérifier les dépendances. Quand l’utilitaire de résolution des problèmes a terminé, il retourne le résultat des vérifications.

![Capture d’écran de la page Résoudre les problèmes de l’agent de mise à jour](../media/update-agent-issues/troubleshoot-page.png)

Les résultats sont affichés dans la page quand ils sont prêts. Les sections des vérifications affichent ce qui est inclus dans chaque vérification.

![Capture d’écran des vérifications sous Résoudre les problèmes de l’agent de mise à jour](../media/update-agent-issues/update-agent-checks.png)

## <a name="prerequisite-checks"></a>Vérifications de la configuration requise

### <a name="operating-system"></a>Système d’exploitation

La vérification du système d’exploitation permet de vérifier si le Runbook Worker hybride exécute l’un des systèmes d’exploitation indiqués dans le tableau suivant.

|Système d’exploitation  |Notes  |
|---------|---------|
|Windows Server 2012 et ultérieur |.NET Framework 4.6 ou version ultérieure est requis. ([Télécharger .NET Framework](/dotnet/framework/install/guide-for-developers))<br/> Windows PowerShell 5.1 est requis.  ([Télécharger Windows Management Framework 5.1](https://www.microsoft.com/download/details.aspx?id=54616))        |

### <a name="net-462"></a>.NET 4.6.2

La fonctionnalité de vérification de .NET Framework vérifie que [.NET Framework 4.6.2](https://www.microsoft.com/en-us/download/details.aspx?id=53345) ou une version ultérieure est installé sur le système.

### <a name="wmf-51"></a>WMF 5.1

La fonctionnalité de vérification de WMF vérifie que le système dispose de la version nécessaire de Windows Management Framework (WMF), c’est-à-dire [Windows Management Framework 5.1](https://www.microsoft.com/download/details.aspx?id=54616).

### <a name="tls-12"></a>TLS 1.2

Cette vérification détermine si vous utilisez TLS 1.2 pour chiffrer vos communications. TLS 1.0 n’est plus pris en charge par la plateforme. Utilisez TLS 1.2 pour communiquer avec Update Management.

## <a name="connectivity-checks"></a>Vérifications de la connectivité

### <a name="registration-endpoint"></a>Point de terminaison d’inscription

Cette vérification détermine si l’agent peut correctement communiquer avec le service d’agent.

Les configurations du pare-feu et du proxy doivent permettre à l’agent Runbook Worker hybride de communiquer avec le point de terminaison d’inscription. Pour obtenir la liste des adresses et des ports à ouvrir, consultez [Planification réseau](../automation-hybrid-runbook-worker.md#network-planning).

### <a name="operations-endpoint"></a>Point de terminaison des opérations

Cette vérification détermine si l’agent peut correctement communiquer avec Job Runtime Data Service.

Les configurations du pare-feu et du proxy doivent permettre à l’agent Runbook Worker hybride de communiquer avec Job Runtime Data Service. Pour obtenir la liste des adresses et des ports à ouvrir, consultez [Planification réseau](../automation-hybrid-runbook-worker.md#network-planning).

## <a name="vm-service-health-checks"></a>Vérification de l’intégrité du service de machine virtuelle

### <a name="monitoring-agent-service-status"></a>État du service de Monitoring Agent

Cette vérification détermine si l’agent Log Analytics pour Windows (`healthservice`) est en cours d’exécution sur l’ordinateur. Pour en savoir plus sur la résolution des problèmes de ce service, consultez [l’agent Log Analytics pour Windows ne s’exécute pas](hybrid-runbook-worker.md#mma-not-running).

Si vous souhaitez réinstaller l’agent Log Analytics pour Windows, consultez [Installer l’agent Log Analytics pour Windows](../../azure-monitor/learn/quick-collect-windows-computer.md#install-the-agent-for-windows).

### <a name="monitoring-agent-service-events"></a>Événements de service de Monitoring Agent

Cette vérification détermine si des événements 4502 apparaissent dans le journal Azure Operations Manager de la machine au cours des dernières 24 heures.

Pour plus d’informations sur cet événement, consultez l’[événement 4502 dans le journal Operations Manager](hybrid-runbook-worker.md#event-4502).

## <a name="access-permissions-checks"></a>Vérification des autorisations d’accès

> [!NOTE]
> L’utilitaire de résolution des problèmes ne route pas le trafic via un serveur proxy s’il y en a un de configuré.

### <a name="crypto-folder-access"></a>Accès au dossier Crypto

La vérification d’accès au dossier Crypto détermine si le compte système Local a accès à C:\ProgramData\Microsoft\Crypto\RSA.

## <a name="troubleshoot-offline"></a><a name="troubleshoot-offline"></a>Résolution des problèmes hors connexion

Vous pouvez utiliser l’utilitaire de résolution des problèmes sur un Runbook Worker hybride hors connexion en exécutant le script localement. Obtenez le script suivant à partir de PowerShell Gallery : [Troubleshoot-WindowsUpdateAgentRegistration](https://www.powershellgallery.com/packages/Troubleshoot-WindowsUpdateAgentRegistration). Pour exécuter le script, vous devez avoir installé WMF 4.0 ou ultérieur. Pour télécharger la dernière version de PowerShell, consultez [Installer plusieurs versions de PowerShell](/powershell/scripting/install/installing-powershell).

La sortie de ce script ressemble à l’exemple suivant :

```output
RuleId                      : OperatingSystemCheck
RuleGroupId                 : prerequisites
RuleName                    : Operating System
RuleGroupName               : Prerequisite Checks
RuleDescription             : The Windows Operating system must be version 6.2.9200 (Windows Server 2012) or higher
CheckResult                 : Passed
CheckResultMessage          : Operating System version is supported
CheckResultMessageId        : OperatingSystemCheck.Passed
CheckResultMessageArguments : {}

RuleId                      : DotNetFrameworkInstalledCheck
RuleGroupId                 : prerequisites
RuleName                    : .NET Framework 4.5+
RuleGroupName               : Prerequisite Checks
RuleDescription             : .NET Framework version 4.5 or higher is required
CheckResult                 : Passed
CheckResultMessage          : .NET Framework version 4.5+ is found
CheckResultMessageId        : DotNetFrameworkInstalledCheck.Passed
CheckResultMessageArguments : {}

RuleId                      : WindowsManagementFrameworkInstalledCheck
RuleGroupId                 : prerequisites
RuleName                    : WMF 5.1
RuleGroupName               : Prerequisite Checks
RuleDescription             : Windows Management Framework version 4.0 or higher is required (version 5.1 or higher is preferable)
CheckResult                 : Passed
CheckResultMessage          : Detected Windows Management Framework version: 5.1.17763.1
CheckResultMessageId        : WindowsManagementFrameworkInstalledCheck.Passed
CheckResultMessageArguments : {5.1.17763.1}

RuleId                      : AutomationAgentServiceConnectivityCheck1
RuleGroupId                 : connectivity
RuleName                    : Registration endpoint
RuleGroupName               : connectivity
RuleDescription             :
CheckResult                 : Failed
CheckResultMessage          : Unable to find Workspace registration information in registry
CheckResultMessageId        : AutomationAgentServiceConnectivityCheck1.Failed.NoRegistrationFound
CheckResultMessageArguments : {}

RuleId                      : AutomationJobRuntimeDataServiceConnectivityCheck
RuleGroupId                 : connectivity
RuleName                    : Operations endpoint
RuleGroupName               : connectivity
RuleDescription             : Proxy and firewall configuration must allow Automation Hybrid Worker agent to communicate with eus2-jobruntimedata-prod-su1.azure-automation.net
CheckResult                 : Passed
CheckResultMessage          : TCP Test for eus2-jobruntimedata-prod-su1.azure-automation.net (port 443) succeeded
CheckResultMessageId        : AutomationJobRuntimeDataServiceConnectivityCheck.Passed
CheckResultMessageArguments : {eus2-jobruntimedata-prod-su1.azure-automation.net}

RuleId                      : MonitoringAgentServiceRunningCheck
RuleGroupId                 : servicehealth
RuleName                    : Monitoring Agent service status
RuleGroupName               : VM Service Health Checks
RuleDescription             : HealthService must be running on the machine
CheckResult                 : Failed
CheckResultMessage          : Log Analytics for Windows service (HealthService) is not running
CheckResultMessageId        : MonitoringAgentServiceRunningCheck.Failed
CheckResultMessageArguments : {Log Analytics agent for Windows, HealthService}

RuleId                      : MonitoringAgentServiceEventsCheck
RuleGroupId                 : servicehealth
RuleName                    : Monitoring Agent service events
RuleGroupName               : VM Service Health Checks
RuleDescription             : Event Log must not have event 4502 logged in the past 24 hours
CheckResult                 : Failed
CheckResultMessage          : Log Analytics agent for Windows service Event Log (Operations Manager) does not exist on the machine
CheckResultMessageId        : MonitoringAgentServiceEventsCheck.Failed.NoLog
CheckResultMessageArguments : {Log Analytics agent for Windows, Operations Manager, 4502}

RuleId                      : CryptoRsaMachineKeysFolderAccessCheck
RuleGroupId                 : permissions
RuleName                    : Crypto RSA MachineKeys Folder Access
RuleGroupName               : Access Permission Checks
RuleDescription             : SYSTEM account must have WRITE and MODIFY access to 'C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys'
CheckResult                 : Passed
CheckResultMessage          : Have permissions to access C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys
CheckResultMessageId        : CryptoRsaMachineKeysFolderAccessCheck.Passed
CheckResultMessageArguments : {C:\\ProgramData\\Microsoft\\Crypto\\RSA\\MachineKeys}

RuleId                      : TlsVersionCheck
RuleGroupId                 : prerequisites
RuleName                    : TLS 1.2
RuleGroupName               : Prerequisite Checks
RuleDescription             : Client and Server connections must support TLS 1.2
CheckResult                 : Passed
CheckResultMessage          : TLS 1.2 is enabled by default on the Operating System.
CheckResultMessageId        : TlsVersionCheck.Passed.EnabledByDefault
CheckResultMessageArguments : {}
```

## <a name="next-steps"></a>Étapes suivantes

[Résoudre les problèmes liés au Runbook Worker hybride](hybrid-runbook-worker.md).

---
title: Activer Azure Monitor pour un environnement hybride
description: Cet article décrit comment activer des Azure Monitor pour machines virtuelles pour un environnement de cloud hybride contenant une ou plusieurs machines virtuelles.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: d994df4d56f4958784256ff9cd92ce1e6f3b3e50
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88642161"
---
# <a name="enable-azure-monitor-for-vms-for-a-hybrid-virtual-machine"></a>Activer Azure Monitor pour machines virtuelles pour une machine virtuelle hybride
Cet article explique comment activer Azure Monitor pour machines virtuelles pour une machine virtuelle en dehors d’Azure, y compris des environnements locaux et autres clouds.

> [!IMPORTANT]
> La méthode recommandée pour activer des machines virtuelles hybrides consiste tout d’abord à activer [Azure Arc pour serveurs](../../azure-arc/servers/overview.md) afin que les machines virtuelles puissent être activées pour Azure Monitor pour machines virtuelles à l’aide de processus similaires aux machines virtuelles Azure. Cet article explique comment intégrer des machines virtuelles hybrides si vous choisissez de ne pas utiliser Azure Arc.

## <a name="prerequisites"></a>Prérequis

- [Créer et configurer un espace de travail Log Analytics](vminsights-configure-workspace.md)
- Pour vous assurer que le système d’exploitation de la machine virtuelle ou du groupe de machines virtuelles identiques que vous activez est pris en charge, consulter [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems). 


## <a name="overview"></a>Vue d’ensemble
Les machines virtuelles en dehors d’Azure nécessitent les mêmes agent Log Analytics et Dependency Agent que ceux utilisés pour les machines virtuelles Azure. Étant donné que vous ne pouvez pas utiliser d’extensions de machine virtuelle pour installer les agents, vous devez les installer manuellement dans le système d’exploitation invité ou les installer par le biais d’une autre méthode. 

Pour plus d’informations sur le déploiement de l’agent Log Analytics, consultez [Connecter des ordinateurs Windows à Azure Monitor](../platform/agent-windows.md) ou [Connecter des ordinateurs Linux à Azure Monitor](../platform/agent-linux.md). Des détails pour l’agent Dependency Agent sont fournis dans cet article. 

## <a name="firewall-requirements"></a>Configuration requise du pare-feu
La configuration requise du pare-feu pour l’agent Log Analytics est fournie dans [Présentation de l’agent Log Analytics](../platform/log-analytics-agent.md#network-requirements). L’agent Map Dependency Agent Azure Monitor pour machines virtuelles ne transmet pas de données par lui-même et ne requiert pas de modifications des pare-feu ni des ports. Log Analytics Agent transmet toujours les données Map au service Azure Monitor, directement ou par le biais de la [passerelle Operations Management Suite](../../azure-monitor/platform/gateway.md), si vos stratégies de sécurité n’autorisent pas les ordinateurs du réseau à se connecter à Internet.


## <a name="dependency-agent"></a>Agent de dépendances

>[!NOTE]
>Les informations suivantes décrites dans cette section s’appliquent également à la solution [Service Map](service-map.md).  

Vous pouvez télécharger le Dependency Agent à partir des emplacements suivants :

| Fichier | Système d''exploitation | Version | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.10.5.10940 | C27A56D0BE9CF162DF73292DFBB2083F5FF749F2B80FCAD2545BC8B14B64A8D7  |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.10.5.10940 | 71B4E1DA5116E61E03317C49C6702B5069F01A0C9A7CB860F6ACFAF5C198740E |


## <a name="install-the-dependency-agent-on-windows"></a>Installer l’agent de dépendances sous Windows

Vous pouvez installer Dependency Agent manuellement sur les ordinateurs Windows en exécutant `InstallDependencyAgent-Windows.exe`. Si vous exécutez ce fichier exécutable sans aucune option, il démarre un assistant de configuration que vous pouvez suivre pour une installation interactive de l’agent. Pour installer ou désinstaller l’agent, vous devez disposer de privilèges *Administrateur* sur le système d’exploitation invité.

Le tableau suivant répertorie les paramètres pris en charge par le programme de configuration pour l’agent à partir de la ligne de commande.

| Paramètre | Description |
|:--|:--|
| /? | Retourne la liste des options de ligne de commande. |
| /S | Effectue une installation silencieuse sans interaction utilisateur. |

Par exemple, pour exécuter le programme d’installation avec le paramètre `/?`, saisissez **InstallDependencyAgent-Windows.exe /?** .

Par défaut, les fichiers Dependency Agent pour Windows sont installés dans le répertoire *C:\Program Files\Microsoft Dependency Agent*. Si le démarrage de l’agent de dépendances échoue une fois l’installation terminée, recherchez des informations détaillées sur l’erreur dans les journaux d’activité. Le répertoire des journaux d’activité est *%Programfiles%\Microsoft Dependency Agent\logs*.

### <a name="powershell-script"></a>Script PowerShell
Pour télécharger et installer l’agent, utilisez l’exemple de script PowerShell suivant :

```powershell
Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S
```


## <a name="install-the-dependency-agent-on-linux"></a>Installer l’agent de dépendances sous Linux

Dependency Agent est installé sur les ordinateurs Linux avec *InstallDependencyAgent-Linux64.bin*, script shell comportant un fichier binaire à extraction automatique. Vous pouvez exécuter le fichier à l’aide de `sh` ou ajouter des autorisations d’exécution au fichier lui-même.

>[!NOTE]
> L’accès racine est requis pour installer ou configurer l’agent.
>

| Paramètre | Description |
|:--|:--|
| -help | Récupérez la liste des options de ligne de commande. |
| -S | Effectuez une installation silencieuse sans invite utilisateur. |
| --check | Vérifiez les autorisations et le système d’exploitation, sans installer l’agent. |

Par exemple, pour exécuter le programme d’installation avec le paramètre `-help`, saisissez **InstallDependencyAgent-Linux64.bin -help**. Installez Dependency Agent pour Linux en tant que racine en exécutant la commande `sh InstallDependencyAgent-Linux64.bin`.

Si le démarrage de l’agent de dépendances échoue, recherchez des informations détaillées sur l’erreur dans les journaux d’activité. Sur les agents Linux, le répertoire des journaux est */var/opt/microsoft/dependency-agent/log*.

Les fichiers de l’agent de dépendances sont placés dans les répertoires suivants :

| Fichiers | Emplacement |
|:--|:--|
| Fichiers principaux | /opt/microsoft/dependency-agent |
| Fichiers journaux | /var/opt/microsoft/dependency-agent/log |
| Fichiers de configuration | /etc/opt/microsoft/dependency-agent/config |
| Fichiers exécutables de service | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Fichiers de stockage binaires | /var/opt/microsoft/dependency-agent/storage |

### <a name="shell-script"></a>Script shell 
Utilisez l’exemple de script d’interpréteur de commandes suivant pour télécharger et installer l’agent :

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
sudo sh InstallDependencyAgent-Linux64.bin -s
```

## <a name="desired-state-configuration"></a>DSC (Desired State Configuration)

Pour déployer l’agent de dépendances avec la configuration de l'état souhaité, vous pouvez utiliser le module xPSDesiredStateConfiguration avec l’exemple de code suivant :

```powershell
configuration VMInsights {

    Import-DscResource -ModuleName xPSDesiredStateConfiguration

    $DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"

    Node localhost
    {
        # Download and install the Dependency agent
        xRemoteFile DAPackage
        {
            Uri = "https://aka.ms/dependencyagentwindows"
            DestinationPath = $DAPackageLocalPath
        }

        xPackage DA
        {
            Ensure="Present"
            Name = "Dependency Agent"
            Path = $DAPackageLocalPath
            Arguments = '/S'
            ProductId = ""
            InstalledCheckRegKey = "HKEY_LOCAL_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"
            InstalledCheckRegValueName = "DisplayName"
            InstalledCheckRegValueData = "Dependency Agent"
            DependsOn = "[xRemoteFile]DAPackage"
        }
    }
}
```



## <a name="troubleshooting"></a>Dépannage

### <a name="vm-doesnt-appear-on-the-map"></a>La machine virtuelle n’apparaît pas sur la carte

Si votre installation de l’agent de dépendances a réussi, mais que vous ne voyez pas votre ordinateur sur la carte, diagnostiquez le problème en procédant comme suit.

1. L’agent de dépendances est-il correctement installé ? Vous pouvez vous en assurer en vérifiant si le service est installé et en cours d’exécution.

    **Windows** : recherchez le service nommé « Microsoft Dependency Agent ».

    **Linux** : recherchez « microsoft-dependency-agent » dans les processus en cours d’exécution.

2. Utilisez-vous le [niveau tarifaire Gratuit de Log Analytics](./solutions.md) ? Le plan Gratuit autorise jusqu’à 5 ordinateurs uniques. Les ordinateurs suivants ne s’affichent pas sur la carte, même si les cinq précédents n’envoient plus de données.

3. L’ordinateur envoie-t-il des données de journal et de performances aux journaux Azure Monitor ? Exécutez la requête suivante sur votre ordinateur :

    ```Kusto
    Usage | where Computer == "computer-name" | summarize sum(Quantity), any(QuantityUnit) by DataType
    ```

    Retourne-t-elle un ou plusieurs résultats ? Les données sont-elles récentes ? Dans ce cas, votre agent Log Analytics fonctionne correctement et communique avec le service. Si ce n’est pas le cas, vérifiez l’agent sur votre serveur : [Résolution des problèmes de l’agent Log Analytics pour Windows](../platform/agent-windows-troubleshoot.md) ou [Résolution des problèmes de l’agent Log Analytics pour Linux](../platform/agent-linux-troubleshoot.md).

#### <a name="computer-appears-on-the-map-but-has-no-processes"></a>L’ordinateur s’affiche sur la carte, mais n’a aucun processus

Si vous voyez votre serveur sur la carte, mais qu’il ne comporte aucune donnée de processus ou de connexion, cela indique que l’agent de dépendances est installé et en cours d’exécution, mais que le pilote en mode noyau ne s’est pas chargé.

Vérifiez le fichier C:\Program Files\Microsoft Dependency Agent\logs\wrapper.log (Windows) ou le fichier /var/opt/microsoft/dependency-agent/log/service.log (Linux). Les dernières lignes du fichier doivent indiquer la raison pour laquelle le noyau ne s’est pas chargé. Par exemple, le noyau n’est peut-être pas pris en charge sous Linux si vous l’avez mis à jour.


## <a name="next-steps"></a>Étapes suivantes

Une fois la supervision activée pour vos machines virtuelles, ces informations peuvent être analysées par Azure Monitor pour machines virtuelles.

- Pour afficher les dépendances des applications détectées, consultez [Utilisation de la fonctionnalité Map d’Azure Monitor pour machines virtuelles dans le but de comprendre les composants d’application](vminsights-maps.md).

- Pour identifier les goulots d’étranglement et l’utilisation globale avec les performances de votre machine virtuelle, consultez [Consulter les performances des machines virtuelles Azure](vminsights-performance.md).
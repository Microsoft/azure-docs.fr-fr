---
title: Solution Wire Data dans Azure Monitor | Microsoft Docs
description: Les données de communication sont des données de performances et réseau consolidées provenant des ordinateurs avec des agents Log Analytics. Les données réseau sont associées à vos données de journaux pour vous aider à mettre les données en corrélation.
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/29/2020
ms.openlocfilehash: 06698ad3ab2ceb76278e23bc1ac0002b9c2284f9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91445777"
---
# <a name="wire-data-20-preview-solution-in-azure-monitor"></a>Solution Wire Data 2.0 (préversion) dans Azure Monitor

![Symbole de Wire Data](media/wire-data/wire-data2-symbol.png)

Les données de communication sont des données de performances et réseau consolidées, collectées à partir d’ordinateurs connectés à Windows et à Linux avec l’agent Log Analytics, notamment les ordinateurs analysés par Operations Manager dans votre environnement. Les données réseau sont associées aux autres données de journaux pour faciliter la mise en corrélation des données.

Outre l’agent Log Analytics, la solution Wire Data utilise des agents Microsoft Dependency Agent installés sur les ordinateurs de votre infrastructure informatique. Les agents de dépendances surveillent les données réseau envoyées par et à ces ordinateurs pour les niveaux de réseau 2 et 3 dans le [modèle OSI](https://en.wikipedia.org/wiki/OSI_model), y compris les différents protocoles et ports utilisés. Les données sont ensuite envoyées à Azure Monitor à l’aide d’agents.  

>[!NOTE]
>La solution Wire Data a été remplacée par la [solution Service Map](service-map.md).  Toutes deux utilisent l’agent Log Analytics et l’agent de dépendances pour collecter les données de connexion réseau dans Azure Monitor. 
> 
>Les clients existants qui utilisent la solution Wire Data peuvent continuer à l’utiliser. Nous publierons des instructions concernant la chronologie de la migration vers Service Map.
>
>Les nouveaux clients doivent installer la [solution Service Map](service-map.md) ou [Azure Monitor pour machines virtuelles](vminsights-overview.md).  Le jeu de données Service Map est comparable à Wire Data.  Azure Monitor pour machines virtuelles comprend le jeu de données Service Map avec des fonctionnalités et des données de performances supplémentaires pour l’analyse. 


Par défaut, Azure Monitor journalise les données pour l’UC, la mémoire, le disque et les données de performances du réseau à partir de compteurs intégrés à Windows et à Linux, ainsi que d’autres compteurs de performances que vous pouvez spécifier. La collecte des données réseau et autres est effectuée en temps réel pour chaque agent, notamment les sous-réseaux et les protocoles de niveau application utilisés par l’ordinateur.  Wire Data examine les données réseau au niveau de l’application, pas au niveau de la couche de transport TCP.  La solution n’examine pas les données ACK et SYN individuelles.  Une fois la connexion effectuée, elle est considérée comme une connexion active et marquée en tant que Connecté. Cette connexion reste active tant que les deux côtés acceptent que le socket est ouvert et que des données peuvent être transférées dans les deux sens.  Si l’un des deux côtés ferme la connexion, celle-ci est marquée en tant que Déconnecté.  Par conséquent, elle ne compte que la bande passante des paquets terminés avec succès et ne signale pas les renvois ou les paquets ayant échoué.

Si vous avez déjà utilisé [sFlow](http://www.sflow.org/) ou d’autres logiciels respectant le [protocole NetFlow de Cisco](https://www.cisco.com/c/en/us/products/collateral/ios-nx-os-software/ios-netflow/prod_white_paper0900aecd80406232.html), les statistiques et les données fournies par les données de communication vous seront familières.

Voici quelques types de requêtes de recherche de journal intégrés :

- Agents qui fournissent des données de communication
- Adresse IP des agents fournissant des données de communication
- Communications sortantes par adresse IP
- Nombre d’octets envoyés par les protocoles d’application
- Nombre d’octets envoyés par un service d’application
- Octets reçus par différents protocoles
- Nombre total d’octets envoyés et reçus par version d’adresse IP
- Latence moyenne pour les connexions qui ont été mesurées de manière fiable
- Processus informatiques ayant initié ou reçu du trafic réseau
- Quantité de trafic réseau pour un processus

Quand vous recherchez à l’aide de données de communication, vous pouvez filtrer et regrouper les données pour afficher des informations sur les principaux agents et protocoles. Vous pouvez aussi savoir quand certains ordinateurs (adresses IP/MAC) ont communiqué les uns avec les autres et connaître la durée de ces communications et la quantité de données envoyées. En fait, vous affichez des métadonnées sur le trafic réseau, qui fonctionne par recherches.

Toutefois, comme il s’agit de métadonnées, elles ne sont pas nécessairement utiles pour résoudre des problèmes complexes. Les données de communication d’Azure Monitor ne capturent pas toutes les données réseau.  Elles ne sont pas censées être utilisées pour résoudre des problèmes complexes au niveau du paquet. L’avantage de l’utilisation de l’agent (par rapport à d’autres méthodes de collecte) est que vous n’avez pas à installer d’appareils, à reconfigurer vos commutateurs réseau ni à effectuer de configurations compliquées. Les données de communication sont simplement basées sur un agent, qu’il vous suffit d’installer sur un ordinateur pour qu’il surveille son propre trafic réseau. Le recours à un agent est aussi utile quand vous souhaitez surveiller les charges de travail en cours d’exécution dans des fournisseurs de cloud, des fournisseurs de services d’hébergement ou Microsoft Azure, dans les cas où l’utilisateur n’est pas propriétaire de la couche d’infrastructure.

## <a name="connected-sources"></a>Sources connectées

Wire Data obtient ses données auprès de l’agent de dépendances Microsoft. Celui-ci dépend de l’agent Log Analytics pour ses connexions à Azure Monitor. Cela signifie que l’agent Log Analytics doit être installé et configuré avec l’agent de dépendances sur le serveur. Le tableau suivant décrit les sources connectées prises en charge par la solution Wire Data.

| **Source connectée** | **Pris en charge** | **Description** |
| --- | --- | --- |
| Agents Windows | Oui | Wire Data analyse et collecte des données provenant d’ordinateurs agents Windows. <br><br> Outre [l’agent Log Analytics pour Windows](../platform/agent-windows.md), les agents Windows nécessitent l’agent de dépendances Microsoft. Pour obtenir la liste complète des versions des systèmes d’exploitation, consultez la page [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems). |
| Agents Linux | Oui | Wire Data analyse et collecte des données provenant d’ordinateurs agents Linux.<br><br> Outre [l’agent Log Analytics pour Linux](../learn/quick-collect-linux-computer.md), les agents Linux nécessitent l’agent de dépendances Microsoft. Pour obtenir la liste complète des versions des systèmes d’exploitation, consultez la page [Systèmes d’exploitation pris en charge](vminsights-enable-overview.md#supported-operating-systems). |
| Groupe d’administration Microsoft System Center Operations Manager | Oui | Wire Data analyse et collecte des données provenant des agents Windows et Linux dans un [groupe d’administration System Center Operations Manager](../platform/om-agents.md) connecté. <br><br> Une connexion directe entre l’ordinateur agent System Center Operations Manager et Azure Monitor est requise. |
| Compte Azure Storage | Non | Wire Data collecte des données provenant des ordinateurs agents. Aucune donnée n’est donc recueillie à partir du Stockage Azure. |

Sous Windows, Microsoft Monitoring Agent (MMA) est utilisé à la fois par System Center Operations Manager et par Azure Monitor pour collecter et envoyer des données. En fonction du contexte, l’agent est nommé agent System Center Operations Manager, agent Log Analytics, MMA ou agent direct. System Center Operations Manager et Azure Monitor fournissent des versions légèrement différentes de MMA. Les deux versions peuvent envoyer leurs rapports à System Center Operations Manager, à Azure Monitor ou aux deux.

Sous Linux, l’agent Log Analytics pour Linux collecte et envoie les données à Azure Monitor. Vous pouvez utiliser Wire Data sur des serveurs équipés d’agents directement connectés à Azure Monitor ou sur des serveurs se connectant à Azure Monitor au moyen de groupes d’administration System Center Operations Manager.

L’agent de dépendances ne transmet pas de données par lui-même et ne demande pas de modifications des pare-feu ni des ports. Les données de Wire Data sont toujours transmises par l’agent Log Analytics à Azure Monitor, directement ou par le biais de la passerelle Log Analytics.

![diagramme de l’agent](./media/wire-data/agents.png)

Si vous êtes un utilisateur de System Center Operations Manager avec un groupe d’administration connecté à Azure Monitor :

- Aucune configuration supplémentaire n’est requise si vos agents System Center Operations Manager peuvent accéder à Internet pour se connecter à Azure Monitor.
- Vous devez configurer la passerelle Log Analytics afin qu’elle fonctionne avec System Center Operations Manager si vos agents System Center Operations Manager ne peuvent pas accéder à Azure Monitor par Internet.

Si vos ordinateurs Windows ou Linux ne peuvent pas se connecter directement au service, vous devez configurer l’agent Log Analytics pour qu’il se connecte à Azure Monitor à l’aide de la passerelle Log Analytics. Vous pouvez télécharger la passerelle Log Analytics à partir du [Centre de téléchargement Microsoft](https://www.microsoft.com/download/details.aspx?id=52666).

## <a name="prerequisites"></a>Prérequis

- Requiert l’offre de solution [Insight & Analytics](https://www.microsoft.com/cloud-platform/operations-management-suite-pricing).
- Si vous utilisez la version précédente de la solution Wire Data, vous devez commencer par la supprimer. Cependant, toutes les données capturées par la solution Wire Data d’origine restent disponibles dans Wire Data 2.0 et la Recherche dans les journaux.
- Des privilèges d’administrateur sont requis pour installer ou désinstaller l’agent de dépendances.
- L’agent de dépendances doit être installé sur un ordinateur équipé d’un système d’exploitation 64 bits.

### <a name="operating-systems"></a>Systèmes d’exploitation

Les sections suivantes listent les systèmes d’exploitation pris en charge par l’agent de dépendances. Wire Data ne prend pas en charge les architectures 32 bits, quel que soit le système d’exploitation.

#### <a name="windows-server"></a>Windows Server

- Windows Server 2019
- Windows Server 2016 1803
- Windows Server 2016
- Windows Server 2012 R2
- Windows Server 2012
- Windows Server 2008 R2 SP1

#### <a name="windows-desktop"></a>Ordinateurs Windows

- Windows 10 1803
- Windows 10
- Windows 8.1
- Windows 8
- Windows 7

#### <a name="supported-linux-operating-systems"></a>Systèmes d’exploitation Linux pris en charge
Les sections suivantes répertorient les systèmes d’exploitation pris en charge par l’agent de dépendances sur Linux.  

- Seules les versions du noyau SMP Linux et par défaut sont prises en charge.
- Les versions non standard du noyau, par exemple PAE et Xen, ne sont prises en charge par aucune distribution Linux. Par exemple, un système avec la chaîne de version « 2.6.16.21-0.8-xen » n’est pas pris en charge.
- Les noyaux personnalisés, y compris les recompilations de noyaux standard, ne sont pas pris en charge.

##### <a name="red-hat-linux-7"></a>Red Hat Linux 7

| Version du SE | Version du noyau |
|:--|:--|
| 7.4 | 3.10.0-693 |
| 7.5 | 3.10.0-862 |
| 7.6 | 3.10.0-957 |

##### <a name="red-hat-linux-6"></a>Red Hat Linux 6

| Version du SE | Version du noyau |
|:--|:--|
| 6.9 | 2.6.32-696 |
| 6.10 | 2.6.32-754 |

##### <a name="centosplus"></a>CentOSPlus
| Version du SE | Version du noyau |
|:--|:--|
| 6.9 | 2.6.32-696.18.7<br>2.6.32-696.30.1 |
| 6.10 | 2.6.32-696.30.1<br>2.6.32-754.3.5 |

##### <a name="ubuntu-server"></a>Serveur Ubuntu

| Version du SE | Version du noyau |
|:--|:--|
| Ubuntu 18.04 | noyau 4.15.\*<br>4.18* |
| Ubuntu 16.04.3 | noyau 4.15.* |
| 16.04 | 4.4.\*<br>4.8.\*<br>4.10.\*<br>4.11.\*<br>4.13.\* |
| 14.04 | 3.13.\*<br>4.4.\* |

##### <a name="suse-linux-11-enterprise-server"></a>SUSE Linux 11 Enterprise Server

| Version du SE | Version du noyau
|:--|:--|
| 11 SP4 | 3.0.* |

##### <a name="suse-linux-12-enterprise-server"></a>SUSE Linux 12 Enterprise Server

| Version du SE | Version du noyau
|:--|:--|
| 12 SP2 | 4.4.* |
| 12 SP3 | 4.4.* |

### <a name="dependency-agent-downloads"></a>Téléchargements de l’agent de dépendances

| Fichier | Système d''exploitation | Version | SHA-256 |
|:--|:--|:--|:--|
| [InstallDependencyAgent-Windows.exe](https://aka.ms/dependencyagentwindows) | Windows | 9.7.4 | A111B92AB6CF28EB68B696C60FE51F980BFDFF78C36A900575E17083972989E0 |
| [InstallDependencyAgent-Linux64.bin](https://aka.ms/dependencyagentlinux) | Linux | 9.7.4 | AB58F3DB8B1C3DEE7512690E5A65F1DFC41B43831543B5C040FCCE8390F2282C |



## <a name="configuration"></a>Configuration

Suivez les étapes ci-dessous afin de configurer la solution Wire Data pour vos espaces de travail.

1. Activez la solution Activity Log Analytics à partir de la [Place de marché Microsoft Azure](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.WireData2OMS?tab=Overview) ou en procédant de la manière décrite dans [Ajouter des solutions de supervision à partir de la galerie de solutions](./solutions.md).
2. Installez l’agent de dépendances sur chacun des ordinateurs sur lesquels vous souhaitez récupérer des données. L’agent de dépendances peut surveiller les connexions aux voisins immédiats ; par conséquent, vous n’avez pas forcément besoin d’un agent sur chaque ordinateur.

> [!NOTE]
> Il n’est pas possible d’ajouter la version précédente de la solution Wire Data à de nouveaux espaces de travail. Si vous avez déjà activé la solution Wire Data d’origine, vous pouvez continuer à l’utiliser. Toutefois, pour utiliser Wire Data 2.0, vous devez tout d’abord supprimer la version d’origine.
> 
 
### <a name="install-the-dependency-agent-on-windows"></a>Installer l’agent de dépendances sous Windows

Des privilèges d’administrateur sont requis pour installer ou désinstaller l’agent.

L’agent de dépendances s’installe sur les ordinateurs sous Windows par le biais de InstallDependencyAgent-Windows.exe. Si vous exécutez ce fichier exécutable sans aucune option, il démarre un Assistant que vous pouvez suivre pour une installation interactive.

Utilisez les étapes suivantes pour installer l’agent de dépendances sur chaque ordinateur exécutant Windows :

1. Installez l’agent Log Analytics en suivant les étapes décrites dans [Collecter les données à partir d’ordinateurs Windows hébergés dans votre environnement](../platform/agent-windows.md).
2. Téléchargez l’agent de dépendances Windows en suivant le lien de la section précédente, puis exécutez-le à l’aide de la commande suivante : `InstallDependencyAgent-Windows.exe`
3. Suivez les instructions de l’Assistant pour installer l’agent.
4. Si le démarrage de l’agent de dépendances échoue, recherchez des informations détaillées sur l’erreur dans les journaux d’activité. Pour les agents Windows, le répertoire des journaux d’activité est %Programfiles%\Microsoft Dependency Agent\logs.

#### <a name="windows-command-line"></a>Ligne de commande Windows

Utilisez les options dans le tableau suivant pour effectuer l’installation à partir d’une ligne de commande. Pour afficher la liste des indicateurs d’installation, exécutez le programme d’installation à l’aide de l’indicateur /? comme suit.

InstallDependencyAgent-Windows.exe /?

| **Indicateur** | **Description** |
| --- | --- |
| <code>/?</code> | Récupérez la liste des options de ligne de commande. |
| <code>/S</code> | Effectuez une installation silencieuse sans invite utilisateur. |

Par défaut, les fichiers de l’agent de dépendances Windows sont placés dans le répertoire C:\Program Files\Microsoft Dependency Agent.

### <a name="install-the-dependency-agent-on-linux"></a>Installer l’agent de dépendances sous Linux

L’accès racine est requis pour installer ou configurer l’agent.

L’agent de dépendances s’installe sur les ordinateurs Linux par le biais de InstallDependencyAgent-Linux64.bin, un script shell avec un fichier binaire à extraction automatique. Vous pouvez exécuter le fichier à l’aide de _sh_ ou ajouter des autorisations d’exécution au fichier lui-même.

Procédez comme suit pour installer l’agent de dépendances sur chaque ordinateur Linux :

1. Installez l’agent Log Analytics en suivant les étapes décrites dans [Collecter les données à partir d’ordinateurs Linux hébergés dans votre environnement](../learn/quick-collect-linux-computer.md#obtain-workspace-id-and-key).
2. Téléchargez l’agent de dépendances Linux en suivant le lien de la section précédente, puis installez-le en tant que racine à l’aide de la commande suivante : sh InstallDependencyAgent-Linux64.bin.
3. Si le démarrage de l’agent de dépendances échoue, recherchez des informations détaillées sur l’erreur dans les journaux d’activité. Sur les agents Linux, le répertoire des journaux est : /var/opt/microsoft/dependency-agent/log.

Pour afficher la liste des indicateurs d’installation, exécutez le programme d’installation avec l’indicateur `-help` comme suit.

```
InstallDependencyAgent-Linux64.bin -help
```

| **Indicateur** | **Description** |
| --- | --- |
| <code>-help</code> | Récupérez la liste des options de ligne de commande. |
| <code>-s</code> | Effectuez une installation silencieuse sans invite utilisateur. |
| <code>--check</code> | Vérifiez les autorisations et le système d’exploitation, sans installer l’agent. |

Les fichiers de l’agent de dépendances sont placés dans les répertoires suivants :

| **Fichiers** | **Lieu** |
| --- | --- |
| Fichiers principaux | /opt/microsoft/dependency-agent |
| Fichiers journaux | /var/opt/microsoft/dependency-agent/log |
| Fichiers de configuration | /etc/opt/microsoft/dependency-agent/config |
| Fichiers exécutables de service | /opt/microsoft/dependency-agent/bin/microsoft-dependency-agent<br><br>/opt/microsoft/dependency-agent/bin/microsoft-dependency-agent-manager |
| Fichiers de stockage binaires | /var/opt/microsoft/dependency-agent/storage |

### <a name="installation-script-examples"></a>Exemples de script d’installation

Pour déployer facilement l’agent de dépendances sur plusieurs serveurs d’un coup, il est plus facile d’utiliser un script. Vous pouvez utiliser les exemples de script suivants pour télécharger et installer l’agent de dépendances sur Windows ou Linux.

#### <a name="powershell-script-for-windows"></a>Script PowerShell pour Windows

```powershell

Invoke-WebRequest "https://aka.ms/dependencyagentwindows" -OutFile InstallDependencyAgent-Windows.exe

.\InstallDependencyAgent-Windows.exe /S

```

#### <a name="shell-script-for-linux"></a>Script shell pour Linux

```
wget --content-disposition https://aka.ms/dependencyagentlinux -O InstallDependencyAgent-Linux64.bin
```

```
sh InstallDependencyAgent-Linux64.bin -s
```

### <a name="desired-state-configuration"></a>DSC (Desired State Configuration)

Pour déployer l’agent de dépendances avec Desired State Configuration, vous pouvez utiliser le module xPSDesiredStateConfiguration et un peu de code comme ceci :

```powershell
Import-DscResource -ModuleName xPSDesiredStateConfiguration

$DAPackageLocalPath = "C:\InstallDependencyAgent-Windows.exe"



Node $NodeName

{

    # Download and install the Dependency agent

    xRemoteFile DAPackage

    {

        Uri = "https://aka.ms/dependencyagentwindows"

        DestinationPath = $DAPackageLocalPath

        DependsOn = "[Package]OI"

    }

    xPackage DA

    {

        Ensure = "Present"

        Name = "Dependency Agent"

        Path = $DAPackageLocalPath

        Arguments = '/S'

        ProductId = ""

        InstalledCheckRegKey = "HKEY\_LOCAL\_MACHINE\SOFTWARE\Wow6432Node\Microsoft\Windows\CurrentVersion\Uninstall\DependencyAgent"

        InstalledCheckRegValueName = "DisplayName"

        InstalledCheckRegValueData = "Dependency Agent"

    }

}

```

### <a name="uninstall-the-dependency-agent"></a>Désinstaller l’agent de dépendances

Utilisez les sections suivantes comme guide pour supprimer l’agent de dépendances.

#### <a name="uninstall-the-dependency-agent-on-windows"></a>Désinstaller l’agent de dépendances sur Windows

Un administrateur peut désinstaller l’agent de dépendances pour Windows au moyen du Panneau de configuration.

Un administrateur peut également exécuter %Programfiles%\Microsoft Dependency Agent\Uninstall.exe pour désinstaller l’agent de dépendances.

#### <a name="uninstall-the-dependency-agent-on-linux"></a>Désinstaller l’agent de dépendances sur Linux

Pour désinstaller complètement l’agent de dépendances de Linux, vous devez supprimer l’agent lui-même et le connecteur qui est installé automatiquement avec lui. Vous pouvez désinstaller les deux à l’aide de la commande ci-dessous :

```
rpm -e dependency-agent dependency-agent-connector
```

## <a name="management-packs"></a>Packs d’administration

Quand Wire Data est activé dans un espace de travail Log Analytics, un pack d’administration de 300 Ko est envoyé à tous les serveurs Windows de cet espace de travail. Si vous utilisez des agents System Center Operations Manager dans un [groupe d’administration connecté](../platform/om-agents.md), le pack d’administration Dependency Monitor est déployé à partir de System Center Operations Manager. Si les agents sont connectés directement, Azure Monitor remet le pack d’administration.

Le pack d’administration se nomme Microsoft.IntelligencePacks.ApplicationDependencyMonitor. Il est enregistré dans : %Programfiles%\Microsoft Monitoring Agent\Agent\Health Service State\Management Packs. La source de données utilisée par le pack d’administration est : %Program files%\Microsoft Monitoring Agent\Agent\Health Service State\Resources&lt;AutoGeneratedID&gt;\Microsoft.EnterpriseManagement.Advisor.ApplicationDependencyMonitorDataSource.dll.

## <a name="using-the-solution"></a>Utilisation de la solution

Utilisez les informations suivantes pour installer et configurer la solution.

- La solution de données de communication acquiert des données à partir d’ordinateurs exécutant Windows Server 2012 R2, Windows 8.1 et systèmes d’exploitation ultérieurs.
- Vous devez installer Microsoft .NET Framework 4.0 ou ultérieur sur les ordinateurs à partir desquels vous souhaitez acquérir des données de communication.
- Ajoutez la solution Wire Data à votre espace de travail Log Analytics en appliquant la procédure décrite sur la page [Ajouter des solutions de surveillance à partir de la galerie de solutions](solutions.md). Aucune configuration supplémentaire n’est requise.
- Si vous souhaitez afficher les données de communication d’une solution en particulier, celle-ci doit être déjà ajoutée à votre espace de travail.

Une fois les agents et la solution installés, la vignette de Wire Data 2.0 s’affiche dans votre espace de travail.

![Vignette de Wire Data](./media/wire-data/wire-data-tile.png)

## <a name="using-the-wire-data-20-solution"></a>Utiliser la solution Wire Data 2.0

Sur la page **Vue d’ensemble** de votre espace de travail Log Analytics dans le portail Azure, cliquez sur la vignette **Wire Data 2.0** pour ouvrir le tableau de bord Wire Data. Le tableau de bord comprend les panneaux figurant dans le tableau suivant. Chaque panneau répertorie jusqu'à 10 éléments répondant à ses critères en ce qui concerne l’étendue et l’intervalle de temps spécifiés. Vous pouvez exécuter une recherche dans les journaux qui renvoie tous les enregistrements. Pour cela, cliquez sur **Afficher tout** en bas du panneau ou cliquez sur l’en-tête de panneau.

| **Panneau** | **Description** |
| --- | --- |
| Agents qui capturent le trafic réseau | Affiche le nombre d’agents qui capturent le trafic réseau et liste les 10 ordinateurs qui en capturent le plus. Cliquez sur le nombre pour exécuter une recherche de <code>WireData \| summarize sum(TotalBytes) by Computer \| take 500000</code> dans les journaux. Cliquez sur un ordinateur dans la liste pour exécuter une recherche dans les journaux qui renvoie le nombre total d’octets capturés. |
| Sous-réseaux locaux | Affiche le nombre de sous-réseaux locaux découverts par les agents.  Cliquez sur le nombre pour exécuter une recherche de <code>WireData \| summarize sum(TotalBytes) by LocalSubnet</code> dans les journaux, qui liste tous les sous-réseaux avec le nombre d’octets envoyés sur chacun d’eux. Cliquez sur un sous-réseau dans la liste pour exécuter une recherche dans les journaux qui renvoie le nombre total d’octets envoyés dessus. |
| Protocole de niveau application | Affiche le nombre de protocoles de niveau application en cours d’utilisation, découverts par les agents. Cliquez sur le nombre pour exécuter une recherche de <code>WireData \| summarize sum(TotalBytes) by ApplicationProtocol</code> dans les journaux. Cliquez sur un protocole dans la liste pour exécuter une recherche dans les journaux qui renvoie le nombre total d’octets envoyés par son biais. |

![Tableau de bord de Wire Data](./media/wire-data/wire-data-dash.png)

Vous pouvez utiliser le panneau **Agents qui capturent le trafic réseau** pour déterminer quelle quantité de bande passante réseau est consommée par les ordinateurs. Il vous permet de trouver facilement l’ordinateur _qui communique le plus_ dans votre environnement. Ces ordinateurs peuvent être surchargés, agir de façon anormale ou utiliser plus de ressources réseau que la normale.

![Capture d’écran du panneau Agents qui capturent le trafic réseau dans le tableau de bord de Wire Data 2.0, montrant la bande passante réseau consommée par chaque ordinateur.](./media/wire-data/log-search-example01.png)

De même, vous pouvez utiliser le panneau **Sous-réseaux locaux** pour déterminer quelle quantité de trafic réseau circule sur vos sous-réseaux. Les utilisateurs définissent souvent des sous-réseaux autour des zones critiques de leurs applications. Ce panneau permet de surveiller ces zones.

![Capture d’écran du panneau Sous-réseaux locaux dans le tableau de bord de Wire Data 2.0, montrant la bande passante réseau consommée par chaque LocalSubnet.](./media/wire-data/log-search-example02.png)

Le Panneau **Protocoles de niveau application** est utile, car il permet de savoir quels protocoles sont en cours d’utilisation. Par exemple, vous pouvez vouloir que SSH ne soit pas utilisé dans votre environnement réseau. Les informations disponibles sur le panneau peuvent le confirmer ou le démentir rapidement.

![Capture d’écran du panneau Protocoles au niveau de l’application dans le tableau de bord de Wire Data 2.0, montrant la bande passante réseau consommée par chaque protocole.](./media/wire-data/log-search-example03.png)

Il est également utile de savoir si le trafic des protocoles augmente ou diminue au fil du temps. Par exemple, si la quantité de données transmises par une application augmente, il peut être nécessaire ou intéressant de le savoir.

## <a name="input-data"></a>Données d’entrée

Les données de communication collectent des métadonnées sur le trafic réseau à l’aide des agents que vous avez activés. Chaque agent envoie des données toutes les 15 secondes environ.

## <a name="output-data"></a>Données de sortie

Un enregistrement de type _WireData_ est créé pour chaque type de données d’entrée. Les propriétés de ces enregistrements sont décrites dans le tableau suivant :

| Propriété | Description |
|---|---|
| Computer | Nom de l’ordinateur sur lequel les données ont été recueillies |
| TimeGenerated | Heure de l’enregistrement |
| LocalIP | Adresse IP de l’ordinateur local |
| SessionState | Connecté ou déconnecté |
| ReceivedBytes | Nombre d’octets reçus |
| ProtocolName | Nom du protocole réseau utilisé |
| IPVersion | Version de l’adresse IP |
| Sens | Entrant ou sortant |
| MaliciousIP | Adresse IP d’une source malveillante connue |
| severity | Niveau de gravité suspecté |
| RemoteIPCountry | Pays/région de l’adresse IP distante |
| ManagementGroupName | Nom du groupe d’administration Operations Manager |
| SourceSystem | Source où les données ont été recueillies |
| SessionStartTime | Heure de début de la session |
| SessionEndTime | Heure de fin de la session |
| LocalSubnet | Sous-réseau sur lequel les données ont été recueillies |
| LocalPortNumber | Numéro de port local |
| RemoteIP | Adresse IP distante utilisée par l’ordinateur distant |
| RemotePortNumber | Numéro de port utilisé par l’adresse IP distante |
| SessionID | Valeur unique qui identifie la session de communication entre deux adresses IP |
| SentBytes | Nombre d’octets envoyés |
| TotalBytes | Nombre total d’octets envoyés au cours de la session |
| ApplicationProtocol | Type de protocole réseau utilisé   |
| ProcessID | ID du processus Windows |
| ProcessName | Chemin d’accès et nom de fichier du processus |
| RemoteIPLongitude | Valeur de longitude de l’adresse IP |
| RemoteIPLatitude | Valeur de latitude de l’adresse IP |

## <a name="next-steps"></a>Étapes suivantes

- [Lancez une recherche dans les journaux d’activité](../log-query/log-query-overview.md) pour afficher des enregistrements détaillés sur les recherches de données de communication.


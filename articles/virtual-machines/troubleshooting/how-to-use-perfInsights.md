---
title: Utilisation de PerfInsights dans Microsoft Azure | Microsoft Docs
description: Découvrez comment utiliser PerfInsights pour résoudre les problèmes de performances liés aux machines virtuelles Windows.
services: virtual-machines-windows
documentationcenter: ''
author: anandhms
manager: dcscontentpm
editor: na
tags: ''
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/31/2018
ms.author: genli
ms.openlocfilehash: 9e298bf39446024f384b9af142fe3000e936bb6d
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656850"
---
# <a name="how-to-use-perfinsights-in-azure"></a>Comment utiliser PerfInsights dans Azure

[PerfInsights](https://aka.ms/perfinsightsdownload) est un outil de diagnostics autonome qui collecte et analyse les données de diagnostic, et fournit un rapport afin de faciliter la résolution des problèmes de performances liés aux machines virtuelles Windows dans Azure. Vous pouvez exécuter PerfInsights sur les machines virtuelles sous forme d’outil autonome, directement à partir du portail en utilisant les [diagnostics de performances pour les machines virtuelles Azure](performance-diagnostics.md) ou en installant [l’extension de machine virtuelle des diagnostics de performances Azure](performance-diagnostics-vm-extension.md).

Si vous rencontrez des problèmes de performances avec des machines virtuelles, exécutez cet outil avant de contacter le support.

## <a name="supported-troubleshooting-scenarios"></a>Scénarios de résolution des problèmes pris en charge

PerfInsights peut collecter et analyser plusieurs types d’informations. Les sections suivantes couvrent les scénarios courants.

### <a name="quick-performance-analysis"></a>Analyse rapide des performances

Ce scénario collecte la configuration de disque et d’autres informations importantes, notamment :

-   Journaux d’événements

-   État du réseau pour toutes les connexions entrantes et sortantes

-   Paramètres de configuration du pare-feu et du réseau

-   Liste des tâches pour toutes les applications en cours d’exécution sur le système

-   Paramètres de configuration de base de données Microsoft SQL Server (si la machine virtuelle est identifiée en tant que serveur exécutant SQL Server)

-   Compteurs de fiabilité de stockage

-   Correctifs logiciels Windows importants

-   Pilotes de filtre installés

Il s’agit d’une collecte passive d’informations qui ne sont pas censées affecter le système. 

>[!Note]
>Ce scénario est automatiquement inclus dans les scénarios suivants :

### <a name="benchmarking"></a>Benchmarking

Ce scénario exécute le test d’évaluation [Diskspd](https://github.com/Microsoft/diskspd) (E/S par seconde et Mbits/s) pour tous les disques joints à la machine virtuelle. 

> [!Note]
> Ce scénario peut affecter le système et ne doit pas être exécuté sur un système de production en direct. Si nécessaire, exécutez ce scénario dans une fenêtre de maintenance dédiée pour éviter tout problème. Une charge de travail accrue qui est provoquée par un test d’évaluation ou un suivi peut nuire aux performances de votre machine virtuelle.
>

### <a name="performance-analysis"></a>Analyse des performances

Ce scénario exécute un suivi du [compteur de performances](/windows/win32/perfctrs/performance-counters-portal) en utilisant les compteurs spécifiés dans le fichier RuleEngineConfig.json. Si la machine virtuelle est identifiée en tant que serveur qui exécute SQL Server, elle exécute un suivi du compteur de performances. Il le fait en utilisant les compteurs trouvés dans le fichier RuleEngineConfig.json. Ce scénario inclut également des données de diagnostics de performances.

### <a name="azure-files-analysis"></a>Analyse de fichiers Azure

Ce scénario exécute une capture du compteur de performances spéciale ainsi qu’un suivi du réseau. La capture inclut tous les compteurs de partages de clients SMB. Voici quelques compteurs de performances de partages de clients SMB clés, qui font partie de la capture :

| **Type**     | **Compteur de partages de clients SMB** |
|--------------|-------------------------------|
| E/S par seconde         | Requêtes de données/s             |
|              | Requêtes de lecture/s             |
|              | Requêtes d’écriture/s            |
| Latence      | Requête de données/s (moyenne)         |
|              | Lecture/s (moyenne)                 |
|              | Écriture/s (moyenne)                |
| Taille d’E/S      | Avg. Octets/requête de données       |
|              | Avg. Octets/lecture               |
|              | Avg. Octets/écriture              |
| Débit   | Octets de données/s                |
|              | Octets de lecture/s                |
|              | Octets d’écriture/s               |
| Longueur de la file d’attente | Avg. Longueur de la file d’attente de lecture        |
|              | Avg. Longueur de la file d’attente d’écriture       |
|              | Avg. Longueur de la file d’attente de données        |

### <a name="advanced-performance-analysis"></a>Analyse avancée des performances

Lorsque vous exécutez une analyse avancée des performances, vous sélectionnez des traces à exécuter en parallèle. Vous pouvez toutes les exécuter (compteur de performances, Xperf, réseau et StorPort) si vous le souhaitez.  

> [!Note]
> Ce scénario peut affecter le système et ne doit pas être exécuté sur un système de production en direct. Si nécessaire, exécutez ce scénario dans une fenêtre de maintenance dédiée pour éviter tout problème. Une charge de travail accrue qui est provoquée par un test d’évaluation ou un suivi peut nuire aux performances de votre machine virtuelle.
>

## <a name="what-kind-of-information-is-collected-by-perfinsights"></a>Quelles informations sont collectées par PerfInsights ?

Les informations portant sur la configuration de la machine virtuelle Windows, des disques ou des pools de stockage, les compteurs de performances, les journaux d’activité et les différentes traces sont recueillies. Cela varie selon le scénario de performances que vous utilisez. Le tableau suivant fournit les détails :

| Données collectées | Analyse rapide des performances | Benchmarking | Analyse des performances | Analyse de fichiers Azure | Analyse avancée des performances |
|----------------------------------|----------------------------|------------------------------------|--------------------------|--------------------------------|----------------------|
| Informations tirées des journaux d’événements       | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Informations système                | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Mappage de volume                        | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Mappage de disque                          | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Tâches en cours d’exécution                     | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Compteurs de fiabilité de stockage      | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Informations sur le stockage               | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Sortie Fsutil                     | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Informations du pilote de filtre                | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Sortie Netstat                    | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Configuration réseau             | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Configuration du pare-feu            | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Configuration de SQL Server          | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Suivis des diagnostics de performances *  | Oui                        | Oui                                | Oui                      | Oui                  | Oui                  |
| Suivi du compteur de performances **      |                            |                                    | Oui                      |                      | Oui                  |
| Suivi du compteur SMB **              |                            |                                    |                          | Oui                  |                      |
| Suivi du compteur SQL Server **       |                            |                                    | Oui                      |                      | Oui                  |
| Suivi XPerf                       |                            |                                    |                          |                      | Oui                  |
| Suivi StorPort                    |                            |                                    |                          |                      | Oui                  |
| Suivi réseau                     |                            |                                    |                          | Oui                  | Oui                  |
| Suivi d’évaluation Diskspd **_       |                            | Oui                                |                          |                      |                      |
|       |                            |                         |                                                   |                      |                      |

### <a name="performance-diagnostics-trace-_"></a>Suivi des diagnostics de performances (_)

Exécute un moteur basé sur des règles en arrière-plan afin de collecter des données et diagnostiquer les problèmes de performances en cours. Les règles actuellement prises en charge sont les suivantes :

- Règle HighCpuUsage : détecte les périodes d’utilisation élevée du processeur et montre les principaux utilisateurs du processeur pendant ces périodes.
- Règle HighDiskUsage : détecte les périodes d’utilisation élevée de disque sur les disques physiques et montre les principaux utilisateurs de disque pendant ces périodes.
- Règle HighResolutionDiskMetric : montre les mesures d’E/S par seconde, de débit et de latence d’E/S par intervalles de 50 millisecondes pour chaque disque physique. Cela permet d’identifier rapidement les périodes de limitation de disque.
- Règle HighMemoryUsage : détecte les périodes d’utilisation élevée de la mémoire et montre les principaux utilisateurs de la mémoire pendant ces périodes.

> [!NOTE] 
> Actuellement, les versions Windows incluant .NET Framework 4.5 ou versions ultérieures sont prises en charge.

### <a name="performance-counter-trace-"></a>Suivi du compteur de performances (\*\*)

Collecte les compteurs de performances suivants :

- \Process, \Processor, \Memory, \Thread, \PhysicalDisk, et \LogicalDisk
- \Cache\Dirty Pages, \Cache\Lazy Write Flushes/sec, \Server\Pool Nonpaged, Failures, et \Server\Pool Paged Failures
- Certains compteurs sous \Network Interface, \IPv4\Datagrams, \IPv6\Datagrams, \TCPv4\Segments, \TCPv6\Segments,  \Network Adapter, \WFPv4\Packets, \WFPv6\Packets, \UDPv4\Datagrams, \UDPv6\Datagrams, \TCPv4\Connection, \TCPv6\Connection, \Network QoS Policy\Packets, \Per Processor Network Interface Card Activity, and \Microsoft Winsock BSP

#### <a name="for-sql-server-instances"></a>Pour les instances SQL Server
- \SQL Server:Buffer Manager, \SQLServer:Resource Pool Stats, et \SQLServer:SQL Statistics\
- \SQLServer:Locks, \SQLServer:General, Statistics
- \SQLServer:Access Methods

#### <a name="for-azure-files"></a>Pour les fichiers Azure
\SMB Client Shares

### <a name="diskspd-benchmark-trace-_"></a>Suivi d’évaluation Diskspd (**_)
Tests de charge de travail d’E/S Diskspd (disque de système d’exploitation [écriture] et disques du pool [lecture/écriture])

## <a name="run-the-perfinsights-tool-on-your-vm"></a>Exécuter l’outil PerfInsights sur votre machine virtuelle

### <a name="what-do-i-have-to-know-before-i-run-the-tool"></a>Que dois-je savoir avant d’exécuter l’outil ? 

#### <a name="tool-requirements"></a>Exigences de l’outil

-  Cet outil doit être exécuté sur la machine virtuelle qui présente le problème de performances. 

-  Les systèmes d’exploitation suivants sont pris en charge : _ Windows Server 2019
   * Windows Server 2016
   * Windows Server 2012 R2
   * Windows Server 2012
   * Windows Server 2008 R2
   * Windows 10
   * Windows 8.1
   * Windows 8

#### <a name="possible-problems-when-you-run-the-tool-on-production-vms"></a>Problèmes possibles lorsque vous exécutez l’outil sur des machines virtuelles de production

-  Pour les scénarios d’évaluation ou d’« Analyse avancée des performances » configurés pour utiliser XPerf ou DiskSpd, l’outil peut nuire aux performances de la machine virtuelle. Ces scénarios ne doivent pas être exécutés dans un environnement de production.

-  Pour les scénarios d’évaluation ou d’« Analyse avancée des performances » configurés pour utiliser DiskSpd, assurez-vous qu’aucune autre activité en arrière-plan n’interfère avec la charge de travail d’E/S.

-  Par défaut, l’outil utilise le disque de stockage temporaire pour collecter les données. Si le suivi est activé sur une durée plus longue, la quantité de données collectées peut être pertinente. Cela peut réduire la disponibilité de l’espace sur le disque temporaire, et donc affecter toute application s’appuyant sur ce disque.

### <a name="how-do-i-run-perfinsights"></a>Comment exécuter PerfInsights ? 

Vous pouvez exécuter PerfInsights sur une machine virtuelle en installant l’[extension de diagnostic de performance des machines virtuelles Azure](performance-diagnostics-vm-extension.md). Vous pouvez également l’exécuter en tant qu’outil autonome. 

**Installer et exécuter PerfInsights à partir du portail Azure**

Pour plus d’informations sur cette option, consultez [Installer l’extension de diagnostic de performance Azure](performance-diagnostics-vm-extension.md#install-the-extension).  

**Exécuter PerfInsights en mode autonome**

Pour exécuter l’outil PerfInsights, suivez ces étapes :


1. Téléchargez [PerfInsights.zip](https://aka.ms/perfinsightsdownload).

2. Débloquez le fichier PerfInsights.zip. Pour ce faire, cliquez avec le bouton droit sur le fichier PerfInsights.zip, puis sélectionnez **Propriétés**. Sous l’onglet **Général**, sélectionnez **Débloquer**, puis **OK**. Cela garantit que l’outil s’exécute sans invite de sécurité supplémentaire.  

    ![Capture d’écran des propriétés PerfInsights, avec mise en surbrillance du déblocage](media/how-to-use-perfInsights/pi-unlock-file.png)

3.  Développez le fichier compressé PerfInsights.zip dans votre disque temporaire (par défaut, il s’agit généralement du disque D). 

4.  Ouvrez l’invite de commandes Windows en tant qu’administrateur, puis exécutez PerfInsights.exe pour afficher les paramètres de ligne de commande disponibles.

    ```
    cd <the path of PerfInsights folder>
    PerfInsights
    ```
    ![Capture d’écran de la sortie de ligne de commande de PerfInsights](media/how-to-use-perfInsights/pi-commandline.png)
    
    La syntaxe de base pour l’exécution de scénarios PerfInsights est la suivante :
    
    ```
    PerfInsights /run <ScenarioName> [AdditionalOptions]
    ```

    Vous pouvez utiliser l’exemple ci-dessous pour exécuter le scénario d’analyse de performances pendant 5 minutes :
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    Vous pouvez utiliser l’exemple suivant pour exécuter le scénario avancé avec les traces de compteur Xperf et Performances pendant 5 minutes :
    
    ```
    PerfInsights /run advanced xp /d 300 /AcceptDisclaimerAndShareDiagnostics
    ```

    Vous pouvez utiliser l’exemple suivant pour exécuter le scénario d’analyse de performances pendant 5 minutes et charger le fichier zip de résultat dans le compte de stockage :
    
    ```
    PerfInsights /run vmslow /d 300 /AcceptDisclaimerAndShareDiagnostics /sa <StorageAccountName> /sk <StorageAccountKey>
    ```

    Vous pouvez rechercher tous les scénarios et options disponibles à l’aide de la commande **/liste** :
    
    ```
    PerfInsights /list
    ```

    >[!Note]
    >Avant d’exécuter un scénario, PerfInsights invite l’utilisateur à accepter de partager les informations de diagnostic et à accepter le CLUF. Utilisez l’option **/AcceptDisclaimerAndShareDiagnostics** pour ignorer ces messages. 
    >
    >Si vous avez un ticket de support actif avec Microsoft et que vous exécutez PerfInsights sur demande de l’ingénieur de support avec qui vous travaillez, assurez-vous de fournir le numéro du ticket de support à l’aide de l’option **/sr**.
    >
    >Par défaut, PerfInsights va tenter de se mettre à jour vers la dernière version si elle est disponible. Utilisez le paramètre **/SkipAutoUpdate** ou **/sau** pour ignorer la mise à jour automatique.  
    >
    >Si le commutateur de durée **/d** n’est pas spécifié, PerfInsights vous invitera à reproduire le problème lors de l’exécution de vmslow, d’azurefiles et des scénarios avancés. 

Quand les traces ou les opérations sont terminées, un nouveau fichier apparaît dans le même dossier que PerfInsights. Le nom du fichier est **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.zip**. Vous pouvez envoyer ce fichier à l’agent de Support pour analyse ou ouvrir le rapport dans le fichier zip pour examiner les résultats et recommandations.

## <a name="review-the-diagnostics-report"></a>Examinez le rapport de diagnostics

Le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.zip** peut inclure un rapport HTML détaillant les conclusions de PerfInsights. Pour consulter le rapport, développez le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.zip**, puis ouvrez le fichier **PerfInsights Report.html**.

Sélectionnez l’onglet **Conclusions**.

![Capture d’écran de l’onglet Vue d’ensemble du rapport PerfInsights.](media/how-to-use-perfInsights/pi-finding-tab.png)
![Capture d’écran de l’onglet Stockage du rapport PerfInsights.](media/how-to-use-perfInsights/pi-findings.png)

> [!NOTE] 
> Les conclusions identifiées comme étant d’importance élevée sont des problèmes connus qui peuvent conduire à des problèmes de performances. Les conclusions identifiées comme étant d’importance moyenne représentent des configurations non optimales ne provoquant pas forcément de problèmes de performances. Les conclusions identifiées comme étant d’importance faible sont des instructions données à titre informatif uniquement.

Examinez les suggestions et les liens pour toutes les conclusions d’importance élevée et moyenne. En savoir plus sur la façon dont elles peuvent affecter les performances, et sur les meilleures pratiques pour les configurations de performances optimisées.

### <a name="storage-tab"></a>Onglet Stockage

La section **Conclusions** affiche les différentes conclusions et recommandations relatives au stockage.

Les sections **Mappage de disque** et **Mappage de volume** décrivent les liens entre volumes logiques et disques physiques.

Dans la perspective du disque physique (Mappage de disque), le tableau affiche tous les volumes logiques en cours d’exécution sur le disque. Dans l’exemple suivant, **PhysicalDrive2** exécute deux volumes logiques créés sur plusieurs partitions (J et H) :

![Capture d’écran de l’onglet du disque](media/how-to-use-perfInsights/pi-disk-tab.png)

Dans la perspective du volume (Mappage de volume), les tables affichent tous les disques physiques sous chaque volume logique. Pour les disques RAID/dynamiques, vous pouvez exécuter un volume logique sur plusieurs disques physiques. Dans l’exemple suivant, *C:\\mount* est un point de montage configuré en tant que *SpannedDisk* sur les disques physiques 2 et 3 :

![Capture d’écran de l’onglet de volume](media/how-to-use-perfInsights/pi-volume-tab.png)

### <a name="sql-tab"></a>Onglet SQL

Si la machine virtuelle cible héberge des instances SQL Server, un onglet supplémentaire appelé **SQL** apparaît dans le rapport :

![Capture d’écran de l’onglet SQL](media/how-to-use-perfInsights/pi-sql-tab.png)

Cette section contient un onglet **Conclusions**, et des sous-onglets supplémentaires pour chacune des instances SQL Server hébergées sur la machine virtuelle.

L’onglet **Conclusions** contient une liste de tous les problèmes de performances relatifs à SQL trouvés, ainsi que les recommandations correspondantes.

Dans l’exemple suivant, **PhysicalDrive0** (en cours d’exécution du lecteur C) s’affiche. Cela est dû au fait que les fichiers **modeldev** et **modellog** se trouvent sur le disque C, et ils sont de types différents (par exemple, fichier de données et journal des transactions, respectivement).

![Capture d’écran des informations du journal](media/how-to-use-perfInsights/pi-log-info.png)

Les onglets des instances spécifiques de SQL Server contiennent une section générale qui affiche des informations de base sur l’instance sélectionnée. Les onglets contiennent également des sections supplémentaires pour les informations avancées, notamment les paramètres, les configurations et les options utilisateur.

### <a name="diagnostic-tab"></a>Onglet Diagnostic
L’onglet **Diagnostic** contient des informations sur les principaux consommateurs d’UC, de disque et de mémoire sur l’ordinateur pour la durée de l’exécution de PerfInsights. Vous pouvez également obtenir des informations sur les correctifs critiques non présents dans le système, la liste des tâches et les événements système importants. 

## <a name="references-to-the-external-tools-used"></a>Références aux outils externes utilisés

### <a name="diskspd"></a>Diskspd

Diskspd est un générateur de charge de stockage et un outil de test de performances de Microsoft. Pour plus d’informations, consultez [Diskspd](https://github.com/Microsoft/diskspd).

### <a name="xperf"></a>XPerf

Xperf est un outil en ligne de commande qui permet de capturer des suivis à partir de Windows Performance Toolkit. Pour plus d’informations, consultez [Windows Performance Toolkit : Xperf](/archive/blogs/ntdebugging/windows-performance-toolkit-xperf).

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez charger les journaux de diagnostic et les rapports vers le Support Microsoft pour un examen approfondi. Le Support peut vous demander de transmettre la sortie générée par PerfInsights pour faciliter le processus de dépannage.

La capture d’écran suivante affiche un message semblable à ce que vous pouvez recevoir :

![Capture d’écran de l’exemple de message envoyé par le Support Microsoft](media/how-to-use-perfInsights/pi-support-email.png)

Suivez les instructions dans le message pour accéder à l’espace de travail de transfert de fichier. Pour plus de sécurité, vous devez modifier votre mot de passe à la première utilisation.

Après vous être connecté, vous voyez s’afficher une boîte de dialogue vous invitant à charger le fichier **PerformanceDiagnostics\_aaaa-MM-jj\_hh-mm-ss-fff.zip** collecté par PerfInsights.

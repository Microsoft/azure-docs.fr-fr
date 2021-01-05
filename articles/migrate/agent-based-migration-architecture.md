---
title: Migration basée sur un agent dans Migration de serveur Azure Migrate
description: Fournit une vue d'ensemble de la migration d’une machine virtuelle VMware basée sur agent dans Azure Migrate.
author: rahulg1190
ms.author: rahugup
ms.manager: bsiva
ms.topic: conceptual
ms.date: 02/17/2020
ms.openlocfilehash: 1389b03d87beb24cba50fec0a782de8b7f935557
ms.sourcegitcommit: ea551dad8d870ddcc0fee4423026f51bf4532e19
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96753890"
---
# <a name="agent-based-migration-architecture"></a>Architecture de la migration basée sur agent

Cet article fournit une vue d’ensemble de l’architecture et des processus utilisés pour la réplication basée sur agent des machines virtuelles VMware avec l’outil [Azure Migrate : Migration de serveur](migrate-services-overview.md#azure-migrate-server-assessment-tool).

Azure Migrate : Migration de serveur vous permet de répliquer des machines virtuelles VMware à l’aide de deux options :

- Migrer des machines virtuelles à l'aide de la réplication basée sur agent, comme décrit dans cet article.
- Migrer des machines virtuelles VMware à l’aide d’une réplication sans agent. Cela permet de migrer des machines virtuelles sans devoir installer quoi que ce soit.

Apprenez-en davantage sur la [sélection et la comparaison](server-migrate-overview.md) des méthodes de migration pour machines virtuelles VMware. 


## <a name="agent-based-migration"></a>Migration basée sur agent

La migration basée sur agent est utilisée pour migrer des machines virtuelles VMware et des serveurs physiques locaux vers Azure. Elle permet également de migrer d'autres serveurs virtualisés locaux ainsi que des machines virtuelles cloud privées et publiques, notamment les instances AWS et les machines virtuelles GCP. Dans Azure Migrate, la migration basée sur agent utilise plusieurs fonctionnalités principales du service [Azure Site Recovery](../site-recovery/site-recovery-overview.md).


## <a name="architectural-components"></a>Composants architecturaux

Le diagramme illustre les composants impliqués dans la migration basée sur agent.

![Diagramme illustrant les composants de la migration par agent, expliqués dans un tableau](./media/agent-based-replication-architecture/architecture.png)

Le tableau résume les composants utilisés pour la migration basée sur agent.

**Composant** | **Détails** | **Installation**
--- | --- | ---
**Appliance de réplication** | L'appliance de réplication (serveur de configuration/serveur de traitement) est une machine locale qui sert de passerelle entre l'environnement local et l'outil Migration de serveur. L'appliance découvre l'inventaire des machines locales, ce qui permet à l'outil Migration de serveur d'orchestrer la réplication et la migration. L'appliance comporte deux composants :<br/><br/> **Serveur de configuration** : Se connecte à l'outil Migration de serveur et coordonne la réplication.<br/> **Serveur de traitement**: Gère la réplication des données. Le serveur de processus reçoit les données des machines, les compresse, les chiffre, puis les envoie à Azure. Dans Azure, l'outil Migration de serveur écrit les données sur les disques managés. | Par défaut, le serveur de processus est installé avec le serveur de configuration sur l’appliance de réplication.
**Service de mobilité** | Le service Mobility est un agent installé sur chaque machine que vous souhaitez répliquer et migrer. Il envoie les données de réplication de la machine au serveur de processus. | Les fichiers d'installation pour différentes versions du service Mobility se trouvent sur l'appliance de réplication. Vous téléchargez et installez l'agent dont vous avez besoin en fonction du système d'exploitation et de la version de la machine que vous souhaitez répliquer.

## <a name="mobility-service-installation"></a>Installation du service Mobilité

Vous pouvez déployer le service Mobilité à l’aide des méthodes suivantes :

- **Installation Push** : Le service Mobility est installé par le serveur de processus lorsque vous activez la protection d'une machine. 
- **Installation manuelle** : vous pouvez installer le service Mobility manuellement sur chaque machine par le biais de l’interface utilisateur ou d’une invite de commandes.

Le service Mobility communique avec l'appliance de réplication et les machines répliquées. Si un logiciel antivirus s'exécute sur l'appliance de réplication, les serveurs de processus ou les machines en cours de réplication, les dossiers suivants doivent être exclus de l'analyse :


- C:\Program Files\Microsoft Azure Recovery Services Agent
- C:\ProgramData\ASR
- C:\ProgramData\ASRLogs
- C:\ProgramData\ASRSetupLogs
- C:\ProgramData\LogUploadServiceLogs
- C:\ProgramData\Microsoft Azure Site Recovery
- C:\Program Files (x86)\Microsoft Azure Site Recovery
- C:\ProgramData\ASR\agent (machines Windows sur lesquelles le service Mobility est installé)

## <a name="replication-process"></a>Processus de réplication

1. Lorsque vous activez la réplication pour une machine, la réplication initiale vers Azure commence.
2. Pendant la réplication initiale, le service Mobility lit les données des disques de la machine et les envoie au serveur de processus.
3. Ces données sont utilisées pour alimenter une copie du disque dans votre abonnement Azure. 
4. La réplication des modifications delta dans Azure commence à l’issue de la réplication initiale. La réplication se fait au niveau du bloc et de façon quasi-continue.
4. Le service Mobility intercepte les données écrites dans la mémoire du disque, en s'intégrant au sous-système de stockage du système d'exploitation. Cette méthode évite les opérations d'E/S sur le disque de la machine de réplication pour une réplication incrémentale. 
5. Le suivi des modifications d’une machine est envoyé au serveur de traitement sur le port HTTPS 9443 entrant. Ce port peut être modifié. Le serveur de processus compresse les données, les chiffre, puis les envoie à Azure. 

## <a name="ports"></a>Ports

**Appareil** | **Connection**
--- | --- 
**Réplication des machines** | Le service Mobility en cours d’exécution sur des machines virtuelles communique avec l'appliance de réplication locale via le port HTTPS 443 entrant, pour la gestion de la réplication.<br/><br/> Les machines envoient des données de réplication au serveur de traitement sur le port HTTPS 9443 entrant. Ce port peut être modifié.
**Appliance de réplication** | L’appliance de réplication orchestre la réplication avec Azure sur le port HTTPS 443 sortant.
**Serveur de traitement** | Le serveur de traitement reçoit les données de réplication, les optimise et les chiffre, puis les envoie au stockage Azure via le port 443 sortant.


## <a name="performance-and-scaling"></a>Performances et mise à l’échelle

Par défaut, vous déployez une seule appliance de réplication qui exécute à la fois le serveur de configuration et le serveur de processus. Si vous ne répliquez que quelques machines, ce déploiement est suffisant. Cependant, si vous répliquez et migrez des centaines de machines, un seul serveur de processus risque de ne pas être capable de gérer tout le trafic de réplication. Dans ce cas, vous pouvez déployer des serveurs de processus supplémentaires et évolutifs.

### <a name="plan-vmware-deployment"></a>Planifier le déploiement de VMware

Si vous répliquez des machines virtuelles VMware, vous pouvez utiliser le [Planificateur de déploiement Azure Site Recovery](../site-recovery/site-recovery-deployment-planner.md) pour VMware afin de déterminer les exigences de performance, notamment le taux de modification quotidien des données et les serveurs de processus nécessaires.

### <a name="replication-appliance-capacity"></a>Capacité de l’appliance de réplication

Utilisez les valeurs de ce tableau pour déterminer si vous avez besoin d'un serveur de processus supplémentaire dans votre déploiement.

- Si votre taux de modification quotidien (taux de variation) est supérieur à 2 To, déployez un serveur de processus supplémentaire.
- Si vous répliquez plus de 200 machines, déployez une appliance de réplication supplémentaire.

**UC** | **Mémoire** | **Espace libre de mise en cache des données** | **Taux de variation** | **Limites de réplication**
--- | --- | --- | --- | ---
8 processeurs virtuels (2 sockets * 4 cœurs \@ 2,5 GHz) | 16 Go | 300 Go | 500 Go ou moins | < 100 machines 
12 processeurs virtuels (2 sockets * 6 cœurs \@ 2,5 GHz) | 18 Go | 600 Go | 501 Go à 1 To | 100-150 machines.
16 processeurs virtuels (2 sockets * 8 cœurs \@ 2,5 GHz) | 32 G1 |  1 To | 1 To à 2 To | 151-200 machines.

### <a name="sizing-scale-out-process-servers"></a>Dimensionnement des serveurs de traitement de scale-out

S'il vous faut de déployer un serveur de traitement de scale-out, utilisez ce tableau pour déterminer la taille du serveur.

**Serveur de traitement** | **Espace libre pour la mise en cache des données** | **Taux de variation** | **Limites de réplication**
--- | --- | --- | --- 
4 processeurs virtuels (2 sockets * 2 cœurs \@ 2,5 GHz), 8 Go de mémoire | 300 Go | 250 Go ou moins | Jusqu'à 85 machines 
8 processeurs virtuels (2 sockets * 4 cœurs \@ 2,5 GHz), 12 Go de mémoire | 600 Go | 251 Go à 1 To    | 86-150 machines.
12 processeurs virtuels (2 sockets * 6 cœurs \@ 2,5 GHz), 24 Go de mémoire | 1 To | 1 à 2 To | 151-225 machines.

## <a name="throttle-upload-bandwidth"></a>Limitez la bande passante de chargement.

le trafic VMware qui est répliqué sur Azure passe par un serveur de processus spécifique. Vous pouvez limiter le débit de chargement en limitant la bande passante sur les machines qui opèrent en tant que serveurs de processus. Vous pouvez influencer la bande passante à l’aide de cette clé de Registre :

- La valeur de Registre HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows Azure Backup\Replication\UploadThreadsPerVM détermine le nombre de threads utilisés pour le transfert des données (réplication initiale ou différentielle) sur un disque. Une valeur plus élevée augmenter la bande passante réseau utilisée pour la réplication. La valeur par défaut est quatre. La valeur maximale est 32. Surveillez le trafic pour optimiser la valeur.
- En outre, vous pouvez limiter la bande passante sur l’ordinateur serveur de processus comme suit :

    1. Sur l’ordinateur serveur de processus, ouvrez le composant logiciel enfichable MMC Sauvegarde Azure. Il y a un raccourci sur le bureau ou dans le dossier C:\Program Files\Microsoft Azure Recovery Services Agent\bin. 
    2. Dans le composant logiciel enfichable, sélectionnez **Modifier les propriétés**.
    3. Dans **Limitation**, sélectionnez la case **Activer la limitation de la bande passante sur Internet pour les opérations de sauvegarde**. Définissez les limites pour les heures ouvrées et non ouvrées. Les plages valides vont de 512 Kbits/s à 1 023 Mbits/s.


## <a name="next-steps"></a>Étapes suivantes

Essayez la [migration basée sur agent](tutorial-migrate-vmware-agent.md) pour les [serveurs physiques](tutorial-migrate-vmware-agent.md) ou [VMware](tutorial-migrate-physical-virtual-machines.md).

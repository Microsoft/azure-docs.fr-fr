---
title: Configuration ANF des machines virtuelles SAP HANA Azure | Microsoft Docs
description: Recommandations en matière de stockage Azure NetApp Files pour SAP HANA.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP, Azure, ANF, HANA, Azure NetApp Files, capture instantanée
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 09/28/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 511801962d07e5fb99000b2fc19adce2489b46d3
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967480"
---
# <a name="nfs-v41-volumes-on-azure-netapp-files-for-sap-hana"></a>Volumes NFS v4.1 sur Azure NetApp Files pour SAP HANA

Azure NetApp Files fournit des partages NFS natifs qui peuvent être utilisés pour les volumes **/hana/shared**, **/hana/data** et **/hana/log**. L’utilisation de partages NFS basés sur ANF pour les volumes **/hana/data** et **/hana/log** nécessite l’utilisation du protocole NFS v4.1. Le protocole NFS v3 n’est pas pris en charge pour l’utilisation de volumes **/hana/data** et **/hana/log** quand les partages sont basés sur ANF. 


> [!IMPORTANT]
> L’utilisation du protocole NFS v3 implémenté sur Azure NetApp Files n’est **pas** prise en charge pour **/hana/data** et **/hana/log**. L’utilisation du NFS 4.1 est obligatoire pour les volumes **/hana/data** et **/hana/log** d’un point de vue fonctionnel. En revanche, pour le volume **/hana/shared**, les systèmes NFS v3 ou NFS v4.1 peuvent tous eux être utilisés d’un point de vue fonctionnel.

## <a name="important-considerations"></a>Points importants à prendre en compte

Lorsque vous envisagez d’utiliser Azure NetApp Files pour SAP Netweaver et SAP HANA, tenez compte des importantes considérations suivantes :

- La taille de pool de capacité minimale est de 4 Tio.  
- La taille de volume minimale est de 100 Gio
- Azure NetApp Files et toutes les machines virtuelles, où les volumes Azure NetApp Files sont montés, doivent être déployés dans le même réseau virtuel Azure ou dans des [réseaux virtuels appairés](../../../virtual-network/virtual-network-peering-overview.md) de la même région.
- Il est important que les machines virtuelles soient déployées à proximité du stockage Azure NetApp pour des raisons de faible latence.  
- Le réseau virtuel sélectionné doit avoir un sous-réseau délégué à Azure NetApp Files.
- Vérifiez que la latence entre le serveur de base de données et le volume ANF est mesurée et inférieure à 1 milliseconde.
- Le débit d’un volume NetApp Azure est une fonction du quota de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Lors du dimensionnement de volumes HANA Azure NetApp, vérifiez que le débit obtenu satisfait à la configuration système requise pour HANA.
- Essayez de « consolider » les volumes pour obtenir des performances plus élevées dans un volume plus grand ; par exemple, utilisez un volume pour /sapmnt, /usr/sap/trans, … dans la mesure du possible.  
- Azure NetApp Files propose une [stratégie d’exportation](../../../azure-netapp-files/azure-netapp-files-configure-export-policy.md) : vous pouvez contrôler les clients autorisés, le type d’accès (lecture et écriture, lecture seule, etc.). 
- La fonctionnalité Azure NetApp Files ne tient pas encore compte des zones. Actuellement, la fonctionnalité Azure NetApp Files n’est pas déployée dans toutes les zones de disponibilité d’une région Azure. Méfiez-vous de l’impact potentiel sur les temps de latence dans certaines régions Azure.   
- L’ID d’utilisateur pour <b>sid</b>adm et l’ID de groupe pour `sapsys` sur les machines virtuelles doivent correspondre à la configuration dans Azure NetApp Files. 

> [!IMPORTANT]
> Pour les charges de travail SAP HANA, une latence faible est critique. Collaborez avec votre représentant Microsoft pour vous assurer que les machines virtuelles et les volumes Azure NetApp Files soient déployés à proximité.  

> [!IMPORTANT]
> En cas d’incompatibilité entre l’ID d’utilisateur pour <b>sid</b>adm et l’ID de groupe pour `sapsys` entre la machine virtuelle et la configuration Azure NetApp, les autorisations pour les fichiers sur les volumes NetApp Azure, montés sur les machines virtuelles, s’afficheront en tant que `nobody`. Veillez à spécifier l’ID d’utilisateur correct pour <b>sid</b>adm et l’ID de groupe pour `sapsys`, lors de l’[intégration d’un nouveau système](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbRxjSlHBUxkJBjmARn57skvdUQlJaV0ZBOE1PUkhOVk40WjZZQVJXRzI2RC4u) à Azure NetApp Files.


## <a name="sizing-for-hana-database-on-azure-netapp-files"></a>Dimensionnement de la base de données HANA sur Azure NetApp Files

Le débit d’un volume NetApp Azure est une fonction de la taille de volume et du niveau de service, comme décrit dans [Niveau de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). 

Il est important de comprendre la relation entre les performance et la taille, et qu’il existe des limites physiques pour une interface logique (LIF) de la machine virtuelle de stockage (SVM).

Le tableau ci-dessous montre qu’il peut être judicieux de créer un grand volume « Standard » pour stocker les sauvegardes et qu’il n’est pas judicieux de créer un volume « Ultra » d’une taille supérieure à 12 To, car la capacité de bande passante physique d’une LIF unique serait dépassée. 

Le débit maximal pour une LIF et une session Linux unique est compris entre 1,2 et 1,4 Go/s. 

| Taille  | Débit Standard | Débit Premium | Débit Ultra |
| --- | --- | --- | --- |
| 1 To | 16 Mo/s | 64 Mo/s | 128 Mo/s |
| 2 To | 32 Mo/s | 128 Mo/s | 256 Mo/s |
| 4 To | 64 Mo/s | 256 Mo/s | 512 Mo/s |
| 10 To | 160 Mo/s | 640 Mo/s | 1280 Mo/s |
| 15 To | 240 Mo/s | 960 Mo/s | 1400 Mo/s |
| 20 To | 320 Mo/s | 1280 Mo/s | 1400 Mo/s |
| 40 To | 640 Mo/s | 1400 Mo/s | 1400 Mo/s |

Il est important de comprendre que les données sont écrites sur les mêmes disques SSD sur le back-end de stockage. Le quota de performances du pool de capacité a été créé afin de pouvoir gérer l’environnement.
Les KPI de stockage sont identiques pour toutes les tailles de base de données HANA. Dans presque tous les cas, cette hypothèse ne reflète pas la réalité et les attentes du client. La taille des systèmes HANA ne signifie pas nécessairement qu’un petit système nécessite un faible débit de stockage et qu’un grand système nécessite un débit de stockage élevé, mais en général nous pouvons nous attendre à des exigences de débit plus élevées pour les instances de base de données HANA plus volumineuses. En raison des règles de dimensionnement de SAP pour le matériel sous-jacent, ces instances HANA plus volumineuses fournissent également davantage de ressources de processeur et un parallélisme plus élevé dans des tâches telles que le chargement des données après un redémarrage d’instances. Par conséquent, les tailles de volume adoptées doivent répondre aux attentes et aux besoins des clients, et ne doivent pas être motivées uniquement par de pures exigences de capacité.

Lorsque vous concevez l’infrastructure pour SAP dans Azure, vous devez connaître certaines exigences de débit de stockage minimal (pour les systèmes de production) de SAP, qui se traduisent par les caractéristiques de débit minimal suivantes :

| Type de volume et type d’E/S | KPI minimal exigé par SAP | Niveau de service Premium | Niveau de service Ultra |
| --- | --- | --- | --- |
| Écriture du volume de journal | 250 Mo/s | 4 To | 2 To |
| Écriture du volume de données | 250 Mo/s | 4 To | 2 To |
| Lecture du volume de données | 400 Mo/s | 6,3 To | 3,2 To |

Étant donné que les trois KPI sont exigés, le volume **/hana/data** doit être dimensionné vers la plus grande capacité afin de respecter les exigences de lecture minimales.

Pour les systèmes HANA, qui ne nécessitent pas de bande passante élevée, les tailles de volume ANF peuvent être plus petites. Dans le cas où un système HANA nécessite un débit plus élevé, le volume peut être adapté en redimensionnant la capacité en ligne. Aucun KPI n’est défini pour les volumes de sauvegarde. Toutefois, le débit du volume de sauvegarde est essentiel pour un environnement performant. Les performances des volumes de journal et de données doivent être conçues afin de répondre aux attentes des clients.

> [!IMPORTANT]
> Indépendamment de la capacité que vous déployez sur un volume NFS unique, le débit est supposé stagner dans la plage de bande passante de 1,2 à 1,4 Go/s exploitée par un consommateur dans une machine virtuelle. Cela concerne l’architecture sous-jacente de l’offre ANF et aux limites de session Linux associées relatives à NFS. Les valeurs de performances et de débit décrites dans l’article [Résultats des tests de performances pour Azure NetApp Files](../../../azure-netapp-files/performance-benchmarks-linux.md) ont été effectuées sur un volume NFS partagé avec plusieurs machines virtuelles clientes et, par conséquent, avec plusieurs sessions. Ce scénario est différent du scénario que nous mesurons dans SAP. Nous y mesurons le débit d’une seule machine virtuelle par rapport à un volume NFS. Hébergé sur ANF.

Pour respecter les exigences de débit minimal SAP pour les données et le journal, et conformément aux instructions pour **/hana/shared**, les tailles recommandées ressemblent à ceci :

| Volume | Taille<br /> Niveau de stockage Premium | Taille<br /> Niveau de stockage Ultra | Protocole NFS pris en charge |
| --- | --- | --- |
| /hana/log/ | 4 Tio | 2 Tio | v4.1 |
| /hana/data | 6,3 Tio | 3,2 Tio | v4.1 |
| /hana/shared scale-up | Min (1 To, 1 x RAM)  | Min (1 To, 1 x RAM) | v3 ou v4.1 |
| /hana/shared scale-out | 1 x RAM de nœud Worker<br /> pour 4 nœuds Worker  | 1 x RAM de nœud Worker<br /> pour 4 nœuds Worker  | v3 ou v4.1 |
| /hana/logbackup | 3 x RAM  | 3 x RAM | v3 ou v4.1 |
| /hana/backup | 2 x RAM  | 2 x RAM | v3 ou v4.1 |

Pour tous les volumes, NFS v4.1 est fortement recommandé.

Les tailles des volumes de sauvegarde sont des estimations. Des exigences précises doivent être définies en fonction de la charge de travail et des processus d’exploitation. Pour les sauvegardes, vous pouvez consolider de nombreux volumes pour différentes instances de SAP HANA sur un (ou deux) volumes plus grands, qui peuvent avoir un niveau de service ANF inférieur.

> [!NOTE]
> Les recommandations relatives aux tailles Azure NetApp Files qui sont indiquées dans ce document ciblent la configuration minimale stipulée par SAP à ses fournisseurs d’infrastructure. Dans les déploiements de clients et les scénarios de charge de travail réels, cela peut ne pas suffire. Prenez ces recommandations en tant que point de départ et adaptez-les en fonction des exigences de votre charge de travail spécifique.  

Par conséquent, vous pouvez envisager de déployer un débit similaire pour les volumes ANF tels qu’ils sont déjà listés pour les disques de stockage Ultra. Prenez également en compte les tailles listées pour les volumes des différentes références SKU de machine virtuelle, comme déjà indiqué dans les tables de disques Ultra.

> [!TIP]
> Vous pouvez redimensionner les volumes Azure NetApp Files de manière dynamique, sans avoir à `unmount` les volumes, à arrêter les machines virtuelles ou à arrêter SAP HANA. Cela permet de répondre aux exigences à la fois attendues et imprévues de votre application en matière de débit.

La documentation sur le déploiement d’une configuration de scale-out de SAP HANA avec un nœud de secours à l’aide de volumes NFS v4.1 hébergés dans ANF est publiée dans [Scale-out de SAP HANA avec le nœud de secours sur les machines virtuelles Azure avec Azure NetApp Files sur SUSE Linux Enterprise Server](./sap-hana-scale-out-standby-netapp-files-suse.md).


## <a name="availability"></a>Disponibilité
Les mises à jour et mises à niveau du système ANF sont appliquées sans affecter l’environnement du client. Le [contrat SLA défini est de 99,99 %](https://azure.microsoft.com/support/legal/sla/netapp/).


## <a name="volumes-and-ip-addresses-and-capacity-pools"></a>Volumes et adresses IP et pools de capacité
Avec ANF, il est important de comprendre comment l’infrastructure sous-jacente est créée. Un pool de capacité est seulement une structure, ce qui simplifie la création d’un modèle de facturation pour ANF. Un pool de capacité n’a pas de relation physique avec l’infrastructure sous-jacente. Si vous créez un pool de capacité, seul un shell qui peut être facturé est créé, rien de plus. Lorsque vous créez un volume, la première SVM est créée sur un cluster de plusieurs systèmes NetApp. Une adresse IP unique est créée afin que cette SVM puisse accéder au volume. Si vous créez plusieurs volumes, tous les volumes sont distribués dans cette SVM sur ce cluster NetApp à plusieurs contrôleurs. Même si vous n’avez qu’une seule adresse IP, les données sont distribuées sur plusieurs contrôleurs. ANF a une logique qui distribue automatiquement les charges de travail des clients une fois que les volumes et/ou la capacité du stockage configuré atteignent un niveau prédéfini interne. Vous remarquerez peut-être de tels cas, car une nouvelle adresse IP est affectée pour accéder aux volumes.

##<a name="log-volume-and-log-backup-volume"></a>Volume du journal et volume de sauvegarde du journal
Le « volume du journal » ( **/hana/log**) est utilisé pour écrire le journal de phase de restauration par progression en ligne. Il existe donc des fichiers ouverts sur ce volume, et cela n’a pas de sens de créer une capture instantanée de ce volume. Les fichiers journaux de phase de restauration par progression en ligne sont archivés ou sauvegardés sur le volume de sauvegarde du journal une fois que le fichier journal de phase de restauration par progression en ligne est plein ou qu’une sauvegarde du journal de phase de restauration par progression est exécutée. Pour garantir des performances de sauvegarde raisonnables, le volume de sauvegarde du journal nécessite un bon débit. Pour optimiser les coûts de stockage, il peut être judicieux de consolider le volume de sauvegarde du journal de plusieurs instances HANA, afin que plusieurs instances HANA tirent parti du même volume et écrivent leurs sauvegardes dans des répertoires différents. Avec une telle consolidation, vous pouvez obtenir un débit plus élevé, car vous devez augmenter la taille du volume. 

Il en va de même pour le volume sur lequel vous écrivez des sauvegardes complètes de base de données HANA.  
 

## <a name="backup"></a>Sauvegarde
En plus des sauvegardes en streaming et de la sauvegarde de bases de données SAP HANA par le service Sauvegarde Azure, comme décrit dans l’article [Guide de sauvegarde pour SAP HANA sur Machines Virtuelles Azure](./sap-hana-backup-guide.md), Azure NetApp Files offre la possibilité d’effectuer des sauvegardes de captures instantanées basées sur le stockage. 

SAP HANA prend en charge :

- Les sauvegardes de captures instantanées basées sur le stockage à partir de SAP HANA 1.0 SPS7.
- Les sauvegardes de captures instantanées basées sur le stockage pour les environnements HANA MDC (Multi Database Container) à partir de SAP HANA 2.0 SPS4.


La création de sauvegardes de captures instantanées basées sur le stockage est une procédure simple en quatre étapes. 
1. Création d’une capture instantanée de base de données HANA (interne) : activité qui doit être effectuée par vous-même ou par des outils. 
1. SAP HANA écrit les données dans les fichiers de données pour créer un état cohérent sur le stockage : HANA effectue cette étape suite à la création d’une capture instantanée HANA.
1. Création d’une capture instantanée sur le volume **/hana/data** sur le stockage : étape qui doit être effectuée par vous-même ou par des outils. Il n’est pas nécessaire d’effectuer de capture instantanée sur le volume **/hana/log**.
1. Suppression de la capture instantanée de base de données HANA (interne) et reprise des opérations normales : étape qui doit être effectuée par vous-même ou par des outils.

> [!WARNING]
> Ne pas effectuer la dernière étape aura un impact sérieux sur la demande en mémoire de SAP HANA et peut entraîner son arrêt.

```
BACKUP DATA FOR FULL SYSTEM CREATE SNAPSHOT COMMENT 'SNAPSHOT-2019-03-18:11:00';
```

![Sauvegarde de capture instantanée ANF pour SAP HANA](media/hana-vm-operations-netapp/storage-snapshot-scenario.png)

```
az netappfiles snapshot create -g mygroup --account-name myaccname --pool-name mypoolname --volume-name myvolname --name mysnapname 
```

![Sauvegarde de capture instantanée ANF pour SAP HANA Partie2](media/hana-vm-operations-netapp/storage-snapshot.png)

```
BACKUP DATA FOR FULL SYSTEM CLOSE SNAPSHOT BACKUP_ID 47110815 SUCCESSFUL SNAPSHOT-2020-08-18:11:00';
```

Cette procédure de sauvegarde de capture instantanée peut être gérée de plusieurs façons et à l’aide de différents outils, par exemple avec le script Python « ntaphana_azure.py » disponible sur GitHub [https://github.com/netapp/ntaphana](https://github.com/netapp/ntaphana). Il s’agit d’un exemple de code, fourni « tel quel » sans aucune maintenance ni prise en charge.



> [!CAUTION]
> Une capture instantanée en elle-même n’est pas une sauvegarde protégée, car elle se trouve sur le même stockage physique que le volume dont vous venez d’effectuer une capture instantanée. Il est obligatoire de « protéger » au moins une capture instantanée par jour à un autre emplacement. Cela peut être fait dans le même environnement, dans une région Azure distante ou dans le stockage d’objets blob Azure.


Pour les utilisateurs de produits de sauvegarde Commvault, la deuxième option est Commvault IntelliSnap V.11.21 et versions ultérieures. Cette version de Commvault (ou les versions ultérieures) offre une prise en charge d’Azure NetApp Files. L’article [Commvault IntelliSnap 11.21](https://documentation.commvault.com/11.21/essential/116350_getting_started_with_backup_and_restore_operations_for_azure_netapp_file_services_smb_shares_and_nfs_shares.html) fournit plus d’informations.


### <a name="back-up-the-snapshot-using-azure-blob-storage"></a>Sauvegarder la capture instantanée à l’aide du stockage d’objets blob Azure
La sauvegarde dans le stockage d’objets blob Azure est une méthode économique et rapide pour enregistrer les sauvegardes de captures instantanées de stockage de base de données HANA basées sur ANF. Pour enregistrer les captures instantanées dans le stockage d’objets blob Azure, nous vous recommandons l’outil azcopy. Téléchargez la dernière version de cet outil et installez-la par exemple dans le répertoire bin où est installé le script Python de GitHub.
Téléchargez la dernière version de l’outil azcopy :

```
root # wget -O azcopy_v10.tar.gz https://aka.ms/downloadazcopy-v10-linux && tar -xf azcopy_v10.tar.gz --strip-components=1
Saving to: ‘azcopy_v10.tar.gz’
```

La fonctionnalité la plus avancée est l’option SYNC. Si vous utilisez l’option SYNC, azcopy maintient la synchronisation entre les répertoires source et de destination. L’utilisation du paramètre **--delete-destination** est importante. Sans ce paramètre, azcopy ne supprime pas les fichiers sur le site de destination, et l’utilisation de l’espace côté destination augmente. Créez un conteneur d’objets blob de blocs dans votre compte Stockage Azure. Ensuite, créez la clé SAS pour le conteneur d’objets blob et synchronisez le dossier de captures instantanées avec le conteneur d’objets blob Azure.

Par exemple, si une capture instantanée quotidienne doit être synchronisée avec le conteneur d’objets blob Azure pour protéger les données, et que seule cette capture instantanée doit être conservée, vous pouvez utiliser la commande ci-dessous.

```
root # > azcopy sync '/hana/data/SID/mnt00001/.snapshot' 'https://azacsnaptmytestblob01.blob.core.windows.net/abc?sv=2021-02-02&ss=bfqt&srt=sco&sp=rwdlacup&se=2021-02-04T08:25:26Z&st=2021-02-04T00:25:26Z&spr=https&sig=abcdefghijklmnopqrstuvwxyz' --recursive=true --delete-destination=true
```

## <a name="next-steps"></a>Étapes suivantes
Lisez l’article suivant :

- [Haute disponibilité de SAP HANA pour les machines virtuelles Azure](./sap-hana-availability-overview.md)
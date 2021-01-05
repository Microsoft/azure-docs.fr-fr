---
title: Tests de performances recommandés - Azure NetApp Files
description: Découvrez des recommandations sur les tests de référence pour les métriques et les performances de volume avec Azure NetApp Files.
author: b-juche
ms.author: b-juche
ms.service: azure-netapp-files
ms.workload: storage
ms.topic: conceptual
ms.date: 08/07/2019
ms.openlocfilehash: b41575208d0793a33864556ec4313ae1a71f2db5
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96184465"
---
# <a name="performance-benchmark-test-recommendations-for-azure-netapp-files"></a>Recommandations sur les tests de performances pour Azure NetApp Files

Cet article fournit des informations sur les tests de référence pour les mesures et les performances de volume avec Azure NetApp Files.

## <a name="overview"></a>Vue d’ensemble

Pour comprendre les caractéristiques en matière de performances d’un volume Azure NetApp Files, vous pouvez utiliser l’outil open source [FIO](https://github.com/axboe/fio), qui exécute une série de tests pour simuler diverses charges de travail. L’outil FIO peut être installé sur des systèmes d’exploitation Windows et Linux.  Il s’agit d’un excellent outil, qui propose un aperçu rapide des IOPS et du débit d’un volume.

### <a name="vm-instance-sizing"></a>Dimensionnement de l’instance de machine virtuelle

Pour de meilleurs résultats, vérifiez que vous utilisez une instance de machine virtuelle présentant la taille adéquate pour ces tests. Les exemples suivants utilisent une instance Standard_D32s_v3. Pour en savoir plus sur les tailles d’instance de machine virtuelle, voir [Tailles des machines virtuelles Windows dans Azure](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-network%2ftoc.json) pour les machines virtuelles Windows, et [Tailles des machines virtuelles Linux dans Azure](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) pour les machines virtuelles Linux.

### <a name="azure-netapp-files-volume-sizing"></a>Dimensionnement des volumes Azure NetApp Files

Pour obtenir le niveau de performances attendu, veillez à choisir le bon niveau de service, ainsi que des quotas de volume de taille adéquate. Pour en savoir plus, voir [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md).

### <a name="virtual-network-vnet-recommendations"></a>Suggestions relatives au réseau virtuel

Vous devez effectuer le test de référence dans le même réseau virtuel que celui d’Azure NetApp Files. L’exemple ci-dessous présente nos suggestions :

![Suggestions relatives au réseau virtuel](../media/azure-netapp-files/azure-netapp-files-benchmark-testing-vnet.png)

## <a name="installation-of-fio"></a>Installation de l’outil FIO

Cet outil est disponible au format binaire pour Linux et pour Windows. Consultez la section relative aux packages binaires de [FIO](https://github.com/axboe/fio) pour installer l’outil sur la plateforme de votre choix.

## <a name="fio-examples-for-iops"></a>Exemples d’IOPS pour FIO 

Dans cette section, ces exemples utilisent la configuration suivante :
* Taille des instances de machine virtuelle : D32s_v3
* Taille et niveau de service du pool de capacité : Premium/50 TiO
* Taille de quota du volume : 48 Tio

Les exemples suivants montrent le nombre de lectures et d’écritures aléatoires de l’outil FIO.

### <a name="fio-8k-block-size-100-random-reads"></a>FIO : taille de bloc de 8 ko, 100 % de lectures aléatoires

`fio --name=8krandomreads --rw=randread --direct=1 --ioengine=libaio --bs=8k --numjobs=4 --iodepth=128 --size=4G --runtime=600 --group_reporting`

### <a name="output-68k-read-iops-displayed"></a>Sortie : Affichage de 68 000 IOPS de lectures

`Starting 4 processes`  
`Jobs: 4 (f=4): [r(4)][84.4%][r=537MiB/s,w=0KiB/s][r=68.8k,w=0 IOPS][eta 00m:05s]`

### <a name="fio-8k-block-size-100-random-writes"></a>FIO : taille de bloc de 8 ko, 100 % d’écritures aléatoires

`fio --name=8krandomwrites --rw=randwrite --direct=1 --ioengine=libaio --bs=8k --numjobs=4 --iodepth=128  --size=4G --runtime=600 --group_reporting`

### <a name="output-73k-write-iops-displayed"></a>Sortie : Affichage de 73 000 IOPS d’écritures

`Starting 4 processes`  
`Jobs: 4 (f=4): [w(4)][26.7%][r=0KiB/s,w=571MiB/s][r=0,w=73.0k IOPS][eta 00m:22s]`

## <a name="fio-examples-for-bandwidth"></a>Exemples de bande passante pour FIO

Les exemples de cette section montrent le nombre de lectures et d’écritures séquentielles de l’outil FIO.

### <a name="fio-64k-block-size-100-sequential-reads"></a>FIO : taille de bloc de 64 ko, 100 % de lectures séquentielles

`fio --name=64kseqreads --rw=read --direct=1 --ioengine=libaio --bs=64k --numjobs=4 --iodepth=128  --size=4G --runtime=600 --group_reporting`

### <a name="output-118-gbits-throughput-displayed"></a>Sortie : débit de sortie de 11,8 Gbit/s (affiché)

`Starting 4 processes`  
`Jobs: 4 (f=4): [R(4)][40.0%][r=1313MiB/s,w=0KiB/s][r=21.0k,w=0 IOPS][eta 00m:09s]`

### <a name="fio-64k-block-size-100-sequential-writes"></a>FIO : taille de bloc de 64 ko, 100 % d’écritures séquentielles

`fio --name=64kseqwrites --rw=write --direct=1 --ioengine=libaio --bs=64k --numjobs=4 --iodepth=128  --size=4G --runtime=600 --group_reporting`

### <a name="output-122-gbits-throughput-displayed"></a>Sortie : débit de sortie de 12,2 Gbit/s (affiché)

`Starting 4 processes`  
`Jobs: 4 (f=4): [W(4)][85.7%][r=0KiB/s,w=1356MiB/s][r=0,w=21.7k IOPS][eta 00m:02s]`

## <a name="volume-metrics"></a>Mesures du volume

Les données de performances de Microsoft Azure NetApp Files sont disponibles par l’intermédiaire des compteurs Azure Monitor. Ces compteurs sont disponibles via les requêtes GET de l’API REST et sur le portail Microsoft Azure. 

Vous pouvez afficher les données d’historique pour les informations suivantes :
* Latence de lecture moyenne 
* Latence d’écriture moyenne 
* IOPS de lectures (en moyenne)
* IOPS d’écritures (en moyenne)
* Taille logique du volume (en moyenne)
* Taille des clichés instantanés de volume (en moyenne)

### <a name="using-azure-monitor"></a>Utilisation d’Azure Monitor 

Vous pouvez accéder aux compteurs Azure NetApp Files volume par volume à partir de la page Mesures, comme indiqué ci-dessous :

![Mesures Azure Monitor](../media/azure-netapp-files/azure-netapp-files-benchmark-monitor-metrics.png)

Dans Azure Monitor, vous pouvez également créer un tableau de bord pour Azure NetApp Files en accédant à la page Mesures, en filtrant l’affichage sur « NetApp » et en spécifiant les compteurs de volume qui vous intéressent : 

![Tableau de bord Azure Monitor](../media/azure-netapp-files/azure-netapp-files-benchmark-monitor-dashboard.png)

### <a name="azure-monitor-api-access"></a>Accès à l’API Azure Monitor

Vous pouvez accéder aux compteurs Azure NetApp Files via des appels à l’API REST. Voir [Métriques prises en charge avec Azure Monitor : Microsoft.NetApp/netAppAccounts/capacityPools/Volumes](../azure-monitor/platform/metrics-supported.md#microsoftnetappnetappaccountscapacitypoolsvolumes) pour accéder aux compteurs portant sur les volumes et les pools de capacité.

L’exemple suivant montre une adresse URL GET permettant d’afficher la taille des volumes logiques :

`#get ANF volume usage`  
`curl -X GET -H "Authorization: Bearer TOKENGOESHERE" -H "Content-Type: application/json" https://management.azure.com/subscriptions/SUBIDGOESHERE/resourceGroups/RESOURCEGROUPGOESHERE/providers/Microsoft.NetApp/netAppAccounts/ANFACCOUNTGOESHERE/capacityPools/ANFPOOLGOESHERE/Volumes/ANFVOLUMEGOESHERE/providers/microsoft.insights/metrics?api-version=2018-01-01&metricnames=VolumeLogicalSize`


## <a name="next-steps"></a>Étapes suivantes

- [Niveaux de service pour Azure NetApp Files](azure-netapp-files-service-levels.md)
- [Test d’évaluation des performances pour Linux](performance-benchmarks-linux.md)
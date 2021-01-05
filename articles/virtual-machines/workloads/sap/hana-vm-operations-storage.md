---
title: Configurations du stockage des machines virtuelles SAP HANA Azure | Microsoft Docs
description: Recommandations de stockage pour les machines virtuelles avec SAP HANA déployé.
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: SAP, Azure HANA, stockage Ultra Disk, stockage Premium
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 11/26/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8c4aa608e892867daaf954284a9dfce997a9ae1f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484275"
---
# <a name="sap-hana-azure-virtual-machine-storage-configurations"></a>Configurations du stockage des machines virtuelles SAP HANA Azure

Azure fournit des types de stockage différents, adaptés aux machines virtuelles Azure exécutant SAP HANA. **Types de stockage Azure certifiés SAP HANA** pouvant être pris en compte pour la liste des déploiements SAP HANA, tels que : 

- SSD Azure Premium ou stockage Premium 
- [Disque Ultra](../../disks-enable-ultra-ssd.md)
- [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) 

Pour en savoir plus sur ces types de disques, consultez les articles [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md) et [Sélectionner un type de disque](../../disks-types.md)

Azure offre deux méthodes de déploiement avec les disques durs virtuels sur le stockage Azure Standard et Premium. Nous vous recommandons de tirer parti du [disque géré Azure](https://azure.microsoft.com/services/managed-disks/) pour les déploiements de stockage par blocs Azure. 

Pour obtenir une liste des types de stockage et les contrats SLA associés pour les débits d’IOPS et de stockage, passez en revue la [documentation Azure sur les disques managés](https://azure.microsoft.com/pricing/details/managed-disks/).

> [!IMPORTANT]
> Indépendamment du type de stockage Azure choisi, le système de fichiers utilisé sur ce stockage doit être pris en charge par SAP pour le système d’exploitation et le SGBD spécifiques. La [Note de support SAP n°2972496](https://launchpad.support.sap.com/#/notes/2972496) répertorie les systèmes de fichiers pris en charge pour les différents systèmes d’exploitation et bases de données, dont SAP HANA. Cela s’applique à tous les volumes SAP HANA qui pourraient accéder à des fins de lecture et d’écriture pour n’importe quelle tâche. En particulier, en utilisant NFS sur Azure pour SAP HANA, des restrictions supplémentaires des versions NFS s’appliquent, comme indiqué plus loin dans cet article 


Les conditions certifiée SAP HANA minimales pour les différents types de stockages sont les suivantes : 

- Le stockage Premium Azure - **/hana/log** doit être pris en charge par l’[Accélérateur d'écriture](../../how-to-enable-write-accelerator.md) Azure. Le volume **/hana/data** peut être placé sur un stockage SSD sans Accélérateur d’écriture Azure ou sur un disque Ultra
- Disque Ultra Azure au moins pour le volume **/hana/log**. Le volume **/hana/data** peut être placé sur un stockage Premium sans Accélérateur d’écriture Azure ou sur disque Ultra pour accélérer les temps de redémarrage
- Volumes **NFS v4.1** par-dessus Azure NetApp Files pour **/hana/log et /hana/data**. Le volume de/hana/shared peut utiliser le protocole NFS v3 ou NFS v4.1

Certains types de stockage peuvent être combinés. Par exemple, il est possible de placer **/hana/data** sur un stockage Premium et **/hana/log** sur un disque de stockage Ultra afin d’obtenir la faible latence requise. Si vous utilisez un volume basé sur ANF pour **/hana/data**, le volume **/hana/log** doit également être basé sur NFS en plus d’ANF. L’utilisation de NFS en plus d’ANF pour l’un des volumes (comme /hana/data) et du stockage Premium Azure ou d’un disque Ultra pour l’autre volume (comme **/hana/log**) n’est **pas prise en charge**.

Localement, vous aviez rarement besoin de vous occuper des sous-systèmes d’E/S et de leurs fonctionnalités. En effet, le fournisseur d’appliance devait s’assurer que les exigences de stockage minimales étaient remplies pour SAP HANA. Comme vous générez l’infrastructure Azure vous-même, vous devez connaître certaines de ces conditions requises par SAP. Voici quelques-unes des caractéristiques de débit minimales recommandées par SAP :

- Lecture/écriture sur **/hana/log** à un débit de 250 Mo/s pour des tailles d’E/S de 1 Mo
- Activité de lecture à un débit de 400 Mo/s au minimum pour **/hana/data** pour des tailles d’E/S de 16 Mo et 64 Mo
- Activité d’écriture à un débit de 250 Mo/s au minimum pour **/hana/data** avec des tailles d’E/S de 16 Mo et 64 Mo

Étant donné qu’une latence de stockage faible est critique pour les systèmes SGBD, même pour ceux comme SAP HANA, conservez les données en mémoire. Le point critique dans le stockage concerne généralement les écritures du journal de transaction sur les systèmes SGBD. D’autres opérations peuvent également être critiques, par exemple, l’écriture des points de sauvegarde ou le chargement des données en mémoire après une récupération sur incident. C’est pourquoi vous devez **obligatoirement** utiliser un stockage Premium Azure, un disque Ultra ou ANF pour les volumes **/hana/data** et **/hana/log**. 


Voici une liste de certains principes de directeur quant au choix de la configuration de stockage pour HANA :

- Choisissez le type de stockage en vous référant aux articles [Types de stockage Azure pour une charge de travail SAP](./planning-guide-storage.md) et [Sélectionner un type de disque](../../disks-types.md)
- Gardez à l’esprit le débit d’E/S global et les limites d’IOPS de la machine virtuelle lors du dimensionnement ou du choix d’une machine virtuelle. Le débit de stockage de machine virtuelle global est décrit dans l’article [Tailles de machine virtuelle à mémoire optimisée](../../sizes-memory.md)
- Lorsque vous choisissez la configuration du stockage, essayez de rester en dessous du débit global de la machine virtuelle avec votre configuration de volume **/hana/data**. L’écriture de points d’enregistrement, SAP HANA peut générer des opérations d’E/S agressives. Vous pouvez facilement atteindre les limites de débit de votre volume **/hana/data** lors de l’écriture d’un point d’enregistrement. Si votre ou vos disques générant le volume **/hana/data** affichent un débit supérieur à celui autorisé par votre machine virtuelle, vous pouvez rencontrer des situations où le débit utilisé par l’écriture du point d’enregistrement interfère avec les demandes de débit des écritures de journal de rétablissement. Cette situation peut avoir un impact sur le débit de l’application
- Si vous utilisez le stockage Premium Azure, la configuration la moins coûteuse consiste à utiliser des gestionnaires de volumes logiques pour créer des jeux de bandes afin de générer les volumes **/hana/data** et **/hana/log**

> [!IMPORTANT]
> Les suggestions relatives aux configurations de stockage doivent être utilisées comme point de départ. Lors de l’exécution de la charge de travail et de l’analyse des modèles d’utilisation du stockage, vous pouvez constater que vous n’utilisez pas la bande passante de stockage ou les IOPS fournies. Dans ce cas, vous pouvez réduire le stockage. En revanche, votre charge de travail peut nécessiter plus de débit de stockage que ce que ces configurations vous recommandent. Par conséquent, vous devrez peut-être augmenter la capacité, les IOPS ou le débit. Sur la question concernant la tension entre la capacité de stockage requise, la latence de stockage nécessaire, le débit de stockage et les IOPS requis et la configuration la moins onéreuse, Azure propose suffisamment de types de stockage différents avec des fonctionnalités différentes et divers tarifs pour trouver le bon compromis pour vous et votre charge de travail HANA.

## <a name="linux-io-scheduler-mode"></a>Mode Planificateur d’E-S Linux
Linux dispose de plusieurs modes de planification d’E-S. Les fournisseurs Linux et SAP recommandent couramment de remplacer le mode planificateur d’E/S **mq-deadline** ou **kyber** par **noop** (absence de plusieurs files d’attente) ou **none** (plusieurs files d’attente) pour les volumes de disque. Pour plus de détails à ce sujet, consultez la [Note SAP 1984787](https://launchpad.support.sap.com/#/notes/1984787). 


## <a name="solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines"></a>Solutions avec stockage Premium et Accélérateur d’écriture Azure pour les machines virtuelles Azure de la série M
L’Accélérateur d’écriture Azure est une fonctionnalité qui est fournie uniquement pour les machines virtuelles Azure de la série M. Comme son nom l’indique, cette fonctionnalité vise à améliorer la latence d’E/S des écritures dans le stockage Azure Premium. Pour SAP HANA, l’Accélérateur des écritures doit être utilisé exclusivement sur le volume **/hana/log**. Par conséquent, **/hana/data** et **/hana/log** sont des volumes distincts avec l’Accélérateur d’écriture Azure prenant en charge le volume **/hana/log** uniquement. 

> [!IMPORTANT]
> En cas d’utilisation du stockage Premium Azure, l’utilisation de l’[Accélérateur d’écriture](../../how-to-enable-write-accelerator.md) Azure ou du volume **/hana/log** est obligatoire. L’Accélérateur d’écriture est disponible pour le stockage Premium et les machines virtuelles de série M et Mv2 uniquement. L’Accélérateur d’écriture ne fonctionne pas en association avec d’autres familles de machines virtuelles Azure, notamment Esv3 ou Edsv4.

Les suggestions de mise en cache pour disques Azure Premium indiquées plus bas supposent que SAP HANA présente les caractéristiques d’E/S suivantes :

- Il n’y a pratiquement aucune charge de travail de lecture sur les fichiers de données HANA. Les exceptions sont les E/S volumineuses après le redémarrage de l’instance HANA ou quand les données sont chargées dans HANA. Les sauvegardes de base de données HANA peuvent constituer un autre cas d’E/S en lecture plus volumineuses sur des fichiers de données. Ainsi, la mise en cache en lecture n’est pas pertinente car, dans la plupart des cas, tous les volumes de fichiers de données doivent être lus complètement. 
- Les points de sauvegarde HANA et la récupération sur incident HANA donnent lieu à des écritures en rafales dans les fichiers de données. L’écriture de point de sauvegarde est asynchrone et n’accapare pas de transactions utilisateur. L’écriture de données au cours d’une récupération sur incident est critique pour les performances et la réactivité du système. Toutefois, une récupération sur incident doit intervenir dans des situations plutôt exceptionnelles.
- Il n’y a pratiquement pas de lectures à partir des fichiers de restauration par progression HANA. Les exceptions sont les grandes E/S pendant les sauvegardes de fichier journal, une récupération sur incident ou le redémarrage d’une instance HANA.  
- La charge principale par rapport au fichier journal de restauration par progression SAP HANA est constituée d’écritures. Selon la nature de la charge de travail, la taille des E/S peut aller de 4 Ko à 1 Mo, voire plus. La latence des écritures par rapport au journal de restauration par progression SAP HANA est critique pour les performances.
- Toutes les écritures doivent être rendues persistantes sur le disque de manière fiable.

**Recommandation : pour observer ces modèles d’E/S définis par SAP HANA, vous devez paramétrer la mise en cache pour les différents volumes à l’aide du stockage Premium Azure comme suit :**

- **/hana/data** - aucune mise en cache ou mise en cache en lecture
- **/hana/log** : aucune mise en cache, à l’exception des machines virtuelles des séries M et Mv2, où Accélérateur d’écriture Azure doit être activé 
- **/hana/shared** : mise en cache en lecture
- **Disque du système d’exploitation**  --ne pas modifier la mise en cache par défaut définie par Azure au moment de la création de la machine virtuelle


Si vous utilisez LVM ou mdadm pour créer des jeux de bandes sur plusieurs disques Azure Premium, vous devez définir des tailles de bande. Ces tailles sont différentes pour **/hana/data** et **/hana/log**. **Recommandation : utilisez les tailles de bande recommandées suivantes :**

- 256 Ko pour **/hana/data**
- 64 Ko pour **/hana/log**

> [!NOTE]
> La taille de bande pour **/hana/data** a été modifiée par rapport aux recommandations antérieures demandant 64 Ko ou 128 Ko à 256 Ko, en fonction des expériences client avec les versions plus récentes de Linux. La taille de 256 Ko offre des performances légèrement meilleures. Nous avons également modifié la recommandation pour les tailles de bande de **/hana/log** de 32 ko à 64 ko afin d’obtenir un débit suffisant avec des tailles d’E/S plus importantes.

> [!NOTE]
> Vous n’avez pas besoin de configurer un niveau de redondance avec des volumes RAID, car le stockage par blocs Azure conserve trois images d’un disque dur virtuel. L’utilisation d’un jeu de bandes avec des disques Azure Premium sert uniquement à configurer des volumes fournissant un débit d’IOPS et/ou d’E/S suffisant.

L’augmentation du nombre de disques durs virtuels Azure sous un jeu de bandes a pour effet d’augmenter les débits d’IOPS et de stockage. Ainsi, si vous placez un jeu de bandes sur trois disques P30 de stockage Azure Premium, vous obtenez en principe trois fois plus de débit d’IOPS et trois fois plus de débit de stockage qu’avec un seul disque P30 de stockage Azure Premium.

> [!IMPORTANT]
> Si vous utilisez LVM ou mdadm comme gestionnaire de volume pour créer des jeux de bandes sur plusieurs disques Azure Premium, les trois volumes SAP HANA FileSystems /data, /log et /shared ne doivent pas être placés dans un groupe de volumes par défaut ou racine. Il est vivement recommandé de suivre les instructions des fournisseurs Linux, qui consistent généralement à créer des groupes de volumes individuels pour /données, //journal et /partagé.


### <a name="azure-burst-functionality-for-premium-storage"></a>Fonctionnalité en rafale Azure pour le stockage Premium
Pour les disques de stockage Premium Azure plus petits ou égaux à 512 Gio en capacité, une fonctionnalité en rafale est proposée. Le fonctionnement exact du mode rafale des disques est décrit dans l’article [Mode rafale des disques](../../disk-bursting.md). En lisant cet article, vous comprendrez le concept d’IOPS et de débit dans les cas où votre charge de travail d’E/S se trouve au-dessous des valeurs d’IOPS et de débit nominales des disques (pour plus d’informations sur le débit nominal, consultez [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/)). Vous allez cumuler le delta d’IOPS et de débit entre votre utilisation actuelle et les valeurs nominales du disque. Les rafales sont limitées à un maximum de 30 minutes.

Idéalement, cette fonctionnalité en rafales sera planifiée pour des volumes ou disques contenant des fichiers de données pour les différents SGBD. La charge de travail d’E/S attendue pour ces volumes, en particulier avec les systèmes de petite ou moyenne taille, devrait ressembler à ceci :

- Une charge de travail de lecture faible à modérée, car les données sont idéalement mises en cache en mémoire, ou comme dans le cas de HANA, doivent être entièrement mises en mémoire
- Rafales d’écritures déclenchées par des points de contrôle de base de données ou des points d’enregistrement émis régulièrement
- Charge de travail de sauvegarde qui lit un flux continu dans les cas où les sauvegardes ne sont pas exécutées via des captures instantanées de stockage
- Pour SAP HANA, charger les données en mémoire après le redémarrage d’une instance

En particulier sur les systèmes SGBD plus petits dans lesquels votre charge de travail gère uniquement quelques centaines de transactions par seconde, une telle fonctionnalité en rafales peut également être utile pour les disques ou volumes qui stockent la transaction ou le journal de rétablissement. La charge de travail attendue sur un disque ou plusieurs volumes ressemble à ceci :

- Des écritures régulières sur le disque qui dépendent de la charge de travail et de la nature de la charge de travail, étant donné que chaque validation émise par l’application est susceptible de déclencher une opération d’E/S
- Une charge de travail plus élevée en débit pour les tâches opérationnelles, par exemple la création ou la reconstruction d’index
- Des rafales d’écritures lorsque vous effectuez des sauvegardes du journal des transactions ou du journal de rétablissement


### <a name="production-recommended-storage-solution-based-on-azure-premium-storage"></a>Une solution de stockage recommandée pour la production basée sur le stockage Premium Azure

> [!IMPORTANT]
> La certification SAP HANA des machines virtuelles Azure de la série M est valable exclusivement avec l’Accélérateur des écritures Azure sur le volume **/hana/log**. Par conséquent, dans les scénarios de production, les déploiements SAP HANA sur des machines virtuelles Azure de la série M doivent être configurés avec l’Accélérateur des écritures Azure sur le volume **/hana/log**.  

> [!NOTE]
> Dans les scénarios impliquant le stockage Azure Premium, nous implémentons des fonctionnalités en rafales dans la configuration. À mesure que vous utilisez des outils de test de stockage de toute forme, gardez à l’esprit le mode de fonctionnement de [la fonctionnalité en rafales de disque Azure Premium](../../disk-bursting.md). En exécutant les tests de stockage fournis par l’outil SAP HWCCT ou HCMT, nous ne nous attendons pas à ce que tous les tests répondent aux critères, car certains des tests dépasseront les crédits de rafales que vous pouvez cumuler. En particulier lorsque tous les tests s’exécutent séquentiellement sans interruption.


> [!NOTE]
> Pour les scénarios de production, vérifiez si un type de machine virtuelle spécifique est pris en charge pour SAP HANA dans la [documentation SAP pour IaaS](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html).

**Recommandation : Les configurations recommandées avec le stockage Premium Azure pour les scénarios de production ressemblent à ce qui suit :**

Configuration pour le volume SAP **/hana/data** :

| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | /hana/data | Débit approvisionné | Débit maximum de rafale | E/S par seconde | IOPS en rafale |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 Gio | 500 Mo/s | 4 x P6 | 200 Mbits/s | 680 Mbits/s | 960 | 14 000 |
| M32ls | 256 Gio | 500 Mo/s | 4 x P6 | 200 Mbits/s | 680 Mbits/s | 960 | 14 000 |
| M64ls | 512 Go | 1 000 Mbits/s | 4 x P10 | 400 Mbits/s | 680 Mbits/s | 2 000 | 14 000 |
| M64s | 1 000 Gio | 1 000 Mbits/s | 4 x P15 | 500 Mo/s | 680 Mbits/s | 4 400 | 14 000 |
| M64ms | 1 750 Gio | 1 000 Mbits/s | 4 x P20 | 600 Mbits/s | 680 Mbits/s | 9 200 | 14 000 |  
| M128s | 2 000 Gio | 2 000 Mbits/s | 4 x P20 | 600 Mbits/s | 680 Mbits/s | 9 200| 14 000 | 
| M128ms | 3,800 Gio | 2 000 Mbits/s | 4 x P30 | 800 Mo/s | pas de rafale | 20 000 | pas de rafale | 
| M208s_v2 | 2 850 Gio | 1 000 Mbits/s | 4 x P30 | 800 Mo/s | pas de rafale | 20 000| pas de rafale | 
| M208ms_v2 | 5 700 Gio | 1 000 Mbits/s | 4 x P40 | 1 000 Mbits/s | pas de rafale | 30,000 | pas de rafale |
| M416s_v2 | 5 700 Gio | 2 000 Mbits/s | 4 x P40 | 1 000 Mbits/s | pas de rafale | 30,000 | pas de rafale |
| M416ms_v2 | 11 400 Gio | 2 000 Mbits/s | 4 x P50 | 2 000 Mbits/s | pas de rafale | 30,000 | pas de rafale |


Pour le volume **/hana/log**. la configuration ressemblerait à ceci :

| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | Volume **/hana/log** | Débit approvisionné | Débit maximum de rafale | E/S par seconde | IOPS en rafale |
| --- | --- | --- | --- | --- | --- | --- | 
| M32ts | 192 Gio | 500 Mo/s | 3 x P10 | 300 Mbits/s | 510 Mbits/s | 1 500 | 10 500 | 
| M32ls | 256 Gio | 500 Mo/s | 3 x P10 | 300 Mbits/s | 510 Mbits/s | 1 500 | 10 500 | 
| M64ls | 512 Go | 1 000 Mbits/s | 3 x P10 | 300 Mbits/s | 510 Mbits/s | 1 500 | 10 500 | 
| M64s | 1 000 Gio | 1 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 | 
| M64ms | 1 750 Gio | 1 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 |  
| M128s | 2 000 Gio | 2 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500|  
| M128ms | 3,800 Gio | 2 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 | 
| M208s_v2 | 2 850 Gio | 1 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 |  
| M208ms_v2 | 5 700 Gio | 1 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 |  
| M416s_v2 | 5 700 Gio | 2 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 |  
| M416ms_v2 | 11 400 Gio | 2 000 Mbits/s | 3 x P15 | 375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 | 


Pour les autres volumes, la configuration ressemblerait à ceci :

| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| M32ts | 192 Gio | 500 Mo/s | 1 x P15 | 1 x P6 | 1 x P6 |
| M32ls | 256 Gio | 500 Mo/s |  1 x P15 | 1 x P6 | 1 x P6 |
| M64ls | 512 Go | 1 000 Mo/s | 1 x P20 | 1 x P6 | 1 x P6 |
| M64s | 1 000 Gio | 1 000 Mbits/s | 1 x P30 | 1 x P6 | 1 x P6 |
| M64ms | 1 750 Gio | 1 000 Mbits/s | 1 x P30 | 1 x P6 | 1 x P6 | 
| M128s | 2 000 Gio | 2 000 Mbits/s | 1 x P30 | 1 x P10 | 1 x P6 | 
| M128ms | 3,800 Gio | 2 000 Mbits/s | 1 x P30 | 1 x P10 | 1 x P6 |
| M208s_v2 | 2 850 Gio | 1 000 Mbits/s |  1 x P30 | 1 x P10 | 1 x P6 |
| M208ms_v2 | 5 700 Gio | 1 000 Mbits/s | 1 x P30 | 1 x P10 | 1 x P6 | 
| M416s_v2 | 5 700 Gio | 2 000 Mbits/s |  1 x P30 | 1 x P10 | 1 x P6 | 
| M416ms_v2 | 11 400 Gio | 2 000 Mbits/s | 1 x P30 | 1 x P10 | 1 x P6 | 


Vérifiez que le débit de stockage des différents volumes suggérés est suffisant pour la charge de travail à exécuter. Si la charge de travail nécessite de plus grands volumes pour **/hana/data** et **/hana/log**, augmentez le nombre de disques durs virtuels de stockage Azure Premium. Le dimensionnement d’un volume avec davantage de disques durs virtuels que le nombre suggéré permet d’augmenter le débit d’IOPS et d’E/S dans les limites définies pour le type de machine virtuelle Azure.

L’Accélérateur des écritures Azure fonctionne uniquement en association avec des [disques managés Azure](https://azure.microsoft.com/services/managed-disks/). Cela signifie que les disques de stockage Azure Premium constituant le volume **/hana/log** doivent être déployés en tant que disques managés. Vous trouverez des instructions et des restrictions plus détaillées sur l’Accélérateur des écritures Azure dans l’article [Accélérateur des écritures](../../how-to-enable-write-accelerator.md).

Pour les machines virtuelles certifiées HANA de la famille Azure [Esv3](../../ev3-esv3-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#esv3-series) et [Edsv4](../../edv4-edsv4-series.md?toc=/azure/virtual-machines/linux/toc.json&bc=/azure/virtual-machines/linux/breadcrumb/toc.json#edsv4-series), vous avez besoin d’ANF pour les volumes **/hana/data** et **/hana/log**. Ou vous devez tirer parti du stockage sur disque Ultra Azure au lieu du stockage Premium Azure uniquement pour le volume **/hana/log**. Par conséquent, les configurations pour le volume **/hana/data** sur le stockage Premium Azure peuvent ressembler à ceci :

| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | /hana/data | Débit approvisionné | Débit maximum de rafale | E/S par seconde | IOPS en rafale |
| --- | --- | --- | --- | --- | --- | --- |
| E20ds_v4 | 160 Gio | 480 Mo/s | 3 x P10 | 300 Mbits/s | 510 Mbits/s | 1 500 | 10 500 |
| E32ds_v4 | 256 Gio | 768 Mbits/s | 3 x P10 |  300 Mbits/s | 510 Mbits/s | 1 500 | 10 500|
| E48ds_v4 | 384 Gio | 1 152 Mo/s | 3 x P15 |  375 Mbits/s |510 Mbits/s | 3 300  | 10 500 | 
| E64ds_v4 | 504 Gio | 1 200 Mbits/s | 3 x P15 |  375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 | 
| E64s_v3 | 432 Gio | 1 200 Mo/s | 3 x P15 |  375 Mbits/s | 510 Mbits/s | 3 300 | 10 500 | 

Pour les autres volumes, y compris **/hana/log** sur disque Ultra, la configuration peut ressembler à ceci :

| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | Volume /hana/log | Débit d’e/s /hana/log | IOPS /hana/log | /hana/shared | /root volume | /usr/sap |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 Gio | 480 Mo/s | 80 Go | 250 Mbits/s | 1 800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E32ds_v4 | 256 Gio | 768 Mbits/s | 128 Go | 250 Mbits/s | 1 800 | 1 x P15 | 1 x P6 | 1 x P6 |
| E48ds_v4 | 384 Gio | 1 152 Mo/s | 192 Go | 250 Mbits/s | 1 800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64ds_v4 | 504 Gio | 1 200 Mbits/s | 256 Go | 250 Mbits/s | 1 800 | 1 x P20 | 1 x P6 | 1 x P6 |
| E64s_v3 | 432 Gio | 1 200 Mbits/s | 220 Go | 250 Mbits/s | 1 800 | 1 x P20 | 1 x P6 | 1 x P6 |


## <a name="azure-ultra-disk-storage-configuration-for-sap-hana"></a>Configuration du stockage sur disque Ultra Azure pour SAP HANA
Un autre type de stockage Azure est appelé [Disque Ultra Azure](../../disks-types.md#ultra-disk). La grande différence entre le stockage Azure proposé jusqu’à présent et le disque Ultra est que les capacités du disque ne sont plus limitées à la taille du disque. En tant que client, vous pouvez définir ces capacités pour le disque Ultra :

- Taille d’un disque allant de 4 à 65 536 Gio
- IOPS allant de 100 à 160K (la valeur maximum dépend également des types de machine virtuelle)
- Débit de stockage de 300 à 2 000 Mo/s

Avec le disque Ultra, vous avez la possibilité de définir un seul disque qui répond à vos exigences en matière de plage de débit de disque, d’IOPS et de taille. Au lieu d’utiliser des gestionnaires de volumes logiques tels que LVM ou MDADM en plus du stockage Azure Premium pour construire des volumes qui respectent les exigences de débit de stockage et d’IOPS, vous pouvez choisir une configuration mixte entre le disque Ultra et le stockage Premium. Vous pouvez ainsi limiter l’utilisation du disque Ultra aux volumes **/hana/data** et **/hana/log** ayant des performances critiques et utiliser le stockage Premium Azure pour les autres volumes

Les autres avantages du disque Ultra peuvent être la meilleure latence de lecture par rapport au stockage Premium. La latence de lecture plus courte peut avoir des avantages quand vous souhaitez réduire le temps de démarrage HANA et la charge ultérieure des données en mémoire. Les avantages du stockage sur disque Ultra peuvent également être observés quand HANA écrit des points d’enregistrement. 

> [!NOTE]
> Le disque Ultra n’est pas encore disponible dans toutes les régions Azure, ni pris en charge par tous les types de machines virtuelles listés ci-dessous. Pour plus d’informations sur la disponibilité du disque Ultra et sa prise en charge par les différentes familles de machines virtuelles, consultez l’article [Quels sont les types de disque disponibles dans Azure ?](../../disks-types.md#ultra-disk).

### <a name="production-recommended-storage-solution-with-pure-ultra-disk-configuration"></a>Solution de stockage recommandée pour la production avec une pure configuration de disque Ultra
Dans cette configuration, vous conservez les volumes **hana/data** et **/hana/log** séparément. Les valeurs suggérées sont dérivées des indicateurs de performance clés que SAP doit certifier pour les configurations SAP HANA et de stockage, comme recommandé dans le [livre blanc sur le stockage SAP TDI](https://www.sap.com/documents/2015/03/74cdb554-5a7c-0010-82c7-eda71af511fa.html).

Les recommandations vont souvent au-delà des conditions minimales requise par SAP, comme indiqué plus haut dans cet article. Les recommandations listées sont un compromis entre les recommandations de taille de SAP et le débit de stockage maximal fourni par les différents types de machines virtuelles.

> [!NOTE]
> Le disque Ultra Azure impose au minimum 2 IOPS par Go de capacité d’un disque


| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | Volume /hana/data | Débit d’e/s /hana/data | IOPS /hana/data | Volume /hana/log | Débit d’e/s /hana/log | IOPS /hana/log |
| --- | --- | --- | --- | --- | --- | --- | --- | -- |
| E20ds_v4 | 160 Gio | 480 Mo/s | 200 Go | 400 Mbits/s | 2 500 | 80 Go | 250 Mo | 1 800 |
| E32ds_v4 | 256 Gio | 768 Mo/s | 300 Go | 400 Mbits/s | 2 500 | 128 Go | 250 Mbits/s | 1 800 |
| E48ds_v4 | 384 Gio | 1152 Mo/s | 460 Go | 400 Mbits/s | 3 000 | 192 Go | 250 Mbits/s | 1 800 |
| E64ds_v4 | 504 Gio | 1 200 Mo/s | 610 Go | 400 Mbits/s | 3 500 |  256 Go | 250 Mbits/s | 1 800 |
| E64s_v3 | 432 Gio | 1 200 Mo/s | 610 Go | 400 Mbits/s | 3 500 | 220 Go | 250 Mo | 1 800 |
| M32ts | 192 Gio | 500 Mo/s | 250 Go | 400 Mbits/s | 2 500 | 96 Go | 250 Mbits/s  | 1 800 |
| M32ls | 256 Gio | 500 Mo/s | 300 Go | 400 Mbits/s | 2 500 | 256 Go | 250 Mbits/s  | 1 800 |
| M64ls | 512 Go | 1 000 Mo/s | 620 Go | 400 Mbits/s | 3 500 | 256 Go | 250 Mbits/s  | 1 800 |
| M64s | 1 000 Gio | 1 000 Mo/s |  1 200 GO | 600 Mbits/s | 5 000 | 512 Go | 250 Mbits/s  | 2 500 |
| M64ms | 1 750 Gio | 1 000 Mo/s | 2 100 Gio | 600 Mbits/s | 5 000 | 512 Go | 250 Mbits/s  | 2 500 |
| M128s | 2 000 Gio | 2 000 Mo/s |2 400 Go | 750 Mo/s | 7 000 | 512 Go | 250 Mbits/s  | 2 500 | 
| M128ms | 3,800 Gio | 2 000 Mo/s | 4 800 Go | 750 Mo/s |9 600 | 512 Go | 250 Mbits/s  | 2 500 | 
| M208s_v2 | 2 850 Gio | 1 000 Mo/s | 3 500 Go | 750 Mo/s | 7 000 | 512 Go | 250 Mbits/s  | 2 500 | 
| M208ms_v2 | 5 700 Gio | 1 000 Mo/s | 7 200 Go | 750 Mo/s | 14 400 | 512 Go | 250 Mbits/s  | 2 500 | 
| M416s_v2 | 5 700 Gio | 2 000 Mo/s | 7 200 Go | 1 000 Mbits/s | 14 400 | 512 Go | 400 Mbits/s  | 4 000 | 
| M416ms_v2 | 11 400 Gio | 2 000 Mo/s | 14 400 Go | 1 500 Mbits/s | 28 800 | 512 Go | 400 Mbits/s  | 4 000 |   

**Les valeurs répertoriées sont destinées à être un point de départ et doivent être évaluées en fonction de la demande réelle.** L’avantage du disque Ultra Azure est que les valeurs d’IOPS et de débit peuvent être adaptées sans avoir à arrêter la machine virtuelle, ni la charge de travail appliquée au système.   

> [!NOTE]
> À ce jour, les captures instantanées du disque de stockage Ultra ne sont pas disponibles. Cela empêche l’utilisation de captures instantanées de machine virtuelle avec les services Sauvegarde Azure


## <a name="nfs-v41-volumes-on-azure-netapp-files"></a>Volumes NFS v4.1 sur Azure NetApp Files
Pour plus d’informations sur ANF pour HANA, lisez le document [Volumes NFS v4.1 sur Azure NetApp Files pour SAP HANA](./hana-vm-operations-netapp.md)




## <a name="cost-conscious-solution-with-azure-premium-storage"></a>Solution économique avec le stockage Premium Azure
Jusqu’à présent, la solution de stockage premium Azure décrite dans ce document à la section [Solutions avec stockage Premium et Accélérateur d’écriture Azure pour les machines virtuelles Azure de la série M](#solutions-with-premium-storage-and-azure-write-accelerator-for-azure-m-series-virtual-machines) était destinée aux scénarios de production SAP HANA pris en charge. Les configurations de production prises en charge ont pour caractéristique de séparer les volumes pour les données SAP HANA et de rétablir le journal en deux volumes différents. Une telle séparation est due au fait que les caractéristiques des charges de travail sont différentes sur les volumes. Et qu’avec les configurations de production proposées, des types de mise en cache différents ou même des types de stockage bloc Azure pourraient être nécessaires. Pour les scénarios hors production, certaines des considérations valables pour les systèmes de production peuvent ne pas s’appliquer à des systèmes hors production de bas de gamme. Par conséquent, les données HANA et le volume du fichier journal peuvent être combinés. Bien qu’une telle approche puisse révéler des comportements non conformes, comme le non-respect de certains KPI de débit ou de latence qui sont nécessaires aux systèmes de production. Le [Stockage SSD Standard Azure](./planning-guide-storage.md#azure-standard-ssd-storage) peut constituer un autre moyen de réduire les coûts dans de tels environnements. N’oubliez pas que le fait de choisir un stockage Azure SSD Standard ou HDD Standard a un impact sur vos contrats SLA de machines virtuelles uniques, comme décrit dans l’article  [SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines).

Une alternative moins coûteuse pour ces configurations peut ressembler à ceci :


| Référence de la machine virtuelle | RAM | Bande passante E/S DE MACHINE VIRTUELLE<br /> Débit | /hana/data et /hana/log<br /> agrégés avec LVM ou MDADM | /hana/shared | /root volume | /usr/sap | comments |
| --- | --- | --- | --- | --- | --- | --- | -- |
| DS14v2 | 112 Gio | 768 Mo/s | 4 x P6 | 1 x E10 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| E16v3 | 128 Go | 384 Mo/s | 4 x P6 | 1 x E10 | 1 x E6 | 1 x E6 | Type de machine virtuelle non certifié HANA <br /> N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| M32ts | 192 Gio | 500 Mo/s | 3 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 5 000<sup>2</sup> |
| E20ds_v4 | 160 Gio | 480 Mo/s | 4 x P6 | 1 x E15 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| E32v3 | 256 Gio | 768 Mo/s | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | Type de machine virtuelle non certifié HANA <br /> N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| E32ds_v4 | 256 Gio | 768 Mbits/s | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| M32ls | 256 Gio | 500 Mo/s | 4 x P10 | 1 x E15 | 1 x E6 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 5 000<sup>2</sup> |
| E48ds_v4 | 384 Gio | 1 152 Mo/s | 6 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| E64v3 | 432 Gio | 1 200 Mo/s | 6 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| E64ds_v4 | 504 Gio | 1 200 Mo/s |  7 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | N’atteindra pas une latence de stockage inférieure à 1 ms<sup>1</sup> |
| M64ls | 512 Go | 1 000 Mo/s | 7 x P10 | 1 x E20 | 1 x E6 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 10 000<sup>2</sup> |
| M64s | 1 000 Gio | 1 000 Mo/s | 7 x P15 | 1 x E30 | 1 x E6 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 10 000<sup>2</sup> |
| M64ms | 1 750 Gio | 1 000 Mo/s | 6 x P20 | 1 x E30 | 1 x E6 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 10 000<sup>2</sup> |
| M128s | 2 000 Gio | 2 000 Mo/s |6 x P20 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 20 000<sup>2</sup> |
| M208s_v2 | 2 850 Gio | 1 000 Mo/s | 4 x P30 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 10 000<sup>2</sup> |
| M128ms | 3,800 Gio | 2 000 Mo/s | 5 x P30 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 20 000<sup>2</sup> |
| M208ms_v2 | 5 700 Gio | 1 000 Mo/s | 4 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 10 000<sup>2</sup> |
| M416s_v2 | 5 700 Gio | 2 000 Mo/s | 4 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 20 000<sup>2</sup> |
| M416ms_v2 | 11 400 Gio | 2 000 Mo/s | 7 x P40 | 1 x E30 | 1 x E10 | 1 x E6 | L’utilisation d’Accélérateur d’écriture pour les données combinées et le volume des journaux limite le taux d’E/S par seconde à 20 000<sup>2</sup> |


<sup>1</sup> [Accélérateur d’écriture Azure](../../how-to-enable-write-accelerator.md) ne peut pas être utilisé avec les familles de machines virtuelles Ev4 et Ev4 VM. En raison de l’utilisation du stockage Premium Azure, la latence des E/S ne sera pas inférieure à 1 ms

<sup>2</sup> La famille de machines virtuelles prend en charge [Accélérateur d’écriture Azure](../../how-to-enable-write-accelerator.md), mais il est possible que la limite d’IOPS d’Accélérateur d’écriture ait un impact sur les capacités d’IOPS des configurations de disque

Dans le cas de la combinaison du volume de données et du volume de journaux pour SAP HANA, les disques qui créent le volume agrégé par bandes ne doivent pas disposer d’un cache de lecture ou d’un cache en lecture/écriture activé.

Certains types de machines virtuelles répertoriés ne sont pas certifiés par SAP et ne figurent donc pas dans le [répertoire du matériel certifié SAP HANA](https://www.sap.com/dmc/exp/2014-09-02-hana-hardware/enEN/iaas.html#categories=Microsoft%20Azure). Selon les commentaires des clients, ces types de machine virtuelle non répertoriés étaient utilisés avec succès pour certaines tâches hors production.


## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations, consultez les pages suivantes :

- [Haute disponibilité de SAP HANA pour les machines virtuelles Azure](./sap-hana-availability-overview.md).
---
title: Types de stockage Azure pour une charge de travail SAP
description: Planification des types de stockage Azure pour les charges de travail SAP
services: virtual-machines-linux,virtual-machines-windows
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
tags: azure-resource-manager
keywords: ''
ms.assetid: d7c59cc1-b2d0-4d90-9126-628f9c7a5538
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure-services
ms.date: 11/26/2020
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 6982b782fdd6b5b269c1562c54be3478c58bbce9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500995"
---
# <a name="azure-storage-types-for-sap-workload"></a>Types de stockage Azure pour une charge de travail SAP
Azure possède de nombreux types de stockage qui diffèrent notablement en termes de capacités, de débit, de latence et de prix. Certains des types de stockage ne sont pas, ou peu, utilisables pour les scénarios SAP. En revanche, plusieurs types de stockage Azure sont bien adaptés ou optimisés pour des scénarios de charge de travail SAP spécifiques. Pour SAP HANA en particulier, certains types de stockage Azure ont été certifiés pour être utilisés avec SAP HANA. Dans ce document, nous passons en revue les différents types de stockage et décrivons leur capacité et leur facilité d'utilisation avec les charges de travail et les composants SAP.

Remarquez les unités utilisées dans cet article. Les fournisseurs de cloud public ont décidé d'utiliser le Gio ([Gibioctet](https://en.wikipedia.org/wiki/Gibibyte)) ou le Tio ([Tebioctet](https://en.wikipedia.org/wiki/Tebibyte)) comme unités de taille, au lieu du gigaoctet ou du téraoctet. C’est pourquoi toute la documentation et la tarification Azure utilisent ces unités.  Dans le document, nous référençons ces unités de taille en unités Mio, Gio et Tio exclusivement. Vous devrez peut-être planifier des Mo, des Go et des To. Par conséquent, vous devez tenir compte de quelques petites différences dans les calculs si vous devez dimensionner un débit de 400 Mio/s au lieu d’un débit de 250 Mio/s.

## <a name="microsoft-azure-storage-resiliency"></a>Résilience du Stockage Microsoft Azure

Le stockage Microsoft Azure comprenant des disques HDD Standard et SSD Standard, le Stockage Premium Azure et un disque Ultra conserve le disque dur virtuel de base (avec SE) et les disques de données attachés à des machines virtuelles ou les disques durs virtuels sur trois nœuds de stockage différents. Le basculement vers un autre réplica et l’amorçage d’un nouveau réplica en cas de défaillance d’un nœud de stockage sont transparents. En raison de cette redondance, il n'est **PAS** nécessaire d’utiliser une quelconque couche de redondance de stockage sur plusieurs disques Azure. Il s’agit du stockage localement redondant (LRS). Le LRS est la valeur par défaut pour ces types de stockage dans Azure. [Azure NetApp Files](https://azure.microsoft.com/services/netapp/) offre une redondance suffisante pour atteindre les mêmes contrats SLA que les autres stockages Azure natifs.

Il existe plusieurs autres méthodes de redondance, qui sont toutes décrites dans l’article [Réplication du Stockage Azure](../../../storage/common/storage-redundancy.md?toc=%2fazure%2fstorage%2fqueues%2ftoc.json) et qui s’appliquent à certains types de stockage proposés par Azure. 

Gardez également à l’esprit que différents types de stockage Azure influencent la disponibilité de machine virtuelle unique. Les contrats SLA sont publiés dans [Contrat SLA pour les machines virtuelles](https://azure.microsoft.com/support/legal/sla/virtual-machines).

### <a name="azure-managed-disks"></a>Disques managés Azure

Les disques managés sont un type de ressources d’Azure Resource Manager. Ils peuvent être utilisés à la place des disques durs virtuels qui sont stockés dans les comptes de stockage Azure. Les disques managés s’alignent automatiquement sur le [groupe à haute disponibilité][disponibilité de gestion des machines virtuelles] de la machine virtuelle à laquelle ils sont attachés. De fait, ils augmentent la disponibilité de votre machine virtuelle et des services exécutés sur celle-ci. Pour plus d’informations, consultez [l’article de vue d’ensemble](../../managed-disks-overview.md).

En rapport avec la résilience, cet exemple illustre l’avantage des disques managés :

- Vous déployez vos deux machines virtuelles SGBD pour votre système SAP dans un groupe à haute disponibilité Azure. 
- Dans la mesure où Azure déploie les machines virtuelles, le disque avec l’image du système d’exploitation sera placé dans un autre cluster de stockage. Cela évite que les deux machines virtuelles soient affectées par un problème d’un seul cluster de stockage Azure
- Lorsque vous créez des disques managés que vous attribuez à ces machines virtuelles pour stocker les données et les fichiers journaux de votre base de données, ces nouveaux disques destinés aux deux machines virtuelles sont également déployés dans des clusters de stockage distincts. Par conséquent, aucun des disques de la première machine virtuelle ne partage les clusters de stockage avec les disques de la deuxième machine virtuelle

Effectuant le déploiement sans disques managés dans les comptes de stockage définis par le client, l’allocation de disque est arbitraire et n’est pas en mesure de savoir que les machines virtuelles sont déployées dans un AvSet à des fins de résilience.

> [!NOTE]
> Pour cette raison et pour plusieurs autres améliorations qui sont exclusivement disponibles via des disques managés, nous exigeons que les nouveaux déploiements de machines virtuelles qui utilisent le stockage de bloc Azure pour leurs disques (tout le stockage Azure sauf Azure NetApp Files) doivent utiliser des disques managés Azure pour les disques virtuels/de système d’exploitation de base, des disques de données qui contiennent des fichiers de base de données SAP. Indépendamment de la nécessité de déployer les machines virtuelles via un groupe à haute disponibilité, sur Zones de disponibilité ou indépendamment des ensembles et des zones. Les disques utilisés pour stocker des sauvegardes ne doivent pas nécessairement être des disques managés.

> [!NOTE]
> Les disques managés Azure assurent uniquement la redondance locale (LRS). 


## <a name="storage-scenarios-with-sap-workloads"></a>Scénarios de stockage avec charges de travail SAP
Le stockage persistant est nécessaire dans la charge de travail SAP dans les différents composants de la pile que vous déployez dans Azure. Ces scénarios présentent des situations au moins similaires à celles-ci :

- Conservez le disque dur virtuel de base de votre machine virtuelle contenant le système d’exploitation et les autres logiciels que vous installez sur ce disque. Ce disque/disque dur virtuel est la racine de votre machine virtuelle. Toutes les modifications qui lui ont été apportées doivent être rendues persistantes. C’est pourquoi la prochaine fois que vous arrêtez et redémarrez la machine virtuelle, toutes les modifications apportées auparavant continuent d’exister. En particulier dans les cas où la machine virtuelle est déployée par Azure sur un autre hôte que celui qu’elle exécutait à l’origine
- Disques de données persistantes. Ces disques sont des disques durs virtuels que vous joignez pour y stocker des données d’application. Il peut s’agir de données et de fichiers journaux/de restauration par progression d’une base de données, de fichiers de sauvegarde ou d’installations de logiciels. Désigne tout disque au-delà de votre disque dur virtuel de base qui héberge le système d’exploitation
- Partages de fichiers ou disques partagés qui contiennent votre répertoire de transport global pour NetWeaver ou S/4HANA. Le contenu de ces partages est consommé par un logiciel s’exécutant sur plusieurs machines virtuelles ou utilisé pour créer des scénarios de cluster de basculement à haute disponibilité.
- Répertoire /sapmnt ou partages de fichiers communs pour les processus EDI ou similaires. Le contenu de ces partages est consommé par un logiciel s’exécutant sur plusieurs machines virtuelles ou utilisé pour créer des scénarios de cluster de basculement à haute disponibilité.

Dans les sections suivantes, les différents types de stockage Azure et leur utilisabilité pour la charge de travail SAP sont abordés et s’appliquent aux quatre scénarios ci-dessus. L’article [Quels sont les types de disque disponibles dans Azure ?](../../disks-types.md) répertorie les façons dont les différents types de stockage Azure doivent être utilisés. Les recommandations relatives à l’utilisation des différents types de stockage Azure pour la charge de travail SAP ne vont pas être très différentes.

Pour en savoir plus sur les restrictions de support sur les types de stockage Azure pour la couche d’application/SAP NetWeaver de S/4HANA, lisez la [note de support SAP 2015553](https://launchpad.support.sap.com/#/notes/2015553) Pour en savoir plus sur les types de stockage Azure certifiés et pris en charge par SAP HANA, consultez l'article [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md).

Les sections décrivant les différents types de stockage Azure vous donnent plus d’informations sur les restrictions et les possibilités d’utilisation du stockage que SAP prend en charge. 

## <a name="storage-recommendations-for-sap-storage-scenarios"></a>Recommandations de stockage pour les scénarios de stockage SAP
Avant d’aborder les détails, nous présentons le résumé et les recommandations au début du document. Tandis que les détails des types particuliers de stockage Azure suivent cette section du document. Les recommandations de stockage SAP sont répertoriées dans le tableau suivant :

| Scénario d’usage | HDD Standard | SSD Standard | Stockage Premium | Disque Ultra | Azure NetApp Files |
| --- | --- | --- | --- | --- | --- |
| Disque de système d’exploitation | non approprié |  restreint adapté (non prod) | recommandé | impossible | impossible |
| Répertoire de transport global | non pris en charge | non pris en charge | recommandé | recommandé | recommandé |
| /sapmnt | non approprié | restreint adapté (non prod) | recommandé | recommandé | recommandé |
| Volume de données SGBD SAP HANA pour les familles de machines virtuelles M/Mv2 | non pris en charge | non pris en charge | recommandé | recommandé | recommandé<sup>2</sup> |
| Volume du journal SGBD SAP HANA pour les familles de machines virtuelles M/Mv2 | non pris en charge | non pris en charge | recommandé<sup>1</sup> | recommandé | recommandé<sup>2</sup> | 
| Volume de données SGBD SAP HANA pour les familles de machines virtuelles Esv3/Edsv4 | non pris en charge | non pris en charge | recommandé | recommandé | recommandé<sup>2</sup> |
| Volume du journal SGBD SAP HANA pour les familles de machines virtuelles Esv3/Edsv4 | non pris en charge | non pris en charge | non pris en charge | recommandé | recommandé<sup>2</sup> | 
| Volume de données SGBD non HANA | non pris en charge | restreint adapté (non prod) | recommandé | recommandé | non pris en charge |
| Volume du journal de SGBD non HANA des familles de machines virtuelles M/Mv2 | non pris en charge | restreint adapté (non prod) | recommandé<sup>1</sup> | recommandé | non pris en charge |
| Volume du journal de SGBD non HANA n’appartenant aux familles de machines virtuelles M/Mv2 | non pris en charge | restreint adapté (non prod) | convient pour une charge de travail faible à moyenne | recommandé | non pris en charge |


<sup>1</sup> Avec utilisation de l’[Accélérateur d’écriture Azure](../../how-to-enable-write-accelerator.md) pour les familles de machines virtuelles M/Mv2 et les volumes journal/restauration par progression <sup>2</sup> ANF requiert la présence de volumes /hana/data et /hana/log 

Caractéristiques que vous pouvez attendre des types de stockage différents :

| Scénario d’usage | HDD Standard | SSD Standard | Stockage Premium | Disque Ultra | Azure NetApp Files |
| --- | --- | --- | --- | --- | --- |
| SLA de débit/IOPS | non | non | oui | Oui | Oui |
| Lectures de latence | high | moyen à élevé | low | sous-milliseconde | sous-milliseconde |
| Écritures de latence | high | moyen à élevé  | faible (sous-milliseconde<sup>1</sup>) | sous-milliseconde | sous-milliseconde |
| HANA pris en charge | non | non | Oui<sup>1</sup> | Oui | Oui |
| Captures instantanées de disque possibles | Oui | Oui | Oui | non | Oui |
| Allocation de disques sur différents clusters de stockage lors de l’utilisation de groupes à haute disponibilité | via des disques managés | via des disques managés | via des disques managés | type de disque non pris en charge avec les machines virtuelles déployées via des groupes à haute disponibilité | non<sup>3</sup> |
| Aligné avec les Zones de disponibilité | Oui | Oui | Oui | Oui | nécessite l’engagement de Microsoft |
| Redondance de zone | pas pour les disques managés | pas pour les disques managés | pas pour les disques managés | non | non |
| Redondance géographique | pas pour les disques managés | pas pour les disques managés | non | non | non |


<sup>1</sup> Avec utilisation de l’[Accélérateur d’écriture Azure](../../how-to-enable-write-accelerator.md) pour les familles de machines virtuelles M/Mv2 et les volumes journal/restauration par progression

<sup>2</sup> Les coûts dépendent des IOPS et du débit approvisionnés

<sup>3</sup> La création de pools de capacité différents ne garantit pas le déploiement de pools de capacité sur des unités de stockage différentes.


> [!IMPORTANT]
> Pour atteindre une latence d’E/S inférieure à 1 milliseconde à l’aide d’Azure NetApp Files (ANF), vous devez collaborer avec Microsoft pour organiser le positionnement correct entre vos machines virtuelles et les partages NFS basés sur ANF. Jusqu’à présent, aucun mécanisme n’a été mis en place pour assurer une proximité automatique entre une machine virtuelle déployée et les volumes NFS hébergés sur ANF. Compte tenu de la configuration différente des régions Azure différentes, la latence du réseau ajoutée peut pousser la latence d’E/S au-delà de 1 milliseconde si la machine virtuelle et le partage NFS ne sont pas alloués à proximité.


> [!IMPORTANT]
> Aucun des disques managés basés sur le stockage de blocs Azure actuellement proposés, ni aucun des fichiers Azure NetApp Files n’offre de redondance zonale ou géographique. Par conséquent, vous devez vous assurer que vos architectures de haute disponibilité et de récupération d’urgence ne reposent sur aucun type de réplication de stockage Azure native pour ces disques managés, les partages NFS ou SMB.


## <a name="azure-premium-storage"></a>Stockage Premium Azure
Le stockage SSD Premium Azure a été introduit dans le but de fournir les avantages suivants :

* Faible latence des E/S
* Contrat de niveau de service pour les IOPS et le débit
* Moins de variabilité dans la latence des E/S

Ce type de stockage cible les charges de travail SGBD, le trafic de stockage qui nécessite une latence faible à un chiffre, et les contrats SLA sur les IOPS et le coût du débit dans le cas du stockage Premium Azure ne correspondent pas au volume de données réel stocké sur ces disques, mais à la catégorie de taille d’un tel disque, indépendamment de la quantité de données stockées sur le disque. Vous pouvez également créer des disques dans Stockage Premium qui ne sont pas directement mappés aux catégories de taille présentées dans l’article [SSD Premium](../../disks-types.md#premium-ssd). Les conclusions de cet article sont les suivantes :

- Le stockage est organisé en plages. Par exemple, un disque dans la plage 513 Gio à 1024 Gio partage les mêmes fonctionnalités et les mêmes coûts mensuels.
- Les IOPS par Gio ne suivent pas de manière linéaire les catégories de taille. Les petits disques de moins de 32 Gio ont des taux d'IOPS par Gio plus élevés. Pour les disques entre 32 Gio et 1024 Gio, le taux d'IOPS par Gio se situe entre 4 et 5 IOPS par Gio. Pour les disques plus volumineux jusqu'à 32 767 Gio, le taux d’IOPS par Gio est inférieur à 1
- Le débit d'entrée/sortie de ce stockage n'est pas linéaire avec la taille de la catégorie de disque. Pour les disques plus petits, comme la catégorie entre 65 Gio et 128 Gio de capacité, le débit est d’environ 780 Ko par Gio. Tandis que pour les disques de grande taille, comme un disque de 32 767 Gio, le débit est de 28 Ko par Gio
- Les contrats SLA d’IOPS et de débit ne peuvent pas être modifiés sans que la capacité du disque soit modifiée


La matrice de capacité pour la charge de travail SAP ressemble à ceci :

| Fonctionnalité| Commentaire| Remarques/liens | 
| --- | --- | --- | 
| Disque dur virtuel de base du système d’exploitation | approprié | tous les systèmes |
| Disque de données | approprié | tous les systèmes - [spécialement pour SAP HANA](../../how-to-enable-write-accelerator.md) |
| Répertoire de transport global SAP | YES | [Pris en charge](https://launchpad.support.sap.com/#/notes/2015553) |
| SAP sapmnt | approprié | tous les systèmes |
| Stockage de sauvegarde | approprié | pour le stockage à court terme des sauvegardes |
| Partages/disque partagé | non disponible | Nécessite des fichiers Azure Premium ou des fournisseurs tiers |
| Résilience | LRS | Aucun GRS ou ZRS n’est disponible pour les disques |
| Latence | faible à moyen | - |
| Contrat de niveau de service IOPS | YES | - |
| IOPS linéaires à la capacité | semi-linéaire entre crochets  | [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/) |
| Maximum d’E/S par seconde par disque | 20 000 [dépendant de la taille du disque](https://azure.microsoft.com/pricing/details/managed-disks/) | Tenez également compte des [limites de machine virtuelle](../../sizes.md) |
| SLA de débit | YES | - |
| Débit linéaire par rapport à la capacité | semi-linéaire entre crochets | [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/) |
| Certifié HANA | YES | [spécialement pour SAP HANA](../../how-to-enable-write-accelerator.md) |
| Captures instantanées de disque possibles | YES | - |
| Captures instantanées de machines virtuelles Sauvegarde Azure possibles | YES | à l’exception des disques mis en cache d’[Accélérateur d’écriture](../../how-to-enable-write-accelerator.md)  |
| Coûts | MEDIUM | - |

Le stockage premium Azure ne remplit pas les KPI de latence du stockage SAP HANA avec les types de cache courants proposés avec le stockage premium Azure. Afin de remplir les KPI de latence de stockage pour les écritures de log SAP HANA, vous devez utiliser la mise en cache d’Accélérateur d'écriture Azure comme décrit dans l'article [Activer l’Accélérateur d’écriture](../../how-to-enable-write-accelerator.md). Accélérateur d’écriture Azure tire parti de tous les autres systèmes SGBD pour l’écriture du journal des transactions et la restauration des écritures de journal. Par conséquent, il est recommandé de l’utiliser dans tous les déploiements de SGBD SAP. Pour SAP HANA, l'utilisation d’Accélérateur d’écriture Azure est obligatoire conjointement avec le Stockage Premium Azure.



**Résumé :** Le Stockage premium Azure est l'un des types de stockage Azure recommandé pour la charge de travail SAP. Cette recommandation s’applique aux systèmes de production et hors production. Le Stockage premium Azure est adapté pour gérer les charges de travail des bases de données. L’utilisation d’Accélérateur d’écriture Azure va améliorer considérablement la latence d’écriture sur les disques premium Azure. Toutefois, pour les systèmes SGBD avec des taux d’IOPS et des taux de débit élevés, vous devez augmenter la capacité de stockage ou utiliser des fonctionnalités telles que les espaces de stockage Windows ou les gestionnaires de volumes logiques dans Linux afin de créer des ensembles de bandes qui vous donnent la capacité souhaitée d’une part, mais également le débit ou les IOPS nécessaires.


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


## <a name="azure-ultra-disk"></a>Disque Ultra Azure
Les disques Ultra Azure fournissent un stockage sur disque présentant un débit élevé, un nombre d’IOPS élevé et une faible latence homogène pour les machines virtuelles Azure IaaS. Entre autres avantages, les disques Ultra permettent de changer dynamiquement les IOPS et le débit des disques en fonction de vos charges de travail sans avoir à redémarrer les machines virtuelles. Les disques Ultra sont adaptés aux charges de travail à forte intensité de données telles que la charge de travail du SGBD SAP. Les disques Ultra ne peuvent être utilisés que comme disques de données et comme disque virtuel de base qui stocke le système d'exploitation. Nous recommandons l’utilisation du Stockage Premium Azure en tant que disque dur virtuel.

En créant un disque Ultra, vous disposez de trois dimensions que vous pouvez définir :

- Capacité du disque. Les plages sont comprises entre 4 Gio et 65 536 Gio
- IOPS provisionnées pour le disque. Des valeurs maximales différentes s’appliquent à la capacité du disque. Lire l’article [Disque Ultra](../../disks-types.md#ultra-disk) pour plus de détails
- Bande passante de stockage approvisionnée. Une bande passante maximale différente s’applique en fonction de la capacité du disque. Lire l’article [Disque Ultra](../../disks-types.md#ultra-disk) pour plus de détails

Le coût d’un seul disque est déterminé par les trois dimensions que vous pouvez définir pour les disques spécifiques séparément. 


La matrice de capacité pour la charge de travail SAP ressemble à ceci :

| Fonctionnalité| Commentaire| Remarques/liens | 
| --- | --- | --- | 
| Disque dur virtuel de base du système d’exploitation | ne fonctionne pas | - |
| Disque de données | approprié | tous les systèmes  |
| Répertoire de transport global SAP | YES | [Pris en charge](https://launchpad.support.sap.com/#/notes/2015553) |
| SAP sapmnt | approprié | tous les systèmes |
| Stockage de sauvegarde | approprié | pour le stockage à court terme des sauvegardes |
| Partages/disque partagé | non disponible | A besoin d’un tiers |
| Résilience | LRS | Aucun GRS ou ZRS n’est disponible pour les disques |
| Latence | très faible | - |
| Contrat de niveau de service IOPS | YES | - |
| IOPS linéaires à la capacité | semi-linéaire entre crochets  | [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/) |
| Maximum d’E/S par seconde par disque | 1 200 à 160 000 | dépend de la capacité du disque |
| SLA de débit | YES | - |
| Débit linéaire par rapport à la capacité | semi-linéaire entre crochets | [Tarification des disques managés](https://azure.microsoft.com/pricing/details/managed-disks/) |
| Certifié HANA | YES | - |
| Captures instantanées de disque possibles | Non | - |
| Captures instantanées de machines virtuelles Sauvegarde Azure possibles | Non | - |
| Coûts | Supérieur au stockage Premium | - |



**Résumé :** Les disques Ultra Azure constituent un stockage adapté à faible latence pour toutes sortes de charges de travail SAP. Jusqu’à présent, il n’est possible d’utiliser un disque Ultra que conjointement avec des machines virtuelles déployées par le biais de Zones de disponibilité Azure (déploiement zonal). Un disque Ultra ne prend pas en charge les captures instantanées de stockage à ce stade. Contrairement à tous les autres types de stockage, le disque Ultra ne peut pas être utilisé comme disque dur virtuel de base. Un disque Ultra est idéal pour les cas où la charge de travail d’E/S fluctue énormément et que vous souhaitez adapter le débit de stockage déployé ou les IOPS aux modèles de charge de travail de stockage au lieu de dimensionner l’utilisation maximale de la bande passante et des IOPS.


## <a name="azure-netapp-files-anf"></a>Azure NetApp Files (ANF)
[Azure NetApp Files](https://azure.microsoft.com/services/netapp/) est le fruit d'une collaboration entre Microsoft et NetApp dans le but de fournir des partages de fichiers NFS et SMB natifs Azure haute performance. L’objectif est de fournir une bande passante élevée et un stockage à faible latence permettant de réaliser des scénarios de déploiement SGBD et, au fil du temps, d’activer les fonctionnalités opérationnelles courantes du stockage NetApp par le biais d’Azure. Les partages NFS/SMB sont proposés dans trois niveaux de service différents qui différencient le débit de stockage et le prix. Les niveaux de service sont décrits dans l’article [Niveaux de service pour Azure NetApp Files](../../../azure-netapp-files/azure-netapp-files-service-levels.md). Pour les différents types de charge de travail SAP, les niveaux de service suivants sont fortement recommandés :

- Charge de travail SGBD SAP :    Performances, idéalement Ultra
- Partage SAPMNT :         Performances, idéalement Ultra
- Répertoire de transport global : Performances, idéalement Ultra

> [!NOTE]
> La taille minimale d’approvisionnement est une unité de 4 Tio appelée pool de capacités. Vous créez ensuite des volumes en dehors de ce pool de capacités. Tandis que le plus petit volume que vous pouvez générer est de 100 Gio. Vous pouvez étendre un pool de capacités par incréments de Tio. Pour connaître la tarification, consultez l'article [Tarification Azure NetApp Files](https://azure.microsoft.com/pricing/details/netapp/)

Le stockage ANF est actuellement pris en charge pour plusieurs scénarios de charge de travail SAP :

- Fourniture de partages SMB ou NFS pour le répertoire de transport global SAP
- Le partage sapmnt dans les scénarios de haute disponibilité, comme décrit dans :
    - [Haute disponibilité pour SAP NetWeaver sur des machines virtuelles Azure sur Windows avec Azure NetApp Files (SMB) pour les applications SAP](./high-availability-guide-windows-netapp-files-smb.md)
    - [Haute disponibilité pour SAP NetWeaver sur les machines virtuelles Azure sur SUSE Linux Enterprise Server avec Azure NetApp Files pour les applications SAP](./high-availability-guide-suse-netapp-files.md)
    - [Haute disponibilité des machines virtuelles Azure pour SAP NetWeaver sur Red Hat Enterprise Linux avec Azure NetApp Files pour les applications SAP](./high-availability-guide-rhel-netapp-files.md)
- Déploiements SAP HANA utilisant des partages NFS v4.1 pour des volumes /hana/data et /hana/log et/ou des volumes NFS v4.1 ou NFS v3 pour des volumes /hana/shared comme documenté dans l’article [Configurations de stockage des machines virtuelles Azure SAP HANA](./hana-vm-operations-storage.md)

> [!NOTE]
> Aucune autre charge de travail SGBD n’est prise en charge pour les partages NFS ou SMB basés sur Azure NetApp Files. Les mises à jour et les modifications seront fournies en cas de changement.

Comme c'est déjà le cas pour le stockage premium Azure, une taille de débit fixe ou linéaire par Go peut poser problème lorsque vous devez respecter un nombre minimal de débits. Comme c'est le cas pour SAP HANA. Avec ANF, ce problème peut s’accentuer davantage qu'avec le disque premium Azure. Dans le cas d’un disque Azure Premium, vous pouvez utiliser plusieurs disques plus petits avec un débit relativement élevé par Gio et les entrelacer pour obtenir une rentabilité et un débit plus élevé avec une capacité inférieure. Ce type d’entrelacement ne fonctionne pas pour les partages NFS ou SMB hébergés sur ANF. Cette restriction a entraîné le déploiement de surcapacités :

- Par exemple, pour atteindre un débit de 250 Mio/s sur un volume NFS hébergé sur ANF, vous devez déployer une capacité de 1,95 Tio du niveau de service Ultra. 
- Pour atteindre 400 Mio/s, il faudrait déployer une capacité de 3,125 Tio. Mais vous aurez peut-être besoin de la capacité d’approvisionnement excédentaire pour atteindre le débit dont vous avez besoin pour le volume. Ce sur-approvisionnement de capacité a un impact sur la tarification des instances HANA plus petites. 
- Dans l’espace de l’utilisation de NFS au-dessus de ANF pour le répertoire SAP /sapmnt, vous vous retrouvez généralement à la capacité minimale de 100 Gio à 150 Gio appliquée par Azure NetApp Files. Toutefois, le client a montré que le débit associé de 12,8 Mio/s (à l’aide du niveau de service Ultra) peut ne pas être suffisant et avoir un impact négatif sur la stabilité du système SAP. Dans ce cas, les clients peuvent éviter des problèmes en augmentant le volume /sapmnt, ce qui signifie qu’un débit supérieur est fourni à ce volume.

La matrice de capacité pour la charge de travail SAP ressemble à ceci :

| Fonctionnalité| Commentaire| Remarques/liens | 
| --- | --- | --- | 
| Disque dur virtuel de base du système d’exploitation | ne fonctionne pas | - |
| Disque de données | approprié | SAP HANA uniquement  |
| Répertoire de transport global SAP | YES | SMB et NFS |
| SAP sapmnt | approprié | tous les systèmes SMB (Windows uniquement) ou NFS (Linux uniquement) |
| Stockage de sauvegarde | approprié | - |
| Partages/disque partagé | YES | SMB 3.0, NFS v3 et NFS v4.1 |
| Résilience | LRS | Aucun GRS ou ZRS n’est disponible pour les disques |
| Latence | très faible | - |
| Contrat de niveau de service IOPS | YES | - |
| IOPS linéaires à la capacité | strictement linéaire  | Dépend du [niveau de service](../../../azure-netapp-files/azure-netapp-files-service-levels.md) |
| SLA de débit | YES | - |
| Débit linéaire par rapport à la capacité | semi-linéaire entre crochets | Dépend du [niveau de service](../../../azure-netapp-files/azure-netapp-files-service-levels.md) |
| Certifié HANA | YES | - |
| Captures instantanées de disque possibles | YES | - |
| Captures instantanées de machines virtuelles Sauvegarde Azure possibles | Non | - |
| Coûts | Supérieur au stockage Premium | - |


Fonctionnalités intégrées supplémentaires du stockage ANF :

- Possibilité d’effectuer des captures instantanées du volume
- Clonage de volumes ANF à partir de captures instantanées
- Restaurer des volumes à partir de captures instantanées (réinitialisation du snap)

**Résumé**: Azure NetApp Files est un stockage à faible latence certifié HANA qui permet de déployer des volumes ou des partages NFS et SMB. Le stockage est fourni avec trois niveaux de service différents qui fournissent des débits et IOPS différents de manière linéaire par capacité du volume en Gio. Le stockage ANF permet de déployer des scénarios de scale-out SAP HANA avec un nœud de secours. Le stockage convient pour fournir des partages de fichiers en fonction des besoins pour /sapmnt ou le répertoire de transport global SAP. Le stockage ANF s’accompagne d’une disponibilité des fonctionnalités proposées en tant que fonctionnalités natives de NetApp.  



## <a name="azure-standard-ssd-storage"></a>Stockage SSD Standard Azure
Par rapport aux disques HDD Standard Azure, les disques SSD Standard Azure offrent une disponibilité, une cohérence, une fiabilité et une latence meilleures. Ce stockage est optimisé pour les charges de travail qui nécessitent une performance constante aux niveaux d’IOPS inférieurs. Il s’agit du stockage minimal utilisé pour les systèmes SAP hors production qui présentent des demandes d’IOPS et de débit faibles. La matrice de capacité pour la charge de travail SAP ressemble à ceci :

| Fonctionnalité| Commentaire| Remarques/liens | 
| --- | --- | --- | 
| Disque dur virtuel de base du système d’exploitation | restreint adapté | systèmes hors production |
| Disque de données | restreint adapté | certains systèmes hors production avec des demandes d’IOPS faible et de latence |
| Répertoire de transport global SAP | Non | [Non pris en charge](https://launchpad.support.sap.com/#/notes/2015553) |
| SAP sapmnt | restreint adapté | systèmes hors production |
| Stockage de sauvegarde | approprié | - |
| Partages/disque partagé | non disponible | A besoin d’un tiers |
| Résilience | LRS, GRS | Aucun ZRS disponible pour les disques |
| Latence | high | trop élevé pour le répertoire de transport global SAP ou les systèmes de production |
| Contrat de niveau de service IOPS | Non | - |
| Maximum d’E/S par seconde par disque | 500 | Indépendamment de la taille du disque |
| SLA de débit | Non | - |
| Certifié HANA | Non | - |
| Captures instantanées de disque possibles | YES | - |
| Captures instantanées de machines virtuelles Sauvegarde Azure possibles | YES | - |
| Coûts | LOW | - |



**Résumé :** Le stockage SSD standard Azure est la recommandation minimale pour les machines virtuelles hors production pour le disque dur virtuel de base, les déploiements SGBD éventuels avec une insensibilité relative à la latence et/ou des taux d’IOPS et de débit faibles. Ce type de stockage Azure n’est plus pris en charge pour l’hébergement du répertoire de transport global SAP. 



## <a name="azure-standard-hdd-storage"></a>Stockage HDD Standard Azure
Le stockage HDD Standard Azure était le seul type de stockage lorsque l'infrastructure Azure était certifiée pour la charge de travail SAP NetWeaver au cours de l’année 2014. En 2014, les machines virtuelles Azure étaient peu volumineuses et modestes en terme de débit du stockage. Par conséquent, ce type de stockage était en mesure de suivre les demandes. Le stockage est idéal pour les charges de travail non sensibles à la latence, que vous ne pouvez pas rencontrer dans l’espace SAP. Avec le débit croissant des machines virtuelles Azure et l’augmentation de la charge de travail que ces machines virtuelles produisent, ce type de stockage n’est plus pris en compte pour l’utilisation avec les scénarios SAP. La matrice de capacité pour la charge de travail SAP ressemble à ceci :

| Fonctionnalité| Commentaire| Remarques/liens | 
| --- | --- | --- | 
| Disque dur virtuel de base du système d’exploitation | non approprié | - |
| Disque de données | non approprié | - |
| Répertoire de transport global SAP | Non | [Non pris en charge](https://launchpad.support.sap.com/#/notes/2015553) |
| SAP sapmnt | Non | Non pris en charge |
| Stockage de sauvegarde | approprié | - |
| Partages/disque partagé | non disponible | Nécessite Azure Files ou un fournisseur tiers |
| Résilience | LRS, GRS | Aucun ZRS disponible pour les disques |
| Latence | high | trop élevé pour l’utilisation du SGBD, le répertoire de transport global SAP ou sapmnt/saploc |
| Contrat de niveau de service IOPS | Non | - |
| Maximum d’E/S par seconde par disque | 500 | Indépendamment de la taille du disque |
| SLA de débit | Non | - |
| Certifié HANA | Non | - |
| Captures instantanées de disque possibles | YES | - |
| Captures instantanées de machines virtuelles Sauvegarde Azure possibles | YES | - |
| Coûts | LOW | - |



**Résumé :** HDD Standard est un type de stockage Azure qui ne doit être utilisé que pour stocker des sauvegardes SAP. Il ne doit pas être utilisé qu’en tant que disque virtuel de base pour les systèmes inactifs, comme les systèmes retirés utilisés pour la recherche de données ici ou là. Cependant, aucune machine virtuelle de développement, d’assurance qualité ou de production active ne doit être basée sur ce stockage. Les fichiers de base de données ne doivent pas non plus être hébergés sur ce stockage


## <a name="azure-vm-limits-in-storage-traffic"></a>Limites des machines virtuelles Azure dans le trafic de stockage
À l’inverse des scénarios locaux, le type de machine virtuelle que vous sélectionnez joue un rôle essentiel dans la bande passante de stockage que vous pouvez atteindre. Pour les différents types de stockage, vous devez prendre en compte les éléments suivants :

| Type de stockage| Linux | Windows | Commentaires |
| --- | --- | --- | --- |
| HDD Standard | [Tailles des machines virtuelles Linux dans Azure](../../sizes.md) | [Tailles des machines virtuelles Windows dans Azure](../../sizes.md) | Il est probablement difficile d’atteindre les limites de stockage des machines virtuelles moyennes ou grandes |
| SSD Standard | [Tailles des machines virtuelles Linux dans Azure](../../sizes.md) | [Tailles des machines virtuelles Windows dans Azure](../../sizes.md) | Il est probablement difficile d’atteindre les limites de stockage des machines virtuelles moyennes ou grandes |
| Stockage Premium | [Tailles des machines virtuelles Linux dans Azure](../../sizes.md) | [Tailles des machines virtuelles Windows dans Azure](../../sizes.md) | Limites d’IOPS ou de débit de stockage facilement atteintes par la machine virtuelle avec la configuration du stockage |
| Stockage sur disque Ultra | [Tailles des machines virtuelles Linux dans Azure](../../sizes.md) | [Tailles des machines virtuelles Windows dans Azure](../../sizes.md) | Limites d’IOPS ou de débit de stockage facilement atteintes par la machine virtuelle avec la configuration du stockage |
| Azure NetApp Files | [Tailles des machines virtuelles Linux dans Azure](../../sizes.md) | [Tailles des machines virtuelles Windows dans Azure](../../sizes.md) | Le trafic de stockage utilise la bande passante du débit réseau et non la bande passante du stockage. |

Vous pouvez noter les limitations suivantes :

- Plus la machine virtuelle est petite, plus le nombre de disques que vous pouvez joindre est réduit. Cela ne s’applique pas à ANF. Étant donné que vous montez des partages NFS ou SMB, le nombre de volumes partagés à joindre est illimité.
- Les machines virtuelles ont des limites de débit et d’IOPS qui peuvent facilement être dépassées avec des disques de stockage Premium et des disques Ultra
- Avec ANF, le trafic vers les volumes partagés consomme la bande passante réseau de la machine virtuelle et non la bande passante du stockage
- Avec des volumes NFS importants dans l'espace de capacité exprimé en Tio sur deux chiffres, le débit d'accès à un tel volume à partir d'une seule machine virtuelle va atteindre un plateau basé sur les limites de Linux pour une seule session interagissant avec le volume partagé. 

Au fur et à mesure que vous augmentez la taille des machines virtuelles Azure dans le cycle de vie d’un système SAP, vous devez évaluer les limites de débit de stockage et d’IOPS du nouveau type de machine virtuelle. Dans certains cas, il peut être judicieux d’ajuster la configuration du stockage aux nouvelles fonctionnalités de la machine virtuelle Azure. 


## <a name="striping-or-not-striping"></a>Agrégation par bandes ou non ?
La création d’un agrégat par bandes à partir de plusieurs disques Azure dans un volume de plus grande taille vous permet d’accumuler les IOPS et le débit des disques individuels en un seul volume. Elle est utilisée pour le stockage standard Azure et le stockage Premium Azure uniquement. Le disque Azure Ultra, dont vous pouvez configurer le débit et les IOPS indépendamment de la capacité du disque, ne nécessite pas l'utilisation de jeux de bandes. Les volumes partagés basés sur NFS ou SMB ne peuvent pas être agrégés par bandes. En raison de la nature non linéaire du débit et des IOPS du stockage Premium Azure, vous pouvez approvisionner une capacité inférieure avec les mêmes IOPS et le même débit que les grands disques de stockage Premium Azure. C’est la méthode qui permet d’obtenir un débit plus élevé ou des IOPS plus élevées à moindre coût en utilisant le stockage Premium Azure. À titre d’exemple, l’entrelacement sur deux disques de stockage Premium P15 vous permet d’obtenir un débit de 

- 250 Mio/s. Un tel volume aura une capacité de 512 Gio. Si vous souhaitez disposer d’un disque unique qui offre un débit de 250 Mio par seconde, vous devez choisir un disque P40 avec une capacité de 2 Tio. 
- 400 Mio/s en distribuant quatre disques de stockage Premium P10 avec une capacité globale de 512 Gio par entrelacement. Si vous souhaitez disposer d’un disque unique avec un débit de 500 Mio minimum par seconde, vous devez choisir un disque de stockage Premium P60 avec 8 Tio. Étant donné que le coût du stockage Premium est quasiment linéaire avec la capacité, l’entrelacement vous permet de constater rapidement les économies réalisées.

Certaines règles doivent être suivies en cas d’entrelacement :

- Aucun stockage configuré dans la machine virtuelle ne doit être utilisé, car le stockage Azure conserve déjà les données redondantes
- Les disques sur lesquels le jeu de bandes est appliqué doivent être de la même taille

L’entrelacement sur plusieurs disques plus petits est la meilleure façon d’obtenir un rapport prix/performances optimal à l’aide du Stockage Premium Microsoft Azure. Il est entendu que l’entrelacement induit des frais de déploiement et de gestion supplémentaires.

Pour obtenir des recommandations spécifiques sur la taille des bandes, consultez la documentation des différents SGBD, comme [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md).




## <a name="next-steps"></a>Étapes suivantes
Lisez les articles :

- [Facteurs à prendre en compte pour le déploiement SGBD des machines virtuelles Azure pour la charge de travail SAP](./dbms_guide_general.md)
- [Configurations du stockage des machines virtuelles SAP HANA Azure](./hana-vm-operations-storage.md)

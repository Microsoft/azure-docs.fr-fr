---
title: Déplacement du stockage du mainframe vers Stockage Azure
description: Les ressources de stockage Azure hautement scalable peuvent aider des organisations basées sur un mainframe à migrer et à moderniser les applications IBM z14.
author: njray
ms.author: larryme
ms.date: 04/02/2019
ms.topic: article
ms.service: storage
ms.openlocfilehash: 8f57210ff6b65ee3a893fb344a48629466e90004
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97095353"
---
# <a name="move-mainframe-storage-to-azure"></a>Déplacement du stockage du mainframe vers Azure

Pour exécuter des charges de travail mainframe dans Microsoft Azure, vous devez pouvoir établir une comparaison entre les fonctionnalités de votre mainframe et celles d’Azure. Les ressources de stockage hautement scalable peuvent aider les organisations à commencer à se moderniser sans abandonner les applications dont elles dépendent.

Azure fournit des fonctionnalités de type mainframe et une capacité de stockage qui est comparable aux systèmes basés sur IBM z14 (le modèle le plus récent à ce jour). Cet article vous indique comment obtenir des résultats comparables sur Azure.

## <a name="mainframe-storage-at-a-glance"></a>Stockage du mainframe en un clin d’œil

Le mainframe IBM caractérise le stockage de deux manières. La première est un appareil de stockage à accès direct (DASD). La deuxième est le stockage séquentiel. Pour la gestion du stockage, le mainframe fournit le sous-système DFSMS (Data Facility Storage Management Subsystem). Il gère l’accès aux données sur les différents appareils de stockage.

[DASD](https://en.wikipedia.org/wiki/Direct-access_storage_device) fait référence à un appareil distinct pour le stockage secondaire (hors mémoire) qui permet d’utiliser une adresse unique pour l’accès direct des données. À l’origine, le terme DASD s’appliquait aux disques rotatifs, aux tambours magnétiques ou aux cellules de données. Toutefois, maintenant le terme peut aussi s’appliquer aux appareils SSD, aux réseaux SAN, au stockage NAS et aux lecteurs optiques. Dans le cadre de ce document, DASD fait référence aux disques rotatifs, aux réseaux SAN et aux disques SSD.

Contrairement au stockage DASD, le stockage séquentiel sur un mainframe fait référence aux appareils tels que les lecteurs de bande où les données sont accessible à partir d’un point de départ, puis lues ou écrites sur une ligne.

Les appareils de stockage sont généralement connectés à l’aide d’une connexion fibre (FICON) ou sont accessibles directement sur le bus ES du mainframe à l’aide de [HiperSockets](https://www.ibm.com/support/knowledgecenter/zosbasics/com.ibm.zos.znetwork/znetwork_85.htm), une technologie d’IBM pour les communications à haut débit entre les partitions sur un serveur avec un hyperviseur.

La plupart des systèmes mainframe divisent le stockage en deux types :

- Le *stockage en ligne* (également connu en tant que stockage à chaud) est nécessaire pour les opérations quotidiennes. Le stockage DASD est généralement utilisé à cet effet. Toutefois, le stockage séquentiel, comme les sauvegardes sur bande quotidiennes (logiques ou physiques), peut également être utilisé à cet effet.

- Le *stockage archive* (également connu en tant que stockage à froid) dont le montage à un moment donné n’est pas garanti. Au lieu de cela, il est monté et accessible en fonction des besoins. Le stockage archive est souvent implémenté à l’aide de sauvegardes sur bande séquentielles (logiques ou physiques) pour le stockage.

## <a name="mainframe-versus-io-latency-and-iops"></a>Mainframe, latence d’E/S et IOPS

Les mainframes sont souvent utilisés pour les applications qui nécessitent des performances d’E/S élevées et une faible latence d’E/S. Pour ce faire, ils utilisent des connexions FICON aux appareils E/S et HiperSockets. La technologie HiperSockets permet de connecter des applications et des appareils directement au canal d’E/S d’un mainframe, pour obtenir une latence de quelques microsecondes.

## <a name="azure-storage-at-a-glance"></a>Stockage Azure en un clin d’œil

Les options [IaaS](https://azure.microsoft.com/overview/what-is-iaas/) d’Azure pour le stockage offrent une capacité de mainframe comparable.

Microsoft offre l’équivalent de plusieurs pétaoctets de stockage aux applications hébergées dans Azure, et vous disposez de plusieurs options de stockage. Celles-ci vont du stockage SSD pour de hautes performances au stockage Blob à faible coût pour le stockage de masse et les archives. En outre, Azure fournit une option de redondance des données pour le stockage. Une option qui requiert davantage d’efforts de configuration dans un environnement mainframe.

Le Stockage Azure est disponible en tant que [Disques Azure](../../../managed-disks-overview.md), [Fichiers Azure](../../../../storage/files/storage-files-introduction.md) et [Objets Blob Azure](../../../../storage/blobs/storage-blobs-overview.md) comme le résume le tableau suivant. Découvrez [quand utiliser chaque option](../../../../storage/common/storage-introduction.md).

<!-- markdownlint-disable MD033 -->

<table>
<thead>
    <tr><th>Type</th><th>Description</th><th>À utiliser quand vous voulez :</th></tr>
</thead>
<tbody>
<tr><td>Azure Files
</td>
<td>
Fournit une interface SMB, des bibliothèques clientes et une interface <a href="/rest/api/storageservices/file-service-rest-api">REST</a> qui permet d’accéder en tout lieu aux fichiers stockés.
</td>
<td><ul>
<li>Développer et transférer une application dans le cloud quand celle-ci utilise les API du système de fichiers natif pour partager des données avec d’autres applications s’exécutant dans Azure.</li>
<li>Stocker les outils de développement et de débogage qui doivent être accessibles à partir de nombreuses machines virtuelles.</li>
</ul>
</td>
</tr>
<tr><td>Objets blob Azure
</td>
<td>Fournit des bibliothèques de client et une interface <a href="/rest/api/storageservices/blob-service-rest-api">REST</a> qui permet de stocker des données non structurées et d’y accéder à grande échelle dans des objets blob de blocs. Prend également en charge <a href="/azure/storage/blobs/data-lake-storage-introduction">Azure Data Lake Storage Gen2</a> pour les solutions d’analytique d’entreprise Big Data.
</td>
<td><ul>
<li>Prendre en charge le streaming et l’accès aléatoire dans une application.</li>
<li>Avoir accès aux données de l’application à partir de n’importe quel endroit.</li>
<li>Générer un Data Lake d’entreprise sur Azure et effectuer une analytique de Big Data.</li>
</ul></td>
</tr>
<tr><td>Disques Azure
</td>
<td>Fournit des bibliothèques clientes et une interface <a href="/rest/api/compute/disks">REST</a> qui permet de stocker de manière permanente les données et d’y accéder à partir d’un disque dur virtuel joint.
</td>
<td><ul>
<li>Développer et transférer des applications qui utilisent les API du système de fichiers natif pour lire et écrire des données sur des disques persistants.</li>
<li>Stocker des données dont l’accès n’est pas requis à l’extérieur de la machine virtuelle à laquelle le disque est joint.</li>
</ul></td>
</tr>
</tbody>
</table>
<!-- markdownlint-enable MD033 -->

## <a name="azure-hot-online-and-cold-archive-storage"></a>Stockage Azure à chaud (en ligne) et à froid (archive)

Le type de stockage d’un système donné dépend de la configuration requise du système, notamment la taille de stockage, le débit et IOPS. Pour le stockage de type DASD sur un mainframe, les applications sur Azure utilisent en général le stockage Disques Azure à la place. Pour le stockage archive de mainframe, stockage Blob est utilisé sur Azure.

Les disques SSD offrent les meilleures performances de stockage sur Azure. Les options suivantes sont disponibles (au moment de la rédaction de ce document) :

| Type         | Taille           | E/S par seconde                  |
|--------------|----------------|-----------------------|
| Ultra SSD    | 4 Go à 64 To  | 1 200 à 160 000 IOPS |
| SSD Premium  | 32 Go à 32 To | 12 à 15 000 IOPS     |
| SSD Standard | 32 Go à 32 To | 12 à 2 000 IOPS      |

Le stockage Blob fournit le plus grand volume de stockage sur Azure. En plus de la taille de stockage, Azure propose un stockage managé et non managé. Avec le stockage managé, Azure s’occupe de la gestion des comptes de stockage sous-jacents. Avec le stockage non managé, l’utilisateur assume la responsabilité de configurer des comptes de stockage Azure de la taille appropriée en vue de satisfaire les besoins de stockage.

## <a name="next-steps"></a>Étapes suivantes

- [Migration de mainframe](/azure/architecture/cloud-adoption/infrastructure/mainframe-migration/overview)
- [Réhébergement du mainframe sur des machines virtuelles Azure](../overview.md)
- [Déplacer le calcul mainframe vers Azure](mainframe-compute-Azure.md)
- [Choisir quand utiliser Objets Blob Azure, Fichiers Azure ou Disques Azure](../../../../storage/common/storage-introduction.md)
- [Disques managés SSD Standard pour les charges de travail des machines virtuelles Azure](../../../disks-types.md#standard-ssd)

### <a name="ibm-resources"></a>Ressources IBM

- [Parallel Sysplex sur IBM Z](https://www.ibm.com/it-infrastructure/z/technologies/parallel-sysplex-resources)
- [IBM CICS and the Coupling Facility: Beyond the Basics](https://www.redbooks.ibm.com/redbooks/pdfs/sg248420.pdf)
- [Création des utilisateurs requis pour l’installation d’une fonctionnalité DB2 pureScale](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.qb.server.doc/doc/t0055374.html?pos=2)
- [DB2icrt - Commande Créer une instance](https://www.ibm.com/support/knowledgecenter/en/SSEPGG_11.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0002057.html)
- [Solution de base de données de clusters Db2 pureScale](https://www.ibmbigdatahub.com/blog/db2-purescale-clustered-database-solution-part-1)
- [IBMData Studio](https://www.ibm.com/developerworks/downloads/im/data/index.html/)

### <a name="azure-government"></a>Azure Government

- [Cloud Microsoft Azure Government pour applications mainframe](https://azure.microsoft.com/resources/microsoft-azure-government-cloud-for-mainframe-applications/)
- [Microsoft et FedRAMP](https://www.microsoft.com/TrustCenter/Compliance/FedRAMP)

### <a name="more-migration-resources"></a>Autres ressources de migration

- [Azure Virtual Datacenter: Lift and Shift Guide](https://azure.microsoft.com/resources/azure-virtual-datacenter-lift-and-shift-guide/)
- [iSCSI GlusterFS](https://glusterdocs.readthedocs.io/en/latest/Administrator%20Guide/GlusterFS%20iSCSI/)
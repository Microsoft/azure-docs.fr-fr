---
title: Présentation de Microsoft Azure Data Box Disk | Microsoft Docs sur les données
description: Description d’Azure Data Box Disk, une solution de cloud qui vous permet de transférer de grandes quantités de données dans Azure
services: databox
author: alkohli
ms.service: databox
ms.subservice: disk
ms.topic: overview
ms.date: 06/18/2019
ms.author: alkohli
Customer intent: As an IT admin, I need to understand what Data Box Disk is and how it works so I can use it to import on-premises data into Azure.
ms.openlocfilehash: ea9353ed8c1938fa9b33585a0650b4507c671451
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92125028"
---
# <a name="what-is-azure-data-box-disk"></a>Qu’est-ce qu’Azure Data Box Disk ?

La solution Microsoft Azure Data Box Disk permet d’envoyer des téraoctets de données locales vers Azure de façon rapide, économique et fiable. Le transfert de données sécurisé est accéléré par l’envoi de+ 1 à 5 disques SSD. Ces disques de 8 To chiffrés sont envoyés à votre centre de données par le biais d’un opérateur régional.

Vous pouvez configurer, connecter et déverrouiller rapidement les disques à l’aide du service Data Box du portail Azure. Copiez vos données vers des disques et renvoyez-les vers Azure. Dans le centre de données Azure, vos données sont automatiquement chargées à partir de lecteurs vers le cloud à l’aide d’un lien de chargement de réseau privé et rapide.

## <a name="use-cases"></a>Cas d'utilisation

Utilisez Data Box Disk pour transférer des téraoctets de données dans des scénarios où vous disposez d’aucune connectivité ou d’une connectivité limitée. Il peut s’agir d’un déplacement de données à usage unique, périodique, ou d’un transfert de données en bloc initial suivi de transferts périodiques.

- **Migration unique** - lorsque de grandes quantités de données locales sont transférées vers Azure. Par exemple, un déplacement des données à partir de bandes hors connexion vers des données d’archivage se trouvant dans le stockage froid Azure.
- **Transfert incrémentiel** : lorsqu’un transfert en bloc initial est effectué à l’aide de Data Box Disk (seed) suivi de transferts incrémentiels sur le réseau. Par exemple, Commvault et Data Box Disk sont utilisés pour déplacer des copies de sauvegarde vers Azure. Cette migration est suivie d’une copie des données incrémentielles par le biais du réseau vers le stockage Azure.
- **Chargements périodiques** - lorsqu’une grande quantité de données est générée régulièrement et doit être déplacée vers Azure. Par exemple, dans le secteur de la production d’énergie, où du contenu vidéo est généré sur des plateformes pétrolières et dans des parcs éoliens.

### <a name="ingestion-of-data-from-data-box"></a>Ingestion des données de la Data Box

Les fournisseurs Azure et non-Azure peuvent ingérer des données d’Azure Data Box. Les services Azure permettant l’ingestion des données d’Azure Data Box sont les suivants :

- **SharePoint Online** : utilisez Azure Data Box et l’Outil de migration SharePoint (SPMT) pour migrer le contenu de votre partage de fichiers vers SharePoint Online. Data Box vous permet de ne plus dépendre de votre liaison WAN pour transférer les données. Pour plus d’informations, consultez [Utiliser Azure Data Box Heavy pour migrer le contenu de votre partage de fichiers vers SharePoint Online](data-box-heavy-migrate-spo.md).

- **Azure File Sync** : réplique les fichiers de votre Data Box vers un partage de fichiers Azure, ce qui vous permet de centraliser vos services de fichiers dans Azure tout en conservant un accès local à vos données. Pour plus d’informations, consultez [Déployer Azure File Sync](../storage/files/storage-sync-files-deployment-guide.md).

- **Magasins HDFS** : migrez des données d’un magasin HDFS (Hadoop Distributed File System) local de votre cluster Hadoop vers un stockage Azure à l’aide de Data Box. Pour plus d’informations, consultez [Migrer un magasin HDFS local vers Stockage Azure à l’aide d’Azure Data Box](../storage/blobs/data-lake-storage-migrate-on-premises-hdfs-cluster.md).

- **Sauvegarde Azure** : vous permet de déplacer grosses sauvegardes de données d’entreprise critiques à l’aide de mécanismes hors connexion vers un coffre Azure Recovery Services. Pour plus d’informations, consultez [Vue d’ensemble du service Sauvegarde Azure](../backup/backup-overview.md).

Vous pouvez utiliser vos données Data Box avec de nombreux fournisseurs de services non-Azure. Exemple :

- **[Commvault](http://documentation.commvault.com/commvault/v11/article?p=97276.htm)**  : vous permet de migrer de grands volumes de données vers Microsoft Azure à l’aide d’Azure Data Box.
- **[Veeam](https://helpcenter.veeam.com/docs/backup/hyperv/osr_adding_data_box.html?ver=100)**  : vous permet de sauvegarder et de répliquer de grandes quantités de données à partir de votre machine Hyper-V vers votre Data Box.

Pour obtenir la liste des fournisseurs de services non-Azure qui s’intègrent à Data Box, consultez le [tableau des partenaires Azure Data Box](https://cloudchampions.blob.core.windows.net/db-partners/PartnersTable.pdf).

## <a name="the-workflow"></a>Workflow

Un flux type inclut les étapes suivantes :

1. **Commande** : créez une commande dans le portail Azure et indiquez les informations d’expédition et le compte de stockage de destination Azure de vos données. Si les disques sont disponibles, Azure chiffre prépare et envoie les disques avec un ID de suivi d’expédition.

2. **Réception** - Une fois que les disques livrés, décompressez et connectez-les à l’ordinateur contenant les données à copier. Déverrouillez les disques.

3. **Copie des données** - Glissez-déplacez les données à copier sur les disques.

4. **Renvoi** - Préparez et expédiez les disques vers le centre de données Azure.

5. **Chargement** - Les données sont automatiquement copiées des disques vers Azure. Les disques sont effacés en toute sécurité conformément aux instructions du National Institute of Standards and Technology (NIST).

Tout au long de ce processus, vous êtes averti par courrier électronique de toutes les modifications d’état. Pour plus d’informations sur le flux détaillé, accédez à [Deploy Data Box Disks in Azure portal](data-box-disk-quickstart-portal.md) (Déployer des Data Box Disks dans le portail Azure).

## <a name="benefits"></a>Avantages

Data Box Disk est conçu pour déplacer de grandes quantités de données vers Azure sans aucun impact sur le réseau. La solution offre les avantages suivants :

- **Vitesse** - Data Box Disk utilise une connexion USB 3.0 pour déplacer jusqu’à 35 To de données vers Azure en moins d’une semaine.

- **Facilité d’utilisation** - Data Box est une solution facile à utiliser.

  - Les disques utilisent une connectivité USB qui ne nécessite pratiquement aucun temps d’installation.
  - Les disques sont compacts, ce qui les rend faciles à gérer.
  - Les disques n’ont pas besoin d’alimentation externe.
  - Les disques peuvent être utilisés avec un serveur de centre de données, un ordinateur de bureau ou un ordinateur portable.
  - La solution offre un suivi de bout en bout avec le portail Azure.

- **Sécurité** - Data Box Disk intègre des protections de sécurité pour les disques, les données et le service.
  - Les disques sont inviolables et prennent en charge la fonctionnalité de mise à jour sécurisée.
  - Les données sur les disques sont sécurisées en permanence à l’aide d’un chiffrement AES 128 bits.
  - Les disques peuvent uniquement être déverrouillés à l’aide d’une clé fournie dans le portail Azure.
  - Le service est protégé par les fonctionnalités de sécurité Azure.
  - Une fois que vos données sont chargées vers Azure, les disques sont nettoyés, conformément aux normes NIST 800-88r1.  

Pour plus d’informations, accédez à [Azure Data Box Disk security and data protection](data-box-disk-security.md) (Protection des données et sécurité Azure Data Box Disk).

## <a name="features-and-specifications"></a>Fonctionnalités et spécifications

| Spécifications                                          | Description              |
|---------------------------------------------------------|--------------------------|
| Poids                                                  | < 2 lbs. par boîte. Jusqu’à 5 disques dans la boîte                |
| Dimensions                                              | Disque - disque SSD de 2,5" |
| Câbles                                                  | 1 câble USB 3.1 par disque|
| Capacité de stockage par commande                              | 40 To (environ 35 To utilisables)|
| Capacité stockage du disque                                   | 8 To (environ 7 To utilisables)|
| Interface de données                                          | USB   |
| Sécurité                                                | Préalablement chiffré à l’aide de BitLocker et mise à jour sécurisée <br> Disques protégés par clé d’accès <br> Données chiffrées en permanence  |
| Vitesse de transfert de données                                      | jusqu’à 430 Mo/s en fonction de la taille du fichier      |
|Gestion                                               | Portail Azure |

## <a name="region-availability"></a>Disponibilité des régions

Pour plus d’informations sur la disponibilité régionale, accédez à [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?products=databox&regions=all). Le déploiement de Data Box Disk est également possible dans Azure Government Cloud. Pour plus d’informations, consultez [Présentation d’Azure Government](../azure-government/documentation-government-welcome.md).

## <a name="pricing"></a>Tarifs

Pour plus d’informations sur les tarifs, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/databox/disk/).

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [Conditions requises pour le système Data Box Disk](data-box-disk-system-requirements.md).
- Prenez connaissance des [limites de Data Box Disk](data-box-disk-limits.md).
- Déployez rapidement [Azure Data Box Disk](data-box-disk-quickstart-portal.md) sur le portail Azure.
---
title: Cas d’utilisation pour Microsoft Azure Data Box Gateway | Microsoft Docs
description: Décrit les cas d’usage pour Azure Data Box Gateway, une solution de stockage d’appliances virtuelles qui vous permet de transférer des données vers Azure.
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: article
ms.date: 10/14/2020
ms.author: alkohli
ms.openlocfilehash: f6daee6d4cfc3c074e004fb3835f62218e48d9ff
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96580981"
---
# <a name="use-cases-for-azure-data-box-gateway"></a>Cas d’utilisation pour Azure Data Box Gateway

Azure Data Box Gateway est un dispositif de passerelle de stockage cloud qui réside localement et envoie vos données image, multimédias et autres vers Azure. Cette passerelle de stockage cloud est une machine virtuelle provisionnée dans votre hyperviseur. Vous écrivez des données sur ce dispositif virtuel en utilisant les protocoles NFS et SMB, qui les envoient ensuite vers Azure. Cet article fournit une description détaillée des scénarios où vous pouvez déployer ce dispositif.

Utilisez Data Box Gateway pour les scénarios suivants :

- Pour ingérer de manière continue d’énormes quantités de données
- Pour l’archivage des données dans le cloud de manière sécurisée et efficace
- Pour le transfert de données incrémentielles sur le réseau après qu’un transfert en bloc a été effectué à l’aide de Data Box

Chacun de ces scénarios est décrit en détail dans les sections qui suivent.


## <a name="continuous-data-ingestion"></a>Ingestion continue des données

L’un des principaux avantages de Data Box Gateway est la possibilité d’ingérer des données en continu dans le dispositif en vue de leur copie dans le cloud, quelle que soit la taille des données.

À mesure que les données sont écrites sur le dispositif de passerelle, celui-ci charge les données vers Stockage Azure. Le dispositif gère automatiquement le stockage en supprimant les fichiers localement (tout en conservant les métadonnées) quand il atteint un certain seuil. La conservation d’une copie locale des métadonnées permet au dispositif de passerelle de charger uniquement les modifications quand le fichier est mis à jour. Les données chargées sur votre dispositif de passerelle doivent être conformes aux instructions figurant dans [Mises en garde relatives au chargement de données](data-box-gateway-limits.md#data-upload-caveats).

À mesure que le dispositif est rempli avec des données, il commence à limiter le débit d’entrée (si nécessaire) afin qu’il corresponde au débit auquel les données sont chargées vers le cloud. Pour superviser l’ingestion en continu sur le dispositif, vous utilisez des alertes. Ces alertes sont générées une fois que la limitation a commencé, et supprimées une fois qu’elle a cessé.

## <a name="cloud-archival-of-data"></a>Archivage des données dans le cloud

Utilisez Data Box Gateway quand vous souhaitez conserver vos données à long terme dans le cloud. Vous pouvez utiliser le niveau de stockage Archive pour la conservation à long terme.

Le niveau Archive est optimisé pour stocker les données qui sont rarement sollicitées pendant au moins 180 jours. Le niveau Archive offre les coûts de stockage les plus bas, mais a les coûts d’accès les plus élevés. Pour plus d’informations, consultez [Niveau d’accès archive](/azure/storage/blobs/storage-blob-storage-tiers#archive-access-tier).

### <a name="move-data-to-the-archive-tier"></a>Déplacer des données vers le niveau Archive

Avant de commencer, vérifiez que vous avez un dispositif Data Box Gateway en cours d’exécution. Suivez les étapes détaillées dans [Tutoriel : Préparer le déploiement d’Azure Data Box Gateway](data-box-gateway-deploy-prep.md) et continuez avec les tutoriels suivants jusqu’à ce que vous ayez un dispositif opérationnel.

- Utilisez le dispositif Data Box Gateway pour charger des données vers Azure à l’aide de la procédure de transfert habituelle décrite dans [Transférer des données avec Azure Data Box Gateway](data-box-gateway-deploy-add-shares.md).
- Une fois les données chargées, vous devrez les déplacer vers le niveau Archive. Vous pouvez définir le niveau du blob de deux manières : à l’aide d’un script Azure PowerShell ou d’une stratégie de gestion de cycle de vie Stockage Azure.  
    - Si vous utilisez Azure PowerShell, suivez ces [étapes](/azure/databox/data-box-how-to-set-data-tier#use-azure-powershell-to-set-the-blob-tier) pour déplacer les données vers le niveau Archive.
    - Si vous utilisez la gestion de cycle de vie Azure, suivez ces étapes pour déplacer les données vers le niveau Archive.
        - [Inscrivez-vous](/azure/storage/common/storage-lifecycle-management-concepts) à la préversion du service de gestion de cycle de vie de blob afin d’utiliser le niveau Archive.
        - Utiliser la stratégie suivante pour [archiver des données au moment de l’ingestion](/azure/storage/blobs/storage-lifecycle-management-concepts#archive-data-after-ingest).
- Une fois les blobs marqués comme Archive, ils ne peuvent plus être modifiés par la passerelle, sauf s’ils sont déplacés vers le niveau chaud ou froid. Si le fichier est dans le stockage local, aucune des modifications apportées à la copie locale (y compris les suppressions) n’est chargée vers le niveau Archive.
- Pour lire des données dans le stockage Archive, vous devez les réhydrater en basculant le niveau du blob sur chaud ou froid. [L’actualisation du partage](data-box-gateway-manage-shares.md#refresh-shares) sur la passerelle ne réactive pas le blob.

Pour plus d’informations, découvrez en détail comment [gérer le cycle de vie de Stockage Blob Azure](/azure/storage/common/storage-lifecycle-management-concepts).

## <a name="initial-bulk-transfer-followed-by-incremental-transfer"></a>Transfert en bloc initial suivi de transfert incrémentiel

Utilisez Data Box avec Data Box Gateway quand vous souhaitez effectuer un chargement en bloc d’une grande quantité de données, suivi de transferts incrémentiels. Utilisez Data Box pour le transfert en bloc dans un mode hors connexion (amorçage initiale) et Data Box Gateway pour les transferts incrémentiels (flux en cours) par le biais du réseau.

### <a name="seed-the-data-with-data-box"></a>Amorcer les données avec Data Box

Suivez ces étapes pour copier les données dans Data Box et les charger dans Stockage Azure.

1. [Commandez votre Data Box](/azure/databox/data-box-deploy-ordered).
2. [Configurez votre Data Box](/azure/databox/data-box-deploy-set-up).
3. [Copiez des données sur Data Box par le biais de SMB](/azure/databox/data-box-deploy-copy-data).
4. [Renvoyez la Data Box et vérifiez le chargement des données sur Azure](/azure/databox/data-box-deploy-picked-up).
5. Une fois le chargement des données sur Azure terminé, toutes les données doivent être dans des conteneurs de stockage Azure. Dans le compte de stockage pour Data Box, accédez au conteneur Blob (et Fichier) pour vérifier que toutes les données ont été copiées. Notez le nom du conteneur, car vous l’utiliserez plus tard. Par exemple, dans la capture d’écran suivante, le conteneur `databox` sera utilisé pour le transfert incrémentiel.

    ![Conteneur avec des données sur Data Box](media/data-box-gateway-use-cases/data-container.png)

Ce transfert en bloc termine la phase d’amorçage initiale.

### <a name="ongoing-feed-with-data-box-gateway"></a>Flux en cours avec Data Box Gateway

Suivez ces étapes pour l’ingestion en continu par Data Box Gateway. 

1. Créez un partage cloud sur Data Box Gateway. Ce partage charge automatiquement toutes les données sur le compte Stockage Azure. Accédez à **Partages** dans votre ressource Data Box Gateway et cliquez sur **+ Ajouter un partage**.

    ![Cliquez sur + Ajouter un partage](media/data-box-gateway-use-cases/add-share.png)

2. Vérifiez que ce partage est mappé au conteneur qui contient les données de départ. Pour **Sélectionner un conteneur d’objets blob**, choisissez **Utiliser l’existant** et accédez au conteneur où les données Data Box ont été transférées.

    ![Paramètres de partage](media/data-box-gateway-use-cases/share-settings-select-existing-container.png)

3. Une fois le partage créé, actualisez-le. Cette opération actualise le partage local avec le contenu d’Azure.

    ![Actualisez le partage](media/data-box-gateway-use-cases/refresh-share.png)

    Quand le partage est synchronisé, Data Box Gateway charge les modifications incrémentielles si les fichiers ont été modifiés sur le client.

## <a name="next-steps"></a>Étapes suivantes

- Passer en revue les [Conditions requises pour le système Data Box Gateway](data-box-gateway-system-requirements.md).
- Comprendre les [limites de Data Box Gateway](data-box-gateway-limits.md).
- Déployer [Azure Data Box Gateway](data-box-gateway-deploy-prep.md) sur le portail Azure.
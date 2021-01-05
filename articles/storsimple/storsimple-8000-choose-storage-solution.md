---
title: Options de transfert de données vers Azure à l’aide d’une appliance | Microsoft Docs
description: Découvrez comment choisir l’appliance appropriée pour le transfert de données local vers Azure entre Data Box Edge, Azure File Sync et la gamme StorSimple 8000.
services: storsimple
author: alkohli
ms.service: storsimple
ms.topic: article
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: f2de6dbba2548f0c281df35edf2883d0c00af667
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96571911"
---
# <a name="compare-storsimple-with-azure-file-sync-and-data-box-edge-data-transfer-options"></a>Comparer StorSimple avec les options de transfert de données Azure File Sync et Data Box Edge 

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]
 
Ce document fournit une vue d'ensemble des options disponibles pour le transfert de données locales vers Azure, en comparant les solutions : Data Box Edge, Azure File Sync (AFS) et Série StorSimple 8000.

- **[Data Box Edge](../databox-online/azure-stack-edge-overview.md)**  : la solution Data Box Edge consiste en un appareil réseau local permettant d’échanger des données avec Azure, et disposant d’une capacité de computing en périphérie utilisant l’intelligence artificielle pour prétraiter les données durant leurs chargement. Data Box Gateway est une version virtuelle de l’appareil, offrant les mêmes fonctionnalités de transfert de données.
- **[Azure File Sync](../storage/files/storage-sync-files-deployment-guide.md)**  : Azure File Sync permet de centraliser les partages de fichiers de votre organisation dans Azure Files, tout en conservant la flexibilité, le niveau de performance et la compatibilité d’un serveur de fichiers local. Azure File Sync transforme Windows Server en un cache rapide de votre partage de fichiers Azure. La disponibilité générale d’Azure File Sync a été annoncée précédemment en 2018.
- **[StorSimple](./storsimple-overview.md)**  : StorSimple est un appareil hybride qui aide les entreprises à consolider leur infrastructure de stockage pour le stockage principal, la protection des données, l’archivage et la récupération d’urgence sur une solution unique en s’intégrant étroitement avec Stockage Azure. Vous pouvez trouver le cycle de vie du produit pour StorSimple [ici](https://support.microsoft.com/lifecycle/search?alpha=Azure%20StorSimple%208000%20Series).

## <a name="comparison-summary"></a>Résumé de la comparaison

|                           |StorSimple 8000   |Azure File Sync   |Data Box Edge           |
|---------------------------|----------------------------------------|-------------------------------|-----------------------------------------|
|**Vue d'ensemble**     |Stockage et archivage hybrides hiérarchisés|Stockage sur serveur de fichiers général avec hiérarchisation cloud et synchronisation multisite.  |Solution de stockage pour prétraiter les données et les envoyer via réseau vers Azure.        |
|**Scénarios**    |Serveur de fichiers, archivage et cible de sauvegarde |Serveur de fichiers, archivage (multisite)   |Transfert de données, pré-traitement de données incluant inférence de Machine Learning, IoT, archivage    |
|**Computing en périphérie** |Non disponible |Non disponible |Prend en charge l’exécution de conteneurs à l’aide d’Azure IoT Edge    |
|**Facteur de forme**  |Appareil physique   |Agent installé sur Windows Server |Appareil physique   |
|**Matériel**     |Appareil physique fourni par Microsoft dans le cadre du service | Fourni par le client |Appareil physique fourni par Microsoft dans le cadre du service  |
|**Format de données**  |Format personnalisé   |Fichiers         |Objets blob ou fichiers    |
|**Prise en charge du protocole** |iSCSI          |SMB, NFS    | SMB ou NFS      |
|**Tarification**      |[StorSimple](https://azure.microsoft.com/pricing/details/storsimple/) |[Azure File Sync](https://azure.microsoft.com/pricing/details/storage/files/)  |[Data Box Edge](https://azure.microsoft.com/pricing/details/storage/databox/edge/)  |

## <a name="next-steps"></a>Étapes suivantes

- Découvrir [Azure Data Box Edge](../databox-online/azure-stack-edge-overview.md) et [Azure Data Box Gateway](../databox-online/data-box-gateway-overview.md)
- Découvrir [Azure File Sync](../storage/files/storage-sync-files-deployment-guide.md)

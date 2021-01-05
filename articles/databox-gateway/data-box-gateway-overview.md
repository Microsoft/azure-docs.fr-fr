---
title: Présentation de Microsoft Azure Data Box Gateway | Microsoft Docs
description: Décrit Azure Data Box Gateway, une solution de stockage d’appliance virtuelle qui vous permet de transférer des données vers Azure
services: databox
author: alkohli
ms.service: databox
ms.subservice: gateway
ms.topic: overview
ms.date: 08/21/2019
ms.author: alkohli
ms.openlocfilehash: aa68b746ef5c22c97fcfdcca889bf71577c19479
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96583209"
---
# <a name="what-is-azure-data-box-gateway"></a>Présentation d’Azure Data Box Gateway

Azure Data Box Gateway est une solution de stockage qui vous permet d’envoyer en toute transparence des données vers Azure. Cet article vous fournit une vue d’ensemble de la solution Azure Data Box Gateway, ses avantages, ses principales fonctionnalités et les scénarios dans lesquels vous pouvez déployer cet appareil.

Data Box Gateway est un appareil virtuel basé sur une machine virtuelle provisionnée dans votre hyperviseur ou environnement virtualisé. L’appareil virtuel réside dans votre environnement local et vous écrivez des données à l’aide des protocoles NFS et SMB. L’appareil transfère ensuite vos données vers l’objet blob de blocs Azure, l’objet blob de pages ou Azure Files.

## <a name="use-cases"></a>Cas d'utilisation

Data Box Gateway peut servir à transférer des données vers le cloud, par exemple pour l’archivage dans la cloud, la récupération d’urgence, ou lorsqu’il est nécessaire de traiter vos données à l’échelle du cloud. Voici les différents scénarios où Data Box Gateway peut être utilisé pour le transfert de données.

- **Archivage dans le cloud** : copiez des centaines de téraoctets de données vers Stockage Azure à l’aide Data Box Gateway, de manière sécurisée et efficace. Les données peuvent être ingérées une seule fois ou de manière continue pour les scénarios d’archivage.

- **Ingestion continue des données** : ingérez continuellement des données dans l’appareil à copier dans le cloud, quelle que soit la taille des données. À mesure que les données sont écrites sur le dispositif de passerelle, celui-ci charge les données vers Stockage Azure.  

- **Transfert en bloc initial suivi de transfert incrémentiel** : utilisez Data Box pour le transfert en bloc dans un mode hors connexion (amorçage initial), et Data Box Gateway pour les transferts incrémentiels (flux en cours) par le biais du réseau.

Pour plus d’informations, consultez [Cas d’usage Azure Data Box Gateway](data-box-gateway-use-cases.md).

## <a name="benefits"></a>Avantages

Data Box Gateway offre les avantages suivants :

- **Transfert de données facile** : transférez des données vers/depuis Stockage Azure aussi facilement qu’avec un partage réseau local.  
- **Hautes performances** : facilite le transport des données réseau grâce à des transferts hautes performances vers et depuis Azure.
- **Accès rapide et taux d’ingestion de données élevés pendant les heures de bureau** : Data Box Gateway est doté d’un cache local que vous définissez en tant que taille de capacité locale lors du provisionnement de l’appareil virtuel. La taille spécifiée du disque de données doit être conforme aux [exigences de configuration minimale des appareils virtuels](data-box-gateway-system-requirements.md#specifications-for-the-virtual-device). Le cache local offre les avantages suivants :
    - Le cache local permet une ingestion des données à un débit élevé. Quand de grandes quantités de données sont ingérées pendant les heures de pointe, le cache peut contenir les données et les charger dans le cloud.
    - Le cache local permet un accès en lecture rapide jusqu’à un certain seuil. Tant que l’appareil n’est pas plein à plus de 50 à 60 %, toutes les lectures à partir de l’appareil sont accessibles à partir du cache, ce qui les rend plus rapides. Une fois que l’espace utilisé sur l’appareil dépasse ce seuil, l’appareil commence à supprimer des fichiers locaux.
 
- **Utilisation de la bande passante limitée** : les données peuvent être écrites dans Azure même lorsque le réseau est restreint afin de limiter l’utilisation pendant les heures de pointe.  

## <a name="key-capabilities"></a>Fonctionnalités clés

Data Box Gateway intègre les fonctionnalités suivantes :

|Fonctionnalité |Description  |
|---------|---------|
|Vitesse     | Transfert de données et bande passante entièrement automatisés et hautement optimisés.|
|Protocoles pris en charge     | Prise en charge des protocoles SMB et NFS standard pour l’ingestion des données. <br> Pour plus d’informations sur les versions prises en charge, consultez [Conditions requises pour Data Box Gateway](data-box-gateway-system-requirements.md).|
|Accès aux données     | Une fois que les données envoyées par l’appareil sont dans le cloud, elle peuvent être modifiées en accédant directement aux API cloud.|
|Accès rapide     | Cache local sur l’appareil pour un accès rapide aux derniers fichiers utilisés.|
|Chargement hors connexion     | Un mode déconnecté prend en charge les scénarios de chargement hors connexion.|
|Actualisation des données     | Possibilité d’actualiser les fichiers locaux avec la dernière version issue du cloud.|
|Résilience     | Résilience réseau intégrée        |


## <a name="specifications"></a>Spécifications

L’appareil virtuel Data Box Gateway présente les spécifications suivantes :

| Spécifications                                          | Description              |
|---------------------------------------------------------|--------------------------|
| Processeurs virtuels (cœurs)   | 4 minimum |
| Mémoire  |8 Go minimum|
| Disponibilité|Nœud unique|
| Disques|Disque de système d’exploitation : 250 Go <br> Disque de données : 2 To minimum, allocation dynamique et sauvegarde sur SSD|
| Interfaces réseau |1 ou plusieurs interfaces réseau virtuelles|
| Protocoles de partage de fichiers natifs|SMB et NFS  |
| Sécurité|Authentification pour déverrouiller l’accès à l’appareil et aux données <br> Chiffrement des données en transit à l’aide du chiffrement AES 256 bits|
| Gestion|Interface utilisateur web locale : configuration initiale, diagnostics et gestion de l’alimentation de l’appareil <br> Portail Azure : gestion quotidienne des appareils Data Box Gateway       |

## <a name="components"></a>Components

La solution Data Box Gateway se compose d’une ressource Data Box Gateway, d’un appareil virtuel Data Box Gateway et d’une interface utilisateur web locale.

- **Appareil virtuel Data Box Gateway** : un appareil basé sur une machine virtuelle provisionnée dans votre hyperviseur ou environnement virtualisé, qui vous permet d’envoyer des données à Azure.
    
- **Ressource Data Box Gateway** : ressource du portail Azure, qui vous permet de gérer un appareil Data Box Gateway à partir d’une interface web accessible depuis divers emplacements géographiques. Utilisez la ressource Data Box Gateway pour voir et gérer un appareil, des partages, des utilisateurs et des alertes. Pour plus d’informations, découvrez comment [Gérer à l’aide du portail Azure](data-box-gateway-manage-shares.md).

- **Interface utilisateur web locale Data Box** : utilisez l’interface utilisateur web locale pour exécuter des diagnostics, arrêter et redémarrer l’appareil, générer un package de support ou contacter le support Microsoft pour créer une demande de service. Pour plus d’informations, découvrez comment [Gérer à l’aide de l’interface utilisateur web locale](data-box-gateway-manage-access-power-connectivity-mode.md).

## <a name="region-availability"></a>Disponibilité des régions

Il n’est pas nécessaire que l’appareil physique Data Box Gateway, la ressource Azure et le compte de stockage cible vers lequel vous transférez les données se trouvent dans la même région.

- **Disponibilité des ressources** : pour obtenir la liste de toutes les régions où la ressource Azure Data Box Gateway est disponible, consultez [Disponibilité des produits Azure par région](https://azure.microsoft.com/global-infrastructure/services/?regions=all&products=databox). Le déploiement de Data Box Gateway est également possible dans Azure Government Cloud. Pour plus d’informations, consultez [Présentation d’Azure Government](https://docs.microsoft.com/azure/azure-government/documentation-government-welcome).

- **Comptes de stockage de destination** : les comptes de stockage qui stockent les données sont disponibles dans toutes les régions Azure.

    Régions où les comptes de stockage qui stockent les données Data Box doivent se trouver à proximité de l’appareil pour garantir des performances optimales. Un compte de stockage situé dans un emplacement éloigné entraîne des temps de latence longs et une baisse des performances.


## <a name="next-steps"></a>Étapes suivantes

- Passer en revue les [Conditions requises pour le système Data Box Gateway](data-box-gateway-system-requirements.md).
- Comprendre les [limites de Data Box Gateway](data-box-gateway-limits.md).
- Déployer [Azure Data Box Gateway](data-box-gateway-deploy-prep.md) sur le portail Azure.


---
title: Migrer des données à partir d'un lac de données et d'un entrepôt de données vers Azure
description: Utilisez Azure Data Factory pour migrer des données de votre lac de données et de votre entrepôt de données vers Azure.
services: data-factory
author: dearandyxu
ms.author: yexu
ms.reviewer: ''
manager: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 7/30/2019
ms.openlocfilehash: 4408546c892299e5bbbc22b00a4b334c36eda616
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81416431"
---
# <a name="use-azure-data-factory-to-migrate-data-from-your-data-lake-or-data-warehouse-to-azure"></a>Utilisez Azure Data Factory pour migrer des données de votre lac de données ou de votre entrepôt de données vers Azure

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Si vous souhaitez migrer votre lac de données ou votre entrepôt de données d’entreprise (EDW, enterprise data warehouse) vers Microsoft Azure, envisagez d’utiliser Azure Data Factory. Azure Data Factory convient particulièrement aux scénarios suivants :

- Migration de charges de travail Big Data à partir d’Amazon Simple Storage Service (Amazon S3) ou d’un système de fichiers DFS hadoop (HDFS) local vers Azure
- Migration d’un EDW à partir d’Oracle Netezza, Teradata ou Amazon Redshift vers Azure

Azure Data Factory peut déplacer des pétaoctets (Po) de données pour la migration d’un lac de données et des dizaines de téraoctets (To) de données pour la migration d’un entrepôt de données.

## <a name="why-azure-data-factory-can-be-used-for-data-migration"></a>Pourquoi Azure Data Factory peut être utilisé pour la migration de données

- Azure Data Factory peut facilement effectuer un scale-up de la puissance de traitement pour déplacer des données serverless en offrant des performances, une résilience et une scalabilité élevées. De plus, vous payez uniquement pour ce que vous utilisez. En outre, notez les points suivants : 
  - Azure Data Factory n’a aucune limite quant au volume de données et au nombre de fichiers.
  - Azure Data Factory peut utiliser votre réseau et votre bande passante de stockage dans leur intégralité pour obtenir le débit de déplacement de données le plus élevé dans votre environnement.
  - Azure Data Factory utilise une méthode de paiement à l’utilisation. Ainsi, vous payez uniquement pour le temps réellement passé à exécuter la migration des données vers Azure.  
- Azure Data Factory peut effectuer un chargement de données historiques unique et des chargements incrémentiels planifiés.
- Azure Data Factory utilise le runtime d’intégration Azure (IR) pour déplacer des données entre des points de terminaison de lac de données et d’entrepôt de données accessibles publiquement. Il peut également utiliser un runtime d’intégration auto-hébergé pour déplacer des données de points de terminaison de lac de données et d’entrepôt de données au sein d’un réseau virtuel Microsoft Azure ou derrière un pare-feu.
- Azure Data Factory offre une sécurité de niveau entreprise : Vous pouvez utiliser Windows Installer (MSI) ou l’identité du service pour une intégration de service à service sécurisée ou Azure Key Vault pour la gestion des informations d’identification.
- Azure Data Factory offre une expérience de création sans code et un tableau de bord de supervision intégré enrichi.  

## <a name="online-vs-offline-data-migration"></a>Migration des données en ligne et hors connexion

Azure Data Factory est un outil de migration de données en ligne standard permettant de transférer des données sur un réseau (Internet, ER ou VPN). Son approche diffère donc de la migration de données hors connexion (les utilisateurs expédient physiquement des appareils de transfert de données de leur organisation vers un centre de données Azure).  

Vous devez prendre en compte trois points importants quand vous choisissez votre approche de migration (en ligne ou hors connexion) :  

- Taille des données à migrer
- Bande passante réseau
- Fenêtre de migration

Supposons, par exemple, que vous envisagiez d’utiliser Azure Data Factory pour effectuer la migration de vos données sous deux semaines (votre *fenêtre de migration*). Notez la ligne de coupe rose/bleue dans le tableau suivant. La cellule rose la plus basse d’une colonne indique la combinaison taille des données/bande passante réseau dont la fenêtre de migration est la plus proche de deux semaines (tout en restant inférieure). (Toutes les combinaisons taille/bande passante figurant dans une cellule bleue correspondent à une fenêtre de migration en ligne supérieure à deux semaines.) 

![Migration en ligne et migration hors connexion](media/data-migration-guidance-overview/online-offline.png) Ce tableau vous permet de savoir si vous pouvez respecter la fenêtre de migration prévue avec une migration en ligne (Azure Data Factory) en fonction de la taille de vos données et de la bande passante réseau disponible. Si la fenêtre de migration en ligne est supérieure à deux semaines, vous devrez plutôt utiliser la migration hors connexion.

> [!NOTE]
> La migration de données en ligne vous permet de charger les données historiques et les flux incrémentiels de bout en bout à l’aide d’un seul outil.  Grâce à cette approche, vos données peuvent rester synchronisées entre le magasin existant et le nouveau magasin sur l’ensemble de la fenêtre de migration. Ainsi, vous pouvez régénérer votre logique ETL sur le nouveau magasin avec les données actualisées.


## <a name="next-steps"></a>Étapes suivantes

- [Migrer des données d’AWS S3 vers Azure](data-migration-guidance-s3-azure-storage.md)
- [Migrer des données du cluster Hadoop local vers Azure](data-migration-guidance-hdfs-azure-storage.md)
- [Migrer des données du serveur Netezza local vers Azure](data-migration-guidance-netezza-azure-sqldw.md)

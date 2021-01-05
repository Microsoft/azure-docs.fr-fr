---
title: Options de transfert de données Azure pour les jeux de données volumineux avec une bande passante réseau faible ou aucune bande passante réseau | Microsoft Docs
description: Votre environnement dispose d’une bande passante réseau limitée ou il ne dispose d’aucune bande passante réseau et vous souhaitez transférer des jeux de données volumineux ? Apprenez à choisir la solution de transfert de données Azure qui convient.
services: storage
author: alkohli
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: 8afedec224fed55f617340af512485726ff44326
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792937"
---
# <a name="data-transfer-for-large-datasets-with-low-or-no-network-bandwidth"></a>Transférer des jeux de données volumineux avec une bande passante réseau faible ou aucune bande passante réseau
 
Cet article fournit une vue d’ensemble des solutions de transfert de données adaptées aux environnements disposant d’une bande passante réseau faible ou ne disposant d’aucune bande passante réseau et au transfert de jeux de données volumineux. Il décrit également les options de transfert de données recommandées et la matrice de fonctionnalités clés correspondant à ce scénario.

Pour une vue d’ensemble de toutes les options de transfert de données disponibles, voir [Choisir une solution de transfert de données Azure](storage-choose-data-transfer-solution.md).

## <a name="offline-transfer-or-network-transfer"></a>Transfert hors connexion ou transfert réseau

Les jeux de données volumineux impliquent que vous disposiez de plusieurs téraoctets ou de plusieurs pétaoctets de données. Vous disposez d’une bande passante réseau faible ou vous ne disposez d’aucune bande passante réseau, votre réseau est lent ou il n’est pas fiable. De plus :

- Vous êtes limité par les coûts de transfert réseau à partir de vos fournisseurs de services Internet (ISP).
- Les directives organisationnelles ou les stratégies de sécurité n’autorisent pas les connexions sortantes en cas de traitement de données sensibles.

Dans les cas ci-dessus, utilisez un appareil physique pour effectuer un transfert de données unique en bloc. Faites votre choix parmi les appareils Data Box Disk, Data Box et Data Box Heavy qui sont fournis par Microsoft, ou Import/Export avec vos propres disques.

Pour vérifier si un appareil physique est adapté, utilisez le tableau suivant. Ce tableau indique le temps prévu pour le transfert de données réseau en fonction des différentes largeurs de bande passante disponibles (sur la base d’une utilisation de 90 %). Si le transfert réseau s’annonce trop lent, il est préférable d’utiliser un appareil physique.  

![Transfert réseau ou transfert hors connexion](media/storage-solution-large-dataset-low-network/storage-network-or-offline-transfer.png)

## <a name="recommended-options"></a>Options recommandées

Les options disponibles dans ce scénario sont les appareils destinés au transfert hors connexion Azure Data Box ou Azure Import/Export.

- **Famille Azure Data Box pour transferts hors connexion** - Si vous êtes limité par le temps, la disponibilité du réseau ou les coûts, utilisez les appareils Data Box fournis par Microsoft pour transférer les volumes de données importants vers Azure. Copiez les données localement à l’aide d’outils tels que Robocopy. En fonction du volume des données à transférer, vous pouvez choisir Data Box Disk, Data Box ou Data Box Heavy.
- **Azure Import/Export** - Utilisez le service Azure Import/Export avec vos propres disques pour importer en toute sécurité des volumes importants de données vers le Stockage Blob Azure et le Stockage Fichier. Vous pouvez également utiliser ce service pour transférer des données de Stockage Blob Azure vers des lecteurs de disque et les expédier vers vos sites locaux.

## <a name="comparison-of-key-capabilities"></a>Comparaison des fonctionnalités clés

Le tableau suivant résume les différences entre les principales fonctionnalités.

|                                     |    Data Box Disk      |    Data Box                                      |    Data Box Heavy              |    Importer/Exporter                       |
|-------------------------------------|---------------------------------|--------------------------------------------------|------------------------------------------|----------------------------------------|
|    **Taille des données**                    |    Jusqu’à 35 To                 |    Jusqu’à 80 To par appareil                       |    Jusqu’à 800 To par appareil               |    Variable                            |
|    **Type de données**                    |    Objets blob Azure                  |    Objets blob Azure<br>Azure Files                    |    Objets blob Azure<br>Azure Files            |    Objets blob Azure<br>Azure Files          |
|    **Facteur de forme**                  |    5 disques SSD par commande             |    1 x appareil de 50 lb de taille bureau par commande    |    1 x gros appareil d’environ 500 lb par commande    |    Jusqu’à 10 disques HDD/SSD par commande        |
|    **Temps d’installation initial**           |    Faible <br>(15 minutes)            |    Faible à modéré <br> (< 30 minutes)               |    Modéré<br>(1 à 2 heures)               |    Modéré à difficile<br>(variable) |
|    **Envoyer des données vers Azure**           |    Oui                          |    Oui                                           |    Oui                                   |    Oui                                 |
|    **Exporter des données à partir d’Azure**       |    Non                           |    Non                                            |    Non                                    |    Oui                                 |
|    **Chiffrement**                   |    AES 128 bits                  |    AES 256 bits                                   |    AES 256 bits                           |    AES 128 bits                         |
|    **Matériel**                     |     Fourni par Microsoft          |    Fourni par Microsoft                            |    Fourni par Microsoft                    |    Fourni par le client                   |
|    **Interface réseau**            |    USB 3.1/SATA                 |    RJ45, SFP+                                   |    RJ45, QSFP+                           |    SATA II/SATA III                    |
|    **Intégration des partenaires**          |    Certains                         |    [Élevée](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureExpressPod)                                          |    [Élevée](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureExpressPod)                                  |    Certains                                |
|    **Expédition**                     |    Managée par Microsoft            |    Managée par Microsoft                             |    Managée par Microsoft                     |    Managée par le client                    |
| **Utilisation en cas de déplacement de données**     |Dans une zone commerciale|Dans une zone commerciale|Dans une zone commerciale|Au-delà des frontières géographiques, par exemple USA - UE|
|    **Tarification**                      |    [Tarification](https://azure.microsoft.com/pricing/details/databox/disk/)                    |   [Tarification](https://azure.microsoft.com/pricing/details/storage/databox/)                                      |  [Tarification](https://azure.microsoft.com/pricing/details/storage/databox/heavy/)                               |   [Tarification](https://azure.microsoft.com/pricing/details/storage-import-export/)                            |


## <a name="next-steps"></a>Étapes suivantes

- Comprendre comment

    - [Transférer des données avec Data Box Disk](../../databox/data-box-disk-quickstart-portal.md).
    - [Transférer des données avec Data Box](../../databox/data-box-quickstart-portal.md).
    - [Transférer des données avec Import/Export](./storage-import-export-data-to-blobs.md).
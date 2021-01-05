---
title: Options de transfert de données Azure pour les jeux de données volumineux avec une bande passante réseau modérée à élevée | Microsoft Docs
description: Votre environnement dispose d'une bande passante réseau modérée à élevée et vous souhaitez transférer des jeux de données volumineux ? Apprenez à choisir la solution de transfert de données Azure qui convient.
services: storage
author: alkohli
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.date: 04/01/2019
ms.author: alkohli
ms.openlocfilehash: f1ded1ce87df1cb57960bbc99cd950ed7b3f97ee
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96573933"
---
# <a name="data-transfer-for-large-datasets-with-moderate-to-high-network-bandwidth"></a>Transférer des jeux de données volumineux avec une bande passante réseau modérée à élevée
 
Cet article fournit une vue d'ensemble des solutions de transfert de données adaptées aux environnements disposant d'une bande passante réseau modérée à élevée et au transfert de jeux de données volumineux. Il décrit également les options de transfert de données recommandées et la matrice de fonctionnalités clés correspondant à ce scénario.

Pour une vue d’ensemble de toutes les options de transfert de données disponibles, voir [Choisir une solution de transfert de données Azure](storage-choose-data-transfer-solution.md).

## <a name="scenario-description"></a>Description du scénario

Les jeux de données volumineux font référence aux données qui se mesurent en To et Po. Une bande passante réseau modérée à élevée est comprise entre 100 Mbits/s et 10 Gbits/s.

## <a name="recommended-options"></a>Options recommandées

Les options recommandées dans ce scénario varient en fonction de votre bande passante réseau.

### <a name="moderate-network-bandwidth-100-mbps---1-gbps"></a>Bande passante réseau modérée (100 Mbits/s - 1 Gbits/s)

Avec une bande passante réseau modérée, vous devez prévoir le temps de transfert des données sur le réseau.

Utilisez le tableau suivant pour estimer le temps nécessaire, puis choisissez un transfert hors connexion ou un transfert réseau. Ce tableau indique le temps prévu pour le transfert de données réseau en fonction des différentes largeurs de bande réseau disponibles (sur la base d'une utilisation de 90 %).  

![Transfert réseau ou transfert hors connexion](media/storage-solution-large-dataset-low-network/storage-network-or-offline-transfer.png)

- Si le transfert réseau s'annonce trop lent, il est préférable d'utiliser un appareil physique. Dans ce cas, nous vous recommandons d'utiliser les appareils de transfert hors connexion de la famille Azure Data Box ou le service Azure Import/Export avec vos propres disques.

    - **Famille Azure Data Box pour transferts hors connexion** - Si vous êtes limité par le temps, la disponibilité du réseau ou les coûts, utilisez les appareils Data Box fournis par Microsoft pour transférer les volumes de données importants vers Azure. Copiez les données localement à l’aide d’outils tels que Robocopy. En fonction du volume des données à transférer, vous pouvez choisir Data Box Disk, Data Box ou Data Box Heavy.
    - **Azure Import/Export** - Utilisez le service Azure Import/Export avec vos propres disques pour importer en toute sécurité des volumes importants de données vers le Stockage Blob Azure et le Stockage Fichier. Vous pouvez également utiliser ce service pour transférer des données de Stockage Blob Azure vers des lecteurs de disque et les expédier vers vos sites locaux.

- Si le transfert réseau prévu est raisonnable, vous pouvez utiliser l'un des outils suivants, détaillés dans [Bande passante réseau élevée](#high-network-bandwidth).


### <a name="high-network-bandwidth-1-gbps---100-gbps"></a>Bande passante réseau élevée (1 Gbits/s - 100 Gbits/s)

Si la bande passante réseau disponible est élevée, utilisez l'un des outils suivants.

- **AzCopy** – utilisez cet outil en ligne de commande pour copier facilement des données vers et à partir du Stockage Blob, Fichier et Table Azure avec des performances optimales. Il prend en charge la concurrence et le parallélisme, ainsi que la possibilité de reprendre les opérations de copie après une interruption.
- **API/kits SDK Stockage Azure** – lorsque vous créez une application, vous pouvez la développer par rapport à l’API REST Stockage Azure et utiliser les kits SDK Azure proposés dans plusieurs langues.
- **Famille Azure Data Box pour les transferts en ligne** - Data Box Edge et Data Box Gateway sont des appareils réseau en ligne capables de déplacer des données vers et depuis Azure. Utilisez l'appareil physique Data Box Edge en cas de besoin simultané d'ingestion en continu et de prétraitement des données avant le téléchargement. Data Box Gateway est une version virtuelle de l’appareil, offrant les mêmes fonctionnalités de transfert de données. Dans chaque cas, le transfert de données est géré par l'appareil.
- **Azure Data Factory** - Data Factory doit être utilisé pour effectuer un scale-out d’une opération de transfert, et lorsque des fonctionnalités d'orchestration et de supervision de qualité professionnelle sont requises. Utilisez Data Factory pour transférer régulièrement des fichiers entre plusieurs services Azure, en local ou les deux. Avec Data Factory, vous pouvez créer et planifier des workflows pilotés par des données (appelés pipelines) qui ingèrent des données provenant de magasins de données disparates et automatisent le déplacement et la transformation des données.

## <a name="comparison-of-key-capabilities"></a>Comparaison des fonctionnalités clés

Les tableaux suivants récapitulent les différences entre les fonctionnalités clés pour les options recommandées.

### <a name="moderate-network-bandwidth"></a>Bande passante réseau modérée

Si vous utilisez le transfert de données hors connexion, reportez-vous au tableau suivant pour comprendre les différences entre les fonctionnalités clés.

|                                     |    Data Box Disk      |    Data Box                                      |    Data Box Heavy            |    Importer/Exporter                       |
|-------------------------------------|---------------------------------|--------------------------------------------------|------------------------------------------|----------------------------------------|
|    **Taille des données**                    |    Jusqu’à 35 To                 |    Jusqu’à 80 To par appareil                       |    Jusqu’à 800 To par appareil               |    Variable                            |
|    **Type de données**                    |    Objets blob Azure                  |    Objets blob Azure<br>Azure Files                    |    Objets blob Azure<br>Azure Files            |    Objets blob Azure<br>Azure Files          |
|    **Facteur de forme**                  |    5 disques SSD par commande             |    1 x appareil de 50 lb de taille bureau par commande    |    1 x gros appareil d’environ 500 lb par commande    |    Jusqu’à 10 disques HDD/SSD par commande        |
|    **Temps d’installation initial**               |    Faible <br>(15 minutes)            |    Faible à modéré <br> (< 30 minutes)               |    Modéré<br>(1 à 2 heures)               |    Modéré à difficile<br>(variable) |
|    **Envoyer des données vers Azure**           |    Oui                          |    Oui                                           |    Oui                                   |    Oui                                 |
|    **Exporter des données à partir d’Azure**           |    Non                           |    Non                                            |    Non                                    |    Oui                                 |
|    **Chiffrement**                   |    AES 128 bits                  |    AES 256 bits                                   |    AES 256 bits                           |    AES 128 bits                         |
|    **Matériel**                     |     Fourni par Microsoft          |    Fourni par Microsoft                            |    Fourni par Microsoft                    |    Fourni par le client                   |
|    **Interface réseau**            |    USB 3.1/SATA                 |    RJ45, SFP+                                   |    RJ45, QSFP+                           |    SATA II/SATA III                    |
|    **Intégration des partenaires**          |    Certains                         |    [Élevée](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureExpressPod)                                          |    [Élevée](https://azuremarketplace.microsoft.com/en-us/marketplace/apps/Microsoft.AzureExpressPod)                                  |    Certains                                |
|    **Expédition**                     |    Managée par Microsoft            |    Managée par Microsoft                             |    Managée par Microsoft                     |    Managée par le client                    |
| **Utilisation en cas de déplacement de données**     |Dans une zone commerciale|Dans une zone commerciale|Dans une zone commerciale|Au-delà des frontières géographiques, par exemple USA - UE|
|    **Tarification**                          |    [Tarification](https://azure.microsoft.com/pricing/details/databox/disk/)                    |   [Tarification](https://azure.microsoft.com/pricing/details/storage/databox/)                                      |  [Tarification](https://azure.microsoft.com/pricing/details/storage/databox/heavy/)                               |   [Tarification](https://azure.microsoft.com/pricing/details/storage-import-export/)                            |


Si vous utilisez le transfert de données en ligne, reportez-vous au tableau de la section suivante, Bande passante réseau élevée.

### <a name="high-network-bandwidth"></a>Bande passante réseau élevée

|                                     |    Outils AzCopy, <br>Azure PowerShell, <br>Azure CLI             |    API REST Stockage Azure, Kits de développement logiciel (SDK)                   |    Data Box Gateway ou Data Box Edge          |    Azure Data Factory                                            |
|-------------------------------------|------------------------------------|----------------------------------------------|----------------------------------|-----------------------------------------------------------------------|
|    **Type de données**              |    Blobs, Fichiers et Tables Azure    |    Blobs, Fichiers et Tables Azure    |    Blob et Fichiers Azure                           |   Prend en charge plus de 70 connecteurs de données pour les formats et les magasins de données    |
|    **Facteur de forme**            |    Outils de ligne de commande                        |    Interface de programmation                    |    Microsoft fournit un appareil virtuel <br>ou physique     |    Service sur le portail Azure                                            |
|    **Installation ponctuelle initiale** |    Facile               |    Modéré                       |    Facile (< 30 minutes) à modérée (1 à 2 heures)            |    Exigeante                                                          |
|    **Prétraitement des données**          |    Non                                        |    Non                                        |    Oui (avec computing en périphérie)                               |    Oui                                                                |
|    **Transfert à partir d'autres clouds**   |    Non                                        |    Non                                        |    Non                                                    |    Oui                                                                |
|    **Type d’utilisateur**                    |    Professionnel de l'informatique ou développeur                                       |    Dev                                       |    Professionnel de l'informatique                                                |    Professionnel de l'informatique                                                             |
|    **Tarification**                      |    Gratuit, des frais de sortie de données s'appliquent         |    Gratuit, des frais de sortie de données s'appliquent         |    [Tarification](https://azure.microsoft.com/pricing/details/storage/databox/edge/)                                               |    [Tarification](https://azure.microsoft.com/pricing/details/data-factory/)                                                            |

## <a name="next-steps"></a>Étapes suivantes

- [Apprendre à transférer des données avec Import/Export](./storage-import-export-data-to-blobs.md).
- Comprendre comment

    - [Transférer des données avec Data Box Disk](../../databox/data-box-disk-quickstart-portal.md).
    - [Transférer des données avec Data Box](../../databox/data-box-quickstart-portal.md).
- [Transférer des données avec AzCopy](./storage-use-azcopy-v10.md).
- Comprendre comment :
    - [Transférer des données avec Data Box Gateway](../../databox-gateway/data-box-gateway-deploy-add-shares.md).
    - [Transformer des données avec Data Box Edge avant de les envoyer à Azure](../../databox-online/azure-stack-edge-deploy-configure-compute.md).
- [Apprendre à transférer des données avec Azure Data Factory](../../data-factory/quickstart-create-data-factory-portal.md).
- Utiliser les API REST pour transférer des données

    - [Dans .NET](/dotnet/api/overview/azure/storage)
    - [Dans Java](/java/api/overview/azure/storage)
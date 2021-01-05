---
title: Présentation de la facturation d’Azure CDN | Microsoft Docs
description: Découvrez la structure de facturation du contenu hébergé par Azure Content Delivery Network, y compris les régions de facturation, les frais de livraison et la gestion des coûts.
services: cdn
documentationcenter: ''
author: asudbring
manager: danielgi
editor: ''
ms.assetid: ''
ms.service: azure-cdn
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 09/13/2019
ms.author: allensu
ms.openlocfilehash: aa2f00a732a3978524fc017481285859c9535387
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96018679"
---
# <a name="understanding-azure-cdn-billing"></a>Présentation de la facturation du CDN Azure

Ce FAQ décrit la structure de facturation du contenu hébergé par Azure Content Delivery Network (CDN).

## <a name="what-is-a-billing-region"></a>Qu’est-ce qu’une région de facturation ?
Une région de facturation est une zone géographique utilisée pour déterminer le taux de facturation appliqué pour la livraison des objets à partir d’Azure CDN. Les zones de facturation actuelles et leurs régions sont les suivantes :

- Zone 1 : Amérique du Nord, Europe, Moyen-Orient et Afrique

- Zone 2 : Asie-Pacifique (Japon inclus)

- Zone 3 : Amérique du Sud

- Zone 4 : Australie et Nouvelle Zélande

- Zone 5 : Inde

Pour plus d’informations sur les régions POP, consultez [Emplacements POP du CDN Azure par région](./cdn-pop-locations.md). Par exemple, un point POP situé au Mexique se trouve dans la région Amérique du Nord et est donc inclus dans la zone 1. 

Pour plus d’informations sur les tarifs d’Azure CDN, consultez [Tarifs Content Delivery Network](https://azure.microsoft.com/pricing/details/cdn/).

## <a name="how-are-delivery-charges-calculated-by-region"></a>Comment les frais de livraison sont-ils calculés par région ?
La région de facturation d’Azure CDN est fonction de l’emplacement du serveur source qui fournit le contenu à l’utilisateur final. La destination (emplacement physique) du client n’est pas considérée comme la région de facturation.

Par exemple, si un utilisateur situé au Mexique émet une requête qui est traitée par un serveur situé dans un point POP aux États-Unis en raison des conditions de peering ou de trafic, les États-Unis sont la région de facturation.

## <a name="what-is-a-billable-azure-cdn-transaction"></a>Qu’est-ce qu’une transaction Azure CDN facturable ?
Toute requête HTTP(S) qui se termine au niveau du CDN est un événement facturable, qui inclut tous les types de réponse : réussite, échec ou autre. Toutefois, différentes réponses peuvent générer différents montants de trafic. Par exemple, la réponse *304 Non modifié* et d’autres réponses avec en-tête seulement génèrent peu de trafic, car ce sont des réponses courtes avec en-tête. De même, les réponses d’erreur (par exemple, *404 Introuvable*) sont facturables, mais entraînent un faible coût en raison de la minuscule charge utile de réponse.

## <a name="what-other-azure-costs-are-associated-with-azure-cdn-use"></a>Quels sont les autres coûts Azure associés à l’utilisation d’Azure CDN ?
L’utilisation d’Azure CDN implique également des frais d’utilisation sur les services utilisés en tant qu’origine de vos objets. Ces coûts représentent généralement une petite partie du coût d’utilisation du CDN global.

Si vous utilisez le stockage Blob Azure en tant qu’origine de votre contenu, les frais de stockage suivants correspondant au remplissage du cache vous sont également facturés :

- Go réels utilisés : stockage réel de vos objets sources.

- Transactions : en fonction des besoins du remplissage du cache.

- Transferts en Go : quantité de données transférées pour remplir les caches du CDN.

> [!NOTE]
> À partir d’octobre 2019, si vous utilisez le service Azure CDN de Microsoft, le coût du transfert de données d’origines hébergées dans Azure vers des PoP (points de présence) de CDN est gratuit. Azure CDN de Verizon et Azure CDN d’Akamai sont soumis aux tarifs décrits ci-dessous.

Pour plus d’informations sur la facturation du service Stockage Azure, consultez [Understanding Windows Azure Storage Billing – Bandwidth, Transactions, and Capacity](https://blogs.msdn.microsoft.com/windowsazurestorage/2010/07/08/understanding-windows-azure-storage-billing-bandwidth-transactions-and-capacity/) (Présentation de la facturation du service Stockage Microsoft Azure : bande passante, transactions et capacité).

Si vous utilisez la *fourniture de service hébergé*, les frais suivants sont appliqués :

- Temps de calcul Azure : instances de calcul qui servent d’origine.

- Transfert de calcul Azure : transferts de données à partir des instances de calcul pour remplir les caches d’Azure CDN.

Si votre client utilise des requêtes de plage d’octets (quel que soit le service d’origine), les considérations suivantes s’appliquent :

- Une *requête de plage d’octets* est une transaction facturable au niveau du CDN. Lorsqu’un client émet une requête de plage d’octets, cette requête concerne un sous-ensemble (plage) de l’objet. La réponse du CDN ne comprend qu’une fraction du contenu qui est demandé. Cette réponse partielle est une transaction facturable, et la quantité de transfert est limitée à la taille de la réponse de plage (plus les en-têtes).

- Lorsqu’une requête arrive pour une seule partie d’un objet (en spécifiant un en-tête de plage d’octets), le CDN peut récupérer l’objet entier dans son cache. Par conséquent, même si la transaction facturable du CDN concerne une réponse partielle, la transaction facturable provenant de l’origine peut impliquer la taille totale de l’objet.

## <a name="how-much-transfer-activity-occurs-to-support-the-cache"></a>Quelles sont les activités de transfert pour assurer la prise en charge du cache ?
À chaque fois qu’un POP CDN doit remplir son cache, il établit une requête destinée à l’origine de l’objet à mettre en cache. En conséquence, l’origine applique une transaction facturable pour chaque absence dans le cache. Le nombre d’absences dans le cache dépend de plusieurs facteurs :

- Capacité de mise en cache du contenu : si le contenu possède des valeurs de durée/expiration élevées et qu’il est fréquemment sollicité et reste donc très utilisé dans le cache, alors la majorité de la charge est gérée par le CDN. Un taux d’accès au cache correct classique est supérieur à 90 %, ce qui signifie que moins de 10 % des requêtes clientes doivent revenir à l’origine, soit pour une absence de cache, soit pour une actualisation d’objet.

- Nombre de nœuds nécessaires pour charger l’objet : à chaque fois qu’un nœud charge un objet à partir de l’origine, une transaction facturable est appliquée. Par conséquent, un contenu plus global (accessible à partir de plus de nœuds) entraîne davantage de transactions facturables.

- Influence de la durée de vie : si un objet a une durée de vie plus longue, cela signifie qu’il doit être récupéré à partir de l’origine moins fréquemment. Cela signifie également que les clients, tels que les navigateurs, peuvent mettre en cache l’objet plus de temps, ce qui peut réduire les transactions vers le CDN.

## <a name="which-origin-services-are-eligible-for-free-data-transfer-with-azure-cdn-from-microsoft"></a>Quels sont les services d’origine éligibles au transfert de données gratuit avec Azure CDN de Microsoft ? 
Si vous utilisez l’un des services Azure suivants en tant qu’origine de CDN, vous n’êtes pas facturé pour le transfert de données de l’origine aux PoP de CDN. 

- Stockage Azure
- Azure Media Services
- Machines virtuelles Azure
- Réseau virtuel
- Load Balancer
- Application Gateway
- Azure DNS
- ExpressRoute
- Passerelle VPN
- Traffic Manager
- Network Watcher
- Pare-feu Azure
- Azure Front Door Service
- Azure Bastion
- Azure App Service
- Azure Functions
- Azure Data Factory
- Gestion des API Azure
- Azure Batch 
- Explorateur de données Azure
- HDInsight
- Azure Cosmos DB
- Azure Data Lake Store
- Azure Machine Learning 
- Azure SQL Database
- Azure SQL Managed Instance
- Cache Azure pour Redis

## <a name="how-do-i-manage-my-costs-most-effectively"></a>Comment gérer plus efficacement les coûts ?
Définissez la plus longue durée de vie possible de votre contenu.
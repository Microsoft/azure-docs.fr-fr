---
title: Vue d’ensemble de Microsoft Azure FXT Edge Filer
description: Décrit le cache de stockage hybride Azure FXT Edge Filer, une solution active d’accélérateur d’accès aux archives et aux fichiers pour les tâches de calcul haute performance.
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: overview
ms.date: 07/01/2019
ms.author: rohogue
ms.openlocfilehash: 72308bf64246a8041bd40fa557280014a13c95eb
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92218780"
---
# <a name="what-is-azure-fxt-edge-filer-hybrid-storage-cache"></a>Qu’est-ce que cache de stockage hybride Azure FXT Edge Filer ?

Azure FXT Edge Filer est une appliance de mise en cache de stockage hybride qui fournit un accès rapide aux fichiers et des archives actives pour les tâches de calcul haute performance (HPC).

Il fonctionne avec plusieurs sources de données, qu’elles soient stockées dans un centre de données local, à distance ou dans le cloud. Azure FXT Edge Filer peut fournir un espace de noms unifié pour les données dans divers systèmes de stockage.

Trois périphériques physiques FXT Edge Filer ou plus fonctionnent ensemble comme un système de fichiers en cluster pour fournir le cache. Pour plus de détails sur l’achat du matériel requis, contactez votre représentant Microsoft.

Pour en savoir plus, lisez les informations produit et la fiche technique sur [Azure FXT Edge Filer](https://azure.microsoft.com/services/fxt-edge-filer/).

## <a name="use-cases"></a>Cas d'utilisation

Azure FXT Edge Filer améliore la productivité de ces workflows :

* Workflow nécessitant un grand nombre de lectures de fichier
* Protocole NFSv3 ou SMB2
* Calcul des batteries de serveurs contenant entre 1 000 et 100 000 cœurs d’unité centrale

### <a name="nas-optimization-and-scaling"></a>Optimisation et mise à l’échelle de périphérique de stockage NAS

Vous pouvez utiliser le cache Azure FXT Edge Filer pour faciliter l’accès aux systèmes NAS NetApp et Dell EMC Isilon existants. Vous pouvez également ajouter Azure Blob ou d’autres solutions de stockage en ligne pour offrir une évolutivité sans avoir à retravailler les processus d’accès aux données côté client.

### <a name="wan-caching"></a>Mise en cache WAN

Azure FXT Edge Filer permet de prendre l’accès rapide aux fichiers des utilisateurs expérimentés lorsque les données dont ils ont besoin sont stockées ailleurs. Fournissez l’accès tout en conservant les systèmes de sauvegarde et autres systèmes de gestion des données dans un centre de données centralisé.

### <a name="active-archive-in-azure-blob"></a>Archivage actif dans un stockage d’objets Blob Azure

Développez votre centre de données dans le stockage cloud avec Azure FXT Edge Filer en tant que point d’accès.

## <a name="features"></a>Fonctionnalités

Deux modèles de matériel sont disponibles.

| Modèle | DRAM | Disque SSD NVMe | Ports réseau |
|-------|------|----------|---------------|
| FXT 6600 | 1536 Go | 25,6 To | 6 x 25 Go / 10 Go + 2 x 1 Go |
| FXT 6400 | 768 Go | 12,8 To | 6 x 25 Go / 10 Go + 2 x 1 Go |

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur Azure FXT Edge Filer, consultez les [spécifications](fxt-specs.md) ou le [tutoriel d’installation](fxt-install.md).
* Consultez la [page produit Azure FXT Edge Filer](https://azure.microsoft.com/services/fxt-edge-filer/) pour découvrir comment acheter Azure FXT Edge Filer.

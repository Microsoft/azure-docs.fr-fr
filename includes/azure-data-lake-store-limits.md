---
title: Fichier Include
description: Fichier Include
services: storage
author: twooley
ms.service: storage
ms.topic: include
ms.date: 09/30/2020
ms.author: twooley
ms.custom: include file
ms.openlocfilehash: 7098f23e9b5b6f56fbbe761335afc65375aea680
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96509496"
---
**Azure Data Lake Storage Gen2** n’est ni un type de compte de stockage ni un service dédié. Il s’agit de la dernière version des fonctionnalités qui sont dédiées à l’analytique du Big Data.  Ces fonctionnalités sont disponibles dans un compte de stockage universel v2 ou BlockBlobStorage, que vous pouvez obtenir en activant la fonctionnalité **Espace de noms hiérarchique** du compte. Pour les cibles de mise à l’échelle, consultez ces articles. 

- [Cibles de scalabilité pour Stockage Blob](../articles/storage/blobs/scalability-targets.md#scale-targets-for-blob-storage).
- [Cibles de scalabilité pour les comptes de stockage standard](../articles/storage/common/scalability-targets-standard-account.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#scale-targets-for-standard-storage-accounts).

**Azure Data Lake Storage Gen1** est un service dédié. C’est un référentiel hyperscale de niveau entreprise pour les charges de travail d’analytique du Big Data. Vous pouvez utiliser Data Lake Storage Gen1 pour capturer des données indépendamment de leur taille, leur type et leur vitesse d’ingestion au même endroit pour les analyser d’un point de vue opérationnel et exploratoire. Aucune limite ne s’applique à la quantité de données que vous pouvez stocker dans un compte Data Lake Storage Gen1.

| **Ressource** | **Limite** | **Commentaires** |
| --- | --- | --- |
| Nombre maximal de comptes Data Lake Storage Gen1 par abonnement, par région |10 | Pour demander une augmentation de cette limite, contactez le support. |
| Nombre maximal de listes de contrôle d’accès (ACL) par fichier ou dossier |32 | Il s’agit d’une limite inconditionnelle. Utilisez des groupes pour gérer l’accès avec moins d’entrées. |
| Nombre maximal de listes de contrôle d’accès (ACL) par défaut, par fichier ou dossier |32 | Il s’agit d’une limite inconditionnelle. Utilisez des groupes pour gérer l’accès avec moins d’entrées. |
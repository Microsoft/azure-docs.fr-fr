---
title: Fichier include
description: Fichier include
services: storage
author: tamram
ms.service: storage
ms.topic: include
ms.date: 03/05/2020
ms.author: tamram
ms.custom: include file
ms.openlocfilehash: b8a72640ebe8daffedb196456df7d40bc41b7b1b
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95559863"
---
Le stockage Azure offre plusieurs types de comptes de stockage. Chaque type prend en charge différentes fonctionnalités et a son propre modèle tarifaire. Avant de créer un compte de stockage, tenez compte de ces différences pour déterminer l’option qui convient le mieux à vos applications. Les types de comptes de stockage proposés sont les suivants :

- **Comptes de stockage à usage général v2** : Type de compte de stockage de base pour les objets blob, les fichiers, les files d’attente et les tables. Recommandé pour la plupart des scénarios utilisant Stockage Azure.
- **Comptes v1 universels** : Type de compte de stockage hérité pour les objets blob, les fichiers, les files d’attente et les tables. Utilisez ce type de compte à la place des comptes v2 universels lorsque cela est possible.
- **Comptes BlockBlobStorage** : comptes de stockage dotés de caractéristiques de performances Premium pour les objets blob de bloc et les objets blob d'ajout. Recommandé pour les scénarios aux taux élevés de transactions, ceux utilisant des objets plus petits ou nécessitant une latence de stockage faible.
- **Comptes FileStorage** : comptes de stockage de fichiers uniquement ayant des caractéristiques de performances Premium. Recommandé pour l’entreprise ou des applications de mise à l’échelle hautes performances.
- **Comptes BlobStorage** : anciens comptes de stockage Blob uniquement. Utilisez ce type de compte à la place des comptes v2 universels lorsque cela est possible.

Le tableau suivant répertorie les types de comptes de stockage disponibles et leurs fonctionnalités :

| Type de compte de stockage | Services pris en charge                       | Niveaux de performances pris en charge      | Niveaux d’accès pris en charge         | Options de réplication               | Modèle de déploiement<div role="complementary" aria-labelledby="deployment-model"><sup>1</sup></div> | Chiffrement<div role="complementary" aria-labelledby="encryption"><sup>2</sup></div> |
|----------------------|------------------------------------------|-----------------------------|--------------------------------|-----------------------------------|------------------------------|------------------------|
| Universel v2   | Objet blob, fichier, file d’attente, table, disque et Data Lake Gen2<div role="complementary" aria-labelledby="data-lake-gen2"><sup>6</sup></div>      | Standard, Premium<div role="complementary" aria-labelledby="premium-performance"><sup>5</sup></div> | Chaud, froid, archive<div role="complementary" aria-labelledby="archive"><sup>3</sup></div> | LRS, GRS, RA-GRS, ZRS, GZRS (préversion), RA-GZRS (préversion)<div role="complementary" aria-labelledby="zone-redundant-storage"><sup>4</sup></div> | Gestionnaire de ressources             | Chiffré              |
| Universel v1   | Objets blob, fichiers, files d’attente, tables et disques       | Standard, Premium<div role="complementary" aria-labelledby="premium-performance"><sup>5</sup></div> | N/A                            | LRS, GRS, RA-GRS                  | Resource Manager, Classic    | Chiffré              |
| BlockBlobStorage   | Objets blob (objets blob de blocs et objets blob d’ajout uniquement) | Premium                       | N/A                            | LRS, ZRS<div role="complementary" aria-labelledby="zone-redundant-storage"><sup>4</sup></div>                               | Gestionnaire de ressources             | Chiffré              |
| FileStorage   | Fichier uniquement | Premium                       | N/A                            | LRS, ZRS<div role="complementary" aria-labelledby="zone-redundant-storage"><sup>4</sup></div>                               | Gestionnaire de ressources             | Chiffré              |
| BlobStorage         | Objets blob (objets blob de blocs et objets blob d’ajout uniquement) | standard                      | Chaud, froid, archive<div role="complementary" aria-labelledby="archive"><sup>3</sup></div> | LRS, GRS, RA-GRS                  | Gestionnaire de ressources             | Chiffré              |

<div id="deployment-model"><sup>1</sup>L’utilisation du modèle de déploiement Azure Resource Manager est recommandée. Les comptes de stockage qui utilisent le modèle de déploiement classique peuvent toujours être créés à certains emplacements, et les comptes classiques existants continuent d’être pris en charge. Pour plus d’informations, consultez <a href="/azure/azure-resource-manager/resource-manager-deployment-model">Déploiement Azure Resource Manager et déploiement Classic : comprendre les modèles de déploiement et l’état de vos ressources</a>.</div><br/>

<div id="encryption"><sup>2</sup>Dans tous les comptes de stockage, les données au repos sont chiffrées à l’aide de Storage Service Encryption (SSE). Pour plus d’informations, consultez <a href="/azure/storage/common/storage-service-encryption">Azure Storage Service Encryption pour les données au repos</a>.</div><br/>

<div id="archive"><sup>3</sup> Le stockage archive et la hiérarchisation au niveau de l’objet blob prennent en charge uniquement les objets blob de blocs. L’archivage est disponible uniquement au niveau de chaque objet blob, pas au niveau du compte de stockage. Pour plus d’informations, consultez <a href="/azure/storage/blobs/storage-blob-storage-tiers">Stockage Blob Azure : Niveaux de stockage chaud, à froid et archivage</a>.</div><br/>

<div id="zone-redundant-storage"><sup>4</sup>Le stockage redondant interzone (ZRS) et le stockage géoredondant interzone (GZRS/RA-GZRS) (préversion) sont disponibles uniquement pour les comptes universels v2 standard, BlockBlobStorage et FileStorage dans certaines régions. Pour plus d’informations sur les options de redondance de Stockage Azure, consultez <a href="/azure/storage/common/storage-redundancy">Redondance du stockage Azure</a>.</div><br/>

<div id="premium-performance"><sup>5</sup>Performances Premium pour les comptes v1 et v2 à usage général est disponible uniquement pour les objets blob de page et de disque. Les performances Premium pour les objets blob de blocs ou d’ajout sont uniquement disponibles sur les comptes BlockBlobStorage. Les performances Premium pour les fichiers ne sont disponibles que sur les comptes FileStorage.</div><br/>

<div id="data-lake-gen2"><sup>6</sup>Azure Data Lake Storage Gen2 est un ensemble de fonctionnalités dédiées à l’analytique du Big Data et intégrées au stockage Blob Azure. Data Lake Storage Gen2 est pris en charge uniquement sur les comptes de stockage v2 universels avec un espace de noms hiérarchique activé. Pour plus d’informations sur Data Lake Storage Gen2, consultez <a href="/azure/storage/blobs/data-lake-storage-introduction">Présentation d’Azure Data Lake Storage Gen2</a>.</div>
---
title: Niveaux de performance du service de stockage d’objets blob de blocs – Stockage Azure
description: Décrit la différence entre les niveaux de performance Premium et Standard pour le stockage d’objets blob de blocs Azure.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/12/2019
ms.service: storage
ms.subservice: blobs
ms.topic: conceptual
ms.reviewer: clausjor
ms.openlocfilehash: 404999cfac5037702eb61fdf74b2c5245ce9eb30
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95526117"
---
# <a name="performance-tiers-for-block-blob-storage"></a>Niveaux de performance du stockage d’objets blob de blocs

À mesure que les entreprises déploient des applications cloud natives sensibles aux performances, il est impératif de disposer d'options de stockage de données économiques offrant différents niveaux de performance.

Le service de stockage d'objets blob de blocs Azure offre deux niveaux de performance différents :

- **Premium** : optimisé pour des taux de transaction élevés et une latence de stockage constante à un chiffre
- **Standard** : optimisé pour une capacité et un débit élevés

Les considérations suivantes s'appliquent aux différents niveaux de performance :

| Domaine |Niveau de performance Standard  |Niveau de performance Premium  |
|---------|---------|---------|
|Disponibilité des régions     |   Toutes les régions      | Dans [certaines régions](https://azure.microsoft.com/global-infrastructure/services/?products=storage)       |
|[Types de compte de stockage](../common/storage-account-overview.md#types-of-storage-accounts) pris en charge     |     Usage général v2, BlobStorage, Usage général v1    |    BlockBlobStorage     |
|Prend en charge les [objets blob de blocs à débit élevé](https://azure.microsoft.com/blog/high-throughput-with-azure-blob-storage/)     |    Oui, avec une taille de PutBlock ou PutBlob supérieure à 4 Mio     |    Oui, avec une taille de PutBlock ou PutBlob supérieure à 256 Kio    |
|Redondance     |     Voir [Types de compte de stockage](../common/storage-account-overview.md#types-of-storage-accounts)   |  Prend actuellement en charge uniquement le stockage localement redondant (LRS) et le stockage redondant interzone (ZRS)<div role="complementary" aria-labelledby="zone-redundant-storage"><sup>1</sup></div>     |

<div id="zone-redundant-storage"><sup>1</sup>Le stockage redondant interzone (ZRS) est disponible dans certaines régions pour les comptes de stockage d’objets blob de blocs avec niveau de performance Premium.</div>

En ce qui concerne les coûts, le niveau de performance Premium offre une tarification optimisée pour les applications aux taux de transaction élevés afin de [réduire le coût total de stockage](https://azure.microsoft.com/blog/reducing-overall-storage-costs-with-azure-premium-blob-storage/) pour ces charges de travail.

## <a name="premium-performance"></a>Niveau de performance Premium

Le stockage d'objets blob de blocs du niveau de performance Premium permet d'accéder aux données à l'aide de matériel hautes performances. Les données sont stockées sur des disques SSD optimisés pour une latence faible. Les disques SSD offrent un débit plus élevé que les disques durs traditionnels.

Le stockage du niveau de performance Premium est adapté aux charges de travail qui nécessitent des temps de réponse rapides et cohérents. Il convient tout particulièrement aux charges de travail qui exécutent une multitude de petites transactions. Voici quelques exemples de charges de travail :

- **Charges de travail interactives**. Ces charges de travail requièrent des mises à jour instantanées et des commentaires de la part des utilisateurs, tels que des applications d’e-commerce et de mappage. Par exemple, dans une application d’e-commerce, les éléments les moins fréquemment consultés ne sont probablement pas mis en cache. Toutefois, ils doivent être affichés instantanément au client sur demande.

- **Analytique**. Dans un scénario IoT, un grand nombre d’opérations d’écriture plus petites peuvent être envoyées (push) chaque seconde dans le cloud. De grandes quantités de données peuvent être saisies, agrégées à des fins d’analyse, puis supprimées presque immédiatement. Les capacités d’ingestion élevées du stockage d’objets blob de blocs Premium le rendent efficace pour ce type de charge de travail.

- **Intelligence artificielle/apprentissage automatique (IA/ML)** . IA/ML traite de la consommation et du traitement de différents types de données, tels que les visuels, la parole et le texte. Ce type de charge de travail de calcul haute performance gère des volumes importants de données nécessitant une réponse rapide et des temps d’ingestion efficaces pour l’analyse des données.

- **Transformation des données**. Les processus qui nécessitent une modification et une conversion constantes de données requièrent des mises à jour instantanées. Pour une représentation exacte des données, les contrôles serveurs consommateurs de ces données doivent voir ces changements se refléter immédiatement.

## <a name="standard-performance"></a>Niveau de performance Standard

Le niveau de performance Standard prend en charge différents [niveaux d'accès](storage-blob-storage-tiers.md) pour stocker les données de la manière la plus économique. Il est optimisé pour une capacité et un débit élevés sur les jeux de données volumineux.

- **Jeux de données de sauvegarde et reprise d’activité après sinistre**. Le stockage de performances standard offre des niveaux à coûts réduits, ce qui en fait un cas d’utilisation parfait pour les jeux de données de reprise d’activité à court terme et à long terme, les sauvegardes secondaires et l’archivage des données de conformité.

- **Contenu multimédia**. Souvent, les images et les vidéos font l’objet d’accès fréquents lorsqu’elles sont créées et stockées, puis ce type de contenu est de moins en moins utilisé au fil du temps. Le stockage de performances standard offre des niveaux adaptés aux besoins en contenus multimédia. 

- **Traitement des données en bloc**. Ces types de charges de travail conviennent pour le stockage standard, car elles requièrent un stockage à haut débit et à coûts réduits plutôt qu’une faible latence constante. De grands ensembles de données brutes sont conservés pour traitement et peuvent être migrés vers des niveaux de stockage plus froids.

## <a name="migrate-from-standard-to-premium"></a>Migrer de Standard à Premium

Vous ne pouvez pas convertir un compte de stockage avec niveau de performance Standard existant en compte de stockage d’objets blob de blocs avec un niveau de performance Premium. Pour migrer vers un compte de stockage avec niveau de performance Premium, vous devez créer un compte BlockBlobStorage et migrer les données vers le nouveau compte. Pour plus d’informations, consultez la rubrique [Créer un compte BlockBlobStorage](storage-blob-create-account-block-blob.md).

Pour copier des blobs entre des comptes de stockage, vous pouvez utiliser la dernière version de l’outil en ligne de commande [AzCopy](../common/storage-use-azcopy-blobs.md). D’autres outils tels qu’Azure Data Factory sont également disponibles pour le déplacement et la transformation des données.

## <a name="blob-lifecycle-management"></a>Gestion de cycle de vie des objets blob

La gestion du cycle de vie du stockage de blobs propose une stratégie élaborée basée sur des règles :

- **Premium** : les données expirent à la fin de leur cycle de vie.
- **Standard** : les données passent au meilleur niveau d’accès et expirent à la fin de leur cycle de vie.

Pour plus d'informations, consultez [Gérer le cycle de vie du service Stockage Blob Azure](storage-lifecycle-management-concepts.md).

Vous ne pouvez pas déplacer les données stockées dans un compte de stockage d’objets blob de blocs Premium entre les niveaux chaud, froid et archive. Toutefois, vous pouvez copier des objets blob d'un compte de stockage d'objets blob de blocs vers le niveau d'accès chaud d'un *autre* compte. Pour copier des données vers un autre compte, utilisez l’API [Put Block From URL](/rest/api/storageservices/put-block-from-url) ou [AzCopy v10](../common/storage-use-azcopy-v10.md). L'API **Put Block From URL** copie les données sur le serveur de manière synchrone. L'appel ne se termine qu'une fois toutes les données déplacées de l'emplacement du serveur d'origine vers l'emplacement de destination.

## <a name="next-steps"></a>Étapes suivantes

Évaluer les niveaux chaud, froid et archive dans les comptes de stockage de blobs et GPv2.

- [Découvrir comment réalimenter les données d’objets blob à partir du niveau Archive](storage-blob-rehydration.md)
- [Évaluer l’utilisation des comptes de stockage actuels en activant les métriques Azure Storage](./monitor-blob-storage.md)
- [Vérifier la tarification du niveau chaud, froid et archive dans les comptes de stockage d’objets blob et GPv2 par région](https://azure.microsoft.com/pricing/details/storage/)
- [Vérifier la tarification des transferts de données](https://azure.microsoft.com/pricing/details/data-transfers/)
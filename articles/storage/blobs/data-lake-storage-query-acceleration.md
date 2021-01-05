---
title: Accélération des requêtes Azure Data Lake Storage
description: L’accélération des requêtes permet aux applications et aux infrastructures d’analytique d’optimiser considérablement le traitement des données en extrayant uniquement les données requises pour une opération de traitement.
author: normesta
ms.topic: conceptual
ms.author: normesta
ms.reviewer: jamesbak
ms.date: 09/09/2020
ms.service: storage
ms.subservice: data-lake-storage-gen2
ms.openlocfilehash: dc1d217dba64c36aa219abbd4d2220a494347689
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95912754"
---
# <a name="azure-data-lake-storage-query-acceleration"></a>Accélération des requêtes Azure Data Lake Storage

L’accélération des requêtes (préversion) est une nouvelle capacité d’Azure Data Lake Storage qui permet aux applications et aux infrastructures d’analytique d’optimiser considérablement le traitement des données en extrayant uniquement les données requises pour effectuer une opération donnée. Cela permet de réduire le temps et la puissance de traitement nécessaires pour obtenir un aperçu essentiel des données stockées.

## <a name="overview"></a>Vue d’ensemble

L’accélération des requêtes accepte les *prédicats de filtrage* et les *projections de colonnes* qui permettent aux applications de filtrer les lignes et les colonnes au moment où les données sont lues sur le disque. Seules les données qui remplissent les conditions d’un prédicat sont transférées à l’application via le réseau. Cela réduit la latence du réseau et le coût du calcul.  

Vous pouvez utiliser SQL pour spécifier les prédicats de filtrage de lignes et les projections de colonne dans une demande d’accélération de requête. Une demande traite un seul fichier. Par conséquent, les fonctionnalités relationnelles avancées de SQL, telles que les jointures et les regroupements par agrégats, ne sont pas prises en charge. L’accélération des requêtes prend en charge les données au format CSV et JSON comme entrée de chaque demande.

La fonctionnalité d’accélération des requêtes n’est pas limitée à Data Lake Storage (comptes de stockage sur lesquels l’espace de noms hiérarchique est activé). L’accélération des requêtes est entièrement compatible avec les blobs des comptes de stockage sur lesquels il **n’y a pas** d’espace de noms hiérarchique activé. Cela signifie que vous pouvez obtenir la même réduction de la latence du réseau et des coûts de calcul lorsque vous traitez des données que vous avez déjà stockées sous forme de blobs dans des comptes de stockage.

Pour obtenir un exemple d’utilisation de l’accélération des requêtes dans une application cliente, consultez [Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage](data-lake-storage-query-acceleration-how-to.md).

## <a name="data-flow"></a>Flux de données

Le diagramme suivant illustre la façon dont une application typique utilise l’accélération des requêtes pour traiter les données.

> [!div class="mx-imgBorder"]
> ![Query acceleration overview](./media/data-lake-storage-query-acceleration/query-acceleration.png)

1. L’application cliente demande des données de fichier en spécifiant des prédicats et des projections de colonne.

2. L’accélération des requêtes analyse la requête SQL spécifiée et distribue le travail pour analyser et filtrer les données.

3. Les processeurs lisent les données sur le disque, les analysent à l’aide du format approprié, puis les filtrent en appliquant les prédicats et les projections de colonne spécifiés.

4. L’accélération des requêtes combine les fragments de réponse pour les transmettre en continu à l’application cliente.

5. L’application cliente reçoit et analyse la réponse transmise en continu. L’application n’a pas besoin de filtrer de données supplémentaires et peut appliquer directement le calcul ou la transformation souhaité.

## <a name="better-performance-at-a-lower-cost"></a>Meilleures performances à moindre coût

L’accélération des requêtes optimise les performances en réduisant la quantité de données transférées et traitées par votre application.

Pour calculer une valeur agrégée, les applications récupèrent généralement **toutes** les données à partir d’un fichier, puis traitent et filtrent les données localement. Une analyse des modèles d’entrée/sortie pour les charges de travail analytiques révèle que les applications n’ont généralement besoin que de 20 % des données qu’elles lisent pour effectuer un calcul donné. Cette statistique est vraie même après l’application de techniques telles que l’[élagage de partition](../../hdinsight/hdinsight-hadoop-optimize-hive-query.md#hive-partitioning). Cela signifie que 80 % de ces données sont inutilement transférées via le réseau, analysées et filtrées par des applications. Ce modèle, conçu essentiellement pour supprimer les données inutiles, entraîne un coût de calcul significatif.  

Même si Azure dispose d’un réseau de pointe, tant en termes de débit que de latence, le transfert inutile de données sur ce réseau reste coûteux pour les performances des applications. En filtrant les données indésirables pendant la demande de stockage, l’accélération des requêtes élimine ce coût.

En outre, la charge d’UC requise pour analyser et filtrer les données inutiles oblige votre application à approvisionner des machines virtuelles plus volumineuses et en plus grand nombre afin de faire son travail. En transférant cette charge de calcul à l’accélération des requêtes, les applications peuvent réaliser des économies substantielles.

## <a name="applications-that-can-benefit-from-query-acceleration"></a>Applications pouvant tirer parti de l’accélération des requêtes

L’accélération des requêtes est conçue pour les infrastructures d’analytique distribuées et les applications de traitement des données. 

Les infrastructures d’analytique distribuée, telles qu’Apache Spark et Apache Hive, incluent une couche d’abstraction de stockage dans l’infrastructure. Ces moteurs incluent également des optimiseurs de requête qui peuvent incorporer la connaissance des capacités du service d’E/S sous-jacent lors de la détermination d’un plan de requête optimal pour les requêtes utilisateur. Ces infrastructures commencent à intégrer l’accélération des requêtes. Par conséquent, les utilisateurs de ces infrastructures constateront une amélioration de la latence des requêtes et une réduction du coût total de possession sans avoir à apporter de modifications aux requêtes. 

L’accélération des requêtes est également conçue pour les applications de traitement de données. Ces types d’applications effectuent généralement des transformations de données à grande échelle qui peuvent ne pas conduire directement à des aperçus analytiques, de sorte qu’elles n’utilisent pas toujours des infrastructures d’analytique distribuées établies. Ces applications ont souvent une relation plus directe avec le service de stockage sous-jacent afin de pouvoir bénéficier directement de fonctionnalités telles que l’accélération des requêtes. 

Pour obtenir un exemple de la façon dont une application peut intégrer l’accélération des requêtes, consultez [Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage](data-lake-storage-query-acceleration-how-to.md).

## <a name="pricing"></a>Tarifs

En raison de l’augmentation de la charge de calcul au sein du service Azure Data Lake Storage, le modèle de tarification pour l’utilisation de l’accélération des requêtes diffère du modèle de transaction normal d’Azure Data Lake Storage. L’accélération des requêtes facture un coût pour la quantité de données analysées, ainsi qu’un coût pour la quantité de données retournées à l’appelant. Pour plus d’informations, consultez [Tarification d’Azure Data Lake Storage Gen2](https://azure.microsoft.com/pricing/details/storage/data-lake/).

Malgré la modification apportée au modèle de facturation, le modèle de tarification de l’accélération des requêtes est conçu pour réduire le coût total de possession d’une charge de travail, compte tenu de la réduction des coûts de machines virtuelles beaucoup plus élevés.

## <a name="next-steps"></a>Étapes suivantes

- [Filtrer des données à l’aide de l’accélération des requêtes d’Azure Data Lake Storage](data-lake-storage-query-acceleration-how-to.md)
- [Informations de référence sur l’accélération des requêtes en langage SQL](query-acceleration-sql-reference.md)
---
title: Présentation de Delta Lake
description: Vue d’ensemble de Delta Lake et de son fonctionnement en tant que composant d’Azure Synapse Analytics
services: synapse-analytics
author: euangMS
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: spark
ms.date: 04/15/2020
ms.author: euang
ms.reviewer: euang
ms.openlocfilehash: 6a38b61ee03aa4853526586ca60542bd3641b66f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91249397"
---
# <a name="what-is-delta-lake"></a>Présentation de Delta Lake

Azure Synapse Analytics est compatible avec Linux Foundation Delta Lake. Delta Lake est une couche de stockage open source qui apporte des transactions ACID (atomicité, cohérence, isolation et durabilité) à Apache Spark et aux charges de travail Big Data.

La version actuelle de Delta Lake incluse dans Azure Synapse offre une prise en charge des langages pour Scala, PySpark et .NET. Des liens vers pour des exemples et une documentation plus détaillés se trouvent en bas de la page.

## <a name="key-features"></a>Fonctionnalités clés

| Fonctionnalité | Description |
| --- | --- |
| **Transactions ACID** | Les lacs de données sont généralement remplis de plusieurs processus et pipelines, dont certains écrivent des données en même temps que les lectures. Avant Delta Lake et l’ajout de transactions, les ingénieurs de données devaient passer par un processus manuel sujet aux erreurs pour garantir l’intégrité des données. Delta Lake apporte des transactions ACID familières aux lacs de données. Il assure la sérialisation, le niveau d’isolation le plus élevé. Pour plus d’informations, consultez [Plongée dans Delta Lake : décompression du journal des transactions](https://databricks.com/blog/2019/08/21/diving-into-delta-lake-unpacking-the-transaction-log.html).|
| **Gestion des métadonnées évolutive** | Dans le Big Data, même les métadonnées elles-mêmes peuvent être du « Big Data ». Delta Lake traite les métadonnées de la même façon que les données, en tirant parti de la puissance de traitement distribuée de Spark pour gérer toutes ses métadonnées. Delta Lake peut donc gérer des tables à l’échelle de plusieurs pétaoctets avec des milliards de partitions et fichiers en toute simplicité. |
| **Temps de voyage (contrôle de version des données)** | La possibilité d’annuler une modification ou de revenir à une version précédente est l’une des principales fonctionnalités des transactions. Delta Lake fournit des captures instantanées de données vous permettant de revenir à des versions antérieures de données à des fins d’audit, de restauration ou de reproduction d’expériences. Pour plus d’informations, consultez [Présentation d’un voyage dans le temps Delta Lake pour les lacs de données à grande échelle](https://databricks.com/blog/2019/02/04/introducing-delta-time-travel-for-large-scale-data-lakes.html). |
| **Format ouvert** | Apache Parquet est le format de base de Delta Lake, qui vous permet de tirer parti de l’efficacité des schémas de compression et d’encodage natifs du format. |
| **Source et récepteur de lot unifié et de diffusion en continu** | Une table dans Delta Lake est une table de lots, ainsi qu’une source et un récepteur de diffusion en continu. L’ingestion de données de diffusion en continu, le renvoi d’historique par lot et les requêtes interactives fonctionnent sans nécessiter de configuration. |
| **Application du schéma** | L’application du schéma permet de s’assurer que les types de données sont corrects et que les colonnes requises sont présentes, évitant ainsi des données incorrectes entraînent une incohérence des données. Pour plus d’informations, consultez [Plongée dans Delta Lake : application et évolution du schéma](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html) |
| **Évolution du schéma** | Delta Lake vous permet d’apporter à un schéma de table des modifications qui peuvent être appliquées automatiquement, sans devoir écrire de DDL de migration. Pour plus d’informations, consultez [Plongée dans Delta Lake : application et évolution du schéma](https://databricks.com/blog/2019/09/24/diving-into-delta-lake-schema-enforcement-evolution.html) |
| **Historique des audits** | Le journal des transactions de Delta Lake enregistre des détails sur chaque modification apportée aux données, en fournissant une piste d’audit complète des modifications. |
| **Mises à jour et suppressions** | Delta Lake prend en charge les API Scala, Java, Python et SQL pour offrir un vaste éventail de fonctionnalités. La prise en charge des opérations de fusion, de mise à jour et de suppression vous aide à répondre aux exigences de conformité. Pour plus d’informations, consultez les articles [Annonce de la mise en production de Delta Lake 0.6.1](https://delta.io/news/delta-lake-0-6-1-released/) , [Annonce de la mise en production de Delta Lake 0.7](https://delta.io/news/delta-lake-0-7-0-released/) et [Upserts et suppressions simples et fiables sur des tables Delta Lake à l’aide d’API Python](https://databricks.com/blog/2019/10/03/simple-reliable-upserts-and-deletes-on-delta-lake-tables-using-python-apis.html), qui contiennent des extraits de code pour les commandes DML de fusion, de mise à jour et de suppression. |
| **100 % compatible avec l’API Apache Spark** | Les développeurs peuvent utiliser Delta Lake avec leurs pipelines de données existants pratiquement sans aucune modification, car il est entièrement compatible avec les implémentations Spark existantes. |

Pour obtenir une documentation complète, consultez la page [Documentation de Delta Lake](https://docs.delta.io/latest/delta-intro.html).

Pour plus d’informations, consultez [Projet Delta Lake](https://github.com/delta-io/delta).

## <a name="next-steps"></a>Étapes suivantes

- [Documentation .NET pour Apache Spark](/dotnet/spark?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json)
- [Azure Synapse Analytics](https://docs.microsoft.com/azure/synapse-analytics)

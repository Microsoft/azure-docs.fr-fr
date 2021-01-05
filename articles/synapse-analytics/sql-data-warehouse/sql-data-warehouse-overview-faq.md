---
title: Questions fréquentes (FAQ) sur le pool SQL dédié (anciennement SQL DW)
description: Cet article liste les questions fréquentes posées par les clients et les développeurs sur le pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 11/04/2019
ms.author: martinle
ms.reviewer: igorstan
ms.openlocfilehash: c8f8ae81f6f030245899ec82dbe16b29846dab23
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460517"
---
# <a name="dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics-frequently-asked-questions"></a>Questions fréquentes sur le pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics

## <a name="general"></a>Général

Q. Qu’est-ce qu’Azure Synapse ?

R. Azure Synapse est un service d’analytique qui regroupe l’entreposage des données et l’analytique de Big Data. Azure Synapse rassemble ces deux mondes avec une expérience unifiée pour la réception, la préparation, la gestion et la remise de données pour les besoins d’apprentissage automatique et d’aide à la décision. Pour plus d’informations, consultez l’article [Qu’est-ce qu’Azure Synapse Analytics](sql-data-warehouse-overview-what-is.md).

Q. Qu’est-il arrivé à Azure SQL Data Warehouse ?

R. Azure Synapse est une évolution d’Azure SQL Data Warehouse. Nous avons utilisé le même entrepôt de données leader du secteur pour atteindre un tout nouveau niveau de performances et de capacités. Vous pouvez continuer à exécuter vos charges de travail d’entrepôt de données existantes en production avec un pool SQL dédié (anciennement SQL DW) dans Azure Synapse. Pour plus d’informations, consultez l’article [Qu’est-ce qu’Azure Synapse Analytics](sql-data-warehouse-overview-what-is.md).

Q. Qu’est-ce qu’un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics ?

R. Un pool SQL dédié (anciennement SQL DW) fait référence aux fonctionnalités d’entreposage de données d’entreprise qui sont généralement disponibles avec Azure Synapse. Pour plus d’informations, consultez l’article [Qu’est-ce qu’Azure Synapse Analytics](sql-data-warehouse-overview-what-is.md).

Q. Comment prendre en main Azure Synapse ?

R. Pour plus d’informations, vous pouvez commencer à utiliser un [compte Azure gratuit](https://azure.microsoft.com/free/sql-data-warehouse/) ou [contacter le service commercial](https://info.microsoft.com/ww-landing-azure-sql-data-warehouse-contactme.html).

Q. Quelles sont les solutions offertes par Azure Synapse en matière de sécurité des données ?

R. Azure Synapse propose plusieurs solutions de protection des données, comme le chiffrement TDE et les audits. Pour plus d’informations, consultez [Sécurité](sql-data-warehouse-overview-manage-security.md).

Q. Où puis-je trouver les normes juridiques et commerciales auxquelles Azure Synapse est conforme ?

R. Consultez la page [Conformité Microsoft](https://www.microsoft.com/trustcenter/compliance/complianceofferings) pour connaître les différentes offres de conformité par produit, notamment SOC et ISO. Tout d’abord, sélectionnez le titre de conformité de votre choix. Ensuite, développez Azure dans la section des services cloud qui figurent dans le champ d’application de Microsoft, sur le côté droit de la page, pour voir quels sont les services conformes Azure Synapse.

Q. Puis-je me connecter à Power BI ?

R. Oui. Bien que Power BI prenne en charge les requêtes directes avec Azure Synapse, il n’est pas destiné à traiter un grand nombre d’utilisateurs ou de données en temps réel. Pour optimiser davantage les performances de Power BI, pensez à utiliser Power BI en complément d’Azure Analysis Services ou d’Analysis Service IaaS.

Q. Quelles sont les limites de capacité d’un pool SQL dédié (anciennement SQL DW) ?

R. Consultez notre page [Limites de capacité](sql-data-warehouse-service-capacity-limits.md) actuelle.

Q. Pourquoi ma Mise à l’échelle/Pause/Reprise est-elle si longue ?

R. Plusieurs facteurs peuvent influer sur la durée des opérations de gestion du calcul. Une cause fréquente des opérations durables est la restauration de transaction. Quand une opération de mise à l’échelle ou de pause est lancée, toutes les sessions entrantes sont bloquées et les requêtes sont vidées. Afin de laisser le système dans un état stable, les transactions doivent être annulées avant qu’une opération puisse commencer. Plus le nombre et la taille du journal des transactions sont importants, plus longtemps l’opération sera bloquée en attente de la restauration du système à un état stable.

## <a name="user-support"></a>Service client

Q. J’ai une suggestion de fonctionnalité, où dois-je l’envoyer ?

R. Si vous souhaitez faire une demande de fonctionnalité, envoyez-la sur notre page [UserVoice](https://feedback.azure.com/forums/307516-sql-data-warehouse).

Q. Comment faire x ?

R. Pour obtenir de l’aide concernant le développement avec Azure Synapse, vous pouvez poser des questions sur notre page [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-sqldw).

Q. Comment envoyer un ticket de support ?

R. Les [Tickets de support](sql-data-warehouse-get-started-create-support-ticket.md) peuvent être déposés sur le Portail Azure.

## <a name="sql-languagefeature-support"></a>Prise en charge de fonctionnalités / langages SQL

Q. Quels types de données sont pris en charge ?

R. Consultez [Types de données](sql-data-warehouse-tables-data-types.md).

Q. Quelles fonctionnalités de table prenez-vous en charge ?

R. De nombreuses fonctionnalités sont prises en charge. Les fonctionnalités qui ne sont pas prises en charge sont disponibles dans [Fonctionnalités de table non prises en charge](sql-data-warehouse-tables-data-types.md).

## <a name="tooling-and-administration"></a>Outils et administration

Q. Un pool SQL dédié (anciennement SQL DW) prend-il en charge les API REST ?

R. Oui. La plupart des fonctionnalités REST utilisables avec SQL Database sont également disponibles avec un pool SQL dédié (anciennement SQL DW). Vous trouverez des informations sur les API dans les pages de documentation REST ou dans les [Bases de données](/rest/api/sql/databases?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="loading"></a>Chargement

Q. Quels pilotes clients prenez-vous en charge ?

R. La prise en charge des pilotes pour un pool SQL dédié (anciennement SQL DW) est détaillée dans la page [Chaînes de connexion](sql-data-warehouse-connection-strings.md).

Q : Quels sont les formats de fichiers pris en charge par PolyBase ?

A : Orc, RC, Parquet et le texte plat délimité

Q : À quelles sources de données puis-je me connecter à l’aide de Polybase ?

A : [Azure Data Lake Storage](sql-data-warehouse-load-from-azure-data-lake-store.md) et [Azure Storage Blobs](sql-data-warehouse-load-from-azure-blob-storage-with-polybase.md)

Q : Les automates à pile sont-ils possibles lors de la connexion à Azure Storage Blobs ou à ADLS ?

A : Non, PolyBase interagit uniquement avec les composants de stockage.

Q : Puis-je me connecter à HDI ?

A : HDI peut utiliser ADLS ou WASB comme couche HDFS. Si vous avez l’un des deux comme couche HDFS, vous pouvez charger ces données dans un pool SQL dédié (anciennement SQL DW). Toutefois, vous ne peut pas générer d’automate à pile vers l’instance HDI.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le pool SQL dédié (anciennement SQL DW) dans Azure Synapse, consultez notre page [Vue d’ensemble](sql-data-warehouse-overview-what-is.md).

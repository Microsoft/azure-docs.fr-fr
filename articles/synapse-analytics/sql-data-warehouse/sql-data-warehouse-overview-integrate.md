---
title: Créer des solutions intégrées
description: Partenaires et outils de solution qui s’intègrent à un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.
services: synapse-analytics
author: mlee3gsd
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql-dw
ms.date: 04/17/2018
ms.author: martinle
ms.reviewer: igorstan
ms.custom: seo-lt-2019
ms.openlocfilehash: 2afc274bf7c040eca6a83abbab24c41767f16482
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96453665"
---
# <a name="integrate-other-services-with-a-dedicated-sql-pool-formerly-sql-dw-in-azure-synapse-analytics"></a>Intégrez d’autres services à un pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics.

La fonctionnalité de pool SQL dédié (anciennement SQL DW) dans Azure Synapse Analytics permet aux utilisateurs d’opérer une intégration à bon nombre des autres services dans Azure. Un pool SQL dédié peut utiliser plusieurs services supplémentaires, parmi lesquels :

* Power BI
* Azure Data Factory
* Azure Machine Learning
* Azure Stream Analytics

Pour plus d’informations sur les services d’intégration dans Azure, consultez l’article [Partenaires d’intégration](sql-data-warehouse-partner-data-integration.md).

## <a name="power-bi"></a>Power BI

L’intégration de Power BI vous permet de combiner la puissance de calcul d’un entrepôt de données avec la création de rapports dynamiques et la visualisation de Power BI. L’intégration de Power BI inclut actuellement les éléments suivants :

* **Connexion directe** : Connexion plus avancée avec un menu déroulant logique sur un entrepôt de données provisionné à l’aide d’un pool SQL dédié (anciennement SQL DW). Avec le pushdown, les analyses sont plus rapides à une plus grande échelle.
* **Ouvrir dans Power BI** : ce bouton transmet les informations d’instance à Power BI, permettant ainsi une connexion plus simple.

Pour plus d’informations, consultez [Intégrer à Power BI](sql-data-warehouse-get-started-visualize-with-power-bi.md) ou la [documentation Power BI](https://powerbi.microsoft.com/blog/exploring-azure-sql-data-warehouse-with-power-bi/).

## <a name="azure-data-factory"></a>Azure Data Factory

Azure Data Factory offre aux utilisateurs une plateforme gérée pour créer des pipelines d’extraction-chargement complexes. L’intégration de pool SQL dédié (anciennement SQL DW) à Azure Data Factory comprend les aspects suivants :

* **Procédures stockées** : Orchestrez l’exécution des procédures stockées.
* **Copier** : Utilisez ADF pour déplacer des données vers un pool SQL dédié (anciennement SQL DW). Cette opération peut utiliser le mécanisme de déplacement de données standard d’ADF ou PolyBase en arrière-plan.

Pour plus d’informations, consultez [Intégration dans Azure Data Factory](../../data-factory/load-azure-sql-data-warehouse.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json).

## <a name="azure-machine-learning"></a>Azure Machine Learning

Azure Machine Learning est un service d’analyse entièrement géré qui vous permet de créer des modèles complexes à l’aide d’un large ensemble d’outils prédictifs. Un pool SQL dédié (anciennement SQL DW) est pris en charge à la fois comme source et destination de ces modèles, et offre les fonctionnalités suivantes :

* **Lire les données** : piloter des modèles à grande échelle en utilisant T-SQL sur un pool SQL dédié (anciennement SQL DW).
* **Écrire les données** : valider en retour des modifications à partir de tout modèle sur un pool SQL dédié (anciennement SQL DW).

Pour plus d’informations, consultez [Intégration dans Azure Machine Learning](sql-data-warehouse-get-started-analyze-with-azure-machine-learning.md).

## <a name="azure-stream-analytics"></a>Azure Stream Analytics

Azure Stream Analytics est une infrastructure complexe, entièrement gérée pour le traitement et l’utilisation des données d’événement générées à partir d’Azure Event Hubs.  L’intégration à un pool SQL dédié (anciennement SQL DW) permet de traiter et de stocker efficacement des données de streaming avec des données relationnelles, ce qui autorise une analyse plus approfondie et plus avancée.  

* **Sortie de la tâche** : envoyez une sortie de tâches Stream Analytics directement à un pool SQL dédié (anciennement SQL DW).

Pour plus d’informations, consultez [Intégration dans Azure Stream Analytics](sql-data-warehouse-integrate-azure-stream-analytics.md).

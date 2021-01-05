---
title: 'Tutoriel : Effectuer la migration des données d’événements vers Azure Synapse Analytics - Azure Event Hubs'
description: Explique comment utiliser Azure Event Grid et Functions pour migrer des données Event Hubs capturées vers Azure Synapse Analytics.
services: event-hubs
ms.date: 12/07/2020
ms.topic: tutorial
ms.custom: devx-track-csharp
ms.openlocfilehash: 77a2b35dea57c71e9e6f07056bd106df2ac109b8
ms.sourcegitcommit: 48cb2b7d4022a85175309cf3573e72c4e67288f5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96854140"
---
# <a name="tutorial-migrate-captured-event-hubs-data-to-azure-synapse-analytics-using-event-grid-and-azure-functions"></a>Tutoriel : Effectuer la migration de données Event Hubs capturées vers Azure Synapse Analytics à l’aide d’Event Grid et d’Azure Functions
Avec Azure Event Hubs [Capture](./event-hubs-capture-overview.md), vous pouvez automatiquement capturer les données de streaming dans Event Hubs et les transmettre vers un Stockage Blob Azure ou Azure Data Lake Store. Ce tutoriel vous montre comment migrer les données Event Hubs capturées du Stockage vers Azure Synapse Analytics à l’aide d’une fonction Azure qui est déclenchée par [Event Grid](../event-grid/overview.md).

[!INCLUDE [event-grid-event-hubs-functions-synapse-analytics.md](../../includes/event-grid-event-hubs-functions-synapse-analytics.md)]

## <a name="next-steps"></a>Étapes suivantes 
Vous pouvez utiliser des outils de visualisation de données puissants avec votre Data Warehouse pour obtenir des insights actionnables.

Cet article montre comment utiliser [Power BI Azure Synapse Analytics](/power-bi/connect-data/service-azure-sql-data-warehouse-with-direct-connect).
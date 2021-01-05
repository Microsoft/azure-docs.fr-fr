---
title: Analytique Azure Batch
description: Les rubriques relatives à l’analyse en mode batch contiennent des informations de référence concernant les événements et alertes disponibles pour les ressources de service Batch.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: 0d55ecd7f10e267a9cb469dffcdf26c131c8ce41
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91849509"
---
# <a name="batch-analytics"></a>Analyse en mode batch

Les rubriques de cette section contiennent des informations de référence concernant les événements et alertes disponibles pour les ressources de service Batch.

Pour plus d’informations sur l’activation et l’utilisation des journaux de diagnostic de Batch, voir [Journalisation des diagnostics Azure Batch](./batch-diagnostics.md).

## <a name="diagnostic-logs"></a>Journaux de diagnostic

Le service Microsoft Azure Batch émet les événements de journal de diagnostic suivants pendant la durée de vie de certaines ressources Batch.

### <a name="service-log-events"></a>Événements du journal de service

- [Création de pool](batch-pool-create-event.md)
- [Début de suppression de pool](batch-pool-delete-start-event.md)
- [Fin de suppression de pool](batch-pool-delete-complete-event.md)
- [Début de redimensionnement de pool](batch-pool-resize-start-event.md)
- [Fin de redimensionnement de pool](batch-pool-resize-complete-event.md)
- [Mise à l’échelle automatique de pool](batch-pool-autoscale-event.md)
- [Début de tâche](batch-task-start-event.md)
- [Fin de tâche](batch-task-complete-event.md)
- [Échec de tâche](batch-task-fail-event.md)
- [Échec de planification de tâche](batch-task-schedule-fail-event.md)

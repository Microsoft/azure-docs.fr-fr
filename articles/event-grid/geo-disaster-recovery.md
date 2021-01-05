---
title: Géorécupération d’urgence dans Azure Event Grid | Microsoft Docs
description: Décrit comment Azure Event Grid prend en charge la géorécupération d’urgence (GeoDR) automatiquement.
ms.topic: conceptual
ms.date: 11/19/2020
ms.openlocfilehash: 10beaf0ae25f3ed9b7bcda5961a89494b18b84d9
ms.sourcegitcommit: f311f112c9ca711d88a096bed43040fcdad24433
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94980846"
---
# <a name="server-side-geo-disaster-recovery-in-azure-event-grid"></a>Géorécupération d’urgence côté serveur dans Azure Event Grid
Event Grid comprend à présent la géorécupération d’urgence (GeoDR) des métadonnées, non seulement pour les nouveaux, mais également pour tous les domaines, rubriques et abonnements aux événements existants. Si une région Azure entière tombe en panne, Event Grid disposera déjà de toutes vos métadonnées d’infrastructure liées aux événements synchronisées avec une région jumelée. Les nouveaux événements recommenceront à circuler sans intervention de votre part. 

La récupération d’urgence est mesurée à l’aide de deux métriques :

- Objectif de point de récupération (RPO) : minutes ou heures de données pouvant être perdues.
- Objectif de temps de récupération (RTO) : nombre de minutes ou d’heures pendant lesquelles le service peut être arrêté.

Le basculement automatique d’Event Grid utilise différents objectifs RPO et RTO pour vos métadonnées (abonnements aux événements, etc.) et vos données (événements). Si vous avez besoin d’une spécification différente de celles ci-dessous, vous pouvez toujours implémenter votre propre [basculement côté client à l’aide des API d’intégrité de rubrique](custom-disaster-recovery.md).

## <a name="recovery-point-objective-rpo"></a>Objectif de point de récupération (RPO)
- **RPO de métadonnées** : zéro minutes. Chaque fois qu’une ressource est créée dans Event Grid, elle est répliquée instantanément dans toutes les régions. Lorsqu’un basculement se produit, aucune métadonnée n’est perdue.
- **RPO de données** : si votre système est sain et a rattrapé le trafic existant au moment du basculement régional, le RPO pour les événements est d’environ 5 minutes.

## <a name="recovery-time-objective-rto"></a>Objectif de délai de récupération (RTO)
- **RTO de métadonnées** : bien que cela se produise généralement beaucoup plus rapidement, dans les 60 minutes qui suivent, Event Grid commence à accepter les appels à créer/mettre à jour/supprimer des rubriques et des abonnements.
- **RTO de données** : comme les métadonnées, cela se produit généralement beaucoup plus rapidement. Toutefois, Event Grid commence à accepter le nouveau trafic dans les 60 minutes suivant un basculement régional.

> [!NOTE]
> Le coût de GeoDR métadonnées sur Event Grid est de 0 $.


## <a name="next-steps"></a>Étapes suivantes
Si vous souhaitez implémenter votre propre logique de basculement côté client, consultez [Créer votre propre récupération d’urgence pour les rubriques personnalisées dans Event Grid](custom-disaster-recovery.md)

---
title: Azure Database pour PostgreSQL – Serveur flexible (préversion) – Maintenance planifiée – Portail Azure
description: Découvrez comment configurer les paramètres de maintenance planifiée pour Azure Database pour PostgreSQL – Serveur flexible à partir du portail Azure.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: be6040b8b84a4b86746d62bd2f1c07f0ffea0a3b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91336290"
---
# <a name="manage-scheduled-maintenance-settings-for-azure-database-for-postgresql--flexible-server"></a>Gérer les paramètres de maintenance planifiée pour Azure Database pour PostgreSQL – Serveur flexible
 
Vous pouvez spécifier des options de maintenance pour chaque serveur flexible dans votre abonnement Azure. Les options incluent la planification de maintenance et les paramètres de notification pour les événements de maintenance à venir et terminés.

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est en préversion.

## <a name="prerequisites"></a>Prérequis
Pour utiliser ce guide pratique, il vous faut :
- Un [Azure Database pour PostgreSQL – Serveur flexible](quickstart-create-server-portal.md)
 
## <a name="specify-maintenance-schedule-options"></a>Spécifier les options de planification de la maintenance
 
1. Dans la page du serveur PostgreSQL, sous le titre **Paramètres**, choisissez **Maintenance** pour ouvrir les options de maintenance planifiée.
2. La planification par défaut (gérée par le système) est un jour aléatoire de la semaine et une fenêtre de 60 minutes pour le début la maintenance entre 23 h et 7 H, heure du serveur local. Si vous souhaitez personnaliser cette planification, choisissez **Planification personnalisée**. Vous pouvez ensuite sélectionner un jour par défaut de la semaine et une fenêtre de 60 minutes pour l’heure de début de la maintenance.
 
## <a name="notifications-about-scheduled-maintenance-events"></a>Notifications relatives aux événements de maintenance planifiée
 
Vous pouvez utiliser Azure Service Health pour [afficher les notifications](../../service-health/service-notifications.md) sur la maintenance planifiée à venir et effectuée sur votre serveur flexible. Vous pouvez également [configurer](../../service-health/resource-health-alert-monitor-guide.md) des alertes dans Azure Service Health pour obtenir des notifications sur les événements de maintenance.
 
## <a name="next-steps"></a>Étapes suivantes  
 
* En savoir plus sur la [maintenance planifiée dans Azure Database pour PostgreSQL – Serveur flexible](concepts-maintenance.md)
* En savoir plus sur [Azure Service Health](../../service-health/overview.md)

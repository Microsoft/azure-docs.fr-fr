---
title: Utiliser Azure Resource Health pour effectuer le monitorage de l’intégrité de la base de données
description: Utilisez Azure Resource Health pour analyser l’intégrité d’Azure SQL Database et Azure SQL Managed Instance vous aide à diagnostiquer les problèmes et à accéder au support quand un problème Azure a une incidence sur vos ressources SQL.
services: sql-database
ms.service: sql-db-mi
ms.subservice: performance
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: conceptual
author: aamalvea
ms.author: aamalvea
ms.reviewer: wiassaf, sstein
ms.date: 02/26/2019
ms.openlocfilehash: a51840daa2c8b19c323ad761ff6e6d1dcef15f75
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497960"
---
# <a name="use-resource-health-to-troubleshoot-connectivity-for-azure-sql-database-and-azure-sql-managed-instance"></a>Utiliser Resource Health pour détecter un problème de connectivité avec Azure SQL Database et Azure SQL Managed Instance
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

[Resource Health](../../service-health/resource-health-overview.md#get-started) pour Azure SQL Database et Azure SQL Managed Instance vous aide à diagnostiquer les problèmes et à accéder au support quand un problème Azure a une incidence sur vos ressources SQL. Il vous informe de l’intégrité (actuelle et passée) de vos ressources et vous aide à atténuer les problèmes. Resource Health propose un support technique dès lors que vous êtes confronté à des problèmes de service Azure et que vous avez besoin d’aide.

![Vue d’ensemble](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-overview.jpg)

## <a name="health-checks"></a>Contrôles d'intégrité

Resource Health détermine l’intégrité de votre ressource SQL en examinant la réussite et échec des connexions à la ressource. Actuellement, Resource Health pour votre ressource SQL Database examine uniquement les échecs de connexion dus à une erreur système et non à une erreur utilisateur. L’état Resource Health est mis à jour toutes les une à deux minutes.

## <a name="health-states"></a>États d'intégrité

### <a name="available"></a>Disponible

Un état **Disponible** signifie que Resource Health n’a pas détecté d’échecs de connexion dus à des erreurs système sur votre ressource SQL.

![Disponible](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-available.jpg)

### <a name="degraded"></a>Détérioré

Un état **Détérioré** signifie que Resource Health a détecté une majorité de connexions établies, mais également des échecs. Il s’agit probables erreurs de connexion temporaires. Pour réduire l’impact des problèmes de connexion dus à des erreurs de connexion temporaires, implémentez une [logique de nouvelle tentative](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) dans votre code.

![Détérioré](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-degraded.jpg)

### <a name="unavailable"></a>Non disponible

État **Indisponible** signifie que Resource Health a détecté des échecs de connexion cohérente dans votre ressource SQL. Si votre ressource reste dans cet état pendant une période prolongée, contactez le support technique.

![Non disponible](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-unavailable.jpg)

### <a name="unknown"></a>Unknown

L’état d’intégrité **Inconnu** indique que Resource Health n’a reçu aucune information sur cette ressource depuis plus de 10 minutes. Même si cet état n’est pas une indication définitive de l’état de la ressource, il s’agit d’un point de données important dans le processus de dépannage. Si la ressource fonctionne comme prévu, son état devient Disponible après quelques minutes. Si vous rencontrez des problèmes avec la ressource, l’état d’intégrité Inconnu peut suggérer qu’un événement de la plateforme influe sur la ressource.

![Unknown](./media/resource-health-to-troubleshoot-connectivity/sql-resource-health-unknown.jpg)

## <a name="historical-information"></a>Informations d’historique

Vous pouvez accéder à jusqu’à 14 jours d’historique d’intégrité dans la section Historique de l’intégrité de Resource Health. La section contient également la raison de l’arrêt (le cas échéant) signalé par Resource Health. Actuellement, Azure indique le temps d’arrêt de votre ressource de base de données avec une granularité de deux minutes. Le temps d’arrêt réel est probablement inférieur à une minute. La moyenne est de huit secondes.

### <a name="downtime-reasons"></a>Raisons des temps d’arrêt

En cas de temps d’arrêt de votre base de données, une analyse est effectuée pour en déterminer la cause. Lorsqu’elle est disponible, la raison du temps d’arrêt est signalée dans la section Historique de l’intégrité de Resource Health. Les raisons des temps d’arrêt sont généralement publiées 30 minutes après un événement.

#### <a name="planned-maintenance"></a>Maintenance planifiée

L’infrastructure Azure effectue régulièrement une maintenance planifiée (mise à niveau des composants matériels ou logiciels dans le centre de données). Pendant la maintenance de la base de données, Azure SQL peut mettre fin à des connexions existantes et en refuser de nouvelles. Les échecs de connexion rencontrés pendant une maintenance planifiée sont généralement temporaires. Une [logique de nouvelle tentative](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) permet d’en réduire l’impact. Si vous êtes toujours confronté à des erreurs de connexion, contactez le support technique.

#### <a name="reconfiguration"></a>Reconfiguration

Les reconfigurations sont considérées comme des conditions transitoires et prévues de temps à autre. Ces événements peuvent être déclenchés par les échecs d’équilibrage de charge ou de logiciel/matériel. Toute application de production client qui se connecte à une base de données cloud doit implémenter une [logique de nouvelle tentative](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors) de connexion robuste, qui aide à remédier à ces situations et rend généralement les erreurs transparentes pour l’utilisateur final.

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur la [logique de nouvelle tentative pour les erreurs temporaires](troubleshoot-common-connectivity-issues.md#retry-logic-for-transient-errors).
- [Diagnostiquer, résoudre et empêcher les erreurs de connexion SQL](troubleshoot-common-connectivity-issues.md).
- Apprenez-en davantage sur la [configuration des alertes Resource Health](../../service-health/resource-health-alert-arm-template-guide.md).
- Découvrez une vue d’ensemble de [Resource Health](../../application-gateway/resource-health-overview.md).
- Consultez le [FAQ Resource Health](../../service-health/resource-health-faq.md).

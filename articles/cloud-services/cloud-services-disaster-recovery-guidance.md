---
title: Gestion d’une interruption de service Azure qui affecte Azure Cloud Services
titleSuffix: Azure Cloud Services
description: Découvrez que faire si une interruption de service Azure affecte Azure Cloud Services.
services: cloud-services
documentationcenter: ''
author: mmccrory
ms.service: cloud-services
ms.workload: cloud-services
ms.topic: article
ms.date: 04/04/2017
ms.author: memccror
ms.openlocfilehash: 6ae1509d552de1d5473c7d995af2db68d7113e79
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92077522"
---
# <a name="what-to-do-in-the-event-of-an-azure-service-disruption-that-impacts-azure-cloud-services"></a>Que faire si une interruption de service Azure affecte Azure Cloud Services
Microsoft s’engage à déployer tous les efforts nécessaires pour vous garantir en permanence la disponibilité de ses services quand vous en avez besoin. Il arrive parfois que des phénomènes incontrôlables entraînent des interruptions de service non planifiées.

Microsoft fournit un Contrat de niveau de service (SLA) pour ses services en guise d’engagement en matière de disponibilité et de connectivité. Le contrat de niveau de service des différents services Azure se trouve à la page [Contrats de niveau de service Azure](https://azure.microsoft.com/support/legal/sla/).

Azure dispose déjà de nombreuses fonctionnalités intégrées de plateforme qui prennent en charge des applications hautement disponibles. Pour plus d’informations sur ces services, consultez [Récupération d’urgence et haute disponibilité pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery).

Cet article aborde un scénario réel de récupération d’urgence, dans lequel une région entière connaît une panne en raison d’une catastrophe naturelle majeure ou d’une interruption de service importante. Bien que ces cas soient rares, vous devez envisager l’éventualité d’une panne affectant l’ensemble d’une région. Si une région entière est confrontée à une interruption de service, les copies localement redondantes de vos données sont temporairement indisponibles. Si vous avez activé la géoréplication, trois copies supplémentaires de vos tables et objets blob Azure Storage sont stockées dans une autre région. En cas de panne régionale totale ou de sinistre rendant la région primaire irrécupérable, Azure remappe toutes les entrées DNS sur la région géorépliquée.

> [!NOTE]
> N’oubliez pas que vous n’avez aucun contrôle sur ce processus et qu’il ne se produit que pour les interruptions du service au niveau du centre de données. Ainsi, vous devez également vous appuyer sur d’autres stratégies de sauvegarde propres à l’application pour atteindre le plus haut niveau de disponibilité. Pour plus d’informations, consultez [Récupération d’urgence et haute disponibilité des applications développées sur Microsoft Azure](/azure/architecture/framework/resiliency/backup-and-recovery). Si vous souhaitez avoir un impact sur votre propre basculement, vous pouvez envisager l’utilisation du [stockage géoredondant avec accès en lecture (RA-GRS)](../storage/common/storage-redundancy.md), qui crée une copie en lecture seule de vos données dans une autre région.
>
>


## <a name="option-1-use-a-backup-deployment-through-azure-traffic-manager"></a>Option 1 : utilisation d’un déploiement de sauvegarde par le biais d’Azure Traffic Manager
La solution de récupération d’urgence la plus robuste consiste à assurer la maintenance de plusieurs déploiements de votre application dans différentes régions, puis à utiliser [Azure Traffic Manager](../traffic-manager/traffic-manager-overview.md) pour diriger le trafic entre eux. Azure Traffic Manager fournit plusieurs [méthodes de routage](../traffic-manager/traffic-manager-routing-methods.md), de sorte que vous pouvez choisir de gérer vos déploiements à l’aide d’un modèle principal/de sauvegarde ou de fractionner le trafic entre eux.

![Équilibrage d’Azure Cloud Services entre différentes régions avec Azure Traffic Manager](./media/cloud-services-disaster-recovery-guidance/using-azure-traffic-manager.png)

Pour obtenir la réponse la plus rapide suite à la perte d’une région, nous vous recommandons fortement de configurer la [surveillance des points de terminaison](../traffic-manager/traffic-manager-monitoring.md) Traffic Manager.

## <a name="option-2-deploy-your-application-to-a-new-region"></a>Option 2 : déploiement de votre application dans une nouvelle région
Maintenir plusieurs déploiements actifs comme décrit dans l’option précédente entraîne des coûts supplémentaires. Si votre objectif de temps de récupération est suffisamment flexible et que vous disposez du code d’origine ou du package Services cloud compilé, vous pouvez créer une instance de votre application dans une autre région et mettre à jour vos enregistrements DNS de façon à ce qu’ils pointent vers le nouveau déploiement.

Pour plus d’informations sur la façon de créer et de déployer une application de service cloud, consultez [Création et déploiement d’un service cloud](cloud-services-how-to-create-deploy-portal.md).

Suivant vos sources de données d’application, vous pouvez être amené à vérifier les procédures de récupération.

* Pour les sources de données Azure Storage, voir [Redondance de Stockage Azure](../storage/common/storage-redundancy.md) pour vérifier les options disponibles selon le modèle de redondance choisi pour votre application.
* Pour les sources SQL Database, consultez [Vue d’ensemble : continuité des activités cloud et récupération d’urgence d’une base de données avec SQL Database](../azure-sql/database/business-continuity-high-availability-disaster-recover-hadr-overview.md) pour vérifier les options disponibles selon le modèle de réplication choisi pour votre application.


## <a name="option-3-wait-for-recovery"></a>Option 3 : attente de récupération
Dans ce cas, aucune action de votre part n’est requise, mais votre service est indisponible jusqu’à ce que la région soit restaurée. Vous pouvez consulter l’état actuel du service dans le [tableau de bord d’état du service Azure](https://azure.microsoft.com/status/).

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur la façon d’implémenter une stratégie de récupération d’urgence et de haute disponibilité, consultez [Récupération d’urgence et haute disponibilité pour les applications Azure](/azure/architecture/framework/resiliency/backup-and-recovery).

Pour une compréhension technique détaillée des fonctionnalités de la plateforme cloud, consultez le [Guide technique de la résilience Azure](/azure/architecture/checklist/resiliency-per-service).
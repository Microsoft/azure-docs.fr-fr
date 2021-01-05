---
title: Présentation de Service Health | Microsoft Docs
description: Découvrez comment Service Health fournit un tableau de bord personnalisable qui suit l’intégrité de vos services Azure dans les régions où vous les utilisez.
ms.topic: conceptual
ms.date: 05/10/2019
ms.openlocfilehash: 8246b0ab93b95c13858e4ff96d0f24b255d05e55
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90967788"
---
# <a name="service-health-overview"></a>Présentation de Service Health

Service Health fournit un tableau de bord personnalisable qui suit l’intégrité de vos services Azure dans les régions où vous les utiliser. Dans ce tableau de bord, vous pouvez suivre les événements actifs, notamment les problèmes de service en cours, la maintenance planifiée à venir ou les conseils d’intégrité pertinents. Lorsque des événements deviennent inactifs, ils sont placés dans votre historique d’intégrité pendant 90 jours. Enfin, vous pouvez utiliser le tableau de bord Service Health pour créer et gérer des alertes d’intégrité de service qui vous informent de façon proactive des problèmes de service vous concernant.

## <a name="service-health-events"></a>Événements Service Health

Service Health suit quatre types d'événements d'intégrité qui peuvent avoir une incidence sur vos ressources :

1. **Problèmes liés aux services** : problèmes touchant les services Azure qui vous affectent en ce moment même. 
2. **Maintenance planifiée** : maintenance à venir qui pourra avoir une incidence sur la disponibilité de vos services.  
3. **Health advisories** (Avis concernant l’intégrité) : modifications apportées aux services Azure qui nécessitent votre attention. Il peut s’agir par exemple de la dépréciation de fonctionnalités Azure ou d’exigences de mise à niveau (par exemple une mise à niveau vers un framework PHP pris en charge).
4. **Avis de sécurité** : notifications liées à la sécurité ou les violations qui peuvent affecter la disponibilité de vos services Azure.

> [!NOTE]
> Pour voir les événements Service Health, les utilisateurs doivent [avoir le rôle Lecteur](../role-based-access-control/role-assignments-portal.md) dans un abonnement.

## <a name="get-started-with-service-health"></a>Prise en main de Service Health

Pour lancer votre tableau de bord Service Health, sélectionnez la vignette Service Health dans le tableau de bord du portail. Si vous avez supprimé la vignette ou utilisez un tableau de bord personnalisé, recherchez Service Health dans Plus de services (en bas à gauche de votre tableau de bord).

![Prise en main de Service Health](./media/service-health-overview/azure-service-health-overview-1.png)

## <a name="see-current-issues-which-impact-your-services"></a>Afficher les problèmes actuels qui ont une incidence sur vos services

La vue **Problèmes liés aux services** affiche tous les problèmes touchant actuellement les services Azure qui ont une incidence sur vos ressources. Vous pouvez savoir quand le problème a commencé et quels services et régions sont concernés. Vous pouvez également accéder aux informations les plus récentes pour comprendre ce qui est fait pour résoudre le problème. 

[![Gérer des problèmes liés au service](./media/service-health-overview/azure-service-health-overview-2.png)](./media/service-health-overview/azure-service-health-overview-2.png#lightbox)

Choisissez l’onglet **Impact potentiel** pour afficher la liste spécifique des ressources que vous détenez pouvant être affectées par le problème. Vous pouvez télécharger une liste au format CSV de ces ressources pour la partager avec votre équipe.

[![Gestion des problèmes liés au service - Impact](./media/service-health-overview/azure-service-health-overview-4.png)](./media/service-health-overview/azure-service-health-overview-4.png#lightbox)

## <a name="see-emerging-issues-which-may-impact-your-services"></a>Consultez les nouveaux problèmes qui peuvent avoir un impact sur vos services

Dans certaines situations, des problèmes de service étendus peuvent être publiés dans la [page d’état Azure](https://status.azure.com) avant que les communications ciblées puissent être envoyées aux clients affectés. Pour être sûr qu’Azure Service Health fournit une vue complète des problèmes susceptibles de vous affecter, les problèmes de page d’état Azure actifs sont signalés dans Service Health en tant que *nouveaux problèmes*. Quand un événement est actif dans la page d’état Azure, une bannière de nouveaux problèmes est présente dans Service Health. Cliquez sur cette bannière pour afficher les détails complets du problème.

![Nouveau problème de service](./media/service-health-overview/azure-service-health-emerging-issue.png)

## <a name="get-links-and-downloadable-explanations"></a>Obtenir des liens et des explications téléchargeables 

Vous pouvez obtenir un lien pour le problème afin de l’utiliser dans votre système de gestion des problèmes. Vous pouvez télécharger des fichiers PDF et parfois CSV pour les partager avec des collaborateurs qui n'ont pas accès au portail Azure.   

[![Gestion des problèmes liés aux services - Gestion des problèmes](./media/service-health-overview/azure-service-health-overview-3.png)](./media/service-health-overview/azure-service-health-overview-3.png#lightbox)

## <a name="get-support-from-microsoft"></a>Obtenir l’aide du support Microsoft

Contactez le support si votre ressource présente encore un état incorrect après la résolution du problème.  Utilisez les liens de support sur la droite de la page.  

## <a name="pin-a-personalized-health-map-to-your-dashboard"></a>Épingler une carte d’intégrité personnalisée à votre tableau de bord

Filtrez Service Health pour afficher vos abonnements, régions et types de ressources stratégiques. Enregistrez le filtre et épinglez une carte mondiale d’intégrité personnalisée à votre tableau de bord du portail. 

[![Mappage de l’intégrité personnalisée des filtres](./media/service-health-overview/azure-service-health-overview-6a.png)](./media/service-health-overview/azure-service-health-overview-6a.png#lightbox)

![Épingler un mappage de l’intégrité personnalisée](./media/service-health-overview/azure-service-health-overview-6b.png)

## <a name="configure-service-health-alerts"></a>Configurer des alertes d’intégrité de service

Service Health s’intègre à Azure Monitor pour vous avertir au moyen d’e-mails, de SMS et de notifications webhook quand certaines de vos ressources stratégiques sont impactées. Configurez une alerte de journal d’activité pour l’événement d’intégrité de service approprié. Acheminez cette alerte aux personnes appropriées de votre organisation à l’aide de groupes d’actions. Pour plus d’informations sur la configuration d’alertes pour Service Health, consultez [cet article](./alerts-activity-log-service-notifications-portal.md).

>[!VIDEO https://www.microsoft.com/en-us/videoplayer/embed/RE2OaXt]

## <a name="next-steps"></a>Étapes suivantes

Configurez des alertes afin d’être averti des problèmes d’intégrité. Pour plus d’informations, consultez les [bonnes pratiques de configuration d’alertes Azure Service Health](https://www.youtube.com/watch?v=k5d5ca8K6tc&list=PLLasX02E8BPBBSqygdRvlTnHfp1POwE8K&index=6&t=0s). 

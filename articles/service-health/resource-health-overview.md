---
title: Vue d’ensemble d’Azure Resource Health
description: Découvrez comment Azure Resource Health vous aide à diagnostiquer et à obtenir un support pour les problèmes de service qui affectent vos ressources Azure.
ms.topic: conceptual
ms.date: 05/10/2019
ms.openlocfilehash: 25425eadfaf54ba56cde3b377a44cd85c2b68959
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90967881"
---
# <a name="resource-health-overview"></a>Vue d’ensemble de Resource Health
 
Azure Resource Health vous aide à diagnostiquer et à obtenir un support pour les problèmes de service qui affectent vos ressources Azure. Il rend compte de l’intégrité actuelle et passée de vos ressources.

Rapports d’[état Azure](https://status.azure.com) sur les problèmes de service qui affectent un large éventail de clients Azure. Resource Health vous donne un tableau de bord personnalisé de l’intégrité de vos ressources. Resource Health vous montre toutes les fois où vos ressources ont été non disponibles en raison de problèmes de service Azure. Ces données vous permettent de voir facilement si un contrat SLA n’a pas été respecté.

## <a name="resource-definition-and-health-assessment"></a>Définition des ressources et évaluation de l’intégrité

Une *ressource* est une instance spécifique d’un service Azure : par exemple, une machine virtuelle, une application web ou une base de données SQL. Resource Health s’appuie sur des signaux émis par les différents services Azure pour évaluer l’intégrité d’une ressource. Si une ressource n’est pas intègre, Resource Health analyse des informations supplémentaires pour déterminer la source du problème. Il indique également les actions que Microsoft prend pour résoudre le problème et identifie les éléments que vous pouvez faire pour y remédier.

Pour plus d’informations sur la procédure d’évaluation de l’intégrité, passez en revue la liste complète de types de ressource et de vérifications d’intégrité dans [Azure Resource Health](resource-health-checks-resource-types.md).

## <a name="health-status"></a>État d’intégrité

L’intégrité d’une ressource présente l’un des états suivants.

### <a name="available"></a>Disponible

*Disponible* signifie qu’il n’y a aucun événement détecté qui affecte l’intégrité de la ressource. Quand la ressource a été récupérée après un temps d’arrêt non planifié au cours des dernières 24 heures, vous voyez la notification « Résolution récente ».

![État « Disponible » pour une machine virtuelle avec une notification « Résolution récente »](./media/resource-health-overview/Available.png)

### <a name="unavailable"></a>Non disponible

*Non disponible* signifie que le service a détecté un événement de plateforme ou hors plateforme en cours ayant un impact sur l’intégrité de la ressource.

#### <a name="platform-events"></a>Événements de plateforme

Les événements de plateforme sont déclenchés par plusieurs composants de l’infrastructure Azure. Ils incluent à la fois les actions planifiées (par exemple, une maintenance planifiée) et les incidents inattendus (par exemple, un redémarrage imprévu de l’hôte ou un matériel hôte dégradé dont on prévoit qu’il tombera en panne après une période donnée).

Resource Health fournit des détails supplémentaires sur l’événement et le processus de récupération. Vous pouvez aussi contacter le support Microsoft même si vous n’avez pas de contrat de support actif.

![État « Non disponible » pour une machine virtuelle en raison d’un événement de plateforme](./media/resource-health-overview/Unavailable.png)

#### <a name="non-platform-events"></a>Événements hors plateforme

Les événements hors plateforme sont déclenchés par des actions effectuées par les utilisateurs. Par exemple, l’arrêt d’une machine virtuelle ou l’utilisation du nombre maximal de connexions à un cache Azure pour Redis.

![État « Non disponible » pour une machine virtuelle en raison d’un événement hors plateforme](./media/resource-health-overview/Unavailable_NonPlatform.png)

### <a name="unknown"></a>Unknown

*Inconnu* signifie que Resource Health n’a reçu aucune information sur cette ressource depuis plus de 10 minutes. Même si cet état n’est pas une indication définitive de l’état de la ressource, il s’agit d’un point de données important dans la résolution des problèmes.

Si la ressource fonctionne comme prévu, son état est devient *Disponible* après quelques minutes.

Si vous rencontrez des problèmes avec la ressource, l’état d’intégrité *Inconnu* peut suggérer qu’un événement de la plateforme influe sur la ressource.

![État « Inconnu » pour une machine virtuelle](./media/resource-health-overview/Unknown.png)

### <a name="degraded"></a>Détérioré

*Détérioré* signifie que votre ressource a détecté une perte de performances, bien qu’elle soit toujours disponible à l’utilisation.

Les diverses ressources ont leurs propres critères pour établir un rapport spécifiant qu’une ressource est détériorée.

![État « Détérioré » pour une machine virtuelle](./media/resource-health-overview/degraded.png)

## <a name="reporting-an-incorrect-status"></a>Signalement d’un état incorrect

Si vous pensez que l’état d’intégrité actuel est incorrect, vous pouvez nous le faire savoir en sélectionnant **Signaler un état d’intégrité incorrect**. En cas de problème avec Azure, nous vous invitons à contacter le support à partir de Resource Health.

![Formulaire pour envoyer des informations sur un état incorrect](./media/resource-health-overview/incorrect-status.png)

## <a name="history-information"></a>Informations d’historique

Vous pouvez consulter jusqu’à 30 jours d’historique dans la section **Historique de l’intégrité** de Resource Health.

![Liste des événements Resource Health sur les deux dernières semaines](./media/resource-health-overview/history-blade.png)

## <a name="get-started"></a>Bien démarrer

Pour ouvrir Resource Health pour une ressource, procédez comme suit :

1. Connectez-vous au portail Azure.
2. Accédez à votre ressource.
3. Dans le menu de la ressource dans le volet gauche, sélectionnez **Resource Health**.

![Ouverture de Resource Health à partir de la vue de ressource](./media/resource-health-overview/from-resource-blade.png)

Vous pouvez également accéder à Resource Health en sélectionnant **Tous les services** et en saisissant **Resource Health** dans la zone de texte de filtre. Dans le volet **Aide + Support**, sélectionnez [Resource Health](https://ms.portal.azure.com/#blade/Microsoft_Azure_Monitoring/AzureMonitoringBrowseBlade/resourceHealth).

![Ouverture de Resource Health à partir de « Tous les services »](./media/resource-health-overview/FromOtherServices.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Resource Health, consultez les références suivantes :
-  [Types de ressources et les contrôles d’intégrité dans Azure Resource Health](resource-health-checks-resource-types.md)
-  [Forum aux questions sur Azure Resource Health](resource-health-faq.md)

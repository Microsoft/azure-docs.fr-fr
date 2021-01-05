---
title: Haute disponibilité et récupération d’urgence
titleSuffix: Azure Digital Twins
description: Décrit les options Azure et Azure Digital Twins qui vous aident à créer des solutions Azure IoT à haute disponibilité dotées de fonctionnalités de récupération d’urgence.
author: baanders
ms.author: baanders
ms.date: 10/14/2020
ms.topic: conceptual
ms.service: digital-twins
ms.openlocfilehash: 35f4aae246f105d832aaf92c5c5797c8a65b44f1
ms.sourcegitcommit: dea56e0dd919ad4250dde03c11d5406530c21c28
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96938541"
---
# <a name="azure-digital-twins-high-availability-and-disaster-recovery"></a>Haute disponibilité et récupération d’urgence pour Azure Digital Twins

La continuité d'activité et reprise d'activité constitue un domaine clé à prendre en compte pour des solutions IoT résilientes. Une conception pensée pour la **haute disponibilité (HA)** et la **récupération d’urgence (DR)** peut vous aider à définir et atteindre les objectifs de temps de disponibilité appropriés pour votre solution.

Cet article décrit les fonctionnalités de haute disponibilité et de récupération d’urgence offertes par le service Azure Digital Twins.

Azure Digital Twins prend en charge les options de fonctionnalité suivantes :
* *HA intra-région* : redondance intégrée pour assurer le temps de disponibilité du service
* *DR inter-régions* : basculement vers une région Azure associée géographiquement dans le cas d’une défaillance inattendue du centre de données

Vous pouvez également consulter la section [*Meilleures pratiques*](#best-practices) pour obtenir des conseils généraux sur la conception de haute disponibilité/récupération d’urgence.

## <a name="intra-region-ha"></a>Haute disponibilité intra-région
 
Azure Digital Twins fournit la haute disponibilité intra-région en implémentant des redondances au sein du service. Cela se répercute dans le [contrat SLA du service](https://azure.microsoft.com/support/legal/sla/digital-twins) pour le temps d’activité. **Les développeurs d’une solution Azure Digital Twins peuvent tirer parti de ces fonctionnalités de haute disponibilité sans le moindre effort supplémentaire.** Bien qu’Azure Digital Twins offre une garantie de disponibilité raisonnablement élevée, des défaillances temporaires peuvent toujours se produire comme avec n’importe quelle plateforme informatique distribuée. Des stratégies de nouvelle tentative appropriées doivent être intégrées dans les composants qui interagissent avec une application cloud de manière à gérer les défaillances temporaires.

## <a name="cross-region-dr"></a>Récupération d’urgence inter-région

Il peut arriver à de rares occasions qu’un centre de données connaisse une interruption prolongée en raison de pannes d’alimentation ou d’autres événements dans la région. Ces événements se produisent rarement et, lors de tels problèmes, la fonction de haute disponibilité intra-région décrite ci-dessus ne peut pas nécessairement s’appliquer dans ces situations. Azure Digital Twins répond à cela avec le basculement initié par Microsoft.

Le **basculement initié par Microsoft** est déclenché par Microsoft dans de rares situations pour faire basculer tous les Azure Digital Twins entre une région affectée et la région géographiquement associée. Ce processus représente l’option par défaut (sans aucun moyen pour les utilisateurs d’y renoncer) et ne nécessite aucune intervention de la part de l’utilisateur. Microsoft se réserve le droit de déterminer dans quelles circonstances exercer cette option. Ce mécanisme n’implique pas le consentement de l’utilisateur avant le basculement de l’instance de l’utilisateur.

>[!NOTE]
> Certains services Azure fournissent également une option supplémentaire appelée **basculement initié par le client**, qui permet aux clients de lancer un basculement uniquement pour leur instance, par exemple pour exécuter un exercice de récupération d’urgence. Ce mécanisme n’est actuellement **pas pris en charge** par Azure Digital Twins. 

## <a name="monitor-service-health"></a>Suivi de l’intégrité des services

À mesure que les instances Azure Digital Twins sont basculées et récupérées, vous pouvez surveiller le processus à l’aide de l’outil [Azure Service Health](https://docs.microsoft.com/azure/service-health/service-health-overview). Service Health effectue le suivi de l’intégrité de vos services Azure dans différentes régions et abonnements, et partage les communications ayant un impact sur le service concernant les pannes et les temps d’arrêt.

Au cours d’un événement de basculement, Service Health peut indiquer quand votre service est à l’arrêt, et quand il redevient opérationnel.

Pour afficher les événements Service Health...
1. Accédez à [Service Health](https://portal.azure.com/?feature.customportal=false#blade/Microsoft_Azure_Health/AzureHealthBrowseBlade/serviceIssues) dans le portail Azure (vous pouvez utiliser ce lien ou rechercher le service à l’aide de la barre de recherche du portail).
1. Utilisez le menu de gauche pour basculer vers la page de l’*historique d’intégrité*.
1. Recherchez un *nom de problème* commençant par **Azure Digital Twins**, puis sélectionnez-le.

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/navigate.png" alt-text="Capture d’écran du portail Azure montrant la page de l’historique d’intégrité. Il y a là une liste de plusieurs problèmes survenus au cours des derniers jours, avec un problème nommé « Azure Digital Twins - West Europe - Mitigated » en surbrillance." lightbox="media/concepts-high-availability-disaster-recovery/navigate.png":::

1. Pour obtenir des informations générales sur la panne, affichez l’onglet *Résumé*.

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/summary.png" alt-text="Dans la page de l’historique d’intégrité, l’onglet Résumé est en surbrillance. L’onglet affiche des informations générales, telles que la ressource affectée, sa région et son abonnement." lightbox="media/concepts-high-availability-disaster-recovery/summary.png":::
1. Pour obtenir plus d’informations et des mises à jour concernant l’évolution du problème au fil du temps, consultez l’onglet des *mises à jour du problème*.

    :::image type="content" source="media/concepts-high-availability-disaster-recovery/issue-updates.png" alt-text="Dans la page de l’historique d’intégrité, l’onglet des mises à jour est en surbrillance. L’onglet affiche plusieurs entrées indiquant l’évolution des états depuis un jour jusqu’à l’état actuel." lightbox="media/concepts-high-availability-disaster-recovery/issue-updates.png":::


Notez que les informations affichées dans cet outil ne sont pas spécifiques d’une seule instance Azure Digital. Après avoir utilisé Service Health pour comprendre ce qui se passe avec le service Azure Digital Twins dans une région ou un abonnement sonnés, vous pouvez effectuer une analyse plus poussée à l’aide de l’outil [Resource Health](troubleshoot-resource-health.md) pour explorer des instances spécifiques et voir si elles sont affectées.

## <a name="best-practices"></a>Meilleures pratiques

Pour connaître les meilleures pratiques en matière de haute disponibilité/récupération d’urgence, consultez les conseils Azure suivants sur cette rubrique : 
* L’article [*Conception d’applications résilientes pour Azure*](/azure/architecture/framework/resiliency/overview) décrit un cadre général qui vous aide à réfléchir aux questions de continuité d’activité et de reprise d’activité. 
* Le document [*Récupération d’urgence et haute disponibilité pour les applications Azure*](/azure/architecture/framework/resiliency/backup-and-recovery) contient des recommandations d’architecture concernant les stratégies permettant de mettre en place la haute disponibilité et la récupération d’urgence dans les applications Azure.

## <a name="next-steps"></a>Étapes suivantes 

En savoir plus sur la prise en main des solutions Azure Digital Twins :
 
* [*Qu’est-ce qu’Azure Digital Twins ?*](overview.md)
* [*Démarrage rapide : Explorer un exemple de scénario*](quickstart-adt-explorer.md)
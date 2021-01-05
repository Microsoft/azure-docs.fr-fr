---
ms.topic: include
ms.service: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.date: 07/09/2020
ms.openlocfilehash: f25c335c568c112c05f81df51d69e83aeff423e2
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025764"
---
## <a name="business-disaster-recovery"></a>Récupération d'urgence

Cette section décrit les fonctionnalités d’Azure Time Series Insights qui permet aux applications et services de continuer de s’exécuter, même si un incident se produit (connu sous le nom *récupération d’urgence*).

### <a name="high-availability"></a>Haute disponibilité

S’agissant d’un service Azure, Azure Time Series Insights offre certaines fonctionnalités de *haute disponibilité* à l’aide de redondances au niveau de la région Azure. Par exemple, Azure prend en charge les fonctionnalités de récupération d’urgence par le biais de la fonctionnalité *disponibilité interrégionale* d’Azure.

Les fonctionnalités de haute disponibilité supplémentaires fournies par l’intermédiaire d’Azure (et disponibles pour n’importe quelle instance Azure Time Series Insights) incluent :

- **Basculement** : Azure fournit la [géoréplication et l’équilibrage de charge](/azure/architecture/resiliency/recovery-loss-azure-region).
- **Restauration des données** et **restauration du stockage** : Azure fournit [plusieurs options pour préserver et récupérer les données](/azure/architecture/resiliency/recovery-data-corruption).
- **Azure Site Recovery** : Azure fournit des fonctionnalités de récupération de site via [Azure Site Recovery](../articles/site-recovery/index.yml).
- **Sauvegarde Azure** : la [Sauvegarde Azure](../articles/backup/backup-architecture.md) prend en charge la sauvegarde locale et dans le cloud des machines virtuelles Azure.

Veillez à activer les fonctionnalités pertinentes d’Azure pour offrir une haute disponibilité mondiale, interrégionale pour vos utilisateurs et appareils.

> [!NOTE]
> Si Azure est configuré pour activer la disponibilité interrégionale, aucune configuration de disponibilité interrégionale supplémentaire n’est requise dans Azure Time Series Insights.

### <a name="iot-and-event-hubs"></a>IoT et Event Hubs

Certains services Azure IoT incluent également des fonctionnalités de récupération d’urgence intégrées :

- [Récupération d’urgence haute disponibilité Azure IoT Hub](../articles/iot-hub/iot-hub-ha-dr.md), qui englobe la redondance intrarégionale
- [Stratégies Azure Event Hubs](../articles/event-hubs/event-hubs-geo-dr.md)
- [Redondance de Stockage Azure](../articles/storage/common/storage-redundancy.md)

L’intégration d’Azure Time Series Insights à d’autres services fournit des opportunités de récupération d’urgence supplémentaires. Par exemple, les données de télémétrie envoyées à votre Event Hub peuvent conservées dans une base de données de stockage Blob Azure de sauvegarde.

### <a name="azure-time-series-insights"></a>Azure Time Series Insights

Il existe plusieurs façons pour que vos données, applications et services Azure Time Series Insights continuent de s’exécuter, même en cas d’interruption. 

Toutefois, vous pouvez déterminer qu’une copie de sauvegarde complète de votre environnement Azure Time Series est également nécessaire, aux fins suivantes :

- Comme *instance de basculement* vers laquelle Azure Time Series Insights peut rediriger les données et le trafic
- Pour préserver les données et les informations d’audit

En règle générale, la meilleure façon de dupliquer un environnement Azure Time Series Insights consiste à créer un deuxième environnement Azure Time Series Insights dans une région Azure de sauvegarde. Les événements sont aussi envoyés à cet environnement secondaire à partir de votre source d’événements principale. Veillez à utiliser un deuxième groupe de consommateurs dédié. Suivez les instructions de récupération d’urgence de cette source, comme indiqué précédemment.

Pour créer un environnement dupliqué :

1. Créez un environnement dans la deuxième région. Pour plus d’informations, consultez [Créer un nouvel environnement de Azure Time Series Insights dans le portail Azure](../articles/time-series-insights/time-series-insights-get-started.md).
1. Créez un deuxième groupe de consommateurs dédié pour votre source d’événements.
1. Connectez cette source d’événements au nouvel environnement. Faites attention à bien désigner le deuxième groupe de consommateurs dédié.
1. Passez en revue la documentation [IoT Hub](../articles/time-series-insights/how-to-ingest-data-iot-hub.md) et [Event Hubs](../articles/time-series-insights/concepts-access-policies.md) d’Azure Time Series Insights.

Si un événement se produit :

1. Si votre région principale est affectée par un sinistre, basculez les opérations sur l’environnement Azure Time Series Insights de sauvegarde.
1. Utilisez votre deuxième région pour sauvegarder et récupérer toutes les données de télémétrie et de requête Azure Time Series Insights.

> [!IMPORTANT]
> Si un basculement se produit :
> 
> * Un délai peut également se produire.
> * Un pic momentané du traitement des message peut se produire, car les opérations sont redirigées.
> 
> Pour plus d’informations, consultez [Réduction de la latence dans Azure Time Series Insights](../articles/time-series-insights/time-series-insights-environment-mitigate-latency.md).
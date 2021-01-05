---
title: Vue d’ensemble d’Event Hubs Dedicated - Azure Event Hubs | Microsoft Docs
description: Cet article fournit une vue d’ensemble d’Azure Event Hubs Dedicated, qui offre des déploiements de concentrateurs d’événements à locataire unique.
ms.topic: article
ms.date: 10/23/2020
ms.openlocfilehash: 1a15206fc35f0d536c7105aa73dfdcfc9967124d
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94358862"
---
# <a name="overview-of-event-hubs-dedicated"></a>Vue d’ensemble d’Event Hubs Dedicated

Les *clusters Event Hubs* offrent des déploiements à locataire unique pour les clients aux besoins de streaming les plus pointus. Cette offre à locataire unique a un SLA de 99,99 % garanti et n'est disponible sur que notre niveau de tarification dédié. Un cluster Event Hubs peut recevoir des millions d’événements par seconde avec une garantie de capacité et de latence inférieure à la seconde. Les espaces de noms et les Event Hubs créées au sein du cluster dédié comprennent toutes les fonctionnalités de l’offre standard et bien davantage, mais sans aucune limite d’entrée. Il inclut également la fonctionnalité populaire [Event Hubs Capture](event-hubs-capture-overview.md) sans coût supplémentaire. Cette fonctionnalité vous permet de regrouper et journaliser automatiquement les flux de données dans Stockage Azure ou Azure Data Lake. 

Les clusters sont mis en service et facturés par **unités de capacité (UC)** , une quantité pré-allouée d'UC et de ressources mémoire. Vous pouvez acheter 1, 2, 4, 8, 12, 16 ou 20 unités de capacité pour chaque cluster. Le volume que vous pouvez ingérer et diffuser par CU dépend de divers facteurs, tels que les suivants : 

- Nombre de producteurs et consommateurs
- Forme de la charge utile
- Débit de sortie

> [!NOTE]
> Par défaut, tous les clusters Event Hubs sont compatibles avec Kafka et prennent en charge les points de terminaison Kafka qui peuvent être utilisés par vos applications Kafka existantes. L’activation de Kafka sur votre cluster n'affecte pas vos cas d'utilisation non-Kafka ; il n'y a aucune option ou besoin de désactiver Kafka sur un cluster.

## <a name="why-dedicated"></a>Pourquoi utiliser des clusters dédiés ?

Event Hubs Dedicated offre trois avantages convaincants pour les clients qui ont besoin d’une capacité de niveau entreprise :

#### <a name="single-tenancy-guarantees-capacity-for-better-performance"></a>L’architecture monolocation garantit la capacité pour de meilleures performances

Un cluster dédié garantit une capacité à pleine échelle. Il peut gérer l’entrée de jusqu’à plusieurs gigaoctets de données de diffusion en continu avec un stockage durable et une latence inférieure à la seconde pour s’adapter à tout pic de trafic. 

#### <a name="inclusive-and-exclusive-access-to-features"></a>Accès inclusif et exclusif aux fonctionnalités 
L’offre Dedicated comprend des fonctionnalités gratuites telles que Capture et fournit un accès exclusif à de futures fonctionnalités comme BYOK (Bring Your Own Key). le service gère également l’équilibrage de la charge, les mises à jour du système d’exploitation, les correctifs de sécurité et le partitionnement. Vous pouvez donc consacrer moins de temps à la maintenance de l’infrastructure, et plus de temps à la création de fonctionnalités côté client.  

#### <a name="cost-savings"></a>Réduction des coûts
À des volumes d’entrée élevés (>100 unités de débit), un cluster coûte nettement moins cher par heure que l’achat d’une quantité comparable d’unités de débit avec l’offre Standard.


## <a name="event-hubs-dedicated-quotas-and-limits"></a>Quotas et limites de l’offre Event Hubs Dedicated

L’offre Event Hubs Dedicated est facturée à un tarif mensuel fixe, avec un minimum de 4 heures d’utilisation. Le niveau dédié offre toutes les fonctionnalités du plan standard, mais avec une capacité à l’échelle l’entreprise et des limites pour les clients ayant des charges de travail exigeantes. 

| Fonctionnalité | standard | Dédié |
| --- |:---|:---|
| Bande passante | 20 unités de débit (jusqu'à 40 unités de débit) | 20 CUS |
| Espaces de noms |  1 | 50 par unité de capacité |
| Event Hubs |  10 par espace de noms | 1 000 par espace de noms |
| Événements d’entrée | Paiement par million d’événements | Inclus |
| Taille des messages | 1 million d’octets | 1 million d’octets |
| Partitions | 32 par hub d’événements | 1 024 par hub d’événements<br/>2 000 par unité de capacité |
| Groupes de consommateurs | 20 par hub d’événements | Aucune limite par unité de capacité, 1 000 par hub d’événements |
| Connexions réparties | 1 000 inclus, 5 000 maximum | 100 000 inclus et maximum |
| Rétention des messages | 7 jours, 84 Go inclus par unité de débit | 90 jours, 10 To inclus per unité de capacité |
| Capture | Paiement par heure | Inclus |

Pour augmenter les quotas et limites, consultez [Quotas et limites d’Azure Event Hubs](event-hubs-quotas.md)

## <a name="how-to-onboard"></a>Intégration : mode d’emploi

L’expérience en libre-service permettant de [créer un cluster Event Hubs](event-hubs-dedicated-cluster-create-portal.md) via le [portail Azure](https://aka.ms/eventhubsclusterquickstart) est désormais disponible en préversion. Si vous avez des questions ou si vous avez besoin d’aide pour l’intégration à Event Hubs Dedicated, contactez l’[équipe Event Hubs](mailto:askeventhubs@microsoft.com).

## <a name="faqs"></a>FAQ

[!INCLUDE [event-hubs-dedicated-clusters-faq](../../includes/event-hubs-dedicated-clusters-faq.md)]

## <a name="next-steps"></a>Étapes suivantes

Contactez votre représentant commercial Microsoft ou le support technique Microsoft pour obtenir des informations supplémentaires sur Event Hubs Dedicated. Vous pouvez également créer un cluster ou en apprendre plus sur les niveaux tarifaires Event Hubs en consultant les liens suivants :

- [Créer un cluster Event Hubs via le portail Azure](https://aka.ms/eventhubsclusterquickstart) 
- [Tarification d’Event Hubs Dedicated](https://azure.microsoft.com/pricing/details/event-hubs/). Vous pouvez également contacter votre représentant commercial Microsoft ou le support technique Microsoft pour obtenir des informations supplémentaires sur la capacité Event Hubs Dedicated.
- L’article [FAQ sur les hubs d’événements](event-hubs-faq.md) traite des informations de tarification et répond à certaines questions fréquemment posées sur Event Hubs.

---
title: Fichier include
description: Fichier include
services: event-hubs
author: spelluru
ms.service: event-hubs
ms.topic: include
ms.date: 10/23/2020
ms.author: spelluru
ms.custom: include file
ms.openlocfilehash: 0335481566ae3f28ac0f1e6bddce7050a65e7dc2
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92516990"
---
### <a name="what-can-i-achieve-with-a-cluster"></a>Que puis-je faire avec un cluster ?

Pour un cluster Event Hubs, la quantité que vous pouvez ingérer et diffuser dépend de divers facteurs tels que vos producteurs, vos consommateurs, la vitesse à laquelle vous ingérez et transformez, et bien plus encore. 

Le tableau suivant présente les résultats de référence que nous avons obtenus au cours de nos tests :

| Forme de la charge utile | Récepteurs | Bande passante en entrée| Messages en entrée | Bande passante en sortie | Messages en sortie | Nombre total d’unités de débit | Unités de débit par unité de capacité |
| ------------- | --------- | ---------------- | ------------------ | ----------------- | ------------------- | --------- | ---------- |
| Lots de 100x1 Ko | 2 | 400 Mo/s | 400 000 messages par seconde | 800 Mo/s | 800 000 messages par seconde | 400 unités de débit | 100 unités de débit | 
| Lots de 10x10 Ko | 2 | 666 Mo/s | 66 600 messages par seconde | 1,33 Go/s | 133 000 messages par seconde | 666 unités de débit | 166 unités de débit |
| Lots de 6x32 Ko | 1 | 1,05 Go/s | 34 000 messages par seconde | 1,05 Go/s | 34 000 messages par seconde | 1 000 unités de débit | 250 unités de débit |

Lors des tests, les critères suivants ont été utilisés :

- Un cluster Event Hubs dédié avec quatre unités de capacité (UC) a été utilisé. 
- Le hub d’événements utilisé pour l’ingestion possédait 200 partitions. 
- Les données ingérées ont été reçues par deux applications réceptrices provenant de toutes les partitions.

### <a name="can-i-scale-updown-my-cluster"></a>Puis-je augmenter/réduire (scale-up/scale-down) la taille de mon cluster ?

Une fois créés, vos clusters sont facturés pour un minimum de 4 heures d’utilisation. Dans la préversion de l’expérience en libre-service, vous pouvez soumettre une [demande de support](https://ms.portal.azure.com/#create/Microsoft.Support) à l’équipe Event Hubs sous **Technique** > **Quota** > **Demander un scale-up ou un scale-down de votre cluster dédié** pour augmenter ou réduire la taille de votre cluster. La demande de réduction de la taille de votre cluster peut prendre jusqu’à 7 jours. 

### <a name="how-does-geo-dr-work-with-my-cluster"></a>Comment la récupération d’urgence géographique fonctionne-t-elle avec mon cluster ?

Vous pouvez former une géopaire en combinant un espace de noms sous un cluster dédié avec un autre espace de noms sous un cluster dédié. Nous déconseillons l’association d’un espace de noms Dedicated avec un espace de noms dans notre offre Standard, car la limite de débit sera incompatible et entraînera des erreurs. 

### <a name="can-i-migrate-my-standard-namespaces-to-belong-to-a-dedicated-tier-cluster"></a>Puis-je migrer mes espaces de noms Standard vers un cluster dédié ?
Nous n’offrons actuellement aucun processus de migration automatisé pour la migration de vos données Event Hubs d’un espace de noms Standard vers un espace de noms Dedicated. 

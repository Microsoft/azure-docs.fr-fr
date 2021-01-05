---
title: Azure Front Door – Supervision de l’intégrité du serveur principal | Microsoft Docs
description: Cet article explique comment Azure Front Door supervise l’intégrité de vos serveurs principaux
services: frontdoor
documentationcenter: ''
author: duongau
ms.service: frontdoor
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: duau
ms.openlocfilehash: 4cbeea8ad20d41daff3d4ad086a36df5e988991f
ms.sourcegitcommit: 3792cf7efc12e357f0e3b65638ea7673651db6e1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91449251"
---
# <a name="health-probes"></a>Sondes d’intégrité

Pour déterminer l’intégrité et de la proximité de chaque serveur principal d’un environnement de service Front Door donné, celui-ci envoie régulièrement une requête HTTP/HTTPS synthétique à chaque serveur principal configuré. Front Door utilise alors les réponses fournies par la sonde pour identifier les « meilleurs » backends vers lesquels acheminer les demandes de vos clients. 

> [!WARNING]
> Dans la mesure où le service Front Door a de nombreux environnements de périphérie, le volume de sonde d’intégrité adressées à vos serveurs principaux peut être conséquent, allant de 25 à 1 200 requêtes par minute, en fonction de la fréquence configurée de la sonde d’intégrité. À la fréquence de sonde par défaut de 30 secondes, le volume des requêtes de sonde sur votre serveur principal doit être d’environ 200 requêtes par minute.

## <a name="supported-protocols"></a>Protocoles pris en charge

La porte d’entrée prend en charge l’envoi de sondes via les protocoles HTTP ou HTTPS. Ces sondes sont envoyées sur les mêmes ports TCP configurés pour le routage des demandes des clients et il n’est pas possible de les remplacer.

## <a name="supported-http-methods-for-health-probes"></a>Méthodes HTTP prises en charge pour les sondes d’intégrité

Le service Front Door prend en charge les méthodes HTTP suivantes pour l’envoi des requêtes de sonde d’intégrité :

1. **GET :** La méthode GET consiste à récupérer toutes les informations (sous forme d’entités) identifiées par l’URI de requête.
2. **HEAD :** La méthode HEAD est identique à la méthode GET, sauf que le serveur NE DOIT PAS retourner de corps de message dans la réponse. Pour les nouveaux profils Front Door, par défaut, la méthode de sonde est définie sur HEAD.

> [!NOTE]
> Afin de limiter la charge et les coûts qui pèsent sur vos serveurs principaux, le service Front Door recommande d’utiliser des requêtes HEAD pour les sondes d’intégrité.

## <a name="health-probe-responses"></a>Réponses des sondes d’intégrité

| Réponses  | Description | 
| ------------- | ------------- |
| Détermination de l’intégrité  |  Le code d’état 200 OK indique que le backend est sain. Tous les autres sont considérés comme un défaillance. Si, pour une raison quelconque (notamment une défaillance réseau), la réponse HTTP valide d’une sonde n’est pas reçue, la sonde est considérée comme défectueuse.|
| Mesure de la latence  | La latence est le temps horloge mesuré entre le moment qui précède immédiatement l’envoi de la demande d’analyse et le moment où le dernier octet de la réponse est reçu. Comme nous utilisons une nouvelle connexion TCP à chaque demande, cette mesure n’est pas orientée vers les backends ayant des connexions chaudes.  |

## <a name="how-front-door-determines-backend-health"></a>Comment une porte d’entrée détermine l’intégrité des backends

Pour déterminer l’intégrité, le service Azure Front Door utilise le même processus en trois étapes ci-dessous pour tous les algorithmes.

1. Excluez les backends désactivés.

2. Excluez les backends qui présentent des erreurs de sonde d’intégrité :
    * Pour effectuer cette sélection, examinez les _n_ dernières réponses de sonde d’intégrité. Si au moins _x_ sont saines, le backend est considéré comme sain.

    * Pour configurer la valeur _n_, modifiez la propriété SampleSize dans les paramètres d’équilibrage de charge.

    * Pour configurer la valeur _x_, modifiez la propriété SuccessfulSamplesRequired dans les paramètres d’équilibrage de charge.

3. Front Door mesure et maintient la latence (durée aller-retour) pour chaque ensemble de backends considérés comme sains.


## <a name="complete-health-probe-failure"></a>Mettre fin à l’échec des sondes d’intégrité

Si les sondes d’intégrité échouent pour chaque backend contenu dans un pool de backends, la porte d’entrée considère que tous les backends sont sains et leur achemine à tous le trafic sous la forme d’une distribution en tourniquet (round robin).

Dès qu’un serveur principal retrouve un état sain, le service Front Door reprend l’algorithme d’équilibrage de charge normal.

## <a name="disabling-health-probes"></a>Désactivation des sondes d’intégrité

Si vous avez un seul serveur principal dans votre pool principal, vous pouvez désactiver les sondes d’intégrité afin de réduire la charge sur le serveur principal de votre application. Même si vous avez plus d’un serveur principal dans le pool principal, il suffit qu’un seul soit activé pour que vous puissiez désactiver les sondes d’intégrité.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [créer une porte d’entrée](quickstart-create-front-door.md).
- Découvrez [comment fonctionne Front Door](front-door-routing-architecture.md).

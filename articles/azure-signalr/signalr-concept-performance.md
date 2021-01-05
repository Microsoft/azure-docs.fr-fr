---
title: Guide des performances pour Azure SignalR Service
description: Vue d’ensemble des performances et de la référence d’Azure SignalR Service. Métriques clés à prendre en compte lors de la planification de la capacité.
author: sffamily
ms.service: signalr
ms.topic: conceptual
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 68cad32be177fa20794399157fca89e87c2f8f59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "74157672"
---
# <a name="performance-guide-for-azure-signalr-service"></a>Guide des performances pour Azure SignalR Service

Un des principaux avantages de l’utilisation d’Azure SignalR Service est la facilité de la mise à l’échelle des applications SignalR. Dans un scénario à grande échelle, les performances sont un facteur important. 

Dans ce guide, nous allons présenter les facteurs qui affectent les performances des applications SignalR. Nous allons décrire les performances typiques dans différents scénarios de cas d’usage. Nous terminerons par présenter l’environnement et les outils que vous pouvez utiliser pour générer un rapport de performances.

## <a name="term-definitions"></a>Définitions des termes

*Entrant* : qualifie un message entrant à destination d’Azure SignalR Service.

*Sortant* : qualifie un message sortant en provenance d’Azure SignalR Service.

*Bande passante* : taille totale de tous les messages en 1 seconde.

*Mode par défaut* : mode de fonctionnement par défaut quand une instance d’Azure SignalR Service a été créée. Azure SignalR Service s’attend à ce que le serveur d’applications établisse une connexion avec lui avant d’accepter des connexions clientes.

*Mode serverless* : mode dans lequel Azure SignalR Service accepte uniquement des connexions clientes. Aucune connexion serveur n’est autorisée.

## <a name="overview"></a>Vue d’ensemble

Azure SignalR Service définit sept niveaux standard pour les différentes capacités de performances. Ce guide répond aux questions suivantes :

-   Quelles sont les performances typiques d’Azure SignalR Service pour chaque niveau ?

-   Azure SignalR Service répond-il à mes exigences de débit de message (par exemple, l’envoi de 100 000 messages par seconde) ?

-   Quel est le niveau qui convient pour mon scénario ? Ou comment puis-je sélectionner le niveau approprié ?

-   Quel est le type de serveur d’applications (taille de machine virtuelle) approprié pour moi ? Combien dois-je en déployer ?

Pour répondre à ces questions, ce guide fournit tout d’abord une explication générale des facteurs qui affectent les performances. Il illustre ensuite le nombre maximal de messages entrants et sortants pour chaque niveau de cas d’usage classique : **écho**, **diffusion**, **envoi au groupe** et **envoi à la connexion** (conversation de pair à pair).

Ce guide ne peut pas couvrir tous les scénarios (ni différents cas d’usage, tailles de message, modèles d’envoi de messages, etc.). Toutefois, il fournit des méthodes pour vous aider à :

- Évaluer votre exigence approximative concernant les messages entrants ou sortants.
- Rechercher les niveaux appropriés en consultant le tableau de performances.

## <a name="performance-insight"></a>Insight des performances

Cette section décrit les méthodes d’évaluation des performances, puis liste tous les facteurs qui affectent celles-ci. Enfin, elle fournit des méthodes pour vous aider à évaluer les exigences de performance.

### <a name="methodology"></a>Méthodologie

Le *débit* et la *latence* sont deux aspects typiques de la vérification des performances. Pour Azure SignalR Service, chaque niveau de référence SKU a sa propre stratégie de limitation de débit. La stratégie définit le *débit maximal autorisé (bande passante entrante et sortante)* comme étant le débit maximal atteint quand 99 % des messages ont une latence inférieure à 1 seconde.

La latence est l’intervalle de temps entre l’envoi du message par la connexion et la réception du message de réponse en provenance d’Azure SignalR Service. Prenons le cas d’usage **écho** à titre d’exemple. Chaque connexion cliente ajoute un horodatage au message. Le hub du serveur d’applications envoie le message d’origine au client. Ainsi, le délai de propagation est facilement calculé par chaque connexion cliente. L’horodatage est attaché pour chaque message dans les cas d’usage **diffusion**, **envoi au groupe** et **envoie à la connexion**.

Pour simuler des milliers de connexions clientes simultanées, plusieurs machines virtuelles sont créées dans un réseau privé virtuel au sein d’Azure. Toutes ces machines virtuelles se connectent à la même instance d’Azure SignalR Service.

Dans le mode par défaut d’Azure SignalR Service, les machines virtuelles du serveur d’applications sont déployées sur le même réseau privé virtuel en tant que machines virtuelles clientes. Toutes les machines virtuelles clientes et toutes les machines virtuelles du serveur d’applications sont déployées sur le même réseau de la même région afin d’éviter la latence inter-régions.

### <a name="performance-factors"></a>Facteurs de performances

En théorie, la capacité d’Azure SignalR Service est limitée par les ressources de calcul : Processeur, mémoire et réseau. Par exemple, plus il y a de connexions à Azure SignalR Service, plus le service utilise de mémoire. Pour le trafic de messages plus grands (où, par exemple, chaque message représente plus de 2 048 octets), Azure SignalR Service doit dépenser plus de cycles processeur. Parallèlement, la bande passante réseau Azure impose aussi une limite pour le trafic maximal.

Le type de transport est un autre facteur qui affecte les performances. Les trois types sont [WebSocket](https://en.wikipedia.org/wiki/WebSocket), [Server-Sent-Event](https://en.wikipedia.org/wiki/Server-sent_events) et [Long-Polling](https://en.wikipedia.org/wiki/Push_technology). 

WebSocket est un protocole de communication bidirectionnelle en duplex intégral sur une connexion TCP unique. Server-Sent-Event est un protocole unidirectionnel pour l’envoi (push) de messages depuis le serveur vers le client. Long-Polling demande que les clients interrogent régulièrement les informations du serveur par le biais d’une requête HTTP. Pour une même API et dans des conditions identiques, WebSocket offre les meilleures performances, Server-Sent-Event est plus lent, tandis que Long-Polling est le plus lent. Azure SignalR Service recommande WebSocket par défaut.

Le coût de routage des messages limite également les performances. Azure SignalR Service joue un rôle en tant que routeur de message, qui route le message depuis un ensemble de clients ou serveurs vers d’autres clients ou serveurs. Un scénario ou API différent nécessite une stratégie de routage différente. 

Pour le cas d’usage **écho**, le client s’envoie un message, et il est également la destination du routage. Ce modèle présente le coût de routage le plus bas. Par contre, pour les cas d’usage **diffusion**, **envoi au groupe** et **envoi à la connexion**, Azure SignalR Service doit rechercher les connexions cibles par le biais de la structure de données distribuée interne. Ce traitement supplémentaire utilise davantage de processeur, de mémoire et de bande passante réseau. Ainsi, les performances sont plus lentes.

En mode par défaut, le serveur d’applications peut également devenir un goulot d’étranglement pour certains scénarios. Le SDK Azure SignalR doit appeler le hub, tandis qu’il maintient une connexion active avec chaque client par le biais des signaux de pulsation.

En mode serverless, le client envoie un message par le biais d’une opération HTTP Post, action moins efficace que le recours à WebSocket.

Un autre facteur est le protocole : JSON et [MessagePack](https://msgpack.org/index.html). MessagePack présente une taille plus petite et offre une vitesse de livraison plus rapide que JSON. Cependant, MessagePack n’améliore pas nécessairement les performances. Les performances d’Azure SignalR Service ne sont pas sensibles aux protocoles, car elles ne décodent pas la charge utile d’un message pendant son transfert depuis les clients vers les serveurs ou vice versa.

En résumé, les facteurs suivants affectent les capacités entrante et sortante :

-   Niveau de référence SKU (processeur/mémoire)

-   Nombre de connexions

-   Taille des messages

-   Débit d’envoi des messages

-   Type de transport (WebSocket, Server-Sent-Event ou Long-Polling)

-   Scénario d’utilisation (coût de routage)

-   Connexions au serveur d’applications et au service (en mode serveur)


### <a name="finding-a-proper-sku"></a>Recherche d’une référence SKU appropriée

Comment pouvez-vous évaluer la capacité entrante/sortante ou trouver le niveau approprié pour un cas d’usage spécifique ?

Supposons que le serveur d’applications est suffisamment puissant et qu’il n’est pas le goulot d’étranglement des performances. Vous pouvez alors vérifier les bandes passantes entrante et sortante maximales pour chaque niveau.

#### <a name="quick-evaluation"></a>Évaluation rapide

Dans un premier temps, nous allons simplifier l’évaluation en supposant certains paramètres par défaut : 

- Le type de transport est WebSocket.
- La taille de message est de 2 048 octets.
- Un message est envoyé toutes les secondes.
- Azure SignalR Service est en mode par défaut.

Chaque niveau a ses propres bandes passantes entrante et sortante maximales. Une fois que la connexion entrante ou sortante dépasse la limite, la fluidité de l’expérience utilisateur n’est pas garantie.

Le cas d’usage **écho** offre la bande passante entrante maximale, car il a le coût de routage le plus bas. Le cas d’usage **diffusion** définit la bande passante des messages sortants maximale.

*Ne dépassez pas* les valeurs mises en évidence dans les deux tableaux suivants.

|       Écho                        | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|-----------------------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions                       | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| **Bande passante entrante** | **2 Mbits/s**    | **4 Mbits/s**    | **10 Mbits/s**   | **20 Mbits/s**    | **40 Mbits/s**    | **100 Mbits/s**   | **200 Mbits/s**    |
| Bande passante sortante | 2 Mbits/s   | 4 Mbits/s   | 10 Mbits/s  | 20 Mbits/s   | 40 Mbits/s   | 100 Mbits/s  | 200 Mbits/s   |


|     Diffusion             | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50  | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|---------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000  | 100 000 |
| Bande passante entrante  | 4 Kbits/s   | 4 Kbits/s   | 4 Kbits/s    | 4 Kbits/s    | 4 Kbits/s    | 4 Kbits/s     | 4 Kbits/s    |
| **Bande passante sortante** | **4 Mbits/s**    | **8 Mbits/s**    | **20 Mbits/s**    | **40 Mbits/s**    | **80  Mbits/s**    | **200 Mbits/s**    | **400  Mbits/s**   |

La *bande passante entrante* et la *bande passante sortante* sont la taille totale des messages par seconde.  Voici leurs formules :
```
  inboundBandwidth = inboundConnections * messageSize / sendInterval
  outboundBandwidth = outboundConnections * messageSize / sendInterval
```

- *inboundConnections* : nombre de connexions envoyant le message.

- *outboundConnections* : nombre de connexions recevant le message.

- *messageSize* : taille d’un message unique (valeur moyenne). Un petit message inférieur à 1 024 octets a un impact sur les performances similaire à un message de 1 024 octets.

- *sendInterval* : durée d’envoi d’un message. En règle générale, elle est d’une seconde par message, ce qui signifie qu’un message est envoyé toutes les secondes. Un intervalle plus petit signifie l’envoi de davantage de messages dans un laps de temps. Par exemple, 0,5 seconde par message signifie l’envoi de deux messages par seconde.

- *Connexions* : seuil maximal validé pour Azure SignalR Service pour chaque niveau. Si le nombre de connexions est augmenté, il peut subir une limitation de connexion.

#### <a name="evaluation-for-complex-use-cases"></a>Évaluation pour les cas d’usage complexes

##### <a name="bigger-message-size-or-different-sending-rate"></a>Taille de message plus grande ou débit d’envoi différent

Le cas d’usage réel est plus complexe. Le message envoyé peut être supérieur à 2 048 octets ou le débit d’envoi des messages n’est pas d’un message par seconde. Prenons comme exemple le cas d’usage « diffusion » de l’unité 100 pour découvrir comment évaluer ses performances.

Le tableau suivant présente un cas d’usage réel de **diffusion**. Toutefois, la taille de message, le nombre de connexions et le débit d’envoi des messages diffèrent de ce que nous avons supposé dans la section précédente. La question est de savoir comment nous pouvons déduire un de ces éléments (taille de message, nombre de connexions ou débit d’envoi des messages) si nous connaissons seulement deux d’entre eux.

| Diffusion  | Taille des messages | Messages entrants par seconde | Connexions | Intervalles d’envoi |
|---|---------------------|--------------------------|-------------|-------------------------|
| 1 | 20 Ko                | 1                        | 100 000     | 5 secondes                      |
| 2 | 256 KB               | 1                        | 8,000       | 5 secondes                      |

La formule suivante est facile à déduire d’après la formule précédente :

```
outboundConnections = outboundBandwidth * sendInterval / messageSize
```

Pour l’unité 100, la bande passante sortante maximale est de 400 Mo d’après le tableau précédent. Pour une taille de message de 20 Ko, le nombre maximal de connexions sortantes doit être de 400 Mo \* 5 / 20 Ko = 100 000, ce qui correspond à la valeur réelle.

##### <a name="mixed-use-cases"></a>Cas d’usage mixtes

Le cas d’usage réel associe généralement les quatre cas d’usage de base : **écho**, **diffusion**, **envoi au groupe** et **envoi à la connexion**. La méthodologie qui vous permet d’évaluer la capacité consiste à effectuer les opérations suivantes :

1. Divisez les cas d’usage mixte en quatre cas d’usage de base.
1. Calculez les bandes passantes des messages entrants et sortants maximales en utilisant les formules précédentes séparément.
1. Additionnez les calculs de bande passante pour obtenir la bande passante entrante/sortante maximale totale. 

Sélectionnez ensuite le niveau approprié à partir des tableaux des bandes passantes entrante/sortante maximales.

> [!NOTE]
> Pour envoyer un message à des centaines voire des milliers de petits groupes, ou dans le cas de milliers de clients s’envoyant un message, le coût de routage devient prédominant. Prenez cet impact en compte.

Dans le cas d’usage de l’envoi d’un message aux clients, vérifiez que le serveur d’applications *n’est pas* le goulot d’étranglement. La section « Étude de cas » suivante donne des indications sur le nombre de serveurs d’applications nécessaires et sur le nombre de connexions serveur à configurer.

## <a name="case-study"></a>Étude de cas

Les sections suivantes passent en revue quatre cas d’usage classiques pour le transport WebSocket : **écho**, **diffusion**, **envoi au groupe** et **envoi à la connexion**. Pour chaque scénario, la section liste les capacités entrante et sortante actuelles pour Azure SignalR Service. Elle explique également les principaux facteurs qui affectent les performances.

En mode par défaut, le serveur d’applications crée cinq connexions serveur avec Azure SignalR Service. Le serveur d’applications utilise par défaut le SDK d’Azure SignalR Service. Dans les résultats de test de performances suivants, les connexions serveur sont portées à 15 (ou plus pour la diffusion et l’envoi d’un message à un grand groupe).

Chaque cas d’usage a ses propres exigences concernant les serveurs d’applications. Le cas d’usage **Diffusion** a besoin d’un petit nombre de serveurs d’applications. Les cas d’usage **écho** ou **envoi à la connexion** ont besoin de nombreux serveurs d’applications.

Dans tous les cas d’usage, la taille de message par défaut est de 2 048 octets, tandis que l’intervalle d’envoi des messages est de 1 seconde.

### <a name="default-mode"></a>Mode par défaut

Les clients, les serveurs d’applications web et Azure SignalR Service sont impliqués dans le mode par défaut. Chaque client représente une connexion unique.

#### <a name="echo"></a>Écho

Tout d’abord, une application web se connecte à Azure SignalR Service. En second lieu, de nombreux clients se connectent à l’application web, qui les redirige vers Azure SignalR Service avec le jeton d’accès et le point de terminaison. Ensuite, les clients établissent des connexions WebSocket avec Azure SignalR Service.

Une fois que tous les clients ont établi les connexions, ils commencent à envoyer un message qui contient un horodatage au hub spécifique toutes les secondes. Le hub renvoie le message à son client d’origine. Chaque client calcule la latence quand il reçoit le message d’écho.

Dans le schéma suivant, les phases 5 à 8 (trafic indiqué en rouge) appartiennent à une boucle. La boucle s’exécute pendant une durée par défaut (5 minutes) et obtient les statistiques sur la latence de tous les messages.

![Trafic pour le cas d’usage écho](./media/signalr-concept-performance/echo.png)

Le comportement du cas d’usage **écho** détermine que la bande passante entrante maximale est égale à la bande passante sortante maximale. Pour plus de détails, consultez le tableau suivant.

|       Écho                        | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|-----------------------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions                       | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Messages entrants/sortants par seconde | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Bande passante entrante/sortante | 2 Mbits/s   | 4 Mbits/s   | 10 Mbits/s  | 20 Mbits/s   | 40 Mbits/s   | 100 Mbits/s  | 200 Mbits/s   |

Dans ce cas d’usage, chaque client appelle le hub défini dans le serveur d’applications. Le hub appelle simplement la méthode définie du côté du client d’origine. Ce hub est le hub le plus léger pour le cas d’usage **écho**.

```
        public void Echo(IDictionary<string, object> data)
        {
            Clients.Client(Context.ConnectionId).SendAsync("RecordLatency", data);
        }
```

Même pour ce hub simple, la pression du trafic sur le serveur d’applications est visible à mesure qu’augmente la charge des messages entrants d’**écho**. De nombreux serveurs d’applications sont donc nécessaires pour les grands niveaux de référence SKU. Le tableau suivant liste le nombre de serveurs d’applications pour chaque niveau.


|    Écho          | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 3      | 3      | 10     | 20      |

> [!NOTE]
> Le nombre de connexions clientes, la taille des messages, le débit d’envoi des messages, le niveau de référence SKU et le processeur/la mémoire du serveur d’applications affectent les performances globales du cas d’usage **écho**.

#### <a name="broadcast"></a>Diffusion

Pour le cas d’usage **diffusion**, quand l’application web reçoit le message, elle le diffuse à tous les clients. Plus les clients destinataires de la diffusion sont nombreux, plus le trafic des messages vers tous les clients est volumineux. Consultez le schéma suivant.

![Trafic pour le cas d’usage « diffusion »](./media/signalr-concept-performance/broadcast.png)

Un petit nombre de clients effectuent la diffusion. La bande passante des messages entrants est petite, mais la bande passante sortante est énorme. La bande passante des messages sortants augmente à mesure que s’accroît le taux de diffusions ou de connexions clientes.

Le tableau suivant récapitule les connexions clientes maximales, le nombre de messages entrants/sortants et la bande passante.

|     Diffusion             | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50  | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|---------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000  | 100 000 |
| Messages entrants par seconde  | 2     | 2     | 2      | 2      | 2      | 2       | 2       |
| Messages sortants par seconde | 2 000 | 4 000 | 10 000 | 20 000 | 40 000 | 100 000 | 200 000 |
| Bande passante entrante  | 4 Kbits/s   | 4 Kbits/s   | 4 Kbits/s    | 4 Kbits/s    | 4 Kbits/s    | 4 Kbits/s     | 4 Kbits/s     |
| Bande passante sortante | 4 Mbits/s   | 8 Mbits/s   | 20 Mbits/s   | 40 Mbits/s   | 80 Mbits/s   | 200 Mbits/s   | 400 Mbits/s   |

Les clients impliqués dans la diffusion qui publient des messages ne sont pas plus de quatre. Ils ont besoin de moins de serveurs d’applications par rapport au cas d’usage **écho**, car le nombre de messages entrants est petit. Deux serveurs d’applications sont suffisants pour les considérations relatives aux performances et au contrat SLA. Toutefois, vous devez augmenter le nombre de connexions serveur par défaut pour éviter un déséquilibre, en particulier pour l’unité 50 et l’unité 100.

|   Diffusion      | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 2      | 2      | 2      | 2       |

> [!NOTE]
> Augmentez le nombre de connexions serveur par défaut de 5 à 40 sur chaque serveur d’applications afin d’éviter des connexions serveur déséquilibrées éventuelles à Azure SignalR Service.
>
> Le nombre de connexions clientes, la taille des messages, le débit d’envoi des messages et le niveau de référence SKU affectent les performances globales pour le cas d’usage **diffusion**.

#### <a name="send-to-group"></a>Envoi au groupe

Le cas d’usage **envoi au groupe** a un modèle de trafic semblable au cas d’usage **diffusion**. La différence est qu’une fois que les clients ont établi des connexions WebSocket avec Azure SignalR Service, ils doivent rejoindre des groupes afin de pouvoir envoyer un message à un groupe spécifique. Le schéma suivant illustre le flux de trafic.

![Trafic pour le cas d’usage « envoi au groupe »](./media/signalr-concept-performance/sendtogroup.png)

Le nombre de membres par groupe et de groupes sont deux facteurs qui affectent les performances. Pour simplifier l’analyse, nous définissons deux types de groupes :

- **Petit groupe** : chaque groupe comporte 10 connexions. Le nombre de groupes est égal à (nombre de connexions maximal) / 10. Par exemple, pour l’unité 1, si le nombre de connexions s’élève à 1 000, nous avons 1 000 / 10 = 100 groupes.

- **Grand groupe** : le nombre de groupes est toujours de 10. Le nombre de membres par groupe est égal à (nombre de connexions maximal) / 10. Par exemple, pour l’unité 1, si le nombre de connexions s’élève à 1 000, chaque groupe a 1000 / 10 = 100 membres.

Le cas d’usage **envoi au groupe** implique un coût de routage pour Azure SignalR Service, car il doit rechercher les connexions cibles par le biais d’une structure de données distribuée. Plus le nombre de connexions d’envoi augmente, plus le coût s’accroît.

##### <a name="small-group"></a>Petit groupe

Le coût de routage est important pour l’envoi de message à de nombreux petits groupes. L’implémentation d’Azure SignalR Service atteint la limite du coût de routage à l’unité 50. L’ajout de ressources en processeur et en mémoire n’étant pas efficace, l’unité 100 ne peut pas en soi apporter d’amélioration. Si vous avez besoin de davantage de bande passante, contactez le support client.

|   Envoi à un petit groupe     | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|--------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000 | 100 000
| Nombre de membres par groupe        | 10    | 10    | 10     | 10     | 10     | 10     | 10 
| Nombre de groupes               | 100   | 200   | 500    | 1 000  | 2 000  | 5 000  | 10 000 
| Messages entrants par seconde  | 200   | 400   | 1 000  | 2 500  | 4 000  | 7 000  | 7 000   |
| Bande passante entrante  | 400 Kbits/s  | 800 Kbits/s  | 2 Mbits/s     | 5 Mbits/s     | 8 Mbits/s     | 14 Mbits/s    | 14 Mbits/s     |
| Messages sortants par seconde | 2 000 | 4 000 | 10 000 | 25 000 | 40 000 | 70 000 | 70 000  |
| Bande passante sortante | 4 Mbits/s    | 8 Mbits/s    | 20 Mbits/s    | 50 Mbits/s     | 80 Mbits/s    | 140 Mbits/s   | 140 Mbits/s    |

De nombreuses connexions clientes appelant le hub, le nombre de serveurs d’applications est également critique pour les performances. Le tableau suivant liste les nombres suggérés de serveurs d’applications.

|  Envoi à un petit groupe   | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 3      | 3      | 10     | 20      |

> [!NOTE]
> Le nombre de connexions clientes, la taille des messages, le débit d’envoi des messages, le coût de routage, le niveau de référence SKU et le processeur/la mémoire du serveur d’applications affectent les performances globales du cas d’usage **envoi à un petit groupe**.

##### <a name="big-group"></a>Grand groupe

Pour le cas d’usage **envoi à un grand groupe**, la bande passante sortante devient le goulot d’étranglement avant d’atteindre la limite du coût de routage. Le tableau suivant indique la bande passante sortante maximale, qui est presque identique à celle du cas d’usage **diffusion**.

|    Envoi à un grand groupe      | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50  | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|---------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000  | 100 000
| Nombre de membres par groupe        | 100   | 200   | 500    | 1 000  | 2 000  | 5 000   | 10 000 
| Nombre de groupes               | 10    | 10    | 10     | 10     | 10     | 10      | 10
| Messages entrants par seconde  | 20    | 20    | 20     | 20     | 20     | 20      | 20      |
| Bande passante entrante  | 80 Kbits/s   | 40 Kbits/s   | 40 Kbits/s    | 20 Kbits/s    | 40 Kbits/s    | 40 Kbits/s     | 40 Kbits/s     |
| Messages sortants par seconde | 2 000 | 4 000 | 10 000 | 20 000 | 40 000 | 100 000 | 200 000 |
| Bande passante sortante | 8 Mbits/s    | 8 Mbits/s    | 20 Mbits/s    | 40 Mbits/s    | 80 Mbits/s    | 200 Mbits/s    | 400 Mbits/s    |

Le nombre de connexions d’envoi n’est pas supérieur à 40. La charge pesant sur le serveur d’applications étant petite, le nombre suggéré d’applications web est petit.

|  Envoi à un grand groupe  | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 2      | 2      | 2      | 2       |

> [!NOTE]
> Augmentez le nombre de connexions serveur par défaut de 5 à 40 sur chaque serveur d’applications afin d’éviter des connexions serveur déséquilibrées éventuelles à Azure SignalR Service.
> 
> Le nombre de connexions clientes, la taille des messages, le débit d’envoi des messages, le coût du routage et le niveau de référence SKU affectent les performances globales pour le cas d’usage **envoi à un grand groupe**.

#### <a name="send-to-connection"></a>Envoi à la connexion

Dans le cas d’usage **envoi à la connexion**, quand des clients établissent des connexions à Azure SignalR Service, chaque client appelle un hub spécial pour obtenir son propre ID de connexion. Le test d’évaluation des performances collecte tous les ID de connexion, les mélange et les réaffecte à tous les clients en tant que cible d’envoi. Les clients continuent d’envoyer le message à la connexion cible jusqu’à ce que le test de performances se termine.

![Trafic pour le cas d’usage « envoi au client »](./media/signalr-concept-performance/sendtoclient.png)

Le coût du routage pour le cas d’usage **envoi à la connexion** est similaire au coût pour le cas d’usage **envoi à un petit groupe**.

À mesure que le nombre de connexions augmente, le coût du routage limite les performances globales. L’unité 50 a atteint la limite. Ainsi, l’unité 100 ne peut pas apporter d’amélioration.

Le tableau suivant est un récapitulatif statistique après de nombreux cycles d’exécution du banc d’essai **envoi à la connexion**.

|   Envoi à la connexion   | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50          | Unité 100         |
|------------------------------------|-------|-------|-------|--------|--------|-----------------|-----------------|
| Connexions                        | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000          | 100 000         |
| Messages entrants/sortants par seconde | 1 000 | 2 000 | 5 000 | 8,000  | 9 000  | 20 000 | 20 000 |
| Bande passante entrante/sortante | 2 Mbits/s    | 4 Mbits/s    | 10 Mbits/s   | 16 Mbits/s    | 18 Mbits/s    | 40 Mbits/s       | 40 Mbits/s       |

Ce cas d’usage nécessite une charge élevée côté serveur d’applications. Le tableau suivant indique le nombre de serveurs d’applications suggéré.

|  Envoi à la connexion  | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 3      | 3      | 10     | 20      |

> [!NOTE]
> Le nombre de connexions clientes, la taille des messages, le débit d’envoi des messages, le coût de routage, le niveau de référence SKU et le processeur/la mémoire pour le serveur d’applications affectent les performances globales du cas d’usage **envoi à la connexion**.

#### <a name="aspnet-signalr-echo-broadcast-and-send-to-small-group"></a>Cas d’usage ASP.NET SignalR « écho », « diffusion » et « envoi à un petit groupe »

Azure SignalR Service fournit les mêmes capacités de performance pour ASP.NET SignalR. 

Le test de performances utilise Azure Web Apps à partir du [plan de service Standard S3](https://azure.microsoft.com/pricing/details/app-service/windows/) pour ASP.NET SignalR.

Le tableau suivant indique le nombre d’applications web suggéré pour le cas d’usage ASP.NET SignalR **écho**.

|   Écho           | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 4     | 4      | 8      | 32      | 40       |

Le tableau suivant indique le nombre d’applications web suggéré pour le cas d’usage ASP.NET SignalR **diffusion**.

|  Diffusion       | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 2     | 2      | 2      | 2      | 2       |

Le tableau suivant indique le nombre d’applications web suggéré pour le cas d’usage ASP.NET SignalR **envoi à un petit groupe**.

|  Envoi à un petit groupe     | Unité 1 | Unité 2 | Unité 5 | Unité 10 | Unité 20 | Unité 50 | Unité 100 |
|------------------|-------|-------|-------|--------|--------|--------|---------|
| Connexions      | 1 000 | 2 000 | 5 000 | 10 000 | 20 000 | 50 000 | 100 000 |
| Nombre de serveurs d’applications | 2     | 2     | 4     | 4      | 8      | 32      | 40       |

### <a name="serverless-mode"></a>Mode serverless

Les clients et Azure SignalR Service sont impliqués dans le mode serverless. Chaque client représente une connexion unique. Le client envoie des messages par le biais de l’API REST à un autre client ou diffuse des messages à tous les clients.

L’envoi de messages à haute densité par le biais de l’API REST n’est pas aussi efficace que l’utilisation de WebSocket. Vous devez systématiquement générer une nouvelle connexion HTTP, ce qui constitue un coût supplémentaire en mode serverless.

#### <a name="broadcast-through-rest-api"></a>Diffusion par le biais de l’API REST
Tous les clients établissent des connexions WebSocket avec Azure SignalR Service. Ensuite, certains clients commencent à diffuser par le biais de l’API REST. L’envoi de message (entrant) s’effectue en totalité par le biais d’une opération HTTP Post, action moins efficace que le recours à WebSocket.

|   Diffusion par le biais de l’API REST     | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50  | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|---------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000  | 100 000 |
| Messages entrants par seconde  | 2     | 2     | 2      | 2      | 2      | 2       | 2       |
| Messages sortants par seconde | 2 000 | 4 000 | 10 000 | 20 000 | 40 000 | 100 000 | 200 000 |
| Bande passante entrante  | 4 Kbits/s    | 4 Kbits/s    | 4 Kbits/s     | 4 Kbits/s     | 4 Kbits/s     | 4 Kbits/s      | 4 Kbits/s      |
| Bande passante sortante | 4 Mbits/s    | 8 Mbits/s    | 20 Mbits/s    | 40 Mbits/s    | 80 Mbits/s    | 200 Mbits/s    | 400 Mbits/s    |

#### <a name="send-to-user-through-rest-api"></a>Envoi à l’utilisateur par le biais de l’API REST
Le test d’évaluation attribue des noms d’utilisateur à tous les clients avant qu’ils ne commencent à se connecter à Azure SignalR Service. Une fois que les clients ont établi des connexions WebSocket avec Azure SignalR Service, ils commencent à envoyer des messages à d’autres par le biais d’une opération HTTP Post.

|   Envoi à l’utilisateur par le biais de l’API REST | Unité 1 | Unité 2 | Unité 5  | Unité 10 | Unité 20 | Unité 50  | Unité 100 |
|---------------------------|-------|-------|--------|--------|--------|---------|---------|
| Connexions               | 1 000 | 2 000 | 5 000  | 10 000 | 20 000 | 50 000  | 100 000 |
| Messages entrants par seconde  | 300   | 600   | 900    | 1 300  | 2 000  | 10 000  | 18 000  |
| Messages sortants par seconde | 300   | 600   | 900    | 1 300  | 2 000  | 10 000  | 18 000 |
| Bande passante entrante  | 600 Kbits/s  | 1,2 Mbits/s  | 1,8Mbits/s   | 2,6 Mbits/s   | 4 Mbits/s     | 10 Mbits/s     | 36 Mbits/s    |
| Bande passante sortante | 600 Kbits/s  | 1,2 Mbits/s  | 1,8Mbits/s   | 2,6 Mbits/s   | 4 Mbits/s     | 10 Mbits/s     | 36 Mbits/s    |

## <a name="performance-test-environments"></a>Environnements du test de performances

Pour tous les cas d’usage répertoriés plus haut, nous avons effectué les tests de performances dans un environnement Azure. Nous avons utilisé au plus 50 machines virtuelles clientes et 20 machines virtuelles de serveur d’applications. Voici quelques détails :

- Taille des machines virtuelles clientes : StandardDS2V2 (2 processeurs virtuels, 7 Go de mémoire)

- Taille des machines virtuelles de serveur d’applications : StandardF4sV2 (4 processeurs virtuels, 8 Go de mémoire)

- Connexions serveur avec le SDK Azure SignalR : 15

## <a name="performance-tools"></a>Outils d’analyse des performances

Vous trouverez les outils de performances pour Azure SignalR Service sur [GitHub](https://github.com/Azure/azure-signalr-bench/).

## <a name="next-steps"></a>Étapes suivantes

Cet article vous a présenté une vue d’ensemble des performances d’Azure SignalR Service dans des scénarios de cas d’usage classiques.

Pour obtenir des détails sur les éléments internes du service et sur sa mise à l’échelle, lisez les guides suivants :

* [Éléments internes Azure SignalR Service](signalr-concept-internals.md)
* [Mise à l’échelle d’Azure SignalR Service](signalr-howto-scale-multi-instances.md)

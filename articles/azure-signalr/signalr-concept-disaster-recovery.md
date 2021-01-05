---
title: Résilience et reprise d’activité après sinistre dans Azure SignalR Service
description: Présentation de la façon de configurer plusieurs instances du service SignalR afin d’assurer la résilience et la reprise d’activité après sinistre
author: chenkennt
ms.service: signalr
ms.topic: conceptual
ms.custom: devx-track-csharp
ms.date: 03/01/2019
ms.author: kenchen
ms.openlocfilehash: b1cb48d1ae858dbcd0df80780b4c3cee3deac75b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90976499"
---
# <a name="resiliency-and-disaster-recovery-in-azure-signalr-service"></a>Résilience et reprise d’activité après sinistre dans Azure SignalR Service

La résilience et la reprise d’activité après sinistre sont des besoins communs des systèmes en ligne. Azure SignalR Service garantit déjà une disponibilité de 99,9 %, mais demeure un service régional.
Votre instance de service s’exécute toujours dans une seule région et ne bascule pas vers une autre région lors d’une panne au niveau régional.

Au lieu de cela, le kit SDK de notre service fournit une fonctionnalité permettant de prendre en charge plusieurs instances du service SignalR et de basculer automatiquement vers d’autres instances lorsque certaines ne sont pas disponibles.
Cette fonctionnalité vous permet de récupérer en cas de sinistre, mais vous devez configurer par vous-même la bonne topologie du système. Vous allez apprendre à le faire dans ce document.

## <a name="high-available-architecture-for-signalr-service"></a>Architecture à haute disponibilité pour le service SignalR

Pour bénéficier d’une résilience inter-région pour le service SignalR, vous devez configurer plusieurs instances du service dans différentes régions. Par conséquent, lorsqu’une région est défaillante, les autres peuvent être utilisées comme sauvegarde.
Lors de la connexion de plusieurs instances de service au serveur d’applications, il existe deux rôles, le rôle principal et le rôle secondaire.
Le rôle principal est une instance qui accepte le trafic en ligne et le rôle secondaire est une instance pleinement fonctionnelle mais de secours du rôle principal.
Dans notre implémentation du kit SDK, la négociation retourne uniquement les points de terminaison principaux, de sorte que, dans le cadre d’un fonctionnement normal, les clients se connectent uniquement aux points de terminaison principaux.
Mais lorsque l’instance principale est défaillante, la négociation retourne les points de terminaison secondaires pour permettre aux clients de continuer à se connecter.
L’instance principale et le serveur d’applications sont connectés via des connexions serveur normales, mais l’instance secondaire et le serveur d’applications sont connectés via un type spécial de connexion, appelée connexion faible.
La principale différence d’une connexion faible est qu’elle n’accepte pas le routage des connexions client, car l’instance secondaire se trouve dans une autre région. Le routage d’un client vers une autre région n’est pas un choix optimal (augmente la latence).

Une instance de service peut avoir différents rôles si elle se connecte à plusieurs serveurs d’applications.
Une installation type pour un scénario inter-région consiste à avoir deux paires (ou plus) d’instances de service SignalR et de serveurs d’applications.
Dans chaque paire, le serveur d’applications et le service SignalR se trouvent dans la même région, et le service SignalR est connecté au serveur d’applications en tant que rôle principal.
Entre deux paires, le serveur d’applications et le service SignalR sont également connectés, mais le service SignalR devient une instance secondaire lorsqu’il se connecte au serveur dans une autre région.

Avec cette topologie, un message provenant d’un serveur peut être remis à tous les clients, car tous les serveurs d’applications et toutes les instances du service SignalR sont interconnectés.
Mais lorsqu’un client est connecté, il est toujours routé vers le serveur d’applications figurant dans la même région pour favoriser une latence réseau optimale.

Le diagramme ci-dessous illustre cette topologie :

![Le diagramme montre deux régions, chacune avec un serveur d’applications et un service SignalR, où chaque serveur est associé au service SignalR dans sa région comme principal et avec le service dans l’autre région comme secondaire.](media/signalr-concept-disaster-recovery/topology.png)

## <a name="configure-app-servers-with-multiple-signalr-service-instances"></a>Configurer les serveurs d’applications avec plusieurs instances du service SignalR

Une fois que le service SignalR et les serveurs d’applications ont été créés dans chaque région, vous pouvez configurer vos serveurs d’applications pour qu’ils se connectent à toutes les instances du service SignalR.

Vous pouvez procéder de deux façons :

### <a name="through-config"></a>Via la configuration

Normalement, vous savez déjà comment définir la chaîne de connexion du service SignalR grâce à des variables d’environnement/paramètres d’application/web.config, dans une entrée de configuration nommée `Azure:SignalR:ConnectionString`.
Si vous avez plusieurs points de terminaison, vous pouvez les définir dans plusieurs entrées de configuration, chacune dans le format suivant :

```
Azure:SignalR:ConnectionString:<name>:<role>
```

Ici, `<name>` est le nom du point de terminaison et `<role>` est son rôle (principal ou secondaire).
Le nom est facultatif mais il sera utile si vous souhaitez personnaliser davantage le comportement de routage entre plusieurs points de terminaison.

### <a name="through-code"></a>Via le code

Si vous préférez stocker les chaînes de connexion à un autre endroit, vous pouvez également les lire dans votre code et les utiliser comme paramètres lorsque vous appelez `AddAzureSignalR()` (dans ASP.NET Core) ou `MapAzureSignalR()` (dans ASP.NET).

Voici l'exemple de code :

ASP.NET Core :

```cs
services.AddSignalR()
        .AddAzureSignalR(options => options.Endpoints = new ServiceEndpoint[]
        {
            new ServiceEndpoint("<connection_string1>", EndpointType.Primary, "region1"),
            new ServiceEndpoint("<connection_string2>", EndpointType.Secondary, "region2"),
        });
```

ASP.NET :

```cs
app.MapAzureSignalR(GetType().FullName, hub,  options => options.Endpoints = new ServiceEndpoint[]
    {
        new ServiceEndpoint("<connection_string1>", EndpointType.Primary, "region1"),
        new ServiceEndpoint("<connection_string2>", EndpointType.Secondary, "region2"),
    };
```

Il est possible de configurer plusieurs instances principales ou secondaires, auquel cas la négociation renverra un point de terminaison dans l’ordre suivant :

1. S’il y a au moins une instance principale en ligne, retourner une instance principale en ligne aléatoire.
2. Si toutes les instances principales sont hors service, retourner une instance secondaire en ligne aléatoire.

## <a name="failover-sequence-and-best-practice"></a>Bonne pratique et séquence de basculement

Vous disposez maintenant de la configuration appropriée de la topologie du système. Chaque fois qu’une instance du service SignalR est défaillante, le trafic en ligne est routé vers d’autres instances.
Voici ce qui se passe lorsqu’une instance principale est défaillante (et est restaurée plus tard) :

1. L’instance de service principale est défaillante : toutes les connexions serveur sur cette instance sont supprimées.
2. Tous les serveurs connectés à cette instance la marquent comme hors connexion. La négociation cesse de retourner ce point de terminaison et commence à retourner un point de terminaison secondaire.
3. Toutes les connexions client sur cette instance se ferment et les clients se reconnectent. Dans la mesure où les serveurs d’applications retournent maintenant un point de terminaison secondaire, les clients se connectent à une instance secondaire.
4. L’instance secondaire prend désormais en charge tout le trafic en ligne. Tous les messages du serveur aux clients peuvent encore être remis, car l’instance secondaire est connectée à tous les serveurs d’applications. Cependant, les messages des clients au serveur sont routés uniquement au serveur d’applications figurant dans la même région.
5. Une fois l’instance principale restaurée et de nouveau en ligne, le serveur d’applications rétablit les connexions à cette dernière et la marque comme étant en ligne. La négociation retourne désormais à nouveau le point de terminaison principal, si bien que les nouveaux clients sont reconnectés à l’instance principale. En revanche, les clients existants ne sont pas supprimés et continuent d’être routés vers l’instance secondaire jusqu'à ce qu’ils se déconnectent eux-mêmes.

Les diagrammes ci-dessous illustrent le basculement dans le service SignalR :

Figure 1 Avant le basculement ![Avant le basculement](media/signalr-concept-disaster-recovery/before-failover.png)

Figure 2 Après le basculement ![Après le basculement](media/signalr-concept-disaster-recovery/after-failover.png)

Fig.3 Peu après la récupération de l’instance principale ![](media/signalr-concept-disaster-recovery/after-recover.png)

Dans le cadre du fonctionnement normal, seuls le serveur d’applications et le service SignalR principaux gèrent le trafic en ligne (en bleu).
Après le basculement, le serveur d’applications et le service SignalR secondaires deviennent également actifs.
Une fois le service SignalR principal à nouveau en ligne, les nouveaux clients se connectent au service SignalR principal. Cependant, les clients existants restent connectés à l’instance secondaire, de sorte que les deux instances gèrent le trafic.
Une fois que tous les clients existants se sont déconnectés, votre système revient à la normale (Figure 1).

Il existe deux modèles principaux pour implémenter une architecture inter-région à haute disponibilité :

1. Le premier consiste à avoir une paire de serveur d’applications et d’instance de service SignalR gérant tout le trafic en ligne, et à avoir une autre paire de secours (configuration active/passive, illustrée à la Figure 1). 
2. L’autre consiste à avoir deux paires (ou plus) de serveurs d’applications et d’instances du service SignalR, gérant chacune une part du trafic en ligne et servant de sauvegarde pour les autres paires (configuration active/active, semblable à la Fig. 3).

Le service SignalR peut prendre en charge ces deux modèles, la principale différence étant la façon dont vous implémentez les serveurs d’applications.
Si les serveurs d’applications présentent une configuration active/passive, le service SignalR présente également une configuration active/passive (car le serveur d’applications principal retourne uniquement son instance de service SignalR principale).
Si les serveurs d’applications présentent une configuration active/active, le service SignalR présente également une configuration active/active (car tous les serveurs d’applications retournent leurs propres instances SignalR principales, afin que tous puissent obtenir le trafic).

Notez que, quel que soit le modèle que vous choisissez d’utiliser, vous devez connecter chaque instance du service SignalR à un serveur d’applications en tant qu’instance principale.

De plus, en raison de la nature de la connexion SignalR (connexion longue), les clients sont confrontés à des interruptions de connexion en cas d’incident et de basculement.
Vous devez gérer de telles situations côté client pour les rendre transparentes pour vos clients finaux. Par exemple, rétablissez une connexion qui a été fermée.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à configurer votre application pour assurer la résilience pour le service SignalR. Pour plus de détails sur la connexion serveur/client et le routage des connexions dans le service SignalR, vous pouvez lire [cet article](signalr-concept-internals.md) sur les éléments internes du service SignalR.

Dans les scénarios de mise à l’échelle, comme le partitionnement, qui utilisent conjointement plusieurs instances pour gérer un grand nombre de connexions, voir [Guide pratique pour mettre à l’échelle plusieurs instances](signalr-howto-scale-multi-instances.md).

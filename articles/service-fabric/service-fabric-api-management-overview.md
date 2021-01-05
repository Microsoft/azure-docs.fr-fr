---
title: Vue d’ensemble d’Azure Service Fabric avec Gestion des API
description: Cet article présente l’utilisation de Gestion des API Azure en tant que passerelle vers vos applications Service Fabric.
ms.topic: conceptual
ms.date: 06/22/2017
ms.openlocfilehash: 32f47d62cc9dda7cc88421dbf616bf69ffe152fc
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575684"
---
# <a name="service-fabric-with-azure-api-management-overview"></a>Vue d’ensemble d’Azure Service Fabric avec Gestion des API

Les applications cloud ont généralement besoin d’une passerelle frontale afin de fournir un point d’entrée unique pour les utilisateurs, les appareils ou d’autres applications. Dans Service Fabric, une passerelle peut être n’importe quel service sans état, comme une [application ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md), ou un autre service conçu pour l’entrée de trafic, par exemple les [concentrateurs d’événements](../event-hubs/index.yml), [IoT Hub](../iot-hub/index.yml) ou [Gestion des API Azure](../api-management/index.yml).

Cet article présente l’utilisation de Gestion des API Azure en tant que passerelle vers vos applications Service Fabric. Gestion des API s’intègre directement dans Service Fabric, ce qui vous permet de publier des API avec un ensemble complet de règles de routage vers vos services Service Fabric principaux.

## <a name="availability"></a>Disponibilité

> [!IMPORTANT]
> Cette fonctionnalité est disponible aux niveaux **Premium** et **Développeur** de Gestion des API en raison de la prise en charge de réseau virtuel requise.

## <a name="architecture"></a>Architecture

Une architecture Service Fabric commune utilise une application web d’une seule page qui effectue des appels HTTP vers les services principaux exposant les API HTTP. L’[exemple d’application Service Fabric de mise en route](https://github.com/Azure-Samples/service-fabric-dotnet-getting-started) montre un exemple de cette architecture.

Dans ce scénario, un service web sans état sert de passerelle vers l’application Service Fabric. Cette approche nécessite que vous écriviez un service web capable de transmettre les requêtes HTTP aux services principaux, comme indiqué dans le diagramme suivant :

![Diagramme montrant la façon dont un service web sans état sert de passerelle vers l’application Service Fabric.][sf-web-app-stateless-gateway]

Les applications sont de plus en plus complexes et il en va de même pour les passerelles qui doivent présenter une API devant une multitude de services principaux. Gestion des API Azure est conçue pour gérer les API complexes avec des règles de routage, un contrôle d’accès, une limitation du débit, une surveillance, une journalisation des événements et une mise en cache des réponses, mais impliquant un minimum de travail de votre part. Gestion des API Azure prend en charge la détection du service Service Fabric, la résolution de partition et la sélection de réplica pour acheminer les requêtes intelligemment et directement vers des services principaux dans Service Fabric, ce qui vous évite d’avoir à écrire votre propre passerelle API sans état. 

Dans ce scénario, l’IU web est toujours prise en charge par un service web, tandis que les appels API HTTP sont gérés et acheminés par le biais de Gestion des API Azure, comme indiqué dans le diagramme suivant :

![Diagramme montrant la façon dont l’IU web est toujours prise en charge par un service web, tandis que les appels API HTTP sont gérés et acheminés par le biais de Gestion des API Azure.][sf-apim-web-app]

## <a name="application-scenarios"></a>Scénarios d’application

Les services de Service Fabric peuvent être sans état ou avec état, et ils peuvent être partitionnés selon l’un des trois modèles : singleton, plage Int64, et nommé. La résolution du point de terminaison de service requiert l’identification d’une partition spécifique d’une instance de service spécifique. Lors de la résolution d’un point de terminaison de service, le nom d’instance de service (par exemple, `fabric:/myapp/myservice`) ainsi que la partition spécifique du service doivent être indiqués, sauf dans le cas d’une partition singleton.

Gestion des API Azure peut être utilisée avec n’importe quelle combinaison de services sans état, services avec état et schéma de partition.

## <a name="send-traffic-to-a-stateless-service"></a>Envoyer le trafic vers un service sans état

Dans le cas le plus simple, le trafic est transféré à une instance de service sans état. Pour ce faire, une opération Gestion des API contient une stratégie de traitement entrant avec un service principal Service Fabric qui effectue un mappage à une instance de service sans état spécifique dans le service principal Service Fabric. Les requêtes envoyées à ce service le sont à une instance aléatoire du service.

**Exemple**

Dans le scénario suivant, une application Service Fabric contient un service sans état nommé `fabric:/app/fooservice`, qui expose une API HTTP interne. Le nom d’instance de service est bien connu et peut être codé en dur directement dans la stratégie de traitement entrant Gestion des API. 

![Diagramme montrant une application Service Fabric qui contient un service sans état qui expose une API HTTP interne.][sf-apim-static-stateless]

## <a name="send-traffic-to-a-stateful-service"></a>Envoyer le trafic vers un service avec état

Comme pour le scénario de service sans état, le trafic peut être transféré à une instance de service avec état. Dans ce cas, une opération Gestion des API contient une stratégie de traitement entrant avec un service principal Service Fabric qui mappe une requête à une partition d’instance de service *avec état* spécifique. La partition à laquelle mapper chaque requête est calculée par le biais d’une méthode lambda qui utilise une entrée à partir de la requête HTTP entrante, comme une valeur dans le chemin d’accès d’URL. La stratégie peut être configurée de manière à envoyer des requêtes pour le réplica principal uniquement, ou à un réplica aléatoire pour les opérations de lecture.

**Exemple**

Dans le scénario suivant, une application Service Fabric contient un service partitionné avec état, nommé `fabric:/app/userservice`, qui expose une API HTTP interne. Le nom d’instance de service est bien connu et peut être codé en dur directement dans la stratégie de traitement entrant Gestion des API.  

Le service est partitionné à l’aide du schéma de partition Int64, avec deux partitions et une plage de clés qui s’étend de `Int64.MinValue` à `Int64.MaxValue`. La stratégie de serveur principal calcule une clé de partition dans cette plage en convertissant la valeur `id` fournie dans le chemin d’accès de requête URL à un entier 64 bits, bien que n’importe quel algorithme puisse être utilisé ici pour calculer la clé de partition. 

![Vue d’ensemble de la topologie Service Fabric avec Gestion des API][sf-apim-static-stateful]

## <a name="send-traffic-to-multiple-stateless-services"></a>Envoyer le trafic vers plusieurs services sans état

Dans des scénarios plus avancés, vous pouvez définir une opération Gestion des API qui mappe les requêtes à plusieurs instances de service. Dans ce cas, chaque opération contient une stratégie qui mappe les requêtes à une instance de service spécifique en fonction des valeurs définies à partir de la requête HTTP entrante, comme le chemin d’accès URL ou la chaîne de requête, et pour les services avec état, une partition dans l’instance de service.

Pour ce faire, une opération Gestion des API contient une stratégie de traitement entrant avec un service principal Service Fabric qui effectue un mappage à une instance de service sans état dans le service principal Service Fabric basé sur les valeurs récupérées dans la requête HTTP entrante. Les requêtes envoyées à un service le sont à une instance aléatoire du service.

**Exemple**

Dans cet exemple, une instance de service sans état est créée pour chaque utilisateur d’une application avec un nom généré de manière dynamique à l’aide de la formule suivante :

- `fabric:/app/users/<username>`

  Chaque service a un nom unique, mais ces noms ne sont pas connus au départ, car les services sont créés en réponse à une entrée utilisateur ou administrateur et ils ne peuvent donc pas être codés en dur dans des stratégies APIM ou des règles de routage. Au lieu de cela, le nom du service auquel vous souhaitez envoyer une requête est généré dans la définition de stratégie principale à partir de la valeur `name` fournie dans le chemin d’accès de requête URL. Par exemple :

  - Une requête vers `/api/users/foo` est acheminée vers l’instance de service `fabric:/app/users/foo`
  - Une requête vers `/api/users/bar` est acheminée vers l’instance de service `fabric:/app/users/bar`

![Diagramme montrant un exemple où une instance de service sans état est créée pour chaque utilisateur d’une application avec un nom généré de manière dynamique.][sf-apim-dynamic-stateless]

## <a name="send-traffic-to-multiple-stateful-services"></a>Envoyer le trafic vers plusieurs services avec état

Tout comme l’exemple de service sans état, une opération Gestion des API peut mapper les requêtes à plusieurs instances de service **avec état**. Dans ce cas, vous aurez peut-être aussi besoin d’effectuer la résolution de partition pour chaque instance de service avec état.

Pour ce faire, une opération Gestion des API contient une stratégie de traitement entrant avec un service principal Service Fabric qui effectue un mappage à une instance de service avec état dans le service principal Service Fabric basé sur les valeurs récupérées dans la requête HTTP entrante. En plus de mapper une requête à une instance de service spécifique, la requête peut être mappée à une partition spécifique au sein de l’instance de service, et éventuellement au réplica principal ou à un réplica secondaire aléatoire au sein de la partition.

**Exemple**

Dans cet exemple, une instance de service avec état est créée pour chaque utilisateur de l’application avec un nom généré de manière dynamique à l’aide de la formule suivante :

- `fabric:/app/users/<username>`

  Chaque service a un nom unique, mais ces noms ne sont pas connus au départ, car les services sont créés en réponse à une entrée utilisateur ou administrateur et ils ne peuvent donc pas être codés en dur dans des stratégies APIM ou des règles de routage. Au lieu de cela, le nom du service auquel vous souhaitez envoyer une requête est généré dans la définition de stratégie principale à partir de la valeur `name` fournie dans le chemin d’accès de requête URL. Par exemple :

  - Une requête vers `/api/users/foo` est acheminée vers l’instance de service `fabric:/app/users/foo`
  - Une requête vers `/api/users/bar` est acheminée vers l’instance de service `fabric:/app/users/bar`

Chaque instance de service est également partitionnée à l’aide du schéma de partition Int64, avec deux partitions et une plage de clés qui s’étend de `Int64.MinValue` à `Int64.MaxValue`. La stratégie de serveur principal calcule une clé de partition dans cette plage en convertissant la valeur `id` fournie dans le chemin d’accès de requête URL à un entier 64 bits, bien que n’importe quel algorithme puisse être utilisé ici pour calculer la clé de partition. 

![Diagramme montrant que chaque instance de service est également partitionnée à l’aide du schéma de partition Int64, avec deux partitions et une plage de clés qui s’étend d’Int64.MinValue à Int64.MaxValue.][sf-apim-dynamic-stateful]

## <a name="next-steps"></a>Étapes suivantes

Suivez le [didacticiel](service-fabric-tutorial-deploy-api-management.md) pour configurer votre premier cluster Service Fabric avec Gestion des API et faire passer les requêtes par le biais de Gestion des API vers vos services.

<!-- links -->

<!-- pics -->
[sf-apim-web-app]: ./media/service-fabric-api-management-overview/sf-apim-web-app.png
[sf-web-app-stateless-gateway]: ./media/service-fabric-api-management-overview/sf-web-app-stateless-gateway.png
[sf-apim-static-stateless]: ./media/service-fabric-api-management-overview/sf-apim-static-stateless.png
[sf-apim-static-stateful]: ./media/service-fabric-api-management-overview/sf-apim-static-stateful.png
[sf-apim-dynamic-stateless]: ./media/service-fabric-api-management-overview/sf-apim-dynamic-stateless.png
[sf-apim-dynamic-stateful]: ./media/service-fabric-api-management-overview/sf-apim-dynamic-stateful.png

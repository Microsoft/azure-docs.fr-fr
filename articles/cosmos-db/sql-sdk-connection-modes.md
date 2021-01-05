---
title: Modes de connectivité du SDK SQL Azure Cosmos DB
description: En savoir plus sur les différents modes de connectivité disponibles dans les kits de développement logiciel (SDK) SQL Azure Cosmos DB.
author: ealsur
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/14/2020
ms.author: maquaran
ms.custom: devx-track-dotnet, contperf-fy21q2
ms.openlocfilehash: c30e97a4bff8fa845f6eb3c3092a00ee541e59f4
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97032794"
---
# <a name="azure-cosmos-db-sql-sdk-connectivity-modes"></a>Modes de connectivité du SDK SQL Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

La façon dont un client se connecte à Azure Cosmos DB a des conséquences importantes sur les performances, notamment en matière de latence côté client. Azure Cosmos DB offre un modèle de programmation RESTful simple et ouvert sur HTTPs appelé mode passerelle. En outre, il offre un protocole TCP efficace, qui est également RESTful dans son modèle de communication et utilise TLS pour l’authentification initiale et le chiffrement du trafic, appelé mode direct.

## <a name="available-connectivity-modes"></a>Modes de connectivité disponibles

Les deux modes de connexion disponibles sont les suivants :

  * Mode passerelle
      
    Le mode de passerelle est pris en charge sur toutes les plateformes SDK. Si votre application s’exécute dans un réseau d’entreprise avec des restrictions de pare-feu strictes, le mode passerelle est la meilleure option, car il utilise le port HTTPS standard et un seul point de terminaison DNS. Toutefois, il existe un compromis en termes de performances : le mode passerelle implique un tronçon réseau supplémentaire chaque fois que les données sont lues ou écrites dans Azure Cosmos DB. Nous vous recommandons le mode de connexion de passerelle quand vous exécutez des applications dans des environnements présentant un nombre limité de connexions de socket.

    Quand vous utilisez le Kit de développement logiciel (SDK) dans Azure Functions, en particulier dans le [plan Consommation](../azure-functions/functions-scale.md#consumption-plan), prenez en compte les [limites de connexions](../azure-functions/manage-connections.md) actuelles.

  * Mode direct

    Le mode direct prend en charge la connectivité via le protocole TCP et offre de meilleures performances, car il y a moins de tronçons réseau. L’application se connecte directement aux réplicas principaux. Le mode direct est actuellement uniquement pris en charge sur les plateformes SDK .NET et Java.
     
:::image type="content" source="./media/performance-tips/connection-policy.png" alt-text="Modes de connectivité Azure Cosmos DB" border="false":::

Ces modes de connectivité conditionnent essentiellement l’itinéraire que le plan de données demande (lectures et écritures de documents) pris de votre ordinateur client vers des partitions dans le serveur principal Azure Cosmos DB. Le mode direct est l’option la plus appropriés pour optimiser les performances. Il permet à votre client d’ouvrir des connexions TCP directement aux partitions dans le back-end Azure Cosmos DB et d’envoyer des demandes *directement*, sans intermédiaire. En revanche, en mode passerelle, les demandes effectuées par votre client sont acheminées vers un serveur « passerelle » dans le Azure Cosmos DB frontal, qui à son tour répartit vos demandes vers les partitions appropriées dans le serveur principal Azure Cosmos DB.

## <a name="service-port-ranges"></a>Plages de ports de service

Lorsque vous utilisez le mode direct, en plus des ports de passerelle, vous devez vous assurer que la plage de ports comprise entre 10000 et 20000 est ouverte, car Azure Cosmos DB utilise des ports TCP dynamiques. Lorsque vous utilisez le mode direct sur des [points de terminaison privés](./how-to-configure-private-endpoints.md), la plage complète des ports TCP (de 0 à 65535) doit être ouverte. Si ces ports ne sont pas ouverts et que vous essayez d’utiliser le protocole TCP, vous pouvez recevoir une erreur 503 de service non disponible.

Le tableau suivant présente un résumé des modes de connectivité disponibles pour les différentes API et les ports de service utilisés pour chaque API :

|Mode de connexion  |Protocole pris en charge  |Kits SDK pris en charge  |API/Port de service  |
|---------|---------|---------|---------|
|Passerelle  |   HTTPS    |  Tous les kits SDK    |   SQL (443), MongoDB (10250, 10255, 10256), Table (443), Cassandra (10350), Graph (443) <br> Le port 10250 mappe à une API Azure Cosmos DB par défaut pour l’instance MongoDB sans géoréplication. Les ports 10255 et 10256 mappent à l’instance avec géoréplication.   |
|Direct    |     TCP    |  Kits SDK .NET Kits SDK Java    | Lors de l’utilisation de points de terminaison publics/de service : les ports de la plage 10000 à 20000<br>Lors de l’utilisation de points de terminaison privés : les ports compris entre 0 et 65535 |

## <a name="next-steps"></a>Étapes suivantes

Pour des optimisations de performances de plateforme SDK spécifiques :

* [Conseils sur les performances du SDK .NET v2](performance-tips.md)

* [Conseils sur les performances du SDK .NET v3](performance-tips-dotnet-sdk-v3-sql.md)
 
* [Conseils sur les performances du SDK Java v4](performance-tips-java-sdk-v4-sql.md)
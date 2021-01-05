---
title: Gérer l’état dans les services Azure Service Fabric
description: Découvrez l’état dans Azure Service Fabric, notamment comment définir et gérer l’état du service dans les services Service Fabric.
author: masnider
ms.topic: conceptual
ms.date: 08/18/2017
ms.author: masnider
ms.openlocfilehash: 9acd3031d1071d1822791b333976aaf76161600f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75614568"
---
# <a name="service-state"></a>État du service
**L’état du service** correspond aux données en mémoire ou sur disque nécessaires au bon fonctionnement du service. Il comprend, par exemple, les variables membres et les structures de données que le service lit et écrit afin d’effectuer des tâches. Selon l’architecture du service, il peut aussi inclure des fichiers ou d’autres ressources stockés sur le disque, par exemple, les fichiers qu’utiliserait une base de données pour stocker les journaux d’activité des transactions et des données.

Prenons l’exemple d’une calculatrice. Un service de calculatrice de base accepte deux nombres et retourne leur somme. Ce calcul n’implique aucune variable membre ni aucune autre information.

À présent, considérons cette même calculatrice, mais avec une méthode supplémentaire permettant de stocker et de retourner la dernière somme calculée. Il s’agit à présent d’un service avec état, c’est-à-dire qu’il contient un certain état dans lequel il écrit quand il calcule une nouvelle somme et qu’il lit quand on lui demande de retourner la dernière somme calculée.

Dans Service Fabric Azure, le premier service porte le nom de « service sans état ». Le second service est appelé « service avec état ».

## <a name="storing-service-state"></a>Stockage de l’état du service
L’état peut être externalisé ou colocalisé avec le code qui le manipule. L’externalisation d’état s’effectue généralement à l’aide d’une base de données externe ou autre magasin de données, qui s’exécute sur différents ordinateurs du réseau ou en dehors du processus sur le même ordinateur. Dans notre exemple de calculatrice, le magasin de données pourrait être une base de données SQL ou une instance de magasin de tables Azure. Chaque requête de calcul de la somme effectue une mise à jour de ces données et demande au service de retourner en résultat la valeur actuelle récupérée à partir du magasin. 

L’état peut également être colocalisé avec le code qui manipule l’état. Les services avec état de Service Fabric sont majoritairement générés suivant ce modèle. Service Fabric fournit l’infrastructure permettant de s’assurer que cet état est hautement disponible, cohérent et durable, et que les services générés de cette façon peuvent facilement évoluer.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les concepts propres à Service Fabric, consultez les articles suivants :

* [Disponibilité des services Service Fabric](service-fabric-availability-services.md)
* [Extensibilité des services Service Fabric](service-fabric-concepts-scalability.md)
* [Partitionnement des services Service Fabric](service-fabric-concepts-partitioning.md)
* [Modèle Reliable Services de Service Fabric](service-fabric-reliable-services-introduction.md)

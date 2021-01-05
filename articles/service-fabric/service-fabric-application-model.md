---
title: Modèle d’application Azure Service Fabric
description: Comment modéliser et décrire des applications et des services dans Azure Service Fabric à l’aide de fichiers manifestes d’application et de service.
ms.topic: conceptual
ms.date: 2/23/2018
ms.openlocfilehash: 84e6b2309fdb206771d4ea01aa03c7f355d6ff19
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85963731"
---
# <a name="model-an-application-in-service-fabric"></a>Modéliser une application dans Service Fabric
Cet article fournit une vue d’ensemble du modèle d’application Azure Service Fabric et explique comment définir une application et un service via des fichiers manifestes.

## <a name="understand-the-application-model"></a>Comprendre le modèle d'application
Une application est une collection de services constitutifs qui exécutent une ou plusieurs fonctions. Un service exécute une fonction complète et autonome, et peut démarrer et s’exécuter indépendamment d’autres services.  Un service est composé d’un code, d’une configuration et de données. Pour chaque service, le code se compose de fichiers binaires exécutables, la configuration comprend des paramètres de service qui peuvent être chargés pendant l'exécution, tandis que les données comportent des données statiques arbitraires destinées à être consommées par le service. Chaque composant de ce modèle d'application hiérarchique peut faire l'objet d'une gestion de versions et d'une mise à niveau indépendantes.

![Modèle d'application Service Fabric][appmodel-diagram]

Un type d'application est une catégorisation d'une application constituée d'un ensemble de types de service. Un type de service est la catégorisation d’un service. La catégorisation peut avoir différents paramètres et différentes configurations, mais la fonctionnalité principale reste la même. Les instances d'un service représentent les variantes de configuration de service d'un même type de service.  

Les classes (ou « types ») d’applications et de services sont décrits dans des fichiers XML (manifestes d’application et manifestes de service).  Les manifestes décrivent des applications et des services et constituent les modèles par rapport auxquels des applications peuvent être instanciées à partir du magasin d’images du cluster.  Les manifestes sont abordés en détail dans [Manifestes des applications et des services](service-fabric-application-and-service-manifests.md). La définition de schéma pour les fichiers ServiceManifest.xml et ApplicationManifest.xml est installée avec le SDK Service Fabric et les outils sous *C:\Program Files\Microsoft SDKs\Service Fabric\schemas\ServiceFabricServiceModel.xsd*. Le schéma XML est documenté dans [Documentation relative au schéma ServiceFabricServiceModel.xsd](service-fabric-service-model-schema.md).

Le code des différentes instances d’application s’exécute sous forme de processus distincts, même si elles sont hébergées par le même nœud Service Fabric. En outre, le cycle de vie de chaque instance d’application peut être géré (par exemple, mis à jour) de façon indépendante. Comme le montre le diagramme suivant, les types d’application sont composés de types de service, eux-mêmes constitués de packages de code, de configuration et de données. Pour simplifier le diagramme, seuls les packages code/configuration/données pour `ServiceType4` sont affichés, même si chaque type de service inclut tout ou partie de ces types de packages.

![Types d’application service Fabric et types de service][cluster-imagestore-apptypes]

Une ou plusieurs instances d'un type de service peuvent être actives dans le cluster. Par exemple, les instances de service avec état, ou réplicas, assurent une grande fiabilité en répliquant l'état entre les réplicas situés sur différents nœuds du cluster. La réplication garantit essentiellement la redondance du service en cas de défaillance d’un nœud du cluster. Un [service partitionné](service-fabric-concepts-partitioning.md) subdivise son état (et les modèles d'accès à cet état) entre les nœuds du cluster.

Le diagramme suivant montre la relation entre les applications et les instances de service, les partitions et les réplicas.

![Partitions et réplicas au sein d'un service][cluster-application-instances]

> [!TIP]
> Vous pouvez afficher la disposition des applications dans un cluster à l’aide de l’outil Service Fabric Explorer disponible à l’adresse http://&lt;adresse_cluster&gt;:19080/Explorer. Pour plus d’informations, voir [Visualisation de votre cluster à l’aide de l’outil Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
> 
> 


## <a name="next-steps"></a>Étapes suivantes
- Découvrez-en plus sur l’[extensibilité des applications](service-fabric-concepts-scalability.md).
- Découvrez-en plus sur l’[état](service-fabric-concepts-state.md), le [partitionnement](service-fabric-concepts-partitioning.md) et la [disponibilité](service-fabric-availability-services.md) des services.
- Pour savoir comment les applications et les services sont définis, consultez [Manifestes des applications et des services](service-fabric-application-and-service-manifests.md).
- [Modèles d’hébergement d’applications](service-fabric-hosting-model.md) décrit la relation entre les réplicas (ou instances) d’un service déployé et le processus hôte du service.

<!--Image references-->
[appmodel-diagram]: ./media/service-fabric-application-model/application-model.png
[cluster-imagestore-apptypes]: ./media/service-fabric-application-model/cluster-imagestore-apptypes.png
[cluster-application-instances]: media/service-fabric-application-model/cluster-application-instances.png



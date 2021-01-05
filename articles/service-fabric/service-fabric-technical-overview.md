---
title: Découvrir la terminologie d’Azure Service Fabric
description: Découvrez la terminologie et les concepts Service Fabric clés utilisés dans le reste de la documentation.
author: masnider
ms.topic: conceptual
ms.date: 09/17/2018
ms.author: masnider
ms.openlocfilehash: 2ac4b81a284ed8c38bc9cefccd08db5afa51d600
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575939"
---
# <a name="service-fabric-terminology-overview"></a>Présentation de la terminologie Service Fabric

Azure Service Fabric est une plateforme de systèmes distribués qui permet d’empaqueter, de déployer et de gérer facilement des microservices scalables et fiables.  Vous pouvez [héberger des clusters Service Fabric n'importe où](service-fabric-deploy-anywhere.md) : dans Azure, dans un centre de données local ou auprès de n'importe quel fournisseur de services cloud.  Service Fabric est l’orchestrateur qui alimente [Azure Service Fabric Mesh](../service-fabric-mesh/index.yml). Vous pouvez utiliser n’importe quelle infrastructure pour écrire vos services, et choisir l’emplacement où exécuter l’application parmi plusieurs options d’environnement. Cet article décrit en détail la terminologie utilisée dans Service Fabric pour que vous compreniez les termes utilisés dans la documentation.

## <a name="infrastructure-concepts"></a>Concepts d’infrastructure

**Cluster** : groupe de machines virtuelles ou physiques connectées au réseau et au sein duquel vos microservices sont déployés et gérés.  Les clusters peuvent être mis à l’échelle pour des milliers de machines.

**Nœud** : une machine ou une machine virtuelle faisant partie d'un cluster est appelée un *nœud*. Un nom (chaîne) est affecté à chaque nœud. Les nœuds présentent des caractéristiques, telles que des propriétés de placement. Chaque machine ou machine virtuelle a un service Windows à démarrage automatique, `FabricHost.exe`, qui commence à s’exécuter dès le démarrage, puis démarre deux exécutables : `Fabric.exe` et `FabricGateway.exe`. Ces deux exécutables constituent le nœud. Pour les scénarios de test, vous pouvez héberger plusieurs nœuds sur une seule et même machine ou sur une seule et même machine virtuelle en exécutant plusieurs instances de `Fabric.exe` et `FabricGateway.exe`.

## <a name="application-and-service-concepts"></a>Concepts de l’application et du service

**Application Service Fabric Mesh** : les applications Service Fabric Mesh sont décrites par le modèle de ressource (fichiers de ressources YAML et JSON) et peuvent être déployées dans n'importe quel environnement dans lequel Service Fabric est exécuté.

**Application native Service Fabric** : les applications natives Service Fabric sont décrites par le modèle d'application native (manifestes d'applications et de services basés sur XML).  Les applications natives Service Fabric ne peuvent pas s’exécuter dans Service Fabric Mesh.

### <a name="service-fabric-mesh-application-concepts"></a>Concepts de l’application Service Fabric Mesh

**Application** : une application est l'unité de déploiement, de contrôle de version et de durée de vie d'une application Mesh. Le cycle de vie de chaque instance d’application peut être géré de manière indépendante.  Les applications sont composées d’un ou de plusieurs packages de code de service et paramètres. Une application est définie à l’aide du schéma de modèle de ressource Azure (RM).  Les services sont décrits en tant que propriétés de la ressource d’application dans le modèle RM.  Les réseaux et volumes utilisés par l’application sont référencés par l’application.  Lorsque vous créez une application, l’application, les services, le réseau et les volumes sont modélisés à l’aide du modèle de ressources Service Fabric.

**Service** : un service dans une application représente un microservice et exécute une fonction complète et autonome. Chaque service se compose d’un ou plusieurs packages de code qui décrivent tout ce qui est nécessaire pour exécuter l’image conteneur associée au package de code.  Le nombre de services dans une application peut être augmenté ou diminué.

**Réseau** : une ressource réseau crée un réseau privé pour vos applications et est indépendante des applications ou services qui peuvent y faire référence. Plusieurs services de différentes applications peuvent faire partie d’un même réseau. Les réseaux sont des ressources pouvant être déployées qui sont référencées par les applications.

**Package de code** : les packages de code décrivent tout ce qui est nécessaire pour exécuter l’image conteneur associée au package de code, notamment :

* Nom, version et registre du conteneur
* Ressources de processeur et de mémoire requises pour chaque conteneur
* Points de terminaison réseau
* Volumes à monter dans le conteneur, faisant référence à une ressource de volume distincte.

Tous les packages de code définis comme faisant partie d’une ressource d’application sont déployés et activés ensemble en tant que groupe.

**Volume** : Les volumes sont des répertoires montés à l’intérieur de vos instances de conteneur, que vous pouvez utiliser pour conserver l’état. Le pilote de volume Azure Files monte un partage Azure Files sur un conteneur et fournit un stockage fiable des données via toute API prenant en charge le stockage de fichiers. Les volumes sont des ressources pouvant être déployées qui sont référencées par les applications.

### <a name="service-fabric-native-application-concepts"></a>Concepts de l’application native Service Fabric

**Application** : Une application est une collection de services constitutifs qui exécutent une ou plusieurs fonctions. Le cycle de vie de chaque instance d’application peut être géré de manière indépendante.

**Service** : Un service exécute une fonction complète et autonome, et peut démarrer et s’exécuter indépendamment d’autres services. Un service est composé d’un code, d’une configuration et de données. Pour chaque service, le code se compose de fichiers binaires exécutables, la configuration comprend des paramètres de service qui peuvent être chargés pendant l'exécution, tandis que les données comportent des données statiques arbitraires destinées à être consommées par le service.

**Type d’application** : nom/version affectés à une collection de types de services. Il est défini dans un fichier `ApplicationManifest.xml` et incorporé dans un répertoire de package d’application. Le répertoire est ensuite copié dans le magasin d’images du cluster Service Fabric. Vous pouvez ensuite créer une application nommée à partir de ce type d’application au sein du cluster.

Pour plus d’informations, consultez l’article [Modèle d’application](service-fabric-application-model.md).

**Package d'application** : répertoire de disque contenant le fichier `ApplicationManifest.xml` du type d'application. Référence les packages de services pour chaque type de service qui constitue le type d’application. Les fichiers du répertoire de package d’application sont copiés dans le magasin d’images du cluster Service Fabric. Par exemple, un package d’un type d’application de messagerie peut contenir des références à un package de services de File d’attente, un package de services frontaux et un package de services de base de données.

**Application nommée** : après avoir copié un package d'application dans le magasin d'images, vous créez une instance de l'application au sein du cluster. Vous créez une instance quand vous spécifiez le type d’application du package d’application à l’aide de son nom ou de sa version. Un nom d’URI est affecté à chaque instance de type d’application, qui ressemble à ceci : `"fabric:/MyNamedApp"`. Dans un cluster, vous pouvez créer plusieurs applications nommées à partir d’un seul type d’application. Vous pouvez également en créer à partir de différents types d’application. Chaque application nommée est gérée, et sa version est gérée indépendamment.

**Type de service** : nom/version affectés aux packages de code, packages de données et packages de configuration d'un service. Le type de service est défini dans le fichier `ServiceManifest.xml` et incorporé dans un répertoire de package de services. Ce répertoire est ensuite référencé par le fichier `ApplicationManifest.xml` du package d’application. Au sein du cluster, après avoir créé une application nommée, vous pouvez créer un service nommé à partir d’un des types de service du type d’application. Le fichier `ServiceManifest.xml` du type de service décrit le service.

Pour plus d’informations, consultez l’article [Modèle d’application](service-fabric-application-model.md).

Il existe deux types de service :

* **Sans état** : utilisez un service sans état si l'état persistant du service est stocké dans un service de stockage externe, par exemple Stockage Azure, Azure SQL Database ou Azure Cosmos DB. Utilisez un service sans état si le service est dépourvu de stockage persistant. Par exemple, pour un service de calculatrice où des valeurs sont transmises au service, un calcul est effectué à l’aide de ces valeurs, et un résultat est retourné.
* **Avec état** : utilisez un service avec état si vous souhaitez que Service Fabric gère l'état de votre service via ses modèles de programmation Reliable Collections ou Reliable Actors. Spécifiez le nombre de partitions sur lesquelles vous souhaitez répartir votre état (pour la scalabilité) lors de la création d’un service nommé. Spécifiez également le nombre de réplications de votre état sur plusieurs nœuds (pour la fiabilité). Chaque service nommé possède un seul réplica principal et plusieurs réplicas secondaires. Vous modifiez l’état de votre service nommé en écrivant dans le réplica principal. Service Fabric réplique ensuite cet état sur tous les réplicas secondaires pour préserver la synchronisation de votre état. Service Fabric détecte automatiquement l’échec d’un réplica principal et promeut un réplica secondaire existant en réplica principal. Service Fabric crée ensuite un nouveau réplica secondaire.  

**Réplicas ou instances** : code (et état pour les services avec état) d’un service déployé et en cours d’exécution. Consultez [Réplicas et instances](service-fabric-concepts-replica-lifecycle.md).

**Reconfiguration** : processus de changement (quel qu’il soit) dans le jeu de réplicas d’un service. Consultez [Reconfiguration](service-fabric-concepts-reconfiguration.md).

**Package de services** : répertoire de disque contenant le fichier `ServiceManifest.xml` du type de service. Ce fichier référence le code, les données statiques et les packages de configuration correspondant au type de service. Les fichiers contenus dans le répertoire de package de services sont référencés par le fichier `ApplicationManifest.xml` du type d’application. Par exemple, un package de services peut faire référence au code, aux données statiques et aux packages de configuration qui composent un service de base de données.

**Service nommé** : après avoir créé une application nommée, vous pouvez créer une instance de l'un de ses types de services au sein du cluster. Vous spécifiez le type de service à l’aide de son nom/sa version. Chaque instance du type de service reçoit un nom d’URI inclus dans l’étendue de l’URI de son application nommée. Par exemple, si vous créez un service nommé « MyDatabase » dans une application nommée « MyNamedApp », l’URI ressemble à ceci : `"fabric:/MyNamedApp/MyDatabase"`. Dans une application nommée, vous pouvez créer plusieurs services nommés. Chaque service nommé peut posséder son propre schéma de partition et son propre nombre d’instances/de réplicas.

**Package de code** : répertoire de disque contenant les fichiers exécutables du type de service (généralement des fichiers DLL/EXE). Les fichiers contenus dans le répertoire de package de code sont référencés par le fichier `ServiceManifest.xml` du type de service. Quand vous créez un service nommé, le package de code est copié vers le ou les nœuds sélectionnés pour exécuter le service nommé. Ensuite, le code commence à s’exécuter. Il existe deux types d’exécutable de code de package :

* **Exécutables invités** : fichiers exécutables qui s'exécutent tels quels sur le système d'exploitation hôte (Windows ou Linux). Ces exécutables ne sont pas liés ou ne font pas référence aux fichiers exécutables de Service Fabric. Par conséquent, ils n’utilisent pas les modèles de programmation de Service Fabric. Ces fichiers exécutables ne peuvent pas utiliser certaines des fonctionnalités de Service Fabric, notamment le service de nommage pour la découverte du point de terminaison. Les exécutables invités ne peuvent pas rapporter des métriques de charge propres à chaque instance de service.
* **Exécutables d'hôte de service** : fichiers exécutables qui utilisent les modèles de programmation de Service Fabric en les liant aux fichiers exécutables de Service Fabric, donnant ainsi accès à des fonctionnalités de Service Fabric. Par exemple, une instance de service nommé peut inscrire les points de terminaison avec le service d’affectation de noms de Service Fabric et indiquer les mesures de chargement.

**Package de données** : répertoire de disque contenant les fichiers de données en lecture seule, statiques du type de service (en général, des fichiers audio et vidéo, et des fichiers de photos). Les fichiers contenus dans le répertoire de package de données sont référencés par le fichier `ServiceManifest.xml` du type de service. Quand vous créez un service nommé, le package de données est copié vers le ou les nœuds sélectionnés pour exécuter le service nommé. Le code s’exécute et peut désormais accéder aux fichiers de données.

**Package de configuration** : répertoire de disque contenant les fichiers de configuration en lecture seule, statiques du type de service (en général, des fichiers texte). Les fichiers contenus dans le répertoire de package de configuration sont référencés par le fichier `ServiceManifest.xml` du type de service. Quand vous créez un service nommé, les fichiers du package de configuration sont copiés vers un ou plusieurs nœuds sélectionnés pour exécuter le service nommé. Ensuite, le code commence à s’exécuter et peut désormais accéder aux fichiers de configuration.

**Conteneurs** : Par défaut, Service Fabric déploie et active les services en tant que processus. Service Fabric peut également déployer des services dans les images de conteneur. La technologie d’un conteneur virtualise le système d’exploitation sous-jacent par rapport aux applications. Chaque application, son runtime, ses dépendances et ses bibliothèques système s’exécutent au sein d’un conteneur. Le conteneur bénéficie d’un accès privé complet à la propre vue isolée du conteneur sur les constructions du système d’exploitation. Service Fabric prend en charge les conteneurs Windows Server et les conteneurs Docker sur Linux. Pour plus d’informations, consultez [Service Fabric et conteneurs](service-fabric-containers-overview.md).

**Schéma de partition** : lors de la création d'un service nommé, vous indiquez un schéma de partition. Les services comportant de grandes quantités d’état fractionnent les données entre les partitions, ce qui permet de répartir l’état entre les nœuds du cluster. Ce fractionnement permet d’adapter l’état de votre service nommé. Dans une partition, les services nommés sans état ont des instances tandis que les services nommés avec état ont des réplicas. En règle générale, les services nommés sans état ne possèdent qu’une partition, car ils sont dépourvus d’état interne. Les instances de partition assurent la disponibilité. En cas d’échec d’une instance, les autres instances continuent de fonctionner normalement, puis Service Fabric en crée une. Comme les services nommés avec état conservent leur état dans les réplicas et que chaque partition possède son propre jeu de réplicas, l’état reste synchronisé. En cas d’échec d’un réplica, Service Fabric en génère un nouveau à partir des réplicas existants.

Pour plus d’informations, consultez l’article [Partitionnement des services fiables Service Fabric](service-fabric-concepts-partitioning.md) .

## <a name="system-services"></a>Services système

Il existe des services système qui sont créés dans chaque cluster et qui fournissent les capacités de plateforme de Service Fabric.

**Service de nommage** : chaque cluster Service Fabric dispose d'un service de nommage qui résout le nom des services vers un emplacement du cluster. Vous gérez le nom et les propriétés des services, comme un système DNS (Domain Name System) Internet du cluster. Les clients communiquent en toute sécurité avec n’importe quel nœud du cluster grâce au service de nommage pour résoudre un nom de service et son emplacement. Les applications se déplacent au sein du cluster, suite, par exemple, à des échecs, à un équilibrage des ressources ou au redimensionnement du cluster. Vous pouvez développer des services et des clients capables de résoudre l’emplacement réseau actuel. Les clients obtiennent l’adresse IP réelle de l’ordinateur et le port utilisé.

Pour plus d’informations sur les API de communication de client et de service qui fonctionnent avec le service de nommage, consultez [Communiquer avec les services](service-fabric-connect-and-communicate-with-services.md).

**Service de magasin d'images** : chaque cluster Service Fabric dispose d'un service de magasin d'images où sont conservés les packages d'application avec version déployés. Copiez un package d’application dans le magasin d’images, puis inscrivez le type d’application contenu dans ce package d’application. Une fois le type d’application configuré, créez une application nommée à partir de celui-ci. Vous pouvez annuler l’inscription d’un type d’application dans le service de magasin d’images après que toutes ses applications nommées ont été supprimées.

Consultez [Comprendre le paramètre ImageStoreConnectionString](service-fabric-image-store-connection-string.md) pour plus d’informations sur le service de magasin d’images.

Pour plus d’informations sur le déploiement d’applications sur le service de magasin d’images, consultez l’article [Déployer une application](service-fabric-deploy-remove-applications.md).

**Service Failover Manager (FM)**  : chaque cluster Service Fabric possède un service Failover Manager chargé des actions suivantes :

 - fonctions liées à la haute disponibilité et à la cohérence des services ;
 - orchestrer les mises à niveau des applications et des clusters ;
 - interagir avec d’autres composants système.

**Service de gestion des réparations** : ce service système optionnel permet d'effectuer des réparations sur un cluster de manière sécurisée, automatisable et transparente. Le service de gestion des réparations est utilisé dans les cas suivants :

   - Maintenance d’Azure sur les clusters Azure Service Fabric avec les niveaux de [durabilité Silver et Gold](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster).
   - Réparations pour l’[application d’orchestration des correctifs](service-fabric-patch-orchestration-application.md)

## <a name="deployment-and-application-models"></a>Modèles de déploiement et d’application

Pour déployer vos services, vous devez décrire la manière dont ils doivent s’exécuter. Service Fabric prend en charge trois modèles de déploiement :

### <a name="resource-model-preview"></a>Modèle de ressource (préversion)

Les ressources Service Fabric sont tout ce qui peut être déployé individuellement sur Service Fabric, dont des applications, des services, des réseaux et des volumes. Les ressources sont définies à l’aide d’un fichier JSON, qui peut être déployé sur un point de terminaison de cluster.  Le schéma de modèle de ressource Azure est utilisé pour Service Fabric Mesh. Un schéma de fichier YAML peut également être utilisé pour créer plus facilement des fichiers de définition. Les ressources peuvent être déployées partout où Service Fabric s’exécute. Le modèle de ressource est la façon la plus simple de décrire vos applications Service Fabric. Son principal objectif est un déploiement et une gestion simples des services en conteneur. Pour en savoir plus, consultez [Présentation du modèle de ressource Azure Service Fabric](../service-fabric-mesh/service-fabric-mesh-service-fabric-resources.md).

### <a name="native-model"></a>Modèle natif

Le modèle d’application natif fournit à vos applications un accès complet de bas niveau à Service Fabric. Les applications et services sont définis en tant que types inscrits dans des fichiers manifeste XML.

Le modèle natif prend en charge les infrastructures Reliable Services et Reliable Actors, qui fournissent un accès aux API de runtime de Service Fabric et aux API de gestion du cluster en C# et Java. Le modèle natif prend également en charge les conteneurs et exécutables arbitraires. Le modèle natif n’est pas pris en charge dans l’[environnement de Service Fabric Mesh](../service-fabric-mesh/service-fabric-mesh-overview.md).

**Reliable Services** : API permettant de générer des services avec et sans état. Les services avec état stockent leur état dans les collections fiables (par exemple, un dictionnaire ou une file d’attente). Vous pouvez également accéder à diverses piles de communication comme l’API web et WCF (Windows Communication Foundation).

**Reliable Actors** : API permettant de générer des objets avec et sans état via le modèle de programmation Actor virtuel. Ce modèle est utile si vous avez un grand nombre d’unités indépendantes d’état/de calcul. Comme ce modèle utilise un modèle de thread en alternance, il convient d’éviter un code qui appelle d’autres acteurs ou services dans la mesure où un acteur individuel ne peut pas traiter les autres demandes entrantes tant que toutes ses demandes sortantes ne sont pas terminées.

Vous pouvez également exécuter vos applications existantes sur Service Fabric :

**Conteneurs** :  Service Fabric prend en charge le déploiement de conteneurs Docker sous Linux et de conteneurs Windows Server sous Windows Server 2016. Il gère également le mode d’isolation Hyper-V. Dans le [modèle d’application](service-fabric-application-model.md)Service Fabric, un conteneur représente un hôte d’application sur lequel sont placés plusieurs réplicas de service. Service Fabric peut exécuter n’importe quel conteneur et le scénario est identique au scénario des exécutables invités, dans lequel vous empaquetez une application existante au sein d’un conteneur. Vous pouvez également exécuter des [services Service Fabric à l’intérieur de conteneurs](service-fabric-services-inside-containers.md).

**Exécutables invités** : Vous pouvez exécuter n’importe quel type de code, tel que Node.js, Python, Java C++ ou dans Azure Service Fabric en tant que service. Dans la terminologie Service Fabric, ces types de services sont appelés exécutables invités et traités comme des services sans état. La haute disponibilité, le contrôle d’intégrité, la gestion du cycle de vie des applications, la haute densité et la fonctionnalité de découverte font partie des avantages de l’exécution d’un exécutable invité dans un cluster Service Fabric.

Pour plus d’informations, consultez l’article [Choisir un modèle de programmation pour votre service](service-fabric-choose-framework.md).

### <a name="docker-compose"></a>Docker Compose 

[Docker Compose](https://docs.docker.com/compose/) fait partie du projet Docker. Service Fabric fournit un support limité pour le [déploiement d’applications à l’aide du modèle Docker Compose](service-fabric-docker-compose.md).

## <a name="environments"></a>Environnements

Service Fabric est une technologie de plateforme open source sur laquelle sont basés plusieurs produits et services différents. Microsoft propose les options suivantes :

 - **Azure Service Fabric Mesh** : service complètement managé pour l’exécution d’applications Service Fabric dans Microsoft Azure.
 - **Azure Service Fabric** : offre de clusters Service Fabric hébergés dans Azure. Il fournit une intégration entre Service Fabric et l’infrastructure Azure, ainsi que la gestion de mise à niveau et de la configuration des clusters Service Fabric.
 - **Service Fabric autonome** : ensemble d’outils d’installation et de configuration pour [déployer des clusters Service Fabric n’importe où](./service-fabric-deploy-anywhere.md) (localement ou sur n’importe quel fournisseur de cloud). Non géré par Azure.
 - **Cluster de développement Service Fabric** : fournit une expérience de développement local sur Windows, Linux ou Mac, pour le développement d’applications Service Fabric.

## <a name="environment-framework-and-deployment-model-support-matrix"></a>Matrice de prise en charge d’environnement, d’infrastructure et de modèle de déploiement

Les différents environnements ont différents niveaux de prise en charge pour les frameworks et les modèles de déploiement. Le tableau suivant décrit les combinaisons d’infrastructure et de modèle de déploiement prises en charge.

| Type d’Application | Décrit par | Azure Service Fabric mesh | Clusters Azure Service Fabric (tout système d’exploitation)| Cluster local | Cluster autonome |
|---|---|---|---|---|---|
| Applications Service Fabric mesh | Modèle de ressource (YAML et JSON) | Prise en charge |Non pris en charge | Windows : pris en charge, Linux et Mac : non pris en charge | Windows : non pris en charge |
|Applications natives Service Fabric | Modèle d’application native (XML) | Non pris en charge| Prise en charge|Prise en charge|Windows : pris en charge|

Le tableau suivant décrit les différents modèles d’application et les outils existants pour ceux-ci par rapport à Service Fabric.

| Type d’Application | Décrit par | Visual Studio | Eclipse | SFCTL | AZ CLI | PowerShell|
|---|---|---|---|---|---|---|
| Applications Service Fabric mesh | Modèle de ressource (YAML et JSON) | VS 2017 |Non pris en charge |Non pris en charge | Pris en charge : environnement Mesh uniquement | Non pris en charge|
|Applications natives Service Fabric | Modèle d’application native (XML) | VS 2017 et VS 2015| Prise en charge|Prise en charge|Prise en charge|Prise en charge|

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Service Fabric :

* [Vue d'ensemble de Service Fabric](service-fabric-overview.md)
* [Pourquoi une approche de microservices pour la conception d’applications ?](service-fabric-overview-microservices.md)
* [Scénarios d’application](service-fabric-application-scenarios.md)

Pour en savoir plus sur Service Fabric Mesh :

* [Vue d’ensemble de Service Fabric Mesh](../service-fabric-mesh/service-fabric-mesh-overview.md)

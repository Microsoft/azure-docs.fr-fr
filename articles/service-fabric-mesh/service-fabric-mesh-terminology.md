---
title: Terminologie d’Azure Service Fabric mesh
description: Cet article décrit en détail la terminologie utilisée dans Azure Service Fabric Mesh pour vous aider à mieux comprendre les termes utilisés dans la documentation.
author: georgewallace
ms.author: gwallace
ms.date: 11/28/2018
ms.topic: conceptual
ms.openlocfilehash: d82df6ee598664c2b85cfa82395ac66f3865cf7f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91843004"
---
# <a name="service-fabric-mesh-terminology"></a>Terminologie de Service Fabric mesh

Azure Service Fabric mesh est un service entièrement géré qui permet aux développeurs de déployer des applications de microservices sans gestion de machines virtuelles, de stockage ou de mise en réseau. Cet article décrit en détail la terminologie utilisée dans Azure Service Fabric Mesh pour vous aider à mieux comprendre les termes utilisés dans la documentation.

## <a name="service-fabric"></a>Service Fabric

[Service Fabric](../service-fabric/index.yml) est une plateforme de systèmes distribués open source qui facilite l’empaquetage, le déploiement et la gestion de microservices évolutifs et fiables. Service Fabric est l’orchestrateur qui sous-tend Service Fabric mesh. Service Fabric offre des options pour la façon de générer et d’exécuter des applications de microservices. Vous pouvez utiliser n’importe quelle infrastructure pour écrire vos services, et choisir l’emplacement où exécuter l’application parmi plusieurs options d’environnement.

## <a name="application-and-service-concepts"></a>Concepts de l’application et du service

**Application Service Fabric Mesh** : les applications Service Fabric Mesh sont décrites par le [modèle de ressource](./service-fabric-mesh-service-fabric-resources.md) (fichiers de ressources YAML et JSON) et peuvent être déployées sur n’importe quel environnement dans lequel Service Fabric s’exécute.

**Application native Service Fabric** : les applications natives Service Fabric sont décrites par le [modèle d’application native](../service-fabric/service-fabric-application-model.md) (manifestes d’applications et de services basés sur XML).  Les applications natives Service Fabric ne peuvent pas s’exécuter dans Service Fabric Mesh.

**Application** : une application Service Fabric Mesh est l’unité de déploiement, le versioning et la durée de vie d’une application Mesh. Le cycle de vie de chaque instance d’application peut être géré de manière indépendante.  Les applications sont composées d’un ou de plusieurs packages de code de service et paramètres. Une application est définie à l’aide du schéma de modèle de ressource Azure (RM).  Les services sont décrits en tant que propriétés de la ressource d’application dans le modèle RM.  Les réseaux et volumes utilisés par l’application sont référencés par l’application.  Lorsque vous créez une application, l’application, les services, le réseau et les volumes sont modélisés à l’aide du modèle de ressources Service Fabric.

**Service** : un service dans une application représente un microservice et exécute une fonction complète et autonome. Chaque service se compose d’un ou plusieurs packages de code qui décrivent tout ce qui est nécessaire pour exécuter l’image conteneur associée au package de code.  Le nombre de réplicas de service dans une application peut être augmenté ou diminué.

**Package de code** : les packages de code décrivent tout ce qui est nécessaire pour exécuter l’image conteneur associée au package de code, notamment :

* Nom, version et registre du conteneur
* Ressources de processeur et de mémoire requises pour chaque conteneur
* Points de terminaison réseau
* Volumes à monter dans le conteneur, faisant référence à une ressource de volume distincte.

## <a name="deployment-and-application-models"></a>Modèles de déploiement et d’application 

Pour déployer vos services, vous devez décrire la manière dont ils doivent s’exécuter. Service Fabric prend en charge trois modèles de déploiement :

### <a name="resource-model"></a>Modèle de ressource
Les ressources Service Fabric sont tout ce qui peut être déployé individuellement sur Service Fabric, dont des applications, des services, des réseaux et des volumes. Les ressources sont définies à l’aide d’un fichier JSON, qui peut être déployé sur un point de terminaison de cluster.  Le schéma de modèle de ressource Azure est utilisé pour Service Fabric Mesh. Un schéma de fichier YAML peut également être utilisé pour créer plus facilement des fichiers de définition. Les ressources peuvent être déployées partout où Service Fabric s’exécute. Le modèle de ressource est la façon la plus simple de décrire vos applications Service Fabric. Son principal objectif est un déploiement et une gestion simples des services en conteneur. Pour en savoir plus, consultez [Présentation du modèle de ressource Azure Service Fabric](./service-fabric-mesh-service-fabric-resources.md).

### <a name="native-model"></a>Modèle natif
Le modèle d’application natif fournit à vos applications un accès complet de bas niveau à Service Fabric. Les applications et services sont définis en tant que types inscrits dans des fichiers manifeste XML.

Le modèle natif prend en charge l’infrastructure Reliable Services, qui fournit un accès aux API de runtime de Service Fabric et aux API de gestion du cluster en C# et Java. Le modèle natif prend également en charge les conteneurs et exécutables arbitraires.

Le modèle natif n’est pas pris en charge dans l’environnement de Service Fabric mesh.  Pour plus d’informations, consultez l’article [Vue d’ensemble des modèles de programmation Service Fabric](../service-fabric/service-fabric-choose-framework.md).

### <a name="docker-compose"></a>Docker Compose 
[Docker Compose](https://docs.docker.com/compose/) fait partie du projet Docker. Service Fabric fournit un support limité pour le déploiement d’applications à l’aide du modèle Docker Compose.

## <a name="environments"></a>Environnements

Service Fabric est une technologie de plateforme open source sur laquelle sont basés plusieurs produits et services différents. Microsoft propose les options suivantes :

 - **Service Fabric Mesh** : service complètement managé pour l’exécution d’applications Service Fabric dans Microsoft Azure.
 - **Azure Service Fabric** : offre de clusters Service Fabric hébergés dans Azure. Il fournit une intégration entre Service Fabric et l’infrastructure Azure, ainsi que la gestion de mise à niveau et de la configuration des clusters Service Fabric.
 - **Service Fabric autonome** : ensemble d’outils d’installation et de configuration pour [déployer des clusters Service Fabric n’importe où](../service-fabric/service-fabric-deploy-anywhere.md) (localement ou sur n’importe quel fournisseur de cloud). Non géré par Azure.
 - **Cluster de développement Service Fabric** : fournit une expérience de développement local sur Windows, Linux ou Mac, pour le développement d’applications Service Fabric.

## <a name="environment-framework-and-deployment-model-support-matrix"></a>Matrice de prise en charge d’environnement, d’infrastructure et de modèle de déploiement
Les différents environnements ont différents niveaux de prise en charge pour les infrastructures et les modèles de déploiement. Le tableau suivant décrit les combinaisons d’infrastructure et de modèle de déploiement prises en charge.

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

Pour en savoir plus sur Service Fabric mesh, voir la [vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md).

Lisez les réponses aux [questions fréquentes](service-fabric-mesh-faq.md).

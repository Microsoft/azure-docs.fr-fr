---
title: Vue d’ensemble de Service Fabric et des conteneurs
description: Voici une vue d’ensemble de Service Fabric et de la méthode à suivre pour déployer des applications de microservices au moyen de conteneurs. Cet article fournit une vue d’ensemble de l’utilisation de conteneurs et des fonctionnalités disponibles dans Service Fabric.
ms.topic: conceptual
ms.date: 7/9/2020
ms.openlocfilehash: cd0ec7dd2247fdd791df362fa34542178c17df4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87091655"
---
# <a name="service-fabric-and-containers"></a>Service Fabric et conteneurs

## <a name="introduction"></a>Introduction

Azure Service Fabric est une plateforme de systèmes distribués qui facilite le packaging, le déploiement et la gestion de conteneurs et de microservices évolutifs et fiables.

Service Fabric est [l’orchestrateur de conteneurs](service-fabric-cluster-resource-manager-introduction.md) de Microsoft qui permet de déployer des microservices sur un cluster de machines. Service Fabric tire profit de l’expérience acquise au cours de ses années d’exécution de services à grande échelle, chez Microsoft.

Les microservices peuvent être développés de nombreuses manières, par le biais des [modèles de programmation de Service Fabric](service-fabric-choose-framework.md), [ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md), ou en déployant [n’importe quel code de votre choix](service-fabric-guest-executables-introduction.md). Si vous souhaitez simplement [déployer et gérer des conteneurs](service-fabric-containers-overview.md), Service Fabric se révèle également un excellent choix.

Par défaut, Service Fabric déploie et active ces services en tant que processus. Ces processus assurent l’activation la plus rapide et offrent la densité la plus élevée en matière de ressources dans un cluster. Service Fabric peut également déployer des services dans les images de conteneur. Au sein de la même application, vous pouvez également combiner des services avec des processus, et des services avec des conteneurs.

Pour vous familiariser rapidement et essayer les conteneurs sur Service Fabric, suivez un guide de démarrage rapide, un tutoriel ou un exemple pratique :  

[Démarrage rapide : Déployer une application conteneur Linux sur Service Fabric](service-fabric-quickstart-containers-linux.md)  
[Démarrage rapide : Déployer une application conteneur Windows sur Service Fabric](service-fabric-quickstart-containers.md)  
[Mettre en conteneur une application .NET](service-fabric-host-app-in-a-container.md)  
[Exemples de conteneurs Service Fabric](https://azure.microsoft.com/resources/samples/service-fabric-containers/)  

## <a name="what-are-containers"></a>Les conteneurs

Les conteneurs constituent la solution au problème de fiabilité de l’exécution des applications dans les environnements informatiques différents, en fournissant un environnement immuable dans lequel exécuter l’application. Les conteneurs wrappent une application et toutes ses dépendances (telles que les bibliothèques et les fichiers de configuration) dans sa propre « zone » isolée qui contient tous les éléments nécessaires à l’exécution du logiciel à l’intérieur du conteneur. Chaque fois que le conteneur s’exécute, l’application qu’il renferme a toujours ce dont elle a besoin pour s’exécuter, telles que les versions appropriées de ses bibliothèques dépendantes, les fichiers de configuration et tout autre élément qui lui est utile pour son exécution.

Les conteneurs s’exécutent directement sur le noyau et disposent d’une vue isolée sur le système de fichiers et d’autres ressources. Une application située dans un conteneur n’a aucune connaissance des autres applications ou processus se trouvant en dehors de son conteneur. Chaque application, avec son runtime, ses dépendances et ses bibliothèques système, s’exécute dans un conteneur, en bénéficiant d’un accès privé complet à la propre vue isolée du conteneur sur le système d’exploitation. Non seulement la fourniture de toutes les dépendances nécessaires à l’exécution de votre application dans différents environnements informatiques est facilitée, mais la sécurité et l’isolation des ressources présentent des avantages importants lors de l’utilisation de conteneurs avec Service Fabric qui, sinon, exécute les services dans un processus.

Par rapport aux machines virtuelles, les conteneurs présentent les avantages suivants :

* **Petit** : les conteneurs utilisent un espace de stockage unique ainsi que les versions et mises à jour de la couche pour une efficacité accrue.
* **Démarrage rapide** : comme les conteneurs n’ont pas besoin d’initialiser l’intégralité d’un système d’exploitation, ils peuvent démarrer beaucoup plus rapidement, généralement en quelques secondes.
* **Portabilité** : une image d’application en conteneur peut être portée de manière à s’exécuter dans le cloud ou en local, à l’intérieur de machines virtuelles ou directement sur des machines physiques.
* **Gouvernance des ressources** : le nombre de ressources physiques qu’un conteneur peut consommer sur son hôte peut être limité.

### <a name="container-types-and-supported-environments"></a>Types de conteneurs et environnements pris en charge

Service Fabric prend en charge les conteneurs sur Linux et sur Windows ; le mode d’isolation Hyper-V est pris en charge sur Windows.

#### <a name="docker-containers-on-linux"></a>Conteneurs Docker sur Linux

Docker fournit des API pour créer et gérer des conteneurs sur des conteneurs du noyau Linux. Docker Hub fournit un dépôt central permettant de stocker et de récupérer des images conteneur.
Pour suivre un tutoriel Linux, voir [Créer votre première application conteneur Service Fabric sur Linux](service-fabric-get-started-containers-linux.md).

#### <a name="windows-server-containers"></a>Conteneurs Windows Server

Windows Server 2016 et versions ultérieures fournissent deux types de conteneurs qui proposent des niveaux d’isolation différents. Les conteneurs Windows Server sont similaires aux conteneurs Docker, en ce sens qu’ils offrent tous l’isolation des systèmes de fichiers et des espaces de noms, alors qu’ils partagent le noyau avec l’hôte sur lequel ils s’exécutent. Sur Linux, cette isolation est généralement fournie via des espaces de noms et cgroups. Les conteneurs Windows Server se comportent de la même manière.

Les conteneurs Windows avec la prise en charge Hyper-V présentent un niveau d’isolation et de sécurité plus élevé, car aucun d’entre eux ne partage le noyau du système d’exploitation avec les autres conteneurs, ou avec l’hôte. Avec ce niveau élevé d’isolation en matière de sécurité, les conteneurs compatibles Hyper-V sont destinés à des scénarios multilocataires, potentiellement hostiles.
Pour suivre un tutoriel Windows, voir [Créer votre première application conteneur Service Fabric sur Windows](service-fabric-get-started-containers.md).

La figure suivante illustre les différents types de virtualisation et niveaux d’isolation disponibles.
![Plateforme Service Fabric][Image1]

## <a name="scenarios-for-using-containers"></a>Scénarios d’utilisation des conteneurs

Voici des exemples pour lesquels le conteneur est un bon choix :

* **Opération lift-and-shift pour IIS** : vous pouvez placer une application [ASP.NET MVC](https://www.asp.net/mvc) existante dans un conteneur au lieu de la faire migrer sur ASP.NET Core. Ces applications ASP.NET MVC dépendent des services Internet (IIS, Internet Information Services). Vous pouvez empaqueter ces applications dans des images de conteneur à partir de l’image IIS créée au préalable, puis les déployer avec Service Fabric. Pour plus d’informations sur les conteneurs Windows, consultez la page [Images conteneurs sur Windows Server](/virtualization/windowscontainers/quick-start/quick-start-windows-server).

* **Mélange de conteneurs et de microservices Service Fabric** : Utilisez une image de conteneur existante pour une partie de votre application. Par exemple, vous pouvez utiliser le [conteneur NGINX](https://hub.docker.com/_/nginx/) pour le système frontal web de votre application et les services avec état pour les calculs les plus intenses du back-end.

* **Réduction de l’impact des services de « voisins bruyants »**  : vous pouvez utiliser la capacité de gouvernance des ressources des conteneurs pour limiter le nombre de ressources utilisées par un service sur un hôte. Si les services sont susceptibles de consommer un grand nombre de ressources et, de ce fait, d’affecter les performances d’autres services (opération de type requête exécutée sur le long terme, par exemple), vous pouvez envisager de les placer dans des conteneurs soumis à la gouvernance des ressources.

## <a name="service-fabric-support-for-containers"></a>Prise en charge des conteneurs par Service Fabric

Service Fabric prend en charge le déploiement de conteneurs Docker sur Linux, et celui de conteneurs Windows Server sur Windows Server 2016 et versions ultérieures avec une prise en charge du mode d’isolation Hyper-V.

> [!NOTE]
> Les conteneurs ne sont pas pris en charge sur les clusters Service Fabric locaux à nœud unique (ni les clusters Linux sur OneBox, ni les clusters Windows sur les installations Service Fabric locales).

Service Fabric fournit un [modèle d’application](service-fabric-application-model.md) dans lequel un conteneur représente un hôte d’application sur lequel sont placés plusieurs réplicas de service. Service Fabric prend également en charge un [scénario d’exécutables invités](service-fabric-guest-executables-introduction.md), dans lequel vous n’utilisez pas les modèles de programmation Service Fabric intégrés, mais empaquetez plutôt une application existante, écrite à l’aide de n’importe quel langage ou framework, à l’intérieur d’un conteneur. Ce scénario est le cas d’utilisation courant pour les conteneurs.

Vous pouvez également exécuter les [services Service Fabric à l’intérieur d’un conteneur](service-fabric-services-inside-containers.md). La prise en charge de cette exécution est pour l’instant limitée.

Service Fabric propose plusieurs fonctionnalités de gestion des conteneurs, celles-ci vous aident à créer des applications composées de microservices mis en conteneur, telles que :

* Activation et déploiement d’images de conteneur
* Gouvernance des ressources, notamment la définition des valeurs des ressources par défaut sur les clusters Azure.
* Authentification de référentiels.
* Mappage des ports de conteneur aux ports hôtes.
* Découverte et communication entre des conteneurs.
* Possibilité de configurer et de définir des variables d’environnement.
* Possibilité de définir des identifiants de sécurité sur le conteneur.
* Différents modes de mise en réseau pour les conteneurs.

Pour une vue d’ensemble complète de la prise en charge des conteneurs sur Azure, par exemple la création d’un cluster Kubernetes avec Azure Kubernetes Service, la création d’un registre Docker privé dans Azure Container Registry, et bien plus encore, voir [Azure for Containers](../containers/index.yml).

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez découvert la prise en charge offerte par Service Fabric pour les conteneurs en cours d’exécution. Nous allons à présent parcourir ces différentes fonctionnalités illustrées par des exemples, et vous montrer comment les utiliser.

[Créer sa première application conteneur Service Fabric sous Linux](service-fabric-get-started-containers-linux.md)  
[Créer sa première application conteneur Service Fabric sous Windows](service-fabric-get-started-containers.md)  
[En savoir plus sur les conteneurs Windows](/virtualization/windowscontainers/about/)

[Image1]: media/service-fabric-containers/Service-Fabric-Types-of-Isolation.png

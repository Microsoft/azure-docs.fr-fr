---
title: Vue d’ensemble d’Azure Service Fabric
description: Service Fabric est une plateforme de systèmes distribués qui permet de créer des microservices évolutifs, fiables et faciles à gérer.
ms.topic: overview
ms.date: 09/22/2020
ms.custom: contperf-fy21q1
ms.openlocfilehash: b6a14ba3f905fbef3fc3796b616cd594941a3d60
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028629"
---
# <a name="overview-of-azure-service-fabric"></a>Vue d’ensemble d’Azure Service Fabric

Azure Service Fabric est une [plateforme de systèmes distribués](#container-orchestration) qui facilite le packaging, le déploiement et la gestion de conteneurs et de microservices évolutifs et fiables. Service Fabric résout également les problèmes non négligeables du [développement et de la gestion](#application-lifecycle-management) des applications natives au cloud.

La singularité de Service Fabric tient au fait qu’il est axé sur la création de services avec état. Vous pouvez utiliser le [modèle de programmation](#stateless-and-stateful-microservices) Service Fabric ou exécuter des services avec état en conteneur écrits dans n’importe quel langage ou code. Vous pouvez créer des [clusters Service Fabric n’importe où](#any-os-any-cloud), y compris Windows Server et Linux en local et d’autres clouds publics, en plus d’Azure.

![La plateforme Service Fabric offre la gestion de cycle de vie, la disponibilité, l’orchestration, les modèles de programmation, l’intégrité et la surveillance, les outils de développement et d’opérations, et la mise à l’échelle automatique (dans Azure, localement, dans d’autres Clouds et sur votre ordinateur de développement)][Image1]

Service Fabric alimente de nombreux services Microsoft aujourd’hui, notamment Azure SQL Database, Azure Cosmos DB, Cortana, Microsoft Power BI, Microsoft Intune, Azure Event Hubs, Azure IoT Hub, Dynamics 365, Skype Entreprise et de nombreux services Azure principaux.

## <a name="container-orchestration"></a>Orchestration de conteneurs

Service Fabric est l’[orchestrateur de conteneurs](service-fabric-cluster-resource-manager-introduction.md) de Microsoft pour le déploiement et la gestion de microservices sur un cluster de machines. Il bénéficie des leçons tirées de l’utilisation des services Microsoft à grande échelle. Service Fabric peut déployer des applications en quelques secondes, à haute densité et avec des centaines, voire des milliers d’applications ou de conteneurs par machine. Avec Service Fabric, vous pouvez combiner des services dans des processus et dans des conteneurs au sein de la même application.

Apprenez-en plus sur les concepts fondamentaux, les modèles de programmation, le cycle de vie d’application, les tests, les clusters et la surveillance de l’intégrité dans [Service Fabric](service-fabric-content-roadmap.md).

## <a name="stateless-and-stateful-microservices"></a>Microservices avec et sans état

Service Fabric fournit un runtime léger et sophistiqué qui prend en charge les microservices sans état et avec état. L’un des principaux atouts de Service Fabric est sa prise en charge robuste de la création de services avec état, que ce soit avec les [modèles de programmation intégrés](service-fabric-choose-framework.md) Service Fabric ou les services avec état en conteneur.

Apprenez-en plus sur les [scénarios d’application](service-fabric-application-scenarios.md) qui tirent parti des services avec état Service Fabric.

## <a name="application-lifecycle-management"></a>Gestion du cycle de vie des applications

Service Fabric offre une prise en charge du cycle de vie complet des applications cloud CI/CD, y compris des conteneurs : du développement au retrait éventuel, en passant par le déploiement, l’analyse, la gestion et la maintenance quotidiennes. Service Fabric est intégré avec des outils de CI/CD tels que [Azure Pipelines](https://www.visualstudio.com/team-services/), [Jenkins](https://jenkins.io/index.html), et [Octopus Deploy](https://octopus.com/) et peut être utilisé avec n’importe quel autre outil de CI/CD populaire.

Pour plus d’informations sur la gestion du cycle de vie des applications, consultez [Cycle de vie des applications](service-fabric-application-lifecycle.md). Pour déployer des applications existantes sur Service Fabric, consultez [Déployer un exécutable invité](service-fabric-deploy-existing-app.md).

## <a name="any-os-any-cloud"></a>Tous les systèmes d’exploitation, tous les clouds

Vous pouvez créer des clusters pour Service Fabric dans de nombreux environnements, notamment [Azure ou en local](service-fabric-deploy-anywhere.md), sous [Windows Server ou sous Linux](service-fabric-linux-windows-differences.md). Vous pouvez même créer des clusters sur d’autres clouds publics. L’environnement de développement dans le Kit de développement logiciel (SDK) Service Fabric est identique à l’environnement de production, sans aucun émulateur impliqué. En d’autres termes, ce qui est exécuté sur votre cluster de développement local est ce qui est déployé sur vos clusters dans les autres environnements.

Pour le [développement Windows](service-fabric-get-started.md), le Kit de développement logiciel (SDK) .NET Service Fabric est intégré à Visual Studio et à PowerShell. Pour le [développement Linux](service-fabric-get-started-linux.md), le Kit SDK Java Service Fabric est intégré à Eclipse, et Yeoman permet de générer des modèles pour des applications Java, .NET Core et conteneurs.

## <a name="compliance"></a>Conformité

Le fournisseur de ressources Azure Service Fabric est disponible dans toutes les régions Azure, et il est conforme à tous les certifications de conformité Azure, notamment : SOC, ISO, PCI DSS, HIPAA et GDPR. Pour une liste complète, consultez [Offres de conformité Microsoft](https://www.microsoft.com/trustcenter/compliance/complianceofferings).

## <a name="next-steps"></a>Étapes suivantes

Créez et déployez votre première application sur Microsoft Azure Service Fabric :

> [!div class="nextstepaction"]
> [Guide de démarrage rapide Service Fabric][sf-quickstart]

[Image1]: media/service-fabric-overview/Service-Fabric-Overview.png
[sf-quickstart]: ./service-fabric-quickstart-dotnet.md

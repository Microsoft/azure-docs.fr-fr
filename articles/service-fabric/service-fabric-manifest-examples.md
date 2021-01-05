---
title: Exemples de manifestes d’applications Azure Service Fabric
description: Découvrez comment configurer les paramètres des manifestes d’applications et de services pour une application Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 06/11/2018
ms.author: pepogors
ms.openlocfilehash: 6f44d2ddaac11545a3a3dc52fd20e02263c6ad67
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75451637"
---
# <a name="service-fabric-application-and-service-manifest-examples"></a>Exemples de manifestes de services et d’applications Service Fabric
Cette section fournit des exemples de manifestes d’applications et de services. Ces exemples ne sont pas destinés à présenter des scénarios importants, mais à démontrer les différents paramètres disponibles et leur fonctionnement. 

Le document suivant est un index des fonctionnalités présentées et des manifestes d’exemples auxquelles elles appartiennent.

|Fonctionnalité|Manifeste|
|---|---|
|[Gouvernance des ressources](service-fabric-resource-governance.md)|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest), [Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Exécuter un service en tant que compte administrateur local](service-fabric-application-runas-security.md)|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Application d’une stratégie par défaut à tous les packages de code de service](service-fabric-application-runas-security.md#apply-a-default-policy-to-all-service-code-packages)|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Créer des principaux d’utilisateur et de groupe](service-fabric-application-runas-security.md)|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|partager un package de données entre des instances de service|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Écraser des points de terminaison de service](service-fabric-service-manifest-resources.md#overriding-endpoints-in-servicemanifestxml)|[Manifeste de l’application Reliable Services](service-fabric-manifest-example-reliable-services-app.md#application-manifest)|
|[Exécuter un script au démarrage du service](service-fabric-run-script-at-service-startup.md)|[Manifeste de service VotingWeb](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[Définir un point de terminaison HTTPS](service-fabric-tutorial-dotnet-app-enable-https-endpoint.md#define-an-https-endpoint-in-the-service-manifest)|[Manifeste de service VotingWeb](service-fabric-manifest-example-reliable-services-app.md#votingweb-service-manifest)|
|[Déclarer un package de configuration](service-fabric-application-and-service-manifests.md)|[Manifeste de service VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[Déclarer un package de données](service-fabric-application-and-service-manifests.md)|[Manifeste de service VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|[Écraser des variables d’environnement](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Configuration du mappage des ports de conteneur à l’hôte](service-fabric-get-started-containers.md#configure-container-port-to-host-port-mapping-and-container-to-container-discovery)| [Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Configurer l’authentification au registre de conteneurs](service-fabric-get-started-containers.md#configure-container-repository-authentication)|[Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Définir le mode d’isolation](service-fabric-get-started-containers.md#configure-isolation-mode)|[Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Spécifier les images conteneur spécifiques au build du système d’exploitation](service-fabric-get-started-containers.md#specify-os-build-specific-container-images)|[Manifeste de l’application de conteneur](service-fabric-manifest-example-container-app.md#application-manifest)|
|[Définir des variables d’environnement](service-fabric-get-started-containers.md#configure-and-set-environment-variables)|[Manifeste de service de conteneur FrontEndService](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest), [Manifeste de service de conteneur BackEndService](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|
|[Configurer un point de terminaison](service-fabric-get-started-containers.md#configure-communication)|[Manifeste de service de conteneur FrontEndService](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest), [Manifeste de service de conteneur BackEndService](service-fabric-manifest-example-container-app.md#backendservice-service-manifest), [Manifeste de service VotingData](service-fabric-manifest-example-reliable-services-app.md#votingdata-service-manifest)|
|transmettre des commandes au conteneur|[Manifeste de service de conteneur FrontEndService](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[Importer un certificat dans un conteneur](service-fabric-securing-containers.md)|[Manifeste de service de conteneur FrontEndService](service-fabric-manifest-example-container-app.md#frontendservice-service-manifest)|
|[Configurer le pilote de volume](service-fabric-containers-volume-logging-drivers.md)|[Manifeste de service de conteneur BackEndService](service-fabric-manifest-example-container-app.md#backendservice-service-manifest)|


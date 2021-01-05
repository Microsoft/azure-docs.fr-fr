---
title: Modèle de données de télémétrie d’Azure Application Insights- Contexte de télémétrie | Microsoft Docs
description: Modèle de données du contexte de télémétrie d’Application Insights
ms.topic: conceptual
ms.date: 05/15/2017
ms.reviewer: sergkanz
ms.openlocfilehash: 76f3be510494a1f005b0080ee8f2390a3fbc3622
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91767838"
---
# <a name="telemetry-context-application-insights-data-model"></a>Contexte de télémétrie : Modèle de données Application Insights

Chaque élément de télémétrie peut avoir des champs de contexte fortement typés. Chaque champ permet un scénario de surveillance spécifique. Utilisez la collection de propriétés personnalisées pour stocker des informations contextuelles personnalisées ou spécifiques à l’application.


## <a name="application-version"></a>Version de l’application

Les informations dans les champs de contexte de l’application concernent toujours l’application qui envoie les données de télémétrie. La version de l’application est utilisée pour analyser la tendance des changements dans le comportement des applications et sa corrélation avec les déploiements.

Longueur maximale : 1 024


## <a name="client-ip-address"></a>Adresse IP du client

Adresse IP de l’appareil client. IPv4 et IPv6 sont pris en charge. Quand des données de télémétrie sont envoyées à partir d’un service, le contexte de l’emplacement concerne l’utilisateur qui a lancé l’opération dans le service. Application Insights extrait les informations d’emplacement géographique de l’adresse IP du client puis les tronque. Ainsi, l’adresse IP du client en elle-même ne peut pas être utilisée comme information identifiable de l’utilisateur final. 

Longueur maximale : 46


## <a name="device-type"></a>Type d’appareil

À l’origine, ce champ était utilisé pour indiquer le type de l’appareil utilisé par l’utilisateur final de l’application. Aujourd’hui, il est utilisé principalement pour distinguer la télémétrie JavaScript avec le type d’appareil « Navigateur » de la télémétrie côté serveur avec le type d’appareil « PC ».

Longueur maximale : 64


## <a name="operation-id"></a>ID d’opération

Identificateur unique de l’opération racine. Cet identificateur permet de regrouper les données de télémétrie pour plusieurs composants. Pour plus d’informations, consultez [Corrélation de la télémétrie](./correlation.md). L’ID d’opération est créé par une demande ou par la vue d’une page. Le reste de la télémétrie définit ce champ sur la valeur correspondant à la demande ou à la vue de page qui la contient. 

Longueur maximale : 128


## <a name="parent-operation-id"></a>ID d’opération parent

Identificateur unique du parent immédiat de l’élément de télémétrie. Pour plus d’informations, consultez [Corrélation de la télémétrie](./correlation.md).

Longueur maximale : 128


## <a name="operation-name"></a>Nom d’opération

Nom (groupe) de l’opération. Le nom de l’opération est créé par une demande ou par la vue d’une page. Tous les autres éléments de télémétrie définissent ce champ sur la valeur correspondant à la demande ou à la vue de page qui la contient. Le nom de l’opération est utilisé pour rechercher tous les éléments de télémétrie pour un groupe d’opérations (par exemple « GET Home/Index »). Cette propriété de contexte est utilisée pour répondre à des questions comme « quelles sont les exceptions généralement levées sur cette page ? ».

Longueur maximale : 1 024


## <a name="synthetic-source-of-the-operation"></a>Source synthétique de l’opération

Nom de la source synthétique. Certaines données de télémétrie de l’application peuvent représenter le trafic synthétique. Il peut s’agir d’un robot d’indexation web indexant le site web, de tests de disponibilité du site ou de traces de bibliothèques de diagnostic, comme le SDK Application Insights lui-même.

Longueur maximale : 1 024


## <a name="session-id"></a>ID de session

ID de session : l’instance de l’interaction de l’utilisateur avec l’application. Les informations dans les champs de contexte de session concernent toujours l’utilisateur final. Quand des données de télémétrie sont envoyées à partir d’un service, le contexte de session concerne l’utilisateur qui a lancé l’opération dans le service.

Longueur maximale : 64


## <a name="anonymous-user-id"></a>ID d’utilisateur anonyme

ID d’utilisateur anonyme. Représente l’utilisateur final de l’application. Quand des données de télémétrie sont envoyées à partir d’un service, le contexte d’utilisateur concerne l’utilisateur qui a lancé l’opération dans le service.

[L’échantillonnage](./sampling.md) est une des techniques pour réduire la quantité de données de télémétrie collectées. L’algorithme d’échantillonnage tente de prélever des échantillons dans ou en dehors de toute la télémétrie corrélée. L’ID d’utilisateur anonyme est utilisé pour la génération de score d’échantillonnage. L’ID d’utilisateur anonyme doit donc être une valeur suffisamment aléatoire. 

Utiliser l’ID d’utilisateur anonyme pour stocker un nom d’utilisateur est une utilisation incorrecte du champ. Utilisez un ID d’utilisateur authentifié.

Longueur maximale : 128


## <a name="authenticated-user-id"></a>ID d’utilisateur authentifié

ID d’utilisateur authentifié. À l’opposé de l’ID d’utilisateur anonyme, ce champ représente l’utilisateur avec un nom convivial. Ces données sont collectées par défaut avec l’élément [`AuthenticatedUserIdTelemetryInitializer`](https://github.com/microsoft/ApplicationInsights-dotnet/blob/develop/WEB/Src/Web/Web/AuthenticatedUserIdTelemetryInitializer.cs) du SDK ASP.NET Framework.  

Longueur maximale : 1 024


## <a name="account-id"></a>ID de compte

Dans les applications multi-locataires, il s’agit de l’ID ou du nom du compte avec lequel l’utilisateur agit. C’est, par exemple, un ID d’abonnement pour le portail Azure ou un nom de blog pour une plateforme de blog.

Longueur maximale : 1 024


## <a name="cloud-role"></a>Rôle cloud

Nom du rôle dont l’application fait partie. Mappé directement au le nom du rôle dans Azure. Peut également être utilisé pour distinguer des microservices, qui font partie d’une même application.

Longueur maximale : 256


## <a name="cloud-role-instance"></a>Instance de rôle cloud

Nom de l’instance où l’application s’exécute. Nom de l’ordinateur pour un ordinateur local, nom de l’instance pour Azure.

Longueur maximale : 256


## <a name="internal-sdk-version"></a>Interne : Version du SDK

Version du SDK. Pour plus d’informations, consultez [cet article](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/EndpointSpecs/SDK-VERSIONS.md).

Longueur maximale : 64


## <a name="internal-node-name"></a>Interne : Nom du nœud

Ce champ représente le nom du nœud utilisé pour la facturation. Utilisez-le pour remplacer la détection standard de nœuds.

Longueur maximale : 256


## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [étendre et filtrer la télémétrie](./api-filtering-sampling.md).
- Pour connaître les types et les modèles de données Application Insights, consultez [Modèle de données](data-model.md).
- Découvrez la [configuration](./configuration-with-applicationinsights-config.md#telemetry-initializers-aspnet) de la collection de propriétés de contexte standard.


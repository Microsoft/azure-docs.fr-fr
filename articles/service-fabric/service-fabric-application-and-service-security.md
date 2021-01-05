---
title: Découvrir la sécurité des applications Azure Service Fabric
description: Explique comment exécuter des applications de microservices de manière sécurisée dans Service Fabric. Découvrez comment exécuter des services et un script de démarrage sous différents comptes de sécurité, authentifier et autoriser des utilisateurs, gérer les secrets des applications, sécuriser les communications avec les services, utiliser une passerelle API et sécuriser des données d’application au repos.
ms.topic: conceptual
ms.date: 03/16/2018
ms.openlocfilehash: f17840f31d2a4c12a1d4618bd16e81dcc2cc8a14
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86256575"
---
# <a name="service-fabric-application-and-service-security"></a>Sécurité des applications et des services Service Fabric
Une architecture de microservices peut présenter de [nombreux avantages](service-fabric-overview-microservices.md). Cependant, la gestion de la sécurité des microservices représente un défi autrement plus complexe que celui constitué par la gestion de la sécurité des applications monolithiques traditionnelles. 

Les applications monolithiques s’exécutent généralement sur un ou plusieurs serveurs au sein d’un réseau, où il est plus facile d’identifier les ports exposés, les API et l’adresse IP. Il existe souvent un périmètre ou une frontière, et une base de données à protéger. Si ce système est compromis en raison d’une violation de la sécurité ou d’une attaque, il est probable que l’attaquant aura accès à tous les éléments du système. Avec les microservices, le système est plus complexe.  Les services sont décentralisés et distribués sur plusieurs hôtes, et migrent d’un hôte à l’autre.  Avec une sécurité adaptée, vous pouvez limiter les privilèges que peut obtenir un attaquant, ainsi que la quantité de données qu’il est possible d’obtenir en une seule attaque par violation du service.  La communication ne se produit pas en interne, mais sur un réseau. Les ports exposés sont nombreux, ainsi que les interactions entre les services. Pour la sécurité de votre application, il est essentiel de connaître ces interactions de service et de savoir quand elles se produisent.

Cet article n’est pas un guide de sécurité des microservices (de tels guides sont déjà disponibles en ligne), mais il décrit comment configurer les différents aspects de la sécurité dans Service Fabric.

## <a name="authentication-and-authorization"></a>Authentification et autorisation
Il est souvent nécessaire de limiter les ressources et les API exposées par un service à certains utilisateurs ou clients approuvés. L’authentification est le processus qui permet de vérifier de manière fiable l’identité d’un utilisateur.  L’autorisation est le processus qui permet aux seuls utilisateurs authentifiés d’accéder aux API et aux services disponibles.

### <a name="authentication"></a>Authentification
L’authentification est la première chose à laquelle vous devez penser si vous devez décider d’une approbation au niveau des API. L’authentification est le processus qui permet de vérifier de manière fiable l’identité d’un utilisateur.  Dans les scénarios de microservices, l’authentification est généralement gérée de manière centralisée. Si vous utilisez une passerelle d’API, vous pouvez [déléguer l’authentification](/azure/architecture/patterns/gateway-offloading) à la passerelle. Si vous utilisez cette approche, vérifiez que les services ne sont pas accessibles directement (sans la passerelle API), sauf si une mesure de sécurité supplémentaire a été mise en place pour authentifier les messages, qu’ils proviennent ou non de la passerelle.

Si les services sont accessibles directement, un service d’authentification, comme Azure Active Directory ou un microservice d’authentification dédié faisant office de service d’émission de jeton de sécurité (STS), peut être utilisé pour authentifier les utilisateurs. Les décisions d’approbation sont partagées entre les services à l’aide de jetons de sécurité ou de cookies. 

Pour ASP.NET Core, le mécanisme principal [d’authentification des utilisateurs](/dotnet/standard/microservices-architecture/secure-net-microservices-web-applications/) est le système d’appartenance ASP.NET Core Identity. ASP.NET Core Identity stocke les informations utilisateur (notamment les revendications, les rôles et les informations de connexion) dans un magasin de données configuré par le développeur. ASP.NET Core Identity prend en charge l’authentification à 2 facteurs.  Les fournisseurs d’authentification externes sont également pris en charge, ce qui permet aux utilisateurs de se connecter à l’aide de processus d’authentification existants, comme ceux de Microsoft, Google, Facebook ou Twitter.

### <a name="authorization"></a>Autorisation
Après l’authentification, les services doivent autoriser l’accès utilisateur ou déterminer ce qu’un utilisateur est autorisé à faire. Ce processus permet à un service d’autoriser uniquement les utilisateurs authentifiés à accéder aux API. L’autorisation est orthogonale et indépendante de l’authentification, qui est le processus permettant d’identifier un utilisateur. L’authentification peut créer une ou plusieurs identités pour l’utilisateur actuel.

[L’autorisation ASP.NET Core](/dotnet/standard/microservices-architecture/secure-net-microservices-web-applications/authorization-net-microservices-web-applications) peut être basée sur les rôles d’utilisateurs ou sur une stratégie personnalisée pouvant inclure l’inspection de revendications ou autres méthodes heuristiques.

## <a name="restrict-and-secure-access-using-an-api-gateway"></a>Limiter et sécuriser l’accès à l’aide d’une passerelle API
Les applications cloud ont généralement besoin d’une passerelle frontale afin de fournir un point d’entrée unique pour les utilisateurs, les appareils ou d’autres applications. Une [passerelle API](/azure/architecture/microservices/gateway) est située entre les clients et les services, et constitue le point d’entrée de tous les services fournis par votre application. Elle agit comme un proxy inverse, en acheminant les requêtes des clients vers les services. Elle peut également effectuer diverses tâches transversales telles que l’authentification et l’autorisation, l’arrêt TLS et la limitation du débit. Si vous ne pouvez pas déployer de passerelle, les clients doivent envoyer des requêtes directement aux services frontaux.

Dans Service Fabric, une passerelle peut être n’importe quel service sans état, comme une [application ASP.NET Core](service-fabric-reliable-services-communication-aspnetcore.md), ou un autre service conçu pour l’entrée de trafic, par exemple [Traefik](https://docs.traefik.io/), [Event Hubs](../event-hubs/index.yml), [IoT Hub](../iot-hub/index.yml) ou la [ Gestion des API Azure](../api-management/index.yml).

Gestion des API s’intègre directement dans Service Fabric, ce qui vous permet de publier des API avec un ensemble complet de règles de routage vers vos services Service Fabric principaux.  Vous pouvez sécuriser l’accès aux services backend, empêcher des attaques de déni de service à l’aide de la limitation, ou vérifier les clés d’API, les jetons JSON Web Token, les certificats et autres informations d’identification. Pour plus d’informations, lisez la [Vue d’ensemble d’Azure Service Fabric avec Gestion des API](service-fabric-api-management-overview.md).

## <a name="manage-application-secrets"></a>Gérer des secrets d’application
Les secrets peuvent être des informations sensibles quelconques, notamment des chaînes de connexion de stockage, des mots de passe ou d’autres valeurs qui ne doivent pas être traitées en texte brut. Cet article utilise Azure Key Vault pour gérer les clés et les secrets. Toutefois, *l’utilisation de* secrets dans une application cloud est indépendante de la plateforme et permet ainsi un déploiement d’applications dans un cluster hébergé à n’importe quel endroit.

La méthode recommandée pour gérer les paramètres de configuration de service consiste à utiliser des [packages de configuration de service][config-package]. Les versions des packages de configuration sont gérées et peuvent être mises à jour par le biais de mises à niveau propagées gérées avec validation de l’intégrité et la restauration automatique. Cette option est préférable à une configuration globale, car elle réduit le risque d’interruption de service globale. Les secrets chiffrés ne font pas exception. Service Fabric dispose de fonctionnalités intégrées pour chiffrer et déchiffrer des valeurs dans un fichier de package de configuration Settings.xml à l’aide du cryptage de certificat.

Le diagramme suivant illustre le flux de base pour la gestion des secrets dans une application Service Fabric :

![vue d’ensemble de la gestion des secrets][overview]

Ce flux comprend quatre étapes principales :

1. Obtenez un certificat de chiffrement de données.
2. Installez le certificat dans votre cluster.
3. Chiffrez les valeurs secrètes lors du déploiement d’une application avec le certificat et injectez-les dans le fichier de configuration Settings.xml d’un service.
4. Lisez les valeurs chiffrées dans Settings.xml en les déchiffrant avec le même certificat de chiffrement. 

[Azure Key Vault][key-vault-get-started] est utilisé ici comme un emplacement de stockage sécurisé pour des certificats et comme un moyen d’installer des certificats sur des clusters Service Fabric dans Azure. Si vous ne déployez pas dans Azure, il est inutile d’utiliser le coffre de clés pour gérer les secrets dans des applications Service Fabric.

Pour obtenir un exemple, consultez [Gérer des secrets d’application](service-fabric-application-secret-management.md).

## <a name="secure-the-hosting-environment"></a>Sécuriser l’environnement d’hébergement
Avec Azure Service Fabric, vous pouvez sécuriser les applications en cours d’exécution dans le cluster sous différents comptes d’utilisateur. Service Fabric permet également de sécuriser les ressources utilisées par des applications au moment du déploiement sous les comptes utilisateurs, par exemple les fichiers, les annuaires et les certificats. Ainsi, les applications en cours d’exécution sont plus sécurisées, même dans un environnement hébergé partagé.

Le manifeste de l’application déclare les principaux de sécurité (utilisateurs et groupes) nécessaires pour exécuter les services et sécuriser les ressources.  Ces principaux de sécurité sont référencés dans les stratégies, comme les stratégies d’exécution, de liaison des points de terminaison, de partage des packages ou d’accès de sécurité.  Les stratégies sont ensuite appliquées à des ressources de service situées dans la section **ServiceManifestImport** du manifeste d’application.

Lorsque vous déclarez des principaux, vous pouvez également définir et créer des groupes d’utilisateurs de manière à pouvoir ajouter à chaque groupe un ou plusieurs utilisateurs qui seront gérés ensemble. Cela est utile lorsqu’il existe plusieurs utilisateurs pour des points d’entrée de service différents et qu’ils doivent disposer de certains privilèges courants disponibles au niveau du groupe.

Par défaut, les applications Service Fabric s’exécutent sous le compte qui exécute le processus Fabric.exe. Service Fabric permet également d’exécuter des applications sous un compte utilisateur ou système local spécifié dans le manifeste de l’application. Pour plus d’informations, consultez [Exécuter un service à l’aide d’un compte d’utilisateur local ou d’un compte de système local](service-fabric-application-runas-security.md).  Vous pouvez également [exécuter un script de démarrage du service en tant qu’utilisateur ou compte système local](service-fabric-run-script-at-service-startup.md).

Lorsque vous exécutez Service Fabric sur un cluster autonome Windows, vous pouvez exécuter un service avec des [comptes de domaine Active Directory](service-fabric-run-service-as-ad-user-or-group.md) ou des [comptes de service gérés de groupe](service-fabric-run-service-as-gmsa.md).

## <a name="secure-containers"></a>Sécuriser les conteneurs
Service Fabric fournit un mécanisme pour les services à l’intérieur d’un conteneur pour accéder à un certificat installé sur les nœuds dans un cluster Windows ou Linux (version 5.7 ou version ultérieure). Ce certificat PFX peut être utilisé pour authentifier l’application ou le service afin de sécuriser la communication avec d’autres services. Pour plus d’informations, consultez [Importer un certificat dans un conteneur](service-fabric-securing-containers.md).

En outre,Service Fabric prend également en charge les gMSA (comptes de service géré de groupe) pour les conteneurs Windows. Pour plus d’informations, consultez [Configurer le gMSA pour les conteneurs Windows](service-fabric-setup-gmsa-for-windows-containers.md).

## <a name="secure-service-communication"></a>Sécuriser les communications des services
Dans Service Fabric, un service s’exécute quelque part dans un cluster Service Fabric, généralement réparti sur plusieurs machines virtuelles. Service Fabric fournit plusieurs options pour sécuriser les communications de votre service.

Vous pouvez activer les points de terminaison HTTPS dans vos services web [ASP.NET Core ou Java](service-fabric-service-manifest-resources.md#example-specifying-an-https-endpoint-for-your-service).

Vous pouvez établir une connexion sécurisée entre le proxy inverse et les services, permettant ainsi un canal sécurisé de bout en bout. La connexion à des services sécurisés est prise en charge uniquement quand le proxy inverse est configuré pour écouter sur le protocole HTTPS. Pour plus d’informations sur la configuration du proxy inverse, lisez [Proxy inverse dans Azure Service Fabric](service-fabric-reverseproxy.md).  L’article [Se connecter à un service sécurisé](service-fabric-reverseproxy-configure-secure-communication.md) explique comment établir une connexion sécurisée entre le proxy inverse et les services.

L’infrastructure d’application Reliable Services fournit quelques piles et outils de communication prédéfinis afin d’améliorer la sécurité. Découvrez comment améliorer la sécurité lorsque vous utilisez la communication à distance du service (en [C#](service-fabric-reliable-services-secure-communication.md) ou [Java](service-fabric-reliable-services-secure-communication-java.md)) ou à l’aide de [WCF](service-fabric-reliable-services-secure-communication-wcf.md).

## <a name="encrypt-application-data-at-rest"></a>Chiffrer les données d’application au repos
Chaque [type de nœud](service-fabric-cluster-nodetypes.md) d’un cluster Service Fabric exécuté dans Azure s’appuie sur un [groupe de machines virtuelles identiques](../virtual-machine-scale-sets/overview.md). Avec un modèle Azure Resource Manager, vous pouvez associer des disques de données aux groupes identiques constituant le cluster Service Fabric.  Si vos services enregistrent des données sur un disque de données attaché, vous pouvez [chiffrer ces disques de données](../virtual-machine-scale-sets/disk-encryption-powershell.md) pour protéger vos données d’application.

<!--TO DO: Enable BitLocker on Windows standalone clusters?
TO DO: Encrypt disks on Linux clusters?-->


<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Étapes suivantes
* [Exécuter un script d’installation au démarrage du service](service-fabric-run-script-at-service-startup.md)
* [Spécifier des ressources dans un manifeste de service](service-fabric-service-manifest-resources.md)
* [Déployer une application](service-fabric-deploy-remove-applications.md)
* [Découvrir la sécurité des clusters](service-fabric-cluster-security.md)

<!-- Links -->
[key-vault-get-started]:../key-vault/general/overview.md
[config-package]: service-fabric-application-and-service-manifests.md
[service-fabric-cluster-creation-via-arm]: service-fabric-cluster-creation-via-arm.md

<!-- Images -->
[overview]:./media/service-fabric-application-and-service-security/overview.png

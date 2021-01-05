---
title: Guide de prise en main pour les développeurs sur Azure | Microsoft Docs
description: Cet article fournit des informations essentielles aux développeurs qui souhaitent commencer à utiliser la plateforme Microsoft Azure pour leurs besoins de développement.
services: ''
cloud: ''
documentationcenter: ''
author: ggailey777
manager: erikre
ms.assetid: ''
ms.service: azure
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 11/18/2019
ms.author: glenga
ms.openlocfilehash: 8694c403b14234a70b0a67f9f4defb7817ba3ae3
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005382"
---
# <a name="get-started-guide-for-azure-developers"></a>Guide de prise en main pour les développeurs Azure

## <a name="what-is-azure"></a>Qu’est-ce qu’Azure ?

Azure est une plateforme cloud complète qui peut héberger vos applications existantes et simplifier le développement de nouvelles applications. Azure peut même améliorer les applications locales. Azure intègre les services cloud dont vous avez besoin pour développer, tester, déployer et gérer vos applications, tout en tirant parti de l’efficacité du cloud computing.

En hébergeant vos applications dans Azure, vous pouvez commencer à petite échelle et faire évoluer aisément vos applications à mesure que la demande client augmente. Azure offre également la fiabilité requise pour les applications à haute disponibilité, en permettant même un basculement entre les différentes régions. Le [portail Azure](https://portal.azure.com) vous permet de gérer facilement tous vos services Azure. Vous pouvez également gérer vos services par programmation en utilisant des API et des modèles spécifiques aux services.

Ce guide est une introduction à la plateforme Azure pour les développeurs d’applications. Il fournit les conseils et les directions dont vous avez besoin pour commencer à élaborer de nouvelles applications dans Azure et à migrer vos applications existantes vers Azure.

## <a name="where-do-i-start"></a>Par où commencer ?

Avec tous les services qu’offre Azure, il peut être ardu d’identifier les services dont vous avez besoin pour prendre en charge votre architecture de solution. Cette section met en évidence les services Azure que les développeurs utilisent couramment. Pour obtenir la liste de tous les services Azure, consultez la [documentation d’Azure](../../index.yml).

Tout d’abord, vous devez choisir la manière dont vous souhaitez héberger votre application dans Azure. Devez-vous gérer toute votre infrastructure en tant que machine virtuelle ? Pouvez-vous utiliser les installations de gestion de plateforme fournies par Azure ? Vous avez peut-être besoin uniquement d’un framework sans serveur pour héberger l’exécution de code ?

Votre application a besoin d’un stockage sur le cloud et Azure propose plusieurs options pour cela. Vous pouvez bénéficier de l’authentification en entreprise d’Azure. Il existe également des outils de développement et de surveillance basés sur le cloud, et la plupart des services d’hébergement offrent une intégration DevOps.

À présent, passons en revue certains services spécifiques que nous vous recommandons d’examiner pour vos applications.

### <a name="application-hosting"></a>Hébergement d’applications

Azure propose plusieurs offres de calcul basées sur le cloud pour exécuter votre application sans avoir à vous soucier des détails d’infrastructure. Vous pouvez facilement effectuer un scale-up ou un scale-out de vos ressources à mesure que l’utilisation de vos applications augmente.

Azure propose des services qui prennent en charge vos besoins d’hébergement et de développement d’applications. Azure fournit une infrastructure IaaS (Infrastructure as a Service) pour vous donner un contrôle total sur l’hébergement de vos applications. Les offres PaaS (Platform as a Service) d’Azure fournissent des services complètement managés nécessaires pour dynamiser vos applications. Azure inclut même un hébergement serverless dans lequel il vous suffit d’écrire votre code.

![Options d’hébergement d’applications Azure](./media/azure-developer-guide/azure-developer-hosting-options.png)


#### <a name="azure-app-service"></a>Azure App Service

Si vous voulez bénéficier du moyen le plus rapide pour publier vos projets web, optez pour Azure App Service. App Service facilite l’extension de vos applications web pour prendre en charge vos clients mobiles et publier des API REST aisément consommées. Cette plateforme offre une authentification à l’aide des réseaux sociaux, une mise à l’échelle automatique basée sur le trafic, des tests en production et des déploiements continus et basés sur des conteneurs.

Vous pouvez créer des applications web, des back-ends d’applications mobiles et des applications API.

Comme ces trois types d’application partagent le runtime App Service, vous pouvez héberger un site web, prendre en charge les clients mobiles et exposer vos API dans Azure, tout cela depuis un même projet ou une même solution. Pour plus d’informations sur App Service, consultez l’article [Vue d’ensemble de Web Apps](../../app-service/overview.md).

App Service a été conçu en tenant compte de DevOps. Il prend en charge divers outils de déploiement d’intégration continue et de publication. Ces outils incluent des webhooks GitHub, Jenkins, Azure DevOps, TeamCity et bien d’autres.

Vous pouvez migrer vos applications existantes vers App Service en utilisant l’[outil de migration en ligne](https://appmigration.microsoft.com/).

> **Quand les utiliser** : utilisez App Service quand vous migrez des applications web existantes vers Azure et quand vous avez besoin d’une plateforme d’hébergement complètement managée pour vos applications web. Vous pouvez également utiliser App Service lorsque vous avez besoin de prendre en charge les clients mobiles ou d’exposer des API REST avec votre application.
>
> **Prise en main** : App Service permet de créer et de déployer facilement votre première [application web](../../app-service/quickstart-dotnetcore.md), [application mobile](/previous-versions/azure/app-service-mobile/app-service-mobile-ios-get-started) ou [application API](../../app-service/app-service-web-tutorial-rest-api.md).
>
> **Essayez-le** : App Service vous permet de configurer une application de courte durée dans le but d’essayer la plateforme sans avoir à vous inscrire pour obtenir un compte Azure. Essayez la plateforme et [créez votre application Azure App Service](https://tryappservice.azure.com/).

#### <a name="azure-virtual-machines"></a>Machines virtuelles Azure

En tant que fournisseur d’infrastructure IaaS (Infrastructure as a Service), Azure vous permet de déployer ou de migrer votre application vers des machines virtuelles Windows ou Linux. Avec Réseau virtuel Azure, le service Machines virtuelles Azure prend en charge le déploiement des machines virtuelles Windows ou Linux sur Azure. Avec les machines virtuelles, vous avez un contrôle total sur la configuration des machines. Lorsque vous utilisez des machines virtuelles, vous êtes responsable de toutes les tâches d’installation, de configuration et de maintenance du logiciel serveur, ainsi que des correctifs du système d’exploitation.

En raison du niveau de contrôle que vous avez avec les machines virtuelles, vous pouvez exécuter un large éventail de charges de travail serveur sur Azure, qui ne sont pas adaptées à un modèle PaaS. Ces charges de travail incluent les serveurs de base de données, Windows Server Active Directory et Microsoft SharePoint. Pour plus d’informations, consultez la documentation relative aux machines virtuelles pour [Linux](../../virtual-machines/linux/index.yml) ou [Windows](../../virtual-machines/windows/index.yml).

> **Quand les utiliser** : utilisez Machines virtuelles lorsque vous voulez un contrôle total sur votre infrastructure d’applications ou pour migrer des charges de travail d’application locales vers Azure sans avoir à apporter de modifications.
>
> **Prise en main** : créez une [machine virtuelle Linux](../../virtual-machines/linux/quick-create-portal.md) ou une [machine virtuelle Windows](../../virtual-machines/windows/quick-create-portal.md) à partir du portail Azure.

#### <a name="azure-functions-serverless"></a>Azure Functions (sans serveur)

Plutôt que de vous soucier de la création et de la gestion d’une application entière ou de l’infrastructure pour exécuter votre code, et si vous pouviez écrire votre code et l’exécuter en réponse à des événements ou à une planification ?  [Azure Functions](../../azure-functions/functions-overview.md) est une offre de type « sans serveur » qui vous permet d’écrire juste le code nécessaire. Avec Functions, vous pouvez déclencher l’exécution du code avec des requêtes HTTP, des webhooks, des événements de service cloud ou selon une planification. Vous pouvez écrire du code dans le langage de développement de votre choix, tel que C\#, F\#, Node.js, Python ou PHP. Avec une facturation basée sur la consommation, vous payez uniquement pour la durée pendant laquelle votre code s’exécute et Azure effectue la mise à l’échelle au besoin.

> **Quand les utiliser** : utilisez Azure Functions lorsque vous avez du code qui est déclenché par d’autres services Azure, par des événements basés sur le web ou selon une planification. Vous pouvez également utiliser Functions quand vous n’avez pas besoin de la surcharge d’un projet hébergé complet ou quand vous voulez payer uniquement la durée d’exécution de votre code. Pour en savoir plus, consultez [Présentation d’Azure Functions](../../azure-functions/functions-overview.md).
>
> **Prise en main** : suivez le didacticiel de démarrage rapide de Functions pour [créer votre première fonction](../../azure-functions/functions-create-first-azure-function.md) à partir du portail.
>
> **Essayez-le** : Azure Functions vous permet d’exécuter votre code sans avoir à vous inscrire pour créer un compte Azure. Essayez-le dès à présent et [créez votre première fonction Azure](https://tryappservice.azure.com/).

#### <a name="azure-service-fabric"></a>Azure Service Fabric

Azure Service Fabric est une plateforme de systèmes distribués. Celle-ci facilite la création, la mise en package, le déploiement et la gestion de microservices scalables et fiables. Elle fournit également des fonctionnalités de gestion complète des applications telles que :

* Approvisionnement
* Déploiement en cours
* Surveillance
* Mise à niveau/application de correctifs
* En cours de suppression

Les applications s’exécutent sur un pool partagé de machines. Vous pouvez commencer à petite échelle puis évoluer pour prendre en charge des centaines ou des milliers de machines, selon les besoins.

Service Fabric prend en charge WebAPI avec Open Web Interface for .NET (OWIN) et ASP.NET Core. Il fournit des kits SDK pour la création de services sur Linux en langage .NET Core ou Java. Pour plus d’informations sur Service Fabric, consultez la [documentation Service Fabric](../../service-fabric/index.yml).

> **Quand l’utiliser** : Service Fabric est un bon choix quand vous créez une application ou réécrivez une application existante pour utiliser une architecture de microservices. Utilisez Service Fabric quand vous avez besoin d’un meilleur contrôle de l’infrastructure sous-jacente ou d’un accès direct à cette dernière.
>
> **Mise en route** [Créez votre première application Azure Service Fabric](../../service-fabric/service-fabric-tutorial-create-dotnet-app.md).

### <a name="enhance-your-applications-with-azure-services"></a>Améliorer vos applications avec les services Azure

Avec l’hébergement d’applications, Azure fournit des offres de service qui peuvent améliorer les fonctionnalités. Azure peut également améliorer le développement et la maintenance de vos applications, à la fois dans le cloud et localement.

#### <a name="hosted-storage-and-data-access"></a>Stockage hébergé et accès aux données

La plupart des applications doivent stocker des données. Par conséquent, quelle que soit la manière dont vous décidez d’héberger votre application dans Azure, prenez en considération un ou plusieurs des services de données et de stockage suivants.

- **Azure Cosmos DB** : Service de base de données multimodèle distribué à l’échelle mondiale. Cette base de données vous permet de mettre à l’échelle de façon élastique le débit et le stockage pour le nombre de régions géographiques de votre choix avec un contrat SLA complet.

  > **Quand l’utiliser** : quand votre application a besoin de document, d’une table ou de bases de données de graphique, y compris des bases de données MongoDB, avec plusieurs modèles de cohérence bien définis.
  >
  > **Prise en main** : [créez une application web Azure Cosmos DB](../../cosmos-db/create-sql-api-dotnet.md). Si vous êtes développeur MongoDB, consultez la rubrique sur la [création d’une application web MongoDB avec Azure Cosmos DB](../../cosmos-db/create-mongodb-dotnet.md).

- **Stockage Azure** : offre un stockage durable, hautement disponible pour les objets blob, les files d’attente, les fichiers et d’autres types de données non relationnelles. Le service Stockage fournit la base de stockage utilisée pour les machines virtuelles.

  > **Quand les utiliser** : lorsque votre application stocke des données non relationnelles, telles que des paires clé-valeur (tables), des objets blob, des partages de fichiers ou des messages (files d’attente).
  >
  > **Prise en main** : choisissez parmi les types de stockages suivants : [blobs](../../storage/blobs/storage-quickstart-blobs-dotnet.md), [tables](../../cosmos-db/tutorial-develop-table-dotnet.md), [files d’attente](../../storage/queues/storage-dotnet-how-to-use-queues.md) et [fichiers](../../storage/files/storage-dotnet-how-to-use-files.md).

- **Azure SQL Database** : version basée sur Azure du moteur Microsoft SQL Server pour le stockage des données tabulaires relationnelles dans le cloud. SQL Database offre des performances prévisibles et une scalabilité sans interruption de service. Il assure aussi la continuité des activités et la protection des données.

  > **Quand les utiliser** : lorsque votre application nécessite un stockage de données avec intégrité référentielle, une prise en charge transactionnelle et une prise en charge des requêtes TSQL.
  >
  > **Prise en main** : [créez une base de données dans Azure SQL Database en quelques minutes à l’aide du portail Azure](../../azure-sql/database/single-database-create-quickstart.md).


Vous pouvez utiliser [Azure Data Factory](../../data-factory/introduction.md) pour déplacer des données locales existantes vers Azure. Si vous n’êtes pas prêt à déplacer des données vers le cloud, les [connexions hybrides](../../app-service/app-service-hybrid-connections.md) dans Azure App Service vous permettent de connecter votre application hébergée par App Service aux ressources locales. Vous pouvez également vous connecter aux services de données et de stockage Azure à partir de vos applications locales.

#### <a name="docker-support"></a>Prise en charge de Docker

Les conteneurs Docker, forme de virtualisation du système d’exploitation, vous permettent de déployer des applications de manière plus efficace et prévisible. Une application en conteneur fonctionne en production de la même façon que sur vos systèmes de développement et de test. Vous pouvez gérer les conteneurs à l’aide des outils Docker standard. Pour déployer et gérer des applications basées sur un conteneur dans Azure, vous pouvez utiliser vos compétences et des outils open source connus.

Azure permet d’utiliser des conteneurs dans vos applications de plusieurs façons différentes.


- **Azure Kubernetes Service** : permet la création, la configuration et la gestion d’un cluster de machines virtuelles préconfigurées pour exécuter des applications en conteneur. Pour plus d’informations sur Azure Kubernetes Service, consultez [Présentation d’Azure Kubernetes Service](../../aks/intro-kubernetes.md).

  > **Quand les utiliser** : lorsque vous devez créer des environnements scalables prêts pour la production qui exploitent d’autres outils de planification et de gestion, ou lorsque vous déployez un cluster Docker Swarm.
  >
  > **Prise en main** : [Déployer un cluster Kubernetes Service](../../aks/tutorial-kubernetes-deploy-cluster.md).

- **Docker Machine** : vous permet d’installer et de gérer un moteur Docker sur les ordinateurs hôtes virtuels en utilisant les commandes docker-machine.

  >**Quand les utiliser** : lorsque vous avez besoin de créer rapidement un prototype d’une application en créant un hôte Docker individuel.

- **Image Docker personnalisée pour App Service** : vous permet d’utiliser des conteneurs Docker à partir d’un registre de conteneurs ou d’un conteneur de clients lorsque vous déployez une application web sur Linux.

  > **Quand les utiliser** : lors du déploiement d’une application web sur Linux sur une image Docker.
  >
  > **Prise en main** : [utilisez une image Docker personnalisée pour App Service sur Linux](../../app-service/quickstart-custom-container.md?pivots=platform-linux%253fpivots%253dplatform-linux).

### <a name="authentication"></a>Authentification

Il est essentiel non seulement de savoir qui utilise vos applications, mais aussi d’empêcher tout accès non autorisé à vos ressources. Azure offre plusieurs moyens d’authentifier vos clients d’application.

- **Azure Active Directory (Azure AD)**  : service Microsoft multilocataire basé sur le cloud qui gère les identités et les accès. Vous pouvez ajouter l’authentification unique (SSO) à vos applications en les intégrant à Azure AD. Vous pouvez accéder aux propriétés d’annuaire en utilisant l’API Graph Azure AD directement ou l’API Microsoft Graph. Vous pouvez exploiter la prise en charge d’Azure AD pour l’infrastructure d’autorisation OAuth2.0 et la connexion OpenID en utilisant les points de terminaison HTTP/REST natifs et les bibliothèques d’authentification Azure AD multiplateformes.

  > **Quand les utiliser** : lorsque vous souhaitez fournir une expérience SSO, travailler avec des données basées sur Graph ou authentifier les utilisateurs de domaine.
  >
  > **Prise en main** : pour en savoir plus, consultez le [Guide du développeur Azure Active Directory](../../active-directory/develop/v2-overview.md).

- **Authentification App Service** : lorsque vous choisissez App Service pour héberger votre application, vous bénéficiez également de la prise en charge de l’authentification intégrée pour Azure AD, ainsi que des fournisseurs d’identité sociale, notamment Facebook, Google, Microsoft et Twitter.

  > **Quand les utiliser** : lorsque vous souhaitez activer l’authentification dans une application App Service en utilisant Azure AD, des fournisseurs d’identité sociale, ou les deux.
  >
  > **Prise en main** : pour en savoir plus sur l’authentification dans App Service, consultez [Authentification et autorisation dans Azure App Service](../../app-service/overview-authentication-authorization.md).

Pour en savoir plus sur les bonnes pratiques de sécurité dans Azure, consultez [Bonnes pratiques et modèle de sécurité dans Azure](../../security/fundamentals/best-practices-and-patterns.md).

### <a name="monitoring"></a>Surveillance

Lorsque votre application est en cours d’exécution dans Azure, vous devez surveiller les performances, déceler les problèmes éventuels et voir comment les clients utilisent votre application. Azure fournit plusieurs options de surveillance.

-   **Application Insights** : service d’analyse extensible hébergé par Azure qui s’intègre à Visual Studio pour surveiller vos applications web en direct. Il vous fournit les données dont vous avez besoin pour améliorer les performances et la convivialité de vos applications, et ce de façon continue. Cette amélioration se produit que vous hébergiez vos applications sur Azure ou non.

    >**Prise en main** : suivez le [didacticiel d’Application Insights](../../azure-monitor/app/app-insights-overview.md).

-   **Azure Monitor** : service qui vous permet de visualiser, d’interroger, de router, d’archiver et de traiter les métriques et les journaux d’activité qui sont générés par votre infrastructure et vos ressources Azure. Monitor fournit les vues de données que vous voyez dans le portail Azure et constitue une source unique pour la surveillance des ressources Azure.

    >**Prise en main** : [Prise en main d’Azure Monitor](../../azure-monitor/overview.md).

### <a name="devops-integration"></a>Intégration DevOps

Qu’il s’agisse de mettre en service des machines virtuelles ou de publier vos applications web avec une intégration continue, Azure s’intègre à la plupart des outils DevOps répandus. Vous pouvez travailler avec les outils dont vous disposez déjà et optimiser votre expérience avec la prise en charge d’outils, notamment :

* Jenkins
* GitHub
* Puppet
* Chef
* TeamCity
* Ansible
* Azure DevOps

> **Prise en main** : pour voir les options DevOps d’une application App Service, consultez [Déploiement continu vers Azure App Service](../../app-service/deploy-continuous-deployment.md).
>
> **Essayez-le :** [essayez plusieurs intégrations DevOps](https://azure.microsoft.com/try/devops/).


## <a name="azure-regions"></a>Régions Azure

Azure est une plateforme cloud mondiale, disponible dans de nombreuses régions du monde. Lorsque vous provisionnez un service, une application ou une machine virtuelle dans Azure, vous êtes invité à sélectionner une région. Cette région représente un centre de données spécifique où votre application s’exécute et vos données sont stockées. Ces régions correspondent à des endroits spécifiques, qui sont publiés dans la page [Régions Azure](https://azure.microsoft.com/regions/).

### <a name="choose-the-best-region-for-your-application-and-data"></a>Choisir la meilleure région pour votre application et vos données

L’un des avantages de l’utilisation d’Azure est de pouvoir déployer des applications dans différents centres de données du monde. La région que vous choisissez peut affecter les performances de votre application. Par exemple, il est préférable de choisir une région proche de la plupart de vos clients afin de réduire la latence des demandes réseau. Vous pouvez également sélectionner votre région pour répondre aux exigences légales de distribution de votre application dans certains pays/régions. Il est toujours recommandé de stocker les données d’application dans le même centre de données ou dans un centre de données aussi proche que possible du centre de données qui héberge votre application.

### <a name="multi-region-apps"></a>Applications multirégions

Même si cela est peu probable, il est possible qu’un centre de données soit mis intégralement hors connexion en raison d’une catastrophe naturelle ou d’une panne Internet. Pour garantir une disponibilité optimale, il est recommandé d’héberger les applications métier critiques dans plusieurs centres de données. L’utilisation de plusieurs régions peut également réduire la latence pour les utilisateurs internationaux et fournir une flexibilité supplémentaire lors des mises à jour d’applications.

Certains services, tels que Machines virtuelles et App Services, utilisent [Azure Traffic Manager](../../traffic-manager/traffic-manager-overview.md) pour obtenir une prise en charge multirégion avec basculement d’une région à une autre, ce qui permet de prendre en charge les applications d’entreprise à haute disponibilité. Pour obtenir un exemple, consultez [Architecture de référence Azure : Exécuter une application web dans plusieurs régions](/azure/architecture/reference-architectures/app-service-web-app/multi-region).

>**Quand les utiliser** : lorsque vous avez des applications d’entreprise à haute disponibilité qui bénéficient du basculement et de la réplication.

## <a name="how-do-i-manage-my-applications-and-projects"></a>Comment gérer applications et projets ?

Azure fournit un ensemble complet d’expériences qui vous permettent de créer et de gérer vos ressources, applications et projets Azure, que ce soit par programmation ou dans le [portail Azure](https://portal.azure.com/).

### <a name="command-line-interfaces-and-powershell"></a>Interfaces de ligne de commande et PowerShell

Azure propose deux moyens de gérer vos applications et services à partir de la ligne de commande. Vous pouvez utiliser des outils tels que Bash, Terminal, l’invite de commandes ou votre outil en ligne de commande préféré. En règle générale, vous pouvez effectuer les mêmes tâches à partir de la ligne de commande et dans le portail Azure, telles que la création et la configuration de machines virtuelles, de réseaux virtuels, d’applications web et d’autres services.

-   [Interface de ligne de commande Azure](/cli/azure/install-azure-cli) : permet de vous connecter à un abonnement Azure et de programmer diverses tâches concernant les ressources Azure à partir de la ligne de commande.

-   [Azure PowerShell](/powershell/azure/): fournit un ensemble de modules avec des applets de commande qui vous permettent de gérer les ressources Azure en utilisant Windows PowerShell.

### <a name="azure-portal"></a>Portail Azure

Le [Portail Azure](https://portal.azure.com) est une application web. Vous pouvez utiliser le portail Azure pour créer, gérer et supprimer des services et des ressources Azure. Il inclut :

* Un tableau de bord configurable
* Outils de gestion des ressources Azure
* Accès aux paramètres d’abonnement et aux informations de facturation. Pour plus d’informations, consultez la [présentation du portail Azure](https://azure.microsoft.com/features/azure-portal/).

### <a name="rest-apis"></a>API REST

Azure repose sur un ensemble d’API REST qui prennent en charge l’interface utilisateur du portail Azure. La plupart de ces API REST sont également prises en charge pour vous permettre de configurer et de gérer par programmation vos ressources et applications Azure à partir de n’importe quel appareil connecté à Internet. Pour accéder à la documentation complète des API REST, consultez [Informations de référence sur les API REST Azure](/rest/api/).

### <a name="apis"></a>API

Avec les API REST, de nombreux services Azure vous permettent de gérer par programmation les ressources de vos applications en utilisant les SDK Azure propres à la plateforme, notamment les SDK pour les plateformes de développement suivantes :

-   [.NET](/dotnet/api/)
-   [Node.JS](/azure/developer/javascript/)
-   [Java](/java/azure)
-   [PHP](https://github.com/Azure/azure-sdk-for-php/blob/master/README.md)
-   [Python](/azure/python/)
-   [Ruby](https://github.com/Azure/azure-sdk-for-ruby/blob/master/README.md)
-   [Go](/azure/go)

Des services tels que [Mobile Apps](/previous-versions/azure/app-service-mobile/app-service-mobile-dotnet-how-to-use-client-library) et [Azure Media Services](../../media-services/previous/media-services-dotnet-how-to-use.md) fournissent des kits SDK côté client pour vous permettre d’accéder aux services à partir d’applications clientes web et mobiles.

### <a name="azure-resource-manager"></a>Azure Resource Manager

L’exécution de votre application sur Azure implique probablement l’utilisation de plusieurs services Azure. Ces services suivent le même cycle de vie et peuvent être considérés comme une unité logique. Par exemple, une application web peut utiliser Web Apps, SQL Database, Stockage, le cache Azure pour Redis et les services Azure Content Delivery Network. [Azure Resource Manager](../../azure-resource-manager/management/overview.md) vous permet d’utiliser les ressources de votre application sous la forme d’un groupe. Vous pouvez déployer, mettre à jour et supprimer toutes les ressources dans le cadre d’une opération unique et coordonnée.

Avec le regroupement et la gestion logiques des ressources connexes, Azure Resource Manager inclut des fonctionnalités de déploiement qui vous permettent de personnaliser le déploiement et la configuration des ressources connexes. Par exemple, vous pouvez utiliser Resource Manager pour déployer et configurer une application. Cette application peut se composer de plusieurs machines virtuelles, d’un équilibreur de charge et d’une base de données dans Azure SQL Database, qui forment une unité unique.

Vous développez ces déploiements à l’aide d’un modèle Azure Resource Manager, qui est un document au format JSON. Les modèles vous permettent de définir un déploiement et de gérer vos applications à l’aide de modèles déclaratifs, plutôt que de scripts. Vos modèles peuvent fonctionner pour différents environnements, par exemple des environnements de test, de préproduction et de production. Par exemple, vous pouvez utiliser des modèles pour ajouter un bouton à un dépôt GitHub qui déploie le code figurant dans le dépôt d’un seul clic dans un ensemble de services Azure.

> **Quand les utiliser** : utilisez les modèles Resource Manager lorsque vous souhaitez un déploiement basé sur un modèle pour votre application que vous pouvez gérer par programmation à l’aide des API REST, de l’interface de ligne de commande Azure et d’Azure PowerShell.
>
> **Prise en main** : pour commencer à utiliser des modèles, consultez [Créer des modèles Azure Resource Manager](../../azure-resource-manager/templates/template-syntax.md).

## <a name="understanding-accounts-subscriptions-and-billing"></a>Présentation des comptes, des abonnements et de la facturation

En tant que développeurs, nous aimons plonger directement dans le code et essayer de faire fonctionner nos applications aussi rapidement que possible. Nous souhaitons vous encourager à commencer à utiliser Azure de la façon la plus simple possible. Dans ce but, Azure propose un [essai gratuit](https://azure.microsoft.com/free/). Certains services ont même une fonctionnalité « Essai gratuit », comme [Azure App Service](https://tryappservice.azure.com/), qui ne vous demande pas de créer un compte. Aussi agréable que cela puisse être de plonger dans le code et le déploiement de votre application dans Azure, il est également important de prendre le temps de comprendre le fonctionnement d’Azure, notamment du point de vue des comptes d’utilisateur, des abonnements et de la facturation.

### <a name="what-is-an-azure-account"></a>Qu’est-ce qu’un compte Azure ?

Pour créer ou utiliser un abonnement Azure, vous devez disposer d’un compte Azure. Un compte Azure est simplement une identité dans Azure AD ou dans un répertoire, telle qu’une organisation professionnelle ou un établissement scolaire, qui est approuvée par Azure AD. Si vous n’appartenez pas à une organisation comme celle-ci, vous pouvez toujours créer un abonnement à l’aide de votre compte Microsoft, qui est approuvé par Azure AD. Pour en savoir plus sur l’intégration de Windows Server Active Directory sur site avec Azure AD, consultez [Intégration de vos identités locales avec Azure Active Directory](../../active-directory/hybrid/whatis-hybrid-identity.md).

Chaque abonnement Azure dispose d’une relation d’approbation avec une instance Azure AD. Cela signifie qu'il approuve ce répertoire pour authentifier les utilisateurs, les services et les appareils. Plusieurs abonnements peuvent approuver le même annuaire, mais un abonnement n’approuve qu’un seul annuaire. Pour en savoir plus, consultez [Association des abonnements Azure avec Azure Active Directory](../../active-directory/fundamentals/active-directory-how-subscriptions-associated-directory.md).

Outre des identités de compte Azure individuelles, également appelées *utilisateurs*, vous pouvez définir des *groupes* dans Azure AD. La création de groupes d’utilisateurs est un bon moyen de gérer l’accès aux ressources dans un abonnement à l’aide du contrôle d’accès en fonction du rôle (RBAC). Pour découvrir comment créer des groupes, consultez [Créer un groupe dans la préversion d’Azure Active Directory](../../active-directory/fundamentals/active-directory-groups-create-azure-portal.md). Vous pouvez également créer et gérer des groupes en [utilisant PowerShell](../../active-directory/enterprise-users/groups-settings-v2-cmdlets.md).

### <a name="manage-your-subscriptions"></a>Gérer vos abonnements

Un abonnement est un regroupement logique de services Azure, associé à un compte Azure. Un seul compte Azure peut contenir plusieurs abonnements. La facturation des services Azure est effectuée par abonnement. Pour obtenir la liste des offres d’abonnement disponibles par type, consultez [Détails de l’offre Microsoft Azure](https://azure.microsoft.com/support/legal/offer-details/). Les abonnements Azure sont dotés d’un Administrateur de compte ayant un contrôle total sur l’abonnement. Ils ont également un administrateur de services fédérés qui a le contrôle sur tous les services figurant dans l’abonnement. Pour plus d’informations sur les administrateurs d’abonnements classiques, consultez [Ajouter ou changer des administrateurs d’abonnements Azure](../../cost-management-billing/manage/add-change-subscription-administrator.md). Les comptes individuels peuvent bénéficier d’un contrôle détaillé des ressources Azure en utilisant [le contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md).

#### <a name="resource-groups"></a>Groupes de ressources

Lorsque vous configurez de nouveaux services Azure, vous le faites dans le cadre d’un abonnement donné. Les services Azure individuels, également appelés « ressources », sont créés dans le contexte d’un groupe de ressources. Les groupes de ressources facilitent le déploiement et la gestion des ressources de votre application. Un groupe de ressources doit contenir toutes les ressources de votre application que vous souhaitez utiliser en tant qu’unité. Vous pouvez déplacer les ressources entre les groupes de ressources et même vers d’autres abonnements. Pour en savoir plus sur le déplacement des ressources, consultez [Déplacer des ressources vers un nouveau groupe de ressources ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md).

Azure Resource Explorer est un excellent outil de visualisation des ressources que vous avez déjà créées dans votre abonnement. Pour en savoir plus, consultez [Affichage et modification des ressources à l’aide d’Azure Resource Explorer](/rest/api/).

#### <a name="grant-access-to-resources"></a>Accorder l’accès à des ressources

Lorsque vous autorisez l’accès à des ressources Azure, il est toujours recommandé de fournir aux utilisateurs les privilèges minimaux nécessaires pour effectuer une tâche donnée.

- **Contrôle d’accès en fonction du rôle Azure (Azure RBAC)**  : dans Azure, vous pouvez accorder l’accès aux comptes d’utilisateur (principaux) dans une étendue spécifiée : abonnement, groupe de ressources ou ressources individuelles. Azure RBAC vous permet de déployer des ressources dans un groupe de ressources et d’accorder des autorisations à un utilisateur ou à un groupe spécifique. Il permet également de limiter l’accès aux seules ressources qui appartiennent au groupe de ressources cible. Vous pouvez également accorder l’accès à une ressource unique, telle qu’une machine virtuelle ou un réseau virtuel. Pour accorder l’accès, vous assignez un rôle à l’utilisateur, au groupe ou au principal de service. Il existe de nombreux rôles prédéfinis et vous pouvez également définir vos propres rôles personnalisés. Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?](../../role-based-access-control/overview.md)

  > **Quand les utiliser** : quand vous avez besoin de gérer avec précision l’accès des utilisateurs et des groupes ou que vous devez faire d’un utilisateur le propriétaire d’un abonnement.
  >
  > **Prise en main** : Pour en savoir plus, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

- **ID d'objet du principal de service** : en plus de fournir l’accès aux groupes et principaux d’utilisateur, vous pouvez accorder le même accès à un principal de service.

  > **Quand les utiliser** : lorsque vous gérez par programmation les ressources Azure ou accordez l’accès pour des applications. Pour plus d’informations, consultez [Créer une application Active Directory et un principal de service](../../active-directory/develop/howto-create-service-principal-portal.md).

#### <a name="tags"></a>Balises

Azure Resource Manager vous permet d’assigner des balises personnalisées à des ressources individuelles. Les balises, qui sont des paires clé-valeur, peuvent être utiles lorsque vous devez organiser les ressources à des fins de facturation ou de surveillance. Les balises vous permettent d’effectuer le suivi des ressources entre plusieurs groupes de ressources. Pour attribuer des balises, procédez comme suit :

* Sur le portail
* Dans le modèle Azure Resource Manager
* Utilisation de l’API REST
* Utilisation de l’interface de ligne de commande Azure (CLI)
* Utilisation de PowerShell

Vous pouvez assigner plusieurs balises à chaque ressource. Pour en savoir plus, consultez [Organisation des ressources Azure à l’aide de balises](../../azure-resource-manager/management/tag-resources.md).

### <a name="billing"></a>Facturation

Dans le cadre de la transition d’une informatique locale aux services hébergés sur le cloud, le suivi et l’estimation de l’utilisation des services et des coûts associés sont des préoccupations importantes. Il est essentiel d’évaluer le coût d’exploitation de nouvelles ressources sur une base mensuelle. Vous pouvez également projeter à quoi ressemblera la facturation pour un mois donné en fonction des dépenses actuelles.

#### <a name="get-resource-usage-data"></a>Obtenir les données d’utilisation des ressources

Azure fournit un ensemble d’API REST de facturation qui donnent accès à la consommation des ressources et aux informations de métadonnées pour les abonnements Azure. Ces API de facturation vous permettent de mieux prévoir et gérer les coûts liés à Azure. Vous pouvez effectuer le suivi et l’analyse des dépenses par incréments horaires et créer des alertes de dépenses. Vous pouvez également prédire la facturation à venir en fonction des tendances d’utilisation actuelles.

>**Prise en main** : pour en savoir plus sur l’utilisation des API de facturation, consultez [Vue d’ensemble des API de facturation Azure et RateCard](../../cost-management-billing/manage/usage-rate-card-overview.md).

#### <a name="predict-future-costs"></a>Prédire les coûts futurs

Bien qu’il soit difficile d’estimer les coûts à l’avance, Azure propose des outils qui peuvent vous aider. Il dispose d’une [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/) pour estimer le coût des ressources déployées. Vous pouvez également utiliser les ressources de facturation dans le portail et les API REST de facturation pour estimer les coûts futurs en fonction de votre consommation actuelle.

>**Prise en main** : consultez [Vue d’ensemble des API de facturation Azure et RateCard](../../cost-management-billing/manage/usage-rate-card-overview.md).
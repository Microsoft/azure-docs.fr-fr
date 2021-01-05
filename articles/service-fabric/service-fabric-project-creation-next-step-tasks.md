---
title: Étapes suivantes de la création de projet Service Fabric
description: Découvrez-en plus sur le projet d’application que vous venez de créer dans Visual Studio.  Apprenez à créer des services à l’aide de didacticiels et approfondissez vos connaissances sur le développement de services pour Service Fabric.
ms.topic: conceptual
ms.date: 12/07/2017
ms.openlocfilehash: 01a69016e0c299fba0365fab5332b572fd2ca87a
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92314474"
---
# <a name="your-service-fabric-application-and-next-steps"></a>Votre application Service Fabric et étapes suivantes
Votre application Azure Service Fabric a été créée. Cet article détaille quelques-uns des didacticiels à essayer, la composition de votre projet, certaines informations supplémentaires pouvant vous intéresser ainsi que les prochaines étapes possibles.

## <a name="get-started-with-tutorials-walk-throughs-and-samples"></a>Bien démarrer avec les didacticiels, les procédures détaillées et les exemples
Vous êtes prêt à commencer ?  

Suivez le didacticiel de l’application .NET. Découvrez comment [générer une application](service-fabric-tutorial-create-dotnet-app.md) avec un frontend ASP.NET Core et un backend avec état, comment [déployer l’application](service-fabric-tutorial-deploy-app-to-party-cluster.md) sur un cluster, [configurer l’intégration et le déploiement continus](service-fabric-tutorial-deploy-app-with-cicd-vsts.md) et [définir la surveillance et les diagnostics](service-fabric-tutorial-monitoring-aspnet.md).

Vous pouvez aussi essayer l’une des procédures pas à pas suivantes pour créer votre premier...
- [Service C# Reliable Services sur Windows](service-fabric-reliable-services-quick-start.md) 
- [Service C# Reliable Actors sur Windows](service-fabric-reliable-actors-get-started.md) 
- [Service exécutable invité sur Windows](quickstart-guest-app.md) 
- [Application de conteneur Windows](service-fabric-get-started-containers.md) 

Sinon, nos [exemples d’applications](/samples/browse/?products=azure) peuvent également vous être utiles.

## <a name="have-questions-or-feedback--need-to-report-an-issue"></a>Avez-vous des questions ou des commentaires ?  Vous souhaitez signaler un problème ?
Consultez les [questions courantes](service-fabric-common-questions.md) et trouvez des réponses se rapportant aux possibilités de Service Fabric et à son utilisation.

Les [options du Support](service-fabric-support.md) dressent une liste de forums sur StackOverflow et MSDN où poser des questions, ainsi qu’une liste d’options permettant de signaler des problèmes, obtenir une assistance et envoyer des commentaires sur les produits.

## <a name="the-application-project"></a>Le projet d'application
Chaque nouvelle application inclut un projet d'application. Il peut y avoir un ou deux projets supplémentaires en fonction du type de service choisi.

Le projet d'application se compose des éléments suivants :

* Un ensemble de références aux services qui composent votre application.
* Trois profils de publication (local à 1 nœud, local à 5 nœuds et cloud) que vous pouvez utiliser pour préserver les préférences d’utilisation d’environnements différents, comme les préférences liées à un point de terminaison de cluster et si vous souhaitez exécuter les déploiements de mise à niveau par défaut.
* Trois fichiers de paramètres d’application (du même type que ci-dessus) que vous pouvez utiliser pour gérer les configurations d’application spécifiques à l’environnement, comme le nombre de partitions à créer pour un service. Découvrez comment [configurer votre application pour plusieurs environnements](service-fabric-manage-multiple-environment-app-configuration.md).
* Un script de déploiement que vous pouvez utiliser pour déployer votre application à partir de la ligne de commande ou dans le cadre d’un pipeline de déploiement et d’intégration continue automatisée. Apprenez-en davantage sur le [déploiement d’applications en utilisant PowerShell](service-fabric-deploy-remove-applications.md).
* Le manifeste d'application qui décrit l'application. Le manifeste se trouve dans le dossier ApplicationPackageRoot. Apprenez-en davantage sur les [manifestes d’application et de service](service-fabric-application-model.md).



## <a name="learn-more-about-the-programming-models"></a>En savoir plus sur les modèles de programmation
Service Fabric offre plusieurs méthodes pour écrire et gérer vos services.  Voici des informations conceptuelles et d’ordre générale sur [Reliable Services avec ou sans état](service-fabric-reliable-services-introduction.md), [Reliable Actors](service-fabric-reliable-actors-introduction.md), [les conteneurs](service-fabric-containers-overview.md), [les exécutables invités](service-fabric-guest-executables-introduction.md) et [les services ASP.NET Core avec et sans état](service-fabric-reliable-services-communication-aspnetcore.md).

## <a name="learn-about-service-communication"></a>En savoir plus sur la communication des services
Une application Service Fabric est composée de différents services effectuant chacun une tâche spéciale. Ces services peuvent communiquer entre eux, et des applications clientes à l’extérieur du cluster peuvent également se connecter à ces services pour communiquer avec eux. Découvrez comment [configurer la communication avec et entre vos services](service-fabric-connect-and-communicate-with-services.md) dans Service Fabric. 

## <a name="learn-about-configuring-application-security"></a>En savoir plus sur la configuration de la sécurité des applications
Vous pouvez sécuriser les applications en cours d’exécution dans le cluster sous différents comptes d’utilisateur. Service Fabric permet également de sécuriser les ressources utilisées par des applications au moment du déploiement sous les comptes utilisateurs, par exemple les fichiers, les annuaires et les certificats. Ainsi, les applications en cours d’exécution sont plus sécurisées, même dans un environnement hébergé partagé.  Découvrez comment [configurer les stratégies de sécurité de votre application](service-fabric-application-runas-security.md).

Votre application peut contenir des informations sensibles, notamment des chaînes de connexion de stockage, des mots de passe ou d’autres valeurs ne devant pas être gérées en texte brut. Découvrez comment [gérer les clés secrètes dans votre application](service-fabric-application-secret-management.md).

## <a name="learn-about-the-application-lifecycle"></a>En savoir plus sur le cycle de vie d’une application
Comme pour les autres plateformes, une application Service Fabric passe généralement par les phases suivantes : la conception, le développement, le test, le déploiement, la mise à niveau, la maintenance et la suppression. [Cet article](service-fabric-application-lifecycle.md) fournit une vue d’ensemble des API et aborde la façon dont elles sont utilisées par les différents rôles pendant les phases du cycle de vie d’une application Service Fabric.

## <a name="next-steps"></a>Étapes suivantes
- [Créer un cluster Windows dans Azure](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
- Visualiser votre cluster, entre autres les applications déployées et la disposition physique, en utilisant [Service Fabric Explorer](service-fabric-visualizing-your-cluster.md)
- [Contrôler la version et mettre à niveau vos services](service-fabric-application-upgrade-tutorial.md)
---
title: Vue d’ensemble du service d’analyse des erreurs
description: Cet article décrit le service d’analyse des erreurs dans Service Fabric qui permet de produire des erreurs et d’exécuter des scénarios de test sur vos services.
ms.topic: conceptual
ms.date: 06/15/2017
ms.openlocfilehash: 4e879b0b39d58f115b4661d47cf2ae564ba14e5d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91531375"
---
# <a name="introduction-to-the-fault-analysis-service"></a>Introduction au service d’analyse des erreurs
Le service d’analyse des erreurs est conçu pour tester les services qui s’appuient sur Microsoft Azure Service Fabric. Avec le service d’analyse des erreurs, vous pouvez provoquer des erreurs significatives et exécuter des scénarios de test complets sur vos applications. Ces erreurs et scénarios exercent et valident les nombreux états et transitions qu’un service connaît tout au long de sa durée de vie, le tout de manière contrôlée, sécurisée et cohérente.

Les actions sont les erreurs isolées introduites dans un service pour le tester. Un développeur de services peut les utiliser en tant blocs de constructions afin d’écrire des scénarios compliqués. Par exemple :

* Redémarrez un nœud pour simuler différentes situations dans lesquelles une machine ou une machine virtuelle est redémarrée.
* Déplacez un réplica de votre service avec état pour simuler l’équilibrage de charge, le basculement ou la mise à niveau de l’application.
* Appelez la perte de quorum sur votre service avec état afin de créer une situation où les opérations d’écriture sont interrompues en raison d’un nombre insuffisant de réplicas de sauvegarde ou secondaire pour l’acceptation des nouvelles données.
* Appelez la perte des données sur un service avec état pour créer une situation où l’intégralité de l’état en mémoire est effacé.

Les scénarios sont des opérations complexes composées d’une ou plusieurs actions. Le service d’analyse des erreurs fournit deux scénarios complets intégrés :

* Scénario de chaos
* Scénario de basculement

## <a name="testing-as-a-service"></a>Tests en tant que service
Le service d’analyse des erreurs est un service du système Service Fabric qui démarre automatiquement avec un cluster Service Fabric. Ce service agit comme un hôte pour l’injection d’erreurs, l’exécution de scénarios de test et l’analyse d’intégrité. 

![Service d’analyse des erreurs][0]

Quand une action d’erreur ou un scénario de test est déclenché, une commande est envoyée au service d’analyse des erreurs pour exécuter l’action d’erreur ou le scénario de test. Le service d’analyse des erreurs est avec état de sorte qu’il peut exécuter les scénarios et les erreurs de façon fiable et valider les résultats. Par exemple, un scénario de test à longue durée d'exécution peut être exécuté de manière fiable par le service d’analyse des erreurs. Et, étant donné que les tests sont exécutés à l'intérieur du cluster, le service peut examiner l'état du cluster et de vos services pour fournir des informations plus détaillées sur les échecs.

## <a name="testing-distributed-systems"></a>Test des systèmes distribués
Service Fabric simplifie considérablement les tâches d’écriture et de gestion des applications évolutives distribuées. De même, le service d’analyse des erreurs facilite le test d’une application distribuée. Trois problématiques liées au test doivent être résolues :

1. Simulation/Génération de défaillances réalistes : Service Fabric permet notamment aux applications distribuées de récupérer de diverses défaillances. Toutefois, pour évaluer la capacité de l’application à récupérer de la sorte, nous devons recourir à un mécanisme de simulation/de génération de ces défaillances réalistes au sein d’un environnement de test contrôlé.
1. La capacité à générer des défaillances liées : les défaillances de base du système, comme les défaillances réseau et les défaillances de machines, sont faciles à générer de manière isolée. Les interactions entre les défaillances isolées peuvent entraîner un nombre incalculable de scénarios de dysfonctionnement. La génération de ces contextes revêt une importance toute particulière.
1. Expérience unifiée au sein de divers niveaux de développement et de déploiement : de nombreux systèmes d’injection d’erreurs peuvent générer différents types de défaillances. Toutefois, cette expérience unifiée est pauvre en transposant les scénarios de test de développement au sein des grands environnements de test, puis en production.

Il existe de nombreux mécanismes permettant de résoudre ces problèmes, mais il manque un élément : un système qui joue le même rôle avec les garanties requises, de l’environnement de test des développeurs aux clusters de production. Le service d’analyse des erreurs permet aux développeurs d’applications de se concentrer sur le test de leur logique métier. Le service d’analyse des erreurs fournit toutes les capacités requises pour tester l’interaction du service avec le système distribué sous-jacent.

### <a name="simulatinggenerating-real-world-failure-scenarios"></a>Simulation/Génération de scénarios de défaillances réalistes
Pour éprouver la solidité d’un système distribué contre les défaillances, nous recourons à un mécanisme de génération d’erreurs. En théorie, il semble facile de générer une défaillance comme un arrêt de nœud. Néanmoins, cette opération soulève des problématiques de cohérence identiques à celles que Service Fabric tente de résoudre. Par exemple, si vous voulez arrêter un nœud, le flux de travail suivant est requis :

1. À partir du client, émettez une requête d’arrêt du nœud.
1. Envoyez la requête au nœud approprié.
   
    a. Si le nœud est introuvable, elle doit être mise en échec.
   
    b. Si le nœud est trouvé, elle doit être renvoyée uniquement si le nœud est arrêté.

Pour vérifier la défaillance dans le cadre d’un test, celui-ci doit savoir que quand cette défaillance est provoquée, elle se produit réellement. Avec Service Fabric, vous êtes assuré que le nœud va être arrêté ou est déjà arrêté, au moment où la commande l’atteint. Dans les deux cas, le test peut examiner correctement l’état et procéder de manière appropriée à la validation. Un système implémenté en dehors de Service Fabric pour exécuter le même ensemble de défaillances pourrait rencontrer une multitude de problèmes de réseau, matériels et logiciels, qui l’empêcheraient de fournir les garanties précédentes. Si les problèmes évoqués précédemment sont identifiés, Service Fabric reconfigure l’état du cluster afin de les contourner. En conséquence, le service d’analyse des erreurs vous procure les garanties adéquates.

### <a name="generating-required-events-and-scenarios"></a>Génération des événements et scénarios requis
Tandis que la simulation des défaillances réalistes peut s’avérer difficile, la génération des défaillances liées est encore plus compliquée. Par exemple, une perte de données se produit dans un service persistant avec état quand les événements suivants se produisent :

1. Seul un quorum d’écriture est traité par la réplication. L’ensemble des réplicas secondaires sont derrière le réplica primaire.
1. L’arrêt des réplicas provoque l’arrêt du quorum d’écriture (en raison de l’arrêt du package de code ou du nœud).
1. Le quorum d’écriture ne peut pas réapparaître en raison de la perte des données associées aux réplicas (causée par la corruption des disques ou la réinitialisation de la machine).

Ces défaillances liées se produisent vraiment, mais pas aussi souvent que les défaillances isolées. Il est primordial de pouvoir tester ces scénarios avant qu’ils ne se produisent en environnement de production. La capacité à simuler ces scénarios avec des charges de travail de production dans des circonstances contrôlées (en plein milieu de la journée avec tous les ingénieurs sur le pont) est encore plus importante. De telles circonstances sont bien plus efficaces qu’un premier test en production à 2h00 du matin.

### <a name="unified-experience-across-different-environments"></a>Expérience unifiée au sein d’environnements différents
Traditionnellement, l’idée est de développer trois différents ensembles d’expériences, un pour l’environnement de développement, un pour les tests et un autre pour la production. Le modèle se présentait ainsi :

1. Dans l’environnement de développement, produisez des transitions d’état prenant en charge des tests d’unités de méthodes individuelles.
1. Dans l’environnement de test, produisez des erreurs dans le cadre de tests de bout en bout illustrant différents scénarios de défaillances.
1. Conservez l’environnement de production intact pour empêcher tout risque de défaillance non naturelle et garantir une réponse humaine extrêmement rapide aux défaillances.

Dans Service Fabric, par le biais du service d’analyse des erreurs, nous proposons de bouleverser cette définition en appliquant une méthodologie unique de l’environnement de développement à la production. Il existe deux moyens de parvenir à cet objectif :

1. Pour provoquer des défaillances contrôlées, utilisez les API de service d’analyse des erreurs d’un environnement à boîtier unique sur l’ensemble du processus, jusqu’aux clusters de production.
1. Pour instaurer dans le cluster une atmosphère provoquant l’introduction automatique de défaillance, recourez au service d’analyse des erreurs afin de générer des erreurs automatiques. En configurant le taux de défaillances, vous êtes à même de tester un même service dans divers environnements.

Avec Service Fabric, la mise à l’échelle des défaillances ne serait pas la même au sein des différents environnements. En revanche, les mécanismes réels seraient identiques. Cela permet de diffuser un code plus rapide vers le pipeline de déploiement et de tester les services sous des charges de travail réelles.

## <a name="using-the-fault-analysis-service"></a>Utilisation du service d’analyse des erreurs
**C#**

Les fonctionnalités du service d’analyse des erreurs sont dans l'espace de noms System.Fabric du package NuGet de Microsoft.ServiceFabric. Pour pouvoir utiliser les fonctionnalités du service d’analyse des erreurs, incorporez le package NuGet en tant que référence dans votre projet.

**PowerShell**

Pour utiliser PowerShell, vous devez installer le Kit de développement logiciel (SDK) Service Fabric. Une fois qu’il est installé, le module PowerShell Service Fabric est chargé automatiquement, et rendu disponible.

## <a name="next-steps"></a>Étapes suivantes
Pour créer des services vraiment à l’échelle du cloud, il est primordial de vérifier, avant et après le déploiement, que ces services peuvent résister à des défaillances réalistes. Dans le monde des services actuel, vous avez tout intérêt à démontrer une capacité à innover et à déployer rapidement du code en production. C’est précisément cela que le service d’analyse des erreurs permet de faire aux développeurs de services.

Commencez à tester vos applications et services à l’aide des [scénarios de test](service-fabric-testability-scenarios.md) intégrés, ou créez vos propres scénarios de test à l’aide des [actions d’erreur](service-fabric-testability-actions.md) fournies par le service d’analyse des erreurs.

<!--Image references-->
[0]: ./media/service-fabric-testability-overview/faultanalysisservice.png

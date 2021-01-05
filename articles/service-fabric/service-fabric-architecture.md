---
title: Architecture d’Azure Service Fabric
description: Cet article explique l’architecture de Service Fabric, une plateforme de systèmes distribués qui permet de créer des applications évolutives, fiables et faciles à gérer pour le cloud.
services: service-fabric
ms.topic: conceptual
ms.date: 01/09/2020
ms.openlocfilehash: 972700dded1841994de9252b4aa4bbc8eaefeaf8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76024712"
---
# <a name="service-fabric-architecture"></a>Architecture de Service Fabric

Service Fabric est constitué de sous-systèmes en couches. Ces sous-systèmes permettent d’écrire des applications qui présentent les caractéristiques suivantes :

* Haute disponibilité
* Évolutif
* Facilité de gestion
* Testable

Le diagramme suivant montre les principaux sous-systèmes de Service Fabric.

![Diagramme de l’architecture de Service Fabric](media/service-fabric-architecture/service-fabric-architecture.png)

Dans un système distribué, la capacité à communiquer en toute sécurité entre les nœuds d’un cluster est essentielle. À la base de la pile se trouve le sous-système de transport, qui fournit une communication sécurisée entre les nœuds. Au-dessus du sous-système de transport se trouve le sous-système de fédération, qui associe les différents nœuds en une seule entité (nommée cluster) afin que Service Fabric puisse détecter les défaillances, effectuer le choix de l’instance responsable et fournir un routage cohérent. Le sous-système de fiabilité, placé au-dessus du sous-système de fédération, assure la fiabilité des services Service Fabric par le biais de mécanismes tels que la réplication, la gestion des ressources et le basculement. Le sous-système de fédération est situé sous le sous-système d’hébergement et d’activation, qui gère le cycle de vie d’une application sur un nœud unique. Le sous-système de gestion gère le cycle de vie des applications et des services. Le sous-système de testabilité permet aux développeurs d’applications de tester les services au moyen d’erreurs simulées avant et après le déploiement des applications et des services dans les environnements de production. Service Fabric permet de résoudre les emplacements des services par le biais de son sous-système de communication. Les modèles de programmation d’application exposés aux développeurs sont placés au-dessus de ces sous-systèmes avec le modèle d’application pour activer les outils.

## <a name="transport-subsystem"></a>Sous-système de transport

Le sous-système de transport implémente un canal de communication de datagramme point à point. Ce canal est utilisé pour les communications au sein des clusters Service Fabric et pour les communications entre le cluster Service Fabric et les clients. Il prend en charge des modèles de communication à sens unique et de type demande-réponse, servant de base pour l’implémentation de la diffusion et de la multidiffusion dans la couche de fédération. Le sous-système de transport sécurise les communications à l'aide de certificats X509 ou de la sécurité Windows. Ce sous-système est utilisé en interne par Service Fabric et n’est pas directement accessible aux développeurs pour la programmation d’applications.

## <a name="federation-subsystem"></a>Sous-système de fédération

Pour pouvoir analyser un ensemble de nœuds dans un système distribué, vous devez disposer d’une vue cohérente du système. Le sous-système de fédération utilise les primitives de communication fournies par le sous-système de transport et assemble les différents nœuds en un seul cluster unifié qu’il peut analyser. Il fournit les primitives des systèmes distribués requises par les autres sous-systèmes : détection des défaillances, choix de l'instance responsable et routage cohérent. Le sous-système de fédération repose sur des tables de hachage distribuées avec un espace de jeton de 128 bits. Il crée une topologie en anneau sur ces nœuds, chaque nœud dans l’anneau se voyant allouer un sous-ensemble de l’espace de jeton pour la propriété. Pour la détection des défaillances, la couche utilise un mécanisme de bail basé sur des pulsations et un arbitrage. Le sous-système de fédération garantit également via des protocoles complexes de jointure et de départ qu'un jeton ne possède qu'un seul propriétaire à tout moment. Cela fournit des garanties de choix d’instance responsable et de routage cohérent.

## <a name="reliability-subsystem"></a>Sous-système de fiabilité

Le sous-système de fiabilité fournit le mécanisme permettant de rendre l’état d’un service de Service Fabric hautement disponible en utilisant le *réplicateur*, *Failover Manager* et l’*équilibreur de ressources*.

* Le réplicateur garantit que les changements de l'état dans le réplica principal de service seront automatiquement répliqués sur les réplicas secondaires, maintenant ainsi la cohérence entre le réplica principal et les réplicas secondaires dans un jeu de réplicas de service. Le réplicateur est responsable de la gestion du quorum entre les réplicas dans le jeu de réplicas. Il interagit avec l’unité de basculement pour obtenir la liste des opérations à répliquer et l’agent de reconfiguration fournit cette liste avec la configuration du jeu de réplicas. Cette configuration indique sur quels réplicas les opérations doivent être répliquées. Service Fabric fournit un réplicateur par défaut appelé Fabric Replicator que l’API du modèle de programmation peut utiliser pour rendre l’état d’un service hautement disponible et fiable.
* Failover Manager garantit que, quand des nœuds sont ajoutés ou supprimés dans un cluster, la charge est automatiquement redistribuée entre les nœuds disponibles. Si un nœud du cluster échoue, le cluster reconfigure automatiquement les réplicas de service pour maintenir la disponibilité.
* Resource Manager place les réplicas de service dans les domaines de défaillance dans le cluster et garantit que toutes les unités de basculement sont opérationnelles. Resource Manager équilibre également les ressources des services dans le pool partagé sous-jacent de nœuds du cluster pour assurer une répartition uniforme de charge optimale.

## <a name="management-subsystem"></a>Sous-système de gestion

Le sous-système de gestion fournit une gestion de bout en bout du cycle de vie des applications et des services. Les applets de commande PowerShell et les API d’administration vous permettent d’approvisionner, de déployer, de corriger et de mettre à niveau des applications, ainsi que de supprimer leurs privilèges d’accès, sans aucune perte de disponibilité. Le sous-système de gestion effectue cela via les services répertoriés ci-dessous.

* **Gestionnaire de cluster** : il s'agit du service principal qui interagit avec le composant de fiabilité Failover Manager pour placer les applications sur les différents nœuds en fonction des contraintes de placement de service. Resource Manager dans le sous-système de basculement garantit que les contraintes ne sont jamais interrompues. Le gestionnaire de cluster gère le cycle de vie des applications de leur approvisionnement à la suppression de leurs privilèges d’accès. Il s’intègre au gestionnaire d’intégrité pour garantir que la disponibilité des applications n’est pas perdue du point de vue de l’intégrité sémantique pendant les mises à niveau.
* **Gestionnaire d'intégrité** : ce service permet la surveillance de l’intégrité des applications, des services et des entités du cluster. Les entités du cluster (telles que les nœuds, les partitions de service et les réplicas) peuvent signaler des informations d’intégrité, lesquelles sont alors regroupées dans le magasin d’intégrité centralisé. Ces informations d’intégrité fournissent un instantané de l’intégrité globale à un instant précis des services et des nœuds distribués entre plusieurs nœuds du cluster, ce qui vous permet de prendre les mesures correctives nécessaires. Les API de requête d’intégrité vous permettent d’exécuter des requêtes sur les événements d’intégrité signalés au sous-système de contrôle d’intégrité. Les API de requête d’intégrité retournent les données brutes d’intégrité stockées dans le magasin d’intégrité ou les données d’intégrité interprétées et agrégées pour une entité spécifique du cluster.
* **Magasin d’images** : Ce service assure le stockage et la distribution des fichiers binaires d’application. Ce service fournit un magasin de fichiers distribués simple dans lequel et à partir duquel les applications sont téléchargées.

## <a name="hosting-subsystem"></a>Sous-système d'hébergement

Le gestionnaire de cluster indique au sous-système d'hébergement (en cours d'exécution sur chaque nœud) quels services il doit gérer pour un nœud particulier. Le sous-système d’hébergement gère ensuite le cycle de vie de l’application sur ce nœud. Il interagit avec les composants de fiabilité et d'intégrité pour garantir que les réplicas sont placés correctement et fonctionnent correctement.

## <a name="communication-subsystem"></a>Sous-système de communication

Ce sous-système fournit une messagerie fiable au sein du cluster et la détection de service via le service d'affectation de noms. Le service d’affectation de noms résout les noms des services dans un emplacement du cluster et permet aux utilisateurs de gérer les propriétés et les noms des services. Grâce au service d'affectation de noms, les clients peuvent communiquer en toute sécurité avec n'importe quel nœud du cluster pour résoudre un nom de service et récupérer les métadonnées d'un service. En utilisant une simple API client d’affectation de noms, les utilisateurs de Service Fabric peuvent développer des services et des clients capables de résoudre l’emplacement réseau actuel malgré le dynamisme des nœuds ou le redimensionnement du cluster.

## <a name="testability-subsystem"></a>Sous-système de testabilité

La testabilité est une suite d'outils conçus spécifiquement pour tester les services créés sur la plateforme Service Fabric. Les outils permettent à un développeur de provoquer facilement des erreurs significatives et d’exécuter des scénarios de test pour explorer et valider les nombreux états et transitions qu’un service connaît tout au long de sa durée de vie, de manière contrôlée et sécurisée. La testabilité fournit également un mécanisme permettant d’effectuer des tests de durée plus longue par itération dans le cadre de diverses défaillances possibles, sans aucune perte de disponibilité. Vous bénéficiez ainsi d’un environnement de test en production.

---
title: Vue d’ensemble de Linkerd
description: Obtenir une vue d’ensemble de Linkerd
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 3181be62a14ec1b3450bd181172b5323ca176427
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77593765"
---
# <a name="linkerd"></a>Linkerd

## <a name="overview"></a>Vue d’ensemble

[Linkerd][linkerd] est un maillage de services (service mesh) facile à utiliser et léger.

## <a name="architecture"></a>Architecture

Linkerd fournit un plan de données composé de sidecars proxys ultralégers et spécialisés pour [Linkerd][linkerd-proxy]. Ces proxys intelligents contrôlent l’ensemble du trafic réseau entrant dans vos charges de travail et applications maillées et sortant de celles-ci. Les proxys exposent également des métriques par le biais de points de terminaison de métriques[Prometheus][prometheus].

Le plan de contrôle gère la configuration et les données de télémétrie agrégées au moyen des [composants][linkerd-architecture] suivants :

- **Contrôleur** : fournit une API qui pilote l’interface de ligne de commande et le tableau de bord Linkerd. Fournit la configuration des proxys.

- **Tap** : établit des espions en temps réel sur les demandes et les réponses.

- **Identité** : fournit des fonctionnalités d’identité et de sécurité qui autorisent mTLS entre les services.

- **Web** : fournit le tableau de bord Linkerd.


Le diagramme d’architecture suivant montre l’interaction entre les différents composants du plan de données et du plan de contrôle.


![Vue d’ensemble des composants et de l’architecture de Linkerd.](media/servicemesh/linkerd/about-architecture.png)


## <a name="selection-criteria"></a>Critères de sélection

Il est important de saisir et de prendre en compte les points suivants quand vous évaluez Linkerd pour vos charges de travail :

- [Principes de conception](#design-principles)
- [Capabilities](#capabilities)
- [Scénarios](#scenarios)


### <a name="design-principles"></a>Principes de conception

Les principes de conception suivants [guident][design-principles] le projet Linkerd :

- **Faire simple** : doit être facile à utiliser et à comprendre.

- **Réduire les besoins en ressources** : minimiser les coûts liés au niveau de performance et aux ressources.

- **Marcher, tout simplement** : ne pas endommager les applications existantes et ne pas nécessiter de configuration complexe.


### <a name="capabilities"></a>Fonctionnalités

Linkerd offre les fonctionnalités suivantes :

- **Maillage** : option de débogage intégrée

- **Gestion du trafic** : fractionnement, délais d’expiration, nouvelles tentatives, entrée

- **Sécurité** : chiffrement (mTLS), rotation automatique des certificats toutes les 24 heures

- **Observabilité** : métriques golden, tap, traçage, profils de service et métriques par route, tableau de bord Web avec graphes topologiques, prometheus, grafana


### <a name="scenarios"></a>Scénarios

Nous vous suggérons d’utiliser Linkerd dans les scénarios suivants où il est particulièrement adapté :

- Utilisation simple avec un minimum de conditions requises pour exploiter les fonctionnalités

- Faible latence et faible surcharge, avec pour axes principaux l’observabilité et la gestion simple du trafic


## <a name="next-steps"></a>Étapes suivantes

La documentation suivante décrit comment installer Linkerd sur AKS (Azure Kubernetes Service) :

> [!div class="nextstepaction"]
> [Installer Linkerd dans AKS (Azure Kubernetes Service)][linkerd-install]

Vous pouvez également explorer plus en détail les fonctionnalités et l’architecture de Linkerd :

- [Fonctionnalités de Linkerd][linkerd-features]
- [Architecture de Linkerd][linkerd-architecture]

<!-- LINKS - external -->
[linkerd]: https://linkerd.io/2/overview/
[linkerd-architecture]: https://linkerd.io/2/reference/architecture/
[linkerd-features]: https://linkerd.io/2/features/
[design-principles]: https://linkerd.io/2/design-principles/
[linkerd-proxy]: https://github.com/linkerd/linkerd2-proxy

[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/

<!-- LINKS - internal -->
[linkerd-install]: ./servicemesh-linkerd-install.md
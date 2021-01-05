---
title: Vue d’ensemble d’Istio
description: Obtenir une vue d’ensemble d’Istio
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 8518e30a54c2486abf84cd9ac026cc4dccb3fa84
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77593898"
---
# <a name="istio"></a>Istio

## <a name="overview"></a>Vue d’ensemble

[Istio][istio] est un maillage de services (service mesh) complet, personnalisable et extensible.

## <a name="architecture"></a>Architecture

Istio fournit un plan de données qui se compose de sidecars basés sur [Envoy][envoy-proxy]. Ces proxys intelligents contrôlent l’ensemble du trafic réseau entrant dans vos charges de travail et applications maillées et sortant de celles-ci.

Le plan de contrôle gère la configuration, la stratégie et la télémétrie au moyen des [composants][what-is-istio] suivants :

- **Mixer** : applique des stratégies de contrôle d’accès et d’utilisation. Collecte auprès des proxys des données de télémétrie qui sont envoyées (push) à [Prometheus][prometheus].

- **Pilot** : assure la découverte des services et fournit la configuration/stratégie de gestion du trafic pour les proxys.

- **Citadel** : fournit des fonctionnalités d’identité et de sécurité qui autorisent mTLS entre les services.

- **Galley** : extrait et fournit la configuration aux composants.

Le diagramme d’architecture suivant montre l’interaction entre les différents composants du plan de données et du plan de contrôle.


![Vue d’ensemble des composants et de l’architecture d’Istio.](media/servicemesh/istio/about-architecture.png)


## <a name="selection-criteria"></a>Critères de sélection

Il est important de saisir et de prendre en compte les points suivants quand vous évaluez Istio pour vos charges de travail :

- [Objectifs de conception](#design-goals)
- [Capabilities](#capabilities)
- [Scénarios](#scenarios)


### <a name="design-goals"></a>Objectifs de conception

Les objectifs de conception suivants [guident][design-goals] le projet Istio :

- **Maximiser la transparence** : permet l’adoption avec un minimum de travail pour dégager une véritable valeur ajoutée du système.

- **Extensibilité** : doit pouvoir croître et s’adapter aux besoins en constante évolution.

- **Portabilité** : doit s’exécuter facilement dans différents genres d’environnements (cloud ou locaux).

- **Uniformité des stratégies** : permet de définir des stratégies cohérentes sur de nombreuses ressources.


### <a name="capabilities"></a>Fonctionnalités

Istio offre les fonctionnalités suivantes :

- **Maillage** : passerelles (plusieurs clusters), machines virtuelles (expansion du maillage)

- **Gestion du trafic** : routage, fractionnement, délais d’expiration, disjoncteurs, nouvelles tentatives, entrée, sortie

- **Stratégie** : contrôle d’accès, limite de débit, quota, adaptateurs de stratégie personnalisée

- **Sécurité** : authentification (JWT), autorisation, chiffrement (mTLS), autorité de certification externe (HashiCorp Vault)

- **Observabilité** : métriques golden, mise en miroir, traçage, adaptateurs personnalisés, prometheus, grafana

### <a name="scenarios"></a>Scénarios

Nous vous suggérons d’utiliser Istio dans les scénarios suivants où il est particulièrement adapté :

- Exigences en matière d’extensibilité et ensemble complet de fonctionnalités

- Expansion du maillage pour inclure des charges de travail basées sur des machines virtuelles

- Maillage de services sur plusieurs clusters

## <a name="next-steps"></a>Étapes suivantes

La documentation suivante décrit comment installer Istio sur AKS (Azure Kubernetes Service) :

> [!div class="nextstepaction"]
> [Installer Istio dans AKS (Azure Kubernetes Service)][istio-install]

Vous pouvez également explorer plus en détail les concepts Istio et des modèles de déploiement supplémentaires :

- [Concepts Istio][what-is-istio]
- [Modèles de déploiement Istio][deployment-models]

<!-- LINKS - external -->
[istio]: https://istio.io
[what-is-istio]: https://istio.io/docs/concepts/what-is-istio/
[design-goals]: https://istio.io/docs/concepts/what-is-istio/#design-goals
[deployment-models]: https://istio.io/docs/concepts/deployment-models/

[envoy-proxy]: https://www.envoyproxy.io/
[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/

<!-- LINKS - internal -->
[istio-install]: ./servicemesh-istio-install.md

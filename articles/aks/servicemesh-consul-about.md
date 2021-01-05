---
title: Vue d’ensemble de Consul
description: Obtenir une vue d’ensemble de Consul
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: 77cb6a693c5a73d0498b0acf9bc4ad8cc9f8f32f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83774002"
---
# <a name="consul"></a>Consul

## <a name="overview"></a>Vue d’ensemble

[Consul][consul] est une solution de mise en réseau de services prenant en charge plusieurs centres de données à l’aide de laquelle vous pouvez connecter et sécuriser des services sur plusieurs plateformes d’exécution. [Connect][consul-features] est le composant qui fournit des fonctionnalités de maillage de services.

## <a name="architecture"></a>Architecture

Consul fournit un plan de données qui se compose par défaut de [sidecars][consul-sidecar] basés sur [Envoy][envoy-proxy]. L’architecture proxy de Consul est enfichable. Ces proxys intelligents contrôlent l’ensemble du trafic réseau entrant dans vos charges de travail et applications maillées et sortant de celles-ci.

Le plan de contrôle gère la configuration et la stratégie par le biais des [composants][consul-architecture] suivants :

- **Serveur** : agent Consul s’exécutant en mode serveur qui gère l’état du cluster Consul.

- **Client** : agent Consul s’exécutant en mode client léger. Chaque nœud de calcul doit avoir un agent client en cours d’exécution. Ce client répartit la configuration et la stratégie entre les charges de travail et la configuration de Consul. 

Le diagramme d’architecture suivant montre l’interaction entre les différents composants du plan de données et du plan de contrôle.

![Vue d’ensemble des composants et de l’architecture de Consul.](media/servicemesh/consul/about-architecture.png)


## <a name="selection-criteria"></a>Critères de sélection

Il est important de saisir et de prendre en compte les points suivants quand vous évaluez Consul pour vos charges de travail :

- [Principes de Consul](#consul-principles)
- [Capabilities](#capabilities)
- [Scénarios](#scenarios)


### <a name="consul-principles"></a>Principes de Consul

Les principes suivants [guident][consul-principles] le projet Consul :

- **Approche basée sur des API** : codifiez l’ensemble de la configuration et de la stratégie.

- **Exécuter et se connecter en tout lieu** : connectez des charges de travail sur plusieurs plateformes d’exécution (Kubernetes, machines virtuelles, serverless).

- **Étendre et intégrer** : connectez des charges de travail de manière sécurisée dans toute l’infrastructure.


### <a name="capabilities"></a>Fonctionnalités

Consul offre les fonctionnalités suivantes :

- **Maillage** : passerelle (plusieurs centres de données), machines virtuelles (nœuds hors cluster), synchronisation de service, option de débogage intégrée

- **Proxys** : Envoy, proxy intégré, enfichabilité, proxy L4 disponible pour les charges de travail Windows

- **Gestion du trafic** : routage, fractionnement, résolution

- **Stratégie** : intentions, listes de contrôle d’accès

- **Sécurité** :autorisation, authentification, chiffrement, identités basées sur SPIFFE, autorité de certification externe (Vault), gestion et rotation des certificats

- **Observabilité** : métriques, tableau de bord de l’interface utilisateur, prometheus, grafana


### <a name="scenarios"></a>Scénarios

Nous vous suggérons d’utiliser Consul dans les scénarios suivants où il est particulièrement adapté :

- Extension des charges de travail existantes connectées à Consul

- Exigences en matière de conformité concernant la gestion des certificats

- Maillage de services dans plusieurs clusters

- Charges de travail basées sur des machines virtuelles à inclure dans le maillage de services



## <a name="next-steps"></a>Étapes suivantes

La documentation suivante décrit comment installer Consul sur AKS (Azure Kubernetes Service) :

> [!div class="nextstepaction"]
> [Installer Consul dans AKS (Azure Kubernetes Service)][consul-install]

Vous pouvez également explorer plus en détail les fonctionnalités et l’architecture de Consul :

- [Didacticiels de prise en main de Consul][consul-getting-started]
- [Fonctionnalités de Consul][consul-features]
- [Architecture de Consul][consul-architecture]
- [Consul - Fonctionnement de Connect][consul-how-connect-works]

<!-- LINKS - external -->
[consul]: https://www.consul.io/mesh.html
[consul-features]: https://www.consul.io/docs/connect/index.html
[consul-architecture]: https://www.consul.io/docs/internals/architecture.html
[consul-sidecar]: https://www.consul.io/docs/connect/proxies.html
[consul-how-connect-works]: https://www.consul.io/docs/connect/connect-internals.html
[consul-principles]: https://www.consul.io/
[consul-getting-started]:https://learn.hashicorp.com/consul?track=gs-consul-service-mesh#gs-consul-service-mesh

[envoy-proxy]: https://www.envoyproxy.io/
[grafana]: https://grafana.com/
[prometheus]: https://prometheus.io/

<!-- LINKS - internal -->
[consul-install]: ./servicemesh-consul-install.md

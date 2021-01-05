---
title: À propos des maillages de services
description: Obtenez une vue d’ensemble des maillages de services, de leur architecture et de leurs fonctionnalités, et découvrez les critères de sélection à prendre en compte avant d’en déployer un.
author: paulbouwer
ms.topic: article
ms.date: 10/09/2019
ms.author: pabouwer
ms.openlocfilehash: eca49a3fac1ea0398ebe1d05bde20fbca3c81232
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "77594309"
---
# <a name="about-service-meshes"></a>À propos des maillages de services

Un maillage de services offre des fonctionnalités axées sur les aspects suivants : gestion du trafic, résilience, stratégie, sécurité, identité forte et observabilité des charges de travail. Votre application est dissociée de ces fonctionnalités opérationnelles, lesquelles sont déplacées par le maillage de services hors de la couche Application, au niveau de la couche d’infrastructure.

## <a name="scenarios"></a>Scénarios

Voici quelques-uns des scénarios que vous pouvez mettre en œuvre avec vos charges de travail quand vous utilisez un maillage de services :

- **Chiffrer tout le trafic dans le cluster** : activez le protocole TLS mutuel entre services spécifiés dans le cluster. Ce scénario peut être étendu à l’entrée et à la sortie du périmètre de réseau. Il fournit une option sécurisée par défaut ne nécessitant aucune modification du code et de l’infrastructure de l’application.

- **Lancements de versions canary et par phases** : spécifiez les conditions de routage d’un sous-ensemble de trafic vers un ensemble de nouveaux services dans le cluster. En cas de test réussi d’une version canary, supprimez le routage conditionnel et augmentez progressivement le pourcentage du trafic vers le nouveau service. Au final, tout le trafic doit être dirigé vers le nouveau service.

- **Gestion et manipulation du trafic** : créez une stratégie sur un service qui limite le débit du trafic vers une version d’un service à partir d’une origine spécifique. Ou créez une stratégie qui applique une stratégie de nouvelle tentative aux classes de défaillances entre services spécifiés. Mettez en miroir le trafic en direct vers les nouvelles versions des services pendant une migration ou pour déboguer des problèmes. Injectez des erreurs entre les services dans un environnement de test pour tester la résilience.

- **Observabilité** : obtenez des insights sur le mode de connexion de vos services au trafic qui transite entre eux. Obtenez des métriques, des journaux et des traces pour l’ensemble du trafic dans le cluster et pour les entrées/sorties. Ajoutez des fonctionnalités de traçage distribué à vos applications.

## <a name="architecture"></a>Architecture

Un maillage de services se compose généralement d’un plan de contrôle et du plan de données.

Le **plan de contrôle** comprend un certain nombre de composants qui prennent en charge la gestion du maillage de services. Parmi ceux-ci figure généralement une interface de gestion qui peut être une interface utilisateur ou une API. Vous trouverez aussi des composants gérant les définitions des règles et des stratégies qui précisent la façon dont le maillage de services doit implémenter des fonctionnalités spécifiques. D’autres composants gèrent des aspects de la sécurité tels que l’identité forte et les certificats pour mTLS. Les maillages de services standard intègrent également un composant de métriques ou d’observabilité qui collecte et agrège les métriques et les données de télémétrie des charges de travail.

Le **plan de données** se compose généralement d’un proxy qui est injecté de manière transparente en tant que sidecar dans vos charges de travail. Ce proxy est configuré pour contrôler tout le trafic réseau entrant et sortant du pod qui contient votre charge de travail. Vous pouvez ainsi configurer le proxy pour sécuriser le trafic avec mTLS, router le trafic de manière dynamique, appliquer des stratégies au trafic et collecter des métriques et des informations de traçage. 

![Architecture standard d’un maillage de services](media/servicemesh/typical-architecture.png)

## <a name="capabilities"></a>Fonctionnalités

Si chaque maillage de services tend naturellement à prendre en charge des scénarios spécifiques, vous constaterez toutefois que la plupart d’entre eux implémentent bon nombre, voire la totalité, des fonctionnalités suivantes.

### <a name="traffic-management"></a>Gestion du trafic 

- **Protocole** : couche 7 (http, grpc)
- **Routage dynamique** : conditionnel, pondération, mise en miroir
- **Résilience** : délais d’expiration, nouvelles tentatives, disjoncteurs
- **Stratégie** : contrôle d’accès, limites de débit, quotas
- **Tests** : injection d’erreurs

### <a name="security"></a>Sécurité

- **Chiffrement** : mTLS, gestion des certificats, autorité de certification externe
- **Identité forte** : SPIFFE ou similaire
- **Auth** : authentification, autorisation

### <a name="observability"></a>Observabilité

- **Métriques** : métriques golden, prometheus, grafana
- **Traçage** : traces entre charges de travail
- **Trafic** : cluster, entrée/sortie

### <a name="mesh"></a>Maillage

- **Calcul pris en charge** : Kubernetes, machines virtuelles
- **Plusieurs clusters** : passerelles, fédération

## <a name="selection-criteria"></a>Critères de sélection

Avant de sélectionner un maillage de services, veillez à bien comprendre vos exigences et les raisons pour lesquelles vous souhaitez en installer un. Essayez de vous poser les questions suivantes.

- **Un contrôleur d’entrée est-il suffisant pour mes besoins ?** Parfois, la mise en place d’une fonctionnalité telle qu’un test a/b ou une répartition du trafic à l’entrée est suffisante pour prendre en charge le scénario requis. N’augmentez pas le niveau de complexité de votre environnement si cela n’en vaut pas la peine.

- **Mes charges de travail et mon environnement peuvent-ils tolérer les surcharges supplémentaires ?** Tous les composants supplémentaires requis pour prendre en charge le maillage de services nécessitent davantage de ressources, notamment en termes de processeur et de mémoire. De plus, tous les proxys et les vérifications de stratégie associées ajoutent de la latence à votre trafic. Si vous avez des charges de travail très sensibles à la latence ou êtes dans l’impossibilité de fournir des ressources supplémentaires pour répondre aux besoins des composants du maillage de services, reconsidérez votre décision.

- **Cela ajoute-t-il une complexité supplémentaire inutile ?** Si vous installez un maillage de services dans le but d’obtenir une fonctionnalité non critique pour l’entreprise ou les équipes opérationnelles, déterminez si l’accroissement de la complexité résultant des tâches d’installation, de maintenance et de configuration en vaut la peine.

- **Est-il possible de suivre une approche incrémentielle ?** Vous pouvez adopter certains maillages de services proposant un grand nombre de fonctionnalités de manière plus incrémentielle. Dans ce cas, vous installez uniquement les composants nécessaires à votre réussite. Une fois que vous avez gagné en confiance, explorez d’autres fonctionnalités à mesure que de nouveaux besoins émergent. Résistez à l’envie d’installer *tout* dès le début.

Si, après mûre réflexion, vous décidez que vous avez besoin d’un maillage de services pour offrir les fonctionnalités requises, votre décision suivante est de savoir *quel maillage de services utiliser*.

Passez en revue les domaines suivants et identifiez ceux qui correspondent le mieux à vos besoins. Vous pourrez ainsi trouver la solution la mieux adaptée à votre environnement et à vos charges de travail. La section [Étapes suivantes](#next-steps) vous permet d’en savoir plus sur des maillages de services spécifiques et leurs caractéristiques dans ces domaines.

- **Trafic** : gestion du trafic, stratégie, sécurité, observabilité

- **Activité** : support commercial, fondation (CNCF), licence OSS, gouvernance

- **Opérations** : installation/mises à niveau, besoins en ressources, exigences en matière de performances, intégrations (métriques, données de télémétrie, tableaux de bord, outils, SMI), charges de travail mixtes (pools de nœuds Linux et Windows), calcul (Kubernetes, machines virtuelles), plusieurs clusters

- **Sécurité** : authentification, identité, gestion et rotation des certificats, autorité de certification externe enfichable


## <a name="next-steps"></a>Étapes suivantes

La documentation suivante fournit plus d’informations sur les maillages de services que vous pouvez tester sur AKS (Azure Kubernetes Service) :

> [!div class="nextstepaction"]
> [En savoir plus sur Istio...][istio-about]

> [!div class="nextstepaction"]
> [En savoir plus sur Linkerd...][linkerd-about]

> [!div class="nextstepaction"]
> [En savoir plus sur Consul...][consul-about]

Vous pouvez également explorer SMI (Service Mesh Interface), une interface standard pour les maillages de services sur Kubernetes :

- [SMI (Service Mesh Interface)][smi]


<!-- LINKS - external -->
[smi]: https://smi-spec.io/

<!-- LINKS - internal -->
[istio-about]: ./servicemesh-istio-about.md
[linkerd-about]: ./servicemesh-linkerd-about.md
[consul-about]: ./servicemesh-consul-about.md
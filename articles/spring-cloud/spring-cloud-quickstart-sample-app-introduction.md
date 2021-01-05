---
title: Démarrage rapide - Présentation de l’exemple d’application - Azure Spring Cloud
description: Décrit l’exemple d’application utilisé dans cette série de guides de démarrage rapide concernant le déploiement dans Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: dd36bb18e84ea299195b77286887a3b279f81469
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "90903578"
---
# <a name="introduction-to-the-sample-app"></a>Présentation de l’exemple d’application

::: zone pivot="programming-language-csharp"
Cette série de guides de démarrage rapide utilise un exemple d’application composé de deux microservices afin de montrer comment déployer une application .NET Core Steeltoe dans le service Azure Spring Cloud. Vous utiliserez les fonctionnalités Azure Spring Cloud, telles que la découverte de service, le serveur de configuration, les journaux, les métriques et le suivi distribué.

## <a name="functional-services"></a>Services fonctionnels

L’exemple d’application est composé de deux microservices :

* Le service `planet-weather-provider` retourne du texte relatif au climat en réponse à une requête HTTP qui spécifie le nom de la planète. Par exemple, il peut retourner « très chaud » pour la planète Mercure. Il obtient les données météorologiques du serveur de configuration. Le serveur de configuration obtient les données météorologiques d’un fichier YAML situé dans un dépôt Git, par exemple :

  ```yaml
  MercuryWeather: very warm
  VenusWeather: quite unpleasant
  MarsWeather: very cool
  SaturnWeather: a little bit sandy
  ```

* Le service `solar-system-weather` retourne des données pour quatre planètes en réponse à une requête HTTP. Il obtient les données en envoyant quatre requêtes HTTP à `planet-weather-provider`. Il utilise le service de découverte de serveur Eureka pour appeler `planet-weather-provider`. Il retourne du code JSON, par exemple :

  ```json
  [{
      "Key": "Mercury",
      "Value": "very warm"
  }, {
      "Key": "Venus",
      "Value": "quite unpleasant"
  }, {
      "Key": "Mars",
      "Value": "very cool"
  }, {
      "Key": "Saturn",
      "Value": "a little bit sandy"
  }]
  ```

Le diagramme suivant montre l’architecture de l’exemple d’application.

:::image type="content" source="media/spring-cloud-quickstart-sample-app-introduction/sample-app-diagram.png" alt-text="Diagramme de l’exemple d’application":::

## <a name="code-repository"></a>Dépôt de code

L’exemple d’application se trouve dans le dossier [steeltoe-sample](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample) du dépôt Azure-Samples/Azure-Spring-Cloud-Samples situé sur GitHub.

Les instructions fournies dans les guides de démarrage rapide suivants font référence au code source.

::: zone-end

::: zone pivot="programming-language-java"
Dans ce guide de démarrage rapide, nous utilisons un exemple d’application de finances personnelles appelé PiggyMetrics pour vous montrer comment déployer une application dans le service Azure Spring Cloud. PiggyMetrics permet de voir le modèle d’architecture de microservices, ainsi que la décomposition des services. Vous verrez comment elle est déployée sur Azure avec les puissantes fonctionnalités d’Azure Spring Cloud, y compris la découverte de service, le serveur de configuration, les journaux, les métriques et le suivi distribué.

Pour suivre les exemples de déploiement Azure Spring Cloud, vous avez seulement besoin de l’emplacement du code source, qui est fourni quand c’est nécessaire.

## <a name="functional-services"></a>Services fonctionnels

PiggyMetrics se décompose en trois microservices principaux. Ce sont tous des applications qui peuvent être déployées de façon indépendante et organisées par domaine métier.

* **Service de compte (à déployer)**  : Contient la logique générale et la validation des entrées utilisateur : éléments des revenus/dépenses, économies et paramètres de compte.
* **Service de statistiques (non utilisé dans ce guide de démarrage rapide)**  : Effectue des calculs sur les paramètres statistiques majeurs et capture les séries chronologiques pour chaque compte. Datapoint contient des valeurs normalisées selon une devise de base et une période de temps. Ces données sont utilisées pour effectuer le suivi de la dynamique des flux de trésorerie pendant la durée de vie du compte.
* **Service de notification (non utilisé dans ce guide de démarrage rapide)**  : Stocke les informations de contact et les paramètres de notification des utilisateurs, comme la fréquence des rappels et des sauvegardes. Le Worker planifié collecte les informations nécessaires auprès d’autres services et envoie des messages par e-mail aux clients abonnés.

## <a name="infrastructure-services"></a>Services d’infrastructure

Il existe plusieurs modèles courants dans les systèmes distribués qui facilitent le fonctionnement des services de base. Azure Spring Cloud fournit des outils puissants qui améliorent le comportement des applications Spring Boot pour implémenter ces modèles : 

* **Service de configuration (hébergé par Azure Spring Cloud)**  : Le service de configuration Azure Spring Cloud est un service de configuration centralisé avec scalabilité horizontale pour les systèmes distribués. Il utilise un dépôt enfichable qui prend actuellement en charge le stockage local, Git et Subversion.
* **Service de découverte (hébergé par Azure Spring Cloud)**  : Il permet la détection automatique des emplacements réseau pour les instances de service, qui pourraient avoir des adresses affectées dynamiquement en raison de la mise à l’échelle automatique, des défaillances et des mises à niveau.
* **Service d’authentification (à déployer)** Les responsabilités liées aux autorisations sont complètement déportées sur un serveur distinct, qui accorde des jetons OAuth2 pour les services des ressources de back-end. Le serveur d’authentification traite les autorisations utilisateur et la communication entre machines au sein d’un périmètre.
* **Passerelle d’API (à déployer)**  : Les trois services principaux exposent une API externe au client. Dans les vrais systèmes, le nombre de fonctions peut grimper très rapidement avec la complexité du système. Des centaines de services peuvent être impliqués dans le rendu d’une page web complexe. La passerelle d’API est un point d’entrée unique dans le système, utilisé pour traiter les demandes et les router vers le service de back-end approprié ou pour appeler plusieurs services de back-end, en agrégeant les résultats. 

## <a name="sample-usage-of-piggymetrics"></a>Exemple d’utilisation de PiggyMetrics

Pour plus d’informations sur l’implémentation complète, consultez [PiggyMetrics](https://github.com/Azure-Samples/piggymetrics). Les exemples référencent le code source quand c’est nécessaire.
::: zone-end

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Provisionner une instance Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md)

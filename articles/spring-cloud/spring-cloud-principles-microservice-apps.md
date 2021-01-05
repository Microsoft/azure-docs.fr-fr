---
title: Java et système d’exploitation de base pour les applications de microservices Azure Spring Cloud
description: Principes pour la maintenance de l’intégrité d’un système d’exploitation de base et Java pour les applications de microservices Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 05/27/2020
ms.custom: devx-track-java
ms.openlocfilehash: 0c90062f1968cc7be5a742a67363f57b9632fdfa
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090673"
---
# <a name="java-and-base-os-for-spring-microservice-apps"></a>Système d’exploitation de base et Java pour les applications de microservices Spring

**Cet article s’applique à :** ✔️ Java

Voici quelques principes pour la maintenance de l’intégrité d’un système d’exploitation de base et Java pour les applications de microservices Spring.
## <a name="principles-for-healthy-java-and-base-os"></a>Principes pour l’intégrité de système d’exploitation de base et Java
* Il doit s’agir du même système d’exploitation de base sur les différents niveaux : De base | Standard | Premium.
    * Actuellement, les applications sur Azure Spring Cloud utilisent une combinaison de Debian 10 et Ubuntu 18.04.
    * Le service de build VMware utilise Ubuntu 18.04.
* Il doit s’agir du même système d’exploitation de base, quels que soient les points de départ du déploiement - source | JAR
    * Actuellement, les applications sur Azure Spring Cloud utilisent une combinaison de Debian 10 et Ubuntu 18.04.
* Le système d’exploitation de base doit être exempt de failles de sécurité.
    * Le système d’exploitation de base Debian 10 a 147 CVE ouverts.
    * Le système d’exploitation Ubuntu 18.04 base a 132 CVE ouverts.
* Il doit utiliser JRE-headless.
    * Actuellement, les applications sur Azure Spring Cloud utilisent JDK. JRE-headless est une image plus petite.
* Il doit utiliser les builds les plus récentes de Java.
    * Actuellement, les applications sur Azure Spring Cloud utilisent Java 8 build 242. Il s’agit d’une build obsolète.
 
Les systèmes Azul recherchent continuellement les modifications apportées aux systèmes d’exploitation de base et tiennent à jour les dernières images générées. Azure Spring Cloud recherche les modifications apportées aux images et les met à jour en continu entre les déploiements.
 
## <a name="faq-for-azure-spring-cloud"></a>FAQ sur Azure Spring Cloud

* Quelles sont les versions de Java prises en charge ? Version principale et numéro de build.
    * Prise en charge des versions LTS - Java 8 et 11.
    * Utilise la build la plus récente, par exemple, à ce jour, Java 8 build 252 et Java 11 build 7.
* Qui a créé ces runtimes Java ?
    * Azul Systems.
* Qu’est-ce que le système d’exploitation de base pour les images ?
    * Ubuntu 20.04 LTS (Focal Fossa). Les applications continueront à rester sur la version LTS la plus récente d’Ubuntu.
    * Consultez [Ubuntu 20.04 LTS (Focal Fossa)](http://releases.ubuntu.com/focal/).
* Comment faire pour télécharger un runtime Java pris en charge pour le développement local ? 
    * Consultez [Installer le JDK pour Azure et Azure Stack](/azure/developer/java/fundamentals/java-jdk-install).
* Comment faire pour obtenir un support pour les problèmes au niveau du runtime Java ?
    * Ouvrez un ticket de support auprès du Support Azure.
 
## <a name="default-deployment-on-azure-spring-cloud"></a>Déploiement par défaut sur Azure Spring Cloud

> ![Déploiement par défaut](media/spring-cloud-principles/spring-cloud-default-deployment.png)
 
## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Déployer votre première application Azure Spring Cloud](spring-cloud-quickstart.md)
* [Prise en charge à long terme de Java pour Azure et Azure Stack](/azure/developer/java/fundamentals/java-jdk-long-term-support)
---
title: Vue d’ensemble de DevOps Starter pour Azure | Microsoft Docs
description: Comprendre l’intérêt de DevOps Starter
services: devops-project
documentationcenter: ''
author: mlearned
manager: gwallace
editor: mlearned
ms.assetid: ''
ms.service: az-devops-project
ms.devlang: na
ms.topic: overview
ms.tgt_pltfrm: na
ms.workload: ''
ms.date: 03/24/2020
ms.author: mlearned
ms.openlocfilehash: 7c36539ef5be503b2cb7e14047e596522ef8e962
ms.sourcegitcommit: 03713bf705301e7f567010714beb236e7c8cee6f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92330646"
---
# <a name="overview-of-devops-starter"></a>Vue d’ensemble de DevOps Starter

 DevOps Starter permet de démarrer facilement sur Azure en utilisant GitHub Actions ou Azure DevOps. Il vous permet de lancer votre application favorite sur le service Azure de votre choix en seulement quelques étapes rapides à partir du portail Azure. 

 DevOps Starter configure tout ce dont vous avez besoin pour développer, déployer et superviser votre application. Vous pouvez utiliser le tableau de bord DevOps Starter pour superviser les validations de code, les builds et les déploiements dans une même vue du portail Azure.

## <a name="advantages-of-using-devops-starter"></a>Avantages de l’utilisation de DevOps Starter

  DevOps Starter prend en charge les deux fournisseurs CI/CD suivants pour automatiser vos déploiements :
  * [Actions GitHub](https://github.com/features/actions)
  * [Azure DevOps](https://azure.microsoft.com/services/devops)

  DevOps Starter automatise la configuration d’un pipeline complet d’intégration continue (CI) et de livraison continue (CD) pour votre application sur Azure.  Vous pouvez démarrer avec du code existant ou utiliser un des exemples d’application fournis. Vous pouvez ensuite déployer rapidement cette application sur différents services Azure, comme Machines virtuelles, App Service, Azure Kubernetes Service (AKS), Azure SQL Database et Azure Service Fabric.  

  Azure DevOps Starter s’occupe de toute la configuration initiale d’un pipeline DevOps, notamment configurer le dépôt Git initial, configurer le pipeline CI/CD, créer une ressource Application Insights pour la supervision et fournir une même vue de l’ensemble de la solution avec la création d’un tableau de bord DevOps Starter dans le portail Azure.

Vous pouvez utiliser DevOps Starter pour :

* Déployer rapidement votre application dans Azure
* Automatiser la configuration d’un workflow ou pipeline CI/CD
* Voir et comprendre comment configurer correctement un workflow ou pipeline CI/CD
* Personnaliser davantage les pipelines de mise en production selon vos scénarios spécifiques

## <a name="how-to-use-devops-starter"></a>Comment utiliser DevOps Starter ?

  DevOps Starter est disponible à partir du portail Azure. Vous créez une ressource DevOps Starter comme toute autre ressource Azure à partir du portail. DevOps Projects fournit une expérience de type Assistant pas à pas pour les différentes options de configuration.  

Vous choisissez plusieurs options de configuration dans le cadre de la configuration initiale. Ces options sont les suivantes :

* Sélectionner le fournisseur CI/CD
* Utiliser l’exemple d’application fourni ou utiliser votre propre code (uniquement pour Azure DevOps)
* Sélectionner un langage pour l’application
* Choisir un framework d’application en fonction du langage
* Sélectionner un service Azure (cible de déploiement)
* Sélectionner votre organisation GitHub ou Azure DevOps
* Choisir votre abonnement Azure
* Choisir l’emplacement des services Azure
* Choisir parmi différents niveaux tarifaires pour les services Azure

Après avoir créé votre projet DevOps Starter, vous pouvez :

* Personnaliser votre workflow GitHub ou pipeline Azure DevOps
* Utiliser des requêtes Pull pour gérer votre flux de code et conserver une qualité élevée
* Tester et générer chaque validation avant de fusionner votre code pour optimiser la qualité

Après avoir utilisé DevOps Starter, vous pouvez également supprimer toutes les ressources à partir d’un même emplacement dans le tableau de bord DevOps Starter sur le portail Azure.

## <a name="devops-starter-and-github-integration"></a>Intégration de DevOps Starter et GitHub

DevOps Starter prend maintenant en charge GitHub Actions en tant que fournisseur CI/CD. Il automatise tout le travail nécessaire dans GitHub pour configurer un workflow CI/CD à l’aide de GitHub Actions. Il crée un dépôt GitHub dans une organisation GitHub existante, puis valide un exemple d’application dans ce dépôt.  

L’automatisation établit également un déclencheur pour le workflow de sorte que chaque nouvelle validation de code lance un travail de génération et de déploiement au sein du workflow. L’application est déployée sur le service Azure de votre choix. Le workflow GitHub peut être personnalisé pour d’autres scénarios. 

## <a name="devops-starter-and-azure-devops-integration"></a>Intégration de DevOps Starter et Azure DevOps

Au moyen d’Azure DevOps, DevOps Starter automatise tout le travail nécessaire dans Azure Pipelines pour configurer un pipeline CI/CD. Il crée un dépôt Git dans une organisation Azure DevOps nouvelle ou existante, puis valide un exemple d’application ou votre code existant dans un nouveau dépôt Git.  

L’automatisation établit également un déclencheur CI pour la build de sorte que chaque nouvelle validation de code lance une build. DevOps Starter crée un déclencheur CD et déploie chaque nouvelle build réussie sur le service Azure de votre choix.  

Les pipelines de build et de mise en production peuvent être personnalisés pour d’autres scénarios. Vous pouvez également cloner les pipelines de build et de mise en production pour les utiliser dans d’autres projets.

## <a name="getting-started-with-devops-starter"></a>Bien démarrer avec DevOps Starter

* [Bien démarrer avec DevOps Starter](./azure-devops-project-github.md)

##  <a name="devops-starter-videos"></a>Vidéos de DevOps Starter

* [Create CI/CD with Azure DevOps Starter](https://www.youtube.com/watch?v=NuYDAs3kNV8)

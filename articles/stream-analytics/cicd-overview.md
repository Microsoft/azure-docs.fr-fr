---
title: Intégration et déploiement continus sur Azure Stream Analytics
description: Cet article donne une vue d’ensemble du pipeline d’intégration et de déploiement continus (CI/CD) pour Azure Stream Analytics.
services: stream-analytics
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 9/22/2020
ms.openlocfilehash: b5057eb8c84e839f504060228986ea759c8bdc3d
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123165"
---
# <a name="continuous-integration-and-deployment-cicd-for-azure-stream-analytics"></a>Intégration et déploiement continus (CI/CD) sur Azure Stream Analytics

Vous avez la possibilité de déployer votre tâche Azure Stream Analytics en continu via une intégration du contrôle de code source. L’intégration du contrôle de code source active un flux de travail dans lequel une mise à jour de code déclenche un déploiement de ressources sur Azure. Cet article décrit les étapes de base pour la création d’un pipeline d’intégration et de déploiement continus (CI/CD).

Si vous débutez avec Azure Stream Analytics, commencez par lire l’article [Démarrage rapide d’Azure Stream Analytics](stream-analytics-quick-create-portal.md).

## <a name="create-a-cicd-pipeline"></a>Créer un pipeline CI/CD

Suivez les étapes de ce guide pour créer un pipeline CI/CD pour Stream Analytics.

1. Développez une requête Azure Stream Analytics.

   Utilisez les outils Azure Stream Analytics pour [Visual Studio Code](./quick-create-visual-studio-code.md) ou [Visual Studio](stream-analytics-quick-create-vs.md) pour [développer et tester des requêtes en local](develop-locally.md). Vous pouvez également [exporter une tâche existante](visual-studio-code-explore-jobs.md#export-a-job-to-a-local-project) vers un projet local.

2. Validez vos projets Azure Stream Analytics dans votre système de contrôle de code source, de la même manière qu’un référentiel Git.

3. Utilisez les [outils CI/CD Azure Stream Analytics](cicd-tools.md) pour compiler vos projets et générer des modèles de gestion des ressources Azure à des fins de déploiement.

4. Exécutez des [tests de script automatisé](cicd-tools.md#automated-test) pour la régression de qualité.

5. [Déployez la tâche](cicd-tools.md#deploy-to-azure) automatiquement dans Azure.

## <a name="auto-build-test-and-deploy"></a>Génération, test et déploiement automatiques

Vous pouvez utiliser la ligne de commande et les [outils CI/CD Azure Stream Analytics](cicd-tools.md) pour exécuter des tâches de génération, de test et de déploiement automatiques. Vous pouvez également configurer un pipeline CI/CD dans [Azure Pipelines](set-up-cicd-pipeline.md). Azure Pipelines permet d’activer des fonctionnalités plus avancées, telles que la gestion des pipelines, la visualisation et les déclencheurs.

## <a name="next-steps"></a>Étapes suivantes

* [Automatiser les compilations, les tests et les déploiements d’une tâche Azure Stream Analytics à l’aide des outils CI/CD](cicd-tools.md)
* [Configurer un pipeline CI/CD pour une tâche Stream Analytics à l’aide d’Azure Pipelines](set-up-cicd-pipeline.md)
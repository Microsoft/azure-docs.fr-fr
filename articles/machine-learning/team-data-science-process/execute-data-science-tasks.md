---
title: Exécuter des tâches de science des données - Team Data Science Process
description: Explique comment un scientifique des données peut réaliser un projet de science des données de manière traçable et collaborative, avec gestion de versions.
author: marktab
manager: marktab
editor: marktab
ms.service: machine-learning
ms.subservice: team-data-science-process
ms.topic: article
ms.date: 11/17/2020
ms.author: tdsp
ms.custom: seodec18, previous-author=deguhath, previous-ms.author=deguhath
ms.openlocfilehash: e47dad8498c48a5da5307517efe493fa5c1aa590
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94748061"
---
# <a name="execute-data-science-tasks-exploration-modeling-and-deployment"></a>Exécuter des tâches de science des données : exploration, modélisation et déploiement

Les tâches courantes de science des données incluent l’exploration, la modélisation et le déploiement. Cet article décrit les tâches de science des données courantes, comme l’exploration interactive des données, l’analyse des données, la création de rapports et la création de modèles. Les options de déploiement d’un modèle dans un environnement de production peuvent inclure :

- [Azure Machine Learning](../index.yml)
- [SQL Server avec les services ML](/sql/advanced-analytics/r/r-services)
- [Microsoft Machine Learning Server](/machine-learning-server/what-is-machine-learning-server)


## <a name="1--exploration"></a>1. <a name='DataQualityReportUtility-1'></a> Exploration 

Un scientifique des données peut effectuer les tâches d’exploration et de création de rapports de plusieurs façons : à l’aide de bibliothèques et de packages disponibles pour Python (matplotlib, par exemple) ou de R (ggplot ou lattice, par exemple). Il peut personnaliser ce code en fonction des besoins d’exploration de données pour des scénarios spécifiques. Les contraintes liées aux données structurées diffèrent de celles liées aux données non structurées, telles que le texte ou les images. 

En outre, des produits tels qu’Azure Machine Learning fournissent une [préparation avancée des données](../how-to-create-register-datasets.md) pour l’exploration des données et le data wrangling, notamment en ce qui concerne la création de caractéristiques. L’utilisateur doit choisir les outils, bibliothèques et packages qui répondent le mieux à ses besoins. 

Le livrable à la fin de cette phase est un rapport d’exploration de données. Ce rapport doit fournir une vue très complète des données à utiliser pour la modélisation et une évaluation indiquant si les données sont appropriées pour passer à l’étape de modélisation. 

## <a name="2--modeling"></a>2. <a name='ModelingUtility-2'></a> Modélisation

Il existe de nombreux kits de ressources et packages pour l’apprentissage de modèles dans une variété de langages. Les scientifiques des données sont libres d’utiliser ceux avec lesquels ils se sentent le mieux, l’important étant qu’ils tiennent compte des contraintes de précision et de latence liées aux cas d’usage d’entreprise et aux scénarios de production.

### <a name="model-management"></a>La gestion des modèles
Une fois que plusieurs modèles ont été générés, vous devez généralement vous doter d’un système qui vous permet de les inscrire et de les gérer. En règle générale, vous avez besoin d’une combinaison de scripts ou d’API et d’une base de données backend ou d’un système de gestion de versions. Voici quelques options que vous pouvez envisager pour ces tâches de gestion :

1. [Azure Machine Learning - Service de gestion des modèles](../index.yml)
2. [ModelDB de MIT](http://modeldb.csail.mit.edu:3000/projects) 
3. [SQL Server comme système de gestion de modèle](https://blogs.technet.microsoft.com/dataplatforminsider/2016/10/17/sql-server-as-a-machine-learning-model-management-system/)
4. [Microsoft Machine Learning Server](/sql/advanced-analytics/r/r-server-standalone)

## <a name="3--deployment"></a>3. <a name='Deployment-3'></a> Déploiement

Le déploiement en production permet à un modèle de jouer un rôle actif dans une entreprise. Les prédictions issues d’un modèle déployé peuvent être utilisées pour les décisions d’entreprise.

### <a name="production-platforms"></a>Plateformes de production
Il existe différentes approches et plateformes pour mettre les modèles en production. Voici quelques options :


- [Déploiement du modèle dans Azure Machine Learning](../how-to-deploy-and-where.md)
- [Déploiement d’un modèle dans SQL Server](/sql/advanced-analytics/tutorials/sqldev-py6-operationalize-the-model)
- [Microsoft Machine Learning Server](/sql/advanced-analytics/r/r-server-standalone)

> [!NOTE]
> Avant de procéder au déploiement, vous devez vérifier que le niveau de la latence du modèle est suffisamment faible pour permettre l’utilisation de ce dernier dans l’environnement de production.
>
>

Des exemples sont disponibles dans des procédures pas à pas qui illustrent toutes les étapes de **scénarios spécifiques**. L’article [Exemples de procédures pas à pas](walkthroughs.md) les liste et les décrit brièvement, en les accompagnant de liens. Ces procédures illustrent comment combiner des outils et services locaux ou cloud dans un flux de travail ou un pipeline pour créer une application intelligente.

> [!NOTE]
> Pour un déploiement à l’aide de Microsoft Azure Machine Learning Studio, consultez [Déploiement d’un service web Azure Machine Learning](../classic/deploy-a-machine-learning-web-service.md).
>
>

### <a name="ab-testing"></a>Test A/B
Quand plusieurs modèles sont en production, il peut être utile d’effectuer un [test A/B](https://en.wikipedia.org/wiki/A/B_testing) pour comparer les performances des modèles. 

 
## <a name="next-steps"></a>Étapes suivantes

[Suivre la progression des projets de science des données](track-progress.md) montre comment un scientifique des données peut suivre la progression d’un projet de science des données.

[Opération de modèle et Intégration continue/Livraison continue](ci-cd-flask.md) montre comment l’utilisation de modèles développés permet de réaliser une intégration continue/livraison continue.

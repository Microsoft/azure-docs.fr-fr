---
title: 'ML Studio (classique) : Créer des points de terminaison de service web – Azure'
description: Créez des points de terminaison de service web dans Azure Machine Learning Studio (classique). Chaque point de terminaison du service web est adressé, limité et géré de façon indépendante.
services: machine-learning
ms.service: machine-learning
ms.subservice: studio
ms.topic: how-to
author: likebupt
ms.author: keli19
ms.custom: seodec18
ms.date: 02/15/2019
ms.openlocfilehash: 1032a90a35e60643e2ce937ed457a1fe3493d4d7
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93322884"
---
# <a name="create-endpoints-for-deployed-azure-machine-learning-studio-classic-web-services"></a>Créer des points de terminaison pour les services web Azure Machine Learning Studio (classique) déployés

**S’APPLIQUE À :**  ![S’applique à ](../../../includes/media/aml-applies-to-skus/yes.png)Machine Learning Studio (classique)   ![Ne s’applique pas à ](../../../includes/media/aml-applies-to-skus/no.png)[Azure Machine Learning](../overview-what-is-machine-learning-studio.md#ml-studio-classic-vs-azure-machine-learning-studio)


> [!NOTE]
> Cette rubrique décrit les techniques applicables à un service web Machine Learning **classique**.

Après le déploiement d’un service web, un point de terminaison par défaut est créé pour ce service. Le point de terminaison par défaut peut être appelé à l’aide de sa clé API. Vous pouvez ajouter des points de terminaison supplémentaires avec leurs propres clés à partir du portail de Services Web.
Chaque point de terminaison du service web est adressé, limité et géré de façon indépendante. Chaque point de terminaison est une URL unique, avec clé d’autorisation que vous pouvez distribuer à vos clients.

## <a name="add-endpoints-to-a-web-service"></a>Ajouter des points de terminaison à un service web

Vous pouvez ajouter un point de terminaison à un service web à l’aide du portail des services web Azure Machine Learning. Une fois le point de terminaison créé, vous pouvez l’exploiter via des API synchrones, des API de lot et des feuilles de calcul Microsoft Excel.

> [!NOTE]
> Si vous avez ajouté des points de terminaison au service web, vous ne pouvez pas supprimer le point de terminaison par défaut.

1. Dans la colonne de navigation de gauche de Machine Learning Studio (classique), cliquez sur Services web.
2. En bas du tableau de bord du service web, cliquez sur **Gérer les points de terminaison**. Le portail des services web Azure Machine Learning s’ouvre sur la page des points de terminaison pour le service web.
3. Cliquez sur **Nouveau**.
4. Tapez un nom et une description pour le point de terminaison. Les noms de point de terminaison doivent compter au maximum 24 caractères, et doivent être composés de lettres minuscules ou de chiffres. Sélectionnez le niveau de journalisation et activez les exemples de données si nécessaire. Pour plus d’informations sur la journalisation, consultez [Activation de la journalisation pour les services web de Machine Learning](web-services-logging.md).

## <a name="scale-a-web-service-by-adding-additional-endpoints"></a><a id="scaling"></a> Mettre à l’échelle un service web en ajoutant des points de terminaison supplémentaires

Par défaut, chaque service web publié est configuré pour prendre en charge 20 requêtes simultanées, avec un maximum de 200 requêtes. Azure Machine Learning Studio (classique) optimise automatiquement ce paramètre pour améliorer les performances de votre service web et la valeur de portail est ignorée.

Si vous envisagez d’appeler l’API avec une charge supérieure à ce que le nombre maximal d’appels simultanés (200) prend en charge, vous devez créer plusieurs points de terminaison sur le même service web. Vous pouvez ensuite répartir la charge entre tous de façon aléatoire.

La mise à l’échelle d’un service web est une tâche courante. Parmi les raisons d’effectuer une mise à l’échelle figurent la nécessité de prendre en charge plus de 200 demandes simultanées, d’augmenter la disponibilité via plusieurs points de terminaison, ou de fournir des points de terminaison distincts pour le service web. Vous pouvez augmenter l’échelle en ajoutant des points de terminaison supplémentaires pour le même service web via le portail du [service web Azure Machine Learning](https://services.azureml.net/).

N’oubliez pas que l’utilisation d’un nombre élevé d’accès concurrentiels peut être préjudiciable si vous n’appelez pas l’API avec un taux tout aussi élevé. Vous pouvez constater des délais d’attente et/ou des pics de latence sporadiques si vous placez une charge relativement faible sur une API configurée pour une charge élevée.

Les API synchrones sont généralement utilisées dans les situations où vous souhaitez une latence faible. La latence implique ici le temps nécessaire à l’API pour compléter une demande et ne prend pas en compte les retards de réseau. Supposons que vous avez une API avec une latence de 50 millisecondes. Pour utiliser toute la capacité disponible avec le niveau de limitation élevé et le nombre maximal d’appels simultanés = 20, vous devez appeler cette API 20 * 1000 / 50 = 400 fois par seconde. De même, un nombre maximum d’appels simultanés de 200 vous permet d’appeler l’API 4000 fois par seconde, en supposant une latence de 50 millisecondes.

## <a name="next-steps"></a>Étapes suivantes

[Utilisation d’un service web Azure Machine Learning](consume-web-services.md).
---
title: Comprendre les fonctionnalités de recherche dans Azure Purview (préversion)
description: Cet article explique comment Azure Purview permet la découverte de données à l’aide de fonctionnalités de recherche.
author: chanuengg
ms.author: csugunan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: conceptual
ms.date: 11/06/2020
ms.openlocfilehash: af8ec9e0aac38240c7da92edd614892ff65712e2
ms.sourcegitcommit: 65db02799b1f685e7eaa7e0ecf38f03866c33ad1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96550161"
---
# <a name="understand-search-features-in-azure-purview"></a>Comprendre les fonctionnalités de recherche dans Azure Purview

Cet article fournit une vue d’ensemble de l’expérience de recherche dans Azure Purview. La recherche est une fonctionnalité de plateforme principale de Purview, qui alimente les expériences de découverte de données et de gouvernance de l’utilisation des données dans une organisation.

## <a name="search"></a>Rechercher

L’expérience de recherche Purview est alimentée par un index de recherche managé. Une fois qu’une source de données est inscrite auprès de Purview, ses métadonnées sont indexées par le service de recherche pour permettre une découverte facile. L’index fournit des fonctionnalités de pertinence de la recherche et complète les demandes de recherche en interrogeant des millions de ressources de métadonnées. Une recherche vous permet de découvrir, de comprendre et d’utiliser les données pour en tirer le meilleur parti.

L’expérience de recherche dans Purview est un processus en trois étapes :

1. La zone de recherche affiche l’historique contenant les mots clés et ressources récemment utilisés.
1. Lorsque vous commencez à taper les séquences de touches, la recherche suggère les mots clés et ressources correspondants. 
1. La page des résultats de la recherche s’affiche avec les ressources correspondant au mot clé entré.

## <a name="reduce-the-time-to-discover-data"></a>Réduire le temps de découverte des données

La découverte de données constitue la première étape d’une charge de travail d’analytique données ou de gouvernance des données pour les consommateurs de données. La découverte de données peut prendre du temps, car vous ne savez peut-être pas où trouver les données que vous souhaitez. Même après avoir trouvé les données, vous avez peut-être des doutes quant à la possibilité d’approuver ou non les données et d’y appliquer des dépendances. 

L’objectif de la recherche dans Azure Purview est d’accélérer le processus de découverte des données en fournissant des gestes, afin de trouver rapidement les données importantes.

## <a name="recent-search-and-suggestions"></a>Recherche et suggestions récentes

Bien souvent, vous pouvez travailler sur plusieurs projets en même temps. Pour faciliter la reprise de projets précédents, la recherche Purview offre la possibilité de voir les mots clés et suggestions de recherche récents. En outre, vous pouvez gérer l’historique de recherche récent en sélectionnant **Afficher tout** dans la liste déroulante de la zone de recherche.

## <a name="filters"></a>Filtres

Les filtres (également appelés *facettes*) sont conçus pour compléter la recherche. Les filtres s’affichent dans la page des résultats de la recherche. Les filtres de la page des résultats de la recherche incluent la classification, l’étiquette de sensibilité, la source de données et les propriétaires. Les utilisateurs peuvent sélectionner des valeurs spécifiques dans un filtre pour afficher uniquement les ressources de données correspondantes et limiter le résultat de la recherche à celles-ci.

## <a name="hit-highlighting"></a>Mise en surbrillance des correspondances

Les mots clés correspondants dans la page des résultats de la recherche sont mis en surbrillance pour faciliter l’identification de la raison pour laquelle une ressource de données a été retournée par une recherche. Le correspondance des mots clés peut apparaître dans plusieurs champs, tels que le nom, la description et le propriétaire de la ressource de données.

La raison pour laquelle une ressource de données est incluse dans une recherche n’est pas toujours évidente, même si la mise en surbrillance des correspondances est activée. Par défaut, la recherche porte sur toutes les propriétés. Par conséquent, une ressource de données peut être retournée en raison d’une correspondance sur une propriété au niveau de la colonne. Et dans la mesure où plusieurs utilisateurs peuvent annoter les ressources de données avec leurs propres classifications et descriptions, les métadonnées ne sont pas toutes affichées dans la liste des résultats de la recherche.

## <a name="sort"></a>Trier

Les utilisateurs ont deux options pour trier les résultats de la recherche. Ils peuvent effectuer un tri sur la ressource ou sur la pertinence de la recherche par défaut.

## <a name="search-relevance"></a>Pertinence de la recherche

La pertinence est l’ordre de tri par défaut dans la page des résultats de la recherche. La pertinence de la recherche trouve des documents incluant le mot clé de recherche (certains ou tous). Les ressources qui contiennent de nombreuses instances du mot clé de recherche sont classées plus haut. En outre, des mécanismes de scoring personnalisés sont constamment réglés pour améliorer la pertinence de la recherche.

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Créer un compte Azure Purview dans le portail Azure](create-catalog-portal.md)
* [Démarrage rapide : Créer un compte Azure Purview à l’aide d’Azure PowerShell ou d’Azure CLI](create-catalog-powershell.md)
* [Démarrage rapide : Utiliser Purview Studio](use-purview-studio.md)

---
title: Query Performance Insight - Azure Database for MariaDB
description: Cet article décrit la fonctionnalité Query Performance Insight dans Azure Database for MariaDB
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: b643ba3305736480e06d7c10d594b2271839038f
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94536330"
---
# <a name="query-performance-insight-in-azure-database-for-mariadb"></a>Query Performance Insight dans Azure Database for MariaDB

**S’applique à :** Azure Database for MariaDB 10.2

L’analyse des performances des requêtes vous aide à identifier rapidement vos requêtes les plus longues, comment elles évoluent au fil du temps et les attentes qui les affectent.

## <a name="common-scenarios"></a>Scénarios courants

### <a name="long-running-queries"></a>Requêtes longues

- Identification des requêtes dont l’exécution est la plus longue au cours des X dernières heures
- Identification des N premières requêtes en attente de ressources
 
### <a name="wait-statistics"></a>Statistiques d’attente

- Compréhension de la nature de l’attente pour une requête
- Fonctionnement des tendances des attentes de ressources en cas de conflit de ressource

## <a name="permissions"></a>Autorisations

Les autorisations **Propriétaire** ou **Contributeur** sont nécessaires pour voir le texte des requêtes dans l’analyse des performances des requêtes. Un **Lecteur** peut voir les graphiques et les tableaux, mais pas le texte des requêtes.

## <a name="prerequisites"></a>Prérequis

Pour que l’analyse des performances des requêtes puisse fonctionner, des données doivent exister dans le [Magasin des requêtes](concepts-query-store.md).

## <a name="viewing-performance-insights"></a>Affichage d’une analyse des performances

La vue [Analyse des performances des requêtes](concepts-query-performance-insight.md) du portail Azure montre des visualisations sur les informations importantes du magasin des requêtes.

Sur la page du portail de votre serveur Azure Database pour MariaDB, sélectionnez **Analyse des performances des requêtes** sous la section **Performances intelligentes** de la barre de menus.

### <a name="long-running-queries"></a>Requêtes longues

L’onglet **Requêtes longues** montre les 5 requêtes dont la durée moyenne d’exécution est la plus longue, agrégées par intervalles de 15 minutes. Vous pouvez afficher plus de requêtes en faisant une sélection dans la liste déroulante **Nombre de requêtes**. Quand vous faites cela, les couleurs du graphique peuvent changer pour un ID de requête spécifique.

Vous pouvez cliquer et faire glisser dans le graphique pour réduire l’affichage à une fenêtre de temps spécifique. Vous pouvez également utiliser les icônes Zoom avant et Zoom arrière pour afficher une période plus courte ou plus longue, respectivement.

![Requêtes longues d’analyse des performances des requêtes](./media/concepts-query-performance-insight/query-performance-insight-landing-page.png)

### <a name="wait-statistics"></a>Statistiques d’attente 

> [!NOTE]
> Les statistiques d’attente sont destinées à la résolution des problèmes de performances des requêtes. Nous vous recommandons de ne les activer qu’à des fins de résolution des problèmes. <br>Si le portail Azure indique le message d’erreur « *En raison du problème rencontré pour « Microsoft.DBforMariaDB », il est impossible de répondre à la demande. Si ce problème persiste ou est inattendu, contactez le support avec ces informations.*  » quand vous visualisez les statistiques d’attente, utilisez une période plus courte.

Les statistiques d’attente fournissent une vue des événements d’attente qui se produisent pendant l’exécution d’une requête spécifique. Pour en savoir plus sur les types d’événements d’attente, consultez la [documentation du moteur MySQL](https://go.microsoft.com/fwlink/?linkid=2098206).

Sélectionnez l’onglet **Statistiques sur les attentes** pour voir les visualisations correspondantes sur les temps d’attente dans le serveur.

Les requêtes affichées dans la vue des statistiques d’attente sont regroupées en fonction des requêtes qui présentent les temps d’attente les plus longs au cours de l’intervalle indiqué.

![Statistiques sur les attentes de l’analyse des performances des requêtes](./media/concepts-query-performance-insight/query-performance-insight-wait-statistics.png)

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus d’informations sur la [surveillance et l’optimisation](concepts-monitoring.md) dans Azure Database for MariaDB.
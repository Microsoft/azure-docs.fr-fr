---
title: Recommandations relatives aux performances de Database Advisor pour Azure SQL Database
description: Azure SQL Database fournit des recommandations pour les bases de données qui peuvent améliorer les performances des requêtes dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: fasttrack-edit, sqldbrb=1
ms.devlang: ''
ms.topic: conceptual
author: danimir
ms.author: danil
ms.reviewer: wiassaf, sstein
ms.date: 03/10/2020
ms.openlocfilehash: 79ccf0f8aae7e915601081f875cea294de52d787
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500850"
---
# <a name="database-advisor-performance-recommendations-for-azure-sql-database"></a>Recommandations relatives aux performances de Database Advisor pour Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Azure SQL Database apprend avec votre application et s’adapte à cette dernière. Azure SQL Database a un certain nombre de conseillers en bases de données qui fournissent des recommandations personnalisées vous permettant d’optimiser les performances. Ces conseillers en bases de données évaluent et analysent en permanence l’historique d’utilisation, et fournissent des recommandations en fonction des modèles de charge de travail qui permettent d’améliorer les performances.

## <a name="performance-overview"></a>Vue d’ensemble sur les performances

La vue d’ensemble sur les performances fournit un résumé des performances de votre base de données et vous aide à effectuer le réglage des performances et la résolution des problèmes.

![Vue d’ensemble des performances pour Azure SQL Database](./media/database-advisor-implement-performance-recommendations/performance-overview-annotated.png)

- La vignette **Recommandations** fournit une répartition des recommandations de paramétrage de votre base de données (les trois premières recommandations sont affichées s’il en existe plus). Cliquez sur cette vignette pour accéder aux **[options de recommandations en matière de performances](database-advisor-find-recommendations-portal.md#viewing-recommendations)** .
- La vignette **Activité de paramétrage** fournit un résumé des actions de paramétrage en cours et terminées pour votre base de données, ce qui vous donne un aperçu rapide de l’historique des activités de paramétrage. Cliquez sur cette vignette pour afficher l'historique de paramétrage complet de votre base de données.
- La vignette **Réglage automatique** affiche la **[configuration de réglage automatique](automatic-tuning-enable.md)** de votre base de données (options de réglage qui sont appliquées automatiquement à votre base de données). Cliquez sur cette vignette pour ouvrir la boîte de dialogue de configuration de l'automatisation.
- La vignette **Requêtes de base de données** affiche le résumé des performances de requête de votre base de données (utilisation DTU globale et requêtes les plus consommatrices de ressources). La page **[Analyse des performances des requêtes](query-performance-insight-use.md)** s’affiche si vous cliquez sur cette vignette.

## <a name="performance-recommendation-options"></a>Options de recommandations en matière de performances

Les options des recommandations en matière de performances disponibles dans Azure SQL Database sont :

| Recommandation en matière de performances | Prise en charge d’une base de données unique et d’une base de données mise en pool | Prise en charge de la base de données d’instance |
| :----------------------------- | ----- | ----- |
| **Recommandations de création d’index** : recommande de créer des index susceptibles d’améliorer les performances de votre charge de travail. | Oui | Non |
| **Recommandations de suppression d’index** : recommande la suppression quotidienne des index redondants et en double, excepté pour les index uniques ainsi que ceux qui n’ont pas été utilisés depuis longtemps (>90 jours). Notez que l’option n’est pas compatible avec les applications utilisant la commutation de partition et les indicateurs d’index. La suppression des index inutilisés n’est pas prise en charge pour les niveaux de service Premium et Critique pour l’entreprise. | Oui | Non |
| **Recommandations de paramétrage de requêtes (préversion)**  : recommande le paramétrage forcé dans les cas où une ou plusieurs requêtes sont constamment recompilées mais ont en fin de compte le même plan d’exécution de requête. | Oui | Non |
| **Recommandations de résolution des problèmes de schéma (préversion)**  : des recommandations pour la correction de schéma s’affichent quand Azure SQL Database détecte une anomalie dans le nombre d’erreurs SQL liées au schéma qui se produisent sur votre base de données. Microsoft déconseille actuellement la recommandation « Résoudre les problèmes de schéma ». | Oui | Non |

![Recommandations relatives aux performances pour Azure SQL Database](./media/database-advisor-implement-performance-recommendations/performance-recommendations-annotated.png)

Pour appliquer les recommandations en matière de performances, consultez [Application des recommandations](database-advisor-find-recommendations-portal.md#applying-recommendations). Pour afficher l’état des recommandations, consultez [Surveillance des opérations](database-advisor-find-recommendations-portal.md#monitoring-operations).

Vous trouverez également un historique complet des actions de réglage qui ont été appliquées dans le passé.

## <a name="create-index-recommendations"></a>Recommandations relatives à la création d’un index

Azure SQL Database supervise continuellement les requêtes en cours d’exécution et identifie les index susceptibles d’améliorer les performances. Lorsque le service détermine avec certitude qu’un index spécifique est manquant, une recommandation **Créer un index** est créée.

Azure SQL Database établit un climat de confiance en estimant le gain de performances que l’index pourrait offrir au fil du temps. Selon le gain estimé, les recommandations sont réparties en trois catégories : gain de performances élevé, moyen ou faible.

Les index créés à l’aide de recommandations sont toujours indiqués comme ayant été créés automatiquement. Vous pouvez visualiser les index créés automatiquement en consultant la [vue sys.indexes](/sql/relational-databases/system-catalog-views/sys-indexes-transact-sql). Les index créés automatiquement ne bloquent pas les commandes ALTER/RENAME.

Si vous tentez de supprimer la colonne qui comporte un index créé automatiquement, la commande aboutit. L’index créé automatiquement est également supprimé dans le cadre de l’exécution de la commande. Les index standard bloquent la commande ALTER/RENAME sur les colonnes indexées.

Une fois que la recommandation de création d’index a été appliquée, Azure SQL Database compare le niveau de performance des requêtes avec le niveau de performance de référence. Si le nouvel index a amélioré les performances, la recommandation est indiquée comme étant fructueuse, et le rapport d’impact est disponible. Si l’index n’a pas entraîné d’amélioration des performances, il est automatiquement annulé. Azure SQL Database applique ce processus pour garantir que les recommandations améliorent les performances de la base de données.

Toutes les recommandations **Créer un index** comportent une stratégie d’abstention qui empêche la mise en œuvre de la recommandation si l’utilisation des ressources d’une base de données ou d’un pool est élevée. La stratégie d’abstention prend en compte l’UC, les E/S de données, les E/S de journal et le stockage disponible.

Si le processeur, les E/S de données ou les E/S de journal dépassent 80 % au cours des dernières 30 minutes, la recommandation de création d’index est reportée. Si l’espace de stockage disponible est inférieur à 10 % une fois que l’index a été créé, la recommandation passe en état d’erreur. Si après deux jours, le paramétrage automatique estime toujours que cet index peut être avantageux, le processus redémarre.

Ce processus est répété jusqu’à ce que le stockage disponible soit suffisant pour créer un index ou jusqu’à ce que l’index ne soit plus considéré comme bénéfique.

## <a name="drop-index-recommendations"></a>Recommandations relatives à la suppression d’index

Outre la détection des index manquants, Azure SQL Database procède à l’analyse continue du niveau de performance des index existants. Si un index n’est pas utilisé, Azure SQL Database recommande sa suppression. Il est recommandé de supprimer un index dans deux cas :

- L’index est un doublon d’un autre index (mêmes colonne, schéma de partition et filtres indexés et inclus).
- L’index n’a pas été utilisé pendant une période prolongée (93 jours).

Les recommandations de suppression d’index passent également par la vérification, suite à l’implémentation. Si le niveau de performance s’améliore, le rapport d’impact est disponible. Si le niveau de performance se détériore, la recommandation est annulée.

## <a name="parameterize-queries-recommendations-preview"></a>Recommandations de paramétrage de requêtes (préversion)

Les recommandations liées au *paramétrage des requêtes* s’affichent lorsque vous disposez d’une ou plusieurs requêtes qui sont constamment recompilées, mais ont en fin de compte le même plan d’exécution de requête. Cette condition offre une opportunité de mise en œuvre d’un paramétrage forcé. Le paramétrage forcé autorise la mise en cache et la réutilisation ultérieure des plans de requête, ce qui améliore le niveau de performance et réduit l’utilisation des ressources.

Chaque requête doit initialement être compilée pour générer un plan d’exécution. Chaque plan généré est ajouté au cache du plan. Les exécutions ultérieures de la même requête peuvent réutiliser ce plan à partir du cache, évitant ainsi toute compilation supplémentaire.

Les requêtes incluant des valeurs non paramétrées peuvent entraîner une détérioration des performances, car le plan d’exécution est recompilé chaque fois que les valeurs non paramétrées diffèrent. Dans de nombreux cas, des requêtes identiques comportant différentes valeurs de paramètre génèrent les mêmes plans d’exécution. Toutefois, ces plans sont toujours ajoutés séparément au cache du plan.

Le processus de recompilation des plans d’exécution utilise des ressources de base de données, augmente la durée des requêtes et dépasse la capacité du cache du plan. Ces événements entraînent à leur tour la suppression des plans de ce cache. Il est possible de modifier ce comportement en définissant l’option de paramétrage forcé au niveau de la base de données.

Pour vous aider à évaluer l’impact de cette recommandation, nous vous fournissons une comparaison entre l’utilisation réelle de l’UC et son utilisation projetée (comme si la recommandation avait été appliquée). Cette recommandation peut vous aider à économiser les ressources d’UC. Elle peut également contribuer à réduire la durée des requêtes et les surcharges de cache du plan, permettant ainsi la conservation dans le cache et la réutilisation d’un plus grand nombre de plans. Vous pouvez appliquer cette recommandation rapidement en sélectionnant la commande **Appliquer**.

Une fois cette recommandation appliquée, le paramétrage forcé est activé en l’espace de quelques minutes dans votre base de données. Il démarre le processus d’analyse, qui dure environ 24 heures. Au terme de cette période, vous pouvez consulter le rapport de validation. Ce rapport présente l’utilisation de l’UC par votre base de données 24 heures avant et après l’application de la recommandation. Azure SQL Database Advisor intègre un mécanisme de sécurité qui annule automatiquement la recommandation appliquée si une baisse des performances est détectée.

## <a name="fix-schema-issues-recommendations-preview"></a>Recommandations liées à la résolution des problèmes de schéma (préversion)

> [!IMPORTANT]
> Microsoft déconseille actuellement la recommandation « Résoudre les problèmes de schéma ». Nous vous recommandons d’utiliser [Intelligent Insights](intelligent-insights-overview.md) pour analyser vos problèmes de performances de base de données, y compris les problèmes de schéma précédemment couverts par la recommandation de résolution de ce type de problèmes.

Les recommandations **Résoudre les problèmes de schéma** s’affichent lorsqu’Azure SQL Database détecte une anomalie dans le nombre d’erreurs SQL liées au schéma qui se produisent sur votre base de données. Cette recommandation apparaît généralement lorsque votre base de données rencontre plusieurs erreurs liées au schéma (nom de colonne non valide, nom d’objet incorrect, etc.) en l’espace d’une heure.

Les « problèmes de schéma » désignent une classe d’erreurs de syntaxe. Ils surviennent lorsque la définition de la requête SQL et la définition du schéma de base de données ne sont pas alignées. Par exemple, il est possible que l’une des colonnes attendues par la requête soit absente de la table cible, ou inversement.

La recommandation « Résoudre les problèmes de schéma » s’affiche lorsqu’Azure SQL Database détecte une anomalie dans le nombre d’erreurs SQL liées au schéma qui se produisent dans votre base de données. Le tableau suivant répertorie les erreurs liées à des problèmes de schéma :

| Code d’erreur SQL | Message |
| --- | --- |
| 201 |La procédure ou fonction ' *’ attend le paramètre ’* ', qui n’a pas été fourni. |
| 207 |Nom de colonne non valide '*'. |
| 208 |Nom d'objet non valide ’*’. |
| 213 |Le nom ou le numéro de colonne des valeurs fournies ne correspond pas à la définition de la table. |
| 2812 |Impossible de trouver la procédure stockée '*'. |
| 8144 |La procédure ou la fonction * a trop d’arguments spécifiés. |

## <a name="custom-applications"></a>Applications personnalisées

Les développeurs peuvent envisager de développer des applications personnalisées à l’aide des recommandations relatives aux performances pour Azure SQL Database. Toutes les recommandations listées dans le portail pour une base de données sont accessibles par le biais de l’API [Get-AzSqlDatabaseRecommendedAction](/powershell/module/az.sql/get-azsqldatabaserecommendedaction).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur le paramétrage automatique des index de base de données et des plans d’exécution de requêtes, consultez l’article [Réglage automatique dans Azure SQL Database](automatic-tuning-overview.md).
- Pour plus d’informations sur l’analyse automatique des performances de base de données à l’aide des diagnostics automatisés et de l’analyse de la cause racine des problèmes de performances, consultez l’article [Intelligent Insights](intelligent-insights-overview.md).
- Pour connaître l’impact de vos principales requêtes sur les performances, consultez [Query Performance Insights](query-performance-insight-use.md) (Analyse des performances des requêtes).
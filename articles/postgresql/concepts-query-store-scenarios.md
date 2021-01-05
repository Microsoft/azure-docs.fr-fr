---
title: Scénarios de magasin des requêtes - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit certains scénarios relatifs au Magasin des requêtes dans Azure Database pour PostgreSQL - Serveur unique.
author: sunilagarwal
ms.author: sunila
ms.service: postgresql
ms.topic: conceptual
ms.date: 5/6/2019
ms.openlocfilehash: ad646e39e9b6a87611dc2f40330a77fe70c92c27
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91710547"
---
# <a name="usage-scenarios-for-query-store"></a>Scénarios d’utilisation du Magasin des requêtes

**S’applique à :** Azure Database pour PostgreSQL - Serveur unique versions 9.6, 10, 11

Vous pouvez utiliser le Magasin des requêtes dans de nombreux scénarios dans lesquels il est essentiel de suivre et de gérer les performances des charges de travail prévisibles. Voici quelques exemples : 
- Identification et ajustement des requêtes les plus coûteuses 
- Test A/B 
- Maintien de performances stables pendant les mises à niveau 
- Identification et amélioration des charges de travail ad hoc 

## <a name="identify-and-tune-expensive-queries"></a>Identifier et ajuster les requêtes coûteuses 

### <a name="identify-longest-running-queries"></a>Identifier les requêtes les plus longues à s’exécuter 
Utilisez la vue [Query Performance Insight](concepts-query-performance-insight.md) dans le portail Azure pour identifier rapidement les requêtes dont l’exécution est la plus longue. En général, ces requêtes ont tendance à consommer une quantité importante de ressources. L’optimisation de vos questions les plus longues à s’exécuter peut améliorer les performances en libérant des ressources qui pourraient être utilisées par d’autres requêtes exécutées sur votre système. 

### <a name="target-queries-with-performance-deltas"></a>Cibler les requêtes avec des différences de performances 
Le Magasin des requêtes découpe les données de performances en fenêtres de temps pour vous permettre de suivre les performances d’une requête. Vous pouvez ainsi identifier exactement quelles requêtes contribuent à une augmentation du temps total passé. Par conséquent, vous pouvez effectuer un dépannage ciblé de votre charge de travail.

### <a name="tuning-expensive-queries"></a>Ajustement des requêtes coûteuses 
Quand vous identifiez une requête dont les performances ne sont pas optimales, l’action à entreprendre dépend de la nature du problème : 
- Utilisez les [Recommandations sur les performances](concepts-performance-recommendations.md) pour déterminer s’il existe des propositions d’index. Si c’est le cas, créez l’index, puis utilisez ensuite le Magasin des requêtes pour évaluer les performances des requêtes. 
- Vérifiez que les statistiques sont à jour pour les tables sous-jacentes utilisées par la requête.
- Envisagez de réécrire les requêtes coûteuses. Par exemple, tirez parti du paramétrage des requêtes et réduisez l’utilisation de code SQL dynamique. Implémentez une logique optimale lors de la lecture des données comme l’application de filtrage de données du côté base de données, et pas du côté application. 


## <a name="ab-testing"></a>Test A/B 
Utilisez le Magasin des requêtes pour comparer les performances des charges de travail avant et après un changement d’application que vous prévoyez d’introduire. Exemples de scénarios d’utilisation du Magasin des requêtes pour évaluer l’impact du changement d’environnement ou d’application sur les performances des charges de travail : 
- Déploiement d’une nouvelle version d’une application. 
- Ajout de ressources supplémentaires sur le serveur. 
- Création d’index manquants sur des tables référencées par des requêtes coûteuses. 
 
Dans n’importe lequel de ces scénarios, appliquez le workflow suivant : 
1. Exécutez votre charge de travail avec le Magasin des requêtes avant le changement planifié pour générer une base de référence des performances. 
2. Appliquez un ou plusieurs changements d’application à un moment spécifique. 
3. Continuez à exécuter la charge de travail suffisamment longtemps pour générer l’image de performances du système après le changement. 
4. Comparez les résultats avant et après le changement. 
5. Décidez s’il faut conserver le changement ou revenir en arrière. 


## <a name="identify-and-improve-ad-hoc-workloads"></a>Identifier et améliorer les charges de travail ad hoc 
Certaines charges de travail n’ont pas de requêtes dominantes que vous pouvez ajuster pour améliorer les performances globales de l’application. Ces charges de travail se caractérisent généralement par un nombre relativement important de requêtes uniques, chacune d’elles consommant une partie des ressources système. Chaque requête unique est rarement exécutée, donc sa consommation individuelle du runtime n’est pas critique. En revanche, étant donné que l’application génère en permanence de nouvelles requêtes, une partie significative des ressources système est consacrée à la compilation des requêtes, ce qui n’est pas optimal. En général, cette situation se produit si votre application génère des requêtes (au lieu d’utiliser des procédures stockées ou des requêtes paramétrables), ou si elle s’appuie sur des frameworks de mappage de relationnel objet qui génèrent des requêtes par défaut. 
 
Si vous contrôlez le code d’application, vous pouvez envisager de récrire la couche d’accès aux données pour utiliser des procédures stockées ou des requêtes paramétrables. Toutefois, cette situation peut également être améliorée sans changements d’application en forçant le paramétrage des requêtes pour la totalité de la base de données (toutes les requêtes) ou pour les modèles de requête individuels ayant le même hachage de requête. 

## <a name="next-steps"></a>Étapes suivantes
- Découvrir plus en détail les [bonnes pratiques relatives à l’utilisation du Magasin des requêtes](concepts-query-store-best-practices.md)
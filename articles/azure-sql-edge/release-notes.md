---
title: Notes de publication pour Azure SQL Edge
description: Notes de publication détaillant les nouveautés ou les modifications apportées aux images Azure SQL Edge
keywords: notes de publication SQL Server
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
ms.subservice: ''
author: VasiyaKrishnan
ms.author: vakrishn
ms.reviewer: sstein
ms.date: 11/24/2020
ms.openlocfilehash: bb0c176ee8c145e1268827d79b453628e6f1bc28
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95808625"
---
# <a name="azure-sql-edge-release-notes"></a>Notes de publication Azure SQL Edge 

Cet article décrit les nouveautés et les modifications apportées à chaque nouvelle build d’Azure SQL Edge.

## <a name="azure-sql-edge---101"></a>Azure SQL Edge - 1.0.1

### <a name="sql-engine-build-number---15020001553"></a>Numéro de build du moteur SQL - 15.0.2000.1553

### <a name="whats-new"></a>Nouveautés

- Autoriser l’expression Datebucket définie dans les colonnes calculées

### <a name="fixes"></a>Correctifs

- Correction de la stratégie de rétention pour la suppression de la table avec la stratégie de rétention activée avec un délai d’expiration infini
- Prise en charge dans le déploiement DacFx des fonctionnalités de streaming et de stratégie de rétention 
- Correction du déploiement DacFx pour activer le déploiement à partir d’un dossier imbriqué dans l’URL SAS 
- Correction PREDICT pour prendre en charge les noms de colonnes longs dans les messages d’erreur

## <a name="azure-sql-edge---100-rtm"></a>Azure SQL Edge - 1.0.0 (RTM)

### <a name="sql-engine-build-number---15020001552"></a>Numéro de build du moteur SQL – 15.0.2000.1552

### <a name="whats-new"></a>Nouveautés
1. Images de conteneur basées sur Ubuntu 18.04. 
2. Prise en charge pour `IGNORE NULL` et `RESPECT NULL` syntaxe avec `LAST_VALUE()` et les fonctions `FIRST_VALUE()`. 
3. Améliorations de la fiabilité pour PREDICT avec ONNX.
4. Prise en charge du nettoyage basé sur la stratégie de conservation des données.
   - Prise en charge de la mémoire tampon en anneau pour la tâche de nettoyage de la conservation des données à des fins de dépannage.
5. Prise en charge de nouvelles fonctionnalités 
   - Récupération rapide
   - Réglage automatique des requêtes
   - Scénarios Activer l’exécution parallèle
6. Améliorations de l’économie d’énergie pour le mode alimentation basse
7. Prise en charge en continu de nouvelles fonctionnalités 
   - [Fenêtres d’instantané](/stream-analytics-query/snapshot-window-azure-stream-analytics) : nouveau type de fenêtre permettant de regrouper les événements qui arrivent exactement au même moment. 
   - Activation de [TopOne](/stream-analytics-query/topone-azure-stream-analytics) et [CollectTop](/stream-analytics-query/collecttop-azure-stream-analytics) comme fonction analytique. Cela permet de renvoyer les enregistrements classés par colonne de votre choix, sans qu’ils fassent nécessairement partie d’une fenêtre. 
   - Améliorations apportées à [MATCH_RECOGNIZE](/stream-analytics-query/match-recognize-stream-analytics). 

### <a name="fixes"></a>Correctifs
1. Messages d'erreur supplémentaires et détails pour le dépannage des opérations de diffusion en continu TSQL. 
2. Améliorations visant à préserver l’autonomie de la batterie en mode inactif. 
3. Correctifs du moteur de diffusion en continu TSQL : 
   - Nettoyage de la tâche de diffusion en continu arrêtée 
   - Correctifs pour les améliorations de la localisation et de la gestion Unicode
   - Amélioration du débogage pour Edge TSQL-streaming, permettant aux utilisateurs d’interroger les erreurs d’échec du travail à partir de get_streaming_job.
4. Nettoyage basé sur la stratégie de conservation des données
   - Correctifs pour les scénarios de création et de nettoyage de stratégie de rétention.
5. Correctifs pour les tâches du minuteur d’arrière-plan pour améliorer les économies d’énergie en mode d’alimentation basse.

### <a name="known-issues"></a>Problèmes connus 
1. La fonction T-SQL Date_Bucket ne peut pas être utilisée dans une colonne calculée.


## <a name="ctp-23"></a>CTP 2.3
### <a name="sql-engine-build-number---15020001549"></a>Numéro de build du moteur SQL – 15.0.2000.1549
### <a name="whats-new"></a>Nouveautés
1. Prise en charge des origines personnalisées dans la fonction Date_Bucket (). 
2. Prise en charge des fichiers BacPac dans le cadre du déploiement SQL.
3. Prise en charge du nettoyage basé sur la stratégie de conservation des données.      
   - Prise en charge DDL pour l’activation de la stratégie de rétention 
   - Procédures stockées de nettoyage et tâche de nettoyage en arrière-plan
   - Événements étendus pour surveiller les tâches de nettoyage

### <a name="fixes"></a>Correctifs
1. Messages d'erreur supplémentaires et détails pour le dépannage des opérations de diffusion en continu TSQL. 
2. Améliorations visant à préserver l’autonomie de la batterie en mode inactif. 
3. Correctifs du moteur de diffusion en continu TSQL : 
   - Résoudre le problème de filigrane bloqué avec la fenêtre récurrente sous-diffusée 
   - Corriger la gestion des exceptions de l’infrastructure pour s’assurer qu’elle est collectée en tant qu’erreur utilisateur


## <a name="ctp-22"></a>CTP 2.2
### <a name="sql-engine-build-number---15020001546"></a>Numéro de build du moteur SQL – 15.0.2000.1546
### <a name="whats-new"></a>Nouveautés
1. Prise en charge des conteneurs non racine 
2. Support pour la collecte de données d’utilisation et de diagnostic 
3. Mises à jour de la diffusion en continu T-SQL
   - Prise en charge des caractères Unicode pour les noms d’objets de flux

### <a name="fixes"></a>Correctifs
1. Mises à jour de la diffusion en continu T-SQL
   - Améliorations du nettoyage de processus
   - Améliorations de la journalisation et des diagnostics
2. Amélioration des performances de l’ingestion de données

## <a name="ctp-21"></a>CTP 2.1 
### <a name="sql-engine-build-number---15020001545"></a>Numéro de build du moteur SQL - 15.0.2000.1545
### <a name="fixes"></a>Correctifs
1. Résoudre les modèles PREDICT avec ONNX pour gérer le problème CPUID dans ARM 
2. Correctif pour améliorer la gestion du chemin de défaillance au démarrage de la diffusion en continu TSQL 
3. Corrigez la valeur incorrecte du retard du filigrane dans les mesures du travail en l’absence de données. 
4. Corrigez le problème avec l’adaptateur de sortie lorsque l’adaptateur a un schéma variable entre les lots.  

## <a name="ctp-20"></a>CTP 2.0 
### <a name="sql-engine-build-number---15020001401"></a>Numéro de build du moteur SQL - 15.0.2000.1401
### <a name="whats-new"></a>Nouveautés
1.  Nom du produit mis à jour vers « Azure SQL Edge »
1.  Fonction Date_bucket

    i.  Support pour les types Date, Heure, DateHeure
3.  PREDICT avec ONNX
    
    i.  Paramètre RUNTIME requis pour ONNX 
    
4.  Support de diffusion en continu TSQL (préversion limitée) 
 
### <a name="known-issues"></a>Problèmes connus

1. <b>Problème :</b> Défaillances potentielles liées à l’application de dacpac au démarrage en raison d’un problème de synchronisation.

    <b>Solution de contournement :</b> Si vous redémarrez SQL Server ou le conteneur, l’application de dacpac sera relancée et le problème devra être résolu
### <a name="request-support"></a>Requête d'assistance
1. Vous pouvez demander de l’aide dans la [page de support](https://ms.portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

4. Assurez-vous que les champs suivants sont sélectionnés : 
    * Type de problème : technique 
    * Service : IoT Edge
    * Type de problème : mon problème est lié à un module IoT Edge
    * Sous-type de problème ! Azure SQL Edge

   ![Échantillon de ticket de support](media/get-support/support-ticket.png)

## <a name="ctp-15"></a>CTP 1.5
### <a name="sql-engine-build-number---15020001331"></a>Numéro de build du moteur SQL - 15.0.2000.1331
### <a name="whats-new"></a>Nouveautés
1. Fonction Date_bucket
    
    i. Support pour le type DateTimeOffset
2. Modèles PREDICT avec ONNX

    i. support nvarchar
 
## <a name="ctp-14"></a>CTP 1.4
### <a name="sql-engine-build-number---15020001247"></a>Numéro de build du moteur SQL - 15.0.2000.1247
### <a name="whats-new"></a>Nouveautés
1.  Modèles PREDICT avec ONNX
 
    i.  Support varchar
    
    ii. Migration vers la version 1.0 du runtime ONNX 
2.  Support des fonctionnalités : les fonctionnalités suivantes sont désactivées :

    i.   Support CDC

    ii.  Table d’historique avec compression

    iii. Facteur d’échelle supérieur pour la lecture anticipée des journaux

    iv.  Pushdown des filtres ES en mode batch

    v.   Optimisations de lecture anticipée
 
## <a name="ctp-13"></a>CTP 1.3
### <a name="sql-engine-build-number---15020001147"></a>Numéro de build du moteur SQL - 15.0.2000.1147
### <a name="whats-new"></a>Nouveautés
1. Déploiement du Portail Azure IoT 

    i.   Support du déploiement d’images AMD64 et ARM

    ii.  Support de la création de travaux de diffusion en continu

    iii. Déploiement dacpac
2. Modèles PREDICT avec ONNX

    i. Supporte des types numériques
3. Support des fonctionnalités : les fonctionnalités suivantes sont désactivées :

    i.  Agrégat de pushdown pour l’analyse des banques de colonnes

    ii. Analyses de carrousel
4. Travail de réduction de l’empreinte et de la consommation de mémoire

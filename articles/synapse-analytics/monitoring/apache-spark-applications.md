---
title: Surveiller des applications Apache Spark à l’aide de Synapse Studio
description: Utilisez Synapse Studio pour surveiller vos applications Apache Spark.
services: synapse-analytics
author: matt1883
ms.service: synapse-analytics
ms.topic: how-to
ms.subservice: monitoring
ms.date: 04/15/2020
ms.author: mahi
ms.reviewer: mahi
ms.openlocfilehash: 16b37258d922db59f520d4e30c45773f6d1108cf
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93341038"
---
# <a name="use-synapse-studio-to-monitor-your-apache-spark-applications"></a>Utiliser Synapse Studio pour surveiller vos applications Apache Spark

Azure Synapse Analytics vous permet d’utiliser Apache Spark pour exécuter des blocs-notes, des travaux et d’autres types d’applications sur vos pools Apache Spark dans votre espace de travail.

Cet article explique comment surveiller vos applications Apache Spark, et ainsi garder un œil sur leur état, leurs problèmes et leur progression.

Ce tutoriel décrit les tâches suivantes :

* Superviser l’exécution des applications Apache Spark
* Afficher les applications Apache Spark terminées
* Afficher les applications Apache Spark annulées
* Déboguer les applications Apache Spark ayant échoué

## <a name="prerequisites"></a>Prérequis

Avant de commencer le didacticiel, veillez à disposer des éléments suivants :

- Un espace de travail Synapse Studio. Pour obtenir des instructions, consultez [Créer un espace de travail Synapse Studio](https://docs.microsoft.com/azure/machine-learning/how-to-manage-workspace#create-a-workspace).

- Un pool Apache Spark.

## <a name="view-apache-spark-applications"></a>Afficher les applications Apache Spark 
Vous pouvez afficher toutes les applications Apache Spark à partir de **Superviser** -> **Applications Apache Spark**.
   ![applications Apache Spark](./media/how-to-monitor-spark-applications/apache-spark-applications.png)

## <a name="view-completed-apache-spark-application"></a>Afficher une application Apache Spark terminée

Ouvrez **Superviser** , puis sélectionnez **Applications Apache Spark**. Pour afficher les détails des applications Apache Spark terminées, sélectionnez l’application Apache Spark concernée.

  ![sélectionner la tâche terminée](./media/how-to-monitor-spark-applications/select-completed-job.png)

1. Consultez **Tâches terminées** , **Statut** et **Durée totale**.

2. Actualisez la requête de journal.

3. Ouvrez le lien vers le serveur d’historique Apache Spark en cliquant sur **Serveur d’historique Spark**.

4. Vérifiez les informations dans **Résumé**.

5. Consultez les **Journaux**. Vous pouvez sélectionner différents types de journaux dans la liste déroulante, télécharger les informations du journal en cliquant sur **Télécharger les journaux** et activer la case à cocher **Filtrer les erreurs et les avertissements** pour filtrer les erreurs et les avertissements dont vous avez besoin.

6. Vous pouvez voir une vue d’ensemble de votre travail dans le graphique du travail ainsi généré. Par défaut, le graphique affiche tous les travaux. Vous pouvez filtrer cet affichage par **ID de travail**.

7. Par défaut, l’affichage **Progress** (Progression) est sélectionné. Vous pouvez vérifier le flux de données en sélectionnant **Progression**/**Lecture**/**Écriture**/**Durée** dans le menu déroulant **Affichage**.

8. Pour lire le travail, cliquez sur le bouton **Lire**. Vous pouvez cliquer sur le bouton **Arrêter** à tout moment pour arrêter.

9. Utilisez le défilement de la souris ou la barre de défilement pour effectuer un zoom avant et un zoom arrière sur le graphique des travaux. Vous pouvez également sélectionner **Zoom ajusté** pour l’ajuster à l’écran.

10. Le nœud de graphique du travail affiche les informations suivantes de chaque phase :

    * ID.

    * Nom ou description.

    * Nombre total de tâches.

    * Lecture de données : somme de la taille d’entrée et de la taille de lecture aléatoire.

    * Écriture de données : somme de la taille de sortie et de la taille des écritures aléatoires.

    * Durée d’exécution : durée entre l’heure de début de la première tentative et l’heure de fin de la dernière tentative.

    * Nombre de lignes : somme des enregistrements d’entrée, des enregistrements de sortie, des enregistrements de lecture aléatoire et des enregistrements d’écriture aléatoire.

    * Avancement.

     ![afficher la tâche terminée](./media/how-to-monitor-spark-applications/view-completed-job.png)
    
11. Cliquez sur **Afficher les détails** dans le graphique pour que les détails de l’index s’affichent.

    ![détails de l’index](./media/how-to-monitor-spark-applications/details-for-stage.png)


## <a name="monitor-running-apache-spark-application"></a>Superviser l’exécution d’applications Apache Spark

Ouvrez **Superviser** , puis sélectionnez **Applications Apache Spark**. Pour afficher les détails des applications Apache Spark en cours d’exécution, sélectionnez l’application Apache Spark concernée et affichez les détails. Si l’application Apache Spark est toujours en cours d’exécution, vous pouvez surveiller la progression.

   ![sélectionner une tâche en cours d’exécution](./media/how-to-monitor-spark-applications/select-running-job.png)

1. Consultez **Tâches terminées** , **Statut** et **Durée totale**.

2. **Annulez** l’exécution de l’application Apache Spark.

3. **Actualisez** la requête de journal.

4. Cliquez sur le bouton **Interface utilisateur Spark** pour accéder à la page Travail Spark.

5. Affichez le graphique. Vous pouvez voir une vue d’ensemble de votre travail dans le graphique du travail ainsi généré. Reportez-vous aux étapes 6, 7, 8, 9 et 10 de la section [Afficher une application Apache Spark terminée](#view-completed-apache-spark-application).

6. Vérifiez les informations dans **Résumé**.

7. Vérifiez les diagnostics dans l’onglet **Diagnostic**.

8. Vérifiez les **journaux** dans cet onglet. Vous pouvez sélectionner différents types de journaux dans la liste déroulante, télécharger les informations du journal en cliquant sur **Télécharger les journaux** et activer la case à cocher **Filtrer les erreurs et les avertissements** pour filtrer les erreurs et les avertissements dont vous avez besoin.

    ![afficher la tâche en cours d’exécution](./media/how-to-monitor-spark-applications/view-running-job.png)

## <a name="view-canceled-apache-spark-application"></a>Afficher une application Apache Spark annulée

Ouvrez **Superviser** , puis sélectionnez **Applications Apache Spark**. Pour afficher les détails des applications Apache Spark annulées, sélectionnez l’application Apache Spark concernée.

 ![sélectionner une tâche annulée](./media/how-to-monitor-spark-applications/select-cancelled-job.png) 

1. Consultez **Tâches terminées** , **Statut** et **Durée totale**.

2. Actualisez la requête de journal.

3. Ouvrez le lien vers le serveur d’historique Apache en cliquant sur **Serveur d’historique Spark**.

4. Affichez le graphique. Vous pouvez voir une vue d’ensemble de votre travail dans le graphique du travail ainsi généré. Reportez-vous aux étapes 6, 7, 8, 9 et 10 de la section [Afficher une application Apache Spark terminée](#view-completed-apache-spark-application).

5. Vérifiez les informations dans **Résumé**.

6. Consultez les **Journaux**. Vous pouvez sélectionner différents types de journaux dans la liste déroulante, télécharger les informations du journal en cliquant sur **Télécharger les journaux** et activer la case à cocher **Filtrer les erreurs et les avertissements** pour filtrer les erreurs et les avertissements dont vous avez besoin.

7. Cliquez sur **Afficher les détails** dans le graphique pour que les détails de l’index s’affichent.

   ![afficher une tâche annulée](./media/how-to-monitor-spark-applications/view-cancelled-job.png)

## <a name="debug-failed-apache-spark-application"></a>Déboguer une application Apache Spark défaillante

Ouvrez **Superviser** , puis sélectionnez **Applications Apache Spark**. Pour afficher les détails des applications Apache Spark ayant échoué, sélectionnez l’application Apache Spark concernée.

![sélectionner une tâche ayant échoué](./media/how-to-monitor-spark-applications/select-failed-job.png)

1. Consultez **Tâches terminées** , **Statut** et **Durée totale**.

2. Actualisez la requête de journal.

3. Ouvrez le lien vers le serveur d’historique Apache Spark en cliquant sur **Serveur d’historique Spark**.

4. Affichez le graphique. Vous pouvez voir une vue d’ensemble de votre travail dans le graphique du travail ainsi généré. Reportez-vous aux étapes 6, 7, 8, 9 et 10 de la section [Afficher une application Apache Spark terminée](#view-completed-apache-spark-application).

5. Vérifiez les informations dans **Résumé**.

6. Consultez les informations sur l’erreur.

   ![informations sur la tâche ayant échoué](./media/how-to-monitor-spark-applications/failed-job-info.png)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la surveillance des exécutions de pipeline, consultez l’article [Surveiller les exécutions de pipeline à l’aide de Synapse Studio](how-to-monitor-pipeline-runs.md).  

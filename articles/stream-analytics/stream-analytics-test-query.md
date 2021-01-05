---
title: Tester une tâche Azure Stream Analytics avec des exemples de données
description: Cet article explique comment utiliser le portail Azure pour tester une tâche Azure Stream Analytics, des exemples d’entrées et charger des exemples de données.
author: mamccrea
ms.author: mamccrea
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 3/6/2020
ms.custom: seodec18
ms.openlocfilehash: 8e08c4c34495b58c105560dba9d818be9ebf5e34
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94490965"
---
# <a name="test-an-azure-stream-analytics-job-in-the-portal"></a>Tester une tâche Azure Stream Analytics sur le portail

Dans Azure Stream Analytics, vous pouvez tester votre requête sans démarrer ou arrêter votre travail. Vous pouvez tester des requêtes sur des données entrantes de vos sources de diffusion en continu ou charger des exemples de données à partir d'un fichier local sur le portail Azure. Vous pouvez également tester les requêtes localement à partir de vos exemples de données locaux ou de vos données actives dans [Visual Studio](stream-analytics-live-data-local-testing.md) et [Visual Studio Code](visual-studio-code-local-run-live-input.md).

## <a name="automatically-sample-incoming-data-from-input"></a>Échantillonner automatiquement les données entrantes à partir d'une entrée

Azure Stream Analytics récupère automatiquement les événements à partir de vos entrées de diffusion en continu. Vous pouvez exécuter des requêtes sur l’exemple par défaut ou définir un délai d’exécution spécifique pour l’exemple.

1. Connectez-vous au portail Azure.

2. Localisez votre tâche Stream Analytics existante et sélectionnez-la.

3. Dans la page de la tâche Stream Analytics, sous le titre **Topologie de la tâche**, sélectionnez **Requête** pour ouvrir la fenêtre Éditeur de requête. 

4. Pour afficher un exemple de liste d’événements entrants, sélectionnez l’icône d’entrée avec un fichier et les exemples d’événements s’affichent automatiquement dans **l’aperçu d’entrée**.

   a. Le type de sérialisation de vos données est automatiquement détecté (JSON ou CSV). Vous pouvez le modifier manuellement en JSON, CSV, AVRO, en modifiant l’option dans le menu déroulant.
    
   b. Utilisez le sélecteur pour afficher vos données au format **Tableau** ou **Brut**.
    
   c. Si les données qui apparaissent ne sont pas à jour, sélectionnez **Actualiser** pour afficher les événements les plus récents.

   Voici un exemple de données au **format tableau** :

   ![Exemple d’entrée Azure Stream Analytics au format tableau](./media/stream-analytics-test-query/asa-sample-table.png)

   Voici un exemple de données au **format brut** :

   ![Exemple d’entrée Azure Stream Analytics au format brut](./media/stream-analytics-test-query/asa-sample-raw.png)

5. Pour tester votre requête avec des données entrantes, sélectionnez **Tester la requête**. Les résultats s’affichent dans l’onglet **Résultats du test**. Vous pouvez également sélectionner **Télécharger les résultats** pour télécharger les résultats.

   ![Exemple de résultats de test de requête Azure Stream Analytics](./media/stream-analytics-test-query/asa-test-query.png)

6. Pour tester votre requête sur un intervalle de temps spécifique d’événements entrants, sélectionnez **Sélectionner un intervalle de temps**.
   
   ![Intervalle de temps Azure Stream Analytics pour un exemple d’événements entrants](./media/stream-analytics-test-query/asa-select-time-range.png)

7. Définissez l’intervalle de temps des événements que vous souhaitez utiliser pour tester votre requête et sélectionnez **Exemple**. Dans cet intervalle de temps, vous pouvez récupérer jusqu’à 1 000 événements ou 1 Mo, selon ce qui se produit en premier.

   ![Intervalle de temps Azure Stream Analytics défini pour un exemple d’événements entrants](./media/stream-analytics-test-query/asa-set-time-range.png)

8. Une fois les événements échantillonnés pour l’intervalle de temps sélectionné, ils s’affichent dans l’onglet **Aperçu d’entrée**.

   ![Afficher les résultats du test Azure Stream Analytics](./media/stream-analytics-test-query/asa-view-test-results.png)

9. Sélectionnez **Réinitialiser** pour voir l’exemple de liste d’événements entrants. Si vous sélectionnez **Réinitialiser**, votre sélection d’intervalle de temps sera perdue. Sélectionnez **Tester la requête** pour tester votre requête et passez en revue les résultats dans l’onglet **Résultats du test**.

10. Lorsque vous apportez des modifications à votre requête, sélectionnez **Enregistrer la requête** pour tester la nouvelle logique de requête. Cela vous permet de modifier de manière itérative votre requête et de la tester à nouveau pour voir comment la sortie change.

11. Après avoir vérifié les résultats affichés dans le navigateur, vous êtes prêt à **Démarrer** le travail.

## <a name="upload-sample-data-from-a-local-file"></a>Charger un exemple de données à partir d’un fichier local

Au lieu d’utiliser des données actives, vous pouvez utiliser des exemples de données à partir d’un fichier local pour tester votre requête Azure Stream Analytics.

1. Connectez-vous au portail Azure.
   
2. Localisez votre tâche Stream Analytics existante et sélectionnez-la.

3. Dans la page de la tâche Stream Analytics, sous le titre **Topologie de la tâche**, sélectionnez **Requête** pour ouvrir la fenêtre Éditeur de requête.

4. Pour tester votre requête avec un fichier local, sélectionnez **Charger un exemple d’entrée** dans l’onglet **Aperçu d’entrée**. 

   ![Capture d’écran montrant l’option Charger un exemple d’entrée.](./media/stream-analytics-test-query/asa-upload-sample-file.png)

5. Chargez votre fichier local pour tester la requête. Vous pouvez uniquement charger des fichiers au format JSON, CSV ou AVRO. Sélectionnez **OK**.

   ![Capture d’écran montrant la boîte de dialogue Charger un exemple de données dans laquelle vous pouvez sélectionner un fichier.](./media/stream-analytics-test-query/asa-upload-sample-json-file.png)

6. Dès que vous chargez le fichier, vous pouvez également voir le contenu du fichier sous la forme d’un tableau ou au format brut. Si vous sélectionnez **Réinitialiser**, les exemples de données retournent vers les données d’entrée entrantes expliquées dans la section précédente. Vous pouvez charger n’importe quel autre fichier pour tester la requête à tout moment.

7. Sélectionnez **Tester la requête** pour tester votre requête par rapport à l’exemple de fichier chargé.

8. Les résultats du test sont affichés en fonction de votre requête. Vous pouvez modifier votre requête et sélectionner **Enregistrer la requête** pour tester la nouvelle logique de requête. Cela vous permet de modifier de manière itérative votre requête et de la tester à nouveau pour voir comment la sortie change.

9. Lorsque vous utilisez plusieurs sorties dans la requête, les résultats sont affichés en fonction de la sortie sélectionnée. 

   ![Sortie sélectionnée Azure Stream Analytics](./media/stream-analytics-test-query/asa-sample-test-selected-output.png)

10. Après avoir vérifié les résultats affichés dans le navigateur, vous pouvez **Démarrer** le travail.

## <a name="limitations"></a>Limites

1.  La stratégie de temps n’est pas prise en charge dans le test du portail :

    * Désordre : tous les événements entrants seront classés.
    * Arrivée tardive : il n’y aura pas d’événement d’arrivée tardive puisque Stream Analytics ne peut utiliser que des données existantes pour le test.
   
2.  Une fonction définie par l’utilisateur en C# n’est pas prise en charge.

3.  Tous les tests sont exécutés avec un travail disposant d’une seule unité de streaming.

4.  La durée du délai d’expiration est d’une minute. Par conséquent, toute requête avec une taille de fenêtre supérieure à une minute ne peut obtenir aucune donnée.

5.  Le machine learning n’est pas pris en charge.

6. L’exemple d’API de données est limité après cinq requêtes exécutées dans une fenêtre de 15 minutes. À l’issue de la fenêtre de 15 minutes, vous pouvez exécuter d’autres exemples de requêtes de données. Cette limitation s’applique au niveau de l’abonnement.

## <a name="troubleshooting"></a>Dépannage

1.  Si vous recevez l’erreur : « Un problème de connectivité réseau s’est produit lors de l’extraction des résultats. Vérifiez les paramètres de votre réseau et de votre pare-feu. », suivez les étapes ci-dessous :

  * Pour vérifier la connexion au service, ouvrez [https://queryruntime.azurestreamanalytics.com/api/home/index](https://queryruntime.azurestreamanalytics.com/api/home/index) dans un navigateur. Si vous ne pouvez pas ouvrir ce lien, mettez à jour les paramètres de votre pare-feu.
  
2. Si vous recevez l’erreur : « La taille de la requête est trop grande. Réduisez la taille des données d’entrée, puis réessayez. », suivez les étapes ci-dessous :

  * Réduisez la taille d’entrée : Testez votre requête avec un exemple de fichier de taille inférieure ou sur un intervalle de temps plus court.
  * Réduisez la taille de la requête : Pour tester une sélection de la requête, sélectionnez-en une partie, puis cliquez sur **Tester la requête sélectionnée**.


## <a name="next-steps"></a>Étapes suivantes
* [Créer une solution IoT à l'aide de Stream Analytics](./stream-analytics-build-an-iot-solution-using-stream-analytics.md) : ce tutoriel vous aidera à créer une solution de bout en bout avec un générateur de données qui simulera le trafic à un poste de péage.

* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)

* [Exemples de requête pour les modes d’utilisation courants dans Stream Analytics](stream-analytics-stream-analytics-query-patterns.md)

* [Comprendre les entrées d’Azure Stream Analytics](stream-analytics-add-inputs.md)

* [Comprendre les sorties d’Azure Stream Analytics](stream-analytics-define-outputs.md)

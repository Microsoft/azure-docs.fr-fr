---
title: Travaux de périphérie Azure Stream Analytics dans Visual Studio
description: Cet article décrit comment écrire, créer et déboguer des travaux Stream Analytics sur IoT Edge avec les outils Stream Analytics pour Visual Studio.
author: su-jie
ms.author: sujie
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/07/2018
ms.custom: seodec18
ms.openlocfilehash: 55ff983169e15c74bf343993b66088932a538c36
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93127516"
---
# <a name="develop-stream-analytics-edge-jobs-using-visual-studio-tools"></a>Développer des tâches Edge Stream Analytics avec les outils Visual Studio

Dans ce tutoriel, découvrez comment utiliser les outils Stream Analytics pour Visual Studio. Vous apprenez à écrire, créer et déboguer vos travaux Stream Analytics Edge. Après avoir créé et testé le travail, vous pourrez accéder au portail Azure pour le déployer sur vos appareils. 

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce didacticiel, vous avez besoin des prérequis suivants :

* Installez [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/), [Visual Studio 2015](https://www.visualstudio.com/vs/older-downloads/) ou [Visual Studio 2013 Update 4](https://www.microsoft.com/download/details.aspx?id=45326). Les éditions Enterprise (Ultimate/Premium), Professional et Community sont prises en charge. L’édition Express n’est pas prise en charge.  

* Suivez les [instructions d’installation](stream-analytics-tools-for-visual-studio-edge-jobs.md) pour installer les outils Stream Analytics pour Visual Studio.
 
## <a name="create-a-stream-analytics-edge-project"></a>Créer un projet Stream Analytics Edge 

Dans Visual Studio, sélectionnez **Fichier** > **Nouveau** > **Projet**. Accédez à la liste **Modèles** à gauche > développez **Azure Stream Analytics** > **Stream Analytics Edge** > **Application Azure Stream Analytics Edge**. Entrez le nom, l’emplacement et le nom de la solution pour votre projet, puis sélectionnez **OK**.

![Nouveau projet Stream Analytics Edge dans Visual Studio](./media/stream-analytics-tools-for-visual-studio-edge-jobs/new-stream-analytics-edge-project.png)

Une fois que vous avez créé le projet, accédez à **l’Explorateur de solutions** pour afficher l’arborescence des dossiers.

![Vue du travail de périphérie Stream Analytics dans l’Explorateur de solutions](./media/stream-analytics-tools-for-visual-studio-edge-jobs/edge-project-in-solution-explorer.png)

 
## <a name="choose-the-correct-subscription"></a>Choisir le bon abonnement

1. Dans le menu **Affichage** de Visual Studio, sélectionnez **Explorateur de serveurs**.  

2. Cliquez avec le bouton droit sur **Azure**  > Sélectionnez **Se connecter à un abonnement Microsoft Azure**  > et connectez-vous avec votre compte Azure.

## <a name="define-inputs"></a>Définir les entrées

1. Dans **l’Explorateur de solutions** , développez le nœud **Entrées** pour afficher l’entrée **EdgeInput.json**. Double-cliquez sur cette sortie pour afficher ses paramètres.  

2. Définissez le type de source sur **Flux de données**. Ensuite, définissez la source sur **Edge Hub** , le format de sérialisation d’événement sur **Json** et l’encodage sur **UTF8**. Pour les besoins de cet exemple, laissez l’alias indiqué dans **Alias d’entrée**. Cet alias peut toutefois être renommé. Si vous renommez l’alias d’entrée, utilisez le nom que vous avez spécifié au moment de la définition de la requête. Sélectionnez **Enregistrer** pour enregistrer les paramètres.  
   ![Configuration d’une entrée de travail Stream Analytics](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-input-configuration.png)
 


## <a name="define-outputs"></a>Définir les sorties

1. Dans **l’Explorateur de solutions** , développez le nœud **Sorties** pour afficher la sortie **EdgeOutput.json**. Double-cliquez sur cette sortie pour afficher ses paramètres.  

2. Veillez à définir le Récepteur pour sélectionner **Edge Hub** , puis définissez le format de sérialisation d’événement sur **Json** , la valeur d’encodage sur **UTF8** et le format sur **Tableau**. Pour les besoins de cet exemple, laissez l’alias indiqué dans **Alias de sortie**. Cet alias peut toutefois être renommé. Si vous renommez l’alias de sortie, utilisez le nom que vous avez spécifié au moment de la définition de la requête. Sélectionnez **Enregistrer** pour enregistrer les paramètres. 
   ![Configuration d’une sortie de travail Stream Analytics](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-output-configuration.png)
 
## <a name="define-the-transformation-query"></a>Définir la requête de transformation

Les travaux Stream Analytics déployés dans les environnements Stream Analytics IoT Edge prennent en charge la plupart des [références du langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference?f=255&MSPPError=-2147217396). Toutefois, les opérations suivantes ne sont pas encore prises en charge pour les travaux Stream Analytics Edge : 


|**Catégorie**  | **Commande**  |
|---------|---------|
|Autres opérateurs | <ul><li>PARTITION BY</li><li>TIMESTAMP BY OVER</li><li>UDF JavaScript</li><li>Agrégats définis par l’utilisateur – (UDA)</li><li>GetMetadataPropertyValue</li><li>Utilisation de plus de 14 agrégats dans une seule étape</li></ul>   |

Quand vous créez un travail Stream Analytics Edge dans le portail, le compilateur affiche automatiquement un avertissement si vous utilisez un opérateur non pris en charge.

Dans Visual Studio, définissez la requête de transformation suivante dans l’éditeur de requête ( **fichier script.asaql** )

```sql
SELECT * INTO EdgeOutput
FROM EdgeInput 
```

## <a name="test-the-job-locally"></a>Tester le travail localement

Pour tester la requête localement, vous devez charger les exemples de données. Pour obtenir les exemples de données, téléchargez les données d’inscription à partir du [référentiel GitHub](https://github.com/Azure/azure-stream-analytics/blob/master/Sample%20Data/Registration.json). Enregistrez-les sur votre ordinateur local. 

1. Pour charger les exemples de données, cliquez avec le bouton droit sur le fichier **EdgeInput.json** et choisissez **Ajouter une entrée locale**  

2. Dans la fenêtre indépendante qui s’affiche > sélectionnez **Parcourir** pour accéder aux exemples de données à partir de votre chemin local > sélectionnez **Enregistrer**.
   ![Configuration d’une entrée locale dans Visual Studio](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-local-input-configuration.png)
 
3. Un fichier nommé **local_EdgeInput.json** est automatiquement ajouté à votre dossier d’entrées.  
4. Vous pouvez l’exécuter localement ou l’envoyer sur Azure. Pour tester la requête, sélectionnez **Exécuter localement**.  
   ![Options d’exécution de travail Stream Analytics dans Visual Studio](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-visual-stuidio-run-options.png)
 
5. La fenêtre d’invite de commandes affiche l’état du travail. Quand le travail s’exécute correctement, un dossier du type « 2018-02-23-11-31-42 » est créé dans le chemin du dossier de projet « Visual Studio 2015\Projects\MyASAEdgejob\MyASAEdgejob\ASALocalRun\2018-02-23-11-31-42 ». Accédez à l’emplacement du dossier pour afficher les résultats dans le dossier local :

   Vous pouvez également vous connecter au portail Azure pour vérifier que le travail a bien été créé. 

   ![Dossier de résultat du travail Stream Analytics](./media/stream-analytics-tools-for-visual-studio-edge-jobs/stream-analytics-job-result-folder.png)

## <a name="submit-the-job-to-azure"></a>Envoyer le travail sur Azure

1. Avant d’envoyer le travail sur Azure, vous devez vous connecter à votre abonnement Azure. Ouvrez **l’Explorateur de serveurs** > cliquez avec le bouton droit sur **Azure** > **Se connecter à un abonnement Microsoft Azure** > connectez-vous à votre abonnement Azure.  

2. Pour envoyer le travail sur Azure, accédez à l’éditeur de requête > sélectionnez **Envoyer sur Azure**.  

3. Une fenêtre contextuelle s’ouvre. Choisissez de mettre à jour un travail Stream Analytics Edge existant, ou créez-en un. Quand vous mettez à jour un travail existant, le travail mis à jour remplace la configuration entière du travail existant. Dans ce scénario, vous allez créer un travail. Sélectionnez **Créer un travail Azure Stream Analytics** > entrez un nom pour votre travail (par exemple, **MyASAEdgeJob** ) > choisissez les paramètres **Abonnement** , **Groupe de ressources** et **Emplacement** appropriés > sélectionnez **Envoyer**.

   ![Envoyer votre travail Stream Analytics sur Azure à partir de Visual Studio](./media/stream-analytics-tools-for-visual-studio-edge-jobs/submit-stream-analytics-job-to-azure.png)
 
   Votre travail Stream Analytics Edge est créé. Vous pouvez maintenant utiliser le tutoriel [Exécuter des travaux sur IoT Edge](stream-analytics-edge.md) pour apprendre à déployer ce travail sur vos appareils. 

## <a name="manage-the-job"></a>Gérer le travail 

Vous pouvez afficher l’état et le diagramme du travail à partir de l’Explorateur de serveurs. À partir de **Stream Analytics** dans l’ **Explorateur de serveurs** , développez l’abonnement et le groupe de ressources où vous avez déployé le travail Stream Analytics Edge. Vous pouvez voir le travail MyASAEdgejob avec l’état **Créé**. Développez le nœud du travail et double-cliquez dessus pour afficher la vue du travail.

![Options de gestion du travail dans l’Explorateur de serveurs](./media/stream-analytics-tools-for-visual-studio-edge-jobs/server-explorer-options.png)
 
Dans la fenêtre de vue du travail, vous pouvez effectuer des opérations comme actualiser le travail, le supprimer et l’ouvrir à partir du portail Azure.

![Diagramme du travail et autres options dans Visual Studio](./media/stream-analytics-tools-for-visual-studio-edge-jobs/job-diagram-and-other-options.png) 

## <a name="next-steps"></a>Étapes suivantes

* [Plus d’informations sur Azure Iot Edge](../iot-edge/about-iot-edge.md)
* [Didacticiel pour ASA sur IoT Edge](../iot-edge/tutorial-deploy-stream-analytics.md)
* [Envoyer des commentaires à l’équipe à l’aide de ce questionnaire](https://forms.office.com/Pages/ResponsePage.aspx?id=v4j5cvGGr0GRqy180BHbR2czagZ-i_9Cg6NhAZlH9ypUMjNEM0RDVU9CVTBQWDdYTlk0UDNTTFdUTC4u)
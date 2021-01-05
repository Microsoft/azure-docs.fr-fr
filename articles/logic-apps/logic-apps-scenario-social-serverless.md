---
title: Créer un tableau de bord d’insights client
description: Gérer des retours d’expérience client, des données de médias sociaux, etc., en générant un tableau de bord client avec Azure Logic Apps et Azure Functions
services: logic-apps
ms.suite: integration
author: jeffhollan
ms.author: jehollan
ms.reviewer: estfan, logicappspm
ms.topic: article
ms.date: 03/15/2018
ms.openlocfilehash: 2fae7d2526e6c95efe83ca8fa742a6d92457b897
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86520747"
---
# <a name="create-a-streaming-customer-insights-dashboard-with-azure-logic-apps-and-azure-functions"></a>Créer un tableau de bord des insights client en streaming avec Azure Logic Apps et Azure Functions

Azure offre des outils [serverless](https://azure.microsoft.com/solutions/serverless/) qui permettent de générer et d’héberger rapidement des applications dans le cloud, sans avoir à réfléchir à l’infrastructure. Dans ce tutoriel, vous pouvez créer un tableau de bord qui se déclenche lors de retours d’expérience client, analyse le retour avec le Machine Learning et publie des insights dans une source telle que Power BI ou Azure Data Lake.

Pour cette solution, utilisez ces composants Azure clés pour les applications serverless : [Azure Functions](https://azure.microsoft.com/services/functions/) et [Azure Logic Apps](https://azure.microsoft.com/services/logic-apps/).
Azure Logic Apps fournit un moteur de flux de travail serverless dans le cloud afin que vous puissiez créer des orchestrations entre les composants serverless et vous connecter à plus de 200 services et API. Azure Functions offre un computing serverless dans le cloud. Cette solution utilise Azure Functions pour le marquage des tweets client en fonction de mots clés prédéfinis.

Dans ce scénario, vous créez une application logique qui déclenche la recherche des retours d’expérience des clients. Voici certains des connecteurs qui peuvent aider à réagir aux retours d’expérience client, y compris Outlook.com, Office 365, Survey Monkey, Twitter et une [requête HTTP à partir d’un formulaire Web](/archive/blogs/logicapps/calling-a-logic-app-from-an-html-form). Le flux de travail que vous créez analyse un hashtag sur Twitter.

Vous pouvez [générer la solution entière dans Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md) et [déployer la solution avec le modèle Azure Resource Manager](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md). Pour obtenir une procédure pas à pas vidéo qui montre comment créer cette solution, [regardez cette vidéo de Channel 9](https://aka.ms/logicappsdemo). 

## <a name="trigger-on-customer-data"></a>Déclencheur sur les données client

1. Dans le portail Azure ou Visual Studio, créez une application logique vide. 

   Si vous êtes débutant avec les applications logiques, consultez le [démarrage rapide pour le portail Azure](../logic-apps/quickstart-create-first-logic-app-workflow.md) ou le [démarrage rapide pour Visual Studio](../logic-apps/quickstart-create-logic-apps-with-visual-studio.md).

2. Dans le Concepteur d’application logique, recherchez et ajoutez le déclencheur Twitter qui a cette action : **When a new tweet is posted** (Quand un nouveau tweet est publié)

3. Configurez le déclencheur pour écouter les tweets en fonction d’un mot-clé ou d’un hashtag.

   Sur les déclencheurs basés sur l’interrogation, tel que le déclencheur Twitter, la propriété de périodicité détermine la fréquence à laquelle l’application logique recherche de nouveaux éléments.

   ![Exemple de déclencheur Twitter][1]

Cette application logique se déclenche désormais sur tout nouveau tweet. Vous pouvez prendre et analyser les données du tweet afin de mieux comprendre les sentiments exprimés. 

## <a name="analyze-tweet-text"></a>Analyser le texte d’un tweet

Pour détecter le sentiment derrière un texte, vous pouvez utiliser [Azure Cognitive Services](https://azure.microsoft.com/services/cognitive-services/).

1. Dans le Concepteur d’application logique, sous le déclencheur, sélectionnez **Nouvelle étape**.

2. Recherchez le connecteur **Analyse de texte**.

3. Sélectionnez l’action **Detect Sentiment (Détecter le sentiment)** .

4. Si vous y êtes invité, fournissez une clé Cognitive Services valide pour le service d’analyse de texte.

5. Sous **Corps de la demande**, sélectionnez le champ **Texte du Tweet**, qui fournit le texte du tweet comme entrée pour l’analyse.

Une fois que vous obtenez les données et les insights du tweet, vous pouvez utiliser plusieurs autres connecteurs appropriés et leurs actions :

* **Power BI - Ajouter des lignes au jeu de données de streaming** : Affichez des tweets entrants dans un tableau de bord Power BI.
* **Azure Data Lake - Ajouter un fichier** : ajoutez des données client à un jeu de données Azure Data Lake à inclure dans des travaux analytiques.
* **SQL - Ajouter des lignes** : stockez des données dans une base de données pour les récupérer ultérieurement.
* **Slack - Envoyer un message** : alertez un canal Slack en cas de retours d’expérience négatifs nécessitant des actions.

Vous pouvez également créer un Azure Functions afin de pouvoir effectuer un traitement personnalisé sur vos données. 

## <a name="process-data-with-azure-functions"></a>Traiter des données avec Azure Functions

Avant de créer une fonction, créez une application de fonctions dans votre abonnement Azure. En outre, pour que votre application logique appelle directement une fonction, la fonction doit avoir une liaison de déclencheur HTTP, par exemple, utilisez le modèle **HttpTrigger**. Apprenez [à créer votre première application de fonctions et fonction dans le portail Azure](../azure-functions/functions-create-first-azure-function.md).

Dans ce scénario, utilisez le texte du tweet comme corps de la demande pour votre Fonction Azure. Dans votre code de fonction, définissez la logique qui détermine si le texte du tweet contient un mot clé ou une phrase. Conservez la fonction aussi simple ou complexe que nécessaire pour le scénario.
À la fin de la fonction, renvoyez une réponse à l’application logique avec certaines données, par exemple, une valeur booléenne simple telle que `containsKeyword` ou un objet complexe.

> [!TIP]
> Pour accéder à une réponse complexe à partir d’une fonction dans une application logique, utilisez l’action **Analyser JSON**.

Lorsque vous avez terminé, enregistrez la fonction, puis ajoutez-la en tant qu’action dans l’application logique que vous générez.

## <a name="add-azure-function-to-logic-app"></a>Ajouter une fonction Azure à l’application logique

1. Dans le Concepteur d’application logique, sous l’action **Détecter un sentiment**, choisissez **Nouvelle étape**.

2. Recherchez le connecteur **Azure Functions** et sélectionnez la fonction que vous avez créée.

3. Sous **Corps de la demande**, sélectionnez **Texte du Tweet**.

![Étape de configuration d’une fonction Azure][2]

## <a name="run-and-monitor-your-logic-app"></a>Exécuter et analyser l’application logique

Pour consulter les exécutions courantes ou précédentes de votre application logique, vous pouvez utiliser le débogage riche et les fonctionnalités d’analyse que fournit Azure Logic Apps dans le portail Azure, Visual Studio ou via les API REST et kits de développement logiciel (SDK) Azure.

Pour tester facilement votre application logique, dans le Concepteur d’application logique, choisissez **Exécuter un déclencheur**. Le déclencheur interroge les tweets selon votre planification spécifiée jusqu'à ce qu’un tweet qui répond à vos critères soit trouvé. Lors de l’exécution, le concepteur affiche une vue active pour cette exécution.

Pour afficher des historiques d’exécution précédentes dans Visual Studio ou le portail Azure : 

* Ouvrez Visual Studio Cloud Explorer. Recherchez votre application logique, ouvrez le menu contextuel de l’application. Sélectionnez **Ouvrir l’historique des exécutions**.

  > [!TIP]
  > Si vous ne voyez pas cette commande dans Visual Studio 2019, vérifiez que vous avez les dernières mises à jour pour Visual Studio.

* Dans le Portail Azure, recherchez votre application logique. Dans le menu de votre application logique, sélectionnez **Vue d’ensemble**. 

## <a name="create-automated-deployment-templates"></a>Créer des modèles de déploiement automatisé

Après avoir créé une solution d’application logique, vous pouvez capturer et déployer votre application comme un [modèle Azure Resource Manager](../azure-resource-manager/templates/overview.md) à n’importe quelle région Azure dans le monde. Vous pouvez utiliser cette fonctionnalité pour modifier les paramètres pour la création de différentes versions de votre application et pour l’intégration de votre solution dans Azure Pipelines. Vous pouvez également inclure Azure Functions dans votre modèle de déploiement afin de gérer la solution entière avec toutes les dépendances en tant que modèle unique. Apprenez comment [automatiser le déploiement d’applications logiques](logic-apps-azure-resource-manager-templates-overview.md).

Vous trouverez un exemple de modèle de déploiement avec une fonction Azure dans le [référentiel de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-function-app-create-dynamic).

## <a name="next-steps"></a>Étapes suivantes

* [Rechercher d’autres exemples et scénarios relatifs à Azure Logic Apps](logic-apps-examples-and-scenarios.md)

<!-- Image References -->
[1]: ./media/logic-apps-scenario-social-serverless/twitter.png
[2]: ./media/logic-apps-scenario-social-serverless/function.png

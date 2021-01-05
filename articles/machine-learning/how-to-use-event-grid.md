---
title: Déclenchement des événements dans les workflows ML (préversion)
titleSuffix: Azure Machine Learning
description: Configurez des applications, des processus ou des workflows de Machine Learning CI/CD pilotés par des événements dans Azure Machine Learning.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.custom: how-to
ms.author: shipatel
author: shivp950
ms.reviewer: larryfr
ms.date: 05/11/2020
ms.openlocfilehash: 1fd177273c9dafb04add64d8a8bfef1d81cc65d0
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93319310"
---
# <a name="trigger-applications-processes-or-cicd-workflows-based-on-azure-machine-learning-events-preview"></a>Déclenchement des applications, des processus et des workflows CI/CD en fonction d’événements Azure Machine Learning (préversion)

Dans cet article, vous allez apprendre à configurer des applications, des processus ou des workflows CI/CD basés sur des événements Azure Machine Learning, comme des e-mails de notification d’échec ou des exécutions de pipeline ML, lorsque certaines conditions sont détectées par [Azure Event Grid](../event-grid/index.yml).

Azure Machine Learning gère l’intégralité du cycle de vie du processus d’apprentissage automatique, y compris l’apprentissage du modèle, le déploiement du modèle et la surveillance. Vous pouvez utiliser Event Grid pour réagir aux événements Azure Machine Learning, comme l’achèvement des exécutions d’apprentissage, l’enregistrement et le déploiement de modèles et la détection de la dérive de données, à l’aide d’architectures serverless modernes. Vous pouvez alors vous abonner et utiliser des événements tels que la modification de l’état d’exécution, la fin d’une exécution, l’inscription de modèles, le déploiement de modèles et la détection d’une dérive de données dans un espace de travail.

Quand utiliser Event Grid pour les actions pilotées par des événements :
* Envoyer des e-mails à la fin d’une exécution et en cas d’échec d’exécution
* Utiliser une fonction Azure après l’inscription d’un modèle
* Événements de diffusion en continu d’Azure Machine Learning sur divers points de terminaison
* Déclencher un pipeline ML quand une dérive est détectée

> [!NOTE] 
> Actuellement, les événements runStatusChanged se déclenchent uniquement lorsque l’état d’exécution est **échec**

## <a name="prerequisites"></a>Prérequis
Pour utiliser Event Grid, vous devez avoir un accès contributeur ou propriétaire à l’espace de travail Azure Machine Learning pour lequel vous allez créer des événements.

## <a name="the-event-model--types"></a>Modèle et types d’événements

Azure Event Grid lit les événements à partir de sources, comme Azure Machine Learning et d’autres services Azure. Ces événements sont ensuite envoyés à des gestionnaires d’événements tels qu’Azure Event Hubs, Azure Functions, Logic Apps et d’autres. Le diagramme suivant montre comment Event Grid connecte les sources et les descripteurs, mais ne constitue pas une liste complète des intégrations prises en charge.

![Modèle Azure Event Grid fonctionnel](./media/concept-event-grid-integration/azure-event-grid-functional-model.png)

Pour plus d’informations sur les sources d’événements et les gestionnaires d’événements, consultez [Qu’est-ce qu’Event Grid ?](../event-grid/overview.md).

### <a name="event-types-for-azure-machine-learning"></a>Types d’événements pour Azure Machine Learning

Azure Machine Learning fournit des événements tout au long du cycle de vie de l’apprentissage automatique : 

| Type d'événement | Description |
| ---------- | ----------- |
| `Microsoft.MachineLearningServices.RunCompleted` | Déclenché lorsque l’exécution d’une expérience d’apprentissage automatique est terminée |
| `Microsoft.MachineLearningServices.ModelRegistered` | Déclenché lorsqu’un modèle d’apprentissage automatique est inscrit dans l’espace de travail |
| `Microsoft.MachineLearningServices.ModelDeployed` | Déclenché lorsqu’un déploiement du service d’inférence avec un ou plusieurs modèles est terminé |
| `Microsoft.MachineLearningServices.DatasetDriftDetected` | Déclenché lorsqu’un travail de détection de dérive de données pour deux jeux de données est terminé |
| `Microsoft.MachineLearningServices.RunStatusChanged` | Déclenché quand un état d’exécution changeait ; actuellement déclenché uniquement quand l’état d’exécution a la valeur « failed » |

### <a name="filter--subscribe-to-events"></a>Filtrer les événements et s’y abonner

Ces événements sont publiés via Azure Event Grid. À l’aide du portail Azure, de PowerShell ou d’Azure CLI, les clients peuvent facilement s’abonner aux événements en [spécifiant un ou plusieurs types d’événements, et en filtrant des conditions](../event-grid/event-filtering.md). 

Lors de la configuration de vos événements, vous pouvez appliquer des filtres de manière à ce qu’ils ne se déclenchent que sur des données d’événements spécifiques. Dans l’exemple ci-dessous, pour les événements de modification de l’état d’exécution, vous pouvez filtrer par type d’exécution. L’événement se déclenche uniquement lorsque les critères sont satisfaits. Pour en savoir plus sur les données d’événement que vous pouvez filtrer, reportez-vous au [schéma Event Grid pour Azure Machine Learning](../event-grid/event-schema-machine-learning.md). 

Les abonnements pour les événements Azure Machine Learning sont protégés par le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Seul le [contributeur ou propriétaire](how-to-assign-roles.md#default-roles) d’un espace de travail peut créer, mettre à jour et supprimer des abonnements aux événements.  Des filtres peuvent être appliqués aux abonnements aux événements pendant la [création](/cli/azure/eventgrid/event-subscription?view=azure-cli-latest&preserve-view=true) de l’abonnement aux événements ou ultérieurement. 


1. Accédez au Portail Azure, sélectionnez un nouvel abonnement ou un abonnement existant. 

1. Sélectionnez l’onglet Filtres et faites défiler jusqu’à Filtres avancés. Pour les champs **Clé** et **Valeur** , fournissez les types de propriété par lesquels vous souhaitez filtrer. Ici, vous pouvez voir que l’événement se déclenche uniquement lorsque le type d’exécution est une exécution de pipeline ou une exécution d’étape de pipeline.  

    :::image type="content" source="media/how-to-use-event-grid/select-event-filters.png" alt-text="Filtrer les événements":::


+ **Filtrer par type d’événement :** Un abonnement aux événements peut spécifier un ou plusieurs types d’événements Azure Machine Learning.

+ **Filtrer par sujet d’événement :** Azure Event Grid prend en charge les filtres de sujet reposant sur les correspondances __commence par__ et __se termine par__ , afin que les événements dont l’objet correspond soient remis à l’abonné. Des événements d’apprentissage automatique différents ont un format de sujet différent.

  | Type d'événement | Format du sujet | Exemple de sujet |
  | ---------- | ----------- | ----------- |
  | `Microsoft.MachineLearningServices.RunCompleted` | `experiments/{ExperimentId}/runs/{RunId}` | `experiments/b1d7966c-f73a-4c68-b846-992ace89551f/runs/my_exp1_1554835758_38dbaa94` |
  | `Microsoft.MachineLearningServices.ModelRegistered` | `models/{modelName}:{modelVersion}` | `models/sklearn_regression_model:3` |
  | `Microsoft.MachineLearningServices.ModelDeployed` | `endpoints/{serviceId}` | `endpoints/my_sklearn_aks` |
  | `Microsoft.MachineLearningServices.DatasetDriftDetected` | `datadrift/{data.DataDriftId}/run/{data.RunId}` | `datadrift/4e694bf5-712e-4e40-b06a-d2a2755212d4/run/my_driftrun1_1550564444_fbbcdc0f` |
  | `Microsoft.MachineLearningServices.RunStatusChanged` | `experiments/{ExperimentId}/runs/{RunId}` | `experiments/b1d7966c-f73a-4c68-b846-992ace89551f/runs/my_exp1_1554835758_38dbaa94` | 

+ **Filtrage avancé :** Azure Event Grid prend également en charge le filtrage avancé basé sur le schéma d’événement publié. D’avantage d’informations sur le schéma des événement Azure Machine Learning peuvent être trouvées dans [Schéma des événements Azure Event Grid pour Azure Machine Learning](../event-grid/event-schema-machine-learning.md).  Voici quelques exemples de filtres avancés que vous pouvez effectuer :

  Pour l’événement `Microsoft.MachineLearningServices.ModelRegistered`, afin de filtrer la valeur de balise du modèle :

  ```
  --advanced-filter data.ModelTags.key1 StringIn ('value1')
  ```

  Pour en savoir plus sur l’application de filtres, consultez [Filtrer des événements pour Event Grid](../event-grid/how-to-filter-events.md).

## <a name="consume-machine-learning-events"></a>Consommer des événements Machine Learning

Les applications qui gèrent des événements Machine Learning doivent suivre certaines pratiques recommandées :

> [!div class="checklist"]
> * Comme plusieurs abonnements peuvent être configurés pour acheminer les événements vers le même gestionnaire d’événements, il est important de ne pas considérer que les événements proviennent d’une source particulière, mais de vérifier le sujet du message pour vous assurer qu’il provient d’un espace de travail de Machine Learning que vous attendez.
> * De même, vérifiez que vous êtes prêt à traiter son eventType, et ne supposez pas que tous les événements reçus seront aux types que vous attendez.
> * Les messages pouvant arriver en désordre et après un certain temps, utilisez les champs etag pour comprendre si vos informations sur les objets sont toujours à jour.  En outre, utilisez les champs de séquence pour comprendre l’ordre des événements sur un objet particulier.
> * Ignorez les champs que vous ne comprenez pas. Cette pratique vous aidera à prendre en charge les nouvelles fonctionnalités qui peuvent être ajoutées à l’avenir.
> * Les opérations Azure Machine Learning annulées ou ayant échoué ne déclenchent pas d’événement. Par exemple, en cas d’échec du déploiement d’un modèle, Microsoft.MachineLearningServices.ModelDeployed ne sera pas déclenché. Prenez en compte ce mode d’échec lors de la conception de vos applications. Vous pouvez toujours utiliser le Kit de développement logiciel (SDK), l’interface CLI ou le portail Azure Machine Learning pour vérifier l’état d’une opération et comprendre les raisons détaillées de l’échec.

Azure Event Grid permet aux clients de créer des gestionnaires de messages découplés, qui peuvent être déclenchés par des événements Azure Machine Learning. Voici quelques exemples notables de gestionnaires de messages :
* Azure Functions
* Azure Logic Apps
* Hubs d'événements Azure
* Pipeline Azure Data Factory
* Webhooks génériques, qui peuvent être hébergés sur la plateforme Azure ou ailleurs

## <a name="set-up-in-azure-portal"></a>Configuré dans le portail Azure

1. Ouvrez le [Portail Azure](https://portal.azure.com) et accédez à votre espace de travail Azure Machine Learning.

1. Dans la barre de gauche, sélectionnez __Événements__ puis sélectionnez **Abonnements aux événements**. 

    ![select-events-in-workspace.png](./media/how-to-use-event-grid/select-event.png)

1. Sélectionnez le type d’événement à consommer. Par exemple, la capture d’écran suivante a sélectionné __Modèle inscrit__ , __Modèle déployé__ , __Exécution terminée__ et __Dérive de jeu de données détectée__ :

    ![add-event-type](./media/how-to-use-event-grid/add-event-type-updated.png)

1. Sélectionnez le point de terminaison sur lequel publier l’événement. Dans la capture d’écran suivante, __Event Hub__ est le point de terminaison sélectionné :

    ![Capture d’écran montrant le volet Créer un abonnement aux événements avec la boîte de dialogue Sélectionner un Event Hub ouverte.](./media/how-to-use-event-grid/select-event-handler.png)

Une fois que vous avez confirmé votre sélection, cliquez sur __Créer__. Après la configuration, ces événements seront envoyés à votre point de terminaison.


### <a name="set-up-with-the-cli"></a>Configurer avec l’interface CLI

Vous pouvez installer la dernière version d’[Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) ou utiliser Azure Cloud Shell fourni dans le cadre de votre abonnement Azure.

Pour installer l’extension Event Grid, utilisez la commande suivante à partir de l’interface CLI :

```azurecli-interactive
az add extension --name eventgrid
```

L’exemple suivant montre comment sélectionner un abonnement Azure et créer un abonnement aux événements pour Azure Machine Learning :

```azurecli-interactive
# Select the Azure subscription that contains the workspace
az account set --subscription "<name or ID of the subscription>"

# Subscribe to the machine learning workspace. This example uses EventHub as a destination. 
az eventgrid event-subscription create --name {eventGridFilterName} \
  --source-resource-id /subscriptions/{subId}/resourceGroups/{RG}/providers/Microsoft.MachineLearningServices/workspaces/{wsName} \
  --endpoint-type eventhub \
  --endpoint /subscriptions/{SubID}/resourceGroups/TestRG/providers/Microsoft.EventHub/namespaces/n1/eventhubs/EH1 \
  --included-event-types Microsoft.MachineLearningServices.ModelRegistered \
  --subject-begins-with "models/mymodelname"
```

## <a name="examples"></a>Exemples

### <a name="example-send-email-alerts"></a>Exemple : Envoyer des alertes par e-mail

Utilisez [Azure Logic Apps](../logic-apps/index.yml) pour configurer des e-mails pour tous vos événements. Personnalisez avec des conditions et spécifiez des destinataires pour permettre la collaboration et la sensibilisation entre les équipes travaillant ensemble.

1. Dans la Portail Azure, accédez à votre espace de travail Azure Machine Learning et sélectionnez l’onglet Événements dans la barre de gauche. À partir de là, sélectionnez __Applications logiques__. 

    ![Capture d’écran montrant une page d’événements de l’espace de travail Machine Learning avec Logic Apps.](./media/how-to-use-event-grid/select-logic-ap.png)

1. Connectez-vous à l’interface utilisateur de l’application logique et sélectionnez le service Machine Learning comme type de rubrique. 

    ![Capture d’écran montrant la boîte de dialogue Quand un événement de ressource se produit avec Machine Learning sélectionné comme type de ressource.](./media/how-to-use-event-grid/select-topic-type.png)

1. Sélectionnez le ou les événements pour lesquels vous souhaitez recevoir des alertes. Par exemple, la capture d’écran suivante __RunCompleted__.

    ![Capture d’écran montrant la boîte de dialogue Quand un événement de ressource se produit avec un type d’événement sélectionné.](./media/how-to-use-event-grid/select-event-runcomplete.png)

1. Vous pouvez utiliser la méthode de filtrage de la section précédente ou ajouter des filtres pour déclencher uniquement l’application logique sur un sous-ensemble de types d’événements. Dans la capture d’écran suivante, un __filtre de préfixe__ de __/datadriftID/runs/__ est utilisé.

    ![filter-events](./media/how-to-use-event-grid/filtering-events.png)

1. Ensuite, ajoutez une étape pour utiliser cet événement et rechercher un e-mail. Il existe plusieurs comptes de messagerie différents que vous pouvez utiliser pour recevoir des événements. Vous pouvez également configurer des conditions sur le moment où envoyer une alerte par e-mail.

    ![Capture d’écran montrant la boîte de dialogue Choisir une action avec une adresse e-mail entrée dans la ligne de recherche.](./media/how-to-use-event-grid/select-email-action.png)

1. Sélectionnez __Envoyer un e-mail__ et renseignez les paramètres. Dans l’objet, vous pouvez inclure le __Type d’événement__ et la __Rubrique__ pour aider à filtrer les événements. Vous pouvez également inclure un lien vers la page de l’espace de travail pour les exécutions dans le corps du message. 

    ![Capture d’écran montrant la boîte de dialogue Envoyer un e-mail avec un objet et un type d’événement ajoutés à la ligne d’objet de la liste à droite.](./media/how-to-use-event-grid/configure-email-body.png)

1. Pour enregistrer cette action, sélectionnez **Enregistrer sous** dans le coin gauche de la page. Dans la barre de droite qui s’affiche, confirmez la création de cette action.

    ![Capture d’écran montrant les boutons Enregistrer sous et Créer dans le Concepteur Logic Apps.](./media/how-to-use-event-grid/confirm-logic-app-create.png)


### <a name="example-data-drift-triggers-retraining"></a>Exemple : La dérive de données déclenche la reformation

Les modèles deviennent obsolètes au fil du temps et ne restent pas fonctionnels dans le contexte dans lequel ils s’exécutent. Pour savoir s’il est temps de reformer le modèle, vous pouvez détecter la dérive des données. 

Cet exemple montre comment utiliser Event Grid avec une application logique Azure pour déclencher la reformation. L’exemple déclenche un pipeline Azure Data Factory lorsqu’une dérive de données se produit entre les jeux de données de formation et de service d’un modèle.

Avant de commencer, effectuez les étapes suivantes :

* Configurer une analyse de jeu de données pour [détecter la dérive de données](how-to-monitor-datasets.md) dans un espace de travail
* Créer un [pipeline Azure Data Factory](../data-factory/index.yml) publié.

Dans cet exemple, un pipeline Data Factory simple est utilisé pour copier des fichiers dans un magasin d’objets blob et exécuter un pipeline Machine Learning publié. Pour plus d’informations sur ce scénario, découvrez comment configurer une [étape de Machine Learning dans Azure Data Factory](../data-factory/transform-data-machine-learning-service.md)

![Capture d’écran montrant le pipeline d’apprentissage dans Ressources Factory avec Copy data1 alimentant ML Execute Pipeline1.](./media/how-to-use-event-grid/adf-mlpipeline-stage.png)

1. Commencez par créer l’application logique. Accédez au [Portail Azure](https://portal.azure.com), recherchez Applications logiques, puis sélectionnez Créer.

    ![search-logic-app](./media/how-to-use-event-grid/search-for-logic-app.png)

1. Entrez les informations demandées. Pour simplifier l’expérience, utilisez le même abonnement et le même groupe de ressources que votre pipeline Azure Data Factory et votre espace de travail Azure Machine Learning.

    ![Capture d’écran montrant le volet de création d’application logique.](./media/how-to-use-event-grid/set-up-logic-app-for-adf.png)

1. Une fois que vous avez créé l’application logique, sélectionnez __Quand un événement lié à une ressource Event Grid se produit__. 

    ![Capture d’écran montrant le Concepteur Logic Apps avec les options Démarrer avec un déclencheur courant, notamment Quand un événement lié à une ressource Event Grid se produit.](./media/how-to-use-event-grid/select-event-grid-trigger.png)

1. Connectez-vous et renseignez les détails de l’événement. Définissez le __Nom de la ressource__ sur le nom de l’espace de travail. Définissez le __type d’événement__ sur __DatasetDriftDetected__.

    ![Capture d’écran montrant la fenêtre Quand un événement de ressource se produit avec un Élément de type d’événement sélectionné.](./media/how-to-use-event-grid/login-and-add-event.png)

1. Ajoutez une nouvelle étape, et recherchez __Azure Data Factory__. Sélectionnez __Créer une exécution du pipeline__. 

    ![Capture d’écran montrant le volet Choisir une action avec l’option Créer une exécution du pipeline sélectionnée.](./media/how-to-use-event-grid/create-adfpipeline-run.png)

1. Connectez-vous et spécifiez le pipeline Azure Data Factory publié à exécuter.

    ![Capture d’écran montrant le volet Créer une exécution de pipeline avec différentes valeurs.](./media/how-to-use-event-grid/specify-adf-pipeline.png)

1. Enregistrez et créez l’application logique à l’aide du bouton **Enregistrer** en haut à gauche de la page. Pour afficher votre application, accédez à votre espace de travail dans le [Portail Azure](https://portal.azure.com) et cliquez sur **Événements**.

    ![Capture d’écran montrant des événements avec Logic App en surbrillance.](./media/how-to-use-event-grid/show-logic-app-webhook.png)

Désormais, le pipeline Data Factory est déclenché lorsque la dérive se produit. Affichez les détails de l’exécution de la dérive de données et du pipeline d’apprentissage automatique sur le [nouveau portail d’espace de travail](https://ml.azure.com). 

![Capture d’écran montrant les points de terminaison de pipeline.](./media/how-to-use-event-grid/view-in-workspace.png)

### <a name="example-deploy-a-model-based-on-tags"></a>Exemple : Déployer un modèle basé sur des balises

Un objet de modèle Azure Machine Learning contient des paramètres sur lesquels vous pouvez baser les déploiements, tels que le nom du modèle, la version, la balise et la propriété. L’événement d’inscription du modèle peut déclencher un point de terminaison et vous pouvez utiliser une fonction Azure pour déployer un modèle en fonction de la valeur de ces paramètres.

Pour obtenir un exemple, consultez le référentiel [https://github.com/Azure-Samples/MachineLearningSamples-NoCodeDeploymentTriggeredByEventGrid](https://github.com/Azure-Samples/MachineLearningSamples-NoCodeDeploymentTriggeredByEventGrid) et suivez les étapes dans le fichier **readme**.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur Event Grid et essayer les événements Azure Machine Learning :

- [À propos d’Event Grid](../event-grid/overview.md)

- [Schéma d’événements pour Azure Machine Learning](../event-grid/event-schema-machine-learning.md)
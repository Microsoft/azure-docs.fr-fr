---
title: Tutoriel - Utiliser des événements IoT Hub pour déclencher des actions Azure Logic Apps
description: Ce tutoriel montre comment utiliser le service de routage d’événements d’Azure Event Grid, et créer des processus automatisés pour effectuer des actions Azure Logic Apps basées sur des événements IoT Hub.
services: iot-hub, event-grid
author: philmea
ms.service: iot-hub
ms.topic: tutorial
ms.date: 09/14/2020
ms.author: philmea
ms.custom: devx-track-azurecli
ms.openlocfilehash: 857ae8d824443e9a8abdac7c4a66e2b014be2be0
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94566348"
---
# <a name="tutorial-send-email-notifications-about-azure-iot-hub-events-using-event-grid-and-logic-apps"></a>Tutoriel : Envoyer des notifications par e-mail concernant des événements Azure IoT Hub à l’aide d’Event Grid et de Logic Apps

Azure Event Grid vous permet de réagir aux événements dans IoT Hub en déclenchant des actions dans vos applications d’entreprise en aval.

Cet article présente un exemple de configuration qui utilise IoT Hub et Event Grid. À la fin, vous disposez d’une application logique Azure configurée pour envoyer un e-mail de notification chaque fois qu’un appareil se connecte à votre hub IoT ou s’en déconnecte. Event Grid peut être utilisé pour obtenir en temps voulu une notification sur la déconnexion des appareils critiques. L’affichage des métriques et des diagnostics dans les journaux/alertes peut prendre plusieurs minutes (à savoir, 20 minutes ou plus, même si nous ne voulons pas chiffrer ce délai). Cela peut être inacceptable pour une infrastructure critique.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

* Un compte e-mail auprès de n’importe quel fournisseur de messagerie pris en charge par Azure Logic Apps, comme Office 365 Outlook ou Outlook.com. Ce compte e-mail permet d’envoyer les notifications d’événements.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

## <a name="create-an-iot-hub"></a>Créer un hub IoT

Vous pouvez rapidement créer un hub IoT à l’aide du terminal Azure Cloud Shell dans le portail.

1. Connectez-vous au [portail Azure](https://portal.azure.com). 

1. En haut à droite dans la page, sélectionnez le bouton Cloud Shell.

   ![Bouton Cloud Shell](./media/publish-iot-hub-events-to-logic-apps/portal-cloud-shell.png)

1. Utilisez la commande suivante pour créer un groupe de ressources :

   ```azurecli
   az group create --name {your resource group name} --location westus
   ```
    
1. Exécutez la commande suivante pour créer un hub IoT :

   ```azurecli
   az iot hub create --name {your iot hub name} --resource-group {your resource group name} --sku S1 
   ```

1. Réduisez le terminal Cloud Shell. Vous retournerez à l’interpréteur de commandes plus tard dans ce tutoriel.

## <a name="create-a-logic-app"></a>Créer une application logique

Créez ensuite une application logique, puis ajoutez un déclencheur Event Grid HTTP qui traite les requêtes en provenance du hub IoT. 

### <a name="create-a-logic-app-resource"></a>Créer une ressource d’application logique

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource**, puis tapez « application logique » dans la zone de recherche et appuyez sur Entrée. Sélectionnez **Application logique**  dans les résultats.

   ![Créer une application logique](./media/publish-iot-hub-events-to-logic-apps/select-logic-app.png)

1. Sur l’écran suivant, sélectionnez **Créer**. 

1. Donnez à votre application logique un nom qui est unique dans votre abonnement, puis sélectionnez les mêmes abonnement, groupe de ressources et emplacement que votre hub IoT. 

   ![Champs pour la création d’une application logique](./media/publish-iot-hub-events-to-logic-apps/create-logic-app-fields.png)

1. Sélectionnez **Revoir + créer**.

1. Vérifiez vos paramètres, puis sélectionnez **Créer**.

1. Une fois la ressource créée, sélectionnez **Accéder à la ressource**. 

1. Dans le concepteur Logic Apps, faites défiler la page vers le bas pour afficher **Modèles**. Choisissez **Application logique vide** afin de générer votre application logique à partir de zéro.

### <a name="select-a-trigger"></a>Sélectionner un déclencheur

Un déclencheur désigne un événement spécifique qui démarre votre application logique. Pour ce didacticiel, le déclencheur qui lance le flux de travail reçoit une demande via HTTP.  

1. Dans la barre de recherche des connecteurs et des déclencheurs, tapez **HTTP**.

1. Parcourez les résultats, puis sélectionnez **Requête - Lors de la réception d’une requête HTTP** comme déclencheur. 

   ![Sélectionner le déclencheur de requête HTTP](./media/publish-iot-hub-events-to-logic-apps/http-request-trigger.png)

1. Sélectionnez **Utiliser l’exemple de charge utile pour générer le schéma**. 

   ![Utiliser un exemple de charge utile](./media/publish-iot-hub-events-to-logic-apps/sample-payload.png)

1. Collez le code JSON du *Schéma d’événement connecté à l’appareil* dans la zone de texte, puis sélectionnez **Terminé** :

   ```json
     [{  
      "id": "f6bbf8f4-d365-520d-a878-17bf7238abd8",
      "topic": "/SUBSCRIPTIONS/<subscription ID>/RESOURCEGROUPS/<resource group name>/PROVIDERS/MICROSOFT.DEVICES/IOTHUBS/<hub name>",
      "subject": "devices/LogicAppTestDevice",
      "eventType": "Microsoft.Devices.DeviceConnected",
      "eventTime": "2018-06-02T19:17:44.4383997Z",
      "data": {
          "deviceConnectionStateEventInfo": {
            "sequenceNumber":
              "000000000000000001D4132452F67CE200000002000000000000000000000001"
          },
        "hubName": "egtesthub1",
        "deviceId": "LogicAppTestDevice",
        "moduleId" : "DeviceModuleID"
      }, 
      "dataVersion": "1",
      "metadataVersion": "1"
    }]
   ```

   Cet événement est publié quand un appareil est connecté à un hub IoT.

> [!NOTE]
> Vous pouvez recevoir une notification contextuelle indiquant **N’oubliez pas d’inclure un en-tête Content-Type défini sur application/JSON dans votre demande**. Vous pouvez ignorer cette suggestion et passer à la section suivante. 

### <a name="create-an-action"></a>Créer une action

Les actions sont toutes les étapes qui se produisent une fois que le déclencheur démarre le flux de travail de l’application logique. Pour ce didacticiel, l’action doit envoyer une notification par e-mail à partir de votre fournisseur d’e-mail. 

1. Sélectionnez **Nouvelle étape**. Une fenêtre permettant de **Choisir une action** s’ouvre.

1. Recherchez **Outlook**.

1. Selon votre fournisseur de messagerie, recherchez et sélectionnez le connecteur correspondant. Ce tutoriel utilise **Outlook.com**. Les étapes pour les autres fournisseurs d’e-mail sont similaires. 

   ![Sélectionner le connecteur du fournisseur d’e-mail](./media/publish-iot-hub-events-to-logic-apps/outlook-step.png)

1. Sélectionnez l’action **Envoyer un e-mail (V2)**. 

1. Sélectionnez **Se connecter**, puis connectez-vous à votre compte de messagerie. Sélectionnez **Oui** pour permettre à l’application d’accéder à vos informations.

1. Créez votre modèle d’e-mail. 

   * **À** : Entrez l’adresse e-mail qui recevra les e-mails de notification. Pour ce didacticiel, utilisez un compte de messagerie auquel vous pouvez accéder pour les tests. 

   * **Objet** : Renseignez le texte de l’objet. Quand vous cliquez sur la zone de texte Objet, vous pouvez sélectionner du contenu dynamique à inclure. Par exemple, ce tutoriel utilise `IoT Hub alert: {eventType}`. Si vous ne voyez pas de contenu dynamique, sélectionnez le lien hypertexte **Ajouter du contenu dynamique** : cela l’active ou le désactive.

   * **Corps**: écrivez le texte de votre e-mail. Sélectionnez les propriétés JSON à partir de l’outil de sélection pour inclure du contenu dynamique en fonction des données d’événement. Si vous ne voyez pas le contenu dynamique, sélectionnez le lien hypertexte **Ajouter du contenu dynamique** sous la zone de texte **Corps**. Si cette opération n’affiche pas les champs souhaités, cliquez sur *plus* dans l’écran Contenu dynamique pour inclure les champs de l’action précédente.

   Votre modèle d’e-mail peut ressembler à cet exemple :

   ![Renseigner les informations d’e-mail](./media/publish-iot-hub-events-to-logic-apps/email-content.png)

1. Sélectionnez **Enregistrer** dans le Concepteur Logic Apps.  

### <a name="copy-the-http-url"></a>Copier l’URL HTTP

Avant de quitter le Concepteur d’applications logiques, copiez l’URL qu’écoutent vos applications logiques dans l’attente d’un déclencheur. Cette URL vous permet de configurer Event Grid. 

1. Développez la zone de configuration de déclencheur **Lors de la réception d’une demande HTTP** en cliquant dessus. 

1. Copiez la valeur du champ **URL HTTP POST** en sélectionnant le bouton de copie en regard de celui-ci. 

   ![Copier l’URL HTTP POST](./media/publish-iot-hub-events-to-logic-apps/copy-url.png)

1. Enregistrez cette URL afin de pouvoir y faire référence dans la section suivante. 

## <a name="configure-subscription-for-iot-hub-events"></a>Configurer l’abonnement pour les événements IoT Hub

Dans cette section, vous configurez votre hub IoT pour publier des événements à mesure qu’ils se produisent. 

1. Accédez à votre hub IoT dans le portail Azure. Pour ce faire, sélectionnez **Groupes de ressources**, puis le groupe de ressources de ce tutoriel. Sélectionnez ensuite votre hub IoT dans la liste de ressources.

1. Sélectionnez **Événements**.

   ![Ouvrir les détails Event Grid](./media/publish-iot-hub-events-to-logic-apps/event-grid.png)

1. Sélectionnez **Abonnement aux événements**. 

   ![Créer un abonnement aux événements](./media/publish-iot-hub-events-to-logic-apps/event-subscription.png)

1. Créez l’abonnement aux événements avec les valeurs suivantes : 

   1. Dans la section **DÉTAILS DES ABONNEMENTS AUX ÉVÉNEMENTS** :
      1. Indiquez un **nom** pour l’abonnement à un événement. 
      2. Sélectionnez **Event Grid Schema** comme **Schéma d’événement**. 
   2. Dans la section **DÉTAILS DE LA RUBRIQUE** :
      1. Confirmez que le **Type de rubrique** est **IoT Hub**. 
      2. Confirmez que le nom du hub IoT est défini en tant que valeur du champ **Ressource source**. 
      3. Entrez un nom pour la **rubrique système** qui sera créé pour vous. Pour en savoir plus sur les rubriques système, consultez [Vue d’ensemble des rubriques système](system-topics.md).
   3. Dans la section **TYPES D’ÉVÉNEMENTS** :
      1. Sélectionnez la liste déroulante **Filtrer sur les types d’événements**.
      1. Décochez les cases **Appareil créé** et **Appareil supprimé**, et laissez uniquement les cases **Appareil connecté** et **Appareil déconnecté** cochées.

         ![sélectionner les types d’événements d’abonnement](./media/publish-iot-hub-events-to-logic-apps/subscription-event-types.png)
   
   4. Dans la section **DÉTAILS DU POINT DE TERMINAISON** : 
       1. Sélectionnez **Webhook** comme **Type de point de terminaison**.
       2. Cliquez sur **Sélectionner un point de terminaison**, collez l’URL copiée à partir de votre application logique, puis confirmez la sélection.

         ![sélectionner l’URL du point de terminaison](./media/publish-iot-hub-events-to-logic-apps/endpoint-webhook.png)

         Quand vous avez terminé, le volet doit ressembler à l’exemple suivant : 

         ![Exemple de formulaire d’abonnement aux événements](./media/publish-iot-hub-events-to-logic-apps/subscription-form.png)

1.  Sélectionnez **Create** (Créer).

## <a name="simulate-a-new-device-connecting-and-sending-telemetry"></a>Simuler la connexion d’un nouvel appareil et l’envoi de données de télémétrie par celui-ci

Testez votre application logique en simulant rapidement une connexion d’appareil à l’aide de l’interface Azure CLI. 

1. Sélectionnez le bouton Cloud Shell pour rouvrir votre terminal.

1. Exécutez la commande suivante pour créer une identité d’appareil simulé :
    
     ```azurecli 
    az iot hub device-identity create --device-id simDevice --hub-name {YourIoTHubName}
    ```

1. Exécutez la commande suivante pour simuler la connexion de votre appareil à IoT Hub et l’envoi de données de télémétrie :

    ```azurecli
    az iot device simulate -d simDevice -n {YourIoTHubName}
    ```

1. Quand l’appareil simulé se connecte à IoT Hub, vous recevrez un e-mail vous notifiant un événement « DeviceConnected ».

1. Une fois la simulation terminée, vous recevez un e-mail vous notifiant l’événement « DeviceDisconnected ». 

    ![Exemple de message d’alerte](./media/publish-iot-hub-events-to-logic-apps/alert-mail.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Ce didacticiel utilise des ressources qui peuvent entraîner des frais sur votre abonnement Azure. Quand vous avez terminé de tester le tutoriel et vos résultats, désactivez ou supprimez les ressources que vous ne souhaitez pas conserver. 

Pour supprimer toutes les ressources créées dans ce tutoriel, supprimez le groupe de ressources. 

1. Sélectionnez **Groupes de ressources**, puis sélectionnez le groupe de ressources que vous avez créé pour ce tutoriel.

2. Dans le volet Groupe de ressources, sélectionnez **Supprimer un groupe de ressources**. Vous êtes invité à entrer le nom du groupe de ressources, après quoi vous pouvez le supprimer. Toutes les ressources qu’il contient sont également supprimées.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez-en plus sur la [réaction aux événements IoT Hub en utilisant Event Grid pour déclencher des actions](../iot-hub/iot-hub-event-grid.md).
* [Apprendre à commander des événements d’état de la connexion et de la déconnexion d’appareils](../iot-hub/iot-hub-how-to-order-connection-state-events.md)
* Découvrez ce que vous pouvez faire d’autre avec [Event Grid](overview.md).

Pour obtenir la liste complète des connecteurs d’application logique pris en charge, consultez la [Vue d’ensemble des connecteurs](/connectors/).

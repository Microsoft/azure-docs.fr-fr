---
title: Surveillance à distance IoT et notifications avec Azure Logic Apps | Microsoft Docs
description: Utilisez Azure Logic Apps pour surveiller la température IoT sur votre hub IoT et envoyer automatiquement des notifications par courrier électronique dans votre boîte aux lettres en cas de détection d’anomalies.
author: robinsh
keywords: surveillance iot, notifications iot, surveillance de température iot
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.tgt_pltfrm: arduino
ms.date: 07/18/2019
ms.author: robinsh
ms.openlocfilehash: cd14ff0688f4230aeedac748ca4b32609bdd2938
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490320"
---
# <a name="iot-remote-monitoring-and-notifications-with-azure-logic-apps-connecting-your-iot-hub-and-mailbox"></a>Surveillance à distance IoT et notifications avec Azure Logic Apps connectant votre IoT Hub et votre boîte aux lettres

![Diagramme de bout en bout](media/iot-hub-monitoring-notifications-with-azure-logic-apps/iot-hub-e2e-logic-apps.png)

[!INCLUDE [iot-hub-get-started-note](../../includes/iot-hub-get-started-note.md)]

[Azure Logic Apps](../logic-apps/index.yml) peut vous aider à orchestrer des flux de travail entre des services locaux et cloud, une ou plusieurs entreprises et entre divers protocoles. Une application logique commence par un déclencheur, suivi d’une ou plusieurs actions qui peuvent être classées à l’aide de contrôles intégrés comme des conditions et des itérateurs. Cette flexibilité fait de Logic Apps une solution d’IoT idéale pour les scénarios de supervision IoT. Par exemple, l’arrivée de données de télémétrie d’un appareil sur un point de terminaison IoT Hub peut initier des flux de travail d’application logique afin d’entreposer les données dans un objet Blob de Stockage Azure, d’envoyer des alertes par e-mail afin d’informer d’anomalies de données, de planifier la visite d’un technicien si un appareil signale un échec, etc.

## <a name="what-you-learn"></a>Contenu

Vous apprenez à créer une application logique qui connecte votre IoT Hub et votre boîte aux lettres pour la surveillance de la température et les notifications.

Le code client exécuté sur votre appareil définit une propriété d’application, `temperatureAlert`, dans chaque message de télémétrie qu’il envoie à votre hub IoT. Lorsque le code client détecte une température supérieure à 30 °C, il définit cette propriété sur `true` ; sinon, il définit la propriété sur `false`.

Les messages arrivant à votre IOT Hub ressemblent à ce qui suit, avec les données de télémétrie contenues dans le corps et la propriété `temperatureAlert` contenue dans les propriétés de l’application (les propriétés système ne sont pas affichées) :

```json
{
  "body": {
    "messageId": 18,
    "deviceId": "Raspberry Pi Web Client",
    "temperature": 27.796111770668457,
    "humidity": 66.77637926438427
  },
  "applicationProperties": {
    "temperatureAlert": "false"
  }
}
```

Pour en savoir plus sur le format de message de IOT Hub, consultez [Créer et lire des messages IOT Hub](iot-hub-devguide-messages-construct.md).

Dans cette rubrique, vous configurez le routage sur votre IOT Hub pour envoyer des messages dont la propriété `temperatureAlert` est `true` par rapport à un point de terminaison Service Bus. Vous configurez ensuite une application logique qui se déclenche sur les messages arrivant au point de terminaison Service Bus et vous envoie une notification par courrier électronique.

## <a name="what-you-do"></a>Procédure

* Créez un espace de noms Service Bus et ajoutez-lui une file d’attente.
* Ajoutez un point de terminaison personnalisé et une règle d’acheminement à votre hub IoT pour router des messages contenant une alerte de température à la file d’attente Service Bus.
* Créez, configurez et testez une application logique pour utiliser des messages de votre file d’attente Service Bus et envoyer des notifications par e-mail au destinataire souhaité.

## <a name="what-you-need"></a>Ce dont vous avez besoin

* Suivre le tutoriel [Simulateur en ligne Raspberry Pi](iot-hub-raspberry-pi-web-simulator-get-started.md) ou un des tutoriels de l’appareil, par exemple [Raspberry Pi avec node.js](iot-hub-raspberry-pi-kit-node-get-started.md). Ceux-ci couvrent les exigences suivantes :

  * Un abonnement Azure actif.
  * Une instance Azure IoT Hub associée à votre abonnement.
  * Une application cliente exécutée sur un appareil qui envoie des messages de télémétrie à votre hub IoT Azure.

## <a name="create-service-bus-namespace-and-queue"></a>Créer un espace de noms et une file d’attente Service Bus

Créez un espace de noms et une file d’attente Service Bus. Plus loin dans cette rubrique, vous allez créer une règle d’acheminement dans votre hub IoT pour diriger les messages contenant une alerte de température à la file d’attente Service Bus, où ils ne seront récupérés par une application logique et la déclencheront pour envoyer un e-mail de notification.

### <a name="create-a-service-bus-namespace"></a>Création d’un espace de noms Service Bus

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez **+ Créer une ressource** > **Intégration** > **Service Bus** .

1. Dans le volet **Créer un espace de noms** , fournissez les informations suivantes :

   **Name**  : Nom de l’espace de noms Service Bus. L’espace de noms doit être unique dans tout Azure.

   **Niveau tarifaire**  : Sélectionnez **De base** dans la liste déroulante. Le niveau de base est suffisant pour ce didacticiel.

   **Groupe de ressources**  : utilisez le même groupe de ressources que celui de votre hub IoT.

   **Emplacement** : Utilisez le même emplacement que celui utilisé par votre IoT Hub.

   ![Créer un espace de noms Service Bus dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/1-create-service-bus-namespace-azure-portal.png)

1. Sélectionnez **Create** (Créer). Attendez que le déploiement se termine avant de passer à l’étape suivante.

### <a name="add-a-service-bus-queue-to-the-namespace"></a>Ajouter une file d’attente Service Bus avec l’espace de noms

1. Ouvrez l’espace de noms Service Bus. La méthode la plus simple pour accéder à l’espace de noms Service Bus consiste à sélectionner **Groupe de ressources** dans le volet de ressources, à sélectionner votre groupe de ressources, puis à sélectionner l’espace de noms Service Bus dans la liste des ressources.

1. Dans le volet **Espace de noms Service Bus** , sélectionnez **+ File d’attente** .

1. Entrez un nom pour la file d’attente, puis sélectionnez **Créer** . Lorsque la file d’attente a été créée avec succès, le volet **Créer une file d’attente** se ferme.

   ![Ajouter une file d’attente Service Bus dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-service-bus-queue.png)

1. Dans le volet **Espace de noms Service Bus** , sous **Entités** , sélectionnez **Files d’attente** . Ouvrez la file d’attente Service Bus à partir de la liste, puis sélectionnez **Stratégies d’accès partagé** >  **+ Ajouter** .

1. Entrez un nom pour la stratégie, cochez **Gérer** , puis sélectionnez **Créer** .

   ![Ajouter une stratégie de file d’attente Service Bus dans le Portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/2-add-service-bus-queue-azure-portal.png)

## <a name="add-a-custom-endpoint-and-routing-rule-to-your-iot-hub"></a>Ajouter un point de terminaison personnalisé et une règle d’acheminement à votre IoT Hub

Ajoutez un point de terminaison personnalisé pour la file d’attente Service Bus à votre hub IoT et créez une règle d’acheminement des messages pour diriger les messages contenant une alerte de température à ce point de terminaison, où ils seront récupérés par votre application logique. La règle d’acheminement utilise une requête de routage, `temperatureAlert = "true"`, pour transférer des messages en fonction de la valeur de la propriété d’application `temperatureAlert` définie par le code client exécuté sur l’appareil. Pour en savoir plus, consultez [Requête de routage de messages en fonction des propriétés de message](./iot-hub-devguide-routing-query-syntax.md#message-routing-query-based-on-message-properties).

### <a name="add-a-custom-endpoint"></a>Ajouter un point de terminaison personnalisé

1. Ouvrez votre IoT Hub. La méthode la plus simple pour accéder au hub IoT consiste à sélectionner **Groupe de ressources** dans le volet de ressources, à sélectionner votre groupe de ressources, puis à sélectionner le hub IoT dans la liste des ressources.

1. Sous **Messagerie** , sélectionnez **Routage des messages** . Dans le volet **Routage des messages** , sélectionnez l’onglet **Points de terminaison personnalisés** , puis sélectionnez **+ Ajouter** . Dans la liste déroulante, sélectionnez **File d’attente Service Bus** .

   ![Capture d’écran mettant en évidence l’option de file d’attente Service Bus.](media/iot-hub-monitoring-notifications-with-azure-logic-apps/select-iot-hub-custom-endpoint.png)

1. Dans le volet **Ajouter un point de terminaison Service Bus** , entrez les informations suivantes :

   **Nom du point de terminaison** : Nom du point de terminaison.

   **Espace de noms Service Bus**  : Sélectionnez l’espace de noms que vous avez créé.

   **File d’attente Service Bus** : Sélectionnez la file d’attente que vous avez créée.

   ![Ajouter un point de terminaison à votre IoT Hub dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/3-add-iot-hub-endpoint-azure-portal.png)

1. Sélectionnez **Create** (Créer). Lorsque le point de terminaison est créé, passez à l’étape suivante.

### <a name="add-a-routing-rule"></a>Ajouter une règle de routage

1. Dans le volet **Routage des messages** , sélectionnez l’onglet **Routes** , puis sélectionnez **+ Ajouter** .

1. Dans le volet **Ajouter une route** , entrez les informations suivantes :

   **Name**  : Nom de la règle d’acheminement.

   **Point de terminaison** : Sélectionnez le point de terminaison que vous avez créé.

   **Source de données**  : Sélectionnez **Messages de télémétrie des appareils** .

   **Requête de routage**  : Entrez `temperatureAlert = "true"`.

   ![Ajouter une règle de routage dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/4-add-routing-rule-azure-portal.png)

1. Sélectionnez **Enregistrer** . Vous pouvez fermer le volet **Routage des messages** .

## <a name="create-and-configure-a-logic-app"></a>Créer et configurer une application logique

Dans la section précédente, vous avez configuré votre hub IoT pour router les messages contenant une alerte de température vers votre file d’attente Service Bus. Vous configurez maintenant une application logique pour surveiller la file d’attente Service Bus et envoyer une notification par e-mail si un message est ajouté à la file d’attente.

### <a name="create-a-logic-app"></a>Créer une application logique

1. Sélectionnez **Créer une ressource** > **Intégration**  > **Application logique** .

1. Entrez les informations suivantes :

   **Name**  : Nom de l’application logique.

   **Groupe de ressources**  : utilisez le même groupe de ressources que celui de votre hub IoT.

   **Emplacement** : Utilisez le même emplacement que celui utilisé par votre IoT Hub.

   ![Créer une application logique dans le Portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-a-logic-app.png)

1. Sélectionnez **Create** (Créer).

### <a name="configure-the-logic-app-trigger"></a>Configurer le déclencheur d’application logique

1. Ouvrez l’application logique. La méthode la plus simple pour accéder à l’application logique consiste à sélectionner **Groupe de ressources** dans le volet de ressources, à sélectionner votre groupe de ressources, puis à sélectionner l’application logique dans la liste des ressources. Lorsque vous sélectionnez l’application logique, le Concepteur d’applications logiques s’ouvre.

1. Dans le Concepteur d’applications logiques, faites défiler vers le bas jusqu’à **Modèles** et sélectionnez **Application logique vide** .

   ![Commencer avec une application logique vide dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/5-start-with-blank-logic-app-azure-portal.png)

1. Sélectionnez l’onglet **Tous** , puis sélectionnez **Service Bus** .

   ![Sélectionner Service Bus pour commencer à créer votre application logique dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/6-select-service-bus-when-creating-blank-logic-app-azure-portal.png)

1. Sous **Déclencheurs** , sélectionnez **Lorsqu’un ou plusieurs messages arrivent dans une file d’attente (saisie semi-automatique)** .

   ![Sélectionner le déclencheur de votre application logique dans le Portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/select-service-bus-trigger.png)

1. Créez une connexion Service Bus.
   1. Entrez un nom de connexion, puis sélectionnez votre espace de noms Service Bus dans la liste. L’écran suivant s’ouvre.

      ![Capture d’écran mettant en évidence l’option quand un ou plusieurs messages arrivent dans une file d’attente (saisie semi-automatique).](media/iot-hub-monitoring-notifications-with-azure-logic-apps/create-service-bus-connection-1.png)

   1. Sélectionnez la stratégie Service Bus (RootManageSharedAccessKey). Sélectionnez ensuite **Créer** .

      ![Créer une connexion Service Bus pour votre application logique dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/7-create-service-bus-connection-in-logic-app-azure-portal.png)

   1. Dans le dernier écran, pour **Nom de la file d’attente** , sélectionnez la file d’attente que vous avez créée dans la liste déroulante. Entrez `175` pour **Nombre maximal de messages** .

      ![Spécifier le nombre maximal de messages pour les connexions Service Bus dans votre application logique](media/iot-hub-monitoring-notifications-with-azure-logic-apps/8-specify-maximum-message-count-for-service-bus-connection-logic-app-azure-portal.png)

   1. Sélectionnez **Enregistrer** en haut du Concepteur d’applications logiques pour enregistrer vos modifications.

### <a name="configure-the-logic-app-action"></a>Configurer l’action d’application logique

1. Créez une connexion de service SMTP.

   1. Sélectionnez **Nouvelle étape** . Dans **Choisir une action** , sélectionnez l’onglet **Tous** .

   1. Saisissez `smtp` dans la zone de recherche, sélectionnez le service **SMTP** dans le résultat de la recherche, puis sélectionnez **Envoyer un e-mail** .

      ![Créer une connexion SMTP dans votre application logique dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/9-create-smtp-connection-logic-app-azure-portal.png)

   1. Entrez les informations SMTP de votre boîte aux lettres, puis sélectionnez **Créer** .

      ![Entrer les informations de connexion SMTP dans votre application logique dans le portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/10-enter-smtp-connection-info-logic-app-azure-portal.png)

      Obtenez les informations SMTP pour [Hotmail/Outlook.com](https://support.office.com/article/Add-your-Outlook-com-account-to-another-mail-app-73f3b178-0009-41ae-aab1-87b80fa94970), [Gmail](https://support.google.com/a/answer/176600?hl=en) et [Yahoo Mail](https://help.yahoo.com/kb/SLN4075.html).

      > [!NOTE]
      > Vous devrez peut-être désactiver le protocole TLS/SSL pour établir la connexion. Si c’est le cas et si vous souhaitez réactiver TLS une fois la connexion établie, consultez l’étape facultative à la fin de cette section.

   1. Dans la liste déroulante **Ajouter un nouveau paramètre** de l’étape **Envoyer un e-mail** , sélectionnez **De** , **À** , **Objet** et **Corps** . Cliquez ou appuyez n’importe où sur l’écran pour fermer la zone de sélection.

      ![Choisir les champs de l’e-mail de connexion SMTP](media/iot-hub-monitoring-notifications-with-azure-logic-apps/smtp-connection-choose-fields.png)

   1. Entrez votre adresse de messagerie pour **De** et **À** , et `High temperature detected` pour **Objet** et **Corps** . Si la boîte de dialogue **Ajoutez du contenu dynamique des applications et des connecteurs utilisés dans ce flux** s’ouvre, sélectionnez **Masquer** pour la fermer. Vous n’utilisez pas de contenu dynamique dans ce didacticiel.

      ![Remplir les champs de l’e-mail de connexion SMTP](media/iot-hub-monitoring-notifications-with-azure-logic-apps/fill-in-smtp-connection-fields.png)

   1. Sélectionnez **Enregistrer** pour enregistrer la connexion SMTP.

1. (Facultatif) Si vous deviez désactiver le protocole TLS pour établir une connexion avec votre fournisseur de messagerie et que vous souhaitez le réactiver, procédez comme suit :

   1. Dans le volet **Application logique** , sous **Outils de développement** , sélectionnez **Connexions d’API** .

   1. Dans la liste des connexions d’API, sélectionnez la connexion SMTP.

   1. Dans le volet **Connexion d’API SMTP** , sous **Général** , sélectionnez **Modifier une connexion d’API** .

   1. Dans le volet **Modifier une connexion d’API** , sélectionnez **Activer le protocole SSL ?** , entrez à nouveau le mot de passe de votre compte de messagerie, puis sélectionnez **Enregistrer** .

      ![Modifier une connexion d’API SMTP dans votre application logique dans le Portail Azure](media/iot-hub-monitoring-notifications-with-azure-logic-apps/re-enable-smtp-connection-ssl.png)

Votre application logique est maintenant prête à traiter les alertes de température de la file d’attente Service Bus et à envoyer des notifications à votre compte de messagerie.

## <a name="test-the-logic-app"></a>Tester l’application logique

1. Démarrez l’application cliente sur votre appareil.

1. Si vous utilisez un appareil physique, rapprochez une source de chaleur du détecteur de chaleur jusqu’à ce que la température dépasse 30 degrés C. Si vous utilisez le simulateur en ligne, le code client générera de manière aléatoire des messages de télémétrie supérieure à 30 °C.

1. Vous devriez recevoir des e-mails de notification envoyés par l’application logique.

   > [!NOTE]
   > Il se peut que votre fournisseur de services de messagerie doive vérifier l’identité de l’expéditeur pour s’assurer que vous êtes bien celui qui envoie l’e-mail.

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé avec succès une application logique qui connecte votre IoT Hub et votre boîte aux lettres pour la surveillance de la température et les notifications.

[!INCLUDE [iot-hub-get-started-next-steps](../../includes/iot-hub-get-started-next-steps.md)]
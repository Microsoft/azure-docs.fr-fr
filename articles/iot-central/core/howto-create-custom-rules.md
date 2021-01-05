---
title: Étendre Azure IoT Central avec des notifications et des règles personnalisées | Microsoft Docs
description: En tant que développeur de solutions, configurez une application IoT Central afin d’envoyer des notifications par e-mail lorsqu’un appareil cesse d’envoyer des données de télémétrie. Cette solution utilise Azure Stream Analytics, Azure Functions et SendGrid.
author: dominicbetts
ms.author: dobett
ms.date: 12/02/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
ms.custom: mvc, devx-track-csharp
manager: philmea
ms.openlocfilehash: f6c8272f736e2f83b4d33f3d61ce83356aa40e5d
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92126754"
---
# <a name="extend-azure-iot-central-with-custom-rules-using-stream-analytics-azure-functions-and-sendgrid"></a>Étendre Azure IoT Central avec des règles personnalisées à l’aide de Stream Analytics, d’Azure Functions et de SendGrid

Ce guide pratique vous montre comment étendre votre application IoT Central en tant que développeur de solutions avec des notifications et des règles personnalisées. L’exemple montre l’envoi d’une notification à un opérateur lorsqu’un appareil cesse d’envoyer des données de télémétrie. La solution utilise une requête [Azure Stream Analytics](../../stream-analytics/index.yml) pour détecter le moment où un appareil a arrêté d’envoyer des données de télémétrie. Le travail Stream Analytics utilise [Azure Functions](../../azure-functions/index.yml) pour envoyer des notifications par e-mail via [SendGrid](https://sendgrid.com/docs/for-developers/partners/microsoft-azure/).

Ce guide pratique vous montre comment étendre IoT Central au-delà de ce qu’il peut déjà faire avec les actions et règles intégrées.

Dans ce guide pratique, vous allez apprendre à effectuer les opérations suivantes :

* Transmettre des données de télémétrie en continu à partir d’une application IoT Central à l’aide de l’*exportation de données continue*.
* Créer une requête Stream Analytics qui détecte le moment où un appareil a arrêté d’envoyer des données.
* Envoyer une notification par e-mail à l’aide des services Azure Functions et SendGrid.

## <a name="prerequisites"></a>Conditions préalables requises

Pour effectuer les étapes décrites dans ce guide pratique, vous avez besoin d’un abonnement Azure actif.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

### <a name="iot-central-application"></a>Application IoT Central

Créez une application IoT Central sur le site web [Gestionnaire d’applications Azure IoT Central](https://aka.ms/iotcentral) avec les paramètres suivants :

| Paramètre | Valeur |
| ------- | ----- |
| Plan tarifaire | standard |
| Modèle d’application | Analytique en magasin – Monitorage des conditions |
| Nom de l'application | Acceptez la valeur par défaut ou choisissez votre propre nom |
| URL | Acceptez la valeur par défaut ou choisissez votre propre préfixe d’URL unique |
| Répertoire | Votre locataire Azure Active Directory |
| Abonnement Azure | Votre abonnement Azure |
| Région | Région la plus proche |

Les exemples et captures d’écran de cet article concernent la région **États-Unis**. Choisissez un emplacement proche de vous et assurez-vous de créer toutes vos ressources dans la même région.

Ce modèle d’application comprend deux appareils à thermostat simulés qui envoient des données de télémétrie.

### <a name="resource-group"></a>Resource group

Utilisez le [portail Azure pour créer un groupe de ressources](https://portal.azure.com/#create/Microsoft.ResourceGroup) appelé **DetectStoppedDevices** afin de contenir les autres ressources que vous créez. Créez vos ressources Azure dans le même emplacement que votre application IoT Central.

### <a name="event-hubs-namespace"></a>Espace de noms Event Hubs

Utilisez le [portail Azure pour créer un espace de noms Event Hubs](https://portal.azure.com/#create/Microsoft.EventHub) avec les paramètres suivants :

| Paramètre | Valeur |
| ------- | ----- |
| Name    | Choisissez le nom de votre espace de noms |
| Niveau tarifaire | De base |
| Subscription | Votre abonnement |
| Resource group | DetectStoppedDevices |
| Location | USA Est |
| Unités de débit | 1 |

### <a name="stream-analytics-job"></a>Travail Stream Analytics

Utilisez le [portail Azure pour créer un travail Stream Analytics](https://portal.azure.com/#create/Microsoft.StreamAnalyticsJob) avec les paramètres suivants :

| Paramètre | Valeur |
| ------- | ----- |
| Name    | Choisissez le nom de votre travail |
| Subscription | Votre abonnement |
| Resource group | DetectStoppedDevices |
| Location | USA Est |
| Environnement d’hébergement | Cloud |
| Unités de diffusion en continu | 3 |

### <a name="function-app"></a>Conteneur de fonctions

Utilisez le [portail Azure pour créer une application de fonction](https://portal.azure.com/#create/Microsoft.FunctionApp) avec les paramètres suivants :

| Paramètre | Valeur |
| ------- | ----- |
| Nom de l’application    | Choisissez le nom de votre application de fonction |
| Subscription | Votre abonnement |
| Resource group | DetectStoppedDevices |
| Système d''exploitation | Windows |
| Plan d’hébergement | Plan de consommation |
| Location | USA Est |
| Pile d’exécution | .NET |
| Stockage | Création |

### <a name="sendgrid-account"></a>Compte SendGrid

Utilisez le [portail Azure pour créer un compte SendGrid](https://portal.azure.com/#create/Sendgrid.sendgrid) avec les paramètres suivants :

| Paramètre | Valeur |
| ------- | ----- |
| Name    | Choisissez le nom de votre compte SendGrid |
| Mot de passe | Créez un mot de passe |
| Subscription | Votre abonnement |
| Resource group | DetectStoppedDevices |
| Niveau tarifaire | F1 Gratuit |
| Informations de contact | Remplissez les informations requises |

Une fois que vous avez créé toutes les ressources nécessaires, votre groupe de ressources **DetectStoppedDevices** doit ressembler à la capture d’écran suivante :

![Détecter le groupe de ressources des appareils arrêtés](media/howto-create-custom-rules/resource-group.png)

## <a name="create-an-event-hub"></a>Créer un hub d’événements

Vous pouvez configurer une application IoT Central pour exporter en continu des données de télémétrie vers un Event Hub. Dans cette section, vous allez créer un Event Hub pour recevoir des données de télémétrie depuis votre application IoT Central. L’Event Hub fournit les données de télémétrie à votre travail Stream Analytics afin qu’elles soient traitées.

1. Dans le portail Azure, accédez à votre espace de noms Event Hubs et sélectionnez **+ Event Hub**.
1. Nommez votre Event Hub **centralexport**, puis sélectionnez **Créer**.

Votre espace de noms Event Hubs se présente comme la capture d’écran suivante :

![Espace de noms Event Hubs](media/howto-create-custom-rules/event-hubs-namespace.png)

## <a name="get-sendgrid-api-key"></a>Obtenir un clé API SendGrid

Votre application de fonction a besoin d’une clé API SendGrid pour envoyer des e-mails. Pour Créer une clé API SendGrid, procédez comme suit :

1. Dans le portail Azure, accédez à votre compte SendGrid. Choisissez ensuite **Gérer** pour accéder à votre compte SendGrid.
1. Dans votre compte SendGrid, choisissez **Paramètres**, puis **Clés API**. Sélectionnez **Créer une clé API** :

    ![Créer une clé API SendGrid](media/howto-create-custom-rules/sendgrid-api-keys.png)

1. Sur la page **Créer une clé API**, créez une clé nommée **AzureFunctionAccess** disposant des autorisations **Accès total**.
1. Notez la clé API, car vous en aurez besoin lorsque vous configurerez votre application de fonction.

## <a name="define-the-function"></a>Définir la fonction

Cette solution utilise une application Azure Functions pour envoyer une notification par e-mail lorsque le travail Stream Analytics détecte un appareil arrêté. Pour créer une application de fonction, procédez comme suit :

1. Dans le portail Azure, accédez à l’instance **App Service** d’instance dans le groupe de ressources **DetectStoppedDevices**.
1. Sélectionnez **+** pour créer une fonction.
1. Sur la page **CHOISIR UN ENVIRONNEMENT DE DÉVELOPPEMENT**, choisissez **Dans le portail** , puis sélectionnez **Continuer**.
1. Sur la page **CRÉER UNE FONCTION**, choisissez **Webhook + API** , puis sélectionnez **Créer**.

Le portail crée une fonction par défaut intitulée **HttpTrigger1** :

![Fonction de déclencheur HTTP par défaut](media/howto-create-custom-rules/default-function.png)

### <a name="configure-function-bindings"></a>Configurer les liaisons de fonction

Pour envoyer des e-mails avec SendGrid, vous devez configurer les liaisons de votre fonction comme suit :

1. Sélectionnez **Intégrer**, puis choisissez la sortie **HTTP ($return)** cliquez sur **Supprimer**.
1. Choisissez **+ Nouvelle sortie**, puis **SendGrid** et enfin **Sélectionner**. Choisissez **Installer** pour installer l’extension SendGrid.
1. Une fois l’installation terminée, sélectionnez **Utiliser la valeur renvoyée de la fonction**. Ajouter une **adresse de destination** valide pour recevoir des notifications par e-mail.  Ajouter une **adresse d’expédition** valide à utiliser comme expéditeur de l’e-mail.
1. Sélectionnez **nouveau** en regard du **paramètre d’application de la clé API SendGrid**. Entrez **SendGridAPIKey** pour la clé, puis saisissez la clé API SendGrid que vous avez notée précédemment comme valeur. Sélectionnez ensuite **Créer**.
1. Choisissez **Enregistrer** pour enregistrer les liaisons SendGrid pour votre fonction.

Les paramètres intégrés doivent ressembler à ceux de la capture d’écran suivante :

![Intégrations d’applications de fonction](media/howto-create-custom-rules/function-integrate.png)

### <a name="add-the-function-code"></a>Ajouter le code de fonction

Pour implémenter votre fonction, ajoutez le code en C# afin d’analyser la requête HTTP entrante et d’envoyer les e-mails comme suit :

1. Choisissez la fonction **HttpTrigger1** dans votre application de fonction et remplacez le code en C# par le code suivant :

    ```csharp
    #r "Newtonsoft.Json"
    #r "..\bin\SendGrid.dll"

    using System;
    using SendGrid.Helpers.Mail;
    using Microsoft.Azure.WebJobs.Host;
    using Microsoft.AspNetCore.Mvc;
    using Microsoft.Extensions.Primitives;
    using Newtonsoft.Json;

    public static SendGridMessage Run(HttpRequest req, ILogger log)
    {
        string requestBody = new StreamReader(req.Body).ReadToEnd();
        log.LogInformation(requestBody);
        var notifications = JsonConvert.DeserializeObject<IList<Notification>>(requestBody);

        SendGridMessage message = new SendGridMessage();
        message.Subject = "Contoso device notification";

        var content = "The following device(s) have stopped sending telemetry:<br/><br/><table><tr><th>Device ID</th><th>Time</th></tr>";
        foreach(var notification in notifications) {
            log.LogInformation($"No message received - Device: {notification.deviceid}, Time: {notification.time}");
            content += $"<tr><td>{notification.deviceid}</td><td>{notification.time}</td></tr>";
        }
        content += "</table>";
        message.AddContent("text/html", content);

        return message;
    }

    public class Notification
    {
        public string deviceid { get; set; }
        public string time { get; set; }
    }
    ```

    Jusqu’à ce que vous enregistriez le nouveau code, le message d’erreur suivant peut s’afficher.

1. Sélectionnez **Enregistrer** pour enregistrer la fonction.

### <a name="test-the-function-works"></a>Tester la fonction pour vérifier qu’elle fonctionne

Pour tester la fonction dans le portail, commencez par sélectionner **Journaux** en bas de l’éditeur de code. Choisissez ensuite **Tester** à droite de l’éditeur de code. Dans le champ **Corps de la demande**, utilisez le JSON suivant :

```json
[{"deviceid":"test-device-1","time":"2019-05-02T14:23:39.527Z"},{"deviceid":"test-device-2","time":"2019-05-02T14:23:50.717Z"},{"deviceid":"test-device-3","time":"2019-05-02T14:24:28.919Z"}]
```

Les messages du journal de la fonction s’affichent dans le panneau **Journaux** :

![Sortie du journal de la fonction](media/howto-create-custom-rules/function-app-logs.png)

Après quelques minutes, l’adresse de **destination** recevra un e-mail avec le contenu suivant :

```txt
The following device(s) have stopped sending telemetry:

Device ID    Time
test-device-1    2019-05-02T14:23:39.527Z
test-device-2    2019-05-02T14:23:50.717Z
test-device-3    2019-05-02T14:24:28.919Z
```

## <a name="add-stream-analytics-query"></a>Ajouter une requête Stream Analytics

La solution utilise une requête Stream Analytics pour détecter le moment où un appareil a arrêté d’envoyer des données de télémétrie pendant plus de 120 secondes. La requête utilise les données de télémétrie de l’Event Hub comme entrée. Le travail envoie les résultats de la requête à l’application de fonction. Dans cette section, vous configurez un travail Azure Stream Analytics :

1. Sur le Portail Azure, accédez à votre travail Stream Analytics ; sous **Topologie des travaux**, sélectionnez **Entrées**, puis **+ Ajouter une entrée de flux** et enfin **Event Hub**.
1. Utilisez les informations dans le tableau suivant pour configurer l’entrée à l’aide de l’Event Hub que vous avez créé précédemment, puis choisissez **Enregistrer** :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Alias d’entrée | centraltelemetry |
    | Subscription | Votre abonnement |
    | Espace de noms Event Hub | Espace de noms de votre Event Hub |
    | Nom de l’Event Hub | Utiliser le nom existant : **centralexport** |

1. Sous **Topologie des travaux**, sélectionnez **Sorties**, choisissez **+ Ajouter**, puis cliquez sur **Fonction Azure**.
1. Utilisez les informations du tableau suivant pour configurer la sortie, puis cliquez sur **Enregistrer** :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Alias de sortie | emailnotification |
    | Subscription | Votre abonnement |
    | Conteneur de fonctions | Votre application de fonction |
    | Fonction  | HttpTrigger1 |

1. Sous **Topologie des travaux**, sélectionnez **Requête**, puis remplacez la requête existante par le SQL suivant :

    ```sql
    with
    LeftSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid1, 
        EventEnqueuedUtcTime AS time1
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    ),
    RightSide as
    (
        SELECT
        -- Get the device ID from the message metadata and create a column
        GetMetadataPropertyValue([centraltelemetry], '[EventHub].[IoTConnectionDeviceId]') as deviceid2, 
        EventEnqueuedUtcTime AS time2
        FROM
        -- Use the event enqueued time for time-based operations
        [centraltelemetry] TIMESTAMP BY EventEnqueuedUtcTime
    )

    SELECT
        LeftSide.deviceid1 as deviceid,
        LeftSide.time1 as time
    INTO
        [emailnotification]
    FROM
        LeftSide
        LEFT OUTER JOIN
        RightSide 
        ON
        LeftSide.deviceid1=RightSide.deviceid2 AND DATEDIFF(second,LeftSide,RightSide) BETWEEN 1 AND 120
        where
        -- Find records where a device didn't send a message 120 seconds
        RightSide.deviceid2 is NULL
    ```

1. Sélectionnez **Enregistrer**.
1. Pour lancer le travail Stream Analytics, choisissez **Vue d’ensemble**, **Démarrer**, puis **Maintenant** et enfin **Démarrer** :

    ![Stream Analytics](media/howto-create-custom-rules/stream-analytics.png)

## <a name="configure-export-in-iot-central"></a>Configurer l’exportation dans IoT Central

Sur le site web [Gestionnaire d’applications Azure IoT Central](https://aka.ms/iotcentral), accédez à l’application IoT Central que vous avez créée à partir du modèle Contoso. Dans cette section, vous allez configurer l’application pour diffuser les données de télémétrie depuis ses appareils simulés vers votre Event Hub. Pour configurer l’exportation :

1. Accédez à la page **Exportation de données**, sélectionnez **+ Nouveau**, puis **Azure Event Hubs**.
1. Utilisez les paramètres suivants pour configurer l’exportation, puis sélectionnez **Enregistrer** :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom d’affichage | Exporter vers Event Hubs |
    | activé | Il en va |
    | Espace de noms Event Hubs | Le nom de votre espace de noms Event Hubs |
    | Event Hub | centralexport |
    | Mesures | Il en va |
    | Appareils | Off |
    | Modèles d’appareil | Off |

![Configuration de l’exportation de données continue](media/howto-create-custom-rules/cde-configuration.png)

Attendez que l’état de l’exportation soit **En cours d’exécution** avant de continuer.

## <a name="test"></a>Test

Pour tester la solution, vous pouvez désactiver l’exportation de données continue depuis IoT Central vers les appareils arrêtés simulés :

1. Dans votre application IoT Central, accédez à la page **Exportation de données** et sélectionnez la configuration d’exportation **Exporter vers Event Hubs**.
1. Définissez l’option **Activer** sur **Non**, puis cliquez sur **Enregistrer**.
1. Après au moins deux minutes, l’adresse de **destination** recevra au moins un e-mail similaire à l’exemple de contenu suivant :

    ```txt
    The following device(s) have stopped sending telemetry:

    Device ID         Time
    Thermostat-Zone1  2019-11-01T12:45:14.686Z
    ```

## <a name="tidy-up"></a>Nettoyer

Pour nettoyer à la suite de cette procédure et éviter les coûts inutiles, supprimez le groupe de ressources **DetectStoppedDevices** dans le portail Azure.

Vous pouvez supprimer l’application IoT Central depuis la page **Gestion** au sein de l’application.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide pratique, vous avez appris à effectuer les opérations suivantes :

* Transmettre des données de télémétrie en continu à partir d’une application IoT Central à l’aide de l’*exportation de données continue*.
* Créer une requête Stream Analytics qui détecte le moment où un appareil a arrêté d’envoyer des données.
* Envoyer une notification par e-mail à l’aide des services Azure Functions et SendGrid.

Maintenant que vous savez comment créer des notifications et des règles personnalisées, il est conseillé de suivre l’étape suivante qui consiste à apprendre comment [Étendre Azure IoT Central avec une analyse personnalisée](howto-create-custom-analytics.md).
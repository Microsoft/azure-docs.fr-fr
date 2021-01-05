---
title: Utilisation d’Azure Notification Hubs avec Java
description: Découvrez comment utiliser Azure Notification Hubs à partir d’un serveur principal Java.
services: notification-hubs
documentationcenter: ''
author: sethmanheim
manager: femila
editor: jwargo
ms.assetid: 4c3f966d-0158-4a48-b949-9fa3666cb7e4
ms.service: notification-hubs
ms.workload: mobile
ms.tgt_pltfrm: java
ms.devlang: java
ms.topic: article
ms.date: 01/04/2019
ms.author: sethm
ms.reviewer: jowargo
ms.lastreviewed: 01/04/2019
ms.custom: devx-track-java
ms.openlocfilehash: 09553f587916e8204541b36f259a450d72c1b270
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87322939"
---
# <a name="how-to-use-notification-hubs-from-java"></a>Utilisation de Notification Hubs à partir de Java

[!INCLUDE [notification-hubs-backend-how-to-selector](../../includes/notification-hubs-backend-how-to-selector.md)]

Cette rubrique décrit les principales fonctionnalités du nouveau kit de développement logiciel (SDK) Java officiel pour Azure Notification Hub, entièrement pris en charge.
Il s’agit d’un projet open source et vous pouvez afficher tout le code du Kit de développement logiciel (SDK) sur [Kit SDK Java].

Vous pouvez accéder à toutes les fonctionnalités de Notification Hubs à partir d’un serveur principal Java/PHP/Python/Ruby en utilisant l’interface REST de Notification Hub, comme décrit dans la rubrique MSDN [API REST de Notification Hub](/previous-versions/azure/reference/dn223264(v=azure.100)). Ce kit de développement logiciel Java fournit un wrapper fin par rapport aux interfaces REST dans Java.

Le SDK prend en charge :

* CRUD sur Notification Hubs
* CRUD sur les inscriptions
* Gestion de l’installation
* Importation/exportation des inscriptions
* Envois réguliers
* Envois planifiés
* Opérations asynchrones via Java NIO
* Plateformes prises en charge : APNs (iOS), FCM (Android), WNS (applications Windows Store), MPNS (Windows Phone), ADM (Amazon Kindle Fire), Baidu (Android sans services Google)

## <a name="sdk-usage"></a>Utilisation du kit de développement logiciel (SDK)

### <a name="compile-and-build"></a>Compilation et génération

Utilisation de [Maven]

Génération de :

```cmd
mvn package
```

## <a name="code"></a>Code

### <a name="notification-hub-cruds"></a>CRUD de Notification Hub

**Création d’un NamespaceManager :**

```java
NamespaceManager namespaceManager = new NamespaceManager("connection string")
```

**Création d’un concentrateur de notification :**

```java
NotificationHubDescription hub = new NotificationHubDescription("hubname");
hub.setWindowsCredential(new WindowsCredential("sid","key"));
hub = namespaceManager.createNotificationHub(hub);
```

 OR

```java
hub = new NotificationHub("connection string", "hubname");
```

**Obtention d’un hub de notification :**

```java
hub = namespaceManager.getNotificationHub("hubname");
```

**Mise à jour d’un concentrateur de notification :**

```java
hub.setMpnsCredential(new MpnsCredential("mpnscert", "mpnskey"));
hub = namespaceManager.updateNotificationHub(hub);
```

**Suppression d’un hub de notification :**

```java
namespaceManager.deleteNotificationHub("hubname");
```

### <a name="registration-cruds"></a>CRUID d’inscription

**Création d’un client de hub de notification :**

```java
hub = new NotificationHub("connection string", "hubname");
```

**Création d’une inscription Windows :**

```java
WindowsRegistration reg = new WindowsRegistration(new URI(CHANNELURI));
reg.getTags().add("myTag");
reg.getTags().add("myOtherTag");
hub.createRegistration(reg);
```

**Création d’une inscription iOS :**

```java
AppleRegistration reg = new AppleRegistration(DEVICETOKEN);
reg.getTags().add("myTag");
reg.getTags().add("myOtherTag");
hub.createRegistration(reg);
```

De même, vous pouvez créer des inscriptions pour Android (FCM), Windows Phone (MPNS) et Kindle Fire (ADM).

**Création de modèles d’inscription :**

```java
WindowsTemplateRegistration reg = new WindowsTemplateRegistration(new URI(CHANNELURI), WNSBODYTEMPLATE);
reg.getHeaders().put("X-WNS-Type", "wns/toast");
hub.createRegistration(reg);
```

**Création d’inscriptions à l’aide du modèle createRegistrationId + upsert :**

Supprime les doublons dus à des réponses perdues, si les ID d’inscription sont stockés sur l’appareil :

```java
String id = hub.createRegistrationId();
WindowsRegistration reg = new WindowsRegistration(id, new URI(CHANNELURI));
hub.upsertRegistration(reg);
```

**Mise à jour d’inscriptions :**

```java
hub.updateRegistration(reg);
```

**Suppression d’inscriptions :**

```java
hub.deleteRegistration(regid);
```

**Interrogation d’inscriptions :**

* **Obtention d’une inscription unique :**

```java
hub.getRegistration(regid);
```

* **Obtention de toutes les inscriptions du hub :**

```java
hub.getRegistrations();
```

* **Obtention des inscriptions avec balise :**

```java
hub.getRegistrationsByTag("myTag");
```

* **Obtention des inscriptions par canal :**

```java
hub.getRegistrationsByChannel("devicetoken");
```

Toutes les requêtes de collection prennent en charge les jetons $top et de continuation.

### <a name="installation-api-usage"></a>Utilisation de l’API d’installation

L’API d’installation est un autre mécanisme de gestion des inscriptions. Au lieu de maintenir plusieurs inscriptions, qui ne sont pas simples et peuvent facilement être mal effectuées, il est maintenant possible d’utiliser un objet d’installation UNIQUE.

L’installation contient tout ce dont vous avez besoin : un canal Push (jeton d’appareil), des balises, des modèles, des vignettes secondaires (pour WNS et APN). Vous n’avez plus besoin d’appeler le service pour obtenir l’ID : il suffit de générer un GUID ou tout autre identificateur, de le conserver sur l’appareil et de l’envoyer à votre serveur principal avec le canal Push (jeton d’appareil).

Sur le serveur principal, effectuez simplement un appel unique à `CreateOrUpdateInstallation`. Ce dernier étant totalement idempotent, n’hésitez pas à réessayer si nécessaire.

Voici un exemple pour Amazon Kindle Fire :

```java
Installation installation = new Installation("installation-id", NotificationPlatform.Adm, "adm-push-channel");
hub.createOrUpdateInstallation(installation);
```

Si vous souhaitez le mettre à jour :

```java
installation.addTag("foo");
installation.addTemplate("template1", new InstallationTemplate("{\"data\":{\"key1\":\"$(value1)\"}}","tag-for-template1"));
installation.addTemplate("template2", new InstallationTemplate("{\"data\":{\"key2\":\"$(value2)\"}}","tag-for-template2"));
hub.createOrUpdateInstallation(installation);
```

Pour des scénarios avancés, utilisez les capacités de mise à jour partielle qui permettent de modifier uniquement des propriétés particulières de l’objet d’installation. La mise à jour partielle est un sous-ensemble d’opérations de correction JSON que vous pouvez exécuter sur l’objet d’installation.

```java
PartialUpdateOperation addChannel = new PartialUpdateOperation(UpdateOperationType.Add, "/pushChannel", "adm-push-channel2");
PartialUpdateOperation addTag = new PartialUpdateOperation(UpdateOperationType.Add, "/tags", "bar");
PartialUpdateOperation replaceTemplate = new PartialUpdateOperation(UpdateOperationType.Replace, "/templates/template1", new InstallationTemplate("{\"data\":{\"key3\":\"$(value3)\"}}","tag-for-template1")).toJson());
hub.patchInstallation("installation-id", addChannel, addTag, replaceTemplate);
```

Supprimer l’Installation :

```java
hub.deleteInstallation(installation.getInstallationId());
```

`CreateOrUpdate`, `Patch` et `Delete` sont finalement cohérents avec `Get`. L’opération demandée rejoint simplement la file d’attente système lors de l’appel et est exécutée en arrière-plan. Get n’est pas conçu pour le scénario d’exécution principal, mais uniquement pour le débogage et le dépannage. Il est étroitement limité par le service.

Le flux d’envoi pour les Installations est identique à celui des Inscriptions. Pour cibler la notification sur l’installation particulière, utilisez simplement la balise « InstallationId:{desired-id} ». Dans ce cas, le code est :

```java
Notification n = Notification.createWindowsNotification("WNS body");
hub.sendNotification(n, "InstallationId:{installation-id}");
```

Pour l’un des modèles :

```java
Map<String, String> prop =  new HashMap<String, String>();
prop.put("value3", "some value");
Notification n = Notification.createTemplateNotification(prop);
hub.sendNotification(n, "InstallationId:{installation-id} && tag-for-template1");
```

### <a name="schedule-notifications-available-for-standard-tier"></a>planification des notifications (disponible pour le niveau STANDARD)

Similaire à l’envoi ordinaire, mais avec un paramètre supplémentaire : scheduledTime, qui indique à quel moment la notification doit être remise. Le service accepte n’importe quel point dans le temps entre maintenant + 5 minutes et maintenant + 7 jours.

**Planification d’une notification native Windows :**

```java
Calendar c = Calendar.getInstance();
c.add(Calendar.DATE, 1);
Notification n = Notification.createWindowsNotification("WNS body");
hub.scheduleNotification(n, c.getTime());
```

### <a name="importexport-available-for-standard-tier"></a>Importation/exportation (disponible pour le niveau STANDARD)

Vous pouvez être amené à effectuer une opération en bloc vis-à-vis des inscriptions. C’est généralement le cas pour l’intégration à un autre système ou pour un correctif massif afin de mettre à jour les balises. Nous vous déconseillons d’utiliser des flux Get/Update si cela concerne des milliers d’inscriptions. La fonction d’importation/exportation du système est conçue pour couvrir ce scénario. Vous fournissez un accès à un conteneur d’objets blob dans votre compte de stockage en tant que source de données entrantes et emplacement de sortie.

**Soumettre un travail d’exportation :**

```java
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ExportRegistrations);
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);
```

**Soumettre un travail d’importation :**

```java
NotificationHubJob job = new NotificationHubJob();
job.setJobType(NotificationHubJobType.ImportCreateRegistrations);
job.setImportFileUri("input file uri with SAS signature");
job.setOutputContainerUri("container uri with SAS signature");
job = hub.submitNotificationHubJob(job);
```

**Attendre qu’un travail soit terminé :**

```java
while(true){
    Thread.sleep(1000);
    job = hub.getNotificationHubJob(job.getJobId());
    if(job.getJobStatus() == NotificationHubJobStatus.Completed)
        break;
}
```

**Obtenez toutes les tâches :**

```java
List<NotificationHubJob> jobs = hub.getAllNotificationHubJobs();
```

**URI avec signature SAP :**

 c’est l’URL d’un fichier blob ou d’un conteneur d’objets blob et d’un jeu de paramètres tels que les autorisations et l’heure d’expiration, ainsi que la signature de toutes ces opérations effectuées à l’aide de la clé SAP du compte. Le Kit de développement logiciel (SDK) Azure Storage Java dispose de nombreuses fonctionnalités, notamment la création de ces URI. Autrement, examinez simplement la classe de test `ImportExportE2E` (à partir de l’emplacement GitHub) qui possède une implémentation basique et compacte de l’algorithme de signature.

### <a name="send-notifications"></a>Envoi de notifications

L’objet de Notification est simplement un corps avec des en-têtes. Certaines méthodes utilitaires vous aident à créer les objets de notifications natives et les modèles.

* **Windows Store et Windows Phone 8.1 (non-Silverlight)**

```java
String toast = "<toast><visual><binding template=\"ToastText01\"><text id=\"1\">Hello from Java!</text></binding></visual></toast>";
Notification n = Notification.createWindowsNotification(toast);
hub.sendNotification(n);
```

* **iOS**

```java
String alert = "{\"aps\":{\"alert\":\"Hello from Java!\"}}";
Notification n = Notification.createAppleNotification(alert);
hub.sendNotification(n);
```

* **Android**

```java
String message = "{\"data\":{\"msg\":\"Hello from Java!\"}}";
Notification n = Notification.createFcmNotification(message);
hub.sendNotification(n);
```

* **Windows Phone 8.0 et 8.1 Silverlight**

```java
String toast = "<?xml version=\"1.0\" encoding=\"utf-8\"?>" +
            "<wp:Notification xmlns:wp=\"WPNotification\">" +
                "<wp:Toast>" +
                    "<wp:Text1>Hello from Java!</wp:Text1>" +
                "</wp:Toast> " +
            "</wp:Notification>";
Notification n = Notification.createMpnsNotification(toast);
hub.sendNotification(n);
```

* **Kindle Fire**

```java
String message = "{\"data\":{\"msg\":\"Hello from Java!\"}}";
Notification n = Notification.createAdmNotification(message);
hub.sendNotification(n);
```

* **Envoi à des balises**
  
```java
Set<String> tags = new HashSet<String>();
tags.add("boo");
tags.add("foo");
hub.sendNotification(n, tags);
```

* **Envoi à l’expression de balise**

```java
hub.sendNotification(n, "foo && ! bar");
```

* **Envoyer une notification modèle**

```java
Map<String, String> prop =  new HashMap<String, String>();
prop.put("prop1", "v1");
prop.put("prop2", "v2");
Notification n = Notification.createTemplateNotification(prop);
hub.sendNotification(n);
```

L’exécution de votre code Java produit normalement une notification qui apparaît sur votre appareil cible.

## <a name="next-steps"></a><a name="next-steps"></a>Étapes suivantes

Cette rubrique vous a montré comment créer un client REST Java simple pour Notification Hubs. À partir de là, vous pouvez :

* Téléchargez la version complète du [Kit SDK Java], qui contient l’ensemble du code du Kit de développement logiciel (SDK).
* Entraînez-vous avec les exemples :
  * [Prise en main de Notification Hubs]
  * [Envoi des dernières nouvelles]
  * [Envoi de dernières nouvelles localisées]
  * [Envoi de notifications aux utilisateurs authentifiés]
  * [Envoi de notifications multiplateforme aux utilisateurs authentifiés]

[Kit SDK Java]: https://github.com/Azure/azure-notificationhubs-java-backend
[Get started tutorial]: ./notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Prise en main de Notification Hubs]: notification-hubs-windows-store-dotnet-get-started-wns-push-notification.md
[Envoi des dernières nouvelles]: notification-hubs-windows-notification-dotnet-push-xplat-segmented-wns.md
[Envoi de dernières nouvelles localisées]: notification-hubs-windows-store-dotnet-xplat-localized-wns-push-notification.md
[Envoi de notifications aux utilisateurs authentifiés]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Envoi de notifications multiplateforme aux utilisateurs authentifiés]: notification-hubs-aspnet-backend-windows-dotnet-wns-notification.md
[Maven]: https://maven.apache.org/

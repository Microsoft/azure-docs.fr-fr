---
title: Fichier include
description: Fichier include
services: azure-communication-services
author: chrwhit
manager: nimag
ms.service: azure-communication-services
ms.subservice: azure-communication-services
ms.date: 08/20/2020
ms.topic: include
ms.custom: include file
ms.author: chrwhit
ms.openlocfilehash: cb8e6934125630590a337ed7bf7f4c81b2b73bb3
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94915166"
---
Commencez avec Azure Communication Services en utilisant la bibliothèque de client Communication Services SMS Java pour envoyer des SMS.

Le fait de suivre ce guide de démarrage rapide entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure.

<!--**TODO: update all these reference links as the resources go live**

[API reference documentation](../../../references/overview.md) | [Library source code](https://github.com/Azure/azure-sdk-for-net-pr/tree/feature/communication/sdk/communication/Azure.Communication.Sms#todo-update-to-public) | [Artifact (Maven)](#todo-nuget) | [Samples](#todo-samples)-->

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- [Java Development Kit (JDK)](/java/azure/jdk/?preserve-view=true&view=azure-java-stable) version 8 ou ultérieure.
- [Apache Maven](https://maven.apache.org/download.cgi).
- Une ressource Communication Services active et la chaîne de connexion. [Créez une ressource Communication Services](../../create-communication-resource.md).
- Un numéro de téléphone permettant de recevoir des SMS. [Obtenez un numéro de téléphone](../get-phone-number.md).

### <a name="prerequisite-check"></a>Vérification du prérequis

- Dans une fenêtre de terminal ou de commande, exécutez `mvn -v` pour vérifier que Maven est installé.
- Pour voir les numéros de téléphone associés à votre ressource Communication Services, connectez-vous au [portail Azure](https://portal.azure.com/), localisez votre ressource Communication Services, puis ouvrez l’onglet **Phone Numbers** (Numéros de téléphone) dans le volet de navigation gauche.

## <a name="setting-up"></a>Configuration

### <a name="create-a-new-java-application"></a>Créer une application Java

Ouvrez votre fenêtre de terminal ou de commande, puis accédez au répertoire dans lequel vous souhaitez créer votre application Java. Exécutez la commande ci-dessous pour générer le projet Java à partir du modèle maven-archetype-quickstart.

```console
mvn archetype:generate -DgroupId=com.communication.quickstart -DartifactId=communication-quickstart -DarchetypeArtifactId=maven-archetype-quickstart -DarchetypeVersion=1.4 -DinteractiveMode=false
```

L’objectif « generate » crée un répertoire portant le même nom que l’artifactId. Sous ce répertoire, le répertoire **src/main/java** contient le code source du projet, le répertoire **src/test/java** contient la source de test, et le fichier **pom.xml** est le modèle d’objet du projet, ou modèle POM.

### <a name="install-the-package"></a>Installer le package

Ouvrez le fichier **pom.xml** dans votre éditeur de texte. Ajoutez l’élément dépendance suivant au groupe de dépendances.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-communication-sms</artifactId>
    <version>1.0.0-beta.3</version>
</dependency>
```

### <a name="set-up-the-app-framework"></a>Configurer le framework d’application

Ajoutez la dépendance `azure-core-http-netty` à votre fichier **pom.xml**.

```xml
<dependency>
    <groupId>com.azure</groupId>
    <artifactId>azure-core-http-netty</artifactId>
    <version>1.3.0</version>
</dependency>
```

Ouvrez **/src/main/java/com/communication/quickstart/App.java** dans un éditeur de texte, ajoutez des directives d’importation, puis supprimez l’instruction `System.out.println("Hello world!");` :

```java
package com.communication.quickstart;

import java.io.IOException;
import java.security.InvalidKeyException;
import java.security.NoSuchAlgorithmException;
import java.util.ArrayList;
import java.util.List;

import com.azure.communication.common.PhoneNumber;
import com.azure.communication.sms.SmsClient;
import com.azure.communication.sms.SmsClientBuilder;
import com.azure.communication.sms.models.SendSmsOptions;
import com.azure.communication.sms.models.SendSmsResponse;
import com.azure.core.http.HttpClient;
import com.azure.core.http.netty.NettyAsyncHttpClientBuilder;

public class App
{
    public static void main( String[] args ) throws IOException, NoSuchAlgorithmException, InvalidKeyException
    {
        // Quickstart code goes here
    }
}

```

## <a name="object-model"></a>Modèle objet

Les classes et les interfaces suivantes gèrent certaines des principales fonctionnalités de la bibliothèque de client Azure Communication Services SMS pour Java.

| Nom                                                             | Description                                                                                     |
| ---------------------------------------------------------------- | ----------------------------------------------------------------------------------------------- |
| SmsClientBuilder              | Cette classe crée le SmsClient. Vous lui fournissez un point de terminaison, des informations d’identification et un client HTTP. |
| SmsClient                    | Cette classe est nécessaire pour toutes les fonctionnalités SMS. Vous l’utilisez pour envoyer des SMS.                |
| SendSmsResponse               | Cette classe contient la réponse du service SMS.                                          |
| PhoneNumber                   | Cette classe contient des informations sur le numéro de téléphone

## <a name="authenticate-the-client"></a>Authentifier le client

Instanciez un `SmsClient` avec votre chaîne de connexion. Le code ci-dessous récupère les chaînes de point de terminaison et d’informations d’identification pour la ressource à partir des variables d’environnement nommées `COMMUNICATION_SERVICES_ENDPOINT_STRING` et `COMMUNICATION_SERVICES_CREDENTIAL_STRING` (les informations d’identification sont la valeur `Key` fournie par le portail Azure). Découvrez comment [gérer la chaîne de connexion de votre ressource](../../create-communication-resource.md#store-your-connection-string).

Ajoutez le code suivant à la méthode `main` :

```java
// Your can find your endpoint and access key from your resource in the Azure Portal
String endpoint = "https://<RESOURCE_NAME>.communication.azure.com";
String accessKey = "SECRET";

// Create an HttpClient builder of your choice and customize it
HttpClient httpClient = new NettyAsyncHttpClientBuilder().build();

// Configure and build a new SmsClient
SmsClient client = new SmsClientBuilder()
    .endpoint(endpoint)
    .accessKey(accessKey)
    .httpClient(httpClient)
    .buildClient();
```

Vous pouvez initialiser le client avec n’importe quel client HTTP personnalisé qui implémente l’interface `com.azure.core.http.HttpClient`. Le code ci-dessus illustre l’utilisation du [client HTTP Netty Azure Core](/java/api/overview/azure/core-http-netty-readme?preserve-view=true&view=azure-java-stable) qui est fourni par `azure-core`.

Vous pouvez également fournir la chaîne de connexion entière à l’aide de la fonction connectionString() au lieu de fournir le point de terminaison et la clé d’accès. 
```java
// Your can find your connection string from your resource in the Azure Portal
String connectionString = "<connection_string>";
SmsClient client = new SmsClientBuilder()
    .connectionString(connectionString)
    .httpClient(httpClient)
    .buildClient();
```

## <a name="send-an-sms-message"></a>Envoyer un message SMS

Envoyez un SMS en appelant la méthode sendMessage. Ajoutez ce code à la fin de la méthode `main` :

```java
List<PhoneNumber> to = new ArrayList<PhoneNumber>();
to.add(new PhoneNumber("<to-phone-number>"));

// SendSmsOptions is an optional field. It can be used
// to enable a delivery report to the Azure Event Grid
SendSmsOptions options = new SendSmsOptions();
options.setEnableDeliveryReport(true);

// Send the message and check the response for a message id
SendSmsResponse response = client.sendMessage(
    new PhoneNumber("<leased-phone-number>"),
    to,
    "<message-text>",
    options);

System.out.println("MessageId: " + response.getMessageId());
```

Vous devez remplacer `<leased-phone-number>` par un numéro de téléphone permettant de recevoir des SMS et associé à votre ressource Communication Services, et `<to-phone-number>` par le numéro de téléphone auquel vous souhaitez envoyer un message.

Le paramètre `enableDeliveryReport` est un paramètre facultatif que vous pouvez utiliser pour configurer la création de rapports de remise. C’est utile pour les scénarios où vous souhaitez émettre des événements quand des SMS sont remis. Consultez le guide de démarrage rapide [Gérer les événements SMS](../handle-sms-events.md) pour configurer la création de rapports de remise pour vos SMS.

<!--todo: the signature of the `sendMessage` method changes when configuring delivery reporting. Need to confirm that this is how our client library is to be used.-->

## <a name="run-the-code"></a>Exécuter le code

Accédez au répertoire contenant le fichier **pom.xml**, puis compilez le projet à l’aide de la commande `mvn`.

```console

mvn compile

```

Ensuite, générez le package.

```console

mvn package

```

Exécutez la commande `mvn` suivante pour exécuter l’application.

```console

mvn exec:java -Dexec.mainClass="com.communication.quickstart.App" -Dexec.cleanupDaemonThreads=false

```
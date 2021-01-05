---
title: Démarrage rapide - API REST Service Azure SignalR
description: Découvrez comment utiliser l’API REST avec les exemples Azure SignalR Service suivants. Trouvez les informations de spécification de l’API REST.
author: sffamily
ms.service: signalr
ms.topic: quickstart
ms.date: 11/13/2019
ms.author: zhshang
ms.openlocfilehash: 2613f91e8c7f1ad3a05792a9a165f4560c09a637
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874523"
---
# <a name="quickstart-broadcast-real-time-messages-from-console-app"></a>Démarrage rapide : Diffuser des messages en temps réel à partir de l’application de console

Le service Azure SignalR fournit une [API REST](https://github.com/Azure/azure-signalr/blob/dev/docs/rest-api.md) permettant de prendre en charge les scénarios de communication du serveur vers le client, par exemple la diffusion. Vous pouvez choisir n’importe quel langage de programmation qui peut appeler l’API REST. Vous pouvez publier des messages pour tous les clients connectés, un client spécifique par nom ou un groupe de clients.

Dans ce démarrage rapide, vous allez apprendre à envoyer des messages à partir d’une application de ligne de commande vers les applications clientes connectées en C#.

## <a name="prerequisites"></a>Prérequis

Ce démarrage rapide peut être exécuté sur macOS, Windows ou Linux.

* [Kit de développement logiciel (SDK) .NET Core](https://www.microsoft.com/net/download/core)
* Un éditeur de texte ou un éditeur de code de votre choix.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur <https://portal.azure.com/> avec votre compte Azure.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Pendant le déploiement du service, passons à la préparation du code. Clonez l’[exemple d’application à partir de GitHub](https://github.com/aspnet/AzureSignalR-samples.git), définissez la chaîne de connexion du service SignalR, et exécutez l’application en local.

1. Ouvrez une fenêtre de terminal git. Passez à un dossier dans lequel vous souhaitez cloner l’exemple de projet.

1. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```
Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="build-and-run-the-sample"></a>Créer et exécuter l’exemple.

Cet exemple est une application de console montrant l’utilisation du service Azure SignalR. Il propose deux modes :

- Mode serveur : utilisez des commandes simples pour appeler l’API REST Service Azure SignalR.
- Mode client : connectez-vous au service Azure SignalR et recevez des messages à partir du serveur.

Vous pouvez également découvrir comment générer un jeton d’accès pour vous authentifier avec le service Azure SignalR.

### <a name="build-the-executable-file"></a>Générer le fichier exécutable

Nous utilisons macOS osx.10.13-x64 comme exemple. Vous trouverez des [références](/dotnet/core/rid-catalog) sur la création sur d’autres plateformes.

```bash
cd AzureSignalR-samples/samples/Serverless/

dotnet publish -c Release -r osx.10.13-x64
```

### <a name="start-a-client"></a>Démarrer un client

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="start-a-server"></a>Démarrer un serveur

```bash
cd bin/Release/netcoreapp2.1/osx.10.13-x64/

Serverless server -c "<ConnectionString>" -h <HubName>
```

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="run-the-sample-without-publishing"></a>Exécuter l’exemple sans le publier

Vous pouvez également exécuter la commande ci-dessous pour démarrer un serveur ou un client

```bash
# Start a server
dotnet run -- server -c "<ConnectionString>" -h <HubName>

# Start a client
dotnet run -- client <ClientName> -c "<ConnectionString>" -h <HubName>
```

### <a name="use-user-secrets-to-specify-connection-string"></a>Utiliser les clés secrètes de l’utilisateur pour spécifier la chaîne de connexion

Vous pouvez exécuter `dotnet user-secrets set Azure:SignalR:ConnectionString "<ConnectionString>"` dans le répertoire racine de l’exemple. Après cela, vous n’avez plus besoin de l’option `-c "<ConnectionString>"`.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="usage"></a>Usage

Une fois le serveur démarré, utilisez la commande pour envoyer un message :

```
send user <User Id>
send users <User List>
send group <Group Name>
send groups <Group List>
broadcast
```

Vous pouvez démarrer plusieurs clients avec des noms différents.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="integration-with-third-party-services"></a><a name="usage"> </a> Intégration avec des services tiers

Le service Azure SignalR permet l’intégration de services tiers au système.

### <a name="definition-of-technical-specifications"></a>Définition des spécifications techniques

Le tableau suivant montre toutes les versions des API REST actuellement prises en charge. Vous pouvez également trouver le fichier de définition spécifique à chaque version

Version | État de l’API | Porte | Spécifique
--- | --- | --- | ---
`1.0-preview` | Disponible | 5002 | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1-preview.json)
`1.0` | Disponible | standard | [Swagger](https://github.com/Azure/azure-signalr/tree/dev/docs/swagger/v1.json)

La liste des API disponibles spécifiques à chaque version se trouvent dans la liste suivante.

API | 1.0-preview | 1.0
--- | --- | ---
[Diffuser à tout le monde](#broadcast) | **&#x2713;** | **&#x2713;**
[Diffuser à un groupe](#broadcast-group) | **&#x2713;** | **&#x2713;**
Diffuser à certains groupes | **&#x2713;** (déprécié) | `N / A`
[Envoyer à un utilisateur](#send-user) | **&#x2713;** | **&#x2713;**
Envoyer à certains utilisateurs | **&#x2713;** (déprécié) | `N / A`
[Ajout d’un utilisateur à un groupe](#add-user-to-group) | `N / A` | **&#x2713;**
[Suppression d’un utilisateur d’un groupe](#remove-user-from-group) | `N / A` | **&#x2713;**
[Vérifier l’existence d’un utilisateur](#check-user-existence) | `N / A` | **&#x2713;**
[Supprimer un utilisateur de tous les groupes](#remove-user-from-all-groups) | `N / A` | **&#x2713;**
[Envoyer à une connexion](#send-connection) | `N / A` | **&#x2713;**
[Ajouter une connexion à un groupe](#add-connection-to-group) | `N / A` | **&#x2713;**
[Supprimer une connexion d’un groupe](#remove-connection-from-group) | `N / A` | **&#x2713;**
[Fermer une connexion cliente](#close-connection) | `N / A` | **&#x2713;**
[Service Health](#service-health) | `N / A` | **&#x2713;**

<a name="broadcast"> </a>
### <a name="broadcast-to-everyone"></a>Diffuser à tout le monde

Version | Méthode HTTP des API | URL de la demande | Corps de la demande
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>` | Identique à ce qui précède

<a name="broadcast-group"> </a>
### <a name="broadcast-to-a-group"></a>Diffuser à un groupe

Version | Méthode HTTP des API | URL de la demande | Corps de la demande
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/group/<group-name>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>` | Identique à ce qui précède

<a name="send-user"> </a>
### <a name="sending-to-a-user"></a>Envoi à un utilisateur

Version | Méthode HTTP des API | URL de la demande | Corps de la demande
--- | --- | --- | ---
`1.0-preview` | `POST` | `https://<instance-name>.service.signalr.net:5002/api/v1-preview/hub/<hub-name>/user/<user-id>` | `{"target": "<method-name>", "arguments": [...]}`
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>` | Identique à ce qui précède

<a name="add-user-to-group"> </a>
### <a name="adding-a-user-to-a-group"></a>Ajout d’un utilisateur à un groupe

Version | Méthode HTTP des API | URL de la demande
--- | --- | ---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="remove-user-from-group"> </a>
### <a name="removing-a-user-from-a-group"></a>Supprimer un utilisateur d’un groupe

Version | Méthode HTTP des API | URL de la demande
--- | --- | ---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>`

<a name="check-user-existence"> </a>
### <a name="check-user-existence-in-a-group"></a>Vérifier l’existence d’un utilisateur dans un groupe

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups/<group-name>`
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/users/<user-id>` 

Code du statut de réponse | Description
---|---
`200` | L’utilisateur existe
`404` | L’utilisateur n’existe pas

<a name="remove-user-from-all-groups"> </a>
### <a name="remove-a-user-from-all-groups"></a>Supprimer un utilisateur de tous les groupes

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/users/<user-id>/groups`

<a name="send-connection"> </a>
### <a name="send-message-to-a-connection"></a>Envoyer un message à une connexion

Version de l'API | Méthode HTTP des API | URL de la demande | Corps de la requête
---|---|---|---
`1.0` | `POST` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>` | `{ "target":"<method-name>", "arguments":[ ... ] }`

<a name="add-connection-to-group"> </a>
### <a name="add-a-connection-to-a-group"></a>Ajouter une connexion à un groupe

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `PUT` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="remove-connection-from-group"> </a>
### <a name="remove-a-connection-from-a-group"></a>Supprimer une connexion d’un groupe

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/groups/<group-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>/groups/<group-name>`

<a name="close-connection"> </a>
### <a name="close-a-client-connection"></a>Fermer une connexion cliente

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>`
`1.0` | `DELETE` | `https://<instance-name>.service.signalr.net/api/v1/hubs/<hub-name>/connections/<connection-id>?reason=<close-reason>`

<a name="service-health"> </a>
### <a name="service-health"></a>Service Health

Version de l'API | Méthode HTTP des API | URL de la demande
---|---|---                             
`1.0` | `GET` | `https://<instance-name>.service.signalr.net/api/v1/health`

Code du statut de réponse | Description
---|---
`200` | Service bon
`5xx` | Erreur de service

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsapi).

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à utiliser les API REST pour diffuser des messages en temps réel de SignalR Service aux clients. Découvrez ensuite comment développer et déployer des fonctions Azure avec la liaison SignalR Service, qui repose sur l’API REST.

> [!div class="nextstepaction"]
> [Développer des fonctions Azure Functions à l’aide de liaisons Azure SignalR Service](signalr-quickstart-azure-functions-csharp.md)

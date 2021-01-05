---
title: Développer avec ASP.NET - Azure SignalR Service
description: Démarrage rapide pour l’utilisation d’Azure SignalR Service afin de créer une salle de conversation avec ASP.NET Framework.
author: sffamily
ms.service: signalr
ms.devlang: dotnet
ms.topic: quickstart
ms.custom: devx-track-csharp
ms.date: 09/28/2020
ms.author: zhshang
ms.openlocfilehash: c39ef505b0cea0ad0c03b81683db8441077cd0d2
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874540"
---
# <a name="quickstart-create-a-chat-room-with-aspnet-and-signalr-service"></a>Démarrage rapide : Créer une salle de conversation avec ASP.NET et SignalR Service

Azure SignalR Service est basé sur [SignalR pour ASP.NET Core 2.1](/aspnet/core/signalr/introduction?preserve-view=true&view=aspnetcore-2.1). Cela signifie qu’il n’est **pas** 100 % compatible avec ASP.NET SignalR. Le service Azure SignalR a réimplémenté le protocole de données ASP.NET SignalR basé sur les dernières technologies ASP.NET Core. Lors de l’utilisation du service Azure SignalR pour ASP.NET SignalR, certaines fonctionnalités d’ASP.NET SignalR ne sont plus prises en charge. Par exemple, Azure SignalR ne relit pas les messages lorsque le client se reconnecte. De plus, le transport Forever Frame et JSONP ne sont pas pris en charge. Certaines modifications du code et la version appropriée des bibliothèques dépendantes sont nécessaires pour que l’application ASP.NET SignalR fonctionne avec Azure SignalR Service.

Pour obtenir la liste complète de comparaisons entre les fonctionnalités d’ASP.NET SignalR et d’ASP.NET Core SignalR, voir la [documentation sur les différences entre les versions](/aspnet/core/signalr/version-differences?preserve-view=true&view=aspnetcore-3.1).

Ce guide de démarrage rapide explique comment démarrer avec ASP.NET et Azure SignalR Service pour une [application de salle de conversation](./signalr-quickstart-dotnet-core.md) similaire.


[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note-dotnet.md)]

## <a name="prerequisites"></a>Prérequis

* [Visual Studio 2019](https://visualstudio.microsoft.com/downloads/)
* [.NET 4.6.1](https://www.microsoft.com/net/download/windows)
* [ASP.NET SignalR 2.4.1](https://www.nuget.org/packages/Microsoft.AspNet.SignalR/)

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com/) avec votre compte Azure.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

[!INCLUDE [Create instance](includes/signalr-quickstart-create-instance.md)]

Le mode *serverless* n’est pas pris en charge pour les applications ASP.NET SignalR. Utilisez toujours *Par défaut* ou *Classique* pour l’instance Azure SignalR Service.

Vous pouvez également créer des ressources Azure utilisées dans ce démarrage rapide en vous reportant à la section [Créer un script SignalR Service](scripts/signalr-cli-create-service.md).

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

## <a name="clone-the-sample-application"></a>Clonage de l’exemple d’application

Pendant le déploiement du service, passons à l’utilisation du code. Clonez l’[exemple d’application à partir de GitHub](https://github.com/aspnet/AzureSignalR-samples/tree/master/aspnet-samples/ChatRoom), définissez la chaîne de connexion du service SignalR, et exécutez l’application en local.

1. Ouvrez une fenêtre de terminal git. Passez à un dossier dans lequel vous souhaitez cloner l’exemple de projet.

1. Exécutez la commande suivante pour cloner l’exemple de référentiel : Cette commande crée une copie de l’exemple d’application sur votre ordinateur.

    ```bash
    git clone https://github.com/aspnet/AzureSignalR-samples.git
    ```

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

## <a name="configure-and-run-chat-room-web-app"></a>Configurer et exécuter une application web de salle de conversation

1. Démarrez Visual Studio et ouvrez la solution dans le dossier *aspnet-samples/ChatRoom/* du référentiel cloné.

1. Dans le navigateur où le portail Azure est ouvert, recherchez et sélectionnez l’instance que vous avez créée.

1. Sélectionnez **Clés** pour afficher les chaînes de connexion de l’instance du service SignalR.

1. Sélectionnez et copiez la chaîne de connexion principale.

1. Maintenant, définissez la chaîne de connexion dans le fichier *web.config*.

    ```xml
    <configuration>
    <connectionStrings>
        <add name="Azure:SignalR:ConnectionString" connectionString="<Replace By Your Connection String>"/>
    </connectionStrings>
    ...
    </configuration>
    ```

1. Dans *Startup.cs*, au lieu d’appeler `MapSignalR()`, vous devez appeler `MapAzureSignalR({YourApplicationName})` et transmettre une chaîne de connexion pour faire en sorte que l’application se connecte au service au lieu d’héberger SignalR elle-même. Remplacez `{YourApplicationName}` par le nom de votre application. Ce nom est unique afin de distinguer cette application d’autres applications. Vous pouvez utiliser la valeur `this.GetType().FullName`.

    ```cs
    public void Configuration(IAppBuilder app)
    {
        // Any connection or hub wire up and configuration should go here
        app.MapAzureSignalR(this.GetType().FullName);
    }
    ```

    Vous devez également référencer le Kit de développement logiciel (SDK) du service avant d’utiliser ces API. Ouvrez **Outils | Gestionnaire de package NuGet | Console du Gestionnaire de package**, puis exécutez la commande suivante :

    ```powershell
    Install-Package Microsoft.Azure.SignalR.AspNet
    ```

    Hormis ces modifications, rien ne change. Vous pouvez toujours utiliser l’interface concentrateur que vous connaissez déjà pour écrire la logique métier.

    > [!NOTE]
    > Dans l’implémentation, un point de terminaison `/signalr/negotiate` est exposé pour la négociation par le Kit de développement logiciel (SDK) Azure SignalR Service. Il renvoie une réponse de négociation spéciale lorsque des clients tentent de se connecter et les redirige vers le point de terminaison du service défini dans la chaîne de connexion.

1. Appuyez sur <kbd>F5</kbd> pour exécuter le projet en mode débogage. Vous pouvez voir que l’application s’exécute localement. Au lieu d’héberger elle-même un runtime SignalR, l’application se connecte désormais à Azure SignalR Service.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

[!INCLUDE [Cleanup](includes/signalr-quickstart-cleanup.md)]

> [!IMPORTANT]
> La suppression d’un groupe de ressources est définitive ; le groupe de ressources et l’ensemble des ressources qu’il contient sont supprimés de manière permanente. Veillez à ne pas supprimer accidentellement des ressources ou un groupe de ressources incorrects. Si vous avez créé les ressources pour l’hébergement de cet exemple dans un groupe de ressources existant contenant des ressources que vous souhaitez conserver, vous pouvez supprimer chaque ressource individuellement à partir de son panneau respectif, au lieu de supprimer l’intégralité du groupe de ressources.

Connectez-vous au [Portail Azure](https://portal.azure.com) et cliquez sur **Groupes de ressources**.

Dans la zone de texte **Filtrer par nom.** , saisissez le nom de votre groupe de ressources. Les instructions relatives à ce démarrage rapide utilisaient un groupe de ressources nommé *SignalRTestResources*. Sur votre groupe de ressources dans la liste des résultats, cliquez sur **...** , puis sur **Supprimer le groupe de ressources**.

![DELETE](./media/signalr-quickstart-dotnet-core/signalr-delete-resource-group.png)

Après quelques instants, le groupe de ressources et toutes les ressources qu’il contient sont supprimés.

Vous rencontrez des problèmes ? Essayez le [guide de résolution des problèmes](signalr-howto-troubleshoot-guide.md) ou [faites-le nous savoir](https://aka.ms/asrs/qsnet).

## <a name="next-steps"></a>Étapes suivantes

En suivant ce guide de démarrage rapide, vous avez créé une ressources Azure SignalR Service et l’avez utilisée avec une application web ASP.NET. Vous devez à présent apprendre à développer des applications en temps réel à l’aide d’Azure SignalR Service avec ASP.NET Core.

> [!div class="nextstepaction"]
> [Azure SignalR Service avec ASP.NET Core](./signalr-quickstart-dotnet-core.md)

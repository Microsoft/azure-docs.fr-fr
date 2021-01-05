---
title: 'Tutoriel : Générer une application de conversation Blazor Server - Azure SignalR'
description: Dans ce tutoriel, vous allez apprendre à générer et à modifier une application Blazor Server avec Azure SignalR Service
author: JialinXin
ms.service: signalr
ms.topic: tutorial
ms.date: 09/09/2020
ms.author: jixin
ms.openlocfilehash: b0059e986b9a6ba8152a1a61f8d696f1caa4646a
ms.sourcegitcommit: 90caa05809d85382c5a50a6804b9a4d8b39ee31e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97755903"
---
# <a name="tutorial-build-a-blazor-server-chat-app"></a>Tutoriel : Créer une application de conversation sur un serveur Blazor Server

Ce tutoriel vous montre comment créer et modifier une application Blazor Server. Vous découvrirez comment effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer une salle de conversation simple avec une application Blazor Server.
> * Modifier les composants Razor.
> * Utiliser la gestion des événements et la liaison de données dans des composants.
> * Effectuer un déploiement rapide sur Azure App Service avec Visual Studio.
> * Migrer le service SignalR local vers Azure SignalR Service.

## <a name="prerequisites"></a>Prérequis
* Installer le [SDK .NET Core 3.0](https://dotnet.microsoft.com/download/dotnet-core/3.0) (Version ultérieure à la version 3.0.100)
* Installer [Visual Studio 2019](https://visualstudio.microsoft.com/vs/) (Version ultérieure à la version 16.3)
> La version préliminaire de Visual Studio 2019, qui est fournie avec le dernier modèle d’application Blazor Server ciblant une version de .Net Core plus récente fonctionne également.

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/asrs/qsblazor)

## <a name="build-a-local-chat-room-in-blazor-server-app"></a>Générer une salle de conversation locale dans une application Blazor Server

À compter de Visual Studio 2019 version 16.2.0, Azure SignalR Service est un processus de publication d’application web intégré, et la gestion de dépendances entre l’application web et le service SignalR serait beaucoup plus pratique. Vous pouvez tester simultanément l’utilisation du service SignalR local dans un environnement de développement local et l’utilisation d’Azure SignalR Service pour Azure App Service sans aucun changement de code.

1. Créer une application de conversation Blazor
   
   Dans Visual Studio, choisissez Créer un projet -> Application Blazor -> (nommez l’application, puis choisissez un dossier) -> Application Blazor Server. Vérifiez que vous avez déjà installé le SDK .NET Core 3.0+ pour permettre à Visual Studio de reconnaître correctement le framework cible.

   [ ![Dans Créer un projet, les modèles Application Blazor sont sélectionnés.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-create.png#lightbox)
   
   Ou exécutez cmd
   ```dotnetcli
   dotnet new blazorserver -o BlazorChat
   ```
   
1. Ajoutez un fichier `BlazorChatSampleHub.cs` afin d’implémenter `Hub` pour la conversation.
   
   ```cs
   using System;
   using System.Threading.Tasks;
   using Microsoft.AspNetCore.SignalR;
   
   namespace BlazorChat
   {
       public class BlazorChatSampleHub : Hub
       {
           public const string HubUrl = "/chat";
   
           public async Task Broadcast(string username, string message)
           {
               await Clients.All.SendAsync("Broadcast", username, message);
           }
   
           public override Task OnConnectedAsync()
           {
               Console.WriteLine($"{Context.ConnectionId} connected");
               return base.OnConnectedAsync();
           }
   
           public override async Task OnDisconnectedAsync(Exception e)
           {
               Console.WriteLine($"Disconnected {e?.Message} {Context.ConnectionId}");
               await base.OnDisconnectedAsync(e);
           }
       }
   }
   ```
   
1. Ajoutez un point de terminaison pour le hub dans `Startup.Configure()`.
   
   ```cs
   app.UseEndpoints(endpoints =>
   {
       endpoints.MapBlazorHub();
       endpoints.MapFallbackToPage("/_Host");
       endpoints.MapHub<BlazorChatSampleHub>(BlazorChatSampleHub.HubUrl);
   });
   ```
   
1. Installez le package `Microsoft.AspNetCore.SignalR.Client` pour utiliser un client SignalR.

   ```dotnetcli
   dotnet add package Microsoft.AspNetCore.SignalR.Client --version 3.1.7
   ```

1. Créez `ChatRoom.razor` sous le dossier `Pages` pour implémenter un client SignalR. Suivez les étapes ci-dessous ou copiez simplement [ChatRoom.razor](https://github.com/aspnet/AzureSignalR-samples/tree/master/samples/BlazorChat/Pages/ChatRoom.razor).

   1. Ajoutez un lien et une référence de page.
      
      ```razor
      @page "/chatroom"
      @inject NavigationManager navigationManager
      @using Microsoft.AspNetCore.SignalR.Client;
      ```

   1. Ajoutez du code au nouveau client SignalR pour envoyer et recevoir des messages.
      
      ```razor
      @code {
          // flag to indicate chat status
          private bool _isChatting = false;
          
          // name of the user who will be chatting
          private string _username;
      
          // on-screen message
          private string _message;
          
          // new message input
          private string _newMessage;
          
          // list of messages in chat
          private List<Message> _messages = new List<Message>();
          
          private string _hubUrl;
          private HubConnection _hubConnection;
      
          public async Task Chat()
          {
              // check username is valid
              if (string.IsNullOrWhiteSpace(_username))
              {
                  _message = "Please enter a name";
                  return;
              };
      
              try
              {
                  // Start chatting and force refresh UI.
                  _isChatting = true;
                  await Task.Delay(1);

                  // remove old messages if any
                  _messages.Clear();
         
                  // Create the chat client
                  string baseUrl = navigationManager.BaseUri;
      
                  _hubUrl = baseUrl.TrimEnd('/') + BlazorChatSampleHub.HubUrl;
      
                  _hubConnection = new HubConnectionBuilder()
                      .WithUrl(_hubUrl)
                      .Build();
      
                  _hubConnection.On<string, string>("Broadcast", BroadcastMessage);
      
                  await _hubConnection.StartAsync();
      
                  await SendAsync($"[Notice] {_username} joined chat room.");
              }
              catch (Exception e)
              {
                  _message = $"ERROR: Failed to start chat client: {e.Message}";
                  _isChatting = false;
              }
          }
      
          private void BroadcastMessage(string name, string message)
          {
              bool isMine = name.Equals(_username, StringComparison.OrdinalIgnoreCase);
      
              _messages.Add(new Message(name, message, isMine));
      
              // Inform blazor the UI needs updating
              StateHasChanged();
          }
      
          private async Task DisconnectAsync()
          {
              if (_isChatting)
              {
                  await SendAsync($"[Notice] {_username} left chat room.");
      
                  await _hubConnection.StopAsync();
                  await _hubConnection.DisposeAsync();
      
                  _hubConnection = null;
                  _isChatting = false;
              }
          }
      
          private async Task SendAsync(string message)
          {
              if (_isChatting && !string.IsNullOrWhiteSpace(message))
              {
                  await _hubConnection.SendAsync("Broadcast", _username, message);
      
                  _newMessage = string.Empty;
              }
          }
      
          private class Message
          {
              public Message(string username, string body, bool mine)
              {
                  Username = username;
                  Body = body;
                  Mine = mine;
              }
      
              public string Username { get; set; }
              public string Body { get; set; }
              public bool Mine { get; set; }
      
              public bool IsNotice => Body.StartsWith("[Notice]");
      
              public string CSS => Mine ? "sent" : "received";
          }
      }
      ```

   1. Ajoutez un composant de rendu avant `@code` pour que l’interface utilisateur interagisse avec le client SignalR.
      
      ```razor
      <h1>Blazor SignalR Chat Sample</h1>
      <hr />
      
      @if (!_isChatting)
      {
          <p>
              Enter your name to start chatting:
          </p>
      
          <input type="text" maxlength="32" @bind="@_username" />
          <button type="button" @onclick="@Chat"><span class="oi oi-chat" aria-hidden="true"></span> Chat!</button>
      
          // Error messages
          @if (_message != null)
          {
              <div class="invalid-feedback">@_message</div>
              <small id="emailHelp" class="form-text text-muted">@_message</small>
          }
      }
      else
      {
          // banner to show current user
          <div class="alert alert-secondary mt-4" role="alert">
              <span class="oi oi-person mr-2" aria-hidden="true"></span>
              <span>You are connected as <b>@_username</b></span>
              <button class="btn btn-sm btn-warning ml-md-auto" @onclick="@DisconnectAsync">Disconnect</button>
          </div>
          // display messages
          <div id="scrollbox">
              @foreach (var item in _messages)
              {
                  @if (item.IsNotice)
                  {
                      <div class="alert alert-info">@item.Body</div>
                  }
                  else
                  {
                      <div class="@item.CSS">
                          <div class="user">@item.Username</div>
                          <div class="msg">@item.Body</div>
                      </div>
                  }
              }
              <hr />
              <textarea class="input-lg" placeholder="enter your comment" @bind="@_newMessage"></textarea>
              <button class="btn btn-default" @onclick="@(() => SendAsync(_newMessage))">Send</button>
          </div>
      }
      ```

1. Mettez à jour `NavMenu.razor` pour insérer un menu d’entrées pour la salle de conversation sous `NavMenuCssClass` comme Pause.

   ```razor
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="chatroom">
           <span class="oi oi-chat" aria-hidden="true"></span> Chat room
       </NavLink>
   </li>
   ```
   
1. Mettez à jour `site.css` pour optimiser les affichages de bulles dans la zone de conversation. Ajoutez le code ci-dessous à la fin.

   ```css
   /* improved for chat text box */
   textarea {
       border: 1px dashed #888;
       border-radius: 5px;
       width: 80%;
       overflow: auto;
       background: #f7f7f7
   }
   
   /* improved for speech bubbles */
   .received, .sent {
       position: relative;
       font-family: arial;
       font-size: 1.1em;
       border-radius: 10px;
       padding: 20px;
       margin-bottom: 20px;
   }
   
   .received:after, .sent:after {
       content: '';
       border: 20px solid transparent;
       position: absolute;
       margin-top: -30px;
   }
   
   .sent {
       background: #03a9f4;
       color: #fff;
       margin-left: 10%;
       top: 50%;
       text-align: right;
   }
   
   .received {
       background: #4CAF50;
       color: #fff;
       margin-left: 10px;
       margin-right: 10%;
   }
   
   .sent:after {
       border-left-color: #03a9f4;
       border-right: 0;
       right: -20px;
   }
   
   .received:after {
       border-right-color: #4CAF50;
       border-left: 0;
       left: -20px;
   }
   
   /* div within bubble for name */
   .user {
       font-size: 0.8em;
       font-weight: bold;
       color: #000;
   }
   
   .msg {
       /*display: inline;*/
   }
   ```

1. Cliquez sur <kbd>F5</kbd> pour exécuter l’application. Vous pouvez discuter comme indiqué ci-dessous.

   [ ![Une conversation animée entre Bob et Alice s’affiche. Alice dit Bonjour, Bob dit Salut.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat.gif#lightbox)
   
[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/asrs/qsblazor)

## <a name="publish-to-azure"></a>Publication dans Azure

   Pour l’instant, l’application Blazor fonctionne sur un service SignalR local et, lors du déploiement sur Azure App Service, il est suggéré d’utiliser [Azure SignalR Service](/aspnet/core/signalr/scale?view=aspnetcore-3.1#azure-signalr-service) qui permet d’effectuer un scale-up d’une application Blazor Server vers un grand nombre de connexions SignalR simultanées. De plus, la portée générale et les centres de données hautes performances du service SignalR contribuent de manière significative à réduire la latence en raison de la zone géographique.

> [!IMPORTANT]
> Dans l’application Blazor Server, les états de l’interface utilisateur sont conservés côté serveur, ce qui signifie qu’un serveur permanent est nécessaire dans ce cas. S’il existe un seul serveur d’applications, le serveur permanent est garanti par conception. Toutefois, s’il y a plusieurs serveurs d’applications, il est possible que la négociation et la connexion du client aillent vers d’autres serveurs et que cela génère des erreurs d’interface utilisateur dans une application Blazor. Vous devez donc activer le serveur rémanent comme ci-dessous dans `appsettings.json` :
> ```json
> "Azure:SignalR:ServerStickyMode": "Required"
> ```

1. Cliquez avec le bouton droit sur le projet, puis accédez à `Publish`.

   * Cible : Azure
   * Cible spécifique : Tous les types de service **Azure App Service** sont pris en charge.
   * App Service : créez un service d’application ou sélectionnez-en un existant.

   [ ![L’animation montre la sélection d’Azure comme cible, puis d’Azure App Service en tant que cible spécifique.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-profile.gif#lightbox)

1. Ajouter une dépendance du service Azure SignalR

   Une fois le profil de publication créé, vous pouvez voir un message recommandé sous **Dépendances de service**. Cliquez sur **Configurer** pour créer un service Azure SignalR ou en sélectionner un existant dans le volet.

   [ ![Dans Publier, le lien vers Configurer est mis en surbrillance.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency.png#lightbox)

   La dépendance de service effectue les opérations mentionnées ci-dessous pour permettre à votre application de basculer automatiquement vers Azure SignalR Service quand vous êtes sur vous êtes sur Azure.

   * Mettez à jour [`HostingStartupAssembly`](/aspnet/core/fundamentals/host/platform-specific-configuration?view=aspnetcore-3.1) pour utiliser Azure SignalR Service.
   * Ajoutez une référence de package NuGet Azure SignalR Service.
   * Mettez à jour les propriétés de profil pour enregistrer les paramètres de dépendance.
   * C’est vous qui choisissez de configurer, ou non, le magasin de secrets.
   * Ajoutez la configuration `appsettings` pour que votre application cible le service Azure SignalR Service sélectionné.

   [ ![Dans Résumé des modifications, les cases à cocher permettent de sélectionner toutes les dépendances.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-dependency-summary.png#lightbox)

1. Publier l’application

   L’application est maintenant prête à être publiée. Et elle va parcourir automatiquement la page une fois la publication terminée. 
   > [!NOTE]
   > Il est possible qu’elle ne fonctionne pas immédiatement lors de la première visite de page en raison du temps de latence au démarrage du déploiement d’Azure App Service. Essayez d’actualiser la page de temps à autre.
   > Par ailleurs, vous pouvez utiliser le mode de débogueur de navigateur avec <kbd>F12</kbd> pour valider que le trafic a déjà été redirigé vers Azure SignalR Service.

   [ ![L’exemple de conversation Blazor SignalR est associé à une zone de texte pour votre nom et à un bouton Chat! (Conversation !) pour démarrer une conversation.](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png) ](media/signalr-tutorial-build-blazor-server-chat-app/blazor-chat-azure.png#lightbox)
   
[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/asrs/qsblazor)

## <a name="further-topic-enable-azure-signalr-service-in-local-development"></a>Rubrique supplémentaire : Activer Azure SignalR Service dans un développement local

1. Ajouter une référence au SDK Azure SignalR

   ```dotnetcli
   dotnet add package Microsoft.Azure.SignalR --version 1.5.1
   ```

1. Ajouter un appel à Azure SignalR Service dans `Startup.ConfigureServices()`.

   ```cs
   public void ConfigureServices(IServiceCollection services)
   {
       ...
       services.AddSignalR().AddAzureSignalR();
       ...
   }
   ```

1. Configurer `ConnectionString` d’Azure SignalR Service dans `appsetting.json` ou avec l’outil [Secret Manager](/aspnet/core/security/app-secrets?tabs=visual-studio&view=aspnetcore-3.1#secret-manager)

> [!NOTE]
> L’étape 2 peut être remplacée en utilisant [`HostingStartupAssembly`](/aspnet/core/fundamentals/host/platform-specific-configuration?view=aspnetcore-3.1) sur le SDK SignalR.
> 
> 1. Ajouter une configuration pour activer Azure SignalR Service dans `appsetting.json`
>    ```js
>    "Azure": {
>      "SignalR": {
>        "Enabled": true,
>        "ConnectionString": <your-connection-string>
>      }
>    }
>    ```
> 
> 1. Affecter un assembly de démarrage d’hébergement pour utiliser le SDK Azure SignalR. Modifiez `launchSettings.json` et ajoutez une configuration comme celle indiquée ci-dessous dans `environmentVariables`.
>    ```js
>    "environmentVariables": {
>        ...,
>        "ASPNETCORE_HOSTINGSTARTUPASSEMBLIES": "Microsoft.Azure.SignalR"
>      }
>    ```

[Vous rencontrez des problèmes ? Faites-le nous savoir.](https://aka.ms/asrs/qsblazor)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer les ressources créées à l’occasion de ce tutoriel, supprimez le groupe de ressources par le biais du portail Azure.

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une salle de conversation simple avec une application Blazor Server.
> * Modifier les composants Razor.
> * Utiliser la gestion des événements et la liaison de données dans des composants.
> * Effectuer un déploiement rapide sur Azure App Service avec Visual Studio.
> * Migrer le service SignalR local vers Azure SignalR Service.

En savoir plus sur la haute disponibilité.
> [!div class="nextstepaction"]
> [Résilience et récupération d’urgence](signalr-concept-disaster-recovery.md)

## <a name="additional-resources"></a>Ressources supplémentaires

* [ASP.NET Core Blazor](/aspnet/core/blazor)

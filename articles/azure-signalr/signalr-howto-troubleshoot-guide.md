---
title: Guide de dépannage pour Azure SignalR Service
description: Découvrez comment résoudre les problèmes courants
author: YanJin
ms.service: signalr
ms.topic: conceptual
ms.date: 11/06/2020
ms.author: yajin1
ms.openlocfilehash: cc17dcef7a554bee2715c79ba7d0c2356db2c6b3
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185655"
---
# <a name="troubleshooting-guide-for-azure-signalr-service-common-issues"></a>Guide de dépannage pour résoudre des problèmes courants rencontrés avec Azure SignalR Service

Ce guide vis à fournir des instructions de dépannage utiles basées sur les problèmes courants rencontrés et résolus par les clients au cours des dernières années.

## <a name="access-token-too-long"></a>Jeton d’accès trop long

### <a name="possible-errors"></a>Erreurs possibles :

* `ERR_CONNECTION_` côté client
* 414 URI trop long
* 413 charge utile maximale dépassée
* La longueur du jeton d’accès ne doit pas dépasser 4 Ko. 413 Entité de demande trop grande

### <a name="root-cause"></a>Cause racine :

Pour HTTP/2, la longueur maximale d’un en-tête est de **4 K**. Par conséquent, si vous utilisez un navigateur pour accéder au service Azure, cette limitation génèrera une erreur `ERR_CONNECTION_`.

Pour les clients HTTP/1.1 ou C#, la longueur maximale de l’URI est de **12 K**, et la longueur maximale de l’en-tête de **16 K**.

Avec le Kit de développement logiciel (SDK) versions **1.0.6** ou supérieures, `/negotiate` lève l’erreur `413 Payload Too Large` quand le jeton d’accès généré est d’une longueur supérieure à **4 K**.

### <a name="solution"></a>Solution :

Par défaut, les revendications de `context.User.Claims` sont incluses lors de la génération du jeton d’accès JWT pour **ASRS**(**A** zure **S** ignal **R** **S** ervice). Ainsi, les revendications sont conservées et peuvent être transmises d’**ASRS** au `Hub` lorsque le client se connecte au `Hub`.

Dans certains cas, des revendications `context.User.Claims` permettent de stocker de grandes quantités d’informations pour le serveur d’applications, dont la plupart ne sont pas utilisées par les `Hub`s mais par d’autres composants.

Le jeton d’accès généré est transmis via le réseau et, pour les connexions WebSocket/SSE, les jetons d’accès sont transmis via des chaînes de requête. C’est pourquoi nous recommandons de transmettre les revendications **nécessaires** du client via **ASRS** à votre serveur d’applications quand le Hub en a besoin.

Il existe un `ClaimsProvider` qui vous permet de personnaliser les revendications transmises à **ASRS** dans le jeton d’accès.

Pour ASP.NET Core :
```cs
services.AddSignalR()
        .AddAzureSignalR(options =>
            {
                // pick up necessary claims
                options.ClaimsProvider = context => context.User.Claims.Where(...);
            });
```

Pour ASP.NET :
```cs
services.MapAzureSignalR(GetType().FullName, options =>
            {
                // pick up necessary claims
                options.ClaimsProvider = context.Authentication?.User.Claims.Where(...);
            });
```

## <a name="tls-12-required"></a>TLS 1.2 requis

### <a name="possible-errors"></a>Erreurs possibles :

* ASP.NET « Aucun serveur disponible » erreur [#279](https://github.com/Azure/azure-signalr/issues/279)
* ASP.NET « la connexion n’est pas active. Impossible d’envoyer des données au service ». Erreur [#324](https://github.com/Azure/azure-signalr/issues/324)
* « Une erreur s’est produite lors de l’envoi de la requête HTTP à https://<API endpoint>. Cette erreur peut être due au fait que le certificat de serveur n’est pas configuré correctement avec HTTP.SYS pour HTTPS. Une autre cause possible de cette erreur est une non-correspondance de la liaison de sécurité entre le client et le serveur. »

### <a name="root-cause"></a>Cause racine :

Le service Azure prend uniquement en charge le protocole TLS 1.2 pour les problèmes de sécurité. Avec le .NET Framework, il est possible que le protocole par défaut ne soit pas TLS 1.2. Par conséquent, les connexions du serveur à ASRS ne peuvent pas aboutir.

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

1. Si cette erreur peut être reproduite localement, désactivez l’option *Uniquement mon code* et levez toutes les exceptions CLR, puis déboguez le serveur d’applications localement pour voir les levées d’exceptions.
    * Désactive l’option *Uniquement mon code*

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/uncheck-just-my-code.png" alt-text="Désactiver l’option Uniquement mon code":::

    * Lever des exceptions CLR

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/throw-clr-exceptions.png" alt-text="Lever des exceptions CLR":::

    * Consultez la levée d’exceptions lors du débogage du code côté serveur d’applications :
        
        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/tls-throws.png" alt-text="Levées d’exception":::

2. Pour les exceptions ASP.NET, vous pouvez également ajouter le code suivant à votre `Startup.cs` pour activer la trace détaillée et voir les erreurs du journal.
```cs
app.MapAzureSignalR(this.GetType().FullName);
// Make sure this switch is called after MapAzureSignalR
GlobalHost.TraceManager.Switch.Level = SourceLevels.Information;
```

### <a name="solution"></a>Solution :

Ajoutez le code suivant à votre démarrage :
```cs
ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls12;
```

## <a name="400-bad-request-returned-for-client-requests"></a>400 Requête incorrecte retournée aux demandes du client

### <a name="root-cause"></a>Cause racine

Vérifiez si la demande de votre client comporte plusieurs chaînes de requête `hub`. `hub` est un paramètre de requête préservé et 400 lève une exception si le service détecte plus d’un `hub` dans la requête.

## <a name="401-unauthorized-returned-for-client-requests"></a>Code 401 - Non autorisé retourné pour les demandes des clients

### <a name="root-cause"></a>Cause racine

Actuellement, la valeur par défaut de la durée de vie du jeton JWT est de 1 heure.

Pour ASP.NET Core SignalR, quand il utilise le type de transport WebSocket, il est OK.

Pour un autre type de transport, un SSE et une interrogation longue d’ASP.NET Core SignalR, cela signifie par défaut que la connexion peut persister au maximum pendant 1 heure.

Pour ASP.NET SignalR, le client envoie une demande KeepAlive `/ping` au service de temps en temps. Quand le `/ping` échoue, le client **abandonne** la connexion et ne se reconnecte jamais. Cela signifie que, pour ASP.NET SignalR, la durée de vie du jeton par défaut fait durer la connexion **au maximum** pendant 1 heure pour tous les types de transport.

### <a name="solution"></a>Solution

Par souci de sécurité, il n’est pas recommandé d’étendre le protocole TTL. Nous vous suggérons d’ajouter une logique de reconnexion à partir du client pour redémarrer la connexion quand une telle erreur 401 se produit. Lorsque le client redémarre la connexion, il négocie avec le serveur d’applications pour obtenir à nouveau le jeton JWT et obtenir un jeton renouvelé.

Pour savoir comment redémarrer des connexions client, voir [ici](#restart_connection).

## <a name="404-returned-for-client-requests"></a>Code 404 retourné pour les demandes des clients

Une connexion persistante SignalR commence par négocier (`/negotiate`) l’accès au Azure SignalR Service, puis établit la connexion réelle à Azure SignalR Service.

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

* Voici [Comment afficher des demandes sortantes](#view_request) pour obtenir la demande du client au service.
* Vérifiez l’URL de la demande quand l’erreur 404 se produit. Si l’URL cible votre application web, et est similaire à `{your_web_app}/hubs/{hubName}`, vérifiez si la valeur de `SkipNegotiation` du client est `true`. Lorsque vous utilisez Azure SignalR, le client reçoit l’URL de redirection quand il négocie pour la première fois avec le serveur d’applications. Le client ne doit **PAS** ignorer la négociation lors de l’utilisation d’Azure SignalR.
* Une autre erreur 404 peut se produire lorsque la demande de connexion est gérée plus de **5** secondes après l’appel de la commande `/negotiate`. Vérifiez l’horodatage de la demande client, puis ouvrez un problème chez nous si la réponse à la demande adressée au service est lente.

## <a name="404-returned-for-aspnet-signalrs-reconnect-request"></a>Erreur 404 retournée pour la demande de reconnexion d’ASP.NET SignalR

Pour ASP.NET SignalR, en cas d’[interruption de la connexion client](#client_connection_drop), celle-ci est rétablie à l’aide du même `connectionId` à trois reprises avant son arrêt. Le commande `/reconnect` peut être utile si la connexion est interrompue en raison de problèmes intermittents du réseau et que la commande `/reconnect` peut rétablir correctement la connexion persistante. Dans d’autres circonstances, par exemple, si la connexion client est interrompue en raison de la perte de la connexion serveur routée, ou parce que le service SignalR rencontre des erreurs internes telles qu’un redémarrage/basculement/déploiement d’instance, la connexion cesse d’exister et la commande `/reconnect` retourne l’erreur `404`. Il s’agit du comportement attendu de la commande `/reconnect` et, après trois tentatives, la connexion s’arrête. Nous vous suggérons de vous doter d’une logique de [redémarrage de connexion](#restart_connection) lorsque la connexion s’arrête.

## <a name="429-too-many-requests-returned-for-client-requests"></a>Erreur 429 (Trop de demandes) retournée aux demandes client

L’erreur 429 est retournée si votre nombre de connexions **simultanées** dépasse la limite.

Pour des instances **Gratuites**, la limite du nombre de connexions **simultanées** est de 20. Pour des instances **Standard**, la limite du nombre de connexions **simultanées** **par unité** est de 1 K, ce qui signifie que Unit100 autorise 100 000 connexions simultanées.

Les connexions incluent les connexions client et serveur. Pour savoir comment les connexions sont comptées, voir [ici](./signalr-concept-messages-and-connections.md#how-connections-are-counted).

## <a name="500-error-when-negotiate-azure-signalr-service-is-not-connected-yet-please-try-again-later"></a>500 Erreur lors de la négociation : Azure SignalR Service n’est pas encore connecté. Réessayez plus tard.

### <a name="root-cause"></a>Cause racine

Cette erreur est signalée quand il n’existe aucune connexion serveur à Azure SignalR Service.

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

Activez la trace côté serveur pour connaître les détails de l’erreur lorsque le serveur tente de se connecter à Azure SignalR Service.

#### <a name="enable-server-side-logging-for-aspnet-core-signalr"></a>Activer la journalisation côté serveur pour ASP.NET Core SignalR

La journalisation côté serveur pour ASP.NET Core SignalR s’intègre avec la [journalisation](/aspnet/core/fundamentals/logging/?tabs=aspnetcore2x&view=aspnetcore-2.1) basée sur `ILogger` fournie dans le framework ASP.NET Core. Vous pouvez activer la journalisation côté serveur à l’aide de `ConfigureLogging`, un exemple d’utilisation comme suit :
```cs
.ConfigureLogging((hostingContext, logging) =>
        {
            logging.AddConsole();
            logging.AddDebug();
        })
```
Les catégories d’enregistreur d’événements pour Azure SignalR commencent toujours par `Microsoft.Azure.SignalR`. Pour activer les journaux détaillés d’Azure SignalR, configurez les préfixes précédents sur le niveau `Debug` dans votre fichier **appsettings.json** comme ci-dessous :
```JSON
{
    "Logging": {
        "LogLevel": {
            ...
            "Microsoft.Azure.SignalR": "Debug",
            ...
        }
    }
}
```

#### <a name="enable-server-side-traces-for-aspnet-signalr"></a>Activer les traces côté serveur pour ASP.NET SignalR

En cas d’utilisation d’une version du Kit de développement logiciel (SDK) >= `1.0.0`, vous pouvez activer les traces en ajoutant le code suivant au fichier `web.config`: ([Détails](https://github.com/Azure/azure-signalr/issues/452#issuecomment-478858102))
```xml
<system.diagnostics>
    <sources>
      <source name="Microsoft.Azure.SignalR" switchName="SignalRSwitch">
        <listeners>
          <add name="ASRS" />
        </listeners>
      </source>
    </sources>
    <!-- Sets the trace verbosity level -->
    <switches>
      <add name="SignalRSwitch" value="Information" />
    </switches>
    <!-- Specifies the trace writer for output -->
    <sharedListeners>
      <add name="ASRS" type="System.Diagnostics.TextWriterTraceListener" initializeData="asrs.log.txt" />
    </sharedListeners>
    <trace autoflush="true" />
  </system.diagnostics>
```

<a name="client_connection_drop"></a>

## <a name="client-connection-drops"></a>Pertes de connexion client

Lorsque le client est connecté à Azure SignalR, la connexion persistante entre le client et Azure SignalR peut parfois s’interrompre pour différentes raisons. Cette section décrit plusieurs causes possibles d’une telle perte de connexion et fournit des conseils sur la façon d’identifier la cause racine.

### <a name="possible-errors-seen-from-the-client-side"></a>Erreurs possibles côté client

* `The remote party closed the WebSocket connection without completing the close handshake`
* `Service timeout. 30.00ms elapsed without receiving a message from service.`
* `{"type":7,"error":"Connection closed with an error."}`
* `{"type":7,"error":"Internal server error."}`

### <a name="root-cause"></a>Cause racine :

Les connexions client peuvent être interrompues dans diverses circonstances :
* Quand le `Hub` lève des exceptions avec la demande entrante.
* Quand la connexion serveur vers laquelle le client est routé s’interrompt, consultez la section ci-dessous pour plus de détails sur les [pertes de connexion serveur](#server_connection_drop).
* Quand un problème de connectivité réseau se produit entre le client et SignalR Service.
* Quand SignalR Service rencontre des erreurs internes, par exemple, de redémarrage, de basculement ou de déploiement d’instance, etc.

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

1. Ouvrez le journal côté serveur d’applications pour voir si quelque chose d’anormal s’est produit.
2. Vérifiez le journal des événements côté serveur d’applications pour voir si le serveur d’applications a redémarré.
3. Ouvrez un problème chez nous en fournissant le laps de temps, et envoyez-nous par e-mail le nom de la ressource.


## <a name="client-connection-increases-constantly"></a>La connexion client augmente constamment

Cela peut être dû à une utilisation incorrecte de la connexion client. Si quelqu’un oublie d’arrêter ou de supprimer le client SignalR, la connexion reste ouverte.

### <a name="possible-errors-seen-from-the-signalrs-metrics-that-is-in-monitoring-section-of-azure-portal-resource-menu"></a>Erreurs possibles observées dans les métriques de SignalR, dans la section Surveillance du menu de ressources du portail Azure

Les connexions client augmentent constamment pendant une longue période dans les métriques d’Azure SignalR.

:::image type="content" source="./media/signalr-howto-troubleshoot-guide/client-connection-increasing-constantly.jpg" alt-text="Connexion client augmentant constamment":::

### <a name="root-cause"></a>Cause racine :

Commande `DisposeAsync` de la connexion client SignalR jamais appelée. La connexion reste ouverte.

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

1. Vérifiez si le client SignalR ne se ferme **jamais**.

### <a name="solution"></a>Solution

Vérifiez si vous fermez la connexion. Appelez manuellement la commande `HubConnection.DisposeAsync()` pour arrêter la connexion après l’avoir utilisée.

Par exemple :

```C#
var connection = new HubConnectionBuilder()
    .WithUrl(...)
    .Build();
try
{
    await connection.StartAsync();
    // Do your stuff
    await connection.StopAsync();
}
finally
{
    await connection.DisposeAsync();
}
```

### <a name="common-improper-client-connection-usage"></a>Utilisation incorrecte courante de la connexion client

#### <a name="azure-function-example"></a>Exemple de fonction Azure 

Ce problème se produit souvent quand quelqu’un établit une connexion client SignalR dans une méthode de fonction Azure au lieu d’en faire un membre statique de votre classe de fonction. Vous pourriez vous attendre à ce qu’une seule connexion client soit établie, mais vous constatez que le nombre de connexions client augmente constamment dans les métriques de la section Supervision du menu de ressources du portail Azure, et que toutes ces connexions sont interrompues uniquement après le redémarrage de la fonction Azure ou d’Azure SignalR Service. En effet, pour **chaque** demande, la fonction Azure crée **une** connexion client, et si vous n’arrêtez pas la connexion client dans la méthode de fonction, le client conserve les connexions actives à Azure SignalR Service.

#### <a name="solution"></a>Solution

* N’oubliez pas de fermer la connexion client si vous utilisez des clients SignalR dans une fonction Azure ou si vous utilisez un client SignalR en tant que singleton.
* Au lieu d’utiliser des clients SignalR dans une fonction Azure, vous pouvez créer des clients SignalR partout ailleurs et utiliser des [Liaisons Azure Functions pour Azure SignalR Service](https://github.com/Azure/azure-functions-signalrservice-extension) afin de [négocier](https://github.com/Azure/azure-functions-signalrservice-extension/blob/dev/samples/simple-chat/csharp/FunctionApp/Functions.cs#L22) l’accès du client à Azure SignalR. Vous pouvez également utiliser la liaison pour [envoyer des messages](https://github.com/Azure/azure-functions-signalrservice-extension/blob/dev/samples/simple-chat/csharp/FunctionApp/Functions.cs#L40). Vous trouverez des exemples pour négocier l’accès du client et envoyer des messages [ici](https://github.com/Azure/azure-functions-signalrservice-extension/tree/dev/samples). Des informations supplémentaires sont disponibles [ici](https://github.com/Azure/azure-functions-signalrservice-extension).
* Lorsque vous utilisez des clients SignalR dans une fonction Azure Function, il peut y avoir une meilleure architecture à votre scénario. Vérifiez si vous concevez une architecture sans serveur appropriée. Vous pouvez vous référer à [Applications sans serveur en temps réel avec les liaisons SignalR Service dans Azure Functions](https://www.nuget.org/packages/Microsoft.Azure.WebJobs.Extensions.SignalRService).

<a name="server_connection_drop"></a>

## <a name="server-connection-drops"></a>Pertes de connexion serveur

Au démarrage du serveur d’applications, en arrière-plan, le Kit de développement logiciel (SDK) Azure commence à initier des connexions serveur à l’Azure SignalR distant. Comme décrit dans [Éléments internes d’Azure SignalR Service](https://github.com/Azure/azure-signalr/blob/dev/docs/internal.md), Azure SignalR achemine les trafics clients entrants vers ces connexions serveur. Une fois qu’une connexion serveur est interrompue, toutes les connexions client qu’elle dessert sont également fermées.

Les connexions entre le serveur d’applications et SignalR Service étant persistantes, elles peuvent rencontrer des problèmes de connectivité réseau. Dans le Kit de développement logiciel (SDK) serveur, nous avons une stratégie **Toujours se reconnecter** pour les connexions serveur. Nous encourageons également les utilisateurs à ajouter une logique de reconnexion continue aux clients avec un délai aléatoire afin d’éviter l’envoi d’une quantité massive de demandes simultanées au serveur.

De nouvelles versions d’Azure SignalR Service sont publiées régulièrement et, parfois des mises à jour correctives ou des mises à niveau de système d’exploitation à l’échelle d’Azure, qui interrompent occasionnellement nos services dépendants. Celles-ci peuvent entraîner une interruption de service pendant une période limitée mais tant que le mécanisme de déconnexion/reconnexion opère côté client, l’impact est minime, comme toute déconnexion/reconnexion intervenant côté client.

Cette section décrit plusieurs causes possibles de perte de connexion serveur, et fournit des conseils sur la façon d’identifier la cause racine.

### <a name="possible-errors-seen-from-server-side"></a>Erreurs possibles côté serveur :

* `[Error]Connection "..." to the service was dropped`
* `The remote party closed the WebSocket connection without completing the close handshake`
* `Service timeout. 30.00ms elapsed without receiving a message from service.`

### <a name="root-cause"></a>Cause racine :

La connexion serveur-service est fermée par **ASRS**(**A** zure **S** ignal **R** **S** ervice).

### <a name="troubleshooting-guide"></a>Guide de résolution des problèmes

1. Ouvrez le journal côté serveur d’applications pour voir si quelque chose d’anormal s’est produit.
2. Vérifiez le journal des événements côté serveur d’applications pour voir si le serveur d’applications a redémarré.
3. Ouvrez un problème chez nous en fournissant le laps de temps, et envoyez-nous par e-mail le nom de la ressource.

## <a name="tips"></a>Conseils

<a name="view_request"></a>

* Comment afficher la demande sortante du client ?
Prenons ASP.NET Core par exemple (ASP.NET est similaire) :
    * À partir d’un navigateur :

        Prenons par exemple Chrome. Vous pouvez utiliser la touche **F12** pour ouvrir la fenêtre de console et basculer vers l’onglet **Réseau**. Il se peut que vous deviez actualiser la page à l’aide de la touche **F5** pour capturer le réseau dès le début.

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/chrome-network.gif" alt-text="Affichage Réseau dans Chrome":::

    * À partir d’un client C# :

        Vous pouvez afficher les trafics web locaux à l’aide de [Fiddler](https://www.telerik.com/fiddler). Les trafics WebSocket sont pris en charge depuis Fiddler version 4.5.

        :::image type="content" source="./media/signalr-howto-troubleshoot-guide/fiddler-view-network-inline.png" alt-text="Affichage Réseau dans Fiddler" lightbox="./media/signalr-howto-troubleshoot-guide/fiddler-view-network.png":::

<a name="restart_connection"></a>

* Comment redémarrer une connexion client ?
    
    Voici les [exemples de code](https://github.com/Azure/azure-signalr/tree/dev/samples) contenant une logique de redémarrage de connexion avec une stratégie *TOUJOURS RÉESSAYER* :

    * [Client C# ASP.NET Core](https://github.com/Azure/azure-signalr/tree/dev/samples/ChatSample/ChatSample.CSharpClient/Program.cs#L64)

    * [Client JavaScript ASP.NET Core](https://github.com/Azure/azure-signalr/tree/dev/samples/ChatSample/ChatSample/wwwroot/index.html#L164)

    * [Client C# ASP.NET](https://github.com/Azure/azure-signalr/tree/dev/samples/AspNet.ChatSample/AspNet.ChatSample.CSharpClient/Program.cs#L78)

    * [Client JavaScript ASP.NET](https://github.com/Azure/azure-signalr/tree/dev/samples/AspNet.ChatSample/AspNet.ChatSample.JavaScriptClient/wwwroot/index.html#L71)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à gérer les problèmes courants. Vous avez également découvert d’autres méthodes de dépannage génériques. 

> [!div class="nextstepaction"]
> [Comment résoudre des problèmes de connectivité et de remise de messages](./signalr-howto-troubleshoot-method.md)
---
title: Vue d’ensemble des API .NET standard Azure Relay | Microsoft Docs
description: Cet article résume une partie de la clé de Azure Relay Connexions hybrides .NET Standard API.
ms.topic: article
ms.custom: devx-track-csharp
ms.date: 06/23/2020
ms.openlocfilehash: 44d5800c08b49118e99a678e31d02e5b7a1f550c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88935668"
---
# <a name="azure-relay-hybrid-connections-net-standard-api-overview"></a>Présentation des API .NET standard pour les connexions hybrides Azure Relay

Cet article passe en revue les principales [API clientes](/dotnet/api/microsoft.azure.relay) .NET standard pour les connexions hybrides Azure Relay.
  
## <a name="relay-connection-string-builder-class"></a>Classe Générateur de chaînes de connexion Relay

La classe [RelayConnectionStringBuilder][RelayConnectionStringBuilder] met en forme des chaînes de connexion propres aux connexions hybride Relay. Vous pouvez l’utiliser pour vérifier le format d’une chaîne de connexion ou pour créer une chaîne de connexion à partir de zéro. Le code suivant montre un exemple :

```csharp
var endpoint = "[Relay namespace]";
var entityPath = "[Name of the Hybrid Connection]";
var sharedAccessKeyName = "[SAS key name]";
var sharedAccessKey = "[SAS key value]";

var connectionStringBuilder = new RelayConnectionStringBuilder()
{
    Endpoint = endpoint,
    EntityPath = entityPath,
    SharedAccessKeyName = sasKeyName,
    SharedAccessKey = sasKeyValue
};
```

Vous pouvez également transmettre une chaîne de connexion directement à la méthode `RelayConnectionStringBuilder`. Cette opération vous permet de vérifier que le format de la chaîne de connexion est valide. Si des paramètres ne sont pas valides, le constructeur génère une `ArgumentException`.

```csharp
var myConnectionString = "[RelayConnectionString]";
// Declare the connectionStringBuilder so that it can be used outside of the loop if needed
RelayConnectionStringBuilder connectionStringBuilder;
try
{
    // Create the connectionStringBuilder using the supplied connection string
    connectionStringBuilder = new RelayConnectionStringBuilder(myConnectionString);
}
catch (ArgumentException ae)
{
    // Perform some error handling
}
```

## <a name="hybrid-connection-stream"></a>Flux de connexion hybride

La classe [HybridConnectionStream][HCStream] est l’objet principal utilisé pour envoyer et recevoir des données à partir d’un point de terminaison Azure Relay si vous travaillez avec un[ HybridConnectionClient][HCClient] ou un [HybridConnectionListener][HCListener].

### <a name="getting-a-hybrid-connection-stream"></a>Obtention d’un flux de connexion hybride

#### <a name="listener"></a>Écouteur

À l’aide d’un objet [HybridConnectionListener][HCListener], vous pouvez obtenir un `HybridConnectionStream`objet comme suit :

```csharp
// Use the RelayConnectionStringBuilder to get a valid connection string
var listener = new HybridConnectionListener(csb.ToString());
// Open a connection to the Relay endpoint
await listener.OpenAsync();
// Get a `HybridConnectionStream`
var hybridConnectionStream = await listener.AcceptConnectionAsync();
```

#### <a name="client"></a>Client

À l’aide d’un objet [HybridConnectionClient][HCClient], vous pouvez obtenir un objet `HybridConnectionStream` comme suit :

```csharp
// Use the RelayConnectionStringBuilder to get a valid connection string
var client = new HybridConnectionClient(csb.ToString());
// Open a connection to the Relay endpoint and get a `HybridConnectionStream`
var hybridConnectionStream = await client.CreateConnectionAsync();
```

### <a name="receiving-data"></a>Réception de données

La classe [HybridConnectionStream][HCStream] permet une communication bidirectionnelle. Dans la plupart des cas, vous recevez en continu à partir du flux. Si vous lisez du texte à partir du flux, vous pouvez également utiliser un objet [StreamReader](/dotnet/api/system.io.streamreader?view=netcore-3.1), qui facilite l’analyse des données. Par exemple, vous pouvez lire les données sous forme de texte plutôt que sous forme de `byte[]`.

Le code suivant lit des lignes individuelles de texte à partir du flux jusqu’à ce qu’une annulation soit demandée :

```csharp
// Create a CancellationToken, so that we can cancel the while loop
var cancellationToken = new CancellationToken();
// Create a StreamReader from the 'hybridConnectionStream`
var streamReader = new StreamReader(hybridConnectionStream);

while (!cancellationToken.IsCancellationRequested)
{
    // Read a line of input until a newline is encountered
    var line = await streamReader.ReadLineAsync();
    if (string.IsNullOrEmpty(line))
    {
        // If there's no input data, we will signal that 
        // we will no longer send data on this connection
        // and then break out of the processing loop.
        await hybridConnectionStream.ShutdownAsync(cancellationToken);
        break;
    }
}
```

### <a name="sending-data"></a>Envoi de données

Une fois qu’une connexion a été établie, vous pouvez envoyer un message au point de terminaison Relay. Parce que l’objet de connexion hérite de la classe [Stream](/dotnet/api/system.io.stream?view=netcore-3.1), envoyez vos données sous forme de `byte[]`. L’exemple suivant vous montre comment procéder :

```csharp
var data = Encoding.UTF8.GetBytes("hello");
await clientConnection.WriteAsync(data, 0, data.Length);
```

Toutefois, si vous souhaitez envoyer du texte directement, sans avoir à coder la chaîne à chaque fois, vous pouvez encapsuler l’objet `hybridConnectionStream` avec un objet [StreamWriter](/dotnet/api/system.io.streamwriter?view=netcore-3.1).

```csharp
// The StreamWriter object only needs to be created once
var textWriter = new StreamWriter(hybridConnectionStream);
await textWriter.WriteLineAsync("hello");
```

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure Relay, consultez les liens suivants :

* [Informations de référence sur Microsoft.Azure.Relay](/dotnet/api/microsoft.azure.relay)
* [Qu’est-ce qu’Azure Relay ?](relay-what-is-it.md)
* [API Relay disponibles](relay-api-overview.md)

[RelayConnectionStringBuilder]: /dotnet/api/microsoft.azure.relay.relayconnectionstringbuilder
[HCStream]: /dotnet/api/microsoft.azure.relay.hybridconnectionstream
[HCClient]: /dotnet/api/microsoft.azure.relay.hybridconnectionclient
[HCListener]: /dotnet/api/microsoft.azure.relay.hybridconnectionlistener

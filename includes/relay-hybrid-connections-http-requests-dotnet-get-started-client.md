---
title: Fichier Include
description: Fichier Include
services: service-bus-relay
author: clemensv
ms.service: service-bus-relay
ms.topic: include
ms.date: 08/16/2018
ms.author: clemensv
ms.custom: include file
ms.openlocfilehash: ce29cd03de46e1d93d7f1f28f9f5184cd59a57e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "79200340"
---
### <a name="create-a-console-application"></a>Création d’une application console

Si vous avez désactivé l’option « Nécessite l’autorisation client » lors de la création du Relai, vous pouvez envoyer des requêtes à l’URL de Connexions hybrides avec n’importe quel navigateur. Pour accéder à des points de terminaison protégés, vous devez créer et mettre un jeton dans l’en-tête `ServiceBusAuthorization`, montré ici.

Dans Visual Studio, créez un nouveau projet **Application de console (.NET Framework)** .

### <a name="add-the-relay-nuget-package"></a>Ajout du package NuGet de relais

1. Cliquez avec le bouton droit sur le projet créé puis sélectionnez **Gérer les packages NuGet**.
2. Sélectionnez l’option **Inclure la version préliminaire**. 
3. Sélectionnez **Parcourir**, puis recherchez **Microsoft.Azure.Relay**. Dans les résultats de la recherche, sélectionnez **Microsoft Azure Relay**.
4. Pour la version, sélectionnez **2.0.0-preview1-20180523**. 
5. Sélectionnez **Installer** pour terminer l’installation. Fermez la boîte de dialogue.

### <a name="write-code-to-send-requests"></a>Écrire du code pour envoyer des requêtes

1. Au début du fichier Program.cs, remplacez les instructions `using` actuelles par les instructions `using` suivantes :
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Net.Http;
    using Microsoft.Azure.Relay;
    ```
2. Ajoutez des constantes à la classe `Program` pour les détails de la connexion hybride. Remplacez les espaces réservés entre crochets par les valeurs obtenues lors de la création de la connexion hybride. Veillez à utiliser le nom de l’espace de noms qualifié complet.
   
    ```csharp
    private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
    private const string ConnectionName = "{HybridConnectionName}";
    private const string KeyName = "{SASKeyName}";
    private const string Key = "{SASKey}";
    ```
3. Ajoutez la méthode suivante à la classe `Program` :
   
    ```csharp
    private static async Task RunAsync()
    {
        var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
        var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
        var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
        var client = new HttpClient();
        var request = new HttpRequestMessage()
        {
            RequestUri = uri,
            Method = HttpMethod.Get,
        };
        request.Headers.Add("ServiceBusAuthorization", token);
        var response = await client.SendAsync(request);
        Console.WriteLine(await response.Content.ReadAsStringAsync());        Console.ReadLine();
    }
    ```
4. Ajoutez la ligne de code suivante à la méthode `Main` dans la classe `Program`.
   
    ```csharp
    RunAsync().GetAwaiter().GetResult();
    ```
   
    Le fichier Program.cs doit ressembler à cela :
   
    ```csharp
    using System;
    using System.IO;
    using System.Threading;
    using System.Threading.Tasks;
    using System.Net.Http;
    using Microsoft.Azure.Relay;
   
    namespace Client
    {
        class Program
        {
            private const string RelayNamespace = "{RelayNamespace}.servicebus.windows.net";
            private const string ConnectionName = "{HybridConnectionName}";
            private const string KeyName = "{SASKeyName}";
            private const string Key = "{SASKey}";
   
            static void Main(string[] args)
            {
                RunAsync().GetAwaiter().GetResult();
            }
   
            private static async Task RunAsync()
            {
               var tokenProvider = TokenProvider.CreateSharedAccessSignatureTokenProvider(
                KeyName, Key);
                var uri = new Uri(string.Format("https://{0}/{1}", RelayNamespace, ConnectionName));
                var token = (await tokenProvider.GetTokenAsync(uri.AbsoluteUri, TimeSpan.FromHours(1))).TokenString;
                var client = new HttpClient();
                var request = new HttpRequestMessage()
                {
                    RequestUri = uri,
                    Method = HttpMethod.Get,
                };
                request.Headers.Add("ServiceBusAuthorization", token);
                var response = await client.SendAsync(request);
                Console.WriteLine(await response.Content.ReadAsStringAsync());
            }
        }
    }
    ```


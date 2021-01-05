---
title: Interrogation des opérations de longue durée | Microsoft Docs
description: 'Azure Media Services propose des API qui envoient des requêtes à Media Services pour démarrer des opérations (par exemple : créer, démarrer, arrêter ou supprimer un canal). Ce sont toutes des opérations de longue durée. Ce sujet montre comment interroger les opérations de longue durée.'
services: media-services
documentationcenter: ''
author: Juliako
writer: juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: 44cecbd8d2cdc95e342d7aaf2b33f6cc0192e182
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89262025"
---
# <a name="delivering-live-streaming-with-azure-media-services"></a>Diffusion vidéo en flux continu avec Azure Media Services

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

## <a name="overview"></a>Vue d’ensemble

Microsoft Azure Media Services propose des API qui envoient des requêtes à Media Services pour démarrer des opérations (par exemple : créer, démarrer, arrêter ou supprimer un canal). Ces opérations sont des opérations de longue durée.

Le kit de développement logiciel (SDK) .NET de Media Services fournit des API qui envoient la requête et attendent que l'opération se termine (en interne, les API interrogent la progression des opérations à intervalles donnés). Par exemple, lorsque vous appelez channel.Start(), la méthode est renvoyée une fois le canal démarré. Vous pouvez également utiliser la version asynchrone : await channel.StartAsync(). Pour plus d’informations sur le modèle asynchrone basé sur des tâches, consultez [TAP](./media-services-mes-schema.md). Les API qui envoient une demande d'opération, puis interrogent l'état de l'opération jusqu'à ce que celle-ci soit terminée sont appelées « méthodes d'interrogation » Ces méthodes (en particulier la version asynchrone) sont recommandées pour les applications clientes riches et/ou les services avec état.

Dans certains scénarios, une application ne peut pas attendre une requête HTTP de longue durée et cherche à interroger manuellement la progression de l'opération. Par exemple, un navigateur qui interagit avec un service Web sans état : lorsque le navigateur demande à créer un canal, le service web lance une opération de longue durée et renvoie l'ID d'opération dans le navigateur. Le navigateur peut alors demander au service Web l'état de l'opération, en fonction de l'ID. Le kit de développement logiciel (SDK) .NET de Media Services fournit des API qui sont utiles dans ce genre de scénario. Ces API sont appelées « méthodes sans interrogation ».
Le modèle de dénomination des « méthodes sans interrogation » est le suivant : Send*NomOpération*Operation (par exemple, SendCreateOperation). Les méthodes Send*NomOpération*Operation retournent l’objet **IOperation**. L’objet retourné contient des informations qui peuvent être utilisées pour suivre l’opération. Les méthodes Send*NomOpération*OperationAsync retournent **Task\<IOperation>** .

Actuellement, les méthodes sans interrogation sont prises en charge par les classes suivantes :  **Channel**, **StreamingEndpoint** et **Program**.

Pour interroger l’état de l’opération, utilisez la méthode **GetOperation** sur la classe **OperationBaseCollection**. Utilisez les intervalles suivants pour vérifier l’état de l’opération : 30 secondes pour les opérations **Channel** et **StreamingEndpoint** ; 10 secondes pour les opérations **Program**.

## <a name="create-and-configure-a-visual-studio-project"></a>Créer et configurer un projet Visual Studio

Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md).

## <a name="example"></a>Exemple

L’exemple suivant définit une classe appelée **ChannelOperations**. Cette définition de classe peut constituer un point de départ pour la définition de classe de votre service Web. Par souci de simplicité, les exemples suivants utilisent les versions synchrones des méthodes.

L'exemple montre également comment le client peut utiliser cette classe.

### <a name="channeloperations-class-definition"></a>Définition de la classe ChannelOperations

```csharp
using Microsoft.WindowsAzure.MediaServices.Client;
using System;
using System.Collections.Generic;
using System.Configuration;
using System.Net;

/// <summary> 
/// The ChannelOperations class only implements 
/// the Channel’s creation operation. 
/// </summary> 
public class ChannelOperations
{
    // Read values from the App.config file.
    private static readonly string _AADTenantDomain =
        ConfigurationManager.AppSettings["AMSAADTenantDomain"];
    private static readonly string _RESTAPIEndpoint =
        ConfigurationManager.AppSettings["AMSRESTAPIEndpoint"];
    private static readonly string _AMSClientId =
        ConfigurationManager.AppSettings["AMSClientId"];
    private static readonly string _AMSClientSecret =
        ConfigurationManager.AppSettings["AMSClientSecret"];

    // Field for service context.
    private static CloudMediaContext _context = null;

    public ChannelOperations()
    {
        AzureAdTokenCredentials tokenCredentials = 
            new AzureAdTokenCredentials(_AADTenantDomain,
                new AzureAdClientSymmetricKey(_AMSClientId, _AMSClientSecret),
                AzureEnvironments.AzureCloudEnvironment);

        var tokenProvider = new AzureAdTokenProvider(tokenCredentials);

        _context = new CloudMediaContext(new Uri(_RESTAPIEndpoint), tokenProvider);
    }

    /// <summary>  
    /// Initiates the creation of a new channel.  
    /// </summary>  
    /// <param name="channelName">Name to be given to the new channel</param>  
    /// <returns>  
    /// Operation Id for the long running operation being executed by Media Services. 
    /// Use this operation Id to poll for the channel creation status. 
    /// </returns> 
    public string StartChannelCreation(string channelName)
    {
        var operation = _context.Channels.SendCreateOperation(
            new ChannelCreationOptions
            {
                Name = channelName,
                Input = CreateChannelInput(),
                Preview = CreateChannelPreview(),
                Output = CreateChannelOutput()
            });

        return operation.Id;
    }

    /// <summary> 
    /// Checks if the operation has been completed. 
    /// If the operation succeeded, the created channel Id is returned in the out parameter.
    /// </summary> 
    /// <param name="operationId">The operation Id.</param> 
    /// <param name="channel">
    /// If the operation succeeded, 
    /// the created channel Id is returned in the out parameter.</param>
    /// <returns>Returns false if the operation is still in progress; otherwise, true.</returns> 
    public bool IsCompleted(string operationId, out string channelId)
    {
        IOperation operation = _context.Operations.GetOperation(operationId);
        bool completed = false;

        channelId = null;

        switch (operation.State)
        {
            case OperationState.Failed:
                // Handle the failure. 
                // For example, throw an exception. 
                // Use the following information in the exception: operationId, operation.ErrorMessage.
                break;
            case OperationState.Succeeded:
                completed = true;
                channelId = operation.TargetEntityId;
                break;
            case OperationState.InProgress:
                completed = false;
                break;
        }
        return completed;
    }

    private static ChannelInput CreateChannelInput()
    {
        return new ChannelInput
        {
            StreamingProtocol = StreamingProtocol.RTMP,
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelInput001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelPreview CreateChannelPreview()
    {
        return new ChannelPreview
        {
            AccessControl = new ChannelAccessControl
            {
                IPAllowList = new List<IPRange>
                    {
                        new IPRange
                        {
                            Name = "TestChannelPreview001",
                            Address = IPAddress.Parse("0.0.0.0"),
                            SubnetPrefixLength = 0
                        }
                    }
            }
        };
    }

    private static ChannelOutput CreateChannelOutput()
    {
        return new ChannelOutput
        {
            Hls = new ChannelOutputHls { FragmentsPerSegment = 1 }
        };
    }
}
```

### <a name="the-client-code"></a>Le code du client

```csharp
ChannelOperations channelOperations = new ChannelOperations();
string opId = channelOperations.StartChannelCreation("MyChannel001");

string channelId = null;
bool isCompleted = false;

while (isCompleted == false)
{
    System.Threading.Thread.Sleep(TimeSpan.FromSeconds(30));
    isCompleted = channelOperations.IsCompleted(opId, out channelId);
}

// If we got here, we should have the newly created channel id.
Console.WriteLine(channelId);
```

## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]
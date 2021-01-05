---
title: Migrer vos données de visage d’un abonnement à un autre – Visage
titleSuffix: Azure Cognitive Services
description: Ce guide vous montre comment migrer vos données de visage stockées d’un abonnement Visage à un autre.
services: cognitive-services
author: nitinme
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: conceptual
ms.date: 09/06/2019
ms.author: nitinme
ms.custom: devx-track-csharp
ms.openlocfilehash: 74861df30ba2854c9299e1f779d0cee59abbc5a8
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92911203"
---
# <a name="migrate-your-face-data-to-a-different-face-subscription"></a>Migrer vos données de visage vers un autre abonnement API Visage

Ce guide vous montre comment déplacer des données de visage, comme un objet PersonGroup enregistré avec des visages, vers un autre abonnement Visage d’Azure Cognitive Services. Pour déplacer les données, vous utilisez la fonctionnalité de capture instantanée. Cela vous évite d’avoir à créer et entraîner à plusieurs reprises un objet PersonGroup ou FaceList quand vous déplacez ou développez vos opérations. Par exemple, vous avez peut-être créé un objet PersonGroup avec un abonnement gratuit et vous souhaitez à présent migrer cet objet vers votre abonnement payant. Ou vous avez besoin de synchroniser les données de visage entre des abonnements dans différentes régions en vue d’une opération à grande échelle.

Cette même stratégie de migration s’applique également aux objets LargePersonGroup et LargeFaceList. Si vous ne connaissez pas bien les concepts utilisés dans ce guide, lisez les définitions correspondantes dans le guide [Concepts de reconnaissance faciale](../concepts/face-recognition.md). Ce guide utilise la bibliothèque cliente .NET de Visage avec C#.

## <a name="prerequisites"></a>Prérequis

Vous devez disposer des éléments suivants :

- Deux clés d’abonnement Visage, une avec les données existantes et l’autre pour les données migrées. Pour vous abonner au service Visage et obtenir votre clé, suivez les instructions dans [Créer un compte Cognitive Services](../../cognitive-services-apis-create-account.md).
- La chaîne d’ID d’abonnement Visage qui correspond à l’abonnement cible. Pour la rechercher, sélectionnez **Vue d’ensemble** dans le portail Azure. 
- N’importe quelle édition de [Visual Studio 2015 ou 2017](https://www.visualstudio.com/downloads/).

## <a name="create-the-visual-studio-project"></a>Créer le projet Visual Studio

Ce guide utilise une application console simple pour exécuter la migration des données de visage. Pour une implémentation complète, consultez l’[exemple d’instantané Visage](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) sur GitHub.

1. Dans Visual Studio, créez un nouveau projet Application de console (.NET Framework). Nommez-le **FaceApiSnapshotSample**.
1. Récupérez les packages NuGet requis. Cliquez avec le bouton droit sur le projet dans l’Explorateur de solutions, puis sélectionnez **Gérer les packages NuGet**. Sélectionnez l’onglet **Parcourir** , puis **Inclure la préversion**. Recherchez et installez le package suivant :
    - [Microsoft.Azure.CognitiveServices.Vision.Face 2.3.0-preview](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Vision.Face/2.2.0-preview)

## <a name="create-face-clients"></a>Créer des instances FaceClient

Dans la méthode **Main** dans *Program.cs* , créez deux instances [FaceClient](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.faceclient?view=azure-dotnet) pour vos abonnements source et cible. Cet exemple utilise un abonnement API Visage dans la région Asie Est comme source et un autre abonnement dans la région USA Ouest comme cible. Cet exemple montre comment migrer des données d’une région Azure vers une autre. 

[!INCLUDE [subdomains-note](../../../../includes/cognitive-services-custom-subdomains-note.md)]

```csharp
var FaceClientEastAsia = new FaceClient(new ApiKeyServiceClientCredentials("<East Asia Subscription Key>"))
    {
        Endpoint = "https://southeastasia.api.cognitive.microsoft.com/>"
    };

var FaceClientWestUS = new FaceClient(new ApiKeyServiceClientCredentials("<West US Subscription Key>"))
    {
        Endpoint = "https://westus.api.cognitive.microsoft.com/"
    };
```

Indiquez les valeurs des clé d’abonnement et les URL de point de terminaison pour vos abonnements source et cible.


## <a name="prepare-a-persongroup-for-migration"></a>Préparer un PersonGroup en vue de sa migration

Vous avez besoin de l’ID de l’objet PersonGroup dans votre abonnement source que vous voulez migrer vers l’abonnement cible. Utilisez la méthode [PersonGroupOperationsExtensions.ListAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperationsextensions.listasync?view=azure-dotnet) pour récupérer une liste de vos objets PersonGroup. Ensuite, obtenez la propriété [PersonGroup.PersonGroupId](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.models.persongroup.persongroupid?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Vision_Face_Models_PersonGroup_PersonGroupId). Ce processus diffère en fonction des objets PersonGroup dont vous disposez. Dans ce guide, l’ID de l’objet PersonGroup source est stocké dans `personGroupId`.

> [!NOTE]
> L’[exemple de code](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample) crée et entraîne un nouvel objet PersonGroup à migrer. Dans la plupart des cas, vous devez déjà disposer d’un objet PersonGroup à utiliser.

## <a name="take-a-snapshot-of-a-persongroup"></a>Prendre un instantané d’un objet PersonGroup

Un instantané est un stockage à distance temporaire pour certains types de données de l’API Visage. Il fonctionne comme une sorte de Presse-papiers pour copier des données d’un abonnement à un autre. Tout d’abord, vous prenez un instantané des données dans l’abonnement source. Ensuite, vous l’appliquez à un nouvel objet de données dans l’abonnement cible.

Utilisez l’instance FaceClient de l’abonnement source pour prendre un instantané de l’objet PersonGroup. Utilisez [TakeAsync](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperationsextensions.takeasync?view=azure-dotnet) avec l’ID de l’objet PersonGroup et l’ID de l’abonnement cible. Si vous avez plusieurs abonnements cibles, ajoutez-les en tant qu’entrées de tableau dans le troisième paramètre.

```csharp
var takeSnapshotResult = await FaceClientEastAsia.Snapshot.TakeAsync(
    SnapshotObjectType.PersonGroup,
    personGroupId,
    new[] { "<Azure West US Subscription ID>" /* Put other IDs here, if multiple target subscriptions wanted */ });
```

> [!NOTE]
> Le processus de prise et d’application d’instantanés n’interrompt pas les appels réguliers aux objets PersonGroup ou FaceList source et cible. N’effectuez pas d’appels simultanés qui modifient l’objet source, tels que des [appels de gestion FaceList](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.facelistoperations?view=azure-dotnet) ou l’appel [PersonGroup - Entraîner](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.persongroupoperations?view=azure-dotnet), par exemple. L’opération de capture instantanée peut s’exécuter avant ou après ces opérations, ou peut rencontrer des erreurs.

## <a name="retrieve-the-snapshot-id"></a>Récupérer l’ID de l’instantané

La méthode utilisée pour prendre des instantanés étant asynchrone, vous devez attendre sa conclusion. Les opérations sur les instantanés ne peuvent pas être annulées. Dans ce code, la méthode `WaitForOperation` surveille l’appel asynchrone. Elle vérifie l’état toutes les 100 ms. Une fois l’opération terminée, récupérez un ID d’opération en analysant le champ `OperationLocation`. 

```csharp
var takeOperationId = Guid.Parse(takeSnapshotResult.OperationLocation.Split('/')[2]);
var operationStatus = await WaitForOperation(FaceClientEastAsia, takeOperationId);
```

Une valeur `OperationLocation` standard ressemble à :

```csharp
"/operations/a63a3bdd-a1db-4d05-87b8-dbad6850062a"
```

La méthode d’assistance `WaitForOperation` est ici :

```csharp
/// <summary>
/// Waits for the take/apply operation to complete and returns the final operation status.
/// </summary>
/// <returns>The final operation status.</returns>
private static async Task<OperationStatus> WaitForOperation(IFaceClient client, Guid operationId)
{
    OperationStatus operationStatus = null;
    do
    {
        if (operationStatus != null)
        {
            Thread.Sleep(TimeSpan.FromMilliseconds(100));
        }

        // Get the status of the operation.
        operationStatus = await client.Snapshot.GetOperationStatusAsync(operationId);

        Console.WriteLine($"Operation Status: {operationStatus.Status}");
    }
    while (operationStatus.Status != OperationStatusType.Succeeded
            && operationStatus.Status != OperationStatusType.Failed);

    return operationStatus;
}
```

Une fois que l’état de l’opération a indiqué `Succeeded`, obtenez l’ID de l’instantané en analysant le champ `ResourceLocation` de l’instance OperationStatus retournée.

```csharp
var snapshotId = Guid.Parse(operationStatus.ResourceLocation.Split('/')[2]);
```

Une valeur `resourceLocation` standard ressemble à :

```csharp
"/snapshots/e58b3f08-1e8b-4165-81df-aa9858f233dc"
```

## <a name="apply-a-snapshot-to-a-target-subscription"></a>Appliquer un instantané à un abonnement cible

Ensuite, créez l’objet PersonGroup dans l’abonnement cible en utilisant un ID généré de manière aléatoire. Ensuite, utilisez l’instance FaceClient de l’abonnement cible pour appliquer l’instantané à cet objet PersonGroup. Transmettez l’ID de l’instantané et le nouvel ID de l’objet PersonGroup.

```csharp
var newPersonGroupId = Guid.NewGuid().ToString();
var applySnapshotResult = await FaceClientWestUS.Snapshot.ApplyAsync(snapshotId, newPersonGroupId);
```


> [!NOTE]
> Un objet instantané est valide 48 heures uniquement. Prenez un instantané uniquement si vous envisagez de l’utiliser peu de temps après pour la migration des données.

Une demande d’application d’instantané retourne un autre ID d’opération. Pour obtenir cet ID, analysez le champ `OperationLocation` de l’instance applySnapshotResult retournée. 

```csharp
var applyOperationId = Guid.Parse(applySnapshotResult.OperationLocation.Split('/')[2]);
```

Le processus d’application de l’instantané est également asynchrone. Là encore, vous devez utiliser `WaitForOperation` pour attendre la fin de son exécution.

```csharp
operationStatus = await WaitForOperation(FaceClientWestUS, applyOperationId);
```

## <a name="test-the-data-migration"></a>Tester la migration des données

Une fois que vous avez appliqué l’instantané, le nouvel objet PersonGroup dans l’abonnement cible est rempli avec les données de visage d’origine. Par défaut, les résultats de l’entraînement sont également copiés. Le nouvel objet PersonGroup est prêt pour les appels d’identification de visage sans avoir besoin d’un réentraînement.

Pour tester la migration des données, exécutez les opérations suivantes et comparez les résultats affichés dans la console :

```csharp
await DisplayPersonGroup(FaceClientEastAsia, personGroupId);
await IdentifyInPersonGroup(FaceClientEastAsia, personGroupId);

await DisplayPersonGroup(FaceClientWestUS, newPersonGroupId);
// No need to retrain the person group before identification,
// training results are copied by snapshot as well.
await IdentifyInPersonGroup(FaceClientWestUS, newPersonGroupId);
```

Utilisez les méthodes d’assistance suivantes :

```csharp
private static async Task DisplayPersonGroup(IFaceClient client, string personGroupId)
{
    var personGroup = await client.PersonGroup.GetAsync(personGroupId);
    Console.WriteLine("Person Group:");
    Console.WriteLine(JsonConvert.SerializeObject(personGroup));

    // List persons.
    var persons = await client.PersonGroupPerson.ListAsync(personGroupId);

    foreach (var person in persons)
    {
        Console.WriteLine(JsonConvert.SerializeObject(person));
    }

    Console.WriteLine();
}
```

```csharp
private static async Task IdentifyInPersonGroup(IFaceClient client, string personGroupId)
{
    using (var fileStream = new FileStream("data\\PersonGroup\\Daughter\\Daughter1.jpg", FileMode.Open, FileAccess.Read))
    {
        var detectedFaces = await client.Face.DetectWithStreamAsync(fileStream);

        var result = await client.Face.IdentifyAsync(detectedFaces.Select(face => face.FaceId.Value).ToList(), personGroupId);
        Console.WriteLine("Test identify against PersonGroup");
        Console.WriteLine(JsonConvert.SerializeObject(result));
        Console.WriteLine();
    }
}
```

Maintenant, vous pouvez utiliser le nouvel objet PersonGroup dans l’abonnement cible. 

Pour remettre à jour l’objet PersonGroup cible à l’avenir, créez un nouvel objet PersonGroup pour recevoir l’instantané. Pour cela, suivez les étapes décrites dans ce guide. Un instantané ne peut être appliqué qu’une seule fois à un objet PersonGroup individuel.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Après avoir terminé la migration des données de visage, supprimez manuellement l’objet instantané.

```csharp
await FaceClientEastAsia.Snapshot.DeleteAsync(snapshotId);
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, consultez la documentation de référence d’API appropriée, explorez un exemple d’application qui utilise la fonctionnalité de capture instantanée ou suivez un guide pratique pour commencer à utiliser les autres opérations d’API mentionnées ici :

- [Documentation de référence sur les instantanés (SDK .NET)](/dotnet/api/microsoft.azure.cognitiveservices.vision.face.snapshotoperations?view=azure-dotnet)
- [Exemple d’instantané Visage](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/app-samples/FaceApiSnapshotSample/FaceApiSnapshotSample)
- [Ajouter des visages](how-to-add-faces.md)
- [Détecter des visages sur une image](HowtoDetectFacesinImage.md)
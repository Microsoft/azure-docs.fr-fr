---
title: 'Exemple : Utiliser la fonctionnalité de grande échelle - Visage'
titleSuffix: Azure Cognitive Services
description: Ce guide est un article sur la montée en puissance des objets PersonGroup et FaceList vers des objets LargePersonGroup et LargeFaceList.
services: cognitive-services
author: SteveMSFT
manager: nitinme
ms.service: cognitive-services
ms.subservice: face-api
ms.topic: sample
ms.date: 05/01/2019
ms.author: sbowles
ms.custom: devx-track-csharp
ms.openlocfilehash: b35b66615bd5c577dd73faca77d3ea20468442f8
ms.sourcegitcommit: d76108b476259fe3f5f20a91ed2c237c1577df14
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/29/2020
ms.locfileid: "92913515"
---
# <a name="example-use-the-large-scale-feature"></a>Exemple : Utilisez la fonctionnalité à grande échelle

Ce guide est un article avancé sur la montée en puissance des objets PersonGroup et FaceList vers les objets LargePersonGroup et LargeFaceList, respectivement. Ce guide décrit le processus de migration. Il requiert des connaissances de base sur les objets PersonGroup et FaceList, l'opération [Former](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/599ae2d16ac60f11b48b5aa4) et les fonctions de reconnaissance faciale. Pour en savoir plus sur ces sujets, consultez le guide conceptuel [Reconnaissance faciale](../concepts/face-recognition.md).

Les objets LargePersonGroup et LargeFaceList sont collectivement désignés sous le nom d'« opérations à grande échelle ». L'objet LargePersonGroup peut contenir 1 million de personnes, avec un maximum de 248 visages par personne. L'objet LargeFaceList peut contenir 1 million de visages. Les opérations à grande échelle sont semblables aux objets PersonGroup et FaceList classiques, mais présentent quelques différences liées à la nouvelle architecture. 

Les exemples sont écrits en C# à l’aide de la bibliothèque de client Visage d’Azure Cognitive Services.

> [!NOTE]
> Pour activer les performances de recherche de visage dans le cadre des opérations Identification et FindSimilar à grande échelle, introduisez une opération Former afin de prétraiter les objets LargeFaceList et LargePersonGroup. La durée de la formation varie de quelques secondes à environ une demi-heure en fonction de la capacité réelle. Pendant la formation, il est possible d'exécuter des opérations Identification et FindSimilar si une formation a déjà eu lieu au préalable. L'inconvénient est que les nouvelles personnes et les nouveaux visages ajoutés n'apparaîtront pas dans le résultat tant qu'une nouvelle formation post-migration à grande échelle n'aura pas été effectuée.

## <a name="step-1-initialize-the-client-object"></a>Étape 1 : Initialiser l’objet client

Lorsque vous utilisez la bibliothèque de client Visage, la clé d’abonnement et le point de terminaison de l’abonnement sont transmis par le biais du constructeur de la classe FaceClient. Par exemple :

```csharp
string SubscriptionKey = "<Subscription Key>";
// Use your own subscription endpoint corresponding to the subscription key.
string SubscriptionEndpoint = "https://westus.api.cognitive.microsoft.com";
private readonly IFaceClient faceClient = new FaceClient(
            new ApiKeyServiceClientCredentials(subscriptionKey),
            new System.Net.Http.DelegatingHandler[] { });
faceClient.Endpoint = SubscriptionEndpoint
```

Pour obtenir la clé d'abonnement et le point de terminaison correspondant, accédez à la Place de marché Azure à partir du portail Azure.
Pour plus d'informations, consultez [Abonnements](https://azure.microsoft.com/services/cognitive-services/directory/vision/).

## <a name="step-2-code-migration"></a>Étape 2 : Migration de code

Cette section porte sur la migration d'une implémentation PersonGroup ou FaceList vers une implémentation LargePersonGroup ou LargeFaceList. Bien que les objets LargePersonGroup et LargeFaceList diffèrent des objets PersonGroup et FaceList en termes de conception et d'implémentation interne, les interfaces API sont similaires pour faciliter la compatibilité descendante.

La migration des données n'est pas prise en charge. Vous devez recréer l'objet LargePersonGroup ou LargeFaceList.

### <a name="migrate-a-persongroup-to-a-largepersongroup"></a>Migrer un objet PersonGroup vers un objet LargePersonGroup

La migration d'un objet PersonGroup vers un objet LargePersonGroup est simple. Au niveau groupe, ceux-ci partagent exactement les mêmes opérations.

Pour une implémentation PersonGroup ou Person, seuls le chemin de l'API ou la classe/le module du kit de développement logiciel (SDK) doivent être modifiés lors de la migration vers les objets LargePersonGroup et LargePersonGroup Person.

Ajoutez tous les visages et toutes les personnes de l'objet PersonGroup dans le nouvel objet LargePersonGroup. Pour plus d'informations, consultez [Ajouter des visages](how-to-add-faces.md).

### <a name="migrate-a-facelist-to-a-largefacelist"></a>Migrer un objet FaceList vers un objet LargeFaceList

| API FaceList | API LargeFaceList |
|:---:|:---:|
| Créer | Créer |
| DELETE | DELETE |
| Obtenir | Obtenir |
| List | List |
| Update | Update |
| - | Former |
| - | Obtenir l’état de l’entraînement |

Le tableau précédent est une comparaison des opérations au niveau de la liste entre FaceList et LargeFaceList. Comme nous pouvons l'observer, LargeFaceList comporte de nouvelles opérations, Former et Accéder à l'état de la formation, par rapport à FaceList. La formation de l'objet LargeFaceList est une condition préalable à l'opération [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237). Aucune formation n'est requise pour l'objet FaceList. L'extrait de code suivant est une fonction d'assistance pour attendre la formation d'un objet LargeFaceList :

```csharp
/// <summary>
/// Helper function to train LargeFaceList and wait for finish.
/// </summary>
/// <remarks>
/// The time interval can be adjusted considering the following factors:
/// - The training time which depends on the capacity of the LargeFaceList.
/// - The acceptable latency for getting the training status.
/// - The call frequency and cost.
///
/// Estimated training time for LargeFaceList in different scale:
/// -     1,000 faces cost about  1 to  2 seconds.
/// -    10,000 faces cost about  5 to 10 seconds.
/// -   100,000 faces cost about  1 to  2 minutes.
/// - 1,000,000 faces cost about 10 to 30 minutes.
/// </remarks>
/// <param name="largeFaceListId">The Id of the LargeFaceList for training.</param>
/// <param name="timeIntervalInMilliseconds">The time interval for getting training status in milliseconds.</param>
/// <returns>A task of waiting for LargeFaceList training finish.</returns>
private static async Task TrainLargeFaceList(
    string largeFaceListId,
    int timeIntervalInMilliseconds = 1000)
{
    // Trigger a train call.
    await FaceClient.LargeTrainLargeFaceListAsync(largeFaceListId);

    // Wait for training finish.
    while (true)
    {
        Task.Delay(timeIntervalInMilliseconds).Wait();
        var status = await faceClient.LargeFaceList.TrainAsync(largeFaceListId);

        if (status.Status == Status.Running)
        {
            continue;
        }
        else if (status.Status == Status.Succeeded)
        {
            break;
        }
        else
        {
            throw new Exception("The train operation is failed!");
        }
    }
}
```

Auparavant, une utilisation standard de l'objet FaceList, avec ajout de visages et FindSimilar, se présentait comme suit :

```csharp
// Create a FaceList.
const string FaceListId = "myfacelistid_001";
const string FaceListName = "MyFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
faceClient.FaceList.CreateAsync(FaceListId, FaceListName).Wait();

// Add Faces to the FaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await faceClient.FaceList.AddFaceFromStreamAsync(FaceListId, stream);
            }
        });

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = faceClient.Face.DetectWithStreamAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await faceClient.Face.FindSimilarAsync(face.FaceId, FaceListId, 20));
    }
}
```

Lorsque vous le migrez vers l'objet LargeFaceList, le code devient :

```csharp
// Create a LargeFaceList.
const string LargeFaceListId = "mylargefacelistid_001";
const string LargeFaceListName = "MyLargeFaceListDisplayName";
const string ImageDir = @"/path/to/FaceList/images";
faceClient.LargeFaceList.CreateAsync(LargeFaceListId, LargeFaceListName).Wait();

// Add Faces to the LargeFaceList.
Parallel.ForEach(
    Directory.GetFiles(ImageDir, "*.jpg"),
    async imagePath =>
        {
            using (Stream stream = File.OpenRead(imagePath))
            {
                await faceClient.LargeFaceList.AddFaceFromStreamAsync(LargeFaceListId, stream);
            }
        });

// Train() is newly added operation for LargeFaceList.
// Must call it before FindSimilarAsync() to ensure the newly added faces searchable.
await TrainLargeFaceList(LargeFaceListId);

// Perform FindSimilar.
const string QueryImagePath = @"/path/to/query/image";
var results = new List<SimilarPersistedFace[]>();
using (Stream stream = File.OpenRead(QueryImagePath))
{
    var faces = faceClient.Face.DetectWithStreamAsync(stream).Result;
    foreach (var face in faces)
    {
        results.Add(await faceClient.Face.FindSimilarAsync(face.FaceId, largeFaceListId: LargeFaceListId));
    }
}
```

Comme illustré précédemment, la gestion des données et la partie FindSimilar sont quasiment identiques. La seule exception est qu'une nouvelle opération Former de prétraitement doit être exécutée dans l'objet LargeFaceList avant que l'opération FindSimilar puisse fonctionner.

## <a name="step-3-train-suggestions"></a>Étape 3 : Suggestions d’entraînement

Bien que la formation accélère les opérations [FindSimilar](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395237) et [Identification](https://westus.dev.cognitive.microsoft.com/docs/services/563879b61984550e40cbbe8d/operations/563879b61984550f30395239), la durée de celle-ci augmente, en particulier à grande échelle. Le tableau suivant fournit une estimation de la durée de formation à différentes échelles.

| Échelle - Visages ou personnes | Estimation de la durée de formation |
|:---:|:---:|
| 1 000 | 1 à 2 secondes |
| 10 000 | 5 à 10 secondes |
| 100 000 | 1 à 2 minutes |
| 1 000 000 | 10 à 30 minutes |

Pour tirer le meilleur parti de la fonctionnalité de grande échelle, nous recommandons les stratégies suivantes.

### <a name="step-31-customize-time-interval"></a>Étape 3.1 : Personnaliser l’intervalle de temps

Comme indiqué dans `TrainLargeFaceList()`, un intervalle (exprimé en millisecondes) permet de retarder le processus infini de vérification de l'état de la formation. Pour LargeFaceList avec davantage de visages, l’utilisation d’un intervalle plus élevé permet de réduire le nombre et le coût des appels. Personnalisez l'intervalle en fonction de la capacité attendue de l'objet LargeFaceList.

La même stratégie s’applique également à LargePersonGroup. Par exemple, lorsque vous formez un objet LargePersonGroup contenant 1 million de personnes, `timeIntervalInMilliseconds` peut s'élever à 60 000, ce qui correspond à un intervalle d'une minute.

### <a name="step-32-small-scale-buffer"></a>Étape 3.2 : Mémoire tampon à petite échelle

Il est possible d'effectuer des recherches de personnes ou de visages dans un objet LargePersonGroup ou LargeFaceList à condition de l'avoir formé au préalable. Dans un scénario dynamique, de nouvelles personnes et de nouveaux visages sont constamment ajoutés et ceux-ci doivent être immédiatement interrogeables, mais la formation peut être plus longue qu'on ne le souhaiterait. 

Pour atténuer ce problème, utilisez un objet LargePersonGroup ou LargeFaceList à petite échelle en guise de mémoire tampon pour les entrées nouvellement ajoutées. La formation de cette mémoire tampon prend moins de temps en raison de sa petite taille. La recherche immédiate sur cette mémoire tampon temporaire doit fonctionner. Utilisez cette mémoire tampon en combinaison avec une formation de l'objet LargePersonGroup ou LargeFaceList principal en exécutant la formation principale sur un intervalle plus court. Par exemple, au milieu de la nuit ou quotidiennement.

Exemple de flux de travail :

1. Créez un objet LargePersonGroup ou LargeFaceList principal correspondant à la collection principale. Créez une mémoire tampon LargePersonGroup ou LargeFaceList correspondant à la collection de mémoire tampon. La collection de mémoire tampon ne concerne que les personnes ou visages nouvellement ajoutés.
1. Ajoutez de nouvelles personnes ou de nouveaux visages à la collection principale et à la collection de mémoire tampon.
1. Formez la collection de mémoire tampon avec un intervalle court afin de garantir la prise en compte des nouvelles entrées.
1. Appelez l'opération Identification ou FindSimilar sur la collection principale et sur la collection de mémoire tampon. Fusionnez les résultats.
1. Lorsque la taille de la collection de mémoire tampon atteint un seuil ou lors d'une période d'inactivité du système, créez une nouvelle collection de mémoire tampon. Déclenchez l'opération Former sur la collection principale.
1. Supprimez l'ancienne collection de mémoire tampon à l'issue de l'opération Former exécutée sur la collection principale.

### <a name="step-33-standalone-training"></a>Étape 3.3 : Formation autonome

Si une latence relativement longue est acceptable, il n'est pas nécessaire de déclencher l'opération Former juste après l'ajout de nouvelles données. Au lieu de cela, elle peut être séparée de la logique principale et déclenchée régulièrement. Cette stratégie convient aux scénarios dynamiques présentant une latence acceptable. Elle peut être appliquée à des scénarios statiques pour réduire la fréquence de formation.

Supposons qu'il existe une fonction `TrainLargePersonGroup` semblable à `TrainLargeFaceList`. Voici un exemple d'implémentation standard de la formation autonome sur un objet LargePersonGroup en appelant la classe [`Timer`](/dotnet/api/system.timers.timer) dans `System.Timers` :

```csharp
private static void Main()
{
    // Create a LargePersonGroup.
    const string LargePersonGroupId = "mylargepersongroupid_001";
    const string LargePersonGroupName = "MyLargePersonGroupDisplayName";
    faceClient.LargePersonGroup.CreateAsync(LargePersonGroupId, LargePersonGroupName).Wait();

    // Set up standalone training at regular intervals.
    const int TimeIntervalForStatus = 1000 * 60; // 1-minute interval for getting training status.
    const double TimeIntervalForTrain = 1000 * 60 * 60; // 1-hour interval for training.
    var trainTimer = new Timer(TimeIntervalForTrain);
    trainTimer.Elapsed += (sender, args) => TrainTimerOnElapsed(LargePersonGroupId, TimeIntervalForStatus);
    trainTimer.AutoReset = true;
    trainTimer.Enabled = true;

    // Other operations like creating persons, adding faces, and identification, except for Train.
    // ...
}

private static void TrainTimerOnElapsed(string largePersonGroupId, int timeIntervalInMilliseconds)
{
    TrainLargePersonGroup(largePersonGroupId, timeIntervalInMilliseconds).Wait();
}
```

Pour plus d’informations sur la gestion des données et les implémentations relatives à l’identification, consultez [Ajouter des visages](how-to-add-faces.md).

## <a name="summary"></a>Résumé

Dans ce guide, vous avez appris à migrer le code PersonGroup ou FaceList existant (et non les données) vers un objet LargePersonGroup ou LargeFaceList :

- Les objets LargePersonGroup et LargeFaceList fonctionnent comme les objets PersonGroup et FaceList, sauf qu'une opération Former est requise par LargeFaceList.
- Adoptez la stratégie de formation appropriée pour la mise à jour dynamique des données des jeux de données à grande échelle.

## <a name="next-steps"></a>Étapes suivantes

Suivez un guide pratique pour savoir comment ajouter des visages à un groupe PersonGroup ou écrire un script pour exécuter l’opération d’identification sur un groupe PersonGroup.

- [Ajouter des visages](how-to-add-faces.md)
- [Démarrage rapide : Bibliothèque de client Visage](../Quickstarts/client-libraries.md)
---
title: Requêtes de performances côté serveur
description: Comment effectuer des requêtes de performances côté serveur via des appels d’API
author: florianborn71
ms.author: flborn
ms.date: 02/10/2020
ms.topic: article
ms.custom: devx-track-csharp
ms.openlocfilehash: 95993b35174b80dae8c878c22554ee60afeb8a14
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92206218"
---
# <a name="server-side-performance-queries"></a>Requêtes de performances côté serveur

De bonnes performances de rendu sur le serveur sont capitales pour obtenir des fréquences d’images stables et une expérience utilisateur satisfaisante. Il est important de superviser soigneusement les caractéristiques des performances sur le serveur, et de les optimiser là où c’est nécessaire. Les données de performances peuvent être interrogées par le biais de fonctions API dédiées.

La plus grande incidence observée sur les performances de rendu vient des données d’entrée des modèles. Vous pouvez ajuster les données d’entrée, comme indiqué dans [Configurer la conversion de modèle](../../how-tos/conversion/configure-model-conversion.md).

Les performances de l’application côté client peuvent également constituer un goulot d’étranglement. Pour une analyse approfondie des performances côté client, il est recommandé d’effectuer une [:::no-loc text="performance trace":::](../../how-tos/performance-tracing.md).

## <a name="clientserver-timeline"></a>Chronologie client/serveur

Avant d’entrer dans les détails des différentes valeurs de latence, il est utile d’examiner les points de synchronisation entre le client et le serveur sur la chronologie :

![Chronologie du pipeline](./media/server-client-timeline.png)

L’illustration montre comment :

* une *Estimation de pose* est lancée par le client à une fréquence d’images constante de 60 Hz (toutes les 16,6 ms) ;
* le serveur démarre ensuite le rendu, en fonction de la pose ;
* le serveur retourne l’image vidéo encodée ;
* le client décode l’image, effectue en sus des opérations d’UC et de GPU, puis affiche l’image.

## <a name="frame-statistics-queries"></a>Requêtes sur les statistiques d’images

Les statistiques d’images offrent quelques informations générales pour la dernière image, telles que la latence. Les données fournies dans la structure `FrameStatistics` sont mesurées côté client, l’API est donc un appel synchrone :

```cs
void QueryFrameData(AzureSession session)
{
    FrameStatistics frameStatistics;
    if (session.GraphicsBinding.GetLastFrameStatistics(out frameStatistics) == Result.Success)
    {
        // do something with the result
    }
}
```

```cpp
void QueryFrameData(ApiHandle<AzureSession> session)
{
    FrameStatistics frameStatistics;
    if (*session->GetGraphicsBinding()->GetLastFrameStatistics(&frameStatistics) == Result::Success)
    {
        // do something with the result
    }
}
```

L’objet `FrameStatistics` récupéré contient les membres suivants :

| Membre | Explication |
|:-|:-|
| latencyPoseToReceive | Latence provenant de la pose estimée de la caméra sur l’appareil client, jusqu’à ce qu’une image serveur de cette pose soit entièrement disponible pour l’application cliente. Cette valeur comprend l’aller-retour réseau, le temps de rendu du serveur, le décodage vidéo et la compensation de l’instabilité. Voir **l’intervalle 1 dans l’illustration ci-dessus.**|
| latencyReceiveToPresent | Latence provenant de la disponibilité d’une image distante reçue jusqu’à ce que l’application cliente appelle PresentFrame sur l’UC. |
| latencyPresentToDisplay  | Latence provenant de la présentation d’une image sur l’UC jusqu’à ce que l’affichage s’allume. Cette valeur comprend le temps du GPU client, une mise en mémoire tampon d’image effectuée par le système d’exploitation, la reprojection matérielle et la durée de balayage de l’affichage dépendant de l’appareil. Voir **l’intervalle 2 dans l’illustration ci-dessus.**|
| timeSinceLastPresent | Délai entre les appels ultérieurs à PresentFrame sur le processeur. Les valeurs supérieures à la durée d’affichage (par exemple 16,6 ms sur un appareil client de 60 Hz) indiquent des problèmes provoqués par l’application cliente qui ne termine pas sa charge de travail processeur à temps. Voir **l’intervalle 3 dans l’illustration ci-dessus.**|
| videoFramesReceived | Nombre d’images reçues du serveur au cours de la dernière seconde. |
| videoFrameReusedCount | Nombre d’images reçues au cours de la dernière seconde et qui ont été utilisées plusieurs fois sur l’appareil. Les valeurs différentes de zéro indiquent que les images devaient être réutilisées et reprojetées en raison d’une instabilité du réseau ou d’une durée d’affichage du serveur excessive. |
| videoFramesSkipped | Nombre d’images reçues au cours de la dernière seconde, qui ont été décodées, mais n’ont pas été affichées à l’écran, car une image plus récente est arrivée. Des valeurs différentes de zéro indiquent que l’instabilité du réseau a entraîné le retard de plusieurs images, puis leur arrivée simultanée en rafale sur l’appareil client. |
| videoFramesDiscarded | Très similaire à **videoFramesSkipped**, mais la raison de l’abandon provient du fait qu’une image a été tellement retardée qu’elle ne peut même plus être mise en corrélation avec une pose en attente. Si cela se produit, il y a une grave contention de réseau.|
| videoFrameMinDelta | Intervalle de temps minimal entre deux images consécutives arrivant au cours de la dernière seconde. Avec videoFrameMaxDelta, cette plage donne une indication de la gigue provoquée par le codec vidéo ou réseau. |
| videoFrameMaxDelta | Temps maximal entre deux images consécutives arrivant au cours de la dernière seconde. Avec videoFrameMinDelta, cette plage donne une indication de la gigue provoquée par le codec vidéo ou réseau. |

La somme de toutes les valeurs de latence est en général nettement supérieure à la durée de l’image disponible à 60 Hz. Cela est normal, car plusieurs images sont actives en parallèle, et les nouvelles demandes d’images sont déclenchées à la fréquence d’images souhaitée, comme indiqué dans l’illustration. Toutefois, si la latence devient trop importante, la qualité de la [reprojection en phase tardive](../../overview/features/late-stage-reprojection.md) en est affectée, et l’expérience globale peut être compromise.

`videoFramesReceived`, `videoFrameReusedCount` et `videoFramesDiscarded` peuvent être utilisés pour évaluer les performances du réseau et du serveur. Si `videoFramesReceived` est faible et `videoFrameReusedCount` est élevé, cela peut indiquer une congestion du réseau ou une baisse des performances du serveur. Une valeur élevée de `videoFramesDiscarded` indique également une congestion du réseau.

Enfin, `timeSinceLastPresent`, `videoFrameMinDelta` et `videoFrameMaxDelta` vous donnent une idée de la variance des trames vidéo entrantes et des appels présents locaux. Une variance élevée signifie une fréquence d’images instable.

Aucune des valeurs ci-dessus ne donne une indication claire de la latence pure du réseau (les flèches rouges dans l’illustration), car la durée exacte pendant laquelle le serveur est occupé à la restitution doit être soustraite de la valeur aller-retour `latencyPoseToReceive`. La partie côté serveur de la latence globale représente des informations qui ne sont pas disponibles pour le client. Toutefois, le paragraphe suivant explique comment cette valeur est évaluée par le biais d’une entrée supplémentaire depuis le serveur, et exposée au moyen de la valeur `networkLatency`.

## <a name="performance-assessment-queries"></a>Requêtes d’évaluation des performances

*Les requêtes d’évaluation des performances* fournissent des informations plus approfondies sur la charge de travail du processeur et du GPU sur le serveur. Du fait que les données sont demandées à partir du serveur, l’interrogation d’un instantané de performances suit le modèle asynchrone habituel :

```cs
PerformanceAssessmentAsync _assessmentQuery = null;

void QueryPerformanceAssessment(AzureSession session)
{
    _assessmentQuery = session.Actions.QueryServerPerformanceAssessmentAsync();
    _assessmentQuery.Completed += (PerformanceAssessmentAsync res) =>
    {
        // do something with the result:
        PerformanceAssessment result = res.Result;
        // ...

        _assessmentQuery = null;
    };
}
```

```cpp
void QueryPerformanceAssessment(ApiHandle<AzureSession> session)
{
    ApiHandle<PerformanceAssessmentAsync> assessmentQuery = *session->Actions()->QueryServerPerformanceAssessmentAsync();
    assessmentQuery->Completed([] (ApiHandle<PerformanceAssessmentAsync> res)
    {
        // do something with the result:
        PerformanceAssessment result = res->GetResult();

        // ...

    });
}
```

Contrairement à l’objet `FrameStatistics`, l’objet `PerformanceAssessment` contient des informations côté serveur :

| Membre | Explication |
|:-|:-|
| timeCPU | Temps moyen UC du serveur par image en millisecondes |
| timeGPU | Temps moyen GPU du serveur par image en millisecondes |
| utilizationCPU | Utilisation totale UC du serveur en pourcentage |
| utilizationGPU | Utilisation totale GPU du serveur en pourcentage |
| memoryCPU | Utilisation totale de la mémoire principale du serveur en pourcentage |
| memoryGPU | Utilisation totale de la mémoire vidéo dédiée en pourcentage du GPU du serveur |
| networkLatency | Moyenne approximative de la latence aller-retour du réseau en millisecondes. Dans l’illustration ci-dessus, cela correspond à la somme des flèches rouges. La valeur est calculée en soustrayant le temps de rendu réel du serveur, de la valeur `latencyPoseToReceive` de `FrameStatistics`. Même si cette approximation n’est pas précise, elle donne quelque indication du temps de réponse du réseau, isolé des valeurs de latence calculées sur le client. |
| polygonsRendered | Nombre de triangles restitués dans une image. Ce nombre comprend également les triangles qui sont supprimés plus tard pendant l’affichage. Cela signifie que ce nombre ne varie pas beaucoup entre les différentes positions de la caméra, mais que les performances peuvent varier considérablement, selon le taux d’élimination des triangles.|

Pour vous aider à évaluer les valeurs, chaque partie est accompagnée d’une classification de qualité telle que **Excellente**, **Bonne**, **Médiocre** ou **Mauvaise**.
Cette mesure d’évaluation fournit une indication approximative de l’état d’intégrité du serveur ; elle ne doit pas être considérée comme absolue. Par exemple, supposons que vous voyez un score « médiocre » pour le temps du GPU. Il est considéré comme médiocre, car il est proche de la limite du budget global de la durée de l’image. Toutefois, dans votre cas, il peut s’agir d’une bonne valeur, car vous affichez un modèle complexe.

## <a name="statistics-debug-output"></a>Sortie de débogage des statistiques

La classe `ARRServiceStats` est une classe C# qui inclut dans un wrapper les statistiques des images et les requêtes d’évaluation des performances ; elle fournit des fonctionnalités pratiques pour retourner des statistiques sous forme de valeurs agrégées ou comme chaîne prédéfinie. Le code suivant est le moyen le plus simple pour afficher des statistiques côté serveur dans votre application cliente.

```cs
ARRServiceStats _stats = null;

void OnConnect()
{
    _stats = new ARRServiceStats();
}

void OnDisconnect()
{
    _stats = null;
}

void Update()
{
    if (_stats != null)
    {
        // update once a frame to retrieve new information and build average values
        _stats.Update(Service.CurrentActiveSession);

        // retrieve a string with relevant stats information
        InfoLabel.text = _stats.GetStatsString();
    }
}
```

Le code ci-dessus remplit l’étiquette de texte avec le texte suivant :

![Sortie de chaîne ArrServiceStats](./media/arr-service-stats.png)

L’API `GetStatsString` forme une chaîne de toutes les valeurs, mais chaque valeur en soi peut également être interrogée par programme à partir de l’instance `ARRServiceStats`.

Il existe également des variantes des membres, qui agrègent les valeurs au fil du temps. Consultez les membres avec le suffixe `*Avg`, `*Max` ou `*Total`. Le membre `FramesUsedForAverage` indique le nombre d’images qui a été utilisé pour cette agrégation.

## <a name="api-documentation"></a>Documentation de l’API

* [C# RemoteManager.QueryServerPerformanceAssessmentAsync()](/dotnet/api/microsoft.azure.remoterendering.remotemanager.queryserverperformanceassessmentasync)
* [C++ RemoteManager::QueryServerPerformanceAssessmentAsync()](/cpp/api/remote-rendering/remotemanager#queryserverperformanceassessmentasync)

## <a name="next-steps"></a>Étapes suivantes

* [Créer des traces de performances](../../how-tos/performance-tracing.md)
* [Configurer la conversion de modèle](../../how-tos/conversion/configure-model-conversion.md)
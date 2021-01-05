---
title: Charger et indexer des vidéos avec Video Indexer
titleSuffix: Azure Media Services
description: Cette rubrique montre comment utiliser les API pour charger et indexer vos vidéos avec Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 11/12/2020
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: a0b7330485d3152a588d43added7d9feaa5c2a14
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "95994488"
---
# <a name="upload-and-index-your-videos"></a>Charger et indexer vos vidéos  

Une fois votre vidéo téléchargée, Video Indexer peut éventuellement l’encoder (opération décrite dans cet article). Lorsque vous créez un compte Video Indexer, vous pouvez choisir un compte d’essai gratuit (où vous obtenez un certain nombre de minutes d’indexation gratuites) ou une option payante (où vous n’êtes pas limités par le quota). Avec l’essai gratuit, Video Indexer fournit jusqu’à 600 heures d’indexation gratuite aux utilisateurs du site web et jusqu’à 2 400 heures d’indexation gratuite aux utilisateurs de l’API. Avec l’option payante, vous créez un compte Video Indexer [connecté à votre abonnement Azure et un compte Azure Media Services](connect-to-azure.md). Vous payez pour les minutes indexées ; pour plus d’informations, consultez [Tarification Media Services](https://azure.microsoft.com/pricing/details/media-services/).

Lorsque vous chargez des vidéos avec l’API Video Indexer, vous disposez des deux options de chargement suivantes : 

* charger votre vidéo à partir d’une URL (par défaut),
* Envoyer le fichier vidéo sous forme de tableau d’octets dans le corps de la demande,
* Utilisez la ressource Azure Media Services existante en fournissant l’[ID de la ressource](../latest/assets-concept.md) (pris en charge dans les comptes payants uniquement).

L’article décrit comment télécharger et indexer vos vidéos avec les options suivantes :

* [Le site Web Video Indexer](#upload-and-index-a-video-using-the-video-indexer-website) 
* [Les API Video Indexer](#upload-and-index-with-api)

## <a name="supported-file-formats-for-video-indexer"></a>Formats de fichiers pris en charge pour Video Indexer

Pour obtenir la liste des formats de fichiers que vous pouvez utiliser avec Video Indexer, consultez l’article [conteneur d’entrée/formats de fichiers](../latest/media-encoder-standard-formats.md#input-containerfile-formats).

## <a name="video-files-storage"></a>Stockage de fichiers vidéo

- Avec un compte Video Indexer payant, vous créez un compte Video Indexer connecté à votre abonnement Azure et un compte Azure Media Services. Pour plus d’informations, consultez [Créer un compte Video Indexer connecté à Azure](connect-to-azure.md).
- Les fichiers vidéo sont stockés dans le stockage Azure par Azure Media Services. Il n’existe aucune limite de temps.
- Vous pouvez toujours supprimer vos fichiers vidéo et audio, ainsi que les métadonnées et les insights qui ont été extraits par Video Indexer. Lorsque vous supprimez un fichier à partir de Video Indexer, le fichier, ses métadonnées et ses insights sont définitivement supprimés de Video Indexer. Toutefois, si vous avez implémenté votre propre solution de sauvegarde dans le stockage Azure, le fichier demeure dans votre stockage Azure.
- La persistance d’une vidéo est identique, que le chargement soit effectué à partir du site web de Video Indexer ou à l’aide de l’API de chargement.
   
## <a name="upload-and-index-a-video-using-the-video-indexer-website"></a>Télécharger et indexer une vidéo à l’aide du site web Video Indexer

> [!NOTE]
> Un nom de vidéo ne doit pas dépasser 80 caractères.

1. Connectez-vous au site web [Video Indexer](https://www.videoindexer.ai/).
1. Pour charger une vidéo, appuyez sur le bouton ou le lien **Charger**.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/video-indexer-upload.png" alt-text="Charger":::
1. Une fois votre vidéo chargée, Video Indexer démarre l’indexation et l’analyse.

    > [!div class="mx-imgBorder"]
    > :::image type="content" source="./media/video-indexer-get-started/progress.png" alt-text="Progression du chargement":::
1. Au terme de l'analyse réalisée par Video Indexer, vous recevrez un e-mail contenant un lien vers votre vidéo et une brève description du contenu de celle-ci. Par exemple : personnes, rubriques, OCR.

## <a name="upload-and-index-with-api"></a>Télécharger et indexer avec API

Utilisez l’API [Télécharger une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) pour télécharger et indexer vos vidéos en fonction d’une URL. L’exemple de code dans l’article inclut le code commenté qui montre comment charger le tableau d’octets. 

### <a name="configurations-and-params"></a>Configurations et paramètres

Cette section décrit certains des paramètres facultatifs et les moments où vous souhaitez les définir. Pour obtenir les informations les plus récentes sur les paramètres, consultez l’API [Télécharger une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?).

#### <a name="externalid"></a>externalID 

Ce paramètre vous permet de spécifier un ID qui sera associé à la vidéo. L’ID peut être appliqué à l’intégration de système externe « Gestion de contenu vidéo» (VCM). Les vidéos situées dans le portail Video Indexer peuvent faire l’objet d’une recherche à l’aide de l’ID externe spécifié.

#### <a name="callbackurl"></a>callbackUrl

[!INCLUDE [callback url](./includes/callback-url.md)]

##### <a name="other-considerations"></a>Autres considérations

- Video Indexer retourne tous les paramètres existants fournis dans l’URL d’origine.
- L’URL fournie doit être encodée.

#### <a name="indexingpreset"></a>indexingPreset

Utilisez ce paramètre si des enregistrements bruts ou externes contiennent un bruit de fond. Ce paramètre est utilisé pour configurer le processus d’indexation. Vous pouvez spécifier les valeurs suivantes :

- `AudioOnly` – Permet d’indexer et d’extraire des insights de l’audio uniquement (la vidéo est ignorée)
- `VideoOnly` – Permet d’indexer et d’extraire des insights de la vidéo uniquement (l’audio est ignoré)
- `Default` – Permet d’indexer et d’extraire des insights de l’audio et de la vidéo
- `DefaultWithNoiseReduction` – Permet d’indexer et d’extraire des insights de l’audio et vidéo, tout en appliquant des algorithmes de réduction du bruit sur les flux audio

> [!NOTE]
> Video Indexer prend en charge jusqu’à deux pistes audio. Si le fichier contient plus de pistes audio, elles sont traitées comme une seule piste.<br/>
Si vous souhaitez indexer les pistes séparément, vous devez extraire le fichier audio approprié et l’indexer comme `AudioOnly`.

Le prix dépend de l’option d’indexation sélectionnée.  

#### <a name="priority"></a>priority

Les vidéos sont indexées par Video Indexer selon leur priorité. Utilisez le paramètre **priority** pour spécifier la priorité d’index. Les valeurs suivantes sont valides : **Basse**, **Normale** (valeur par défaut) et **Haute**.

Le paramètre **priority** est uniquement pris en charge dans les comptes payants.

#### <a name="streamingpreset"></a>streamingPreset

Une fois votre vidéo chargée, Video Indexer peut éventuellement l’encoder. Il passe ensuite à l’indexation et à l’analyse de la vidéo. Lorsque Video Indexer a terminé l’analyse, vous recevrez une notification avec l’ID de la vidéo.  

Lorsque vous utilisez l’API [Charger une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Upload-video?) ou [Réindexer une vidéo](https://api-portal.videoindexer.ai/docs/services/operations/operations/Re-index-video?), un des paramètres facultatifs est `streamingPreset`. Si vous définissez `streamingPreset` sur `Default`, `SingleBitrate`, ou `AdaptiveBitrate`, le processus d’encodage est déclenché. Une fois les travaux d’indexation et d’encodage effectués, la vidéo est publiée et vous pouvez donc la diffuser. Le point de terminaison de streaming à partir duquel vous souhaitez diffuser la vidéo doit se trouver dans l’état **En cours d’exécution**.

Pour SingleBitrate, le coût de l’encodeur standard s’applique à chaque sortie. Si la hauteur de la vidéo est supérieure ou égale à 720, Video Indexer l’encode au format 1280 x 720. Sinon, au format 640 x 468.
Le paramètre par défaut est [l’encodage sensible au contenu](../latest/content-aware-encoding.md).

Pour exécuter les travaux d’indexation et d’encodage, le [compte Azure Media Services connecté à votre compte Video Indexer](connect-to-azure.md), nécessite des unités réservées. Pour plus d’informations, consultez [Mise à l’échelle du traitement multimédia](../previous/media-services-scale-media-processing-overview.md). Dans la mesure où il s’agit de tâches de calcul intensif, un type d’unité S3 est fortement recommandé. Le nombre d’unités de demande définit le nombre maximal de travaux pouvant s’exécuter en parallèle. La suggestion de base de référence est 10 unités de demande S3. 

Si vous souhaitez uniquement indexer votre vidéo sans l’encoder, définissez `streamingPreset` sur `NoStreaming`.

#### <a name="videourl"></a>videoUrl

URL du fichier audio/vidéo à indexer. L’URL doit pointer vers un fichier multimédia (les pages HTML ne sont pas prises en charge). Le fichier peut être protégé par un jeton d’accès fourni dans le cadre de l’URI et le point de terminaison qui traite le fichier doit être sécurisé avec TLS 1.2 ou version ultérieure. L’URL doit être encodée. 

Si le paramètre `videoUrl` n’est pas spécifié, Video Indexer s’attend à ce que vous passiez le fichier en tant que contenu de corps multipart/form.

### <a name="code-sample"></a>Exemple de code

L’extrait de code C# suivant illustre l’utilisation conjointe de toutes les API Video Indexer.

**Instructions pour l’exécution de l’exemple de code suivant**

Après avoir copié ce code dans votre plateforme de développement, vous devez fournir deux paramètres : URL de la vidéo et clé d’authentification de Gestion des API.

* Clé API : la clé API est votre clé d’abonnement de gestion des API personnelle, qui vous permet d’obtenir un jeton d’accès afin d’effectuer des opérations sur votre compte Video Indexer. 

    Pour obtenir votre clé API, effectuez les étapes suivantes :

    * Accédez à https://api-portal.videoindexer.ai/.
    * Connexion
    * Accédez à **Products** -> **Authorization** -> **Authorization subscription**.
    * Copiez la **clé primaire**.
* URL de la vidéo : URL du fichier audio/vidéo à indexer. L’URL doit pointer vers un fichier multimédia (les pages HTML ne sont pas prises en charge). Le fichier peut être protégé par un jeton d’accès fourni dans le cadre de l’URI et le point de terminaison qui traite le fichier doit être sécurisé avec TLS 1.2 ou version ultérieure. L’URL doit être encodée.

Le résultat de l’exécution de l’exemple de code inclut une URL de widget d’insight et une URL de widget de lecteur qui vous permettra d’examiner respectivement les insights et la vidéo chargée. 


```csharp
public async Task Sample()
{
    var apiUrl = "https://api.videoindexer.ai";
    var apiKey = "..."; // replace with API key taken from https://aka.ms/viapi

    System.Net.ServicePointManager.SecurityProtocol =
        System.Net.ServicePointManager.SecurityProtocol | System.Net.SecurityProtocolType.Tls12;

    // create the http client
    var handler = new HttpClientHandler();
    handler.AllowAutoRedirect = false;
    var client = new HttpClient(handler);
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);

    // obtain account information and access token
    string queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"generateAccessTokens", "true"},
            {"allowEdit", "true"},
        });
    HttpResponseMessage result = await client.GetAsync($"{apiUrl}/auth/trial/Accounts?{queryParams}");
    var json = await result.Content.ReadAsStringAsync();
    var accounts = JsonConvert.DeserializeObject<AccountContractSlim[]>(json);
    
    // take the relevant account, here we simply take the first, 
    // you can also get the account via accounts.First(account => account.Id == <GUID>);
    var accountInfo = accounts.First();

    // we will use the access token from here on, no need for the apim key
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // upload a video
    var content = new MultipartFormDataContent();
    Console.WriteLine("Uploading...");
    // get the video from URL
    var videoUrl = "VIDEO_URL"; // replace with the video URL

    // as an alternative to specifying video URL, you can upload a file.
    // remove the videoUrl parameter from the query params below and add the following lines:
    //FileStream video =File.OpenRead(Globals.VIDEOFILE_PATH);
    //byte[] buffer =new byte[video.Length];
    //video.Read(buffer, 0, buffer.Length);
    //content.Add(new ByteArrayContent(buffer));

    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"name", "video_name"},
            {"description", "video_description"},
            {"privacy", "private"},
            {"partition", "partition"},
            {"videoUrl", videoUrl},
        });
    var uploadRequestResult = await client.PostAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos?{queryParams}", content);
    var uploadResult = await uploadRequestResult.Content.ReadAsStringAsync();

    // get the video ID from the upload result
    string videoId = JsonConvert.DeserializeObject<dynamic>(uploadResult)["id"];
    Console.WriteLine("Uploaded");
    Console.WriteLine("Video ID:");
    Console.WriteLine(videoId);

    // wait for the video index to finish
    while (true)
    {
        await Task.Delay(10000);

        queryParams = CreateQueryString(
            new Dictionary<string, string>()
            {
                {"accessToken", accountInfo.AccessToken},
                {"language", "English"},
            });

        var videoGetIndexRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/Index?{queryParams}");
        var videoGetIndexResult = await videoGetIndexRequestResult.Content.ReadAsStringAsync();

        string processingState = JsonConvert.DeserializeObject<dynamic>(videoGetIndexResult)["state"];

        Console.WriteLine("");
        Console.WriteLine("State:");
        Console.WriteLine(processingState);

        // job is finished
        if (processingState != "Uploaded" && processingState != "Processing")
        {
            Console.WriteLine("");
            Console.WriteLine("Full JSON:");
            Console.WriteLine(videoGetIndexResult);
            break;
        }
    }

    // search for the video
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", accountInfo.AccessToken},
            {"id", videoId},
        });

    var searchRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/Search?{queryParams}");
    var searchResult = await searchRequestResult.Content.ReadAsStringAsync();
    Console.WriteLine("");
    Console.WriteLine("Search:");
    Console.WriteLine(searchResult);

    // Generate video access token (used for get widget calls)
    client.DefaultRequestHeaders.Add("Ocp-Apim-Subscription-Key", apiKey);
    var videoTokenRequestResult = await client.GetAsync($"{apiUrl}/auth/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/AccessToken?allowEdit=true");
    var videoAccessToken = (await videoTokenRequestResult.Content.ReadAsStringAsync()).Replace("\"", "");
    client.DefaultRequestHeaders.Remove("Ocp-Apim-Subscription-Key");

    // get insights widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
            {"widgetType", "Keywords"},
            {"allowEdit", "true"},
        });
    var insightsWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/InsightsWidget?{queryParams}");
    var insightsWidgetLink = insightsWidgetRequestResult.Headers.Location;
    Console.WriteLine("Insights Widget url:");
    Console.WriteLine(insightsWidgetLink);

    // get player widget url
    queryParams = CreateQueryString(
        new Dictionary<string, string>()
        {
            {"accessToken", videoAccessToken},
        });
    var playerWidgetRequestResult = await client.GetAsync($"{apiUrl}/{accountInfo.Location}/Accounts/{accountInfo.Id}/Videos/{videoId}/PlayerWidget?{queryParams}");
    var playerWidgetLink = playerWidgetRequestResult.Headers.Location;
     Console.WriteLine("");
     Console.WriteLine("Player Widget url:");
     Console.WriteLine(playerWidgetLink);
     Console.WriteLine("\nPress Enter to exit...");
     String line = Console.ReadLine();
     if (line == "enter")
     {
         System.Environment.Exit(0);
     }

}

private string CreateQueryString(IDictionary<string, string> parameters)
{
    var queryParameters = HttpUtility.ParseQueryString(string.Empty);
    foreach (var parameter in parameters)
    {
        queryParameters[parameter.Key] = parameter.Value;
    }

    return queryParameters.ToString();
}

public class AccountContractSlim
{
    public Guid Id { get; set; }
    public string Name { get; set; }
    public string Location { get; set; }
    public string AccountType { get; set; }
    public string Url { get; set; }
    public string AccessToken { get; set; }
}
```

### <a name="common-errors"></a>Erreurs courantes

Les codes d’état répertoriés dans le tableau suivant peuvent être renvoyés par l’opération de chargement (Upload).

|Code d’état|ErrorType (dans le corps de la réponse)|Description|
|---|---|---|
|409|VIDEO_INDEXING_IN_PROGRESS|La même vidéo est déjà en cours de traitement dans le compte en question.|
|400|VIDEO_ALREADY_FAILED|La même vidéo n’a pas pu être traitée dans le compte en question moins de 2 heures auparavant. Les clients API doivent attendre au moins 2 heures avant de recharger une vidéo.|
|429||Les comptes de version d’essai sont autorisés à effectuer 5 chargements par minute. Les comptes payants sont autorisés à effectuer 50 chargements par minute.|

## <a name="uploading-considerations-and-limitations"></a>Considérations et limitations relatives au chargement
 
- Un nom de vidéo ne doit pas dépasser 80 caractères.
- Lors du chargement de votre vidéo à partir de l’URL (par défaut), le point de terminaison doit être sécurisé avec TLS 1.2 (ou version ultérieure).
- La taille du chargement avec l’option URL est limitée à 30 Go.
- La longueur de l’URL de la requête est limitée à 6 144 caractères, et la longueur de l’URL de la chaîne de requête est limitée à 4 096 caractères.
- La taille du chargement avec l’option Tableau d’octets est limitée à 2 Go.
- L’option Tableau d’octets expire au bout de 30 minutes.
- L’URL fournie dans le paramètre `videoURL` doit être encodée.
- L’indexation des actifs multimédias Media Services a la même restriction que l’indexation à partir d’une URL.
- Video Indexer a une limite de temps maximale de 4 heures par fichier.
- L’URL doit être accessible (par exemple, une URL publique). 

    S’il s’agit d’une URL privée, le jeton d’accès doit être fourni dans la requête.
- L’URL doit emmener vers un fichier multimédia valide et non vers une page Web, par exemple un lien vers la page de `www.youtube.com`.
- Vous pouvez charger jusqu’à 50 films par minute dans un compte payant et jusqu’à 5 films par minute dans un compte de version d’essai.

> [!Tip]
> Il est recommandé d’utiliser la version 4.6.2 du .NET Framework. ou une version ultérieure, car les anciens .NET Framework ne sont pas définis par défaut sur TLS 1.2.
>
> Si vous devez utiliser une version antérieure du .NET Framework, ajoutez une ligne dans votre code avant d’effectuer l’appel de l’API REST :  <br/> System.Net.ServicePointManager.SecurityProtocol = SecurityProtocolType.Tls | SecurityProtocolType.Tls11 | SecurityProtocolType.Tls12;

## <a name="next-steps"></a>Étapes suivantes

[Examiner la sortie d’Azure Video Indexer générée par l’API](video-indexer-output-json-v2.md)

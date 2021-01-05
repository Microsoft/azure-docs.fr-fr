---
title: Gestion des éléments multimédias et des entités connexes avec le Kit de développement logiciel (SDK) Media Services .NET
description: Apprenez à gérer des éléments multimédias et leurs entités associées avec le Kit de développement logiciel (SDK) pour .NET.
author: juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 1bd8fd42-7306-463d-bfe5-f642802f1906
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 04/01/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: b615e6526eb4187d354c4e276147743156b10bc1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89266008"
---
# <a name="managing-assets-and-related-entities-with-media-services-net-sdk"></a>Gestion des éléments multimédias et des entités connexes avec le Kit de développement logiciel (SDK) Media Services .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

> [!div class="op_single_selector"]
> * [.NET](media-services-dotnet-manage-entities.md)
> * [REST](media-services-rest-manage-entities.md)
> 
> 

> [!NOTE]
> Aucune nouvelle fonctionnalité ni fonction n’est ajoutée à Media Services v2. <br/>Découvrez la dernière version, [Media Services v3](../latest/index.yml). Consultez aussi [Conseils de migration de v2 vers v3](../latest/migrate-from-v2-to-v3.md).

Cette rubrique montre comment gérer les entités Azure Media Services avec .NET.

À compter du 1er avril 2017, les enregistrements de travaux dans votre compte de plus de 90 jours seront automatiquement supprimés, ainsi que leurs enregistrements de tâches associés, même si le nombre total d’enregistrements est inférieur au quota maximum. Par exemple, le 1er avril 2017, tout enregistrement de travail dans votre compte antérieur au 31 décembre 2016 sera automatiquement supprimé. Si vous devez archiver les informations sur le travail/la tâche, vous pouvez utiliser le code décrit dans cette rubrique.

## <a name="prerequisites"></a>Prérequis

Configurez votre environnement de développement et ajoutez des informations de connexion au fichier app.config selon la procédure décrite dans l’article [Développement Media Services avec .NET](media-services-dotnet-how-to-use.md). 

## <a name="get-an-asset-reference"></a>Obtenir une référence pointant vers un élément multimédia
L’obtention d’une référence pointant vers un élément multimédia existant dans Media Services est une tâche fréquente. L’exemple de code suivant montre comment vous pouvez obtenir une référence pointant vers un élément multimédia à partir de la collection Assets sur l’objet de contexte du serveur, en fonction d’un ID d’élément multimédia. L’exemple de code suivant utilise une requête Linq pour obtenir une référence pointant vers un objet IAsset existant.

```csharp
    static IAsset GetAsset(string assetId)
    {
        // Use a LINQ Select query to get an asset.
        var assetInstance =
            from a in _context.Assets
            where a.Id == assetId
            select a;
        // Reference the asset as an IAsset.
        IAsset asset = assetInstance.FirstOrDefault();

        return asset;
    }
```

## <a name="list-all-assets"></a>Répertorier tous les éléments multimédias
Lorsque le nombre d’éléments multimédias de votre stockage augmente, il est utile de les répertorier. L’exemple de code suivant montre comment parcourir la collection Assets sur l’objet de contexte du serveur. Pour chaque élément multimédia, l’exemple de code écrit également certaines de ses valeurs de propriétés vers la console. Par exemple, chaque élément multimédia peut contenir plusieurs fichiers multimédias. L’exemple de code écrit tous les fichiers associés à chaque élément multimédia.

```csharp
    static void ListAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IAsset asset in _context.Assets)
        {
            // Display the collection of assets.
            builder.AppendLine("");
            builder.AppendLine("******ASSET******");
            builder.AppendLine("Asset ID: " + asset.Id);
            builder.AppendLine("Name: " + asset.Name);
            builder.AppendLine("==============");
            builder.AppendLine("******ASSET FILES******");

            // Display the files associated with each asset. 
            foreach (IAssetFile fileItem in asset.AssetFiles)
            {
                builder.AppendLine("Name: " + fileItem.Name);
                builder.AppendLine("Size: " + fileItem.ContentFileSize);
                builder.AppendLine("==============");
            }
        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="get-a-job-reference"></a>Obtenir une référence pointant vers un travail

Lorsque vous utilisez des tâches de traitement dans le code de Media Services, vous devez souvent obtenir une référence pointant vers un travail existant basé sur un ID. L’exemple de code suivant montre comment obtenir une référence pointant vers un objet IJob à partir de la collection Jobs.

Il est possible que vous deviez obtenir une référence pointant vers un travail lors du démarrage d’un travail d’encodage à long terme et vérifier l’état du travail sur un thread. Dans ce cas, lorsque la méthode est retournée à partir d’un thread, vous devez récupérer une référence actualisée pointant vers le travail.

```csharp
    static IJob GetJob(string jobId)
    {
        // Use a Linq select query to get an updated 
        // reference by Id. 
        var jobInstance =
            from j in _context.Jobs
            where j.Id == jobId
            select j;
        // Return the job reference as an Ijob. 
        IJob job = jobInstance.FirstOrDefault();

        return job;
    }
```

## <a name="list-jobs-and-assets"></a>Répertorier les travaux et les éléments multimédias
Une tâche associée importante consiste à répertorier les éléments multimédias avec leur travail associé dans Media Services. L’exemple de code suivant montre comment répertorier chaque objet IJob, puis afficher les propriétés de chaque travail, toutes les tâches associées, ainsi que tous les éléments multimédias d’entrée et de sortie. Cet exemple de code peut être utilise pour bien d’autres tâches. Par exemple, si vous souhaitez répertorier les éléments multimédias de sortie à partir d’un ou plusieurs travaux d’encodage que vous avez exécutés précédemment, ce code montre comment accéder aux éléments multimédias de sortie. Lorsque vous avez une référence pointant vers un élément multimédia de sortie, vous pouvez alors livrer le contenu à d’autres utilisateurs ou applications en le téléchargeant ou en fournissant des URL. 

Pour plus d’informations sur les options pour la livraison des éléments multimédias, consultez la page [Livraison d’éléments multimédias à l’aide du Kit de développement logiciel (SDK) Media Services pour .NET](media-services-deliver-streaming-content.md).

```csharp
    // List all jobs on the server, and for each job, also list 
    // all tasks, all input assets, all output assets.

    static void ListJobsAndAssets()
    {
        string waitMessage = "Building the list. This may take a few "
            + "seconds to a few minutes depending on how many assets "
            + "you have."
            + Environment.NewLine + Environment.NewLine
            + "Please wait..."
            + Environment.NewLine;
        Console.Write(waitMessage);

        // Create a Stringbuilder to store the list that we build. 
        StringBuilder builder = new StringBuilder();

        foreach (IJob job in _context.Jobs)
        {
            // Display the collection of jobs on the server.
            builder.AppendLine("");
            builder.AppendLine("******JOB*******");
            builder.AppendLine("Job ID: " + job.Id);
            builder.AppendLine("Name: " + job.Name);
            builder.AppendLine("State: " + job.State);
            builder.AppendLine("Order: " + job.Priority);
            builder.AppendLine("==============");


            // For each job, display the associated tasks (a job  
            // has one or more tasks). 
            builder.AppendLine("******TASKS*******");
            foreach (ITask task in job.Tasks)
            {
                builder.AppendLine("Task Id: " + task.Id);
                builder.AppendLine("Name: " + task.Name);
                builder.AppendLine("Progress: " + task.Progress);
                builder.AppendLine("Configuration: " + task.Configuration);
                if (task.ErrorDetails != null)
                {
                    builder.AppendLine("Error: " + task.ErrorDetails);
                }
                builder.AppendLine("==============");
            }

            // For each job, display the list of input media assets.
            builder.AppendLine("******JOB INPUT MEDIA ASSETS*******");
            foreach (IAsset inputAsset in job.InputMediaAssets)
            {

                if (inputAsset != null)
                {
                    builder.AppendLine("Input Asset Id: " + inputAsset.Id);
                    builder.AppendLine("Name: " + inputAsset.Name);
                    builder.AppendLine("==============");
                }
            }

            // For each job, display the list of output media assets.
            builder.AppendLine("******JOB OUTPUT MEDIA ASSETS*******");
            foreach (IAsset theAsset in job.OutputMediaAssets)
            {
                if (theAsset != null)
                {
                    builder.AppendLine("Output Asset Id: " + theAsset.Id);
                    builder.AppendLine("Name: " + theAsset.Name);
                    builder.AppendLine("==============");
                }
            }

        }

        // Display output in console.
        Console.Write(builder.ToString());
    }
```

## <a name="list-all-access-policies"></a>Répertorier toutes les stratégies d’accès
Dans Media Services, vous pouvez définir une stratégie d’accès sur un élément multimédia ou ses fichiers. Une stratégie d’accès définit les autorisations pour un fichier ou un élément multimédia (le type d’accès et la durée). Dans votre code Media Services, vous définissez généralement une stratégie d’accès en créant un objet IAccessPolicy et en l’associant à un élément multimédia existant. Ensuite, vous créez un objet ILocator, qui vous permet de fournir un accès direct aux éléments multimédias dans les Media Services. Le projet Visual Studio qui accompagne cette documentation contient plusieurs exemples de code qui montrent comment créer et attribuer des localisateurs et les stratégies d’accès aux éléments multimédias.

L’exemple de code suivant montre comment répertorier toutes les stratégies d’accès sur le serveur et quel type d’autorisations leur est associé. Une autre méthode d’affichage des stratégies d’accès consiste à répertorier tous les objets ILocator sur le serveur, puis répertorier les stratégies d’accès associées à chaque localisateur à l’aide de leur propriété AccessPolicy.

```csharp
    static void ListAllPolicies()
    {
        foreach (IAccessPolicy policy in _context.AccessPolicies)
        {
            Console.WriteLine("");
            Console.WriteLine("Name:  " + policy.Name);
            Console.WriteLine("ID:  " + policy.Id);
            Console.WriteLine("Permissions: " + policy.Permissions);
            Console.WriteLine("==============");

        }
    }
```
    
## <a name="limit-access-policies"></a>Limiter les stratégies d’accès 

>[!NOTE]
> Un nombre limite de 1 000 000 a été défini pour les différentes stratégies AMS (par exemple, pour la stratégie de localisateur ou pour ContentKeyAuthorizationPolicy). Vous devez utiliser le même ID de stratégie si vous utilisez toujours les mêmes jours / autorisations d’accès, par exemple, les stratégies pour les localisateurs destinées à demeurer en place pendant une longue période (stratégies sans chargement). 

Par exemple, vous pouvez créer un ensemble générique de stratégies avec le code suivant qui ne s’exécute qu’une seule fois dans votre application. Vous pouvez enregistrer les ID dans un fichier journal pour une utilisation ultérieure :

```csharp
    double year = 365.25;
    double week = 7;
    IAccessPolicy policyYear = _context.AccessPolicies.Create("One Year", TimeSpan.FromDays(year), AccessPermissions.Read);
    IAccessPolicy policy100Year = _context.AccessPolicies.Create("Hundred Years", TimeSpan.FromDays(year * 100), AccessPermissions.Read);
    IAccessPolicy policyWeek = _context.AccessPolicies.Create("One Week", TimeSpan.FromDays(week), AccessPermissions.Read);

    Console.WriteLine("One year policy ID is: " + policyYear.Id);
    Console.WriteLine("100 year policy ID is: " + policy100Year.Id);
    Console.WriteLine("One week policy ID is: " + policyWeek.Id);
```

Ensuite, vous pouvez utiliser les ID existants dans votre code comme suit :

```csharp
    const string policy1YearId = "nb:pid:UUID:2a4f0104-51a9-4078-ae26-c730f88d35cf";


    // Get the standard policy for 1 year read only
    var tempPolicyId = from b in _context.AccessPolicies
                       where b.Id == policy1YearId
                       select b;
    IAccessPolicy policy1Year = tempPolicyId.FirstOrDefault();

    // Get the existing asset
    var tempAsset = from a in _context.Assets
                where a.Id == assetID
                select a;
    IAsset asset = tempAsset.SingleOrDefault();

    ILocator originLocator = _context.Locators.CreateLocator(LocatorType.OnDemandOrigin, asset,
        policy1Year,
        DateTime.UtcNow.AddMinutes(-5));
    Console.WriteLine("The locator base path is " + originLocator.BaseUri.ToString());
```

## <a name="list-all-locators"></a>Répertorier tous les localisateurs
Un localisateur est une URL qui fournit un chemin d’accès direct pour accéder à un élément multimédia, ainsi que les autorisations pour accéder à l’élément multimédia, comme défini par la stratégie d’accès associée au localisateur. Chaque élément multimédia peut avoir une collection d’objets ILocator associée à sa propriété Locators. Le contexte de serveur possède également une collection Locators contenant tous les localisateurs.

L’exemple de code suivant répertorie tous les localisateurs sur le serveur. Il affiche l’ID des éléments multimédias et les stratégies d’accès associés à chaque localisateur. Il affiche également le type d’autorisation, la date d’expiration et le chemin d’accès complet à l’élément multimédia.

Notez qu’un chemin d’accès de localisateur vers un élément multimédia est simplement une URL de base pointant vers l’élément multimédia. Pour créer un chemin d’accès direct vers les fichiers individuels auxquels peut accéder un utilisateur ou une application, votre code doit ajouter le chemin d’accès de fichier spécifique au chemin d’accès du localisateur. Pour plus d’informations sur la procédure à suivre, consultez [Livraison d’éléments multimédias à l’aide du Kit de développement logiciel (SDK) Media Services pour .NET](media-services-deliver-streaming-content.md).

```csharp
    static void ListAllLocators()
    {
        foreach (ILocator locator in _context.Locators)
        {
            Console.WriteLine("***********");
            Console.WriteLine("Locator Id: " + locator.Id);
            Console.WriteLine("Locator asset Id: " + locator.AssetId);
            Console.WriteLine("Locator access policy Id: " + locator.AccessPolicyId);
            Console.WriteLine("Access policy permissions: " + locator.AccessPolicy.Permissions);
            Console.WriteLine("Locator expiration: " + locator.ExpirationDateTime);
            // The locator path is the base or parent path (with included permissions) to access  
            // the media content of an asset. To create a full URL to a specific media file, take 
            // the locator path and then append a file name and info as needed.  
            Console.WriteLine("Locator base path: " + locator.Path);
            Console.WriteLine("");
        }
    }
```

## <a name="enumerating-through-large-collections-of-entities"></a>Énumérer les grandes collections d'entités
Lors de l'interrogation des entités, il existe une limite de 1 000 entités retournées simultanément car l'API REST v2 publique limite les résultats des requêtes à 1 000 résultats. Vous devez utiliser Skip et Take lors de l'énumération de grandes collections d'entités. 

La fonction suivante effectue une itération sur toutes les tâches dans le compte Media Services fourni. Media Services renvoie 1 000 tâches dans Collection de tâches. La fonction utilise Skip et Take pour s'assurer que toutes les tâches sont énumérées (au cas où vous avez plus de 1 000 tâches dans votre compte).

```csharp
    static void ProcessJobs()
    {
        try
        {

            int skipSize = 0;
            int batchSize = 1000;
            int currentBatch = 0;

            while (true)
            {
                // Loop through all Jobs (1000 at a time) in the Media Services account
                IQueryable _jobsCollectionQuery = _context.Jobs.Skip(skipSize).Take(batchSize);
                foreach (IJob job in _jobsCollectionQuery)
                {
                    currentBatch++;
                    Console.WriteLine("Processing Job Id:" + job.Id);
                }

                if (currentBatch == batchSize)
                {
                    skipSize += batchSize;
                    currentBatch = 0;
                }
                else
                {
                    break;
                }
            }
        }
        catch (Exception ex)
        {
            Console.WriteLine(ex.Message);
        }
    }
```

## <a name="delete-an-asset"></a>Supprimer un élément multimédia
L’exemple suivant montre la suppression d’un élément multimédia.

```csharp
    static void DeleteAsset( IAsset asset)
    {
        // delete the asset
        asset.Delete();

        // Verify asset deletion
        if (GetAsset(asset.Id) == null)
            Console.WriteLine("Deleted the Asset");

    }
```

## <a name="delete-a-job"></a>Supprimer un travail
Pour supprimer un travail, vous devez vérifier l’état associé, comme indiqué dans la propriété State. Les travaux terminés ou annulés peuvent être supprimés, tandis que pour les travaux dans d’autres états, par exemple en file d’attente, planifiés ou en cours de traitement, il faut d’abord procéder à une annulation, puis supprimer le travail.

L’exemple de code suivant montre une méthode de suppression de travail qui vérifie l’état des travaux, puis supprime ceux terminés ou annulés. Ce code est basé sur la section précédente de cette rubrique pour obtenir une référence pointant vers un travail : Obtenir une référence pointant vers un travail.

```csharp
    static void DeleteJob(string jobId)
    {
        bool jobDeleted = false;

        while (!jobDeleted)
        {
            // Get an updated job reference.  
            IJob job = GetJob(jobId);

            // Check and handle various possible job states. You can 
            // only delete a job whose state is Finished, Error, or Canceled.   
            // You can cancel jobs that are Queued, Scheduled, or Processing,  
            // and then delete after they are canceled.
            switch (job.State)
            {
                case JobState.Finished:
                case JobState.Canceled:
                case JobState.Error:
                    // Job errors should already be logged by polling or event 
                    // handling methods such as CheckJobProgress or StateChanged.
                    // You can also call job.DeleteAsync to do async deletes.
                    job.Delete();
                    Console.WriteLine("Job has been deleted.");
                    jobDeleted = true;
                    break;
                case JobState.Canceling:
                    Console.WriteLine("Job is cancelling and will be deleted "
                        + "when finished.");
                    Console.WriteLine("Wait while job finishes canceling...");
                    Thread.Sleep(5000);
                    break;
                case JobState.Queued:
                case JobState.Scheduled:
                case JobState.Processing:
                    job.Cancel();
                    Console.WriteLine("Job is scheduled or processing and will "
                        + "be deleted.");
                    break;
                default:
                    break;
            }

        }
    }
```


## <a name="delete-an-access-policy"></a>Supprimer une stratégie d’accès
L’exemple de code suivant montre comment obtenir une référence pointant vers une stratégie d’accès en fonction d’un ID de stratégie, puis comment supprimer cette stratégie.

```csharp
    static void DeleteAccessPolicy(string existingPolicyId)
    {
        // To delete a specific access policy, get a reference to the policy.  
        // based on the policy Id passed to the method.
        var policyInstance =
                from p in _context.AccessPolicies
                where p.Id == existingPolicyId
                select p;
        IAccessPolicy policy = policyInstance.FirstOrDefault();

        policy.Delete();

    }
```


## <a name="media-services-learning-paths"></a>Parcours d’apprentissage de Media Services
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Fournir des commentaires
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

---
title: Conserver les données de sortie dans Stockage Azure avec l’API de service Batch
description: Découvrez comment utiliser l’API de service Batch pour conserver les données de sortie d’un travail et d’une tâche Batch dans le Stockage Azure.
ms.topic: how-to
ms.date: 07/30/2020
ms.custom: seodec18, devx-track-csharp
ms.openlocfilehash: 720c064c6b382bc62565c0828422181c761df8e8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88936926"
---
# <a name="persist-task-data-to-azure-storage-with-the-batch-service-api"></a>Conserver les données de tâche dans le stockage Azure avec l’API de service Batch

[!INCLUDE [batch-task-output-include](../../includes/batch-task-output-include.md)]

L’API de service Batch prend en charge la conservation des données de sortie dans le Stockage Azure pour les tâches et les tâches du Gestionnaire de travaux qui s’exécutent sur des pools avec la configuration de machine virtuelle. Quand vous ajoutez une tâche, vous pouvez spécifier un conteneur dans le stockage Azure comme destination pour le résultat de la tâche. Quand la tâche est terminée, le service Batch écrit les données de sortie dans ce conteneur.

L’un des avantages de l’utilisation de l’API de service Batch pour conserver le résultat de la tâche est que vous n’avez pas besoin de modifier l’application exécutée par la tâche. Au lieu de cela, il vous suffit de modifier légèrement votre application cliente pour conserver la sortie de la tâche dans le code qui crée la tâche.

> [!IMPORTANT]
> La conservation des données de tâche dans Stockage Azure avec l’API de service Batch ne fonctionne pas avec les pools créés avant le [1er février 2018](https://github.com/Azure/Batch/blob/master/changelogs/nodeagent/CHANGELOG.md#1204).

## <a name="when-do-i-use-the-batch-service-api-to-persist-task-output"></a>Quand utiliser l’API de service Batch pour conserver le résultat de la tâche ?

Azure Batch offre plusieurs manières de conserver les sorties de tâche. L’utilisation de l’API de service Batch est une approche pratique qui convient particulièrement aux scénarios suivants :

- Vous souhaitez écrire du code pour conserver le résultat de la tâche à partir de votre application cliente, sans modifier l’application exécutée par votre tâche.
- Vous souhaitez conserver le résultat des tâches Batch et des tâches du Gestionnaire de travaux dans des pools créés avec la configuration de machine virtuelle.
- Vous souhaitez conserver le résultat vers un conteneur de stockage Azure avec un nom arbitraire.
- Vous souhaitez conserver le résultat vers un conteneur de stockage Azure nommé conformément à la [norme relative aux Conventions applicables aux fichiers Batch](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files).

Si votre scénario diffère de ceux répertoriés ci-dessus, vous devrez peut-être envisager une approche différente. Par exemple, l’API de service Batch ne prend pas en charge actuellement la diffusion en continu du résultat vers le stockage Azure pendant l’exécution de la tâche. Pour diffuser le résultat en continu, utilisez la bibliothèque de Conventions applicables aux fichiers Batch, disponible pour .NET. Pour d’autres langages, vous devez implémenter votre propre solution. Pour en savoir plus sur les autres options de conservation des sorties de tâche, consultez l’article [Conserver les sorties de travail et de tâche dans Stockage Azure](batch-task-output.md).

## <a name="create-a-container-in-azure-storage"></a>Créer un conteneur dans le stockage Azure

Pour conserver le résultat de la tâche dans le stockage Azure, vous devez créer un conteneur qui sert de destination pour vos fichiers de sortie. Créez le conteneur avant d’exécuter la tâche, de préférence avant d’envoyer votre travail. Pour créer le conteneur, utilisez la bibliothèque cliente de stockage Azure ou le SDK approprié. Pour plus d’informations sur les API de stockage Azure, consultez la [documentation sur le stockage Azure](../storage/index.yml).

Par exemple, si vous écrivez votre application en C#, utilisez la [bibliothèque cliente de stockage Azure pour .NET](https://www.nuget.org/packages/WindowsAzure.Storage/). L’exemple suivant montre comment créer un conteneur :

```csharp
CloudBlobContainer container = storageAccount.CreateCloudBlobClient().GetContainerReference(containerName);
await container.CreateIfNotExists();
```

## <a name="get-a-shared-access-signature-for-the-container"></a>Obtenir une signature d’accès partagé pour le conteneur

Après avoir créé le conteneur, obtenez une signature d’accès partagé avec un accès en écriture au conteneur. Une signature d’accès partagé fournit un accès délégué au conteneur. La signature d’accès partagé accorde l’accès avec un jeu spécifié d’autorisations et sur un intervalle de temps spécifié. Le service Batch a besoin d’une signature d’accès partagé avec des autorisations d’écriture pour pouvoir écrire le résultat de la tâche dans le conteneur. Pour plus d’informations sur les signatures d’accès partagé, consultez [Utilisation des signatures d’accès partagé \(SAP\) dans le stockage Azure](../storage/common/storage-sas-overview.md).

Quand vous obtenez une signature d’accès partagé à l’aide des API de stockage Azure, l’API retourne une chaîne de jeton de signature d’accès partagé. Cette chaîne de jeton inclut tous les paramètres de la signature d’accès partagé, notamment les autorisations et l’intervalle pendant lequel la signature d’accès partagé est valide. Pour utiliser la signature d’accès partagé pour accéder à un conteneur dans le stockage Azure, vous devez ajouter la chaîne de jeton de signature d’accès partagé à l’URI de ressource. L’URI de ressource, avec le jeton de signature d’accès partagé ajouté, fournit un accès authentifié au stockage Azure.

L’exemple suivant montre comment obtenir une chaîne de jeton de signature d’accès partagé en écriture seule pour le conteneur, puis ajoute la signature d’accès partagé à l’URI du conteneur :

```csharp
string containerSasToken = container.GetSharedAccessSignature(new SharedAccessBlobPolicy()
{
    SharedAccessExpiryTime = DateTimeOffset.UtcNow.AddDays(1),
    Permissions = SharedAccessBlobPermissions.Write
});

string containerSasUrl = container.Uri.AbsoluteUri + containerSasToken;
```

## <a name="specify-output-files-for-task-output"></a>Spécifier les fichiers de sortie pour le résultat de la tâche

Pour spécifier les fichiers de sortie pour une tâche, créez une collection d’objets [OutputFile](/dotnet/api/microsoft.azure.batch.outputfile) et affectez-la à la propriété [CloudTask.OutputFiles](/dotnet/api/microsoft.azure.batch.cloudtask.outputfiles#Microsoft_Azure_Batch_CloudTask_OutputFiles) quand vous créez la tâche.

L’exemple de code C# suivant crée une tâche qui écrit des nombres aléatoires dans un fichier nommé `output.txt`. L’exemple crée un fichier de sortie pour `output.txt` à écrire dans le conteneur. Il crée également des fichiers de sortie pour tout fichier journal qui correspond au modèle de fichier `std*.txt` (_par exemple_`stdout.txt` et `stderr.txt`). L’URL du conteneur a besoin de la signature d’accès partagé créée précédemment pour le conteneur. Le service Batch utilise la signature d’accès partagé pour authentifier l’accès au conteneur :

```csharp
new CloudTask(taskId, "cmd /v:ON /c \"echo off && set && (FOR /L %i IN (1,1,100000) DO (ECHO !RANDOM!)) > output.txt\"")
{
    OutputFiles = new List<OutputFile>
    {
        new OutputFile(
            filePattern: @"..\std*.txt",
            destination: new OutputFileDestination(
         new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId)),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
        new OutputFile(
            filePattern: @"output.txt",
            destination: 
         new OutputFileDestination(new OutputFileBlobContainerDestination(
                    containerUrl: containerSasUrl,
                    path: taskId + @"\output.txt")),
            uploadOptions: new OutputFileUploadOptions(
            uploadCondition: OutputFileUploadCondition.TaskCompletion)),
}
```

> [!NOTE]
> Si vous utilisez cet exemple avec Linux, veillez à remplacer les barres obliques inverses par des barres obliques.

### <a name="specify-a-file-pattern-for-matching"></a>Spécifier un modèle de fichier pour la correspondance

Quand vous spécifiez un fichier de sortie, vous pouvez utiliser la propriété [OutputFile.FilePattern](/dotnet/api/microsoft.azure.batch.outputfile.filepattern#Microsoft_Azure_Batch_OutputFile_FilePattern) pour spécifier un modèle de fichier pour la correspondance. Le modèle de fichier peut faire correspondre zéro fichier, un seul fichier ou un ensemble de fichiers créés par la tâche.

La propriété **FilePattern** prend en charge les caractères génériques de système de fichiers standard tels que `*` (pour les correspondances non récursives) et `**` (pour les correspondances récursives). Par exemple, l’exemple de code ci-dessus spécifie le modèle de fichier pour établir une mise en correspondance avec `std*.txt` de manière non récursive :

`filePattern: @"..\std*.txt"`

Pour charger un seul fichier, spécifiez un modèle de fichier sans caractère générique. Par exemple, l’exemple de code ci-dessus spécifie le modèle de fichier pour établir une mise en correspondance avec `output.txt` :

`filePattern: @"output.txt"`

### <a name="specify-an-upload-condition"></a>Spécifier une condition de chargement

La propriété [OutputFileUploadOptions.UploadCondition](/dotnet/api/microsoft.azure.batch.outputfileuploadoptions.uploadcondition#Microsoft_Azure_Batch_OutputFileUploadOptions_UploadCondition) autorise le chargement conditionnel des fichiers de sortie. Un scénario courant consiste à charger un ensemble de fichiers si la tâche réussit, et un autre ensemble de fichiers en cas d’échec. Par exemple, vous pourriez souhaiter charger des fichiers journaux détaillés uniquement quand la tâche échoue et se termine avec un code de sortie différent de zéro. De même, vous pourriez souhaiter charger des fichiers de résultats uniquement si la tâche réussit, car ces fichiers pourraient être manquants ou incomplets si la tâche échoue.

L’exemple de code ci-dessus affecte la valeur **TaskCompletion** à la propriété **UploadCondition**. Ce paramètre spécifie que le fichier doit être chargé une fois la tâche terminée, quelle que soit la valeur du code de sortie.

`uploadCondition: OutputFileUploadCondition.TaskCompletion`

Pour plus d’informations sur les autres paramètres, consultez l’énumération [OutputFileUploadCondition](/dotnet/api/microsoft.azure.batch.common.outputfileuploadcondition).

### <a name="disambiguate-files-with-the-same-name"></a>Lever l’ambiguïté des fichiers portant le même nom

Les tâches d’un travail peuvent générer des fichiers ayant le même nom. Par exemple, `stdout.txt` et `stderr.txt` sont créés pour chaque tâche qui s’exécute dans un travail. Étant donné que chaque tâche s’exécute dans son propre contexte, ces fichiers ne sont pas en conflit sur le système de fichiers du nœud. Toutefois, quand vous chargez des fichiers à partir de plusieurs tâches vers un conteneur partagé, vous devez lever l’ambiguïté des fichiers portant le même nom.

La propriété [OutputFileBlobContainerDestination.Path](/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination.path#Microsoft_Azure_Batch_OutputFileBlobContainerDestination_Path) spécifie l’objet blob ou le répertoire virtuel de destination pour les fichiers de sortie. Vous pouvez utiliser la propriété **Path** pour nommer l’objet blob ou le répertoire virtuel de telle sorte que les fichiers de sortie portant le même nom soient nommés de manière unique dans le stockage Azure. L’utilisation de l’ID de tâche dans le chemin est un bon moyen de garantir le caractère unique des noms et de faciliter l’identification des fichiers.

Si la propriété **FilePattern** a comme valeur une expression générique, tous les fichiers qui correspondent au modèle sont chargés vers le répertoire virtuel spécifié par la propriété **Path**. Par exemple, si le conteneur est `mycontainer`, l’ID de tâche est `mytask` et le modèle de fichier est `..\std*.txt`, les URI absolus des fichiers de sortie dans le stockage Azure ressembleront à ceci :

```
https://myaccount.blob.core.windows.net/mycontainer/mytask/stderr.txt
https://myaccount.blob.core.windows.net/mycontainer/mytask/stdout.txt
```

Si la propriété **FilePattern** est définie pour établir une correspondance avec un seul nom de fichier, autrement dit si elle ne contient pas de caractère générique, alors la valeur de la propriété **Path** spécifie le nom de l’objet blob complet. Si vous prévoyez des conflits d’affectation de noms avec un même fichier de plusieurs tâches, incluez le nom du répertoire virtuel dans le cadre du nom de fichier pour lever l’ambiguïté de ces fichiers. Par exemple, définissez la propriété **Path** pour qu’elle inclue l’ID de tâche, le caractère délimiteur (en général une barre oblique) et le nom de fichier :

`path: taskId + @"/output.txt"`

Les URI absolus des fichiers de sortie pour un ensemble de tâches ressembleront à ceci :

```
https://myaccount.blob.core.windows.net/mycontainer/task1/output.txt
https://myaccount.blob.core.windows.net/mycontainer/task2/output.txt
```

Pour plus d’informations sur les répertoires virtuels dans le stockage Azure, consultez [Création d’une liste d’objets blob dans un conteneur](../storage/blobs/storage-quickstart-blobs-dotnet.md#list-the-blobs-in-a-container).

## <a name="diagnose-file-upload-errors"></a>Diagnostiquer les erreurs de chargement de fichier

Si le chargement des fichiers de sortie dans le stockage Azure échoue, la tâche bascule à l’état **Terminée** et la propriété [TaskExecutionInformation.FailureInformation](/dotnet/api/microsoft.azure.batch.taskexecutioninformation.failureinformation#Microsoft_Azure_Batch_TaskExecutionInformation_FailureInformation) est définie. Examinez la propriété **FailureInformation** pour identifier l’erreur qui s’est produite. Voici par exemple une erreur qui se produit lors du chargement de fichier si le conteneur est introuvable :

```
Category: UserError
Code: FileUploadContainerNotFound
Message: One of the specified Azure container(s) was not found while attempting to upload an output file
```

À chaque chargement de fichier, Batch écrit deux fichiers journaux dans le nœud de calcul, `fileuploadout.txt` et `fileuploaderr.txt`. Vous pouvez examiner ces fichiers journaux pour en savoir plus sur un échec spécifique. Si le chargement de fichier n’a jamais été tenté, par exemple, parce que la tâche proprement dite n’a pas pu être exécutée, ces fichiers journaux n’existent pas.

## <a name="diagnose-file-upload-performance"></a>Diagnostiquer les performances de chargement de fichier

Le fichier `fileuploadout.txt` contient la progression du chargement. Vous pouvez examiner ce fichier pour en savoir plus sur la durée nécessaire au chargement de vos fichiers. N’oubliez pas que de nombreux facteurs contribuent aux performances de chargement, notamment la taille du nœud, les autres activités sur le nœud au moment du chargement, si le conteneur cible est dans la même région que le pool Batch, le nombre de nœuds qui effectuent des chargements en même temps vers le compte de stockage, et ainsi de suite.

## <a name="use-the-batch-service-api-with-the-batch-file-conventions-standard"></a>Utiliser l’API de service Batch avec la norme relative aux Conventions applicables aux fichiers Batch

Quand vous conservez le résultat de la tâche avec l’API de service Batch, vous pouvez nommer votre conteneur de destination et les objets blob comme vous le souhaitez. Vous pouvez également choisir de les nommer en respectant la [norme relative aux Conventions applicables aux fichiers Batch](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/batch/Microsoft.Azure.Batch.Conventions.Files). Cette norme détermine les noms du conteneur de destination et de l’objet blob dans le stockage Azure pour un fichier de sortie donnée en fonction des noms du projet et de la tâche. Si vous utilisez cette norme pour nommer les fichiers de sortie, vos fichiers de sortie sont visibles dans le [portail Azure](https://portal.azure.com).

Si vous développez en C#, vous pouvez utiliser les méthodes intégrées à la [bibliothèque Conventions applicables aux fichiers Batch pour .NET](https://www.nuget.org/packages/Microsoft.Azure.Batch.Conventions.Files). Cette bibliothèque crée les chemins de conteneurs et d’objets blob pour vous avec des noms corrects. Par exemple, vous pouvez appeler l’API pour obtenir le nom correct pour le conteneur, en fonction du nom du travail :

```csharp
string containerName = job.OutputStorageContainerName();
```

Vous pouvez utiliser la méthode [CloudJobExtensions.GetOutputStorageContainerUrl](/dotnet/api/microsoft.azure.batch.conventions.files.cloudjobextensions.getoutputstoragecontainerurl) pour retourner une URL de signature d’accès partagé utilisée pour écrire dans le conteneur. Vous pouvez ensuite passer cette signature d’accès partagé au constructeur [OutputFileBlobContainerDestination](/dotnet/api/microsoft.azure.batch.outputfileblobcontainerdestination).

Si vous développez dans un langage autre que C#, vous devez implémenter la norme relative aux Conventions de fichiers vous-même.

## <a name="code-sample"></a>Exemple de code

L’exemple de projet [PersistOutputs](https://github.com/Azure/azure-batch-samples/tree/master/CSharp/ArticleProjects/PersistOutputs) est l’un des [exemples de code Azure Batch](https://github.com/Azure/azure-batch-samples) disponibles sur GitHub. Cette solution Visual Studio montre comment utiliser la bibliothèque cliente Batch pour .NET pour conserver le résultat de la tâche dans l’espace de stockage durable. Pour exécuter l’exemple, procédez comme suit :

1. Ouvrez le projet dans **Visual Studio 2019**.
2. Ajoutez vos **informations d’identification de compte** Batch et Stockage à **AccountSettings.settings** dans le projet Microsoft.Azure.Batch.Samples.Common.
3. **Générez** la solution sans l’exécuter. Restaurez les packages NuGet si vous y êtes invité.
4. Utilisez le portail Azure pour charger un [package d’application](batch-application-packages.md) pour **PersistOutputsTask**. Insérez le fichier `PersistOutputsTask.exe` et ses assemblys dépendants dans le package .zip, puis définissez l’ID de l’application sur PersistOutputsTask et la version du package d’application sur 1.0.
5. **Démarrez** (exécutez) le projet **PersistOutputs**.
6. Quand vous êtes invité à choisir la technologie de persistance à utiliser pour l’exécution de l’exemple, entrez **2** pour exécuter l’exemple à l’aide de l’API de service Batch pour conserver le résultat de la tâche.
7. Si vous le souhaitez, réexécutez l’exemple, entrez **3** pour conserver la sortie avec l’API de service Batch et également nommer le chemin d’objet blob et de conteneur de destination conformément à la norme relative aux Conventions de fichiers.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la conservation de la sortie des tâches avec la bibliothèque File Conventions pour .NET, consultez [Conserver les données des travaux et des tâches dans Stockage Azure avec la bibliothèque File Conventions Batch pour .NET](batch-task-output-file-conventions.md).
- Pour en savoir plus sur d’autres approches de conservation des données de sortie dans Azure Batch, consultez [Conserver la sortie d’un travail et d’une tâche dans le stockage Azure](batch-task-output.md).

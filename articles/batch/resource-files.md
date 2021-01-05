---
title: Création et utilisation de fichiers de ressources
description: Découvrez comment créer des fichiers de ressources Batch à partir de diverses sources d’entrée. Cet article décrit quelques méthodes courantes pour les créer et les placer sur une machine virtuelle.
ms.date: 03/18/2020
ms.topic: how-to
ms.openlocfilehash: 84a5e9780b4fa0abfec5b736e04d385f14716873
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109287"
---
# <a name="creating-and-using-resource-files"></a>Création et utilisation de fichiers de ressources

Une tâche Azure Batch requiert généralement une certaine forme de données à traiter. Les fichiers de ressources permettent de fournir ces données à votre machine virtuelle Batch via une tâche. Tous les types de tâches prennent en charge les fichiers de ressources : tâches, tâches de démarrage, tâches de préparation de travail, tâches de mise en production de travail, etc. Cet article décrit quelques méthodes courantes pour créer des fichiers de ressources et les placer sur une machine virtuelle.  

Les fichiers de ressources permettent de placer des données sur une machine virtuelle dans Batch, mais le type de données et leur utilisation sont flexibles. Il existe toutefois des cas d’utilisation courants :

1. Approvisionner des fichiers communs sur chaque machine virtuelle à l’aide de fichiers de ressources sur une tâche de démarrage
1. Approvisionner des données d’entrée devant être traitées par des tâches

Les fichiers communs peuvent être, par exemple, des fichiers sur une tâche de démarrage utilisée pour installer des applications exécutées par vos tâches. Les données d’entrée peuvent être des données brutes d’image ou de vidéo, ou toute information devant être traitée par Batch.

## <a name="types-of-resource-files"></a>Types de fichiers de ressources

Différentes options sont disponibles pour générer des fichiers de ressources. Le processus de création de fichiers de ressources varie en fonction de l’emplacement de stockage des données d’origine.

Options de création d’un fichier de ressources :

- [URL de conteneur de stockage](#storage-container-url) : Génère un fichier de ressources à partir d’un conteneur de stockage dans Azure
- [Nom de conteneur de stockage](#storage-container-name) : Génère un fichier de ressources à partir du nom d’un conteneur dans un compte de Stockage Azure lié à Batch
- [Point de terminaison web](#web-endpoint) : Génère un fichier de ressources à partir d’une URL HTTP valide

### <a name="storage-container-url"></a>URL de conteneur de stockage

L’utilisation d’une URL de conteneur de stockage, avec les autorisations appropriées, signifie que vous pouvez accéder aux fichiers d’un quelconque conteneur de stockage dans Azure. 

Dans cet exemple C#, les fichiers ont déjà été chargés dans un conteneur de stockage Azure en tant que stockage d’objets blob. Pour accéder aux données nécessaires à la création d’un fichier de ressources, nous devons tout d’abord accéder au conteneur de stockage.

Créez un URI de signature d’accès partagé (SAS) avec les autorisations appropriées pour accéder au conteneur de stockage. Définissez le délai d’expiration et les autorisations pour la SAS. Dans ce cas, aucune heure de début n’est spécifiée afin que la SAS soit valide immédiatement et expire deux heures après sa génération.

```csharp
SharedAccessBlobPolicy sasConstraints = new SharedAccessBlobPolicy
{
    SharedAccessExpiryTime = DateTime.UtcNow.AddHours(2),
    Permissions = SharedAccessBlobPermissions.Read | SharedAccessBlobPermissions.List
};
```

> [!NOTE]
> Pour l’accès au conteneur, vous devez disposer des autorisations `Read` et `List`, tandis que pour l’accès aux objets blob, vous n’avez besoin que de l’autorisation `Read`.

Une fois que les autorisations sont configurées, créez le jeton SAS et mettez en forme l’URL SAS pour l’accès au conteneur de stockage. À l’aide de l’URL SAS mise en forme pour le conteneur de stockage, générez un fichier de ressources avec [`FromStorageContainerUrl`](/dotnet/api/microsoft.azure.batch.resourcefile.fromstoragecontainerurl).

```csharp
CloudBlobContainer container = blobClient.GetContainerReference(containerName);

string sasToken = container.GetSharedAccessSignature(sasConstraints);
string containerSasUrl = String.Format("{0}{1}", container.Uri, sasToken);

ResourceFile inputFile = ResourceFile.FromStorageContainerUrl(containerSasUrl);
```

Une alternative à la génération d’une URL SAS consiste à activer l’accès en lecture anonyme public pour un conteneur et ses objets blob dans Stockage Blob Azure. En procédant ainsi, vous pouvez accorder un accès en lecture seule à ces ressources sans partager votre clé de compte et sans exiger de signature d’accès partagé. L’accès en lecture public est généralement utilisé dans les situations où vous voulez conférer à certains objets blob un accès en lecture anonyme permanent. Si ce scénario convient à votre solution, consultez l’article [Accès anonyme aux objets Blob](../storage/blobs/anonymous-read-access-configure.md) article pour en savoir plus sur la gestion de l’accès à vos données d’objet blob.

### <a name="storage-container-name"></a>Nom de conteneur de stockage

Plutôt que de configurer et de créer une URL SAS, vous pouvez utiliser le nom de votre conteneur de stockage Azure pour accéder à vos données d’objet blob. Le conteneur de stockage que vous utilisez doit se trouver dans le compte de stockage Azure lié à votre compte Batch. Ce compte de stockage est également appelé compte « autostorage ». L’utilisation du conteneur « autostorage » vous permet de contourner la configuration et la création d’une URL SAS pour accéder à un conteneur de stockage.

Dans cet exemple, nous partons du principe que les données à utiliser pour la création de fichier de ressources se trouvent déjà dans un compte de Stockage Azure lié à votre compte Batch. Si vous n’avez pas de compte « autostorage », consultez les étapes décrites dans [Créer un compte Batch](batch-account-create-portal.md) pour plus d’informations sur la création et la liaison d’un compte.

Avec un compte de stockage lié, vous n’avez pas besoin de créer et de configurer une URL SAS à un conteneur de stockage. Indiquez plutôt le nom du conteneur de stockage dans votre compte de stockage lié.

```csharp
ResourceFile inputFile = ResourceFile.FromAutoStorageContainer(containerName);
```

### <a name="web-endpoint"></a>Point de terminaison web

Les données qui ne sont pas téléchargées sur le Stockage Azure peuvent toujours être utilisées pour créer des fichiers de ressources. Vous pouvez spécifier une quelconque URL HTTP valide contenant vos données d’entrée. L’URL est fournie à l’API Batch, et les données sont ensuite utilisées pour créer un fichier de ressources.

Dans l’exemple C# suivant, les données d’entrée sont hébergées sur un point de terminaison GitHub fictif. L’API récupère le fichier du point de terminaison web valide et génère un fichier de ressources qui sera utilisé par votre tâche. Aucune information d’identification n’est nécessaire pour ce scénario.

```csharp
ResourceFile inputFile = ResourceFile.FromUrl("https://github.com/foo/file.txt", filePath);
```

## <a name="tips-and-suggestions"></a>Conseils et suggestions

Chaque tâche Azure Batch utilise des fichiers différemment. C’est la raison pour laquelle Batch propose des options de gestion des fichiers sur des tâches. Les scénarios suivants ne sont pas censés être exhaustifs, mais couvrent quelques situations courantes et fournissent des suggestions.

### <a name="many-resource-files"></a>Plusieurs fichiers de ressources

Votre travail Batch peut contenir plusieurs tâches qui utilisent toutes les mêmes fichiers communs. Si des fichiers de tâche communs sont partagés entre plusieurs tâches, l’utilisation d’un package d’application pour contenir les fichiers au lieu de fichiers de ressources peut être une meilleure option. Les packages d’application permettent d’optimiser la vitesse de téléchargement. Les données dans des packages d’application sont en outre mises en cache entre les tâches. Ainsi, si vos fichiers de tâche ne changent pas souvent, les packages d’application peuvent donc convenir à votre solution. Grâce aux packages d’application, vous n’avez pas besoin de gérer manuellement plusieurs fichiers de ressources ni de générer des URL SAS pour accéder aux fichiers dans le Stockage Azure. Batch fonctionne en arrière-plan avec le Stockage Azure pour stocker et déployer des packages d’application sur des nœuds de calcul.

Si chaque tâche comporte de nombreux fichiers uniques à celle-ci, les fichiers de ressources sont très probablement la meilleure option car les tâches qui utilisent des fichiers uniques doivent généralement être mises à jour ou remplacées, ce qui n’est pas si simple avec le contenu de packages d’application. Les fichiers de ressources offrent une flexibilité supplémentaire pour la mise à jour, l’ajout ou la modification de fichiers individuels.

### <a name="number-of-resource-files-per-task"></a>Nombre de fichiers de ressources par tâche

Si plusieurs centaines de fichiers de ressources sont spécifiés sur une tâche, Batch peut rejeter la tâche car trop importante. Des petites tâches sont préférables, en réduisant le nombre de fichiers de ressources sur la tâche elle-même.

S’il est impossible de réduire le nombre de fichiers nécessaires à votre tâche, vous pouvez optimiser la tâche en créant un fichier de ressources unique qui fait référence à un conteneur de stockage de fichiers de ressources. Pour cela, placez vos fichiers de ressources dans un conteneur de Stockage Azure et utilisez les différents [modes](/dotnet/api/microsoft.azure.batch.resourcefile#methods) de « Conteneur » de fichiers de ressources. Utilisez les options de préfixe d’objet blob pour spécifier des collections de fichiers à télécharger pour vos tâches.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur [packages d’application](batch-application-packages.md) comme alternative aux fichiers de ressources.
- Pour plus d’informations sur l’utilisation de conteneurs pour les fichiers de ressources, consultez les [charges de travail de conteneur](batch-docker-container-workloads.md).
- Pour savoir comment collecter et enregistrer les données de sortie de vos tâches, consultez [Conserver les résultats des tâches et des travaux](batch-task-output.md).
- Découvrez les [outils et API Batch](batch-apis-tools.md) disponibles pour créer des solutions Batch.
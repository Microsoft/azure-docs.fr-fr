---
title: Tutoriel - Générer une application hautement disponible avec le stockage Blob
titleSuffix: Azure Storage
description: Utilisez le stockage géoredondant interzone (RA-GZRS) avec accès en lecture pour rendre vos données d’application hautement disponibles.
services: storage
author: tamram
ms.service: storage
ms.topic: tutorial
ms.date: 04/16/2020
ms.author: tamram
ms.reviewer: artek
ms.custom: mvc, devx-track-python, devx-track-js, devx-track-csharp
ms.subservice: blobs
ms.openlocfilehash: 1c1ba7d8cd0e4202003a98153a48e0593d1fcd04
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95543151"
---
# <a name="tutorial-build-a-highly-available-application-with-blob-storage"></a>Tutoriel : Générer une application hautement disponible avec le stockage Blob

Ce tutoriel est la première partie d’une série d’étapes. Il vous apprend à rendre vos données d’application hautement disponibles dans Azure.

À l’issue de ce tutoriel, vous disposez d’une application console qui charge et récupère un objet blob à partir d’un compte de stockage [géoredondant interzone avec accès en lecture](../common/storage-redundancy.md) (RA-GZRS).

La géo-redondance dans le stockage Azure réplique les transactions de manière asynchrone d’une région primaire vers une région secondaire se trouvant à des centaines de kilomètres. Ce processus de réplication garantit que les données de la région secondaire sont cohérentes. L’application de console utilise le modèle [Disjoncteur](/azure/architecture/patterns/circuit-breaker) pour déterminer à quel point de terminaison se connecter, en basculant automatiquement d’un point de terminaison à l’autre au fur et à mesure des simulations d’échecs et de récupérations.

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créez un compte de stockage.
> * Définir la chaîne de connexion
> * Exécuter l’application console

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel :

# <a name="net"></a>[.NET](#tab/dotnet)

* Installez [Visual Studio 2019](https://www.visualstudio.com/downloads/) avec la charge de travail **Développement Azure**.

  ![Développement Azure (sous Web & Cloud)](media/storage-create-geo-redundant-storage/workloads.png)

# <a name="python"></a>[Python](#tab/python)

* Installez [Python](https://www.python.org/downloads/)
* Téléchargez et installez le [SDK Stockage Azure pour Python](https://github.com/Azure/azure-storage-python).

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

* Installez [Node.js](https://nodejs.org).

---

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au [portail Azure](https://portal.azure.com/).

## <a name="create-a-storage-account"></a>Créez un compte de stockage.

Un compte de stockage fournit un espace de noms unique pour stocker les objets de données de Stockage Azure et y accéder.

Suivez ces étapes pour créer un compte de stockage géoredondant interzone (RA-GZRS) avec accès en lecture :

1. Sélectionnez le bouton **Créer une ressource** dans le portail Azure.
2. Sélectionnez **Compte de stockage - blob, fichier, table, file d’attente** sur la page **Nouveau**.
4. Remplissez le formulaire de compte de stockage avec les informations suivantes, comme indiqué dans l’image ci-après et sélectionnez **Créer** :

   | Paramètre       | Exemple de valeur | Description |
   | ------------ | ------------------ | ------------------------------------------------- |
   | **Abonnement** | *Mon abonnement* | Pour plus d’informations sur vos abonnements, consultez [Abonnements](https://account.azure.com/Subscriptions). |
   | **ResourceGroup** | *myResourceGroup* | Pour les noms de groupe de ressources valides, consultez [Naming conventions](/azure/architecture/best-practices/resource-naming) (Conventions d’affectation de nom). |
   | **Nom** | *mystorageaccount* | Un nom unique pour votre compte de stockage. |
   | **Lieu** | *USA Est* | Choisissez un emplacement. |
   | **Performances** | *Standard* | Les performances Standard sont une bonne option pour l’exemple de scénario. |
   | **Type de compte** | *StorageV2* | Il est recommandé d’utiliser un compte de stockage v2 à usage général. Pour plus d’informations sur les types de comptes de stockage Azure, consultez [Vue d’ensemble des comptes de stockage Azure](../common/storage-account-overview.md). |
   | **Réplication**| *Stockage géo-redondant interzone avec accès en lecture (RA-GRS)* | La région primaire est redondante dans une zone et est répliquée vers une région secondaire, avec accès en lecture à la région secondaire activée. |
   | **Niveau d’accès**| *Chaud* | Utilisez le niveau de stockage chaud pour les données fréquemment consultées. |

    ![créer un compte de stockage](media/storage-create-geo-redundant-storage/createragrsstracct.png)

## <a name="download-the-sample"></a>Télécharger l’exemple

# <a name="net"></a>[.NET](#tab/dotnet)

[Téléchargez l’exemple de projet](https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) et extrayez (décompressez) le fichier storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. Vous pouvez aussi utiliser [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. L’exemple de projet contient une application console.

```bash
git clone https://github.com/Azure-Samples/storage-dotnet-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="python"></a>[Python](#tab/python)

[Téléchargez l’exemple de projet](https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs/archive/master.zip) et extrayez (décompressez) le fichier storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.zip. Vous pouvez aussi utiliser [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. L’exemple de projet contient une application Python de base.

```bash
git clone https://github.com/Azure-Samples/storage-python-circuit-breaker-pattern-ha-apps-using-ra-grs.git
```

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

[Téléchargez l’exemple de projet](https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs) et décompressez le fichier. Vous pouvez aussi utiliser [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. L’exemple de projet contient une application Node.js de base.

```bash
git clone https://github.com/Azure-Samples/storage-node-v10-ha-ra-grs
```

---

## <a name="configure-the-sample"></a>Configurer l'exemple

# <a name="net"></a>[.NET](#tab/dotnet)

Dans l’application, vous devez fournir la chaîne de connexion de votre compte de stockage. Vous pouvez stocker cette chaîne de connexion dans une variable d’environnement sur l’ordinateur local exécutant l’application. Suivez l’un des exemples ci-dessous, en fonction de votre système d’exploitation, pour créer la variable d’environnement.

Dans le portail Azure, accédez à votre compte de stockage. Sélectionnez **Clés d’accès** sous **Paramètres** dans votre compte de stockage. Copiez la **chaîne de connexion** de la clé primaire ou secondaire. Exécutez l’une des commandes suivantes en fonction de votre système d’exploitation, en remplaçant \<yourconnectionstring\> par la chaîne de connexion réelle. Cette commande enregistre une variable d’environnement sur la machine locale. Dans Windows, la variable d’environnement n’est pas disponible tant que vous n’avez pas rechargé l’**invite de commandes** ou l’interpréteur de commandes que vous utilisez.

### <a name="linux"></a>Linux

```
export storageconnectionstring=<yourconnectionstring>
```

### <a name="windows"></a>Windows

```powershell
setx storageconnectionstring "<yourconnectionstring>"
```

# <a name="python"></a>[Python](#tab/python)

Dans l’application, vous devez fournir les informations d’identification de votre compte de stockage. Vous pouvez stocker ces informations dans des variables d’environnement sur la machine locale exécutant l’application. Suivez l’un des exemples ci-dessous, en fonction de votre système d’exploitation, pour créer les variables d’environnement.

Dans le portail Azure, accédez à votre compte de stockage. Sélectionnez **Clés d’accès** sous **Paramètres** dans votre compte de stockage. Collez les valeurs **Nom du compte de stockage** et **Clé** dans les commandes suivantes, en remplaçant les espaces réservés \<youraccountname\> et \<youraccountkey\>. Cette commande enregistre les variables d’environnement sur la machine locale. Dans Windows, la variable d’environnement n’est pas disponible tant que vous n’avez pas rechargé l’**invite de commandes** ou l’interpréteur de commandes que vous utilisez.

### <a name="linux"></a>Linux

```
export accountname=<youraccountname>
export accountkey=<youraccountkey>
```

### <a name="windows"></a>Windows

```powershell
setx accountname "<youraccountname>"
setx accountkey "<youraccountkey>"
```

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Pour exécuter cet exemple, vous devez ajouter les informations d’identification de votre compte de stockage au fichier `.env.example` et le renommer `.env`.

```
AZURE_STORAGE_ACCOUNT_NAME=<replace with your storage account name>
AZURE_STORAGE_ACCOUNT_ACCESS_KEY=<replace with your storage account access key>
```

Pour trouver ces informations dans le portail Azure, accédez à votre compte de stockage et sélectionnez **Clés d’accès** dans la section **Paramètres**.

Installez les dépendances nécessaires. Pour cela, ouvrez une invite de commandes, accédez à l’exemple de dossier et entrez `npm install`.

---

## <a name="run-the-console-application"></a>Exécuter l’application console

# <a name="net"></a>[.NET](#tab/dotnet)

Dans Visual Studio, appuyez sur **F5** ou sélectionnez **Démarrer** pour commencer le débogage de l’application. Visual Studio restaure automatiquement les packages NuGet manquants si cette option est configurée. Pour en savoir plus, consultez [Installation et réinstallation de packages avec la restauration de packages](/nuget/consume-packages/package-restore#package-restore-overview).

Une fenêtre de console apparaît et l’application commence à s’exécuter. L’application charge l’image **HelloWorld.png** de la solution dans le compte de stockage. L’application vérifie que l’image s’est répliquée sur le point de terminaison RA-GZRS secondaire. Elle commence ensuite à télécharger l’image jusqu’à 999 fois. Chaque lecture est représentée par un **P** ou un **S**. **P** représente le point de terminaison principal et **S** le point de terminaison secondaire.

![Exécution de l’application console](media/storage-create-geo-redundant-storage/figure3.png)

Dans l’exemple de code, la tâche `RunCircuitBreakerAsync` dans le fichier `Program.cs` est utilisée pour télécharger une image du compte de stockage à l’aide de la méthode [DownloadToFileAsync](/dotnet/api/microsoft.azure.storage.blob.cloudblob.downloadtofileasync). Avant le téléchargement, une classe [OperationContext](/dotnet/api/microsoft.azure.cosmos.table.operationcontext) est définie. Le contexte d’opération définit les gestionnaires d’événements, qui se déclenchent quand un téléchargement se termine correctement ou si un téléchargement échoue et effectue une nouvelle tentative.

# <a name="python"></a>[Python](#tab/python)

Pour exécuter l’application sur un terminal ou une invite de commandes, accédez au répertoire **circuitbreaker.py**, puis entrez `python circuitbreaker.py`. L’application charge l’image **HelloWorld.png** de la solution dans le compte de stockage. L’application vérifie que l’image s’est répliquée sur le point de terminaison RA-GZRS secondaire. Elle commence ensuite à télécharger l’image jusqu’à 999 fois. Chaque lecture est représentée par un **P** ou un **S**. **P** représente le point de terminaison principal et **S** le point de terminaison secondaire.

![Exécution de l’application console](media/storage-create-geo-redundant-storage/figure3.png)

Dans l’exemple de code, la méthode `run_circuit_breaker` dans le fichier `circuitbreaker.py` est utilisée pour télécharger une image à partir du compte de stockage à l’aide de la méthode [get_blob_to_path](/python/api/azure-storage-blob/azure.storage.blob.baseblobservice.baseblobservice?view=azure-python-previous#get-blob-to-path-container-name--blob-name--file-path--open-mode--wb---snapshot-none--start-range-none--end-range-none--validate-content-false--progress-callback-none--max-connections-2--lease-id-none--if-modified-since-none--if-unmodified-since-none--if-match-none--if-none-match-none--timeout-none-).

La fonction Nouvelle tentative de l’objet de stockage est définie sur une stratégie linéaire de nouvelles tentatives. La fonction Nouvelle tentative indique s’il faut renouveler une requête et spécifie le nombre de secondes à attendre avant de renouveler la requête. Indiquez la valeur true pour **retry\_to\_secondary** si la requête doit être renvoyée à la base de données secondaire en cas d’échec de la requête à la base de données principale. Dans l’exemple d’application, une stratégie personnalisée de nouvelles tentatives est définie dans la fonction `retry_callback` de l’objet de stockage.

Avant le téléchargement, l’objet du service [retry_callback](/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient?view=azure-python) et la fonction [response_callback](/python/api/azure-storage-common/azure.storage.common.storageclient.storageclient?view=azure-python) sont définis. Ces fonctions définissent les gestionnaires d’événements qui se déclenchent quand un téléchargement se termine correctement ou si un téléchargement échoue et effectue une nouvelle tentative.

# <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Pour exécuter l’exemple, ouvrez une invite de commandes, accédez à l’exemple de dossier et entrez `node index.js`.

L’exemple crée un conteneur dans votre compte de stockage d’objets blob, il charge **HelloWorld.png** dans le conteneur, puis à plusieurs reprises vérifie si le conteneur et l’image ont été répliqués dans la région secondaire. Après la réplication, vous êtes invité à entrer **D** ou **Q** (suivi de Entrée) pour télécharger ou quitter. Vous devez obtenir un résultat semblable à l’exemple qui suit :

```
Created container successfully: newcontainer1550799840726
Uploaded blob: HelloWorld.png
Checking to see if container and blob have replicated to secondary region.
[0] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[1] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
...
[31] Container has not replicated to secondary region yet: newcontainer1550799840726 : ContainerNotFound
[32] Container found, but blob has not replicated to secondary region yet.
...
[67] Container found, but blob has not replicated to secondary region yet.
[68] Blob has replicated to secondary region.
Ready for blob download. Enter (D) to download or (Q) to quit, followed by ENTER.
> D
Attempting to download blob...
Blob downloaded from primary endpoint.
> Q
Exiting...
Deleted container newcontainer1550799840726
```

---

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

### <a name="net"></a>[.NET](#tab/dotnet)

### <a name="retry-event-handler"></a>Gestionnaire d’événements de nouvelle tentative

Le Gestionnaire d’événements `OperationContextRetrying` est appelé quand le téléchargement de l’image échoue et qu’une nouvelle tentative est définie. Si le nombre maximal de tentatives définies dans l’application est atteint, le paramètre [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) de la requête passe à `SecondaryOnly`. Ce paramètre oblige l’application à essayer de télécharger l’image à partir du point de terminaison secondaire. Cette configuration réduit le temps nécessaire pour demander l’image puisque les nouvelles tentatives ne sont pas indéfiniment effectuées sur le point de terminaison principal.

```csharp
private static void OperationContextRetrying(object sender, RequestEventArgs e)
{
    retryCount++;
    Console.WriteLine("Retrying event because of failure reading the primary. RetryCount = " + retryCount);

    // Check if we have had more than n retries in which case switch to secondary.
    if (retryCount >= retryThreshold)
    {

        // Check to see if we can fail over to secondary.
        if (blobClient.DefaultRequestOptions.LocationMode != LocationMode.SecondaryOnly)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.SecondaryOnly;
            retryCount = 0;
        }
        else
        {
            throw new ApplicationException("Both primary and secondary are unreachable. Check your application's network connection. ");
        }
    }
}
```

### <a name="request-completed-event-handler"></a>Gestionnaire d’événements de demande terminée

Le Gestionnaire d’événements `OperationContextRequestCompleted` est appelé quand le téléchargement de l’image est réussi. Si l’application utilise le point de terminaison secondaire, elle continue à utiliser ce point de terminaison jusqu’à 20 fois. Au bout de 20 fois, l’application redéfinit le paramètre [LocationMode](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions.locationmode) sur `PrimaryThenSecondary` et réessaie le point de terminaison principal. Si une requête réussit, l’application poursuit la lecture à partir du point de terminaison principal.

```csharp
private static void OperationContextRequestCompleted(object sender, RequestEventArgs e)
{
    if (blobClient.DefaultRequestOptions.LocationMode == LocationMode.SecondaryOnly)
    {
        // You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        //    then switch back to the primary and see if it's available now.
        secondaryReadCount++;
        if (secondaryReadCount >= secondaryThreshold)
        {
            blobClient.DefaultRequestOptions.LocationMode = LocationMode.PrimaryThenSecondary;
            secondaryReadCount = 0;
        }
    }
}
```

### <a name="python"></a>[Python](#tab/python)

### <a name="retry-event-handler"></a>Gestionnaire d’événements de nouvelle tentative

Le Gestionnaire d’événements `retry_callback` est appelé quand le téléchargement de l’image échoue et qu’une nouvelle tentative est définie. Si le nombre maximal de tentatives définies dans l’application est atteint, le paramètre [LocationMode](/python/api/azure-storage-common/azure.storage.common.models.locationmode?view=azure-python) de la requête passe à `SECONDARY`. Ce paramètre oblige l’application à essayer de télécharger l’image à partir du point de terminaison secondaire. Cette configuration réduit le temps nécessaire pour demander l’image puisque les nouvelles tentatives ne sont pas indéfiniment effectuées sur le point de terminaison principal.

```python
def retry_callback(retry_context):
    global retry_count
    retry_count = retry_context.count
    sys.stdout.write(
        "\nRetrying event because of failure reading the primary. RetryCount= {0}".format(retry_count))
    sys.stdout.flush()

    # Check if we have more than n-retries in which case switch to secondary
    if retry_count >= retry_threshold:

        # Check to see if we can fail over to secondary.
        if blob_client.location_mode != LocationMode.SECONDARY:
            blob_client.location_mode = LocationMode.SECONDARY
            retry_count = 0
        else:
            raise Exception("Both primary and secondary are unreachable. "
                            "Check your application's network connection.")
```

### <a name="request-completed-event-handler"></a>Gestionnaire d’événements de demande terminée

Le Gestionnaire d’événements `response_callback` est appelé quand le téléchargement de l’image est réussi. Si l’application utilise le point de terminaison secondaire, elle continue à utiliser ce point de terminaison jusqu’à 20 fois. Au bout de 20 fois, l’application redéfinit le paramètre [LocationMode](/python/api/azure-storage-common/azure.storage.common.models.locationmode?view=azure-python) sur `PRIMARY` et réessaie le point de terminaison principal. Si une requête réussit, l’application poursuit la lecture à partir du point de terminaison principal.

```python
def response_callback(response):
    global secondary_read_count
    if blob_client.location_mode == LocationMode.SECONDARY:

        # You're reading the secondary. Let it read the secondary [secondaryThreshold] times,
        # then switch back to the primary and see if it is available now.
        secondary_read_count += 1
        if secondary_read_count >= secondary_threshold:
            blob_client.location_mode = LocationMode.PRIMARY
            secondary_read_count = 0
```

### <a name="nodejs"></a>[Node.JS](#tab/nodejs)

Avec le kit SDK Node.js V10, les gestionnaires de rappel sont inutiles. À la place, l’exemple crée un pipeline configuré avec des options de nouvelle tentative et un point de terminaison secondaire. L’application peut ainsi basculer automatiquement vers le pipeline secondaire si elle ne parvient pas à atteindre vos données via le pipeline principal.

```javascript
const accountName = process.env.AZURE_STORAGE_ACCOUNT_NAME;
const storageAccessKey = process.env.AZURE_STORAGE_ACCOUNT_ACCESS_KEY;
const sharedKeyCredential = new SharedKeyCredential(accountName, storageAccessKey);

const primaryAccountURL = `https://${accountName}.blob.core.windows.net`;
const secondaryAccountURL = `https://${accountName}-secondary.blob.core.windows.net`;

const pipeline = StorageURL.newPipeline(sharedKeyCredential, {
  retryOptions: {
    maxTries: 3,
    tryTimeoutInMs: 10000,
    retryDelayInMs: 500,
    maxRetryDelayInMs: 1000,
    secondaryHost: secondaryAccountURL
  }
});
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans la première partie de la série, vous avez appris à rendre une application hautement disponible avec des comptes de stockage RA-GZRS.

Passez à la deuxième partie de la série pour apprendre à simuler un échec et à forcer votre application à utiliser le point de terminaison RA-GZRS secondaire.

> [!div class="nextstepaction"]
> [Simuler une défaillance lors de la lecture de données à partir de la région primaire](simulate-primary-region-failure.md)
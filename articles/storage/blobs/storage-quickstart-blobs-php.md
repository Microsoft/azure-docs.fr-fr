---
title: Guide de démarrage rapide Azure - Créer un objet blob dans un stockage d’objets à l’aide de PHP | Microsoft Docs
description: Apprenez rapidement à transférer des objets vers/à partir de Stockage Blob Azure avec PHP. Charger, télécharger et répertorier des objets blob de blocs dans un conteneur dans Stockage Blob Azure.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 11/14/2018
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.openlocfilehash: af35927abfa491891dffe2c2397d6daf8c1ccde2
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95543117"
---
#  <a name="transfer-objects-tofrom-azure-blob-storage-using-php"></a>Transférer des objets vers/à partir de Stockage Blob Azure avec PHP
Dans ce guide de démarrage rapide, vous apprenez à utiliser PHP pour charger, télécharger et lister des objets blob de blocs dans un conteneur de stockage blob Azure. 

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [storage-quickstart-prereq-include](../../../includes/storage-quickstart-prereq-include.md)]

Vérifiez que les prérequis suivants sont installés :

* [PHP](https://php.net/downloads.php)
* [SDK Stockage Azure pour PHP](https://github.com/Azure/azure-storage-php)

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application
L’[exemple d’application](https://github.com/Azure-Samples/storage-blobs-php-quickstart.git) utilisé dans ce démarrage rapide est une application PHP de base.  

Utilisez [git](https://git-scm.com/) pour télécharger une copie de l’application dans votre environnement de développement. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-php-quickstart.git
```

Cette commande clone le dépôt dans votre dossier git local. Pour ouvrir l’exemple d’application PHP, recherchez le dossier storage-blobs-php-quickstart, puis ouvrez le fichier phpqs.php.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage
Dans l’application, vous devez fournir le nom de votre compte de stockage et votre clé de compte pour créer l’instance **BlobRestProxy** associée à votre application. Il est recommandé de stocker ces identificateurs dans une variable d’environnement sur l’ordinateur local exécutant l’application. Utilisez l’un des exemples suivants, en fonction de votre système d’exploitation, pour créer la variable d’environnement. Remplacez les valeurs **youraccountname** et **youraccountkey** par votre nom de compte et votre clé de compte.

# <a name="linux"></a>[Linux](#tab/linux)

```bash
export ACCOUNT_NAME=<youraccountname>
export ACCOUNT_KEY=<youraccountkey>
```

# <a name="windows"></a>[Windows](#tab/windows)

```cmd
setx ACCOUNT_NAME=<youraccountname>
setx ACCOUNT_KEY=<youraccountkey>
```
---

## <a name="configure-your-environment"></a>Configurer votre environnement
Prenez le dossier dans votre dossier Git local et placez-le dans un répertoire pris en charge par votre serveur PHP. Ensuite, ouvrez une invite de commandes étendue à ce même répertoire, puis entrez : `php composer.phar install`

## <a name="run-the-sample"></a>Exécution de l'exemple
Cet exemple permet de créer un fichier de test dans le dossier '.'. L’exemple de programme charge le fichier de test sur Stockage Blob, liste les objets blob du conteneur, puis télécharge le fichier sous un nouveau nom. 

Exécutez l’exemple. La sortie suivante est un exemple de sortie retournée durant l’exécution de l’application :
  
```
Uploading BlockBlob: HelloWorld.txt
These are the blobs present in the container: HelloWorld.txt: https://myexamplesacct.blob.core.windows.net/blockblobsleqvxd/HelloWorld.txt

This is the content of the blob uploaded: Hello Azure!
```
Quand vous appuyez sur le bouton affiché, l’exemple de programme supprime le conteneur de stockage et les fichiers. Avant de continuer, recherchez les deux fichiers dans le dossier de votre serveur. Vous pouvez les ouvrir et constater qu’ils sont identiques.

Vous pouvez également utiliser un outil comme l’[Explorateur Stockage Azure](https://storageexplorer.com) pour afficher les fichiers dans Stockage Blob. L’Explorateur Stockage Azure est un outil multiplateforme gratuit qui vous permet d’accéder aux informations de votre compte de stockage. 

Une fois que vous avez vérifié les fichiers, appuyez sur n’importe quelle touche pour terminer la démonstration et supprimer les fichiers de test. Maintenant que vous avez compris l’exemple, ouvrez le fichier example.rb pour examiner le code. 

## <a name="understand-the-sample-code"></a>Comprendre l’exemple de code

Ensuite, nous allons parcourir l’exemple de code pas à pas pour vous montrer son fonctionnement.

### <a name="get-references-to-the-storage-objects"></a>Obtenir des références aux objets de stockage
La première chose à faire est de créer les références aux objets utilisés pour accéder et gérer Stockage Blob. Ces objets reposent les uns sur les autres, chacun est utilisé par le suivant dans la liste.

* Créez une instance de l’objet **BlobRestProxy** Stockage Azure pour configurer les informations d’identification de connexion. 
* Créez l’objet **BlobService** pointant vers le service BLOB de votre compte de stockage. 
* Créez l’objet **Container**, qui représente le conteneur auquel vous accédez. Les conteneurs sont utilisés pour organiser vos objets blob de la même façon que vous utilisez des dossiers pour organiser vos fichiers sur votre ordinateur.

Une fois que vous avez l’objet conteneur **blobClient**, vous pouvez créer l’objet blob **Block** qui pointe vers l’objet blob spécifique qui vous intéresse. Vous pouvez ensuite effectuer des opérations de chargement, de téléchargement et de copie.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur les noms des conteneurs et des objets blob, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

Dans cette section, vous configurez une instance du client Stockage Azure, instanciez l’objet du service blob, créez un conteneur et définissez les autorisations sur le conteneur pour que les objets blob soient publics. Le conteneur est appelé **quickstartblobs**. 

```PHP
    # Setup a specific instance of an Azure::Storage::Client
    $connectionString = "DefaultEndpointsProtocol=https;AccountName=".getenv('account_name').";AccountKey=".getenv('account_key');
    
    // Create blob client.
    $blobClient = BlobRestProxy::createBlobService($connectionString);
    
    # Create the BlobService that represents the Blob service for the storage account
    $createContainerOptions = new CreateContainerOptions();
    
    $createContainerOptions->setPublicAccess(PublicAccessType::CONTAINER_AND_BLOBS);
    
    // Set container metadata.
    $createContainerOptions->addMetaData("key1", "value1");
    $createContainerOptions->addMetaData("key2", "value2");

    $containerName = "blockblobs".generateRandomString();

    try    {
        // Create container.
        $blobClient->createContainer($containerName, $createContainerOptions);
```

### <a name="upload-blobs-to-the-container"></a>Charger des objets blob dans le conteneur

Stockage Blob prend en charge les objets blob de blocs, d’ajout et de pages. Comme les objets blob de blocs sont les plus couramment utilisés, nous les utilisons dans ce démarrage rapide.  

Pour charger un fichier dans un objet blob, récupérez le chemin complet du fichier en joignant le nom de répertoire au nom de fichier sur votre disque local. Vous pouvez ensuite charger le fichier sur le chemin spécifié à l’aide de la méthode **createBlockBlob()** . 

L’exemple de code prend un fichier local et le charge vers Azure. Le fichier est stocké en tant que **myfile** et le nom de l’objet blob en tant que **fileToUpload** dans le code. L'exemple suivant charge le fichier dans votre conteneur nommé **quickstartblobs**.

```PHP
    $myfile = fopen("HelloWorld.txt", "w") or die("Unable to open file!");
    fclose($myfile);

    # Upload file as a block blob
    echo "Uploading BlockBlob: ".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    
    $content = fopen($fileToUpload, "r");

    //Upload blob
    $blobClient->createBlockBlob($containerName, $fileToUpload, $content);
```

Pour effectuer une mise à jour partielle du contenu d’un objet blob de bloc, exécutez la méthode **createblocklist()** . Les objets blob de blocs peuvent atteindre une taille maximale de 4.7 To et peuvent représenter toutes sortes d’éléments allant des feuilles de calcul Excel aux fichiers vidéo volumineux. Les objets blob de pages sont principalement utilisés pour les fichiers VHD utilisés pour stocker des machines virtuelles IaaS. Les objets blob d’ajout sont utilisés pour la journalisation, par exemple, quand vous voulez écrire dans un fichier et continuer à ajouter d’autres informations. Les objets blob d’ajout doivent être utilisés dans un modèle enregistreur unique. La plupart des objets stockés dans Stockage Blob sont des objets blob de blocs.

### <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Vous pouvez obtenir la liste des fichiers du conteneur à l’aide de la méthode **listBlobs()** . Le code suivant récupère la liste des objets blob, puis effectue une itération sur ces derniers en affichant les noms des objets blob trouvés dans un conteneur.  

```PHP
    $listBlobsOptions = new ListBlobsOptions();
    $listBlobsOptions->setPrefix("HelloWorld");
    
    echo "These are the blobs present in the container: ";
    
    do{
        $result = $blobClient->listBlobs($containerName, $listBlobsOptions);
        foreach ($result->getBlobs() as $blob)
        {
            echo $blob->getName().": ".$blob->getUrl()."<br />";
        }
        
        $listBlobsOptions->setContinuationToken($result->getContinuationToken());
    } while($result->getContinuationToken());
```

### <a name="get-the-content-of-your-blobs"></a>Obtenir le contenu de vos objets blob

Obtenez le contenu de vos objets blob à l’aide de la méthode **getBlob()**. Le code suivant affiche le contenu de l’objet blob chargé dans une section précédente.

```PHP
    $blob = $blobClient->getBlob($containerName, $fileToUpload);
    fpassthru($blob->getContentStream());
```

### <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous n’avez plus besoin des objets blob chargés dans ce guide de démarrage rapide, vous pouvez supprimer l’intégralité du conteneur à l’aide de la méthode **deleteContainer()** . Si les fichiers créés ne sont plus nécessaires, utilisez la méthode **deleteBlob()** pour supprimer ces fichiers.

```PHP
    // Delete blob.
    echo "Deleting Blob".PHP_EOL;
    echo $fileToUpload;
    echo "<br />";
    $blobClient->deleteBlob($_GET["containerName"], $fileToUpload);

    // Delete container.
    echo "Deleting Container".PHP_EOL;
    echo $_GET["containerName"].PHP_EOL;
    echo "<br />";
    $blobClient->deleteContainer($_GET["containerName"]);

    //Deleting local file
    echo "Deleting file".PHP_EOL;
    echo "<br />";
    unlink($fileToUpload);   
```

## <a name="resources-for-developing-php-applications-with-blobs"></a>Ressources sur le développement d’applications PHP avec des objets blob

Consultez ces ressources supplémentaires sur le développement PHP avec le stockage Blob :

- Affichez, téléchargez et installez le [code source de la bibliothèque de client PHP](https://github.com/Azure/azure-storage-php) pour Stockage Azure sur GitHub.
- Explorez les [exemples de Stockage Blob](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=php&term=blob) écrits avec la bibliothèque de client PHP.

## <a name="next-steps"></a>Étapes suivantes
 
Dans ce guide de démarrage rapide, vous avez appris à transférer des fichiers entre un disque local et Stockage Blob Azure avec PHP. Pour en savoir plus sur l’utilisation de PHP, accédez à notre Centre de développement PHP.

> [!div class="nextstepaction"]
> [Centre de développement PHP](https://azure.microsoft.com/develop/php/)


Pour plus d’informations sur l’Explorateur Stockage et les objets blob, consultez [Gérer les ressources de stockage Blob Azure avec l’Explorateur Stockage](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
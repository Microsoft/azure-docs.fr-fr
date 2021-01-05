---
title: 'Démarrage rapide : Bibliothèque de client de stockage blob Azure v2.1 pour Python'
description: Dans le cadre de ce guide de démarrage rapide, vous allez créer un compte de stockage et un conteneur dans un stockage d’objets (blob). Ensuite, vous utilisez la bibliothèque de client de stockage v2.1 pour Python, afin de charger un objet blob dans Stockage Azure, de télécharger un objet blob et de répertorier les objets blob dans un conteneur.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 07/24/2020
ms.service: storage
ms.subservice: blobs
ms.topic: quickstart
ms.custom: seo-python-october2019, devx-track-python
ms.openlocfilehash: 34205caf03d2f2d7255f75ea6203c5572c4c429b
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95523300"
---
# <a name="quickstart-manage-blobs-with-python-v21-sdk"></a>Démarrage rapide : Gérer des objets blob avec le SDK Python v2.1

Dans ce guide de démarrage rapide, vous découvrez comment gérer des objets blob avec Python. Les objets blob sont des objets pouvant contenir de grandes quantités de texte ou de données binaires, notamment des images, des documents, des médias en streaming et des données d’archive. Vous allez charger, télécharger et lister des objets blob, et vous allez créer et supprimer des conteneurs.

> [!NOTE]
> Ce guide de démarrage rapide utilise une version héritée de la bibliothèque de client Stockage Blob Azure. Pour démarrer avec la dernière version, consultez [Démarrage rapide : Gérer les blobs avec le kit SDK Python v12](storage-quickstart-blobs-python.md).

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).
- Un compte de stockage Azure. [Créer un compte de stockage](../common/storage-account-create.md).
- [Python](https://www.python.org/downloads/)
- [SDK Stockage Azure pour Python](https://github.com/Azure/azure-sdk-for-python).

[!INCLUDE [storage-multi-protocol-access-preview](../../../includes/storage-multi-protocol-access-preview.md)]

## <a name="download-the-sample-application"></a>Téléchargement de l'exemple d'application

[L’exemple d’application](https://github.com/Azure-Samples/storage-blobs-python-quickstart.git) dans ce démarrage rapide est une application Python de base.  

Utilisez la commande [git](https://git-scm.com/) suivante pour télécharger l’application dans votre environnement de développement. 

```bash
git clone https://github.com/Azure-Samples/storage-blobs-python-quickstart.git 
```

Pour exécuter le programme Python, ouvrez le fichier *example.py* à la racine du dépôt.  

[!INCLUDE [storage-copy-account-key-portal](../../../includes/storage-copy-account-key-portal.md)]

## <a name="configure-your-storage-connection-string"></a>Configurer votre chaîne de connexion de stockage

Dans l’application, fournissez votre nom de compte de stockage et votre clé de compte pour créer un objet `BlockBlobService`.

1. Ouvrez le fichier *example.py* dans l’Explorateur de solutions de votre IDE.

1. Remplacez les valeurs `accountname` et `accountkey` par le nom et la clé de votre compte de stockage :

    ```python
    block_blob_service = BlockBlobService(
        account_name='accountname', account_key='accountkey')
    ```

1. Enregistrez et fermez le fichier.

## <a name="run-the-sample"></a>Exécution de l'exemple

L’exemple de programme crée un fichier de test dans votre dossier *Documents*, charge le fichier sur Stockage Blob, liste les objets blob du fichier, puis télécharge le fichier sous un nouveau nom.

1. Installez les dépendances :

    ```console
    pip install azure-storage-blob==2.1.0
    ```

1. Accédez à l’exemple d’application :

    ```console
    cd storage-blobs-python-quickstart
    ```

1. Exécutez l’exemple :

    ```console
    python example.py
    ```

    Vous pouvez voir des messages similaires à ce qui suit :
  
    ```output
    Temp file = C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    Uploading to Blob storage as blobQuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    List blobs in the container
             Blob name: QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078.txt

    Downloading blob to     C:\Users\azureuser\Documents\QuickStart_9f4ed0f9-22d3-43e1-98d0-8b2c05c01078_DOWNLOADED.txt
    ```

1. Avant de continuer, accédez à votre dossier *Documents* et recherchez les deux fichiers.

    * *QuickStart_\<universally-unique-identifier\>*
    * *QuickStart_\<universally-unique-identifier\>_DOWNLOADED*

1. Vous pouvez les ouvrir et constater qu’ils sont identiques.

    Vous pouvez également utiliser un outil comme l’[Explorateur Stockage Azure](https://storageexplorer.com). Il convient bien pour visualiser les fichiers dans Stockage Blob. L’Explorateur Stockage Azure est un outil multiplateforme gratuit qui vous permet d’accéder aux informations de votre compte de stockage. 

1. Une fois que vous avez regardé les fichiers, appuyez sur n’importe quelle touche pour terminer l’exemple et supprimer les fichiers de test.

## <a name="learn-about-the-sample-code"></a>Découvrir plus d’informations sur l’exemple de code

Comme vous savez maintenant ce que fait l’exemple, ouvrez le fichier *example.py* pour examiner le code.

### <a name="get-references-to-the-storage-objects"></a>Obtenir des références aux objets de stockage

Dans cette section, vous instanciez les objets, créez un conteneur, puis définissez des autorisations sur le conteneur pour que les objets blob soient publics. Vous appellerez le conteneur `quickstartblobs`. 

```python
# Create the BlockBlockService that the system uses to call the Blob service for the storage account.
block_blob_service = BlockBlobService(
    account_name='accountname', account_key='accountkey')

# Create a container called 'quickstartblobs'.
container_name = 'quickstartblobs'
block_blob_service.create_container(container_name)

# Set the permission so the blobs are public.
block_blob_service.set_container_acl(
    container_name, public_access=PublicAccess.Container)
```

Commencez par créer les références aux objets utilisés pour accéder au stockage Blob et le gérer. Ces objets reposent les uns sur les autres, chacun est utilisé par le suivant dans la liste.

* Instanciez l’objet **BlockBlobService**, qui pointe vers le service blob de votre compte de stockage. 

* Instanciez l’objet **CloudBlobContainer** qui représente le conteneur auquel vous accédez. Le système utilise des conteneurs pour organiser vos objets blob de la même façon que vous utilisez des dossiers pour organiser vos fichiers sur votre ordinateur.

Une fois que vous avez le conteneur d’objets blob cloud, instanciez l’objet **CloudBlockBlob** qui pointe vers l’objet blob qui vous intéresse. Vous pouvez ensuite charger, télécharger et copier l’objet blob si nécessaire.

> [!IMPORTANT]
> Les noms de conteneurs doivent être en minuscules. Pour plus d’informations sur les noms des conteneurs et des objets blob, consultez [Affectation de noms et références aux conteneurs, objets blob et métadonnées](/rest/api/storageservices/naming-and-referencing-containers--blobs--and-metadata).

### <a name="upload-blobs-to-the-container"></a>Charger des objets blob dans le conteneur

Stockage Blob prend en charge les objets blob de blocs, d’ajout et de pages. Les objets blob de blocs peuvent atteindre une taille maximale de 4.7 To et peuvent représenter toutes sortes d’éléments allant des feuilles de calcul Excel aux fichiers vidéo volumineux. Vous pouvez utiliser des objets blob d’ajout pour la journalisation quand vous voulez écrire dans un fichier, puis continuer à ajouter d’autres informations. Les objets blob de pages sont principalement utilisés pour les fichiers de disque dur virtuel qui servent de base à une infrastructure comme des machines virtuelles de service (machines virtuelles IaaS). Les objets blob de blocs sont les plus couramment utilisés. Ce guide de démarrage rapide utilise des objets blob de blocs.

Pour charger un fichier dans un objet blob, récupérez le chemin d’accès complet du fichier en joignant le nom de répertoire au nom de fichier sur votre disque local. Vous pouvez ensuite charger le fichier vers le chemin d’accès spécifié à l’aide de la méthode `create_blob_from_path`. 

L’exemple de code crée un fichier local que le système utilise pour le chargement et le téléchargement, stockant le fichier chargé par le système en tant que *chemin_complet_du_fichier* et le nom de l’objet blob en tant que *nom_de_fichier_local*. L’exemple suivant charge le fichier sur votre conteneur nommé `quickstartblobs` :

```python
# Create a file in Documents to test the upload and download.
local_path = os.path.expanduser("~\Documents")
local_file_name = "QuickStart_" + str(uuid.uuid4()) + ".txt"
full_path_to_file = os.path.join(local_path, local_file_name)

# Write text to the file.
file = open(full_path_to_file, 'w')
file.write("Hello, World!")
file.close()

print("Temp file = " + full_path_to_file)
print("\nUploading to Blob storage as blob" + local_file_name)

# Upload the created file, use local_file_name for the blob name.
block_blob_service.create_blob_from_path(
    container_name, local_file_name, full_path_to_file)
```

Il existe plusieurs méthodes de chargement que vous pouvez utiliser avec Stockage Blob. Par exemple, avec un flux de données en mémoire, vous pouvez utiliser la méthode `create_blob_from_stream` au lieu de la méthode `create_blob_from_path`. 

### <a name="list-the-blobs-in-a-container"></a>Créer la liste des objets blob d’un conteneur

Le code suivant crée un `generator` pour la méthode `list_blobs`. Le code boucle dans la liste des objets blob du conteneur et affiche leur nom sur la console.

```python
# List the blobs in the container.
print("\nList blobs in the container")
generator = block_blob_service.list_blobs(container_name)
for blob in generator:
    print("\t Blob name: " + blob.name)
```

### <a name="download-the-blobs"></a>Télécharger les objets blob


Téléchargez les objets blob sur votre disque local à l’aide de la méthode `get_blob_to_path`.
Le code suivant télécharge l’objet blob que vous avez chargé précédemment. Le système ajoute *_DOWNLOADED* au nom de l’objet blob, ce qui vous permet de voir les deux fichiers sur votre disque local.

```python
# Download the blob(s).
# Add '_DOWNLOADED' as prefix to '.txt' so you can see both files in Documents.
full_path_to_file2 = os.path.join(local_path, local_file_name.replace(
   '.txt', '_DOWNLOADED.txt'))
print("\nDownloading blob to " + full_path_to_file2)
block_blob_service.get_blob_to_path(
    container_name, local_file_name, full_path_to_file2)
```

### <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous n’avez plus besoin des objets blob chargés dans ce démarrage rapide, vous pouvez supprimer l’intégralité du conteneur à l’aide de la méthode `delete_container`. Si vous préférez supprimer les fichiers individuels, utilisez la méthode `delete_blob`.

```python
# Clean up resources. This includes the container and the temp files.
block_blob_service.delete_container(container_name)
os.remove(full_path_to_file)
os.remove(full_path_to_file2)
```

## <a name="resources-for-developing-python-applications-with-blobs"></a>Ressources sur le développement d’applications Python avec des objets blob

Pour plus d’informations sur le développement Python avec Stockage Blob, consultez ces ressources supplémentaires :

### <a name="binaries-and-source-code"></a>Fichiers binaires et code source

- Affichez, téléchargez et installez le [code source de la bibliothèque de client Python](https://github.com/Azure/azure-storage-python) pour Stockage Azure sur GitHub.

### <a name="client-library-reference-and-samples"></a>Référence et exemples de la bibliothèque de client

- Pour plus d’informations sur la bibliothèque de client Python, consultez les [bibliothèques Stockage Azure pour Python](/python/api/overview/azure/storage).
- Explorez les [exemples de Stockage Blob](https://azure.microsoft.com/resources/samples/?sort=0&service=storage&platform=python&term=blob) écrits avec la bibliothèque de client Python.

## <a name="next-steps"></a>Étapes suivantes
 
Dans ce démarrage rapide, vous avez appris à transférer des fichiers entre un disque local et un stockage Blob Azure avec Python. 

Pour plus d’informations sur l’Explorateur Stockage et les objets blob, consultez [Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage](../../vs-azure-tools-storage-explorer-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json).
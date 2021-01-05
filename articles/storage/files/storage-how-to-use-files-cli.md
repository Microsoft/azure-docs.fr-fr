---
title: Démarrage rapide pour la gestion de partages de fichiers Azure à l’aide d’Azure CLI
description: Ce démarrage rapide montre comment utiliser Azure CLI pour gérer Azure Files. Créez un groupe de ressources et un compte de stockage, puis créez et utilisez un partage de fichiers Azure.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/26/2018
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurecli
ms.openlocfilehash: dd07c09c4ed5be311bf5a485b9bff938c976a2f5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659231"
---
# <a name="quickstart-create-and-manage-azure-file-shares-using-azure-cli"></a>Démarrage rapide : créer et gérer des partages de fichiers Azure à l’aide d’Azure CLI
Ce guide vous explique les bases de l’utilisation des [partages de fichiers Azure](storage-files-introduction.md) avec Azure CLI. Le partage de fichiers Azure est similaire à d’autres partages de fichiers, mais est stocké dans le cloud et s’appuie sur la plateforme Azure. Les partages de fichiers Azure prennent en charge le protocole SMB (Server Message Block) standard, le protocole NFS (Network File System) (en préversion) et permettent de partager des fichiers sur plusieurs machines, applications et instances. 

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.4 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

- Par défaut, les commandes Azure CLI retournent le format JavaScript Object Notation (JSON). JSON est la méthode standard d’envoi et de réception de messages des API REST. Pour simplifier l’utilisation des réponses JSON, quelques exemples dans cet article utilisent le paramètre *query* sur des commandes d’Azure CLI. Ce paramètre utilise le [langage de requête JMESPath](http://jmespath.org/) pour analyser JSON. Pour en savoir plus sur l’utilisation des résultats des commandes d’Azure CLI en suivant le langage de requête JMESPath, consultez le [didacticiel JMESPath](http://jmespath.org/tutorial.html).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. Si vous n’avez pas déjà un groupe de ressources Azure, vous pouvez utiliser la commande [az group create](/cli/azure/group) pour en créer un. 

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *USA Ouest 2* :

```azurecli-interactive 
export resourceGroupName="myResourceGroup"
region="westus2"

az group create \
    --name $resourceGroupName \
    --location $region \
    --output none
```

## <a name="create-a-storage-account"></a>Créez un compte de stockage.
Un compte de stockage est un pool partagé de stockage dans lequel vous pouvez déployer des partages de fichiers Azure, ou d’autres ressources de stockage comme les objets blob ou les files d’attente. Un compte de stockage peut contenir un nombre illimité de partages de fichiers. Un partage peut stocker un nombre illimité de fichiers, dans les limites de capacité du compte de stockage.

L’exemple suivant crée un compte de stockage à l’aide de la commande [az storage account create](/cli/azure/storage/account). Les noms de compte de stockage doivent être uniques. Par conséquent, utilisez `$RANDOM` pour ajouter un numéro au nom pour le rendre unique.

```azurecli-interactive 
export storageAccountName="mystorageacct$RANDOM"

az storage account create \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --location $region \
    --kind StorageV2 \
    --sku Standard_LRS \
    --enable-large-file-share \
    --output none
```

> [!Note]  
> Les partages de plus de 5 Tio (maximum de 100 Tio par partage) sont uniquement disponibles dans les comptes de stockage localement redondant (LRS) et redondant interzone (ZRS). Pour créer un compte de stockage géo-redondant (GRS) ou géo-redondant interzone (GZRS), supprimez le paramètre `--enable-large-file-share`.

### <a name="get-the-storage-account-key"></a>Obtenir la clé du compte de stockage
Les clés de comptes de stockage contrôlent l’accès aux ressources dans un compte de stockage. Les clés sont automatiquement créées en même temps que le compte de stockage. Vous pouvez récupérer les clés de votre compte de stockage avec la commande [az storage account keys list](/cli/azure/storage/account/keys) : 

```azurecli-interactive 
export storageAccountKey=$(az storage account keys list \
    --resource-group $resourceGroupName \
    --account-name $storageAccountName \
    --query "[0].value" | tr -d '"')
```

## <a name="create-an-azure-file-share"></a>Crée un partage de fichiers Azure
Vous pouvez désormais créer votre premier partage de fichiers Azure. Créez des partages de fichiers avec la commande [az storage share create](/cli/azure/storage/share). Cet exemple crée un partage de fichiers Azure nommé *myshare* : 

```azurecli-interactive
shareName="myshare"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --quota 1024 \
    --enabled-protocols SMB \
    --output none
```

Le nom des partages ne peut contenir que des minuscules, des nombres et des traits d’union uniques (mais ne peut commencer par un trait d’union). Pour plus d’informations sur la façon de nommer des partages de fichiers et des fichiers, consultez [Affectation de noms et références aux partages, répertoires, fichiers et métadonnées](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Utiliser votre partage de fichiers Azure
Azure Files offre deux méthodes pour utiliser les fichiers et dossiers au sein de votre partage de fichiers Azure : le [protocole Server Message Block (SMB)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) de norme industrielle et le [protocole REST de fichier](/rest/api/storageservices/file-service-rest-api). 

Pour monter un partage de fichiers avec SMB, consultez le document suivant en fonction de votre système d’exploitation :
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)
- [Windows](storage-how-to-use-files-windows.md)

### <a name="using-an-azure-file-share-with-the-file-rest-protocol"></a>Utilisation d’un partage de fichiers Azure avec le protocole REST de fichier 
Vous pouvez utiliser directement le protocole REST de fichier (c’est-à-dire fabriquer vous-même les appels HTTP REST manuellement), mais la méthode la plus courante d’utiliser le protocole REST de fichier est d’utiliser l’interface Azure CLI, le [module Azure PowerShell](storage-how-to-use-files-powershell.md) ou le SDK de Stockage Azure. Toutes ces solutions offrent un bon wrapper autour du protocole REST de fichier dans le langage de script/programmation de votre choix.  

Dans la plupart des scénarios Azure Files, vous allez utiliser votre partage de fichiers Azure sur le protocole SMB, car cela vous permet d’utiliser les applications et outils existants que l’on s’attend à utiliser. Toutefois, utiliser l’API REST de fichier plutôt que SMB est avantageux pour plusieurs raisons, par exemple :

- Vous naviguez sur votre partage de fichiers à partir d’Azure Bash Cloud Shell (qui ne peut pas monter les partages de fichiers sur SMB).
- Vous tirez profit de ressources serverless, comme [Azure Functions](../../azure-functions/functions-overview.md) par exemple. 
- Vous créez un service à valeur ajoutée qui interagit avec de nombreux partages de fichiers Azure, par exemple en effectuant des sauvegardes ou des analyses antivirus.

Les exemples suivants montrent comment utiliser l’interface Azure CLI pour manipuler votre partage de fichiers Azure avec le protocole REST de fichier. 

### <a name="create-a-directory"></a>Créer un répertoire
Pour créer un répertoire nommé *myDirectory* à la racine de votre partage de fichiers Azure, exécutez la commande [`az storage directory create`](/cli/azure/storage/directory) :

```azurecli-interactive
az storage directory create \
   --account-name $storageAccountName \
   --account-key $storageAccountKey \
   --share-name $shareName \
   --name "myDirectory" \
   --output none
```

### <a name="upload-a-file"></a>Charger un fichier
Pour illustrer comment charger un fichier à l’aide de la commande [`az storage file upload`](/cli/azure/storage/file), créez d’abord un fichier à charger sur le lecteur de travail de Cloud Shell. Dans l’exemple suivant, vous créez puis chargez le fichier :

```azurecli-interactive
cd ~/clouddrive/
date > SampleUpload.txt

az storage file upload \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --source "SampleUpload.txt" \
    --path "myDirectory/SampleUpload.txt"
```

Si vous exécutez Azure CLI en local, remplacez `~/clouddrive` par un chemin existant sur votre machine.

Après avoir chargé le fichier, vous pouvez exécuter la commande [`az storage file list`](/cli/azure/storage/file) pour vérifier que le fichier a bien été chargé dans votre partage de fichiers Azure :

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory" \
    --output table
```

### <a name="download-a-file"></a>Téléchargement d’un fichier
Vous pouvez exécuter la commande [`az storage file download`](/cli/azure/storage/file) pour télécharger une copie du fichier chargé sur le disque de travail de Cloud Shell :

```azurecli-interactive
# Delete an existing file by the same name as SampleDownload.txt, if it exists, because you've run this example before
rm -f SampleDownload.txt

az storage file download \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --dest "SampleDownload.txt" \
    --output none
```

### <a name="copy-files"></a>Copie des fichiers
Une tâche courante consiste à copier les fichiers d’un partage de fichiers vers un autre. Pour illustrer cette fonctionnalité, créez un nouveau partage. Copiez le fichier que vous avez chargé dans ce nouveau partage à l’aide de la commande [az storage file copy](/cli/azure/storage/file/copy) : 

```azurecli-interactive
otherShareName="myshare2"

az storage share create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $otherShareName \
    --quota 1024 \
    --enabled-protocols SMB \
    --output none

az storage directory create \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --name "myDirectory2" \
    --output none

az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-share $shareName \
    --source-path "myDirectory/SampleUpload.txt" \
    --destination-share $otherShareName \
    --destination-path "myDirectory2/SampleCopy.txt"
```

Maintenant, si vous répertoriez les fichiers dans le nouveau partage, vous devez voir votre fichier copié :

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $otherShareName \
    --path "myDirectory2" \
    --output table
```

Bien que la commande `az storage file copy start` soit pratique pour les déplacements de fichiers entre les partages de fichiers Azure, pour les migrations et les déplacements de données plus importants, nous préconisons `rsync` sur macOS et Linux et `robocopy` sur Windows. Pour effectuer les déplacements de données, `rsync` et `robocopy` utilisent SMB plutôt que l’API FileREST.

## <a name="create-and-manage-share-snapshots"></a>Créer et gérer des instantanés de partage
Une autre tâche utile que vous pouvez effectuer avec un partage de fichiers Azure consiste à créer des instantanés de partage. Un instantané conserve une copie d’un point dans le temps d’un partage de fichiers Azure. Les instantanés de partage sont similaires à des technologies de systèmes d’exploitation que vous connaissez peut-être déjà :

- Les instantanés du [gestionnaire de volumes logiques (LVM)](https://en.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)#Basic_functionality) pour les systèmes Linux.
- Les instantanés du [système de fichiers Apple (APFS)](https://developer.apple.com/library/content/documentation/FileManagement/Conceptual/APFS_Guide/Features/Features.html) pour macOS.
- Le [service VSS](/windows/desktop/VSS/volume-shadow-copy-service-portal) pour les systèmes de fichiers Windows, comme NTFS et ReFS.
 
Vous pouvez créer un instantané de partage à l’aide de la commande [`az storage share snapshot`](/cli/azure/storage/share) :

```azurecli-interactive
snapshot=$(az storage share snapshot \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --query "snapshot" | tr -d '"')
```

### <a name="browse-share-snapshot-contents"></a>Parcourir le contenu des instantanés de partage
Vous pouvez parcourir le contenu d’un instantané de partage en envoyant l’horodatage de l’instantané de partage capturé dans la variable `$snapshot` dans la commande `az storage file list` :

```azurecli-interactive
az storage file list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --snapshot $snapshot \
    --output table
```

### <a name="list-share-snapshots"></a>Répertorier les instantanés de partage
Pour afficher la liste d’instantanés pris pour votre partage, utilisez la commande suivante :

```azurecli-interactive
az storage share list \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --include-snapshot \
    --query "[? name== '$shareName' && snapshot!=null].snapshot" \
    --output tsv
```

### <a name="restore-from-a-share-snapshot"></a>Restaurer à partir d’un instantané de partage
Vous pouvez restaurer un fichier avec la commande `az storage file copy start` que vous avez utilisée précédemment. Supprimez tout d’abord le fichier SampleUpload.txt que vous avez chargé, afin de pouvoir le restaurer à partir de l’instantané :

```azurecli-interactive
# Delete SampleUpload.txt
az storage file delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --share-name $shareName \
    --path "myDirectory/SampleUpload.txt" \
    --output none

# Build the source URI for a snapshot restore
URI=$(az storage account show \
    --resource-group $resourceGroupName \
    --name $storageAccountName \
    --query "primaryEndpoints.file" | tr -d '"')

URI=$URI$shareName"/myDirectory/SampleUpload.txt?sharesnapshot="$snapshot

# Restore SampleUpload.txt from the share snapshot
az storage file copy start \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --source-uri $URI \
    --destination-share $shareName \
    --destination-path "myDirectory/SampleUpload.txt"
```

### <a name="delete-a-share-snapshot"></a>Supprimer un instantané de partage
Vous pouvez supprimer un instantané de partage avec la commande [`az storage share delete`](/cli/azure/storage/share). Utilisez la variable qui contient la référence `$SNAPSHOT` au paramètre `--snapshot` :

```azurecli-interactive
az storage share delete \
    --account-name $storageAccountName \
    --account-key $storageAccountKey \
    --name $shareName \
    --snapshot $snapshot \
    --output none
```

## <a name="clean-up-resources"></a>Nettoyer les ressources
Lorsque vous avez terminé, vous pouvez exécuter la commande [`az group delete`](/cli/azure/group) pour supprimer le groupe de ressources et toutes les ressources associées : 

```azurecli-interactive 
az group delete --name $resourceGroupName
```

Vous pouvez également supprimer les ressources individuellement.
- Pour supprimer les partages de fichiers Azure que vous avez créés pour cet article :

    ```azurecli-interactive
    az storage share list \
            --account-name $storageAccountName \
            --account-key $storageAccountKey \
            --query "[].name" \
            --output tsv | \
        xargs -L1 bash -ec '\
            az storage share delete \
                --account-name "$storageAccountName" \
                --account-key "$storageAccountKey" \
                --name $0 \
                --delete-snapshots include \
                --output none'
    ```

- Pour supprimer le compte de stockage. (Ceci supprime implicitement les partages de fichiers Azure que vous avez créés et toute autre ressource de stockage que vous avez peut-être créée, un conteneur de stockage d’objets Blob Azure par exemple.)

    ```azurecli-interactive
    az storage account delete \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --yes
    ```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Qu’est-ce qu’Azure Files ?](storage-files-introduction.md)
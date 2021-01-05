---
title: Modifier le chemin d’accès d’objet BLOB par défaut
description: Apprenez à configurer une fonction Azure pour renommer un chemin d’accès de fichier d’objet blob par défaut, puis déplacer les objets blob vers un autre emplacement.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/16/2018
ms.author: alkohli
ms.openlocfilehash: 6af095c7abdb9aa61e57d543ff2ab2f9192dadc8
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011436"
---
# <a name="change-a-blob-path-from-the-default-path"></a>Modifier le chemin d’accès d’objet blob par défaut

Lorsque le service StorSimple Data Manager transforme les données, par défaut il place les objets blob transformés dans un conteneur de stockage comme spécifié lors de la création du référentiel cible. À mesure que les objets blob arrivent à cet emplacement, vous voudrez déplacer ces objets blob vers un autre emplacement. Cet article décrit comment configurer une fonction Azure pour renommer un chemin d’accès de fichier d’objet blob par défaut, puis déplacer les objets blob vers un autre emplacement.

## <a name="prerequisites"></a>Conditions préalables requises

Assurez-vous que vous avez une définition de travail correctement configurée dans votre service StorSimple Data Manager.

## <a name="create-an-azure-function"></a>Création d’une fonction Azure

Pour créer une fonction Azure, procédez comme suit :

1. Accédez au [portail Azure](https://portal.azure.com/).

2. Cliquez sur **+ Créer une ressource**. Dans la zone de **recherche**, saisissez **Function App**, puis appuyez sur **Entrée**. Sélectionnez et cliquez sur **Function App** dans la liste des applications affichées.

    ![Saisissez « Function App » dans la zone de recherche](./media/storsimple-data-manager-change-default-blob-path/search-function-app.png)

3. Cliquez sur **Créer**.

    ![Bouton « Créer » de la fenêtre Function App](./media/storsimple-data-manager-change-default-blob-path/create-function-app.png)

4. Dans le panneau de configuration **Function App**, procédez comme suit :

    1. Donnez un **Nom de l’application** unique.
    2. Sélectionnez **l’Abonnement** dans la liste déroulante. Cet abonnement doit être identique à celui associé à votre service StorSimple Data Manager.
    3. Sélectionnez **Créer nouveau**.
    4. Pour la liste déroulante **Plan d’hébergement**, sélectionnez **Plan de consommation**.
    5. Spécifiez un emplacement sur lequel s’exécute votre fonction. Vous souhaitez utiliser la même région que celle où se trouvent le service StorSimple Data Manager et le compte de stockage associé à la définition de travail.
    6. Sélectionnez un compte de stockage existant ou créez un nouveau compte de stockage. Un compte de stockage est utilisé en interne pour la fonction.

        ![Entrer les données de configuration de la nouvelle application Function App](./media/storsimple-data-manager-change-default-blob-path/function-app-parameters.png)

    7. Cliquez sur **Créer**. L’application Function App est créée.
     
        ![Application de fonction créée](./media/storsimple-data-manager-change-default-blob-path/function-app-created.png)

5. Sélectionnez **Fonctions**, puis cliquez sur **+ Nouvelle fonction**.

    ![Cliquez sur + Nouvelle fonction](./media/storsimple-data-manager-change-default-blob-path/create-new-function.png)

6. Sélectionnez **C#** pour le langage. Dans le tableau de vignettes de modèle, sélectionnez **C#** dans la vignette **QueueTrigger-CSharp**.

7. Dans le **Déclencheur de file d’attente** :

    1. Entrez un **nom** pour votre fonction.
    2. Dans la zone **Nom de file d’attente**, saisissez le nom de la définition du travail de transformation des données.
    3. Sous **Connexion au compte de stockage**, cliquez sur **Nouveau**. Dans la liste des comptes de stockage, sélectionnez le compte associé à votre définition de travail. Notez le nom de la connexion (en surbrillance). Ce nom est requis plus loin dans la fonction Azure.

        ![Créer une fonction C#](./media/storsimple-data-manager-change-default-blob-path/new-function-parameters.png)

    4. Cliquez sur **Créer**. La **Fonction** est créée.

     
10. Dans la fenêtre Fonction, exécutez le fichier _.csx_.

    ![Créer une fonction C# 2](./media/storsimple-data-manager-change-default-blob-path/new-function-run-csx.png)
    
    Procédez comme suit.

    1. Collez le code suivant :

        ```
        using System;
        using System.Configuration;
        using Microsoft.WindowsAzure.Storage.Blob;
        using Microsoft.WindowsAzure.Storage.Queue;
        using Microsoft.WindowsAzure.Storage;
        using System.Collections.Generic;
        using System.Linq;

        public static void Run(QueueItem myQueueItem, TraceWriter log)
        {
            CloudStorageAccount storageAccount = CloudStorageAccount.Parse(ConfigurationManager.AppSettings["STORAGE_CONNECTIONNAME"]);

            string storageAccUriEndswith = "windows.net/";
            string uri = myQueueItem.TargetLocation.Replace("%20", " ");
            log.Info($"Blob Uri: {uri}");

            // Remove storage account uri string
            uri = uri.Substring(uri.IndexOf(storageAccUriEndswith) + storageAccUriEndswith.Length);

            string containerName = uri.Substring(0, uri.IndexOf("/")); 

            // Remove container name string
            uri = uri.Substring(containerName.Length + 1);

            // Current blob path
            string blobName = uri; 

            string volumeName = uri.Substring(containerName.Length + 1);
            volumeName = uri.Substring(0, uri.IndexOf("/"));

            // Remove volume name string
            uri = uri.Substring(volumeName.Length + 1);

            string newContainerName = uri.Substring(0, uri.IndexOf("/")).ToLower();
            string newBlobName = uri.Substring(newContainerName.Length + 1);

            log.Info($"Container name: {containerName}");
            log.Info($"Volume name: {volumeName}");
            log.Info($"New container name: {newContainerName}");

            log.Info($"Blob name: {blobName}");
            log.Info($"New blob name: {newBlobName}");

            // Create the blob client.
            CloudBlobClient blobClient = storageAccount.CreateCloudBlobClient();

            // Container reference
            CloudBlobContainer container = blobClient.GetContainerReference(containerName);
            CloudBlobContainer newContainer = blobClient.GetContainerReference(newContainerName);
            newContainer.CreateIfNotExists();

            if(!container.Exists())
            {
                log.Info($"Container - {containerName} not exists");
                return;
            }

            if(!newContainer.Exists())
            {
                log.Info($"Container - {newContainerName} not exists");
                return;
            }

            CloudBlockBlob blob = container.GetBlockBlobReference(blobName);
            if (!blob.Exists())
            {
                // Skip to copy the blob to new container, if source blob doesn't exist
                log.Info($"The specified blob does not exist.");
                log.Info($"Blob Uri: {blob.Uri}");
                return;
            }

            CloudBlockBlob blobCopy = newContainer.GetBlockBlobReference(newBlobName);
            if (!blobCopy.Exists())
            {
                blobCopy.StartCopy(blob);
                // Delete old blob, after copy to new container
                blob.DeleteIfExists();
                log.Info($"Blob file path renamed completed successfully");
            }
            else
            {
                log.Info($"Blob file path renamed already done");
                // Delete old blob, if already exists.
                blob.DeleteIfExists();
            }
        }

        public class QueueItem
        {
            public string SourceLocation {get;set;}
            public long SizeInBytes {get;set;}
            public string Status {get;set;}
            public string JobID {get;set;}
            public string TargetLocation {get; set;}
        }

        ```

    2. Remplacez la valeur **STORAGE_CONNECTIONNAME** de la ligne 11 par la connexion à votre compte de stockage (étape 7c).

        ![Copier le nom de connexion de stockage](./media/storsimple-data-manager-change-default-blob-path/new-function-storage-connection-name.png)

    3. **Enregistrez** la fonction.

        ![Enregistrer la fonction](./media/storsimple-data-manager-change-default-blob-path/save-function.png)

12. Pour terminer la fonction, ajoutez un fichier de plus en procédant comme suit :

    1. Cliquez sur **Afficher les fichiers**.

       ![Lien « Afficher les fichiers »](./media/storsimple-data-manager-change-default-blob-path/view-files.png)

    2. Cliquez sur **+ Ajouter**.
        
        ![Ajouter une nouvelle fonction ajouter un fichier](./media/storsimple-data-manager-change-default-blob-path/new-function-add-file.png)
    
    3. Saisissez **project.json** et appuyez sur **Entrée**. Collez le code suivant dans le fichier **project.json** :

        ```
        {
        "frameworks": {
            "net46":{
            "dependencies": {
                "windowsazure.storage": "8.1.1"
            }
            }
        }
        }

        ```

    
    4. Cliquez sur **Enregistrer**.

        ![Nouveau projet de fonction json](./media/storsimple-data-manager-change-default-blob-path/new-function-project-json.png)

Vous avez créé une fonction Azure. Cette fonction est déclenchée à chaque fois qu’un nouvel objet blob est généré par le travail de transformation des données.

## <a name="next-steps"></a>Étapes suivantes

[Utilisez l’interface utilisateur de StorSimple Data Manager pour transformer vos données](storsimple-data-manager-ui.md)

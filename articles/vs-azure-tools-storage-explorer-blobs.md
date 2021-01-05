---
title: Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage | Microsoft Docs
description: Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage. Créer un conteneur d’objets BLOB, afficher le contenu d’un conteneur d’objets BLOB, supprimer ou copier un conteneur d’objets BLOB, et bien plus encore.
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: 2f09e545-ec94-4d89-b96c-14783cc9d7a9
ms.service: storage
ms.devlang: multiple
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 05/21/2019
ms.author: cawa
ms.openlocfilehash: 48f00d964a87790b8d8c9b1d8eceaed26d15199e
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95531965"
---
# <a name="manage-azure-blob-storage-resources-with-storage-explorer"></a>Gérer les ressources de Stockage Blob Azure avec l’Explorateur Stockage

## <a name="overview"></a>Vue d’ensemble

[Azure Blob Storage](./storage/blobs/storage-quickstart-blobs-dotnet.md) est un service permettant de stocker de gros volumes de données non structurées, telles que du texte ou des données binaires, accessibles depuis n’importe où dans le monde via HTTP ou HTTPS.
Vous pouvez utiliser le stockage d'objets blob pour exposer les données publiquement dans le monde ou pour le stockage privé de données d'applications. Dans cet article, vous apprenez à vous servir de l’Explorateur Stockage pour utiliser des conteneurs d’objets blob et des objets blob.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir suivre les étapes de cet article, vous devrez :

* [Télécharger et installer l’Explorateur Stockage](https://www.storageexplorer.com)
* [Vous connecter à un service ou un compte de stockage Azure](vs-azure-tools-storage-manage-with-storage-explorer.md#connect-to-a-storage-account-or-service)

## <a name="create-a-blob-container"></a>Création d’un conteneur d’objets blob

Tous les objets blob doivent résider dans un conteneur d’objets blob, c’est-à-dire un simple regroupement logique d’objets blob. Un compte peut contenir un nombre illimité de conteneurs, et chaque conteneur peut stocker un nombre illimité d’objets blob.

Les étapes suivantes expliquent comment créer un conteneur d’objets blob dans l’Explorateur Stockage.

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage dans lequel vous souhaitez créer le conteneur d’objets blob.
3. Cliquez avec le bouton droit sur **Conteneurs d’objets blob** puis, dans le menu contextuel, sélectionnez **Créer un conteneur d’objets blob**.

   ![Création de conteneurs d’objets blob - Menu contextuel][0]
4. Une zone de texte apparaît sous le dossier **Conteneurs d’objets blob** . Entrez le nom de votre conteneur d’objets blob. Consultez [Créer un conteneur](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) pour obtenir plus d’informations sur la liste des règles et restrictions applicables aux noms de conteneurs d’objets blob.

   ![Création de conteneurs d’objets blob - Zone de texte][1]
5. Appuyez sur **Entrée** pour créer le conteneur d’objets blob, ou sur **ÉCHAP** pour annuler. Une fois le conteneur créé, il apparaît sous le dossier **Conteneurs d’objets blob** correspondant au compte de stockage sélectionné.

   ![Conteneur d’objets blob créé][2]

## <a name="view-a-blob-containers-contents"></a>Affichage du contenu d’un conteneur d’objets blob

Les conteneurs d’objets blob contiennent des objets blob et des dossiers (qui peuvent eux-mêmes contenir des objets blob).

Les étapes suivantes expliquent comment voir le contenu d’un conteneur d’objets blob dans l’Explorateur Stockage :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob que vous souhaitez afficher.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Cliquez avec le bouton droit sur le conteneur d’objets blob que vous souhaitez afficher puis, dans le menu contextuel, sélectionnez **Ouvrir l’éditeur de conteneur d’objets blob**.
   Vous pouvez également double-cliquer sur le conteneur d’objets blob que vous souhaitez afficher.

   ![Ouvrir l’éditeur de conteneurs d’objets blob - Menu contextuel][19]
5. Le volet principal affiche le contenu du conteneur d’objets blob.

   ![Éditeur de conteneurs d’objets blob][3]

## <a name="delete-a-blob-container"></a>Suppression d’un conteneur d’objets blob

Vous pouvez facilement créer et supprimer des conteneurs d’objets blob selon vos besoins (pour savoir comment supprimer des objets blob, reportez-vous à la section [Gestion des objets blob dans un conteneur d’objets blob](#managing-blobs-in-a-blob-container).)

Les étapes suivantes expliquent comment supprimer un conteneur d’objets blob dans l’Explorateur Stockage :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob que vous souhaitez afficher.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Cliquez avec le bouton droit sur le conteneur d’objets blob que vous souhaitez supprimer puis, dans le menu contextuel, sélectionnez **Supprimer**.
   Vous pouvez également appuyer sur **Supprimer** pour supprimer le conteneur d’objets blob actuellement sélectionné.

   ![Supprimer un conteneur d’objets blob - Menu contextuel][4]
5. Cliquez sur **Oui** dans la boîte de dialogue de confirmation.

   ![Supprimer un conteneur d’objets blob - Confirmation][5]

## <a name="copy-a-blob-container"></a>Copie d’un conteneur d’objets blob

L’Explorateur Stockage vous permet de copier un conteneur d’objets blob dans le Presse-papiers, puis de le coller dans un autre compte de stockage. (pour savoir comment copier des objets blob, reportez-vous à la section [Gestion des objets blob dans un conteneur d’objets blob](#managing-blobs-in-a-blob-container).)

Les étapes suivantes expliquent comment copier un conteneur d’objets blob d’un compte de stockage à un autre.

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob que vous souhaitez copier.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Cliquez avec le bouton droit sur le conteneur d’objets blob que vous souhaitez afficher puis, dans le menu contextuel, sélectionnez **Copy Blob Container**(Copier le conteneur d’objets blob).

   ![Copier un conteneur d’objets blob - Menu contextuel][6]
5. Cliquez avec le bouton droit sur le compte de stockage cible dans lequel vous souhaitez coller le conteneur d’objets blob puis, dans le menu contextuel, sélectionnez **Paste Blob Container**(Coller le conteneur d’objets blob).

   ![Coller un conteneur d’objets blob - Menu contextuel][7]

## <a name="get-the-sas-for-a-blob-container"></a>Obtenir la signature d’accès partagé pour un conteneur d’objets blob

Une [signature d’accès partagé (SAP)](./storage/common/storage-sas-overview.md) fournit un accès délégué aux ressources de votre compte de stockage.
Cela vous permet d’octroyer à un client des autorisations d’accès limité à des objets de votre compte de stockage pendant une période donnée et avec un ensemble défini d’autorisations, sans partager les clés d’accès de votre compte.

Les étapes suivantes expliquent comment créer une signature d’accès partagé pour un conteneur d’objets blob :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob pour lequel vous souhaitez obtenir une SAP.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Cliquez avec le bouton droit sur le conteneur d’objets blob souhaité puis, dans le menu contextuel, sélectionnez **Get Shared Access Signature**(Obtenir une signature d’accès partagé).

   ![Obtenir une signature d’accès partagé - Menu contextuel][8]
5. Dans la boîte de dialogue **Signature d’accès partagé** , spécifiez la stratégie, les dates de début et d’expiration, le fuseau horaire et les niveaux d’accès de la ressource.

   ![Obtenir une signature d’accès partagé - Options][9]
6. Une fois les options SAP spécifiées, sélectionnez **Créer**.
7. Vous accédez alors à une deuxième boîte de dialogue **Signature d’accès partagé** dans laquelle vous pouvez visualiser le conteneur d’objets blob, ainsi que les URL et les chaînes de requête que vous pouvez utiliser pour accéder à la ressource de stockage.
   Sélectionnez **Copier** en regard de l’URL que vous souhaitez copier dans le Presse-papiers.

   ![Copier les URL de SAP][10]
8. Lorsque vous avez terminé, sélectionnez **Fermer**.

## <a name="manage-access-policies-for-a-blob-container"></a>Gestion des stratégies d’accès d’un conteneur d’objets blob

Les étapes suivantes montrent comment gérer (ajouter et supprimer) les stratégies d’accès d’un conteneur d’objet :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob pour lequel vous souhaitez gérer les stratégies d’accès.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Sélectionnez le conteneur d’objets blob souhaité puis, dans le menu contextuel, sélectionnez **Manage Access Policies**(Gérer les stratégies d’accès).

   ![Gérer les stratégies d’accès - Menu contextuel][11]
5. La boîte de dialogue **Stratégies d’accès** répertorie les stratégies d’accès déjà créées pour le conteneur d’objets blob sélectionné.

   ![Options de stratégie d’accès][12]
6. Suivez ces étapes en fonction de la tâche de gestion des stratégies d’accès :

   * **Ajouter une nouvelle stratégie d’accès** : sélectionnez **Ajouter**. Une fois la stratégie générée, la boîte de dialogue **Stratégies d’accès** affiche la stratégie d’accès que vous venez d’ajouter (avec les paramètres par défaut).
   * **Modifier une stratégie d’accès** : apportez les modifications souhaitées, puis cliquez sur **Enregistrer**.
   * **Supprimer une stratégie d’accès** : sélectionnez **Supprimer** en regard de la stratégie d’accès à supprimer.

## <a name="set-the-public-access-level-for-a-blob-container"></a>Définir le niveau d’accès public pour un conteneur d’objets blob

Par défaut, chaque conteneur d’objets blob est défini sur « No public access » (Aucun accès public).

Les étapes suivantes expliquent comment spécifier un niveau d’accès public pour un conteneur d’objets blob.

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob pour lequel vous souhaitez gérer les stratégies d’accès.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Sélectionnez le conteneur d’objets blob souhaité puis, dans le menu contextuel, sélectionnez **Set Public Access Level**(Définir le niveau d’accès public).

   ![Définir le niveau d’accès public - Menu contextuel][13]
5. Dans la boîte de dialogue **Set Container Public Access Level** (Définir le niveau d’accès public du conteneur), spécifiez le niveau d’accès souhaité.

   ![Définir le niveau d’accès public - Options][14]
6. Sélectionnez **Appliquer**.

## <a name="managing-blobs-in-a-blob-container"></a>Gestion des objets blob dans un conteneur d’objets blob

Une fois que vous avez créé un conteneur d’objets blob, vous pouvez effectuer de nombreuses tâches, par exemple charger un objet blob dans ce conteneur, télécharger un objet blob sur votre ordinateur local, ouvrir un objet blob sur votre ordinateur local, etc.

Les étapes suivantes expliquent comment gérer les objets blob (et les dossiers) dans un conteneur d’objets blob.

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez le compte de stockage contenant le conteneur d’objets blob que vous souhaitez gérer.
3. Développez le dossier **Conteneurs d’objets blob** du compte de stockage.
4. Double-cliquez sur le conteneur d’objets blob que vous souhaitez afficher.
5. Le volet principal affiche le contenu du conteneur d’objets blob.

   ![Afficher le conteneur d’objets blob][3]
6. Le volet principal affiche le contenu du conteneur d’objets blob.
7. Suivez ces étapes en fonction de la tâche que vous souhaitez effectuer :

   * **Télécharger des fichiers vers un conteneur d’objets blob**

     1. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**, puis **Télécharger des fichiers** dans le menu contextuel.

        ![Télécharger des fichiers - Menu][15]
     2. Dans la boîte de dialogue **Télécharger des fichiers**, sélectionnez le bouton des points de suspension ( **…** ) situé sur le côté droit de la zone **Fichiers** pour sélectionner les fichiers que vous souhaitez charger.

        ![Télécharger des fichiers - Options][16]
     3. Spécifiez le **Type d’objet blob**. Consultez [Créer un contenu](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) pour plus d’informations.
     4. Si vous le souhaitez, spécifiez un dossier cible dans lequel charger les fichiers sélectionnés. Si le dossier cible n’existe pas, il sera créé.
     5. Sélectionnez **Télécharger**.
   * **Télécharger un dossier dans un conteneur d’objets blob**

     1. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**, puis **Télécharger un dossier** dans le menu contextuel.

        ![Télécharger un dossier - Menu][17]
     2. Dans la boîte de dialogue **Télécharger un dossier**, sélectionnez le bouton des points de suspension ( **…** ) situé sur le côté droit de la zone **Dossier** pour sélectionner le dossier que vous souhaitez charger.

        ![Télécharger un dossier - Options][18]
     3. Spécifiez le **Type d’objet blob**. Consultez [Créer un contenu](storage/blobs/storage-quickstart-blobs-dotnet.md#create-a-container) pour plus d’informations.
     4. Si vous le souhaitez, spécifiez un dossier cible dans lequel charger le contenu du dossier sélectionné. Si le dossier cible n’existe pas, il sera créé.
     5. Sélectionnez **Télécharger**.
   * **Télécharger un objet blob sur votre ordinateur local**

     1. Sélectionnez l’objet blob que vous souhaitez télécharger.
     2. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**.
     3. Dans la boîte de dialogue **Specify where to save the downloaded blob** (Indiquer où enregistrer l’objet blob téléchargé), spécifiez l’emplacement dans lequel vous souhaitez enregistrer l’objet blob téléchargé ainsi que le nom que vous souhaitez lui donner.  
     4. Sélectionnez **Enregistrer**.
   * **Ouvrir un objet blob sur votre ordinateur local**

     1. Sélectionnez l’objet blob que vous souhaitez ouvrir.
     2. Dans la barre d’outils du volet principal, sélectionnez **Ouvrir**.
     3. L’objet blob est téléchargé et ouvert à l’aide de l’application associée au type de fichier sous-jacent de l’objet blob.
   * **Copier un objet blob dans le Presse-papiers**

     1. Sélectionnez l’objet blob que vous souhaitez copier.
     2. Dans la barre d’outils du volet principal, sélectionnez **Copier**.
     3. Dans le volet de gauche, accédez à un autre conteneur d’objets blob et double-cliquez dessus pour l’afficher dans le volet principal.
     4. Dans la barre d’outils du volet principal, sélectionnez **Coller** pour créer une copie de l’objet blob.
   * **Supprimer un blob.**

     1. Sélectionnez l’objet blob que vous souhaitez supprimer.
     2. Dans la barre d’outils du volet principal, sélectionnez **Supprimer**.
     3. Cliquez sur **Oui** dans la boîte de dialogue de confirmation.

## <a name="next-steps"></a>Étapes suivantes

* Consultez les [dernières notes de publication et vidéos de l’Explorateur Stockage](https://www.storageexplorer.com).
* Découvrez comment [créer des applications à l’aide d'objets blob, de tables, de files d’attente et de fichiers Azure](https://azure.microsoft.com/documentation/services/storage/).

[0]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-create-context-menu.png
[1]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create.png
[2]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-create-done.png
[3]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-editor.png
[4]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-context-menu.png
[5]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-delete-confirmation.png
[6]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-copy-context-menu.png
[7]: ./media/vs-azure-tools-storage-explorer-blobs/blob-containers-paste-context-menu.png
[8]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-context-menu.png
[9]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-options.png
[10]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-get-sas-urls.png
[11]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-context-menu.png
[12]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-manage-access-policies-options.png
[13]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-context-menu.png
[14]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-set-public-access-level-options.png
[15]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-menu.png
[16]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-files-options.png
[17]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-menu.png
[18]: ./media/vs-azure-tools-storage-explorer-blobs/blob-upload-folder-options.png
[19]: ./media/vs-azure-tools-storage-explorer-blobs/blob-container-open-editor-context-menu.png
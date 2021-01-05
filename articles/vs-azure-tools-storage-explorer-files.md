---
title: Utilisation de l’Explorateur Stockage avec le Stockage Fichier Azure | Microsoft Docs
description: Apprenez à utiliser l’Explorateur Stockage pour travailler avec des fichiers et des partages de fichiers.
services: storage
documentationcenter: na
author: cawaMS
manager: paulyuk
editor: ''
ms.assetid: ''
ms.service: storage
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: conceptual
ms.date: 03/09/2017
ms.author: cawa
ms.openlocfilehash: 84f6473c25a5be11eeda7cd2b311d93a7226a78c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96488389"
---
# <a name="using-storage-explorer-with-azure-file-storage"></a>Utilisation de l’Explorateur de stockage avec le Stockage Fichier Azure

Azure Stockage Fichier est un service qui propose des partages de fichiers dans le cloud en utilisant le protocole SMB standard. Les protocoles SMB 2.1 et SMB 3.0 sont pris en charge. Avec le stockage de fichiers Azure, vous pouvez migrer vers Azure des applications héritées qui s’appuient sur des partages de fichiers, rapidement et sans réécritures onéreuses. Vous pouvez utiliser Stockage Fichier pour exposer les données publiquement au monde ou pour le stockage privé de données d’applications. Dans cet article, vous allez apprendre à utiliser l’Explorateur Stockage pour travailler avec des fichiers et des partages de fichiers.

## <a name="prerequisites"></a>Prérequis

Pour pouvoir suivre les étapes de cet article, vous devrez :

- [Télécharger et installer l’Explorateur Stockage](https://www.storageexplorer.com/)

- [Vous connecter à un service ou un compte de stockage Azure](./vs-azure-tools-storage-manage-with-storage-explorer.md#connect-to-a-storage-account-or-service)

## <a name="create-a-file-share"></a>Créer un partage de fichiers

Tous les fichiers doivent résider dans un partage de fichiers, c’est-à-dire un simple regroupement logique de fichiers. Un compte peut contenir un nombre illimité de partages de fichiers, et chaque partage de fichiers peut stocker un nombre illimité de fichiers.

Les étapes suivantes expliquent comment créer un partage de fichiers dans l’Explorateur Stockage.

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage dans lequel vous souhaitez créer le partage de fichiers.

1. Cliquez avec le bouton droit sur **Partages de fichiers**, puis sélectionnez **Créer un partage de fichiers** dans le menu contextuel.

    ![Créer un partage de fichiers](media/vs-azure-tools-storage-explorer-files/image1.png)

1. Une zone de texte apparaît sous le dossier **Partages de fichiers**. Entrez le nom de votre partage de fichiers. Consultez la section relative aux [règles d’affectation des noms de partages de fichiers](./storage/blobs/storage-quickstart-blobs-dotnet.md) pour obtenir la liste des règles et restrictions applicables aux noms de partages de fichiers.

    ![Affectation d’un nom au partage](media/vs-azure-tools-storage-explorer-files/image2.png)

1. Appuyez ensuite sur **Entrée** pour créer le partage de fichiers, ou sur **ÉCHAP** pour annuler. Une fois le partage de fichiers créé, il apparaît sous le dossier **Partages de fichiers** correspondant au compte de stockage sélectionné.

    ![Nouveau partage](media/vs-azure-tools-storage-explorer-files/image3.png)

## <a name="view-a-file-shares-contents"></a>Afficher le contenu d’un partage de fichiers

Les partages de fichiers contiennent des fichiers et des dossiers (qui peuvent également contenir des fichiers).

Les étapes suivantes expliquent comment afficher le contenu d’un partage de fichiers dans l’Explorateur Stockage :

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers que vous souhaitez afficher.

1. Développez le dossier **Partages de fichiers** du compte de stockage.

1. Cliquez avec le bouton droit sur le partage de fichiers que vous souhaitez afficher puis, dans le menu contextuel, sélectionnez **Ouvrir**. Vous pouvez également double-cliquer sur le partage de fichiers que vous souhaitez afficher.

    ![Ouvrir le partage](media/vs-azure-tools-storage-explorer-files/image4.png)

1. Le volet principal affiche le contenu du partage de fichiers.
    
    ![Capture d’écran du volet principal pour un partage de fichiers dans l’Explorateur de stockage montrant le contenu du partage.](media/vs-azure-tools-storage-explorer-files/image5.png)

## <a name="delete-a-file-share"></a>Supprimer un partage de fichiers

Vous pouvez facilement créer et supprimer des partages de fichiers selon vos besoins. (pour savoir comment supprimer des fichiers, reportez-vous à la section [Gestion des fichiers dans un partage de fichiers](./vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container)).

Les étapes suivantes expliquent comment supprimer un partage de fichiers dans l’Explorateur Stockage :

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers que vous souhaitez afficher.

1. Développez le dossier **Partages de fichiers** du compte de stockage.

1. Cliquez avec le bouton droit sur le partage de fichiers que vous souhaitez supprimer puis, dans le menu contextuel, sélectionnez **Supprimer**. Vous pouvez également appuyer sur **Supprimer** pour supprimer le partage de fichiers actuellement sélectionné.

    ![DELETE](media/vs-azure-tools-storage-explorer-files/image6.png)

1. Cliquez sur **Oui** dans la boîte de dialogue de confirmation.
    
    ![Boîte de dialogue de confirmation](media/vs-azure-tools-storage-explorer-files/image7.png)

## <a name="copy-a-file-share"></a>Copier un partage de fichiers

L’Explorateur Stockage vous permet de copier un partage de fichiers dans le Presse-papiers, puis de coller ce partage de fichiers dans un autre compte de stockage. (pour savoir comment copier des fichiers, reportez-vous à la section [Gestion des fichiers dans un partage de fichiers](./vs-azure-tools-storage-explorer-blobs.md#managing-blobs-in-a-blob-container)).

Les étapes suivantes expliquent comment copier un partage de fichiers d’un compte de stockage à un autre.

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers que vous souhaitez copier.

1. Développez le dossier **Partages de fichiers** du compte de stockage.

1. Cliquez avec le bouton droit sur le partage de fichiers que vous souhaitez copier puis, dans le menu contextuel, sélectionnez **Copy File Share** (Copier le partage de fichiers).

    ![Copier le partage de fichiers](media/vs-azure-tools-storage-explorer-files/image8.png)

1. Cliquez avec le bouton droit sur le compte de stockage cible dans lequel vous souhaitez coller le partage de fichiers puis, dans le menu contextuel, sélectionnez **Paste File Share** (Coller le partage de fichiers).

    ![Coller le partage de fichiers](media/vs-azure-tools-storage-explorer-files/image9.png)

## <a name="get-the-sas-for-a-file-share"></a>Obtenir la signature d’accès partagé (SAP) pour un partage de fichiers

Une [signature d’accès partagé (SAP)](./storage/common/storage-sas-overview.md) fournit un accès délégué aux ressources de votre compte de stockage. Cela vous permet d’octroyer à un client des autorisations d’accès limité à des objets de votre compte de stockage pendant une période donnée et avec un ensemble défini d’autorisations, sans partager les clés d’accès de votre compte.

Les étapes suivantes expliquent comment créer une signature d’accès partagé pour un partage de fichiers :

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers pour lequel vous souhaitez obtenir une SAP.

1. Développez le dossier **Partages de fichiers** du compte de stockage.

1. Cliquez avec le bouton droit sur le partage de fichiers puis, dans le menu contextuel, sélectionnez **Get Shared Access Signature** (Obtenir une signature d’accès partagé).

    ![Obtenir une signature d’accès partagé](media/vs-azure-tools-storage-explorer-files/image10.png)

1. Dans la boîte de dialogue **Signature d’accès partagé** , spécifiez la stratégie, les dates de début et d’expiration, le fuseau horaire et les niveaux d’accès de la ressource.

    ![Boîte de dialogue SAP](media/vs-azure-tools-storage-explorer-files/image11.png)

1. Une fois les options SAP spécifiées, sélectionnez **Créer**.

1. Vous accédez alors à une deuxième boîte de dialogue **Signature d’accès partagé** dans laquelle vous pouvez visualiser le partage de fichiers, ainsi que les URL et les chaînes de requête que vous pouvez utiliser pour accéder à la ressource de stockage. Sélectionnez **Copier** en regard de l’URL que vous souhaitez copier dans le Presse-papiers.
    
    ![Deuxième boîte de dialogue SAP](media/vs-azure-tools-storage-explorer-files/image12.png)

1. Lorsque vous avez terminé, sélectionnez **Fermer**.

## <a name="manage-access-policies-for-a-file-share"></a>Gérer les stratégies d’accès d’un partage de fichiers

Les étapes suivantes montrent comment gérer (ajouter et supprimer) les stratégies d’accès d’un partage de fichiers : Les stratégies d’accès sont utilisées pour créer des URL SAP permettant d’accéder à la ressource du fichier de stockage pendant une période définie.

1. Ouvrez l’Explorateur de stockage.

1. Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers pour lequel vous souhaitez gérer les stratégies d’accès.

1. Développez le dossier **Partages de fichiers** du compte de stockage.

1. Sélectionnez le partage de fichiers souhaité puis, dans le menu contextuel, sélectionnez **Manage Access Policies** (Gérer les stratégies d’accès).

    ![Gérer les stratégies d’accès - Menu contextuel](media/vs-azure-tools-storage-explorer-files/image13.png)

1. La boîte de dialogue **Stratégies d’accès** répertorie les stratégies d’accès déjà créées pour le partage de fichiers sélectionné.
    
    ![Stratégies d’accès](media/vs-azure-tools-storage-explorer-files/image14.png)

1. Suivez ces étapes en fonction de la tâche de gestion des stratégies d’accès :
    
    - **Ajouter une nouvelle stratégie d’accès** : sélectionnez **Ajouter**. Une fois la stratégie générée, la boîte de dialogue **Stratégies d’accès** affiche la stratégie d’accès que vous venez d’ajouter (avec les paramètres par défaut).

    - **Modifier une stratégie d’accès** : apportez les modifications souhaitées, puis cliquez sur **Enregistrer**.

    - **Supprimer une stratégie d’accès** : sélectionnez **Supprimer** en regard de la stratégie d’accès à supprimer.

1. Créez une URL de SAP à l’aide de la stratégie d’accès que vous avez élaborée précédemment :
    
    ![Obtenir une SAP](media/vs-azure-tools-storage-explorer-files/image15.png)
    
    ![Propriétés et nom de la SAP](media/vs-azure-tools-storage-explorer-files/image16.png)

## <a name="managing-files-in-a-file-share"></a>Gestion des fichiers dans un partage de fichiers

Une fois que vous avez créé un partage de fichiers, vous pouvez effectuer de nombreuses tâches, par exemple charger un fichier dans ce partage de fichiers, télécharger un fichier sur votre ordinateur local, ouvrir un fichier sur votre ordinateur local, etc.

Les étapes suivantes expliquent comment gérer les fichiers (et les dossiers) dans un partage de fichiers.

1.  Ouvrez l’Explorateur de stockage.

1.  Dans le volet gauche, développez le compte de stockage contenant le partage de fichiers que vous souhaitez gérer.

1.  Développez le dossier **Partages de fichiers** du compte de stockage.

1.  Double-cliquez sur le partage de fichiers que vous souhaitez afficher.

1.  Le volet principal affiche le contenu du partage de fichiers.

    ![Capture d’écran du volet principal pour le partage de fichiers myazurefileshare dans l’Explorateur de stockage, montrant le contenu du partage avec le premier dossier sélectionné.](media/vs-azure-tools-storage-explorer-files/image17.png)

1.  Le volet principal affiche le contenu du partage de fichiers.

1.  Suivez ces étapes en fonction de la tâche que vous souhaitez effectuer :

    - **Charger des fichiers dans un partage de fichiers**

        a.  Dans la barre d’outils du volet principal, sélectionnez **Télécharger**, puis **Télécharger des fichiers** dans le menu contextuel.

        ![Charger des fichiers](media/vs-azure-tools-storage-explorer-files/image18.png)
        
        b. Dans la boîte de dialogue **Télécharger des fichiers**, sélectionnez le bouton des points de suspension ( **…** ) situé sur le côté droit de la zone **Fichiers** pour sélectionner les fichiers que vous souhaitez charger.

        ![Ajout de fichiers](media/vs-azure-tools-storage-explorer-files/image19.png)

        c. Sélectionnez **Télécharger**.

    - **Charger un dossier dans un partage de fichiers**
        
        a. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**, puis **Télécharger un dossier** dans le menu contextuel.

        ![Télécharger un dossier - Menu](media/vs-azure-tools-storage-explorer-files/image20.png)

        b. Dans la boîte de dialogue **Télécharger un dossier**, sélectionnez le bouton des points de suspension ( **…** ) situé sur le côté droit de la zone **Dossier** pour sélectionner le dossier que vous souhaitez charger.

        c. Si vous le souhaitez, spécifiez un dossier cible dans lequel charger le contenu du dossier sélectionné. Si le dossier cible n’existe pas, il sera créé.

        d. Sélectionnez **Télécharger**.

    - **Télécharger un fichier sur votre ordinateur local**
        
        a. Sélectionnez le fichier que vous souhaitez télécharger.
        
        b. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**.
        
        c. Dans la boîte de dialogue **Specify where to save the downloaded file** (Indiquer où enregistrer le fichier téléchargé), spécifiez l’emplacement dans lequel vous souhaitez enregistrer le fichier téléchargé ainsi que le nom que vous souhaitez lui donner.

        d. Sélectionnez **Enregistrer**.

    - **Ouvrir un fichier sur votre ordinateur local**
        
        a.  Sélectionnez le fichier que vous souhaitez ouvrir.
        
        b.  Dans la barre d’outils du volet principal, sélectionnez **Ouvrir**.
        
        c.  Le fichier est téléchargé et ouvert à l’aide de l’application associée au type de fichier sous-jacent du fichier.

    - **Copier un fichier dans le Presse-papiers**

        a. Sélectionnez le fichier que vous souhaitez copier.

        b. Dans la barre d’outils du volet principal, sélectionnez **Copier**.

        c. Dans le volet de gauche, accédez à un autre partage de fichiers et double-cliquez dessus pour l’afficher dans le volet principal.

        d. Dans la barre d’outils du volet principal, sélectionnez **Coller** pour créer une copie du fichier.

    - **Supprimer un fichier**

        a. Sélectionnez le fichier que vous souhaitez supprimer.

        b. Dans la barre d’outils du volet principal, sélectionnez **Supprimer**.

        c. Cliquez sur **Oui** dans la boîte de dialogue de confirmation.

## <a name="next-steps"></a>Étapes suivantes

- Consultez les [dernières notes de publication et vidéos de l’Explorateur Stockage](https://www.storageexplorer.com/).

- Découvrez comment [créer des applications à l’aide d'objets blob, de tables, de files d’attente et de fichiers Azure](https://azure.microsoft.com/documentation/services/storage/).
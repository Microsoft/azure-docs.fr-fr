---
title: Gérer les ressources Data Lake Storage Gen1 – Explorateur Stockage Azure
description: Découvrez comment accéder aux données et ressources Azure Data Lake Storage Gen1 et les gérer dans l’Explorateur Stockage Azure.
author: jejiang
ms.service: data-lake-store
ms.topic: how-to
ms.date: 02/05/2018
ms.author: jejiang
ms.openlocfilehash: 7f251e6ba2d94c0fcede3387ac12461951de40f1
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92108743"
---
# <a name="manage-azure-data-lake-storage-gen1-resources-by-using-storage-explorer"></a>Gérer les ressources Azure Data Lake Storage Gen1 en utilisant l’Explorateur Stockage

[Azure Data Lake Storage Gen1](./data-lake-store-overview.md) est un service destiné à stocker de grandes quantités de données non structurées, telles que des données de texte ou binaires. Vous pouvez accéder aux données en tout lieu via HTTP ou HTTPS. Dans l’Explorateur Stockage Azure, Data Lake Storage Gen1 vous permet d’accéder aux ressources et aux données Data Lake Storage Gen1 et de les gérer. Il en va de même avec d’autres entités Azure telles que les objets blob et les files d’attente. À présent, vous pouvez utiliser le même outil pour gérer vos différentes entités Azure au même endroit.

Autre avantage : vous n’avez pas besoin de posséder une autorisation d’abonnement pour gérer les données Data Lake Storage Gen1. Dans l’Explorateur Stockage, vous pouvez joindre le chemin d’accès Data Lake Storage Gen1 au nœud **Local and Attached** (Local et attaché) tant qu’un utilisateur l’autorise.

## <a name="prerequisites"></a>Conditions préalables requises

Pour effectuer les étapes indiquées dans cet article, vous avez besoin des éléments requis suivants :

* Un abonnement Azure. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial).
* Un compte Data Lake Storage Gen1. Pour savoir comment en créer un, voir [Prise en main d’Azure Data Lake Storage Gen1](./data-lake-store-get-started-portal.md).

## <a name="install-storage-explorer"></a>Installer l’Explorateur Stockage

Installez la version la plus récente de l’Explorateur Stockage Azure à partir de la [page web du produit](https://azure.microsoft.com/features/storage-explorer/). Vous pouvez l’installer sur Windows, Linux et Mac.

## <a name="connect-to-an-azure-subscription"></a>Connexion à un abonnement Azure

1. Dans l’Explorateur Stockage, sélectionnez l’icône de plug-in sur la gauche.

   ![Icône de plug-in](./media/data-lake-store-in-storage-explorer/plug-in-icon.png)

1. Sélectionnez **Ajouter un compte Azure**, puis **Connexion**.

   ![Boîte de dialogue « Connexion au stockage Azure »](./media/data-lake-store-in-storage-explorer/connect-to-azure-subscription.png)

1. Dans la boîte de dialogue de **connexion à votre compte**, saisissez vos informations d’identification Azure.

    ![Boîte de dialogue de connexion à Azure](./media/data-lake-store-in-storage-explorer/sign-in.png)

1. Sélectionnez votre abonnement dans la liste, puis **Appliquer**.

    ![Informations d’abonnement et bouton « Appliquer »](./media/data-lake-store-in-storage-explorer/apply-subscription.png)

    Le volet **EXPLORATEUR** se met à jour et affiche les comptes de l’abonnement sélectionné.

    ![Liste de comptes](./media/data-lake-store-in-storage-explorer/account-list.png)

Vous êtes connecté à Data Lake Storage Gen1 dans votre abonnement Azure.

## <a name="connect-to-data-lake-storage-gen1"></a>Se connecter à Data Lake Storage Gen1

Vous pouvez accéder aux ressources qui n’existent pas dans votre abonnement si quelqu’un vous fournit l’URI des ressources. Une fois connecté, vous pouvez vous connecter à Data Lake Storage Gen1 à l’aide de l’URI.

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez **Local and Attached** (Local et attaché).
3. Cliquez avec le bouton droit sur **Data Lake Store**, puis sélectionnez **Connexion à Data Lake Store**.

      ![« Connexion à Data Lake Store » dans le menu contextuel](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-uri-attach.png)

4. Entrez l’URI. L’outil accède à l’emplacement de l’URL que vous venez d’entrer.

      ![Boîte de dialogue « Connexion à Data Lake Store », avec la zone de texte permettant d’entrer l’URI](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-uri-attach-dialog.png)

      ![Résultat de la connexion à Data Lake Storage Gen1](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-attach-finish.png)

## <a name="view-a-data-lake-storage-gen1-accounts-contents"></a>Afficher le contenu d’un compte Data Lake Storage Gen1

Les ressources d’un compte Data Lake Storage Gen1 contiennent des fichiers et dossiers.

La procédure suivante indique comment afficher le contenu d’un compte Data Lake Storage Gen1 dans l’Explorateur Stockage :

1. Ouvrez l’Explorateur de stockage.
2. Dans le volet gauche, développez l’abonnement qui contient le compte Data Lake Storage Gen1 que vous souhaitez afficher.
3. Développez **Data Lake Store**.
4. Cliquez avec le bouton droit sur le nœud du compte Data Lake Storage Gen1 que vous souhaitez afficher, puis sélectionnez **Ouvrir**. Vous pouvez également double-cliquer sur le compte Data Lake Storage Gen1 pour l’ouvrir.

   Le volet principal affiche le contenu du compte Data Lake Storage Gen1.

   ![Volet principal avec une liste de dossiers](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-toolbar-mainpane.png)

## <a name="manage-resources-in-data-lake-storage-gen1"></a>Gérer des données dans Data Lake Storage Gen1

Vous pouvez gérer les ressources Data Lake Storage Gen1 en effectuant les opérations suivantes :

* rechercher les ressources Data Lake Storage Gen1 dans plusieurs comptes Data Lake Storage Gen1 ;  
* utiliser une chaîne de connexion pour vous connecter à Data Lake Storage Gen1 et en assurer directement la gestion ;
* afficher les ressources Data Lake Storage Gen1 partagées par d’autres utilisateurs via une liste de contrôle d’accès dans **Local and Attached** (Local et attaché) ;
* effectuer des opérations CRUD sur fichier/dossier : prise en charge de fichiers à sélection multiple et de dossiers récursifs ;
* faire glisser, déposer et ajouter un dossier pour accéder rapidement aux emplacements récents ; cette opération permet de bénéficier d’une expérience similaire à celle de l’Explorateur de fichiers ;
* copier et ouvrir le lien hypertexte Data Lake Storage Gen1 en un clic dans l’Explorateur Stockage ;
* afficher le journal d’activité dans le volet inférieur droit pour afficher l’état de l’activité ;
* afficher les propriétés des fichiers et les statistiques des dossiers.

## <a name="manage-resources-in-azure-storage-explorer"></a>Gérer les ressources dans l’Explorateur Stockage Azure

Après avoir créé un compte Data Lake Storage Gen1, vous pouvez :

* charger des fichiers et des dossiers, télécharger des fichiers et des dossiers, et ouvrir des ressources sur votre ordinateur local ;
* épingler des éléments à l’**accès rapide**, créer un nouveau dossier, copier une URL et sélectionner tout ;
* copier et coller, renommer, supprimer, obtenir des statistiques sur les dossiers et actualiser les éléments.

Les éléments suivants indiquent comment gérer les ressources au sein d’un compte Data Lake Storage Gen1. Suivez les étapes correspondant à la tâche que vous souhaitez effectuer.

### <a name="upload-files"></a>Charger des fichiers

1. Dans la barre d’outils du volet principal, sélectionnez **Charger**, puis **Charger des fichiers** dans le menu déroulant.

   ![Élément de menu « Charger des fichiers »](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-files-menu.png)

2. Dans la boîte de dialogue **Sélectionner les fichiers à charger**, sélectionnez les fichiers à charger.

   ![Boîte de dialogue de chargement de fichiers](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-files-dialog.png)

3. Sélectionnez **Ouvrir** pour commencer le chargement.

### <a name="upload-a-folder"></a>Charger un dossier

1. Dans la barre d’outils du volet principal, sélectionnez **Charger**, puis **Charger un dossier** dans le menu déroulant.

   ![Élément de menu « Charger un dossier »](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-menu.png)

2. Dans la boîte de dialogue **Sélectionner le dossier à charger**, sélectionnez le dossier à charger. Puis cliquez sur **Sélectionner un dossier**.

   ![Boîte de dialogue de chargement de dossiers](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-dialog.png)

   Le chargement commence.

   ![Boîte de dialogue avec le chargement en cours](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-upload-folder-drag.png)

> [!NOTE]
> Vous pouvez directement faire glisser les dossiers et fichiers sur un ordinateur local pour démarrer le chargement.

### <a name="download-folders-or-files-to-your-local-computer"></a>Télécharger des dossiers ou des fichiers sur votre ordinateur local

1. Sélectionnez les dossiers ou fichiers que vous souhaitez télécharger.
2. Dans la barre d’outils du volet principal, sélectionnez **Télécharger**.
3. Dans la boîte de dialogue **Select a folder to save the downloaded files into** (Sélectionner un dossier pour y enregistrer les fichiers téléchargés), spécifiez l’emplacement et le nom.
4. Sélectionnez **Enregistrer**.

### <a name="open-a-folder-or-file-from-your-local-computer"></a>Ouvrir un dossier ou un fichier à partir de votre ordinateur local

1. Sélectionnez le dossier ou le fichier que vous souhaitez ouvrir.
2. Dans la barre d’outils du volet principal, sélectionnez **Ouvrir**. Ou cliquez avec le bouton droit sur le dossier ou le fichier sélectionné, puis sélectionnez **Ouvrir** dans le menu contextuel.

Le fichier est téléchargé et ouvert à l’aide de l’application associée au type sous-jacent du fichier. Le dossier est également ouvert dans le volet principal.

![Fichier ouvert](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-open.png)

### <a name="copy-folders-or-files-to-the-clipboard"></a>Copier des dossiers ou des fichiers dans le Presse-papiers

1. Sélectionnez les dossiers ou fichiers que vous souhaitez copier.
2. Dans la barre d’outils du volet principal, sélectionnez **Copier**. Ou cliquez avec le bouton droit sur les dossiers ou fichiers sélectionnés, puis sélectionnez **Copier** dans le menu contextuel.
3. Dans le volet gauche, accédez à un autre compte Data Lake Storage Gen1, puis double-cliquez dessus pour l’afficher dans le volet principal.
4. Dans la barre d’outils du volet principal, sélectionnez **Coller** pour en créer une copie. Ou sélectionnez **Coller** dans le menu contextuel de destination.

![Sélections pour la copie d’un dossier](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-copy-paste.png)

> [!NOTE]
> Les opérations de Copier/Coller entre différents types de stockage ne sont pas prises en charge. Vous pouvez copier des dossiers ou fichiers Data Lake Storage Gen1 et les coller dans un autre compte Data Lake Storage Gen1, mais vous *ne pouvez pas* copier des dossiers ou fichiers Data Lake Storage Gen1 et les coller dans le stockage Blob Azure, et inversement.
>
> Pour l’opération de Copier/Coller, téléchargez les dossiers ou fichiers sur l’ordinateur local, puis chargez-les vers l’emplacement de destination. L’outil n’effectue *pas* l’action sur le serveur principal. L’exécution de l’opération Copier/Coller sur les fichiers volumineux est lente. L’optimisation de l’opération Copier/Déplacer des fichiers hautes performances est en cours.

### <a name="delete-folders-or-files"></a>Supprimer des dossiers ou des fichiers

1. Sélectionnez les dossiers ou fichiers que vous souhaitez supprimer.
2. Dans la barre d’outils du volet principal, sélectionnez **Supprimer**. Ou cliquez avec le bouton droit sur les dossiers ou fichiers sélectionnés, puis sélectionnez **Supprimer** dans le menu contextuel.
3. Cliquez sur **Oui** dans la boîte de dialogue de confirmation.

![Sélections pour la suppression d’un dossier](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-delete.png)

### <a name="pin-to-quick-access"></a>Épingler sur Accès rapide

1. Sélectionnez le dossier à épingler.
2. Dans la barre d’outils du volet principal, sélectionnez **Épingler sur Accès rapide**.

   Dans le volet gauche, le dossier sélectionné est ajouté au nœud **Accès rapide**.

   ![Sélections pour l’épinglage d’un dossier sur « Accès rapide »](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-quick-access.png)

Une fois qu’un dossier a été épinglé au nœud **Accès rapide**, vous pouvez facilement accéder aux ressources.

### <a name="use-deep-links"></a>Utiliser les liens ciblés

Si vous disposez d’une URL, vous pouvez l’entrer dans le chemin d’adresse de l’Explorateur de fichiers ou d’un navigateur. Storage Explorer.exe s’exécute ensuite automatiquement pour accéder à l’emplacement de l’URL.

![Lien ciblé dans l’Explorateur de fichiers](./media/data-lake-store-in-storage-explorer/storageexplorer-adls-deep-link.png)

## <a name="next-steps"></a>Étapes suivantes

* Consultez les [dernières notes de publication et vidéos de l’Explorateur Stockage](https://www.storageexplorer.com).
* Découvrez comment [gérer Azure Cosmos DB dans l’Explorateur Stockage Azure](../cosmos-db/storage-explorer.md).
* [Prise en main de l’Explorateur Stockage](../vs-azure-tools-storage-manage-with-storage-explorer.md).
* [Prise en main d’Azure Data Lake Storage Gen1](./data-lake-store-overview.md)
* Visionnez une [vidéo YouTube pour découvrir comment utiliser Azure Cosmos DB dans l’Explorateur Stockage Azure](https://www.youtube.com/watch?v=iNIbg1DLgWo&feature=youtu.be).
---
title: Guide de démarrage rapide pour la gestion des partages de fichiers Azure avec le portail Azure
description: Découvrez comment créer et gérer les partages de fichiers Azure dans le portail Azure. Créez un compte de stockage, un partage de fichiers Azure et utilisez votre partage de fichiers Azure.
author: roygara
ms.service: storage
ms.topic: quickstart
ms.date: 10/18/2018
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 6a88124397812f7599ce54b46b23d22e626cf520
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629816"
---
# <a name="quickstart-create-and-manage-azure-file-shares-with-the-azure-portal"></a>Guide de démarrage rapide : créer et gérer les partages de fichiers Azure avec le portail Azure 
[Azure Files](storage-files-introduction.md) est le système de fichiers cloud facile à utiliser de Microsoft. Les partages de fichiers Azure peuvent être montés dans Windows, Linux et macOS. Ce guide vous explique les bases de l’utilisation du partage de fichiers avec le [portail Azure](https://portal.azure.com/).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-a-storage-account"></a>Créez un compte de stockage.
[!INCLUDE [storage-files-create-storage-account-portal](../../../includes/storage-files-create-storage-account-portal.md)]

## <a name="create-an-azure-file-share"></a>Crée un partage de fichiers Azure
Pour créer un partage de fichiers Azure :

1. Sélectionnez le compte de stockage dans votre tableau de bord.
2. Dans la page du compte de stockage, dans la section **Services**, sélectionnez **Fichiers**.
    ![Capture d’écran de la section des services du compte de stockage ; sélectionnez le service Fichiers](media/storage-how-to-use-files-portal/create-file-share-1.png)

3. Dans le menu situé en haut de la page **Service de fichiers**, cliquez sur **Partage de fichiers**. La page **Nouveau partage de fichier** s’affiche.
4. Dans **Nom**, saisissez *myshare*.
5. Cliquez sur **OK** pour créer le partage de fichiers Azure.

Le nom des partages ne doit contenir que des minuscules, des nombres et des traits d’union uniques, mais ne peut commencer par un trait d’union. Pour plus d’informations sur la façon de nommer des partages de fichiers et des fichiers, consultez la rubrique [Affectation de noms et références aux partages, répertoires, fichiers et métadonnées](/rest/api/storageservices/Naming-and-Referencing-Shares--Directories--Files--and-Metadata).

## <a name="use-your-azure-file-share"></a>Utiliser votre partage de fichiers Azure
Azure Files offre trois méthodes pour utiliser les fichiers et dossiers au sein de votre partage de fichiers Azure : le [protocole SMB (Server Message Block)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) standard, le protocole NFS (Network File System) en préversion et le [protocole REST de fichier](/rest/api/storageservices/file-service-rest-api). 

Pour monter un partage de fichiers avec SMB, consultez le document suivant en fonction de votre système d’exploitation :
- [Windows](storage-how-to-use-files-windows.md)
- [Linux](storage-how-to-use-files-linux.md)
- [macOS](storage-how-to-use-files-mac.md)

### <a name="using-an-azure-file-share-from-the-azure-portal"></a>Utilisation d’un partage de fichiers dans le portail Azure
Toutes les requêtes faites via le portail Azure sont faites avec l’API REST File, ce qui vous permet de créer, modifier et supprimer des fichiers et répertoires sur des clients sans accès SMB. Il est possible de travailler directement avec le protocole REST de fichier (c’est-à-dire de fabriquer vous-même les appels HTTP REST manuellement), mais la méthode la plus courante (hormis l’utilisation du portail Azure) d’utiliser le protocole REST de fichier consiste à employer le [module Azure PowerShell](storage-how-to-use-files-powershell.md), l’interface [Azure CLI](storage-how-to-use-files-cli.md) ou un kit SDK Stockage Azure. Toutes ces solutions offrent un bon wrapper autour du protocole REST de fichier dans le langage de script/programmation de votre choix. 

Nous pensons que la plupart des utilisateurs d’Azure Files voudront utiliser leur partage de fichiers Azure via le protocole SMB, ce qui leur permet de se servir des applications et outils existants qu’ils s’attendent à pouvoir utiliser, mais il existe plusieurs raisons pour lesquelles il est préférable d’utiliser l’API REST File plutôt que SMB, par exemple :

- Vous devez effectuer une modification rapide à votre partage de fichiers en déplacement (depuis un ordinateur, une tablette ou un appareil mobile sans accès SMB, par exemple).
- Vous devez exécuter un script ou une application depuis un client qui ne peut pas monter un partage SMB, tel que des clients locaux, dont le port 445 n’est pas bloqué.
- Vous tirez profit de ressources serverless, comme [Azure Functions](../../azure-functions/functions-overview.md) par exemple. 

Les exemples suivants montrent comment utiliser le portail Azure pour manipuler votre partage de fichiers Azure avec le protocole REST de fichier. 

Maintenant que vous avez créé un partage de fichiers Azure, vous pouvez monter le partage de fichiers avec SMB sur [Windows](storage-how-to-use-files-windows.md), [Linux](storage-how-to-use-files-linux.md) ou [macOS](storage-how-to-use-files-mac.md). Vous pouvez également utiliser votre partage de fichiers Azure avec le portail Azure. 

#### <a name="create-a-directory"></a>Créer un répertoire
Pour créer un répertoire nommé *myDirectory* à la racine de votre partage de fichiers Azure :

1. Dans la page **Service de fichiers**, sélectionnez le partage de fichiers **myshare**. La page de votre partage de fichiers s’ouvre.
2. Dans le menu situé en haut de la page, sélectionnez **+ Ajouter un répertoire**. La page **Nouveau répertoire** s’affiche.
3. Saisissez *myDirectory* et cliquez sur **OK**.

#### <a name="upload-a-file"></a>Charger un fichier 
Pour charger un fichier, vous devez tout d’abord créer ou sélectionner un fichier à charger. Vous pouvez le faire selon la méthode de votre choix. Une fois que vous avez sélectionné le fichier que vous voulez charger :

1. Cliquez sur le répertoire **myDirectory**. Le volet **myDirectory** s’ouvre.
2. Dans le menu situé en haut, cliquez sur **Télécharger**. Le panneau **Télécharger des fichiers** s’ouvre.  
    ![Capture d’écran du panneau Télécharger des fichiers](media/storage-how-to-use-files-portal/upload-file-1.png)

3. Cliquez sur l’icône de dossier pour ouvrir une fenêtre permettant de parcourir vos fichiers locaux. 
4. Sélectionnez un fichier, puis sur **Ouvrir**. 
5. Dans la page **Télécharger des fichiers**, vérifiez le nom du fichier, puis cliquez sur **Télécharger**.
6. Une fois terminé, le fichier apparaît dans la liste sur la page **myDirectory**.

#### <a name="download-a-file"></a>Téléchargement d’un fichier
Vous pouvez télécharger une copie du fichier que vous avez chargé en cliquant avec le bouton droit sur le fichier. Après avoir cliqué sur le bouton de téléchargement, l’expérience exacte dépend du système d’exploitation et du navigateur que vous utilisez.

## <a name="clean-up-resources"></a>Nettoyer les ressources
[!INCLUDE [storage-files-clean-up-portal](../../../includes/storage-files-clean-up-portal.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Qu’est-ce qu’Azure Files ?](storage-files-introduction.md)
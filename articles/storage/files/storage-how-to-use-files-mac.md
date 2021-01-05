---
title: Montage du partage de fichiers Azure via SMB avec MacOS | Microsoft Docs
description: Découvrez comment monter un partage de fichiers Azure via SMB avec macOS à l’aide de Finder ou de Terminal. Azure Files est le système de fichiers cloud facile à utiliser de Microsoft.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/23/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 119f4c0ea434bc431b40c905d9142e187b7d9474
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91326063"
---
# <a name="mount-azure-file-share-over-smb-with-macos"></a>Montage du partage de fichiers Azure via SMB avec MacOS
[Azure Files](storage-files-introduction.md) est le système de fichiers cloud facile à utiliser de Microsoft. Les partages de fichiers Azure peuvent être montés avec le protocole SMB 3, conforme à la norme du secteur, par macOS High Sierra 10.13 et versions ultérieures. Cet article expose deux méthodes de montage d’un partage de fichiers Azure sur macOS : l’une avec l’interface utilisateur Finder et l’autre avec Terminal.

## <a name="prerequisites-for-mounting-an-azure-file-share-on-macos"></a>Conditions préalables pour le montage d’un partage de fichiers Azure sous MacOS
* **Nom du compte de stockage** : Pour monter un partage de fichiers Azure, vous avez besoin du nom du compte de stockage.

* **Clé du compte de stockage** : Pour monter un partage de fichiers Azure, vous avez besoin de la clé de stockage primaire (ou secondaire). Actuellement, les clés SAS ne sont pas prises en charge pour le montage.

* **Vérifiez que le port 445 est ouvert** : SMB communique via le port TCP 445. Sur votre machine client (Mac), vérifiez que votre pare-feu ne bloque pas le port TCP 445.

## <a name="mount-an-azure-file-share-via-finder"></a>Montage d’un partage de fichiers Azure via Finder
1. **Ouvrez Finder** : Finder est ouvert par défaut sur macOS, mais vous pouvez vérifier qu’il s’agit bien de l’application actuellement sélectionnée en cliquant sur l’icône de visage macOS dans la barre :  
    ![L’icône MacOS (visage)](./media/storage-how-to-use-files-mac/mount-via-finder-1.png)

2. **Sélectionnez « Se connecter au serveur » dans le menu « Aller »**  : En utilisant le chemin UNC des prérequis, remplacez la double barre oblique inverse (`\\`) par `smb://` et toutes les autres barres obliques inverses (`\`) par des barres obliques normales (`/`). Votre lien doit se présenter comme suit : ![Boîte de dialogue « Se connecter au serveur »](./media/storage-how-to-use-files-mac/mount-via-finder-2.png)

3. **Utilisez le nom du compte de stockage et la clé du compte de stockage quand un nom d’utilisateur et un mot de passe vous sont demandés** : Quand vous cliquez sur « Connexion », dans la boîte de dialogue « Se connecter au serveur », vous êtes invité à entrer le nom d’utilisateur et le mot de passe (votre nom d’utilisateur macOS est automatiquement prérempli). Vous avez la possibilité de placer le nom/la clé du compte de stockage dans votre trousseau MacOS.

4. **Utilisez le partage de fichiers Azure à votre guise** : Après avoir remplacé le nom d’utilisateur et le mot de passe par le nom du partage et par la clé du compte de stockage, vous pouvez monter le partage. Vous pouvez l’utiliser comme un partage de fichiers / un dossier local normal. Vous pouvez notamment faire glisser et déposer des fichiers dans le partage :

    ![Une capture instantanée d’un partage de fichiers Azure monté](./media/storage-how-to-use-files-mac/mount-via-finder-3.png)

## <a name="mount-an-azure-file-share-via-terminal"></a>Montage d’un partage de fichiers Azure via Terminal
1. Remplacez  `<storage-account-name>`, `<storage-account-key>` et `<share-name>` par les valeurs correspondant à votre environnement.

    ```
    open smb://<storage-account-name>:<storage-account-key>@<storage-account-name>.file.core.windows.net/<share-name>
    ```

2. **Utilisez le partage de fichiers Azure à votre guise** : Le partage de fichiers Azure est monté sur le point de montage spécifié par la commande précédente.  

    ![Une capture instantanée du partage de fichiers Azure monté](./media/storage-how-to-use-files-mac/mount-via-terminal-1.png)

## <a name="next-steps"></a>Étapes suivantes
* [Assistance Apple : Connectez votre Mac à des ordinateurs partagés et des serveurs](https://support.apple.com/guide/mac-help/connect-mac-shared-computers-servers-mchlp1140/mac)
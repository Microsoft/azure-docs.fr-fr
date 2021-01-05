---
title: Aperçu de la préparation d’une image d’attachement d’application MSIX pour Windows Virtual Desktop – Azure
description: Comment créer une image d’attachement d’application MSIX pour un pool d’hôtes Windows Virtual Desktop.
author: Heidilohr
ms.topic: how-to
ms.date: 12/14/2020
ms.author: helohr
manager: lizross
ms.openlocfilehash: 204cc9a05d62caf62179100fa3496be422a3ec0c
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97417430"
---
# <a name="prepare-an-msix-image-for-windows-virtual-desktop"></a>Préparer une image MSIX pour Windows Virtual Desktop

> [!IMPORTANT]
> L’application MSIX est actuellement disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Attachement d’application MSIX (préversion) est une solution d’application en couches qui vous permet d’attacher dynamiquement des applications d’un package MSIX à une session utilisateur. Le système de package MSIX sépare les applications du système d’exploitation, ce qui facilite la création d’images pour les machines virtuelles. Les packages MSIX vous permettent également de mieux contrôler les applications auxquelles vos utilisateurs peuvent accéder sur leurs machines virtuelles. Vous pouvez même séparer les applications de l’image principale et les donner aux utilisateurs ultérieurement.

## <a name="create-a-vhd-or-vhdx-package-for-msix"></a>Créer un package VHD ou VHDX pour MSIX

Les packages MSIX doivent être au format VHD ou VHDX pour fonctionner correctement. Cela signifie que, pour commencer, vous devez créer un package VHD ou VHDX.

>[!NOTE]
>Si ce n’est déjà fait, veillez à activer Hyper-V en suivant les instructions fournies dans [Installer Hyper-V sur Windows 10](/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v).

Pour créer un package VHD ou VHDX pour MSIX :

1. Tout d’abord, ouvrez PowerShell.
2. Ensuite, exécutez l'applet de commande suivante pour créer un VHD :

    ```powershell
    New-VHD -SizeBytes <size>MB -Path c:\temp\<name>.vhd -Dynamic -Confirm:$false
    ```

    >[!NOTE]
    > Assurez-vous que le VHD est suffisamment grand pour contenir le package MSIX développé.

3. Exécutez l’applet de commande suivante pour monter le VHD que vous venez de créer :

    ```powershell
    $vhdObject = Mount-VHD c:\temp\<name>.vhd -Passthru
    ```

4. Ensuite, exécutez cette applet de commande pour initialiser le VHD :

    ```powershell
    $disk = Initialize-Disk -Passthru -Number $vhdObject.Number
    ```

5. Après cela, exécutez cette applet de commande pour créer une partition pour le VHD initialisé :

    ```powershell
    $partition = New-Partition -AssignDriveLetter -UseMaximumSize -DiskNumber $disk.Number
    ```

6. Exécutez cette applet de commande pour formater la partition :

    ```powershell
    Format-Volume -FileSystem NTFS -Confirm:$false -DriveLetter $partition.DriveLetter -Force
    ```

7. Enfin, créez un dossier parent sur le VHD monté. Cette étape est obligatoire, car le package MSIX doit avoir un dossier parent pour fonctionner correctement. Peu importe le nom que vous donnez au dossier parent, tant que le dossier parent existe.

## <a name="expand-msix"></a>Développer MSIX

Après cela, vous devez étendre l’image MSIX en décompressant ses fichiers dans le VHD.

Pour étendre l’image MSIX :

1. [Téléchargez l’outil msixmgr](https://aka.ms/msixmgr) et enregistrez le dossier .zip dans un dossier au sein d’une machine virtuelle hôte de session.

2. Décompressez le dossier .zip de l’outil msixmgr.

3. Placez le package MSIX source dans le même dossier que celui où vous avez décompressé l’outil msixmgr.

4. Ouvrez une invite de commandes en tant qu’administrateur et accédez au dossier dans lequel vous avez téléchargé et décompressé l’outil msixmgr.

5. Exécutez l’applet de commande suivante pour décompresser le MSIX dans le VHD que vous avez créé dans la section précédente.

    ```powershell
    msixmgr.exe -Unpack -packagePath <package>.msix -destination "f:\<name of folder you created earlier>" -applyacls
    ```

    Le message suivant doit apparaître une fois la décompression terminée :

    > Listes de contrôle d’accès décompressées et appliquées pour le package : <package name>.msix

    >[!NOTE]
    > Si vous utilisez des packages du Microsoft Store pour Entreprises ou Éducation au sein de votre réseau ou sur des appareils qui ne sont pas connectés à Internet, vous devez télécharger et installer les licences de package à partir du Microsoft Store pour exécuter les applications. Pour obtenir les licences, consultez [Utiliser des packages hors connexion](app-attach.md#use-packages-offline).

6. Accédez au VHD monté et ouvrez le dossier de l’application pour vous assurer que le contenu du package y figure.

7. Démontez le disque dur virtuel.

## <a name="upload-msix-image-to-share"></a>Charger l’image MSIX pour partager

Une fois que vous avez créé le package MSIX, vous devez charger le fichier VHD, VHDX ou CIM résultant dans un partage accessible aux machines virtuelles de vos utilisateurs.

## <a name="next-steps"></a>Étapes suivantes

Posez vos questions concernant cette fonctionnalité à la [communauté technique de Windows Virtual Desktop](https://techcommunity.microsoft.com/t5/Windows-Virtual-Desktop/bd-p/WindowsVirtualDesktop).

Vous pouvez également laisser vos commentaires sur Windows Virtual Desktop sur le [Hub de commentaires Windows Virtual Desktop](https://support.microsoft.com/help/4021566/windows-10-send-feedback-to-microsoft-with-feedback-hub-app).

Voici d’autres articles qui peuvent vous être utiles :

- [Glossaire Attachement d’application MSIX](app-attach-glossary.md)
- [FAS sur l’attachement d’application MSIX](app-attach-faq.md)
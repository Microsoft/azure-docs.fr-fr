---
title: Télécharger un fichier de disque dur virtuel vers Azure DevTest Labs à l’aide d’AzCopy | Microsoft Docs
description: Cet article explique pas à pas comment utiliser l’utilitaire de ligne de commande AzCopy pour télécharger un fichier VHD sur le compte de stockage d’un lab dans Azure DevTest Labs.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 1d8ede0f78726b04ac862a00b559b8d42c3ed1cd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88650772"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-azcopy"></a>Télécharger un fichier de disque dur virtuel dans le compte de stockage d’un laboratoire avec AzCopy

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

Dans Azure DevTest Labs, les fichiers de disque dur virtuel peuvent être utilisés pour créer des images personnalisées, qui servent à approvisionner des machines virtuelles. Les étapes suivantes vous guident lors de l’utilisation de l’utilitaire de ligne de commande AzCopy pour télécharger un fichier de disque dur virtuel pour le compte de stockage d’un laboratoire. Une fois que vous avez téléchargé votre fichier de disque dur virtuel, consultez la [section Étapes suivantes](#next-steps) qui répertorie quelques articles qui expliquent comment créer une image personnalisée à partir du fichier de disque dur virtuel téléchargé. Pour plus d’informations sur les disques et les disques durs virtuels dans Azure, consultez [Présentation des disques managés](../virtual-machines/managed-disks-overview.md)

> [!NOTE] 
>  
> L’utilitaire de ligne de commande AzCopy fonctionne exclusivement sur Windows.

## <a name="step-by-step-instructions"></a>Instructions pas à pas

Les étapes suivantes vous guident lors du téléchargement d’un fichier de disque dur virtuel dans Azure DevTest Labs à l’aide d’[AzCopy](https://aka.ms/downloadazcopy). 

1. Obtenez le nom du compte de stockage du laboratoire à l’aide du portail Azure :

1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).

1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.

1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  

1. Dans le panneau du laboratoire, sélectionnez **Configuration**. 

1. Dans le panneau **Configuration** du laboratoire, sélectionnez **Custom images (VHDs)** (Images personnalisées (disques durs virtuels)).

1. Dans le panneau **Images personnalisées**, sélectionnez **+Ajouter**. 

1. Dans le panneau **Image personnalisée**, sélectionnez **Disque dur virtuel**.

1. Dans le panneau **Disque dur virtuel**, sélectionnez **Upload a VHD using PowerShell** (Télécharger un disque dur virtuel à l’aide de PowerShell).

    ![Télécharger un disque dur virtuel à l’aide de PowerShell](./media/devtest-lab-upload-vhd-using-azcopy/upload-image-using-psh.png)

1. Le panneau **Upload an image using PowerShell** (Télécharger une image à l’aide de PowerShell) affiche un appel à l’applet de commande **Add-AzureVhd**. Le premier paramètre (*Destination*) contient l’URI d’un conteneur d’objets blob (*uploads*) au format suivant :

    ```
    https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/...
    ``` 

1. Prenez note de l’URI complet, car vous en aurez besoin dans des étapes ultérieures.

1. Téléchargez le fichier de disque dur virtuel à l’aide d’AzCopy :
 
1. [Téléchargez et installez la dernière version d’AzCopy](https://aka.ms/downloadazcopy).

1. Ouvrez une fenêtre Commande et naviguez jusqu’au répertoire d’installation d’AzCopy. Si vous le souhaitez, vous pouvez ajouter l’emplacement d’installation d’AzCopy au chemin de votre système. Par défaut, AzCopy est installé dans le répertoire suivant :

    ```command-line
    %ProgramFiles(x86)%\Microsoft SDKs\Azure\AzCopy
    ```

1. À l’aide de la clé du compte de stockage et de l’URI du conteneur d’objets blob, exécutez la commande suivante à l’invite de commandes. La valeur *vhdFileName* doit être entre guillemets. Le processus de téléchargement d’un fichier de disque dur virtuel peut durer un certain temps en fonction de sa taille et de votre vitesse de connexion.   

    ```command-line
    AzCopy /Source:<sourceDirectory> /Dest:<blobContainerUri> /DestKey:<storageAccountKey> /Pattern:"<vhdFileName>" /BlobType:page
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Créer une image personnalisée dans Azure DevTest Labs à partir d’un fichier de disque dur virtuel à l’aide du portail Azure](devtest-lab-create-template.md)
- [Créer une image personnalisée dans Azure DevTest Labs à partir d’un fichier de disque dur virtuel à l’aide de PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)
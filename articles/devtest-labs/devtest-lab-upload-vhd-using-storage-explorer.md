---
title: Charger un fichier VHD sur Azure DevTest Labs à l’aide de l’Explorateur Stockage
description: Télécharger le fichier de disque dur virtuel dans le compte de stockage du laboratoire avec l’explorateur de stockage Microsoft Azure
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: d4e421932ebba5d4f389000c12bcf44ac2f37599
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91282525"
---
# <a name="upload-vhd-file-to-labs-storage-account-using-microsoft-azure-storage-explorer"></a>Télécharger le fichier de disque dur virtuel dans le compte de stockage du laboratoire avec l’explorateur de stockage Microsoft Azure

[!INCLUDE [devtest-lab-upload-vhd-selector](../../includes/devtest-lab-upload-vhd-selector.md)]

Dans Azure DevTest Labs, les fichiers de disque dur virtuel peuvent être utilisés pour créer des images personnalisées, qui servent à approvisionner des machines virtuelles. Cet article montre comment utiliser l’[Explorateur de stockage Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md) pour charger un fichier de disque dur virtuel vers le compte de stockage d’un laboratoire. Une fois que vous avez téléchargé votre fichier de disque dur virtuel, consultez la [section Étapes suivantes](#next-steps) qui répertorie quelques articles qui expliquent comment créer une image personnalisée à partir du fichier de disque dur virtuel téléchargé. Pour plus d’informations sur les disques et les disques durs virtuels dans Azure, consultez [Présentation des disques managés](../virtual-machines/managed-disks-overview.md)

## <a name="step-by-step-instructions"></a>Instructions pas à pas

Les étapes suivantes vous guident lors du téléchargement d’un fichier de disque dur virtuel dans Azure DevTest Labs avec l’[Explorateur de stockage Microsoft Azure](../vs-azure-tools-storage-manage-with-storage-explorer.md).

1. [Téléchargez et installez la version la plus récente de l’Explorateur de stockage Microsoft Azure](https://www.storageexplorer.com).

1. Obtenez le nom du compte de stockage du laboratoire à l’aide du portail Azure :

    1. Connectez-vous au [portail Azure](https://go.microsoft.com/fwlink/p/?LinkID=525040).
    
    1. Sélectionnez **Tous les services**, puis **DevTest Labs** dans la liste.
    
    1. Sélectionnez le laboratoire souhaité dans la liste des laboratoires.  
    
    1. Dans le panneau du laboratoire, sélectionnez **Configuration**. 
    
    1. Dans le panneau **Configuration** du laboratoire, sélectionnez **Custom images (VHDs)** (Images personnalisées (disques durs virtuels)).
    
    1. Dans le panneau **Images personnalisées**, sélectionnez **+Ajouter**. 
    
    1. Dans le panneau **Image personnalisée**, sélectionnez **Disque dur virtuel**.
    
    1. Dans le panneau **Disque dur virtuel**, sélectionnez **Upload a VHD using PowerShell** (Télécharger un disque dur virtuel à l’aide de PowerShell).
    
        ![Télécharger un disque dur virtuel à l’aide de PowerShell][0]
    
    1. Le panneau **Upload an image using PowerShell** (Télécharger une image à l’aide de PowerShell) affiche un appel à l’applet de commande **Add-AzureVhd**. Le premier paramètre (*Destination*) contient le nom du compte de stockage du laboratoire au format suivant :
    
        `https://<STORAGE-ACCOUNT-NAME>.blob.core.windows.net/uploads/...`

    1. Prenez note du nom du compte de stockage, car vous en aurez besoin dans des étapes ultérieures.
    
1. Connectez-vous au compte lié à un abonnement Azure à l’aide de l’Explorateur de stockage Microsoft Azure.

    > [!TIP] 
    > 
    > Ce dernier prend en charge plusieurs options de connexion. Cette section montre la connexion à un compte de stockage associé à votre abonnement Azure. Pour voir les autres options de connexion prises en charge par l’Explorateur de stockage, consultez [Prise en main de l’Explorateur de stockage](../vs-azure-tools-storage-manage-with-storage-explorer.md).
 
    1. Ouvrez l’Explorateur de stockage.
    
    1. Dans l’Explorateur de stockage, sélectionnez **Paramètres de compte Azure**. 
    
        ![paramètres de compte Azure][1]
    
    1. Le volet gauche affiche les comptes Microsoft auxquels vous vous êtes connecté. Pour vous connecter à un autre compte, sélectionnez **Ajouter un compte**et suivez les indications des boîtes de dialogue pour vous connecter avec un compte Microsoft associé à un ou plusieurs abonnements Azure actifs.
    
        ![Ajouter un compte][2]
    
    1. Une fois que vous êtes connecté avec un compte Microsoft, le volet gauche indique les abonnements Azure associés à ce compte. Sélectionnez les abonnements Azure que vous souhaitez utiliser, puis sélectionnez **Appliquer**. (La case à cocher **Tous les abonnements** permet de sélectionner ou de désélectionner l’ensemble des abonnements Azure répertoriés.)
    
        ![Sélectionner les abonnements Azure][3]
    
    1. Le volet gauche affiche les comptes de stockage associés aux abonnements Azure sélectionnés.
    
        ![Abonnements Azure sélectionnés][4]

1. Recherchez le compte de stockage du laboratoire :

    1. Dans le volet gauche de l’Explorateur de stockage, localisez et développez le nœud de l’abonnement Azure qui possède le laboratoire.
    
    1. Sous le nœud de l’abonnement, développez **Comptes de stockage**.

    1. Développez le nœud de compte de stockage du laboratoire pour afficher les nœuds pour **Conteneurs d’objets blob**, **Partages de fichiers**, **Files d’attente** et **Tables**.
    
    1. Développez le nœud **Conteneurs d’objets blob**.
    
    1. Sélectionnez le conteneur d’objets blob de téléchargements (uploads) pour afficher son contenu dans le volet droit.
        
        ![Répertoire du téléchargement][5]

1. Téléchargez le fichier de disque dur virtuel à l’aide de l’Explorateur de stockage :

    1. Dans le volet droit de l’Explorateur de stockage, vous devez voir une liste de blobs dans le conteneur de blobs **uploads** du compte de stockage du laboratoire. Dans la barre d’outils de l’éditeur de blobs, sélectionnez **Télécharger** 
        
        ![Capture d’écran montrant le volet droit de l’Explorateur Stockage avec le bouton « Charger » sélectionné.][6]
    
    1. Dans le menu déroulant **Télécharger**, sélectionnez **Télécharger des fichiers...** .
    
    1. Dans la boîte de dialogue **Télécharger des fichiers**, sélectionnez les points de suspension.
        
        ![Capture d’écran montrant la boîte de dialogue « Charger des fichiers » avec les points de suspension sélectionnés.][8]  

    1. Dans la boîte de dialogue **Sélectionner les fichiers à télécharger**, recherchez le fichier de disque dur virtuel souhaité, sélectionnez-le, puis sélectionnez **Ouvrir**.
    
    1. Lorsque vous êtes à nouveau dans la boîte de dialogue **Télécharger des fichiers**, définissez **Type d’objet BLOB** sur **Objet blob de pages**.
    
    1. Sélectionnez **Télécharger**.

        ![Sélectionner un fichier][9]  
    
    1. Le volet **Journal d’activité** de l’Explorateur de stockage affiche l’état du téléchargement (ainsi que des liens permettant d’annuler le téléchargement). Le processus de téléchargement d’un fichier de disque dur virtuel peut durer un certain temps en fonction de sa taille et de votre vitesse de connexion. 

        ![État du téléchargement de fichier][10]  

## <a name="next-steps"></a>Étapes suivantes

- [Créer une image personnalisée dans Azure DevTest Labs à partir d’un fichier de disque dur virtuel à l’aide du portail Azure](devtest-lab-create-template.md)
- [Créer une image personnalisée dans Azure DevTest Labs à partir d’un fichier de disque dur virtuel à l’aide de PowerShell](devtest-lab-create-custom-image-from-vhd-using-powershell.md)

[0]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-image-using-psh.png
[1]: ./media/devtest-lab-upload-vhd-using-storage-explorer/settings-icon.png
[2]: ./media/devtest-lab-upload-vhd-using-storage-explorer/add-account-link.png
[3]: ./media/devtest-lab-upload-vhd-using-storage-explorer/subscriptions-list.png
[4]: ./media/devtest-lab-upload-vhd-using-storage-explorer/storage-accounts-list.png
[5]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-dir.png
[6]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-button.png
[7]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-files.png
[8]: ./media/devtest-lab-upload-vhd-using-storage-explorer/select-file.png
[9]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-file.png
[10]: ./media/devtest-lab-upload-vhd-using-storage-explorer/upload-status.png

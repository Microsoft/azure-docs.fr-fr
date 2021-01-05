---
title: Azure Lab Services - Charger une image personnalisée sur Shared Image Gallery
description: Décrit comment charger une image personnalisée sur Shared Image Gallery. Les services informatiques d’université devraient particulièrement apprécier la fonctionnalité d’importation d’images.
ms.date: 09/30/2020
ms.topic: how-to
ms.openlocfilehash: 93b4141636b629168e9bb7a73e71a9fe4bfc39f5
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94654641"
---
# <a name="upload-a-custom-image-to-shared-image-gallery"></a>Charger une image personnalisée sur Shared Image Gallery

Shared Image Gallery vous permet d’importer vos propres images personnalisées pour créer des labos dans Azure Lab Services. Les services informatiques d’université devraient particulièrement apprécier la fonctionnalité d’importation d’images pour les raisons suivantes : 

* Vous n’avez pas besoin de créer manuellement des images avec un modèle de machine virtuelle de laboratoire.
* Vous pouvez charger des images créées à l’aide d’autres outils tels que SCCM, Endpoint Manager, etc.

Cet article décrit les étapes à suivre pour apporter une image personnalisée et l’utiliser dans Azure Lab Services. 

> [!IMPORTANT]
> Lorsque vous déplacez vos images d’un environnement de laboratoire physique vers Az Labs, vous devez les restructurer en conséquence. Ne réutilisez pas simplement vos images existantes à partir de laboratoires physiques. <br/>Pour plus d’informations, consultez le billet de blog sur le [déplacement d’un laboratoire physique vers Azure Lab Services](https://techcommunity.microsoft.com/t5/azure-lab-services/moving-from-a-physical-lab-to-azure-lab-services/ba-p/1654931).

## <a name="bring-custom-image-from-a-physical-lab-environment"></a>Apporter une image personnalisée depuis un environnement de laboratoire physique

Les étapes suivantes montrent comment importer une image personnalisée qui démarre dans un environnement de laboratoire physique. Un disque dur virtuel est alors créé à partir de cet environnement et importé dans Shared Image Gallery dans Azure pour pouvoir être utilisé dans Azure Lab Services.

De nombreuses options existent pour créer un disque dur virtuel à partir d’un environnement de laboratoire physique. Les étapes suivantes montrent comment créer un disque dur virtuel à partir d’une machine virtuelle Windows Hyper-V :

1. Commencez par une machine virtuelle Hyper-V dans votre environnement de laboratoire physique créé à partir de votre image.
    1. La machine virtuelle doit être créée en tant que machine virtuelle de génération 1.
    1. La machine virtuelle doit utiliser une taille de disque fixe. Vous pouvez également spécifier la taille de disque dans cette fenêtre. La taille de disque ne doit pas dépasser 128 Go.<br/>    
    Les images dont la taille de disque dépasse 128 Go ne sont pas prises en charge par Azure Lab Services. 
       
        :::image type="content" source="./media/upload-custom-image-shared-image-gallery/connect-virtual-hard-disk.png" alt-text="Connecter un disque dur virtuel":::   
    1. Créez l’image de machine virtuelle comme à l’ordinaire.
1. [Connectez-vous à la machine virtuelle et préparez-la pour Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md).
    1. [Définir les configurations Windows pour Azure](../virtual-machines/windows/prepare-for-upload-vhd-image.md#set-windows-configurations-for-azure)
    1. [Vérifier les services Windows minimum requis pour garantir la connectivité des machines virtuelles](../virtual-machines/windows/prepare-for-upload-vhd-image.md#check-the-windows-services)
    1. [Mettre à jour les paramètres de Registre du Bureau à distance](../virtual-machines/windows/prepare-for-upload-vhd-image.md#update-remote-desktop-registry-settings)
    1. [Configurer les règles du Pare-feu Windows](../virtual-machines/windows/prepare-for-upload-vhd-image.md#configure-windows-firewall-rules)
    1. Installer les mises à jour Windows
    1. [Installer l’agent de machine virtuelle Azure et la configuration supplémentaire indiquée ici](../virtual-machines/windows/prepare-for-upload-vhd-image.md#complete-the-recommended-configurations) 
    
    Les étapes suivantes créent une image spécialisée. Si vous créez une image généralisée, vous devez également exécuter [SysPrep](../virtual-machines/windows/prepare-for-upload-vhd-image.md#determine-when-to-use-sysprep). <br/>
        Vous devez créer une image spécialisée si vous souhaitez conserver le répertoire Utilisateur (qui peut contenir des fichiers, des informations de compte d’utilisateur, etc.) qui est requis par les logiciels inclus dans l’image.
1. Étant donné que **Hyper-V** crée un fichier **VHDX** par défaut, vous devez le convertir en fichier VHD.
    1. Accédez à **Gestionnaire Hyper-V** -> **Action** -> **Modifier le disque**.
    1. Dans la fenêtre, vous avez l’option de **convertir** le disque de VHDX en VHD
    1. Si vous voulez augmenter la taille de disque, veillez à ne pas dépasser 128 Go.        
        :::image type="content" source="./media/upload-custom-image-shared-image-gallery/choose-action.png" alt-text="Choisir une action":::   
1. Chargez le disque dur virtuel vers Azure pour créer un disque managé.
    1. Vous pouvez utiliser l’explorateur de stockage ou AzCopy dans la ligne de commande, comme le décrit l’article [Charger un disque dur virtuel sur Azure ou copier un disque managé dans une autre région](../virtual-machines/windows/disks-upload-vhd-to-managed-disk-powershell.md).        
    Si votre ordinateur se met en veille ou se verrouille, le processus de chargement peut s’interrompre et échouer.
    1. Le résultat de cette étape est que vous disposez désormais d’un disque managé, qui apparaît dans le portail Azure. 
        Vous pouvez utiliser l’onglet du portail Azure « Taille\Performance » pour choisir votre taille de disque. Comme mentionné précédemment, la taille ne doit pas dépasser 128 Go.
1. Prenez un instantané du disque managé.
    Pour cela, vous pouvez utiliser PowerShell, le portail Azure ou l’explorateur de stockage, selon la procédure décrite dans [Créer un instantané à l’aide du portail ou de PowerShell](../virtual-machines/windows/snapshot-copy-managed-disk.md).
1. Dans Shared Image Gallery, créez une définition et une version d’image :
    1. [Créez une définition d’image](../virtual-machines/windows/shared-images-portal.md#create-an-image-definition).
    1. Vous devez également spécifier ici si vous créez une image spécialisée ou généralisée.
1. Créez le labo dans Azure Lab Services et sélectionnez l’image personnalisée dans Shared Image Gallery.

    Si vous avez augmenté la taille du disque après l’installation du système d’exploitation sur la machine virtuelle Hyper-V d’origine, vous devrez également étendre le lecteur C dans Windows pour utiliser l’espace de disque non alloué. Pour cela, connectez-vous au modèle de machine virtuelle après avoir créé le labo, puis suivez la procédure décrite dans [Étendre un volume de base](/windows-server/storage/disk-management/extend-a-basic-volume). Vous pouvez pour cela utiliser l’interface utilisateur ou PowerShell.

## <a name="next-steps"></a>Étapes suivantes

* [Présentation de Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md)
* [Utilisation de Shared Image Gallery](how-to-use-shared-image-gallery.md)
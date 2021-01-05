---
title: Utiliser l’Explorateur Stockage Azure pour gérer des disques managés Azure
description: Découvrez comment charger, télécharger et migrer un disque managé Azure d’une région à une autre, et créer un instantané d’un disque managé, à l’aide de l’Explorateur Stockage Azure.
author: roygara
ms.author: rogarana
ms.date: 09/25/2019
ms.topic: how-to
ms.service: virtual-machines
ms.subservice: disks
ms.openlocfilehash: 9dfce7b76eed5bfc9f4979c0e3041b6c65c28422
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88749373"
---
# <a name="use-azure-storage-explorer-to-manage-azure-managed-disks"></a>Utiliser l’Explorateur Stockage Azure pour gérer des disques managés Azure

Explorateur Stockage 1.10.0 permet aux utilisateurs de charger, télécharger et copier des disques managés, et créer des instantanés. En raison de ces fonctionnalités supplémentaires, vous pouvez utiliser Explorateur Stockage pour migrer des données locales vers Azure et migrer des données entre des régions Azure.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :
- Abonnement Azure
- Un ou plusieurs disques managés Azure
- La version la plus récente de [l’Explorateur Stockage Azure](https://azure.microsoft.com/features/storage-explorer/)

## <a name="connect-to-an-azure-subscription"></a>Connexion à un abonnement Azure

Si votre Explorateur Stockage n’est pas connecté à Azure, vous ne pourrez pas l’utiliser pour gérer les ressources. Cette section explique comment connecter Explorateur Stockage à votre compte Azure pour gérer les ressources.

1. Lancez Explorateur Stockage Azure et cliquez sur l’icône de **plug-in** sur la gauche.

    ![Cliquer sur l’icône de plug-in](media/disks-upload-vhd-to-managed-disk-storage-explorer/plug-in-icon.png)

1. Sélectionnez **Ajouter un compte Azure**, puis cliquez sur **Suivant**.

    ![Ajouter un compte Azure](media/disks-upload-vhd-to-managed-disk-storage-explorer/connect-to-azure.png)

1. Dans la boîte de dialogue **Connexion à Azure**, entrez vos informations d’identification Azure.

    ![Boîte de dialogue Connexion à Azure](media/disks-upload-vhd-to-managed-disk-storage-explorer/sign-in.png)

1. Sélectionnez votre abonnement dans la liste, puis cliquez sur **Appliquer**.

    ![Sélectionnez votre abonnement](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-subscription.png)

## <a name="upload-a-managed-disk-from-an-on-prem-vhd"></a>Charger un disque managé à partir d’un disque dur virtuel local

1. Dans le volet gauche, développez **Disques** et sélectionnez le groupe de ressources dans lequel vous souhaitez charger votre disque.

    ![Sélectionner le groupe de ressources 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. Sélectionnez **Télécharger**.

    ![Sélectionner Télécharger](media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-button.png)

1. Dans **Charger le disque dur virtuel**, indiquez votre disque dur virtuel source, le nom du disque, le type de système d’exploitation, la région dans laquelle vous souhaitez charger le disque, ainsi que le type de compte. Dans certaines régions, les zones de disponibilité sont prises en charge. Pour ces régions, vous pouvez sélectionner la zone de votre choix.
1. Sélectionnez **Créer** pour commencer le chargement de votre disque.

    ![Boîte de dialogue Charger un disque dur virtuel](media/disks-upload-vhd-to-managed-disk-storage-explorer/upload-vhd-dialog.png)

1. L’état du chargement s’affiche à présent dans **Activités**.

    ![État du chargement](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-uploading.png)

1. Si le chargement est terminé et que vous ne voyez pas le disque dans le volet droit, sélectionnez **Actualiser**.

## <a name="download-a-managed-disk"></a>Télécharger un disque managé

Les étapes suivantes expliquent comment télécharger un disque managé sur un disque dur virtuel local. L’état d’un disque doit être **Non attaché** pour pouvoir être téléchargé. Vous ne pouvez pas télécharger un disque **attaché**.

1. Dans le volet gauche, développez **Disques**, si nécessaire, et sélectionnez le groupe de ressources à partir duquel vous souhaitez télécharger votre disque.

    ![Sélectionner le groupe de ressources 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. Dans le volet droit, sélectionnez le disque à télécharger.
1. Sélectionnez **Télécharger** puis choisissez l’emplacement où enregistrer le disque.

    ![Télécharger un disque managé](media/disks-upload-vhd-to-managed-disk-storage-explorer/download-button.png)

1. Sélectionnez **Enregistrer** pour lancer le téléchargement de votre disque. L’état du téléchargement s’affiche dans **Activités**.

    ![État du téléchargement](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-downloading.png)

## <a name="copy-a-managed-disk"></a>Copier un disque managé

Avec Explorateur Stockage, vous pouvez copier un disque managé dans ou entre différentes régions. Pour copier un disque :

1. Dans la liste déroulante **Disques** sur la gauche, sélectionnez le groupe de ressources qui contient le disque que vous souhaitez copier.

    ![Sélectionner le groupe de ressources 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. Dans le volet droit, sélectionnez le disque à copier et choisissez **Copier**.

    ![Copier un disque managé](media/disks-upload-vhd-to-managed-disk-storage-explorer/copy-button.png)

1. Dans le volet gauche, sélectionnez le groupe de ressources dans lequel vous souhaitez coller le disque.

    ![Sélectionner le groupe de ressources 2](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg2.png)

1. Sélectionnez **Coller** dans le volet droit.

    ![Coller un disque managé](media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-button.png)

1. Dans la boîte de dialogue **Coller le disque**, renseignez les valeurs. Vous pouvez également spécifier une zone de disponibilité dans les régions prises en charge.

    ![Boîte de dialogue Coller le disque](media/disks-upload-vhd-to-managed-disk-storage-explorer/paste-disk-dialog.png)

1. Sélectionnez **Coller** pour lancer la copie de votre disque. L’état est affiché dans **Activités**.

    ![État Copier/Coller](media/disks-upload-vhd-to-managed-disk-storage-explorer/activity-copying.png)

## <a name="create-a-snapshot"></a>Créer un instantané

1. Dans la liste déroulante **Disques** sur la gauche, sélectionnez le groupe de ressources qui contient le disque dont vous voulez créer un instantané.

    ![Sélectionner le groupe de ressources 1](media/disks-upload-vhd-to-managed-disk-storage-explorer/select-rg1.png)

1. Sur la droite, sélectionnez le disque dont vous voulez créer un instantané, puis sélectionnez **Créer un instantané**.

    ![Créer un instantané](media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-button.png)

1. Dans **Créer un instantané**, spécifiez le nom de l’instantané, ainsi que le groupe de ressources dans lequel vous souhaitez le créer. Sélectionnez ensuite **Créer**.

    ![Boîte de dialogue Créer un instantané](media/disks-upload-vhd-to-managed-disk-storage-explorer/create-snapshot-dialog.png)

1. Une fois l’instantané créé, vous pouvez sélectionner **Ouvrir dans le portail** sous **Activités** pour afficher l’instantané dans le portail Azure.

    ![Ouvrir l’instantané dans le portail](media/disks-upload-vhd-to-managed-disk-storage-explorer/open-in-portal.png)

## <a name="next-steps"></a>Étapes suivantes


Découvrez comment [Créer une machine virtuelle à partir d’un disque dur virtuel à l’aide du portail Azure](windows/create-vm-specialized-portal.md).

Découvrez comment [Attacher un disque de données managé à une machine virtuelle Windows à l’aide du portail Azure](windows/attach-managed-disk-portal.md).
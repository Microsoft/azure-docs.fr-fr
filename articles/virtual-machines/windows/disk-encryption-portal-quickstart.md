---
title: Créer et chiffrer une machine virtuelle Windows à l’aide du portail Azure
description: Dans ce guide de démarrage rapide, vous apprenez à utiliser le portail Azure pour créer et chiffrer une machine virtuelle Windows
author: msmbaldwin
ms.author: mbaldwin
ms.service: virtual-machines-windows
ms.subservice: security
ms.topic: quickstart
ms.date: 10/02/2019
ms.openlocfilehash: 98764df41d5488afaaf7a24e0f1aa2535b0691d7
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896954"
---
# <a name="quickstart-create-and-encrypt-a-windows-virtual-machine-with-the-azure-portal"></a>Démarrage rapide : Créer et chiffrer une machine virtuelle Windows à l’aide du portail Azure

Le Portail Azure peut être utilisé pour créer des machines virtuelles Azure. Le Portail Azure est une interface utilisateur basée sur un navigateur permettant de créer des machines virtuelles et leurs ressources associées. Dans ce guide de démarrage rapide, vous utilisez le portail Azure pour déployer une machine virtuelle Windows, créer un coffre de clés pour le stockage des clés de chiffrement et chiffrer la machine virtuelle.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).


## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. Choisissez **Créer une ressource** en haut à gauche du Portail Azure.
1. Dans la page Nouveau, sous Populaire, sélectionnez **Windows Server 2016 Datacenter**.
1. Sous l’onglet De base, sous Détails du projet, vérifiez que l’abonnement approprié est sélectionné.
1. Pour « Groupe de ressources », sélectionnez **Créer**. Entrez le nom *myResourceGroup* , puis sélectionnez **OK**.
1. Dans **Nom de la machine virtuelle** , entrez *MyVM*.
1. Pour **Région** , sélectionnez *(États-Unis) USA Est*.
1. Assurez-vous que la **Taille** est *Standard D2s v3*.
1. Sous **Compte Administrateur** , sélectionnez **Mot de passe**. Entrez un nom d'utilisateur et un mot de passe.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-windows-vm-creation.png" alt-text="Écran Création de machine virtuelle Windows":::

    > [!WARNING]
    > L’onglet « Disques » contient un champ « type de chiffrement » sous **Options de disque**. Ce champ permet de spécifier les options de chiffrement pour [Disques managés](../managed-disks-overview.md) et CMK, et non pour Azure Disk Encryption.
    >
    > Pour éviter toute confusion, nous vous suggérons d’ignorer complètement l’onglet *Disques* pendant ce tutoriel.

1. Sélectionnez l’onglet « Gestion » et vérifiez que vous avez bien un compte de stockage des diagnostics. Si vous n’en avez pas, sélectionnez « Créer », donnez un nom à votre nouveau compte, puis sélectionnez « OK ».

    :::image type="content" source="../media/disk-encryption/portal-quickstart-vm-creation-storage.png" alt-text="Écran de la création du groupe de ressources":::

1. Cliquez sur « Vérifier + créer ».
1. Sur la page **Create a virtual machine** (Créer une machine virtuelle), vous pouvez voir les détails de la machine virtuelle que vous allez créer. Lorsque vous êtes prêt, sélectionnez **Créer**.

Quelques minutes sont nécessaires pour le déploiement de votre machine virtuelle. Lorsque le déploiement est terminé, passez à la section suivante.

## <a name="encrypt-the-virtual-machine"></a>Chiffrer la machine virtuelle

1. Une fois le déploiement de la machine virtuelle terminé, sélectionnez **Accéder à la ressource**.
1. Dans la barre latérale de gauche, sélectionnez **Disques**.
1. Dans la barre en haut, sélectionnez **Paramètres supplémentaires**.
1. Sous **Paramètres de chiffrement** > **Disques à chiffrer** , sélectionnez **Disques de système d’exploitation et de données**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-disks-to-encryption.png" alt-text="Capture d’écran montrant le système d’exploitation et les disques de données.":::

1. Sous **Paramètres de chiffrement** , choisissez **Sélectionner un coffre de clés et une clé de chiffrement**.
1. Dans l’écran **Sélectionner une clé dans Azure Key Vault** , sélectionnez **Créer**.

    :::image type="content" source="../media/disk-encryption/portal-qs-keyvault-create.png" alt-text="Capture d’écran montrant l’option Créer.":::

1. À gauche de **Coffre de clés et clé** , sélectionnez **Cliquer pour sélectionner une clé**.
1. Dans l’écran **Sélectionner une clé dans Azure Key Vault** , sous le champ **Key Vault** , sélectionnez **Créer**.
1. Dans l’écran **Créer un coffre de clés** , assurez-vous que le groupe de ressources est *myResourceGroup* et nommez votre coffre de clés.  Chaque coffre de clés dans Azure doit avoir un nom unique.
1. Sous l’onglet **Stratégies d’accès** , cochez la case **Azure Disk Encryption pour chiffrer des volumes**.

    :::image type="content" source="../media/disk-encryption/portal-quickstart-keyvault-enable.png" alt-text="sélection dans Disques et Chiffrement":::

1. Sélectionnez **Revoir + créer**.  
1. Une fois que le coffre de clés a été validé, sélectionnez **Créer**. Ceci vous ramène à l’écran **Sélectionner une clé dans Azure Key Vault**.
1. Laissez le champ **Clé**  vide et choisissez **Sélectionner**.
1. En haut de l’écran de chiffrement, cliquez sur **Enregistrer**. Un message s’affiche pour vous avertir que la machine virtuelle va redémarrer. Cliquez sur **Oui**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Dès que vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources, la machine virtuelle et toutes les ressources associées. Pour cela, sélectionnez le groupe de ressources de la machine virtuelle, sélectionnez Supprimer, puis confirmez le nom du groupe de ressources à supprimer.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un coffre de clés qui a été activé pour les clés de chiffrement, vous avez créé une machine virtuelle, puis vous avez activé le chiffrement pour la machine virtuelle.  

> [!div class="nextstepaction"]
> [Vue d’ensemble d’Azure Disk Encryption](disk-encryption-overview.md)
---
title: Réinitialiser les Services Bureau à distance ou le mot de passe administrateur associé dans une machine virtuelle Windows | Microsoft Docs
description: Découvrez comment réinitialiser un mot de passe de compte ou des services Bureau à distance sur une machine virtuelle Windows à l’aide du Portail Azure ou d’Azure PowerShell.
services: virtual-machines-windows
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 45c69812-d3e4-48de-a98d-39a0f5675777
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 03/25/2019
ms.author: genli
ms.openlocfilehash: 80bcac36271d86334e11cb3cd161cd96777e6fe5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87832448"
---
# <a name="reset-remote-desktop-services-or-its-administrator-password-in-a-windows-vm"></a>Réinitialiser les Services Bureau à distance ou le mot de passe administrateur associé dans une machine virtuelle Windows
Si vous ne pouvez pas vous connecter à une machine virtuelle Windows, vous pouvez réinitialiser le mot de passe d’administrateur local ou la configuration du service Bureau à distance (pas de prise en charge sur les contrôleurs de domaine Windows). Pour réinitialiser le mot de passe, utilisez le Portail Azure ou l’extension d’accès aux machines virtuelles dans Azure PowerShell. Une fois connecté à la machine virtuelle, réinitialisez le mot de passe de cet administrateur local.  
Si vous utilisez PowerShell, assurez-vous d’avoir [installé et configuré le dernier module PowerShell](/powershell/azure/) et d’être connecté à votre abonnement Azure. Vous pouvez également [effectuer ces étapes pour les machines virtuelles créées à l’aide du modèle de déploiement classique](/previous-versions/azure/virtual-machines/windows/classic/reset-rdp).

Vous pouvez réinitialiser les informations d’identification et les Services Bureau à distance de plusieurs manières :

- [Réinitialisation en utilisant le portail Azure](#reset-by-using-the-azure-portal)

- [Réinitialisation en utilisant l’extension VMAccess et PowerShell](#reset-by-using-the-vmaccess-extension-and-powershell)

## <a name="reset-by-using-the-azure-portal"></a>Réinitialisation en utilisant le portail Azure

Commencez par vous connecter au [Portail Azure](https://portal.azure.com), puis, dans le menu de gauche, sélectionnez **Machines virtuelles**. 

### <a name="reset-the-local-administrator-account-password"></a>**Réinitialiser le mot de passe de compte d’administrateur local**

1. Sélectionnez votre machine virtuelle Windows, puis **Réinitialiser le mot de passe** sous **Support + dépannage**. La fenêtre **Réinitialiser le mot de passe** s’affiche.

2. Sélectionnez **Réinitialiser le mot de passe**, entrez un nom d’utilisateur et un mot de passe, puis sélectionnez **Mettre à jour**. 

3. Essayez à nouveau de vous connecter à votre machine virtuelle.

### <a name="reset-the-remote-desktop-services-configuration"></a>**Réinitialiser la configuration des Services Bureau à distance**

Ce processus a pour effet d’activer le service Bureau à distance dans la machine virtuelle et de créer une règle de pare-feu pour le port RDP par défaut (3389).

1. Sélectionnez votre machine virtuelle Windows, puis **Réinitialiser le mot de passe** sous **Support + dépannage**. La fenêtre **Réinitialiser le mot de passe** s’affiche. 

2. Sélectionnez **Réinitialiser la configuration uniquement** , puis sélectionnez **Mettre à jour**. 

3. Essayez à nouveau de vous connecter à votre machine virtuelle.

## <a name="reset-by-using-the-vmaccess-extension-and-powershell"></a>Réinitialisation en utilisant l’extension VMAccess et PowerShell

Commencez par vérifier que le [dernier module PowerShell est installé et configuré](/powershell/azure/) et que vous êtes connecté à votre abonnement Azure à l’aide de l’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount).

### <a name="reset-the-local-administrator-account-password"></a>**Réinitialiser le mot de passe de compte d’administrateur local**

- Réinitialisez le mot de passe ou le nom d’utilisateur de l’administrateur à l’aide de l’applet de commande PowerShell [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension). La version du paramètre `typeHandlerVersion` doit être 2.0 ou ultérieure, car la version 1 est déconseillée. 

    ```powershell
    $SubID = "<SUBSCRIPTION ID>" 
    $RgName = "<RESOURCE GROUP NAME>" 
    $VmName = "<VM NAME>" 
    $Location = "<LOCATION>" 
 
    Connect-AzAccount 
    Select-AzSubscription -SubscriptionId $SubID 
    Set-AzVMAccessExtension -ResourceGroupName $RgName -Location $Location -VMName $VmName -Credential (get-credential) -typeHandlerVersion "2.0" -Name VMAccessAgent 
    ```

    > [!NOTE] 
    > Si vous saisissez un nom différent de celui du compte d’administrateur local actuel sur votre machine virtuelle, l’extension VMAccess ajoute un compte d’administrateur local avec ce nom et affecte le mot de passe spécifié à ce compte. Si le compte d’administrateur local sur votre machine virtuelle est désactivé, l’extension VMAccess réinitialise le mot de passe. Si le compte est désactivé, l’extension VMAccess l’active.

### <a name="reset-the-remote-desktop-services-configuration"></a>**Réinitialiser la configuration des Services Bureau à distance**

1. Réinitialisez l’accès à distance à votre machine virtuelle avec l’applet de commande PowerShell [Set-AzVMAccessExtension](/powershell/module/az.compute/set-azvmaccessextension). L’exemple suivant réinitialise l’extension d’accès nommée `myVMAccess`, sur la machine virtuelle nommée `myVM`, dans le groupe de ressources nommé `myResourceGroup` :

    ```powershell
    Set-AzVMAccessExtension -ResourceGroupName "myResoureGroup" -VMName "myVM" -Name "myVMAccess" -Location WestUS -typeHandlerVersion "2.0" -ForceRerun
    ```

    > [!TIP]
    > En toutes circonstances, une machine virtuelle ne peut avoir qu’un seul agent d’accès de machine virtuelle. Pour définir les propriétés de l’agent d’accès aux machines virtuelles, utilisez l’option `-ForceRerun`. Lorsque vous utilisez `-ForceRerun`, vérifiez que vous utilisez le même nom pour l’agent d’accès à la machine virtuelle que vous avez peut-être utilisé dans une commande précédente.

1. Si vous ne pouvez toujours pas vous connecter à distance à votre machine virtuelle, consultez [Résolution des problèmes de connexion Bureau à distance avec une machine virtuelle Azure exécutant Windows](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Si vous perdez la connexion au contrôleur de domaine Windows, vous devez le restaurer à partir d’une sauvegarde de contrôleur de domaine.

## <a name="next-steps"></a>Étapes suivantes

- Si l’extension d’accès aux machines virtuelles Azure ne répond pas et que vous ne pouvez pas réinitialiser le mot de passe, vous pouvez [réinitialiser le mot de passe Windows local en mode hors connexion](reset-local-password-without-agent.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). Cette méthode est plus avancée et nécessite que vous connectiez le disque dur virtuel de la machine virtuelle problématique à une autre machine virtuelle. Suivez d’abord la procédure décrite dans cet article et n’utilisez la méthode de réinitialisation du mot de passe en mode hors connexion qu’en cas d’échec de cette procédure.

- [En savoir plus sur les extensions et fonctionnalités des machines virtuelles Azure](../extensions/features-windows.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)

- [Connecter une machine virtuelle Azure avec RDP ou SSH](/previous-versions/azure/dn535788(v=azure.100))

- [Résoudre les problèmes de connexion Bureau à distance à une machine virtuelle Azure exécutant Windows](troubleshoot-rdp-connection.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json)
---
title: Configurer des identités managées sur des groupes de machines virtuelles identiques à l’aide de PowerShell – Azure AD
description: Instructions pas à pas pour configurer des identités managées affectées par le système et l’utilisateur sur un groupe de machines virtuelles identiques en utilisant PowerShell.
services: active-directory
documentationcenter: ''
author: barclayn
manager: daveba
editor: ''
ms.service: active-directory
ms.subservice: msi
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 12/15/2020
ms.author: barclayn
ms.collection: M365-identity-device-management
ms.openlocfilehash: bd7ec1f6cdfc8ecfe32de04f5d06b42f9492b88c
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590900"
---
# <a name="configure-managed-identities-for-azure-resources-on-virtual-machine-scale-sets-using-powershell"></a>Configurer des identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant PowerShell

[!INCLUDE [preview-notice](../../../includes/active-directory-msi-preview-notice.md)]

Les identités managées pour ressources Azure fournissent automatiquement aux services Azure une identité managée dans Azure Active Directory. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code. 

Cet article explique comment effectuer les opérations d’identités managées pour ressources Azure sur un groupe de machines virtuelles identiques en utilisant PowerShell :

- Activer et désactiver l’identité managée affectée par le système sur un groupe de machines virtuelles identiques
- Ajouter et supprimer une identité managée affectée par l’utilisateur sur un groupe de machines virtuelles identiques

[!INCLUDE [az-powershell-update](../../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

- Si vous n’êtes pas familiarisé avec les identités managées pour ressources Azure, consultez la [section Vue d’ensemble](overview.md). **Veillez à lire la [différence entre les identités managées affectées par le système et celles affectées par l’utilisateur](overview.md#managed-identity-types)** .

- Si vous n’avez pas encore de compte Azure, [inscrivez-vous à un essai gratuit](https://azure.microsoft.com/free/) avant de continuer.

- Pour effectuer les opérations de gestion dans cet article, votre compte doit disposer de ces affectations de contrôle d’accès basé sur les rôles Azure :

    > [!NOTE]
    > Aucune attribution de rôle d'annuaire Azure AD supplémentaire n’est requise.

    - [Contributeur de machines virtuelles](../../role-based-access-control/built-in-roles.md#virtual-machine-contributor) pour créer un groupe de machines virtuelles identiques, puis activer et supprimer une identité managée affectée par le système et/ou par l’utilisateur dans un groupe de machines virtuelles identiques.
    - [Contributeur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-contributor) pour créer une identité managée affectée par l’utilisateur.
    - [Opérateur d’identité managée](../../role-based-access-control/built-in-roles.md#managed-identity-operator) pour attribuer et supprimer une identité managée affectée par l’utilisateur dans un groupe de machines virtuelles identiques.

- Pour exécuter les exemples de scripts, vous avez deux options :
    - Utiliser [Azure Cloud Shell](../../cloud-shell/overview.md), que vous pouvez ouvrir en utilisant le bouton **Essayer** dans le coin supérieur droit des blocs de code.
    - Exécuter les scripts localement en installant la dernière version d’[Azure PowerShell](/powershell/azure/install-az-ps), puis en vous connectant à Azure en utilisant `Connect-AzAccount`. 

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

Dans cette section, vous allez découvrir comment activer et supprimer une identité managée affectée par le système en utilisant Azure PowerShell.

### <a name="enable-system-assigned-managed-identity-during-the-creation-of-an-azure-virtual-machine-scale-set"></a>Activer une identité managée affectée par le système lors de la création d’un groupe de machines virtuelles identiques Azure

Pour créer un groupe de machines virtuelles identiques avec l’identité managée affectée par le système activée :

1. Pour créer un groupe de machines virtuelles identiques avec une identité managée affectée par le système, voir l’*Exemple 1* dans l’article de référence sur l’applet de commande [New-AzVmssConfig](/powershell/module/az.compute/new-azvmssconfig).  Ajoutez le paramètre `-IdentityType SystemAssigned` à la cmdlet `New-AzVmssConfig` :

    ```azurepowershell-interactive
    $VMSS = New-AzVmssConfig -Location $Loc -SkuCapacity 2 -SkuName "Standard_A0" -UpgradePolicyMode "Automatic" -NetworkInterfaceConfiguration $NetCfg -IdentityType SystemAssigned`
    ```

### <a name="enable-system-assigned-managed-identity-on-an-existing-azure-virtual-machine-scale-set"></a>Activer une identité managée affectée par le système sur un groupe de machines virtuelles identiques Azure existant

Si vous devez activer une identité managée affectée par le système sur un groupe de machines virtuelles identiques Azure existant :

1. Vérifiez que le compte Azure que vous utilisez appartient à un rôle qui vous donne des autorisations en écriture sur le groupe de machines virtuelles identiques, comme « Contributeur de machines virtuelles ».
   
1. Récupérez les propriétés du groupe de machines virtuelles identiques en utilisant l’applet de commande [`Get-AzVmss`](/powershell/module/az.compute/get-azvmss). Ensuite, pour activer une identité managée affectée par le système, utilisez le commutateur `-IdentityType` avec l’applet de commande [Update-AzVmss](/powershell/module/az.compute/update-azvmss) :

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name -myVmss -IdentityType "SystemAssigned"
   ```

### <a name="disable-the-system-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Désactiver l’identité managée affectée par le système d’un groupe de machines virtuelles identiques Azure

Si vous disposez d’un groupe de machines virtuelles identiques qui n’a plus besoin de l’identité managée affectée par le système, mais qui a toujours besoin d’identités managées affectées par l’utilisateur, utilisez l’applet de commande suivante :

1. Vérifiez que votre compte appartient à un rôle qui vous donne des autorisations en écriture sur le groupe de machines virtuelles identiques, comme « Contributeur de machines virtuelles ».

1. Exécutez l’applet de commande suivante :

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "UserAssigned"
   ```

1. Si vous avez un groupe de machines virtuelles identiques qui n’a plus besoin de l’identité managée affectée par le système et qui n’a pas d’identité managée affectée par l’utilisateur, utilisez la commande suivante :

    ```azurepowershell-interactive
    Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
    ```
    
## <a name="user-assigned-managed-identity"></a>Identité managée affectée par l’utilisateur

Dans cette section, vous allez découvrir comment ajouter et supprimer une identité managée affectée par l’utilisateur dans un groupe de machines virtuelles identiques en utilisant Azure PowerShell.

### <a name="assign-a-user-assigned-managed-identity-during-creation-of-an-azure-virtual-machine-scale-set"></a>Attribuer une identité managée affectée par l’utilisateur lors de la création d’un groupe de machines virtuelles identiques Azure

La création d’un groupe de machines virtuelles identiques avec une identité managée affectée par l’utilisateur via PowerShell n’est pas prise en charge actuellement. Pour savoir comment ajouter une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques existant, lisez la section suivante. Revenez ultérieurement pour des mises à jour.

### <a name="assign-a-user-assigned-managed-identity-to-an-existing-azure-virtual-machine-scale-set"></a>Attribuer une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques Azure existant

Pour attribuer une identité managée affectée par l’utilisateur à un groupe de machines virtuelles identiques Azure existant :

1. Vérifiez que votre compte appartient à un rôle qui vous donne des autorisations en écriture sur le groupe de machines virtuelles identiques, comme « Contributeur de machines virtuelles ».

1. Récupérez les propriétés du groupe de machines virtuelles identiques en utilisant l’applet de commande `Get-AzVM`. Ensuite, pour attribuer une identité managée affectée par l’utilisateur au groupe de machines virtuelles identiques, utilisez les commutateurs `-IdentityType` et `-IdentityID` avec l’applet de commande [Update-AzVmss](/powershell/module/az.compute/update-azvmss). Remplacez `<VM NAME>`, `<SUBSCRIPTION ID>`, `<RESROURCE GROUP>`, `<USER ASSIGNED ID1>` et `USER ASSIGNED ID2` par vos propres valeurs.

   [!INCLUDE [ua-character-limit](~/includes/managed-identity-ua-character-limits.md)]

   ```azurepowershell-interactive
   Update-AzVmss -ResourceGroupName <RESOURCE GROUP> -Name <VMSS NAME> -IdentityType UserAssigned -IdentityID "<USER ASSIGNED ID1>","<USER ASSIGNED ID2>"
   ```

### <a name="remove-a-user-assigned-managed-identity-from-an-azure-virtual-machine-scale-set"></a>Supprimer d’un groupe de machines virtuelles identiques Azure une identité managée affectée par l’utilisateur

Si votre groupe de machines virtuelles identiques dispose de plusieurs identités managées affectées par l’utilisateur, vous pouvez toutes les supprimer, sauf la dernière, au moyen des commandes suivantes. N’oubliez pas de remplacer les valeurs des paramètres `<RESOURCE GROUP>` et `<VIRTUAL MACHINE SCALE SET NAME>` par vos propres valeurs. `<USER ASSIGNED IDENTITY NAME>` est la propriété de nom de l’identité managée affectée par l’utilisateur, qui doit rester sur le groupe de machines virtuelles identiques. Ces informations sont accessibles dans la section d’identité du groupe de machines virtuelles identiques à l’aide de `az vmss show` :

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType UserAssigned -IdentityID "<USER ASSIGNED IDENTITY NAME>"
```
Si votre groupe de machines virtuelles identiques ne dispose pas d’une identité managée affectée par le système, et souhaitez supprimer toutes les identités managées affectées par l’utilisateur qu’elle contient, utilisez la commande suivante :

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType None
```
Si votre groupe de machines virtuelles identiques dispose à la fois d’identités managées affectées par l’utilisateur et d’identités affectées par le système, vous pouvez supprimer toutes les identités managées affectées par l’utilisateur en choisissant de n’utiliser que l’identité managée affectée par le système.

```azurepowershell-interactive
Update-AzVmss -ResourceGroupName myResourceGroup -Name myVmss -IdentityType "SystemAssigned"
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des identités managées pour les ressources Azure](overview.md)
- Pour obtenir les guides de démarrages rapides complets sur la création de machines virtuelles Azure, consultez :
  
  - [Créer une machine virtuelle Windows avec PowerShell](../../virtual-machines/windows/quick-create-powershell.md) 
  - [Créer une machine virtuelle Linux avec PowerShell](../../virtual-machines/linux/quick-create-powershell.md)

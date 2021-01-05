---
title: Attribuer une stratégie d’accès Azure Key Vault
description: Comment utiliser le Portail Azure, Azure CLI ou Azure PowerShell pour attribuer une stratégie d’accès Key Vault à un principal de service ou à une identité d’application.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: azure-resource-manager
ms.service: key-vault
ms.subservice: general
ms.topic: how-to
ms.date: 08/27/2020
ms.author: mbaldwin
ms.openlocfilehash: fb28b4b678b37f69331b2ecff6272fd7aa64d191
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93287616"
---
# <a name="assign-a-key-vault-access-policy-using-azure-powershell"></a>Attribuer une stratégie d’accès Key Vault à l’aide d’Azure PowerShell

Une stratégie d’accès Key Vault détermine si un principal de service donné, à savoir une application ou un groupe d’utilisateurs, peut effectuer différentes opérations sur des [secrets](../secrets/index.yml), [clés](../keys/index.yml) et [certificats](../certificates/index.yml) de Key Vault. Vous pouvez attribuer des stratégies d’accès à l’aide du portail Azure, d’[Azure CLI](assign-access-policy-portal.md) ou d’[Azure PowerShell](assign-access-policy-cli.md) (cet article).

[!INCLUDE [key-vault-access-policy-limits.md](../../../includes/key-vault-access-policy-limits.md)]

Pour plus d’informations sur la création de groupes dans Azure Active Directory à l’aide d’Azure PowerShell, consultez [New-AzureADGroup](/powershell/module/azuread/new-azureadgroup) et [Add-AzADGroupMember](/powershell/module/az.resources/add-azadgroupmember).

## <a name="configure-powershell-and-sign-in"></a>Configurer PowerShell et se connecter

1. Pour exécuter des commandes localement, installez [Azure PowerShell](/powershell/azure/) si ce n’est déjà fait.

    Pour exécuter des commandes directement dans le cloud, utilisez [Azure Cloud Shell](../../cloud-shell/overview.md).

1. PowerShell local uniquement :

    1. Installez le [module Azure Active Directory PowerShell](https://www.powershellgallery.com/packages/AzureAD).

    1. Connectez-vous à Azure :

        ```azurepowershell-interactive
        Login-AzAccount
        ```
    
## <a name="acquire-the-object-id"></a>Acquérir l’ID d’objet

Déterminez l’ID d’objet de l’application, du groupe ou de l’utilisateur auquel vous souhaitez attribuer la stratégie d’accès :

- Applications et autres principaux de service : utilisez la cmdlet [Get-AzADServicePrincipal](/powershell/module/az.resources/get-azadserviceprincipal) avec le paramètre `-SearchString` pour filtrer les résultats sur le nom du principal de service souhaité :

    ```azurepowershell-interactive
    Get-AzADServicePrincipal -SearchString <search-string>
    ```

- Groupes : utilisez la cmdlet [Get-AzADGroup](/powershell/module/az.resources/get-azadgroup?view=azps-2.7.0) avec le paramètre `-SearchString` pour filtrer les résultats sur le nom du groupe souhaité :

    ```azurepowershell-interactive
    Get-AzADGroup -SearchString <search-string>
    ```
    
    Dans la sortie, l’ID d’objet est indiqué en tant que `Id`.

- Utilisateurs : utilisez la cmdlet [AzADUser](/powershell/module/az.resources/get-azaduser), en transmettant l’adresse e-mail de l’utilisateur au paramètre `-UserPrincipalName`.

    ```azurepowershell-interactive
     Get-AzAdUser -UserPrincipalName <email-address-of-user>
    ```

    Dans la sortie, l’ID d’objet est indiqué en tant que `Id`.

## <a name="assign-the-access-policy"></a>Attribuer la stratégie d’accès

Utilisez la cmdlet [Set-AzKeyVaultAccessPolicy](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy) pour attribuer la stratégie d’accès :

```azurepowershell-interactive
Set-AzKeyVaultAccessPolicy -VaultName <key-vault-name> -ObjectId <Id> -PermissionsToSecrets <secrets-permissions> -PermissionsToKeys <keys-permissions> -PermissionsToCertificates <certificate-permissions    
```

Vous avez uniquement besoin d’inclure `-PermissionsToSecrets`, `-PermissionsToKeys` et `-PermissionsToCertificates` lors de l’attribution d’autorisations à ces types particuliers. Les valeurs autorisées pour `<secret-permissions>`, `<key-permissions>` et `<certificate-permissions>` sont indiquées dans la documentation [Set-AzKeyVaultAccessPolicy – Paramètres](/powershell/module/az.keyvault/set-azkeyvaultaccesspolicy#parameters).

## <a name="next-steps"></a>Étapes suivantes

- [Sécurité d’Azure Key Vault : Gestion des identités et des accès](overview-security.md#identity-and-access-management)
- [Sécuriser votre coffre de clés](secure-your-key-vault.md)
- [Guide du développeur Azure Key Vault](developers-guide.md)
- [Bonnes pratiques relatives à Azure Key Vault](best-practices.md)
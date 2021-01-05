---
title: Contrôler l’accès aux partages de fichiers Azure - Authentification AD DS en local
description: Découvrez comment affecter des autorisations à une identité Active Directory Domain Services qui représente votre compte de stockage. Vous pouvez ainsi contrôler l’accès avec l’authentification basée sur l’identité.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: af88f0b3403fb80acbb7dacebe293ac583e35799
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91716045"
---
# <a name="part-two-assign-share-level-permissions-to-an-identity"></a>Deuxième partie : affecter des autorisations au niveau du partage à une identité

Avant de commencer cet article, veillez à lire le précédent intitulé [Activer l’authentification AD DS pour votre compte](storage-files-identity-ad-ds-enable.md).

Une fois que vous avez activé l’authentification AD DS (Active Directory Domain Services) sur votre compte de stockage, vous devez configurer les autorisations au niveau du partage pour pouvoir accéder à vos partages de fichiers. L’identité avec laquelle vous souhaitez accéder aux ressources de partage de fichiers Azure doit être une identité hybride qui existe à la fois dans AD DS et Azure AD. Par exemple, vous avez dans votre AD DS un utilisateur user1@onprem.contoso.com que vous avez synchronisé avec Azure AD en tant que user1@contoso.com à l’aide d’Azure AD Connect Sync. Pour permettre à cet utilisateur d’accéder à Azure Files, vous devez affecter les autorisations au niveau du partage à user1@contoso.com. Le même concept s’applique aux groupes ou principaux de service. Pour cette raison, vous devez synchroniser les utilisateurs et les groupes de votre AD DS avec Azure AD à l’aide d’Azure AD Connect Sync. 

Les autorisations au niveau du partage doivent être affectées à l’identité Azure AD représentant le même utilisateur ou groupe dans votre AD DS pour prendre en charge l’authentification AD DS auprès de votre partage de fichiers Azure. L’authentification et l’autorisation sur des identités qui existent uniquement dans Azure AD, notamment les identités managées Azure (MSI), ne sont pas prises en charge avec l’authentification AD DS. Cet article explique comment affecter des autorisations au niveau du partage pour un partage de fichiers à une identité.


## <a name="share-level-permissions"></a>Autorisations au niveau du partage

En général, nous vous recommandons d’utiliser des autorisations au niveau du partage pour la gestion de l’accès de haut niveau pour un groupe Azure AD représentant un groupe d’utilisateurs et d’identités, puis à tirer parti des listes de contrôle d’accès Windows pour un contrôle d’accès précis au niveau des répertoires/fichiers. 

Trois rôles intégrés Azure sont disponibles pour accorder aux utilisateurs des autorisations au niveau du partage :

- **Lecteur de partage SMB de données de fichier de stockage** autorise l’accès en lecture aux partages de fichiers Stockage Azure sur SMB.
- **Collaborateur de partage SMB de données de fichier de stockage** autorise l’accès en lecture, écriture et suppression aux partages de fichiers Stockage Azure sur SMB.
- **Contributeur élevé de partage SMB de données de fichier de stockage** autorise l’accès en lecture, écriture, suppression et modification aux listes de contrôle d’accès Windows dans les partages de fichiers Stockage Azure sur SMB.

> [!IMPORTANT]
> Le contrôle administratif total d’un partage de fichiers, y compris la possibilité de prendre possession d’un fichier, nécessite l’utilisation de la clé de compte de stockage. Le contrôle administratif n'est pas pris en charge avec les informations d'identification Azure AD.

Vous pouvez utiliser le portail Azure, Azure PowerShell ou Azure CLI pour attribuer les rôles intégrés à l’identité Azure AD d’un utilisateur afin d’accorder des autorisations au niveau du partage.

## <a name="assign-an-azure-role"></a>Affecter un rôle Azure

### <a name="azure-portal"></a>Portail Azure

Pour affecter un rôle Azure à une identité Azure AD par le biais du [portail Azure](https://portal.azure.com), suivez ces étapes :

1. Dans le portail Azure, accédez à votre partage de fichiers ou [créez un partage de fichiers](storage-how-to-create-file-share.md).
1. Sélectionnez **Contrôle d’accès (IAM)** .
1. Sélectionnez **Ajouter une attribution de rôle**.
1. Dans le panneau **Ajouter une attribution de rôle**, sélectionnez le rôle intégré approprié (Lecteur de partage SMB de données de fichier de stockage, Collaborateur de partage SMB de données de fichier de stockage) dans la liste **Rôle**. Conservez le paramètre par défaut **Attribuer l’accès à** : **Utilisateur, groupe ou principal du service Azure AD**. Sélectionnez l’identité Azure AD cible par son nom ou son adresse e-mail. **L’identité Azure AD sélectionnée doit être une identité hybride et ne peut pas être une identité cloud uniquement.** Cela signifie que la même identité est également représentée dans AD DS.
1. Sélectionnez **Enregistrer** pour terminer l’opération d’attribution de rôle.

### <a name="powershell"></a>PowerShell

L’exemple PowerShell suivant montre comment affecter un rôle Azure à une identité Azure AD, en fonction du nom de connexion. Pour plus d’informations sur l’attribution de rôles Azure avec PowerShell, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du module Azure PowerShell](../../role-based-access-control/role-assignments-powershell.md).

Avant d’exécuter l’exemple de script suivant, remplacez les valeurs d’espace réservé, y compris les crochets, par vos valeurs.

```powershell
#Get the name of the custom role
$FileShareContributorRole = Get-AzRoleDefinition "<role-name>" #Use one of the built-in roles: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

### <a name="cli"></a>Interface de ligne de commande
  
La commande CLI 2.0 suivante affecte un rôle Azure à une identité Azure AD, selon le nom de connexion. Pour plus d’informations sur l’attribution de rôles Azure avec Azure CLI, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide de l’interface de ligne de commande Azure](../../role-based-access-control/role-assignments-cli.md). 

Avant d’exécuter l’exemple de script suivant, n’oubliez pas de remplacer les valeurs d’espace réservé, y compris les crochets, par vos propres valeurs.

```azurecli-interactive
#Assign the built-in role to the target identity: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
az role assignment create --role "<role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez affecté des autorisations au niveau du partage, vous devez configurer les autorisations au niveau des répertoires et des fichiers. Passez à l’article suivant.

[Troisième partie : configurer les autorisations au niveau des répertoires et des fichiers sur SMB](storage-files-identity-ad-ds-configure-permissions.md)

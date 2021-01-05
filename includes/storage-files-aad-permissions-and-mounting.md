---
title: Fichier include
description: Fichier include
services: storage
author: roygara
ms.service: storage
ms.topic: include
ms.date: 08/26/2020
ms.author: rogara
ms.custom: include file
ms.openlocfilehash: 4773446ec0007ffbed99bc01939d1f92f5823d99
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95561085"
---
## <a name="assign-access-permissions-to-an-identity"></a>Assigner des autorisations d’accès à une identité

Pour pouvoir accéder à des ressources Azure Files avec l’authentification par identité, il faut que l’identité (utilisateur, groupe ou principal de service) dispose des autorisations nécessaires au niveau du partage. Ce processus est semblable à la spécification des autorisations de partage Windows, où vous spécifiez le type d’accès dont dispose un utilisateur particulier sur un partage de fichiers. Les instructions dans cette section montrent comment assigner des autorisations de lecture, d’écriture ou de suppression pour un partage de fichiers à une identité. 

Nous avons introduit trois rôles intégrés Azure pour accorder aux utilisateurs des autorisations au niveau du partage :

- **Lecteur de partage SMB de données de fichier de stockage** autorise l’accès en lecture aux partages de fichiers Stockage Azure sur SMB.
- **Collaborateur de partage SMB de données de fichier de stockage** autorise l’accès en lecture, écriture et suppression aux partages de fichiers Stockage Azure sur SMB.
- **Contributeur élevé de partage SMB de données de fichier de stockage** autorise l’accès NTFS en lecture, écriture, suppression et modification aux partages de fichiers Stockage Azure sur SMB.

> [!IMPORTANT]
> Le contrôle administratif total d’un partage de fichiers, y compris la possibilité de prendre possession d’un fichier, nécessite l’utilisation de la clé de compte de stockage. Le contrôle administratif n'est pas pris en charge avec les informations d'identification Azure AD.

Vous pouvez utiliser le portail Azure, PowerShell ou Azure CLI pour attribuer les rôles intégrés à l’identité Azure AD d’un utilisateur afin d’accorder des autorisations au niveau du partage. N’oubliez pas que l’attribution de rôle Azure au niveau du partage peut prendre un certain temps. 

> [!NOTE]
> N’oubliez pas de [synchroniser vos informations d’identification AD DS avec Azure AD](../articles/active-directory/hybrid/how-to-connect-install-roadmap.md) si vous envisagez d’utiliser AD DS localement pour l’authentification. La synchronisation du hachage de mot de passe d’AD DS vers Azure AD est facultative. L’autorisation au niveau du partage sera accordée à l’identité Azure AD synchronisée à partir de votre instance AD DS locale.

La recommandation générale est d’utiliser l’autorisation de niveau partage pour la gestion de l’accès de haut niveau pour un groupe AD représentant un groupe d’utilisateurs et d’identités, puis à tirer parti des autorisations NTFS pour un contrôle d’accès précis au niveau de l’annuaire/fichier. 

### <a name="assign-an-azure-role-to-an-ad-identity"></a>Affecter un rôle Azure à une identité AD

# <a name="portal"></a>[Portail](#tab/azure-portal)
Pour affecter un rôle Azure à une identité Azure AD par le biais du [portail Azure](https://portal.azure.com), suivez ces étapes :

1. Sur le Portail Azure, accédez à votre partage de fichiers ou [Créez un partage de fichiers](../articles/storage/files/storage-how-to-create-file-share.md).
2. Sélectionnez **Contrôle d’accès (IAM)** .
3. Sélectionnez **Ajouter une attribution de rôle**.
4. Dans le panneau **Ajouter une attribution de rôle**, sélectionnez le rôle intégré approprié (Lecteur de partage SMB de données de fichier de stockage, Collaborateur de partage SMB de données de fichier de stockage) dans la liste **Rôle**. Conservez le paramètre par défaut **Attribuer l’accès à** : **Utilisateur, groupe ou principal du service Azure AD**. Sélectionnez l’identité Azure AD cible par son nom ou son adresse e-mail.
5. Sélectionnez **Enregistrer** pour terminer l’opération d’attribution de rôle.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

L’exemple PowerShell suivant montre comment affecter un rôle Azure à une identité Azure AD, en fonction du nom de connexion. Pour plus d’informations sur l’affectation de rôles Azure avec PowerShell, consultez [Gérer l’accès à l’aide de RBAC et Azure PowerShell](../articles/role-based-access-control/role-assignments-powershell.md).

Avant d’exécuter l’exemple de script suivant, n’oubliez pas de remplacer les valeurs d’espace réservé, y compris les crochets, par vos propres valeurs.

```powershell
#Get the name of the custom role
$FileShareContributorRole = Get-AzRoleDefinition "<role-name>" #Use one of the built-in roles: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
#Constrain the scope to the target file share
$scope = "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
#Assign the custom role to the target identity with the specified scope.
New-AzRoleAssignment -SignInName <user-principal-name> -RoleDefinitionName $FileShareContributorRole.Name -Scope $scope
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
  
La commande CLI 2.0 suivante montre comment attribuer un rôle Azure à une identité Azure AD, selon le nom de connexion. Pour plus d’informations sur l’affectation de rôles Azure avec Azure CLI, consultez [Gérer l’accès à l’aide de RBAC et Azure CLI](../articles/role-based-access-control/role-assignments-cli.md). 

Avant d’exécuter l’exemple de script suivant, n’oubliez pas de remplacer les valeurs d’espace réservé, y compris les crochets, par vos propres valeurs.

```azurecli-interactive
#Assign the built-in role to the target identity: Storage File Data SMB Share Reader, Storage File Data SMB Share Contributor, Storage File Data SMB Share Elevated Contributor
az role assignment create --role "<role-name>" --assignee <user-principal-name> --scope "/subscriptions/<subscription-id>/resourceGroups/<resource-group>/providers/Microsoft.Storage/storageAccounts/<storage-account>/fileServices/default/fileshares/<share-name>"
```
---

## <a name="configure-ntfs-permissions-over-smb"></a>Configurer les autorisations NTFS sur SMB

Une fois que vous avez attribué des autorisations au niveau du partage avec RBAC, vous devez attribuer les autorisations NTFS appropriées au niveau du fichier, du répertoire ou de la racine. Considérez les autorisations au niveau du partage comme un contrôle d’accès de niveau supérieur qui détermine si un utilisateur peut accéder au partage. En revanche, les autorisations NTFS agissent à un niveau plus granulaire pour déterminer les opérations que l’utilisateur peut effectuer au niveau du répertoire ou du fichier.

Azure Files prend en charge l’ensemble des autorisations NTFS de base et avancées. Vous pouvez voir et configurer les autorisations NTFS sur les répertoires et les fichiers dans un partage de fichiers Azure en montant le partage, puis en utilisant l’Explorateur de fichiers Windows ou en exécutant la commande Windows [icacls](/windows-server/administration/windows-commands/icacls) ou [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl). 

Pour configurer NTFS avec des autorisations de superutilisateur, vous devez monter le partage avec votre clé de compte de stockage à partir de votre machine virtuelle jointe à un domaine. Suivez les instructions de la section suivante pour monter un partage de fichiers Azure à partir de l’invite de commandes et configurer les autorisations NTFS de manière appropriée.

Les ensembles d’autorisations suivants sont pris en charge dans le répertoire racine d’un partage de fichiers :

- BUILTIN\Administrators:(OI)(CI)(F)
- NT AUTHORITY\SYSTEM:(OI)(CI)(F)
- BUILTIN\Users:(RX)
- BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
- NT AUTHORITY\Authenticated Users:(OI)(CI)(M)
- NT AUTHORITY\SYSTEM:(F)
- CREATOR OWNER:(OI)(CI)(IO)(F)

### <a name="mount-a-file-share-from-the-command-prompt"></a>Monter un partage de fichiers Azure à partir de l’invite de commandes

Utilisez la commande Windows **net use** pour monter le partage de fichiers Azure. N’oubliez pas de remplacer les valeurs d’espace réservé dans l’exemple suivant par vos propres valeurs. Pour plus d’informations sur le montage de partages de fichiers, consultez [Utilisation d’un partage de fichiers Azure avec Windows](../articles/storage/files/storage-how-to-use-files-windows.md). 

```
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
 net use <desired-drive letter>: \\<storage-account-name>.file.core.windows.net\<fileshare-name>
} 
else 
{
 Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}

```

Si vous rencontrez des problèmes lors de la connexion à Azure Files, consultez [l’outil de dépannage que nous avons publié pour les erreurs de montage Azure Files sur Windows](https://azure.microsoft.com/blog/new-troubleshooting-diagnostics-for-azure-files-mounting-errors-on-windows/). Nous fournissons également des [conseils](../articles/storage/files/storage-files-faq.md#on-premises-access) pour contourner des situations où le port 445 est bloqué. 


### <a name="configure-ntfs-permissions-with-windows-file-explorer"></a>Configurer des autorisations NTFS avec l’Explorateur de fichiers Windows

Utilisez l’Explorateur de fichiers Windows pour accorder des autorisations complètes sur tous les répertoires et fichiers du partage de fichiers, y compris le répertoire racine.

1. Ouvrez l’Explorateur de fichiers Windows, cliquez avec le bouton droit sur le fichier/répertoire, puis sélectionnez **Propriétés**.
2. Sélectionnez l'onglet **Sécurité** .
3. Sélectionnez **Modifier…** pour modifier les autorisations.
4. Vous pouvez modifier les autorisations des utilisateurs existants ou sélectionner **Ajouter…** pour accorder des autorisations à de nouveaux utilisateurs.
5. Dans la fenêtre d’invite d’ajout de nouveaux utilisateurs, entrez le nom d’utilisateur cible auquel vous souhaitez accorder l’autorisation dans la zone **Entrer les noms des objets à sélectionner**, puis sélectionnez **Vérifier les noms** pour trouver le nom UPN complet de l’utilisateur cible.
7.    Sélectionnez **OK**.
8.    Dans l’onglet **Sécurité**, sélectionnez toutes les autorisations que vous souhaitez accorder au nouvel utilisateur.
9.    Sélectionnez **Appliquer**.

### <a name="configure-ntfs-permissions-with-icacls"></a>Configurer des autorisations NTFS avec icacls

Utilisez la commande Windows suivante pour accorder des autorisations complètes sur tous les répertoires et fichiers du partage de fichiers, y compris le répertoire racine. N’oubliez pas de remplacer les valeurs d’espace réservé dans l’exemple par vos propres valeurs.

```
icacls <mounted-drive-letter>: /grant <user-email>:(f)
```

Pour plus d’informations sur l’utilisation d’icacls afin de définir des autorisations NTFS et sur les différents types d’autorisation pris en charge, voir les [informations de référence sur la ligne de commande pour icacls](/windows-server/administration/windows-commands/icacls).

## <a name="mount-a-file-share-from-a-domain-joined-vm"></a>Monter un partage de fichiers Azure à partir d’une machine virtuelle jointe à un domaine

Le processus suivant vérifie que vos autorisations de partage de fichiers et d’accès ont été configurées correctement, et que vous pouvez accéder à un partage de fichiers Azure à partir d’une machine virtuelle jointe à un domaine. N’oubliez pas que l’attribution de rôle Azure au niveau du partage peut prendre un certain temps. 

Connectez-vous à la machine virtuelle à l’aide de l’identité Azure AD à laquelle vous avez accordé les autorisations, comme illustré dans l’image suivante. Si vous avez activé l’authentification AD DS locale pour Azure Files, utilisez vos informations d’identification AD DS. Pour l’authentification Azure AD DS, connectez-vous avec les informations d’identification Azure AD.

![Capture d’écran montrant l’écran de connexion Azure AD pour l’authentification utilisateur](media/storage-files-aad-permissions-and-mounting/azure-active-directory-authentication-dialog.png)

Utilisez la commande suivante pour monter le partage de fichiers Azure. Remplacez les valeurs d’espace réservé par vos propres valeurs. Comme vous avez déjà effectué une authentification, vous n’avez pas besoin de fournir la clé de compte de stockage ni les informations d’identification AD DS locales ou Azure AD DS. L’expérience d’authentification unique est prise en charge pour l’authentification avec l’instance AD DS locale ou avec Azure AD DS. Si vous rencontrez des problèmes lors du montage avec les informations d’identification AD DS, consultez [Résoudre les problèmes liés à Azure Files sous Windows](../articles/storage/files/storage-troubleshoot-windows-file-connection-problems.md) pour obtenir de l’aide.

```
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
 net use <desired-drive letter>: \\<storage-account-name>.file.core.windows.net\<fileshare-name>
} 
else 
{
 Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN, Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}
```
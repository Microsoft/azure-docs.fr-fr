---
title: Utiliser Azure AD Domain Services pour autoriser l’accès aux données de fichiers via SMB
description: Découvrez comment activer l’authentification basée sur l’identité sur SMB (Server Message Block) pour Azure Files avec Azure Active Directory Domain Services. Vos machines virtuelles Windows jointes à un domaine peuvent alors accéder aux partages de fichiers Azure en utilisant des informations d’identification Azure AD.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 04/21/2020
ms.author: rogarana
ms.subservice: files
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.openlocfilehash: deed7c3dce2d7da1940a8a4871efac7453e2cf60
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97033678"
---
# <a name="enable-azure-active-directory-domain-services-authentication-on-azure-files"></a>Activer l’authentification Azure Active Directory Domain Services sur Azure Files

[Azure Files](storage-files-introduction.md) prend en charge l’authentification basée sur l’identité sur le protocole SMB (Server Message Block) par le biais de deux types de services de domaine : Active Directory Domain Services local (AD DS) et Azure Active Directory Domain Services (Azure AD DS). Nous vous recommandons vivement de consulter la section [Fonctionnement](./storage-files-active-directory-overview.md#how-it-works) pour sélectionner le service de domaine approprié pour l’authentification. L’installation est différente selon le service de domaine que vous choisissez. Cet article se concentre sur l’activation et la configuration d’Azure AD DS pour l’authentification auprès de partages de fichiers Azure.

Si vous débutez avec les partages de fichiers Azure, nous vous recommandons de lire notre [guide de planification](storage-files-planning.md) avant de vous lancer dans les articles suivants.

> [!NOTE]
> Azure Files prend en charge l’authentification Kerberos avec Azure AD DS et le chiffrement RC4-HMAC. Le chiffrement Kerberos AES n’est pas encore pris en charge.
> Azure Files prend en charge l’authentification pour Azure AD DS avec une synchronisation complète avec Azure AD. Si vous avez activé la synchronisation délimitée dans Azure AD DS qui synchronise uniquement un ensemble limité d’identités d’Azure AD, l’authentification et l’autorisation ne sont pas prises en charge.

## <a name="prerequisites"></a>Prérequis

Avant d’activer Azure AD sur SMB pour des partages de fichiers Azure, assurez-vous que les conditions préalables suivantes sont remplies :

1.  **Sélectionner ou créer un locataire Azure AD.**

    Vous pouvez utiliser un locataire nouveau ou existant pour l’authentification Azure AD sur SMB. Le locataire et le partage de fichiers auquel vous voulez accéder doivent être associés au même abonnement.

    Pour créer un locataire Azure AD, vous pouvez [Ajouter un locataire Azure AD et un abonnement Azure AD](/windows/client-management/mdm/add-an-azure-ad-tenant-and-azure-ad-subscription). Si vous avez déjà un locataire Azure AD, mais que vous voulez en créer un pour l’utiliser avec des partages de fichiers Azure, voir [Créer un locataire Azure Active Directory](/rest/api/datacatalog/create-an-azure-active-directory-tenant).

1.  **Activer Azure AD Domain Services sur le locataire Azure AD.**

    Pour prendre en charge l’authentification avec les informations d’identification Azure AD, vous devez activer Azure AD Domain Services pour votre locataire Azure AD. Si vous n’êtes pas l’administrateur du locataire Azure AD, contactez-le et suivez les instructions pas à pas pour [Activer Azure Active Directory Domain Services à l’aide du portail Azure](../../active-directory-domain-services/tutorial-create-instance.md).

    Il faut généralement environ 15 minutes pour déployer Azure AD DS. Vérifiez que l’état d’intégrité d’Azure AD DS est **En cours d’exécution**, avec la synchronisation de hachage de mot de passe activée, avant de passer à l’étape suivante.

1.  **Joindre une machine virtuelle à Azure AD DS.**

    Pour accéder à un partage de fichiers à l’aide des informations d’identification Azure AD à partir d’une machine virtuelle, cette machine virtuelle doit être jointe à Azure AD DS. Pour plus d’informations sur la jonction de domaine d’une machine virtuelle, consultez [Joindre une machine virtuelle Windows Server à un domaine géré](../../active-directory-domain-services/join-windows-vm.md).

    > [!NOTE]
    > L’authentification Azure AD Domain Services sur SMB avec des partages de fichiers Azure est prise en charge uniquement sur des machines virtuelles Azure s’exécutant sur des versions de système d’exploitation postérieures à Windows 7 ou Windows Server 2008 R2.

1.  **Sélectionner ou créer un partage de fichiers Azure.**

    Sélectionnez un partage de fichiers nouveau ou existant associé au même abonnement que celui de votre locataire Azure AD. Pour plus d’informations sur la création d’un partage de fichiers, consultez [Créer un partage de fichiers dans Azure Files](storage-how-to-create-file-share.md).
    Pour des performances optimales, nous recommandons que votre partage de fichiers soit dans la même région que celle de la machine virtuelle à partir de laquelle vous voulez accéder au partage.

1.  **Vérifier la connectivité d’Azure Files en montant les partages de fichiers Azure à l’aide de votre clé de compte de stockage.**

    Pour vérifier que votre machine virtuelle et le partage de fichiers sont configurés correctement, essayez de monter le partage de fichiers à l’aide de votre clé de compte de stockage. Pour plus d’informations, consultez [Monter un partage de fichiers Azure et accéder au partage dans Windows](storage-how-to-use-files-windows.md).

## <a name="regional-availability"></a>Disponibilité régionale

L’authentification Azure Files avec Azure AD DS est disponible dans [toutes les régions publiques et Gov Azure](https://azure.microsoft.com/global-infrastructure/locations/).

## <a name="overview-of-the-workflow"></a>Vue d’ensemble du workflow

Avant d’activer l’authentification Azure AD Domain Services sur SMB pour des partages de fichiers Azure, vérifiez que vos environnements Azure AD et Stockage Azure sont correctement configurés. Nous vous recommandons d’examiner les [conditions préalables](#prerequisites) pour vous assurer que vous avez parcouru toutes les étapes requises.

Ensuite, procédez comme suit pour accorder l’accès aux ressources Azure Files avec les informations d’identification Azure AD :

1. Activez l’authentification Azure AD Domain Services sur SMB pour votre compte de stockage afin d’inscrire le compte de stockage auprès du déploiement Azure AD DS associé.
2. Attribuez des autorisations d’accès à un partage à une identité Azure AD (utilisateur, groupe ou principal de service).
3. Configurez les autorisations NTFS sur SMB pour les fichiers et les répertoires.
4. Montez un partage de fichiers Azure à partir d’une machine virtuelle jointe à un domaine.

Le diagramme ci-dessous illustre le workflow de bout en bout pour l’activation de l’authentification Azure AD Domain Services sur SMB pour Azure Files.

![Diagramme montrant le workflow AD Azure sur SMB pour Azure Files](media/storage-files-active-directory-enable/azure-active-directory-over-smb-workflow.png)

## <a name="enable-azure-ad-ds-authentication-for-your-account"></a>Activer l’authentification Azure AD Domain Services pour votre compte

Pour activer l’authentification Azure AD DS sur SMB pour Azure Files, vous pouvez définir une propriété sur les comptes de stockage à l’aide du Portail Azure, d’Azure PowerShell ou d’Azure CLI. En définissant cette propriété, le « domaine rejoint » implicitement le compte de stockage avec le déploiement Azure AD DS associé. L’authentification Azure AD Domain Services sur SMB est alors activée pour tous les partages de fichiers nouveaux et existants dans le compte de stockage.

N’oubliez pas que vous pouvez activer l’authentification Azure AD DS sur SMB uniquement après avoir correctement déployé Azure AD DS sur votre locataire Azure AD. Pour plus d’informations, voir les [Conditions préalables](#prerequisites).

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour activer l’authentification Azure AD DS sur SMB avec le [portail Azure](https://portal.azure.com), procédez comme suit :

1. Dans le Portail Azure, accédez à votre compte de stockage existant, ou [créez un compte de stockage](../common/storage-account-create.md).
1. Dans la section **Paramètres**, sélectionnez **Configuration**.
1. Sous **Accès basé sur l’identité pour les partages de fichiers**, mettez le bouton bascule **Azure Active Directory Domain Service (AAD DS)** en position **Activé**.
1. Sélectionnez **Enregistrer**.

L’image suivante montre comment activer l’authentification Azure AD DS sur SMB pour votre compte de stockage.

![Activer l’authentification Azure AD DS sur SMB dans le portail Azure](media/storage-files-active-directory-enable/portal-enable-active-directory-over-smb.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour activer l’authentification Azure AD DS sur SMB avec Azure PowerShell, installez le dernier module Azure (version 2.4 ou ultérieure) ou le module Stockage Azure (version 1.5 ou ultérieure). Pour en savoir plus sur l’installation de PowerShell, consultez [Installer Azure PowerShell sur Windows avec PowerShellGet](/powershell/azure/install-Az-ps).

Pour créer un compte de stockage, appelez [New-AzStorageAccount](/powershell/module/az.storage/New-azStorageAccount?view=azps-2.5.0), puis affectez au paramètre **EnableAzureActiveDirectoryDomainServicesForFile** la valeur **true**. Dans l’exemple ci-dessous, remplacez les valeurs d’espace réservé par vos propres valeurs. (Si vous utilisiez le module en préversion précédent, le paramètre pour l’activation de la fonctionnalité est **EnableAzureFilesAadIntegrationForSMB**.)

```powershell
# Create a new storage account
New-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -Location "<azure-region>" `
    -SkuName Standard_LRS `
    -Kind StorageV2 `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```

Pour activer cette fonctionnalité sur les comptes de stockage existants, utilisez la commande suivante :

```powershell
# Update a storage account
Set-AzStorageAccount -ResourceGroupName "<resource-group-name>" `
    -Name "<storage-account-name>" `
    -EnableAzureActiveDirectoryDomainServicesForFile $true
```


# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour activer l’authentification Azure AD sur SMB avec Azure CLI, installez la dernière version de l’interface CLI (version 2.0.70 ou ultérieure). Pour plus d’informations sur l'installation de l’interface de ligne de commande Azure, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli?view=azure-cli-latest).

Pour créer un compte de stockage, appelez [az storage account create](/cli/azure/storage/account?view=azure-cli-latest#az-storage-account-create) et définissez la propriété `--enable-files-aadds` avec la valeur **true**. Dans l’exemple ci-dessous, remplacez les valeurs d’espace réservé par vos propres valeurs. (Si vous utilisiez le module en préversion précédent, le paramètre pour l’activation des fonctionnalités est **file-aad**.)

```azurecli-interactive
# Create a new storage account
az storage account create -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```

Pour activer cette fonctionnalité sur les comptes de stockage existants, utilisez la commande suivante :

```azurecli-interactive
# Update a new storage account
az storage account update -n <storage-account-name> -g <resource-group-name> --enable-files-aadds $true
```
---

[!INCLUDE [storage-files-aad-permissions-and-mounting](../../../includes/storage-files-aad-permissions-and-mounting.md)]

Vous avez maintenant activé l’authentification Azure AD DS sur SMB et attribué un rôle personnalisé qui fournit l’accès à un partage de fichiers avec une identité Azure AD. Pour accorder à d’autres utilisateurs l’accès à votre partage de fichiers, suivez les instructions fournies dans les sections [Assigner des autorisations d’accès](#assign-access-permissions-to-an-identity) pour utiliser une identité et [Configurer les autorisations NTFS sur SMB](#configure-ntfs-permissions-over-smb).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure Files et l’utilisation d’Azure AD sur SMB, voir les ressources suivantes :

- [Vue d’ensemble de la prise en charge de l’authentification basée sur l’identité Azure Files pour l’accès SMB](storage-files-active-directory-overview.md)
- [FORUM AUX QUESTIONS](storage-files-faq.md)
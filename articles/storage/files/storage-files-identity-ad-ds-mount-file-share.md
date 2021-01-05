---
title: Montage d’un partage de fichiers Azure sur une machine virtuelle jointe à Azure AD DS
description: Découvrez comment monter un partage de fichiers sur vos machines jointes à Azure Active Directory Domain Services local.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 06/22/2020
ms.author: rogarana
ms.openlocfilehash: 9807563c768b82c823ff754aaa679ddc917bf62d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87535057"
---
# <a name="part-four-mount-a-file-share-from-a-domain-joined-vm"></a>Partie 4 : Monter un partage de fichiers à partir d’une machine virtuelle jointe à un domaine

Avant de commencer à lire cet article, veillez à parcourir l’article précédent, intitulé [Configurer les autorisations au niveau des répertoires et des fichiers via SMB](storage-files-identity-ad-ds-configure-permissions.md).

Le processus décrit dans le présent article vérifie que vos autorisations de partage de fichiers et d’accès ont été configurées correctement, et que vous pouvez accéder à un partage de fichiers Azure à partir d’une machine virtuelle jointe à un domaine. L’attribution de rôles Azure au niveau du partage peut prendre un certain temps. 

Connectez-vous au client en utilisant les informations d’identification auxquelles vous avez accordé les autorisations adéquates, comme illustré dans l’image suivante.

![Capture d’écran montrant l’écran de connexion Azure AD pour l’authentification utilisateur](media/storage-files-aad-permissions-and-mounting/azure-active-directory-authentication-dialog.png)

## <a name="mounting-prerequisites"></a>Composants requis du montage

Avant de monter le partage de fichiers, assurez-vous que vous avez tenu compte des composants requis ci-après :

- Si vous montez le partage de fichiers à partir d’un client qui l’a précédemment monté à l’aide de votre clé de compte de stockage, assurez-vous que vous avez déconnecté le partage et supprimé les informations d’identification persistantes de la clé de compte de stockage, et que vous utilisez actuellement des informations d’identification relatives à AD DS pour l’authentification.
- Votre client doit pouvoir visualiser votre instance AD DS. Si votre machine ou machine virtuelle ne se trouve pas sur le réseau géré par votre instance AD DS, vous devez activer le VPN de façon à atteindre AD DS à des fins d’authentification.

Remplacez les valeurs d’espace réservé par vos propres valeurs, puis utilisez la commande suivante pour monter le partage de fichiers Azure :

```PSH
# Always mount your share using.file.core.windows.net, even if you setup a private endpoint for your share.
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

Si vous rencontrez des problèmes lors du montage en utilisant les informations d’identification AD DS, consultez la page [Impossible de monter Azure Files avec les informations d’identification AD](storage-troubleshoot-windows-file-connection-problems.md#unable-to-mount-azure-files-with-ad-credentials) pour obtenir de l’aide.

Si le montage de votre partage de fichiers a réussi, cela signifie que vous avez correctement activé et configuré l’authentification AD DS locale pour vos partages de fichiers Azure.

## <a name="next-steps"></a>Étapes suivantes

Si l’identité que vous avez créée dans AD DS pour représenter le compte de stockage se trouve dans un domaine ou une unité d’organisation qui applique la rotation du mot de passe, passez à l’article suivant pour obtenir des instructions sur la mise à jour de votre mot de passe :

[Mettre à jour le mot de passe de l’identité de votre compte de stockage dans AD DS](storage-files-identity-ad-ds-update-password.md)

---
title: Contrôler ce qu’un utilisateur peut faire au niveau du fichier – Partages de fichiers Azure
description: Découvrez comment configurer les autorisations de listes de contrôle d’accès (ACL, access-control list) Windows pour l’authentification AD DS locale sur des partages de fichiers Azure. Vous permettant de tirer parti du contrôle d’accès granulaire.
author: roygara
ms.service: storage
ms.subservice: files
ms.topic: how-to
ms.date: 09/16/2020
ms.author: rogarana
ms.openlocfilehash: 02b8d72ab88f9eca2e1fac4858c14826dae57dbe
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629170"
---
# <a name="part-three-configure-directory-and-file-level-permissions-over-smb"></a>Troisième partie : configurer les autorisations au niveau des répertoires et des fichiers sur SMB 

Avant de commencer à lire cet article, veillez à parcourir l’article précédent, intitulé [Attribuer des autorisations au niveau du partage à une identité](storage-files-identity-ad-ds-assign-permissions.md) pour vous assurer que vos autorisations de partage sont définies.

Une fois que vous avez attribué des autorisations au niveau du partage avec Azure RBAC, vous devez configurer les ACL Windows appropriées au niveau du fichier, du répertoire ou de la racine pour tirer parti du contrôle d’accès granulaire. Considérez les autorisations Azure RBAC au niveau du partage comme un gardien de niveau supérieur qui détermine si un utilisateur peut accéder au partage. En revanche, les ACL Windows fonctionnent à un niveau plus granulaire pour déterminer les opérations que l’utilisateur peut effectuer au niveau du répertoire ou du fichier. Les autorisations au niveau du partage et au niveau du fichier/répertoire sont appliquées lorsqu’un utilisateur tente d’accéder à un fichier/répertoire. Par conséquent, s’il existe une différence entre ces deux niveaux, seules les autorisations dont le niveau est le plus restrictif sont appliquées. Par exemple, si un utilisateur dispose d’un accès en lecture/écriture au niveau du fichier, mais uniquement en lecture au niveau du partage, il peut uniquement lire ce fichier. Il en va de même dans le cas contraire : si un utilisateur avait un accès en lecture/écriture au niveau du partage, mais uniquement en lecture au niveau du fichier, il ne peut toujours que lire le fichier.

## <a name="azure-rbac-permissions"></a>Autorisations Azure RBAC

Le tableau suivant contient les autorisations Azure RBAC associées à cette configuration :


| Rôle intégré  | Autorisations NTFS  | Accès obtenu  |
|---------|---------|---------|
|Lecteur de partage SMB de données de fichier de stockage | Contrôle total, Modifier, Lire, Écrire, Exécuter | Lire et exécuter  |
|     |   Lire |     Lire  |
|Contributeur de partage SMB de données de fichier de stockage  |  Contrôle total    |  Modifier, Lire, Écrire, Exécuter |
|     |  Modifier         |  Modifier    |
|     |  Lire et exécuter |  Lire et exécuter |
|     |  Lire           |  Lire    |
|     |  Write          |  Write   |
|Contributeur élevé de partage SMB de données de fichier de stockage | Contrôle total  |  Modifier, Lire, Écrire, Édition, Exécuter |
|     |  Modifier          |  Modifier |
|     |  Lire et exécuter  |  Lire et exécuter |
|     |  Lire            |  Lire   |
|     |  Write           |  Write  |



## <a name="supported-permissions"></a>Autorisations prises en charge

Azure Files prend en charge l’ensemble des ACL Windows de base et avancées. Vous pouvez voir et configurer les ACL Windows sur les répertoires et les fichiers d’un partage de fichiers Azure en montant le partage puis en utilisant l’Explorateur de fichiers Windows ou en exécutant la commande Windows [icacls](/windows-server/administration/windows-commands/icacls) ou la commande [Set-ACL](/powershell/module/microsoft.powershell.security/set-acl). 

Pour configurer des ACL avec des autorisations de superutilisateur, vous devez monter le partage avec votre clé de compte de stockage à partir de votre machine virtuelle jointe à un domaine. Suivez les instructions de la section suivante pour monter un partage de fichiers Azure à partir de l’invite de commandes et configurer les ACL Windows.

Les autorisations suivantes sont incluses dans le répertoire racine d’un partage de fichiers :

- BUILTIN\Administrators:(OI)(CI)(F)
- BUILTIN\Users:(RX)
- BUILTIN\Users:(OI)(CI)(IO)(GR,GE)
- NT AUTHORITY\Authenticated Users:(OI)(CI)(M)
- NT AUTHORITY\SYSTEM:(OI)(CI)(F)
- NT AUTHORITY\SYSTEM:(F)
- CREATOR OWNER:(OI)(CI)(IO)(F)

|Utilisateurs|Définition|
|---|---|
|BUILTIN\Administrators|Tous les utilisateurs qui sont des administrateurs de domaine de l’environnement local AD DS.
|BUILTIN\Users|Groupe de sécurité intégré dans AD. Il comprend NT AUTHORITY\Authenticated Users par défaut. Pour un serveur de fichiers traditionnel, vous pouvez configurer la définition d’appartenance par serveur. Pour Azure Files, il n’existe pas de serveur d’hébergement. BUILTIN\Users qui inclut le même ensemble d’utilisateurs que NT AUTHORITY\Authenticated Users.|
|NT AUTHORITY\SYSTEM|Compte de service du système d’exploitation du serveur de fichiers. Ce compte de service n’est pas applicable dans un contexte Azure Files. Il est inclus dans le répertoire racine pour être cohérent avec l’expérience Windows Files Server pour les scénarios hybrides.|
|NT AUTHORITY\Authenticated Users|Tous les utilisateurs dans Active Directory qui peuvent obtenir un jeton Kerberos valide.|
|CREATOR OWNER|Chaque objet, répertoire ou fichier, a un propriétaire pour cet objet. Si des ACL sont affectées à l’objet « CREATOR OWNER » sur cet objet, l’utilisateur qui est le propriétaire de cet objet dispose des autorisations définies par la liste de contrôle d’accès.|



## <a name="mount-a-file-share-from-the-command-prompt"></a>Monter un partage de fichiers Azure à partir de l’invite de commandes

Utilisez la commande Windows `net use` pour monter le partage de fichiers Azure. N’oubliez pas de remplacer les valeurs d’espace réservé dans l’exemple suivant par vos propres valeurs. Pour plus d’informations sur le montage de partages de fichiers, consultez [Utilisation d’un partage de fichiers Azure avec Windows](storage-how-to-use-files-windows.md). 

```
$connectTestResult = Test-NetConnection -ComputerName <storage-account-name>.file.core.windows.net -Port 445
if ($connectTestResult.TcpTestSucceeded)
{
  net use <desired-drive-letter>: \\<storage-account-name>.file.core.windows.net\<share-name> /user:Azure\<storage-account-name> <storage-account-key>
} 
else 
{
  Write-Error -Message "Unable to reach the Azure storage account via port 445. Check to make sure your organization or ISP is not blocking port 445, or use Azure P2S VPN,   Azure S2S VPN, or Express Route to tunnel SMB traffic over a different port."
}

```

Si vous rencontrez des problèmes lors de la connexion à Azure Files, consultez [l’outil de dépannage que nous avons publié pour les erreurs de montage Azure Files sur Windows](https://azure.microsoft.com/blog/new-troubleshooting-diagnostics-for-azure-files-mounting-errors-on-windows/). Nous fournissons également des [conseils](./storage-files-faq.md#on-premises-access) pour contourner des situations où le port 445 est bloqué. 

## <a name="configure-windows-acls"></a>Configurer des ACL Windows

Une fois votre partage de fichiers monté avec la clé de compte de stockage, vous devez configurer les ACL Windows (également appelées autorisations NTFS). Vous pouvez configurer les ACL Windows en utilisant l’Explorateur de fichiers Windows ou icacls.

Si vous avez des répertoires ou des fichiers dans des serveurs de fichiers locaux avec des listes de contrôle d’accès discrétionnaire Windows configurées sur les identités AD DS, vous pouvez les copier sur Azure Files en conservant les ACL avec des outils de copie de fichiers traditionnels comme Robocopy ou [Azure AzCopy v10.4+](https://github.com/Azure/azure-storage-azcopy/releases). Si vos répertoires et fichiers sont hiérarchisés pour Azure Files par le biais d’Azure File Sync, vos ACL sont reportées et conservées dans leur format natif.

### <a name="configure-windows-acls-with-windows-file-explorer"></a>Configurer des ACL Windows avec l’Explorateur de fichiers Windows

Utilisez l’Explorateur de fichiers Windows pour accorder des autorisations complètes sur tous les répertoires et fichiers du partage de fichiers, y compris le répertoire racine.

1. Ouvrez l’Explorateur de fichiers Windows, cliquez avec le bouton droit sur le fichier/répertoire, puis sélectionnez **Propriétés**.
1. Sélectionnez l'onglet **Sécurité** .
1. Sélectionnez **Modifier…** pour modifier les autorisations.
1. Vous pouvez modifier les autorisations des utilisateurs existants ou sélectionner **Ajouter…** pour accorder des autorisations à de nouveaux utilisateurs.
1. Dans la fenêtre d’invite d’ajout de nouveaux utilisateurs, entrez le nom d’utilisateur cible auquel vous souhaitez accorder des autorisations dans la zone **Entrer les noms des objets à sélectionner**, puis sélectionnez **Vérifier les noms** pour trouver le nom UPN complet de l’utilisateur cible.
1.    Sélectionnez **OK**.
1.    Dans l’onglet **Sécurité**, sélectionnez toutes les autorisations que vous souhaitez accorder au nouvel utilisateur.
1.    Sélectionnez **Appliquer**.

### <a name="configure-windows-acls-with-icacls"></a>Configurer des ACL Windows avec icacls

Utilisez la commande Windows suivante pour accorder des autorisations complètes sur tous les répertoires et fichiers du partage de fichiers, y compris le répertoire racine. N’oubliez pas de remplacer les valeurs d’espace réservé dans l’exemple par vos propres valeurs.

```
icacls <mounted-drive-letter>: /grant <user-email>:(f)
```

Pour plus d’informations sur l’utilisation de la commande icacls pour définir des ACL Windows et sur les différents types d’autorisation pris en charge, consultez les [informations de référence sur la ligne de commande pour icacls](/windows-server/administration/windows-commands/icacls).

## <a name="next-steps"></a>Étapes suivantes

Maintenant que la fonctionnalité est activée et configurée, passez à l’article suivant, où vous montez votre partage de fichiers Azure à partir d’une machine virtuelle jointe à un domaine.

[Quatrième partie : monter un partage de fichiers à partir d’une machine virtuelle jointe à un domaine](storage-files-identity-ad-ds-mount-file-share.md)
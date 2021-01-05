---
title: Résoudre les problèmes liés à Azure Files sous Windows
description: Résolution des problèmes liés à Azure Files sous Windows. Consultez les problèmes courants liés à Azure Files lorsque vous vous connectez à partir de clients Windows et découvrez les résolutions possibles. Uniquement pour les partages SMB
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 09/13/2019
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: aef332e54fa650e1abbebe671560238d7eb318de
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492044"
---
# <a name="troubleshoot-azure-files-problems-in-windows-smb"></a>Résoudre les problèmes liés à Azure Files sous Windows (SMB)

Cet article liste les problèmes courants liés à Microsoft Azure Files en cas de connexion à partir de clients Windows. Il fournit également les causes possibles et les solutions de ces problèmes. En plus des étapes de résolutions présentées dans cet article, vous pouvez aussi utiliser [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) pour être sûr que l’environnement du client Windows est configuré correctement. AzFileDiagnostics détecte automatiquement la plupart des problèmes mentionnés dans cet article et vous aide à configurer votre environnement pour que les performances soient optimales.

> [!IMPORTANT]
> Le contenu de cet article s’applique uniquement aux partages SMB. Pour plus d’informations sur les partages NFS, consultez [Résoudre les problèmes des partages de fichiers NFS Azure](storage-troubleshooting-files-nfs.md).

<a id="error5"></a>
## <a name="error-5-when-you-mount-an-azure-file-share"></a>Error 5 quand vous montez un partage de fichiers Azure

Quand vous essayez de monter un partage de fichiers, vous pouvez recevoir l’erreur suivante :

- Erreur système 5. L’accès est refusé.

### <a name="cause-1-unencrypted-communication-channel"></a>Cause 1 : Canal de communication non chiffré

Pour des raisons de sécurité, les connexions aux partages de fichiers Azure sont bloquées si le canal de communication n’est pas chiffré et si la tentative de connexion n’est pas effectuée depuis le centre de données sur lequel résident les partages de fichiers Azure. Les connexions non chiffrées dans le même centre de données peuvent également être bloquées si le paramètre [Transfert sécurisé requis](../common/storage-require-secure-transfer.md) est activé sur le compte de stockage. Un canal de communication chiffré est fourni uniquement si le système d’exploitation client de l’utilisateur prend en charge le chiffrement SMB.

Windows 8, Windows Server 2012 et les versions ultérieures de chaque demande de négociation système incluant SMB 3.0, prenant en charge le chiffrement.

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

1. Connectez-vous à partir d’un client qui prend en charge le chiffrement SMB (Windows 8, Windows Server 2012 ou versions ultérieures), ou à partir d’une machine virtuelle se trouvant dans le même centre de données que le compte de stockage Azure utilisé pour le partage de fichiers Azure.
2. Vérifiez que le paramètre [Transfert sécurisé obligatoire](../common/storage-require-secure-transfer.md) est désactivé sur le compte de stockage si le client ne prend pas en charge le chiffrement SMB.

### <a name="cause-2-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Cause 2 : Des règles de pare-feu ou de réseau virtuel sont activées sur le compte de stockage 

Si des règles de pare-feu et de réseau virtuel sont configurées sur le compte de stockage, le trafic réseau n’a pas d’accès, sauf si l’adresse IP du client ou un réseau virtuel est autorisé à accéder.

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Vérifiez que les règles de pare-feu et de réseau virtuel sont configurées correctement sur le compte de stockage. Pour vérifier si des règles de pare-feu ou de réseau virtuel sont à l’origine du problème, définissez temporairement le paramètre du compte de stockage sur **Autoriser l’accès à partir de tous les réseaux**. Pour plus d’informations, consultez [Configurer les pare-feu et les réseaux virtuels dans le Stockage Azure](../common/storage-network-security.md).

### <a name="cause-3-share-level-permissions-are-incorrect-when-using-identity-based-authentication"></a>Cause 3 : Les autorisations au niveau du partage sont incorrectes lors de l’utilisation de l’authentification basée sur l’identité

Si les utilisateurs accèdent au partage de fichiers Azure à l’aide de l’authentification Active Directory (AD) ou Azure Active Directory Domain Services (Azure AD DS), l’accès au partage de fichiers échoue avec l’erreur « Accès refusé » si les autorisations au niveau du partage sont incorrectes. 

### <a name="solution-for-cause-3"></a>Solution pour la cause 3

Vérifiez que les autorisations sont configurées correctement :

- **Active Directory (AD)** , voir [Affecter des autorisations au niveau du partage à une identité](./storage-files-identity-ad-ds-assign-permissions.md).

    Les attributions d’autorisations au niveau du partage sont prises en charge pour les groupes et les utilisateurs qui ont été synchronisés d’Active Directory (AD) à Azure Active Directory (Azure AD) à l’aide d’Azure AD Connect.  Vérifiez que les groupes et utilisateurs auxquels des autorisations au niveau du partage sont affectées ne sont pas des groupes « cloud uniquement » non pris en charge.
- **Azure Active Directory Domain Services (Azure AD DS)** , voir [Assigner des autorisations d’accès à une identité](./storage-files-identity-auth-active-directory-domain-service-enable.md?tabs=azure-portal#assign-access-permissions-to-an-identity).

<a id="error53-67-87"></a>
## <a name="error-53-error-67-or-error-87-when-you-mount-or-unmount-an-azure-file-share"></a>Les messages « Erreur 53 », « Erreur 67 » ou « Erreur 87 » s’affichent lorsque vous montez ou démontez un partage de fichiers Azure

Lorsque vous essayez de monter un partage de fichiers depuis un centre de données local ou un centre de données différent, les erreurs suivantes pourraient survenir :

- Erreur système 53. Le chemin d’accès réseau est introuvable.
- Erreur système 67. Le nom du réseau est introuvable.
- Erreur système 87. Le paramètre est incorrect.

### <a name="cause-1-port-445-is-blocked"></a>Cause 1 : Le port 445 est bloqué

Une erreur système 53 ou 67 peut se produire si la communication sortante du port 445 vers le centre de données Azure Files est bloquée. Pour afficher le récapitulatif des FAI qui autorisent ou interdisent l’accès depuis le port 445, consultez [TechNet](https://social.technet.microsoft.com/wiki/contents/articles/32346.azure-summary-of-isps-that-allow-disallow-access-from-port-445.aspx).

Pour vérifier si votre pare-feu ou votre ISP bloque le port 445, utilisez l’outil [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) ou l’applet de commande `Test-NetConnection`. 

Pour utiliser la cmdlet `Test-NetConnection`, vous devez avoir installé le module Azure PowerShell. Pour en savoir plus, veuillez consulter [Installer un module Azure PowerShell](/powershell/azure/install-Az-ps). N’oubliez pas de remplacer `<your-storage-account-name>` et `<your-resource-group-name>` avec les noms appropriés de votre compte de stockage.

   
```azurepowershell
$resourceGroupName = "<your-resource-group-name>"
$storageAccountName = "<your-storage-account-name>"

# This command requires you to be logged into your Azure account, run Login-AzAccount if you haven't
# already logged in.
$storageAccount = Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName

# The ComputerName, or host, is <storage-account>.file.core.windows.net for Azure Public Regions.
# $storageAccount.Context.FileEndpoint is used because non-Public Azure regions, such as sovereign clouds
# or Azure Stack deployments, will have different hosts for Azure file shares (and other storage resources).
Test-NetConnection -ComputerName ([System.Uri]::new($storageAccount.Context.FileEndPoint).Host) -Port 445
```
    
Si la connexion a réussi, vous devez voir la sortie suivante :
    
  
```azurepowershell
ComputerName     : <your-storage-account-name>
RemoteAddress    : <storage-account-ip-address>
RemotePort       : 445
InterfaceAlias   : <your-network-interface>
SourceAddress    : <your-ip-address>
TcpTestSucceeded : True
```
 

> [!Note]  
> La commande ci-dessus retourne l’adresse IP actuelle du compte de stockage. Cette adresse IP est susceptible de changer à tout moment. Ne codez pas cette adresse IP en dur dans un script ou dans une configuration de pare-feu.

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

#### <a name="solution-1---use-azure-file-sync"></a>Solution 1 - Utiliser Azure File Sync
Azure File Sync peut transformer votre serveur Windows local en un cache rapide de votre partage de fichiers Azure. Vous pouvez utiliser tout protocole disponible dans Windows Server pour accéder à vos données localement, notamment SMB, NFS et FTPS. Azure File Sync fonctionne sur le port 443 et peut donc être utilisé comme une solution de contournement pour accéder à Azure Files à partir de clients dont le port 445 est bloqué. [Découvrez comment configurer Azure File Sync](./storage-sync-files-extend-servers.md).

#### <a name="solution-2---use-vpn"></a>Solution 2 - Utiliser un VPN
En configurant un VPN pour votre compte de stockage spécifique, le trafic passera par un tunnel sécurisé plutôt que par Internet. Suivez les [instructions pour configurer un VPN](storage-files-configure-p2s-vpn-windows.md) pour accéder à Azure Files depuis Windows.

#### <a name="solution-3---unblock-port-445-with-help-of-your-ispit-admin"></a>Solution 3 - Débloquer le port 445 avec l’aide de votre ISP/administrateur informatique
Contactez votre service informatique ou ISP pour ouvrir le port 445 sortant aux [plages IP Azure](https://www.microsoft.com/download/details.aspx?id=41653).

#### <a name="solution-4---use-rest-api-based-tools-like-storage-explorerpowershell"></a>Solution 4 - Utiliser les outils basés sur l’API REST, par exemple l’Explorateur Stockage/PowerShell
Azure Files prend également en charge REST en plus de SMB. L’accès REST fonctionne sur le port 443 (tcp standard). Il existe divers outils écrits à l’aide de l’API REST qui permettent une expérience d’interface utilisateur riche. L’[Explorateur Stockage](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=windows) est l’un d’eux. [Téléchargez et installez l’Explorateur Stockage](https://azure.microsoft.com/features/storage-explorer/), puis connectez-vous à votre partage de fichiers grâce à Azure Files. Vous pouvez aussi utiliser [PowerShell](./storage-how-to-use-files-powershell.md) qui utilise également API REST.

### <a name="cause-2-ntlmv1-is-enabled"></a>Cause 2 : NTLMv1 est activé

Les messages Erreur système 53 ou Erreur système 87 peuvent également s’afficher si la communication NTLMv1 est activée sur le client. Azure Files prend uniquement en charge l’authentification NTLMv2. Le fait d’activer NTLMv1 crée un client moins sécurisé. Par conséquent, les communications sont bloquées pour Azure Files. 

Pour déterminer s’il s’agit de la cause de l’erreur, vérifiez que la sous-clé de Registre suivante est définie sur une valeur de 3 :

**HKLM\SYSTEM\CurrentControlSet\Control\Lsa &gt; LmCompatibilityLevel**

Pour plus d’informations, consultez la rubrique [LmCompatibilityLevel](/previous-versions/windows/it-pro/windows-2000-server/cc960646(v=technet.10)) sur TechNet.

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Rétablissez la valeur **LmCompatibilityLevel** à la valeur par défaut 3 dans la sous-clé de Registre suivante :

  **HKLM\SYSTEM\CurrentControlSet\Control\Lsa**

<a id="error1816"></a>
## <a name="error-1816---not-enough-quota-is-available-to-process-this-command"></a>Erreur 1816 - Quota insuffisant pour traiter cette commande

### <a name="cause"></a>Cause

L'erreur 1816 se produit lorsque vous atteignez la limite autorisée de descripteurs ouverts simultanément pour un fichier ou un répertoire du partage de fichiers Azure. Pour plus d’informations, consultez la page sur la [de tarification Azure Files](./storage-files-scale-targets.md#azure-files-scale-targets).

### <a name="solution"></a>Solution

Réduisez le nombre de handles ouverts simultanément en fermant certains d’entre eux, puis réessayez. Pour plus d’informations, consultez [Liste de contrôle des performances et de l’extensibilité de Microsoft Azure Storage](../blobs/storage-performance-checklist.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Pour afficher les descripteurs ouverts pour un partage de fichiers, un répertoire ou un fichier, utilisez l’applet de commande PowerShell [Get-AzStorageFileHandle](/powershell/module/az.storage/get-azstoragefilehandle).  

Pour fermer les descripteurs ouverts pour un partage de fichiers, un répertoire ou un fichier, utilisez l’applet de commande PowerShell [Close-AzStorageFileHandle](/powershell/module/az.storage/close-azstoragefilehandle).

> [!Note]  
> Les applets de commande AzStorageFileHandle et Close-AzStorageFileHandle sont incluses dans le module PowerShell Az version 2.4 ou ultérieure. Pour installer le module PowerShell Az le plus récent, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

<a id="noaaccessfailureportal"></a>
## <a name="error-no-access-when-you-try-to-access-or-delete-an-azure-file-share"></a>Erreur « Aucun accès » lorsque vous tentez d’accéder à un partage de fichiers Azure ou d’en supprimer un  
Quand vous tentez d’accéder à un partage de fichiers Azure ou d’en supprimer un dans le portail, vous recevrez peut-être l’erreur suivante :

Aucun accès  
Code d’erreur : 403 

### <a name="cause-1-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Cause 1 : Des règles de pare-feu ou de réseau virtuel sont activées sur le compte de stockage

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

Vérifiez que les règles de pare-feu et de réseau virtuel sont configurées correctement sur le compte de stockage. Pour vérifier si des règles de pare-feu ou de réseau virtuel sont à l’origine du problème, définissez temporairement le paramètre du compte de stockage sur **Autoriser l’accès à partir de tous les réseaux**. Pour plus d’informations, consultez [Configurer les pare-feu et les réseaux virtuels dans le Stockage Azure](../common/storage-network-security.md).

### <a name="cause-2-your-user-account-does-not-have-access-to-the-storage-account"></a>Cause 2 : Votre compte d’utilisateur n’a pas accès au compte de stockage

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Accédez au compte de stockage où se trouve le partage de fichiers Azure, cliquez sur **Contrôle d’accès (IAM)** et vérifiez que votre compte d’utilisateur a accès au compte de stockage. Pour en savoir plus, consultez [Guide pratique pour sécuriser votre compte de stockage avec le contrôle d’accès Azure en fonction du rôle (Azure RBAC)](../blobs/security-recommendations.md#data-protection).

<a id="open-handles"></a>
## <a name="unable-to-modify-moverename-or-delete-a-file-or-directory"></a>Impossible de modifier, déplacer/renommer ou supprimer un fichier ou un répertoire
L’un des principaux objectifs d’un partage de fichiers est que plusieurs utilisateurs et applications puissent interagir simultanément avec les fichiers et les répertoires du partage. Pour faciliter cette interaction, les partages de fichiers offrent plusieurs moyens de négocier l’accès aux fichiers et aux répertoires.

Lorsque vous ouvrez un fichier à partir d’un partage de fichiers Azure monté via SMB, votre application/système d’exploitation demande un descripteur de fichier, qui est une référence au fichier. Entre autres choses, votre application spécifie un mode de partage de fichiers lorsqu’elle demande un descripteur de fichier, qui spécifie le niveau d’exclusivité de votre accès au fichier appliqué par Azure Files : 

- `None` : vous disposez d’un accès exclusif. 
- `Read` : d’autres utilisateurs peuvent lire le fichier alors que vous l’avez ouvert.
- `Write` : d’autres utilisateurs peuvent écrire dans le fichier alors que vous l’avez ouvert. 
- `ReadWrite` : combinaison des modes de partage `Read` et `Write`.
- `Delete` : d’autres utilisateurs peuvent supprimer le fichier alors que vous l’avez ouvert. 

Bien qu’il s’agisse d’un protocole sans état, le protocole FileREST n’a pas de concept de descripteurs de fichiers. Il fournit un mécanisme similaire pour assurer l’accès aux fichiers et dossiers que votre script, application ou service peut utiliser : les baux de fichiers. Lorsqu’un fichier est loué, il est traité comme l’équivalent d’un descripteur de fichier avec le mode de partage de fichiers `None`. 

Bien que les descripteurs de fichiers et les baux jouent un rôle important, il peut arriver que les descripteurs de fichiers et les baux soient orphelins. Dans ce cas, cela peut entraîner des problèmes lors de la modification ou de la suppression de fichiers. Vous pouvez voir des messages d’erreur tels que :

- Le processus ne peut pas accéder au fichier car ce fichier est utilisé par un autre processus.
- L’action ne peut pas être effectuée, car le fichier est ouvert dans un autre programme.
- Le document est verrouillé pour modification par un autre utilisateur.
- La ressource spécifiée est marquée pour suppression par un client SMB.

La résolution de ce problème varie selon qu’il est dû à un descripteur de fichier ou à un bail orphelin. 

### <a name="cause-1"></a>Cause 1
Un descripteur de fichier empêche la modification ou la suppression d’un fichier/répertoire. Vous pouvez utiliser la cmdlet PowerShell [Get-AzStorageFileHandle](/powershell/module/az.storage/get-azstoragefilehandle) pour afficher les descripteurs ouverts. 

Si tous les clients SMB ont fermé leurs descripteurs ouverts sur un fichier/répertoire et que le problème persiste, vous pouvez forcer la fermeture d’un descripteur de fichier.

### <a name="solution-1"></a>Solution 1
Pour forcer la fermeture d’un descripteur de fichier, utilisez la cmdlet PowerShell [Close-AzStorageFileHandle](/powershell/module/az.storage/close-azstoragefilehandle). 

> [!Note]  
> Les applets de commande AzStorageFileHandle et Close-AzStorageFileHandle sont incluses dans le module PowerShell Az version 2.4 ou ultérieure. Pour installer le module PowerShell Az le plus récent, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

### <a name="cause-2"></a>Cause 2
Un bail de fichier empêche la modification ou la suppression d’un fichier. Vous pouvez vérifier si un fichier dispose d’un bail à l’aide de la commande PowerShell suivante, en remplaçant `<resource-group>`, `<storage-account>`, `<file-share>` et `<path-to-file>` par les valeurs appropriées pour votre environnement :

```PowerShell
# Set variables 
$resourceGroupName = "<resource-group>"
$storageAccountName = "<storage-account>"
$fileShareName = "<file-share>"
$fileForLease = "<path-to-file>"

# Get reference to storage account
$storageAccount = Get-AzStorageAccount `
        -ResourceGroupName $resourceGroupName `
        -Name $storageAccountName

# Get reference to file
$file = Get-AzStorageFile `
        -Context $storageAccount.Context `
        -ShareName $fileShareName `
        -Path $fileForLease

$fileClient = $file.ShareFileClient

# Check if the file has a file lease
$fileClient.GetProperties().Value
```

Si un fichier a un bail, l’objet retourné doit contenir les propriétés suivantes :

```Output
LeaseDuration         : Infinite
LeaseState            : Leased
LeaseStatus           : Locked
```

### <a name="solution-2"></a>Solution 2
Pour supprimer un bail d’un fichier, vous pouvez libérer ou rompre le bail. Pour libérer le bail, vous avez besoin du LeaseId du bail, que vous définissez lors de sa création. Vous n’avez pas besoin du LeaseId pour rompre le bail.

L’exemple suivant montre comment rompre le bail pour le fichier indiqué en cause 2 (cet exemple continue avec les variables PowerShell de la cause 2) :

```PowerShell
$leaseClient = [Azure.Storage.Files.Shares.Specialized.ShareLeaseClient]::new($fileClient)
$leaseClient.Break() | Out-Null
```

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-windows"></a>Ralentissement des copies de fichiers vers et à partir d’Azure Files sous Windows

Les performances peuvent être ralenties lorsque vous essayez de transférer des fichiers vers le service Azure File.

- Si vous n’avez pas d’exigence de taille d’E/S minimum spécifique, nous vous recommandons d’utiliser une taille d’E/S de 1 Mio pour des performances optimales.
-   Si vous connaissez la taille finale d’un fichier que vous étendez avec des écritures, et si votre logiciel ne présente aucun problème de compatibilité lorsque la fin non écrite du fichier contient des zéros, définissez la taille du fichier à l’avance au lieu que chaque écriture soit une écriture d’extension.
-   Utilisez la méthode de copie appropriée :
    -   Utilisez [AZCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) pour les transferts entre deux partages de fichiers.
    -   Utilisez [Robocopy](./storage-files-deployment-guide.md#robocopy) entre des partages de fichiers sur un ordinateur local.

### <a name="considerations-for-windows-81-or-windows-server-2012-r2"></a>Informations pour Windows 8.1 ou Windows Server 2012 R2

Pour les clients qui exécutent Windows 8.1 ou Windows Server 2012 R2, assurez-vous que le correctif logiciel [KB3114025](https://support.microsoft.com/help/3114025) est installé. Ce correctif logiciel améliore les performances de création et d’ouverture des handles.

Vous pouvez exécuter le script suivant pour vérifier si le correctif logiciel a été installé :

`reg query HKLM\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies`

Si le correctif logiciel est installé, la sortie suivante s’affiche :

`HKEY_Local_MACHINE\SYSTEM\CurrentControlSet\Services\LanmanWorkstation\Parameters\Policies {96c345ef-3cac-477b-8fcd-bea1a564241c} REG_DWORD 0x1`

> [!Note]
> Les images de Windows Server 2012 R2 dans Azure Marketplace ont le correctif logiciel KB3114025 installé par défaut à compter de décembre 2015.

<a id="shareismissing"></a>
## <a name="no-folder-with-a-drive-letter-in-my-computer-or-this-pc"></a>Aucun dossier avec une lettre de lecteur dans « Poste de travail » ou « Ce PC »

Si vous mappez un partage de fichiers Azure en tant qu’administrateur via Net use, le partage n’apparaît pas.

### <a name="cause"></a>Cause

Par défaut, l’Explorateur Windows ne s’exécute pas en tant qu’administrateur. Si vous exécutez Net use depuis une invite de commandes administrateur, vous mappez le lecteur réseau en tant qu’administrateur. Étant donné que les lecteurs mappés sont centrés sur l’utilisateur, le compte d’utilisateur qui est connecté n’affiche pas les lecteurs s’ils sont montés sous un compte d’utilisateur différent.

### <a name="solution"></a>Solution
Montez le partage à partir d’une ligne de commande non administrateur. Vous pouvez également suivre [cette rubrique TechNet](/previous-versions/windows/it-pro/windows-server-2008-R2-and-2008/ee844140(v=ws.10)) pour configurer la valeur de Registre **EnableLinkedConnections**.

<a id="netuse"></a>
## <a name="net-use-command-fails-if-the-storage-account-contains-a-forward-slash"></a>La commande Net use échoue si le compte de stockage contient une barre oblique

### <a name="cause"></a>Cause

La commande Net use interprète une barre oblique (/) comme une option de ligne de commande. Si le nom de votre compte utilisateur commence par une barre oblique, le mappage du lecteur échoue.

### <a name="solution"></a>Solution

Vous pouvez utiliser l’une des étapes suivantes pour contourner le problème :

- Exécutez la commande PowerShell suivante :

  `New-SmbMapping -LocalPath y: -RemotePath \\server\share -UserName accountName -Password "password can contain / and \ etc"`

  Depuis un fichier de commandes, vous pouvez exécuter la commande de cette façon :

  `Echo new-smbMapping ... | powershell -command –`

- Placez des guillemets doubles autour de la clé pour résoudre ce problème, sauf si la barre oblique est le premier caractère. Si c’est le cas, utilisez le mode interactif et entrez votre mot de passe séparément, ou régénérez vos clés pour obtenir une clé qui ne commence pas par une barre oblique.

<a id="cannotaccess"></a>
## <a name="application-or-service-cannot-access-a-mounted-azure-files-drive"></a>L’application ou service ne peut pas accéder à un lecteur Azure Files monté

### <a name="cause"></a>Cause

Les lecteurs sont montés par l’utilisateur. Si vous n’exécutez pas l’application ou service depuis le compte utilisateur ayant monté le lecteur, l’application ou service ne verra pas le lecteur.

### <a name="solution"></a>Solution

Utilisez l’une des solutions suivantes :

-   Montez le lecteur depuis le compte utilisateur qui dispose de l’application. Vous pouvez utiliser un outil tel que PsExec.
- Transmettez le nom et la clé du compte de stockage dans les paramètres de nom d’utilisateur et de mot de passe de la commande Net use.
- Utilisez la commande cmdkey pour ajouter les informations d’identification dans le Gestionnaire d’informations d’identification. Effectuez cette procédure à partir d’une ligne de commande dans le contexte du compte de service, via une connexion interactive ou en utilisant `runas`.
  
  `cmdkey /add:<storage-account-name>.file.core.windows.net /user:AZURE\<storage-account-name> /pass:<storage-account-key>`
- Mappez le partage directement sans l’aide d’une lettre de lecteur mappée. Certaines applications ne se reconnectent pas correctement à la lettre de lecteur. Il est donc conseillé d’utiliser le chemin d’accès UNC complet qui est plus fiable. 

  `net use * \\storage-account-name.file.core.windows.net\share`

Après avoir suivi ces instructions, vous pouvez recevoir le message d’erreur suivant en exécutant net use pour le compte de service réseau ou système : « Erreur système 1312. Une session ouverte spécifiée n’existe pas. Il se peut qu’elle ait été déjà fermée. » Si cela se produit, vérifiez que le nom d’utilisateur transmis à Net use inclut des informations de domaine (par exemple : « [nom du compte de stockage].file.core.windows.net »).

<a id="doesnotsupportencryption"></a>
## <a name="error-you-are-copying-a-file-to-a-destination-that-does-not-support-encryption"></a>Erreur « Vous copiez un fichier vers une destination qui ne prend pas en charge le chiffrement »

Lorsqu’un fichier est copié sur le réseau, le fichier est déchiffré sur l’ordinateur source, transmis en clair, puis de nouveau chiffré une fois à destination. Toutefois, vous pouvez voir l’erreur suivante quand vous essayez de copier un fichier chiffré : « Vous copiez le fichier vers une destination qui ne prend pas en charge le chiffrement. »

### <a name="cause"></a>Cause
Ce problème peut se survenir si vous utilisez le système de fichiers EFS (Encrypting File System). Les fichiers chiffrés BitLocker peuvent être copiés vers Azure Files. Toutefois, Azure Files ne prend pas en charge le système de fichiers EFS NTFS.

### <a name="workaround"></a>Solution de contournement
Pour copier un fichier sur le réseau, vous devez d’abord le déchiffrer. Appliquez l'une des méthodes suivantes :

- Utilisez la commande **copy /d**. Les fichiers chiffrés peuvent ainsi être enregistrés comme fichiers déchiffrés une fois à destination.
- Définissez la clé de Registre suivante :
  - Chemin d’accès = HKLM\Software\Policies\Microsoft\Windows\System
  - Type de valeur = DWORD
  - Nom = CopyFileAllowDecryptedRemoteDestination
  - Valeur = 1

Notez bien que la définition de la clé de Registre affecte toutes les opérations de copie sur les partages réseau.

## <a name="slow-enumeration-of-files-and-folders"></a>Lenteur de l’énumération des fichiers et dossiers

### <a name="cause"></a>Cause

Ce problème peut se produire quand il n’y a pas suffisamment de mémoire cache sur la machine cliente pour les grands répertoires.

### <a name="solution"></a>Solution

Pour résoudre ce problème, ajustez la valeur de Registre **DirectoryCacheEntrySizeMax** pour permettre la mise en cache des listes de répertoires plus grands sur la machine cliente :

- Localisation : HKLM\System\CCS\Services\Lanmanworkstation\Parameters
- Nom de la valeur : DirectoryCacheEntrySizeMax 
- Type de valeur : DWORD
 
 
Par exemple, spécifiez la valeur 0x100000 pour voir si les performances sont meilleures.

## <a name="error-aaddstenantnotfound-in-enabling-azure-active-directory-domain-service-azure-ad-ds-authentication-for-azure-files-unable-to-locate-active-tenants-with-tenant-id-aad-tenant-id"></a>Erreur AadDsTenantNotFound active l’authentification Azure Active Directory Domain Services (Azure AD DS) pour Azure Files « Impossible de localiser des abonnés actifs avec l’ID aad-tenant-id »

### <a name="cause"></a>Cause

L’erreur AadDsTenantNotFound se produit lorsque vous tentez d’[activer l’authentification Azure Active Directory Domain Services (Azure AD DS) pour Azure Files](storage-files-identity-auth-active-directory-domain-service-enable.md) sur un compte de stockage où [Azure AD Domain Services (Azure AD DS)](../../active-directory-domain-services/overview.md) n’est pas créé sur le locataire Azure AD de l’abonnement associé.  

### <a name="solution"></a>Solution

Activez Azure AD DS sur le locataire Azure AD de l’abonnement sur lequel votre compte de stockage est déployé. Pour créer un domaine managé, vous devez disposer des privilèges d’administrateur du locataire Azure AD. Si vous n’êtes pas l’administrateur du locataire Azure AD, contactez-le et suivez les instructions pas à pas pour [Créer et configurer un domaine managé Azure Active Directory Domain Services](../../active-directory-domain-services/tutorial-create-instance.md).

[!INCLUDE [storage-files-condition-headers](../../../includes/storage-files-condition-headers.md)]

## <a name="unable-to-mount-azure-files-with-ad-credentials"></a>Impossible de monter Azure Files avec les informations d’identification AD 

### <a name="self-diagnostics-steps"></a>Étapes des autodiagnostics
Tout d’abord, assurez-vous que vous avez suivi les quatre étapes permettant d’[activer l’authentification AD pour Azure Files](./storage-files-identity-auth-active-directory-enable.md).

Ensuite, essayez de [monter un partage Azure Files avec la clé de compte de stockage](./storage-how-to-use-files-windows.md). Si le montage échoue, téléchargez l’outil [AzFileDiagnostics.ps1](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) qui vous permet de valider le client exécutant l’environnement, de détecter les problèmes d’incompatibilité des configurations client qui risquent de compromettre l’accès d’Azure Files, de fournir des conseils prescriptifs sur les correctifs autonomes et de collecter les suivis de diagnostic.

Troisièmement, vous pouvez exécuter l’applet de commande Debug-AzStorageAccountAuth pour effectuer un ensemble de vérifications de base sur la configuration AD avec l’utilisateur AD connecté. Cette applet de commande est prise en charge sur [AzFilesHybrid 0.1.2 et versions ultérieures](https://github.com/Azure-Samples/azure-files-samples/releases). Vous devez exécuter cette applet de commande avec un utilisateur AD disposant de l’autorisation de propriétaire sur le compte de stockage cible.  
```PowerShell
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

Debug-AzStorageAccountAuth -StorageAccountName $StorageAccountName -ResourceGroupName $ResourceGroupName -Verbose
```
L’applet de commande effectue dans l’ordre les vérifications ci-dessous, puis fournit des conseils en cas d’échec :
1. CheckADObjectPasswordIsCorrect : Vérifier que le mot de passe configuré sur l’identité AD qui représente le compte de stockage correspond à celui de la clé kerb1 ou kerb2 du compte de stockage. Si le mot de passe est incorrect, vous pouvez exécuter [Update-AzStorageAccountADObjectPassword](./storage-files-identity-ad-ds-update-password.md) pour réinitialiser le mot de passe. 
2. CheckADObject : Confirmer qu’il existe un objet dans Active Directory qui représente le compte de stockage et qu’il possède le nom de principal du service (SPN) correct. Si le SPN n’est pas correctement configuré, exécutez la cmdlet Set-AD retournée dans la cmdlet debug pour configurer le SPN.
3. CheckDomainJoined : Vérifier que l’ordinateur client est joint à AD. Si votre ordinateur n’est pas joint à AD, consultez [cet article](/windows-server/identity/ad-fs/deployment/join-a-computer-to-a-domain) pour l’instruction de jonction de domaine.
4. CheckPort445Connectivity : Vérifier que le port 445 est ouvert pour la connexion SMB. Si le port requis n’est pas ouvert, reportez-vous à l’outil de dépannage [AzFileDiagnostics.ps1](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Windows) pour les problèmes de connectivité avec Azure Files.
5. CheckSidHasAadUser : Vérifiez que l’utilisateur AD connecté est synchronisé avec Azure AD. Si vous souhaitez rechercher si un utilisateur AD spécifique est synchronisé avec Azure AD, vous pouvez spécifier les paramètres -UserName et -Domain dans les paramètres d’entrée. 
6. CheckGetKerberosTicket : Tenter d’obtenir un ticket Kerberos pour se connecter au compte de stockage. S’il n’existe pas de jeton Kerberos valide, exécutez la cmdlet get cifs/storage-account-name.file.core.windows.net, puis examinez le code d’erreur pour l’analyse de la cause racine de l’échec de la récupération du ticket.
7. CheckStorageAccountDomainJoined : Vérifier si l’authentification Active Directory a été activée et si les propriétés AD du compte sont renseignées. Si ce n’est pas le cas, reportez-vous à l’instruction [ici](./storage-files-identity-ad-ds-enable.md) pour activer l’authentification AD DS sur Azure Files. 

## <a name="unable-to-configure-directoryfile-level-permissions-windows-acls-with-windows-file-explorer"></a>Impossible de configurer des autorisations au niveau des répertoires/fichiers (listes de contrôle d’accès Windows) avec l’Explorateur de fichiers Windows

### <a name="symptom"></a>Symptôme

Vous pouvez être confronté aux symptômes décrits ci-dessous quand vous tentez de configurer des listes de contrôle d’accès Windows avec l’Explorateur de fichiers sur un partage de fichiers monté :
- Quand vous cliquez sur Modifier l’autorisation sous l’onglet Sécurité, l’Assistant Autorisation ne se charge pas. 
- Quand vous essayez de sélectionner un nouvel utilisateur ou un nouveau groupe, l’emplacement du domaine n’affiche pas le bon domaine AD DS. 

### <a name="solution"></a>Solution

Pour contourner le problème, nous vous recommandons d’utiliser l’[outil icacls](/windows-server/administration/windows-commands/icacls) afin de configurer des autorisations au niveau des répertoires/fichiers. 

## <a name="errors-when-running-join-azstorageaccountforauth-cmdlet"></a>Erreurs lors de l’exécution de la cmdlet join-AzStorageAccountForAuth

### <a name="error-the-directory-service-was-unable-to-allocate-a-relative-identifier"></a>Erreur : « Le service d’annuaire n’a pas pu allouer un identificateur relatif »

Cette erreur peut se produire si un contrôleur de domaine titulaire du rôle FSMO du maître RID n’est pas disponible ou a été supprimé du domaine, puis restauré à partir d’une sauvegarde.  Vérifiez que tous les contrôleurs de domaine sont en cours d’exécution et disponibles.

### <a name="error-cannot-bind-positional-parameters-because-no-names-were-given"></a>Erreur : « Impossible de lier les paramètres positionnels, car aucun nom n’a été fourni »

Cette erreur est probablement déclenchée par une erreur de syntaxe dans la commande Join-AzStorageAccountforAuth.  Vérifiez que la commande ne contient pas de fautes de frappe ou d’erreurs syntaxe, et que la version la plus récente du module AzFilesHybrid (https://github.com/Azure-Samples/azure-files-samples/releases) est installée.  

## <a name="azure-files-on-premises-ad-ds-authentication-support-for-aes-256-kerberos-encryption"></a>Prise en charge de l’authentification AD DS locale Azure Files pour le chiffrement Kerberos AES 256

Nous avons introduit la prise en charge du chiffrement Kerberos AES 256 pour l’authentification AD DS locale Azure Files avec le [module AzFilesHybrid v0.2.2](https://github.com/Azure-Samples/azure-files-samples/releases). Si vous avez activé l’authentification AD DS avec une version de module antérieure à la v0.2.2, vous devez télécharger le module AzFilesHybrid le plus récent (v0.2.2+) et exécuter la commande PowerShell ci-dessous. Si vous n’avez pas encore activé l’authentification AD DS sur votre compte de stockage, vous pouvez suivre ces [instructions](./storage-files-identity-ad-ds-enable.md#option-one-recommended-use-azfileshybrid-powershell-module) pour l’activation. 

```PowerShell
$ResourceGroupName = "<resource-group-name-here>"
$StorageAccountName = "<storage-account-name-here>"

Update-AzStorageAccountAuthForAES256 -ResourceGroupName $ResourceGroupName -StorageAccountName $StorageAccountName
```


## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.
Si vous avez encore besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.
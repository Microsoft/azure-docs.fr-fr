---
title: Résoudre les problèmes liés à Azure Files dans Linux | Microsoft Docs
description: Résolution des problèmes liés à Azure Files dans Linux. Consultez les problèmes courants liés à Azure Files lorsque vous vous connectez à partir de clients Linux et découvrez les résolutions possibles.
author: jeffpatt24
ms.service: storage
ms.topic: troubleshooting
ms.date: 10/16/2018
ms.author: jeffpatt
ms.subservice: files
ms.openlocfilehash: e680ba10c507ef83591b56652ee8e95c4d665dda
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96492061"
---
# <a name="troubleshoot-azure-files-problems-in-linux-smb"></a>Résoudre les problèmes liés à Azure Files dans Linux (SMB)

Cet article liste les problèmes courants liés à Azure Files quand vous vous connectez à partir de clients Linux. Il fournit également les causes possibles et les solutions de ces problèmes. 

En plus des étapes de dépannage présentées dans cet article, vous pouvez utiliser [AzFileDiagnostics](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Linux) pour vérifier que le client Linux dispose des prérequis. Azfilediagnostics automatise la détection de la plupart des problèmes mentionnés dans cet article. Il vous aide à configurer votre environnement pour obtenir des performances optimales. Vous pouvez également trouver ces informations dans l’[utilitaire de résolution des problèmes de partages Azure Files](https://support.microsoft.com/help/4022301/troubleshooter-for-azure-files-shares). L’utilitaire de résolution des problèmes indique les étapes pour vous aider en cas de problèmes de connexion, de mappage et de montage de partages Azure Files.

> [!IMPORTANT]
> Le contenu de cet article s’applique uniquement aux partages SMB. Pour plus d’informations sur les partages NFS, consultez [Résoudre les problèmes des partages de fichiers NFS Azure](storage-troubleshooting-files-nfs.md).

## <a name="cannot-connect-to-or-mount-an-azure-file-share"></a>Connexion ou montage impossible d’un partage de fichiers Azure

### <a name="cause"></a>Cause

Causes courantes de ce problème :

- Vous utilisez un client de distribution Linux non compatible. Nous vous recommandons d’utiliser les distributions Linux suivantes pour vous connecter à un partage de fichiers Azure :

|   | SMB 2.1 <br>(Montages sur des machines virtuelles au sein de la même région Azure) | SMB 3.0 <br>(Montages en local et entre les régions) |
| --- | :---: | :---: |
| **Serveur Ubuntu** | 14.04+ | 16.04+ |
| **RHEL** | 7+ | 7.5+ |
| **CentOS** | 7+ |  7.5+ |
| **Debian** | 8+ |   |
| **openSUSE** | 13.2+ | 42.3+ |
| **SUSE Linux Enterprise Server** | 12 | 12 SP3+ |

- Les utilitaires CIFS (cifs-utils) ne sont pas installés sur le client.
- La version de SMB/CIFS minimale, à savoir 2.1, n’est pas installée sur le client.
- Le chiffrement SMB 3.0 n’est pas pris en charge sur le client. Le tableau précédent fournit une liste des distributions de Linux qui prennent en charge le montage à partir d’emplacements locaux et entre les régions grâce au chiffrement. Les autres distributions requièrent un noyau 4.11 et versions ultérieures.
- Vous tentez de vous connecter à un compte de stockage via le port TCP 445, qui n’est pas pris en charge.
- Vous tentez de vous connecter à un partage de fichiers Azure à partir d’une machine virtuelle Azure qui ne se trouve pas dans la même région que le compte de stockage.
- Si le paramètre [Transfert sécurisé requis]( https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer) est activé sur le compte de stockage, Azure Files autorise uniquement les connexions utilisant SMB 3.0 avec chiffrement.

### <a name="solution"></a>Solution

Pour résoudre le problème, utilisez [l’outil de résolution des erreurs de montage Azure Files sur Linux](https://github.com/Azure-Samples/azure-files-samples/tree/master/AzFileDiagnostics/Linux). Cet outil offre les possibilités suivantes :

* Il vous permet de valider l’environnement d’exécution du client.
* Il détecte la configuration client incompatible qui provoquerait l’échec de l’accès d’Azure Files.
* Il fournit des instructions pour la résolution automatique.
* Il collecte les suivis des diagnostics.

<a id="mounterror13"></a>
## <a name="mount-error13-permission-denied-when-you-mount-an-azure-file-share"></a>« Erreur de montage (13) : Autorisation refusée » quand vous montez un partage de fichiers Azure

### <a name="cause-1-unencrypted-communication-channel"></a>Cause 1 : Canal de communication non chiffré

Pour des raisons de sécurité, les connexions aux partages de fichiers Azure sont bloquées si le canal de communication n’est pas chiffré et si la tentative de connexion n’est pas effectuée depuis le centre de données sur lequel résident les partages de fichiers Azure. Les connexions non chiffrées dans le même centre de données peuvent également être bloquées si le paramètre [Transfert sécurisé requis](../common/storage-require-secure-transfer.md) est activé sur le compte de stockage. Un canal de communication chiffré est fourni uniquement si le système d’exploitation client de l’utilisateur prend en charge le chiffrement SMB.

Pour en savoir plus, consultez [Prérequis pour le montage d’un partage de fichiers Azure avec Linux et le package cifs-utils](storage-how-to-use-files-linux.md#prerequisites). 

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

1. Connectez-vous à partir d’un client qui prend en charge le chiffrement SMB ou à partir d’une machine virtuelle se trouvant dans le même centre de données que le compte de stockage Azure utilisé pour le partage de fichiers Azure.
2. Vérifiez que le paramètre [Transfert sécurisé obligatoire](../common/storage-require-secure-transfer.md) est désactivé sur le compte de stockage si le client ne prend pas en charge le chiffrement SMB.

### <a name="cause-2-virtual-network-or-firewall-rules-are-enabled-on-the-storage-account"></a>Cause 2 : Des règles de pare-feu ou de réseau virtuel sont activées sur le compte de stockage 

Si des règles de pare-feu et de réseau virtuel sont configurées sur le compte de stockage, le trafic réseau n’a pas d’accès, sauf si l’adresse IP du client ou un réseau virtuel est autorisé à accéder.

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Vérifiez que les règles de pare-feu et de réseau virtuel sont configurées correctement sur le compte de stockage. Pour vérifier si des règles de pare-feu ou de réseau virtuel sont à l’origine du problème, définissez temporairement le paramètre du compte de stockage sur **Autoriser l’accès à partir de tous les réseaux**. Pour plus d’informations, consultez [Configurer les pare-feu et les réseaux virtuels dans le Stockage Azure](../common/storage-network-security.md).

<a id="permissiondenied"></a>
## <a name="permission-denied-disk-quota-exceeded-when-you-try-to-open-a-file"></a>« [autorisation refusée] Quota de disque dépassé » lorsque vous essayez d’ouvrir un fichier

Dans Linux, vous recevez un message d’erreur semblable au suivant :

**\<filename> [autorisation refusée] Quota de disque dépassé**

### <a name="cause"></a>Cause

Vous avez atteint la limite supérieure de handles ouverts simultanément autorisés pour un fichier ou un répertoire.

Il existe un quota de 2 000 handles ouverts sur un seul fichier ou répertoire. Quand vous avez 2 000 handles ouverts, un message d’erreur s’affiche pour signaler que le quota est atteint.

### <a name="solution"></a>Solution

Réduisez le nombre de handles ouverts simultanément en en fermant certains, puis réessayez l’opération.

Pour afficher les descripteurs ouverts pour un partage de fichiers, un répertoire ou un fichier, utilisez l’applet de commande PowerShell [Get-AzStorageFileHandle](/powershell/module/az.storage/get-azstoragefilehandle).  

Pour fermer les descripteurs ouverts pour un partage de fichiers, un répertoire ou un fichier, utilisez l’applet de commande PowerShell [Close-AzStorageFileHandle](/powershell/module/az.storage/close-azstoragefilehandle).

> [!Note]  
> Les applets de commande AzStorageFileHandle et Close-AzStorageFileHandle sont incluses dans le module PowerShell Az version 2.4 ou ultérieure. Pour installer le module PowerShell Az le plus récent, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

<a id="slowfilecopying"></a>
## <a name="slow-file-copying-to-and-from-azure-files-in-linux"></a>Ralentissement des copies de fichiers vers et à partir d’Azure Files dans Linux

- Si vous n’avez pas d’exigence de taille d’E/S minimum spécifique, nous vous recommandons d’utiliser une taille d’E/S de 1 Mio pour des performances optimales.
- Utilisez la méthode de copie appropriée :
    - Utilisez [AZCopy](../common/storage-use-azcopy-v10.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json) pour les transferts entre deux partages de fichiers.
    - Utilisez cp ou dd avec parallel peut améliorer la vitesse de copie. Le nombre de threads dépend de votre charge de travail et cas d’usage. Les exemples suivants utilisent six : 
    - exemple cp (cp utilise la taille de bloc par défaut du système de fichiers comme taille de segment) : `find * -type f | parallel --will-cite -j 6 cp {} /mntpremium/ &`:.
    - exemple dd (cette commande définit explicitement la taille du bloc sur 1 Mio) : `find * -type f | parallel --will-cite-j 6 dd if={} of=/mnt/share/{} bs=1M`
    - Outils tiers open source tels que :
        - [GNU Parallel](https://www.gnu.org/software/parallel/).
        - [Fpart](https://github.com/martymac/fpart) : trie les fichiers et les regroupe en partitions.
        - [Fpsync](https://github.com/martymac/fpart/blob/master/tools/fpsync) : utilise Fpart et un outil de copie pour générer plusieurs instances afin de migrer des données de src_dir vers dst_url.
        - [Multi](https://github.com/pkolano/mutil) : cp et md5sum multithread basés sur GNU Core Utilities.
- Définir la taille de fichier à l’avance, au lieu de faire de chaque écriture une écriture étendue, permet d’améliorer la vitesse de copie dans les scénarios où la taille du fichier est connue. Si l’extension des écritures doit être évitée, vous pouvez définir une taille de fichier de destination avec la commande `truncate - size <size><file>`. Après cela, la commande `dd if=<source> of=<target> bs=1M conv=notrunc` copie un fichier source sans avoir à mettre à jour la taille du fichier cible à plusieurs reprises. Par exemple, vous pouvez définir la taille de fichier de destination pour chaque fichier que vous souhaitez copier (en supposant qu’un partage est monté sous /mnt/Share) :
    - `$ for i in `` find * -type f``; do truncate --size ``stat -c%s $i`` /mnt/share/$i; done`
    - puis copiez les fichiers sans étendre les écritures en parallèle : `$find * -type f | parallel -j6 dd if={} of =/mnt/share/{} bs=1M conv=notrunc`

<a id="error115"></a>
## <a name="mount-error115-operation-now-in-progress-when-you-mount-azure-files-by-using-smb-30"></a>« Erreur de montage (115) : L’opération est en cours » quand vous montez Azure Files à l’aide de SMB 3.0

### <a name="cause"></a>Cause

Certaines distributions Linux ne prennent pas encore en charge les fonctionnalités de chiffrement dans SMB 3.0. Il est possible que les utilisateurs reçoivent un message d’erreur « 115 » s’ils essaient de monter Azure Files en utilisant SMB 3.0 en raison d’une fonctionnalité manquante. SMB 3.0 avec le chiffrement complet est pris en charge uniquement lorsque vous utilisez Ubuntu 16.04 ou version ultérieure.

### <a name="solution"></a>Solution

La fonctionnalité de chiffrement pour SMB 3.0 pour Linux a été introduite dans le noyau 4.11. Cette fonctionnalité permet le montage du partage d’un fichier Azure en local ou à partir d’une autre région Azure. Certaines distributions Linux peuvent avoir des modifications rétroportées du noyau 4.11 vers des versions antérieures du noyau Linux qu’elles maintiennent. Pour vous aider à déterminer si votre version de Linux prend en charge SMB 3.0 avec chiffrement, consultez [Utiliser Azure Files avec Linux](storage-how-to-use-files-linux.md). 

Si votre client SMB Linux ne prend pas en charge le chiffrement, montez Azure Files à l’aide de SMB 2.1 à partir d’une machine virtuelle Azure Linux se trouvant dans le même centre de données que le partage de fichiers. Vérifiez que le paramètre [Transfert sécurisé requis]( https://docs.microsoft.com/azure/storage/common/storage-require-secure-transfer) est désactivé sur le compte de stockage. 

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
## <a name="unable-to-delete-a-file-or-directory-in-an-azure-file-share"></a>Impossible de supprimer un fichier ou répertoire d’un partage de fichiers Azure

### <a name="cause"></a>Cause
Ce problème se produit généralement quand le fichier ou le répertoire a un descripteur ouvert. 

### <a name="solution"></a>Solution

Si les clients SMB ont fermé tous les descripteurs ouverts et que le problème persiste, effectuez les étapes suivantes :

- Utilisez l’applet de commande PowerShell [Get-AzStorageFileHandle](/powershell/module/az.storage/get-azstoragefilehandle) pour afficher les descripteurs ouverts.

- Utilisez l’applet de commande PowerShell [Close-AzStorageFileHandle](/powershell/module/az.storage/close-azstoragefilehandle) pour fermer les descripteurs ouverts. 

> [!Note]  
> Les applets de commande AzStorageFileHandle et Close-AzStorageFileHandle sont incluses dans le module PowerShell Az version 2.4 ou ultérieure. Pour installer le module PowerShell Az le plus récent, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

<a id="slowperformance"></a>
## <a name="slow-performance-on-an-azure-file-share-mounted-on-a-linux-vm"></a>Ralentissement des performances dans un partage de fichiers Azure monté sur une machine virtuelle

### <a name="cause-1-caching"></a>Cause 1 : Mise en cache

L’une des causes possibles du ralentissement des performances est la désactivation de la mise en cache. La mise en cache peut être utile que si vous accédez plusieurs fois à un fichier. Sinon, il peut s’agir d’une surcharge. Vérifiez si vous utilisez le cache avant de le désactiver.

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

Pour vérifier si la mise en cache est désactivée, recherchez l’entrée **cache=** .

**cache=none** indique que la mise en cache est désactivée. Remontez le partage avec la commande de montage par défaut ou en ajoutant explicitement l’option **cache=strict** à la commande de montage pour vérifier l’activation de la mise en cache par défaut ou du mode de mise en cache « strict ».

Dans certains scénarios, l’option de montage **serverino** peut entraîner la commande **ls** à exécuter stat dans chaque entrée de répertoire. Ce comportement provoque une dégradation des performances lorsque vous répertoriez le contenu d’un répertoire volumineux. Vous pouvez vérifier les options de montage dans l’entrée **/etc/fstab** :

`//azureuser.file.core.windows.net/cifs /cifs cifs vers=2.1,serverino,username=xxx,password=xxx,dir_mode=0777,file_mode=0777`

Vous pouvez également vérifier si les options correctes sont utilisées en exécutant la commande **sudo mount | grep cifs** et en vérifiant sa sortie. Voici un exemple de sortie :

```
//azureuser.file.core.windows.net/cifs on /cifs type cifs (rw,relatime,vers=2.1,sec=ntlmssp,cache=strict,username=xxx,domain=X,uid=0,noforceuid,gid=0,noforcegid,addr=192.168.10.1,file_mode=0777, dir_mode=0777,persistenthandles,nounix,serverino,mapposix,rsize=1048576,wsize=1048576,actimeo=1)
```

Si l’option **cache=strict** ou **serverino** n’est pas présente, démontez et remontez Azure Files en exécutant la commande de montage à partir de la [documentation](./storage-how-to-use-files-linux.md). Revérifiez ensuite que les options sont correctes pour l’entrée **/etc/fstab**.

### <a name="cause-2-throttling"></a>Cause 2 : Limitation

Vous rencontrez peut-être des limitations et vos requêtes sont envoyées dans une file d’attente. Vous pouvez cela vérifier en tirant parti des [mesures de stockage Azure dans Azure Monitor](../blobs/monitor-blob-storage.md).

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Vérifiez que votre application est incluse dans les [objectifs de mise à l’échelle Azure Files](storage-files-scale-targets.md#azure-files-scale-targets).

<a id="timestampslost"></a>
## <a name="time-stamps-were-lost-in-copying-files-from-windows-to-linux"></a>Les horodatages ont été perdus lors de la copie des fichiers de Windows vers Linux

Sur les plateformes Linux/Unix, la commande **cp -p** échoue si les fichiers 1 et 2 sont détenus par différents utilisateurs.

### <a name="cause"></a>Cause

L’indicateur Force **f** de COPYFILE provoque l’exécution de la commande **cp -p -f** sur Unix. Cette commande échoue également dans la conservation de l’horodatage du fichier que vous ne possédez pas.

### <a name="workaround"></a>Solution de contournement

Utilisez le nom de l’utilisateur du compte de stockage pour copier les fichiers :

- `Useadd : [storage account name]`
- `Passwd [storage account name]`
- `Su [storage account name]`
- `Cp -p filename.txt /share`

## <a name="ls-cannot-access-ltpathgt-inputoutput-error"></a>ls : impossible d’accéder à « &lt;chemin&gt; » : Erreur d’entrée/sortie

Quand vous tentez de répertorier les fichiers d’un partage de fichiers Azure à l’aide de la commande ls, cette commande se bloque lors de l’affichage de la liste des fichiers. Vous obtenez l’erreur suivante :

**ls : impossible d’accéder à « &lt;chemin&gt; » : Erreur d’entrée/sortie**


### <a name="solution"></a>Solution
Mettez à niveau le noyau Linux vers les versions suivantes qui incluent un correctif pour ce problème :

- 4.4.87+
- 4.9.48+
- 4.12.11+
- Toutes les versions à partir de 4.13

## <a name="cannot-create-symbolic-links---ln-failed-to-create-symbolic-link-t-operation-not-supported"></a>Impossible de créer des liens symboliques - ln : échec de création du lien symbolique « t » : Opération non prise en charge

### <a name="cause"></a>Cause
Par défaut, le montage des partages de fichiers Azure sur Linux à l’aide de CIFS n’active pas la prise en charge des liens symboliques (symlinks). Une erreur comme celle-ci s’affiche :
```
ln -s linked -n t
ln: failed to create symbolic link 't': Operation not supported
```
### <a name="solution"></a>Solution
Le client Linux CIFS ne prend pas en charge la création de liens symboliques de type Windows via le protocole SMB 2 ou 3. Le client Linux prend actuellement en charge un autre style de liens symboliques appelés [liens symboliques Mishall+French](https://wiki.samba.org/index.php/UNIX_Extensions#Minshall.2BFrench_symlinks) pour créer et suivre les opérations. Les clients ayant besoin de liens symboliques peuvent utiliser l’option de montage « mfsymlinks ». Nous recommandons « mfsymlinks », car il s’agit également du format utilisé par les ordinateurs Mac.

Pour utiliser les liens symboliques, ajoutez le code suivant à la fin de votre commande de montage CIFS :

```
,mfsymlinks
```

La commande ressemble alors à ceci :

```
sudo mount -t cifs //<storage-account-name>.file.core.windows.net/<share-name> <mount-point> -o vers=<smb-version>,username=<storage-account-name>,password=<storage-account-key>,dir_mode=0777,file_mode=0777,serverino,mfsymlinks
```

Vous pouvez ensuite créer des liens symboliques comme suggéré sur le [wiki](https://wiki.samba.org/index.php/UNIX_Extensions#Storing_symlinks_on_Windows_servers).

[!INCLUDE [storage-files-condition-headers](../../../includes/storage-files-condition-headers.md)]

<a id="error112"></a>
## <a name="mount-error112-host-is-down-because-of-a-reconnection-time-out"></a>« Erreur de montage (112) : L’hôte est hors service » en raison de l’expiration du délai de reconnexion

L’erreur de montage 112 se produit sur le client Linux s’il est resté inactif pendant une période prolongée. Après une durée d’inactivité prolongée, le client se déconnecte, et la connexion arrive à expiration.  

### <a name="cause"></a>Cause

La connexion peut être inactive pour les raisons suivantes :

-   Des échecs de communication réseau qui empêchent le rétablissement d’une connexion TCP avec le serveur quand l’option de montage « conditionnel » par défaut est utilisée
-   Les correctifs de reconnexion récents qui ne sont pas présents dans les anciens noyaux.

### <a name="solution"></a>Solution

Ce problème de reconnexion dans le noyau Linux est maintenant résolu dans le cadre des modifications suivantes :

- [Résoudre le problème de reconnexion pour ne pas différer la reconnexion de la session SMB3 longtemps après la reconnexion du socket](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/fs/cifs?id=4fcd1813e6404dd4420c7d12fb483f9320f0bf93)
- [Appeler le service d’écho immédiatement après la reconnexion du socket](https://git.kernel.org/pub/scm/linux/kernel/git/torvalds/linux.git/commit/?id=b8c600120fc87d53642476f48c8055b38d6e14c7)
- [CIFS : Corriger une éventuelle corruption de la mémoire pendant la reconnexion](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=53e0e11efe9289535b060a51d4cf37c25e0d0f2b)
- [CIFS : Corriger un éventuel double verrouillage du mutex pendant la reconnexion (pour les noyaux v4.9 et ultérieurs)](https://git.kernel.org/cgit/linux/kernel/git/torvalds/linux.git/commit/?id=96a988ffeb90dba33a71c3826086fe67c897a183)

Toutefois, il se peut que ces modifications n’aient pas encore été appliquées à l’ensemble des distributions Linux. Si vous utilisez une distribution Linux populaire, vous pouvez vérifier la page [Utiliser Azure Files avec Linux](storage-how-to-use-files-linux.md) pour voir quelle version de votre distribution a les modifications de noyau nécessaires.

### <a name="workaround"></a>Solution de contournement

Vous pouvez contourner ce problème en spécifiant un montage inconditionnel. Un montage inconditionnel force le client à attendre jusqu'à ce qu’une connexion soit établie ou jusqu'à ce qu’elle soit explicitement interrompue. Vous pouvez l’utiliser pour empêcher des erreurs dues à des délais d’attente réseau. Toutefois, cette solution de contournement peut provoquer des attentes indéfinies. Préparez-vous à arrêter les connexions si nécessaire.

Si vous ne pouvez pas effectuer de mise à niveau vers les dernières versions du noyau, vous pouvez contourner ce problème en conservant un fichier dans le partage de fichiers Azure dans lequel vous écrivez toutes les 30 secondes au plus. Il doit s’agir d’une opération d’écriture, telle que la réécriture de la date de création ou de modification du fichier. Sinon, vous pouvez obtenir les résultats mis en cache, et votre opération peut ne pas déclencher la reconnexion.

## <a name="cifs-vfs-error--22-on-ioctl-to-get-interface-list-when-you-mount-an-azure-file-share-by-using-smb-30"></a>« CIFS VFS: error -22 on ioctl to get interface list » lorsque vous montez un partage de fichiers Azure à l’aide de SMB 3.0

### <a name="cause"></a>Cause
Cette erreur est consignée, car Azure Files [ne prend actuellement pas en charge SMB Multichannel](/rest/api/storageservices/features-not-supported-by-the-azure-file-service).

### <a name="solution"></a>Solution
Cette erreur peut être ignorée.


### <a name="unable-to-access-folders-or-files-which-name-has-a-space-or-a-dot-at-the-end"></a>Impossible d’accéder aux dossiers ou aux fichiers dont le nom contient un espace ou un point à la fin

Vous ne pouvez pas accéder à des dossiers ou des fichiers à partir du partage de fichiers Azure monté sur Linux. Les commandes telles que du et ls et/ou les applications tierces peuvent échouer avec une erreur « Aucun fichier ou répertoire de ce type » lors de l’accès au partage. Toutefois, vous pouvez charger des fichiers vers ces dossiers via le portail.

### <a name="cause"></a>Cause :

Les dossiers ou fichiers ont été chargés à partir d’un système qui code les caractères à la fin du nom en un caractère différent. Les fichiers chargés à partir d’un ordinateur Macintosh peuvent avoir un caractère « 0xF028 » ou « 0xF029 » au lieu de 0x20 (Space) ou 0X2E (point).

### <a name="solution"></a>Solution

Utilisez l’option mapchars sur le partage lors du montage du partage sur Linux : 

au lieu de :

```bash
sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
```

utiliser :

```bash
sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino,mapchars
```


## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contactez le support technique.

Si vous avez encore besoin d’aide, [contactez le support technique](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre rapidement votre problème.
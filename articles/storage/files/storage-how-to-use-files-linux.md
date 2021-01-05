---
title: Utiliser Azure Files avec Linux | Microsoft Docs
description: Découvrez comment monter un partage de fichiers Azure via SMB sur Linux. Consultez la liste des conditions requises. Passez en revue les considérations relatives à la sécurité SMB sur les clients Linux.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 10/19/2019
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: c271107b85e4903153c29b58aadadd37fb051b76
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96022562"
---
# <a name="use-azure-files-with-linux"></a>Utiliser Azure Files avec Linux
[Azure Files](storage-files-introduction.md) est le système de fichiers cloud facile à utiliser de Microsoft. Les partages de fichiers Azure peuvent être montés dans des distributions Linux à l’aide du [client SMB en mode noyau](https://wiki.samba.org/index.php/LinuxCIFS). Cet article présente deux méthodes de montage d’un partage de fichiers Azure : à la demande avec la commande `mount` et au démarrage en créant une entrée dans `/etc/fstab`.

La méthode recommandée pour monter un partage de fichiers Azure sur Linux est d’utiliser SMB 3.0. Par défaut, Azure Files exige un chiffrement en transit, que seul SMB 3.0 prend en charge. Azure Files prend aussi en charge SMB 2.1, qui ne prend pas en charge le chiffrement en transit, mais des impératifs de sécurité peuvent vous interdire de monter des partages de fichiers Azure avec SMB 2.1 à partir d’une autre région Azure ou de votre environnement local. À moins que votre application ne l’exige précisément, il y a peu de raisons d’utiliser SMB 2.1, puisque les distributions Linux les plus répandues et les plus récentes prennent en charge SMB 3.0 :  

| Distribution Linux | SMB 2.1 <br>(Montages sur des machines virtuelles au sein de la même région Azure) | SMB 3.0 <br>(Montages à partir du site local et entre les régions) |
| --- | :---: | :---: |
| Ubuntu | 14.04+ | 16.04+ |
| Red Hat Enterprise Linux (RHEL) | 7+ | 7.5+ |
| CentOS | 7+ |  7.5+ |
| Debian | 8+ | > 10 |
| OpenSUSE | 13.2+ | 42.3+ |
| SUSE Linux Enterprise Server | 12+ | 12 SP2+ |

Si vous utilisez une distribution Linux non listée dans le tableau ci-dessus, vous pouvez vérifier si votre distribution Linux prend en charge SMB 3.0 avec le chiffrement en vérifiant la version du noyau Linux. SMB 3.0 avec chiffrement a été ajouté à la version 4.11 du noyau Linux. La commande `uname` retourne la version du noyau Linux en cours d’utilisation :

```bash
uname -r
```

## <a name="prerequisites"></a>Conditions préalables requises
<a id="smb-client-reqs"></a>

* <a id="install-cifs-utils"></a>**Vérifiez que le package cifs-utils est installé.**  
    Le package cifs-utils peut être installé à l’aide du gestionnaire de package sur la distribution Linux de votre choix. 

    Sur les distributions **Ubuntu** et **basées sur Debian**, utilisez le gestionnaire de packages `apt` :

    ```bash
    sudo apt update
    sudo apt install cifs-utils
    ```

    Sur **Fedora**, **Red Hat Enterprise Linux 8+** et **CentOS 8 +**, utilisez le gestionnaire de packages `dnf` :

    ```bash
    sudo dnf install cifs-utils
    ```

    Sur les versions plus anciennes de **Red Hat Enterprise Linux** et **CentOS**, utilisez le gestionnaire de packages `yum` :

    ```bash
    sudo yum install cifs-utils 
    ```

    Sur **openSUSE**, utilisez le gestionnaire de packages `zypper` :

    ```bash
    sudo zypper install cifs-utils
    ```

    Sur les autres distributions, utilisez le gestionnaire de package approprié ou effectuez une [compilation à partir de la source](https://wiki.samba.org/index.php/LinuxCIFS_utils#Download).

* **La dernière version de l’interface de ligne de commande Azure (CLI).** Pour plus d'informations sur l'installation d’Azure CLI, consultez [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli?view=azure-cli-latest) et sélectionnez votre système d’exploitation. Si vous préférez utiliser le module Azure PowerShell dans PowerShell 6+, rien ne vous en empêche. Sachez cependant que les instructions ci-dessous s’appliquent à Azure CLI.

* **Vérifiez que le port 445 est ouvert** : SMB communique via le port TCP 445. Assurez-vous que votre pare-feu ne bloque pas les ports TCP 445 de la machine cliente.  Remplacez `<your-resource-group>` et `<your-storage-account>`, puis exécutez le script suivant :
    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))
    fileHost=$(echo $smbPath | tr -d "/")

    nc -zvw3 $fileHost 445
    ```

    Si la connexion a été établie, vous devez obtenir une sortie semblable à la suivante :

    ```
    Connection to <your-storage-account> 445 port [tcp/microsoft-ds] succeeded!
    ```

    Si vous ne parvenez pas à ouvrir le port 445 sur votre réseau d’entreprise ou si vous n’y êtes pas autorisé par un fournisseur de services Internet, vous pouvez utiliser une connexion VPN ou ExpressRoute pour contourner le port 445. Pour plus d’informations, consultez [Considérations relatives à la mise en réseau pour un accès direct à un partage de fichiers Azure](storage-files-networking-overview.md).

## <a name="mounting-azure-file-share"></a>Montage d’un partage de fichiers Azure
Pour utiliser un partage de fichiers Azure avec votre distribution Linux, vous devez créer un répertoire qui servira de point de montage pour le partage de fichiers Azure. Vous pouvez créer un point de montage n’importe où sur votre système Linux, mais il est d’usage de le créer sous /mnt. Après le point de montage, utilisez la commande `mount` pour accéder au partage de fichiers Azure.

Vous pouvez monter le même partage de fichiers Azure sur plusieurs points de montage, si vous le souhaitez.

### <a name="mount-the-azure-file-share-on-demand-with-mount"></a>Montage du partage de fichiers Azure à la demande avec `mount`
1. **Créez un dossier pour le point de montage** : Remplacez `<your-resource-group>`, `<your-storage-account>` et `<your-file-share>` par les informations correspondant à votre environnement :

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Utilisez la commande de montagne pour monter le partage de fichiers Azure**. Dans l’exemple ci-dessous, les autorisations de dossiers et de fichiers Linux locales sont par défaut 0755, ce qui signifie lecture, écriture et exécution pour le propriétaire (en fonction du propriétaire Linux des fichiers/répertoires), lecture et exécution pour les utilisateurs du groupe propriétaire, et lecture et exécution pour les autres utilisateurs du système. Vous pouvez utiliser les options de montage `uid` et `gid` pour définir l’ID d’utilisateur et l’ID de groupe pour le montage. Vous pouvez aussi utiliser `dir_mode` et `file_mode` pour définir des autorisations personnalisées comme vous le souhaitez. Pour plus d’informations sur la façon de définir les autorisations, consultez la [notation numérique UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) sur Wikipédia. 

    ```bash
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')

    sudo mount -t cifs $smbPath $mntPath -o vers=3.0,username=$storageAccountName,password=$storageAccountKey,serverino
    ```

    > [!Note]  
    > La commande de montage ci-dessus effectue le montage avec SMB 3.0. Si votre distribution Linux ne prend pas en charge SMB 3.0 avec chiffrement ou si elle ne prend en charge que SMB 2.1, vous ne pouvez effectuer le montage qu’à partir d’une machine virtuelle Azure de la même région que le compte de stockage. Pour monter votre partage de fichiers Azure sur une distribution Linux qui ne prend pas en charge SMB 3.0 avec chiffrement, vous devez [désactiver le chiffrement en transit pour le compte de stockage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

Quand vous avez terminé d’utiliser le partage de fichiers Azure, vous pouvez utiliser `sudo umount $mntPath` pour démonter le partage.

### <a name="create-a-persistent-mount-point-for-the-azure-file-share-with-etcfstab"></a>Création d’un point de montage persistant pour le partage de fichiers Azure avec `/etc/fstab`
1. **Créez un dossier pour le point de montage** : Vous pouvez créer un dossier pour un point de montage n’importe où sur le système de fichiers, mais il est d’usage de le créer sous le dossier /mnt. Par exemple, la commande suivante crée un nouveau répertoire. Remplacez `<your-resource-group>`, `<your-storage-account>` et `<your-file-share>` par les informations correspondant à votre environnement :

    ```bash
    resourceGroupName="<your-resource-group>"
    storageAccountName="<your-storage-account>"
    fileShareName="<your-file-share>"

    mntPath="/mnt/$storageAccountName/$fileShareName"

    sudo mkdir -p $mntPath
    ```

1. **Créez un fichier d’informations d’identification pour stocker le nom d’utilisateur (nom du compte de stockage) et le mot de passe (clé du compte de stockage) pour le partage de fichiers.** 

    ```bash
    if [ ! -d "/etc/smbcredentials" ]; then
        sudo mkdir "/etc/smbcredentials"
    fi

    storageAccountKey=$(az storage account keys list \
        --resource-group $resourceGroupName \
        --account-name $storageAccountName \
        --query "[0].value" | tr -d '"')
    
    smbCredentialFile="/etc/smbcredentials/$storageAccountName.cred"
    if [ ! -f $smbCredentialFile ]; then
        echo "username=$storageAccountName" | sudo tee $smbCredentialFile > /dev/null
        echo "password=$storageAccountKey" | sudo tee -a $smbCredentialFile > /dev/null
    else 
        echo "The credential file $smbCredentialFile already exists, and was not modified."
    fi
    ```

1. **Modifiez les autorisations du fichier d’informations d’identification afin que seule la racine puisse lire ou modifier le fichier du mot de passe.** Étant donné que la clé du compte de stockage est avant tout un mot de passe de super administrateur pour le compte de stockage, il est important de définir des autorisations sur le fichier de sorte que seule la racine puisse y accéder. Ainsi, les utilisateurs de privilège inférieurs ne peuvent pas récupérer la clé du compte de stockage.   

    ```bash
    sudo chmod 600 $smbCredentialFile
    ```

1. **Utilisez la commande suivante pour ajouter la ligne suivante à `/etc/fstab`**  : Dans l’exemple ci-dessous, les autorisations de dossiers et de fichiers Linux locales sont par défaut 0755, ce qui signifie lecture, écriture et exécution pour le propriétaire (en fonction du propriétaire Linux des fichiers/répertoires), lecture et exécution pour les utilisateurs du groupe propriétaire, et lecture et exécution pour les autres utilisateurs du système. Vous pouvez utiliser les options de montage `uid` et `gid` pour définir l’ID d’utilisateur et l’ID de groupe pour le montage. Vous pouvez aussi utiliser `dir_mode` et `file_mode` pour définir des autorisations personnalisées comme vous le souhaitez. Pour plus d’informations sur la façon de définir les autorisations, consultez la [notation numérique UNIX](https://en.wikipedia.org/wiki/File_system_permissions#Numeric_notation) sur Wikipédia.

    ```bash
    # This command assumes you have logged in with az login
    httpEndpoint=$(az storage account show \
        --resource-group $resourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | tr -d '"')
    smbPath=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint))$fileShareName

    if [ -z "$(grep $smbPath\ $mntPath /etc/fstab)" ]; then
        echo "$smbPath $mntPath cifs nofail,vers=3.0,credentials=$smbCredentialFile,serverino" | sudo tee -a /etc/fstab > /dev/null
    else
        echo "/etc/fstab was not modified to avoid conflicting entries as this Azure file share was already present. You may want to double check /etc/fstab to ensure the configuration is as desired."
    fi

    sudo mount -a
    ```
    
    > [!Note]  
    > La commande de montage ci-dessus effectue le montage avec SMB 3.0. Si votre distribution Linux ne prend pas en charge SMB 3.0 avec chiffrement ou si elle ne prend en charge que SMB 2.1, vous ne pouvez effectuer le montage qu’à partir d’une machine virtuelle Azure de la même région que le compte de stockage. Pour monter votre partage de fichiers Azure sur une distribution Linux qui ne prend pas en charge SMB 3.0 avec chiffrement, vous devez [désactiver le chiffrement en transit pour le compte de stockage](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

### <a name="using-autofs-to-automatically-mount-the-azure-file-shares"></a>Utilisation d’autofs pour monter automatiquement le ou les partages de fichiers Azure

1. **Vérifiez que le package autofs est installé.**  

    Vous pouvez installer le package autofs à l’aide du gestionnaire de package sur la distribution Linux de votre choix. 

    Sur les distributions **Ubuntu** et **basées sur Debian**, utilisez le gestionnaire de packages `apt` :
    ```bash
    sudo apt update
    sudo apt install autofs
    ```
    Sur **Fedora**, **Red Hat Enterprise Linux 8+** et **CentOS 8 +**, utilisez le gestionnaire de packages `dnf` :
    ```bash
    sudo dnf install autofs
    ```
    Sur les versions plus anciennes de **Red Hat Enterprise Linux** et **CentOS**, utilisez le gestionnaire de packages `yum` :
    ```bash
    sudo yum install autofs 
    ```
    Sur **openSUSE**, utilisez le gestionnaire de packages `zypper` :
    ```bash
    sudo zypper install autofs
    ```
2. **Créez un point de montage pour le ou les partages** :
   ```bash
    sudo mkdir /fileshares
    ```
3. **Créez un fichier de configuration autofs personnalisé** :
    ```bash
    sudo vi /etc/auto.fileshares
    ```
4. **Ajoutez les entrées suivantes à /etc/auto.fileshares** :
   ```bash
   echo "$fileShareName -fstype=cifs,credentials=$smbCredentialFile :$smbPath"" > /etc/auto.fileshares
   ```
5. **Ajoutez l’entrée suivante à /etc/auto.master** :
   ```bash
   /fileshares /etc/auto.fileshares --timeout=60
   ```
6. **Redémarrez autofs** :
    ```bash
    sudo systemctl restart autofs
    ```
7.  **Accédez au dossier désigné pour le partage** :
    ```bash
    cd /fileshares/$filesharename
    ```
## <a name="securing-linux"></a>Sécurisation de Linux
Pour monter un partage de fichiers Azure sur Linux, le port 445 doit être accessible. De nombreuses organisations bloquent le port 445 en raison des risques de sécurité inhérents à SMB 1. SMB 1, aussi appelé CIFS (Common Internet File System), est un protocole de système de fichiers hérité intégré à de nombreuses distributions Linux. SMB 1 est un protocole obsolète, inefficace et qui pose surtout des problèmes de sécurité. La bonne nouvelle, c’est qu’Azure Files ne prend pas en charge SMB 1 et qu’à partir de la version 4.18 de son noyau, Linux permet de désactiver SMB 1. Dans tous les cas, nous vous [recommandons vivement](https://aka.ms/stopusingsmb1) de désactiver SMB 1 sur vos clients Linux avant d’utiliser des partages de fichiers SMB en production.

À partir de la version 4.18 du noyau Linux, le module de noyau SMB, appelé `cifs` pour des raisons d’héritage, expose un nouveau paramètre de module (souvent appelé *param* par diverses documentations externes) sous le nom `disable_legacy_dialects`. Bien qu’ils aient été introduits dans la version 4.18 du noyau Linux, certains fournisseurs ont rétroporté cette modification sur des noyaux plus anciens qu’ils prennent en charge. Pour des raisons pratiques, le tableau suivant détaille la disponibilité de ce paramètre de module sur les distributions Linux courantes.

| Distribution | Possibilité de désactivation de SMB 1 |
|--------------|-------------------|
| Ubuntu 14.04-16.04 | Non |
| Ubuntu 18.04 | Oui |
| Ubuntu 19.04+ | Oui |
| Debian 8-9 | Non |
| Debian 10+ | Oui |
| Fedora 29+ | Oui |
| CentOS 7 | Non | 
| CentOS 8+ | Oui |
| Red Hat Enterprise Linux 6.x-7.x | Non |
| Red Hat Enterprise Linux 8+ | Oui |
| openSUSE Leap 15.0 | Non |
| openSUSE Leap 15.1+ | Oui |
| openSUSE Tumbleweed | Oui |
| SUSE Linux Enterprise 11.x-12.x | Non |
| SUSE Linux Enterprise 15 | Non |
| SUSE Linux Enterprise 15.1 | Non |

Vous pouvez vérifier si votre distribution Linux prend en charge le paramètre de module `disable_legacy_dialects` via la commande suivante.

```bash
sudo modinfo -p cifs | grep disable_legacy_dialects
```

Cette commande doit générer le message suivant :

```output
disable_legacy_dialects: To improve security it may be helpful to restrict the ability to override the default dialects (SMB2.1, SMB3 and SMB3.02) on mount with old dialects (CIFS/SMB1 and SMB2) since vers=1.0 (CIFS/SMB1) and vers=2.0 are weaker and less secure. Default: n/N/0 (bool)
```

Avant de désactiver SMB 1, vous devez vérifier que le module SMB n’est pas actuellement chargé sur votre système (ce qui se produit automatiquement si vous avez monté un partage SMB). Pour cela, vous pouvez utiliser la commande suivante, qui ne doit générer aucun résultat si SMB n’est pas chargé :

```bash
lsmod | grep cifs
```

Pour décharger le module, commencez par démonter tous les partages SMB (à l’aide de la commande `umount` comme décrit ci-dessus). Vous pouvez identifier tous les partages SMB montés sur votre système à l’aide de la commande suivante :

```bash
mount | grep cifs
```

Une fois que vous avez démonté tous les partages de fichiers SMB, vous pouvez décharger le module sans risque. Pour cela, utilisez la commande `modprobe` :

```bash
sudo modprobe -r cifs
```

Vous pouvez charger manuellement le module avec SMB 1 déchargé à l’aide de la commande `modprobe` :

```bash
sudo modprobe cifs disable_legacy_dialects=Y
```

Enfin, vous pouvez vérifier que le module SMB a bien été chargé avec le paramètre en examinant les paramètres chargés dans `/sys/module/cifs/parameters` :

```bash
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

Pour désactiver de façon permanente SMB 1 sur les distributions Ubuntu et Debian, vous devez créer un fichier (si vous n’avez pas déjà d’options personnalisées pour d’autres modules) sous le nom `/etc/modprobe.d/local.conf` avec le paramètre. Pour ce faire, vous pouvez utiliser la commande suivante :

```bash
echo "options cifs disable_legacy_dialects=Y" | sudo tee -a /etc/modprobe.d/local.conf > /dev/null
```

Vous pouvez vérifier que cela a fonctionné en chargeant le module SMB :

```bash
sudo modprobe cifs
cat /sys/module/cifs/parameters/disable_legacy_dialects
```

## <a name="next-steps"></a>Étapes suivantes
Consultez ces liens pour en savoir plus sur Azure Files :

* [Planification d’un déploiement Azure Files](storage-files-planning.md)
* [FORUM AUX QUESTIONS](./storage-files-faq.md)
* [Dépannage](storage-troubleshoot-linux-file-connection-problems.md)
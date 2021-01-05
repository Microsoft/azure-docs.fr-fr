---
title: Étapes détaillées pour créer une paire de clés SSH
description: Découvrez les étapes détaillées pour créer et gérer une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/31/2020
ms.author: cynthn
ms.openlocfilehash: 34a84ed333172ea0931c529d2dbeee1b774ae8c5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016362"
---
# <a name="detailed-steps-create-and-manage-ssh-keys-for-authentication-to-a-linux-vm-in-azure"></a>Étapes détaillées : Créer et gérer des clés SSH pour l’authentification sur une machine virtuelle Linux dans Azure

Avec une paire de clés SSH (Secure Shell), vous pouvez créer une machine virtuelle Linux qui utilise des clés SSH pour l’authentification. Cet article décrit comment créer et utiliser une paire de fichiers de clés publique-privée RSA SSH pour les connexions de client SSH.

Pour connaître les commandes rapides, consultez [Comment créer une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure](mac-create-ssh-keys.md).

Pour créer des clés SSH et les utiliser pour se connecter à un ordinateur **Windows**, consultez [Comment utiliser des clés SSH avec Windows sur Azure](ssh-from-windows.md). Vous pouvez également utiliser le [portail Azure](../ssh-keys-portal.md) pour créer et gérer des clés SSH pour la création de machines virtuelles dans le portail.

[!INCLUDE [virtual-machines-common-ssh-overview](../../../includes/virtual-machines-common-ssh-overview.md)]

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="ssh-keys-use-and-benefits"></a>Utilisation et avantages des clés SSH

Quand vous créez une machine virtuelle Azure en spécifiant la clé publique, Azure copie cette clé (au format `.pub`) dans le dossier `~/.ssh/authorized_keys` sur la machine virtuelle. Les clés SSH dans `~/.ssh/authorized_keys` servent à demander au client la clé privée correspondante sur une connexion SSH. Lorsqu’une machine virtuelle Linux Azure utilise des clés SSH pour l’authentification, Azure configure le serveur SSHD pour ne pas autoriser les connexions par mot de passe, mais seulement les clés SSH. En créant une machine virtuelle Linux Azure avec des clés SSH, vous sécurisez le déploiement de machines virtuelles et évitez l’étape de configuration post-déploiement classique qui consiste à désactiver les mots de passe dans le fichier `sshd_config`.

Si vous ne voulez pas utiliser de clés SSH, vous pouvez configurer votre machine virtuelle Linux pour utiliser une authentification par mot de passe. Si votre machine virtuelle n’est pas exposée à Internet, l’utilisation de mots de passe peut être suffisante. Toutefois, vous devez toujours gérer vos mots de passe pour chaque machine virtuelle Linux et maintenir des stratégies de mot de passe efficaces, par exemple, en spécifiant une longueur minimale de mot de passe et des mises à jour régulières. 

## <a name="generate-keys-with-ssh-keygen"></a>Générer des clés avec ssh-keygen

Pour créer les clés, la commande par défaut est `ssh-keygen`, disponible avec les utilitaires OpenSSH dans Azure Cloud Shell, un hôte macOS ou Linux, et Windows 10. `ssh-keygen` pose une série de questions, puis écrit une clé privée et la clé publique correspondante. 

Par défaut, les clés SSH sont conservées dans le `~/.ssh`répertoire.  Si vous n’avez pas de répertoire `~/.ssh`, la commande `ssh-keygen` en crée un pour vous avec les autorisations appropriées.

### <a name="basic-example"></a>Exemple de base

La commande `ssh-keygen` suivante génère des fichiers de clés publique et privée SSH RSA 4096 bits par défaut dans le répertoire `~/.ssh`. Si une paire de clés SSH existe dans l’emplacement actuel, les fichiers sont remplacés.

```bash
ssh-keygen -m PEM -t rsa -b 4096
```

### <a name="detailed-example"></a>Exemple détaillé
L’exemple suivant montre d’autres options de commande pour créer une paire de clés SSH RSA. Si une paire de clés SSH existe dans l’emplacement actuel, les fichiers sont remplacés. 

```bash
ssh-keygen \
    -m PEM \
    -t rsa \
    -b 4096 \
    -C "azureuser@myserver" \
    -f ~/.ssh/mykeys/myprivatekey \
    -N mypassphrase
```

**Explication des commandes**

`ssh-keygen` = programme utilisé pour créer les clés

`-m PEM` = mettre en forme la clé au format PEM

`-t rsa` = Type de clé à créer, dans notre cas au format RSA

`-b 4096` = Nombre de bits dans la clé, dans notre cas 4096

`-C "azureuser@myserver"` = commentaire ajouté à la fin du fichier de la clé publique pour l’identifier facilement. Une adresse e-mail est généralement utilisée en commentaire, mais vous pouvez utiliser ce qui convient le mieux à votre infrastructure.

`-f ~/.ssh/mykeys/myprivatekey` = Nom du fichier de clé privée, si vous choisissez de ne pas utiliser le nom par défaut. Le fichier de clé publique correspondant avec l’extension `.pub` est généré dans le même répertoire. Le répertoire doit exister.

`-N mypassphrase` = Phrase secrète supplémentaire utilisée pour accéder au fichier de clé privée. 

### <a name="example-of-ssh-keygen"></a>Exemple de ssh-keygen

```bash
ssh-keygen -t -m PEM rsa -b 4096 -C "azureuser@myserver"
Generating public/private rsa key pair.
Enter file in which to save the key (/home/azureuser/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/azureuser/.ssh/id_rsa.
Your public key has been saved in /home/azureuser/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:vFfHHrpSGQBd/oNdvNiX0sG9Vh+wROlZBktNZw9AUjA azureuser@myserver
The key's randomart image is:
+---[RSA 4096]----+
|        .oE=*B*+ |
|          o+o.*++|
|           .oo++*|
|       .    .B+.O|
|        S   o=BO.|
|         . .o++o |
|        . ... .  |
|         ..  .   |
|           ..    |
+----[SHA256]-----+
```

#### <a name="saved-key-files"></a>Fichiers de clés enregistrés

`Enter file in which to save the key (/home/azureuser/.ssh/id_rsa): ~/.ssh/id_rsa`

Nom de la paire de clés dans cet article. Le nom par défaut de la paire de clés est `id_rsa`. Comme certains outils recherchent ce nom de fichier de clé privée `id_rsa`, il est utile de le conserver. Le répertoire `~/.ssh/` est l’emplacement par défaut des paires de clés SSH et du fichier de configuration SSH. Si aucun chemin d’accès complet n’est spécifié, `ssh-keygen` crée les clés dans le répertoire de travail actuel, et non dans le répertoire par défaut `~/.ssh`.

#### <a name="list-of-the-ssh-directory"></a>Liste du répertoire `~/.ssh`

```bash
ls -al ~/.ssh
-rw------- 1 azureuser staff  1675 Aug 25 18:04 id_rsa
-rw-r--r-- 1 azureuser staff   410 Aug 25 18:04 id_rsa.pub
```

#### <a name="key-passphrase"></a>Phrase secrète de la clé

`Enter passphrase (empty for no passphrase):`

Nous vous recommandons *vivement* d’ajouter une phrase secrète à votre clé privée. Sans phrase secrète pour protéger le fichier de clé, toute personne ayant le fichier peut l’utiliser pour se connecter à un serveur contenant la clé publique correspondante. L’ajout d’une phrase secrète offre donc une protection supplémentaire si un utilisateur malveillant parvient à accéder à votre fichier de clé privée. Vous avez ainsi plus de temps pour changer les clés.

## <a name="generate-keys-automatically-during-deployment"></a>Générer les clés automatiquement pendant le déploiement

Si vous utilisez [Azure CLI](/cli/azure) pour créer votre machine virtuelle, vous pouvez éventuellement générer des fichiers de clés SSH publiques et privées en exécutant la commande [az vm create](/cli/azure/vm) avec l’option `--generate-ssh-keys`. Les clés sont stockées dans le répertoire ~/.ssh. Notez que cette option de commande ne remplace pas les clés qui existent déjà dans l’emplacement, le cas échéant.

## <a name="provide-ssh-public-key-when-deploying-a-vm"></a>Fournir la clé publique SSH pendant le déploiement d’une machine virtuelle

Pour créer une machine virtuelle Linux qui utilise des clés SSH pour l’authentification, fournissez votre clé publique SSH à la création de la machine virtuelle à l’aide du portail Azure, de l’interface CLI, des modèles Resource Manager ou d’autres méthodes. Si vous utilisez le portail, vous entrez la clé publique elle-même. Si vous utilisez [Azure CLI](/cli/azure) pour créer votre machine virtuelle avec une clé publique existante, spécifiez la valeur ou l’emplacement de cette clé publique en exécutant la commande [az vm create](/cli/azure/vm) avec l’option `--ssh-key-value`. 

Si vous n’êtes pas familiarisé avec le format des clés publiques SSH, vous pouvez voir votre clé publique en exécutant `cat` comme suit, en remplaçant `~/.ssh/id_rsa.pub` par l’emplacement de votre propre fichier de clé publique :

```bash
cat ~/.ssh/id_rsa.pub
```

Le résultat est semblable à ce qui suit (illustré ici de manière rédigée) :

```
ssh-rsa XXXXXXXXXXc2EAAAADAXABAAABAXC5Am7+fGZ+5zXBGgXS6GUvmsXCLGc7tX7/rViXk3+eShZzaXnt75gUmT1I2f75zFn2hlAIDGKWf4g12KWcZxy81TniUOTjUsVlwPymXUXxESL/UfJKfbdstBhTOdy5EG9rYWA0K43SJmwPhH28BpoLfXXXXXG+/ilsXXXXXKgRLiJ2W19MzXHp8z3Lxw7r9wx3HaVlP4XiFv9U4hGcp8RMI1MP1nNesFlOBpG4pV2bJRBTXNXeY4l6F8WZ3C4kuf8XxOo08mXaTpvZ3T1841altmNTZCcPkXuMrBjYSJbA8npoXAXNwiivyoe3X2KMXXXXXdXXXXXXXXXXCXXXXX/ azureuser@myserver
```

Si vous copiez et collez le contenu du fichier de clé publique dans le portail Azure ou dans un modèle Resource Manager, ne copiez pas les espaces blancs supplémentaires et n’introduisez pas de sauts de ligne. Par exemple, si vous utilisez macOS, vous pouvez diriger le fichier de clé publique (par défaut, `~/.ssh/id_rsa.pub`) sur **pbcopy** pour en copier le contenu (d’autres programmes Linux ont la même fonction, par exemple, `xclip`).

Si vous préférez utiliser une clé publique avec un format multiligne, vous pouvez générer une clé au format RFC4716 dans un conteneur PEM à partir de la clé publique précédemment créée.

Pour créer une clé au format RFC4716 à partir d’une clé publique SSH existante, procédez comme suit :

```bash
ssh-keygen \
-f ~/.ssh/id_rsa.pub \
-e \
-m RFC4716 > ~/.ssh/id_ssh2.pem
```

## <a name="ssh-to-your-vm-with-an-ssh-client"></a>Établir une connexion SSH sur votre machine virtuelle avec un client SSH
Une fois la clé publique déployée sur votre machine virtuelle Azure et la clé privée sur votre système local, établissez une connexion SSH sur votre machine virtuelle à l’aide de l’adresse IP ou du nom DNS de votre machine virtuelle. Remplacez *azureuser* et *myvm.westus.cloudapp.azure.com* dans la commande suivante par le nom d’administrateur et le nom de domaine complet (ou adresse IP) :

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si vous avez fourni un mot de passe lorsque vous avez créé votre paire de clés, saisissez-le lorsque vous y êtes invité pendant le processus de connexion. (Le serveur est ajouté à votre dossier `~/.ssh/known_hosts` et vous n’avez pas à vous connecter à nouveau tant que la clé publique sur votre machine virtuelle Azure n’est pas modifiée ou que le nom de serveur n’est pas supprimé du dossier `~/.ssh/known_hosts`.)

Si la machine virtuelle utilise la stratégie juste-à-temps, vous devez demander à y accéder avant de pouvoir vous connecter à cette machine virtuelle. Pour plus d’informations sur la stratégie juste-à-temps, consultez [Gérer l’accès à la machine virtuelle à l’aide de la stratégie juste-à-temps](../../security-center/security-center-just-in-time.md).

## <a name="use-ssh-agent-to-store-your-private-key-passphrase"></a>Utiliser ssh-agent pour stocker votre phrase secrète de clé privée

Pour éviter de taper la phrase secrète de votre fichier de clé privée à chaque connexion SSH, vous pouvez utiliser `ssh-agent` pour la mettre en cache. Si vous utilisez un Mac, le trousseau OSX stocke et sécurise la phrase secrète de clé privée quand vous appelez `ssh-agent`.

Vérifiez et utilisez `ssh-agent` et `ssh-add` pour indiquer au système SSH les fichiers de clés et ne pas avoir à utiliser la phrase secrète de manière interactive.

```bash
eval "$(ssh-agent -s)"
```

Ensuite, ajoutez la clé privée à `ssh-agent` à l’aide de la commande `ssh-add`.

```bash
ssh-add ~/.ssh/id_rsa
```

La phrase secrète de clé privée est maintenant stockée dans `ssh-agent`.

## <a name="use-ssh-copy-id-to-copy-the-key-to-an-existing-vm"></a>Utiliser ssh-copy-id pour copier la clé sur une machine virtuelle existante

Si vous avez déjà créé une machine virtuelle, vous pouvez ajouter une clé publique SSH à votre machine virtuelle Linux à l’aide de la commande `ssh-copy-id`.

```bash
ssh-copy-id -i ~/.ssh/id_rsa.pub azureuser@myserver
```

## <a name="create-and-configure-an-ssh-config-file"></a>Créer et configurer un fichier de configuration SSH

Vous pouvez créer et configurer un fichier config SSH (`~/.ssh/config`) pour accélérer les connexions et optimiser le comportement de votre client SSH. 

L’exemple suivant montre une configuration simple que vous pouvez utiliser pour vous connecter rapidement comme utilisateur sur une machine virtuelle spécifique à l’aide de la clé privée SSH par défaut. 

Créez le fichier.

```bash
touch ~/.ssh/config
```

Modifier le fichier pour ajouter la nouvelle configuration SSH

```bash
vim ~/.ssh/config
```

Ajoutez les paramètres de configuration appropriés pour votre machine virtuelle hôte. Dans cet exemple, le nom de la machine virtuelle est *myvm* et le nom du compte *azureuser*.

```bash
# Azure Keys
Host myvm
  Hostname 102.160.203.241
  User azureuser
# ./Azure Keys
```

Vous pouvez ajouter des configurations d’hôtes supplémentaires pour que chaque hôte utilise sa propre paire de clés dédiée. Consultez [Fichier config SSH](https://www.ssh.com/ssh/config/) pour connaître d’autres options de configuration avancées.

Maintenant que vous avez une paire de clés SSH et un fichier de configuration SSH configuré, vous pouvez vous connecter à votre machine virtuelle Linux rapidement et en toute sécurité. Quand vous exécutez la commande suivante, SSH localise et charge tous les paramètres du bloc `Host myvm` dans le fichier config SSH.

```bash
ssh myvm
```

La première fois que vous vous connectez à un serveur à l’aide d’une clé SSH, la commande vous invite à entrer la phrase secrète du fichier de clé.

## <a name="next-steps"></a>Étapes suivantes

L’étape suivante consiste à créer des machines virtuelles Linux de Azure à l’aide de la nouvelle clé publique SSH. Les machines virtuelles Azure créées avec une clé publique SSH comme identifiant de connexion sont mieux sécurisées que celles créées avec la méthode de connexion par défaut (les mots de passe).

* [Créer une machine virtuelle Linux avec le portail Azure](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Créer une machine virtuelle Linux avec Azure CLI](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Créer une machine virtuelle Linux à l’aide d’un modèle Azure](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

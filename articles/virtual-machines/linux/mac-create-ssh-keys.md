---
title: Créer et utiliser une paire de clés SSH pour les machines virtuelles Linux dans Azure
description: Comment créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure afin d’améliorer la sécurité du processus d’authentification.
author: cynthn
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.topic: how-to
ms.date: 12/06/2019
ms.author: cynthn
ms.openlocfilehash: 33ba816227db4cf958fd30c9dac1a0745505c504
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87513687"
---
# <a name="quick-steps-create-and-use-an-ssh-public-private-key-pair-for-linux-vms-in-azure"></a>Étapes rapides : Créer et utiliser une paire de clés publique et privée SSH pour les machines virtuelles Linux dans Azure

Avec une paire de clés SSH (Secure Shell), vous pouvez créer des machines virtuelles dans Azure qui utilisent des clés SSH pour l’authentification. Cet article décrit comment générer et utiliser rapidement une paire de clés publique et privée SSH pour des machines virtuelles Linux. Vous pouvez effectuer ces étapes avec Azure Cloud Shell, un hôte Linux ou macOS. 

> [!NOTE]
> Les machines virtuelles créées à l’aide de clés SSH sont par défaut configurées avec les mots de passe désactivés, ce qui accroît grandement la difficulté des attaques par force brute visant à deviner les mots de passe. 

Pour obtenir plus d’informations et des exemples, consultez [Étapes détaillées pour créer des paires de clés SSH](create-ssh-keys-detailed.md).

Pour découvrir d’autres méthodes permettant de générer et d’utiliser des clés SSH sur un ordinateur Windows, consultez [Utilisation de clés SSH avec Windows sur Azure](ssh-from-windows.md).

[!INCLUDE [virtual-machines-common-ssh-support](../../../includes/virtual-machines-common-ssh-support.md)]

## <a name="create-an-ssh-key-pair"></a>Création d’une paire de clés SSH

Utilisez la commande `ssh-keygen` pour générer des fichiers de clés SSH publiques et privées. Par défaut, ces fichiers sont créés dans le répertoire ~/.ssh. Vous pouvez spécifier un autre emplacement et un mot de passe facultatif (*phrase secrète*) pour accéder au fichier de clé privée. Si une paire de clés SSH portant le même nom existe dans l’emplacement choisi, les fichiers sont remplacés.

La commande suivante crée une paire de clés SSH à l’aide du chiffrement RSA avec une longueur de 4096 :

```bash
ssh-keygen -m PEM -t rsa -b 4096
```

Si vous utilisez [Azure CLI](/cli/azure) pour créer votre machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create), vous pouvez éventuellement générer des fichiers de clés SSH publiques et privées à l’aide de l’option `--generate-ssh-keys`. Les fichiers de clés sont stockés dans le répertoire ~/.ssh, sauf indication contraire avec l’option `--ssh-dest-key-path`. Si une paire de clés SSH existe déjà et que l’option `--generate-ssh-keys` est utilisée, une nouvelle paire de clés n’est pas générée, et la paire de clés existante est utilisée. Dans la commande suivante, remplacez *VMname* et *RGname* par vos propres valeurs :

```azurecli
az vm create --name VMname --resource-group RGname --image UbuntuLTS --generate-ssh-keys 
```

## <a name="provide-an-ssh-public-key-when-deploying-a-vm"></a>Fournir une clé publique SSH lors du déploiement d’une machine virtuelle

Pour créer une machine virtuelle Linux qui utilise des clés SSH pour l’authentification, spécifiez votre clé publique SSH quand vous créez la machine virtuelle à l’aide du portail Azure, d’Azure CLI, de modèles Azure Resource Manager ou d’autres méthodes :

* [Créer une machine virtuelle Linux avec le portail Azure](quick-create-portal.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Créer une machine virtuelle Linux avec Azure CLI](quick-create-cli.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Créer une machine virtuelle Linux à l’aide d’un modèle Azure](create-ssh-secured-vm-from-template.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Si vous n’êtes pas familiarisé avec le format d’une clé publique SSH, vous pouvez afficher votre clé publique à l’aide de la commande `cat` suivante, en remplaçant `~/.ssh/id_rsa.pub` par le chemin et le nom de votre propre fichier de clé publique, si nécessaire :

```bash
cat ~/.ssh/id_rsa.pub
```

Exemple de valeur de clé publique type :

```
ssh-rsa AAAAB3NzaC1yc2EAABADAQABAAACAQC1/KanayNr+Q7ogR5mKnGpKWRBQU7F3Jjhn7utdf7Z2iUFykaYx+MInSnT3XdnBRS8KhC0IP8ptbngIaNOWd6zM8hB6UrcRTlTpwk/SuGMw1Vb40xlEFphBkVEUgBolOoANIEXriAMvlDMZsgvnMFiQ12tD/u14cxy1WNEMAftey/vX3Fgp2vEq4zHXEliY/sFZLJUJzcRUI0MOfHXAuCjg/qyqqbIuTDFyfg8k0JTtyGFEMQhbXKcuP2yGx1uw0ice62LRzr8w0mszftXyMik1PnshRXbmE2xgINYg5xo/ra3mq2imwtOKJpfdtFoMiKhJmSNHBSkK7vFTeYgg0v2cQ2+vL38lcIFX4Oh+QCzvNF/AXoDVlQtVtSqfQxRVG79Zqio5p12gHFktlfV7reCBvVIhyxc2LlYUkrq4DHzkxNY5c9OGSHXSle9YsO3F1J5ip18f6gPq4xFmo6dVoJodZm9N0YMKCkZ4k1qJDESsJBk2ujDPmQQeMjJX3FnDXYYB182ZCGQzXfzlPDC29cWVgDZEXNHuYrOLmJTmYtLZ4WkdUhLLlt5XsdoKWqlWpbegyYtGZgeZNRtOOdN6ybOPJqmYFd2qRtb4sYPniGJDOGhx4VodXAjT09omhQJpE6wlZbRWDvKC55R2d/CSPHJscEiuudb+1SG2uA/oik/WQ== username@domainname
```

Si vous copiez et collez le contenu du fichier de clé publique pour l’utiliser dans le portail Azure ou dans un modèle Resource Manager, veillez à ne pas copier pas les espaces blancs de fin. Pour copier une clé publique dans macOS, vous pouvez diriger le fichier de clé publique vers `pbcopy`. De même, sous Linux, vous pouvez diriger le fichier de clé publique vers des programmes comme `xclip`.

La clé publique que vous placez sur votre machine virtuelle Linux dans Azure est stockée par défaut dans ~/.ssh/id_rsa.pub, sauf si vous avez spécifié un autre emplacement quand vous avez créé la paire de clés. Pour utiliser [Azure CLI 2.0](/cli/azure) pour créer votre machine virtuelle avec une clé publique existante, spécifiez la valeur et éventuellement l’emplacement de cette clé publique à l’aide de la commande [az vm create](/cli/azure/vm#az-vm-create) avec l’option `--ssh-key-values`. Dans la commande suivante, remplacez *myVM*, *myResourceGroup*, *UbuntuLTS*, *azureuser* et *mysshkey.pub* par votre propres valeurs :


```azurecli
az vm create \
  --resource-group myResourceGroup \
  --name myVM \
  --image UbuntuLTS \
  --admin-username azureuser \
  --ssh-key-values mysshkey.pub
```

Pour utiliser plusieurs clés SSH avec votre machine virtuelle, entrez-les dans une liste séparée par des espaces, comme ceci `--ssh-key-values sshkey-desktop.pub sshkey-laptop.pub`.


## <a name="ssh-into-your-vm"></a>Se connecter avec SSH à votre machine virtuelle

Une fois la clé publique déployée sur votre machine virtuelle Azure et la clé privée sur votre système local, connectez-vous avec SSH à votre machine virtuelle à l’aide de l’adresse IP ou du nom DNS de votre machine virtuelle. Dans la commande suivante, remplacez *azureuser* et *myvm.westus.cloudapp.azure.com* par le nom d’utilisateur de l’administrateur et le nom de domaine complet (ou l’adresse IP) :

```bash
ssh azureuser@myvm.westus.cloudapp.azure.com
```

Si vous avez spécifié une phrase secrète quand vous avez créé votre paire de clés, entrez-la quand vous y êtes invité pendant le processus de connexion. La machine virtuelle est ajoutée à votre fichier ~/.ssh/known_hosts et vous n’avez pas à vous connecter à nouveau tant que la clé publique sur votre machine virtuelle Azure n’est pas modifiée ou que le nom de serveur n’est pas supprimé du fichier ~/.ssh/known_hosts.

Si la machine virtuelle utilise la stratégie juste-à-temps, vous devez demander à y accéder avant de pouvoir vous connecter à cette machine virtuelle. Pour plus d’informations sur la stratégie juste-à-temps, consultez [Gérer l’accès à la machine virtuelle à l’aide de la stratégie juste-à-temps](../../security-center/security-center-just-in-time.md).

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation des paires de clés SSH, consultez [Étapes détaillées pour créer et gérer des paires de clés SSH](create-ssh-keys-detailed.md).

* Si vous rencontrez des difficultés avec les connexions SSH à des machines virtuelles Azure, consultez [Résoudre les problèmes de connexion SSH à une machine virtuelle Linux Azure](../troubleshooting/troubleshoot-ssh-connection.md).

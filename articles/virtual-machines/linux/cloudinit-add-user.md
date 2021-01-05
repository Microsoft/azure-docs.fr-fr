---
title: Utiliser cloud-init pour ajouter un utilisateur à une machine virtuelle Linux sur Azure
description: Guide pratique pour utiliser cloud-init pour ajouter un utilisateur sur une machine virtuelle Linux lors de la création avec l’interface Azure CLI
author: rickstercdn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 569ceb4c7158ba9dc08c99c234951fb4507b69f6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87370068"
---
# <a name="use-cloud-init-to-add-a-user-to-a-linux-vm-in-azure"></a>Utiliser cloud-init pour ajouter un utilisateur sur une machine virtuelle Linux sur Azure
Cet article montre comment utiliser [cloud-init](https://cloudinit.readthedocs.io) pour ajouter un utilisateur sur une machine virtuelle Linux ou un groupe de machines virtuelles identiques au moment de l’approvisionnement dans Azure. Ce script cloud-init s’exécute au premier démarrage une fois que les ressources ont été approvisionnées par Azure. Pour plus d’informations sur le fonctionnement de cloud-init en mode natif dans Azure et sur les versions de Linux prises en charge, consultez [Présentation de cloud-init](using-cloud-init.md).

## <a name="add-a-user-to-a-vm-with-cloud-init"></a>Ajouter un utilisateur à une machine virtuelle avec cloud-init
L’une des premières tâches à effectuer sur une nouvelle machine virtuelle Linux consiste à ajouter un utilisateur supplémentaire pour vous-même afin d’éviter l’utilisation de *root*. Les clés SSH sont un bon réflexe de sécurité et de facilité d’utilisation. Les clés sont ajoutées au fichier *~/.ssh/authorized_keys* avec ce script cloud-init.

Pour ajouter un utilisateur à une machine virtuelle Linux, créez dans l’interpréteur de commandes actuel un fichier nommé *cloud_init_add_user.txt* et collez-y la configuration suivante. Pour cet exemple, créez le fichier dans Cloud Shell et non sur votre ordinateur local. Vous pouvez utiliser l’éditeur de votre choix. Entrez `sensible-editor cloud_init_add_user.txt` pour créer le fichier et afficher la liste des éditeurs disponibles. Choisissez le n°1 pour utiliser l’éditeur **nano**. Vérifiez que l’intégralité du fichier cloud-init est copiée, en particulier la première ligne.  Vous devez fournir votre propre clé publique (comme le contenu de *~/.ssh/id_rsa.pub*) pour la valeur de `ssh-authorized-keys:`, elle a été raccourcie ici pour simplifier l’exemple.

```yaml
#cloud-config
users:
  - default
  - name: myadminuser
    groups: sudo
    shell: /bin/bash
    sudo: ['ALL=(ALL) NOPASSWD:ALL']
    ssh-authorized-keys:
      - ssh-rsa AAAAB3<snip>
```
> [!NOTE] 
> Le fichier #cloud-config inclut le paramètre `- default`. Cela permet d’ajouter l’utilisateur à l’utilisateur administrateur existant créé lors de l’approvisionnement. Si vous créez un utilisateur sans le paramètre `- default`, l’utilisateur administrateur généré automatiquement créé par la plateforme Azure serait remplacé. 

Avant de déployer cette image, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Maintenant, créez une machine virtuelle avec [az vm create](/cli/azure/vm) et spécifiez le fichier cloud-init avec `--custom-data cloud_init_add_user.txt` comme suit :

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_add_user.txt \
  --generate-ssh-keys 
```

Établissez une connexion SSH à l’adresse IP publique de votre machine virtuelle indiquée dans la sortie de la commande précédente. Entrez votre propre **publicIpAddress** comme suit :

```bash
ssh <publicIpAddress>
```

Pour confirmer que l’utilisateur a été ajouté à la machine virtuelle et aux groupes spécifiés, affichez le contenu du fichier */etc/group* comme suit :

```bash
cat /etc/group
```

L’exemple de sortie suivant montre que l’utilisateur indiqué dans le fichier *cloud_init_add_user.txt* a été ajouté à la machine virtuelle et au groupe approprié :

```bash
root:x:0:
<snip />
sudo:x:27:myadminuser
<snip />
myadminuser:x:1000:
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des exemples cloud-init supplémentaires de modifications de configuration, consultez les rubriques suivantes :
 
- [Ajouter un utilisateur Linux supplémentaire à une machine virtuelle](cloudinit-add-user.md)
- [Exécuter un gestionnaire de package pour mettre à jour les packages existants au premier démarrage](cloudinit-update-vm.md)
- [Use cloud-init to set hostname for a Linux VM in Azure](cloudinit-update-vm-hostname.md) (Utiliser cloud-init pour définir un nom d’hôte pour une machine virtuelle Linux dans Azure) 
- [Installer un package d’application, mettre à jour des fichiers de configuration et injecter des clés](tutorial-automate-vm-deployment.md)

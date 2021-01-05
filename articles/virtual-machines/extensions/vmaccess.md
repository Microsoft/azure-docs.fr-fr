---
title: Réinitialiser l’accès à une machine virtuelle Linux Azure
description: Comment gérer les utilisateurs administratifs et réinitialiser l’accès sur des machines virtuelles Linux à l’aide de l’extension VMAccess et d’Azure CLI
services: virtual-machines-linux
documentationcenter: ''
author: axayjo
manager: gwallace
editor: ''
tags: azure-resource-manager
ms.assetid: 261a9646-1f93-407e-951e-0be7226b3064
ms.service: virtual-machines-linux
ms.subservice: extensions
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: article
ms.date: 05/10/2018
ms.author: akjosh
ms.openlocfilehash: e46f7f3d51d2841a2287c86f9e8dddd7460b1a5d
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016435"
---
# <a name="manage-administrative-users-ssh-and-check-or-repair-disks-on-linux-vms-using-the-vmaccess-extension-with-the-azure-cli"></a>Gérer les utilisateurs administratifs, SSH, et vérifier ou réparer les disques de machines virtuelles Linux à l’aide de l’extension VMAccess avec Azure CLI
## <a name="overview"></a>Vue d’ensemble
Le disque de votre machine virtuelle Linux affiche des erreurs. Vous avez d'une certaine manière réinitialisé le mot de passe racine de votre machine virtuelle Linux ou supprimé accidentellement votre clé privée SSH. Dans les anciens centres de données, vous deviez aller sur place et ouvrir le KVM pour accéder à la console du serveur. Considérez l’extension Azure VMAccess comme ce commutateur KVM qui vous permet d’accéder à la console pour réinitialiser l’accès à Linux ou effectuer la maintenance au niveau du disque.

Cet article vous explique comment utiliser l’extension Azure VMAccess pour vérifier ou réparer un disque, réinitialiser l’accès des utilisateurs administratifs, gérer les comptes d’utilisateur ou mettre à jour la configuration SSH sous Linux lors de leur exécution en tant que machines virtuelles Azure Resource Manager. Si vous avez besoin de gérer des machines virtuelles classiques, vous pouvez suivre les instructions figurant dans la [Documentation de la machine virtuelle classique](/previous-versions/azure/virtual-machines/linux/classic/reset-access-classic). 
 
> [!NOTE]
> Si vous utilisez l’extension VMAccess pour réinitialiser le mot de passe de votre machine virtuelle après l’installation de l’extension de connexion AAD, vous devez réexécuter l’extension de connexion AAD pour réactiver la connexion AAD sur votre machine.

## <a name="prerequisites"></a>Conditions préalables requises
### <a name="operating-system"></a>Système d’exploitation

L’extension d’accès aux machines virtuelles peut être exécutée sur ces distributions de Linux :

| Distribution | Version |
|---|---|
| Ubuntu | 16.04 LTS, 14.04 LTS et 12.04 LTS |
| Debian | Debian 7.9+, 8.2+ |
| Red Hat | RHEL 6.7+, 7.1+ |
| Oracle Linux | 6.4+, 7.0+ |
| SUSE | 11 et 12 |
| openSUSE | openSUSE Leap 42.2+ |
| CentOS | CentOS 6.3+, 7.0+ |
| CoreOS | 494.4.0+ |

## <a name="ways-to-use-the-vmaccess-extension"></a>Méthodes d’utilisation de l’extension VMAccess
Il existe deux façons d’utiliser l’extension VMAccess sur vos machines virtuelles Linux :

* Utiliser Azure CLI et les paramètres nécessaires.
* [À l’aide de fichiers JSON bruts que l’extension VMAccess va traiter](#use-json-files-and-the-vmaccess-extension) et exploiter.

Les exemples suivants utilisent des commandes [az vm user](/cli/azure/vm/user). Pour suivre ces étapes, vous avez besoin de la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et devez vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

## <a name="update-ssh-key"></a>Mettre à jour la clé SSH
L’exemple suivant met à jour la clé SSH pour l’utilisateur `azureuser` sur la machine virtuelle `myVM` :

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

> **REMARQUE  :** la commande `az vm user update` ajoute le texte de la nouvelle clé publique au fichier `~/.ssh/authorized_keys` pour l’utilisateur administrateur sur la machine virtuelle. Cette opération ne remplace pas ou ne supprime pas les clés SSH existantes. Cela ne supprime pas les clés préalables définies au moment du déploiement ou de mises à jour ultérieures via l’extension VMAccess.

## <a name="reset-password"></a>Réinitialiser le mot de passe
L’exemple suivant réinitialise le mot de passe pour l’utilisateur `azureuser` sur la machine virtuelle `myVM` :

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username azureuser \
  --password myNewPassword
```

## <a name="restart-ssh"></a>Redémarrer SSH
L’exemple suivant redémarre le démon SSH et réinitialise la configuration SSH à ses valeurs par défaut sur une machine virtuelle nommée `myVM` :

```azurecli-interactive
az vm user reset-ssh \
  --resource-group myResourceGroup \
  --name myVM
```

## <a name="create-an-administrativesudo-user"></a>Créer un utilisateur administratif/sudo
L’exemple suivant crée un utilisateur nommé `myNewUser` avec des autorisations **sudo**. Le compte utilise une clé SSH pour l’authentification sur la machine virtuelle nommée `myVM`. Cette méthode est conçue pour vous aider à réaccéder à une machine virtuelle en cas de perte ou d’oubli des informations d’identification actuelles. En guise de bonne pratique, vous devez limiter les comptes avec des autorisations **sudo**.

```azurecli-interactive
az vm user update \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser \
  --ssh-key-value ~/.ssh/id_rsa.pub
```

## <a name="delete-a-user"></a>Supprimer un utilisateur
L’exemple suivant supprime un utilisateur nommé `myNewUser` sur la machine virtuelle `myVM` :

```azurecli-interactive
az vm user delete \
  --resource-group myResourceGroup \
  --name myVM \
  --username myNewUser
```

## <a name="use-json-files-and-the-vmaccess-extension"></a>Utilisation de fichiers JSON et de l’extension VMAccess
Les exemples suivants utilisent des fichiers JSON bruts. Utilisez [az vm extension set](/cli/azure/vm/extension), puis appelez vos fichiers JSON. Ces fichiers JSON peuvent également être appelés à partir de modèles Azure. 

### <a name="reset-user-access"></a>Réinitialisation de l’accès utilisateur
Si vous ne pouvez plus accéder à la racine de votre machine virtuelle Linux, vous pouvez lancer un script VMAccess pour mettre à jour la clé SSH ou le mot de passe d’un utilisateur.

Pour mettre à jour la clé SSH d’un utilisateur, créez un fichier nommé `update_ssh_key.json` et ajoutez des paramètres au format suivant. Remplacez par vos propres valeurs les paramètres `username` et `ssh_key` :

```json
{
  "username":"azureuser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNGxxxxxx2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7xxxxxxwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5wxxtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== azureuser@myVM"
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings update_ssh_key.json
```

Pour réinitialiser le mot de passe d’un utilisateur, créez un fichier nommé `reset_user_password.json` et ajoutez des paramètres au format suivant. Remplacez par vos propres valeurs les paramètres `username` et `password` :

```json
{
  "username":"azureuser",
  "password":"myNewPassword" 
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_user_password.json
```

### <a name="restart-ssh"></a>Redémarrer SSH
Pour redémarrer le démon SSH et réinitialiser la configuration SSH à ses valeurs par défaut, créez un fichier nommé `reset_sshd.json`. Ajoutez le contenu suivant :

```json
{
  "reset_ssh": true
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings reset_sshd.json
```

### <a name="manage-administrative-users"></a>Gérer les utilisateurs administratifs

Pour créer un utilisateur avec des autorisations **sudo** qui utilise une clé SSH au moment de l’authentification, créez un fichier nommé `create_new_user.json` et ajoutez des paramètres au format suivant. Remplacez par vos propres valeurs les paramètres `username` et `ssh_key`. Cette méthode est conçue pour vous aider à réaccéder à une machine virtuelle en cas de perte ou d’oubli des informations d’identification actuelles. En guise de bonne pratique, vous devez limiter les comptes avec des autorisations **sudo**.

```json
{
  "username":"myNewUser",
  "ssh_key":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAACAQCZ3S7gGp3rcbKmG2Y4vGZFMuMZCwoUzZNG1vHY7P2XV2x9FfAhy8iGD+lF8UdjFX3t5ebMm6BnnMh8fHwkTRdOt3LDQq8o8ElTBrZaKPxZN2thMZnODs5Hlemb2UX0oRIGRcvWqsd4oJmxsXa/Si98Wa6RHWbc9QZhw80KAcOVhmndZAZAGR+Wq6yslNo5TMOr1/ZyQAook5C4FtcSGn3Y+WczaoGWIxG4ZaWk128g79VIeJcIQqOjPodHvQAhll7qDlItVvBfMOben3GyhYTm7k4YwlEdkONm4yV/UIW0la1rmyztSBQIm9sZmSq44XXgjVmDHNF8UfCZ1ToE4r2SdwTmZv00T2i5faeYnHzxiLPA3Enub7iUo5IdwFArnqad7MO1SY1kLemhX9eFjLWN4mJe56Fu4NiWJkR9APSZQrYeKaqru4KUC68QpVasNJHbuxPSf/PcjF3cjO1+X+4x6L1H5HTPuqUkyZGgDO4ynUHbko4dhlanALcriF7tIfQR9i2r2xOyv5gxJEW/zztGqWma/d4rBoPjnf6tO7rLFHXMt/DVTkAfn5woYtLDwkn5FMyvThRmex3BDf0gujoI1y6cOWLe9Y5geNX0oj+MXg/W0cXAtzSFocstV1PoVqy883hNoeQZ3mIGB3Q0rIUm5d9MA2bMMt31m1g3Sin6EQ== myNewUser@myVM",
  "password":"myNewUserPassword"
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings create_new_user.json
```

Pour supprimer un utilisateur, créez un fichier nommé `delete_user.json` avec le contenu suivant. Remplacez par vos propres valeurs le paramètre `remove_user` :

```json
{
  "remove_user":"myNewUser"
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings delete_user.json
```

### <a name="check-or-repair-the-disk"></a>Vérification ou réparation du disque
L’utilisation de VMAccess vous permet également de vérifier et de réparer un disque que vous avez ajouté à la machine virtuelle Linux.

Pour vérifier puis réparer le disque, créez un fichier nommé `disk_check_repair.json` et ajoutez des paramètres au format suivant. Remplacez par vos propres valeurs le nom de `repair_disk` :

```json
{
  "check_disk": "true",
  "repair_disk": "true, mydiskname"
}
```

Exécutez le script VMAccess avec :

```azurecli-interactive
az vm extension set \
  --resource-group myResourceGroup \
  --vm-name myVM \
  --name VMAccessForLinux \
  --publisher Microsoft.OSTCExtensions \
  --version 1.4 \
  --protected-settings disk_check_repair.json
```
## <a name="troubleshoot-and-support"></a>Dépannage et support technique

### <a name="troubleshoot"></a>Dépanner

Vous pouvez récupérer les données sur l’état des déploiements d’extension à partir du portail Azure et à l’aide de l’interface de ligne de commande Azure. Pour afficher l’état du déploiement des extensions pour une machine virtuelle donnée, exécutez la commande suivante à l’aide de l’interface de ligne de commande Azure.

```azurecli
az vm extension list --resource-group myResourceGroup --vm-name myVM -o table
```

### <a name="support"></a>Support

Si vous avez besoin d’une aide supplémentaire à quelque étape que ce soit dans cet article, vous pouvez contacter les experts Azure sur les [forums MSDN Azure et Stack Overflow](https://azure.microsoft.com/support/forums/). Vous pouvez également signaler un incident au support Azure. Accédez au [site du support Azure](https://azure.microsoft.com/support/options/) , puis cliquez sur Obtenir un support. Pour plus d’informations sur l’utilisation du support Azure, lisez le [FAQ du support Microsoft Azure](https://azure.microsoft.com/support/faq/).

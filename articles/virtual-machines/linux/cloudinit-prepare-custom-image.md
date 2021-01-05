---
title: Préparer une image de machine virtuelle Azure pour une utilisation avec cloud-init
description: Comment préparer une image de machine virtuelle Azure existante pour le déploiement avec cloud-init
author: danis
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.date: 06/24/2019
ms.author: danis
ms.openlocfilehash: 6785bb1f5ebde4f746dc8aa1ea9f586f406f8431
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87373672"
---
# <a name="prepare-an-existing-linux-azure-vm-image-for-use-with-cloud-init"></a>Préparer une image de machine virtuelle Azure Linux existante pour une utilisation avec cloud-init
Cet article explique comment prendre une machine virtuelle Azure existante et la préparer pour la redéployer et utiliser cloud-init. L’image obtenue peut être utilisée pour déployer une nouvelle machine virtuelle ou des groupes de machines virtuelles identiques, qui peuvent par la suite être personnalisés par cloud-init au moment du déploiement.  Ces scripts cloud-init s’exécutent au premier démarrage une fois que les ressources ont été approvisionnées par Azure. Pour plus d’informations sur le fonctionnement de cloud-init en mode natif dans Azure et sur les versions de Linux prises en charge, consultez [Présentation de cloud-init](using-cloud-init.md)

## <a name="prerequisites"></a>Prérequis
Ce document suppose que vous disposez déjà d’une machine virtuelle Azure exécutant une version prise en charge du système d’exploitation Linux. Vous avez déjà configuré la machine en fonction de vos besoins, installé tous les modules nécessaires, traité toutes les mises à jour requises et effectué des tests pour vous assurer qu’elle répond à vos exigences. 

## <a name="preparing-rhel-76--centos-76"></a>Préparation RHEL 7.6/CentOS 7.6
Vous avez besoin d’appliquer le protocole SSH à votre machine virtuelle Linux et d’exécuter les commandes suivantes pour installer cloud-init.

```bash
sudo yum makecache fast
sudo yum install -y gdisk cloud-utils-growpart
sudo yum install - y cloud-init 
```

Mettez à jour la section `cloud_init_modules` dans `/etc/cloud/cloud.cfg` pour inclure les modules suivants :

```bash
- disk_setup
- mounts
```

Voici un exemple de ce à quoi une section `cloud_init_modules` classique ressemble.

```bash
cloud_init_modules:
 - migrator
 - bootcmd
 - write-files
 - growpart
 - resizefs
 - disk_setup
 - mounts
 - set_hostname
 - update_hostname
 - update_etc_hosts
 - rsyslog
 - users-groups
 - ssh
```

Un certain nombre de tâches relatives à l’approvisionnement et à la gestion des disques éphémères doit être mis à jour dans `/etc/waagent.conf`. Exécutez les commandes suivantes pour mettre à jour les paramètres appropriés.

```bash
sed -i 's/Provisioning.Enabled=y/Provisioning.Enabled=n/g' /etc/waagent.conf
sed -i 's/Provisioning.UseCloudInit=n/Provisioning.UseCloudInit=y/g' /etc/waagent.conf
sed -i 's/ResourceDisk.Format=y/ResourceDisk.Format=n/g' /etc/waagent.conf
sed -i 's/ResourceDisk.EnableSwap=y/ResourceDisk.EnableSwap=n/g' /etc/waagent.conf
cloud-init clean
```

Autorisez uniquement Azure comme source de données pour l’agent Linux Azure en créant un fichier `/etc/cloud/cloud.cfg.d/91-azure_datasource.cfg` à l’aide de l’éditeur de votre choix et de la ligne suivante :

```bash
# Azure Data Source config
datasource_list: [ Azure ]
```

Si votre image Azure existante dispose d’un fichier d’échange configuré et que vous souhaitez modifier la configuration du fichier d’échange pour de nouvelles images à l’aide de cloud-init, vous devez supprimer le fichier d’échange existant.

Pour les images basées sur Red Hat : suivez les instructions dans le document RedHat suivant qui explique comment [supprimer le fichier d’échange](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/6/html/storage_administration_guide/swap-removing-file).

Pour les images CentOS avec le fichier d’échange activé, vous pouvez exécuter la commande suivante pour désactiver le fichier d’échange :

```bash
sudo swapoff /mnt/resource/swapfile
```

Vérifiez que la référence de fichier d’échange est supprimée de `/etc/fstab` - cela doit ressembler à la sortie suivante :

```output
# /etc/fstab
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
UUID=99cf66df-2fef-4aad-b226-382883643a1c / xfs defaults 0 0
UUID=7c473048-a4e7-4908-bad3-a9be22e9d37d /boot xfs defaults 0 0
```

Pour gagner de l’espace et supprimer le fichier d’échange, vous pouvez exécuter la commande suivante :

```bash
rm /mnt/resource/swapfile
```

## <a name="extra-step-for-cloud-init-prepared-image"></a>Étape supplémentaire pour l’image préparée pour cloud-init
> [!NOTE]
> Si votre image était auparavant une image configurée et préparée pour **cloud-init**, vous devez effectuer les étapes suivantes.

Les trois commandes suivantes sont uniquement utilisées si la machine virtuelle que vous personnalisez pour être une nouvelle image source spécialisée a été précédemment approvisionnée par cloud-init.  Vous n’avez PAS besoin d’exécuter ces commandes si l’image a été configurée à l’aide de l’agent Linux Azure.

```bash
sudo cloud-init clean --logs
sudo waagent -deprovision+user -force
```

## <a name="finalizing-linux-agent-setting"></a>Finalisation du paramétrage de l’agent Linux 
Toutes les images de plateforme Azure ont l’agent Linux Azure installé, qu’il ait été configuré par cloud-init ou non.  Exécutez la commande suivante pour terminer le déprovisionnement de l’utilisateur à partir de la machine Linux. 

```bash
sudo waagent -deprovision+user -force
```

Pour plus d’informations sur les commandes de déprovisionnement de l’agent Linux Azure, consultez [Présentation et utilisation de l’agent Linux Azure](../extensions/agent-linux.md) pour plus d’informations.

Quittez la session SSH, puis à partir de votre interpréteur de commandes bash, exécutez les commandes Azure CLI suivantes pour désallouer, généraliser et créer une image de machine virtuelle Azure.  Remplacez `myResourceGroup` et `sourceVmName` par les informations appropriées reflétant votre machine virtuelle source.

```azurecli
az vm deallocate --resource-group myResourceGroup --name sourceVmName
az vm generalize --resource-group myResourceGroup --name sourceVmName
az image create --resource-group myResourceGroup --name myCloudInitImage --source sourceVmName
```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des exemples cloud-init supplémentaires de modifications de configuration, consultez les rubriques suivantes :
 
- [Ajouter un utilisateur Linux supplémentaire à une machine virtuelle](cloudinit-add-user.md)
- [Exécuter un gestionnaire de package pour mettre à jour les packages existants au premier démarrage](cloudinit-update-vm.md)
- [Use cloud-init to set hostname for a Linux VM in Azure](cloudinit-update-vm-hostname.md) (Utiliser cloud-init pour définir un nom d’hôte pour une machine virtuelle Linux dans Azure) 
- [Installer un package d’application, mettre à jour des fichiers de configuration et injecter des clés](tutorial-automate-vm-deployment.md)

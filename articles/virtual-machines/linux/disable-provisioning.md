---
title: Désactiver ou supprimer l’agent d’approvisionnement
description: Découvrez comment désactiver ou supprimer l’agent d’approvisionnement dans des images et des machines virtuelles Linux.
author: danielsollondon
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 07/06/2020
ms.author: danis
ms.reviewer: cynthn
ms.openlocfilehash: 2a17825d062496e6600966dc7c90b14749507e4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86494511"
---
# <a name="disable-or-remove-the-linux-agent-from-vms-and-images"></a>Désactiver ou supprimer l’agent Linux des machines virtuelles et des images

Avant de supprimer l’agent Linux, vous devez comprendre ce que la machine virtuelle ne sera pas en mesure de faire après la suppression de l’agent Linux.

Les [extensions](../extensions/overview.md) de machine virtuelle Azure sont de petites applications permettant d’exécuter des tâches de configuration et d’automatisation post-déploiement sur des machines virtuelles Azure. Les extensions sont installées et managées par le plan de contrôle Azure. Il s’agit de la tâche de l’[Agent Linux Azure](../extensions/agent-linux.md) pour traiter les commandes d’extension de plateforme et garantir l’état correct de l’extension dans la machine virtuelle.

La plateforme Azure héberge de nombreuses extensions, notamment des applications de monitoring, de sécurité, d’utilitaire et de configuration de machine virtuelle. Il existe un choix important d’extensions internes et tierces ainsi que des exemples de scénarios clés utilisés pour les extensions :
* Prise en charge des services Azure internes, tels qu’Azure Backup, Monitoring, Disk Encryption, Security, Site Replication, etc.
* Réinitialisations SSH/mots de passe
* Configuration de machine virtuelle : exécution de scripts personnalisés, installation de Chef, agents Puppet, etc.
* Produits tiers, tels que les produits antivirus, les outils de vulnérabilité, les machines virtuelles et les outils d’analyse des applications.
* Les extensions peuvent être englobées dans un nouveau déploiement de machine virtuelle. Par exemple, elles peuvent faire partie d’un déploiement plus vaste en configurant des applications lors du provisionnement de machines virtuelles, ou être exécutées en post-déploiement sur des systèmes d’exploitation prenant en charge ces extensions.

## <a name="disabling-extension-processing"></a>Désactivation du traitement des extensions

Il existe plusieurs façons de désactiver le traitement des extensions, en fonction de vos besoins, mais avant de continuer, vous **devez** supprimer toutes les extensions déployées sur la machine virtuelle, par exemple à l’aide de l’interface AZ CLI, vous pouvez [lister](/cli/azure/vm/extension?view=azure-cli-latest#az-vm-extension-list) et [supprimer](/cli/azure/vm/extension?view=azure-cli-latest#az-vm-extension-delete) :

```bash
az vm extension delete -g MyResourceGroup --vm-name MyVm -n extension_name
```
> [!Note]
> 
> Si vous n’effectuez pas cette opération, la plateforme essaiera d’envoyer la configuration et le délai d’expiration de l’extension après 40 min.

### <a name="disable-at-the-control-plane"></a>Désactiver sur le plan de contrôle
Si vous ne savez pas si vous avez besoin d’extensions à l’avenir, vous pouvez conserver l’agent Linux installé sur la machine virtuelle, puis désactiver la fonctionnalité de traitement d’extension à partir de la plateforme. Cette option est disponible dans la `Microsoft.Compute` version api`2018-06-01` ou version ultérieure, et n’a pas de dépendance sur la version de l’agent Linux installée.

```bash
az vm update -g <resourceGroup> -n <vmName> --set osProfile.allowExtensionOperations=false
```
Vous pouvez facilement réactiver ce traitement d’extension à partir de la plateforme, avec la commande ci-dessus, en lui affectant la valeur « true ».

## <a name="remove-the-linux-agent-from-a-running-vm"></a>Supprimer l’agent Linux d’une machine virtuelle en cours d’exécution

Vérifiez que vous avez **supprimé** toutes les extensions existantes de la machine virtuelle, comme indiqué ci-dessus.

### <a name="step-1-remove-the-azure-linux-agent"></a>Étape 1 : Supprimer l’agent Linux Azure

Si vous supprimez simplement l’agent Linux, et non les artefacts de configuration associés, vous pouvez le réinstaller à une date ultérieure. Exécutez l’un des éléments suivants, en tant que racine, pour supprimer l’agent Linux Azure :

#### <a name="for-ubuntu-1804"></a>Pour Ubuntu >=18.04
```bash
apt -y remove walinuxagent
```

#### <a name="for-redhat--77"></a>Pour Redhat >= 7.7
```bash
yum -y remove WALinuxAgent
```

#### <a name="for-suse"></a>Pour SUSE
```bash
zypper --non-interactive remove python-azure-agent
```

### <a name="step-2-optional-remove-the-azure-linux-agent-artifacts"></a>Étape 2 : (Facultatif) Supprimer les artefacts de l’agent Linux Azure
> [!IMPORTANT] 
>
> Vous pouvez supprimer tous les artefacts associés de l’agent Linux, mais cela signifie que vous ne pourrez pas le réinstaller à une date ultérieure. Par conséquent, il est vivement recommandé de désactiver d’abord l’agent Linux, en le supprimant à l’aide de la méthode ci-dessus uniquement. 

Si vous savez que vous ne réinstallez jamais l’agent Linux, vous pouvez exécuter la commande suivante :

#### <a name="for-ubuntu-1804"></a>Pour Ubuntu >=18.04
```bash
apt -y purge walinuxagent
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-redhat--77"></a>Pour Redhat >= 7.7
```bash
yum -y remove WALinuxAgent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

#### <a name="for-suse"></a>Pour SUSE
```bash
zypper --non-interactive remove python-azure-agent
rm -f /etc/waagent.conf.rpmsave
rm -rf /var/lib/waagent
rm -f /var/log/waagent.log
```

## <a name="preparing-an-image-without-the-linux-agent"></a>Préparation d’une image sans l’agent Linux
Si vous disposez d’une image qui contient déjà cloud-init et que vous souhaitez supprimer l’agent Linux, mais que vous désirez continuer à utiliser cloud-init, exécutez l’étape 2 (et éventuellement l’étape 3) en tant que racine pour supprimer l’agent Azure Linux, puis supprimez la configuration cloud-init et les données mises en cache, puis préparez la machine virtuelle pour créer une image personnalisée.

```bash
cloud-init clean --logs --seed 
```

## <a name="deprovision-and-create-an-image"></a>Annuler l’approvisionnement et créer une image
L’agent Linux a la possibilité de nettoyer certaines métadonnées d’image existantes, avec l’étape « waagent-deprovision + user », mais une fois qu’il a été supprimé, vous devrez effectuer des actions comme indiqué ci-dessous et supprimer toutes les autres données sensibles qu’il contient.

- Supprimer toutes les clés d’hôte SSH existantes

   ```bash
   rm /etc/ssh/ssh_host_*key*
   ```
- Supprimer le compte administrateur

   ```bash
   touch /var/run/utmp
   userdel -f -r <admin_user_account>
   ```
- Supprimer le mot de passe racine

   ```bash
   passwd -d root
   ```

Une fois que vous avez effectué les étapes ci-dessus, vous pouvez créer l’image personnalisée à l’aide de l’interface CLI Azure.


**Créer une image managée normale**
```bash
az vm deallocate -g <resource_group> -n <vm_name>
az vm generalize -g <resource_group> -n <vm_name>
az image create -g <resource_group> -n <image_name> --source <vm_name>
```

**Créer une version d’image dans une galerie d’images partagées**

```bash
az sig image-version create \
    -g $sigResourceGroup 
    --gallery-name $sigName 
    --gallery-image-definition $imageDefName 
    --gallery-image-version 1.0.0 
    --managed-image /subscriptions/00000000-0000-0000-0000-00000000xxxx/resourceGroups/imageGroups/providers/images/MyManagedImage
```
### <a name="creating-a-vm-from-an-image-that-does-not-contain-a-linux-agent"></a>Création d’une machine virtuelle à partir d’une image qui ne contient pas d’agent Linux
Lorsque vous créez la machine virtuelle à partir de l’image sans agent Linux, vous devez vous assurer que la configuration de déploiement de la machine virtuelle indique que les extensions ne sont pas prises en charge sur cette machine virtuelle.

> [!NOTE] 
> 
> Si vous n’effectuez pas cette opération, la plateforme essaiera d’envoyer la configuration et le délai d’expiration de l’extension après 40 min.

Pour déployer la machine virtuelle avec des extensions désactivées, vous pouvez utiliser l’interface CLI Azure avec [--enable-agent](/cli/azure/vm#az-vm-create).

```bash
az vm create \
    --resource-group $resourceGroup \
    --name $prodVmName \
    --image RedHat:RHEL:8.1-ci:latest \
    --admin-username azadmin \
    --ssh-key-value "$sshPubkeyPath" \
    --enable-agent false
```

Vous pouvez également effectuer cette opération à l’aide de modèles Azure Resource Manager (ARM), en définissant `"provisionVMAgent": false,`.

```json
"osProfile": {
    "computerName": "[parameters('virtualMachineName')]",
    "adminUsername": "[parameters('adminUsername')]",
    "linuxConfiguration": {
        "disablePasswordAuthentication": "true",
        "provisionVMAgent": false,
        "ssh": {
            "publicKeys": [
                {
                    "path": "[concat('/home/', parameters('adminUsername'), '/.ssh/authorized_keys')]",
                    "keyData": "[parameters('adminPublicKey')]"
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez [Provisionnement Linux](provisioning.md).

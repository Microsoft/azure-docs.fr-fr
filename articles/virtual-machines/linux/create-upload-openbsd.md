---
title: Créer et charger une image OpenBSD
description: Découvrir comment créer et télécharger un disque dur virtuel (VHD) contenant le système d’exploitation OpenBSD pour créer une machine virtuelle Azure via Azure CLI
author: gbowerman
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 05/24/2017
ms.author: guybo
ms.openlocfilehash: 08b18dae6cec3f30ba9ecc69a3537eec428cc9ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87372720"
---
# <a name="create-and-upload-an-openbsd-disk-image-to-azure"></a>Créer et charger une image de disque OpenBSD sur Azure
Cet article montre comment créer et charger un disque dur virtuel (VHD) contenant le système d’exploitation OpenBSD. Après l’avoir chargé, vous pouvez l’utiliser comme image personnelle pour créer une machine virtuelle (VM) dans Azure via Azure CLI.


## <a name="prerequisites"></a>Conditions préalables requises
Cet article part du principe que vous disposez des éléments suivants :

* **Abonnement Azure** : si vous ne possédez pas de compte, vous pouvez en créer un en quelques minutes. Si vous disposez d’un abonnement MSDN, consultez [Crédit Azure mensuel pour les abonnés Visual Studio](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/). Dans le cas contraire, découvrez comment [créer un compte d'essai gratuit](https://azure.microsoft.com/pricing/free-trial/).  
* **Azure CLI** : Vérifiez que vous avez installé la dernière version [d’Azure CLI](/cli/azure/install-azure-cli) et que vous êtes connecté à votre compte Azure avec [az login](/cli/azure/reference-index).
* **Système d’exploitation OpenBSD installé dans un fichier .vhd** : un système d’exploitation OpenBSD pris en charge ([version 6.6 AMD64](https://ftp.openbsd.org/pub/OpenBSD/6.6/amd64/)) doit être installé sur un disque dur virtuel. Plusieurs outils permettent de créer des fichiers .vhd. Par exemple, vous pouvez utiliser une solution de virtualisation comme Hyper-V pour créer le fichier .vhd et installer le système d'exploitation. Pour obtenir des instructions pour installer et utiliser Hyper-V, consultez la page [Installer Hyper-V et créer une machine virtuelle](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).


## <a name="prepare-openbsd-image-for-azure"></a>Préparer l’image OpenBSD pour Azure
Sur la machine virtuelle sur laquelle vous avez installé le système d’exploitation OpenBSD 6.1, qui a ajouté la prise en charge de Hyper-V, procédez comme suit :

1. Si le protocole DHCP n’est pas activé lors de l’installation, activez le service comme suit :

    ```sh    
    echo dhcp > /etc/hostname.hvn0
    ```

2. Configurez une console série comme suit :

    ```sh
    echo "stty com0 115200" >> /etc/boot.conf
    echo "set tty com0" >> /etc/boot.conf
    ```

3. Configurez l’installation du package comme suit :

    ```sh
    echo "https://ftp.openbsd.org/pub/OpenBSD" > /etc/installurl
    ```
   
4. Par défaut, l’utilisateur `root` est désactivé sur les machines virtuelles dans Microsoft Azure. Les utilisateurs peuvent exécuter des commandes avec des privilèges élevés à l’aide de la commande `doas` sur une machine virtuelle OpenBSD. Doas est activé par défaut. Pour plus d’informations, voir [doas.conf](https://man.openbsd.org/doas.conf.5). 

5. Installez et configurez les éléments requis pour l’agent Azure comme suit :

    ```sh
    pkg_add py-setuptools openssl git
    ln -sf /usr/local/bin/python2.7 /usr/local/bin/python
    ln -sf /usr/local/bin/python2.7-2to3 /usr/local/bin/2to3
    ln -sf /usr/local/bin/python2.7-config /usr/local/bin/python-config
    ln -sf /usr/local/bin/pydoc2.7  /usr/local/bin/pydoc
    ```

6. La dernière version de l’agent Azure se trouve toujours sur [GitHub](https://github.com/Azure/WALinuxAgent/releases). Installez l’agent comme suit :

    ```sh
    git clone https://github.com/Azure/WALinuxAgent 
    cd WALinuxAgent
    python setup.py install
    waagent -register-service
    ```

    > [!IMPORTANT]
    > Après avoir installé l’agent Azure, il est judicieux de vérifier qu’il s’exécute comme suit :
    >
    > ```bash
    > ps auxw | grep waagent
    > root     79309  0.0  1.5  9184 15356 p1  S      4:11PM    0:00.46 python /usr/local/sbin/waagent -daemon (python2.7)
    > cat /var/log/waagent.log
    > ```

7. Annulez l’approvisionnement du système pour le nettoyer et le préparer pour son réapprovisionnement. La commande suivante supprime également le dernier compte d'utilisateur alloué et les données associées :

    ```sh
    waagent -deprovision+user -force
    ```

Vous pouvez à présent arrêter votre machine virtuelle.


## <a name="prepare-the-vhd"></a>Préparer le disque dur virtuel
Azure ne prend pas en charge le format VHDX, seulement le **VHD fixe**. Vous pouvez convertir le disque au format VHD fixe à l’aide du Gestionnaire Hyper-V ou de l’applet de commande Powershell [convert-vhd](/powershell/module/hyper-v/convert-vhd?view=win10-ps). Voici un exemple.

```powershell
Convert-VHD OpenBSD61.vhdx OpenBSD61.vhd -VHDType Fixed
```

## <a name="create-storage-resources-and-upload"></a>Créer des ressources de stockage et charger
Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Pour charger votre disque dur virtuel, créez un compte de stockage avec la commande [az storage account create](/cli/azure/storage/account). Le nom du compte de stockage devant être unique, entrez votre propre nom. L’exemple qui suit permet de créer un compte de stockage nommé *mystorageaccount* :

```azurecli
az storage account create --resource-group myResourceGroup \
    --name mystorageaccount \
    --location eastus \
    --sku Premium_LRS
```

Pour contrôler l’accès au compte de stockage, obtenez la clé de stockage avec la commande [az storage account keys list](/cli/azure/storage/account/keys) comme suit :

```azurecli
STORAGE_KEY=$(az storage account keys list \
    --resource-group myResourceGroup \
    --account-name mystorageaccount \
    --query "[?keyName=='key1']  | [0].value" -o tsv)
```

Pour séparer logiquement les disques durs virtuels que vous chargez, créez un conteneur dans le compte de stockage avec la commande [az storage container create](/cli/azure/storage/container) :

```azurecli
az storage container create \
    --name vhds \
    --account-name mystorageaccount \
    --account-key ${STORAGE_KEY}
```

Enfin, chargez votre disque dur virtuel avec la commande [az storage blob upload](/cli/azure/storage/blob) comme suit :

```azurecli
az storage blob upload \
    --container-name vhds \
    --file ./OpenBSD61.vhd \
    --name OpenBSD61.vhd \
    --account-name mystorageaccount \
    --account-key ${STORAGE_KEY}
```


## <a name="create-vm-from-your-vhd"></a>Créer la machine virtuelle à partir de votre disque dur virtuel
Vous pouvez créer une machine virtuelle avec un [exemple de script](../scripts/virtual-machines-linux-cli-sample-create-vm-vhd.md) ou directement avec la commande [az vm create](/cli/azure/vm). Pour spécifier le disque dur virtuel OpenBSD que vous avez chargé, utilisez le paramètre `--image` comme suit :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myOpenBSD61 \
    --image "https://mystorageaccount.blob.core.windows.net/vhds/OpenBSD61.vhd" \
    --os-type linux \
    --admin-username azureuser \
    --ssh-key-value ~/.ssh/id_rsa.pub
```

Obtenez l’adresse IP de votre machine virtuelle OpenBSD avec la commande [az vm list-ip-addresses](/cli/azure/vm) comme suit :

```azurecli
az vm list-ip-addresses --resource-group myResourceGroup --name myOpenBSD61
```

Vous pouvez désormais appliquer clé SSH à votre machine virtuelle OpenBSD comme d’habitude :
        
```bash
ssh azureuser@<ip address>
```


## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur la prise en charge de Hyper-V sur OpenBSD6.1, voir [OpenBSD 6.1](https://www.openbsd.org/61.html) et [hyperv.4](https://man.openbsd.org/hyperv.4).

Pour savoir comment créer une machine virtuelle à partir d’un disque géré, voir [az disk](/cli/azure/disk). 

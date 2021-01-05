---
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 05/07/2020
ms.author: danlep
ms.openlocfilehash: be170144fddeb1a69592f1714ec745d559665832
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82982424"
---
## <a name="create-a-docker-enabled-virtual-machine"></a>Créer une machine virtuelle compatible Docker

À des fins de test, utilisez une machine virtuelle Ubuntu compatible Docker pour accéder à un registre de conteneurs Azure. Pour utiliser l’authentification Azure Active Directory auprès du registre, installez également l’[interface de ligne de commande Azure][azure-cli] sur la machine virtuelle. Si vous disposez déjà d’une machine virtuelle, ignorez cette étape de création.

Vous pouvez utiliser le même groupe de ressources pour votre machine virtuelle et votre registre de conteneurs. Cette configuration simplifie le nettoyage à la fin, mais n’est pas nécessaire. Si vous choisissez de créer un groupe de ressources distinct pour la machine virtuelle et le réseau virtuel, exécutez [az group create][az-group-create]. L’exemple suivant suppose que vous avez défini des variables d’environnement pour le nom du groupe de ressources et l’emplacement du Registre :

```azurecli
az group create --name $RESOURCE_GROUP --location $REGISTRY_LOCATION
```

À présent, déployez une machine virtuelle Azure Ubuntu par défaut avec [az vm create][az-vm-create]. L’exemple suivant crée une machine virtuelle nommée *myDockerVM*.

```azurecli
VM_NAME=myDockerVM

az vm create \
  --resource-group $RESOURCE_GROUP \
  --name $VM_NAME \
  --image UbuntuLTS \
  --admin-username azureuser \
  --generate-ssh-keys
```

Il faut quelques minutes pour que la machine virtuelle soit créée. Après l’exécution de la commande, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Utilisez cette adresse pour établir des connexions SSH avec la machine virtuelle.

### <a name="install-docker-on-the-vm"></a>Installer Docker sur la machine virtuelle

Établissez une connexion SSH avec la machine virtuelle dès qu’elle est en cours d’exécution. Remplacez *publicIpAddress* par l’adresse IP publique de votre machine virtuelle.

```bash
ssh azureuser@publicIpAddress
```

Exécutez les commandes suivantes pour installer Docker sur la machine virtuelle Ubuntu :

```bash
sudo apt-get update
sudo apt install docker.io -y
```

Après l’installation, exécutez la commande suivante pour vérifier que Docker s’exécute correctement sur la machine virtuelle :

```bash
sudo docker run -it hello-world
```

Sortie :

```
Hello from Docker!
This message shows that your installation appears to be working correctly.
[...]
```

### <a name="install-the-azure-cli"></a>Installer l’interface de ligne de commande Microsoft Azure

Suivez les étapes de la section [Installer Azure CLI avec apt](/cli/azure/install-azure-cli-apt?view=azure-cli-latest) pour installer Azure CLI sur votre machine virtuelle Ubuntu. Par exemple :

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Quittez la connexion SSH.

[azure-cli]: /cli/azure/install-azure-cli
[az-vm-create]: /cli/azure/vm#az-vm-create
[az-group-create]: /cli/azure/group
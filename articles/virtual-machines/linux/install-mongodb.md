---
title: Installer MongoDB sur une machine virtuelle Linux avec Azure CLI
description: Découvrez comment installer et configurer MongoDB sur une machine virtuelle Linux avec Azure CLI
author: cynthn
manager: gwallace
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.devlang: azurecli
ms.topic: how-to
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 12/15/2017
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: 49a0e48977393aeab7ff93b79e28acc55a87b51a
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016180"
---
# <a name="how-to-install-and-configure-mongodb-on-a-linux-vm"></a>Guide pratique d’installation et de configuration de MongoDB sur une machine virtuelle Linux

[MongoDB](https://www.mongodb.org) est une base de données NoSQL open-source qui offre des performances élevées. Cet article montre comment installer et configurer MongoDB sur une machine virtuelle Linux avec Azure CLI. Présentations d’exemples détaillant comment :

* [Installer et configurer une instance MongoDB de base manuellement](#manually-install-and-configure-mongodb-on-a-vm)
* [Création d'une instance MongoDB de base à l'aide d'un modèle Resource Manager](#create-basic-mongodb-instance-on-centos-using-a-template)
* [Créer un cluster partitionné MongoDB complexe avec jeux de réplicas à l’aide d’un modèle Resource Manager](#create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template)


## <a name="manually-install-and-configure-mongodb-on-a-vm"></a>Installer et configurer MongoDB manuellement sur une machine virtuelle
MongoDB [propose des instructions d’installation](https://docs.mongodb.com/manual/administration/install-on-linux/) pour les distributions Linux, notamment Red Hat / CentOS, SUSE, Ubuntu et Debian. L’exemple suivant permet de créer une machine virtuelle nommée *CentOS*. Pour créer cet environnement, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index).

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). L’exemple suivant crée une machine virtuelle nommée *myVM* avec un utilisateur nommé *azureuser* utilisant l’authentification par clé publique SSH

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image CentOS \
    --admin-username azureuser \
    --generate-ssh-keys
```

SSH à la machine virtuelle à l’aide de votre propre nom d’utilisateur et de l’`publicIpAddress` indiquée dans la sortie de l’étape précédente :

```bash
ssh azureuser@<publicIpAddress>
```

Pour ajouter les sources d’installation pour MongoDB, créez un fichier de référentiel **yum** comme suit :

```bash
sudo touch /etc/yum.repos.d/mongodb-org-3.6.repo
```

Ouvrez le fichier de référentiel MongoDB à modifier, par exemple avec `vi` ou `nano`. Ajoutez les lignes suivantes :

```sh
[mongodb-org-3.6]
name=MongoDB Repository
baseurl=https://repo.mongodb.org/yum/redhat/$releasever/mongodb-org/3.6/x86_64/
gpgcheck=1
enabled=1
gpgkey=https://www.mongodb.org/static/pgp/server-3.6.asc
```

Installez MongoDB à l’aide de **yum** comme suit :

```bash
sudo yum install -y mongodb-org
```

Par défaut, SELinux est appliqué sur les images CentOS, ce qui vous empêche d’accéder à MongoDB. Installez les outils de gestion de stratégie et configurez SELinux afin d’autoriser MongoDB fonctionner sur le port TCP 27017 par défaut, comme suit :

```bash
sudo yum install -y policycoreutils-python
sudo semanage port -a -t mongod_port_t -p tcp 27017
```

Démarrez le service MongoDB comme suit :

```bash
sudo service mongod start
```

Vérifiez l’installation de MongoDB en vous connectant à l’aide du client `mongo` local :

```bash
mongo
```

Testez maintenant l’instance MongoDB en ajoutant des données, puis en recherchant :

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```

Si vous le souhaitez, configurez MongoDB pour démarrer automatiquement lors du redémarrage du système :

```bash
sudo chkconfig mongod on
```


## <a name="create-basic-mongodb-instance-on-centos-using-a-template"></a>Créeation d’une instance MongoDB de base sur CentOS à l’aide d’un modèle
Vous pouvez créer une instance MongoDB de base sur une machine virtuelle CentOS unique en utilisant le modèle de démarrage rapide Azure suivant à partir de GitHub. Ce gabarit utilise l’extension de script personnalisé pour Linux pour ajouter un référentiel **yum** à votre nouvelle machine virtuelle CentOS, puis installer MongoDB.

* [Instance MongoDB de base sur CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-on-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json

Pour créer cet environnement, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index). Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Ensuite, déployez le modèle MongoDB avec [az group deployment create](/cli/azure/group/deployment). À l’invite, renseignez vos propres valeurs uniques pour *newStorageAccountName*, *dnsNameForPublicIP*, ainsi que vos nom d’utilisateur et mot de passe administrateur :

```azurecli
az group deployment create --resource-group myResourceGroup \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-on-centos/azuredeploy.json
```

Connectez-vous à la machine virtuelle avec l’adresse DNS publique de votre machine virtuelle. Vous pouvez afficher l’adresse DNS publique avec [az vm show](/cli/azure/vm) :

```azurecli
az vm show -g myResourceGroup -n myLinuxVM -d --query [fqdns] -o tsv
```

SSH vers votre machine virtuelle avec votre propre nom d’utilisateur et votre adresse DNS publique :

```bash
ssh azureuser@mypublicdns.eastus.cloudapp.azure.com
```

Vérifiez l’installation de MongoDB en vous connectant à l’aide du client `mongo` local, comme suit :

```bash
mongo
```

Testez maintenant l’instance en ajoutant des données, puis en recherchant comme suit :

```sh
> db
test
> db.foo.insert( { a : 1 } )  
> db.foo.find()  
{ "_id" : ObjectId("57ec477cd639891710b90727"), "a" : 1 }
> exit
```


## <a name="create-a-complex-mongodb-sharded-cluster-on-centos-using-a-template"></a>Création d’un cluster partitionné MongoDB complexe sur CentOS à l’aide d’un modèle
Vous pouvez créer une instance MongoDB complexe sur un cluster partitionné en utilisant le modèle de démarrage rapide Azure suivant à partir de GitHub. Ce modèle suit les [meilleures pratiques pour les clusters partitionnés MongoDB](https://docs.mongodb.com/manual/core/sharded-cluster-components/) pour fournir la redondance et la haute disponibilité. Le modèle crée deux partitions, avec trois nœuds dans chaque jeu de réplicas. Un jeu de réplicas de serveur de configuration avec trois nœuds est également créé, ainsi que deux serveurs de routeur **mongos** pour assurer la cohérence des applications au sein des partitions.

* [Cluster de partitionnement MongoDB sur CentOS](https://github.com/Azure/azure-quickstart-templates/tree/master/mongodb-sharding-centos) - https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json

> [!WARNING]
> Le déploiement de ce cluster partitionné MongoDB complexe requiert plus de 20 cœurs, ce qui est généralement le nombre de cœurs par défaut par région pour un abonnement. Ouvrez une demande de support Azure pour augmenter votre nombre de cœurs.

Pour créer cet environnement, vous devez installer la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et vous connecter à un compte Azure avec [az login](/cli/azure/reference-index). Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create --name myResourceGroup --location eastus
```

Ensuite, déployez le modèle MongoDB avec [az group deployment create](/cli/azure/group/deployment). Définissez vos propres noms de ressources et tailles si nécessaire, comme par exemple pour *mongoAdminUsername*, *sizeOfDataDiskInGB* et *configNodeVmSize* :

```azurecli
az group deployment create --resource-group myResourceGroup \
  --parameters '{"adminUsername": {"value": "azureuser"},
    "adminPassword": {"value": "P@ssw0rd!"},
    "mongoAdminUsername": {"value": "mongoadmin"},
    "mongoAdminPassword": {"value": "P@ssw0rd!"},
    "dnsNamePrefix": {"value": "mypublicdns"},
    "environment": {"value": "AzureCloud"},
    "numDataDisks": {"value": "4"},
    "sizeOfDataDiskInGB": {"value": 20},
    "centOsVersion": {"value": "7.0"},
    "routerNodeVmSize": {"value": "Standard_DS3_v2"},
    "configNodeVmSize": {"value": "Standard_DS3_v2"},
    "replicaNodeVmSize": {"value": "Standard_DS3_v2"},
    "zabbixServerIPAddress": {"value": "Null"}}' \
  --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/mongodb-sharding-centos/azuredeploy.json \
  --name myMongoDBCluster \
  --no-wait
```

Ce déploiement peut nécessiter plus d’une heure pour déployer et configurer toutes les instances de machine virtuelle. L’indicateur `--no-wait` est utilisé à la fin de la commande précédente pour renvoyer le contrôle à l’invite de commandes une fois le déploiement du modèle accepté par la plateforme Azure. Vous pouvez ensuite afficher l’état du déploiement avec [az group deployment show](/cli/azure/group/deployment). L’exemple suivant vérifie l’état du déploiement de *myMongoDBCluster* dans le groupe de ressources *myResourceGroup* :

```azurecli
az group deployment show \
    --resource-group myResourceGroup \
    --name myMongoDBCluster \
    --query [properties.provisioningState] \
    --output tsv
```

## <a name="next-steps"></a>Étapes suivantes
Dans ces exemples, vous vous connectez à l’instance MongoDB localement à partir de la machine virtuelle. Si vous souhaitez vous connecter à l’instance MongoDB à partir d’une autre machine virtuelle ou d’un autre réseau, vérifiez que les bonnes [règles de groupe de sécurité réseau sont créées](nsg-quickstart.md).

Ces exemples montrent le déploiement de l’environnement MongoDB central à des fins de développement. Appliquez les options de configuration de sécurité requises pour votre environnement. Pour plus d’informations, voir les [documents relatifs à la sécurité de MongoDB](https://docs.mongodb.com/manual/security/).

Pour en savoir plus sur la création avec des modèles, voir [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).

Les modèles Azure Resource Manager utilisent l’extension de script personnalisé pour télécharger et exécuter des scripts sur vos machines virtuelles. Pour plus d’informations, consultez [Utilisation de l’extension de script personnalisé Azure avec des machines virtuelles Linux](../extensions/custom-script-linux.md).

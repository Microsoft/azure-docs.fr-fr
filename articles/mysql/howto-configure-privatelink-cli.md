---
title: Liaison privée - Azure CLI - Azure Database pour MySQL
description: Découvrez comment configurer une instance Private Link pour Azure Database pour MySQL à partir d’Azure CLI
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 01/09/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: da6a2b97d2656d56fa2aa0e7259fba433bd7b81e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95998581"
---
# <a name="create-and-manage-private-link-for-azure-database-for-mysql-using-cli"></a>Créer et gérer une liaison privée pour Azure Database pour MySQL en utilisant l’interface CLI

Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer en privé avec des ressources Private Link. Dans cet article, vous apprendrez à utiliser Azure CLI pour créer une machine virtuelle dans un réseau virtuel Azure et un serveur Azure Database pour MySQL avec un point de terminaison privé Azure.

> [!NOTE]
> La fonctionnalité de lien privé est disponible uniquement pour les serveurs Azure Database pour MySQL dans les niveaux tarifaires Usage général ou Mémoire optimisée. Vérifiez que le serveur de base de données se trouve dans l’un de ces niveaux tarifaires.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.28 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Avant de pouvoir créer des ressources, vous devez créer un groupe de ressources qui hébergera le réseau virtuel. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Cet exemple crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *westeurope* :

```azurecli-interactive
az group create --name myResourceGroup --location westeurope
```

## <a name="create-a-virtual-network"></a>Création d'un réseau virtuel
Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet). Cet exemple crée un réseau virtuel par défaut nommé *myVirtualNetwork* avec un sous-réseau nommé *mySubnet* :

```azurecli-interactive
az network vnet create \
 --name myVirtualNetwork \
 --resource-group myResourceGroup \
 --subnet-name mySubnet
```

## <a name="disable-subnet-private-endpoint-policies"></a>Désactiver les stratégies Private Endpoint du sous-réseau 
Azure déploie des ressources sur un sous-réseau au sein d’un réseau virtuel. vous devez donc créer ou mettre à jour le sous-réseau pour désactiver les [stratégies réseau](../private-link/disable-private-endpoint-network-policy.md) de point de terminaison privé. Mettez à jour une configuration de sous-réseau nommée *mySubnet* avec [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) :

```azurecli-interactive
az network vnet subnet update \
 --name mySubnet \
 --resource-group myResourceGroup \
 --vnet-name myVirtualNetwork \
 --disable-private-endpoint-network-policies true
```
## <a name="create-the-vm"></a>Création de la machine virtuelle 
Créez une machine virtuelle avec la commande az vm create. Lorsque vous y êtes invité, indiquez un mot de passe à utiliser comme informations d’identification pour vous connecter à la machine virtuelle. Cet exemple crée une machine virtuelle nommée *myVm* : 
```azurecli-interactive
az vm create \
  --resource-group myResourceGroup \
  --name myVm \
  --image Win2019Datacenter
```
Notez l’adresse IP publique de la machine virtuelle. Vous utiliserez cette adresse pour vous connecter à la machine virtuelle à partir d’Internet à l’étape suivante.

## <a name="create-an-azure-database-for-mysql-server"></a>Création d’un serveur Azure Database pour MySQL 
Créez un serveur Azure Database pour MySQL avec la commande az mysql server create. N’oubliez pas que le nom de votre serveur MySQL doit être unique dans Azure. Par conséquent, remplacez la valeur d’espace réservé entre crochets par votre propre valeur unique : 

```azurecli-interactive
# Create a server in the resource group 
az mysql server create \
--name mydemoserver \
--resource-group myResourcegroup \
--location westeurope \
--admin-user mylogin \
--admin-password <server_admin_password> \
--sku-name GP_Gen5_2
```

> [!NOTE]
> Il peut arriver que l’instance Azure Database pour MySQL et le sous-réseau de réseau virtuel se trouvent dans des abonnements différents. Dans ce cas, vous devez vérifier les configurations suivantes :
> - Assurez-vous que le fournisseur de ressources **Microsoft.DBforMySQL** est inscrit pour les deux abonnements. Pour plus d’informations, reportez-vous à [resource-manager-registration][resource-manager-portal]

## <a name="create-the-private-endpoint"></a>Créer l’instance Private Endpoint 
Créez un point de terminaison privé pour le serveur MySQL dans votre réseau virtuel : 

```azurecli-interactive
az network private-endpoint create \  
    --name myPrivateEndpoint \  
    --resource-group myResourceGroup \  
    --vnet-name myVirtualNetwork  \  
    --subnet mySubnet \  
    --private-connection-resource-id $(az resource show -g myResourcegroup -n mydemoserver --resource-type "Microsoft.DBforMySQL/servers" --query "id" -o tsv) \    
    --group-id mysqlServer \  
    --connection-name myConnection  
 ```

## <a name="configure-the-private-dns-zone"></a>Configurer la zone DNS privée 
Créez une zone DNS privée pour le domaine du serveur MySQL et créez un lien d’association avec le réseau virtuel. 
```azurecli-interactive
az network private-dns zone create --resource-group myResourceGroup \ 
   --name  "privatelink.mysql.database.azure.com" 
az network private-dns link vnet create --resource-group myResourceGroup \ 
   --zone-name  "privatelink.mysql.database.azure.com"\ 
   --name MyDNSLink \ 
   --virtual-network myVirtualNetwork \ 
   --registration-enabled false 

#Query for the network interface ID  
networkInterfaceId=$(az network private-endpoint show --name myPrivateEndpoint --resource-group myResourceGroup --query 'networkInterfaces[0].id' -o tsv)
 
 
az resource show --ids $networkInterfaceId --api-version 2019-04-01 -o json 
# Copy the content for privateIPAddress and FQDN matching the Azure database for MySQL name 
 
 
#Create DNS records 
az network private-dns record-set a create --name myserver --zone-name privatelink.mysql.database.azure.com --resource-group myResourceGroup  
az network private-dns record-set a add-record --record-set-name myserver --zone-name privatelink.mysql.database.azure.com --resource-group myResourceGroup -a <Private IP Address>
```

> [!NOTE] 
> Le nom de domaine complet dans le paramètre DNS du client n’est pas résolu en adresse IP privée configurée. Vous devez configurer une zone DNS pour le FQDN configuré, comme indiqué [ici](../dns/dns-operations-recordsets-portal.md).

## <a name="connect-to-a-vm-from-the-internet"></a>Se connecter à une machine virtuelle à partir d’Internet

Connectez-vous à la machine virtuelle *myVm* à partir d’Internet comme suit :

1. Dans la barre de recherche du portail, entrez *myVm*.

1. Sélectionnez le bouton **Connexion**. Après avoir sélectionné le bouton **Connecter**, **Se connecter à la machine virtuelle** s’ouvre.

1. Sélectionnez **Télécharger le fichier RDP**. Azure crée un fichier de protocole RDP (Remote Desktop Protocol) ( *.rdp*) et le télécharge sur votre ordinateur.

1. Ouvrez le fichier *downloaded.rdp*.

    1. Si vous y êtes invité, sélectionnez **Connexion**.

    1. Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle.

        > [!NOTE]
        > Vous devrez peut-être sélectionner **Plus de choix** > **Utiliser un autre compte**, pour spécifier les informations d’identification que vous avez entrées lorsque vous avez créé la machine virtuelle.

1. Sélectionnez **OK**.

1. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez un avertissement de certificat, sélectionnez **Oui** ou **Continuer**.

1. Une fois que le bureau de la machine virtuelle s’affiche, réduisez-le pour revenir à votre poste de travail local.  

## <a name="access-the-mysql-server-privately-from-the-vm"></a>Accéder au serveur MySQL en privé à partir de la machine virtuelle

1. Dans le Bureau à distance de  *myVM*, ouvrez PowerShell.

2. Entrez  `nslookup mydemomysqlserver.privatelink.mysql.database.azure.com`. 

    Vous recevez un message similaire à celui ci :
    ```azurepowershell
    Server:  UnKnown
    Address:  168.63.129.16
    Non-authoritative answer:
    Name:    mydemomysqlserver.privatelink.mysql.database.azure.com
    Address:  10.1.3.4
    ```

3. Testez la connexion de liaison privée pour le serveur MySQL à l’aide de tout client disponible. Dans l’exemple ci-dessous, j’ai utilisé [MySQL Workbench](https://dev.mysql.com/doc/workbench/en/wb-installing-windows.html) pour effectuer l’opération.


4. Dans **Nouvelle connexion**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de connexion| Sélectionnez le nom de connexion de votre choix.|
    | HostName | Sélectionnez *mydemoserver.privatelink.mysql.database.azure.com*. |
    | Nom d’utilisateur | Entrez le nom d’utilisateur au format *username@servername* qui est fourni lors de la création du serveur MySQL. |
    | Mot de passe | Entrez le mot de passe fourni lors de la création du serveur MySQL. |
    ||

5. Sélectionnez Se connecter.

6. Parcourez les bases de données dans le menu de gauche.

7. (Facultatif) Créez ou interrogez des informations à partir de la base de données MySQL.

8. Fermez la connexion Bureau à distance à myVm.

## <a name="clean-up-resources"></a>Nettoyer les ressources 
Lorsque vous n'en avez plus besoin, vous pouvez utiliser az group delete pour supprimer le groupe de ressources, ainsi que toutes les ressources qu’il contient : 

```azurecli-interactive
az group delete --name myResourceGroup --yes 
```

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [Qu’est-ce qu’Azure Private Endpoint ?](../private-link/private-endpoint-overview.md)

<!-- Link references, to text, Within this same GitHub repo. -->
[resource-manager-portal]: ../azure-resource-manager/management/resource-providers-and-types.md
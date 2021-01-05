---
title: Tutoriel - Se connecter à un serveur SQL Azure avec Azure Private Endpoint - Azure CLI
description: Ce tutoriel explique comment créer un serveur SQL Azure avec un point de terminaison privé à l’aide d’Azure CLI.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: tutorial
ms.date: 11/03/2020
ms.author: allensu
ms.openlocfilehash: 8cfe44b9433ee1daac028253aa45c97804c88ae5
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95544103"
---
# <a name="tutorial---connect-to-an-azure-sql-server-using-an-azure-private-endpoint---azure-cli"></a>Tutoriel - Se connecter à un serveur SQL Azure avec Azure Private Endpoint - Azure CLI

Azure Private Endpoint est le composant fondamental de Private Link dans Azure. Il permet à des ressources Azure, comme des machines virtuelles, de communiquer de façon privée avec des ressources Private Link.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un réseau virtuel et un hôte Azure bastion.
> * Création d’une machine virtuelle
> * Créer un serveur SQL Azure et un point de terminaison privé
> * Tester la connectivité au point de terminaison privé du serveur SQL

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Connectez-vous au portail Azure et vérifiez que votre abonnement est actif en exécutant la commande `az login`.
* Vérifiez votre version d’Azure CLI dans un terminal ou une fenêtre de commande en exécutant `az --version`. Pour obtenir la version la plus récente, consultez les [notes de publication les plus récentes](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si vous ne disposez pas de la dernière version, mettez à jour votre installation en suivant le [guide d’installation pour votre système d’exploitation ou votre plateforme](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az_group_create) :

* Nommé **CreateSQLEndpointTutorial-rg**. 
* Dans l’emplacement **eastus**.

```azurecli-interactive
az group create \
    --name CreateSQLEndpointTutorial-rg \
    --location eastus
```

## <a name="create-a-virtual-network-and-bastion-host"></a>Créer un réseau virtuel et un hôte bastion

Dans cette section, vous allez créer un réseau virtuel, un sous-réseau et un hôte bastion. 

L’hôte bastion sera utilisé pour se connecter de façon sécurisée à la machine virtuelle afin de tester le point de terminaison privé.

Créez un réseau virtuel avec la commande [az network vnet create](/cli/azure/network/vnet#az_network_vnet_create)

* Nommé **myVNet**.
* Préfixe d’adresse **10.0.0.0/16**.
* Sous-réseau nommé **myBackendSubnet**.
* Préfixe du sous-réseau **10.0.0.0/24**.
* Dans le groupe de ressources **CreateSQLEndpointTutorial-rg**.
* Emplacement **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreateSQLEndpointTutorial-rg\
    --location eastus \
    --name myVNet \
    --address-prefixes 10.0.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.0.0.0/24
```

Mettez à jour le sous-réseau afin de désactiver les stratégies réseau du point de terminaison privé avec [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) :

```azurecli-interactive
az network vnet subnet update \
    --name myBackendSubnet \
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Utilisez [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP publique pour l’hôte bastion :

* Créez une adresse IP publique redondante de zone standard nommée **myBastionIP**.
* Dans **CreateSQLEndpointTutorial-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myBastionIP \
    --sku Standard
```

Utilisez [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) pour créer un sous-réseau bastion :

* Nommé **AzureBastionSubnet**.
* Préfixe d’adresse **10.0.1.0/24**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreateSQLEndpointTutorial-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Utilisez [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) pour créer un hôte bastion :

* Nom : **myBastionHost**
* Dans **CreateSQLEndpointTutorial-rg**.
* Associé à l’adresse IP publique **myBastionIP**.
* Associé au réseau virtuel **myVNet**.
* À l’emplacement **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Le déploiement de l’hôte Azure Bastion peut prendre quelques minutes.

## <a name="create-test-virtual-machine"></a>Créer une machine virtuelle de test

Dans cette section, vous allez créer une machine virtuelle qui sera utilisée pour tester le point de terminaison privé.

Créez une machine virtuelle avec la commande  [az vm create](/cli/azure/vm#az_vm_create). Lorsque vous y êtes invité, indiquez le mot de passe à utiliser comme informations d’identification pour la machine virtuelle :

* Nommée **myVM**.
* Dans **CreateSQLEndpointTutorial-rg**.
* Dans le réseau **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Image de serveur **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

## <a name="create-an-azure-sql-server"></a>Créer un serveur SQL Azure

Dans cette section, vous allez créer un serveur et une base de données SQL.

Utilisez [az sql server create](/cli/azure/sql/server#az_sql_server_create) pour créer un serveur SQL :

* Remplacez **\<sql-server-name>** par le nom unique de votre serveur.
* Remplacez **\<your-password>** par votre mot de passe.
* Dans **CreateSQLEndpointTutorial-rg**.
* Dans la région **eastus**.

```azurecli-interactive
az sql server create \
    --name <sql-server-name> \
    --resource-group CreateSQLEndpointTutorial-rg \
    --location eastus \
    --admin-user sqladmin \
    --admin-password <your-password>
```

Utilisez [az sql db create](/cli/azure/sql/db#az_sql_db_create) pour créer une base de données :

* Nommée **myDataBase**.
* Dans **CreateSQLEndpointTutorial-rg**.
* Remplacez **\<sql-server-name>** par le nom unique de votre serveur.

```azurecli-interactive
az sql db create \
    --resource-group CreateSQLEndpointTutorial-rg  \
    --server <sql-server-name> \
    --name myDataBase \
    --sample-name AdventureWorksLT
```

## <a name="create-private-endpoint"></a>Créer un point de terminaison privé

Dans cette section, vous allez créer le point de terminaison privé.

Utilisez [az sql server list](/cli/azure/sql/server#az_sql_server_list) pour placer l’ID de ressource du serveur SQL dans une variable d’interpréteur de commandes.

Utilisez [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) pour créer le point de terminaison et la connexion :

* Nommé **myPrivateEndpoint**.
* Dans le groupe de ressources **CreateSQLEndpointTutorial-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Connexion nommée **myConnection**.

```azurecli-interactive
id=$(az sql server list \
    --resource-group CreateSQLEndpointTutorial-rg \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreateSQLEndpointTutorial-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-ids sqlServer \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Configurer la zone DNS privée

Dans cette section, vous allez créer et configurer la zone DNS privée au moyen de [az network private-dns zone create](/cli/azure/ext/privatedns/network/private-dns/zone#ext_privatedns_az_network_private_dns_zone_create).  

Vous allez utiliser [az network private-dns link vnet create](/cli/azure/ext/privatedns/network/private-dns/link/vnet#ext_privatedns_az_network_private_dns_link_vnet_create) pour créer le lien de réseau virtuel vers la zone DNS.

Vous allez créer un groupe de zones DNS avec [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create).

* Zone nommée **privatelink.database.windows.net**
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreateSQLEndpointTutorial-rg**.
* Lien DNS nommé **myDNSLink**.
* Associé à **myPrivateEndpoint**.
* Groupe de zones nommé **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --name "privatelink.database.windows.net"

az network private-dns link vnet create \
    --resource-group CreateSQLEndpointTutorial-rg \
    --zone-name "privatelink.database.windows.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreateSQLEndpointTutorial-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.database.windows.net" \
   --zone-name sql
```

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter au serveur SQL via le point de terminaison privé.

1. Connectez-vous au [portail Azure](https://portal.azure.com) 
 
2. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

3. Sélectionnez **CreateSQLEndpointTutorial-rg**.

4. Sélectionnez **myVM**.

5. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

6. Sélectionnez le bouton bleu **Utiliser le bastion**.

7. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

8. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

9. Entrez `nslookup <sqlserver-name>.database.windows.net`. Remplacez **\<sqlserver-name>** par le nom du serveur SQL que vous avez créé dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mysqlserver8675.privatelink.database.windows.net
    Address:  10.0.0.5
    Aliases:  mysqlserver8675.database.windows.net
    ```

    Une adresse IP privée **10.0.0.5** est retournée pour le nom du serveur SQL.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.


10. Installez [SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms?preserve-view=true&view=sql-server-2017) sur **myVM**.

11. Ouvrez **SQL Server Management Studio**.

12. Dans **Se connecter au serveur**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Type de serveur | Sélectionnez **Moteur de base de données**.|
    | Nom du serveur | Entrez **\<sql-server-name>.database.windows.net** |
    | Authentification | Sélectionnez **Authentification SQL Server**. |
    | Nom d’utilisateur | Entrez le nom d’utilisateur que vous avez entré lors de la création du serveur. |
    | Mot de passe | Entrez le mot de passe que vous avez entré lors de la création du serveur. |
    | Se souvenir du mot de passe | Sélectionnez **Oui**. |

13. Sélectionnez **Connecter**.

14. Parcourez les bases de données dans le menu de gauche.

15. (Facultatif) Créez ou interrogez des informations à partir de **mysqldatabase**.

16. Fermez la connexion Bastion à **myVM**. 

## <a name="clean-up-resources"></a>Nettoyer les ressources 

Lorsque vous avez fini d’utiliser le point de terminaison privé, le serveur SQL et la machine virtuelle, supprimez le groupe de ressources et toutes les ressources qu’il contient : 

```azurecli-interactive
az group delete \
    --name CreateSQLEndpointTutorial-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez créé les éléments suivants :

* Un réseau virtuel et un hôte bastion.
* Une machine virtuelle
* Un serveur SQL Azure avec un point de terminaison privé

Vous avez utilisé la machine virtuelle pour tester de façon sécurisée la connectivité au serveur SQL via le point de terminaison privé.

Découvrez comment créer un service Private Link :
> [!div class="nextstepaction"]
> [Créer un service Private Link](create-private-link-service-portal.md)
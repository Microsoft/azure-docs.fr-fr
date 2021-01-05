---
title: Démarrage rapide – Créer un point de terminaison privé Azure avec Azure CLI
description: Découvrez dans ce guide de démarrage rapide comment créer un point de terminaison privé avec Azure CLI.
services: private-link
author: asudbring
ms.service: private-link
ms.topic: quickstart
ms.date: 11/07/2020
ms.author: allensu
ms.openlocfilehash: bba912930a9dff0a79e0b0d81025b7524c238db0
ms.sourcegitcommit: 22da82c32accf97a82919bf50b9901668dc55c97
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/08/2020
ms.locfileid: "94368676"
---
# <a name="quickstart-create-a-private-endpoint-using-azure-cli"></a>Démarrage rapide : Créer un point de terminaison privé à l’aide d’Azure CLI

Démarrez avec Azure Private Link en utilisant un point de terminaison privé pour vous connecter en toute sécurité à une application web Azure.

Dans ce guide de démarrage rapide, vous allez créer un point de terminaison privé pour une application web Azure et déployer une machine virtuelle pour tester la connexion privée.  

Des points de terminaison privés peuvent être créés pour différents types de services Azure, tels qu’Azure SQL et Stockage Azure.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Une application web Azure avec un **niveau PremiumV2** ou un plan App Service supérieur déployé dans votre abonnement Azure.  
    * Pour plus d’informations et pour obtenir un exemple, consultez [Démarrage rapide : Créer une application web ASP.NET Core dans Azure](../app-service/quickstart-dotnetcore.md). 
    * Pour obtenir un tutoriel détaillé sur la création d’une application web et d’un point de terminaison, consultez [Tutoriel : Se connecter à une application web à l’aide d’Azure Private Endpoint](tutorial-private-endpoint-webapp-portal.md).
* Connectez-vous au portail Azure et vérifiez que votre abonnement est actif en exécutant la commande `az login`.
* Vérifiez votre version d’Azure CLI dans un terminal ou une fenêtre de commande en exécutant `az --version`. Pour obtenir la version la plus récente, consultez les [notes de publication les plus récentes](/cli/azure/release-notes-azure-cli?tabs=azure-cli).
  * Si vous ne disposez pas de la dernière version, mettez à jour votre installation en suivant le [guide d’installation pour votre système d’exploitation ou votre plateforme](/cli/azure/install-azure-cli).

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az_group_create) :

* Nommé **CreatePrivateEndpointQS-rg**. 
* Dans l’emplacement **eastus**.

```azurecli-interactive
az group create \
    --name CreatePrivateEndpointQS-rg \
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
* Dans le groupe de ressources **CreatePrivateEndpointQS-rg**.
* Emplacement **eastus**.

```azurecli-interactive
az network vnet create \
    --resource-group CreatePrivateEndpointQS-rg\
    --location eastus \
    --name myVNet \
    --address-prefixes 10.0.0.0/16 \
    --subnet-name myBackendSubnet \
    --subnet-prefixes 10.0.0.0/24
```

Mettez à jour le sous-réseau pour désactiver les stratégies réseau de point de terminaison privé pour le point de terminaison privé à l’aide de la commande [az network vnet subnet update](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-update) :

```azurecli-interactive
az network vnet subnet update \
    --name myBackendSubnet \
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet \
    --disable-private-endpoint-network-policies true
```

Utilisez [az network public-ip create](/cli/azure/network/public-ip#az-network-public-ip-create) pour créer une adresse IP publique pour l’hôte bastion :

* Créez une adresse IP publique redondante de zone standard nommée **myBastionIP**.
* Dans **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network public-ip create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myBastionIP \
    --sku Standard
```

Utilisez [az network vnet subnet create](/cli/azure/network/vnet/subnet#az-network-vnet-subnet-create) pour créer un sous-réseau bastion :

* Nommé **AzureBastionSubnet**.
* Préfixe d’adresse **10.0.1.0/24**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreatePrivateEndpointQS-rg**.

```azurecli-interactive
az network vnet subnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name AzureBastionSubnet \
    --vnet-name myVNet \
    --address-prefixes 10.0.1.0/24
```

Utilisez [az network bastion create](/cli/azure/network/bastion#az-network-bastion-create) pour créer un hôte bastion :

* Nom : **myBastionHost**
* Dans **CreatePrivateEndpointQS-rg**.
* Associé à l’adresse IP publique **myBastionIP**.
* Associé au réseau virtuel **myVNet**.
* À l’emplacement **eastus**.

```azurecli-interactive
az network bastion create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myBastionHost \
    --public-ip-address myBastionIP \
    --vnet-name myVNet \
    --location eastus
```

Le déploiement de l’hôte Azure Bastion peut prendre quelques minutes.

## <a name="create-test-virtual-machine"></a>Créer une machine virtuelle de test

Dans cette section, vous allez créer une machine virtuelle qui sera utilisée pour tester le point de terminaison privé.

Créez une machine virtuelle avec la commande  [az vm create](/cli/azure/vm#az_vm_create). Lorsque vous y êtes invité, indiquez un mot de passe à utiliser comme informations d’identification pour la machine virtuelle :

* Nommée **myVM**.
* Dans **CreatePrivateEndpointQS-rg**.
* Dans le réseau **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Image de serveur **Win2019Datacenter**.

```azurecli-interactive
az vm create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name myVM \
    --image Win2019Datacenter \
    --public-ip-address "" \
    --vnet-name myVNet \
    --subnet myBackendSubnet \
    --admin-username azureuser
```

## <a name="create-private-endpoint"></a>Créer un point de terminaison privé

Dans cette section, vous allez créer le point de terminaison privé.

Utilisez [az webapp list](/cli/azure/webapp#az_webapp_list) pour placer l’ID de ressource de l’application web que vous avez créée précédemment dans une variable d’environnement.

Utilisez [az network private-endpoint create](/cli/azure/network/private-endpoint#az_network_private_endpoint_create) pour créer le point de terminaison et la connexion :

* Nommé **myPrivateEndpoint**.
* Dans le groupe de ressources **CreatePrivateEndpointQS-rg**.
* Dans le réseau virtuel **myVNet**.
* Dans le sous-réseau **myBackendSubnet**.
* Connexion nommée **myConnection**.
* Votre application web **\<webapp-resource-group-name>** .

```azurecli-interactive
id=$(az webapp list \
    --resource-group <webapp-resource-group-name> \
    --query '[].[id]' \
    --output tsv)

az network private-endpoint create \
    --name myPrivateEndpoint \
    --resource-group CreatePrivateEndpointQS-rg \
    --vnet-name myVNet --subnet myBackendSubnet \
    --private-connection-resource-id $id \
    --group-id sites \
    --connection-name myConnection  
```

## <a name="configure-the-private-dns-zone"></a>Configurer la zone DNS privée

Dans cette section, vous allez créer et configurer la zone DNS privée au moyen de [az network private-dns zone create](/cli/azure/ext/privatedns/network/private-dns/zone#ext_privatedns_az_network_private_dns_zone_create).  

Vous utiliserez [az network private-dns link vnet create](/cli/azure/ext/privatedns/network/private-dns/link/vnet#ext_privatedns_az_network_private_dns_link_vnet_create) pour créer le lien de réseau virtuel vers la zone DNS.

Vous créerez un groupe de zones DNS au moyen de [az network private-endpoint dns-zone-group create](/cli/azure/network/private-endpoint/dns-zone-group#az_network_private_endpoint_dns_zone_group_create).

* Zone nommée **privatelink.azurewebsites.net**.
* Dans le réseau virtuel **myVNet**.
* Dans le groupe de ressources **CreatePrivateEndpointQS-rg**.
* Lien DNS nommé **myDNSLink**.
* Associé à **myPrivateEndpoint**.
* Groupe de zones nommé **MyZoneGroup**.

```azurecli-interactive
az network private-dns zone create \
    --resource-group CreatePrivateEndpointQS-rg \
    --name "privatelink.azurewebsites.net"

az network private-dns link vnet create \
    --resource-group CreatePrivateEndpointQS-rg \
    --zone-name "privatelink.azurewebsites.net" \
    --name MyDNSLink \
    --virtual-network myVNet \
    --registration-enabled false

az network private-endpoint dns-zone-group create \
   --resource-group CreatePrivateEndpointQS-rg \
   --endpoint-name myPrivateEndpoint \
   --name MyZoneGroup \
   --private-dns-zone "privatelink.azurewebsites.net" \
   --zone-name webapp
```

## <a name="test-connectivity-to-private-endpoint"></a>Tester la connectivité au point de terminaison privé

Dans cette section, vous allez utiliser la machine virtuelle que vous avez créée à l’étape précédente pour vous connecter au serveur SQL via le point de terminaison privé.

1. Connectez-vous au [portail Azure](https://portal.azure.com) 
 
2. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.

3. Sélectionnez **CreatePrivateEndpointQS-rg**.

4. Sélectionnez **myVM**.

5. Dans la page de vue d’ensemble pour **myVM**, sélectionnez **Se connecter**, puis **Bastion**.

6. Sélectionnez le bouton bleu **Utiliser le bastion**.

7. Entrez le nom d’utilisateur et le mot de passe que vous avez utilisés lors de la création de la machine virtuelle.

8. Ouvrez Windows PowerShell sur le serveur après vous être connecté.

9. Entrez `nslookup <your-webapp-name>.azurewebsites.net`. Remplacez **\<your-webapp-name>** par le nom de l’application web que vous avez créée dans les étapes précédentes.  Vous recevez un message similaire à ce qui est montré ci-dessous :

    ```powershell
    Server:  UnKnown
    Address:  168.63.129.16

    Non-authoritative answer:
    Name:    mywebapp8675.privatelink.azurewebsites.net
    Address:  10.0.0.5
    Aliases:  mywebapp8675.azurewebsites.net
    ```

    L’adresse IP privée **10.0.0.5** est retournée pour le nom de l’application web.  Cette adresse se trouve dans le sous-réseau du réseau virtuel que vous avez créé précédemment.

10. Dans la connexion bastion à **myVM**, ouvrez Internet Explorer.

11. Entrez l’URL de votre application web : **https://\<your-webapp-name>.azurewebsites.net**.

12. Vous recevrez la page d’application web par défaut si votre application n’a pas été déployée :

    :::image type="content" source="./media/create-private-endpoint-portal/web-app-default-page.png" alt-text="Page d’application web par défaut." border="true":::

13. Fermez la connexion à **myVM**.

## <a name="clean-up-resources"></a>Nettoyer les ressources 
Lorsque vous avez fini d’utiliser le point de terminaison privé et la machine virtuelle, utilisez [az group delete](/cli/azure/group#az_group_delete) pour supprimer le groupe de ressources et toutes les ressources qu’il contient :

```azurecli-interactive
az group delete \
    --name CreatePrivateEndpointQS-rg
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :

* Un réseau virtuel et un hôte bastion.
* Une machine virtuelle.
* Un point de terminaison privé pour une application web Azure.

Vous avez utilisé la machine virtuelle pour tester la connectivité de façon sécurisée à l’application web via le point de terminaison privé.

Pour plus d’informations sur les services qui prennent en charge un point de terminaison privé, consultez :
> [!div class="nextstepaction"]
> [Disponibilité des liaisons privées](private-link-overview.md#availability)

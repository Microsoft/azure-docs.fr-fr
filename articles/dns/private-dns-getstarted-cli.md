---
title: Démarrage rapide - Créer une zone Azure DNS privée avec l’interface Azure CLI
description: Dans ce guide de démarrage rapide, vous allez créer et tester une zone DNS privée et un enregistrement DNS privé dans Azure DNS. Il s’agit d’un guide pas à pas pour la création et la gestion de votre première zone privée DNS et de votre premier enregistrement à l’aide d’Azure CLI.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: rohink
ms.custom: devx-track-azurecli
ms.openlocfilehash: df180f0aefc817004e99d63998d000498c4d15aa
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92310153"
---
# <a name="quickstart-create-an-azure-private-dns-zone-using-the-azure-cli"></a>Démarrage rapide : Créer une zone Azure DNS privée avec l’interface Azure CLI

Ce guide de démarrage rapide vous guide tout au long des étapes de création de votre première zone DNS privée et de votre premier enregistrement DNS privé à l’aide de l’interface Azure CLI.

Une zone DNS permet d’héberger les enregistrements DNS d’un domaine particulier. Pour commencer à héberger votre domaine dans le DNS Azure, vous devez créer une zone DNS pour ce nom de domaine. Chaque enregistrement DNS pour votre domaine est ensuite créé à l’intérieur de cette zone DNS. Pour publier une zone DNS privée sur votre réseau virtuel, vous spécifiez la liste des réseaux virtuels qui sont autorisés à résoudre les enregistrements dans la zone.  On les appelle réseaux virtuels *liés*. Quand l’inscription automatique est activée, Azure DNS met également à jour les enregistrements de zone chaque fois qu’une machine virtuelle est créée, change d’adresse IP ou est supprimée.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous préférez, vous pouvez suivre ce guide de démarrage rapide en utilisant [Azure PowerShell](private-dns-getstarted-powershell.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Tout d’abord, créez un groupe de ressources pour contenir la zone DNS : 

```azurecli
az group create --name MyAzureResourceGroup --location "East US"
```

## <a name="create-a-private-dns-zone"></a>Créer une zone DNS privée

L’exemple suivant crée un réseau virtuel nommé **myAzureVNet**. Il crée ensuite une zone DNS nommée **private.contoso.com** dans le groupe de ressources **MyAzureResourceGroup**, puis il lie la zone DNS au réseau virtuel **MyAzureVnet** et active l’inscription automatique.

```azurecli
az network vnet create \
  --name myAzureVNet \
  --resource-group MyAzureResourceGroup \
  --location eastus \
  --address-prefix 10.2.0.0/16 \
  --subnet-name backendSubnet \
  --subnet-prefixes 10.2.0.0/24

az network private-dns zone create -g MyAzureResourceGroup \
   -n private.contoso.com

az network private-dns link vnet create -g MyAzureResourceGroup -n MyDNSLink \
   -z private.contoso.com -v myAzureVNet -e true
```

Si vous souhaitez créer une zone dédiée à la résolution de noms (pas d’inscription automatique de nom d’hôte), vous pouvez utiliser le paramètre `-e false`.

### <a name="list-dns-private-zones"></a>Répertorier les zones privées DNS

Pour énumérer des zones DNS, utilisez `az network private-dns zone list`. Pour obtenir de l’aide, consultez l’article `az network dns zone list --help`.

Spécifier le groupe de ressources permet de ne lister que les zones du groupe de ressources :

```azurecli
az network private-dns zone list \
  -g MyAzureResourceGroup
```

Omettre le groupe de ressources permet de lister toutes les zones de l’abonnement :

```azurecli
az network private-dns zone list 
```

## <a name="create-the-test-virtual-machines"></a>Créer les machines virtuelles de test

Maintenant, créez deux machines virtuelles afin de pouvoir tester votre zone DNS privée :

```azurecli
az vm create \
 -n myVM01 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter

az vm create \
 -n myVM02 \
 --admin-username AzureAdmin \
 -g MyAzureResourceGroup \
 -l eastus \
 --subnet backendSubnet \
 --vnet-name myAzureVnet \
 --nsg NSG01 \
 --nsg-rule RDP \
 --image win2016datacenter
```

L’exécution de cette opération nécessite quelques minutes.

## <a name="create-an-additional-dns-record"></a>Créer un enregistrement DNS supplémentaire

Pour créer un enregistrement DNS, utilisez la commande `az network private-dns record-set [record type] add-record`. Pour obtenir de l’aide, concernant l’ajout d’enregistrements A par exemple, consultez `az network private-dns record-set A add-record --help`.

 L’exemple suivant crée un enregistrement avec le nom relatif **db** dans la zone DNS **private.contoso.com**, dans le groupe de ressources **MyAzureResourceGroup**. Le nom complet du jeu d’enregistrements est **db.private.contoso.com**. Le type d’enregistrement est « A », avec l’adresse IP « 10.2.0.4 ».

```azurecli
az network private-dns record-set a add-record \
  -g MyAzureResourceGroup \
  -z private.contoso.com \
  -n db \
  -a 10.2.0.4
```

### <a name="view-dns-records"></a>Afficher les enregistrements DNS

Pour répertorier les enregistrements DNS dans votre zone, utilisez :

```azurecli
az network private-dns record-set list \
  -g MyAzureResourceGroup \
  -z private.contoso.com
```

## <a name="test-the-private-zone"></a>Tester la zone privée

Maintenant vous pouvez désormais tester la résolution de noms pour votre zone privée **private.contoso.com**.

### <a name="configure-vms-to-allow-inbound-icmp"></a>Configurer des machines virtuelles pour autoriser l’ICMP entrant

Vous pouvez utiliser la commande test ping pour tester la résolution de nom. Par conséquent, configurez le pare-feu sur les deux machines virtuelles pour autoriser les paquets ICMP entrants.

1. Connectez-vous à myVM01 et ouvrez une fenêtre Windows PowerShell avec les privilèges d’administrateur.
2. Exécutez la commande suivante :

   ```powershell
   New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
   ```

Répétez l’opération pour myVM02.

### <a name="ping-the-vms-by-name"></a>Faites un test ping sur les machines virtuelles par leur nom

1. À partir de l’invite de commandes Windows PowerShell myVM02, faites un test ping de myVM01 en utilisant le nom d’hôte enregistré automatiquement :

   ```
   ping myVM01.private.contoso.com
   ```

   La sortie doit ressembler à ceci :

   ```
   PS C:\> ping myvm01.private.contoso.com

   Pinging myvm01.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time=1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 1ms, Average = 0ms
   PS C:\>
   ```

2. Maintenant faites un test ping sur le nom **db** que vous avez créé précédemment :

   ```
   ping db.private.contoso.com
   ```

   La sortie doit ressembler à ceci :

   ```
   PS C:\> ping db.private.contoso.com

   Pinging db.private.contoso.com [10.2.0.4] with 32 bytes of data:
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128
   Reply from 10.2.0.4: bytes=32 time<1ms TTL=128

   Ping statistics for 10.2.0.4:
       Packets: Sent = 4, Received = 4, Lost = 0 (0% loss),
   Approximate round trip times in milli-seconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources **MyAzureResourceGroup** afin de supprimer les ressources créées dans cette guide de démarrage rapide.

```azurecli
az group delete --name MyAzureResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Scénarios Azure DNS Private Zones](private-dns-scenarios.md)


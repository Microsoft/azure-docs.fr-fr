---
title: Démarrage rapide - Créer une zone Azure DNS privée avec Azure PowerShell
description: Dans ce guide de démarrage rapide, vous allez découvrir comment créer et gérer votre première zone privée DNS et votre premier enregistrement à l’aide d’Azure PowerShell.
services: dns
author: rohinkoul
ms.service: dns
ms.topic: quickstart
ms.date: 10/20/2020
ms.author: rohink
ms.openlocfilehash: ee6dde6b34cccd415f9bf2052f65dcbe940715c1
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424388"
---
# <a name="quickstart-create-an-azure-private-dns-zone-using-azure-powershell"></a>Démarrage rapide : Créer une zone Azure DNS privée avec Azure PowerShell

Cet article vous indique la procédure à suivre pour créer votre premier enregistrement et votre première zone DNS privée à l’aide d’Azure PowerShell.

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Une zone DNS permet d’héberger les enregistrements DNS d’un domaine particulier. Pour commencer à héberger votre domaine dans le DNS Azure, vous devez créer une zone DNS pour ce nom de domaine. Chaque enregistrement DNS pour votre domaine est ensuite créé à l’intérieur de cette zone DNS. Pour publier une zone DNS privée sur votre réseau virtuel, vous spécifiez la liste des réseaux virtuels qui sont autorisés à résoudre les enregistrements dans la zone.  On les appelle réseaux virtuels *liés*. Quand l’inscription automatique est activée, Azure DNS met également à jour les enregistrements de zone chaque fois qu’une machine virtuelle est créée, change d’adresse IP ou est supprimée.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous préférez, vous pouvez suivre ce guide de démarrage rapide en utilisant [Azure CLI](private-dns-getstarted-cli.md).

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-group"></a>Créer le groupe de ressources

Tout d’abord, créez un groupe de ressources pour contenir la zone DNS : 

```azurepowershell
New-AzResourceGroup -name MyAzureResourceGroup -location "eastus"
```

## <a name="create-a-private-dns-zone"></a>Créer une zone DNS privée

Une zone DNS est créée à l’aide de l’applet de commande `New-AzPrivateDnsZone` .

L’exemple suivant crée un réseau virtuel nommé **myAzureVNet**. Il crée ensuite une zone DNS nommée **private.contoso.com** dans le groupe de ressources **MyAzureResourceGroup** , puis il lie la zone DNS au réseau virtuel **MyAzureVnet** et active l’inscription automatique.

```azurepowershell
Install-Module -Name Az.PrivateDns -force

$backendSubnet = New-AzVirtualNetworkSubnetConfig -Name backendSubnet -AddressPrefix "10.2.0.0/24"
$vnet = New-AzVirtualNetwork `
  -ResourceGroupName MyAzureResourceGroup `
  -Location eastus `
  -Name myAzureVNet `
  -AddressPrefix 10.2.0.0/16 `
  -Subnet $backendSubnet

$zone = New-AzPrivateDnsZone -Name private.contoso.com -ResourceGroupName MyAzureResourceGroup

$link = New-AzPrivateDnsVirtualNetworkLink -ZoneName private.contoso.com `
  -ResourceGroupName MyAzureResourceGroup -Name "mylink" `
  -VirtualNetworkId $vnet.id -EnableRegistration
```

Si vous souhaitez créer une zone dédiée à la résolution de noms (aucune inscription automatique de nom d’hôte), vous pouvez omettre le paramètre `-EnableRegistration`.

### <a name="list-dns-private-zones"></a>Répertorier les zones privées DNS

En omettant le nom de la zone dans `Get-AzPrivateDnsZone`, vous pouvez énumérer toutes les zones d’un groupe de ressources. Cette opération renvoie un tableau d’objets de la zone.

```azurepowershell
$zones = Get-AzPrivateDnsZone -ResourceGroupName MyAzureResourceGroup
$zones
```

En omettant le nom de zone et le nom du groupe de ressources de `Get-AzPrivateDnsZone`, vous pouvez énumérer toutes les zones dans l’abonnement Azure.

```azurepowershell
$zones = Get-AzPrivateDnsZone
$zones
```

## <a name="create-the-test-virtual-machines"></a>Créer les machines virtuelles de test

Maintenant, créez deux machines virtuelles afin de pouvoir tester votre zone DNS privée :

```azurepowershell
New-AzVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM01" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389

New-AzVm `
    -ResourceGroupName "myAzureResourceGroup" `
    -Name "myVM02" `
    -Location "East US" `
    -subnetname backendSubnet `
    -VirtualNetworkName "myAzureVnet" `
    -addressprefix 10.2.0.0/24 `
    -OpenPorts 3389
```

L’exécution de cette opération nécessite quelques minutes.

## <a name="create-an-additional-dns-record"></a>Créer un enregistrement DNS supplémentaire

Vous pouvez utiliser l’applet de commande `New-AzPrivateDnsRecordSet` pour créer des jeux d’enregistrements. L’exemple suivant crée un enregistrement avec le nom relatif **db** dans la zone DNS **private.contoso.com** , dans le groupe de ressources **MyAzureResourceGroup**. Le nom complet du jeu d’enregistrements est **db.private.contoso.com**. Le type d’enregistrement est « A », avec l’adresse IP « 10.2.0.4 », et la durée de vie est de 3600secondes.

```azurepowershell
New-AzPrivateDnsRecordSet -Name db -RecordType A -ZoneName private.contoso.com `
   -ResourceGroupName MyAzureResourceGroup -Ttl 3600 `
   -PrivateDnsRecords (New-AzPrivateDnsRecordConfig -IPv4Address "10.2.0.4")
```

### <a name="view-dns-records"></a>Afficher les enregistrements DNS

Pour répertorier les enregistrements DNS dans votre zone, utilisez :

```azurepowershell
Get-AzPrivateDnsRecordSet -ZoneName private.contoso.com -ResourceGroupName MyAzureResourceGroup
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
   Approximate round trip times in milliseconds:
       Minimum = 0ms, Maximum = 0ms, Average = 0ms
   PS C:\>
   ```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources **MyAzureResourceGroup** pour supprimer les ressources créées dans cet article.

```azurepowershell
Remove-AzResourceGroup -Name MyAzureResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Scénarios Azure DNS Private Zones](private-dns-scenarios.md)

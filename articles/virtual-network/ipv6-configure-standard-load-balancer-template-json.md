---
title: Déployer une application double pile IPv6 dans un réseau virtuel Azure - Modèle Resource Manager
titlesuffix: Azure Virtual Network
description: Cet article montre comment déployer une application double pile IPv6 avec Standard Load Balancer dans un réseau virtuel Azure à l’aide de modèles de machine virtuelle Azure Resource Manager.
services: virtual-network
documentationcenter: na
author: KumudD
manager: mtillman
ms.service: virtual-network
ms.devlang: NA
ms.topic: how-to
ms.workload: infrastructure-services
ms.date: 03/31/2020
ms.author: kumud
ms.openlocfilehash: f98ffde9ce9b5c564e9f9c6c0b95e11f76719597
ms.sourcegitcommit: 30906a33111621bc7b9b245a9a2ab2e33310f33f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/22/2020
ms.locfileid: "95244342"
---
# <a name="deploy-an-ipv6-dual-stack-application-in-azure-virtual-network---template"></a>Déployer une application double pile IPv6 sur un réseau virtuel Azure - Modèle

Cet article fournit une liste de tâches de configuration IPv6 avec la partie du modèle de machine virtuelle Azure Resource Manager correspondante. Utilisez le modèle décrit dans cet article pour déployer dans Azure une application double pile (IPv4 + IPv6) avec Standard Load Balancer incluant un réseau virtuel double pile avec des sous-réseaux IPv4 et IPv6, un Standard Load Balancer avec des configurations front-end doubles (IPv4 + IPv6), des machines virtuelles dont les cartes réseau ont une configuration double IP, un groupe de sécurité réseau et des adresses IP publiques. 

## <a name="required-configurations"></a>Configurations requises

Recherchez les sections dans le modèle pour les localiser.

### <a name="ipv6-addressspace-for-the-virtual-network"></a>Paramètre addressSpace IPv6 du réseau virtuel

Section de modèle à ajouter :

```JSON
        "addressSpace": {
          "addressPrefixes": [
            "[variables('vnetv4AddressRange')]",
            "[variables('vnetv6AddressRange')]"    
```

### <a name="ipv6-subnet-within-the-ipv6-virtual-network-addressspace"></a>Sous-réseau IPv6 dans le paramètre addressSpace du réseau virtuel IPv6

Section de modèle à ajouter :
```JSON
          {
            "name": "V6Subnet",
            "properties": {
              "addressPrefix": "[variables('subnetv6AddressRange')]"
            }

```

### <a name="ipv6-configuration-for-the-nic"></a>Configuration IPv6 pour la carte réseau

Section de modèle à ajouter :
```JSON
          {
            "name": "ipconfig-v6",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
          "privateIPAddressVersion":"IPv6",
              "subnet": {
                "id": "[variables('v6-subnet-id')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[concat(resourceId('Microsoft.Network/loadBalancers','loadBalancer'),'/backendAddressPools/LBBAP-v6')]"
                }
```

### <a name="ipv6-network-security-group-nsg-rules"></a>Règles du groupe de sécurité réseau iPv6

```JSON
          {
            "name": "default-allow-rdp",
            "properties": {
              "description": "Allow RDP",
              "protocol": "Tcp",
              "sourcePortRange": "33819-33829",
              "destinationPortRange": "5000-6000",
              "sourceAddressPrefix": "fd00:db8:deca:deed::/64",
              "destinationAddressPrefix": "fd00:db8:deca:deed::/64",
              "access": "Allow",
              "priority": 1003,
              "direction": "Inbound"
            }
```

## <a name="conditional-configuration"></a>Configuration conditionnelle

Si vous utilisez une appliance virtuelle réseau, ajoutez des itinéraires IPv6 dans la table de routage. Sinon, cette configuration est facultative.

```JSON
    {
      "type": "Microsoft.Network/routeTables",
      "name": "v6route",
      "apiVersion": "[variables('ApiVersion')]",
      "location": "[resourceGroup().location]",
      "properties": {
        "routes": [
          {
            "name": "v6route",
            "properties": {
              "addressPrefix": "fd00:db8:deca:deed::/64",
              "nextHopType": "VirtualAppliance",
              "nextHopIpAddress": "fd00:db8:ace:f00d::1"
            }
```

## <a name="optional-configuration"></a>Configuration facultative

### <a name="ipv6-internet-access-for-the-virtual-network"></a>Accès Internet IPv6 pour le réseau virtuel

```JSON
{
            "name": "LBFE-v6",
            "properties": {
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses','lbpublicip-v6')]"
              }
```

### <a name="ipv6-public-ip-addresses"></a>Adresses IP publiques IPv6

```JSON
    {
      "apiVersion": "[variables('ApiVersion')]",
      "type": "Microsoft.Network/publicIPAddresses",
      "name": "lbpublicip-v6",
      "location": "[resourceGroup().location]",
      "sku": {
        "name": "Standard"
      },
      "properties": {
        "publicIPAllocationMethod": "Static",
        "publicIPAddressVersion": "IPv6"
      }
```

### <a name="ipv6-front-end-for-load-balancer"></a>Serveur frontal IPv6 pour l’équilibreur de charge

```JSON
          {
            "name": "LBFE-v6",
            "properties": {
              "publicIPAddress": {
                "id": "[resourceId('Microsoft.Network/publicIPAddresses','lbpublicip-v6')]"
              }
```

### <a name="ipv6-back-end-address-pool-for-load-balancer"></a>Pool d’adresses IPv6 back-end pour l’équilibreur de charge

```JSON
              "backendAddressPool": {
                "id": "[concat(resourceId('Microsoft.Network/loadBalancers', 'loadBalancer'), '/backendAddressPools/LBBAP-v6')]"
              },
              "protocol": "Tcp",
              "frontendPort": 8080,
              "backendPort": 8080
            },
            "name": "lbrule-v6"
```

### <a name="ipv6-load-balancer-rules-to-associate-incoming-and-outgoing-ports"></a>Règles d’équilibreur de charge IPv6 pour associer des ports entrants et sortants

```JSON
          {
            "name": "ipconfig-v6",
            "properties": {
              "privateIPAllocationMethod": "Dynamic",
          "privateIPAddressVersion":"IPv6",
              "subnet": {
                "id": "[variables('v6-subnet-id')]"
              },
              "loadBalancerBackendAddressPools": [
                {
                  "id": "[concat(resourceId('Microsoft.Network/loadBalancers','loadBalancer'),'/backendAddressPools/LBBAP-v6')]"
                }
```

## <a name="sample-vm-template-json"></a>Exemple de JSON de modèle de machine virtuelle
Pour déployer une application double pile IPv6 dans un réseau virtuel Azure à l’aide d’un modèle Azure Resource Manager, consultez l’exemple de modèle [ici](https://azure.microsoft.com/resources/templates/ipv6-in-vnet-stdlb/).

## <a name="next-steps"></a>Étapes suivantes

Des informations sont disponibles sur la tarification des [adresses IP publiques](https://azure.microsoft.com/pricing/details/ip-addresses/), de la [bande passante réseau](https://azure.microsoft.com/pricing/details/bandwidth/) ou de l’[équilibreur de charge](https://azure.microsoft.com/pricing/details/load-balancer/).

---
title: Fichier Include
description: Fichier Include
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: include
ms.date: 03/21/2018
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 069d7af9cee0bee673c8f0b32659ce362fe4dd70
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025923"
---
### <a name="to-modify-the-local-network-gateway-gatewayipaddress"></a>Pour modifier la passerelle de réseau local « gatewayIpAddress »

Si le périphérique VPN auquel vous souhaitez vous connecter a changé son adresse IP publique, vous devez modifier la passerelle de réseau local pour refléter cette modification. L’adresse IP de passerelle peut être modifiée sans supprimer de connexion de passerelle VPN existante (si vous en possédez une). Pour modifier l’adresse IP de passerelle, remplacez les valeurs « Site2 » et « TestRG1 » par vos propres valeurs à l’aide de la commande [az network local-gateway update](/cli/azure/network/local-gateway).

```azurecli
az network local-gateway update --gateway-ip-address 23.99.222.170 --name Site2 --resource-group TestRG1
```

Vérifiez que l’adresse IP est correcte dans la sortie :

```
"gatewayIpAddress": "23.99.222.170",
```
---
title: Utiliser une adresse IP privée pour le routage interne d'un point de terminaison d'entrée
description: Cet article fournit des informations sur l'utilisation d'adresses IP privées pour le routage interne, ce qui expose le point de terminaison d'entrée d'un cluster au reste du réseau virtuel.
services: application-gateway
author: caya
ms.service: application-gateway
ms.topic: how-to
ms.date: 11/4/2019
ms.author: caya
ms.openlocfilehash: 8be37ed1da0da4da3db43ef4c1cd01ed962f24ed
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93397306"
---
# <a name="use-private-ip-for-internal-routing-for-an-ingress-endpoint"></a>Utiliser une adresse IP privée pour le routage interne d'un point de terminaison d'entrée 

Cette fonctionnalité permet d'exposer le point de terminaison d'entrée du `Virtual Network` via une adresse IP privée.

## <a name="pre-requisites"></a>Conditions préalables  
Application Gateway avec [configuration d'une adresse IP privée](./configure-application-gateway-with-private-frontend-ip.md)

Il existe deux façons de configurer le contrôleur afin qu'il utilise une adresse IP privée pour l'entrée.

## <a name="assign-to-a-particular-ingress"></a>Attribuer à une entrée particulière
Pour exposer une entrée particulière sur une adresse IP privée, utilisez l'annotation [`appgw.ingress.kubernetes.io/use-private-ip`](./ingress-controller-annotations.md#use-private-ip) dans l'entrée.

### <a name="usage"></a>Usage
```yaml
appgw.ingress.kubernetes.io/use-private-ip: "true"
```

Pour les instances d'Application Gateway sans adresse IP privée, les entrées annotées avec `appgw.ingress.kubernetes.io/use-private-ip: "true"` seront ignorées. Cela sera indiqué dans l'événement d'entrée et le journal du pod AGIC.

* Erreur telle d'indiquée dans l'événement d'entrée

    ```bash
    Events:
    Type     Reason       Age               From                                                                     Message
    ----     ------       ----              ----                                                                     -------
    Warning  NoPrivateIP  2m (x17 over 2m)  azure/application-gateway, prod-ingress-azure-5c9b6fcd4-bctcb  Ingress default/hello-world-ingress requires Application Gateway 
    applicationgateway3026 has a private IP address
    ```

* Erreur telle d'indiquée dans les journaux AGIC

    ```bash
    E0730 18:57:37.914749       1 prune.go:65] Ingress default/hello-world-ingress requires Application Gateway applicationgateway3026 has a private IP address
    ```


## <a name="assign-globally"></a>Attribuer globalement
Si vous devez limiter toutes les entrées à exposer sur l'adresse IP privée, utilisez `appgw.usePrivateIP: true` dans la configuration `helm`.

### <a name="usage"></a>Usage
```yaml
appgw:
    subscriptionId: <subscriptionId>
    resourceGroup: <resourceGroupName>
    name: <applicationGatewayName>
    usePrivateIP: true
```

Le contrôleur d'entrée filtre alors les configurations d'adresse IP sur une adresse IP privée lors de la configuration des écouteurs frontaux sur l'instance d'Application Gateway.
AGIC panique et se bloque si `usePrivateIP: true`, et aucune adresse IP privée n'est attribuée.

> [!NOTE]
> La référence SKU Application Gateway v2 requiert une adresse IP publique. Si vous souhaitez que l'instance d'Application Gateway soit privée, joignez un [`Network Security Group`](../virtual-network/network-security-groups-overview.md) au sous-réseau de cette dernière pour limiter le trafic.
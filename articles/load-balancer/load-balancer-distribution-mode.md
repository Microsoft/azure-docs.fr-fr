---
title: Configurer le mode de distribution d’Azure Load Balancer
titleSuffix: Azure Load Balancer
description: Dans cet article, procédez à la configuration du mode de distribution pour Azure Load Balancer afin de prendre en charge l’affinité d’IP source.
services: load-balancer
documentationcenter: na
author: asudbring
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.custom: seodec18
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/19/2019
ms.author: allensu
ms.openlocfilehash: 4e4fa9819343c90b5639fc485af8acb48968d62e
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94695654"
---
# <a name="configure-the-distribution-mode-for-azure-load-balancer"></a>Configuration du mode de distribution pour Azure Load Balancer

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="hash-based-distribution-mode"></a>Mode de distribution basé sur le hachage

Par défaut, le mode de distribution pour Azure Load Balancer est un hachage à cinq tuples. 

Le tuple est composé des éléments suivants :
* **IP source**
* **Port source**
* **IP de destination**
* **Port de destination**
* **Type de protocole**

Le hachage est utilisé pour mapper le trafic aux serveurs disponibles. L’algorithme fournit l’adhérence uniquement dans une session de transport. Les paquets de la même session sont dirigés vers la même instance IP de centre de données derrière le point de terminaison d’équilibrage de charge. Lorsque le client démarre une nouvelle session à partir du même IP source, le port source change et contraint le trafic à se diriger vers un autre point de terminaison du centre de données.

![Mode de distribution basé sur le hachage à cinq tuples](./media/load-balancer-distribution-mode/load-balancer-distribution.png)

## <a name="source-ip-affinity-mode"></a>Mode d’affinité d’IP source

L’équilibreur de charge peut également être configuré à l’aide du mode de distribution d’affinité d’IP source. Ce mode de distribution est également connu sous le nom d’affinité de session ou d’affinité d’IP client. Il utilise un hachage à deux tuples (IP source et IP de destination) ou à trois tuples (IP source, IP de destination et type de protocole) pour mapper le trafic vers les serveurs disponibles. En utilisant l’affinité d’IP source, les connexions démarrées depuis le même ordinateur client s’orientent vers le même point de terminaison du centre de données.

L’image suivante illustre une configuration à deux tuples. Notez comment la distribution à deux tuples passe de l’équilibreur de charge à la machine virtuelle 1 (VM1). La machine virtuelle VM1 est ensuite sauvegardée par VM2 et VM3.

![Mode de distribution d’affinité de session à deux tuples](./media/load-balancer-distribution-mode/load-balancer-session-affinity.png)

Le mode d’affinité d’IP source permet de résoudre une incompatibilité entre Azure Load Balancer et la Passerelle des services Bureau à distance. En utilisant ce mode, vous pouvez générer une batterie de Passerelle des services Bureau à distance dans un seul service cloud.

Le chargement de médias constitue un autre scénario d’utilisation. Le chargement des données se produit via UDP, mais le plan de contrôle est obtenu via TCP :

* Un client démarre une session TCP à l’adresse publique à charge équilibrée et est dirigé vers une adresse DIP spécifique. Ce canal reste actif, de sorte que l’intégrité de la connexion puisse être surveillée.
* Une nouvelle session UDP issue du même ordinateur client est démarrée sur le même point de terminaison public d’équilibrage de charge. La connexion est dirigée vers le même point de terminaison DIP que la connexion TCP précédente. Le chargement de médias peut être exécuté à débit élevé alors que le canal de contrôle via TCP est maintenu.

> [!NOTE]
> Lorsqu’un jeu d’équilibrage de charge est modifié (suppression ou ajout d’une machine virtuelle), la distribution des demandes du client est recalculée. Vous ne pouvez pas dépendre de nouvelles connexions à partir de clients existants qui se retrouvent sur le même serveur. En outre, le mode de distribution d’affinité d’IP source peut entraîner une distribution inégale du trafic. Les clients qui s’exécutent derrière des serveurs proxy peuvent être vus comme une application cliente unique.

## <a name="configure-source-ip-affinity-settings"></a>Configurer les paramètres d’affinité IP source

### <a name="azure-portal"></a>Portail Azure

Vous pouvez modifier la configuration du mode de distribution en changeant la règle d’équilibrage de charge dans le portail.

1. Connectez-vous au portail Azure et recherchez le groupe de ressources contenant l’équilibreur de charge que vous souhaitez modifier en cliquant sur **Groupes de ressources**.
2. Dans l’écran Vue d’ensemble de l’équilibreur de charge, sous **Paramètres**, cliquez sur **Règles d’équilibrage de charge**.
3. Dans l’écran Règles d’équilibrage de charge, cliquez sur la règle d’équilibrage de charge dont vous souhaitez modifier le mode de distribution.
4. Sous la règle, modifiez le mode de distribution en changeant la sélection dans la zone de liste déroulante **Persistance de session**.  Les options suivantes sont disponibles :
    
    * **Aucun (basé sur le hachage)**  : spécifie que les demandes successives provenant du même client peuvent être gérées par n’importe quelle machine virtuelle.
    * **Adresse IP du client (tuple de 2 éléments d’affinité d’adresse IP source)** : spécifie que les demandes successives provenant de la même adresse IP du client seront gérées par le même ordinateur virtuel.
    * **Adresse IP et protocole du client (tuple de 3 éléments d’affinité d’adresse IP source)** : spécifie que les demandes successives provenant de la même combinaison d’adresse IP et de protocole du client seront gérées par le même ordinateur virtuel.

5. Choisissez le mode de distribution, puis cliquez sur **Enregistrer**.

### <a name="azure-powershell"></a>Azure PowerShell

Pour les machines virtuelles déployées avec Resource Manager, utilisez PowerShell pour modifier les paramètres de distribution de l’équilibreur de charge et utiliser une règle d’équilibrage de charge existante. La commande suivante met à jour le mode de distribution : 

```azurepowershell-interactive
$lb = Get-AzLoadBalancer -Name MyLb -ResourceGroupName MyLbRg
$lb.LoadBalancingRules[0].LoadDistribution = 'sourceIp'
Set-AzLoadBalancer -LoadBalancer $lb
```

Pour les machines virtuelles classiques, utilisez Azure PowerShell pour modifier les paramètres de distribution. Ajoutez un point de terminaison Azure à une machine virtuelle et configurez le mode de distribution de l’équilibreur de charge :

```azurepowershell-interactive
Get-AzureVM -ServiceName mySvc -Name MyVM1 | Add-AzureEndpoint -Name HttpIn -Protocol TCP -PublicPort 80 -LocalPort 8080 –LoadBalancerDistribution sourceIP | Update-AzureVM
```

Définissez la valeur de l’élément `LoadBalancerDistribution` sur le volume d’équilibrage de charge requis. Spécifiez sourceIP pour l’équilibrage de charge à deux tuples (IP source et IP de destination). Spécifiez sourceIPProtocol pour l’équilibrage de charge à trois tuples (IP source, IP de destination et type de protocole). Spécifiez none pour le comportement par défaut de l’équilibrage de charge à cinq tuples.

Récupérez la configuration du mode de distribution d’équilibrage de charge d’un point de terminaison en utilisant ces paramètres :

```azurepowershell
PS C:\> Get-AzureVM –ServiceName MyService –Name MyVM | Get-AzureEndpoint

VERBOSE: 6:43:50 PM - Completed Operation: Get Deployment
LBSetName : MyLoadBalancedSet
LocalPort : 80
Name : HTTP
Port : 80
Protocol : tcp
Vip : 65.52.xxx.xxx
ProbePath :
ProbePort : 80
ProbeProtocol : tcp
ProbeIntervalInSeconds : 15
ProbeTimeoutInSeconds : 31
EnableDirectServerReturn : False
Acl : {}
InternalLoadBalancerName :
IdleTimeoutInMinutes : 15
LoadBalancerDistribution : sourceIP
```

Si l’élément `LoadBalancerDistribution` n’est pas présent, Azure Load Balancer utilise l’algorithme à cinq tuples par défaut.

### <a name="configure-distribution-mode-on-load-balanced-endpoint-set"></a>Configurer le mode de distribution sur un jeu de points de terminaison d’équilibrage de charge

Lorsque les points de terminaison font partie d’un jeu de points de terminaison d’équilibrage de charge, le mode de distribution doit être configuré sur ce jeu :

```azurepowershell-interactive
Set-AzureLoadBalancedEndpoint -ServiceName MyService -LBSetName LBSet1 -Protocol TCP -LocalPort 80 -ProbeProtocolTCP -ProbePort 8080 –LoadBalancerDistribution sourceIP
```

### <a name="configure-distribution-mode-for-cloud-services-endpoints"></a>Configurer le mode de distribution pour les points de terminaison de services cloud

Utilisez le Kit de développement logiciel (SDK) Azure pour .NET 2.5 pour mettre à jour votre service cloud. Les paramètres de point de terminaison des services cloud sont définis dans le fichier .csdef. Pour mettre à jour le mode de distribution d’équilibrage de charge pour un déploiement de services cloud, une mise à niveau du déploiement s’impose.

Voici un exemple de modifications apportées aux paramètres de point de terminaison dans .csdef :

```xml
<WorkerRole name="worker-role-name" vmsize="worker-role-size" enableNativeCodeExecution="[true|false]">
    <Endpoints>
    <InputEndpoint name="input-endpoint-name" protocol="[http|https|tcp|udp]" localPort="local-port-number" port="port-number" certificate="certificate-name" loadBalancerProbe="load-balancer-probe-name" loadBalancerDistribution="sourceIP" />
    </Endpoints>
</WorkerRole>
<NetworkConfiguration>
    <VirtualNetworkSite name="VNet"/>
    <AddressAssignments>
<InstanceAddress roleName="VMRolePersisted">
    <PublicIPs>
    <PublicIP name="public-ip-name" idleTimeoutInMinutes="timeout-in-minutes"/>
    </PublicIPs>
</InstanceAddress>
    </AddressAssignments>
</NetworkConfiguration>
```

## <a name="api-example"></a>Exemple d’API

L’exemple suivant montre comment reconfigurer le mode de distribution d’équilibrage de charge pour un jeu d’équilibrage de charge spécifié dans un déploiement. 

### <a name="change-distribution-mode-for-deployed-load-balanced-set"></a>Modifier le mode de distribution pour un jeu d’équilibrage de charge déployé

Utilisez le modèle de déploiement Azure Classic pour modifier une configuration de déploiement existante. Ajoutez l’en-tête `x-ms-version` et définissez la valeur sur la version 2014-09-01 ou une version ultérieure.

#### <a name="request"></a>Requête

```http
POST https://management.core.windows.net/<subscription-id>/services/hostedservices/<cloudservice-name>/deployments/<deployment-name>?comp=UpdateLbSet   x-ms-version: 2014-09-01
Content-Type: application/xml
```

```xml
<LoadBalancedEndpointList xmlns="http://schemas.microsoft.com/windowsazure" xmlns:i="https://www.w3.org/2001/XMLSchema-instance">
    <InputEndpoint>
        <LoadBalancedEndpointSetName> endpoint-set-name </LoadBalancedEndpointSetName>
        <LocalPort> local-port-number </LocalPort>
        <Port> external-port-number </Port>
        <LoadBalancerProbe>
            <Port> port-assigned-to-probe </Port>
            <Protocol> probe-protocol </Protocol>
            <IntervalInSeconds> interval-of-probe </IntervalInSeconds>
            <TimeoutInSeconds> timeout-for-probe </TimeoutInSeconds>
        </LoadBalancerProbe>
        <Protocol> endpoint-protocol </Protocol>
        <EnableDirectServerReturn> enable-direct-server-return </EnableDirectServerReturn>
        <IdleTimeoutInMinutes>idle-time-out</IdleTimeoutInMinutes>
        <LoadBalancerDistribution>sourceIP</LoadBalancerDistribution>
    </InputEndpoint>
</LoadBalancedEndpointList>
```

Comme décrit précédemment, définissez l’élément `LoadBalancerDistribution` sur sourceIP pour une affinité à deux tuples, sur sourceIPProtocol pour une affinité à trois tuples ou sur none pour ne définir aucune affinité (affinité à cinq tuples).

#### <a name="response"></a>response

```http
HTTP/1.1 202 Accepted
Cache-Control: no-cache
Content-Length: 0
Server: 1.0.6198.146 (rd_rdfe_stable.141015-1306) Microsoft-HTTPAPI/2.0
x-ms-servedbyregion: ussouth2
x-ms-request-id: 9c7bda3e67c621a6b57096323069f7af
Date: Thu, 16 Oct 2014 22:49:21 GMT
```

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble d’Azure Load Balancer](load-balancer-overview.md)
* [Prise en main de la configuration d’un équilibreur de charge sur Internet](quickstart-load-balancer-standard-public-powershell.md)
* [Configuration des paramètres du délai d’expiration TCP inactif pour votre équilibrage de charge](load-balancer-tcp-idle-timeout.md)
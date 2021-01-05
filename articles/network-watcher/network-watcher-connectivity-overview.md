---
title: Présentation de la résolution des problèmes de connexion d’Azure Network Watcher | Microsoft Docs
description: Cette page fournit une vue d’ensemble de la fonctionnalité de résolution des problèmes de connexion de Network Watcher
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/11/2017
ms.author: damendo
ms.openlocfilehash: 9c855fff9e5791b9c0cf870acfc6de53e7a700b2
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97653994"
---
# <a name="introduction-to-connection-troubleshoot-in-azure-network-watcher"></a>Présentation de la résolution des problèmes de connexion dans Azure Network Watcher

La fonctionnalité de résolution des problèmes de connexion de Network Watcher permet de vérifier la connexion TCP directe entre deux machines virtuelles, le nom de domaine complet, l’URI ou l’adresse IPv4. Les scénarios de réseau sont complexes. Ils sont implémentés à l’aide de groupes de sécurité réseau, de pare-feu, d’itinéraires définis par l’utilisateur et de ressources fournies par Azure. Ces configurations complexes rendent difficile la résolution des problèmes de connectivité. Network Watcher permet de réduire le temps de détection des problèmes de connectivité. Les résultats obtenus peuvent indiquer si le problème de connectivité est dû à un problème de configuration de la plateforme ou à un problème de configuration utilisateur. La connectivité peut être vérifiée avec [PowerShell](network-watcher-connectivity-powershell.md), [Azure CLI](network-watcher-connectivity-cli.md) et l’[API REST](network-watcher-connectivity-rest.md).

> [!IMPORTANT]
> Pour résoudre les problèmes de connexion, la machine virtuelle à partir de laquelle vous procédez doit disposer de l’extension de machine virtuelle `AzureNetworkWatcherExtension` installée. Pour installer l’extension sur une machine virtuelle Windows, consultez la page [Azure Network Watcher Agent virtual machine extension for Windows](../virtual-machines/extensions/network-watcher-windows.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) (Extension de machine virtuelle d’agent Azure Network Watcher pour Windows). Pour une machine virtuelle Linux, consultez la page [Azure Network Watcher Agent virtual machine extension for Linux](../virtual-machines/extensions/network-watcher-linux.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) (Extension de machine virtuelle d’agent Azure Network Watcher pour Linux). L’extension n’est pas nécessaire sur le point de terminaison de destination.

## <a name="response"></a>response

Le tableau suivant présente les propriétés retournées une fois la résolution des problèmes de connexion terminée.

|Propriété  |Description  |
|---------|---------|
|ConnectionStatus     | État de la vérification de la connectivité. Les résultats possibles sont **Joignable** et **Inaccessible**.        |
|AvgLatencyInMs     | Latence moyenne pendant la vérification de la connectivité, en millisecondes (affichée uniquement si l’état de la vérification est Joignable).        |
|MinLatencyInMs     | Latence minimale pendant la vérification de la connectivité, en millisecondes (affichée uniquement si l’état de la vérification est Joignable).        |
|MaxLatencyInMs     | Latence maximale pendant la vérification de la connectivité, en millisecondes (affichée uniquement si l’état de la vérification est Joignable).        |
|ProbesSent     | Nombre de sondes envoyées lors de la vérification. La valeur maximale est de 100.        |
|ProbesFailed     | Nombre de sondes ayant échoué lors de la vérification. La valeur maximale est de 100.        |
|Hops     | Chemin tronçon par tronçon entre la source et la destination.        |
|Hops[].Type     | Type de ressource. Les valeurs possibles sont **Source**, **VirtualAppliance**, **VnetLocal**, et **Internet**.        |
|Hops[].Id | Identificateur unique du tronçon.|
|Hops[].Address | Adresse IP du tronçon.|
|Hops[].ResourceId | ID de ressource du tronçon si le tronçon est une ressource Azure. S’il s’agit d’une ressource internet, l’ID de ressource est **Internet**. |
|Hops[].NextHopIds | Identificateur unique du prochain tronçon.|
|Hops[].Issues | Ensemble des problèmes rencontrés lors de la vérification au niveau de ce tronçon. En l’absence de problème, la valeur est vide.|
|Hops[].Issues[].Origin | Au niveau du tronçon actuel, emplacement où le problème s’est produit. Les valeurs possibles sont les suivantes :<br/> **Inbound** : le problème se trouve sur le lien entre le tronçon précédent et le tronçon actuel<br/>**Outbound** : le problème se trouve sur le lien entre le tronçon actuel et le tronçon suivant<br/>**Local** : le problème se trouve sur le tronçon en cours.|
|Hops[].Issues[].Severity | Niveau de gravité du problème détecté. Les valeurs possibles sont **Error** et **Warning**. |
|Hops[].Issues[].Type |Type de problème détecté. Les valeurs possibles sont les suivantes : <br/>**UC**<br/>**Mémoire**<br/>**GuestFirewall**<br/>**DnsResolution**<br/>**NetworkSecurityRule**<br/>**UserDefinedRoute** |
|Hops[].Issues[].Context |Détails relatifs au problème détecté.|
|Hops[].Issues[].Context[].key |Clé de la paire clé/valeur retournée.|
|Hops[].Issues[].Context[].value |Valeur de la paire clé/valeur retournée.|

Vous trouverez ci-dessous un exemple de problème détecté sur un tronçon.

```json
"Issues": [
    {
        "Origin": "Outbound",
        "Severity": "Error",
        "Type": "NetworkSecurityRule",
        "Context": [
            {
                "key": "RuleName",
                "value": "UserRule_Port80"
            }
        ]
    }
]
```
## <a name="fault-types"></a>Types d’erreur

La résolution des problèmes de connexion retourne les types d’erreur liés à la connexion. Le tableau suivant fournit une liste des types d’erreur actuels retournés.

|Type  |Description  |
|---------|---------|
|UC     | Utilisation élevée du processeur.       |
|Mémoire     | Utilisation élevée de la mémoire.       |
|GuestFirewall     | Le trafic est bloqué à cause de la configuration d’un pare-feu de machine virtuelle.        |
|DNSResolution     | Échec de la résolution DNS pour l’adresse de destination.        |
|NetworkSecurityRule    | Le trafic est bloqué par une règle de groupe de sécurité réseau (règle retournée)        |
|UserDefinedRoute|Le trafic est ignoré en raison d’un itinéraire défini par l’utilisateur ou le système. |

### <a name="next-steps"></a>Étapes suivantes

Découvrez comment résoudre les problèmes de connexion à l’aide du [portail Azure](network-watcher-connectivity-portal.md), de [PowerShell](network-watcher-connectivity-powershell.md), d’[Azure CLI](network-watcher-connectivity-cli.md) ou de l’[API REST](network-watcher-connectivity-rest.md).
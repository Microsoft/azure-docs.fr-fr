---
title: Créer une règle Azure Load Balancer pour un cluster
description: Configurez Azure Load Balancer pour ouvrir les ports pour votre cluster Azure Service Fabric.
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: 7e09c7b0b3e2bfa5a5ff834e243f5098cbbd947b
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92319898"
---
# <a name="open-ports-for-a-service-fabric-cluster"></a>Ouvrir des ports pour un cluster Service Fabric

L’équilibreur de charge déployé avec votre cluster Azure Service Fabric dirige le trafic vers votre application en cours d’exécution sur un nœud. Si vous modifiez votre application pour utiliser un autre port, vous devez exposer ce port (ou acheminer un port différent) dans Azure Load Balancer.

Une fois votre cluster Service Fabric déployé vers Azure, un équilibreur de charge est créé automatiquement pour vous. Si vous ne disposez pas d’un équilibrage de charge, consultez [Configurer un équilibreur de charge connecté à Internet](../load-balancer/quickstart-load-balancer-standard-public-portal.md).


[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="configure-service-fabric"></a>Configurer Service Fabric

Le fichier de configuration de votre application Service Fabric **ServiceManifest.xml** définit les points de terminaison que votre application s’attend à utiliser. Une fois le fichier de configuration mis à jour pour définir un point de terminaison, l’équilibreur de charge doit être mis à jour pour exposer ce port (ou un autre). Pour plus d’informations sur la façon de créer le point de terminaison Service Fabric, consultez [Configurer un point de terminaison](service-fabric-service-manifest-resources.md).

## <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle Load Balancer ouvre un port connecté à internet et transfère le trafic vers le port du nœud interne utilisé par votre application. Si vous ne disposez pas d’un équilibrage de charge, consultez [Configurer un équilibreur de charge connecté à Internet](../load-balancer/quickstart-load-balancer-standard-public-portal.md).

Pour créer une règle Load Balancer, vous devez collecter les informations suivantes :

- Nom de l’équilibreur de charge.
- Groupe de ressources de l’équilibreur de charge et du cluster Service Fabric.
- Port externe.
- Port interne.

## <a name="azure-cli"></a>Azure CLI
Il suffit d’une seule commande pour créer une règle d’équilibreur de charge avec l’interface **Azure CLI**. Vous devez simplement connaître le nom de l’équilibreur de charge et du groupe de ressources pour créer une nouvelle règle.

>[!NOTE]
>Si vous avez besoin déterminer le nom de l’équilibreur de charge, utilisez cette commande pour obtenir rapidement une liste de tous les équilibreurs de charge et des groupes de ressources associés.
>
>`az network lb list --query "[].{ResourceGroup: resourceGroup, Name: name}"`
>


```azurecli
az network lb rule create --backend-port 40000 --frontend-port 39999 --protocol Tcp --lb-name LB-svcfab3 -g svcfab_cli -n my-app-rule
```

La commande Azure CLI dispose de quelques paramètres décrits dans le tableau suivant :

| Paramètre | Description |
| --------- | ----------- |
| `--backend-port`  | Le port que l’application Service Fabric écoute. |
| `--frontend-port` | Le port que l’équilibreur de charge expose aux connexions externes. |
| `-lb-name` | Le nom de l’équilibreur de charge à modifier. |
| `-g`       | Le groupe de ressources qui dispose de l’équilibreur de charge et du cluster Service Fabric. |
| `-n`       | Le nom voulu pour la règle. |


>[!NOTE]
>Pour plus d’informations sur la création d’un équilibreur de charge avec l’interface Azure CLI, consultez [Créer un équilibreur de charge avec l’interface Azure CLI](../load-balancer/quickstart-load-balancer-standard-internal-cli.md).

## <a name="powershell"></a>PowerShell

PowerShell est un peu plus compliqué que l’interface Azure CLI. Suivez ces étapes conceptuels pour créer une règle :

1. Récupérez l’équilibreur de charge d’Azure.
2. Créez une règle.
3. Ajoutez la règle pour l'équilibreur de charge.
4. Mettez à jour l’équilibreur de charge.

>[!NOTE]
>Si vous devez déterminer le nom de l’équilibreur de charge, utilisez cette commande pour obtenir rapidement une liste de tous les équilibreurs de charge et des groupes de ressources associés.
>
>`Get-AzLoadBalancer | Select Name, ResourceGroupName`

```powershell
# Get the load balancer
$lb = Get-AzLoadBalancer -Name LB-svcfab3 -ResourceGroupName svcfab_cli

# Create the rule based on information from the load balancer.
$lbrule = New-AzLoadBalancerRuleConfig -Name my-app-rule7 -Protocol Tcp -FrontendPort 39990 -BackendPort 40009 `
                                            -FrontendIpConfiguration $lb.FrontendIpConfigurations[0] `
                                            -BackendAddressPool  $lb.BackendAddressPools[0] `
                                            -Probe $lb.Probes[0]

# Add the rule to the load balancer
$lb.LoadBalancingRules.Add($lbrule)

# Update the load balancer on Azure
$lb | Set-AzLoadBalancer
```

En ce qui concerne la commande `New-AzLoadBalancerRuleConfig`, le `-FrontendPort` représente le port que l’équilibrage de charge expose aux connexions externes, et le `-BackendPort` représente le port d’écoute de l’application Service Fabric.

>[!NOTE]
>Pour plus d’informations sur la création d’un équilibreur de charge avec l’interface PowerShellI, consultez [Créer un équilibreur de charge avec PowerShell](../load-balancer/quickstart-load-balancer-standard-internal-powershell.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la [mise en réseau dans Service Fabric](service-fabric-patterns-networking.md).
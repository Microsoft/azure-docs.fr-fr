---
title: Paramètres DNS du Pare-feu Azure
description: Vous pouvez configurer Pare-feu Azure avec les paramètres de serveur DNS et de proxy DNS.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/06/2020
ms.author: victorh
ms.openlocfilehash: 197d48a2f5368111ec194a18f86aedf5ad78e1b2
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565617"
---
# <a name="azure-firewall-dns-settings"></a>Paramètres DNS du Pare-feu Azure

Vous pouvez configurer un serveur DNS personnalisé et activer le proxy DNS pour le pare-feu Azure. Configurez ces paramètres lorsque vous déployez le pare-feu ou configurez-les plus tard à partir de la page **Paramètres DNS**.

## <a name="dns-servers"></a>Serveurs DNS

Un serveur DNS gère et résout les noms de domaine en adresses IP. Par défaut, le Pare-feu Azure utilise Azure DNS pour la résolution de noms. Le paramètre **Serveur DNS** vous permet de configurer vos propres serveurs DNS pour la résolution de noms du Pare-feu Azure. Vous pouvez configurer un ou plusieurs serveurs.

> [!NOTE]
> Pour les instances Pare-feu Azure gérés à l’aide d’Azure Firewall Manager, les paramètres DNS sont configurés dans la stratégie Pare-feu Azure associée.

### <a name="configure-custom-dns-servers---azure-portal"></a>Configurer des serveurs DNS personnalisés – Portail Azure

1. Sous **Paramètres** du pare-feu Azure, sélectionnez **Paramètres DNS**.
2. Sous **Serveurs DNS**, vous pouvez saisir ou ajouter des serveurs DNS existants qui ont été précédemment spécifiés dans votre réseau virtuel.
3. Sélectionnez **Enregistrer**.

Le pare-feu dirige désormais le trafic DNS vers les serveurs DNS spécifiés pour la résolution de noms.

:::image type="content" source="media/dns-settings/dns-servers.png" alt-text="Capture d’écran montrant les paramètres des serveurs DNS.":::

### <a name="configure-custom-dns-servers---azure-cli"></a>Configurer des serveurs DNS personnalisés – Azure CLI

L’exemple suivant met à jour Pare-feu Azure avec des serveurs DNS personnalisés à l’aide de l’interface de ligne de commande Azure.

```azurecli-interactive
az network firewall update \
    --name fwName \ 
    --resource-group fwRG \
    --dns-servers 10.1.0.4 10.1.0.5
```

> [!IMPORTANT]
> La commande `az network firewall` nécessite l’installation de l’extension Azure CLI `azure-firewall` . Vous pouvez l’installer en utilisant la commande `az extension add --name azure-firewall`. 

### <a name="configure-custom-dns-servers---azure-powershell"></a>Configurer des serveurs DNS personnalisés – Azure PowerShell

L’exemple suivant met à jour Pare-feu Azure avec des serveurs DNS personnalisés à l’aide d’Azure PowerShell.

```azurepowershell
$dnsServers = @("10.1.0.4", "10.1.0.5")
$azFw = Get-AzFirewall -Name "fwName" -ResourceGroupName "fwRG"
$azFw.DNSServer = $dnsServers

$azFw | Set-AzFirewall
```

## <a name="dns-proxy"></a>Proxy DNS

Vous pouvez configurer le Pare-feu Azure pour qu’il agisse comme proxy DNS. Un proxy DNS est un intermédiaire pour les requêtes DNS entre des machines virtuelles clientes et un serveur DNS. Si vous configurez un serveur DNS personnalisé, activez le proxy DNS pour éviter les incompatibilités de résolution DNS et activez le filtrage de nom de domaine complet (FQDN) dans les règles de réseau.

Si vous n’activez pas le proxy DNS, les requêtes DNS du client peuvent se déplacer vers un serveur DNS à une heure différente ou retourner une réponse différente de celle du pare-feu. Le proxy DNS place le Pare-feu Azure sur le trajet des demandes du client pour éviter toute incohérence.

Lorsque Pare-feu Azure est un proxy DNS, deux types de fonctions de mise en cache sont possibles :

- **Cache positif** : la résolution DNS est réussie. Le pare-feu utilise la durée de vie (TTL, time to live) du paquet ou de l’objet. 

- **Cache négatif** : la résolution DNS ne produit aucune réponse ni aucune résolution. Le pare-feu met en cache ces informations pendant une heure.

Le proxy DNS stocke toutes les adresses IP résolues à partir de noms de domaine complets dans des règles de réseau. Nous vous recommandons d’utiliser des noms de domaine complets qui se résolvent en une seule adresse IP.  

### <a name="dns-proxy-configuration"></a>Configuration du proxy DNS

La configuration du proxy DNS nécessite trois étapes :
1. Activez le proxy DNS dans les paramètres DNS de Pare-feu Azure.
2. Si vous le souhaitez, configurez votre serveur DNS personnalisé ou utilisez la valeur par défaut fournie.
3. Configurez l’adresse IP privée de Pare-feu Azure en tant qu’adresse DNS personnalisée dans les paramètres du serveur DNS de votre réseau virtuel. Ce réglage garantit que le trafic DNS est dirigé vers Pare-feu Azure.

#### <a name="configure-dns-proxy---azure-portal"></a>Configurer le proxy DNS – Portail Azure

Pour configurer le proxy DNS, vous devez configurer le paramètre des serveurs DNS de votre réseau virtuel pour utiliser l’adresse IP privée du pare-feu. Activez ensuite le proxy DNS dans les **paramètres DNS** de Pare-feu Azure.

##### <a name="configure-virtual-network-dns-servers"></a>Configurer les serveurs DNS du réseau virtuel 

1. Sélectionnez le réseau virtuel dans lequel le trafic DNS sera routé via l’instance Pare-feu Azure.
2. Sous **Paramètres**, sélectionnez **Serveurs DNS**.
3. Sous **Serveurs DNS**, sélectionnez **Personnalisé**.
4. Saisissez l’adresse IP privée du pare-feu.
5. Sélectionnez **Enregistrer**.
6. Redémarrez les machines virtuelles connectées au réseau virtuel, afin que les nouveaux paramètres de serveur DNS leur soient attribués. Les machines virtuelles continuent d’utiliser leurs paramètres DNS actuels jusqu’à ce qu’elles soient redémarrées.

##### <a name="enable-dns-proxy"></a>Activer le proxy DNS

1. Sélectionnez votre instance Pare-feu Azure.
2. Sous **Paramètres**, sélectionnez **Paramètres DNS**.
3. Le **proxy DNS** est désactivé par défaut. Lorsque ce paramètre est activé, le pare-feu écoute le port 53 et transfère les requêtes DNS aux serveurs DNS configurés.
4. Vérifiez la configuration des **serveurs DNS** pour vous assurer que les paramètres sont adaptés à votre environnement.
5. Sélectionnez **Enregistrer**.

:::image type="content" source="media/dns-settings/dns-proxy.png" alt-text="Capture d’écran montrant les paramètres du proxy DNS.":::

#### <a name="configure-dns-proxy---azure-cli"></a>Configurer le proxy DNS – Azure CLI

Vous pouvez utiliser l’interface de ligne de commande Azure pour configurer les paramètres du proxy DNS dans Pare-feu Azure. Vous pouvez également l’utiliser pour mettre à jour des réseaux virtuels afin d’utiliser Pare-feu Azure comme serveur DNS.

##### <a name="configure-virtual-network-dns-servers"></a>Configurer les serveurs DNS du réseau virtuel

L’exemple suivant configure le réseau virtuel pour qu’il utilise Pare-feu Azure comme serveur DNS.
 
```azurecli-interactive
az network vnet update \
    --name VNetName \ 
    --resource-group VNetRG \
    --dns-servers <firewall-private-IP>
```

##### <a name="enable-dns-proxy"></a>Activer le proxy DNS

L’exemple suivant active la fonctionnalité de proxy DNS dans Pare-feu Azure.

```azurecli-interactive
az network firewall update \
    --name fwName \ 
    --resource-group fwRG \
    --enable-dns-proxy true
```

#### <a name="configure-dns-proxy---azure-powershell"></a>Configurer le proxy DNS – Azure PowerShell

Vous pouvez utiliser Azure PowerShell pour configurer les paramètres du proxy DNS dans Pare-feu Azure. Vous pouvez également l’utiliser pour mettre à jour des réseaux virtuels afin d’utiliser Pare-feu Azure comme serveur DNS.

##### <a name="configure-virtual-network-dns-servers"></a>Configurer les serveurs DNS du réseau virtuel

L’exemple suivant configure le réseau virtuel pour qu’il utilise Pare-feu Azure comme serveur DNS.

```azurepowershell
$dnsServers = @("<firewall-private-IP>")
$VNet = Get-AzVirtualNetwork -Name "VNetName" -ResourceGroupName "VNetRG"
$VNet.DhcpOptions.DnsServers = $dnsServers

$VNet | Set-AzVirtualNetwork
```

##### <a name="enable-dns-proxy"></a>Activer le proxy DNS

L’exemple suivant active la fonctionnalité de proxy DNS dans Pare-feu Azure.

```azurepowershell
$azFw = Get-AzFirewall -Name "fwName" -ResourceGroupName "fwRG"
$azFw.DNSEnableProxy = $true

$azFw | Set-AzFirewall
```

## <a name="next-steps"></a>Étapes suivantes

[Filtrage de nom de domaine complet dans les règles de réseau](fqdn-filtering-network-rules.md)

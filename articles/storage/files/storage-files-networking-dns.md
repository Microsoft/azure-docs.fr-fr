---
title: Configuration du transfert DNS pour Azure Files | Microsoft Docs
description: Découvrez comment configurer le transfert DNS pour Azure Files.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 3/19/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 9abe306668a4b20e42e45c498bf85b540dfaaee5
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94630190"
---
# <a name="configuring-dns-forwarding-for-azure-files"></a>Configuration du transfert DNS pour Azure Files
Azure Files vous permet de créer des points de terminaison privés pour les comptes de stockage contenant vos partages de fichiers. Bien qu’utiles pour une grande variété d’applications, les points de terminaison privés le sont particulièrement lorsque vous devez vous connecter à vos partages de fichiers Azure à partir de votre réseau local, à l’aide d’une connexion VPN ou ExpressRoute avec appairage privé. 

Pour que les connexions à votre compte de stockage passent par le tunnel réseau, le nom de domaine complet (FQDN) de votre compte de stockage doit être résolu en adresse IP privée de votre point de terminaison privé. Pour ce faire, vous devez transférer le suffixe du point de terminaison de stockage (`core.windows.net` pour les régions du cloud public) au service de DNS privé d’Azure, qui est accessible à partir de votre réseau virtuel. Ce guide explique comment configurer le transfert DNS pour résoudre correctement l’adresse IP du point de terminaison privé de votre compte de stockage.

Avant d’effectuer les étapes décrites dans cet article, nous vous recommandons fortement de lire [Planification d’un déploiement Azure Files](storage-files-planning.md) et [Considérations relatives aux réseaux Azure Files](storage-files-networking-overview.md).

## <a name="overview"></a>Vue d’ensemble
Azure Files fournit deux principaux types de points de terminaison pour accéder aux partages de fichiers Azure : 
- Les points de terminaison publics, qui ont une adresse IP publique et sont accessibles partout dans le monde.
- Les points de terminaison privés, qui existent au sein d’un réseau virtuel et ont une adresse IP privée comprise dans l’espace d’adressage de ce réseau virtuel.

Le compte de stockage Azure comprend à la fois des points de terminaison publics et des points de terminaison privés. Un compte de stockage est une construction de gestion qui représente un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente.

Chaque compte de stockage a un nom de domaine complet (FQDN). Pour les régions du cloud public, ce nom de domaine complet suit le modèle `storageaccount.file.core.windows.net`, où `storageaccount` correspond au nom du compte de stockage. Lorsque vous exécutez des requêtes sur ce nom (par exemple, pour monter le partage sur votre station de travail à l’aide de SMB), votre système d’exploitation effectue une recherche DNS pour résoudre le nom de domaine complet en une adresse IP qu’il peut utiliser pour l’envoi de requêtes SMB.

Par défaut, `storageaccount.file.core.windows.net` est résolu en l’adresse IP du point de terminaison public. Le point de terminaison public d’un compte de stockage est hébergé sur un cluster de stockage Azure qui héberge les points de terminaison publics de nombreux autres comptes de stockage. Lorsque vous créez un point de terminaison privé, une zone DNS privée est associée au réseau virtuel auquel elle a été ajoutée, et un enregistrement CNAME mappe `storageaccount.file.core.windows.net` à une entrée d’enregistrement A pour l’adresse IP privée du point de terminaison privé de votre compte de stockage. Cela vous permet d’utiliser le nom de domaine complet `storageaccount.file.core.windows.net` dans le réseau virtuel et de le résoudre en l’adresse IP du point de terminaison privé.

Étant donné que l’objectif ultime est d’accéder aux partages de fichiers Azure qui sont hébergés dans le compte de stockage à partir d’un emplacement local et à l’aide d’un tunnel réseau, tel qu’une connexion VPN ou ExpressRoute, vous devez configurer vos serveurs DNS locaux pour qu’ils transfèrent les requêtes envoyées au service Azure Files vers le service DNS privé Azure. Pour ce faire, vous devez configurer le *transfert conditionnel* de `*.core.windows.net` (ou le suffixe du point de terminaison de stockage approprié pour les clouds nationaux US Government, Allemagne ou Chine) sur un serveur DNS hébergé appartenant à votre réseau virtuel Azure. Ce serveur DNS transfère ensuite la requête de manière récursive au service DNS privé d’Azure, qui résoudra le nom de domaine complet du compte de stockage en l’adresse IP privée appropriée.

La configuration du transfert DNS pour Azure Files nécessite que vous exécutiez une machine virtuelle pour héberger un serveur DNS qui doit transférer les requêtes. Toutefois, vous n’aurez à effectuer cette étape qu’une seule fois pour tous les partages de fichiers Azure qui sont hébergés dans votre réseau virtuel. En outre, cette exigence n’est pas seulement celle d’Azure Files : tous les services Azure qui prennent en charge les points de terminaison privés auxquels vous souhaitez accéder à partir d’un site local peuvent utiliser le transfert DNS que vous allez configurer dans ce guide : stockage Blob Azure, SQL Azure, Cosmos DB, etc. 

Ce guide présente les étapes de configuration du transfert DNS pour le point de terminaison de stockage Azure. Ainsi, outre Azure Files, les requêtes de résolution de noms DNS pour tous les autres services de stockage Azure (stockage Blob Azure, stockage Table Azure, stockage File d’attente Azure, etc.) seront transférées vers le service DNS privé d’Azure. Si vous le souhaitez, vous pouvez également ajouter des points de terminaison supplémentaires pour d’autres services Azure. Le transfert DNS vers vos serveurs DNS locaux sera également configuré, ce qui permettra aux ressources cloud de votre réseau virtuel (par exemple, un serveur DFS-N) de résoudre les noms de machines locales. 

## <a name="prerequisites"></a>Prérequis
Avant de pouvoir configurer le transfert DNS vers Azure Files, vous devez disposer des éléments suivants :

- Un compte de stockage contenant le partage de fichiers Azure que vous souhaitez monter. Pour savoir comment créer un compte de stockage et un partage de fichiers Azure, consultez [Créer un partage de fichiers Azure](storage-how-to-create-file-share.md).
- Un point de terminaison privé pour le compte de stockage. Pour savoir comment créer un point de terminaison privé pour Azure Files, consultez [Créer un point de terminaison privé](storage-files-networking-endpoints.md#create-a-private-endpoint).
- La [dernière version](/powershell/azure/install-az-ps) du module Azure PowerShell

> [!Important]  
> Ce guide part du principe que vous utilisez le serveur DNS dans Windows Server, dans votre environnement local. Toutes les étapes décrites dans ce guide sont valables pour n’importe quel serveur DNS, et non uniquement le serveur DNS Windows.

## <a name="manually-configuring-dns-forwarding"></a>Configuration manuelle du transfert DNS
Si des serveurs DNS sont déjà en place dans votre réseau virtuel Azure, ou si vous préférez déployer vos propres machines virtuelles comme des serveurs DNS selon la méthodologie utilisée par votre organisation, vous pouvez configurer manuellement le DNS avec les applets de commande PowerShell qui sont intégrées au serveur DNS.

Sur vos serveurs DNS locaux, créez un redirecteur conditionnel à l’aide de `Add-DnsServerConditionalForwarderZone`. Ce redirecteur conditionnel doit être déployé sur tous vos serveurs DNS locaux afin de transférer correctement le trafic vers Azure. N’oubliez pas de remplacer `<azure-dns-server-ip>` par les adresses IP correspondant à votre environnement.

```powershell
$vnetDnsServers = "<azure-dns-server-ip>", "<azure-dns-server-ip>"

$storageAccountEndpoint = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty StorageEndpointSuffix

Add-DnsServerConditionalForwarderZone `
        -Name $storageAccountEndpoint `
        -MasterServers $vnetDnsServers
```

Sur les serveurs DNS de votre réseau virtuel Azure, vous devrez également mettre en place un redirecteur afin que les requêtes destinées à la zone DNS du compte de stockage soient dirigées vers le service DNS privé d’Azure, qui est accessible via l’adresse IP réservée `168.63.129.16` (n’oubliez pas de remplir `$storageAccountEndpoint` si vous exécutez les commandes dans une autre session PowerShell).

```powershell
Add-DnsServerConditionalForwarderZone `
        -Name $storageAccountEndpoint `
        -MasterServers "168.63.129.16"
```

## <a name="using-the-azure-files-hybrid-module-to-configure-dns-forwarding"></a>Utilisation du module Azure Files Hybrid pour configurer le transfert DNS
Pour que la configuration du transfert DNS soit aussi simple que possible, nous l’avons automatisée à l’aide du module Azure Files Hybrid. Les applets de commande fournies pour la manipulation du DNS dans ce module vous aideront à déployer des serveurs DNS dans votre réseau virtuel Azure et à mettre à jour vos serveurs DNS locaux en vue d’un transfert. 

Si vous n’avez jamais utilisé le module Azure Files Hybrid, vous devez d’abord l’installer sur votre station de travail. Téléchargez la [dernière version](https://github.com/Azure-Samples/azure-files-samples/releases) du module Azure PowerShell Azure Files Hybrid :

```PowerShell
# Unzip the downloaded file
Expand-Archive -Path AzFilesHybrid.zip

# Change the execution policy to unblock importing AzFilesHybrid.psm1 module
Set-ExecutionPolicy -ExecutionPolicy Unrestricted

# Navigate to where AzFilesHybrid is unzipped and stored and run to copy the files into your path
.\AzFilesHybrid\CopyToPSPath.ps1 

# Import AzFilesHybrid module
Import-Module -Name AzFilesHybrid
```

Le déploiement de la solution de transfert DNS comporte deux étapes : la création d’un ensemble de règles de transfert DNS qui définissent les services Azure auxquels vous souhaitez transférer les requêtes, et le déploiement des redirecteurs DNS. 

L’exemple suivant transfère les requêtes au compte de stockage, y compris les requêtes envoyées à Azure Files, au stockage Blob Azure, au stockage Table Azure et au stockage File d’attente Azure. Si vous le souhaitez, vous pouvez ajouter le transfert d’un autre service Azure à la règle via le paramètre `-AzureEndpoints` de l’applet de commande `New-AzDnsForwardingRuleSet`. N’oubliez pas de remplacer `<virtual-network-resource-group>`, `<virtual-network-name>` et `<subnet-name>` par les valeurs correspondant à votre environnement.

```PowerShell
# Create a rule set, which defines the forwarding rules
$ruleSet = New-AzDnsForwardingRuleSet -AzureEndpoints StorageAccountEndpoint

# Deploy and configure DNS forwarders
New-AzDnsForwarder `
        -DnsForwardingRuleSet $ruleSet `
        -VirtualNetworkResourceGroupName "<virtual-network-resource-group>" `
        -VirtualNetworkName "<virtual-network-name>" `
        -VirtualNetworkSubnetName "<subnet-name>"
```

Vous pouvez également trouver utile ou nécessaire de fournir plusieurs autres paramètres :

| Nom du paramètre | Type | Description |
|----------------|------|-------------|
| `DnsServerResourceGroupName` | `string` | Par défaut, les serveurs DNS seront déployés dans le même groupe de ressources que le réseau virtuel. Si ce n’est pas souhaité, ce paramètre vous permet de choisir un autre groupe de ressources pour leur déploiement. |
| `DnsForwarderRootName` | `string` | Par défaut, les serveurs DNS déployés dans Azure portent le nom de `DnsFwder-*`, où l’astérisque est remplacé par un itérateur. Ce paramètre modifie la racine du nom (c’est-à-dire, `DnsFwder`). |
| `VmTemporaryPassword` | `SecureString` | Par défaut, un mot de passe aléatoire est choisi pour le compte temporaire par défaut dont dispose une machine virtuelle avant de rejoindre le domaine. Une fois la machine jointe au domaine, le compte par défaut est désactivé. |
| `DomainToJoin` | `string` | Domaine auquel joindre la ou les machines virtuelles DNS. Par défaut, ce domaine est choisi en fonction du domaine de l’ordinateur sur lequel vous exécutez les applets de commande. |
| `DnsForwarderRedundancyCount` | `int` | Nombre de machines virtuelles DNS à déployer pour votre réseau virtuel. Par défaut, `New-AzDnsForwarder` déploie deux serveurs DNS sur votre réseau virtuel Azure dans un groupe à haute disponibilité afin de garantir la redondance. Ce nombre peut être modifié selon vos besoins. |
| `OnPremDnsHostNames` | `HashSet<string>` | Liste spécifiée manuellement répertoriant les noms d’hôtes DNS locaux sur lesquels créer des redirecteurs. Ce paramètre est utile lorsque vous ne souhaitez pas appliquer de redirecteurs sur tous les serveurs DNS locaux, par exemple, lorsque vous disposez d’une plage de clients avec des noms DNS spécifiés manuellement. |
| `Credential` | `PSCredential` | Informations d’identification à utiliser lors de la mise à jour des serveurs DNS. Utiles lorsque le compte d’utilisateur avec lequel vous vous êtes connecté ne dispose pas des autorisations nécessaires pour modifier les paramètres DNS. |
| `SkipParentDomain` | `SwitchParameter` | Par défaut, les redirecteurs DNS sont appliqués au domaine de votre environnement dont le niveau est le plus élevé. Par exemple, si `northamerica.corp.contoso.com` est un domaine enfant de `corp.contoso.com`, le redirecteur sera créé pour les serveurs DNS associés à `corp.contoso.com`. Ce paramètre entraînera la création des redirecteurs dans `northamerica.corp.contoso.com`. |

## <a name="confirm-dns-forwarders"></a>Vérifier les redirecteurs DNS
Avant de vérifier si les redirecteurs DNS ont été correctement appliqués, nous vous recommandons de vider le cache DNS de votre station de travail locale à l’aide de `Clear-DnsClientCache`. Pour vérifier si vous pouvez résoudre correctement le nom de domaine complet de votre compte de stockage, utilisez `Resolve-DnsName` ou `nslookup`.

```powershell
# Replace storageaccount.file.core.windows.net with the appropriate FQDN for your storage account.
# Note the proper suffix (core.windows.net) depends on the cloud your deployed in.
Resolve-DnsName -Name storageaccount.file.core.windows.net
```

Si la résolution de noms réussit, vous devez voir que l’adresse IP résolue correspond à l’adresse IP de votre compte de stockage.

```Output
Name                              Type   TTL   Section    NameHost
----                              ----   ---   -------    --------
storageaccount.file.core.windows. CNAME  29    Answer     csostoracct.privatelink.file.core.windows.net
net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 1769
Section    : Answer
IP4Address : 192.168.0.4
```

Si vous avez déjà configuré une connexion VPN ou ExpressRoute, vous pouvez également utiliser `Test-NetConnection` pour voir si une connexion TCP peut être établie avec votre compte de stockage.

```PowerShell
Test-NetConnection -ComputerName storageaccount.file.core.windows.net -CommonTCPPort SMB
```

## <a name="see-also"></a>Voir aussi
- [Planification d’un déploiement Azure Files](storage-files-planning.md)
- [Considérations relatives aux réseaux Azure Files](storage-files-networking-overview.md)
- [Configuration des points de terminaison réseau Azure Files](storage-files-networking-endpoints.md)
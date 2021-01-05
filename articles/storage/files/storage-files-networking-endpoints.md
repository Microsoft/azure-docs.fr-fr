---
title: Configuration des points de terminaison réseau Azure Files | Microsoft Docs
description: Découvrez comment configurer des points de terminaison réseau Azure File.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 12/04/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 079d7aa9b654a318c7269a41605c3e146b08f127
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96621329"
---
# <a name="configuring-azure-files-network-endpoints"></a>Configuration des points de terminaison réseau Azure Files

Azure Files fournit deux principaux types de points de terminaison pour accéder aux partages de fichiers Azure : 
- Les points de terminaison publics, qui ont une adresse IP publique et sont accessibles partout dans le monde.
- Les points de terminaison privés, qui existent au sein d’un réseau virtuel et ont une adresse IP privée comprise dans l’espace d’adressage de ce réseau virtuel.

Le compte de stockage Azure comprend à la fois des points de terminaison publics et des points de terminaison privés. Un compte de stockage est une construction de gestion qui représente un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente.

Cet article se concentre sur la configuration des points de terminaison d’un compte de stockage pour accéder directement au partage de fichiers Azure. La plupart des détails fournis dans ce document s’appliquent également à la façon dont Azure File Sync interagit avec les points de terminaison publics et privés du compte de stockage. Toutefois, pour plus d’informations sur la configuration des réseaux dans le cadre d’un déploiement Azure File Sync, consultez [Configuration des paramètres de proxy et de pare-feu d’Azure File Sync](storage-sync-files-firewall-and-proxy.md).

Avant de lire le présent guide, nous vous recommandons de lire [Considérations relatives aux réseaux Azure Files](storage-files-networking-overview.md).

## <a name="prerequisites"></a>Prérequis

- Cet article suppose que vous avez déjà créé un abonnement Azure. Si vous n’avez pas d’abonnement, vous pouvez [créer un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Cet article part du principe que vous avez déjà créé un partage de fichiers Azure dans un compte de stockage auquel vous souhaitez vous connecter à partir d’un emplacement local. Pour savoir comment créer un partage de fichiers Azure, consultez [Créer un partage de fichiers Azure](storage-how-to-create-file-share.md).
- Si vous envisagez d’utiliser Azure PowerShell, [installez-en la dernière version](/powershell/azure/install-az-ps).
- Si vous envisagez d’utiliser Azure CLI, [installez-en la dernière version](/cli/azure/install-azure-cli).

## <a name="endpoint-configurations"></a>Configurations de point de terminaison

Vous pouvez configurer vos points de terminaison pour limiter l’accès réseau à votre compte de stockage. Il existe deux approches pour limiter l’accès d’un compte de stockage à un réseau virtuel :

- [Créer un ou plusieurs points de terminaison privés pour le compte de stockage](#create-a-private-endpoint) et limiter tous les accès au point de terminaison public. Seul le trafic en provenance des réseaux virtuels souhaités accède alors aux partages de fichiers Azure du compte de stockage.
- [Limiter le point de terminaison public à un ou plusieurs réseaux virtuels](#restrict-public-endpoint-access). Cette approche s’appuie sur une fonctionnalité du réseau virtuel appelée *points de terminaison de service*. Quand vous limitez le trafic à un compte de stockage via un point de terminaison de service, vous continuez d’accéder au compte de stockage via l’adresse IP publique, mais l’accès n’est possible qu’à partir des emplacements que vous spécifiez dans votre configuration.

### <a name="create-a-private-endpoint"></a>Créer un Private Endpoint

La création d’un point de terminaison privé pour votre compte de stockage entraîne le déploiement des ressources Azure suivantes :

- **Un point de terminaison privé** : ressource Azure représentant le point de terminaison privé du compte de stockage. Vous pouvez le voir comme une ressource qui connecte un compte de stockage à une interface réseau.
- **Une interface réseau (NIC)**  : interface réseau qui gère une adresse IP privée au sein du réseau virtuel ou du sous-réseau spécifié. Il s’agit de la même ressource que celle déployée lors du déploiement d’une machine virtuelle. Toutefois, au lieu d’être attribuée à une machine virtuelle, elle est détenue par le point de terminaison privé.
- **Une zone DNS privée** : si vous n’avez jamais déployé de point de terminaison privé pour ce réseau virtuel, une nouvelle zone DNS privée sera déployée pour votre réseau virtuel. Un enregistrement DNS A est également créé pour le compte de stockage dans cette zone DNS. Si vous avez déjà déployé un point de terminaison privé dans ce réseau virtuel, un nouvel enregistrement A sera ajouté à la zone DNS existante pour le compte de stockage. Le déploiement d’une zone DNS est facultatif, mais fortement recommandé. Il peut être obligatoire si vous montez vos partages de fichiers Azure avec un principal de service Active Directory ou à l’aide de l’API FileREST.

> [!Note]  
> Cet article utilise le suffixe DNS de compte de stockage pour les régions publiques Azure, `core.windows.net`. Ce commentaire vaut aussi pour les clouds souverains Azure que sont notamment le cloud Azure US Government et le cloud Azure Chine (il vous suffit de remplacer les suffixes appropriés pour votre environnement). 

# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-private-portal](../../../includes/storage-files-networking-endpoints-private-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-private-powershell](../../../includes/storage-files-networking-endpoints-private-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-private-cli](../../../includes/storage-files-networking-endpoints-private-cli.md)]
---

## <a name="verify-connectivity"></a>Vérifier la connectivité

# <a name="portal"></a>[Portail](#tab/azure-portal)

Si vous disposez d’une machine virtuelle dans votre réseau virtuel, ou si vous avez configuré le transfert DNS comme décrit dans [Configuration du transfert DNS pour Azure Files](storage-files-networking-dns.md), vous pouvez vérifier que votre point de terminaison privé a été correctement configuré en exécutant les commandes suivantes à partir de PowerShell, de la ligne de commande ou du terminal (fonctionne sur Windows, Linux et macOS). Vous devez remplacer `<storage-account-name>` par le nom du compte de stockage approprié :

```
nslookup <storage-account-name>.file.core.windows.net
```

Si tout a fonctionné correctement, vous devriez voir la sortie suivante, où `192.168.0.5` correspond à l’adresse IP privée du point de terminaison privé de votre réseau virtuel (sortie affichée pour Windows) :

```Output
Server:  UnKnown
Address:  10.2.4.4

Non-authoritative answer:
Name:    storageaccount.privatelink.file.core.windows.net
Address:  192.168.0.5
Aliases:  storageaccount.file.core.windows.net
```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si vous disposez d’une machine virtuelle dans votre réseau virtuel, ou si vous avez configuré le transfert DNS comme décrit dans [Configuration du transfert DNS pour Azure Files](storage-files-networking-dns.md), vous pouvez vérifier que votre point de terminaison privé a été correctement configuré en exécutant les commandes suivantes :

```PowerShell
$storageAccountHostName = [System.Uri]::new($storageAccount.PrimaryEndpoints.file) | `
    Select-Object -ExpandProperty Host

Resolve-DnsName -Name $storageAccountHostName
```

Si tout a fonctionné correctement, vous devriez voir la sortie suivante, où `192.168.0.5` correspond à l’adresse IP privée du point de terminaison privé de votre réseau virtuel :

```Output
Name                             Type   TTL   Section    NameHost
----                             ----   ---   -------    --------
storageaccount.file.core.windows CNAME  60    Answer     storageaccount.privatelink.file.core.windows.net
.net

Name       : storageaccount.privatelink.file.core.windows.net
QueryType  : A
TTL        : 600
Section    : Answer
IP4Address : 192.168.0.5
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si vous disposez d’une machine virtuelle dans votre réseau virtuel, ou si vous avez configuré le transfert DNS comme décrit dans [Configuration du transfert DNS pour Azure Files](storage-files-networking-dns.md), vous pouvez vérifier que votre point de terminaison privé a été correctement configuré en exécutant les commandes suivantes :

```bash
httpEndpoint=$(az storage account show \
        --resource-group $storageAccountResourceGroupName \
        --name $storageAccountName \
        --query "primaryEndpoints.file" | \
    tr -d '"')

hostName=$(echo $httpEndpoint | cut -c7-$(expr length $httpEndpoint) | tr -d "/")
nslookup $hostName
```

Si tout a fonctionné correctement, vous devriez voir la sortie suivante, où `192.168.0.5` correspond à l’adresse IP privée du point de terminaison privé de votre réseau virtuel. Vous devez toujours utiliser storageaccount.file.core.windows.net pour monter votre partage de fichiers au lieu du chemin d’accès `privatelink`.

```Output
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
storageaccount.file.core.windows.net      canonical name = storageaccount.privatelink.file.core.windows.net.
Name:   storageaccount.privatelink.file.core.windows.net
Address: 192.168.0.5
```
---

## <a name="restrict-public-endpoint-access"></a>Restreindre l’accès au point de terminaison public

Pour limiter l’accès au point de terminaison public, vous devez commencer par désactiver l’accès général au point de terminaison public. La désactivation de l’accès au point de terminaison public n’affecte pas les points de terminaison privés. Une fois le point de terminaison public désactivé, vous pouvez sélectionner des réseaux ou adresses IP spécifiques qui peuvent continuer à y accéder. En général, la plupart des stratégies de pare-feu d’un compte de stockage limitent l’accès réseau à un ou plusieurs réseaux virtuels.

#### <a name="disable-access-to-the-public-endpoint"></a>Désactiver l’accès au point de terminaison public

Lorsque l’accès au point de terminaison public est désactivé, le compte de stockage reste accessible via ses points de terminaison privés. Si ce n’est pas le cas, les requêtes valides envoyées au point de terminaison public du compte de stockage seront rejetées, à moins qu’elles ne proviennent d’une [source spécifiquement autorisée](#restrict-access-to-the-public-endpoint-to-specific-virtual-networks). 

# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-disable-portal](../../../includes/storage-files-networking-endpoints-public-disable-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-disable-powershell](../../../includes/storage-files-networking-endpoints-public-disable-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-disable-cli](../../../includes/storage-files-networking-endpoints-public-disable-cli.md)]

---

#### <a name="restrict-access-to-the-public-endpoint-to-specific-virtual-networks"></a>Limiter l’accès au point de terminaison public à certains réseaux virtuels

Lorsque vous limitez l’accès au compte de stockage à certains réseaux virtuels, vous autorisez les requêtes à être envoyées au point de terminaison public à partir des réseaux virtuels spécifiés. Cette approche s’appuie sur une fonctionnalité du réseau virtuel appelée *points de terminaison de service*. Cela peut être utilisé avec ou sans points de terminaison privés.

# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-portal](../../../includes/storage-files-networking-endpoints-public-restrict-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-powershell](../../../includes/storage-files-networking-endpoints-public-restrict-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-cli](../../../includes/storage-files-networking-endpoints-public-restrict-cli.md)]

---

## <a name="see-also"></a>Voir aussi

- [Considérations relatives aux réseaux Azure Files](storage-files-networking-overview.md)
- [Configuration du transfert DNS pour Azure Files](storage-files-networking-dns.md)
- [Configuration de S2S VPN pour Azure Files](storage-files-configure-s2s-vpn.md)
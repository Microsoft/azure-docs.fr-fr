---
title: Configuration des points de terminaison réseau Azure File Sync | Microsoft Docs
description: Découvrez comment configurer des points de terminaison réseau Azure File Sync.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 5/11/2020
ms.author: rogarana
ms.subservice: files
ms.custom: devx-track-azurepowershell, devx-track-azurecli
ms.openlocfilehash: 61ff5d05eb74804af69b90d839115a8468619275
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921715"
---
# <a name="configuring-azure-file-sync-network-endpoints"></a>Configuration des points de terminaison réseau Azure File Sync
Azure Files et Azure File Sync fournissent deux principaux types de points de terminaison pour l’accès aux partages de fichiers Azure : 
- Les points de terminaison publics, qui ont une adresse IP publique et sont accessibles partout dans le monde.
- Les points de terminaison privés, qui existent au sein d’un réseau virtuel et ont une adresse IP privée comprise dans l’espace d’adressage de ce réseau virtuel.

Pour Azure Files et Azure File Sync, les objets de gestion Azure, le compte de stockage et le service de synchronisation du stockage, respectivement, contrôlent les points de terminaison publics et privés. Le compte de stockage est une construction de gestion qui représente un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente. Le service de synchronisation de stockage est une construction de gestion qui représente les serveurs inscrits, qui sont des serveurs de fichiers Windows avec une relation d’approbation établie avec Azure File Sync, ainsi que des groupes de synchronisation, qui définissent la topologie de la relation de synchronisation. 

Cet article se concentre sur la configuration des points de terminaison réseau pour Azure Files et Azure File Sync. Pour en savoir plus sur la configuration des points de terminaison réseau pour accéder directement aux partages de fichiers Azure, plutôt que d’effectuer une mise en cache locale avec Azure File Sync, consultez [Configuration des points de terminaison réseau Azure Files](storage-files-networking-endpoints.md).

Avant de lire le présent guide, nous vous recommandons de lire [Considérations relatives aux réseaux Azure File Sync](storage-sync-files-networking-overview.md).

## <a name="prerequisites"></a>Prérequis 
Cet article suppose que vous avez :
- Vous avez un abonnement Azure. Si vous n’avez pas d’abonnement, vous pouvez [créer un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
- Vous avez déjà créé un partage de fichiers Azure dans un compte de stockage auquel vous souhaitez vous connecter à partir d’un emplacement local. Pour savoir comment créer un partage de fichiers Azure, consultez [Créer un partage de fichiers Azure](storage-how-to-create-file-share.md).
- Vous avez déjà créé un service de synchronisation de stockage et vous l’avez inscrit auprès de votre serveur de fichiers Windows. Pour savoir comment déployer Azure File Sync, consultez [Déploiement d’Azure File Sync](storage-sync-files-deployment-guide.md).

De plus :
- Si vous envisagez d’utiliser Azure PowerShell, [installez-en la dernière version](/powershell/azure/install-az-ps).
- Si vous envisagez d’utiliser Azure CLI, [installez-en la dernière version](/cli/azure/install-azure-cli?view=azure-cli-latest&preserve-view=true).

## <a name="create-the-private-endpoints"></a>Créer les points de terminaison privés
Lorsque vous créez un point de terminaison privé pour une ressource Azure, les ressources suivantes sont déployées :

- **Un point de terminaison privé** : Ressource Azure représentant le point de terminaison privé pour le compte de stockage ou le service de synchronisation de stockage. Vous pouvez la voir comme une ressource qui connecte votre ressource Azure à une interface réseau.
- **Une interface réseau (NIC)**  : interface réseau qui gère une adresse IP privée au sein du réseau virtuel ou du sous-réseau spécifié. Il s’agit de la même ressource que celle déployée lors du déploiement d’une machine virtuelle. Toutefois, au lieu d’être attribuée à une machine virtuelle, elle est détenue par le point de terminaison privé.
- **Une zone DNS privée** : si vous n’avez jamais déployé de point de terminaison privé pour ce réseau virtuel, une nouvelle zone DNS privée sera déployée pour votre réseau virtuel. Un enregistrement DNS A est également créé pour la ressource Azure dans cette zone DNS. Si vous avez déjà déployé un point de terminaison privé dans ce réseau virtuel, un nouvel enregistrement A est ajouté à la zone DNS existante pour la ressource Azure. Le déploiement d’une zone DNS est facultatif, mais il est fortement recommandé pour simplifier la gestion DNS requise.

> [!Note]  
> Cet article utilise les suffixes DNS pour les régions publiques Azure, `core.windows.net` pour les comptes de stockage et `afs.azure.net` pour les services de synchronisation de stockage. Ce commentaire vaut aussi pour les clouds souverains Azure que sont notamment le cloud Azure US Government (il vous suffit de remplacer les suffixes appropriés pour votre environnement).

### <a name="create-the-storage-account-private-endpoint"></a>Créer le point de terminaison privé du compte de stockage
# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-private-portal](../../../includes/storage-files-networking-endpoints-private-portal.md)]

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
[!INCLUDE [storage-files-networking-endpoints-private-powershell](../../../includes/storage-files-networking-endpoints-private-powershell.md)]

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
[!INCLUDE [storage-files-networking-endpoints-private-cli](../../../includes/storage-files-networking-endpoints-private-cli.md)]

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

Si tout a fonctionné correctement, vous devriez voir la sortie suivante, où `192.168.0.5` correspond à l’adresse IP privée du point de terminaison privé de votre réseau virtuel :

```Output
Server:         127.0.0.53
Address:        127.0.0.53#53

Non-authoritative answer:
storageaccount.file.core.windows.net      canonical name = storageaccount.privatelink.file.core.windows.net.
Name:   storageaccount.privatelink.file.core.windows.net
Address: 192.168.0.5
```

---

### <a name="create-the-storage-sync-private-endpoint"></a>Créer le point de terminaison privé de synchronisation de stockage
> [!Important]  
> Pour pouvoir utiliser des points de terminaison privés sur la ressource de service de synchronisation de stockage, vous devez utiliser l’agent Azure File Sync version 10.1 ou ultérieure. Les versions d’agent antérieures à 10.1 ne prennent pas en charge les points de terminaison privés sur le service de synchronisation de stockage. Toutes les versions antérieures de l’agent prennent en charge les points de terminaison privés sur la ressource du compte de stockage.

# <a name="portal"></a>[Portail](#tab/azure-portal)
Accédez au **Centre de liaisons privées** en tapant *Liaison privée* dans la barre de recherche en haut du portail Azure. Dans la table des matières du Centre de liaisons privées, sélectionnez **Points de terminaison privés**, puis **+ Ajouter** pour créer un point de terminaison privé.

[![Capture d’écran du Centre de liaisons privées](media/storage-sync-files-networking-endpoints/create-storage-sync-private-endpoint-0.png)](media/storage-sync-files-networking-endpoints/create-storage-sync-private-endpoint-0.png#lightbox)

L’Assistant obtenu comprend plusieurs pages dont vous devez suivre les instructions.

Dans le panneau **De base**, sélectionnez le groupe de ressources, le nom et la région souhaités pour votre point de terminaison privé. Vous pouvez choisir ceux que vous voulez. Ils ne doivent pas nécessairement correspondre au service de synchronisation du stockage. Vous devez simplement créer le point de terminaison privé dans la même région que le réseau virtuel dans lequel vous souhaitez créer le point de terminaison privé.

![Capture d’écran de la section De base de la fenêtre Créer un point de terminaison privé](media/storage-sync-files-networking-endpoints/create-storage-sync-private-endpoint-1.png)

Dans le panneau **Ressource**, activez la case d’option **Se connecter à une ressource Azure de mon annuaire**. Sous **Type de ressource**, sélectionnez **Microsoft.StorageSync/storageSyncServices** pour le type de ressource. 

Le panneau **Configuration** vous permet de sélectionner le réseau virtuel et le sous-réseau auxquels vous souhaitez ajouter votre point de terminaison privé. Sélectionnez le même réseau virtuel que celui que vous avez utilisé pour le compte de stockage ci-dessus. Le panneau Configuration contient également les informations relatives à la création et à la mise à jour de la zone DNS privée.

Cliquez sur **Vérifier + créer** pour créer le point de terminaison privé.

Vous pouvez vérifier que votre point de terminaison privé a été correctement configuré en exécutant les commandes suivantes à partir de PowerShell. 

```powershell
$privateEndpointResourceGroupName = "<your-private-endpoint-resource-group>"
$privateEndpointName = "<your-private-endpoint-name>"

Get-AzPrivateEndpoint `
        -ResourceGroupName $privateEndpointResourceGroupName `
        -Name $privateEndpointName `
        -ErrorAction Stop | `
    Select-Object -ExpandProperty NetworkInterfaces | `
    Select-Object -ExpandProperty Id | `
    ForEach-Object { Get-AzNetworkInterface -ResourceId $_ } | `
    Select-Object -ExpandProperty IpConfigurations | `
    Select-Object -ExpandProperty PrivateLinkConnectionProperties | `
    Select-Object -ExpandProperty Fqdns | `
    ForEach-Object { Resolve-DnsName -Name $_ } | `
    Format-List
```

Si tout fonctionne correctement, vous devez voir la sortie suivante, où `192.168.1.4`, `192.168.1.5`, `192.168.1.6` et `192.168.1.7` sont les adresses IP privées attribuées au point de terminaison privé :

```Output
Name     : mysssmanagement.westus2.afs.azure.net
Type     : CNAME
TTL      : 60
Section  : Answer
NameHost : mysssmanagement.westus2.privatelink.afs.azure.net


Name       : mysssmanagement.westus2.privatelink.afs.azure.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 192.168.1.4

Name     : myssssyncp.westus2.afs.azure.net
Type     : CNAME
TTL      : 60
Section  : Answer
NameHost : myssssyncp.westus2.privatelink.afs.azure.net


Name       : myssssyncp.westus2.privatelink.afs.azure.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 192.168.1.5

Name     : myssssyncs.westus2.afs.azure.net
Type     : CNAME
TTL      : 60
Section  : Answer
NameHost : myssssyncs.westus2.privatelink.afs.azure.net


Name       : myssssyncs.westus2.privatelink.afs.azure.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 192.168.1.6

Name     : mysssmonitoring.westus2.afs.azure.net
Type     : CNAME
TTL      : 60
Section  : Answer
NameHost : mysssmonitoring.westus2.privatelink.afs.azure.net


Name       : mysssmonitoring.westus2.privatelink.afs.azure.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 192.168.1.7

```

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour créer un point de terminaison privé pour votre service de synchronisation du stockage, vous devez d’abord obtenir une référence à ce dernier. Pensez à remplacer `<storage-sync-service-resource-group>` et `<storage-sync-service>` par les valeurs correctes de votre environnement. Les commandes PowerShell suivantes supposent que vous utilisez déjà les informations de réseau virtuel ci-dessus. 

```powershell
$storageSyncServiceResourceGroupName = "<storage-sync-service-resource-group>"
$storageSyncServiceName = "<storage-sync-service>"

$storageSyncService = Get-AzStorageSyncService `
        -ResourceGroupName $storageSyncServiceResourceGroupName `
        -Name $storageSyncServiceName `
        -ErrorAction SilentlyContinue

if ($null -eq $storageSyncService) {
    $errorMessage = "Storage Sync Service $storageSyncServiceName not found "
    $errorMessage += "in resource group $storageSyncServiceResourceGroupName."
    Write-Error -Message $errorMessage -ErrorAction Stop
}
```

Pour créer un point de terminaison privé, vous devez créer une connexion de service sous forme de liaison privée au service de synchronisation du stockage. La connexion sous forme de liaison privée est une entrée pour la création du point de terminaison privé.

```PowerShell 
# Disable private endpoint network policies
$subnet.PrivateEndpointNetworkPolicies = "Disabled"
$virtualNetwork = $virtualNetwork | `
    Set-AzVirtualNetwork -ErrorAction Stop

# Create a private link service connection to the storage account.
$privateEndpointConnection = New-AzPrivateLinkServiceConnection `
        -Name "$storageSyncServiceName-Connection" `
        -PrivateLinkServiceId $storageSyncService.ResourceId `
        -GroupId "Afs" `
        -ErrorAction Stop

# Create a new private endpoint.
$privateEndpoint = New-AzPrivateEndpoint `
        -ResourceGroupName $storageSyncServiceResourceGroupName `
        -Name "$storageSyncServiceName-PrivateEndpoint" `
        -Location $virtualNetwork.Location `
        -Subnet $subnet `
        -PrivateLinkServiceConnection $privateEndpointConnection `
        -ErrorAction Stop
```

La création d’une zone DNS privée Azure permet de faire correspondre les noms d’hôte du service de synchronisation du stockage, tels que `mysssmanagement.westus2.afs.azure.net`, aux adresses IP privées appropriées pour le service de synchronisation du stockage dans le réseau virtuel. Même si cela est facultatif du point de vue de la création d’un point de terminaison privé, cela est explicitement requis pour que l’agent Azure File Sync puisse accéder au service de synchronisation du stockage. 

```powershell
# Get the desired Storage Sync Service suffix (afs.azure.net for public cloud).
# This is done like this so this script will seamlessly work for non-public Azure.
$azureEnvironment = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty Name

switch($azureEnvironment) {
    "AzureCloud" {
        $storageSyncSuffix = "afs.azure.net"
    }

    "AzureUSGovernment" {
        $storageSyncSuffix = "afs.azure.us"
    }
    
    default {
        Write-Error 
                -Message "The Azure environment $_ is not currently supported by Azure File Sync." `
                -ErrorAction Stop
    }
}

# For public cloud, this will generate the following DNS suffix:
# privatelink.afs.azure.net
$dnsZoneName = "privatelink.$storageSyncSuffix"

# Find a DNS zone matching desired name attached to this virtual network.
$dnsZone = Get-AzPrivateDnsZone | `
    Where-Object { $_.Name -eq $dnsZoneName } | `
    Where-Object {
        $privateDnsLink = Get-AzPrivateDnsVirtualNetworkLink `
                -ResourceGroupName $_.ResourceGroupName `
                -ZoneName $_.Name `
                -ErrorAction SilentlyContinue
        
        $privateDnsLink.VirtualNetworkId -eq $virtualNetwork.Id
    }

if ($null -eq $dnsZone) {
    # No matching DNS zone attached to virtual network, so create new one.
    $dnsZone = New-AzPrivateDnsZone `
            -ResourceGroupName $virtualNetworkResourceGroupName `
            -Name $dnsZoneName `
            -ErrorAction Stop

    $privateDnsLink = New-AzPrivateDnsVirtualNetworkLink `
            -ResourceGroupName $virtualNetworkResourceGroupName `
            -ZoneName $dnsZoneName `
            -Name "$virtualNetworkName-DnsLink" `
            -VirtualNetworkId $virtualNetwork.Id `
            -ErrorAction Stop
}
```
Maintenant que vous disposez d’une référence à la zone DNS privée, vous devez créer un enregistrement A pour votre service de synchronisation du stockage.

```PowerShell 
$privateEndpointIpFqdnMappings = $privateEndpoint | `
    Select-Object -ExpandProperty NetworkInterfaces | `
    Select-Object -ExpandProperty Id | `
    ForEach-Object { Get-AzNetworkInterface -ResourceId $_ } | `
    Select-Object -ExpandProperty IpConfigurations | `
    ForEach-Object { 
        $privateIpAddress = $_.PrivateIpAddress; 
        $_ | `
            Select-Object -ExpandProperty PrivateLinkConnectionProperties | `
            Select-Object -ExpandProperty Fqdns | `
            Select-Object `
                @{ 
                    Name = "PrivateIpAddress"; 
                    Expression = { $privateIpAddress } 
                }, `
                @{ 
                    Name = "FQDN"; 
                    Expression = { $_ } 
                } 
    }

foreach($ipFqdn in $privateEndpointIpFqdnMappings) {
    $privateDnsRecordConfig = New-AzPrivateDnsRecordConfig `
        -IPv4Address $ipFqdn.PrivateIpAddress
    
    $dnsEntry = $ipFqdn.FQDN.Substring(0, 
        $ipFqdn.FQDN.IndexOf(".", $ipFqdn.FQDN.IndexOf(".") + 1))

    New-AzPrivateDnsRecordSet `
            -ResourceGroupName $virtualNetworkResourceGroupName `
            -Name $dnsEntry `
            -RecordType A `
            -ZoneName $dnsZoneName `
            -Ttl 600 `
            -PrivateDnsRecords $privateDnsRecordConfig `
            -ErrorAction Stop | `
        Out-Null
}
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
Pour créer un point de terminaison privé pour votre service de synchronisation du stockage, vous devez d’abord obtenir une référence à ce dernier. Pensez à remplacer `<storage-sync-service-resource-group>` et `<storage-sync-service>` par les valeurs correctes de votre environnement. Les commandes CLI suivantes supposent que vous utilisez déjà les informations de réseau virtuel ci-dessus. 

```bash
storageSyncServiceResourceGroupName="<storage-sync-service-resource-group>"
storageSyncServiceName="<storage-sync-service>"

storageSyncService=$(az resource show \
        --resource-group $storageSyncServiceResourceGroupName \
        --name $storageSyncServiceName \
        --resource-type "Microsoft.StorageSync/storageSyncServices" \
        --query "id" | \
    tr -d '"')

storageSyncServiceRegion=$(az resource show \
        --resource-group $storageSyncServiceResourceGroupName \
        --name $storageSyncServiceName \
        --resource-type "Microsoft.StorageSync/storageSyncServices" \
        --query "location" | \
    tr -d '"')
```

Pour créer un point de terminaison privé, vous devez d’abord vérifier que la stratégie réseau du point de terminaison privé du sous-réseau est définie sur Désactivée. Vous pouvez ensuite créer un point de terminaison privé avec la commande `az network private-endpoint create`.

```bash
# Disable private endpoint network policies
az network vnet subnet update \
        --ids $subnet \
        --disable-private-endpoint-network-policies \
        --output none

# Get virtual network location
region=$(az network vnet show \
        --ids $virtualNetwork \
        --query "location" | \
    tr -d '"')

# Create a private endpoint
privateEndpoint=$(az network private-endpoint create \
        --resource-group $storageSyncServiceResourceGroupName \
        --name "$storageSyncServiceName-PrivateEndpoint" \
        --location $region \
        --subnet $subnet \
        --private-connection-resource-id $storageSyncService \
        --group-id "Afs" \
        --connection-name "$storageSyncServiceName-Connection" \
        --query "id" | \
    tr -d '"')
```

La création d’une zone DNS privée Azure permet de faire correspondre les noms d’hôte du service de synchronisation du stockage, tels que `mysssmanagement.westus2.afs.azure.net`, aux adresses IP privées appropriées pour le service de synchronisation du stockage dans le réseau virtuel. Même si cela est facultatif du point de vue de la création d’un point de terminaison privé, cela est explicitement requis pour que l’agent Azure File Sync puisse accéder au service de synchronisation du stockage. 

```bash
# Get the desired storage account suffix (afs.azure.net for public cloud).
# This is done like this so this script will seamlessly work for non-public Azure.
azureEnvironment=$(az cloud show \
        --query "name" |
    tr -d '"')

storageSyncSuffix=""
if [ $azureEnvironment == "AzureCloud" ]
then
    storageSyncSuffix="afs.azure.net"
elif [ $azureEnvironment == "AzureUSGovernment" ]
then
    storageSyncSuffix="afs.azure.us"
else
    echo "Unsupported Azure environment $azureEnvironment."
fi

# For public cloud, this will generate the following DNS suffix:
# privatelinke.afs.azure.net.
dnsZoneName="privatelink.$storageSyncSuffix"

# Find a DNS zone matching desired name attached to this virtual network.
possibleDnsZones=""
possibleDnsZones=$(az network private-dns zone list \
        --query "[?name == '$dnsZoneName'].id" \
        --output tsv)

dnsZone=""
possibleDnsZone=""
for possibleDnsZone in $possibleDnsZones
do
    possibleResourceGroupName=$(az resource show \
            --ids $possibleDnsZone \
            --query "resourceGroup" | \
        tr -d '"')
    
    link=$(az network private-dns link vnet list \
            --resource-group $possibleResourceGroupName \
            --zone-name $dnsZoneName \
            --query "[?virtualNetwork.id == '$virtualNetwork'].id" \
            --output tsv)
    
    if [ -z $link ]
    then
        echo "1" > /dev/null
    else 
        dnsZoneResourceGroup=$possibleResourceGroupName
        dnsZone=$possibleDnsZone
        break
    fi  
done

if [ -z $dnsZone ]
then
    # No matching DNS zone attached to virtual network, so create a new one
    dnsZone=$(az network private-dns zone create \
            --resource-group $virtualNetworkResourceGroupName \
            --name $dnsZoneName \
            --query "id" | \
        tr -d '"')
    
    az network private-dns link vnet create \
            --resource-group $virtualNetworkResourceGroupName \
            --zone-name $dnsZoneName \
            --name "$virtualNetworkName-DnsLink" \
            --virtual-network $virtualNetwork \
            --registration-enabled false \
            --output none
    
    dnsZoneResourceGroup=$virtualNetworkResourceGroupName
fi
```

Maintenant que vous disposez d’une référence à la zone DNS privée, vous devez créer un enregistrement A pour votre service de synchronisation du stockage.

```bash
privateEndpointNIC=$(az network private-endpoint show \
        --ids $privateEndpoint \
        --query "networkInterfaces[0].id" | \
    tr -d '"')

privateIpAddresses=$(az network nic show \
        --ids $privateEndpointNIC \
        --query "ipConfigurations[].privateIpAddress" \
        --output tsv) 

hostNames=$(az network nic show \
        --ids $privateEndpointNIC \
        --query "ipConfigurations[].privateLinkConnectionProperties.fqdns[]" \
        --output tsv)

i=0
for privateIpAddress in $privateIpAddresses
do
    j=0
    targetHostName=""
    for hostName in $hostNames
    do
        if [ $i == $j ]
        then
            targetHostName=$hostName
            break
        fi

        j=$(expr $j + 1)
    done

    endpointName=$(echo $targetHostName | \
        cut -c1-$(expr $(expr index $targetHostName ".") - 1))

    az network private-dns record-set a create \
        --resource-group $dnsZoneResourceGroup \
        --zone-name $dnsZoneName \
        --name "$endpointName.$storageSyncServiceRegion" \
        --output none
    
    az network private-dns record-set a add-record \
        --resource-group $dnsZoneResourceGroup \
        --zone-name $dnsZoneName \
        --record-set-name "$endpointName.$storageSyncServiceRegion" \
        --ipv4-address $privateIpAddress \
        --output none

    i=$(expr $i + 1)
done
```
---

## <a name="restrict-access-to-the-public-endpoints"></a>Limiter l’accès aux points de terminaison publics
Vous pouvez restreindre l’accès aux points de terminaison publics du compte de stockage et du service de synchronisation du stockage. Restreindre l’accès au point de terminaison public fournit une sécurité supplémentaire en garantissant que les paquets réseau sont uniquement acceptés à partir des emplacements approuvés. 

### <a name="restrict-access-to-the-storage-account-public-endpoint"></a>Restreindre l’accès au point de terminaison public du compte de stockage
Les restrictions d’accès au point de terminaison public s’effectuent à l’aide des paramètres de pare-feu du compte de stockage. En général, la plupart des stratégies de pare-feu d’un compte de stockage limitent l’accès réseau à un ou plusieurs réseaux virtuels. Il existe deux approches pour limiter l’accès d’un compte de stockage à un réseau virtuel :

- [Créer un ou plusieurs points de terminaison privés pour le compte de stockage](#create-the-storage-account-private-endpoint) et désactiver l’accès au point de terminaison public. Seul le trafic en provenance des réseaux virtuels souhaités accède alors aux partages de fichiers Azure du compte de stockage.
- Limiter le point de terminaison public à un ou plusieurs réseaux virtuels. Cette approche s’appuie sur une fonctionnalité du réseau virtuel appelée *points de terminaison de service*. Quand vous limitez le trafic à un compte de stockage via un point de terminaison de service, vous continuez d’accéder au compte de stockage via l’adresse IP publique.

#### <a name="disable-access-to-the-storage-account-public-endpoint"></a>Désactiver l’accès au point de terminaison public du compte de stockage
Lorsque l’accès au point de terminaison public est désactivé, le compte de stockage reste accessible via ses points de terminaison privés. Si ce n’est pas le cas, les requêtes valides envoyées au point de terminaison public du compte de stockage seront rejetées. 

# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-disable-portal](../../../includes/storage-files-networking-endpoints-public-disable-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-disable-powershell](../../../includes/storage-files-networking-endpoints-public-disable-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-disable-cli](../../../includes/storage-files-networking-endpoints-public-disable-cli.md)]

---

#### <a name="restrict-access-to-the-storage-account-public-endpoint-to-specific-virtual-networks"></a>Limiter l’accès au point de terminaison public du compte de stockage à certains réseaux virtuels
Lorsque vous limitez l’accès au compte de stockage à certains réseaux virtuels, vous autorisez les requêtes à être envoyées au point de terminaison public à partir des réseaux virtuels spécifiés. Cette approche s’appuie sur une fonctionnalité du réseau virtuel appelée *points de terminaison de service*. Cela peut être utilisé avec ou sans points de terminaison privés.

# <a name="portal"></a>[Portail](#tab/azure-portal)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-portal](../../../includes/storage-files-networking-endpoints-public-restrict-portal.md)]

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-powershell](../../../includes/storage-files-networking-endpoints-public-restrict-powershell.md)]

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
[!INCLUDE [storage-files-networking-endpoints-public-restrict-cli](../../../includes/storage-files-networking-endpoints-public-restrict-cli.md)]

---

### <a name="disable-access-to-the-storage-sync-service-public-endpoint"></a>Désactiver l’accès au point de terminaison public du service de synchronisation du stockage
Azure File Sync vous permet de restreindre l’accès à des réseaux virtuels spécifiques par le biais de points de terminaison privés uniquement. Azure File Sync ne prend pas en charge les points de terminaison de service pour restreindre l’accès au point de terminaison public à des réseaux virtuels spécifiques. Cela signifie que les deux états pour le point de terminaison public du service de synchronisation du stockage sont Activé et Désactivé.

# <a name="portal"></a>[Portail](#tab/azure-portal)
Cela n’est pas possible via le portail Azure. Sélectionnez l’onglet Azure PowerShell pour obtenir des instructions sur la façon de désactiver le point de terminaison public du service de synchronisation du stockage. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour désactiver l’accès au point de terminaison public du service de synchronisation du stockage, définissez la propriété `incomingTrafficPolicy` sur le service de synchronisation du stockage sur `AllowVirtualNetworksOnly`. Si vous souhaitez activer l’accès au point de terminaison public du service de synchronisation du stockage, définissez `incomingTrafficPolicy` sur `AllowAllTraffic` à la place. Pensez à remplacer `<storage-sync-service-resource-group>` et `<storage-sync-service>`.

```powershell
$storageSyncServiceResourceGroupName = "<storage-sync-service-resource-group>"
$storageSyncServiceName = "<storage-sync-service>"

$storageSyncService = Get-AzResource `
        -ResourceGroupName $storageSyncServiceResourceGroupName `
        -ResourceName $storageSyncServiceName `
        -ResourceType "Microsoft.StorageSync/storageSyncServices"

$storageSyncService.Properties.incomingTrafficPolicy = "AllowVirtualNetworksOnly"
$storageSyncService = $storageSyncService | Set-AzResource -Confirm:$false -Force -UsePatchSemantics
```

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
<a name="azure-cli-does-not-support-setting-the-incomingtrafficpolicy-property-on-the-storage-sync-service-please-select-the-azure-powershell-tab-to-get-instructions-on-how-to-disable-the-storage-sync-service-public-endpoint"></a>L’interface Azure CLI ne prend pas en charge la définition de la propriété `incomingTrafficPolicy` sur le service de synchronisation de stockage. Sélectionnez l’onglet Azure PowerShell pour obtenir des instructions sur la façon de désactiver le point de terminaison public du service de synchronisation du stockage.
---

## <a name="see-also"></a>Voir aussi
- [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Déployer Azure File Sync](storage-sync-files-deployment-guide.md)

---
title: Paramètres de pare-feu et de proxy locaux d’Azure File Sync | Microsoft Docs
description: Comprenez les paramètres de proxy et de pare-feu locaux d’Azure File Sync. Passez en revue les détails de configuration des ports, des réseaux et des connexions spéciales à Azure.
author: roygara
ms.service: storage
ms.topic: how-to
ms.date: 09/30/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: cffa6b1200b7236b3c0a3e48b50c58275cf4c57b
ms.sourcegitcommit: 5ae2f32951474ae9e46c0d46f104eda95f7c5a06
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/23/2020
ms.locfileid: "95316618"
---
# <a name="azure-file-sync-proxy-and-firewall-settings"></a>Paramètres de proxy et de pare-feu d’Azure File Sync
Azure File Sync connecte vos serveurs locaux à Azure Files, activant des fonctionnalités de synchronisation multisite et de hiérarchisation cloud. Pour cela, un serveur local doit donc être connecté à Internet. Un administrateur informatique doit déterminer la meilleure voie d’accès aux services cloud Azure pour le serveur.

Cet article fournit des informations sur les exigences spécifiques et les options disponibles pour connecter un serveur en toute sécurité à Azure File Sync.

Avant de lire le présent guide, nous vous recommandons de lire [Considérations relatives aux réseaux Azure File Sync](storage-sync-files-networking-overview.md).

## <a name="overview"></a>Vue d’ensemble
Azure File Sync fait office de service d’orchestration entre votre serveur Windows Server, votre partage de fichiers Azure et plusieurs autres services Azure pour synchroniser les données définies dans votre groupe de synchronisation. Pour qu’Azure File Sync fonctionne correctement, vous devez configurer vos serveurs de manière qu’ils puissent communiquer avec les services Azure suivants :

- Stockage Azure
- Azure File Sync
- Azure Resource Manager
- Services d’authentification

> [!Note]  
> Comme l’agent Azure File Sync sur Windows Server initialise toutes les requêtes vers les services cloud, seul le trafic sortant est à prendre en compte du point de vue du pare-feu. <br /> Aucun service Azure n’établit de connexion avec l’agent Azure File Sync.

## <a name="ports"></a>Ports
Azure File Sync déplace les métadonnées et données de fichiers exclusivement via le protocole HTTPS et nécessite que le port 443 soit ouvert en sortie.
Par conséquent, tout le trafic est chiffré.

## <a name="networks-and-special-connections-to-azure"></a>Réseaux et connexions spéciales à Azure
L’agent Azure File Sync ne présente aucune exigence particulière concernant les canaux spéciaux, comme [ExpressRoute](../../expressroute/expressroute-introduction.md), vers Azure.

Azure File Sync prend en charge n’importe quel mode d’accès à Azure disponible, s’adaptant automatiquement aux différentes caractéristiques de réseau, telles que la bande passante et la latence, tout en offrant un contrôle d’administration pour optimiser les paramètres. Toutes les fonctionnalités ne sont pas disponibles pour le moment. Si vous souhaitez configurer un comportement spécifique, indiquez-le nous sur le [forum UserVoice consacré à Azure Files](https://feedback.azure.com/forums/217298-storage?category_id=180670).

## <a name="proxy"></a>Proxy
Azure File Sync prend en charge les paramètres de proxy au niveau des ordinateurs et relatifs à l’application.

**Les paramètre de proxy spécifiques à l’application** permettent de configurer un proxy spécifiquement pour le trafic d’Azure File Sync. Les paramètres de proxy spécifiques à l’application sont pris en charge sur l’agent version 4.0.1.0 ou ultérieure, et peuvent être configurés pendant l’installation de l’agent ou à l’aide de la cmdlet PowerShell Set-StorageSyncProxyConfiguration.

Commandes PowerShell pour configurer les paramètres de proxy spécifiques à l’application :
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Set-StorageSyncProxyConfiguration -Address <url> -Port <port number> -ProxyCredential <credentials>
```
**Les configurations de proxy au niveau des machines** sont transparentes pour l’agent Azure File Sync, car l’ensemble du trafic du serveur est acheminé à travers ce proxy.

Pour configurer les paramètres de proxy à l’échelle de la machine, suivez les étapes ci-dessous : 

1. Configurer les paramètres de proxy pour les applications .NET 

   - Modifiez ces deux fichiers :  
     C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\machine.config  
     C:\Windows\Microsoft.NET\Framework\v4.0.30319\Config\machine.config

   - Ajoutez la section < system.net > dans les fichiers machine.config (sous la section < system.serviceModel >).  Remplacez 127.0.01:8888 par l’adresse IP et le port du serveur proxy. 
     ```
      <system.net>
        <defaultProxy enabled="true" useDefaultCredentials="true">
          <proxy autoDetect="false" bypassonlocal="false" proxyaddress="http://127.0.0.1:8888" usesystemdefault="false" />
        </defaultProxy>
      </system.net>
     ```

2. Définir les paramètres de proxy WinHTTP 

   - Exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges ou de PowerShell pour afficher les paramètres de proxy existant :   

     netsh winhttp show proxy

   - Exécutez la commande suivante à partir d’une invite de commandes avec élévation de privilèges ou de PowerShell pour définir le paramètre de proxy (remplacez 127.0.01:8888 par l’adresse IP et le port du serveur proxy) :  

     netsh winhttp set proxy 127.0.0.1:8888

3. Redémarrez le service Agent de synchronisation de stockage en exécutant la commande suivante à partir d’une invite de commandes avec élévation de privilèges ou de PowerShell : 

      net stop filesyncsvc

      Remarque : Le service Agent de synchronisation de stockage (filesyncsvc) démarrera automatiquement une fois arrêté.

## <a name="firewall"></a>Pare-feu
Comme mentionné plus haut, le port 443 doit être ouvert en sortie. Selon les stratégies en place dans votre centre de données, votre succursale ou votre région, il se peut que vous souhaitiez ou deviez restreindre le trafic sur ce port à des domaines spécifiques.

Le tableau suivant décrit les domaines requis pour la communication :

| Service | Point de terminaison cloud public | Point de terminaison Azure Government | Usage |
|---------|----------------|---------------|------------------------------|
| **Azure Resource Manager** | `https://management.azure.com` | https://management.usgovcloudapi.net | N’importe quel appel utilisateur (par exemple, PowerShell) est acheminé vers/à travers cette URL, y compris l’appel pour l’inscription initiale du serveur. |
| **Azure Active Directory** | https://login.windows.net<br>`https://login.microsoftonline.com` | https://login.microsoftonline.us | Les appels à Azure Resource Manager doivent être effectués par un utilisateur authentifié. Cette URL est utilisée pour l’authentification utilisateur. |
| **Azure Active Directory** | https://graph.microsoft.com/ | https://graph.microsoft.com/ | Dans le cadre du déploiement d’Azure File Sync, un principal de service est créé dans l’annuaire Azure Active Directory de l’abonnement. Cette URL est utilisée pour cela. Le principal créé sert à déléguer un ensemble minimal de droits au service Azure File Sync. L’utilisateur qui effectue la configuration initiale d’Azure File Sync doit être un utilisateur authentifié avec des privilèges de propriétaire d’abonnement. |
| **Azure Active Directory** | https://secure.aadcdn.microsoftonline-p.com | Utilisez l’URL du point de terminaison public. | L’accès à cette URL est assuré par la bibliothèque d’authentification Active Directory que l’interface utilisateur d’inscription du serveur Azure File Sync utilise pour se connecter à l’administrateur. |
| **Stockage Azure** | &ast;.core.windows.net | &ast;.core.usgovcloudapi.net | Quand le serveur télécharge un fichier, il effectue ce déplacement de données plus efficacement quand il communique directement avec le partage de fichiers Azure dans le compte de stockage. Le serveur présente une clé SAS qui permet uniquement un accès ciblé au partage de fichiers. |
| **Azure File Sync** | &ast;.one.microsoft.com<br>&ast;.afs.azure.net | &ast;.afs.azure.us | Après l’inscription initiale du serveur, le serveur reçoit une URL régionale pour l’instance du service Azure File Sync dans cette région. Le serveur peut utiliser cette URL pour communiquer directement et plus efficacement avec l’instance qui gère sa synchronisation. |
| **Infrastructure à clé publique Microsoft** | https://www.microsoft.com/pki/mscorp/cps<br><http://ocsp.msocsp.com> | https://www.microsoft.com/pki/mscorp/cps<br><http://ocsp.msocsp.com> | Une fois que l’agent Azure File Sync est installé, l’URL de l’infrastructure à clé publique est utilisée pour télécharger les certificats intermédiaires qui sont nécessaires pour communiquer avec le service Azure File Sync et le partage de fichiers Azure. L’URL OCSP est utilisée pour vérifier l’état d’un certificat. |
| **Microsoft Update** | &ast;.update.microsoft.com<br>&ast;.download.windowsupdate.com<br>&ast;.dl.delivery.mp.microsoft.com<br>&ast;.emdl.ws.microsoft.com | &ast;.update.microsoft.com<br>&ast;.download.windowsupdate.com<br>&ast;.dl.delivery.mp.microsoft.com<br>&ast;.emdl.ws.microsoft.com | Une fois l’agent Azure File Sync installé, les URL de Microsoft Update sont utilisées pour télécharger les mises à jour de l’agent Azure File Sync. |

> [!Important]
> Lorsque le trafic est autorisé vers &ast;.afs.azure.net, il n’est possible que vers le service de synchronisation. Aucun autre service Microsoft n’utilise ce domaine.
> Quand le trafic vers &ast;.one.microsoft.com est autorisé, la destination du trafic à partir du serveur n’est pas limitée au service de synchronisation. Les sous-domaines incluent de nombreux autres services Microsoft.

Si &ast;.afs.azure.net ou &ast;.one.microsoft.com est trop vaste, vous pouvez limiter les communications du serveur aux seules instances régionales explicites du service Azure Files Sync. Les instances à définir dépendent de la région du service de synchronisation de stockage où vous avez déployé et inscrit le serveur. Cette région est appelée « URL de point de terminaison principal » dans le tableau ci-dessous.

Pour des raisons de récupération d’urgence et de continuité d’activité (BCDR) vous avez peut-être spécifié des partages de vos fichiers Azure dans un compte de stockage globalement redondant (GRS). Si tel est le cas, vos partages de fichiers Azure basculeront vers la région jumelée en cas de panne régionale durable. Azure File Sync utilise les même paires régionales en tant que stockage. Par conséquent, si vous utilisez des comptes de stockage GRS, vous devez activer des URL supplémentaires pour permettre à votre serveur de communiquer avec la région jumelée pour Azure File Sync. Le tableau ci-dessous appelle cela « Région Jumelée ». De plus, il existe une URL du profil de gestionnaire de trafic qui doit être activée également. Ainsi, le trafic réseau peut être à nouveau routé en toute transparence vers la région jumelée en cas de basculement et est appelé « URL de détection » dans le tableau ci-dessous.

| Cloud  | Région | URL de point de terminaison principal | Région jumelée | URL de détection |
|--------|--------|----------------------|---------------|---------------|
| Public |Australie Est | https:\//australiaeast01.afs.azure.net<br>https:\//kailani-aue.one.microsoft.com | Sud-Australie Est | https:\//tm-australiaeast01.afs.azure.net<br>https:\//tm-kailani-aue.one.microsoft.com |
| Public |Sud-Australie Est | https:\//australiasoutheast01.afs.azure.net<br>https:\//kailani-aus.one.microsoft.com | Australie Est | https:\//tm-australiasoutheast01.afs.azure.net<br>https:\//tm-kailani-aus.one.microsoft.com |
| Public | Brésil Sud | https:\//brazilsouth01.afs.azure.net | États-Unis - partie centrale méridionale | https:\//tm-brazilsouth01.afs.azure.net |
| Public | Centre du Canada | https:\//canadacentral01.afs.azure.net<br>https:\//kailani-cac.one.microsoft.com | Est du Canada | https:\//tm-canadacentral01.afs.azure.net<br>https:\//tm-kailani-cac.one.microsoft.com |
| Public | Est du Canada | https:\//canadaeast01.afs.azure.net<br>https:\//kailani-cae.one.microsoft.com | Centre du Canada | https:\//tm-canadaeast01.afs.azure.net<br>https:\//tm-kailani.cae.one.microsoft.com |
| Public | Inde centrale | https:\//centralindia01.afs.azure.net<br>https:\//kailani-cin.one.microsoft.com | Inde Sud | https:\//tm-centralindia01.afs.azure.net<br>https:\//tm-kailani-cin.one.microsoft.com |
| Public | USA Centre | https:\//centralus01.afs.azure.net<br>https:\//kailani-cus.one.microsoft.com | USA Est 2 | https:\//tm-centralus01.afs.azure.net<br>https:\//tm-kailani-cus.one.microsoft.com |
| Public | Asie Est | https:\//eastasia01.afs.azure.net<br>https:\//kailani11.one.microsoft.com | Asie Sud-Est | https:\//tm-eastasia01.afs.azure.net<br>https:\//tm-kailani11.one.microsoft.com |
| Public | USA Est | https:\//eastus01.afs.azure.net<br>https:\//kailani1.one.microsoft.com | USA Ouest | https:\//tm-eastus01.afs.azure.net<br>https:\//tm-kailani1.one.microsoft.com |
| Public | USA Est 2 | https:\//eastus201.afs.azure.net<br>https:\//kailani-ess.one.microsoft.com | USA Centre | https:\//tm-eastus201.afs.azure.net<br>https:\//tm-kailani-ess.one.microsoft.com |
| Public | Allemagne Nord | https:\//germanynorth01.afs.azure.net | Allemagne Centre-Ouest | https:\//tm-germanywestcentral01.afs.azure.net |
| Public | Allemagne Centre-Ouest | https:\//germanywestcentral01.afs.azure.net | Allemagne Nord | https:\//tm-germanynorth01.afs.azure.net |
| Public | Japon Est | https:\//japaneast01.afs.azure.net | OuJapon Est | https:\//tm-japaneast01.afs.azure.net |
| Public | OuJapon Est | https:\//japanwest01.afs.azure.net | Japon Est | https:\//tm-japanwest01.afs.azure.net |
| Public | Centre de la Corée | https:\//koreacentral01.afs.azure.net/ | Corée du Sud | https:\//tm-koreacentral01.afs.azure.net/ |
| Public | Corée du Sud | https:\//koreasouth01.afs.azure.net/ | Centre de la Corée | https:\//tm-koreasouth01.afs.azure.net/ |
| Public | Centre-Nord des États-Unis | https:\//northcentralus01.afs.azure.net | États-Unis - partie centrale méridionale | https:\//tm-northcentralus01.afs.azure.net |
| Public | Europe Nord | https:\//northeurope01.afs.azure.net<br>https:\//kailani7.one.microsoft.com | Europe Ouest | https:\//tm-northeurope01.afs.azure.net<br>https:\//tm-kailani7.one.microsoft.com |
| Public | États-Unis - partie centrale méridionale | https:\//southcentralus01.afs.azure.net | Centre-Nord des États-Unis | https:\//tm-southcentralus01.afs.azure.net |
| Public | Inde Sud | https:\//southindia01.afs.azure.net<br>https:\//kailani-sin.one.microsoft.com | Inde centrale | https:\//tm-southindia01.afs.azure.net<br>https:\//tm-kailani-sin.one.microsoft.com |
| Public | Asie Sud-Est | https:\//southeastasia01.afs.azure.net<br>https:\//kailani10.one.microsoft.com | Asie Est | https:\//tm-southeastasia01.afs.azure.net<br>https:\//tm-kailani10.one.microsoft.com |
| Public | Sud du Royaume-Uni | https:\//uksouth01.afs.azure.net<br>https:\//kailani-uks.one.microsoft.com | Ouest du Royaume-Uni | https:\//tm-uksouth01.afs.azure.net<br>https:\//tm-kailani-uks.one.microsoft.com |
| Public | Ouest du Royaume-Uni | https:\//ukwest01.afs.azure.net<br>https:\//kailani-ukw.one.microsoft.com | Sud du Royaume-Uni | https:\//tm-ukwest01.afs.azure.net<br>https:\//tm-kailani-ukw.one.microsoft.com |
| Public | Centre-USA Ouest | https:\//westcentralus01.afs.azure.net | USA Ouest 2 | https:\//tm-westcentralus01.afs.azure.net |
| Public | Europe Ouest | https:\//westeurope01.afs.azure.net<br>https:\//kailani6.one.microsoft.com | Europe Nord | https:\//tm-westeurope01.afs.azure.net<br>https:\//tm-kailani6.one.microsoft.com |
| Public | USA Ouest | https:\//westus01.afs.azure.net<br>https:\//kailani.one.microsoft.com | USA Est | https:\//tm-westus01.afs.azure.net<br>https:\//tm-kailani.one.microsoft.com |
| Public | USA Ouest 2 | https:\//westus201.afs.azure.net | Centre-USA Ouest | https:\//tm-westus201.afs.azure.net |
| Gouvernement américain | Gouvernement des États-Unis – Arizona | https:\//usgovarizona01.afs.azure.us | Gouvernement des États-Unis – Texas | https:\//tm-usgovarizona01.afs.azure.us |
| Gouvernement américain | Gouvernement des États-Unis – Texas | https:\//usgovtexas01.afs.azure.us | Gouvernement des États-Unis – Arizona | https:\//tm-usgovtexas01.afs.azure.us |

- Si vous utilisez des comptes de stockage localement redondants (LRS) ou des comptes de stockage de redondants dans une zone (ZRS), vous devez uniquement activer l’URL répertoriée sous « URL de point de terminaison principal ».

- Si vous utilisez des comptes de stockage géoredondants (GRS), activez les trois URL.

**Exemple :** vous déployez un service de synchronisation de stockage dans `"West US"` et inscrivez votre serveur avec celui-ci. Les URL permettant au serveur de communiquer sont dans ce cas :

> - https:\//westus01.afs.azure.net (point de terminaison primaire : USA Ouest)
> - https:\//eastus01.afs.azure.net (région de basculement couplée : USA Est)
> - https:\//tm-westus01.afs.azure.net (URL de détection de la région primaire)

### <a name="allow-list-for-azure-file-sync-ip-addresses"></a>Liste verte des adresses IP Azure File Sync
Azure File Sync prend en charge l’utilisation d’[étiquettes de service](../../virtual-network/service-tags-overview.md), qui représentent un groupe de préfixes d’adresses IP pour un service Azure donné. Vous pouvez utiliser des étiquettes de service pour créer des règles de pare-feu qui permettent la communication avec le service Azure File Sync. L’étiquette de service `StorageSyncService` est utilisée pour Azure File Sync.

Si vous utilisez Azure File Sync dans Azure, vous pouvez autoriser le trafic à l’aide du nom d’étiquette de service directement dans votre groupe de sécurité réseau. Pour en savoir plus, consultez [Groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md).

Si vous utilisez Azure File Sync localement, l’API d’étiquette de service vous permet d’obtenir des plages d’adresses IP spécifiques pour la liste verte de votre pare-feu. Il existe deux méthodes pour obtenir ces informations :

- La liste actuelle des plages d’adresses IP pour tous les services Azure qui prennent en charge les étiquettes de service est publiée chaque semaine dans le centre de téléchargement Microsoft sous la forme d’un document JSON. Chaque Cloud Azure possède son propre document JSON avec les plages d’adresses IP pertinentes pour ce Cloud :
    - [Azure public](https://www.microsoft.com/download/details.aspx?id=56519)
    - [Azure US Government](https://www.microsoft.com/download/details.aspx?id=57063)
    - [Azure Chine](https://www.microsoft.com/download/details.aspx?id=57062)
    - [Azure Allemagne](https://www.microsoft.com/download/details.aspx?id=57064)
- L’API de détection des étiquettes de service (préversion) permet la récupération par programmation de la liste actuelle des étiquettes de service. En préversion, l’API de détection des étiquettes de service peut renvoyer des informations qui sont moins récentes que les informations renvoyées par les documents JSON publiés sur le centre de téléchargement Microsoft. Vous pouvez utiliser l’aire de l’API en fonction de vos préférences d’automatisation :
    - [REST API](/rest/api/virtualnetwork/servicetags/list)
    - [Azure PowerShell](/powershell/module/az.network/Get-AzNetworkServiceTag)
    - [Azure CLI](/cli/azure/network#az-network-list-service-tags)

Étant donné que l’API de détection d’étiquettes de service n’est pas mise à jour aussi fréquemment que les documents JSON publiés dans le centre de téléchargement Microsoft, nous vous recommandons d’utiliser le document JSON pour mettre à jour la liste verte de votre pare-feu local. Pour cela, procédez comme suit :

```PowerShell
# The specific region to get the IP address ranges for. Replace westus2 with the desired region code 
# from Get-AzLocation.
$region = "westus2"

# The service tag for Azure File Sync. Do not change unless you're adapting this
# script for another service.
$serviceTag = "StorageSyncService"

# Download date is the string matching the JSON document on the Download Center. 
$possibleDownloadDates = 0..7 | `
    ForEach-Object { [System.DateTime]::Now.AddDays($_ * -1).ToString("yyyyMMdd") }

# Verify the provided region
$validRegions = Get-AzLocation | `
    Where-Object { $_.Providers -contains "Microsoft.StorageSync" } | `
    Select-Object -ExpandProperty Location

if ($validRegions -notcontains $region) {
    Write-Error `
            -Message "The specified region $region is not available. Either Azure File Sync is not deployed there or the region does not exist." `
            -ErrorAction Stop
}

# Get the Azure cloud. This should automatically based on the context of 
# your Az PowerShell login, however if you manually need to populate, you can find
# the correct values using Get-AzEnvironment.
$azureCloud = Get-AzContext | `
    Select-Object -ExpandProperty Environment | `
    Select-Object -ExpandProperty Name

# Build the download URI
$downloadUris = @()
switch($azureCloud) {
    "AzureCloud" { 
        $downloadUris = $possibleDownloadDates | ForEach-Object {  
            "https://download.microsoft.com/download/7/1/D/71D86715-5596-4529-9B13-DA13A5DE5B63/ServiceTags_Public_$_.json"
        }
    }

    "AzureUSGovernment" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/6/4/D/64DB03BF-895B-4173-A8B1-BA4AD5D4DF22/ServiceTags_AzureGovernment_$_.json"
        }
    }

    "AzureChinaCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/9/D/0/9D03B7E2-4B80-4BF3-9B91-DA8C7D3EE9F9/ServiceTags_China_$_.json"
        }
    }

    "AzureGermanCloud" {
        $downloadUris = $possibleDownloadDates | ForEach-Object { 
            "https://download.microsoft.com/download/0/7/6/076274AB-4B0B-4246-A422-4BAF1E03F974/ServiceTags_AzureGermany_$_.json"
        }
    }

    default {
        Write-Error -Message "Unrecognized Azure Cloud: $_" -ErrorAction Stop
    }
}

# Find most recent file
$found = $false 
foreach($downloadUri in $downloadUris) {
    try { $response = Invoke-WebRequest -Uri $downloadUri -UseBasicParsing } catch { }
    if ($response.StatusCode -eq 200) {
        $found = $true
        break
    }
}

if ($found) {
    # Get the raw JSON 
    $content = [System.Text.Encoding]::UTF8.GetString($response.Content)

    # Parse the JSON
    $serviceTags = ConvertFrom-Json -InputObject $content -Depth 100

    # Get the specific $ipAddressRanges
    $ipAddressRanges = $serviceTags | `
        Select-Object -ExpandProperty values | `
        Where-Object { $_.id -eq "$serviceTag.$region" } | `
        Select-Object -ExpandProperty properties | `
        Select-Object -ExpandProperty addressPrefixes
} else {
    # If the file cannot be found, that means there hasn't been an update in
    # more than a week. Please verify the download URIs are still accurate
    # by checking https://docs.microsoft.com/azure/virtual-network/service-tags-overview
    Write-Verbose -Message "JSON service tag file not found."
    return
}
```

Vous pouvez ensuite utiliser les plages d’adresses IP dans `$ipAddressRanges` pour mettre à jour votre pare-feu. Pour plus d’informations sur la mise à jour de votre pare-feu, consultez le site web de votre équipement réseau/pare-feu.

## <a name="test-network-connectivity-to-service-endpoints"></a>Tester la connectivité réseau aux points de terminaison de service
Une fois qu’un serveur est inscrit auprès du service Azure File Sync, vous pouvez utiliser les applets de commande Test-StorageSyncNetworkConnectivity et ServerRegistration.exe pour tester les communications avec tous les points de terminaison (URL) spécifiques à ce serveur. Cette cmdlet peut aider à résoudre les problèmes qui se posent quand une communication incomplète empêche le serveur de collaborer pleinement avec Azure File Sync, et peut être utilisée pour affiner les configurations de proxy et de pare-feu.

Pour exécuter le test de connectivité réseau, installez la version 9.1 (ou ultérieure) d’Azure File Sync et exécutez les commandes PowerShell suivantes :
```powershell
Import-Module "C:\Program Files\Azure\StorageSyncAgent\StorageSync.Management.ServerCmdlets.dll"
Test-StorageSyncNetworkConnectivity
```

## <a name="summary-and-risk-limitation"></a>Résumé et limitation des risques
La liste fournie plus haut contient les URL avec lesquelles Azure File Sync communique actuellement. Les pare-feux doivent être en mesure d’autoriser le trafic sortant vers ces domaines. Microsoft s’efforce de maintenir cette liste à jour.

La configuration de règles de pare-feu restreignant les domaines peut permettre de renforcer la sécurité. Si ces configurations de pare-feu sont utilisées, vous ne devez pas oublier que des URL seront ajoutées et peuvent même changer au fil du temps. Consultez cet article régulièrement.

## <a name="next-steps"></a>Étapes suivantes
- [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Déployer Azure File Sync](storage-sync-files-deployment-guide.md)
- [Superviser Azure File Sync](storage-sync-files-monitoring.md)

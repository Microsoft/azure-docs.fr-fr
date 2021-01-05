---
title: Considérations relatives à la mise en réseau Azure File Sync | Microsoft Docs
description: Découvrez comment configurer la mise en réseau pour utiliser Azure File Sync afin de mettre en cache des fichiers locaux.
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 06/03/2020
ms.author: rogarana
ms.subservice: files
ms.openlocfilehash: 76d97e3312c1df51193d8a881f3ee07fcd155d75
ms.sourcegitcommit: 9826fb9575dcc1d49f16dd8c7794c7b471bd3109
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/14/2020
ms.locfileid: "94629357"
---
# <a name="azure-file-sync-networking-considerations"></a>Considérations relatives à la mise en réseau Azure File Sync
Vous pouvez vous connecter à un partage de fichiers Azure de deux manières :

- En accédant au partage directement via les protocoles SMB ou FileREST. Ce modèle d’accès est principalement utilisé pour éliminer autant de serveurs locaux que possible.
- En créant un cache du partage de fichiers Azure sur un serveur local (ou sur une machine virtuelle Azure) avec Azure File Sync, puis en accédant aux données du partage de fichiers à partir du serveur local avec le protocole de votre choix (SMB, NFS, FTPS, etc.) en fonction de votre cas d’usage. Ce modèle d’accès est pratique car il combine le meilleur des performances locales et de l’échelle du cloud, ainsi que des services pouvant être attachés sans serveur (serverless), tels que Sauvegarde Azure.

Cet article se concentre sur la configuration de la mise en réseau lorsque votre cas d’usage nécessite l’utilisation d’Azure File Sync pour mettre en cache des fichiers locaux plutôt que de monter directement le partage de fichiers Azure sur SMB. Pour plus d’informations sur les considérations relatives à la mise en réseau pour un déploiement Azure Files, consultez [Considérations relatives à la mise en réseau Azure Files](storage-files-networking-overview.md).

La configuration de la mise en réseau pour Azure File Sync s’étend sur deux objets Azure différents : un service de synchronisation du stockage et un compte de stockage Azure. Un compte de stockage est une construction de gestion qui représente un pool de stockage partagé dans lequel vous pouvez déployer plusieurs partages de fichiers, ainsi que d’autres ressources de stockage, telles que des conteneurs d’objets blob ou des files d’attente. Le service de synchronisation du stockage est une construction de gestion qui représente les serveurs inscrits, qui sont des serveurs de fichiers Windows avec une relation d’approbation établie avec Azure File Sync, ainsi que des groupes de synchronisation, qui définissent la topologie de la relation de synchronisation. 

## <a name="connecting-windows-file-server-to-azure-with-azure-file-sync"></a>Connexion du serveur de fichiers Windows à Azure avec Azure File Sync 
Pour configurer et utiliser Azure Files et Azure File Sync avec un serveur de fichiers Windows local, il n’est pas nécessaire de disposer d’une mise en réseau spéciale sur Azure au-delà d’une connexion Internet de base. Pour déployer Azure File Sync, installez l’agent Azure File Sync sur le serveur de fichiers Windows que vous souhaitez synchroniser avec Azure. L’agent Azure File Sync effectue la synchronisation avec un partage de fichiers Azure via deux canaux :

- Le protocole FileREST, qui est un protocole HTTPS pour accéder à votre partage de fichiers Azure. Étant donné que le protocole FileREST utilise le protocole HTTPS standard pour le transfert de données, seul le port 443 doit être accessible en sortie. Azure File Sync n’utilise pas le protocole SMB pour transférer des données de vos serveurs Windows locaux vers votre partage de fichiers Azure.
- Le protocole de synchronisation Azure File Sync, qui est un protocole HTTPS permettant d’échanger des connaissances de synchronisation, c’est-à-dire les informations de version sur les fichiers et les dossiers de votre environnement, entre les points de terminaison de votre environnement. Ce protocole est également utilisé pour échanger des métadonnées sur les fichiers et les dossiers de votre environnement, tels que les horodateurs et les listes de contrôle d’accès. 

Comme Azure Files offre un accès direct au protocole SMB sur les partages de fichiers Azure, les clients se demandent souvent s’ils ont besoin de configurer une mise en réseau spéciale pour monter les partages de fichiers Azure avec SMB pour que l’agent Azure File Sync y accède. Ce comportement n’est pas nécessaire et il est même fortement déconseillé, à l’exception des scénarios d’administrateur, en raison de l’absence de détection rapide des modifications apportées directement au partage de fichiers Azure : les modifications peuvent ne pas être découvertes pendant plus de 24 heures en fonction de la taille et du nombre d’éléments dans le partage de fichiers Azure. Si vous souhaitez utiliser le partage de fichiers Azure directement, par exemple si vous n’utilisez pas Azure File Sync pour la mise en cache locale, vous pouvez en savoir plus sur les considérations relatives à la mise en réseau pour l’accès direct en consultant [Présentation de la mise en réseau Azure Files](storage-files-networking-overview.md).

Même si Azure File Sync ne nécessite pas de configuration de mise en réseau spéciale, certains clients peuvent souhaiter configurer des paramètres de mise en réseau avancés pour activer les scénarios suivants :

- Interopérabilité avec la configuration du serveur proxy de votre organisation.
- Ouvrir le pare-feu local de votre organisation pour accéder aux services Azure Files et Azure File Sync.
- Tunnel Azure Files et Azure File Sync via ExpressRoute ou une connexion VPN.

### <a name="configuring-proxy-servers"></a>Configuration de serveurs proxy
De nombreuses organisations utilisent un serveur proxy comme intermédiaire entre les ressources au sein de leur réseau local et les ressources en dehors de leur réseau, comme dans Azure. Les serveurs proxy sont utiles pour de nombreuses applications, telles que l’isolement réseau et la sécurité, ainsi que la supervision et la journalisation. Azure File Sync peut interagir entièrement avec un serveur proxy. Toutefois, vous devez configurer manuellement les paramètres de point de terminaison proxy pour votre environnement avec Azure File Sync. Cette opération doit être effectuée via PowerShell à l’aide des cmdlets de serveur Azure File Sync. 

Pour plus d’informations sur la configuration d’Azure File Sync à l’aide d’un serveur proxy, consultez [Configuration d’Azure File Sync avec un serveur proxy](storage-sync-files-firewall-and-proxy.md).

### <a name="configuring-firewalls-and-service-tags"></a>Configuration des pare-feu et des balises de service
Vous pouvez choisir d’isoler vos serveurs de fichiers de la plupart des emplacements Internet pour des raisons de sécurité. Pour utiliser Azure File Sync dans votre environnement, vous devez ajuster votre pare-feu afin de l’ouvrir pour sélectionner les services Azure. Pour ce faire, vous pouvez récupérer les plages d’adresses IP pour les services demandés via un mécanisme appelé [balises de service](../../virtual-network/service-tags-overview.md).

Azure File Sync requiert les plages d’adresses IP pour les services suivants, tels qu’identifiés par leurs balises de service :

| Service | Description | Balise du service |
|---------|-------------|-------------|
| Azure File Sync | Le service Azure File Sync, tel que représenté par l’objet Service de synchronisation du stockage, est responsable de l’activité principale de la synchronisation des données entre un partage de fichiers Azure et un serveur de fichiers Windows. | `StorageSyncService` |
| Azure Files | Toutes les données synchronisées via Azure File Sync sont stockées dans le partage de fichiers Azure. Les fichiers modifiés sur vos serveurs de fichiers Windows sont répliqués sur votre partage de fichiers Azure, et les fichiers hiérarchisés sur votre serveur de fichiers local sont téléchargés en toute transparence lorsqu’un utilisateur les demande. | `Storage` |
| Azure Resource Manager | Azure Resource Manager est l’interface de gestion pour Azure. Tous les appels de gestion, notamment l’inscription de serveur Azure File Sync et les tâches de serveur de synchronisation en cours, sont effectués via Azure Resource Manager. | `AzureResourceManager` |
| Azure Active Directory | Azure Active Directory, ou Azure AD, contient les principaux d’utilisateur requis pour autoriser l’inscription du serveur sur un service de synchronisation du stockage, et les principaux du service requis pour qu’Azure File Sync soit autorisé à accéder à vos ressources cloud. | `AzureActiveDirectory` |

Si vous utilisez Azure File Sync dans Azure, même dans une région différente, vous pouvez utiliser le nom de balise de service directement dans votre groupe de sécurité réseau pour autoriser le trafic vers ce service. Pour en savoir plus, consultez [Groupes de sécurité réseau](../../virtual-network/network-security-groups-overview.md). 

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

Pour en savoir plus sur l’utilisation de l’API de balise de service pour récupérer les adresses de vos services, consultez [Liste verte des adresses IP Azure File Sync](storage-sync-files-firewall-and-proxy.md#allow-list-for-azure-file-sync-ip-addresses).

### <a name="tunneling-traffic-over-a-virtual-private-network-or-expressroute"></a>Tunneling du trafic sur un réseau privé virtuel ou ExpressRoute
Certaines organisations requièrent que la communication avec Azure passe par un tunnel réseau, tel qu’un réseau privé virtuel (VPN) ou ExpressRoute, pour offrir une couche supplémentaire de sécurité ou pour garantir que la communication avec Azure suit un itinéraire déterministe. 

Quand vous établissez un tunnel réseau entre votre réseau local et Azure, vous effectuez le peering de votre réseau local avec un ou plusieurs réseaux virtuels dans Azure. Un [réseau virtuel](../../virtual-network/virtual-networks-overview.md), ou VNet, est similaire à un réseau classique que vous exploiteriez localement. À l’instar d’un compte de stockage Azure ou d’une machine virtuelle Azure, un réseau virtuel est une ressource Azure déployée dans un groupe de ressources. 

Azure Files et File Sync prennent en charge les mécanismes suivants pour transférer le trafic entre vos serveurs locaux et Azure :

- [Passerelle VPN Azure](../../vpn-gateway/vpn-gateway-about-vpngateways.md) : une passerelle VPN est un type spécifique de passerelle de réseau virtuel utilisé pour envoyer du trafic chiffré d’un réseau virtuel Azure vers un autre emplacement (par exemple local) via Internet. Une passerelle VPN Azure est une ressource Azure qui peut être déployée dans un groupe de ressources aux côtés d’un compte de stockage ou d’autres ressources Azure. Comme Azure File Sync est destiné à être utilisé avec un serveur de fichiers Windows local, vous utilisez normalement un [VPN site à site (S2S)](../../vpn-gateway/design.md#s2smulti), bien qu’il soit techniquement possible d’utiliser un [VPN point à site (P2S)](../../vpn-gateway/point-to-site-about.md). 

    Les connexions VPN site à site (S2S) connectent votre réseau virtuel Azure et le réseau local de votre organisation. Une connexion VPN S2S vous permet de configurer une connexion VPN une seule fois pour un serveur VPN ou un appareil hébergé sur le réseau de votre entreprise, au lieu de le faire que pour chaque appareil client devant accéder à votre partage de fichiers Azure. Pour simplifier le déploiement d’une connexion VPN S2S, consultez [Configurer un VPN site à site (S2S) pour une utilisation avec Azure Files](storage-files-configure-s2s-vpn.md).

- [ExpressRoute](../../expressroute/expressroute-introduction.md), qui vous permet de créer un itinéraire défini entre Azure et votre réseau local qui ne traverse pas Internet. ExpressRoute peut être utile quand les performances du réseau doivent être prises en compte, car il fournit un chemin dédié entre votre centre de données local et Azure. ExpressRoute est également une bonne option quand une stratégie ou des exigences réglementaires de votre organisation exigent un chemin d’accès déterministe à vos ressources dans le cloud.

### <a name="private-endpoints"></a>Instances Private Endpoint
Outre les points de terminaison publics par défaut qu’Azure Files et File Sync fournissent via le compte de stockage et le service de synchronisation du stockage, Azure Files et File Sync offrent la possibilité de disposer d’un ou de plusieurs points de terminaison privés par ressource. Lorsque vous créez un point de terminaison privé pour une ressource Azure, il obtient une adresse IP privée provenant de l’espace d’adressage de votre réseau virtuel, de la même façon que votre serveur de fichiers Windows local a une adresse IP de l’espace d’adressage dédié de votre réseau local. 

> [!Important]  
> Pour utiliser des points de terminaison privés sur la ressource de service de synchronisation du stockage, vous devez utiliser l’agent Azure File Sync version 10.1 ou ultérieure. Les versions d’agent antérieures à 10.1 ne prennent pas en charge les points de terminaison privés sur le service de synchronisation de stockage. Toutes les versions antérieures de l’agent prennent en charge les points de terminaison privés sur la ressource du compte de stockage.

Un point de terminaison privé individuel est associé à un sous-réseau de réseau virtuel Azure spécifique. Les comptes de stockage et les services de synchronisation du stockage peuvent avoir des points de terminaison privés dans plusieurs réseaux virtuels.

L’utilisation de points de terminaison privés vous permet d’effectuer les opérations suivantes :
- Établir une connexion sécurisée avec vos ressources Azure à partir de réseaux locaux en utilisant une connexion VPN ou ExpressRoute avec un peering privé.
- Sécuriser vos ressources Azure en désactivant les points de terminaison publics pour Azure Files et File Sync. Par défaut, la création d’un point de terminaison privé n’a pas pour effet de bloquer les connexions au point de terminaison public.
- Renforcer la sécurité du réseau virtuel en bloquant l’exfiltration des données du réseau virtuel (et des limites du peering).

Pour créer un point de terminaison privé, consultez [Configuration des points de terminaison privés pour Azure File Sync](storage-sync-files-networking-endpoints.md).

### <a name="private-endpoints-and-dns"></a>Points de terminaison privés et DNS
Quand vous créez un point de terminaison privé, par défaut, nous créons aussi une zone DNS privée (ou en mettons une existante à jour) correspondant au sous-domaine `privatelink`. Pour les régions de cloud public, ces zones DNS sont `privatelink.file.core.windows.net` pour Azure Files et `privatelink.afs.azure.net` pour Azure File Sync.

> [!Note]  
> Cet article utilise le suffixe DNS de compte de stockage pour les régions publiques Azure, `core.windows.net`. Ce commentaire vaut aussi pour les clouds souverains Azure que sont notamment le cloud Azure US Government et le cloud Azure Chine (il vous suffit de remplacer les suffixes appropriés pour votre environnement). 

Lorsque vous créez des points de terminaison privés pour un compte de stockage et un service de synchronisation du stockage, nous créons des enregistrements A pour eux dans leurs zones DNS privées respectives. Nous mettons également à jour l’entrée DNS publique de sorte que les noms de domaine complets standard soient CNAME pour le nom de liaison privée approprié. Cela permet aux noms de domaine complets de pointer vers les adresses IP du point de terminaison privés lorsque le demandeur se trouve dans le réseau virtuel et de pointer vers les adresses IP du point de terminaison public lorsque le demandeur se trouve en dehors du réseau virtuel. 

Pour Azure Files, chaque point de terminaison privé a un nom de domaine complet unique, qui suit le modèle `storageaccount.privatelink.file.core.windows.net`, mappé à une adresse IP privée pour le point de terminaison privé. Pour Azure File Sync, chaque point de terminaison privé possède quatre noms de domaine complets, pour les quatre points de terminaison différents qu’Azure File Sync expose : gestion, synchronisation (principale), synchronisation (secondaire) et supervision. Les noms de domaine complets pour ces points de terminaison suivent normalement le nom du service de synchronisation du stockage, sauf si le nom contient des caractères non-ASCII. Par exemple, si le nom de votre service de synchronisation du stockage est `mysyncservice` dans la région USA Ouest 2, les points de terminaison équivalents sont `mysyncservicemanagement.westus2.afs.azure.net`, `mysyncservicesyncp.westus2.afs.azure.net`, `mysyncservicesyncs.westus2.afs.azure.net` et `mysyncservicemonitoring.westus2.afs.azure.net`. Chaque point de terminaison privé pour un service de synchronisation du stockage contient 4 adresses IP distinctes. 

Sachant que votre zone DNS privée Azure est connectée au réseau virtuel contenant le point de terminaison privé, vous pouvez observer la configuration DNS en appelant l’applet de commande `Resolve-DnsName` à partir de PowerShell dans une machine virtuelle Azure (ou `nslookup` dans Windows et Linux) :

```powershell
Resolve-DnsName -Name "storageaccount.file.core.windows.net"
```

Pour cet exemple, le compte de stockage `storageaccount.file.core.windows.net` est résolu en adresse IP privée du point de terminaison privé, à savoir `192.168.0.4`.

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


Name                   : privatelink.file.core.windows.net
QueryType              : SOA
TTL                    : 269
Section                : Authority
NameAdministrator      : azureprivatedns-host.microsoft.com
SerialNumber           : 1
TimeToZoneRefresh      : 3600
TimeToZoneFailureRetry : 300
TimeToExpiration       : 2419200
DefaultTTL             : 300
```

Si vous exécutez la même commande en local, vous constaterez que le même nom de compte de stockage est résolu en adresse IP publique du compte de stockage. `storageaccount.file.core.windows.net` est un enregistrement CNAME pour `storageaccount.privatelink.file.core.windows.net` qui, de son côté, est un enregistrement CNAME pour le cluster de stockage Azure hébergeant le compte de stockage :

```Output
Name                              Type   TTL   Section    NameHost
----                              ----   ---   -------    --------
storageaccount.file.core.windows. CNAME  60    Answer     storageaccount.privatelink.file.core.windows.net
net
storageaccount.privatelink.file.c CNAME  60    Answer     file.par20prdstr01a.store.core.windows.net
ore.windows.net

Name       : file.par20prdstr01a.store.core.windows.net
QueryType  : A
TTL        : 60
Section    : Answer
IP4Address : 52.239.194.40
```

Cela reflète le fait qu’Azure Files et Azure File Sync peuvent exposer leurs points de terminaison publics et un ou plusieurs points de terminaison privés par ressource. Pour vous assurer que les noms de domaine complets de vos ressources sont résolus en adresses IP privées de points de terminaison privés, vous devez modifier la configuration de vos serveurs DNS locaux. Il existe plusieurs façons de procéder :

- En modifiant le fichier hosts sur vos clients afin que les noms de domaine complets de vos comptes de stockage et du service de synchronisation du stockage soient résolus en adresses IP privées. Cela est fortement déconseillé pour les environnements de production, car vous devez alors apporter ces modifications à chaque client qui a besoin d’accéder à vos points de terminaison privés. Les modifications apportées à vos ressources/points de terminaison privés (suppressions, modifications, etc.) ne sont pas gérées automatiquement.
- Création de zones DNS sur vos serveurs locaux pour `privatelink.file.core.windows.net` et `privatelink.afs.azure.net` avec des enregistrements A pour vos ressources Azure. L’avantage de cette méthode est que les clients de votre environnement local peuvent résoudre automatiquement les ressources Azure sans avoir à configurer chaque client. Cependant, cette solution s’avère tout aussi fragile pour ce qui est de la modification du fichier hosts, car les modifications n’apparaîtront pas. Bien que fragile, cette solution peut s’avérer être le meilleur choix pour certains environnements.
- Transférez les zones `core.windows.net` et `afs.azure.net` de vos serveurs DNS locaux vers votre zone DNS privée Azure. Il est possible d’atteindre l’hôte DNS privé Azure via une adresse IP spéciale (`168.63.129.16`) qui est uniquement accessible à l’intérieur des réseaux virtuels qui sont liés à la zone DNS privée Azure. Pour contourner cette limitation, vous pouvez exécuter des serveurs DNS supplémentaires à l’intérieur de votre réseau virtuel, qui transféreront `core.windows.net` et `afs.azure.net` vers les zones DNS privées Azure équivalentes. Pour simplifier cette configuration, nous avons prévu des applets de commande PowerShell qui assurent le déploiement automatique des serveurs DNS dans votre réseau virtuel Azure et les configurent comme vous le souhaitez. Pour savoir comment configurer le transfert DNS, consultez [Configuration du DNS avec Azure Files](storage-files-networking-dns.md).

## <a name="encryption-in-transit"></a>Chiffrement en transit
Les connexions établies à partir de l’agent Azure File Sync vers votre service de synchronisation du stockage ou le partage de fichiers Azure sont toujours chiffrées. Bien que les comptes de stockage Azure disposent d’un paramètre permettant de désactiver le chiffrement en transit pour les communications vers Azure Files (et les autres services de stockage Azure qui sont gérés à partir du compte de stockage), la désactivation de ce paramètre n’affecte pas le chiffrement d’Azure File Sync lors de la communication avec Azure Files. Par défaut, le chiffrement en transit est activé pour tous les comptes de stockage Azure. 

Pour plus d’informations sur le chiffrement en transit, voir [ Exiger un transfert sécurisé dans Stockage Azure](../common/storage-require-secure-transfer.md?toc=%2fazure%2fstorage%2ffiles%2ftoc.json).

## <a name="see-also"></a>Voir aussi
- [Planification d’un déploiement de synchronisation de fichiers Azure](storage-sync-files-planning.md)
- [Déployer Azure File Sync](storage-sync-files-deployment-guide.md)
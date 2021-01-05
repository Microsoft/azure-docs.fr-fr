---
title: Protocoles Azure Files disponibles - NFS et SMB
description: Découvrez les protocoles disponibles avant de créer un partage de fichiers Azure, notamment les protocoles SMB (Server Message Block) et NFS (Network File System).
author: roygara
ms.service: storage
ms.topic: conceptual
ms.date: 12/04/2020
ms.author: rogarana
ms.subservice: files
ms.custom: references_regions
ms.openlocfilehash: 288d1740c1c4c67dd8756de5fc0652966ae89488
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620972"
---
# <a name="azure-file-share-protocols"></a>Protocoles de partage de fichiers Azure

Azure Files offre deux protocoles pour la connexion et le montage de vos partages de fichiers Azure. Le [protocole SMB (Server Message Block)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview) et le [protocole NFS (Network File System)](https://en.wikipedia.org/wiki/Network_File_System) (préversion). Actuellement, Azure Files ne prend pas en charge l’accès multiprotocole, de sorte qu’un partage ne peut être qu’un partage NFS ou un partage SMB. Pour cette raison, nous vous recommandons de déterminer quel protocole correspond le mieux à vos besoins avant de créer des partages de fichiers Azure.

## <a name="differences-at-a-glance"></a>Aperçu des différences

|Fonctionnalité  |NFS (préversion)  |SMB  |
|---------|---------|---------|
|Protocoles d’accès     |NFS 4.1         |SMB 2.1, SMB 3.0         |
|Systèmes d’exploitation pris en charge     |Noyau Linux version 4.3+         |Windows 2008 R2+, noyau Linux version 4.11+         |
|[Niveaux disponibles](storage-files-planning.md#storage-tiers)     |Stockage Premium         |Stockage Premium, transaction optimisée, accès chaud, accès froid         |
|[Redondance](storage-files-planning.md#redundancy)     |LRS, ZRS         |LRS, ZRS, GRS         |
|Authentification     |Authentification basée sur l’hôte uniquement        |Authentification basée sur l’identité, authentification basée sur l’utilisateur         |
|Autorisations     |Autorisations de style UNIX         |Autorisations de style NTFS         |
|Sémantique du système de fichiers     |Compatible POSIX         |Non compatible POSIX         |
|Respect de la casse     |Respect de la casse         |Non-respect de la casse         |
|Prise en charge des liens physiques     |Prise en charge         |Non pris en charge         |
|Prise en charge des liens symboliques     |Prise en charge         |Non pris en charge         |
|Suppression ou modification de fichiers ouverts     |Prise en charge         |Non pris en charge         |
|Verrouillage     |Gestionnaire conseil de verrouillage réseau de plage d’octets         |Prise en charge         |
|Liste sécurisée d’adresses IP publiques | Non pris en charge | Prise en charge|
|Interopérabilité de protocole| Non pris en charge | FileREST|

## <a name="nfs-shares-preview"></a>Partages NFS (préversion)

Le montage des partages de fichiers Azure avec NFS 4.1 est actuellement en préversion. Il offre une intégration plus étroite avec Linux. Il s’agit d’une offre pleinement compatible POSIX qui constitue un standard pour les variantes d’Unix et d’autres systèmes d’exploitation *nix. Ce service de stockage de fichiers de qualité professionnelle peut être mis à l’échelle pour répondre à vos besoins de stockage et être accessible simultanément à des milliers d’instances de calcul.

### <a name="limitations"></a>Limites

[!INCLUDE [files-nfs-limitations](../../../includes/files-nfs-limitations.md)]

#### <a name="regional-availability"></a>Disponibilité régionale

[!INCLUDE [files-nfs-regional-availability](../../../includes/files-nfs-regional-availability.md)]

### <a name="best-suited"></a>Application optimale

NFS avec Azure Files est idéal pour :

- Des charges de travail qui requièrent des partages de fichiers compatibles POSIX, le respect de la casse ou des autorisations de style UNIX (UID/GID).
- Charges de travail orientées Linux qui ne nécessitent pas d’accès Windows.

### <a name="security"></a>Sécurité

Toutes les données Azure Files sont chiffrées au repos. Pour un chiffrement en transit, Azure fournit une couche de chiffrement pour toutes les données en transit entre les centres de données Azure en utilisant [MACSec](https://en.wikipedia.org/wiki/IEEE_802.1AE). De ce fait, un chiffrement est appliqué lorsque les données sont transférées entre des centres de données Azure. Contrairement à Azure Files qui utilise le protocole SMB, les partages de fichiers utilisant le protocole NFS n’offrent pas l’authentification basée sur l’utilisateur. L’authentification pour les partages NFS est basée sur les règles de sécurité réseau configurées. Par conséquent, pour vous assurer que seules des connexions sécurisées sont établies avec votre partage NFS, vous devez utiliser des points de terminaison de service ou des points de terminaison privés. Pour accéder aux partages à partir d’un emplacement local, en plus d’un point de terminaison privé, vous devez configurer un réseau VPN ou ExpressRoute. Les demandes qui ne proviennent pas des sources suivantes sont rejetées :

- [Point de terminaison privé](storage-files-networking-overview.md#private-endpoints)
- [Passerelle VPN Azure](../../vpn-gateway/vpn-gateway-about-vpngateways.md)
    - [Réseau VPN de point à site (P2S)](../../vpn-gateway/point-to-site-about.md)
    - [De site à site](../../vpn-gateway/design.md#s2smulti)
- [ExpressRoute](../../expressroute/expressroute-introduction.md)
- [Point de terminaison public restreint](storage-files-networking-overview.md#storage-account-firewall-settings)

Pour plus d’informations sur les options réseau disponibles, consultez [Considérations réseau Azure Files](storage-files-networking-overview.md).

## <a name="smb-shares"></a>Partages SMB

Les partages de fichiers Azure montés avec SMB offrent davantage de fonctionnalités Azure Files et ne présentent aucune restriction des fonctionnalités Azure Files, car elles sont en disponibilité générale.

### <a name="features"></a>Fonctionnalités

- Synchronisation des fichiers Azure
- Authentification basée sur l’identité
- Support Sauvegarde Azure
- Instantanés
- Suppression réversible
- Chiffrement en transit et chiffrement au repos

### <a name="best-suited"></a>Application optimale

SMB avec Azure Files est idéal pour :

- Les environnements de production
- Les clients qui nécessitent une ou plusieurs des fonctionnalités listées dans [Fonctionnalités](#features)

## <a name="next-steps"></a>Étapes suivantes

- [Créer un partage de fichiers NFS](storage-files-how-to-create-nfs-shares.md)
- [créer un partage de fichiers SMB](storage-how-to-create-file-share.md)

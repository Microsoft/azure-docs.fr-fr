---
title: Prise en charge de Network File System 3.0 dans le stockage d’objets blob Azure (préversion) | Microsoft Docs
description: Le stockage Blob Azure prend désormais en charge le protocole NFS (Network File System) 3.0. Cette prise en charge permet aux clients Windows et Linux de monter un conteneur dans le stockage Blob à partir d’une machine virtuelle Azure ou d’un ordinateur local.
author: normesta
ms.subservice: blobs
ms.service: storage
ms.topic: conceptual
ms.date: 08/04/2020
ms.author: normesta
ms.reviewer: yzheng
ms.custom: references_regions
ms.openlocfilehash: c48a7f4b5b1512ec5bcd289e4fb8bd7884ca7394
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96922547"
---
# <a name="network-file-system-nfs-30-protocol-support-in-azure-blob-storage-preview"></a>Prise en charge du protocole NFS (Network File System) 3.0 dans le stockage Blob Azure (préversion)

Le stockage Blob Azure prend désormais en charge le protocole NFS (Network File System) 3.0. Cette prise en charge permet aux clients Windows ou Linux de monter un conteneur dans le stockage Blob à partir d’une machine virtuelle Azure ou d’un ordinateur local. 

> [!NOTE]
> La prise en charge du protocole NFS 3.0 dans Stockage Blob Azure est en préversion publique. Les comptes de stockage GPV2 avec performances de niveau Standard sont pris en charge dans les régions suivantes : Australie Est, Corée Centre et USA Centre Sud. La préversion prend également en charge les objets blobs de blocs avec niveau de performances Premium dans toutes les régions publiques.

## <a name="general-workflow-mounting-a-storage-account-container"></a>Workflow général : Monter un conteneur de compte de stockage

Pour monter un conteneur de compte de stockage, vous devez effectuer les opérations suivantes.

1. Inscrivez la fonctionnalité de protocole NFS 3.0 avec votre abonnement.

2. Vérifier que l’appareil est enregistré.

3. Créez un réseau virtuel Azure (Vnet).

4. Configurez la sécurité du réseau.

5. Créez et configurez un compte de stockage qui accepte le trafic uniquement à partir du réseau virtuel.

6. Créez un conteneur dans le compte de stockage.

7. Montez le conteneur.

Pour obtenir des instructions pas à pas, consultez [Monter le stockage Blob à l’aide du protocole NFS (Network File System) 3.0 (préversion)](network-file-system-protocol-support-how-to.md).

> [!IMPORTANT]
> Il est important d’effectuer ces tâches dans l’ordre. Vous ne pouvez pas monter les conteneurs que vous créez avant d’activer le protocole NFS 3.0 sur votre compte. De plus, une fois que vous avez activé le protocole NFS 3.0 sur votre compte, vous ne pouvez pas le désactiver.

## <a name="network-security"></a>Sécurité du réseau

Votre compte de stockage doit se trouver dans un réseau virtuel. Un réseau virtuel permet aux clients de se connecter en toute sécurité à votre compte de stockage. La seule façon de sécuriser les données dans votre compte consiste à utiliser un réseau virtuel et d’autres paramètres de sécurité réseau. Tous les autres outils utilisé pour sécuriser les données, notamment l’autorisation de clé de compte, la sécurité Azure Active Directory (AD) et les listes de contrôle d’accès (ACL), ne sont pas encore pris en charge dans les comptes sur lesquels la prise en charge du protocole NFS 3.0 est activée. 

Pour en savoir plus, consultez les [recommandations de sécurité réseau pour le stockage Blob](security-recommendations.md#networking).

## <a name="supported-network-connections"></a>Connexions réseau prises en charge

Un client peut se connecter via un point de terminaison public ou [un point de terminaison privé](../common/storage-private-endpoints.md) à partir de l’un des emplacements réseau suivants :

- Le réseau virtuel que vous configurez pour votre compte de stockage. 

  Dans cet article, nous ferons référence à ce réseau virtuel en tant que *réseau virtuel principal*. Pour en savoir plus, consultez [Autoriser l'accès à partir d'un réseau virtuel](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Un réseau virtuel associé qui se trouve dans la même région que le réseau virtuel principal.

  Vous devrez configurer votre compte de stockage pour autoriser l’accès à ce réseau virtuel associé. Pour en savoir plus, consultez [Autoriser l'accès à partir d'un réseau virtuel](../common/storage-network-security.md#grant-access-from-a-virtual-network).

- Un réseau local connecté à votre réseau virtuel principal à l’aide d'une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou d’une [passerelle ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md). 

  Pour en savoir plus, consultez [Configuration de l’accès à partir de réseaux locaux](../common/storage-network-security.md#configuring-access-from-on-premises-networks).

- Un réseau local connecté à un réseau associé.

  Cette opération peut être effectuée avec une [passerelle VPN](../../vpn-gateway/vpn-gateway-about-vpngateways.md) ou une [passerelle ExpressRoute](../../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md) ainsi qu’avec le [transit de la passerelle](/azure/architecture/reference-architectures/hybrid-networking/vnet-peering#gateway-transit). 

> [!IMPORTANT]
> Si vous vous connectez à partir d’un réseau local, assurez-vous que votre client autorise les communications sortantes via les ports 111 et 2048. Le protocole NFS 3.0 utilise ces ports.

## <a name="azure-storage-features-not-yet-supported"></a>Fonctionnalités de stockage Azure pas encore prises en charge

Les fonctionnalités de stockage Azure suivantes ne sont pas prises en charge lorsque vous activez le protocole NFS 3.0 sur votre compte. 

- Sécurité d’Azure Active Directory (AD)

- Listes de contrôle d’accès (ACL) de type POSIX

- Possibilité d’activer la prise en charge de NFS 3.0 sur des comptes de stockage existants

- Possibilité de désactiver la prise en charge de NFS 3.0 dans un compte de stockage (après l’avoir activée)

- Possibilité d’écrire dans des objets blob à l’aide d’API REST ou de kits de développement logiciel (SDK). 
  
## <a name="nfs-30-features-not-yet-supported"></a>Fonctionnalités NFS 3.0 pas encore prises en charge

Les fonctionnalités NFS 3.0 suivantes ne sont pas encore prises en charge avec Azure Data Lake Storage Gen2.

- NFS 3.0 sur UDP. Seul NFS 3.0 sur TCP est pris en charge.

- Verrouillage des fichiers avec NLM (Network Lock Manager). Les commandes de montage doivent inclure le paramètre `-o nolock`.

- Montage des sous-répertoires. Vous pouvez monter uniquement le répertoire racine (conteneur).

- Répertorier les montages (par exemple : à l’aide de la commande `showmount -a`)

- Répertorier les exportations (par exemple : à l’aide de la commande `showmount -e`)

- Exportation d’un conteneur en lecture seule

## <a name="pricing"></a>Tarifs

Pendant la préversion, les données stockées dans votre compte de stockage sont facturées selon le même taux de capacité que le stockage Blob par Go par mois. 

Une transaction n’est pas facturée pendant la préversion. La tarification des transactions est sujette à modification et sera déterminée lorsqu’elle sera en disponibilité générale.

## <a name="next-steps"></a>Étapes suivantes

Pour démarrer, consultez [Monter le stockage Blob sur Linux à l’aide du protocole NFS (Network File System) 3.0 (préversion)](network-file-system-protocol-support-how-to.md).

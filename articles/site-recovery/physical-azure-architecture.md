---
title: Architecture de la reprise d’activité après sinistre de serveurs physiques dans Azure Site Recovery
description: Cet article fournit une vue d’ensemble des composants et de l’architecture utilisés lors de la récupération d’urgence de serveurs physiques locaux sur Azure avec le service Azure Site Recovery.
ms.topic: conceptual
ms.date: 02/11/2020
ms.openlocfilehash: 966636e269043d81912b552711635f34b412f22f
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654725"
---
# <a name="physical-server-to-azure-disaster-recovery-architecture"></a>Serveur physique pour l’architecture de récupération d’urgence sur Azure

Cet article décrit l’architecture et les processus utilisés quand vous répliquez, basculez et récupérez des serveurs Windows et Linux physiques entre un site local et Azure, à l’aide du service [Azure Site Recovery](site-recovery-overview.md).

## <a name="architectural-components"></a>Composants architecturaux

Le tableau et le graphique suivants fournissent une vue d’ensemble des composants utilisés pour la réplication de serveurs physiques vers Azure.

| **Composant** | **Prérequis** | **Détails** |
| --- | --- | --- |
| **Microsoft Azure** | Un abonnement Azure et un réseau Azure. | Les données répliquées de machines physiques locales sont stockées dans des disques managés Azure. Les machines virtuelles Azure sont créées avec les données répliquées quand vous exécutez un basculement depuis le site local vers Azure. Les machines virtuelles Azure se connectent au réseau virtuel Azure lors de leur création. |
| **Serveur de traitement** | Installé par défaut avec le serveur de configuration. | Fait office de passerelle de réplication. Reçoit les données de réplication, les optimise grâce à la mise en cache, la compression et le chiffrement et les envoie vers le stockage Azure.<br/><br/> En outre, le serveur de traitement installe le service Mobilité sur les serveurs que vous souhaitez répliquer.<br/><br/> À mesure que s’étend votre déploiement, vous pouvez ajouter des serveurs de traitement distincts afin de gérer de plus grands volumes de trafic de réplication. |
| **Serveur cible maître** | Installé par défaut avec le serveur de configuration. | Gère les données de réplication pendant la restauration automatique à partir d’Azure.<br/><br/> Pour les déploiements à grande échelle, vous pouvez ajouter un serveur cible maître distinct à des fins de restauration automatique. |
| **Serveurs répliqués** | Le service Mobilité est installé sur chaque serveur que vous répliquez. | Nous vous recommandons d’autoriser l’installation automatique à partir du serveur de traitement. Vous pouvez également installer le service manuellement ou utiliser une méthode de déploiement automatisée, telle que Configuration Manager. |

**Architecture de serveur physique vers Azure**

![Components](./media/physical-azure-architecture/arch-enhanced.png)

## <a name="set-up-outbound-network-connectivity"></a>Configurer la connectivité réseau sortante

Pour que Site Recovery fonctionne comme prévu, vous devez modifier la connectivité réseau sortante pour permettre au réseau d’effectuer la réplication.

> [!NOTE]
> Site Recovery ne prend pas en charge l’utilisation d’un proxy d’authentification pour contrôler la connectivité réseau.

### <a name="outbound-connectivity-for-urls"></a>Connectivité sortante pour les URL

Si vous utilisez un proxy de pare-feu basé sur des URL pour contrôler la connectivité sortante, autorisez l’accès à ces URL :

| **Nom**                  | **Commerciale**                               | **Secteur public**                                 | **Description** |
| ------------------------- | -------------------------------------------- | ---------------------------------------------- | ----------- |
| Stockage                   | `*.blob.core.windows.net`                  | `*.blob.core.usgovcloudapi.net` | Permet d’écrire les données dans le compte de stockage de cache dans la région source à partir de la machine virtuelle. |
| Azure Active Directory    | `login.microsoftonline.com`                | `login.microsoftonline.us`                   | Fournit l’autorisation et l’authentification aux URL du service Site Recovery. |
| Réplication               | `*.hypervrecoverymanager.windowsazure.com` | `*.hypervrecoverymanager.windowsazure.com`   | Permet à la machine virtuelle de communiquer avec le service Site Recovery. |
| Service Bus               | `*.servicebus.windows.net`                 | `*.servicebus.usgovcloudapi.net`             | Permet à la machine virtuelle d’écrire des données de surveillance et de diagnostic Site Recovery. |


## <a name="replication-process"></a>Processus de réplication

1. Vous configurez le déploiement, notamment celui des composants locaux et des composants Azure. Dans le coffre Recovery Services, vous spécifiez la source et la cible de réplication, configurez le serveur de configuration, créez une stratégie de réplication et activez la réplication.
1. Les machines sont répliquées à l’aide de la stratégie, et une copie initiale des données de serveur est répliquée sur le stockage Azure.
1. La réplication des modifications delta dans Azure commence à l’issue de la réplication initiale. Les modifications suivies pour une machine sont conservées dans un fichier avec l’extension _.hrl_.
   - Les machines communiquent avec le serveur de configuration sur le port HTTPS 443 entrant pour la gestion de la réplication.
   - Les machines envoient des données de réplication au serveur de traitement sur le port HTTPS 9443 entrant (modification possible).
   - Le serveur de configuration orchestre la gestion de la réplication avec Azure sur le port HTTPS 443 sortant.
   - Le serveur de processus reçoit des données de machines sources, les optimise et les chiffre, puis les envoie au stockage Azure via le port HTTPS 443 sortant.
   - Si vous activez la cohérence multimachine virtuelle, les machines du groupe de réplication communiquent entre elles sur le port 20004. Le mode multimachine virtuelle est utilisé si vous regroupez plusieurs machines dans des groupes de réplication qui partagent des points de récupération cohérents en cas d’incident et avec les applications lorsqu’ils basculent. Ces groupes sont utiles si les machines exécutent la même charge de travail et doivent être cohérentes.
1. Le trafic est répliqué sur des points de terminaison publics de stockage Azure via Internet. L’autre solution consiste à utiliser le [peering public](../expressroute/about-public-peering.md) Azure ExpressRoute.

   > [!NOTE]
   > La réplication n’est pas prise en charge sur un VPN site à site à partir d’un site local ou d’une [homologation privée](concepts-expressroute-with-site-recovery.md#on-premises-to-azure-replication-with-expressroute) Azure ExpressRoute.

**Processus de réplication de serveurs physiques vers Azure**

![Processus de réplication](./media/physical-azure-architecture/v2a-architecture-henry.png)

## <a name="failover-and-failback-process"></a>Processus de basculement et de restauration automatique

Une fois la réplication configurée, vous pouvez exécuter un exercice de récupération d’urgence (test de basculement) pour vérifier que tout fonctionne comme prévu. Ensuite, vous pouvez effectuer un basculement et une restauration automatique en fonction des besoins. Envisagez ce qui suit :

- Le basculement planifié n’est pas pris en charge.
- Vous devez effectuer une restauration automatique vers une machine virtuelle VMware locale. Vous avez besoin d’une infrastructure VMware locale, même lorsque vous répliquez des serveurs physiques locaux vers Azure.
- Vous basculez une seule machine ou créez des plans de récupération pour basculer ensemble plusieurs machines virtuelles.
- Quand vous exécutez un basculement, les machines virtuelles Azure sont créées à partir des données répliquées dans le stockage Azure.
- Une fois le basculement initial déclenché, validez-le pour accéder à la charge de travail à partir de la machine virtuelle Azure.
- Lorsque votre site local principal est à nouveau disponible, vous pouvez lancer la restauration automatique.
- Configurez une infrastructure de restauration automatique comprenant les éléments suivants :
  - **Serveur de traitement temporaire dans Azure** : pour effectuer une restauration automatique à partir d’Azure, configurez une machine virtuelle Azure faisant office de serveur de traitement pour gérer la réplication à partir d’Azure. Vous pourrez supprimer cette machine virtuelle une fois la restauration automatique terminée.
  - **Connexion VPN** : pour la restauration automatique, vous avez besoin d’une connexion VPN (ou Azure ExpressRoute) du réseau Azure vers le site local.
  - **Serveur cible maître distinct** : par défaut, le serveur cible maître installé avec le serveur de configuration sur la machine virtuelle VMware locale gère la restauration automatique. Si vous devez restaurer automatiquement de grands volumes de trafic, vous devez configurer un serveur cible maître local distinct.
  - **Stratégie de restauration automatique** : pour répliquer vers votre site local, vous avez besoin d’une stratégie de restauration automatique. La stratégie a été créée automatiquement lors de la création de votre stratégie de réplication à partir du site local vers Azure.
  - **Infrastructure VMware** : Pour la restauration automatique, vous avez besoin d’une infrastructure VMware. Il est impossible d’effectuer cette procédure vers un serveur physique.
- Une fois les composants en place, la restauration automatique s’effectue en trois étapes :
  - **Étape 1** : Reprotégez les machines virtuelles Azure afin qu’elles répliquent à partir d’Azure vers les machines virtuelles VMware locales.
  - **Étape 2** : Exécutez un basculement vers le site local.
  - **Étape 3** : Une fois les charges de travail automatiquement restaurées, vous réactivez la réplication.

**Restauration automatique VMware à partir d’Azure**

![Restauration automatique](./media/physical-azure-architecture/enhanced-failback.png)

## <a name="next-steps"></a>Étapes suivantes

Pour configurer la récupération d’urgence de serveurs physiques sur Azure, voir le [guide pratique](physical-azure-disaster-recovery.md).

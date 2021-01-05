---
title: Protéger un serveur de fichiers avec Azure Site Recovery
description: Cet article explique comment protéger un serveur de fichiers avec Azure Site Recovery
author: Sharmistha-Rai
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 07/31/2019
ms.author: sharrai
ms.custom: mvc
ms.openlocfilehash: 1c88a7e78d37556f0f1f89e16154dc1808f24323
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89425515"
---
# <a name="protect-a-file-server-by-using-azure-site-recovery"></a>Protéger un serveur de fichiers avec Azure Site Recovery 

[Azure Site Recovery](site-recovery-overview.md) contribue à votre stratégie de récupération d’urgence et de continuité d’activité en garantissant le bon fonctionnement et la disponibilité de vos applications métier pendant les interruptions planifiées et non planifiées. Site Recovery gère et orchestre la récupération d’urgence des machines locales et des machines virtuelles Azure. La récupération d’urgence comprend la réplication, le basculement et la récupération de différentes charges de travail.

Cet article explique comment protéger un serveur de fichiers avec Site Recovery et fournit d’autres recommandations adaptées à différents environnements. 

- [Répliquer les ordinateurs serveurs de fichiers Azure IaaS](#disaster-recovery-recommendation-for-azure-iaas-virtual-machines)
- [Répliquer un serveur de fichiers local avec Site Recovery](#replicate-an-on-premises-file-server-by-using-site-recovery)

## <a name="file-server-architecture"></a>Architecture du serveur de fichiers
L’objectif d’un système de partage de fichiers distribués ouvert est de fournir un environnement dans lequel un groupe d’utilisateurs dispersés géographiquement peut collaborer pour travailler efficacement sur des fichiers et avoir la garantie que ses spécifications relatives à l’intégrité sont appliquées. Un écosystème de serveurs de fichiers local classique qui prend en charge un grand nombre d’utilisateurs simultanés et un grand nombre d’éléments de contenu utilise la réplication du système de fichiers DFS (DFSR) pour la planification de la réplication et la limitation de la bande passante. 

La DFSR utilise un algorithme de compression appelé Connexion Bureau à distance (RDC), qui peut être utilisé pour mettre à jour efficacement des fichiers sur un réseau à bande passante limitée. Elle détecte les insertions, suppressions et réorganisations de données dans les fichiers. La DFSR est activée pour répliquer uniquement les blocs de fichiers modifiés lorsque les fichiers sont mis à jour. Il existe également des environnements de serveurs de fichiers dans lesquels des sauvegardes quotidiennes sont effectuées pendant les heures creuses, destinés aux besoins d’urgence. La DFSR n’est pas implémentée.

Le diagramme suivant illustre l’environnement de serveurs de fichiers avec la DFSR implémentée.
        
![Architecture DFSR](media/site-recovery-file-server/dfsr-architecture.JPG)

Dans le diagramme précédent, plusieurs serveurs de fichiers, appelés membres, participent activement à la réplication des fichiers dans un groupe de réplication. Le contenu du dossier répliqué sera accessible à tous les clients qui enverront des requêtes à l’un des membres, même en cas de mise hors connexion de l’un d’eux.

## <a name="disaster-recovery-recommendations-for-file-servers"></a>Recommandations relatives à la récupération d’urgence pour les serveurs de fichiers

* **Répliquez un serveur de fichiers local avec Site Recovery** : Les serveurs de fichiers peuvent être répliqués sur Azure avec Site Recovery. Lorsqu’un ou plusieurs serveurs de fichiers locaux sont inaccessibles, les machines virtuelles de récupération peuvent être mises dans Azure. Les machines virtuelles peuvent ensuite traiter les demandes des clients, localement, à condition qu’il existe une connectivité VPN de site à site et qu’Active Directory soit configuré dans Azure. Vous pouvez utiliser cette méthode dans le cas d’un environnement configuré pour la DFSR ou d’un environnement de serveur de fichiers simple sans DFSR. 

* **Étendez la DFSR à une machine virtuelle Azure IaaS** : Dans un environnement de serveurs de fichiers en cluster avec la DFSR implémentée, vous pouvez étendre la DFSR locale à Azure. Une machine virtuelle Azure est alors activée pour jouer le rôle de serveur de fichiers. 

    * Une fois les dépendances de connectivité VPN de site à site et Active Directory traités et la DFSR appliquée, lorsqu’un ou plusieurs des serveurs de fichiers locaux ne sont pas accessibles, les clients peuvent toujours se connecter à la machine virtuelle Azure, qui traitera les demandes.

    * Vous pouvez utiliser cette approche si vos machines virtuelles ont des configurations qui ne sont pas prises en charge par Site Recovery. Par exemple, un disque de cluster partagé, parfois couramment utilisé dans des environnements de serveurs de fichiers. La DFSR fonctionne également bien dans des environnements à faible bande passante avec un taux de variation moyen. Vous devez prendre en compte le coût supplémentaire que représente le fait d’avoir une machine virtuelle Azure opérationnelle en permanence. 

* **Utilisez la synchronisation de fichiers pour répliquer vos fichiers** : Si vous prévoyez d’utiliser le cloud ou si vous utilisez déjà une machine virtuelle Azure, vous pouvez utiliser Azure File Sync. Azure File Sync offre la synchronisation de partages de fichiers entièrement managés, accessibles via le protocole de normes industrielles SMB [(Server Message Block)](/windows/win32/fileio/microsoft-smb-protocol-and-cifs-protocol-overview). Les partages de fichiers Azure peuvent ensuite être montés simultanément par des déploiements cloud ou locaux de Windows, Linux et MacOS. 

Le diagramme suivant vous permet de déterminer la stratégie à utiliser pour votre environnement de serveurs de fichiers.

![Arbre de décision](media/site-recovery-file-server/decisiontree.png)


### <a name="factors-to-consider-in-your-decisions-about-disaster-recovery-to-azure"></a>Facteurs à prendre en compte quand vous prenez des décisions sur la récupération d’urgence vers Azure

|Environnement  |Recommandation  |Éléments à prendre en considération |
|---------|---------|---------|
|Environnement de serveurs de fichiers avec ou sans DFSR|   [Utiliser Site Recovery pour la réplication](#replicate-an-on-premises-file-server-by-using-site-recovery)   |    Site Recovery ne prend pas en charge les clusters de disque partagés ni le périphérique de stockage NAS. Si votre environnement utilise ces configurations, utilisez l’une des autres approches, le cas échéant. <br> Site Recovery ne prend pas en charge SMB 3.0. La machine virtuelle répliquée intègre les modifications uniquement lorsque celles qui sont apportées aux fichiers sont mises à jour dans l’emplacement d’origine des fichiers.<br>  Site Recovery offre un processus de réplication de données quasi synchrone. Par conséquent, en cas de basculement non planifié, une perte de données potentielle est susceptible de se produire et peut créer des problèmes de non-correspondance USN.
|Environnement de serveurs de fichiers avec DFSR     |  [Étendre la DFSR à une machine virtuelle Azure IaaS](#extend-dfsr-to-an-azure-iaas-virtual-machine)  |    La DFSR fonctionne bien dans des environnements avec une bande passante extrêmement faible. Cette approche nécessite une machine virtuelle Azure opérationnelle en permanence. Vous devez prendre en compte le coût de la machine virtuelle dans votre planification.         |
|Machine virtuelle Azure IaaS     |     File Sync    |     Si vous utilisez la synchronisation de fichiers dans un scénario de récupération d’urgence, pendant le basculement, vous devez prendre des mesures manuelles pour vous assurer que les partages de fichiers sont accessibles à la machine du client de manière transparente. La synchronisation de fichiers exige que le port 445 soit ouvert à partir de la machine du client.     |


### <a name="site-recovery-support"></a>Prise en charge de Site Recovery
Étant donné que la réplication Site Recovery est indépendante des applications, ces recommandations sont censées être vraies pour les scénarios suivants.

| Source  |Vers un site secondaire  |Vers Azure
|---------|---------|---------|
|Azure|  -|Oui|
|Hyper-V|  Oui  |Oui
|VMware  |Oui|  Oui
|Serveur physique|  Oui  |Oui
 

> [!IMPORTANT]
> Avant de passer à l’une des trois approches suivantes, vérifiez que ces dépendances sont prises en charge.



**Connectivité de site à site** : Une connexion directe entre le site local et le réseau Azure doit être établie pour permettre la communication entre les serveurs. Utilisez une connexion VPN de site à site sécurisée vers un réseau virtuel Azure utilisé en tant que site de récupération d’urgence. Pour plus d’informations, consultez [Établir une connexion VPN de site à site entre un site local et un réseau virtuel Azure](../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md).

**Active Directory** : La DFSR dépend d’Active Directory. Cela signifie que la forêt Active Directory avec des contrôleurs de domaine locaux est étendue au site de récupération d’urgence dans Azure. Même si vous n’utilisez pas la DFSR, si les utilisateurs prévus doivent recevoir une autorisation d’accès ou une vérification, vous devez suivre ces étapes. Pour plus d’informations, reportez-vous à [Étendre Active Directory local à Azure](./site-recovery-active-directory.md).

## <a name="disaster-recovery-recommendation-for-azure-iaas-virtual-machines"></a>Recommandations relatives à la récupération d’urgence pour les machines virtuelles Azure IaaS

Si vous configurez et gérez la récupération d’urgence de serveurs de fichiers hébergés sur des machines virtuelles Azure IaaS, vous avez le choix entre deux options, selon que vous souhaitez ou non passer à [Azure Files](../storage/files/storage-files-introduction.md) :

* [Utiliser la synchronisation de fichiers](#use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine)
* [Utiliser Site Recovery](#replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery)

## <a name="use-file-sync-to-replicate-files-hosted-on-an-iaas-virtual-machine"></a>Utiliser la synchronisation de fichiers pour répliquer des fichiers hébergés sur une machine virtuelle IaaS

Azure Files peut être utilisé pour remplacer complètement ou compléter les serveurs de fichiers locaux traditionnels ou les périphériques NAS. Les partages de fichiers Azure peuvent également être répliqués avec la synchronisation de fichiers vers des serveurs Windows, localement ou dans le cloud, pour les performances et la mise en cache distribuée des données, le cas échéant. Les étapes suivantes décrivent la recommandation de récupération d’urgence pour les machines virtuelles Azure qui exécutent les mêmes fonctionnalités que les serveurs de fichiers classiques :
* Protéger les machines à l’aide Site Recovery. Suivez les étapes de la rubrique [Répliquer une machine virtuelle Azure vers une autre région Azure](azure-to-azure-quickstart.md).
* Utilisez la synchronisation de fichiers pour répliquer des fichiers provenant de la machine virtuelle qui fait office de serveur de fichiers dans le cloud.
* Utilisez la fonctionnalité [plan de récupération](site-recovery-create-recovery-plans.md) Site Recovery pour ajouter des scripts afin de [monter le partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md) et d’accéder au partage dans votre machine virtuelle.

Les étapes suivantes décrivent brièvement comment utiliser la synchronisation de fichiers :

1. [Créez un compte de stockage dans Azure](../storage/common/storage-account-create.md?toc=/azure/storage/files/toc.json). Si vous avez choisi le stockage géoredondant avec accès en lecture pour vos comptes de stockage, vous obtenez un accès en lecture à vos données à partir de la région secondaire, en cas d’urgence. Pour plus d’informations, consultez [Récupération d’urgence et basculement de compte de stockage](../storage/common/storage-disaster-recovery-guidance.md?toc=%2fazure%2fstorage%2ffiless%2ftoc.json).
2. [Créer un partage de fichiers](../storage/files/storage-how-to-create-file-share.md).
3. [Démarrer la synchronisation de fichiers](../storage/files/storage-sync-files-deployment-guide.md) sur votre serveur de fichiers Azure.
4. Créez un groupe de synchronisation. Les points de terminaison dans un groupe de synchronisation sont synchronisés entre eux. Un groupe de synchronisation doit contenir au moins un point de terminaison cloud, qui représente un partage de fichiers Azure. Un groupe de synchronisation doit également contenir un point de terminaison de serveur, qui représente un chemin d’accès sur un serveur Windows.
5. Vos fichiers restent désormais synchronisés sur votre partage de fichiers Azure et sur votre serveur local.
6. En cas d’urgence dans votre environnement local, effectuer un basculement en utilisant un [plan de récupération](site-recovery-create-recovery-plans.md). Ajoutez le script pour [monter le partage de fichiers Azure](../storage/files/storage-how-to-use-files-windows.md) et accéder au partage dans votre machine virtuelle.

### <a name="replicate-an-iaas-file-server-virtual-machine-by-using-site-recovery"></a>Répliquer une machine virtuelle de serveur de fichiers IaaS avec Site Recovery

Si vous avez des clients locaux qui accèdent à la machine virtuelle de serveur de fichiers IaaS, suivez toutes les étapes ci-après. Sinon, passez à l’étape 3.

1. Établissez une connexion VPN site à site entre un site local et un réseau Azure.
2. Étendez Active Directory local.
3. [Configurez la récupération d’urgence](azure-to-azure-tutorial-enable-replication.md) pour l’ordinateur serveur de fichiers IaaS dans une région secondaire.


Pour plus d’informations sur la récupération d’urgence dans une région secondaire, reportez-vous à [cet article](./azure-to-azure-architecture.md).


## <a name="replicate-an-on-premises-file-server-by-using-site-recovery"></a>Répliquer un serveur de fichiers local avec Site Recovery

Les étapes suivantes décrivent la réplication pour une machine virtuelle VMware. Pour connaitre les étapes à suivre pour répliquer une machines virtuelle Hyper-V, consultez [ce didacticiel](./hyper-v-azure-tutorial.md).

1. [Préparez les ressources Azure](tutorial-prepare-azure.md) pour la réplication de machines locales.
2. Établissez une connexion VPN site à site entre un site local et un réseau Azure. 
3. Étendez Active Directory local.
4. [Préparez les serveurs VMware locaux](./vmware-azure-tutorial-prepare-on-premises.md).
5. [Configurez la récupération d’urgence](./vmware-azure-tutorial.md) vers Azure pour des machines virtuelles locales.

## <a name="extend-dfsr-to-an-azure-iaas-virtual-machine"></a>Étendre la DFSR à une machine virtuelle Azure IaaS

1. Établissez une connexion VPN site à site entre un site local et un réseau Azure. 
2. Étendez Active Directory local.
3. [Créer et approvisionner une machine virtuelle de serveur de fichiers](../virtual-machines/windows/quick-create-portal.md?toc=%2Fazure%2Fvirtual-machines%2Fwindows%2Ftoc.json) sur le réseau virtuel Windows Azure.
Vérifiez que la machine virtuelle est ajoutée au même réseau virtuel Windows Azure, lequel dispose de l’inter-connectivité avec l’environnement local. 
4. Installer et [configurer la DFSR](https://techcommunity.microsoft.com/t5/storage-at-microsoft/dfs-replication-initial-sync-in-windows-server-2012-r2-attack-of/ba-p/424877) sur Windows Server.
5. [Implémenter un espace de noms DFS](/windows-server/storage/dfs-namespaces/deploying-dfs-namespaces).
6. Une fois l’espace de noms DFS implémenté, il est possible d’effectuer le basculement de dossiers partagés à partir de sites de production vers des sites de récupération d’urgence en mettant à jour les cibles de dossiers d’espaces de noms DFS. Une fois ces modifications de l’espace de noms DFS répliquées via Active Directory, les utilisateurs sont connectés de manière transparente aux cibles de dossiers appropriées.

## <a name="use-file-sync-to-replicate-your-on-premises-files"></a>Utiliser la synchronisation de fichiers pour répliquer vos fichiers locaux
Vous pouvez utiliser la synchronisation de fichiers pour répliquer des fichiers vers le cloud. En cas d’urgence et d’indisponibilité de votre serveur de fichiers local, vous pouvez monter les emplacements de fichier souhaités à partir du cloud et continuer à traiter les demandes provenant des machines du client.
Pour intégrer la synchronisation de fichiers avec Site Recovery :

* Protégez les machines des serveurs de fichiers à l’aide de Site Recovery. Suivez les étapes dans [ce didacticiel](./vmware-azure-tutorial.md).
* Utilisez la synchronisation de fichiers pour répliquer des fichiers provenant de la machine qui fait office de serveur de fichiers dans le cloud.
* Utilisez la fonctionnalité plan de récupération dans Site Recovery pour ajouter des scripts destinés à monter le partage de fichiers Azure sur la machine virtuelle de serveur de fichiers basculée dans Azure.

Procédez comme suit pour utiliser la synchronisation de fichiers :

1. [Créez un compte de stockage dans Azure](../storage/common/storage-account-create.md?toc=/azure/storage/files/toc.json). Si vous avez choisi le stockage géoredondant avec accès en lecture (recommandé) pour vos comptes de stockage, vous bénéficiez d’un accès en lecture à vos données à partir de la région secondaire en cas d’urgence. Pour plus d’informations, consultez [Récupération d’urgence et basculement de compte de stockage](../storage/common/storage-disaster-recovery-guidance.md?toc=%2fazure%2fstorage%2ffiless%2ftoc.json).
2. [Créer un partage de fichiers](../storage/files/storage-how-to-create-file-share.md).
3. [Déployer la synchronisation de fichiers](../storage/files/storage-sync-files-deployment-guide.md) dans votre serveur de fichiers local.
4. Créez un groupe de synchronisation. Les points de terminaison dans un groupe de synchronisation sont synchronisés entre eux. Un groupe de synchronisation doit contenir au moins un point de terminaison cloud, qui représente un partage de fichiers Azure. Un groupe de synchronisation doit également contenir un point de terminaison de serveur, qui représente un chemin d’accès sur un serveur Windows local.
5. Vos fichiers restent désormais synchronisés sur votre partage de fichiers Azure et sur votre serveur local.
6. En cas d’urgence dans votre environnement local, effectuer un basculement en utilisant un [plan de récupération](site-recovery-create-recovery-plans.md). Ajoutez le script pour monter le partage de fichiers Azure et accéder au partage dans votre machine virtuelle.

> [!NOTE]
> Assurez-vous que le port 445 est ouvert. Azure Files utilise le protocole SMB. SMB communique via le port TCP 445. Vérifiez si votre pare-feu ne bloque pas le port TCP 445 à partir d’une machine du client.


## <a name="do-a-test-failover"></a>Exécuter un test de basculement

1. Accédez au portail Azure et sélectionnez votre coffre Recovery Service.
2. Sélectionnez le plan de récupération créé pour l’environnement de serveurs de fichiers.
3. Sélectionnez **Test de basculement**.
4. Pour démarrer le test de basculement, sélectionnez le point de récupération et le réseau virtuel Azure.
5. Lorsque l’environnement secondaire est opérationnel, effectuez vos validations.
6. Une fois les validations terminées, sélectionnez **Nettoyage du test de basculement** sur le plan de récupération. L’environnement de test de basculement est alors nettoyé.

Pour plus d’informations sur l’exécution d’un basculement de test, consultez [Basculement de test dans Site Recovery](site-recovery-test-failover-to-azure.md).

Pour obtenir des conseils sur l’exécution d’un test de basculement pour Active Directory et DNS, reportez-vous à [Considérations relatives au test de basculement pour Active Directory et DNS](site-recovery-active-directory.md).

## <a name="do-a-failover"></a>Effectuer un basculement

1. Accédez au portail Azure et sélectionnez votre coffre Recovery Services.
2. Sélectionnez le plan de récupération créé pour l’environnement de serveurs de fichiers.
3. Sélectionnez **Basculement**.
4. Pour démarrer le processus de basculement, sélectionnez le point de récupération.

Pour plus d’informations sur le processus de basculement, consultez [Basculement dans Site Recovery](site-recovery-failover.md).

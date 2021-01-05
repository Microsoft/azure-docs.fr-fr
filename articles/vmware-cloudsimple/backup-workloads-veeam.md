---
title: Azure VMware Solution by CloudSimple – Sauvegarder des machines virtuelles de charge de travail sur un cloud privé à l’aide de Veeam
description: Indique comment sauvegarder vos machines virtuelles qui s’exécutent dans un cloud privé CloudSimple basé sur Azure à l’aide de Veeam B&R 9.5
author: sharaths-cs
ms.author: b-shsury
ms.date: 08/16/2019
ms.topic: article
ms.service: azure-vmware-cloudsimple
ms.reviewer: cynthn
manager: dikamath
ms.openlocfilehash: cb883b70d1ae9f2894c7679633a586586922aa18
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92419930"
---
# <a name="back-up-workload-vms-on-cloudsimple-private-cloud-using-veeam-br"></a>Sauvegarder des machines virtuelles de charge de travail sur le cloud privé CloudSimple à l’aide de Veeam B&R

Ce guide vous indique comment sauvegarder vos machines virtuelles qui s’exécutent dans un cloud privé CloudSimple basé sur Azure à l’aide de Veeam B&R 9.5.

## <a name="about-the-veeam-back-up-and-recovery-solution"></a>À propos de la solution de sauvegarde et de récupération Veeam

La solution VEAM comprend les composants suivants :

**Serveur de sauvegarde**

Le serveur de sauvegarde est un serveur Windows (VM) qui sert de centre de contrôle pour Veeam et qui exécute les fonctions suivantes : 

* Coordonne les tâches de sauvegarde, de réplication, de vérification de la récupération et de restauration
* Contrôle la planification des travaux et l’allocation des ressources
* Vous permet de configurer et de gérer les composants de l’infrastructure de sauvegarde et d’indiquer les paramètres globaux de l’infrastructure de sauvegarde

**Serveurs proxy**

Les serveurs proxy sont installés entre le serveur de sauvegarde et les autres composants de l’infrastructure de sauvegarde. Ils gèrent les fonctions suivantes :

* Récupération de données de machine virtuelle à partir du stockage de production
* Compression
* Déduplication
* Chiffrement
* Transmission de données au référentiel de sauvegarde

**Référentiel de sauvegarde**

Le référentiel de sauvegarde est l’emplacement de stockage où Veeam conserve les fichiers de sauvegarde, les copies de machines virtuelles et les métadonnées pour les machines virtuelles répliquées.  Le référentiel peut être un serveur Windows ou Linux avec des disques locaux (ou un NFS/SMB monté) ou une appliance de déduplication du stockage matériel.

### <a name="veeam-deployment-scenarios"></a>Scénarios de déploiement Veeam
Vous pouvez utiliser Azure pour fournir un référentiel de sauvegarde et une cible de stockage pour la sauvegarde et l’archivage à long terme. Tout le trafic réseau de sauvegarde entre les machines virtuelles dans le cloud privé et le référentiel de sauvegarde dans Azure circule sur lien à bande passante élevée et à faible latence. Le trafic de réplication entre les régions circule sur le réseau interne de fond de panier Azure, ce qui réduit les coûts de bande passante pour les utilisateurs.

**Déploiement basique**

Pour les environnements dont la sauvegarde est inférieure à 30 to, CloudSimple recommande la configuration suivante :

* Le serveur de sauvegarde Veeam et le serveur proxy sont installés sur la même machine virtuelle dans le cloud privé.
* Un référentiel de sauvegarde principal basé sur Linux dans Azure configuré comme cible pour les travaux de sauvegarde.
* `azcopy` utilisé pour copier les données du référentiel de sauvegarde principal vers un conteneur d’objets blob Azure répliqué dans une autre région.

![Diagramme montrant les scénarios de déploiement de base de Veeam.](media/veeam-basicdeployment.png)

**Déploiement avancé**

Pour les environnements dont la sauvegarde est inférieure à 30 to, CloudSimple recommande la configuration suivante :

* Un serveur proxy par nœud dans le cluster vSAN, comme recommandé par Veeam.
* Référentiel de sauvegarde principal basé sur Windows dans le cloud privé pour mettre en cache cinq jours de données pour les restaurations rapides.
* Référentiel de sauvegarde Linux dans Azure en tant que cible pour les travaux de copie de sauvegarde pour une durée de rétention plus longue. Ce référentiel doit être configuré en tant que référentiel de sauvegarde scale-out.
* `azcopy` utilisé pour copier les données du référentiel de sauvegarde principal vers un conteneur d’objets blob Azure répliqué dans une autre région.

![Scénarios de déploiement basique](media/veeam-advanceddeployment.png)

Dans la figure précédente, notez que le proxy de sauvegarde est une machine virtuelle disposant d’un accès avec ajout à chaud à des disques de machine virtuelle de charge de travail sur la banque de données vSAN. Veeam utilise le mode de transport du proxy de sauvegarde d’appliance virtuelle pour vSAN.

## <a name="requirements-for-veeam-solution-on-cloudsimple"></a>Exigences pour la solution Veeam sur CloudSimple

Pour profitez de la solution Veeam, veuillez :

* Fournir vos propres licences Veeam.
* Déployez et gérez Veeam pour sauvegarder les charges de travail s’exécutant dans le cloud privé CloudSimple.

Cette solution vous donne un contrôle total sur l’outil de sauvegarde Veeam et vous offre la possibilité d’utiliser l’interface Veeam native ou le plug-in Veeam vCenter pour gérer les travaux de sauvegarde de machine virtuelle.

Si vous êtes déjà un utilisateur de Veeam, vous pouvez ignorer la section sur les composants de solution Veeam et passer directement aux [scénarios de déploiement Veeam](#veeam-deployment-scenarios).

## <a name="install-and-configure-veeam-backups-in-your-cloudsimple-private-cloud"></a>Installez et configurez des sauvegardes Veeam dans votre Cloud privé CloudSimple

Les sections suivantes vous indiquent comment installer et configurer une solution de sauvegarde Veeam pour votre Cloud privé CloudSimple.

Le processus de déploiement comprend les étapes suivantes :

1. [Interface utilisateur de vCenter : Configurer des services d'infrastructure dans votre cloud privé](#vcenter-ui-set-up-infrastructure-services-in-your-private-cloud)
2. [Portail CloudSimple : Configurer un réseau de cloud privé pour Veeam](#cloudsimple-private-cloud-set-up-private-cloud-networking-for-veeam)
3. [Portail CloudSimple : Élevez les privilèges](#cloudsimple-private-cloud-escalate-privileges-for-cloudowner)
4. [Portail Azure : Connecter votre réseau virtuel au cloud privé](#azure-portal-connect-your-virtual-network-to-the-private-cloud)
5. [Portail Azure : Créer un référentiel de sauvegarde dans Azure](#azure-portal-connect-your-virtual-network-to-the-private-cloud)
6. [Portail Azure : Configurer le stockage d’objets blob Azure pour la conservation des données à long terme](#configure-azure-blob-storage-for-long-term-data-retention)
7. [Interface utilisateur vCenter du Cloud privé : Installez Veeam B&R](#vcenter-console-of-private-cloud-install-veeam-br)
8. [Console Veeam : Configurer la sauvegarde Veeam et le logiciel de récupération](#veeam-console-install-veeam-backup-and-recovery-software)
9. [Portail CloudSimple : Configurez l'accès Veeam et annulez l’élévation des privilèges](#cloudsimple-portal-set-up-veeam-access-and-de-escalate-privileges)

### <a name="before-you-begin"></a>Avant de commencer

Avant de commencer le déploiement de Veeam, vous devez disposer des éléments suivants :

* Un abonnement Azure dont vous êtes propriétaire
* Un groupe de ressources Azure créé au préalable
* Un réseau virtuel Azure dans votre abonnement
* Un compte de stockage Azure.
* Un [cloud privé](create-private-cloud.md) créé à l'aide du portail CloudSimple.  

Les éléments suivants sont nécessaires pendant la phase d’implémentation :

* Modèles VMware pour Windows pour l’installation de Veeam (par exemple, Windows Server 2012 R2 – 64 images-bits)
* Un réseau local virtuel (VLAN) disponible désigné pour le réseau de sauvegarde
* CIDR du sous-réseau à attribuer au réseau de sauvegarde
* Support d’installation Veeam 9.5 u3 (ISO) chargé dans la banque de données vSAN du cloud privé

### <a name="vcenter-ui-set-up-infrastructure-services-in-your-private-cloud"></a>Interface utilisateur de vCenter : Configurer des services d'infrastructure dans votre cloud privé

Configurez des services d'infrastructure dans le cloud privé afin de faciliter la gestion de vos charges de travail et de vos outils.

* Vous pouvez ajouter un fournisseur d’identité externe comme indiqué dans [Configurer les sources d’identité vCenter pour utiliser Active Directory](set-vcenter-identity.md) si l’une des conditions suivantes s’applique :

  * Vous souhaitez identifier des utilisateurs à partir de votre Active Directory (AD) local dans votre cloud privé.
  * Vous souhaitez configurer une instance d'AD dans votre cloud privé pour tous les utilisateurs.
  * Vous souhaitez utiliser Azure AD.
* Pour fournir des services de recherche d'adresses IP, de gestion des adresses IP et de résolution de noms pour vos charges de travail dans le cloud privé, configurez un serveur DNS et DHCP comme décrit dans [Configurer des applications et des charges de travail DNS et DHCP dans votre cloud privé CloudSimple](dns-dhcp-setup.md).

### <a name="cloudsimple-private-cloud-set-up-private-cloud-networking-for-veeam"></a>Cloud privé CloudSimple : Configurer un réseau de cloud privé pour Veeam

Accédez au portail CloudSimple afin de configurer un réseau de cloud privé pour la solution Veeam.

Créez un réseau local virtuel (VLAN) pour le réseau de sauvegarde et attribuez-lui un CIDR de sous-réseau. Pour plus d'informations, consultez [Créer et gérer des réseaux VLAN/sous-réseaux](create-vlan-subnet.md).

Créez des règles de pare-feu entre le sous-réseau de gestion et le réseau de sauvegarde pour autoriser le trafic réseau sur les ports utilisés par Veeam. Consultez la rubrique Veeam [Ports utilisés](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95). Pour plus d'informations sur la création de règle de pare-feu, consultez la page [Configurer des tables et des règles de pare-feu](firewall.md).

Le tableau suivant fournit une liste des ports.

| Icône | Description | Icône | Description |
| ------------ | ------------- | ------------ | ------------- |
| Serveur de sauvegarde  | vCenter  | HTTPS/TCP  | 443 |
| Serveur de sauvegarde <br> *Requis pour le déploiement des composants de sauvegarde et de réplication de Veeam* | Proxy de sauvegarde  | TCP/UDP  | 135, 137 à 139 et 445 |
    | Serveur de sauvegarde   | DNS  | UDP  | 53  | 
    | Serveur de sauvegarde   | Serveur de notification de mise à jour Veeam  | TCP  | 80  | 
    | Serveur de sauvegarde   | Serveur de mise à jour de licences Veeam  | TCP  | 443  | 
    | Proxy de sauvegarde   | vCenter |   |   | 
    | Proxy de sauvegarde  | Référentiel de sauvegarde Linux   | TCP  | 22  | 
    | Proxy de sauvegarde  | Référentiel de sauvegarde Windows  | TCP  | 49152 - 65535   | 
    | Référentiel de sauvegarde Windows  | Proxy de sauvegarde  | TCP  | 2500 – 5000  | 
    | Référentiel de sauvegarde source<br> *Utilisé pour les travaux de copie de sauvegarde*  | Référentiel de sauvegarde cible  | TCP  | 2500 – 5000  | 

Créez des règles de pare-feu entre le sous-réseau de charge de travail et le réseau de sauvegarde, comme indiqué à la page [Configurer des tables et des règles de pare-feu](firewall.md).  Pour la sauvegarde et la restauration prenant en charge les applications, [des ports supplémentaires](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95) doivent être ouverts sur les machines virtuelles de charge de travail qui hébergent des applications particulères.

Par défaut, CloudSimple fournit un lien 1 Gbit/s ExpressRoute. Un lien de bande passante plus élevée peut être nécessaire pour les environnements de plus grande taille. Pour plus d’informations sur les liens de bande passante plus élevée, contactez le support Azure.

Pour poursuivre la configuration, vous avez besoin de la clé d’autorisation, de l’URI du circuit pair et de l’accès à votre abonnement Azure.  Ces informations sont disponibles sur la page Connexion au réseau virtuel du portail CloudSimple. Pour plus d’instructions, consultez [Obtenir des informations de peering pour la connexion entre le réseau virtuel Azure et CloudSimple](virtual-network-connection.md). Si vous ne parvenez pas à obtenir ces informations, [contactez le support](https://portal.azure.com/#blade/Microsoft_Azure_Support/HelpAndSupportBlade/newsupportrequest).

### <a name="cloudsimple-private-cloud-escalate-privileges-for-cloudowner"></a>Cloud privé CloudSimple : Élever les privilèges pour cloudowner

L'utilisateur « cloudowner » par défaut ne dispose pas des privilèges suffisants pour installer Veeam dans le vCenter de cloud privé. Les privilèges vCenter de l'utilisateur doivent donc être élevés. Pour plus d'informations, consultez [Élever les privilèges](escalate-private-cloud-privileges.md).

### <a name="azure-portal-connect-your-virtual-network-to-the-private-cloud"></a>Portail Azure : Connecter votre réseau virtuel au cloud privé

Connectez votre réseau virtuel au cloud privé en suivant les instructions de [Connexion au réseau virtuel Azure à l'aide d’ExpressRoute](azure-expressroute-connection.md).

### <a name="azure-portal-create-a-backup-repository-vm"></a>Portail Azure : Créer une machine virtuelle de référentiel de sauvegarde

1. Créez une machine virtuelle standard D2 v3 avec (2 processeurs virtuels et 8 Go de mémoire).
2. Sélectionnez l’image basée sur CentOS 7.4.
3. Configurez un groupe de sécurité réseau (NSG) pour la machine virtuelle. Vérifiez que la machine virtuelle n’a pas d’adresse IP publique et qu’elle n’est pas accessible à partir de l’Internet public.
4. Créez un compte d’utilisateur avec un nom d’utilisateur et un mot de passe pour la machine virtuelle. Pour plus d’informations, consultez la page [Créer une machine virtuelle Linux dans le portail Azure](../virtual-machines/linux/quick-create-portal.md).
5. Créez un Gio HDD standard 1 x 512 et joignez-le à la machine virtuelle du référentiel.  Pour plus d’informations, consultez la page [Joindre un disque de données managées à une machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/attach-managed-disk-portal.md).
6. [Créez un volume XFS sur le disque managé](https://www.digitalocean.com/docs/volumes/how-to/). Connectez-vous à la machine virtuelle à l’aide des informations d’identification mentionnées précédemment. Exécutez le script suivant pour créer un volume logique. Ajoutez-y le disque, créez une [partition](https://www.digitalocean.com/docs/volumes/how-to/partition/) de système de fichiers XFS et [montez](https://www.digitalocean.com/docs/volumes/how-to/mount/) la partition sous le chemin /backup1.

    Exemple de script :

    ```
    sudo pvcreate /dev/sdc
    sudo vgcreate backup1 /dev/sdc
    sudo lvcreate -n backup1 -l 100%FREE backup1
    sudo mkdir -p /backup1
    sudo chown veeamadmin /backup1
    sudo chmod 775 /backup1
    sudo mkfs.xfs -d su=64k -d sw=1 -f /dev/mapper/backup1-backup1
    sudo mount -t xfs /dev/mapper/backup1-backup1 /backup1
    ```

7. Exposez /backup1 comme point de montage NFS sur le serveur de sauvegarde Veeam qui s’exécute dans le cloud privé. Pour plus d’informations, consultez la page [Configuration d’un montage NFS sur CentOS 6](https://www.digitalocean.com/community/tutorials/how-to-set-up-an-nfs-mount-on-centos-6). Utilisez ce nom de partage NFS lorsque vous configurez le référentiel de sauvegarde dans le serveur de sauvegarde Veeam.

8. Configurez les règles de filtrage dans le groupe de sécurité réseau pour la machine virtuelle du référentiel de sauvegarde afin d’autoriser explicitement tout le trafic réseau vers la machine virtuelle ou depuis celle-ci.

> [!NOTE]
> Veeam Backup & Replication utilise le protocole SSH pour communiquer avec les référentiels de sauvegarde Linux et requiert l’utilitaire SCP sur les référentiels Linux. Vérifiez que le démon SSH est correctement configuré et que le SCP est disponible sur l’hôte Linux.

### <a name="configure-azure-blob-storage-for-long-term-data-retention"></a>Configurer le stockage d’objets blob Azure pour la conservation des données à long terme

1. Créez un compte de stockage à usage général (GPv2) de type standard et un conteneur d’objets blob, comme indiqué dans la vidéo de Microsoft [Prise en main du stockage Azure](https://azure.microsoft.com/resources/videos/get-started-with-azure-storage).
2. Créez un conteneur de stockage Azure, comme indiqué dans la référence [Créer un conteneur](/rest/api/storageservices/create-container).
2. Téléchargez `azcopy` l’utilitaire de ligne de commande pour Linux à partir de Microsoft. Vous pouvez utilisez les commandes suivantes dans l’interpréteur de commandes Bash de CentOS 7.5.

    ```
    wget -O azcopy.tar.gz https://aka.ms/downloadazcopylinux64
    tar -xf azcopy.tar.gz
    sudo ./install.sh
    sudo yum -y install libunwind.x86_64
    sudo yum -y install icu
    ```

3. Utilisez la `azcopy` commande pour copier les fichiers de sauvegarde vers et à partir du conteneur d’objets blob.  Pour plus d’informations sur les commandes, consultez la page [Transférer des données avec AzCopy sur Linux](../storage/common/storage-use-azcopy-v10.md).

### <a name="vcenter-console-of-private-cloud-install-veeam-br"></a>Console vCenter du cloud privé : Installez Veeam B&R

Accédez à vCenter à partir de votre cloud privé pour créer un compte de service Veeam, installer Veeam B&R 9.5 et configurer Veeam à l’aide du compte de service.

1. Créez un nouveau rôle nommé « rôle Veeam Backup » et attribuez-lui les autorisations nécessaires, comme recommandé par Veeam. Pour plus d’informations, consultez la rubrique Veeam [Autorisations nécessaires](https://helpcenter.veeam.com/docs/backup/vsphere/required_permissions.html?ver=95).
2. Créez un nouveau groupe « groupe d’utilisateurs Veeam » dans vCenter et attribuez-lui le « Veeam Backup ».
3. Créez un nouvel utilisateur « compte de service Veeam» et ajoutez-le au « groupe d’utilisateurs Veeam ».

    ![Création d’un compte de service Veeam](media/veeam-vcenter01.png)

4. Créez un groupe de ports distribués dans vCenter à l’aide du réseau local virtuel de sauvegarde. Pour plus d’informations, consultez la vidéo VMware [Création d’un groupe de ports distribués dans le client Web vSphere](https://www.youtube.com/watch?v=wpCd5ZbPOpA).
5. Créez les machines virtuelles pour Veeam Backup et les serveurs proxy dans vCenter, conformément aux [configurations requises Veeam](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html?ver=95). Vous pouvez utiliser Windows 2012 R2 ou Linux. Pour plus d’informations, consultez la page [Exigences relatives à l’utilisation des référentiels de sauvegarde Linux](https://www.veeam.com/kb2216).
6. Montez le Veeam ISO installable en tant que périphérique CD-ROM dans la machine virtuelle du serveur Veeam Backup.
7. À l’aide d’une session RDP sur l’ordinateur Windows 2012 R2 (la cible de l’installation de Veeam), [installez Veeam B&R 9.5u3](https://helpcenter.veeam.com/docs/backup/vsphere/install_vbr.html?ver=95) dans une machine virtuelle Windows 2012 R2.
8. Recherchez l’adresse IP interne de la machine virtuelle du serveur Veeam Backup et configurez l’adresse IP pour qu’elle soit statique sur le serveur DHCP. Les étapes exactes requises dépendent du serveur DHCP. Par exemple, les <a href="https://www.netgate.com/docs/pfsense/dhcp/dhcp-server.html" target="_blank">mappages statiques DHCP</a> de l’article Netgate explique comment configurer un serveur DHCP à l’aide d’un routeur pfSense.

### <a name="veeam-console-install-veeam-backup-and-recovery-software"></a>Console Veeam : Installez le logiciel de sauvegarde et de récupération Veeam

À l’aide de la console Veeam, configurez le logiciel de sauvegarde et de récupération Veeam. Pour plus d’informations, consultez la page [Veeam Backup & Replication v9 – Installation et déploiement](https://www.youtube.com/watch?v=b4BqC_WXARk).

1. Ajoutez VMware vSphere en tant qu’environnement de serveur managé. Lorsque vous y êtes invité, fournissez les informations d’identification du compte de service Veeam que vous avez créé au début de la [console vCenter du Cloud privé : Installez Veeam B&R](#vcenter-console-of-private-cloud-install-veeam-br).

    * Utilisez les paramètres par défaut pour le contrôle de charge et les paramètres avancés par défaut.
    * Définissez l’emplacement du serveur de montage sur le serveur de sauvegarde.
    * Remplacez l’emplacement de sauvegarde de la configuration du serveur Veeam par le référentiel distant.

2. Ajoutez le serveur Linux dans Azure en tant que référentiel de sauvegarde.

    * Utilisez les paramètres par défaut pour le contrôle de charge et les paramètres avancés par défaut. 
    * Définissez l’emplacement du serveur de montage sur le serveur de sauvegarde.
    * Remplacez l’emplacement de sauvegarde de la configuration du serveur Veeam par le référentiel distant.

3. Activez le chiffrement de la sauvegarde de configuration en cliquant sur **Accueil > Paramètres de sauvegarde de configuration**.

4. Ajoutez une machine virtuelle Windows Server en tant que serveur proxy pour l’environnement VMware. À l’aide de « règles de trafic » pour un proxy, chiffrez les données de sauvegarde sur le réseau.

5. Configuration des travaux de sauvegarde.
    * Pour configurer des travaux de sauvegarde, suivez les instructions de la page [Créer un travail de sauvegarde](https://www.youtube.com/watch?v=YHxcUFEss4M).
    * Activez le chiffrement des fichiers de sauvegarde dans **Paramètres avancés > Stockage**.

6. Configuration des travaux de copie de sauvegarde.

    * Pour configurer des travaux de copie de sauvegarde, suivez les instructions de la page [Créer un travail de copie sauvegarde](https://www.youtube.com/watch?v=LvEHV0_WDWI&t=2s).
    * Activez le chiffrement des fichiers de sauvegarde dans **Paramètres avancés > Stockage**.

### <a name="cloudsimple-portal-set-up-veeam-access-and-de-escalate-privileges"></a>Portail CloudSimple : Configurez l'accès Veeam et annulez l’élévation des privilèges
Créez une adresse IP publique pour le serveur de sauvegarde et de récupération Veeam. Pour plus d'informations, consultez [Allouer des adresses IP publiques](public-ips.md).

Créez une règle de pare-feu permettant au serveur de sauvegarde Veeam de créer une connexion sortante vers le site web Veeam pour télécharger les mises à jour/correctifs sur le port TCP 80. Pour plus d'informations, consultez [Configurer des règles et tables de pare-feu](firewall.md).

Pour annuler l'élévation des privilèges, consultez [Annuler l'élévation des privilèges](escalate-private-cloud-privileges.md#de-escalate-privileges).

## <a name="references"></a>References

### <a name="cloudsimple-references"></a>Références CloudSimple

* [Créer un cloud privé](create-private-cloud.md)
* [Créer et gérer des réseaux locaux virtuels/sous-réseaux](create-vlan-subnet.md)
* [Sources d’identité vCenter](set-vcenter-identity.md)
* [Configuration du DNS et du DHCP des charges de travail](dns-dhcp-setup.md)
* [Élevez les privilèges](escalate-privileges.md)
* [Configurer des règles et tables de pare-feu](firewall.md)
* [Autorisations du cloud privé](learn-private-cloud-permissions.md)
* [Allouer des adresses IP publiques](public-ips.md)

### <a name="veeam-references"></a>Références Veeam

* [Ports utilisés](https://helpcenter.veeam.com/docs/backup/vsphere/used_ports.html?ver=95)
* [Autorisations requises](https://helpcenter.veeam.com/docs/backup/vsphere/required_permissions.html?ver=95)
* [Configuration requise](https://helpcenter.veeam.com/docs/backup/vsphere/system_requirements.html?ver=95)
* [Installation de Veeam Backup & Replication](https://helpcenter.veeam.com/docs/backup/vsphere/install_vbr.html?ver=95)
* [Autorisations et modules et pour le support des systèmes d'exploitation multiples FLR et du référentiel pour Linux](https://www.veeam.com/kb2216)
* [Veeam Backup & Replication v9 – Installation et déploiement – Vidéo](https://www.youtube.com/watch?v=b4BqC_WXARk)
* [Veeam v9 – Création d’un travail de sauvegarde – Vidéo](https://www.youtube.com/watch?v=YHxcUFEss4M)
* [Veeam v9 – Création d’un travail de copie de sauvegarde – Vidéo](https://www.youtube.com/watch?v=LvEHV0_WDWI&t=2s)

### <a name="azure-references"></a>Références Azure

* [Configurer une passerelle de réseau virtuel pour ExpressRoute à l’aide du portail Azure](../expressroute/expressroute-howto-add-gateway-portal-resource-manager.md)
* [Connecter un réseau virtuel à un circuit – autre abonnement](../expressroute/expressroute-howto-linkvnet-portal-resource-manager.md#connect-a-vnet-to-a-circuit---different-subscription)
* [Créer une machine virtuelle Linux dans le portail Azure](../virtual-machines/linux/quick-create-portal.md)
* [Joindre un disque de données managées à une machine virtuelle Windows dans le portail Azure](../virtual-machines/windows/attach-managed-disk-portal.md)
* [Prise en main d’un stockage Azure – Vidéo](https://azure.microsoft.com/resources/videos/get-started-with-azure-storage)
* [Créer un conteneur](/rest/api/storageservices/create-container)
* [Transférer des données avec AzCopy sur Linux](../storage/common/storage-use-azcopy-v10.md)

### <a name="vmware-references"></a>Références VMware

* [Création d’un groupe de ports distribués dans le client web vSphere– Vidéo](https://www.youtube.com/watch?v=wpCd5ZbPOpA)

### <a name="other-references"></a>Autres références

* [Créez un volume XFS sur le disque managé – RedHat](https://access.redhat.com/documentation/en-us/red_hat_enterprise_linux/7/html/storage_administration_guide/ch-xfs)
* [Configuration d’un montage NFS sur CentOS 7– HowToForge](https://www.howtoforge.com/nfs-server-and-client-on-centos-7)
* [Configuration du serveur DHCP – Netgate](https://www.netgate.com/docs/pfsense/dhcp/dhcp-server.html)

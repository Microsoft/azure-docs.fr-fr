---
title: Configurer la récupération d’urgence Citrix XenDesktop/XenApp avec Azure Site Recovery
description: Cet article décrit comment configurer la récupération d’urgence des déploiements Citrix XenDesktop et XenApp à l’aide d’Azure Site Recovery.
author: ponatara
manager: abhemraj
ms.service: site-recovery
ms.topic: conceptual
ms.date: 11/27/2018
ms.author: ponatara
ms.openlocfilehash: 90d54a8ded99dd8ab43aed688036add6aede20ab
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86134844"
---
# <a name="set-up-disaster-recovery-for-a-multi-tier-citrix-xenapp-and-xendesktop-deployment"></a>Configurer la récupération d’urgence d’un déploiement Citrix XenApp et XenDesktop multiniveau



Citrix XenDesktop est une solution de virtualisation de bureau qui fournit des postes de travail et des applications sous la forme d’un service à la demande à n’importe quel utilisateur, n’importe où. Avec la technologie de remise FlexCast, XenDesktop peut fournir rapidement et en toute sécurité des applications et des postes de travail aux utilisateurs.
Aujourd’hui, Citrix XenApp ne fournit aucune fonctionnalité de récupération d’urgence.

Une solution de récupération d’urgence satisfaisante doit autoriser la modélisation des plans de récupération autour des architectures d’application complexes décrites ci-dessus. Elle doit également permettre d’ajouter des étapes personnalisées pour gérer les mappages d’applications entre les différents niveaux, assurant ainsi une solution à clic unique en cas de sinistre entraînant un délai de récupération inférieur.

Ce document fournit des instructions pas à pas pour la création d’une solution de récupération d’urgence pour vos déploiements locaux de Citrix XenApp sur les plateformes Hyper-V et VMware vSphere. Il décrit également comment effectuer un test de basculement (exercice de récupération d’urgence) et un basculement non planifié vers Azure à l’aide de plans de récupération, ainsi que les configurations prises en charge et les prérequis.


## <a name="prerequisites"></a>Prérequis

Avant de commencer, veillez à bien comprendre ce qui suit :

1. [Réplication d’une machine virtuelle dans Azure](./vmware-azure-tutorial.md)
1. [Conception d’un réseau de récupération](./concepts-on-premises-to-azure-networking.md)
1. [Réalisation d’un test de basculement vers Azure](site-recovery-test-failover-to-azure.md)
1. [Exécution d’un basculement vers Azure](site-recovery-failover.md)
1. [Réplication d’un contrôleur de domaine](site-recovery-active-directory.md)
1. [Réplication de SQL Server](site-recovery-sql.md)

## <a name="deployment-patterns"></a>Modèles de déploiement

Une batterie de serveurs Citrix XenApp et XenDesktop présente généralement le modèle de déploiement suivant :

**Modèle de déploiement**

Déploiement Citrix XenApp et XenDesktop avec serveur DNS Active Directory, serveur SQL Database, Citrix Delivery Controller, serveur StoreFront, XenApp Master (VDA) et serveur de licences Citrix XenApp

![Modèle de déploiement 1](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-deployment.png)


## <a name="site-recovery-support"></a>Prise en charge de Site Recovery

Pour les besoins de cet article, des déploiements de Citrix sur des machines virtuelles VMware gérées par vSphere 6.0 / System Center VMM 2012 R2 ont été utilisés pour la configuration de la récupération d’urgence.

### <a name="source-and-target"></a>Source et cible

**Scénario** | **Vers un site secondaire** | **Vers Azure**
--- | --- | ---
**Hyper-V** | Non compris | Oui
**VMware** | Non compris | Oui
**Serveur physique** | Non compris | Oui

### <a name="versions"></a>Versions
Les clients peuvent déployer des composants XenApp en tant que machines virtuelles s’exécutant sur Hyper-V ou VMware ou en tant que serveurs physiques. Azure Site Recovery peut protéger des déploiements physiques et virtuels sur Azure.
XenApp 7.7 ou ultérieur étant pris en charge dans Azure, seuls les déploiements avec ces versions peuvent être basculés vers Azure pour la migration ou la récupération d’urgence.

### <a name="things-to-keep-in-mind"></a>Points à prendre en compte

1. La protection et la récupération des déploiements locaux à l’aide de machines de système d’exploitation de serveur pour délivrer des applications XenApp publiées et des postes de travail XenApp publiés sont prises en charge.

2. La protection et la récupération des déploiements locaux à l’aide de machines de système d’exploitation de bureau pour fournir une infrastructure VDI (Virtual Desktop Infrastructure) pour des postes de travail virtuels clients, notamment Windows 10, ne sont pas prises en charge. En effet, Site Recovery ne prend pas en charge la récupération des ordinateurs exécutant des systèmes d’exploitation de bureau.  De plus, certains systèmes d’exploitation de bureau virtuels clients (par exemple Windows 7) ne sont pas encore prises en charge pour la gestion des licences dans Azure. [Apprenez-en plus](https://azure.microsoft.com/pricing/licensing-faq/) sur les licences pour les bureaux client/serveur dans Azure.

3.  Azure Site Recovery ne peut pas répliquer et protéger les clones MCS ou PVS locaux existants.
Vous devez recréer ces clones à l’aide de l’approvisionnement Azure ARM à partir du Delivery Controller.

4. NetScaler ne peut pas être protégé à l’aide d’Azure Site Recovery, car NetScaler repose sur FreeBSD et Azure Site Recovery ne prend pas en charge la protection du système d’exploitation FreeBSD. Vous devez déployer et configurer un nouveau dispositif NetScaler à partir de la Place de marché Azure après le basculement vers Azure.


## <a name="replicating-virtual-machines"></a>Réplication des machines virtuelles

Les composants suivants du déploiement Citrix XenApp doivent être protégés pour activer la réplication et la récupération.

* Protection du serveur DNS Active Directory
* Protection du serveur SQL Database
* Protection de Citrix Delivery Controller
* Protection du serveur StoreFront
* Protection de XenApp Master (VDA)
* Protection du serveur de licences Citrix XenApp


**Réplication du serveur DNS Active Directory**

Pour obtenir des instructions pour la réplication et la configuration d’un contrôleur de domaine dans Azure, consultez [Protéger Active Directory et DNS avec Azure Site Recovery](site-recovery-active-directory.md).

**Réplication du serveur SQL Database**

Pour obtenir des instructions techniques détaillées sur les options recommandées pour la protection des serveurs SQL, consultez [Protéger SQL Server avec la récupération d’urgence SQL Server et Azure Site Recovery](site-recovery-sql.md).

Suivez [ces instructions](./vmware-azure-tutorial.md) pour commencer à répliquer les autres machines virtuelles vers Azure.

![Protection des composants XenApp](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-enablereplication.png)

**Paramètres Calcul et réseau**

Une fois les machines protégées (l’état indiqué est « Protégé » sous les éléments répliqués), vous devez configurer les paramètres Calcul et réseau.
Dans Calcul et réseau > Propriétés de calcul, vous pouvez spécifier la taille de la cible et le nom de la machine virtuelle Azure.
Si besoin, modifiez ce nom afin de respecter les exigences d’Azure. Vous pouvez également afficher et ajouter des informations sur le réseau cible, le sous-réseau et l’adresse IP qui seront affectés à la machine virtuelle Azure.

Notez les points suivants :

* Vous pouvez définir l’adresse IP cible. Si vous ne fournissez pas d’adresse IP, la machine ayant basculé utilisera le service DHCP. Si vous définissez une adresse qui n’est pas disponible au moment du basculement, ce dernier échoue. Vous pouvez utiliser la même adresse IP cible pour le test de basculement si cette adresse est disponible sur le réseau de test de basculement.

* Pour le serveur DNS/Active Directory, conserver l’adresse locale vous permet de spécifier la même adresse que le serveur DNS pour le réseau virtuel Azure.

Le nombre de cartes réseau est déterminé par la taille spécifiée pour la machine virtuelle cible, comme suit :

*   Si le nombre de cartes réseau sur la machine source est inférieur ou égal au nombre de cartes autorisé pour la taille de la machine cible, la cible présente le même nombre de cartes que la source.
*   Si le nombre de cartes de la machine virtuelle source dépasse la valeur de taille cible autorisée, la taille cible maximale est utilisée.
* Par exemple, si une machine source présente deux cartes réseau et que la taille de la machine cible en accepte quatre, la machine cible présentera deux cartes. Si la machine source inclut deux cartes, mais que la taille cible prise en charge accepte une seule carte, la machine cible présentera une seule carte.
*   Si la machine virtuelle possède plusieurs cartes réseau, elles se connectent toutes au même réseau.
*   Si la machine virtuelle a plusieurs cartes réseau, la première qui s’affiche dans la liste devient la carte réseau par défaut dans la machine virtuelle Azure.


## <a name="creating-a-recovery-plan"></a>Création d’un plan de récupération

Une fois la réplication activée pour les machines virtuelles XenApp, l’étape suivante consiste à créer un plan de récupération.
Un plan de récupération regroupe les machines virtuelles ayant les mêmes exigences de basculement et de récupération.  

**Étapes pour créer un plan de récupération**

1. Ajoutez les machines virtuelles XenApp au Plan de récupération.
2. Cliquez sur Plans de récupération -> + Plan de récupération. Spécifiez un nom intuitif pour le plan de récupération.
3. Pour les machines virtuelles VMware : sélectionnez Serveur de processus VMware comme source, Microsoft Azure comme cible et Resource Manager comme modèle de déploiement, puis cliquez sur Sélectionner les éléments.
4. Pour les machines virtuelles Hyper-V : sélectionnez Serveur VMM comme source, Microsoft Azure comme cible et Resource Manager comme modèle de déploiement, cliquez sur Sélectionner les éléments, puis sélectionnez les machines virtuelles de déploiement XenApp.

### <a name="adding-virtual-machines-to-failover-groups"></a>Ajout de machines virtuelles à des groupes de basculement

Vous pouvez personnaliser les plans de récupération pour ajouter des groupes de basculement pour un ordre de démarrage, des scripts ou des actions manuelles spécifiques. Les groupes suivants doivent être ajoutés au plan de récupération.

1. Groupe de basculement1 : AD DNS
2. Groupe de basculement2 : Machines virtuelles SQL Server
2. Groupe de basculement3 : Machine virtuelle d’image VDA maître
3. Groupe de basculement4 : Delivery Controller et machines virtuelles de serveur StoreFront


### <a name="adding-scripts-to-the-recovery-plan"></a>Ajout de scripts au plan de récupération

Vous pouvez exécuter des scripts avant ou après un groupe spécifique dans un plan de récupération. Vous pouvez aussi inclure et effectuer des actions manuelles pendant le basculement.

Le plan de récupération personnalisé ressemble à ceci :

1. Groupe de basculement1 : AD DNS
2. Groupe de basculement2 : Machines virtuelles SQL Server
3. Groupe de basculement3 : Machine virtuelle d’image VDA maître

   >[!NOTE]     
   >Les étapes 4, 6 et 7 contenant des actions de script ou manuelles s’appliquent uniquement à un environnement XenApp local avec des catalogues MCS/PVS.

4. Action manuelle ou de script de Groupe 3 : Arrêter machines virtuelles VDA maîtres.
Lorsqu’elle est basculée sur Azure, la machine virtuelle VDA maître est en cours d’exécution. Pour créer des catalogues MCS à l’aide de l’hébergement Azure, la machine virtuelle VDA maîtresse doit être à l’état Arrêtée (désallouée). Arrêtez la machine virtuelle à partir du Portail Azure.

5. Groupe de basculement4 : Delivery Controller et machines virtuelles de serveur StoreFront
6. Action manuelle ou de script 1 de Groupe 3 :

    ***Ajouter une connexion hôte Azure ARM***

    Créez une connexion hôte Azure sur l’ordinateur Delivery Controller pour approvisionner de nouveaux catalogues MCS dans Azure. Suivez les étapes décrites dans [cet article](https://www.citrix.com/blogs/2016/07/21/connecting-to-azure-resource-manager-in-xenapp-xendesktop/).

7. Action manuelle ou de script 2 de Groupe 3 :

    ***Recréer les catalogues MCS dans Azure***

    Les clones MCS ou PVS existants sur le site principal ne seront pas répliqués vers Azure. Vous devez recréer ces clones à l’aide de la VDA maîtresse répliquée et de l’approvisionnement Azure à partir du Delivery Controller. Suivez les étapes de cet [article](https://www.citrix.com/blogs/2016/09/12/using-xenapp-xendesktop-in-azure-resource-manager/) pour créer des catalogues MCS dans Azure.

![Plan de récupération pour les composants XenApp](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-recoveryplan.png)


   >[!NOTE]
   >Vous pouvez utiliser des scripts à l’[emplacement](https://github.com/Azure/azure-quickstart-templates/tree/master/asr-automation-recovery/scripts) pour mettre à jour le système DNS avec les nouvelles adresses IP des machines virtuelles basculées ou pour attacher un équilibreur de charge sur la machine virtuelle basculée, si nécessaire.


## <a name="doing-a-test-failover"></a>Exécution d’un test de basculement

Suivez [ce guide](site-recovery-test-failover-to-azure.md) pour effectuer un test de basculement.

![Plan de récupération](./media/site-recovery-citrix-xenapp-and-xendesktop/citrix-tfo.png)


## <a name="doing-a-failover"></a>Exécution d’un basculement

Suivez [ce guide](site-recovery-failover.md) lorsque vous effectuez un basculement.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la réplication des déploiements Citrix XenApp et XenDesktop, consultez [ce livre blanc](https://aka.ms/citrix-xenapp-xendesktop-with-asr). Examinez les conseils pour [répliquer d’autres applications](site-recovery-workload.md) à l’aide de Site Recovery.

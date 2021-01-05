---
title: Se connecter à des machines virtuelles Azure en basculement local avec Azure Site Recovery
description: Explique comment se connecter à des machines virtuelles Azure après le basculement d’un environnement local vers Azure à l’aide d’Azure Site Recovery
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/13/2019
ms.author: harshacs
ms.openlocfilehash: 123a68885346062b9e8a53b8d5066204b6b20f5e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89568776"
---
# <a name="connect-to-azure-vms-after-failover-from-on-premises"></a>Se connecter à des machines virtuelles Azure après le basculement d’un environnement local 


Cet article explique comment configurer la connectivité en vue de se connecter à des machines virtuelles Azure après un basculement.

Quand vous configurez la récupération d’urgence pour des machines virtuelles locales et des serveurs physiques vers Azure, [Azure Site Recovery](site-recovery-overview.md) lance la réplication des machines sur Azure. Par la suite, quand des interruptions se produisent, vous pouvez basculer vers Azure à partir de votre site local. Quand le basculement se produit, Site Recovery crée des machines virtuelles Azure en utilisant les données locales répliquées. Au moment de planifier la récupération d’urgence, vous devez trouver le moyen de vous connecter aux applications qui s’exécutent sur ces machines virtuelles Azure après le basculement.

Dans cet article, vous apprendrez comment :

> [!div class="checklist"]
> * Préparer les ordinateurs locaux avant le basculement.
> * Préparer les machines virtuelles Azure après le basculement. 
> * Conserver les adresses IP sur les machines virtuelles Azure après le basculement.
> * Attribuer de nouvelles adresses IP aux machines virtuelles Azure après le basculement.

## <a name="prepare-on-premises-machines"></a>Préparer les ordinateurs locaux

Pour assurer la connectivité avec les machines virtuelles Azure, préparez vos ordinateurs locaux avant le basculement.

### <a name="prepare-windows-machines"></a>Préparer les ordinateurs Windows

Sur les ordinateurs Windows locaux, effectuez les tâches suivantes :

1. Configurez les paramètres Windows. Cela consiste notamment à supprimer les routes statiques persistantes ou le proxy WinHTTP et à définir la stratégie SAN des disques sur **OnlineAll**. [Suivez](../virtual-machines/windows/prepare-for-upload-vhd-image.md#set-windows-configurations-for-azure) ces instructions.

2. Vérifiez que [ces services](../virtual-machines/windows/prepare-for-upload-vhd-image.md#check-the-windows-services) sont en cours d’exécution.

3. Activez le bureau à distance (RDP) pour autoriser les connexions à distance à l’ordinateur local. [Découvrez comment](../virtual-machines/windows/prepare-for-upload-vhd-image.md#update-remote-desktop-registry-settings) activer RDP avec PowerShell.

4. Pour accéder à une machine virtuelle Azure via Internet après le basculement, dans le Pare-feu Windows de l’ordinateur local, autorisez TCP et UDP dans le profil public, puis définissez RDP comme application autorisée pour tous les profils.

5. Si vous voulez accéder à une machine virtuelle Azure via un réseau privé virtuel (VPN) site à site après le basculement, dans le Pare-feu Windows de l’ordinateur local, autorisez RDP pour les profils de domaine et privé. [Découvrez](../virtual-machines/windows/prepare-for-upload-vhd-image.md#configure-windows-firewall-rules) comment autoriser le trafic RDP.
6. Veillez à ce qu’aucune mise à jour Windows ne soit en attente sur la machine virtuelle locale au moment de déclencher un basculement. Si c’est le cas, les mises à jour risquent de s’installer sur la machine virtuelle Azure après le basculement, et vous ne pourrez pas vous y connecter tant que les mises à jour n’auront pas abouti.

### <a name="prepare-linux-machines"></a>Préparer les ordinateurs Linux

Sur les ordinateurs Linux locaux, effectuez les tâches suivantes :

1. Vérifiez que le service Secure Shell est configuré pour démarrer automatiquement au démarrage du système.
2. Vérifiez que les règles de pare-feu autorisent une connexion SSH.


## <a name="configure-azure-vms-after-failover"></a>Configurer les machines virtuelles Azure après le basculement

Après le basculement, effectuez les tâches suivantes sur les machines virtuelles Azure qui ont été créées.

1. Pour vous connecter à la machine virtuelle via Internet, attribuez-lui une adresse IP publique. Vous ne pouvez pas attribuer à la machine virtuelle Azure la même adresse IP publique que celle que vous avez utilisée pour votre ordinateur local. [En savoir plus](../virtual-network/virtual-network-public-ip-address.md)
2. Vérifiez que les règles de groupe de sécurité réseau (NSG) de la machine virtuelle autorisent les connexions entrantes sur le port RDP ou SSH.
3. Cochez [Diagnostics de démarrage](../virtual-machines/troubleshooting/boot-diagnostics.md#enable-boot-diagnostics-on-existing-virtual-machine) pour afficher la machine virtuelle.


> [!NOTE]
> Le service Azure Bastion offre un accès RDP et SSH privé aux machines virtuelles Azure. [Découvrez plus en détail](../bastion/bastion-overview.md) ce service.

## <a name="set-a-public-ip-address"></a>Définir une adresse IP publique

Plutôt que d’affecter manuellement une adresse IP publique à une machine virtuelle Azure, vous pouvez affecter l’adresse pendant le basculement à l’aide d’un script ou d’un runbook Azure Automation dans un [plan de récupération](site-recovery-create-recovery-plans.md) Site Recovery, ou vous pouvez configurer le routage au niveau du DNS en utilisant Azure Traffic Manager. [Apprenez-en davantage](concepts-public-ip-address-with-site-recovery.md) sur la configuration d’une adresse publique.


## <a name="assign-an-internal-address"></a>Attribuer une adresse interne

Pour définir l’adresse IP interne d’une machine virtuelle Azure après le basculement, vous disposez de deux options :

- **Conserver la même adresse IP** : vous pouvez utiliser la même adresse IP sur la machine virtuelle Azure que celle allouée à l’ordinateur local.
- **Adresse IP différente** : vous pouvez utiliser une adresse IP différente pour la machine virtuelle Azure.


## <a name="retain-ip-addresses"></a>Conserver les adresses IP

Site Recovery vous permet de conserver les mêmes adresses IP quand vous basculez vers Azure. En conservant la même adresse IP, vous évitez des problèmes réseau potentiels après le basculement, mais vous introduisez une certaine complexité.

- Si la machine virtuelle Azure cible utilise la même adresse IP/le même sous-réseau que votre site local, vous ne pouvez pas les connecter entre eux avec une connexion VPN site à site ou ExpressRoute, en raison du chevauchement d’adresse. Les sous-réseaux doivent être uniques.
- Pour assurer la disponibilité des applications sur les machines virtuelles Azure, une connexion est nécessaire entre le site local et Azure après le basculement. Azure ne prend pas en charge les VLAN étirés. Si vous voulez conserver les adresses IP, vous devez donc placer l’espace IP sur Azure en basculant l’ensemble du sous-réseau, en plus de l’ordinateur local.
- Le basculement du sous-réseau est l’assurance qu’aucun sous-réseau spécifique n’est disponible simultanément en local et sur Azure.

La conservation des adresses IP passe par les étapes suivantes :

- Dans les propriétés Calcul et réseau de l’élément répliqué, définissez l’adressage réseau et IP de la machine virtuelle Azure cible en le calquant sur la configuration locale.
- Les sous-réseaux doivent être gérés dans le cadre du processus de récupération d’urgence. Vous avez besoin d’un réseau virtuel Azure qui corresponde au réseau local. Après le basculement, les routes réseau doivent être modifiées en tenant compte du fait que le sous-réseau a été déplacé vers Azure et en prenant en considération les nouveaux emplacements d’adresses IP.  

### <a name="failover-example"></a>Exemple de basculement

Intéressons-nous à un exemple.

- La société fictive Woodgrove Bank héberge ses applications métier localement. Ses applications mobiles sont hébergées dans Azure.
- L’environnement local et Azure sont reliés par un VPN site à site. 
- Woodgrove utilise Site Recovery pour répliquer les ordinateurs locaux sur Azure.
- Sachant que ses applications locales utilisent des adresses IP codées en dur, son souhait est de conserver les mêmes adresses IP dans Azure.
- Les ordinateurs locaux sur lesquels s’exécutent les applications sont présents dans trois sous-réseaux :
    - 192.168.1.0/24
    - 192.168.2.0/24
    - 192.168.3.0/24
- Les applications s’exécutant sur Azure sont situées dans deux sous-réseaux du **réseau Azure**, à savoir :
    - 172.16.1.0/24
    - 172.16.2.0/24

Pour conserver ses adresses, voici comment la société procède.

1. Quand elle active la réplication, elle indique que les ordinateurs doivent être répliqués sur le **réseau Azure**.
2. Elle crée un **réseau de récupération** dans Azure. Ce réseau virtuel est le reflet du sous-réseau 192.168.1.0/24 de son réseau local.
3. Woodgrove configure une [connexion de réseau virtuel à réseau virtuel](../vpn-gateway/vpn-gateway-howto-vnet-vnet-resource-manager-portal.md) entre les deux réseaux. 

    > [!NOTE]
    > Selon les besoins des applications, une connexion de réseau virtuel à réseau virtuel pourrait être configurée avant le basculement, sous forme d’étape manuelle/étape scriptée/runbook Azure Automation dans un [plan de récupération](site-recovery-create-recovery-plans.md) Site Recovery ou à l’issue du basculement.

4. Avant le basculement, dans les propriétés de l’ordinateur dans Site Recovery, l’adresse IP cible est définie comme étant l’adresse de l’ordinateur local, comme décrit dans la procédure suivante.
5. Après le basculement, les machines virtuelles Azure sont créées avec la même adresse IP. Woodgrove se connecte à partir du **Réseau Azure** vers le **Réseau de récupération** VNet à l’aide de VNET Peering (la connectivité de transit est activée).
6. En local, Woodgrove doit apporter des modifications au réseau, notamment en modifiant les routes en fonction du déplacement de 192.168.1.0/24 vers Azure.  

**Infrastructure avant le basculement**

![Avant le basculement de sous-réseau](./media/site-recovery-network-design/network-design7.png)


**Infrastructure après le basculement**

![Après le basculement de sous-réseau](./media/site-recovery-network-design/network-design9.png)


### <a name="set-target-network-settings"></a>Définir les paramètres réseau de la cible

Avant le basculement, spécifiez les paramètres réseau et l’adresse IP de la machine virtuelle Azure cible.

1.  Dans le coffre Recovery Services -> **Éléments répliqués**, sélectionnez l’ordinateur local.
2. Dans la page **Calcul et réseau** de l’ordinateur, cliquez sur **Modifier** pour configurer les paramètres réseau et de carte pour la machine virtuelle Azure cible.
3. Dans **Propriétés du réseau**, sélectionnez le réseau cible sur lequel la machine virtuelle Azure se trouvera une fois celle-ci créée à l’issue du basculement.
4. Dans **Interfaces réseau**, configurez les cartes réseau du réseau cible. Par défaut, Site Recovery présente toutes les cartes réseau détectées sur l’ordinateur local.
    - Dans **Type d’interface réseau cible**, vous pouvez définir chaque carte réseau comme étant **Principale**, **Secondaire** ou **Ne pas créer** si vous n’avez pas besoin de la carte réseau en question sur le réseau cible. Vous devez définir une carte réseau principale pour le basculement. Notez que la modification du réseau cible affecte toutes les cartes réseau de la machine virtuelle Azure.
    - Cliquez sur le nom de la carte réseau pour spécifier le sous-réseau sur lequel la machine virtuelle Azure doit être déployée.
    - Remplacez **Dynamique** par l’adresse IP privée que vous voulez attribuer à la machine virtuelle Azure cible. Si aucune adresse IP n’est spécifiée, Site Recovery attribue l’adresse IP disponible suivante du sous-réseau à la carte réseau au moment du basculement.
    - [Apprenez-en davantage](site-recovery-manage-network-interfaces-on-premises-to-azure.md) sur la gestion des cartes réseau pour un basculement local vers Azure.


## <a name="get-new-ip-addresses"></a>Obtenir de nouvelles adresses IP

Dans ce scénario, la machine virtuelle Azure obtient une nouvelle adresse IP après le basculement. Pour configurer une nouvelle adresse IP pour la machine virtuelle créée après le basculement, effectuez les étapes suivantes :

1. Accédez à **Éléments répliqués**.
2. Sélectionnez la machine virtuelle Azure de votre choix.
3. Sélectionnez **Calcul et réseau**, puis **Modifier**.

     ![Personnaliser les configurations de mise en réseau du basculement](media/azure-to-azure-customize-networking/edit-networking-properties.png)

4. Pour mettre à jour les paramètres réseau de basculement, sélectionnez **Modifier** pour la carte réseau que vous souhaitez configurer. Dans la page suivante qui s’affiche, spécifiez l’adresse IP précréée correspondante dans les emplacements de basculement de test et de basculement.

    ![Modifiez la configuration de la carte réseau](media/azure-to-azure-customize-networking/nic-drilldown.png)

5. Sélectionnez **OK**.

Site Recovery respecte à présent ces paramètres et s’assure que, lors du basculement, la machine virtuelle est connectée à la ressource sélectionnée par le biais de la carte réseau correspondante, si elle est disponible dans la plage d’adresses IP cible. Dans ce scénario, il n’est pas nécessaire de basculer tout le sous-réseau. Une mise à jour DNS sera nécessaire pour mettre à jour les enregistrements de la machine ayant basculé afin qu’ils pointent vers la nouvelle adresse IP de la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes
[Découvrir](site-recovery-active-directory.md) la réplication des services locaux Active Directory et DNS vers Azure.

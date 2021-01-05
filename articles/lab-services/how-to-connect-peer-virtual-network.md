---
title: Se connecter à un réseau pair dans Azure Lab Services | Microsoft Docs
description: Découvrez comment connecter votre réseau lab avec un autre réseau en tant que pair. Par exemple, connectez votre réseau local d’organisation/université avec le réseau virtuel du Lab dans Azure.
ms.topic: article
ms.date: 06/26/2020
ms.openlocfilehash: 99ea72f11c8f389a15171a1fe3b376646494903a
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96434241"
---
# <a name="connect-your-labs-network-with-a-peer-virtual-network-in-azure-lab-services"></a>Connecter un réseau lab avec un réseau virtuel pair dans Azure Lab Services

Cet article fournit des informations concernant le Peering de votre réseau lab avec un autre réseau.

## <a name="overview"></a>Vue d’ensemble

Le Peering d’un réseau virtuel vous permet de connecter des réseaux virtuels Azure en toute transparence. Une fois homologués, les réseaux virtuels apparaissent comme un seul réseau à des fins de connectivité. Le trafic entre les machines virtuelles des réseaux virtuels appairés est acheminé via l’infrastructure principale de Microsoft de façon assez semblable au trafic entre des machines virtuelles d’un même réseau virtuel via des IP privées seulement. Pour en savoir plus, consultez [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

Il vous faudra peut-être connecter votre réseau lab avec un réseau virtuel pair dans certains scénarios, notamment dans les scénarios suivants :

- Les machines virtuelles du lab disposent d’un logiciel qui se connecte aux serveurs de licences locaux pour obtenir une licence.
- Les machines virtuelles du lab doivent accéder à des jeux de données (ou à d’autres fichiers) sur les partages réseau de l’université.

Certains des réseaux locaux sont connectés au réseau virtuel Azure via [ExpressRoute](../expressroute/expressroute-introduction.md) ou via la [Passerelle de réseau virtuel](../vpn-gateway/vpn-gateway-about-vpngateways.md). Ces services doivent être configurés en dehors de Azure Lab Services. Pour en savoir plus sur la connexion d’un réseau local à Azure à l’aide d’ExpressRoute, consultez [Vue d’ensemble d’ExpressRoute](../expressroute/expressroute-introduction.md). Pour la connectivité locale à l’aide d’une passerelle de réseau virtuel, la passerelle, le réseau virtuel spécifié et le compte lab doivent tous être situés dans la même région.

> [!NOTE]
> Lors de la création d’un Réseau virtuel Azure qui sera appairé avec un compte de labo, il est important de comprendre l’incidence de la région du réseau virtuel sur l’endroit où sont créés les labos.  Pour plus d’informations, consultez la section du guide de l’administrateur relative aux [régions et emplacements](./administrator-guide.md#regionslocations).

## <a name="configure-at-the-time-of-lab-account-creation"></a>Effectuer la configuration au moment de la création du compte lab

Lors de la [création du nouveau compte de laboratoire](tutorial-setup-lab-account.md), vous pouvez choisir un réseau virtuel existant qui s’affiche dans la liste déroulante **Appairer un réseau virtuel** dans l’onglet **Avancé**.  La liste affiche uniquement les réseaux virtuels de la même région que le compte de laboratoire. Le réseau virtuel sélectionné est connecté (par Peering) aux laboratoires créés sous le compte de laboratoire.  Toutes les machines virtuelles des laboratoires qui sont créées après cette modification auront accès aux ressources sur le réseau virtuel appairé.

![Sélectionner le réseau virtuel à appairer](./media/how-to-connect-peer-virtual-network/select-vnet-to-peer.png)

### <a name="address-range"></a>Plage d’adresses

Il est également possible de spécifier une **plage d’adresses** de machines virtuelles pour les laboratoires.  La propriété **Plage d’adresses** s’applique uniquement si un **réseau virtuel pair** est activé pour le laboratoire. Si la plage d’adresses est spécifiée, toutes les machines virtuelles des labs sous le compte lab seront créées dans cette plage d’adresses. La plage d’adresses doit être en notation CIDR (par exemple 10.20.0.0/20) et ne doit pas chevaucher des plages d’adresses existantes.  Quand vous spécifiez une plage d’adresses, il est important de réfléchir au nombre de *laboratoires* qui seront créés et de spécifier une plage d’adresses suffisante. Azure Lab Services suppose un maximum de 512 machines virtuelles par laboratoire.  Par exemple, une plage d’adresses IP avec « /23 » ne peut créer qu’un seul laboratoire.  Une plage avec un « /21 » permettra la création de quatre laboratoires.

Si la **plage d’adresses** n’est pas spécifiée, Azure Lab Services utilise la plage d’adresses par défaut qui lui est attribuée par Azure lors de la création du réseau virtuel à appairer avec votre réseau virtuel.  La plage est souvent semblable au format 10.x.0.0/16.  Cela peut entraîner un chevauchement des plages d’adresses IP. Veillez donc à spécifier la plage d’adresses dans les paramètres du laboratoire ou à vérifier la plage d’adresses de votre réseau virtuel en cours d’appairage.

> [!NOTE]
> La création de laboratoire peut échouer si le compte lab est appairé à un réseau virtuel, mais si sa plage d’adresses IP est trop étroite. Vous pouvez manquer d’espace dans la plage d’adresses si le compte lab contient trop de laboratoires (chaque laboratoire utilise 512 adresses). 
> 
> En cas d’échec de la création du laboratoire, contactez le propriétaire ou l’administrateur de votre compte lab pour demander l’élargissement de la plage d’adresses. L’administrateur peut élargir la plage d’adresses en suivant les étapes de la section [Spécifier une plage d’adresses pour des machines virtuelles dans un compte lab](#specify-an-address-range-for-vms-in-the-lab-account). 

## <a name="configure-after-the-lab-account-is-created"></a>Effectuer la configuration une fois le compte de laboratoire créé

La même propriété peut être activée à partir de l’onglet **Configuration de Labs** de la page **Compte Lab** si vous n’avez pas configuré un réseau pair au moment de la création d’un compte lab. Les modifications apportées à ce paramètre s’appliquent uniquement aux labs créés après la modification. Comme vous pouvez le voir sur l’image, vous pouvez activer ou désactiver le champ **Appairer un réseau virtuel** pour les labs dans le compte lab.

![Activer ou désactiver le Peering de réseaux virtuels une fois le lab créé](./media/how-to-connect-peer-virtual-network/select-vnet-to-peer-existing-lab.png)

Lorsque vous sélectionnez un réseau virtuel pour le champ **Appairer un réseau virtuel**, l’option **Autoriser le créateur du lab à choisir l’emplacement du lab** est désactivée. En effet, les laboratoires du compte de laboratoire doivent être dans la même région que le compte de laboratoire lui-même pour pouvoir se connecter aux ressources dans le réseau virtuel appairé.

> [!IMPORTANT]
> Le paramètre du réseau virtuel appairé s’applique uniquement aux laboratoires créés après la modification, et non aux laboratoires existants.


## <a name="specify-an-address-range-for-vms-in-the-lab-account"></a>Spécifier une plage d’adresses pour les machines virtuelles dans le compte lab
La procédure suivante comporte des étapes pour spécifier une plage d’adresses pour les machines virtuelles dans le laboratoire. Si vous mettez à jour la plage que vous avez spécifiée précédemment, la plage d’adresses modifiée s’applique uniquement aux machines virtuelles qui sont créées après que la modification a été effectuée. 

Voici quelques restrictions à garder à l’esprit lors de la spécification de la plage d’adresses. 

- Le préfixe doit être inférieur ou égal à 23. 
- Si un réseau virtuel est associé au compte de laboratoire, la plage d’adresses fournie ne peut pas chevaucher une plage d’adresses de réseau virtuel associé.

1. Dans la page **Compte lab**, sélectionnez **Paramètres lab** dans le menu de gauche.
2. Pour le champ **Plage d’adresses**, spécifiez la plage d’adresses pour les machines virtuelles qui seront créées dans le laboratoire. La plage d’adresses doit être mentionnée dans la notation CIDR (Classless Inter-Domain Routing), par exemple : 10.20.0.0/23). Les machines virtuelles du laboratoire seront créées dans cette plage d’adresses.
3. Sélectionnez **Enregistrer** dans la barre d’outils. 

    ![Configurer la plage d’adresses](./media/how-to-manage-lab-accounts/labs-configuration-page-address-range.png)

## <a name="next-steps"></a>Étapes suivantes

Voir les articles suivants :

- [Autoriser le créateur du lab à choisir l’emplacement du lab](allow-lab-creator-pick-lab-location.md)
- [Attacher une galerie d’images partagées à un lab](how-to-attach-detach-shared-image-gallery.md)
- [Ajouter un utilisateur comme propriétaire de labo](how-to-add-user-lab-owner.md)
- [Afficher les paramètres de pare-feu pour un labo](how-to-configure-firewall-settings.md)
- [Configurer les autres paramètres d’un lab](how-to-configure-lab-accounts.md)
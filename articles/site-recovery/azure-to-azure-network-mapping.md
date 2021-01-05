---
title: Mappage de réseaux virtuels entre deux régions dans Azure Site Recovery
description: Découvrez comment mapper des réseaux virtuels entre deux régions Azure pour la récupération d’urgence sur des machines virtuelles Azure avec Azure Site Recovery.
author: Harsha-CS
manager: rochakm
ms.service: site-recovery
ms.topic: conceptual
ms.date: 10/15/2019
ms.author: harshacs
ms.openlocfilehash: ff1f80641dc3db1f6b69fc0223c60022f8cf8435
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95811628"
---
# <a name="set-up-network-mapping-and-ip-addressing-for-vnets"></a>Configurer le mappage réseau et l’adressage IP pour les réseaux virtuels

Cet article explique comment mapper deux instances de réseaux virtuels Azure situés dans différentes régions Azure et comment configurer l’adressage IP entre les réseaux. Le mappage réseau fournit un comportement par défaut pour la sélection du réseau cible en fonction du réseau source, au moment de l’activation de la réplication.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de mapper des réseaux, vous devez avoir des [réseaux virtuels Azure](../virtual-network/virtual-networks-overview.md) dans les régions Azure source et cible. 

## <a name="set-up-network-mapping-manually-optional"></a>Configurer le mappage réseau manuellement (facultatif)

Mappez les réseaux comme suit :

1. Dans **Infrastructure Site Recovery**, cliquez sur **+Mappage réseau**.

    ![ Créer un mappage réseau](./media/site-recovery-network-mapping-azure-to-azure/network-mapping1.png)

3. Dans **Ajouter un mappage réseau**, sélectionnez les emplacements source et cible. Dans notre exemple, la machine virtuelle source est en cours d’exécution dans la région Asie Est et elle est répliquée dans la région Asie Sud-Est.

    ![Sélectionner la source et la cible](./media/site-recovery-network-mapping-azure-to-azure/network-mapping2.png)
3. Maintenant, créez un mappage réseau dans la direction opposée. Dans notre exemple, la source sera désormais la région Asie Sud-Est, et la cible dans la région Asie Est.

    ![Volet Ajouter mappage réseau - Sélectionner les emplacements source et cible pour le réseau cible](./media/site-recovery-network-mapping-azure-to-azure/network-mapping3.png)


## <a name="map-networks-when-you-enable-replication"></a>Mapper des réseaux lorsque vous activez la réplication

Si vous n’avez pas préparé le mappage réseau avant de configurer la récupération d’urgence pour les machines virtuelles Azure, vous pouvez spécifier un réseaux cible lorsque vous [configurez et activez la réplication](azure-to-azure-how-to-enable-replication.md). Quand vous procédez ainsi, voici ce qui se passe :

- En fonction de la cible sélectionnée, Site Recovery crée automatiquement des mappages réseau de la région source à la région cible et de la région cible à la région source.
- Par défaut, Site Recovery crée dans la région cible un réseau qui est identique au réseau source. Site Recovery ajoute **-asr** comme suffixe au nom du réseau source. Vous pouvez personnaliser le réseau cible.
- Si le mappage réseau s’est déjà produit pour un réseau source, le réseau cible mappé correspond toujours à la valeur par défaut lors de l’activation de la réplication pour davantage de machines virtuelles. Vous pouvez choisir de modifier le réseau virtuel cible en sélectionnant l’un des autres options disponibles dans la liste déroulante. 
- Pour modifier le réseau virtuel cible par défaut des nouvelles réplications, vous devez modifier le mappage réseau existant.
- Si vous souhaitez modifier un mappage réseau de la région A vers la région B, commencez par supprimer le mappage réseau de la région B vers la région A. Après la suppression du mappage inverse, modifiez le mappage réseau de la région A vers la région B, puis créez le mappage inverse adéquat.

>[!NOTE]
>* La modification du mappage réseau n’affecte que les valeurs par défaut des nouvelles réplications de machines virtuelles. Cela n’affecte pas les sélections de réseaux virtuels cibles pour les réplications existantes. 
>* Si vous souhaitez modifier le réseau cible d’une réplication existante, accédez aux paramètres Calcul et réseau de l’élément répliqué.

## <a name="specify-a-subnet"></a>Spécifier un sous-réseau

Le sous-réseau de la machine virtuelle cible est sélectionné en fonction du nom du sous-réseau de la machine virtuelle source.

- S’il existe sur le réseau cible un sous-réseau portant le même nom que le sous-réseau de la machine virtuelle source, il sera défini pour la machine virtuelle cible.
- S’il n’y a aucun sous-réseau du même nom sur le réseau cible, le premier sous-réseau dans l’ordre alphabétique est défini comme sous-réseau cible.
- Vous pouvez modifier le sous-réseau cible dans les paramètres **Calcul et réseau** de la machine virtuelle.

    ![Fenêtre Propriétés de calcul et réseau](./media/site-recovery-network-mapping-azure-to-azure/modify-subnet.png)


## <a name="set-up-ip-addressing-for-target-vms"></a>Configurer l’adressage IP pour les machines virtuelles cibles

L’adresse IP pour chaque carte réseau sur une machine virtuelle cible est configurée comme suit :

- **DHCP** : si la carte réseau de la machine virtuelle source utilise le protocole DHCP, la carte réseau de la machine virtuelle cible est également configurée pour utiliser DHCP.
- **Adresse IP statique** : si la carte réseau de la machine virtuelle source utilise un adressage IP statique, la carte réseau de la machine virtuelle cible utilise également une adresse IP statique.


## <a name="ip-address-assignment-during-failover"></a>Affectation d’adresses IP lors du basculement

**Sous-réseaux source et cible** | **Détails**
--- | ---
Même espace d’adressage | L’adresse IP de la carte réseau de la machine virtuelle source est définie comme l’adresse IP de la carte réseau de la machine virtuelle cible.<br/><br/> Si l’adresse n’est pas disponible, l’adresse IP suivante disponible est définie comme cible.
Espace d’adressage différent | L’adresse IP suivante disponible dans le sous-réseau cible est définie comme adresse de la carte réseau de la machine virtuelle cible.



## <a name="ip-address-assignment-during-test-failover"></a>Affectation d’adresses IP lors du test de basculement

**Réseau cible** | **Détails**
--- | ---
Le réseau cible est le réseau virtuel pour le basculement | - L’adresse IP cible est statique avec la même adresse IP. <br/><br/>  - Si la même adresse IP est déjà attribuée, alors l’adresse IP est la prochaine disponible à la fin de la plage du sous-réseau. Par exemple : si l’adresse IP source est 10.0.0.19 et si le réseau de basculement utilise la plage 10.0.0.0/24, alors l’adresse IP suivante attribuée à la machine virtuelle cible est 10.0.0.254.
Le réseau cible n’est pas le réseau virtuel pour le basculement | - L’adresse IP cible est statique avec la même adresse IP.<br/><br/>  - Si la même adresse IP est déjà attribuée, alors l’adresse IP est la prochaine disponible à la fin de la plage du sous-réseau.<br/><br/> Exemple : si l’adresse IP statique source est 10.0.0.19 et que le basculement se trouve sur un réseau qui n’est pas le réseau de basculement, avec la plage 10.0.0.0/24, alors l’adresse IP statique cible est 10.0.0.19 si elle est disponible, et dans le cas contraire, 10.0.0.254.

- Le réseau virtuel de basculement est le réseau cible que vous sélectionnez lorsque vous configurez la récupération d’urgence.
- Nous vous recommandons de toujours utiliser un réseau de non production pour le test de basculement.
- Vous pouvez modifier l’adresse IP cible dans les paramètres **Calcul et réseau** de la machine virtuelle.


## <a name="next-steps"></a>Étapes suivantes

- Consultez les [conseils sur la mise en réseau](./azure-to-azure-about-networking.md) pour la récupération d’urgence de machines virtuelles Azure.
- [En savoir plus](site-recovery-retain-ip-azure-vm-failover.md) sur la conservation des adresses IP après le basculement.

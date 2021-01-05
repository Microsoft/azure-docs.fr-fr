---
title: Fichier include
description: Fichier include
services: bastion
author: cherylmc
ms.service: bastion
ms.topic: include
ms.date: 11/05/2020
ms.author: cherylmc
ms.custom: include file
ms.openlocfilehash: 57b0bb9ab8ceb34021a38db0d0abf2c9c919e808
ms.sourcegitcommit: 0b9fe9e23dfebf60faa9b451498951b970758103
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/07/2020
ms.locfileid: "94356646"
---
### <a name="can-i-still-deploy-multiple-bastion-hosts-across-peered-virtual-networks"></a>Puis-je toujours déployer plusieurs hôtes bastions sur des réseaux virtuels appairés ?

Oui. Par défaut, un utilisateur voit l’hôte bastion qui est déployé dans le même réseau virtuel dans lequel réside la machine virtuelle. Toutefois, dans le menu **Connexion**, un utilisateur peut voir plusieurs hôtes bastions détectés sur des réseaux appairés. Il peut sélectionner l’hôte bastion qu’il préfère utiliser pour se connecter à la machine virtuelle déployée dans le réseau virtuel.

### <a name="if-my-peered-vnets-are-deployed-in-different-subscriptions-will-connectivity-via-bastion-work"></a>Si mes réseaux virtuels appairés sont déployés dans des abonnements différents, la connectivité via Bastion fonctionnera-t-elle ?

Oui, la connectivité via Bastion continue à fonctionner pour les réseaux virtuels appairés sur différents abonnements pour un seul locataire. Elle n’est pas prise en charge pour les abonnements qui se trouvent dans deux locataires différents. Pour afficher Bastion dans le menu déroulant **Connexion**, l’utilisateur doit sélectionner les abonnements auxquels il a accès dans **Abonnement > abonnement global**.

:::image type="content" source="./media/bastion-faq-peering-include/global-subscriptions.png" alt-text="Filtre des abonnements globaux" lightbox="./media/bastion-faq-peering-include/global-subscriptions.png":::

### <a name="i-have-access-to-the-peered-vnet-but-i-cant-see-the-vm-deployed-there"></a>J’ai accès au réseau virtuel appairé, mais je ne peux pas y voir la machine virtuelle déployée.

Assurez-vous que l’utilisateur dispose d’un accès **en lecture** à la machine virtuelle et au réseau virtuel appairé. En outre, vérifiez sous IAM que l’utilisateur dispose d’un accès **en lecture** aux ressources suivantes :

* Rôle de lecteur sur la machine virtuelle.
* Rôle de lecteur sur la carte réseau avec adresse IP privée de la machine virtuelle.
* Rôle de lecteur sur la ressource Azure Bastion.
* Rôle de lecteur sur le réseau virtuel (inutile s’il n’y a pas de réseau virtuel appairé).

|Autorisations|Description|Type d'autorisation|
|---|---| ---|
|Microsoft.Network/bastionHosts/read |Obtient un hôte Bastion|Action|
|Microsoft.Network/virtualNetworks/BastionHosts/action |Obtient des références d’hôte bastion dans un réseau virtuel.|Action|
|Microsoft.Network/virtualNetworks/bastionHosts/default/action|Obtient des références d’hôte bastion dans un réseau virtuel.|Action|
|Microsoft.Network/networkInterfaces/read|Obtient une définition d’interface réseau.|Action|
|Microsoft.Network/networkInterfaces/ipconfigurations/read|Obtient une définition de la configuration de l’adresse IP de l’interface réseau.|Action|
|Microsoft.Network/virtualNetworks/read|Obtenir la définition de réseau virtuel.|Action|
|Microsoft.Network/virtualNetworks/subnets/virtualMachines/read|Obtient les références à toutes les machines virtuelles dans un sous-réseau de réseau virtuel.|Action|
|Microsoft.Network/virtualNetworks/virtualMachines/read|Obtient les références à toutes les machines virtuelles dans un réseau virtuel.|Action|
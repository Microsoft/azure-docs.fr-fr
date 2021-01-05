---
title: Créer une machine virtuelle avec une adresse IP publique statique - Portail Azure
description: Découvrez comment créer une machine virtuelle avec une adresse IP publique statique à l’aide du portail Azure.
services: virtual-network
documentationcenter: na
author: asudbring
manager: KumudD
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: ''
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/12/2020
ms.author: allensu
ms.openlocfilehash: 1ae0b869b24c4e05c88b936eceb1b9b1db3a9405
ms.sourcegitcommit: 4bee52a3601b226cfc4e6eac71c1cb3b4b0eafe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94506309"
---
# <a name="create-a-virtual-machine-with-a-static-public-ip-address-using-the-azure-portal"></a>Créer une machine virtuelle avec une adresse IP publique statique à partir du portail Azure

Une adresse IP publique vous permet de communiquer avec une machine virtuelle à partir d’Internet. 

Attribuez une adresse IP publique statique, plutôt qu’une adresse dynamique, pour être certain que l’adresse ne changera jamais.   

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com).

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. En haut à gauche de l’écran du portail Azure, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**, ou recherchez **Machine virtuelle** dans la zone de recherche.
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **Créer nouveau**. </br> Dans **Nom**, entrez **myResourceGroup**. </br> Sélectionnez **OK**. |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **USA Est**. |
    | Options de disponibilité | Sélectionnez **Aucune redondance d’infrastructure requise**. |
    | Image | Sélectionnez **Windows Server 2019 Datacenter - Gen1** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | Acceptez le nom réseau par défaut. |
    | Subnet | Acceptez la configuration de sous-réseau par défaut. |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. </br> Dans **Créer une adresse IP publique**, dans le champ Nom, entrez **myPublicIP**. </br> Pour **Référence SKU**, sélectionnez **Standard**. </br> Pour **Affectation**, sélectionnez **Statique**. </br> Sélectionnez **OK**.  |
    | Groupe de sécurité réseau de la carte réseau | Sélectionnez **De base**|
    | Aucun port d’entrée public | Sélectionnez **Autoriser les ports sélectionnés**. |
    | Sélectionner des ports d’entrée | Sélectionnez **RDP (3389)** |

    > [!WARNING]
    > Le port 3389 est sélectionné pour autoriser l’accès à distance à la machine virtuelle Windows Server depuis Internet. Il est déconseillé d’ouvrir le port 3389 à Internet pour gérer les charges de travail de production. </br> Pour un accès sécurisé aux machines virtuelles Azure, consultez **[Qu’est-ce qu’Azure Bastion ?](/azure/bastion/bastion-overview)**
   
5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, supprimez-le, ainsi que toutes les ressources qu’il contient :

1. Entrez **myResourceGroup** dans le champ **Recherche** en haut du portail. Quand **myResourceGroup** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Entrez **myResourceGroup** dans **TAPER NOM DU GROUPE DE RESSOURCES :** puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Consultez [Ajouter, modifier ou supprimer des adresses IP](virtual-network-network-interface-addresses.md) :

* Pour remplacer une adresse IP publique dynamique par une adresse statique.
* Pour utiliser des adresses IP privées.

Les adresses IP publiques ont un [coût nominal](https://azure.microsoft.com/pricing/details/ip-addresses). Le nombre d’adresses IP publiques que vous pouvez utiliser dans un abonnement est [limité](../azure-resource-manager/management/azure-subscription-service-limits.md?toc=%2fazure%2fvirtual-network%2ftoc.json#azure-resource-manager-virtual-networking-limits).

La référence SKU de l’adresse IP publique de la machine virtuelle doit correspondre à la référence SKU de l’adresse IP publique d’Azure Load Balancer lorsqu’elle est ajoutée à un pool de back-ends. Pour plus d’informations, consultez [Azure Load Balancer](../load-balancer/skus.md).

Vous pouvez télécharger la liste des plages (préfixes) pour les clouds Azure [Public](https://www.microsoft.com/download/details.aspx?id=56519), [Gouvernement américain](https://www.microsoft.com/download/details.aspx?id=57063), [Chine](https://www.microsoft.com/download/details.aspx?id=57062), et [Allemagne](https://www.microsoft.com/download/details.aspx?id=57064).

- Découvrez plus en détail les [adresses IP publiques statiques](virtual-network-ip-addresses-overview-arm.md#allocation-method).
- En savoir plus sur les [adresses IP publiques](virtual-network-ip-addresses-overview-arm.md#public-ip-addresses) dans Azure.
- En savoir plus sur tous les [paramètres d’adresse IP publique](virtual-network-public-ip-address.md#create-a-public-ip-address).
- En savoir plus sur les [adresses IP privées](virtual-network-ip-addresses-overview-arm.md#private-ip-addresses) et l’affectation d’une [adresse IP privée statique](virtual-network-network-interface-addresses.md#add-ip-addresses) à une machine virtuelle Azure.

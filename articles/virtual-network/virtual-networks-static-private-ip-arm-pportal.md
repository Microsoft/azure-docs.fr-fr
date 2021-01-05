---
title: Configurer des adresses IP privées pour des machines virtuelles - Portail Azure
description: Découvrez comment configurer des adresses IP privées pour des machines virtuelles à l’aide du portail Azure.
services: virtual-network
documentationcenter: na
author: KumudD
manager: twooley
tags: azure-resource-manager
ms.assetid: 11245645-357d-4358-9a14-dd78e367b494
ms.service: virtual-network
ms.subservice: ip-services
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/07/2020
ms.author: kumud
ms.openlocfilehash: c8fdba59a8d31c064745c7a1904204359b386a7f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96015920"
---
# <a name="configure-a-private-ip-address-for-a-vm-using-the-azure-portal"></a>Configurer une adresse IP privée pour une machine virtuelle à l’aide du portail Azure

[!INCLUDE [virtual-networks-static-private-ip-intro-include](../../includes/virtual-networks-static-private-ip-intro-include.md)]

[!INCLUDE [virtual-networks-static-ip-scenario-include](../../includes/virtual-networks-static-ip-scenario-include.md)]

Les étapes de l’exemple suivant partent du principe qu’un environnement simple a déjà été créé. Si vous souhaitez exécuter les étapes telles qu’elles sont présentées dans ce document, commencez par [créer un réseau virtuel](quick-create-portal.md#create-a-virtual-network). Toutefois, à l’étape 3, utilisez ces valeurs à la place :

| Paramètre | Valeur |
| ------- | ----- |
| Nom | *TestVNet* |
| Espace d’adressage | *192.168.0.0/16* |
| Resource group | **TestRG** (si nécessaire, sélectionnez **Créer** pour le créer) |
| Sous-réseau - Nom | *FrontEnd* |
| Plage d’adresses du sous-réseau | *192.168.1.0/24* |

## <a name="create-a-vm-for-testing-static-private-ip-addresses"></a>Créer une machine virtuelle pour tester des adresses IP privées statiques
Quand vous créez une machine virtuelle en mode de déploiement Resource Manager, vous ne pouvez pas définir une adresse IP privée statique à l’aide du portail Azure. Au lieu de cela, vous devez d’abord créer la machine virtuelle. Ensuite, vous pouvez définir son adresse IP privée comme statique.

Pour créer une machine virtuelle nommée *DNS01* dans le sous-réseau *FrontEnd* d’un réseau virtuel nommé *TestVNet*, effectuez ces étapes :

1. Dans le menu du [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource**.

    ![Créer une ressource, portail Azure](./media/virtual-networks-static-ip-arm-pportal/create-a-resource.png)
2. Sélectionnez **Calcul** > **Machine virtuelle**.

    ![Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/compute-virtual-machine.png)
3. Dans **De base**, spécifiez des valeurs pour les éléments, comme décrit dans le tableau suivant. Ensuite, sélectionnez **Suivant&nbsp;:&nbsp;Disques**, puis **Suivant&nbsp;:&nbsp;Mise en réseau**.

    | Élément | Valeur |
    | --- | --- |
    | **Abonnement** | Votre abonnement actif |
    | **Groupe de ressources** | **TestRG** (sélectionnez dans la liste déroulante) |
    | **Nom de la machine virtuelle** | *DNS01* |
    | **Région** | **(États-Unis) USA Est** |
    | **Image** | **Windows Server 2019 Datacenter** |
    | **Taille** | **Taille de la machine virtuelle** : **B1ls**, **Offre** : **Standard** |
    | **Nom d’utilisateur** | Nom d’utilisateur de votre compte d’administrateur |
    | **Mot de passe** | Mot de passe du nom d’utilisateur de votre compte d’administrateur |
    | **Confirmer le mot de passe** | Retapez le mot de passe |

    ![Onglet De base, Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/create-a-virtual-machine-basics.png)
4. Dans **Réseau**, spécifiez des valeurs pour les éléments comme décrit dans le tableau suivant, puis sélectionnez **Suivant**.

    | Élément | Valeur |
    | --- | --- |
    | **Réseau virtuel** | **TestVNet** |
    | **Sous-réseau** | **FrontEnd** |

    ![Onglet Réseau, Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/create-a-virtual-machine-networking.png)
5. Dans **Gestion**, sous **Compte de stockage de diagnostics**, choisissez **vnetstorage**. Si ce compte de stockage n’apparaît pas dans la liste, sélectionnez **Créer**, spécifiez *vnetstorage* comme **Nom**, puis sélectionnez **OK**. Pour finir, sélectionnez **Vérifier&nbsp;+&nbsp;créer**.

    ![Onglet Gestion, Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/create-a-virtual-machine-management.png)
6. Dans **Vérifier + créer**, passez en revue les informations de vue d’ensemble, puis sélectionnez **Créer**.

    ![Onglet Vérifier + créer, Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/create-a-virtual-machine-review-create.png)

Le message suivant s’affiche une fois la machine virtuelle créée.

![Message de fin du déploiement, Créer une machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/deployment-is-complete.png)

## <a name="retrieve-private-ip-address-information-for-a-vm"></a>Récupérer les informations sur l’adresse IP privée d’une machine virtuelle
Pour afficher les informations sur l’adresse IP privée de votre nouvelle machine virtuelle

1. Accédez au [portail Azure](https://portal.azure.com) pour rechercher votre machine virtuelle. Recherchez et sélectionnez **Machines virtuelles**.

    ![Machines virtuelles, zone Rechercher, portail Azure](./media/virtual-networks-static-ip-arm-pportal/search-box-virtual-machines.png)

2. Sélectionnez le nom de votre nouvelle machine virtuelle (**DNS01**).

    ![Liste des machines virtuelles, portail Azure](./media/virtual-networks-static-ip-arm-pportal/virtual-machine-list.png)

3. Choisissez **Réseau** et sélectionnez la seule interface réseau listée.

    ![Interface réseau, réseau, machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/networking-network-interface.png)

4. Choisissez **Configurations IP**, puis sélectionnez la configuration IP listée dans le tableau.

    ![Configuration IP, interface réseau, réseau, machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/network-interface-ip-configurations.png)

5. Dans **Paramètres d’adresse IP privée**, sous le réseau virtuel/sous-réseau **TestVNet/FrontEnd**, notez la valeur **Affectation** (**Dynamique** ou **Statique**) et l’**adresse IP**.

    ![Affectation dynamique ou statique, anciens paramètres d’adresse IP privée, configuration IP, interface réseau, réseau, machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/private-ip-address-settings-old.png)

## <a name="add-a-static-private-ip-address-to-an-existing-vm"></a>Ajouter une adresse IP privée statique à une machine virtuelle existante
Pour ajouter une adresse IP privée statique à votre nouvelle machine virtuelle

1. Dans la page Configuration IP, définissez l’affectation de votre adresse IP privée sur **Statique**.
2. Définissez *192.168.1.101* comme **Adresse IP** privée, puis sélectionnez **Enregistrer**.
   
    ![Affectation dynamique ou statique, nouveaux paramètres d’adresse IP privée, configuration IP, interface réseau, réseau, machine virtuelle, portail Azure](./media/virtual-networks-static-ip-arm-pportal/private-ip-address-settings-new.png)

> [!NOTE]
> Si, après avoir sélectionné **Enregistrer**, vous remarquez que l’affectation est toujours définie sur **Dynamique**, cela signifie que l’adresse IP que vous avez tapée est déjà utilisée. Essayez une autre adresse IP.

## <a name="remove-a-static-private-ip-address-from-a-vm"></a>Supprimer une adresse IP privée statique d’une machine virtuelle
Pour supprimer l’adresse IP privée statique de votre machine virtuelle

Dans la page Configuration IP, définissez l’affectation de votre adresse IP privée sur **Dynamique**, puis sélectionnez **Enregistrer**.

## <a name="set-ip-addresses-within-the-operating-system"></a>Définir des adresses IP au sein du système d’exploitation

À partir du système d’exploitation d’une machine virtuelle, vous ne devez pas affecter de manière statique l’adresse IP *privée* qui est affectée à la machine virtuelle Azure. N’effectuez l’affectation statique d’une adresse IP privée qu’en cas de nécessité, par exemple quand vous [affectez de nombreuses adresses IP à des machines virtuelles](virtual-network-multiple-ip-addresses-portal.md). Si vous définissez manuellement l’adresse IP privée dans le système d’exploitation, assurez-vous qu’elle correspond à l’adresse IP privée affectée à l’[interface réseau](virtual-network-network-interface-addresses.md#change-ip-address-settings) Azure. Sinon, vous risquez de perdre la connectivité à la machine virtuelle. En savoir plus sur les paramètres [d’adresse IP privée](virtual-network-network-interface-addresses.md#private).

De plus, vous ne devez jamais affecter manuellement l’adresse IP *publique* affectée à une machine virtuelle Azure au sein du système d’exploitation de la machine virtuelle.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur la gestion des [paramètres d’adresse IP](virtual-network-network-interface-addresses.md).

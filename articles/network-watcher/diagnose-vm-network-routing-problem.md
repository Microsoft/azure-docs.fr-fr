---
title: 'Tutoriel : Diagnostiquer un problème de routage réseau de machine virtuelle – portail Azure'
titleSuffix: Azure Network Watcher
description: Dans ce didacticiel, vous allez découvrir comment diagnostiquer un problème de routage réseau d’une machine virtuelle à l’aide de la fonctionnalité de tronçon suivant d’Azure Network Watcher.
services: network-watcher
documentationcenter: network-watcher
author: damendo
editor: ''
tags: azure-resource-manager
Customer intent: I need to diagnose virtual machine (VM) network routing problem that prevents communication to different destinations.
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: network-watcher
ms.workload: infrastructure
ms.date: 04/20/2018
ms.author: damendo
ms.custom: mvc
ms.openlocfilehash: 52d398fa9c258528ef8f87842ba94f139bbf737b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "76845205"
---
# <a name="tutorial-diagnose-a-virtual-machine-network-routing-problem-using-the-azure-portal"></a>Tutoriel : Diagnostiquer un problème de routage réseau d’une machine virtuelle à l’aide du portail Azure

Lorsque vous déployez une machine virtuelle (VM), Azure crée à cet effet plusieurs itinéraires par défaut. Vous pouvez créer des itinéraires personnalisés pour remplacer les itinéraires par défaut d’Azure. Il arrive qu’une machine virtuelle ne puisse pas communiquer avec d’autres ressources à cause d’un itinéraire personnalisé. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une machine virtuelle
> * Tester la communication avec une URL à l’aide de la fonctionnalité de tronçon suivant de Network Watcher
> * Tester la communication avec une adresse IP
> * Diagnostiquer un problème de routage et découvrir comment le résoudre

Si vous préférez, vous pouvez diagnostiquer un problème de routage réseau d’une machine virtuelle à l’aide [d’Azure CLI](diagnose-vm-network-routing-problem-cli.md) ou [d’Azure PowerShell](diagnose-vm-network-routing-problem-powershell.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="log-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-vm"></a>Créer une machine virtuelle

1. Sélectionnez **+ Créer une ressource** en haut à gauche du portail Azure.
2. Sélectionnez **Compute**, puis **Windows Server 2016 Datacenter** ou **Ubuntu Server 17.10 VM**.
3. Entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis cliquez sur **OK** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|myVm|
    |Nom d'utilisateur| Entrez un nom d’utilisateur de votre choix.|
    |Mot de passe| Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    |Abonnement| Sélectionnez votre abonnement.|
    |Resource group| Sélectionnez **Créer** et entrez **myResourceGroup**.|
    |Emplacement| Sélectionnez **USA Est**.|

4. Choisissez une taille de machine virtuelle, puis cliquez sur **Sélectionner**.
5. Sous **Paramètres**, acceptez toutes les valeurs par défaut, puis cliquez sur **OK**.
6. Sous **Créer** dans le **résumé**, sélectionnez **Créer** pour démarrer le déploiement de la machine virtuelle. Le déploiement de la machine virtuelle ne nécessite que quelques minutes. Attendez la fin du déploiement de la machine virtuelle avant d’effectuer les étapes restantes.

## <a name="test-network-communication"></a>Tester la communication réseau

Pour tester une communication réseau avec Network Watcher, vous devez d’abord activer un observateur réseau pour au moins une région Azure, puis utiliser la fonctionnalité de tronçon suivant de Network Watcher.

### <a name="enable-network-watcher"></a>Activer Network Watcher

Si vous disposez déjà d’un observateur réseau activé dans au moins une région, passez à l’étape [Use next hop](#use-next-hop) (Utiliser le tronçon suivant).

1. Dans le portail Azure, sélectionnez **Tous les services**. Dans la zone **Filtre**, entrez *Network Watcher*. Quand la mention **Network Watcher** apparaît dans les résultats, sélectionnez-la.
2. Sélectionnez la zone **Régions** pour la développer, puis sélectionnez **...** à droite de la région **USA Est**, comme illustré dans l’image suivante :

    ![Activer Network Watcher](./media/diagnose-vm-network-traffic-filtering-problem/enable-network-watcher.png)

3. Sélectionnez **Activer Network Watcher**.

### <a name="use-next-hop"></a>Utiliser le tronçon suivant

Azure crée automatiquement des itinéraires vers les destinations par défaut. Vous pouvez créer des itinéraires personnalisés pour remplacer les itinéraires par défaut. Parfois, les itinéraires personnalisés peuvent entraîner l’échec de la communication. Utilisez la fonctionnalité de tronçon suivant de Network Watcher pour identifier l’itinéraire d’Azure pour acheminer le trafic.

1. Dans le portail Azure, sélectionnez **Tronçon suivant** sous **Network Watcher**.
2. Sélectionnez votre abonnement, entrez ou sélectionnez les valeurs suivantes, puis sélectionnez **Tronçon suivant**, comme illustré dans l’image qui suit :

    |Paramètre                  |Valeur                                                   |
    |---------                |---------                                               |
    | Resource group          | Sélectionner myResourceGroup                                 |
    | Machine virtuelle         | Sélectionner myVm                                            |
    | interface réseau       | myvm : le nom de votre interface réseau peut être différent.   |
    | Adresse IP source       | 10.0.0.4                                               |
    | Adresse IP de destination  | 13.107.21.200 : l’une des adresses de <www.bing.com>. |

    ![Tronçon suivant](./media/diagnose-vm-network-routing-problem/next-hop.png)

    Après quelques secondes, le résultat vous informe que le type de tronçon suivant est **Internet** et que **l’ID de la table de routage** est défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir qu’il existe un itinéraire système valide vers la destination.

3. Définissez la valeur de la zone **Adresse IP de destination** sur *172.31.0.100*, puis sélectionnez à nouveau **Tronçon suivant**. Le résultat retourné vous informe que **Aucun** est la valeur de la zone **Type de tronçon suivant** et que **l’ID de la table de routage** est également défini sur **System Route** (Itinéraire système). Ce résultat vous permet de savoir que s’il existe un itinéraire système valide vers la destination, il n’existe aucun tronçon suivant pour acheminer le trafic vers la destination.

## <a name="view-details-of-a-route"></a>Afficher les détails d’un itinéraire

1. Pour procéder à une analyse plus approfondie du routage, passez en revue les itinéraires réels de l’interface réseau. Dans la zone de recherche située en haut du portail, entrez *myvm* (ou le nom de l’interface réseau que vous avez activée). Quand le nom **myvm** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sous **SUPPORT + DÉPANNAGE**, sélectionnez **Itinéraires effectifs**, comme illustré dans l’image suivante :

    ![Itinéraires effectifs](./media/diagnose-vm-network-routing-problem/effective-routes.png)

    Lorsque vous avez exécuté le test à l’aide de la valeur 13.107.21.200 dans la zone [Use next hop](#use-next-hop) (Utiliser le tronçon suivant), l’itinéraire présentant le préfixe d’adresse 0.0.0.0/0 a été utilisé pour acheminer le trafic vers l’adresse, car aucun autre itinéraire n’inclut cette adresse. Par défaut, toutes les adresses non spécifiées dans le préfixe d’adresse d’un autre itinéraire sont acheminées vers Internet.

    Toutefois, lorsque vous avez exécuté le test à l’aide de 172.31.0.100, le résultat vous a informé de l’absence de type de tronçon suivant. Comme vous pouvez le voir dans l’image précédente, même s’il existe un itinéraire par défaut pour le préfixe 172.16.0.0/12, qui inclut l’adresse 172.31.0.100, la zone **TYPE DE TRONÇON SUIVANT** est définie sur **Aucun**. Azure crée un itinéraire par défaut pour 172.16.0.0/12, mais ne spécifie pas de type de tronçon suivant tant qu’aucune raison ne motive cette spécification. Si, par exemple, vous avez ajouté la plage d’adresses 172.16.0.0/12 à l’espace d’adressage du réseau virtuel, Azure modifie la zone **TYPE DE TRONÇON SUIVANT** pour la définir sur **Réseau virtuel** pour l’itinéraire. Une vérification permet ensuite d’afficher **Réseau virtuel** comme le **TYPE DE TRONÇON SUIVANT**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, supprimez-le, ainsi que toutes les ressources qu’il contient :

1. Entrez *myResourceGroup* dans le champ **Recherche** en haut du portail. Quand **myResourceGroup** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Entrez *myResourceGroup* dans **TAPER NOM DU GROUPE DE RESSOURCES :** puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une machine virtuelle et diagnostiqué un problème de routage réseau à partir de celle-ci. Vous avez appris qu’Azure crée plusieurs itinéraires par défaut et testé le routage vers deux destinations différentes. En savoir plus sur le [routage dans Azure](../virtual-network/virtual-networks-udr-overview.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json) et la [création d’itinéraires personnalisés](../virtual-network/manage-route-table.md?toc=%2fazure%2fnetwork-watcher%2ftoc.json#create-a-route).

Pour les connexions sortantes d’une machine virtuelle, vous pouvez également déterminer la latence, le trafic réseau autorisé et refusé entre la machine virtuelle et un point de terminaison, ainsi que l’itinéraire utilisé vers un point de terminaison, à l’aide de la fonctionnalité [Résolution des problèmes de connexion](network-watcher-connectivity-portal.md) de Network Watcher. Découvrez comment surveiller la communication entre une machine virtuelle et un point de terminaison, par exemple une adresse IP ou une URL, au fil du temps à l’aide de la fonctionnalité de contrôle de la connexion de Network Watcher.

> [!div class="nextstepaction"]
> [Surveiller une connexion réseau](connection-monitor.md)
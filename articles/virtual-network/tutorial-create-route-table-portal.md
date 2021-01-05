---
title: Router le trafic réseau - Tutoriel - Portail Azure
titlesuffix: Azure Virtual Network
description: Dans ce didacticiel, découvrez comment acheminer le trafic réseau avec une table de routage à l’aide du portail Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
Customer intent: I want to route traffic from one subnet, to a different subnet, through a network virtual appliance.
ms.service: virtual-network
ms.devlang: azurecli
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 03/13/2020
ms.author: kumud
ms.openlocfilehash: d630a41f9b83a852605ffad2a85ad6dd14bbac73
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86079647"
---
# <a name="tutorial-route-network-traffic-with-a-route-table-using-the-azure-portal"></a>Tutoriel : Acheminer le trafic réseau avec une table de routage à l’aide du portail Azure

Azure achemine le trafic entre tous les sous-réseaux au sein d’un réseau virtuel par défaut. Vous pouvez créer vos propres itinéraires pour remplacer le routage par défaut d’Azure. Les itinéraires personnalisés sont utiles, par exemple, quand vous souhaitez router le trafic entre des sous-réseaux via une appliance virtuelle réseau (NVA). Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une appliance NVA qui route le trafic
> * Créer une table de routage
> * Créer un itinéraire
> * Associer une table de routage à un sous-réseau
> * Déployer des machines virtuelles sur différents sous-réseaux
> * Router le trafic d’un sous-réseau vers un autre via une NVA

Ce tutoriel utilise le [portail Azure](https://portal.azure.com). Vous pouvez également utiliser [Azure CLI](tutorial-create-route-table-cli.md) ou [Azure PowerShell](tutorial-create-route-table-powershell.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-an-nva"></a>Créer une NVA

Les appliances virtuelles réseau (NVA) sont des machines virtuelles qui facilitent les fonctions réseau, telles que le routage et l’optimisation du pare-feu. Ce didacticiel suppose que vous utilisez **Windows Server 2016 Datacenter**. Si vous le souhaitez, vous pouvez sélectionner un autre système d’exploitation.

1. Dans le menu du [Portail Azure](https://portal.azure.com) ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

1. Choisissez **Sécurité** > **Windows Server 2016 Datacenter**.

    ![Windows Server 2016 Datacenter, créer une machine virtuelle, portail Azure](./media/tutorial-create-route-table-portal/vm-ws2016-datacenter.png)

1. Dans la page **Créer une machine virtuelle**, sous **Notions de base**, entrez ou sélectionnez ces informations :

    | Section | Paramètre | Action |
    | ------- | ------- | ----- |
    | **Détails du projet** | Abonnement | Choisissez votre abonnement. |
    | | Resource group | Sélectionnez **Créer**, entrez *myResourceGroup* et sélectionnez **OK**. |
    | **Détails de l’instance** | Nom de la machine virtuelle | Entrez *myVmNva*. |
    | | Région | Choisissez **(États-Unis) USA Est**. |
    | | Options de disponibilité | Choisissez **Aucune redondance d’infrastructure requise**. |
    | | Image | Choisissez **Windows Server 2016 Datacenter**. |
    | | Taille | Conservez la valeur par défaut, **Standard DS1 v2**. |
    | **Compte administrateur** | Nom d’utilisateur | Entrez un nom d’utilisateur de votre choix. |
    | | Mot de passe | Entrez le mot de passe de votre choix, qui doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm). |
    | | Confirmer le mot de passe | Entrez de nouveau le mot de passe. |
    | **Règles des ports d’entrée** | Aucun port d’entrée public | Sélectionnez **Aucun**. |
    | **Faites des économies** | Vous disposez déjà d’une licence Windows Server ? | Sélectionnez **Non**. |

    ![Notions de base, créer une machine virtuelle, portail Azure](./media/tutorial-create-route-table-portal/basics-create-virtual-machine.png)

    Ensuite, sélectionnez **Suivant : Disques >** .

1. Sous **Disques**, sélectionnez les paramètres répondant à vos besoins, puis sélectionnez **Suivant : Mise en réseau >** .

1. Sous **Réseaux** :

    1. Pour **Réseau virtuel**, sélectionnez **Créer**.
    
    1. Dans la boîte de dialogue **Créer un réseau virtuel**, sous **Nom**, entrez *myVirtualNetwork*.

    1. Dans **Espace d’adressage**, remplacez la plage d’adresses existante par *10.0.0.0/16*.

    1. Dans **Sous-réseaux**, sélectionnez l’icône **Supprimer** pour supprimer le sous-réseau existant, puis entrez les combinaisons suivantes de **Nom du sous-réseau** et **Plage d’adresses**. Une fois que vous avez entré un nom et une plage valides, une nouvelle ligne vide apparaît en dessous.

        | Nom du sous-réseau | Plage d’adresses |
        | ----------- | ------------- |
        | *Public* | *10.0.0.0/24* |
        | *Privé* | *10.0.1.0/24* |
        | *DMZ* | *10.0.2.0/24* |

    1. Sélectionnez **OK** pour quitter la boîte de dialogue.

    1. Dans **Sous-réseau**, choisissez **DMZ (10.0.2.0/24)** .

    1. Dans **Adresse IP publique**, choisissez **Aucune**, car cette machine virtuelle n’est pas appelée à se connecter via Internet.

    1. Sélectionnez **Suivant : Gestion >** .

1. Sous **Gestion** :

    1. Dans **Compte de stockage des diagnostics**, sélectionnez **Créer**.
    
    1. Dans la boîte de dialogue **Créer un compte de stockage**, entrez ou sélectionnez ces informations :

        | Paramètre | Valeur |
        | ------- | ----- |
        | Nom | *mynvastorageaccount* |
        | Type de compte | **Stockage (v1 universel)** |
        | Performances | **Standard** |
        | Réplication | **Stockage localement redondant (LRS)** |
    
    1. Sélectionnez **OK** pour quitter la boîte de dialogue.

    1. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Revoir + créer**, et Azure valide votre configuration.

1. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

    La création de la machine virtuelle ne nécessite que quelques minutes. Attendez qu’Azure ait terminé la création de la machine virtuelle. La page **Votre déploiement est en cours** affiche les détails de déploiement.

1. Lorsque votre machine virtuelle est prête, sélectionnez **Accéder à la ressource**.

## <a name="create-a-route-table"></a>Créer une table de routage

1. Dans le menu du [Portail Azure](https://portal.azure.com) ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

2. Dans la zone de recherche, entrez *table de route*. Quand le terme **table de route** s’affiche dans les résultats de recherche, sélectionnez-le.

3. Dans la page **table de route**, sélectionnez **Créer**.

4. Dans **Créer une table de routage**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | *myRouteTablePublic* |
    | Abonnement | Votre abonnement |
    | Resource group | **myResourceGroup** |
    | Emplacement | **(États-Unis) USA Est** |
    | Propagation de la route de la passerelle de réseau virtuel | **Activé** |

    ![Créer une table de route, portail Azure](./media/tutorial-create-route-table-portal/create-route-table.png)

5. Sélectionnez **Create** (Créer).

## <a name="create-a-route"></a>Créer un itinéraire

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre table de route. Recherchez et sélectionnez **Tables d’itinéraires**.

1. Choisissez le nom de votre table de route (**myRouteTablePublic**).

1. Choisissez **Itinéraires** > **Ajouter**.

    ![Ajouter une route, table de route, portail Azure](./media/tutorial-create-route-table-portal/add-route.png)

1. Dans **Ajouter un itinéraire**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de l’itinéraire | *ToPrivateSubnet* |
    | Préfixe de l’adresse | *10.0.1.0/24* (plage d’adresses du sous-réseau *Privé* créé) |
    | Type de tronçon suivant | **Appliance virtuelle** |
    | adresse de tronçon suivant | *10.0.2.4* (adresse dans la plage d’adresses du sous-réseau *DMZ*) |

1. Sélectionnez **OK**.

## <a name="associate-a-route-table-to-a-subnet"></a>Associer une table de routage à un sous-réseau

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre réseau virtuel. Recherchez et sélectionnez **Réseaux virtuels**.

1. Choisissez le nom de votre réseau virtuel (**myVirtualNetwork**).

1. Dans la barre de menus du réseau virtuel, choisissez **Sous-réseaux**.

1. Dans la barre de menus du réseau virtuel, choisissez **Public**.

1. Dans **Table de routage**, sélectionnez la table de route que vous avez créée (**myRouteTablePublic**), puis sélectionnez **Enregistrer** pour l’associer au sous-réseau *Public*.

    ![Associer une table de route, liste de sous-réseaux, réseau virtuel, portail Azure](./media/tutorial-create-route-table-portal/associate-route-table.png)

## <a name="turn-on-ip-forwarding"></a>Activer le transfert IP

Ensuite, activez le transfert IP pour votre nouvelle machine virtuelle NVA, *myVmNva*. Quand Azure envoie le trafic réseau vers *myVmNva*, si le trafic est destiné à une autre adresse IP, le transfert IP envoie le trafic vers l’emplacement approprié.

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre machine virtuelle. Recherchez et sélectionnez **Machines virtuelles**.

1. Choisissez le nom de votre machine virtuelle (**myVmNva**).

1. Dans la barre de menus de votre machine virtuelle NVA, sélectionnez **Réseaux**.

1. Sélectionnez **myvmnva123**. Il s’agit de l’interface réseau créée par Azure pour votre machine virtuelle. Azure ajoute des chiffres pour garantir un nom unique.

    ![Réseau, machine virtuelle NVA (appliance virtuelle réseau), portail Azure](./media/tutorial-create-route-table-portal/virtual-machine-networking.png)

1. Dans la barre de menus de l’interface réseau, sélectionnez **Configurations IP**.

1. Dans la page **Configurations IP**, définissez **Transfert IP** sur **Activé**, puis sélectionnez **Enregistrer**.

    ![Activer le transfert IP, configurations IP, interface réseau, machine virtuelle NVA (appliance virtuelle réseau), portail Azure](./media/tutorial-create-route-table-portal/enable-ip-forwarding.png)

## <a name="create-public-and-private-virtual-machines"></a>Créer des machines virtuelles publique et privée

Créez une machine virtuelle publique et une machine virtuelle privée dans le réseau virtuel. Vous les utiliserez plus tard pour voir si Azure achemine le trafic du sous-réseau *Public* vers le sous-réseau *Privé* via la NVA.

Pour créer la machine virtuelle publique et la machine virtuelle privée, suivez les étapes de la section [Créer une NVA](#create-an-nva) plus haut. Vous n’avez pas besoin d’attendre la fin du déploiement ou d’accéder à la ressource de machine virtuelle. Vous utiliserez la plupart des mêmes paramètres, sauf indication spécifique ci-dessous.

Avant de sélectionner **Créer** pour créer la machine virtuelle publique ou privée, accédez aux deux sous-sections suivantes ([Machine virtuelle publique](#public-vm) et [Machine virtuelle privée](#private-vm)), qui affichent les valeurs qui doivent être différentes. Vous pouvez passer à la section suivante ([Router le trafic via une NVA](#route-traffic-through-an-nva)) une fois qu’Azure a terminé le déploiement des deux machines virtuelles.

### <a name="public-vm"></a>Machine virtuelle publique

| Onglet | Paramètre | Valeur |
| --- | ------- | ----- |
| Concepts de base | Resource group | **myResourceGroup** |
| | Nom de la machine virtuelle | *myVmPublic* |
| | Aucun port d’entrée public | **Autoriser les ports sélectionnés** |
| | Sélectionner des ports d’entrée | **RDP** |
| Mise en réseau | Réseau virtuel | **myVirtualNetwork** |
| | Subnet | **Public (10.0.0.0/24)** |
| | Adresse IP publique | Valeur par défaut |
| Gestion | Compte de stockage de diagnostics | **mynvastorageaccount** |

### <a name="private-vm"></a>Machine virtuelle privée

| Onglet | Paramètre | Valeur |
| --- | ------- | ----- |
| Concepts de base | Resource group | **myResourceGroup** |
| | Nom de la machine virtuelle | *myVmPrivate* |
| | Aucun port d’entrée public | **Autoriser les ports sélectionnés** |
| | Sélectionner des ports d’entrée | **RDP** |
| Mise en réseau | Réseau virtuel | **myVirtualNetwork** |
| | Subnet | **Privé (10.0.1.0/24)** |
| | Adresse IP publique | Valeur par défaut |
| Gestion | Compte de stockage de diagnostics | **mynvastorageaccount** |

## <a name="route-traffic-through-an-nva"></a>Router le trafic via une NVA

### <a name="sign-in-to-myvmprivate-over-remote-desktop"></a>Se connecter à myVmPrivate via le Bureau à distance

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre machine virtuelle privée. Recherchez et sélectionnez **Machines virtuelles**.

1. Choisissez le nom de votre machine virtuelle privée (**myVmPrivate**).

1. Dans la barre de menus de la machine virtuelle, sélectionnez **Se connecter** pour créer une connexion Bureau à distance à la machine virtuelle privée.

1. Dans la page **Se connecter avec RDP**, sélectionnez **Télécharger le fichier RDP**. Azure crée un fichier de protocole RDP (Remote Desktop Protocol) ( *.rdp*) et le télécharge sur votre ordinateur.

1. Ouvrez le fichier *.rdp* téléchargé. Si vous y êtes invité, sélectionnez **Connexion**. Sélectionnez **Plus de choix** > **Utiliser un autre compte**, puis entrez le nom d’utilisateur et le mot de passe que vous avez spécifiés lors de la création de la machine virtuelle privée.

1. Sélectionnez **OK**.

1. Si vous recevez un avertissement lié au certificat pendant le processus de connexion, sélectionnez **Oui** pour vous connecter à la machine virtuelle.

### <a name="enable-icmp-through-the-windows-firewall"></a>Activer le protocole ICMP via le pare-feu Windows

Dans une étape ultérieure, vous utiliserez l’outil de traçage d’itinéraire pour tester le routage. L’outil de traçage d’itinéraire utilise le protocole ICMP (Internet Control Message Protocol), que le pare-feu Windows refuse par défaut. Activez le protocole ICMP (Internet Control Message Protocol) via le pare-feu Windows.

1. Dans le Bureau à distance de *myVmPrivate*, ouvrez PowerShell.

1. Entrez cette commande :

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```

    Vous allez utiliser le traçage d’itinéraire pour tester le routage dans ce tutoriel. Pour les environnements de production, nous ne recommandons pas d’autoriser le protocole ICMP (Internet Control Message Protocol) via le pare-feu Windows.

### <a name="turn-on-ip-forwarding-within-myvmnva"></a>Activer le transfert IP dans myVmNva

Vous avez [activé le transfert IP](#turn-on-ip-forwarding) pour l’interface réseau de la machine virtuelle à l’aide d’Azure. Le système d’exploitation de la machine virtuelle doit également transférer le trafic réseau. Activez le transfert IP pour le système d’exploitation de la machine virtuelle *myVmNva* avec ces commandes.

1. À partir d’une invite de commandes sur la machine virtuelle *myVmPrivate*, ouvrez un Bureau à distance à la machine virtuelle *myVmNva* :

    ```cmd
    mstsc /v:myvmnva
    ```

1. À partir de PowerShell sur la machine virtuelle *myVmNva*, entrez cette commande pour activer le transfert IP :

    ```powershell
    Set-ItemProperty -Path HKLM:\SYSTEM\CurrentControlSet\Services\Tcpip\Parameters -Name IpEnableRouter -Value 1
    ```

1. Redémarrez la machine virtuelle *myVmNva* : Dans la barre des tâches, sélectionnez **Démarrer** > **Alimentation**, **Autres (Planifié)**  > **Continuer**.

    Cette action déconnecte également la session Bureau à distance.

1. Lorsque la machine virtuelle *myVmNva* a redémarré, créez une session Bureau à distance sur la machine virtuelle *myVmPublic*. Tout en conservant la connexion à la machine virtuelle *myVmPrivate*, ouvrez une invite de commandes et exécutez cette commande :

    ```cmd
    mstsc /v:myVmPublic
    ```
1. Dans le Bureau à distance de *myVmPublic*, ouvrez PowerShell.

1. Activez le protocole ICMP (Internet Control Message Protocol) via le pare-feu Windows en entrant cette commande :

    ```powershell
    New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4
    ```

## <a name="test-the-routing-of-network-traffic"></a>Tester le routage du trafic réseau

Nous allons tout d’abord tester le routage du trafic réseau de la machine virtuelle *myVmPublic* vers la machine virtuelle *myVmPrivate*.

1. À partir de PowerShell, sur la machine virtuelle *myVmPublic*, entrez cette commande :

    ```powershell
    tracert myVmPrivate
    ```

    La réponse ressemble à cet exemple :

    ```powershell
    Tracing route to myVmPrivate.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.1.4]
    over a maximum of 30 hops:

    1    <1 ms     *        1 ms  10.0.2.4
    2     1 ms     1 ms     1 ms  10.0.1.4

    Trace complete.
    ```

    Vous voyez que le premier tronçon est en direction de 10.0.2.4, ce qui correspond à l’adresse IP privée de l’appliance virtuelle réseau (NVA). Le second tronçon est dirigé vers l’adresse IP privée de la machine virtuelle *myVmPrivate* : 10.0.1.4. Précédemment, vous avez ajouté l’itinéraire à la table de routage *myRouteTablePublic* et l’avez associé au sous-réseau *Public*. Azure a donc envoyé le trafic via la NVA et non directement au sous-réseau *Privé*.

1. Fermez la session Bureau à distance sur la machine virtuelle *myVmPublic*. Cela n’interrompt pas la connexion à la machine virtuelle *myVmPrivate*.

1. À partir d’une invite de commandes sur la machine virtuelle *myVmPrivate*, entrez cette commande :

    ```cmd
    tracert myVmPublic
    ```

    Cette commande teste le routage du trafic réseau de la machine virtuelle *myVmPrivate* vers la machine virtuelle *myVmPublic*. La réponse ressemble à cet exemple :

    ```cmd
    Tracing route to myVmPublic.vpgub4nqnocezhjgurw44dnxrc.bx.internal.cloudapp.net [10.0.0.4]
    over a maximum of 30 hops:

    1     1 ms     1 ms     1 ms  10.0.0.4

    Trace complete.
    ```

    Vous pouvez voir qu’Azure achemine le trafic directement de la machine virtuelle *myVmPrivate* vers la machine virtuelle *myVmPublic*. Par défaut, Azure achemine directement le trafic entre les sous-réseaux.

1. Fermez les sessions Bureau à distance sur la machine virtuelle *myVmPrivate*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, supprimez *myResourceGroup* ainsi que toutes les ressources qu’il contient :

1. Accédez au [portail Azure](https://portal.azure.com) pour gérer votre groupe de ressources. Recherchez et sélectionnez **Groupes de ressources**.

1. Choisissez le nom de votre groupe de ressources (**myResourceGroup**).

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Dans la boîte de dialogue de confirmation, entrez *myResourceGroup* pour **TAPEZ LE NOM DU GROUPE DE RESSOURCES**, puis sélectionnez **Supprimer**. Azure supprime *myResourceGroup* et toutes les ressources liées à ce groupe de ressources, y compris les tables de route, les comptes de stockage, les réseaux virtuels, les machines virtuelles, les interfaces réseau et les adresses IP publiques.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une table de routage que vous avez associée à un sous-réseau. Vous avez créé une appliance virtuelle réseau (NVA) simple qui a routé le trafic d’un sous-réseau public vers un sous-réseau privé. Vous pouvez maintenant déployer à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/networking) différentes appliances virtuelles réseau préconfigurées, qui fournissent de nombreuses fonctions réseau utiles. Pour en savoir plus sur le routage, consultez [Routage du trafic de réseau virtuel](virtual-networks-udr-overview.md) et [Créer, modifier ou supprimer une table de routage](manage-route-table.md).

Alors que vous pouvez déployer de nombreuses ressources Azure dans un réseau virtuel, Azure ne peut pas déployer des ressources pour certains services PaaS dans un réseau virtuel. Il est possible de restreindre l’accès aux ressources de certains services Azure PaaS, bien que la restriction doive uniquement concerner le trafic provenant d’un sous-réseau de réseau virtuel. Pour apprendre à restreindre l’accès réseau aux ressources Azure PaaS, consultez le tutoriel suivant.

> [!div class="nextstepaction"]
> [Restreindre l’accès réseau aux ressources PaaS](tutorial-restrict-network-access-to-resources.md)

> [!NOTE] 
> Les services Azure coûtent de l’argent. Azure Cost Management vous aide à définir des budgets et à configurer des alertes pour maintenir le contrôle des dépenses. Analysez, gérez et optimisez vos coûts Azure avec Cost Management. Pour plus d’informations, consultez le [guide de démarrage rapide sur l’analyse des coûts](https://docs.microsoft.com/azure/cost-management-billing/costs/quick-acm-cost-analysis?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn).
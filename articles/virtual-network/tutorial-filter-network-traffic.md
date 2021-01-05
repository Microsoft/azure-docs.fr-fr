---
title: Filtrer le trafic réseau - Tutoriel - Portail Azure
titlesuffix: Azure Virtual Network
description: Dans ce tutoriel, vous allez apprendre à filtrer le trafic réseau vers un sous-réseau, avec un groupe de sécurité réseau, à l’aide du portail Azure.
services: virtual-network
documentationcenter: virtual-network
author: KumudD
tags: azure-resource-manager
Customer intent: I want to filter network traffic to virtual machines that perform similar functions, such as web servers.
ms.service: virtual-network
ms.devlang: ''
ms.topic: tutorial
ms.tgt_pltfrm: virtual-network
ms.workload: infrastructure
ms.date: 12/13/2018
ms.author: kumud
ms.openlocfilehash: b5a136ae05b3cd410ca252b6d5a1df443aff6f7a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "75350138"
---
# <a name="tutorial-filter-network-traffic-with-a-network-security-group-using-the-azure-portal"></a>Tutoriel : Filtrer le trafic réseau avec un groupe de sécurité réseau à l’aide du portail Azure

Vous pouvez filtrer le trafic réseau entrant dans un sous-réseau de réseau virtuel ou qui en sort, avec un groupe de sécurité réseau. Les groupes de sécurité réseau contiennent des règles de sécurité qui filtrent le trafic réseau par adresse IP, port et protocole. Les règles de sécurité sont appliquées aux ressources déployées dans un sous-réseau. Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un groupe de sécurité réseau et les règles associées
> * Créer un réseau virtuel et associer un groupe de sécurité réseau à un sous-réseau
> * Déployer des machines virtuelles dans un sous-réseau
> * Tester les filtres de trafic

Si vous préférez, vous pouvez suivre ce tutoriel en utilisant [Azure CLI](tutorial-filter-network-traffic-cli.md) ou [PowerShell](tutorial-filter-network-traffic-powershell.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-virtual-network"></a>Créez un réseau virtuel

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**. 
2. Sélectionnez **Mise en réseau**, puis **Réseau virtuel**.
3. Entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Créer** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Nom                    | myVirtualNetwork                                   |
    | Espace d’adressage           | 10.0.0.0/16                                        |
    | Abonnement            | Sélectionnez votre abonnement.                          |
    | Resource group          | Sélectionnez **Créer** et entrez *myResourceGroup*. |
    | Emplacement                | Sélectionnez **USA Est**.                                |
    | Nom du sous-réseau            | mySubnet                                           |
    | Plage d’adresses du sous-réseau  | 10.0.0.0/24                                        |

## <a name="create-application-security-groups"></a>Créer des groupes de sécurité d’application

Un groupe de sécurité d’application vous permet de regrouper des serveurs avec des fonctions similaires, tels que des serveurs Web.

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**. 
2. Dans le champ **Rechercher dans la Place de marché**, entrez *Groupe de sécurité d’application*. Lorsque **Groupe de sécurité d’application** apparaît dans les résultats de recherche, sélectionnez cette entrée. Cliquez à nouveau sur **Groupe de sécurité d’application** sous **Tout**, puis sélectionnez **Créer**.
3. Saisissez ou sélectionnez les informations suivantes, puis sélectionnez **Créer** :

    | Paramètre        | Valeur                                                         |
    | ---            | ---                                                           |
    | Nom           | myAsgWebServers                                               |
    | Abonnement   | Sélectionnez votre abonnement.                                     |
    | Resource group | Sélectionnez **Utiliser l’existant**, puis **myResourceGroup**. |
    | Emplacement       | USA Est                                                       |

4. Répétez l’étape 3 en spécifiant les valeurs suivantes :

    | Paramètre        | Valeur                                                         |
    | ---            | ---                                                           |
    | Nom           | myAsgMgmtServers                                              |
    | Abonnement   | Sélectionnez votre abonnement.                                     |
    | Resource group | Sélectionnez **Utiliser l’existant**, puis **myResourceGroup**. |
    | Emplacement       | USA Est                                                       |

## <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**. 
2. Sélectionnez **Mise en réseau**, puis **Groupe de sécurité réseau**.
3. Saisissez ou sélectionnez les informations suivantes, puis sélectionnez **Créer** :

    |Paramètre|Valeur|
    |---|---|
    |Nom|myNsg|
    |Abonnement| Sélectionnez votre abonnement.|
    |Resource group | Sélectionnez **Utiliser l’existant**, puis *myResourceGroup*.|
    |Emplacement|USA Est|

## <a name="associate-network-security-group-to-subnet"></a>Associer le groupe de sécurité réseau au sous-réseau

1. Dans le champ *Rechercher des ressources, services et documents* en haut du portail, commencez à taper *myNsg*. Quand le nom **myNsg** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sous **PARAMÈTRES**, sélectionnez **Sous-réseaux**, puis **+ Associer**, comme indiqué dans l’image suivante :

    ![Associer un groupe de sécurité réseau à un sous-réseau](./media/tutorial-filter-network-traffic/associate-nsg-subnet.png)

3. Sous **Associer un sous-réseau**, sélectionnez **Réseau virtuel** puis **myVirtualNetwork**. Sélectionnez **Sous-réseau**, puis **mySubnet**, et enfin sélectionnez **OK**.

## <a name="create-security-rules"></a>Créer des règles de sécurité

1. Sous **PARAMÈTRES**, sélectionnez **Règles de sécurité de trafic entrant**, puis **+ Ajouter**, comme indiqué dans l’image suivante :

    ![Ajouter une règle de sécurité de trafic entrant](./media/tutorial-filter-network-traffic/add-inbound-rule.png)

2. Créez une règle de sécurité qui autorise les ports 80 et 443 dans le groupe de sécurité d’application **myAsgWebServers**. Sous **Ajouter une règle de sécurité de trafic entrant**, entrez ou sélectionnez les valeurs suivantes, acceptez les valeurs par défaut restantes, puis sélectionnez **Ajouter** :

    | Paramètre                 | Valeur                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | Destination             | Sélectionnez **Groupe de sécurité d’application**, puis **myAsgWebServers** pour **Groupe de sécurité d’application**.  |
    | Plages de ports de destination | Entrer 80 et 443                                                                                                    |
    | Protocol                | Sélectionner TCP                                                                                                      |
    | Nom                    | Allow-Web-All                                                                                                   |

3. Répétez l’étape 2 en utilisant les valeurs suivantes :

    | Paramètre                 | Valeur                                                                                                           |
    | ---------               | ---------                                                                                                       |
    | Destination             | Sélectionnez **Groupe de sécurité d’application**, puis **myAsgMgmtServers** pour **Groupe de sécurité d’application**. |
    | Plages de ports de destination | Entrer 3389                                                                                                      |
    | Protocol                | Sélectionner TCP                                                                                                      |
    | Priority                | Entrer 110                                                                                                       |
    | Nom                    | Allow-RDP-All                                                                                                   |

    Dans ce tutoriel, le protocole RDP (port 3389) est exposé à Internet pour la machine virtuelle qui est affectée au groupe de sécurité d’application *myAsgMgmtServers*. Pour les environnements de production, au lieu d’exposer le port 3389 sur Internet, il est recommandé de vous connecter aux ressources Azure que vous souhaitez gérer à l’aide d’une connexion réseau VPN ou privée.

Une fois que vous avez terminé les étapes 1 à 3, passez en revue les règles que vous avez créées. Votre liste doit ressembler à la liste dans l’image suivante :

![Règles de sécurité](./media/tutorial-filter-network-traffic/security-rules.png)

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Créez deux machines virtuelles dans le réseau virtuel.

### <a name="create-the-first-vm"></a>Créer la première machine virtuelle

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**. 
2. Sélectionnez **Compute**, puis **Windows Server 2016 Datacenter**.
3. Entrez ou sélectionnez les informations suivantes, puis acceptez les valeurs par défaut pour les autres paramètres :

    |Paramètre|Valeur|
    |---|---|
    |Abonnement| Sélectionnez votre abonnement.|
    |Resource group| Sélectionnez **Utiliser l’existant**, puis **myResourceGroup**.|
    |Nom|myVmWeb|
    |Emplacement| Sélectionnez **USA Est**.|
    |Nom d'utilisateur| Entrez un nom d’utilisateur de votre choix.|
    |Mot de passe| Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/windows/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|

   

4. Choisissez une taille de machine virtuelle, puis cliquez sur **Sélectionner**.
5. Sous **Réseau**, sélectionnez les valeurs suivantes, puis acceptez les autres valeurs par défaut :

    |Paramètre|Valeur|
    |---|---|
    |Réseau virtuel |Sélectionnez **myVirtualNetwork**.|
    |Groupe de sécurité réseau de la carte réseau |Sélectionnez **Aucun**.|
  

6. Sélectionnez **Vérifier + créer** en bas à gauche, puis sélectionnez **Créer** pour démarrer le déploiement de la machine virtuelle.

### <a name="create-the-second-vm"></a>Créer la seconde machine virtuelle

Répétez les étapes 1 à 6, mais à l’étape 3, nommez la machine virtuelle *myVmMgmt*. Le déploiement de la machine virtuelle ne nécessite que quelques minutes. Ne passez à l’étape suivante que lorsque la machine virtuelle est déployée.

## <a name="associate-network-interfaces-to-an-asg"></a>Associer des interfaces réseau à un groupe de sécurité d’application

Lorsque le portail a créé les machines virtuelles, il a créé une interface réseau pour chaque machine virtuelle, et attaché l’interface réseau à la machine virtuelle. Ajoutez l’interface réseau de chaque machine virtuelle à l’un des groupes de sécurité d’application que vous avez créés précédemment :

1. Dans le champ *Rechercher des ressources, services et documents* en haut du portail, commencez à taper *myVmWeb*. Sélectionnez la machine virtuelle **myVmPrivate** lorsqu’elle apparaît dans les résultats de la recherche.
2. Sous **PARAMÈTRES**, sélectionnez **Mise en réseau**.  Sélectionnez **Configurer les groupes de sécurité d’application**, puis sélectionnez **myAsgWebServers** pour **Groupes de sécurité d’application**. Enfin cliquez sur **Enregistrer**, comme illustré dans l’image suivante :

    ![Associer à un groupe de sécurité d’application](./media/tutorial-filter-network-traffic/associate-to-asg.png)

3. Répétez les étapes 1 et 2, en recherchant la machine virtuelle **myVmMgmt** et en sélectionnant le groupe de sécurité d’application (ASG) **myAsgMgmtServers**.

## <a name="test-traffic-filters"></a>Tester les filtres de trafic

1. Connectez-vous à la machine virtuelle *myVmMgmt*. Entrez *myVmMgmt* dans la zone de recherche en haut du portail. Quand **myVmMgmt** apparaît dans les résultats de la recherche, sélectionnez-la. Sélectionnez le bouton **Connexion**.
2. Sélectionnez **Télécharger le fichier RDP**.
3. Ouvrez le fichier RDP téléchargé et sélectionnez **Connecter**. Entrez le nom d’utilisateur et le mot de passe spécifiés lors de la création de la machine virtuelle. Vous devrez peut-être sélectionner **Plus de choix**, puis **Utiliser un autre compte**, pour spécifier les informations d’identification que vous avez entrées lorsque vous avez créé la machine virtuelle.
4. Sélectionnez **OK**.
5. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Si vous recevez l’avertissement, sélectionnez **Oui** ou **Continuer** pour poursuivre le processus de connexion.

    La connexion réussit, car le port 3389 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgMgmtServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmMgmt*.

6. Connectez-vous à la machine virtuelle *myVmWeb* à partir de la machine virtuelle *myVmMgmt* en entrant la commande suivante dans une session PowerShell :

    ``` 
    mstsc /v:myVmWeb
    ```

    Vous pouvez vous connecter à la machine virtuelle myVmWeb à partir de la machine virtuelle myVmMgmt, car les machines virtuelles du même réseau virtuel peuvent communiquer entre elles sur n’importe quel port, par défaut. Cependant, vous ne pouvez pas créer une connexion Bureau à distance à la machine virtuelle *myVmWeb* depuis Internet, car la règle de sécurité du groupe *myAsgWebServers* n’autorise pas le trafic entrant provenant d’Internet sur le port 3389. Par défaut, le trafic entrant est refusé pour toutes les ressources.

7. Pour installer Microsoft IIS sur la machine virtuelle *myVmWeb*, entrez la commande suivante à partir d’une session PowerShell sur la machine virtuelle *myVmWeb* :

    ```powershell
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    ```

8. Une fois l’installation d’IIS terminée, déconnectez-vous de la machine virtuelle *myVmWeb*, ce qui vous laisse dans la connexion Bureau à distance de la machine virtuelle *myVmMgmt*.
9. Déconnectez-vous de la machine virtuelle *myVmMgmt*.
10. Dans le champ *Rechercher des ressources, services et documents* en haut du portail Azure, commencez à taper *myVmWeb* depuis votre ordinateur. Quand **myVmWeb** apparaît dans les résultats de la recherche, sélectionnez cette entrée. Notez l’**adresse IP publique** de votre machine virtuelle. L’adresse affichée dans l’image suivante est 137.135.84.74, mais votre adresse est différente :

    ![Adresse IP publique](./media/tutorial-filter-network-traffic/public-ip-address.png)
  
11. Pour confirmer que vous pouvez accéder au serveur web *myVmWeb* depuis Internet, ouvrez un navigateur web sur votre ordinateur et accédez à `http://<public-ip-address-from-previous-step>`. L’écran d’accueil ISS s’affiche, car le port 80 autorise le trafic entrant depuis Internet vers le groupe de sécurité d’application *myAsgWebServers* dans lequel se situe l’interface réseau attachée à la machine virtuelle *myVmWeb*.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du groupe de ressources, supprimez-le, ainsi que toutes les ressources qu’il contient :

1. Entrez *myResourceGroup* dans le champ **Recherche** en haut du portail. Quand **myResourceGroup** apparaît dans les résultats de la recherche, sélectionnez-le.
2. Sélectionnez **Supprimer le groupe de ressources**.
3. Entrez *myResourceGroup* dans **TAPER NOM DU GROUPE DE RESSOURCES :** puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé un groupe de sécurité réseau et vous l’avez associé à un sous-réseau d’un réseau virtuel. Pour en savoir plus sur les groupes de sécurité réseau, consultez [Vue d’ensemble d’un groupe de sécurité réseau](security-overview.md) et [Gérer un groupe de sécurité réseau](manage-network-security-group.md).

Azure achemine par défaut le trafic entre les sous-réseaux. À la place, vous pouvez choisir par exemple d’acheminer le trafic entre les sous-réseaux via une machine virtuelle, agissant comme un pare-feu. Pour apprendre à créer une table de routage, passez au didacticiel suivant.

> [!div class="nextstepaction"]
> [Créer une table de routage](./tutorial-create-route-table-portal.md)

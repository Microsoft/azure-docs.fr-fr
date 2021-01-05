---
title: 'Tutoriel : Équilibrer la charge de machines virtuelles à l’intérieur d’une zone - Portail Azure'
titleSuffix: Azure Load Balancer
description: Ce tutoriel montre comment créer un équilibreur de charge standard avec serveur frontal zonal pour équilibrer la charge de machines virtuelles dans une zone de disponibilité à l’aide du portail Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines within a specific zone in a region.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: f91c9c0f401a455543b12af81eed48bd1a3349bd
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696436"
---
# <a name="tutorial-load-balance-vms-within-an-availability-zone-with-standard-load-balancer-by-using-the-azure-portal"></a>Tutoriel : Équilibrer la charge de machines virtuelles dans une zone de disponibilité avec Standard Load Balancer à l’aide du portail Azure

Ce tutoriel crée une instance publique [Azure Standard Load Balancer](https://aka.ms/azureloadbalancerstandard) avec un serveur frontal zonal utilisant une adresse IP publique standard à l’aide du portail Azure. Dans ce scénario, vous pouvez spécifier une zone particulière pour vos instances de serveur frontal et de serveur principal, afin d’aligner votre chemin de données et vos ressources avec une zone spécifique. Vous apprenez à effectuer les fontions suivantes :

> [!div class="checklist"]
> * Créer un équilibreur de charge Standard Load Balancer avec un serveur frontal zonal.
> * Créer des groupes de sécurité réseau pour définir des règles applicables au trafic entrant.
> * Créer des machines virtuelles zonales et les attacher à un équilibreur de charge.
> * Créer une sonde d’intégrité d’équilibreur de charge.
> * Créer des règles de trafic pour l’équilibreur de charge.
> * Créer un site Internet Information Services (IIS) de base.
> * Afficher un équilibrage de charge en action.

Pour plus d’informations sur l’utilisation des zones de disponibilité avec un équilibreur de charge standard, voir [Standard Load Balancer et zones de disponibilité](load-balancer-standard-availability-zones.md).

Si vous préférez, utilisez [Azure CLI](./quickstart-load-balancer-standard-public-cli.md) pour effectuer ce tutoriel.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-public-standard-load-balancer-instance"></a>Créer une instance Standard Load Balancer publique

Standard Load Balancer prend uniquement en charge une adresse IP publique standard. Lorsque vous créez une nouvelle adresse IP publique en créant l’équilibreur de charge, celle-ci est automatiquement configurée comme version de référence SKU standard. Elle est également automatiquement redondante dans une zone.

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.
2. Sous l’onglet **De base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Vérifier + créer** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer** et tapez *MyResourceGroupZLB* dans la zone de texte.|
    | Nom                   | *myLoadBalancer*                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | Sélectionnez **Standard**.                          |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. |
    | Nom de l’adresse IP publique              | Tapez *myPublicIP* dans la zone de texte.   |
    |Zone de disponibilité| Sélectionnez **1**.    |
3. Sous l’onglet **Vérifier + créer**, cliquez sur **Créer**.   

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous créez un réseau virtuel. Vous créez également deux machines virtuelles dans la même zone (à savoir, la zone 1) pour la région à ajouter au pool principal de votre équilibreur de charge. Vous installez ensuite IIS sur les machines virtuelles pour faciliter le test de l’équilibreur de charge redondant interzone. Si une machine virtuelle échoue, la sonde d’intégrité de la machine virtuelle dans la même zone échoue. Le trafic continue à être pris en charge par les autres machines virtuelles au sein de la même zone.

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupZLB (sélectionnez un groupe de ressources existant) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Europe Ouest      |
| **\<IPv4-address-space>**   | 10.0.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.0.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource**. Dans la zone de recherche, entrez **Groupe de sécurité réseau**. Dans la page de groupe de sécurité réseau, sélectionnez **Créer**.
2. Dans la page **Création d’un groupe de sécurité réseau**, entrez ces valeurs :
   - **myNetworkSecurityGroup**, pour le nom du groupe de sécurité réseau.
   - **myResourceGroupLBAZ**, pour le nom du groupe de ressources existant.
   
     ![Créer un groupe de sécurité réseau](./media/tutorial-load-balancer-standard-zonal-portal/create-network-security-group.png)

### <a name="create-nsg-rules"></a>Créer les règles du groupe de sécurité réseau

Dans cette section, vous créez des règles de groupe de sécurité réseau pour autoriser les connexions entrantes qui utilisent les protocoles HTTP et RDP (Remote Desktop Protocol) de Microsoft à l’aide du portail Azure.

1. Sélectionnez **Toutes les ressources** dans le menu le plus à gauche du portail Azure. Ensuite, recherchez et sélectionnez **myNetworkSecurityGroup**. Il se trouve dans le groupe de ressources **myResourceGroupZLB**.
2. Sous **Paramètres**, sélectionnez **Règles de sécurité de trafic entrant**. Sélectionnez ensuite **Ajouter**.
3. Entrez ces valeurs pour la règle de sécurité de trafic entrant nommée **myHTTPRule** afin d’autoriser les connexions HTTP entrantes qui utilisent le port 80 :
    - **Service Tag**, (Identification) pour **Source**.
    - **Internet**, pour **Source service tag** (Identification de source).
    - **80**, pour **Destination port ranges** (Plages de ports de destination).
    - **TCP**, pour **Protocole**.
    - **Autoriser** pour **Action**.
    - **100**, pour **Priorité**.
    - **myHTTPRule**, pour **Nom**.
    - **Autoriser HTTP**, pour **Description**.
4. Sélectionnez **OK**.
 
   ![Créer les règles du groupe de sécurité réseau](./media/load-balancer-standard-public-availability-zones-portal/8-load-balancer-nsg-rules.png)

5. Répétez les étapes 2 à 4 pour créer une autre règle nommée **myRDPRule**. Cette règle permet une connexion RDP entrante utilisant le port 3389, avec les valeurs suivantes :
    - **Service Tag**, (Identification) pour **Source**.
    - **Internet**, pour **Source service tag** (Identification de source).
    - **3389**, pour **Destination port ranges** (Plages de ports de destination).
    - **TCP**, pour **Protocole**.
    - **Autoriser** pour **Action**.
    - **200**, pour **Priorité**.
    - **myRDPRule**, pour **Nom**.
    - **Autoriser RDP**, pour **Description**.

      ![Créer une règle RDP](./media/tutorial-load-balancer-standard-zonal-portal/create-rdp-rule.png)

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** > **Calcul** > **Windows Server 2016 Datacenter**. Entrez ces valeurs pour la machine virtuelle :
    - **myVM1**, pour le nom de la machine virtuelle.        
    - **azureuser**, pour le nom d’utilisateur administrateur.    
    - **myResourceGroupZLB**, pour le **Groupe de ressources**. Sélectionnez **Utiliser l’existant**, puis **myResourceGroupZLB**.
2. Sélectionnez **OK**.
3. Sélectionnez **DS1_V2** pour la taille de la machine virtuelle. Choisissez **Sélectionner**.
4. Entrez ces valeurs pour les paramètres de la machine virtuelle :
    - **zone 1**, pour la zone de disponibilité dans laquelle vous placez la machine virtuelle.
    -  **MyVNet**. Vérifiez qu’il est sélectionné en tant que réseau virtuel.
    - **myVM1PIP**, pour l’adresse IP publique standard que vous créez. Sélectionnez **Créer nouveau**. Puis pour le type de nom, sélectionnez **myVM1PIP**. Pour **Zone**, sélectionnez **1**. Par défaut, la référence SKU de l’adresse IP est standard.
    - **myBackendSubnet**. Assurez-vous qu’il est sélectionné en tant que sous-réseau.
    - **myNetworkSecurityGroup**, pour le nom du pare-feu du groupe de sécurité réseau qui existe déjà.
5. Sélectionnez **Désactivé** pour désactiver les diagnostics de démarrage.
6. Sélectionnez **OK**. Vérifiez les paramètres sur la page de résumé. Sélectionnez ensuite **Créer**.
7. Répétez les étapes 1 à 6 pour créer une deuxième machine virtuelle nommée **myVM2**, dans la Zone 1. Choisissez **myVnet** comme réseau virtuel. Choisissez **myVM2PIP** comme adresse IP publique standard. Choisissez **myBackendSubnet** comme sous-réseau. Et choisissez **myNetworkSecurityGroup** comme groupe de sécurité réseau.

    ![Créer des machines virtuelles](./media/tutorial-load-balancer-standard-zonal-portal/create-virtual-machine.png) 

### <a name="install-iis-on-vms"></a>Installer IIS sur des machines virtuelles

1. Dans le menu le plus à gauche, sélectionnez **Toutes les ressources**. Puis dans la liste de ressources, sélectionnez **myVM1**. Il se trouve dans le groupe de ressources **myResourceGroupZLB**.
2. Sur la page **Vue d’ensemble**, sélectionnez **Connexion** pour utiliser RDP pour accéder à la machine virtuelle.
3. Connectez-vous à la machine virtuelle avec le nom d’utilisateur et le mot de passe que vous avez indiqué lors de la création de la machine virtuelle. Pour spécifier les informations d’identification que vous avez entrées lorsque vous avez créé la machine virtuelle, vous devrez peut-être sélectionner **Plus de choix**. Ensuite, sélectionnez **Utiliser un autre compte**. Et enfin sélectionner **OK**. Un avertissement de certificat peut s’afficher pendant le processus de connexion. Sélectionnez **Oui** pour poursuivre le processus de connexion.
4. Sur le bureau du serveur, accédez à **Outils d’administration Windows** > **Windows PowerShell**.
6. Dans la fenêtre **PowerShell**, exécutez les commandes suivantes pour installer le serveur IIS. Ces commandes suppriment également le fichier iisstart.htm par défaut, puis ajoutent un nouveau fichier iisstart.htm qui affiche le nom de la machine virtuelle :

   ```azurepowershell-interactive
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from" + $env:computername)
   ```
7. Fermez la session RDP avec **myVM1**.
8. Répétez les étapes 1 à 7 pour installer IIS sur **myVM2**.

## <a name="create-load-balancer-resources"></a>Créer les ressources d’équilibreur de charge

Dans cette section, vous configurez les paramètres de l’équilibreur de charge pour un pool d’adresses principales et une sonde d’intégrité. Vous spécifiez également des règles de traduction d’adresse réseau et d’équilibreur de charge.


### <a name="create-a-backend-address-pool"></a>Créer un pool d’adresses principal

Pour distribuer le trafic vers les machines virtuelles, un pool d’adresses principales contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. Créez le pool d’adresses principal **myBackendPool** pour inclure **VM1** et **VM2**.

1. Dans le menu le plus à gauche, sélectionnez **Toutes les ressources**. Puis sélectionnez **myLoadBalancer** à partir de la liste des ressources.
2. Sous **Paramètres**, sélectionnez **Backend Pools (Pools principaux)** . Sélectionnez ensuite **Ajouter**.
3. Sur la page **Ajouter un pool principal**, procédez comme suit :
    - Saisissez **myBackEndPool** comme nom du pool principal.
    - Pour **Réseau virtuel**, dans le menu déroulant, sélectionnez **myVNet**. 
    - Dans **Machine virtuelle** et **Adresse IP**, ajoutez **myVM1** et **myVM2** ainsi que leurs adresses IP publiques.
4. Sélectionnez **Ajouter**.
5. Vérifiez que le paramétrage du pool principal de l’équilibreur de charge affiche les deux machines virtuelles : **myVM1** et **myVM2**.
 
    ![Créer un pool principal](./media/tutorial-load-balancer-standard-zonal-portal/create-backend-pool.png) 

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Utilisez une sonde d’intégrité pour permettre à l’équilibrage de charge de surveiller l’état de votre application. La sonde d’intégrité ajoute ou supprime dynamiquement des machines virtuelles de la rotation d’équilibrage de charge en fonction de leur réponse aux vérifications d’intégrité. Créez une sonde d’intégrité **myHealthProbe** pour surveiller l’intégrité des machines virtuelles.

1. Dans le menu le plus à gauche, sélectionnez **Toutes les ressources**. Puis sélectionnez **myLoadBalancer** à partir de la liste des ressources.
2. Sous **Paramètres**, sélectionnez **Sondes d’intégrité**. Sélectionnez ensuite **Ajouter**.
3. Utilisez ces valeurs pour créer la sonde d’intégrité :
    - **myHealthProbe**, pour le nom de la sonde d’intégrité.
    - **HTTP**, pour le type de protocole.
    - **80**, pour le numéro de port.
    - **15**, pour **l’intervalle** en secondes entre les tentatives de la sonde.
    - **2**, pour le nombre de **seuils de défaillance** ou d’échecs de sonde consécutifs qui se produisent avant qu’une machine virtuelle soit considérée comme défaillante.
4. Sélectionnez **OK**.

   ![Ajouter une sonde d’intégrité](./media/load-balancer-standard-public-availability-zones-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibreur de charge définit la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Créez une règle d’équilibreur de charge nommée **myLoadBalancerRuleWeb** pour l’écoute sur le port 80 dans le serveur frontal **FrontendLoadBalancer**. La règle envoie le trafic réseau équilibré en charge au pool d’adresses principales **myBackEndPool**, également à l’aide du port 80. 

1. Dans le menu le plus à gauche, sélectionnez **Toutes les ressources**. Puis sélectionnez **myLoadBalancer** à partir de la liste des ressources.
2. Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**. Sélectionnez ensuite **Ajouter**.
3. Utilisez ces valeurs pour configurer la règle d’équilibrage de charge :
    - Choisissez **myHTTPRule**, pour le nom de la règle d’équilibrage de charge.
    - **TCP**, pour le type de protocole.
    - **80**, pour le numéro de port.
    - **80**, pour le port principal.
    - **myBackendPool**, pour le nom du pool principal.
    - **myHealthProbe**, pour le nom de la sonde d’intégrité.
4. Sélectionnez **OK**.
    
    ![Ajouter une règle d’équilibrage de charge](./media/tutorial-load-balancer-standard-zonal-portal/load-balancing-rule.png)

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge
1. Recherchez l’adresse IP publique de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Sélectionnez **Toutes les ressources**. Puis sélectionnez **myPublicIP**. 

2. Copiez l’adresse IP publique. Puis collez la dans la barre d’adresse de votre navigateur. La page par défaut qui inclut le nom de la page du serveur web s’affiche dans le navigateur.

      ![Serveur web IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)
3. Pour voir l’équilibrage de charge en action, forcez l’arrêt de la machine virtuelle qui s’affiche. Actualisez le navigateur pour voir l’autre nom de serveur affiché dans le navigateur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, l’équilibreur de charge et toutes les ressources associées. Sélectionnez le groupe de ressources qui contient l’équilibreur de charge. Puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Passez maintenant à l’article suivant pour savoir comment équilibrer la charge des machines virtuelles entre les zones de disponibilité.
> [!div class="nextstepaction"]
> [Équilibrer la charge de machines virtuelles entre des zones de disponibilité](tutorial-load-balancer-standard-public-zone-redundant-portal.md)
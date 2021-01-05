---
title: 'Tutoriel : Équilibrer la charge des machines virtuelles dans les zones de disponibilité - Portail Azure'
titleSuffix: Azure Load Balancer
description: Ce didacticiel montre comment créer un équilibreur de charge standard avec frontend redondant interzone pour équilibrer la charge des machines virtuelles entre des zones de disponibilité à l’aide du portail Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to create a load balancer that load balances incoming internet traffic to virtual machines across availability zones in a region, so that the customers can still access the web service if a datacenter is unavailable.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/27/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: 4e07285eca0fd10b73b386fcf139cdad5b94ddc2
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696402"
---
# <a name="tutorial-load-balance-vms-across-availability-zones-with-a-standard-load-balancer-using-the-azure-portal"></a>Tutoriel : Équilibrer la charge des machines virtuelles sur les zones de disponibilité avec un équilibreur de charge standard à l’aide du portail Azure

L’équilibrage de charge offre un niveau plus élevé de disponibilité en répartissant les demandes entrantes sur plusieurs machines virtuelles. Ce tutoriel détaille les étapes à suivre pour créer un Standard Load Balancer public qui équilibre la charge des machines virtuelles entre des zones de disponibilité. Vous pouvez ainsi protéger vos applications et vos données dans l’éventualité peu probable d’une défaillance ou d’une perte d’un centre de données entier. Avec la redondance dans une zone, une ou plusieurs zones de disponibilité peuvent échouer sans empêcher le chemin de données de survivre tant qu’une zone de la région reste intègre. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un équilibreur de charge standard
> * Créer des groupes de sécurité réseau pour définir des règles applicables au trafic entrant
> * Créer des machines virtuelles redondantes interzone déployées dans plusieurs zones et les attacher à un équilibreur de charge
> * Créer une sonde d’intégrité d’équilibreur de charge
> * Créer des règles de trafic pour l’équilibrage de charge
> * Créer un site IIS de base
> * Afficher un équilibrage de charge en action

Pour plus d’informations sur l’utilisation des zones de disponibilité avec un équilibreur de charge standard, voir [Équilibreur de charge standard et zones de disponibilité](load-balancer-standard-availability-zones.md).

Si vous préférez, vous pouvez suivre ce didacticiel en utilisant [Azure CLI](./quickstart-load-balancer-standard-public-cli.md).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-standard-load-balancer"></a>Créer un équilibreur de charge standard

L’équilibreur de charge standard prend uniquement en charge une adresse IP publique standard. Lorsque vous créez une nouvelle adresse IP publique en créant l’équilibreur de charge, celle-ci est automatiquement configurée comme version de référence standard et est également automatiquement redondante interzone.

1. En haut à gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.
2. Sous l’onglet **De base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Vérifier + créer** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer** et tapez *MyResourceGroupLBAZ* dans la zone de texte.|
    | Nom                   | *myLoadBalancer*                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | Sélectionnez **Standard**.                          |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. |
    | Nom de l’adresse IP publique              | Tapez *myPublicIP* dans la zone de texte.   |
    |Zone de disponibilité| Sélectionnez **Redondant dans une zone**.    |
   

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez créer un réseau virtuel, des machines virtuelles dans différentes zones de la région, puis installer IIS sur les machines virtuelles afin de tester l’équilibreur de charge redondant interzone. Ainsi, si une zone échoue, la sonde d’intégrité de la machine virtuelle de la même zone échoue, et le trafic continue à être pris en charge par les machines virtuelles dans les autres zones.

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLBAZ (sélectionnez un groupe de ressources existant) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Europe Ouest      |
| **\<IPv4-address-space>**   | 10.0.0.0/16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.0.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

## <a name="create-a-network-security-group"></a>Créer un groupe de sécurité réseau

Créez un groupe de sécurité réseau pour définir les connexions entrantes vers votre réseau virtuel.

1. Sur le côté gauche de l’écran, cliquez sur **Créer une ressource**, dans la zone de recherche tapez *Groupe de sécurité réseau* et dans la page du groupe de sécurité réseau, cliquez sur **Créer**.
2. Dans la page Création d’un groupe de sécurité réseau, entrez ces valeurs :
    - *myNetworkSecurityGroup* : pour le nom du groupe de sécurité réseau.
    - *myResourceGroupLBAZ* : pour le nom du groupe de ressources existant.
   
![La capture d’écran montre le volet Créer un groupe de sécurité réseau.](./media/load-balancer-standard-public-availability-zones-portal/create-nsg.png)

### <a name="create-network-security-group-rules"></a>Créer des règles pour le groupe de sécurité réseau

Dans cette section, vous allez créer des règles pour le groupe de sécurité réseau à l’aide du portail Azure afin d’autoriser les connexions entrantes via les protocoles HTTP et RDP.

1. Dans le portail Azure, cliquez sur **Toutes les ressources** dans le menu de gauche, puis recherchez et cliquez sur **myNetworkSecurityGroup** qui se trouve dans le groupe de ressources **myResourceGroupLBAZ**.
2. Sous **Paramètres**, cliquez sur **Règles de sécurité entrantes**, puis sur **Ajouter**.
3. Entrez ces valeurs pour la règle de sécurité entrante nommée *myHTTPRule* afin d’autoriser les connexions HTTP entrantes à l’aide du port 80 :
    - *Service Tag* : pour **Source**.
    - *Internet* : pour **Balise de service source**
    - *80* : pour **Plages de port de destination**
    - *TCP* : pour **Protocole**
    - *Allow* : pour **Action**
    - *100* pour **Priorité**
    - *myHTTPRule* : pour le nom de la règle de l’équilibreur de charge.
    - *Allow HTTP* : pour la description de la règle de l’équilibreur de charge.
4. Cliquez sur **OK**.
 
   ![La capture d’écran montre le volet Ajouter une règle de sécurité de trafic entrant.](./media/load-balancer-standard-public-availability-zones-portal/8-load-balancer-nsg-rules.png)
5. Répétez les étapes 2 à 4 pour créer une autre règle nommée *myRDPRule* pour autoriser une connexion RDP entrante à l’aide du port 3389 avec les valeurs suivantes :
    - *Service Tag* : pour **Source**.
    - *Internet* : pour **Balise de service source**
    - *3389* : pour **Plages de port de destination**
    - *TCP* : pour **Protocole**
    - *Allow* : pour **Action**
    - *200* pour **Priorité**
    - *myRDPRule* pour le nom
    - *Allow RDP* pour la description

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Dans différentes zones (zone 1, zone 2 et zone 3) de la région, créez des machines virtuelles qui peuvent jouer le rôle de serveurs principaux pour l’équilibreur de charge.

1. Sur le côté gauche de l’écran, cliquez sur **Créer une ressource** > **Calcul** > **Windows Server 2016 Datacenter** et entrez ces valeurs pour la machine virtuelle :
    - *myVM1* : pour le nom de la machine virtuelle.        
    - *azureuser* : pour le nom d’utilisateur administrateur.    
    - *myResourceGroupLBAZ* : pour **Groupe de ressources**, sélectionnez **Utiliser existant**, puis *myResourceGroupLBAZ*.
2. Cliquez sur **OK**.
3. Sélectionnez **DS1_V2** pour la taille de la machine virtuelle, puis cliquez sur **Sélectionner**.
4. Entrez ces valeurs pour les paramètres de la machine virtuelle :
    - *zone 1* : pour la zone dans laquelle vous placez la machine virtuelle.
    -  *myVNet* : vérifiez qu’il est sélectionné en tant que réseau virtuel.
    - *myBackendSubnet* : vérifiez qu’il est sélectionné en tant que sous-réseau.
    - *myNetworkSecurityGroup* : pour le nom du groupe de sécurité réseau (pare-feu).
5. Cliquez sur **Désactivé** pour désactiver les diagnostics de démarrage.
6. Cliquez sur **OK**, vérifiez les paramètres sur la page de résumé, puis cliquez sur **Créer**.
7. Créez une deuxième machine virtuelle nommée *VM2* dans Zone 2 et une troisième machine virtuelle dans Zone 3, avec *myVnet* en tant que réseau virtuel, *myBackendSubnet* en tant que sous-réseau, \**myNetworkSecurityGroup* en tant que groupe de sécurité réseau à l’aide des étapes 1 à 16.

### <a name="install-iis-on-vms"></a>Installer IIS sur des machines virtuelles

1. Cliquez sur **Toutes les ressources** dans le menu de gauche, puis dans la liste de ressources, cliquez sur **myVM1** qui se trouve dans le groupe de ressources *myResourceGroupLBAZ*.
2. Sur la page **Vue d’ensemble**, cliquez sur **Connexion** à RDP dans la machine virtuelle.
3. Connectez-vous à la machine virtuelle avec le nom d’utilisateur *azureuser*.
4. Sur le bureau du serveur, accédez à **Outils d’administration Windows**>**Windows PowerShell**.
5. Dans la fenêtre PowerShell, exécutez les commandes suivantes pour installer le serveur IIS, supprimez le fichier iisstart.htm par défaut, puis ajoutez un nouveau fichier iisstart.htm qui affiche le nom de la machine virtuelle :
   ```azurepowershell-interactive
    
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from" + $env:computername)
   ```
6. Fermez la session RDP avec *myVM1*.
7. Répétez les étapes 1 à 6 pour installer IIS et le fichier iisstart.htm mis à jour sur *myVM2* et *myVM3*.

## <a name="create-load-balancer-resources"></a>Créer les ressources d’équilibreur de charge

Dans cette section, vous allez configurer les paramètres de l’équilibreur de charge pour un pool d’adresses principal et une sonde d’intégrité, puis spécifier les règles de l’équilibreur de charge et NAT.


### <a name="create-a-backend-address-pool"></a>Créer un pool d’adresses principal

Pour distribuer le trafic vers les machines virtuelles, un pool d’adresses principal contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. Créez le pool d’adresses principal *myBackendPool* pour inclure *VM1*, *VM2* et *VM3*.

1. Cliquez sur **Toutes les ressources** dans le menu de gauche, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, sur **Pools principaux**, puis sur **Ajouter**.
3. Sur la page **Ajouter un pool principal**, procédez comme suit :
    - Tapez *myBackEndPool* comme nom du pool principal.
    - Pour **Réseau virtuel**, dans le menu déroulant, cliquez sur **myVNet**
    - Pour **Machine virtuelle**, dans le menu déroulant, cliquez sur **myVM1**.
    - Pour **Adresse IP**, dans le menu déroulant, cliquez sur l’adresse IP de myVM1.
4. Cliquez sur **Ajouter une nouvelle ressource principal** pour ajouter chaque machine virtuelle (*myVM2* et *myVM3*) au pool de principal de l’équilibreur de charge.
5. Cliquez sur **Add**.

    ![Ajout au pool d’adresses principal -](./media/load-balancer-standard-public-availability-zones-portal/add-backend-pool.png)

3. Vérifiez que le paramètre du pool principal de l’équilibreur de charge affiche les trois machines virtuelles : **myVM1**, **myVM2** et **myVM3**.

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Pour permettre à l’équilibrage de charge de surveiller l’état de votre application, vous utilisez une sonde d’intégrité. La sonde d’intégrité ajoute ou supprime dynamiquement des machines virtuelles de la rotation d’équilibrage de charge en fonction de leur réponse aux vérifications d’intégrité. Créez une sonde d’intégrité *myHealthProbe* pour surveiller l’intégrité des machines virtuelles.

1. Cliquez sur **Toutes les ressources** dans le menu de gauche, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, sur **Sondes d’intégrité**, puis sur **Ajouter**.
3. Utilisez ces valeurs pour créer la sonde d’intégrité :
    - *myHealthProbe* : pour le nom de la sonde d’intégrité.
    - **HTTP** : pour le type de protocole.
    - *80* : pour le numéro de port.
    - *15* : pour **l’intervalle** en secondes entre les tentatives de la sonde.
    - *2* : pour le nombre de **seuils de défaillance** ou d’échecs de sonde consécutifs qui se produisent avant qu’une machine virtuelle soit considérée comme défaillante.
4. Cliquez sur **OK**.

   ![Ajout d’une sonde](./media/load-balancer-standard-public-availability-zones-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Créez une règle d’équilibreur de charge *myLoadBalancerRuleWeb* pour écouter le port 80 dans le frontal *FrontendLoadBalancer* et envoyer le trafic réseau équilibré en charge vers le pool d’adresses principal *myBackEndPool* qui utilisé également le port 80. 

1. Cliquez sur **Toutes les ressources** dans le menu de gauche, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, **Règles d’équilibrage de charge**, puis sur **Ajouter**.
3. Utilisez ces valeurs pour configurer la règle d’équilibrage de charge :
    - *myHTTPRule* : pour que le nom de la règle d’équilibrage de charge.
    - **TCP** : pour le type de protocole.
    - *80* : pour le numéro de port.
    - *80* : pour le port principal.
    - *myBackendPool* : pour le nom du pool principal.
    - *myHealthProbe* : pour le nom de la sonde d’intégrité.
4. Cliquez sur **OK**.
    
    
    ![Ajout d’une règle d’équilibrage de charge](./media/load-balancer-standard-public-availability-zones-portal/load-balancing-rule.png)

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge
1. Recherchez l’adresse IP publique de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Cliquez sur **Toutes les ressources**, puis sur **myPublicIP**.

2. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. La page par défaut du serveur Web IIS s’affiche sur le navigateur.

      ![Serveur Web IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

Pour visualiser la distribution du trafic par l’équilibreur de charge entre les machines virtuelles des différentes zones, vous pouvez forcer l’actualisation de votre navigateur web.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, l’équilibreur de charge et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources qui contient l’équilibreur de charge et sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur l’équilibrage de la charge d’une machine virtuelle dans une zone de disponibilité spécifique.
> [!div class="nextstepaction"]
> [Équilibrer la charge de machines virtuelles dans une zone de disponibilité](tutorial-load-balancer-standard-public-zonal-portal.md)
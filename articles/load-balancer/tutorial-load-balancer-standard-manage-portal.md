---
title: 'Tutoriel : Équilibrer la charge du trafic Internet sur les machines virtuelles - Portail Azure'
titleSuffix: Azure Load Balancer
description: Ce didacticiel vous montre comment créer et gérer un Standard Load Balancer (équilibreur de charge standard) avec le portail Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: I want to create and Standard Load Balancer so that I can load balance internet traffic to VMs and add and remove VMs from the load-balanced set.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 03/11/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: 8961a50490bdbf8b456e87e1c00577c2c8afd050
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80240364"
---
# <a name="tutorial-load-balance-internet-traffic-to-vms-using-the-azure-portal"></a>Tutoriel : Équilibrer la charge du trafic Internet sur les machines virtuelles avec le portail Azure

L’équilibrage de charge offre un niveau plus élevé de disponibilité et d’évolutivité en répartissant les demandes entrantes sur plusieurs machines virtuelles. Dans ce tutoriel, vous découvrez les différents composants d’Azure Standard Load Balancer qui répartissent le trafic Internet entre les machines virtuelles et fournissent une haute disponibilité. Vous allez apprendre à effectuer les actions suivantes :


> [!div class="checklist"]
> * Créer un équilibreur de charge Azure
> * Créer des ressources d’équilibreur de charge
> * Créer des machines virtuelles et installer le serveur IIS
> * Afficher un équilibreur de charge en action
> * Ajouter et supprimer des machines virtuelles d’un équilibreur de charge

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

## <a name="create-a-standard-load-balancer"></a>Créer un équilibreur de charge standard

Dans cette section, vous allez créer un équilibreur de charge standard qui équilibre la charge de machines virtuelles. L’équilibreur de charge standard prend uniquement en charge une adresse IP publique standard. Lorsque vous créez un équilibreur de charge standard, vous devez également créer une adresse IP publique standard configurée en tant que frontale (nommée *LoadBalancerFrontend* par défaut) pour cet équilibreur de charge standard. 

1. En haut à gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.
2. Sous l’onglet **De base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Vérifier + créer** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer**, puis entrez *myResourceGroupSLB* dans la zone de texte.|
    | Nom                   | *myLoadBalancer*                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | Sélectionnez **Standard**.                          |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. |
    | Nom de l’adresse IP publique              | Tapez *myPublicIP* dans la zone de texte.   |
    |Zone de disponibilité| Sélectionnez **Redondant dans une zone**.    |

3. Sous l’onglet **Vérifier + créer**, cliquez sur **Créer**.

   ![Créer un équilibreur de charge standard](./media/quickstart-load-balancer-standard-public-portal/create-standard-load-balancer.png)

## <a name="create-load-balancer-resources"></a>Créer des ressources d’équilibreur de charge

Dans cette section, vous allez configurer des paramètres d’équilibreur de charge pour un pool d’adresses principal et une sonde d’intégrité, puis spécifier une règle d’équilibreur de charge.

### <a name="create-a-backend-address-pool"></a>Créer un pool d’adresses principal

Pour distribuer le trafic aux machines virtuelles, un pool d’adresses principal contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. Créez le pool d’adresses principal *myBackendPool* afin d’inclure des machines virtuelles pour l’équilibrage de charge du trafic Internet.

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, sur **Pools principaux**, puis sur **Ajouter**.
3. Dans la page **Ajouter un pool de backends**, entrez *myBackendPool* comme nom de votre pool principal, puis sélectionnez **Ajouter**.

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Pour permettre à l’équilibreur de charge de surveiller l’état de votre application, vous utilisez une sonde d’intégrité. La sonde d’intégrité ajoute ou supprime de façon dynamique des machines virtuelles dans la rotation de l’équilibreur de charge en fonction de leur réponse aux contrôles d’intégrité. Créez une sonde d’intégrité *myHealthProbe* pour surveiller l’intégrité des machines virtuelles.

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, sur **Sondes d’intégrité**, puis sur **Ajouter**.
3. Utilisez ces valeurs pour créer la sonde d’intégrité :
     
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez *MyHealthProbe*. |
    | Protocol | Sélectionnez **HTTP**. |
    | Port | Entrez *80*.|
    | Intervalle | Entrez *15* pour **l’intervalle** en secondes entre les tentatives de la sonde. |
    | Seuil de défaillance sur le plan de l’intégrité | Sélectionnez *2* pour le **Seuil de défaillance sur le plan de l’intégrité**, soit le nombre d’échecs de sonde consécutifs qui peuvent se produire avant qu’une machine virtuelle soit considérée comme non saine.|
    
4. Sélectionnez **OK**.

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibreur de charge sert à définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP frontale pour le trafic entrant et le pool d’adresses IP principal pour recevoir le trafic, ainsi que le port source et le port de destination requis. Créez une règle d’équilibreur de charge *myLoadBalancerRuleWeb* pour écouter le port 80 dans le *FrontendLoadBalancer* frontal et envoyer le trafic réseau équilibré en charge au pool d’adresses principal *myBackEndPool* qui utilise également le port 80.

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Cliquez sur **Paramètres**, **Règles d’équilibrage de charge**, puis sur **Ajouter**.
3. Pour configurer la règle d’équilibrage de charge, utilisez les valeurs suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez *MyHTTPRule*. |
    | Protocol | Sélectionnez **TCP**. |
    | Port | Entrez *80*.|
    | Port principal | Entrez *80*. |
    | Pool principal | Sélectionnez *MyBackendPool*.|
    | Sonde d’intégrité | Sélectionnez *myHealthProbe*. |
    
4. Laissez les autres valeurs par défaut et sélectionnez **OK**.

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez créer un réseau virtuel ainsi que trois machines virtuelles pour le pool principal de l’équilibreur de charge, puis installer IIS sur les machines virtuelles afin de tester l’équilibreur de charge.

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupSLB (sélectionnez un groupe de ressources existant) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Europe Ouest      |
| **\<IPv4-address-space>**   | 10.1.0.0/16          |
| **\<subnet-name>**          | mySubnet        |
| **\<subnet-address-range>** | 10.1.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Le service Standard Load Balancer ne prend en charge que des machines virtuelles auxquelles sont associées des adresses IP standard dans le pool principal. Dans cette section, vous allez créer trois machines virtuelles (*myVM1*, *myVM2* et *myVM3*) avec une adresse IP publique standard dans trois zones différentes (*Zone 1*, *Zone 2* et *Zone 3*) qui sont ajoutées au pool principal de l’équilibreur de charge standard créé précédemment.

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Windows Server 2016 Datacenter**. 
   
1. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base** :
   - **Abonnement** > **Groupe de ressources** : Sélectionnez **myResourceGroupSLB**.
   - **Détails de l’instance** > **Nom de la machine virtuelle** : Entrez *MyVM1*.
   - **Détails de l’instance** > **Région** > sélectionnez **Europe Ouest**.
   - **Détails de l’instance** > **Options de disponibilité** > sélectionnez **Zones de disponibilité**. 
   - **Détails de l’instance** > **Zones de disponibilité** > sélectionnez **1**.
  
1. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**. 
   
   - Vérifiez que les éléments suivants sont sélectionnés :
       - **Réseau virtuel** : **MyVNet**
       - **Sous-réseau** : **MyBackendSubnet**
       - **Adresse IP publique** > sélectionnez **Créer**, puis, dans la fenêtre **Créer une adresse IP publique**, pour **Référence (SKU)**, sélectionnez **Standard** et, pour **Zone de disponibilité**, sélectionnez **Redondant interzone**.
      
   - Pour créer un groupe de sécurité réseau (NSG, un type de pare-feu), sous **Groupe de sécurité réseau**, sélectionnez **Avancé**. 
       1. Dans le champ **Configurer le groupe de sécurité réseau**, sélectionnez **Créer**. 
       1. Tapez *myNetworkSecurityGroup*, puis sélectionnez **OK**.

   - Pour que la machine virtuelle fasse partie du pool principal de l’équilibreur de charge, procédez comme suit :
        - Dans **Équilibrage de charge**, pour **Placer cette machine virtuelle derrière une solution d’équilibrage de charge existante ?** , sélectionnez **Oui**.
        - Dans **Paramètres d’équilibrage de charge**, pour **Options d’équilibrage de charge**, sélectionnez **Équilibreur de charge Azure**.
        - Pour **Sélectionner un équilibreur de charge**, sélectionnez *MyLoadBalancer*. 
1. Sélectionnez l’onglet **Gestion** ou sélectionnez **Suivant** > **Gestion**. Sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**. 
1. Sélectionnez **Revoir + créer**.   
1. Passez en revue les paramètres, puis sélectionnez **Créer**.
1. Suivez les étapes pour créer deux machines virtuelles supplémentaires, *myVM2* et *myVM3*, avec une adresse IP publique de référence (SKU) standard dans **Zone de disponibilité** **2** et **3**, avec tous les autres paramètres identiques à ceux de *myVM1*.  

### <a name="create-network-security-group-rule"></a>Créer une règle de groupe de sécurité réseau

Dans cette section, vous créez une règle de groupe de sécurité réseau pour autoriser les connexions entrantes utilisant le protocole HTTP.

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis, dans la liste de ressources, cliquez sur **myNetworkSecurityGroup** dans le groupe de ressources **myResourceGroupSLB**.
2. Sous **Paramètres**, cliquez sur **Règles de sécurité entrantes**, puis sur **Ajouter**.
3. Entrez ces valeurs pour la règle de sécurité entrante nommée *myHTTPRule* afin d’autoriser les connexions HTTP entrantes à l’aide du port 80 :
    - *Service Tag* : pour **Source**.
    - *Internet* : pour **Balise de service source**
    - *80* : pour **Plages de port de destination**
    - *TCP* : pour **Protocole**
    - *Allow* : pour **Action**
    - *100* pour **Priorité**
    - *myHTTPRule* pour le nom
    - *Allow HTTP* pour la description
4. Sélectionnez **Ajouter**.

### <a name="install-iis-on-vms"></a>Installer IIS sur des machines virtuelles

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis, dans la liste de ressources, cliquez sur **myVM1** dans le groupe de ressources *myResourceGroupSLB*.
2. Sur la page **Vue d’ensemble**, cliquez sur **Connexion** à RDP dans la machine virtuelle.
3. Dans la fenêtre indépendante **Se connecter à la machine virtuelle**, sélectionnez **Télécharger un fichier RDP**, puis ouvrez le fichier RDP téléchargé.
4. Cliquez sur **Connecter** dans la fenêtre **Connexion Bureau à distance**.
5. Connectez-vous à la machine virtuelle avec les informations d’identification fournies lors de sa création. Cela lance une session Bureau à distance avec une machine virtuelle, *myVM1*.
6. Sur le bureau du serveur, accédez à **Outils d’administration Windows**>**Windows PowerShell**.
7. Dans la fenêtre PowerShell, exécutez les commandes suivantes pour installer le serveur IIS, supprimez le fichier iisstart.htm par défaut, puis ajoutez un nouveau fichier iisstart.htm qui affiche le nom de la machine virtuelle :

   ```azurepowershell-interactive
    
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
   ```
6. Fermez la session RDP avec *myVM1*.
7. Répétez les étapes 1 à 6 pour installer IIS et le fichier iisstart.htm mis à jour sur *myVM2* et *myVM3*.

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge
1. Recherchez l’adresse IP publique de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis cliquez sur **myPublicIP**.

2. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. La page par défaut du serveur Web IIS s’affiche sur le navigateur.

      ![Serveur Web IIS](./media/tutorial-load-balancer-standard-zonal-portal/load-balancer-test.png)

Pour visualiser la distribution de trafic par l’équilibreur de charge aux trois machines virtuelles exécutant votre application, vous pouvez forcer l’actualisation de votre navigateur web.

## <a name="remove-or-add-vms-from-the-backend-pool"></a>Supprimer ou ajouter des machines virtuelles à partir du pool principal
Vous devrez peut-être effectuer la maintenance sur la machine virtuelle exécutant votre application, avec par exemple l’installation des mises à jour du système d’exploitation. Pour faire face à une augmentation du trafic vers votre application, vous devrez peut-être ajouter des machines virtuelles supplémentaires. Cette section explique comment supprimer ou ajouter une machine virtuelle (*myVM1*) dans l’équilibreur de charge.

### <a name="remove-vm-from-a-backend-pool"></a>Supprimer un machine virtuelle d’un pool principal
Pour supprimer *myVM1* du pool principal, procédez comme suit :

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis cliquez sur **myLoadBalancer** dans la liste des ressources.
2. Sous **Paramètres**, cliquez sur **Pools principaux** et, dans la liste du pool principal, cliquez sur **myBackendPool**.
3. Dans la page **myBackendPool**, supprimez *VM1*, sélectionnez l’icône Supprimer à la fin de la ligne qui affiche *myVM1*, puis cliquez sur **Enregistrer**.

Lorsque *myVM1* ne figure plus dans le pool d’adresses principal, vous pouvez effectuer des tâches de maintenance sur *myVM1*, comme l’installation de mises à jour logicielles. En l’absence de *VM1*, la charge est maintenant équilibrée entre *myVM2* et *myVM3*. 

### <a name="add-vm-to-a-backend-pool"></a>Ajouter une machine virtuelle à un pool principal
Pour ré-ajouter *myVM1* au pool principal, procédez comme suit :

1. Dans le menu de gauche, sélectionnez **Tous les services**, **Toutes les ressources**, puis **myVM1** dans la liste des ressources.
2. Dans la page **VM1**, sous **Paramètres**, sélectionnez **Mise en réseau**.
3. Dans la page **Mise en réseau**, sélectionnez l’onglet **Équilibrage de charge**, puis **Ajouter l’équilibrage de charge**.
4. Dans la page **Ajouter l’équilibrage de charge**, procédez comme suit :
   1. Pour **Options d’équilibrage de charge**, sélectionnez **Azure Load Balancer**.
   2. Pour **Sélectionner un équilibreur de charge**, choisissez *MyLoadBalancer*.
   3. Pour **Sélectionner un pool principal**, choisissez *myBackendPool*. 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin du groupe de ressources, de l’équilibreur de charge et de toutes les ressources associées, supprimez-les. Pour ce faire, sélectionnez le groupe de ressources *myResouceGroupSLB* contenant l’équilibreur de charge, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé un équilibreur de charge standard, attaché des machines virtuelles à celui-ci, configuré la règle de trafic de l’équilibreur de charge et la sonde d’intégrité, puis testé l’équilibreur de charge. Vous avez également supprimé une machine virtuelle de l’ensemble à charge équilibrée et l’avez rajoutée au pool d’adresses principal. Pour en savoir plus sur Azure Load Balancer, consultez les didacticiels qui lui sont consacrés.

> [!div class="nextstepaction"]
> [Didacticiels Azure Load Balancer](tutorial-load-balancer-standard-public-zone-redundant-portal.md)

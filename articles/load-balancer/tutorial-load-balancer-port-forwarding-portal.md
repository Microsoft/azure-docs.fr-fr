---
title: 'Tutoriel : Configurer le réacheminement de port - Portail Azure'
titleSuffix: Azure Load Balancer
description: Ce didacticiel montre comment configurer la redirection de port à l’aide d’Azure Load Balancer pour créer des connexions à des machines virtuelles dans un réseau virtuel Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: twooley
Customer intent: As an IT administrator, I want to configure port forwarding in Azure Load Balancer to remotely connect to VMs in an Azure virtual network.
ms.service: load-balancer
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/26/2019
ms.author: allensu
ms.custom: seodec18
ms.openlocfilehash: 63d1a08dc588f0303ccb1ae13bd4c28af2a393c7
ms.sourcegitcommit: 2e72661f4853cd42bb4f0b2ded4271b22dc10a52
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92043651"
---
# <a name="tutorial-configure-port-forwarding-in-azure-load-balancer-using-the-portal"></a>Tutoriel : Configurer la redirection de port dans Azure Load Balancer à l’aide du portail

La redirection de port vous permet de vous connecter à des machines virtuelles dans un réseau virtuel Azure à l’aide de l’adresse IP publique et du numéro de port d’un Azure Load Balancer. 

Dans ce didacticiel, vous avez configuré la redirection de port sur un Azure Load Balancer. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Créer un équilibreur de charge standard public pour équilibrer le trafic réseau sur des machines virtuelles. 
> * Créer un réseau virtuel et des machines virtuelles avec une règle de groupe de sécurité réseau (NSG). 
> * Ajouter les machines virtuelles au pool d’adresses principales de l’équilibreur de charge.
> * Créer une sonde d’intégrité d’équilibreur de charge et des règles de trafic.
> * Créer des règles de redirection de port NAT de trafic entrant d’équilibreur de charge.
> * Installer et configurer IIS sur les machines virtuelles pour voir l’équilibrage de charge et la redirection de port en action.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer. 

Pour toutes les étapes de ce didacticiel, connectez-vous au portail Azure à l’adresse [https://portal.azure.com](https://portal.azure.com).

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure.

## <a name="create-a-standard-load-balancer"></a>Créer un équilibreur de charge standard

Commencez par créer un équilibreur de charge standard public qui permet d’équilibrer la charge du trafic sur les machines virtuelles. Un équilibreur de charge standard prend uniquement en charge une adresse IP publique standard. Lorsque vous créez un équilibreur de charge standard, vous créez également une adresse IP publique standard qui est configurée en tant qu’équilibreur de charge frontal et nommée **LoadBalancerFrontend** par défaut. 

1. En haut à gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.
2. Sous l’onglet **De base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Vérifier + créer** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer** et tapez *MyResourceGroupLB* dans la zone de texte.|
    | Nom                   | *myLoadBalancer*                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | Sélectionnez **Standard**.                          |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. |
    | Nom de l’adresse IP publique              | Tapez *myPublicIP* dans la zone de texte.   |
    |Zone de disponibilité| Sélectionnez **Redondant dans une zone**.    |
     
    >[!NOTE]
     >Veillez à créer votre équilibreur de charge et toutes ses ressources dans un emplacement prenant en charge les zones de disponibilité. Pour plus d’informations, consultez [Régions prenant en charge les zones de disponibilité](../availability-zones/az-region.md). 

3. Sous l’onglet **Vérifier + créer**, cliquez sur **Créer**.  
  
## <a name="create-and-configure-back-end-servers"></a>Créer et configurer des serveurs principaux

Créez un réseau virtuel avec deux machines virtuelles, puis ajoutez les machines virtuelles au pool principal de votre équilibreur de charge. 

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Dans les étapes de cette section, vous devrez remplacer les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB (sélectionnez un groupe de ressources existant) |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | Europe Ouest      |
| **\<IPv4-address-space>**   | 10.3.0.0\16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.3.0.0\24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

### <a name="create-vms-and-add-them-to-the-load-balancer-back-end-pool"></a>Créer des machines virtuelles et les ajouter au pool principal de l’équilibreur de charge

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Windows Server 2016 Datacenter**. 
   
1. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base** :
   - **Abonnement** > **Groupe de ressources** : Faites défiler la liste et sélectionnez **MyResourceGroupLB**.
   - **Nom de la machine virtuelle** : Tapez *MyVM1*.
   - **Région** : Sélectionnez **Europe Ouest**. 
   - **Nom d’utilisateur** : Tapez *azureuser*.
   - **Mot de passe** : Tapez *Azure1234567*. 
     Retapez le mot de passe dans le champ **Confirmer le mot de passe**.
   
1. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**. 
   
   Vérifiez que les éléments suivants sont sélectionnés :
   - **Réseau virtuel** : **MyVNet**
   - **Sous-réseau** : **MyBackendSubnet**
   
1. Sous **Adresse IP publique**, sélectionnez **Créer**, sélectionnez **Standard** dans la page **Créer une adresse IP publique**, puis sélectionnez **OK**. 
   
1. Sous **Groupe de sécurité réseau**, sélectionnez **Avancé** pour créer un groupe de sécurité réseau (NSG), un type de pare-feu. 
   1. Dans le champ **Configurer le groupe de sécurité réseau**, sélectionnez **Créer**. 
   1. Tapez *MyNetworkSecurityGroup*, puis sélectionnez **OK**. 
   
   >[!NOTE]
   >Observez que, par défaut, le NSG comprend déjà une règle de trafic entrant pour ouvrir le port 3389, le port (RDP) de Bureau à distance.
   
1. Ajoutez la machine virtuelle au pool principal d’un équilibreur de charge que vous créez :
   
   1. Sous **ÉQUILIBRAGE DE CHARGE** > **Placer cette machine virtuelle derrière une solution d’équilibrage de charge existante ?** , sélectionnez **Oui**. 
   1. Pour **Options d’équilibrage de charge**, faites défiler la liste déroulante et sélectionnez **Azure Load Balancer**. 
   1. Pour **Sélectionner un équilibreur de charge**, faites défiler la liste déroulante et sélectionnez **MyLoadBalancer**. 
   1. Sous **Sélectionner un pool principal**, sélectionnez **Créer**, puis tapez *MyBackendPool* et sélectionnez **Créer**. 
   
   ![Créer un réseau virtuel](./media/tutorial-load-balancer-port-forwarding-portal/create-vm-networking.png)
   
1. Sélectionnez l’onglet **Gestion** ou sélectionnez **Suivant** > **Gestion**. Sous **Supervision**, définissez **Diagnostics de démarrage** sur **Désactivé**.
   
1. Sélectionnez **Revoir + créer**.
   
1. Passez en revue les paramètres puis, lorsque la validation a réussi, sélectionnez **Créer**. 

1. Suivez la procédure de création d’une deuxième machine virtuelle nommée *MyVM2*, avec tous les autres paramètres identiques à ceux de MyVM1. 
   
   Pour **Groupe de sécurité réseau**, après avoir sélectionné **Avancé**, faites défiler la liste déroulante et sélectionnez le **MyNetworkSecurityGroup** que vous avez créé. 
   
   Sous **Sélectionnez un pool principal**, vérifiez que **MyBackendPool** est sélectionné. 

### <a name="create-an-nsg-rule-for-the-vms"></a>Créer une règle NSG pour les machines virtuelles

Créez une règle de groupe de sécurité réseau (NSG) pour les machines virtuelles afin d’autoriser les connexions Internet (HTTP) de trafic entrant.

>[!NOTE]
>Par défaut, le NSG comprend déjà une règle qui ouvre le port 3389, le port (RDP) de Bureau à distance.

1. Sélectionnez **Toutes les ressources** dans le menu de gauche. Dans la liste des ressources, sélectionnez **MyNetworkSecurityGroup** dans le groupe de ressources **MyResourceGroupLB**.
   
1. Sous **Paramètres**, sélectionnez **Règles de sécurité de trafic entrant**, puis sélectionnez **Ajouter**.
   
1. Dans la boîte de dialogue **Ajouter une règle de sécurité de trafic entrant**, tapez ou sélectionnez les informations suivantes :
   
   - **Source** : sélectionnez **Balise du service**.  
   - **Étiquette du service source** : sélectionnez **Internet**. 
   - **Plages de ports de destination** : tapez *80*.
   - **Protocole** : Sélectionnez **TCP**. 
   - **Action** : sélectionnez **Autoriser**.  
   - **Priorité** : tapez *100*. 
   - **Nom** : tapez *MyHTTPRule*. 
   - **Description** : tapez *Autoriser HTTP*. 
   
1. Sélectionnez **Ajouter**. 
   
   ![Créer une règle NSG](./media/tutorial-load-balancer-port-forwarding-portal/8-load-balancer-nsg-rules.png)
   
## <a name="create-load-balancer-resources"></a>Créer les ressources d’équilibreur de charge

Dans cette section, vous inspectez le pool principal de l’équilibreur de charge et configurez des règles de sonde d’intégrité et de trafic d’équilibreur de charge.

### <a name="view-the-back-end-address-pool"></a>Afficher le pool d’adresses principales

Pour distribuer le trafic vers les machines virtuelles, l’équilibreur de charge utilise un pool d’adresses principales qui contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. 

Vous avez créé votre pool principal d’équilibreur de charge et y avez ajouté des machines virtuelles lors de la création des machines virtuelles. Vous pouvez également créer des pools principaux et ajouter ou supprimer des machines virtuelles à partir de la page **Pools principaux** de l’équilibreur de charge. 

1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis sélectionnez **MyLoadBalancer** dans la liste de ressources.
   
1. Sous **Paramètres**, sélectionnez **Backend Pools (Pools principaux)** .
   
1. Dans la page **Pools back-ends**, développez **MyBackendPool**, puis vérifiez que **VM1** et **VM2** sont listées.

1. Sélectionnez **MyBackendPool**. 
   
   Dans la page **MyBackendPool**, sous **MACHINE VIRTUELLE** et **ADRESSE IP**, vous pouvez supprimer ou ajouter des machines virtuelles disponibles au pool.

Vous pouvez créer de nouveaux pools principaux en sélectionnant **Ajouter** dans la page **Pools principaux**.

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Pour permettre à l’équilibreur de charge de superviser l’état d’une machine virtuelle, vous utilisez une sonde d’intégrité. La sonde d’intégrité ajoute ou supprime dynamiquement des machines virtuelles de la rotation d’équilibrage de charge en fonction de leur réponse aux vérifications d’intégrité. 

1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis sélectionnez **MyLoadBalancer** dans la liste de ressources.
   
1. Sous **Paramètres**, sélectionnez **Health probes** (Sondes d’intégrité), puis **Ajouter**.
   
1. Dans la page **Ajouter une sonde d’intégrité**, tapez ou sélectionnez les valeurs suivantes :
   
   - **Name** : Entrez *MyHealthProbe*.
   - **Protocole** : Faites défiler et sélectionnez **HTTP**. 
   - **Port** : Entrez *80*. 
   - **Chemin d’accès** : Acceptez */* comme URI par défaut. Vous pouvez remplacer cette valeur avec n’importe quel autre URI. 
   - **Intervalle** : Entrez *15*. L’intervalle est le nombre de secondes entre les tentatives de la sonde.
   - **Seuil de défaillance sur le plan de l’intégrité** : Entrez *2*. Cette valeur est le nombre d’échecs de sonde consécutifs qui se produisent avant qu’une machine virtuelle soit considérée comme défaillante.
   
1. Sélectionnez **OK**.
   
   ![Ajouter une sonde](./media/tutorial-load-balancer-port-forwarding-portal/4-load-balancer-probes.png)

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibreur de charge définit la distribution du trafic vers les machines virtuelles. La règle définit la configuration IP front-end pour le trafic entrant, le pool d’adresses IP front-end pour recevoir le trafic ainsi que les ports source et de destination nécessaires. 

La règle d’équilibreur de charge nommée **MyLoadBalancerRule** écoute sur le port 80 dans le front-end **LoadBalancerFrontEnd**. La règle envoie le trafic réseau au pool d’adresses front-end **MyBackEndPool**, ainsi que sur le port 80. 

1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis sélectionnez **MyLoadBalancer** dans la liste de ressources.
   
1. Sous **Paramètres**, sélectionnez **Règles d'équilibrage de charge**, puis **Ajouter**.
   
1. Dans la page **Ajouter une règle d’équilibrage de charge**, tapez ou sélectionnez les valeurs suivantes :
   
   - **Name** : Entrez *MyLoadBalancerRule*.
   - **Protocole** : Sélectionnez **TCP**.
   - **Port** : Entrez *80*.
   - **Port principal** : Entrez *80*.
   - **Pool principal** : Sélectionnez **MyBackendPool**.
   - **Sonde d’intégrité** : Sélectionnez **MyHealthProbe**. 
   
1. Sélectionnez **OK**.
   
   ![Ajouter une règle d’équilibreur de charge](./media/tutorial-load-balancer-port-forwarding-portal/5-load-balancing-rules.png)

## <a name="create-an-inbound-nat-port-forwarding-rule"></a>Créer une règle de redirection de port NAT de trafic entrant

Créez une règle de traduction d’adresses réseau (NAT) de trafic entrant d’équilibreur de charge pour rediriger le trafic d’un port spécifique de l’adresse IP frontale vers un port spécifique d’une machine virtuelle principale.

1. Sélectionnez **Toutes les ressources** dans le menu de gauche, puis sélectionnez **MyLoadBalancer** dans la liste des ressources.
   
1. Sous **Paramètres**, sélectionnez **Règles NAT de trafic entrant**, puis sélectionnez **Ajouter**. 
   
1. Dans la page **Ajouter une règle NAT de trafic entrant**, tapez ou sélectionnez les valeurs suivantes :
   
   - **Name** : Tapez *MyNATRuleVM1*.
   - **Port** : Tapez *4221*.
   - **Machine virtuelle cible** : Sélectionnez **MyVM1** dans la liste déroulante.
   - **Configuration IP réseau** : Sélectionnez **ipconfig1** dans la liste déroulante.
   - **Mappage de port** : Sélectionnez **Personnalisé**.
   - **Port cible** : Entrez *3389*.
   
1. Sélectionnez **OK**.
   
1. Répétez les étapes pour ajouter une règle NAT de trafic entrant nommée *MyNATRuleVM2* en utilisant **Port** : *4222* et **Machine virtuelle cible** : **MyVM2**.

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge

Dans cette section, vous allez installer Internet Information Services (IIS) sur les serveurs principaux et personnaliser la page web par défaut pour afficher le nom de la machine. Vous utiliserez ensuite l’adresse IP publique de l’équilibreur de charge pour tester l’équilibreur de charge. 

Chaque machine virtuelle back-end affiche une version différente de la page web IIS par défaut. Ainsi, vous pouvez voir l’équilibreur de charge répartir les requêtes entre les deux machines virtuelles.

### <a name="connect-to-the-vms-with-rdp"></a>Se connecter aux machines virtuelles avec le Bureau à distance

Connectez-vous à chaque machine virtuelle avec le Bureau à distance (RDP). 

1. Dans le portail, sélectionnez **Toutes les ressources** dans le menu de gauche. Dans la liste des ressources, sélectionnez chacune des machines virtuelles du groupe de ressources **MyResourceGroupLB**.
   
1. Dans la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Télécharger le fichier RDP**. 
   
1. Ouvrez le fichier RDP que vous avez téléchargé, puis sélectionnez **Se connecter**.
   
1. Dans l’écran Sécurité Windows, sélectionnez **Plus de choix**, puis **Utiliser un autre compte**. 
   
   Entrez le nom d’utilisateur *azureuser* et le mot de passe *Azure1234567*, puis sélectionnez **OK**.
   
1. Répondez **Oui** à toute invite de certificat. 
   
   Le bureau de la machine virtuelle s’ouvre dans une nouvelle fenêtre. 

### <a name="install-iis-and-replace-the-default-iis-web-page"></a>Installer IIS et remplacer la page web IIS par défaut 

Utilisez PowerShell pour installer IIS et remplacer la page web IIS par défaut par une page qui affiche le nom de la machine virtuelle.

1. Sur MyVM1 et MyVM2, lancez **Windows PowerShell** à partir du menu **Démarrer**. 

2. Exécutez les commandes suivantes pour installer IIS et remplacer la page web IIS par défaut :
   
   ```powershell-interactive
    # Install IIS
      Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # Remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    #Add custom htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
    
   ```
   
1. Fermez les connexions RDP à MyVM1 et MyVM2 en sélectionnant **Déconnexion**. N’arrêtez pas les machines virtuelles.

### <a name="test-load-balancing"></a>Tester l’équilibrage de charge

1. Dans le portail, dans la page **Vue d’ensemble** de **MyLoadBalancer**, copiez l’adresse IP publique sous **Adresse IP publique**. Pointez sur l’adresse, puis sélectionnez l’icône **Copier** pour la copier. Dans cet exemple, il s’agit de **40.67.218.235**. 
   
1. Collez ou tapez l’adresse IP publique de l’équilibreur de charge (*40.67.218.235*) dans la barre d’adresses de votre navigateur Internet. 
   
   La page par défaut personnalisée du serveur web IIS s’affiche dans le navigateur. Le message est soit **Hello World from MyVM1**, soit **Hello World from MyVM2**.
   
   ![Nouvelle page IIS par défaut](./media/tutorial-load-balancer-port-forwarding-portal/9-load-balancer-test.png) 
   
1. Actualisez le navigateur pour voir l’équilibreur de charge répartir le trafic entre les machines virtuelles. Parfois, la page **MyVM1** s’affiche, et parfois la page **MyVM2** s’affiche, selon la répartition des requêtes entre les machines virtuelles back-end.
   
   >[!NOTE]
   >Vous devrez peut-être effacer le cache de votre navigateur ou ouvrir une nouvelle fenêtre de navigateur entre les tentatives.

## <a name="test-port-forwarding"></a>Tester la redirection de port

Avec la redirection de port, vous pouvez appliquer le Bureau à distance à une machine virtuelle principale à l’aide de l’adresse IP de l’équilibreur de charge et de la valeur de port frontal définie dans la règle NAT. 

1. Dans le portail, dans la page **Vue d’ensemble** de **MyLoadBalancer**, copiez son adresse IP publique. Pointez sur l’adresse, puis sélectionnez l’icône **Copier** pour la copier. Dans cet exemple, il s’agit de **40.67.218.235**. 
   
1. Ouvrez une invite de commandes et utilisez la commande suivante pour créer une session Bureau à distance avec MyVM2, en utilisant l’adresse IP publique de l’équilibreur de charge et le port frontal que vous avez défini dans la règle NAT de la machine virtuelle. 
   
   ```
   mstsc /v:40.67.218.235:4222
   ```
  
1. Ouvrez le fichier RDP téléchargé et sélectionnez **Se connecter**.
   
1. Dans l’écran Sécurité Windows, sélectionnez **Plus de choix**, puis **Utiliser un autre compte**. 
   
   Entrez le nom d’utilisateur *azureuser* et le mot de passe *Azure1234567*, puis sélectionnez **OK**.
   
1. Répondez **Oui** à toute invite de certificat. 
   
   Le bureau de MyVM2 s’ouvre dans une nouvelle fenêtre. 

La connexion RDP réussit, car la règle NAT de trafic entrant **MyNATRuleVM2** dirige le trafic du port frontal 4222 de l’équilibreur de charge vers le port 3389 de MyVM2 (port RDP).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer l’équilibreur de charge et toutes les ressources associées quand vous n’en avez plus besoin, ouvrez le groupe de ressources **MyResourceGroupLB**, puis sélectionnez **Supprimer un groupe de ressources**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé un équilibreur de charge public standard. Vous avez créé et configuré des ressources réseau, des serveurs back-end, une sonde d’intégrité et des règles à utiliser avec l’équilibreur de charge. Vous avez installé IIS sur les machines virtuelles principales et utilisé l’adresse IP publique de l’équilibreur de charge pour tester l’équilibreur de charge. Vous avez configuré et testé la redirection de port d’un port spécifié sur l’équilibreur de charge vers un port sur une machine virtuelle principale. 

Pour en savoir plus sur Azure Load Balancer, consultez d’autres didacticiels consacrés à l’équilibreur de charge.

> [!div class="nextstepaction"]
> [Didacticiels Azure Load Balancer](tutorial-load-balancer-standard-public-zone-redundant-portal.md)

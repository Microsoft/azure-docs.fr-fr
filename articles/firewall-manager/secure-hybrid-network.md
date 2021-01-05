---
title: 'Tutoriel : Sécuriser votre réseau virtuel hub à l’aide d’Azure Firewall Manager'
description: Dans ce tutoriel, vous allez apprendre à sécuriser votre réseau virtuel avec Azure Firewall Manager à l’aide du portail Azure.
services: firewall-manager
author: vhorne
ms.service: firewall-manager
ms.topic: tutorial
ms.date: 06/30/2020
ms.author: victorh
ms.openlocfilehash: 3d4d1e65c2200aee178abefb46d3e330acbd3108
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023684"
---
# <a name="tutorial-secure-your-hub-virtual-network-using-azure-firewall-manager"></a>Tutoriel : Sécuriser votre réseau virtuel hub à l’aide d’Azure Firewall Manager

Lorsque vous connectez votre réseau local à un réseau virtuel Azure pour créer un réseau hybride, la possibilité de contrôler l’accès à vos ressources réseau Azure représente une part importante dans un plan de sécurité générale.

À l’aide d’Azure Firewall Manager, vous pouvez créer un réseau virtuel hub pour sécuriser le trafic de votre réseau hybride à destination d’adresses IP privées, d’Azure PaaS et d’Internet. Vous pouvez utiliser Azure Firewall Manager pour contrôler l’accès réseau d’un réseau hybride à l’aide de stratégies définissant le trafic réseau autorisé et refusé.

Firewall Manager prend également en charge une architecture de hub virtuel sécurisé. Pour obtenir une comparaison des types d’architectures de hub virtuel sécurisé et de réseau virtuel hub, consultez [Quelles sont les options d’architecture d’Azure Firewall Manager ?](vhubs-and-vnets.md)

Pour ce tutoriel, vous créez trois réseaux virtuels :

- **VNet-Hub** : Le pare-feu se trouve dans ce réseau virtuel.
- **VNet-Spoke** : Le réseau virtuel spoke correspond à la charge de travail sur Azure.
- **VNet-Onprem** : Le réseau virtuel local représente un réseau local. Dans un déploiement réel, il peut être connecté via un VPN ou une connexion ExpressRoute. Par souci de simplicité, ce tutoriel utilise une connexion de passerelle VPN, sachant qu’un réseau virtuel situé sur Azure est utilisé pour représenter un réseau local.

![Réseau hybride](media/tutorial-hybrid-portal/hybrid-network-firewall.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une stratégie de pare-feu
> * Créer les réseaux virtuels
> * Configurer et déployer le pare-feu
> * Créer et connecter les passerelles VPN
> * Appairer les réseaux virtuels hub et spoke
> * Créer les itinéraires
> * Créer les machines virtuelles
> * Tester le pare-feu


## <a name="prerequisites"></a>Prérequis

Un réseau hybride utilise le modèle d’architecture Hub and Spoke pour router le trafic entre des réseaux virtuels Azure et des réseaux locaux. L’architecture Hub and Spoke présente les conditions suivantes :

- Vous devez définir **AllowGatewayTransit** lors du Peering de VNet-Hub à VNet-Spoke. Dans une architecture réseau Hub and Spoke, le transit par passerelle permet aux réseaux virtuels spoke d’exploiter la passerelle VPN du hub, évitant ainsi de devoir déployer des passerelles VPN dans chaque réseau virtuel spoke. 

   De plus, les routes vers les réseaux locaux ou les réseaux virtuels connectés à la passerelle sont propagés automatiquement aux tables de routage pour les réseaux virtuels homologués à l’aide du transit par passerelle. Pour plus d’informations, consultez [Configurer le transit par passerelle VPN pour le peering de réseaux virtuels](../vpn-gateway/vpn-gateway-peering-gateway-transit.md).

- Vous devez définir **UseRemoteGateways** lors du Peering de VNet-Spoke à VNet-Hub. Si **UseRemoteGateways avec** est défini et que **AllowGatewayTransit** sur le Peering distant est également défini, le réseau virtuel spoke utilise les passerelles du réseau virtuel distant pour le transit.
- Pour router le trafic de sous-réseau spoke par le biais du pare-feu de hub, vous avez besoin d’une route définie par l’utilisateur (UDR, User-Defined Route) qui pointe vers le pare-feu avec le paramètre **Propagation de la route de la passerelle de réseau virtuel** activé. Cette option empêche la distribution des routes vers les sous-réseaux spoke. Cela empêche que les routes apprises entrent en conflit avec votre UDR.
- Vous devez configurer une UDR sur le sous-réseau de passerelle hub qui pointe vers l’adresse IP du pare-feu comme prochain tronçon vers les réseaux spoke. Aucun UDR n’est requis sur le sous-réseau du Pare-feu Azure, puisqu’il apprend les itinéraires à partir de BGP.

Consultez la section [Créer des itinéraires](#create-the-routes) de ce didacticiel pour voir comment ces itinéraires sont créés.

>[!NOTE]
>Le Pare-feu Azure doit avoir une connectivité Internet directe. Si votre AzureFirewallSubnet prend connaissance d’un itinéraire par défaut pour votre réseau local via le protocole BGP, vous devez le remplacer par un UDR 0.0.0.0/0 avec la valeur **NextHopType** définie sur **Internet** pour garantir une connectivité Internet directe.
>
>Le Pare-feu Azure peut être configuré pour prendre en charge le tunneling forcé. Pour plus d’informations, consultez la page [Tunneling forcé du Pare-feu Azure](../firewall/forced-tunneling.md).

>[!NOTE]
>Le trafic entre les réseaux virtuels directement appairés est acheminé directement même si l’UDR pointe vers le Pare-feu Azure en tant que passerelle par défaut. Pour envoyer un trafic de sous-réseau à sous-réseau au pare-feu dans ce scénario, un UDR doit contenir explicitement le préfixe du réseau cible dans les deux sous-réseaux.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-a-firewall-policy"></a>Créer une stratégie de pare-feu

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la barre de recherche du portail Azure, tapez **Firewall Manager**, puis appuyez sur **Entrée**.
3. Dans la page Azure Firewall Manager, sélectionnez **Afficher les stratégies de pare-feu Azure**.

   ![Stratégie de pare-feu](media/tutorial-hybrid-portal/firewall-manager-policy.png)

1. Sélectionnez **Créer une stratégie de pare-feu Azure**.
1. Sélectionnez votre abonnement et, comme Groupe de ressources, sélectionnez **Créer** et créez un groupe de ressources nommé **FW-Hybrid-Test**.
2. Comme nom de la stratégie, tapez **Pol-Net01**.
3. Comme Région, sélectionnez **USA Est**.
4. Sélectionnez **Suivant : Règles**.
5. Sélectionnez **Ajouter une collection de règles**.
6. Dans le champ **Nom**, tapez **RCNet01**.
7. Comme **Type de collection de règles**, sélectionnez **Réseau**.
8. Pour **Priorité**, tapez **100**.
9. Pour **Action**, sélectionnez **Autoriser**.
10. Sous **Règles**, pour **Nom**, tapez **AllowWeb**.
11. Pour **Adresses sources**, tapez **192.168.1.0/24**.
12. Pour **Protocole**, sélectionnez **TCP**.
13. Pour **Ports de destination**, tapez **80**.
14. Pour **Type de destination**, sélectionnez **Adresse IP**.
15. Pour **Destination**, tapez **10.6.0.0/16**.
16. Sur la ligne suivante de la règle, entrez les informations suivantes :
 
    Nom : tapez **AllowRDP**.<br>
    Adresse IP source : tapez **192.168.1.0/24**.<br>
    Protocole : sélectionnez **TCP**.<br>
    Ports de destination : tapez **3389**.<br>
    Type de destination : sélectionnez **Adresse IP**<br>
    Pour Destination, tapez **10.6.0.0/16**.

1. Sélectionnez **Ajouter**.
2. Sélectionnez **Vérifier + créer**.
3. Passez en revue les détails, puis sélectionnez **Créer**.

## <a name="create-the-firewall-hub-virtual-network"></a>Créer le réseau virtuel du hub de pare-feu

> [!NOTE]
> La taille du sous-réseau AzureFirewallSubnet est /26. Pour plus d’informations sur la taille du sous-réseau, consultez le [FAQ Pare-feu Azure](../firewall/firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Mise en réseau**, sélectionnez **Réseau virtuel**.
4. Dans le champ **Nom**, tapez **VNet-hub**.
5. Pour **Espace d’adressage**, tapez **10.5.0.0/16**.
6. Pour **Abonnement**, sélectionnez votre abonnement.
7. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
8. Pour **Emplacement**, sélectionnez **USA Est**.
9. Sous **Sous-réseau**, pour **Nom**, entrez **AzureFirewallSubnet**. Le pare-feu se trouvera dans ce sous-réseau et le nom du sous-réseau **doit** être AzureFirewallSubnet.
10. Pour **Plage d’adresses**, tapez **10.5.0.0/26**. 
11. Acceptez les autres paramètres par défaut, puis sélectionnez **Créer**.

## <a name="create-the-spoke-virtual-network"></a>Créer le réseau virtuel spoke

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Mise en réseau**, sélectionnez **Réseau virtuel**.
4. Pour **Nom**, tapez **VNet-Spoke**.
5. Pour **Espace d’adressage**, tapez **10.6.0.0/16**.
6. Pour **Abonnement**, sélectionnez votre abonnement.
7. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
8. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
9. Sous **Sous-réseau**, tapez **SN-Workload** pour **Nom**.
10. Pour **Plage d’adresses**, tapez **10.6.0.0/24**.
11. Acceptez les autres paramètres par défaut, puis sélectionnez **Créer**.

## <a name="create-the-on-premises-virtual-network"></a>Créer le réseau virtuel local

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Mise en réseau**, sélectionnez **Réseau virtuel**.
4. Pour **Nom**, tapez **VNet-OnPrem**.
5. Pour **Espace d’adressage**, entrez **192.168.0.0/16**.
6. Pour **Abonnement**, sélectionnez votre abonnement.
7. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
8. Pour **Emplacement**, sélectionnez le même emplacement que celui utilisé précédemment.
9. Sous **Sous-réseau**, tapez **SN-Corp** pour **Nom**.
10. Pour **Plage d’adresses**, entrez **192.168.1.0/24**.
11. Acceptez les autres paramètres par défaut, puis sélectionnez **Créer**.

Une fois le réseau virtuel déployé, créez un deuxième sous-réseau pour la passerelle.

1. Sur la page **VNet-Onprem**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **+Sous-réseau**.
3. Pour **Nom**, tapez **GatewaySubnet**.
4. Pour **Plage d’adresses (bloc CIDR)** , tapez **192.168.2.0/24**.
5. Sélectionnez **OK**.

### <a name="create-a-public-ip-address"></a>Créer une adresse IP publique

Il s’agit de l’adresse IP publique utilisée pour la passerelle locale.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **adresse IP publique** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Adresse IP publique**, puis **Créer**.
4. Pour le nom, tapez **VNet-Onprem-GW-pip**.
5. Pour le groupe de ressources, tapez **FW-Hybrid-Test**.
6. Pour **Emplacement**, sélectionnez **USA Est**.
7. Acceptez les autres valeurs par défaut, puis sélectionnez **Créer**.

## <a name="configure-and-deploy-the-firewall"></a>Configurer et déployer le pare-feu

Quand les stratégies de sécurité sont associées à un hub, on utilise le terme de *réseau virtuel hub*.

Convertissez le réseau virtuel **VNet-Hub** en *réseau virtuel hub* et sécurisez-le avec le Pare-feu Azure.

1. Dans la barre de recherche du portail Azure, tapez **Firewall Manager**, puis appuyez sur **Entrée**.
3. Dans la page Azure Firewall Manager, sous **Ajouter la sécurité aux réseaux virtuels**, sélectionnez **Afficher les réseaux virtuels hub**.
4. Sélectionnez **Convertir des réseaux virtuels**.
5. Sélectionnez **VNet-hub**, puis **Suivant : Pare-feu Azure**.
6. Comme **Stratégie de pare-feu**, sélectionnez **Pol-Net01**.
7. Sélectionnez **Suivant " Vérifier + confirmer**
8. Passez en revue les détails, puis sélectionnez **Confirmer**.


   Le déploiement prend quelques minutes.
7. Une fois le déploiement terminé, accédez au groupe de ressources **FW-Hybrid-Test**, puis sélectionnez le pare-feu.
9. Notez l’adresse **IP privée du pare-feu** dans la page **Vue d’ensemble**. Vous l’utiliserez plus tard lors de la création de l’itinéraire par défaut.

## <a name="create-and-connect-the-vpn-gateways"></a>Créer et connecter les passerelles VPN

Les réseaux virtuels hub et local sont connectés via des passerelles VPN.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Créer une passerelle VPN pour le réseau virtuel hub

Maintenant, créez la passerelle VPN pour le réseau virtuel hub. Les configurations de réseau virtuel à réseau virtuel nécessitent un VPN de type RouteBased. La création d’une passerelle VPN nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle VPN sélectionnée.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **passerelle de réseau virtuel** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Passerelle de réseau virtuel**, puis sélectionnez **Créer**.
4. Dans le champ **Nom**, tapez **GW-hub**.
5. Pour **Région**, sélectionnez **(États-Unis) USA Est**.
6. Pour **Type de passerelle**, sélectionnez **VPN**.
7. Pour **Type de VPN**, sélectionnez **Basé sur itinéraires**.
8. Pour **Référence (SKU)** , sélectionnez **De base**.
9. Pour **Réseau virtuel**, sélectionnez **VNet-hub**.
10. Pour **Adresse IP publique**, sélectionnez **Créer nouveau**, puis tapez le nom **VNet-hub-GW-pip**.
11. Acceptez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer**.
12. Vérifiez la configuration, puis sélectionnez **Créer**.

### <a name="create-a-vpn-gateway-for-the-on-premises-virtual-network"></a>Créer une passerelle VPN pour le réseau virtuel local

À présent, créez la passerelle VPN pour le réseau virtuel local. Les configurations de réseau virtuel à réseau virtuel nécessitent un VPN de type RouteBased. La création d’une passerelle VPN nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle VPN sélectionnée.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **passerelle de réseau virtuel** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Passerelle de réseau virtuel**, puis sélectionnez **Créer**.
4. Pour **Nom**, tapez **GW-Onprem**.
5. Pour **Région**, sélectionnez **(États-Unis) USA Est**.
6. Pour **Type de passerelle**, sélectionnez **VPN**.
7. Pour **Type de VPN**, sélectionnez **Basé sur itinéraires**.
8. Pour **Référence (SKU)** , sélectionnez **De base**.
9. Pour **Réseau virtuel**, sélectionnez **VNet-Onprem**.
10. Pour **Adresse IP publique**, sélectionnez **Utiliser existant*, puis sélectionnez **VNet-Onprem-GW-pip** comme nom.
11. Acceptez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer**.
12. Vérifiez la configuration, puis sélectionnez **Créer**.

### <a name="create-the-vpn-connections"></a>Créer les connexions VPN

Vous pouvez maintenant créer les connexions VPN entre les passerelles hub et locale.

Dans cette étape, vous créez la connexion entre le réseau virtuel hub et le réseau virtuel local. Une clé partagée est référencée dans les exemples. Vous pouvez utiliser vos propres valeurs pour cette clé partagée. Il est important que la clé partagée corresponde aux deux connexions. La création d’une connexion peut prendre quelques instants.

1. Ouvrez le groupe de ressources **FW-Hybrid-Test** et sélectionnez la passerelle **GW-hub**.
2. Sélectionnez **Connexions** dans la colonne de gauche.
3. Sélectionnez **Ajouter**.
4. Pour le nom de la connexion, tapez **Hub-to-Onprem**.
5. Pour **Type de connexion**, sélectionnez **Réseau virtuel à réseau virtuel**.
6. Pour **Passerelle du deuxième réseau virtuel**, sélectionnez **GW-Onprem**.
7. Pour **Clé partagée (PSK)** , tapez **AzureA1b2C3**.
8. Sélectionnez **OK**.

Créez la connexion entre les réseaux virtuels hub et local. Cette étape est similaire à la précédente, sauf que vous créez la connexion du réseau virtuel OnPrem vers le réseau virtuel hub. Vérifiez que les clés partagées correspondent. Après quelques minutes, la connexion est établie.

1. Ouvrez le groupe de ressources **FW-Hybrid-Test** et sélectionnez la passerelle **GW-Onprem**.
2. Sélectionnez **Connexions** dans la colonne de gauche.
3. Sélectionnez **Ajouter**.
4. Pour le nom de la connexion, tapez **Onprem-to-Hub**.
5. Pour **Type de connexion**, sélectionnez **Réseau virtuel à réseau virtuel**.
6. Pour **Passerelle du deuxième réseau virtuel**, sélectionnez **GW-hub**.
7. Pour **Clé partagée (PSK)** , tapez **AzureA1b2C3**.
8. Sélectionnez **OK**.


#### <a name="verify-the-connection"></a>Vérifier la connexion

Après environ cinq minutes, l’état des deux connexions doit être **Connecté**.

![Connexions de passerelle](media/secure-hybrid-network/gateway-connections.png)

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Appairer les réseaux virtuels hub et spoke

À présent, appairez les réseaux virtuels hub et spoke.

1. Ouvrez le groupe de ressources **FW-Hybrid-Test** et sélectionnez le réseau virtuel **VNet-hub**.
2. Dans la colonne de gauche, sélectionnez **Peerings**.
3. Sélectionnez **Ajouter**.
4. Pour **Nom**, tapez **HubtoSpoke**.
5. Pour **Réseau virtuel**, sélectionnez **VNet-spoke**.
6. Pour le nom du peering de VNetSpoke à VNet-hub, tapez **SpoketoHub**.
7. Sélectionnez **Autoriser le transit par passerelle**.
8. Sélectionnez **OK**.

### <a name="configure-additional-settings-for-the-spoketohub-peering"></a>Configurer des paramètres supplémentaires pour le peering SpoketoHub

Vous devez activer l’option **Autoriser le trafic transféré** sur le peering SpoketoHub.

1. Ouvrez le groupe de ressources **FW-Hybrid-Test** et sélectionnez le réseau virtuel **VNet-Spoke**.
2. Dans la colonne de gauche, sélectionnez **Peerings**.
3. Sélectionnez le peering **SpoketoHub**.
4. Sous **Autoriser le trafic transféré de VNet-hub à VNet-Spoke**, sélectionnez **Activé**.
5. Sélectionnez **Enregistrer**.

## <a name="create-the-routes"></a>Créer les itinéraires

Ensuite, créez deux itinéraires :

- Un itinéraire à partir du sous-réseau de passerelle hub vers le sous-réseau spoke via l’adresse IP du pare-feu
- Un itinéraire par défaut à partir du sous-réseau spoke via l’adresse IP du pare-feu

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **table de routage** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Table de routage**.
4. Sélectionnez **Create** (Créer).
5. Pour le nom, tapez **UDR-Hub-Spoke**.
6. Sélectionnez le groupe de ressources **FW-Hybrid-Test**.
8. Pour **Emplacement**, sélectionnez **(États-Unis) USA Est**.
9. Sélectionnez **Create** (Créer).
10. Une fois la table de routage créée, sélectionnez-la pour ouvrir la page correspondante.
11. Sélectionnez **Routes** dans la colonne de gauche.
12. Sélectionnez **Ajouter**.
13. Pour le nom de la route, tapez **ToSpoke**.
14. Pour le préfixe d’adresse, tapez **10.6.0.0/16**.
15. Pour le type de tronçon suivant, sélectionnez **Appliance virtuelle**.
16. Pour l’adresse du tronçon suivant, tapez l’adresse IP privée du pare-feu que vous avez notée précédemment.
17. Sélectionnez **OK**.

À présent, associez la route au sous-réseau.

1. Sur la page **UDR-Hub-Spoke - Routes**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **Associer**.
4. Sous **Réseau virtuel**, sélectionnez **VNet-hub**.
5. Sous **Sous-réseau**, sélectionnez **GatewaySubnet**.
6. Sélectionnez **OK**.

À présent, créez la route par défaut à partir du sous-réseau spoke.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **table de routage** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Table de routage**.
5. Sélectionnez **Create** (Créer).
6. Pour le nom, tapez **UDR-DG**.
7. Sélectionnez le groupe de ressources **FW-Hybrid-Test**.
8. Pour **Emplacement**, sélectionnez **(États-Unis) USA Est**.
4. Pour **Propagation de la route de la passerelle de réseau virtuel**, sélectionnez **Désactivée**.
1. Sélectionnez **Create** (Créer).
2. Une fois la table de routage créée, sélectionnez-la pour ouvrir la page correspondante.
3. Sélectionnez **Routes** dans la colonne de gauche.
4. Sélectionnez **Ajouter**.
5. Pour le nom de la route, tapez **ToHub**.
6. Pour le préfixe d’adresse, tapez **0.0.0.0/0**.
7. Pour le type de tronçon suivant, sélectionnez **Appliance virtuelle**.
8. Pour l’adresse du tronçon suivant, tapez l’adresse IP privée du pare-feu que vous avez notée précédemment.
9. Sélectionnez **OK**.

À présent, associez la route au sous-réseau.

1. Sur la page **UDR-DG - Routes**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **Associer**.
4. Sous **Réseau virtuel**, sélectionnez **VNet-spoke**.
5. Sous **Sous-réseau**, sélectionnez **SN-Workload**.
6. Sélectionnez **OK**.

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Maintenant, créez les machines virtuelles de charge de travail spoke et locale, et placez-les dans les sous-réseaux appropriés.

### <a name="create-the-workload-virtual-machine"></a>Créer la machine virtuelle de charge de travail

Créez une machine virtuelle dans le réseau virtuel spoke, exécutant IIS, sans adresse IP publique.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Populaire**, sélectionnez **Windows Server 2016 Datacenter**.
3. Entrez ces valeurs pour la machine virtuelle :
    - **Groupe de ressources** : sélectionnez **FW-Hybrid-Test**.
    - **Nom de la machine virtuelle** : *VM-Spoke-01*.
    - **Région** -  *(États-Unis) USA Est)* .
    - **Nom d’utilisateur** : *azureuser*.
    - **Mot de passe** : tapez votre mot de passe.

4. Sélectionnez **Suivant : Disques**.
5. Acceptez les valeurs par défaut, puis sélectionnez **Suivant : Mise en réseau**.
6. Sélectionnez **VNet-Spoke** pour le réseau virtuel et **SN-Workload** pour le sous-réseau.
7. Pour **Adresse IP publique**, sélectionnez **Aucune**.
8. Pour **Ports d’entrée publics**, sélectionnez **Autoriser les ports sélectionnés**, puis sélectionnez **HTTP (80)** et **RDP (3389)** .
9. Sélectionnez **Suivant : Gestion**.
10. Pour **Diagnostics de démarrage**, sélectionnez **Désactivé**.
11. Sélectionnez **Vérifier + Créer**, vérifiez les paramètres sur la page de résumé, puis sélectionnez **Créer**.

### <a name="install-iis"></a>Installer IIS

1. Dans le portail Azure, ouvrez Cloud Shell et assurez-vous qu’il est défini sur **PowerShell**.
2. Exécutez la commande suivante pour installer IIS sur la machine virtuelle et modifier l’emplacement si nécessaire :

   ```azurepowershell-interactive
   Set-AzVMExtension `
           -ResourceGroupName FW-Hybrid-Test `
           -ExtensionName IIS `
           -VMName VM-Spoke-01 `
           -Publisher Microsoft.Compute `
           -ExtensionType CustomScriptExtension `
           -TypeHandlerVersion 1.4 `
           -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server; powershell      Add-Content -Path \"C:\\inetpub\\wwwroot\\Default.htm\" -Value $($env:computername)"}' `
           -Location EastUS
   ```

### <a name="create-the-on-premises-virtual-machine"></a>Créer la machine virtuelle locale

Il s’agit d’une machine virtuelle que vous utilisez pour vous connecter au moyen du Bureau à distance et de l’adresse IP publique. À partir de là, vous vous connectez au serveur local via le pare-feu.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Populaire**, sélectionnez **Windows Server 2016 Datacenter**.
3. Entrez ces valeurs pour la machine virtuelle :
    - **Groupe de ressources** : sélectionnez Existant, puis **FW-Hybrid-Test**.
    - **Nom de la machine virtuelle** - *VM-Onprem*.
    - **Région** -  *(États-Unis) USA Est)* .
    - **Nom d’utilisateur** : *azureuser*.
    - **Mot de passe** : tapez votre mot de passe.

4. Sélectionnez **Suivant : Disques**.
5. Acceptez les valeurs par défaut, puis sélectionnez **Suivant : Réseaux**.
6. Sélectionnez **VNet-Onprem** comme réseau virtuel et vérifiez que le sous-réseau est **SN-Corp**.
7. Pour **Ports d’entrée publics**, sélectionnez **Autoriser les ports sélectionnés**, puis sélectionnez **RDP (3389)** .
8. Sélectionnez **Suivant : Gestion**.
9. Pour **Diagnostics de démarrage**, sélectionnez **Désactivé**.
10. Sélectionnez **Vérifier + Créer**, vérifiez les paramètres sur la page de résumé, puis sélectionnez **Créer**.

## <a name="test-the-firewall"></a>Tester le pare-feu

1. Tout d’abord, notez l’adresse IP privée de la machine virtuelle VM-spoke-01 dans la page Vue d’ensemble de VM-Spoke-01.

2. À partir du portail Azure, connectez-vous à la machine virtuelle **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
3. Ouvrez un navigateur web sur **VM-Onprem** et accédez à http://\<VM-spoke-01 private IP\>.

   La page web **VM-spoke-01** doit s’afficher : ![Page web VM-Spoke-01](media/secure-hybrid-network/vm-spoke-01-web.png)

4. À partir de la machine virtuelle **VM-Onprem**, ouvrez une session de Bureau à distance sur **VM-spoke-01** à l’adresse IP privée.

   La connexion doit réussir et vous devriez pouvoir vous connecter.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

<!---- You can ping the server on the spoke VNet.--->
- Vous pouvez parcourir le serveur web sur le réseau virtuel spoke.
- Vous pouvez vous connecter au serveur sur le réseau virtuel spoke à l’aide de RDP.

Modifiez ensuite l’action de collecte des règles du réseau de pare-feu en **Refuser** pour vérifier que les règles de pare-feu fonctionnent comme prévu.

1. Ouvrez le groupe de ressources **FW-Hybrid-test** et sélectionnez la stratégie de pare-feu **Pol-Net01**.
2. Sous **Paramètres**, sélectionnez **Règles**.
3. Sous **Règles réseau**, sélectionnez la collection de règles **RCNet01**, sélectionnez les points de suspension (...), puis **Modifier**.
4. Comme **Action de collection de règles**, sélectionnez **Refuser**.
5. Sélectionnez **Enregistrer**.

Fermez tous les navigateurs et bureaux distants existants sur **VM-Onprem** avant de tester les règles modifiées. Une fois la mise à jour de la collection de règles terminée, réexécutez les tests. Cette fois, ils doivent échouer.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez garder vos ressources de pare-feu pour le prochain didacticiel, ou, si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources **FW-Hybrid-Test** pour supprimer toutes les ressources associées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Sécuriser votre réseau étendu virtuel avec Azure Firewall Manager](secure-cloud-network.md)
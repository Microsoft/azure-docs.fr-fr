---
title: 'Tutoriel : Déployer et configurer un Pare-feu Azure dans un réseau hybride à l’aide du portail Azure'
description: Ce tutoriel vous apprend à déployer et configurer un Pare-feu Azure à l’aide du portail Azure.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: tutorial
ms.date: 11/17/2020
ms.author: victorh
customer intent: As an administrator, I want to control network access from an on-premises network to an Azure virtual network.
ms.openlocfilehash: 78e934a90b8d4e8feccf18a5cada3ec4920e1642
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94734449"
---
# <a name="tutorial-deploy-and-configure-azure-firewall-in-a-hybrid-network-using-the-azure-portal"></a>Tutoriel : Déployer et configurer un Pare-feu Azure dans un réseau hybride à l’aide du portail Azure

Lorsque vous connectez votre réseau local à un réseau virtuel Azure pour créer un réseau hybride, la possibilité de contrôler l’accès à vos ressources réseau Azure représente une part importante dans un plan de sécurité générale.

Vous pouvez utiliser le Pare-feu Azure pour contrôler l’accès réseau d’un réseau hybride à l’aide de règles définissant le trafic réseau autorisé et refusé.

Pour ce tutoriel, vous créez trois réseaux virtuels :

- **VNet-Hub** : Le pare-feu se trouve dans ce réseau virtuel.
- **VNet-Spoke** : Le réseau virtuel spoke correspond à la charge de travail sur Azure.
- **VNet-Onprem** : Le réseau virtuel local représente un réseau local. Dans un déploiement réel, il peut être connecté via un VPN ou une connexion ExpressRoute. Par souci de simplicité, ce tutoriel utilise une connexion de passerelle VPN, sachant qu’un réseau virtuel situé sur Azure est utilisé pour représenter un réseau local.

![Pare-feu dans un réseau hybride](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Déclarer les variables
> * Créer le réseau virtuel du hub de pare-feu
> * Créer le réseau virtuel spoke
> * Créer le réseau virtuel local
> * Configurer et déployer le pare-feu
> * Créer et connecter les passerelles VPN
> * Appairer les réseaux virtuels hub et spoke
> * Créer les itinéraires
> * Créer les machines virtuelles
> * Tester le pare-feu

Si vous souhaitez plutôt utiliser Azure PowerShell pour suivre cette procédure, consultez [Déployer et configurer un Pare-feu Azure dans un réseau hybride à l’aide d’Azure PowerShell](tutorial-hybrid-ps.md).

## <a name="prerequisites"></a>Prérequis

Un réseau hybride utilise le modèle d’architecture Hub and Spoke pour router le trafic entre des réseaux virtuels Azure et des réseaux locaux. L’architecture Hub and Spoke présente les conditions suivantes :

- Vous devez définir **AllowGatewayTransit** lors du Peering de VNet-Hub à VNet-Spoke. Dans une architecture réseau Hub and Spoke, le transit par passerelle permet aux réseaux virtuels spoke d’exploiter la passerelle VPN du hub, évitant ainsi de devoir déployer des passerelles VPN dans chaque réseau virtuel spoke. 

   De plus, les routes vers les réseaux locaux ou les réseaux virtuels connectés à la passerelle sont propagés automatiquement aux tables de routage pour les réseaux virtuels homologués à l’aide du transit par passerelle. Pour plus d’informations, consultez [Configurer le transit par passerelle VPN pour le peering de réseaux virtuels](../vpn-gateway/vpn-gateway-peering-gateway-transit.md).

- Vous devez définir **UseRemoteGateways** lors du Peering de VNet-Spoke à VNet-Hub. Si **UseRemoteGateways avec** est défini et que **AllowGatewayTransit** sur le Peering distant est également défini, le réseau virtuel spoke utilise les passerelles du réseau virtuel distant pour le transit.
- Pour router le trafic de sous-réseau spoke par le biais du pare-feu de hub, vous pouvez utiliser une route définie par l’utilisateur (UDR, User-Defined Route) qui pointe vers le pare-feu avec l’option **Propagation de la route de la passerelle de réseau virtuel** désactivée. L’option désactivée **Propagation de la route de la passerelle de réseau virtuel** empêche la distribution des routes vers les sous-réseaux spoke. Cela empêche que les routes apprises entrent en conflit avec votre UDR. Si vous souhaitez conserver la **Propagation de la route de la passerelle de réseau virtuel** activée, veillez à définir des routes spécifiques vers le pare-feu pour remplacer celles qui sont publiées à partir du site local sur le protocole BGP.
- Vous devez configurer une UDR sur le sous-réseau de passerelle hub qui pointe vers l’adresse IP du pare-feu comme prochain tronçon vers les réseaux spoke. Aucun UDR n’est requis sur le sous-réseau du Pare-feu Azure, puisqu’il apprend les itinéraires à partir de BGP.

Consultez la section [Créer des itinéraires](#create-the-routes) de ce didacticiel pour voir comment ces itinéraires sont créés.

>[!NOTE]
>Le Pare-feu Azure doit avoir une connectivité Internet directe. Si votre AzureFirewallSubnet prend connaissance d’un itinéraire par défaut pour votre réseau local via le protocole BGP, vous devez le remplacer par un UDR 0.0.0.0/0 avec la valeur **NextHopType** définie sur **Internet** pour garantir une connectivité Internet directe.
>
>Le Pare-feu Azure peut être configuré pour prendre en charge le tunneling forcé. Pour plus d’informations, consultez la page [Tunneling forcé du Pare-feu Azure](forced-tunneling.md).

>[!NOTE]
>Le trafic entre les réseaux virtuels directement appairés est acheminé directement même si l’UDR pointe vers le Pare-feu Azure en tant que passerelle par défaut. Pour envoyer un trafic de sous-réseau à sous-réseau au pare-feu dans ce scénario, un UDR doit contenir explicitement le préfixe du réseau cible dans les deux sous-réseaux.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="create-the-firewall-hub-virtual-network"></a>Créer le réseau virtuel du hub de pare-feu

Tout d’abord, créez le groupe de ressources qui doit contenir les ressources de ce tutoriel :

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Dans la page d’accueil du portail Azure, sélectionnez **Groupes de ressources** > **Ajouter**.
3. Pour **Abonnement**, sélectionnez votre abonnement.
1. Pour **Nom du groupe de ressources**, tapez **FW-Hybrid-Test**.
2. Pour **Région**, sélectionnez **(États-Unis) USA Est**. Toutes les ressources que vous créez par la suite doivent se trouver dans le même emplacement.
3. Sélectionnez **Vérifier + créer**.
4. Sélectionnez **Create** (Créer).

À présent, créez le réseau virtuel :

> [!NOTE]
> La taille du sous-réseau AzureFirewallSubnet est /26. Pour plus d’informations sur la taille du sous-réseau, consultez le [FAQ Pare-feu Azure](firewall-faq.md#why-does-azure-firewall-need-a-26-subnet-size).

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Mise en réseau**, sélectionnez **Réseau virtuel**.
1. Sélectionnez **Create** (Créer).
1. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
1. Dans le champ **Nom**, tapez **VNet-hub**.
1. Sélectionnez **Suivant : Adresses IP**.
1. Pour **Espace d’adressage IPv4**, supprimez l’adresse et le type par défaut **10.5.0.0/16**.
1. Sous **Nom du sous-réseau**, sélectionnez **Ajouter un sous-réseau**.
1. Sous **Nom du sous-réseau**, entrez **AzureFirewallSubnet**. Le pare-feu se trouvera dans ce sous-réseau et le nom du sous-réseau **doit** être AzureFirewallSubnet.
1. Pour **Plage d’adresses de sous-réseau**, tapez **10.5.0.0/26**. 
1. Sélectionnez **Ajouter**.
1. Sélectionnez **Vérifier + créer**.
1. Sélectionnez **Create** (Créer).

## <a name="create-the-spoke-virtual-network"></a>Créer le réseau virtuel spoke

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Mise en réseau**, sélectionnez **Réseau virtuel**.
7. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
1. Pour **Nom**, tapez **VNet-Spoke**.
2. Pour **Région**, sélectionnez **(États-Unis) USA Est**.
3. Sélectionnez **Suivant : Adresses IP**.
4. Pour **Espace d’adressage IPv4**, supprimez l’adresse et le type par défaut **10.6.0.0/16**.
6. Sous **Nom du sous-réseau**, sélectionnez **Ajouter un sous-réseau**.
7. Pour **Nom du sous-réseau**, tapez **SN-Workload**.
8. Pour **Plage d’adresses de sous-réseau**, tapez **10.6.0.0/24**. 
9. Sélectionnez **Ajouter**.
10. Sélectionnez **Vérifier + créer**.
11. Sélectionnez **Create** (Créer).

## <a name="create-the-on-premises-virtual-network"></a>Créer le réseau virtuel local

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Dans **Mise en réseau**, sélectionnez **Réseau virtuel**.
7. Pour **Groupe de ressources**, sélectionnez **FW-Hybrid-Test**.
1. Pour **Nom**, tapez **VNet-OnPrem**.
2. Pour **Région**, sélectionnez **(États-Unis) USA Est**.
3. Sélectionnez **Suivant : Adresses IP**
4. Pour **Espace d’adressage IPv4**, supprimez l’adresse et le type par défaut **192.168.0.0/16**.
5. Sous **Nom du sous-réseau**, sélectionnez **Ajouter un sous-réseau**.
7. Pour **Nom du sous-réseau**, tapez **SN-Corp**.
8. Pour **Plage d’adresses de sous-réseau**, tapez **192.168.1.0/24**. 
9. Sélectionnez **Ajouter**.
10. Sélectionnez **Vérifier + créer**.
11. Sélectionnez **Create** (Créer).

À présent, créez un second sous-réseau pour la passerelle.

1. Sur la page **VNet-Onprem**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **+Sous-réseau**.
3. Pour **Nom**, tapez **GatewaySubnet**.
4. Pour **Plage d’adresses du sous-réseau**, entrez **192.168.2.0/24**.
5. Sélectionnez **OK**.

## <a name="configure-and-deploy-the-firewall"></a>Configurer et déployer le pare-feu

À présent, déployez le pare-feu dans le réseau virtuel du hub de pare-feu.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Dans la colonne de gauche, sélectionnez **Mise en réseau**, puis recherchez et sélectionnez **Pare-feu**.
4. Sur la page **Créer un pare-feu**, utilisez le tableau suivant pour configurer le pare-feu :

   |Paramètre  |Valeur  |
   |---------|---------|
   |Abonnement     |\<your subscription\>|
   |Resource group     |**FW-Hybrid-Test** |
   |Nom     |**AzFW01**|
   |Région     |**USA Est**|
   |Choisir un réseau virtuel     |**Utiliser l’existant** :<br> **VNet-hub**|
   |Adresse IP publique     |Ajouter nouveau : <br>**fw-pip**. |

5. Sélectionnez **Revoir + créer**.
6. Passez en revue le récapitulatif, puis sélectionnez **Créer** pour créer le pare-feu.

   Le déploiement prend quelques minutes.
7. Une fois le déploiement terminé, accédez au groupe de ressources **FW-Hybrid-Test**, puis sélectionnez le pare-feu **AzFW01**.
8. Notez l’adresse IP privée. Vous l’utiliserez plus tard lors de la création de l’itinéraire par défaut.

### <a name="configure-network-rules"></a>Configurer des règles de réseau

Tout d’abord, ajoutez une règle de réseau pour autoriser le trafic web.

1. Sur la page **AzFW01**, sélectionnez **Règles**.
2. Sélectionnez l’onglet **Collection de règles de réseau**.
3. Sélectionnez **Ajouter une collection de règles de réseau**.
4. Dans le champ **Nom**, tapez **RCNet01**.
5. Pour **Priorité**, tapez **100**.
6. Pour **Action**, sélectionnez **Autoriser**.
6. Sous **Règles**, pour **Nom**, tapez **AllowWeb**.
7. Pour **Protocole**, sélectionnez **TCP**.
8. Pour **Type de source**, sélectionnez **Adresse IP**.
9. Pour **Source**, tapez **192.168.1.0/24**.
10. Pour **Type de destination**, sélectionnez **Adresse IP**.
11. Pour **Adresse de destination**, tapez **10.6.0.0/16**.
12. Pour **Ports de destination**, tapez **80**.

À présent, ajoutez une règle pour autoriser le trafic RDP.

Sur la deuxième ligne de la règle, tapez les informations suivantes :

1. Pour **Nom**, tapez **AllowRDP**.
2. Pour **Protocole**, sélectionnez **TCP**.
3. Pour **Type de source**, sélectionnez **Adresse IP**.
4. Pour **Source**, tapez **192.168.1.0/24**.
5. Pour **Type de destination**, sélectionnez **Adresse IP**.
6. Pour **Adresse de destination**, tapez **10.6.0.0/16**.
7. Pour **Ports de destination**, tapez **3389**.
8. Sélectionnez **Ajouter**.

## <a name="create-and-connect-the-vpn-gateways"></a>Créer et connecter les passerelles VPN

Les réseaux virtuels hub et local sont connectés via des passerelles VPN.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Créer une passerelle VPN pour le réseau virtuel hub

Maintenant, créez la passerelle VPN pour le réseau virtuel hub. Les configurations de réseau virtuel à réseau virtuel nécessitent un VPN de type RouteBased. La création d’une passerelle VPN nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle VPN sélectionnée.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Dans la zone de recherche, tapez **passerelle de réseau virtuel**.
3. Sélectionnez **Passerelle de réseau virtuel**, puis sélectionnez **Créer**.
4. Dans le champ **Nom**, tapez **GW-hub**.
5. Pour **Région**, sélectionnez la même région que celle utilisée précédemment.
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
5. Pour **Région**, sélectionnez la même région que celle utilisée précédemment.
6. Pour **Type de passerelle**, sélectionnez **VPN**.
7. Pour **Type de VPN**, sélectionnez **Basé sur itinéraires**.
8. Pour **Référence (SKU)** , sélectionnez **De base**.
9. Pour **Réseau virtuel**, sélectionnez **VNet-Onprem**.
10. Pour **Adresse IP publique**, sélectionnez **Créer nouveau**, puis tapez le nom **VNet-Onprem-GW-pip**.
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

![Connexions de passerelle](media/tutorial-hybrid-portal/gateway-connections.png)

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Appairer les réseaux virtuels hub et spoke

À présent, appairez les réseaux virtuels hub et spoke.

1. Ouvrez le groupe de ressources **FW-Hybrid-Test** et sélectionnez le réseau virtuel **VNet-hub**.
2. Dans la colonne de gauche, sélectionnez **Peerings**.
3. Sélectionnez **Ajouter**.
4. Sous **Ce réseau virtuel** :
 
   
   |Nom du paramètre  |Valeur  |
   |---------|---------|
   |Nom du lien de peering| HubtoSpoke|
   |Trafic vers le réseau virtuel distant|   Autoriser (par défaut)      |
   |Trafic transféré à partir du réseau virtuel distant    |   Autoriser (par défaut)      |
   |Passerelle de réseau virtuel     |  Utiliser la passerelle de ce réseau virtuel       |
    
5. Sous **Réseau virtuel distant** :

   |Nom du paramètre  |Valeur  |
   |---------|---------|
   |Nom du lien de peering | SpoketoHub|
   |Modèle de déploiement de réseau virtuel| Resource manager|
   |Abonnement|\<your subscription\>|
   |Réseau virtuel| VNet-Spoke
   |Trafic vers le réseau virtuel distant     |   Autoriser (par défaut)      |
   |Trafic transféré à partir du réseau virtuel distant    |   Autoriser (par défaut)      |
   |Passerelle de réseau virtuel     |  Utiliser la passerelle du réseau virtuel distant       |

5. Sélectionnez **Ajouter**.

   :::image type="content" source="media/tutorial-hybrid-portal/firewall-peering.png" alt-text="VNET Peering":::

## <a name="create-the-routes"></a>Créer les itinéraires

Ensuite, créez deux itinéraires :

- Un itinéraire à partir du sous-réseau de passerelle hub vers le sous-réseau spoke via l’adresse IP du pare-feu
- Un itinéraire par défaut à partir du sous-réseau spoke via l’adresse IP du pare-feu

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **table de routage** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Table de routage**.
4. Sélectionnez **Create** (Créer).
6. Sélectionnez le groupe de ressources **FW-Hybrid-Test**.
8. Pour **Région**, sélectionnez le même emplacement que celui utilisé précédemment.
1. Pour le nom, tapez **UDR-Hub-Spoke**.
9. Sélectionnez **Vérifier + créer**.
10. Sélectionnez **Create** (Créer).
11. Une fois la table de routage créée, sélectionnez-la pour ouvrir la page correspondante.
12. Sélectionnez **Routes** dans la colonne de gauche.
13. Sélectionnez **Ajouter**.
14. Pour le nom de la route, tapez **ToSpoke**.
15. Pour le préfixe d’adresse, tapez **10.6.0.0/16**.
16. Pour le type de tronçon suivant, sélectionnez **Appliance virtuelle**.
17. Pour l’adresse du tronçon suivant, tapez l’adresse IP privée du pare-feu que vous avez notée précédemment.
18. Sélectionnez **OK**.

À présent, associez la route au sous-réseau.

1. Sur la page **UDR-Hub-Spoke - Routes**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **Associer**.
3. Sous **Réseau virtuel**, sélectionnez **VNet-hub**.
1. Sous **Sous-réseau**, sélectionnez **GatewaySubnet**.
2. Sélectionnez **OK**.

À présent, créez la route par défaut à partir du sous-réseau spoke.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Tapez **table de routage** dans la zone de recherche, puis appuyez sur **Entrée**.
3. Sélectionnez **Table de routage**.
5. Sélectionnez **Create** (Créer).
7. Sélectionnez le groupe de ressources **FW-Hybrid-Test**.
8. Pour **Région**, sélectionnez le même emplacement que celui utilisé précédemment.
1. Pour le nom, tapez **UDR-DG**.
4. Pour **Propager la route de la passerelle**, sélectionnez **Non**.
5. Sélectionnez **Vérifier + créer**.
6. Sélectionnez **Create** (Créer).
7. Une fois la table de routage créée, sélectionnez-la pour ouvrir la page correspondante.
8. Sélectionnez **Routes** dans la colonne de gauche.
9. Sélectionnez **Ajouter**.
10. Pour le nom de la route, tapez **ToHub**.
11. Pour le préfixe d’adresse, tapez **0.0.0.0/0**.
12. Pour le type de tronçon suivant, sélectionnez **Appliance virtuelle**.
13. Pour l’adresse du tronçon suivant, tapez l’adresse IP privée du pare-feu que vous avez notée précédemment.
14. Sélectionnez **OK**.

À présent, associez la route au sous-réseau.

1. Sur la page **UDR-DG - Routes**, sélectionnez **Sous-réseaux**.
2. Sélectionnez **Associer**.
3. Sous **Réseau virtuel**, sélectionnez **VNet-spoke**.
1. Sous **Sous-réseau**, sélectionnez **SN-Workload**.
2. Sélectionnez **OK**.

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Maintenant, créez les machines virtuelles de charge de travail spoke et locale, et placez-les dans les sous-réseaux appropriés.

### <a name="create-the-workload-virtual-machine"></a>Créer la machine virtuelle de charge de travail

Créez une machine virtuelle dans le réseau virtuel spoke, exécutant IIS, sans adresse IP publique.

1. Dans la page d’accueil du portail Azure, sélectionnez **Créer une ressource**.
2. Sous **Populaire**, sélectionnez **Windows Server 2016 Datacenter**.
3. Entrez ces valeurs pour la machine virtuelle :
    - **Groupe de ressources** : sélectionnez **FW-Hybrid-Test**.
    - **Nom de la machine virtuelle** : *VM-Spoke-01*.
    - **Région** : région que vous avez utilisée précédemment.
    - **Nom d’utilisateur** : \<type a user name\>.
    - **Mot de passe** : \<type a password\>
4. Pour **Ports d’entrée publics**, sélectionnez **Autoriser les ports sélectionnés**, puis sélectionnez **HTTP (80)** et **RDP (3389)** .
4. Sélectionnez **Suivant : Disques**.
5. Acceptez les valeurs par défaut, puis sélectionnez **Suivant : Mise en réseau**.
6. Sélectionnez **VNet-Spoke** pour le réseau virtuel et **SN-Workload** pour le sous-réseau.
7. Pour **Adresse IP publique**, sélectionnez **Aucune**. 
9. Sélectionnez **Suivant : Gestion**.
10. Pour **Diagnostics de démarrage**, sélectionnez **Désactiver**.
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
    - **Région** : région que vous avez utilisée précédemment.
    - **Nom d’utilisateur** : \<type a user name\>.
    - **Mot de passe** : \<type a user password\>.
7. Pour **Ports d’entrée publics**, sélectionnez **Autoriser les ports sélectionnés**, puis sélectionnez **RDP (3389)** .
4. Sélectionnez **Suivant : Disques**.
5. Acceptez les valeurs par défaut, puis sélectionnez **Suivant : Réseaux**.
6. Sélectionnez **VNet-Onprem** pour le réseau virtuel et **SN-Corp** pour le sous-réseau.
8. Sélectionnez **Suivant : Gestion**.
10. Pour **Diagnostics de démarrage**, sélectionnez **Désactiver**.
10. Sélectionnez **Vérifier + Créer**, vérifiez les paramètres sur la page de résumé, puis sélectionnez **Créer**.

## <a name="test-the-firewall"></a>Tester le pare-feu

1. Tout d’abord, notez l’adresse IP privée de la machine virtuelle **VM-spoke-01**.

2. À partir du portail Azure, connectez-vous à la machine virtuelle **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
3. Ouvrez un navigateur web sur **VM-Onprem** et accédez à http://\<VM-spoke-01 private IP\>.

   La page web **VM-spoke-01** doit s’afficher : ![Page web VM-Spoke-01](media/tutorial-hybrid-portal/VM-Spoke-01-web.png)

4. À partir de la machine virtuelle **VM-Onprem**, ouvrez une session de Bureau à distance sur **VM-spoke-01** à l’adresse IP privée.

   La connexion doit réussir et vous devriez pouvoir vous connecter.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

<!---- You can ping the server on the spoke VNet.--->
- Vous pouvez parcourir le serveur web sur le réseau virtuel spoke.
- Vous pouvez vous connecter au serveur sur le réseau virtuel spoke à l’aide de RDP.

Modifiez ensuite l’action de collecte des règles du réseau de pare-feu en **Refuser** pour vérifier que les règles de pare-feu fonctionnent comme prévu.

1. Sélectionnez le pare-feu **AzFW01**.
2. Sélectionnez **Règles**.
3. Sélectionnez l’onglet **Collection de règles de réseau**, puis sélectionnez la collection de règles **RCNet01**.
4. Pour **Action**, sélectionnez **Refuser**.
5. Sélectionnez **Enregistrer**.

Fermez les bureaux à distance existants avant de tester les règles modifiées. Maintenant, réexécutez les tests. Cette fois, ils doivent échouer.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez garder vos ressources de pare-feu pour le prochain didacticiel, ou, si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources **FW-Hybrid-Test** pour supprimer toutes les ressources associées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

Ensuite, vous pouvez surveiller les journaux d’activité de Pare-feu Azure.

> [!div class="nextstepaction"]
> [Tutoriel : Superviser les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)
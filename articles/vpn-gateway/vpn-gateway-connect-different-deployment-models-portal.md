---
title: 'Connecter des réseaux virtuels classiques à des réseaux virtuels Resource Manager : Portail | Microsoft Docs'
description: Étapes pour connecter des réseaux virtuels classiques à des réseaux virtuels Resource Manager à l’aide d’une passerelle VPN et du portail
services: vpn-gateway
author: cherylmc
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/24/2019
ms.author: cherylmc
ms.openlocfilehash: f307ff9e7d609628bc22374fc5874dcbf993e4c2
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94661186"
---
# <a name="connect-virtual-networks-from-different-deployment-models-using-the-portal"></a>Connecter des réseaux virtuels utilisant des modèles de déploiement différents dans le portail

Cet article vous explique comment connecter des réseaux virtuels classiques à des réseaux virtuels Resource Manager afin de permettre aux ressources situées dans les modèles de déploiement distincts de communiquer entre elles. Les étapes décrites dans cet article utilisent principalement le portail Azure, mais vous pouvez également créer cette configuration à l’aide de PowerShell en sélectionnant l’article dans cette liste.

> [!div class="op_single_selector"]
> * [Portail](vpn-gateway-connect-different-deployment-models-portal.md)
> * [PowerShell](vpn-gateway-connect-different-deployment-models-powershell.md)
> 
> 

La connexion d’un réseau virtuel classique à un réseau virtuel Resource Manager est semblable à la connexion d’un réseau virtuel à un emplacement de site local. Les deux types de connectivité font appel à une passerelle VPN pour offrir un tunnel sécurisé utilisant Ipsec/IKE. Vous pouvez créer une connexion entre des réseaux virtuels situés dans des abonnements différents et des régions différentes. Vous pouvez également connecter des réseaux virtuels qui disposent déjà de connexions à des réseaux locaux, à condition que la passerelle avec laquelle ils ont été configurés soit dynamique ou basée sur un itinéraire. Pour plus d’informations sur les connexions de réseau virtuel à réseau virtuel, consultez le [Forum Aux Questions sur l’interconnexion de réseaux virtuels](#faq) à la fin de cet article. 

Si vous n’avez pas encore de passerelle de réseau virtuel et que vous ne souhaitez pas en créer une, vous voudrez peut-être connecter vos réseaux virtuels en utilisant le peering de réseaux virtuels. L’homologation de réseaux virtuels (ou VNet Peering) n’utilise pas de passerelle VPN. Pour plus d’informations, consultez l’article [Peering de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

### <a name="before-you-begin"></a><a name="before"></a>Avant de commencer



* Ces étapes supposent que les deux réseaux virtuels ont déjà été créés. Si vous utilisez cet article en guise d’exercice et que vous ne disposez pas de réseaux virtuels, vous trouverez des liens dans les étapes pour vous aider à les créer.
* Vérifiez que les plages d’adresses des réseaux virtuels ne se chevauchent pas ou ne chevauchent aucune des plages des autres connexions susceptibles d’être utilisées par les passerelles.
* Installez les dernières applets de commande PowerShell pour le Gestionnaire de ressources et pour la Gestion des services (classique). Dans cet article, nous utilisons le portail Azure et PowerShell. PowerShell est requis pour créer la connexion à partir du réseau virtuel classique vers le réseau virtuel Resource Manager. Pour plus d’informations, consultez [Installer et configurer Azure PowerShell](/powershell/azure/). 

### <a name="example-settings"></a><a name="values"></a>Exemples de paramètres

Vous pouvez utiliser ces valeurs pour créer un environnement de test ou vous y référer pour mieux comprendre les exemples de cet article.

**Réseau virtuel classique**

Nom du réseau virtuel = ClassicVNet <br>
Espace d’adressage = 10.0.0.0/24 <br>
Nom du sous-réseau = Subnet-1 <br>
Plage d’adresses de sous-réseau = 10.0.0.0/27 <br>
Abonnement = abonnement à utiliser <br>
Groupe de ressources = ClassicRG <br>
Emplacement = USA Ouest <br>
Sous-réseau de passerelle = 10.0.0.32/28 <br>
Site local = RMVNetLocal <br>

**Réseau virtuel Resource Manager**

Nom du réseau virtuel = RMVNet <br>
Espace d’adressage = 192.168.0.0/16 <br>
Groupe de ressources = RG1 <br>
Emplacement = USA Est <br>
Nom du sous-réseau = Subnet-1 <br>
Plage d’adresses = 192.168.1.0/24 <br>
Sous-réseau de passerelle = 192.168.0.0/26 <br>
Nom de passerelle de réseau virtuel = RMGateway <br>
Type de passerelle = VPN <br>
Type de VPN = Route-based <br>
Référence (SKU) = VpnGw1 <br>
Emplacement = USA Est <br>
Réseau virtuel = RMVNet <br> (associer la passerelle VPN à ce réseau virtuel) Première configuration IP = rmgwpip <br> (adresse IP publique de passerelle) Passerelle de réseau local = ClassicVNetLocal <br>
Nom de connexion = RMtoClassic

### <a name="connection-overview"></a><a name="connectoverview"></a>Vue d’ensemble de la connexion

Pour cette configuration, vous créez une connexion de passerelle VPN via un tunnel VPN IPsec/IKE entre les réseaux virtuels. Assurez-vous que vos plages de réseau virtuel ne se chevauchent pas entre elles ou avec un réseau local auquel elles se connectent.

Le tableau suivant montre comment les réseaux virtuels et les sites locaux sont définis :

| Réseau virtuel | Espace d'adressage | Région | Se connecte au site de réseau local |
|:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |USA Ouest | RMVNetLocal (192.168.0.0/16) |
| RMVNet | (192.168.0.0/16) |USA Est |ClassicVNetLocal (10.0.0.0/24) |

## <a name="section-1---configure-the-classic-vnet-settings"></a><a name="classicvnet"></a>Section 1 - Configurer les paramètres de réseau virtuel classique

Dans cette section, vous créez le réseau virtuel classique, le réseau local (site local) et la passerelle de réseau virtuel. Les captures d’écran sont fournies à titre d’exemple. Assurez-vous de remplacer ces valeurs par les vôtres,ou utilisez les valeurs [d’exemple](#values).

### <a name="1-create-a-classic-vnet"></a>1. <a name="classicvnet"></a>Créer un réseau virtuel classique

Si vous n’avez pas de réseau virtuel classique et que vous exécutez ces étapes en guise d’exercice, vous pouvez créer un réseau virtuel à l’aide de [cet article](/previous-versions/azure/virtual-network/virtual-networks-create-vnet-classic-pportal) et les valeurs de paramètres [d’exemple](#values) ci-dessus.

Si vous disposez déjà d’un réseau virtuel avec une passerelle VPN, vérifiez que la passerelle est dynamique. Si elle est statique, vous devez d’abord supprimer la passerelle VPN avant de passer à [Configurer le site local](#local).

1. Ouvrez le [portail Azure](https://ms.portal.azure.com) et connectez-vous avec votre compte Azure.
2. Cliquez sur **+ Créer une ressource** pour ouvrir la page « Nouveau ».
3. Dans le champ « Rechercher dans le marketplace », saisissez « réseau virtuel ». Si vous sélectionnez Mise en réseau -> Réseau virtuel à la place, l’option de création d’un réseau virtuel classique ne s’affiche pas.
4. Localisez « Réseau virtuel » dans la liste renvoyée et cliquez dessus pour ouvrir la page correspondante. 
5. Dans la page Réseau virtuel, sélectionnez « Classique » pour créer un réseau virtuel classique. Si vous sélectionnez ici la valeur par défaut, vous vous retrouverez avec un réseau virtuel Resource Manager à la place.

### <a name="2-configure-the-local-site"></a>2. <a name="local"></a>Configurer le site local

1. Accédez à **Toutes les ressources** et recherchez le **réseau virtuel classique** dans la liste.
2. Cliquez sur **Passerelle** dans la section **Paramètres** du menu, puis sur la bannière pour créer une passerelle.
  ![Configurer une passerelle VPN](./media/vpn-gateway-connect-different-deployment-models-portal/gatewaygraphic.png "Configurer une passerelle VPN")
3. Dans la page **Nouvelle connexion VPN**, pour **Type de connexion**, sélectionnez **De site à site**.
4. Sous **Site local**, cliquez sur **Configurer les paramètres requis**. La page **Site local** s’ouvre.
5. Dans la page **Site local**, créez un nom pour faire référence au réseau virtuel Resource Manager. Par exemple, « RMVNetLocal ».
6. Si la passerelle VPN pour le réseau virtuel Resource Manager a déjà une adresse IP publique, utilisez cette valeur pour le champ **Adresse IP de passerelle VPN**. Si vous effectuez ces étapes en guise d’exercice, ou si ne disposez pas encore dune passerelle de réseau virtuel pour votre réseau virtuel Resource Manager, vous pouvez créer une adresse IP d’espace réservé. Assurez-vous que l’adresse IP avec espace réservé utilise un format valide. Remplacez ensuite l’adresse IP avec espace réservé par l’adresse IP publique de la passerelle de réseau virtuel Resource Manager.
7. Pour **Espace d’adressage du client**, utilisez les [valeurs](#connectoverview) des espaces d’adressage IP de réseau virtuel pour le réseau virtuel Resource Manager. Ce paramètre sert à spécifier les espaces d’adressage à router vers le réseau virtuel Resource Manager. Dans l’exemple, nous utilisons 192.168.0.0/16, la plage d’adresses pour le RMVNet.
8. Cliquez sur **OK** pour enregistrer les valeurs et revenez à la page **Nouvelle connexion VPN**.

### <a name="3-create-the-virtual-network-gateway"></a><a name="classicgw"></a>3. Créer la passerelle de réseau virtuel

1. Dans la page **Nouvelle connexion VPN**, cochez la case **Créer une passerelle immédiatement**.
2. Cliquez sur **Configuration de passerelle facultative** pour ouvrir la page **Configuration de la passerelle**.

   ![Ouvrir la page Configuration de la passerelle](./media/vpn-gateway-connect-different-deployment-models-portal/optionalgatewayconfiguration.png "Ouvrir la page Configuration de la passerelle")
3. Cliquez sur **Sous-réseau - Configurer les paramètres requis** pour ouvrir la page **Ajouter un sous-réseau**. Le **Nom** est déjà configuré avec la valeur requise : **GatewaySubnet**.
4. La **Plage d’adresses** fait référence à la plage pour le sous-réseau de passerelle. Bien que vous puissiez créer un sous-réseau de passerelle avec une plage d’adresses /29 (3 adresses), nous vous recommandons de créer un sous-réseau de passerelle qui contient plus d’adresses IP. Cela afin d’accueillir des configurations futures nécessitant la disponibilité d’un plus grand nombre d’adresses IP. Si possible, utilisez /27 ou /28. Si vous suivez ces étapes dans le cadre d’un exercice, vous pouvez vous référer aux [exemples de valeurs](#values). Pour cet exemple, nous utilisons « 10.0.0.32/28 ». Cliquez sur **OK** pour créer le sous réseau de passerelle.
5. Dans la page **Configuration de la passerelle**, le paramètre **Taille** désigne la référence SKU de passerelle. Sélectionnez la référence SKU de passerelle pour votre passerelle VPN.
6. Vérifiez que **Type de routage** est défini sur **Dynamique**, puis cliquez sur **OK** pour revenir à la page **Nouvelle connexion VPN**.
7. Dans la page **Nouvelle connexion VPN**, cliquez sur **OK** pour commencer à créer votre passerelle VPN. La création d’une passerelle VPN peut prendre jusqu’à 45 minutes.

### <a name="4-copy-the-virtual-network-gateway-public-ip-address"></a><a name="ip"></a>4. Copier l’adresse IP publique de passerelle de réseau virtuel

Une fois la passerelle de réseau virtuel créée, vous pouvez afficher l’adresse IP de la passerelle. 

1. Accédez à votre réseau virtuel classique, puis cliquez sur **Vue d’ensemble**.
2. Cliquez sur **Connexions VPN** afin d’ouvrir la page Connexions VPN. Dans la page Connexions VPN, vous pouvez afficher l’adresse IP publique. Il s’agit de l’adresse IP publique affectée à votre passerelle de réseau virtuel. Prenez note de l’adresse IP. Vous en aurez besoin aux étapes suivantes pour vos paramètres de configuration de passerelle de réseau local Resource Manager. 
3. Vous pouvez afficher l’état de vos connexions à la passerelle. Le site de réseau local que vous avez créé est répertorié comme « Connexion ». Le statut sera modifié une fois les connexions créées. Vous pouvez fermer cette page quand vous avez terminé de consulter l’état.

## <a name="section-2---configure-the-resource-manager-vnet-settings"></a><a name="rmvnet"></a>Section 2 - Configurer les paramètres du réseau virtuel Resource Manager

Dans cette section, vous créez la passerelle de réseau virtuel et la passerelle de réseau local pour votre réseau virtuel Resource Manager. Les captures d’écran sont fournies à titre d’exemple. Assurez-vous de remplacer ces valeurs par les vôtres,ou utilisez les valeurs [d’exemple](#values).

### <a name="1-create-a-virtual-network"></a>1. Créez un réseau virtuel

**Exemples de valeurs :**

* Nom du réseau virtuel = RMVNet <br>
* Espace d’adressage = 192.168.0.0/16 <br>
* Groupe de ressources = RG1 <br>
* Emplacement = USA Est <br>
* Nom du sous-réseau = Subnet-1 <br>
* Plage d’adresses = 192.168.1.0/24 <br>

Si vous n’avez pas de réseau virtuel Resource Manager et que vous exécutez ces étapes en guise d’exercice, créez un réseau virtuel à l’aide des étapes de l’article [Création d’un réseau virtuel](../virtual-network/quick-create-portal.md) et en utilisant les exemples de valeurs.

### <a name="2-create-a-virtual-network-gateway"></a><a name="creategw"></a>2. Créer une passerelle de réseau virtuel

Dans cette étape, vous créez la passerelle de réseau virtuel de votre réseau virtuel. La création d’une passerelle nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle sélectionnée.

[!INCLUDE [About gateway subnets](../../includes/vpn-gateway-about-gwsubnet-portal-include.md)]

**Exemples de valeurs :**

* Nom de passerelle de réseau virtuel = RMGateway <br>
* Type de passerelle = VPN <br>
* Type de VPN = Route-based <br>
* Référence (SKU) = VpnGw1 <br>
* Emplacement = USA Est <br>
* Réseau virtuel = RMVNet <br>
* Sous-réseau de passerelle = 192.168.0.0/26 <br>
* Première configuration IP = rmgwpip <br>

[!INCLUDE [Add gateway](../../includes/vpn-gateway-add-gw-rm-portal-empty.md)]

[!INCLUDE [vpn-gateway-no-nsg-include](../../includes/vpn-gateway-no-nsg-include.md)]

### <a name="3-create-a-local-network-gateway"></a><a name="createlng"></a>3. Créer une passerelle de réseau local

**Exemples de valeurs :** Passerelle de réseau local = ClassicVNetLocal

| Réseau virtuel | Espace d'adressage | Région | Se connecte au site de réseau local |Adresse IP publique de la passerelle|
|:--- |:--- |:--- |:--- |:--- |
| ClassicVNet |(10.0.0.0/24) |USA Ouest | RMVNetLocal (192.168.0.0/16) |L’adresse IP publique qui est affectée à la passerelle ClassicVNet|
| RMVNet | (192.168.0.0/16) |USA Est |ClassicVNetLocal (10.0.0.0/24) |L’adresse IP publique qui est affectée à la passerelle RMVNet.|

La passerelle de réseau local spécifie la plage d’adresses et l’adresse IP publique associées à votre réseau virtuel classique et à la passerelle de réseau virtuel. Si vous effectuez ces étapes en guise d’exercice, consultez les exemples de valeurs.

[!INCLUDE [Add local network gateway](../../includes/vpn-gateway-add-local-network-gateway-portal-ip-empty.md)]

## <a name="section-3---modify-the-classic-vnet-local-site-settings"></a><a name="modifylng"></a>Section 3 - Modifier les paramètres de site local du réseau virtuel classique

Dans cette section, vous remplacez l’adresse IP avec espace réservé que vous avez utilisée lorsque vous avez spécifié les paramètres du site local avec l’adresse IP de passerelle de réseau virtuel Resource Manager. Cette section utilise les applets de commande PowerShell (SM) classiques.

1. Dans le portail Azure, accédez au réseau virtuel classique.
2. Dans la page de votre réseau virtuel, cliquez sur **Vue d’ensemble**.
3. Dans la section **Connexions VPN**, cliquez sur le nom de votre site local dans le graphique.

   ![VPN-connections](./media/vpn-gateway-connect-different-deployment-models-portal/vpnconnections.png "Connexions VPN")
4. Dans la page **Connexions VPN de site à site**, cliquez sur le nom du site.

   ![Site-name](./media/vpn-gateway-connect-different-deployment-models-portal/sitetosite3.png "Nom du site local")
5. Dans la page de connexion de votre site local, cliquez sur le nom du site local pour ouvrir la page **Site local**.

   ![Open-local-site](./media/vpn-gateway-connect-different-deployment-models-portal/openlocal.png "Ouvrir le site local")
6. Dans la page **Site local**, remplacez l’**adresse IP de passerelle VPN** par l’adresse IP de la passerelle Resource Manager.

   ![Gateway-ip-address](./media/vpn-gateway-connect-different-deployment-models-portal/gwipaddress.png "Adresse IP de la passerelle")
7. Cliquez sur **OK** pour mettre à jour l’adresse IP.

## <a name="section-4---create-resource-manager-to-classic-connection"></a><a name="RMtoclassic"></a>Section 4 - Créer une connexion entre Resource Manager et le réseau virtuel classique

Dans cette procédure, vous configurez la connexion entre le réseau virtuel Resource Manager et le réseau virtuel classique à l’aide du portail Azure.

1. Dans **Toutes les ressources**, recherchez la passerelle de réseau local. Dans notre exemple, la passerelle de réseau local est **ClassicVNetLocal**.
2. Cliquez sur **Configuration** et vérifiez que la valeur d’adresse IP est la passerelle VPN pour le réseau virtuel classique. Mettez à jour si nécessaire, puis cliquez sur **Enregistrer**. Fermez la page.
3. Dans **Toutes les ressources**, cliquez sur la passerelle de réseau local.
4. Cliquez sur **Connexions** afin d’ouvrir la page Connexions.
5. Dans la page **Connexions**, cliquez sur **+** pour ajouter une connexion.
6. Dans la page **Ajouter une connexion**, donnez un nom à la connexion. Par exemple, « RMtoClassic ».
7. L’option **Site à site** est déjà sélectionnée dans cette page.
8. Sélectionnez la passerelle de réseau virtuel que vous souhaitez associer à ce site.
9. Créer une **clé partagée**. Cette clé est également utilisée dans la connexion que vous créez entre le réseau virtuel classique et le réseau virtuel Resource Manager. Vous pouvez générer la clé ou en créer une. Dans l’exemple, nous avons utilisé « abc123 », mais vous pouvez (et devriez) utiliser une valeur plus complexe.
10. Cliquez sur **OK** pour créer la connexion.

## <a name="section-5---create-classic-to-resource-manager-connection"></a><a name="classictoRM"></a>Section 5 - Créer une connexion entre le réseau virtuel classique et Resource Manager

Dans cette procédure, vous configurez la connexion entre le réseau virtuel Resource Manager et le réseau virtuel classique à l’aide du portail Azure. Pour ce faire, vous devez utiliser PowerShell. Vous ne pouvez pas créer cette connexion dans le portail. Assurez-vous que vous avez téléchargé et installé les applets de commande PowerShell classiques et Resource Manager (RM).

### <a name="1-connect-to-your-azure-account"></a>1. Se connecter au compte Azure

Ouvrez la console PowerShell avec des droits élevés et connectez-vous à votre compte Azure. Une fois que vous êtes connecté, vos paramètres de compte sont téléchargés pour être reconnus par Azure PowerShell. Les applets de commande suivantes vous invitent à entrer les informations d’identification de connexion de votre compte Azure pour le modèle de déploiement Resource Manager :

```powershell
Connect-AzAccount
```

Obtenez la liste de vos abonnements Azure.

```powershell
Get-AzSubscription
```

Si vous avez plusieurs abonnements, spécifiez celui que vous souhaitez utiliser.

```powershell
Select-AzSubscription -SubscriptionName "Name of subscription"
```

Connectez-vous ensuite pour utiliser les applets de commande PowerShell classiques (Service Management). Utilisez la commande suivante pour ajouter votre compte Azure pour le modèle de déploiement classique :

```powershell
Add-AzureAccount
```

Obtenez la liste de vos abonnements. Cette étape peut être nécessaire lors de l’ajout des applets de commande Service Management, selon le module Azure installé.

```powershell
Get-AzureSubscription
```

Si vous avez plusieurs abonnements, spécifiez celui que vous souhaitez utiliser.

```powershell
Select-AzureSubscription -SubscriptionName "Name of subscription"
```

### <a name="2-view-the-network-configuration-file-values"></a>2. Afficher le fichier de configuration réseau

Lorsque vous créez un réseau virtuel dans le portail Azure, le nom complet utilisé par Azure n’est pas visible dans le portail Azure. Par exemple, un réseau virtuel qui semble être nommé « ClassicVNet1 » dans le portail Azure peut avoir un nom beaucoup plus long dans le fichier de configuration réseau. Le nom pourrait être semblable au suivant : « Group ClassicRG ClassicVNet ». Dans ces étapes, vous téléchargez le fichier de configuration réseau et affichez les valeurs.

Créez un répertoire sur votre ordinateur, puis exportez le fichier de configuration réseau dans ce répertoire. Dans cet exemple, le fichier de configuration réseau est exporté vers C:\AzureNet.

```powershell
Get-AzureVNetConfig -ExportToFile C:\AzureNet\NetworkConfig.xml
```

Dans un éditeur de texte, ouvrez le fichier, puis affichez le nom de votre réseau virtuel classique. Utilisez les noms indiqués dans le fichier de configuration réseau lors de l’exécution de vos applets de commande PowerShell.

- Les noms des réseaux virtuels sont répertoriés comme suit : **VirtualNetworkSite name =**
- Les noms de site sont répertoriés comme suit : **LocalNetworkSite name=**

### <a name="3-create-the-connection"></a>3. Créer la connexion

Configurez la clé partagée et créez la connexion à partir du réseau virtuel classique vers le réseau virtuel Resource Manager. Vous ne pouvez pas définir la clé partagée à l’aide du portail. Veillez à exécuter les étapes suivantes lorsque vous êtes connecté à l’aide de la version classique des applets de commande PowerShell. Pour ce faire, utilisez **Add-AzureAccount**. Sinon, vous ne pourrez pas définir « -AzureVNetGatewayKey ».

- Dans cet exemple, **-VNetName** est le nom du réseau virtuel classique, comme indiqué dans votre fichier de configuration réseau. 
- **-LocalNetworkSiteName** est le nom que vous avez spécifié pour le site local, comme indiqué dans votre fichier de configuration réseau.
- **-SharedKey** est une valeur que vous pouvez générer et spécifier. Dans l’exemple, nous avons utilisé *abc123*, mais vous pouvez générer quelque chose de plus complexe. L’important, c’est que la valeur que vous spécifiez ici doit être identique à celle spécifiée lors de la création de la connexion entre Resource Manager et classique.

```powershell
Set-AzureVNetGatewayKey -VNetName "Group ClassicRG ClassicVNet" `
-LocalNetworkSiteName "172B9E16_RMVNetLocal" -SharedKey abc123
```

## <a name="section-6---verify-your-connections"></a><a name="verify"></a>Section 6 - Vérifier vos connexions

Vous pouvez vérifier vos connexions à l’aide du portail Azure ou de PowerShell. Lors de la vérification, vous devrez peut-être patienter quelques minutes, le temps que la connexion soit créée. Lorsqu’une connexion est réussie, l’état de connectivité passe de « Connexion » à « Connecté ».

### <a name="to-verify-the-connection-from-your-classic-vnet-to-your-resource-manager-vnet"></a>Pour vérifier la connexion de votre réseau virtuel classique à votre réseau virtuel Resource Manager

[!INCLUDE [vpn-gateway-verify-connection-azureportal-classic](../../includes/vpn-gateway-verify-connection-azureportal-classic-include.md)]

### <a name="to-verify-the-connection-from-your-resource-manager-vnet-to-your-classic-vnet"></a>Pour vérifier la connexion de votre réseau virtuel Resource Manager à votre réseau virtuel classique

[!INCLUDE [vpn-gateway-verify-connection-portal-rm](../../includes/vpn-gateway-verify-connection-portal-rm-include.md)]

## <a name="vnet-to-vnet-faq"></a><a name="faq"></a>Forum Aux Questions sur l’interconnexion de réseaux virtuels

[!INCLUDE [vpn-gateway-vnet-vnet-faq](../../includes/vpn-gateway-faq-vnet-vnet-include.md)]
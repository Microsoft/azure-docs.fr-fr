---
title: Déployer le modèle de solution Ethereum de consortium de preuve d’autorité sur Azure
description: Utiliser la solution Ethereum de consortium de preuve d’autorité pour déployer et configurer un réseau Ethereum de consortium sur Azure
ms.date: 07/23/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.custom: devx-track-js
ms.openlocfilehash: e680bc601b7f230314c1063523a003e95a849c0a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024396"
---
# <a name="deploy-ethereum-proof-of-authority-consortium-solution-template-on-azure"></a>Déployer le modèle de solution Ethereum de consortium de preuve d’autorité sur Azure

Vous pouvez utiliser [le modèle de solution Azure en préversion Ethereum de consortium de preuve d’autorité](https://azuremarketplace.microsoft.com/marketplace/apps/microsoft-azure-blockchain.azure-blockchain-ethereum) pour déployer, configurer et gouverner un réseau Ethereum de consortium de preuve d’autorité de plusieurs membres avec des connaissances minimales d’Azure et d’Ethereum.

Le modèle de solution peut être utilisé par chaque membre du consortium pour approvisionner une empreinte réseau blockchain à l’aide de services de calcul, de mise en réseau et de stockage Azure. L’empreinte réseau de chaque membre de consortium se compose d’un ensemble de nœuds de validateur avec équilibrage de charge grâce auquel une application ou un utilisateur peut interagir pour soumettre des transactions Ethereum.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="choose-an-azure-blockchain-solution"></a>Choisir une solution Azure Blockchain

Avant d'opter pour le modèle de solution Ethereum de consortium de preuve d'autorité, comparez votre scénario avec les cas d’utilisation courants des options Azure Blockchain disponibles.

> [!IMPORTANT]
> Envisagez d’utiliser [Azure Blockchain Service](../service/overview.md) plutôt que le modèle de solution Ethereum sur Azure. Azure Blockchain Service est un service géré Azure pris en charge. Parity Ethereum est passé en développement et maintenance pilotés par la communauté. Pour plus d’informations, consultez [Transition de Parity Ethereum vers la DAO OpenEthereum](https://www.parity.io/parity-ethereum-openethereum-dao/).

Option | Modèle de service | Cas d’utilisation courant
-------|---------------|-----------------
Modèles de solution | IaaS | Les modèles de solution correspondent à des modèles Azure Resource Manager que vous pouvez utiliser pour approvisionner une topologie de réseau blockchain entièrement configurée. Les modèles déploient et configurent les services de calcul, de mise en réseau et de stockage Microsoft Azure pour un type de réseau blockchain donné. Les modèles de solution sont fournis sans contrat SLA. Utilisez la [page de questions Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html) pour obtenir de l’aide.
[Azure Blockchain Service](../service/overview.md) | PaaS | Azure Blockchain Service (préversion) simplifie la formation, la gestion et la gouvernance des réseaux blockchain de consortium. Utilisez Azure Blockchain Service pour les solutions nécessitant PaaS, la gestion de consortium ou la confidentialité des contrats et transactions.
[Azure Blockchain Workbench](../workbench/overview.md) | IaaS et PaaS | La préversion d’Azure Blockchain Workbench est une collection de services et de fonctionnalités Azure conçus pour vous aider à créer et à déployer des applications blockchain afin de partager des processus métier et des données avec d’autres organisations. Utilisez Azure Blockchain Workbench pour le prototypage d’une solution blockchain ou d’une preuve de concept d’application blockchain. Azure Blockchain Workbench est fourni sans contrat de niveau de service. Utilisez la [page de questions Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html) pour obtenir de l’aide.

## <a name="solution-architecture"></a>Architecture de solution

Le modèle de solution Ethereum vous permet de déployer un ou plusieurs réseaux de consortium de preuve d'autorité Ethereum de plusieurs membres reposant sur une région.

![architecture de déploiement](./media/ethereum-poa-deployment/deployment-architecture.png)

Chaque déploiement d’un membre de consortium inclut :

* des machines virtuelles pour l’exécution des validateurs PoA ;
* Azure Load Balancer, pour la répartition des requêtes de RPC, de peering et de Governance DApp ;
* Azure Key Vault, pour sécuriser les identités de validateur ;
* un Stockage Azure, pour héberger les informations réseau persistantes et coordonner les baux ;
* Azure Monitor, pour l’agrégation des journaux d’activité et des statistiques de performances ;
* une passerelle de réseau virtuel (facultative) pour autoriser les connexions VPN sur les réseaux virtuels privés.

Par défaut, le RPC et les points de terminaison du peering sont accessibles via l’adresse IP publique pour simplifier la connectivité entre

les abonnements et les clouds. Pour les contrôles d'accès de niveau applicatif, vous pouvez utiliser les [contrats d’octroi d’autorisations de Parity](https://openethereum.github.io/Permissioning.html). Les réseaux déployés derrière des VPN, qui tirent parti des passerelles de réseau virtuel pour garantir une connectivité entre les abonnements, sont pris en charge. Les déploiements de VPN et de réseau virtuel étant plus complexes, vous pouvez commencer par un modèle d’adresse IP publique lors du prototypage d’une solution.

Des conteneurs Docker sont utilisés à des fins de fiabilité et de modularité. Azure Container Registry est utilisé pour héberger et diffuser des images avec contrôle de version dans le cadre de chaque déploiement. Les images de conteneur sont composées de la manière suivante :

* Orchestrateur - Génère des identités et des contrats de gouvernance. Stocke les identités dans un magasin d’identités.
* Client Parity - Utilise les identités du magasin d’identités sous forme de bail. Détecte les homologues et s’y connecte.
* Agent EthStats - Collecte les journaux d’activité locaux et les statistiques via RPC et envoie (push) les informations à Azure Monitor.
* Governance DApp - Interface Web qui interagit avec les contrats de gouvernance.

### <a name="validator-nodes"></a>Nœuds validateurs

Dans le protocole de preuve d’autorité, les nœuds de validateur se substituent aux nœuds d’explorateur traditionnels. Chaque validateur possède une identité Ethereum unique lui permettant de participer au processus de création de blocs. Chaque membre du consortium peut approvisionner deux ou plusieurs nœuds de validateur dans cinq régions, à des fins de géo-redondance. Les nœuds de validateur communiquent avec d’autres nœuds de validateur pour parvenir à un consensus sur l’état du registre distribué sous-jacent. Pour garantir une participation équitable sur le réseau, chaque membre du consortium n’a pas le droit d’utiliser plus de validateurs que le premier membre sur le réseau. Par exemple, si le premier membre déploie trois validateurs, chaque membre ne peut utiliser qu’un maximum de trois validateurs.

### <a name="identity-store"></a>Magasin d’identités

Un magasin d’identités est déployé dans l’abonnement de chaque membre, qui préserve la sécurité des identités Ethereum générées. Pour chaque validateur, le conteneur d’orchestration génère une clé privée Ethereum et la stocke dans Azure Key Vault.

## <a name="deploy-ethereum-consortium-network"></a>Déployer un réseau de consortium Ethereum

Pour cette procédure pas à pas, nous supposerons que vous créez un réseau de consortium Ethereum multiniveau. Le flux suivant illustre un déploiement impliquant plusieurs parties :

1. Trois membres génèrent chacun un compte Ethereum à l’aide de MetaMask
1. Le *Membre A* déploie Ethereum PoA, en fournissant son adresse Ethereum publique
1. Le *Membre A* fournit l’URL du consortium au *Membre B* et au *Membre C*
1. Le *Membre B* et le *Membre C* déploient Ethereum PoA, en fournissant leur adresse Ethereum publique et l’URL de consortium du *Membre A*
1. Le *Membre A* donne son vote au *Membre B* pour devenir administrateur
1. Le *Membre A* et le *Membre B* votent tous les deux pour que le *Membre C* devienne administrateur

Les sections suivantes vous expliquent comment configurer l’empreinte du premier membre sur le réseau.

### <a name="create-resource"></a>Créer une ressource

Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** en haut à gauche.

Sélectionnez **Blockchain** > **Ethereum Proof-of-Authority Consortium (préversion)** .

### <a name="basics"></a>Concepts de base

Sous **Informations de base**, spécifiez les valeurs des paramètres standard pour un déploiement.

![Concepts de base](./media/ethereum-poa-deployment/basic-blade.png)

Paramètre | Description | Valeur d'exemple
----------|-------------|--------------
Créer un réseau ou rejoindre un réseau existant | Vous pouvez créer un réseau ou rejoindre un réseau de consortium existant. Rejoindre un réseau existant implique des paramètres supplémentaires. | Création
Adresse de messagerie | Une fois votre déploiement terminé, vous recevez par e-mail une notification contenant des informations sur votre déploiement. | Adresse e-mail valide
Nom d’utilisateur de la machine virtuelle | Nom d’utilisateur de l’administrateur pour chaque machine virtuelle déployée | 1 à 64 caractères alphanumériques
Type d'authentification | Méthode d’authentification auprès de la machine virtuelle. | Mot de passe
Mot de passe | Mot de passe du compte administrateur pour chacune des machines virtuelles déployées. Dans un premier temps, toutes les machines virtuelles disposent du même mot de passe. Vous pouvez modifier ce mot de passe au terme de l’approvisionnement. | 12 à 72 caractères 
Abonnement | Abonnement vers lequel vous déployez le réseau de consortium. |
Groupe de ressources| Groupe de ressources sur lequel déployez le réseau de consortium. | myResourceGroup
Emplacement | Région Azure du groupe de ressources. | USA Ouest 2

Sélectionnez **OK**.

### <a name="deployment-regions"></a>Régions de déploiement

Sous *Régions de déploiement*, spécifiez le nombre de régions et d’emplacements pour chacun. Vous pouvez effectuer le déploiement dans un maximum de cinq régions. La première région doit correspondre à l’emplacement du groupe de ressources de la section *De base*. Pour les réseaux de développement ou de test, vous pouvez utiliser une seule région par membre. Pour la production, effectuez le déploiement dans au moins deux régions pour bénéficier d’une haute disponibilité.

![régions de déploiement](./media/ethereum-poa-deployment/deployment-regions.png)

Paramètre | Description | Valeur d'exemple
----------|-------------|--------------
Nombre de régions|Nombre de régions dans lesquelles déployer le réseau de consortium| 2
Première région | Première région dans laquelle déployer le réseau de consortium | USA Ouest 2
Deuxième région | Deuxième région dans laquelle déployer le réseau de consortium Les régions supplémentaires sont visibles lorsque le nombre de régions est supérieur ou égal à deux. | USA Est 2

Sélectionnez **OK**.

### <a name="network-size-and-performance"></a>Taille et performances du réseau

Sous *Taille et performances du réseau*, spécifiez des valeurs pour la taille du réseau de consortium. La taille de stockage du nœud de validateur détermine la taille potentielle de la blockchain. Cette taille peut être modifiée après le déploiement.

![Taille et performances du réseau](./media/ethereum-poa-deployment/network-size-and-performance.png)

Paramètre | Description | Valeur d'exemple
----------|-------------|--------------
Nombre de nœuds de validateur avec équilibrage de charge | Nombre de nœuds de validateur à configurer dans le réseau. | 2
Performances de stockage des nœuds de validateur | Type de disque managé pour chacun des nœuds validateurs déployés. Pour plus d'informations sur la tarification, consultez [Tarification du stockage](https://azure.microsoft.com/pricing/details/managed-disks/). | SSD Standard
Taille de machine virtuelle du nœud de validateur | Taille de machine virtuelle utilisée pour les nœuds de validateur. Pour plus d'informations sur la tarification, consultez [Tarification des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/windows/). | Standard D2 v3

La machine virtuelle et le niveau de stockage affectent les performances réseau.  Utilisez le tableau suivant pour vous aider à choisir la rentabilité :

Référence SKU de machine virtuelle|Niveau de stockage|Price|Débit|Latence
---|---|---|---|---
F1|SSD Standard|low|low|high
D2_v3|SSD Standard|moyenne|moyenne|moyenne
F16s|SSD Premium|high|high|low

Sélectionnez **OK**.

### <a name="ethereum-settings"></a>Paramètres Ethereum

Sous *Paramètres Ethereum*, spécifiez les paramètres de configuration liés à Ethereum.

![Paramètres Ethereum](./media/ethereum-poa-deployment/ethereum-settings.png)

Paramètre | Description | Valeur d'exemple
----------|-------------|--------------
ID du membre de consortium | ID associé à chaque membre participant au réseau de consortium. Il est utilisé pour configurer les espaces d’adressage IP afin d’éviter les collisions. Pour un réseau privé, l’ID de membre doit être unique dans les différentes organisations du même réseau.  Un ID de membre unique est nécessaire même si une organisation effectue des déploiements dans plusieurs régions. Notez la valeur de ce paramètre, car vous devez le partager avec les autres membres participants pour éviter tout risque de collision. La plage valide s'étend de 0 à 255. | 0
ID réseau | ID du réseau Ethereum de consortium en cours de déploiement. Chaque réseau Ethereum possède son propre ID de réseau, 1 étant l’ID du réseau public. Les valeurs valides sont comprises entre 5 et 999 999 999. | 10101010
Adresse Ethereum de l’administrateur | Adresse du compte Ethereum utilisé pour participer à la gouvernance PoA. Vous pouvez utiliser MetaMask pour générer une adresse Ethereum. |
Options avancées | Options avancées pour les paramètres Ethereum | Activer
Effectuer un déploiement à l’aide d’une adresse IP publique | En cas de sélection de Réseau virtuel privé, le réseau est déployé derrière une passerelle de réseau virtuel et supprime l’accès de peering. Pour un réseau virtuel privé, tous les membres doivent utiliser une passerelle de réseau virtuel afin de garantir la compatibilité de la connexion. | Adresse IP publique
Limite de gaz par bloc | Limite de gaz par bloc de départ du réseau | 50000000
Période de rescellage des blocs (s) | Fréquence à laquelle des blocs vides sont créés en l’absence de transaction sur le réseau. Une fréquence supérieure aboutit à une finalité plus rapide, mais avec des coûts de stockage plus élevés. | 15
Contrat d’autorisation de transaction | Bytecode du contrat d’autorisation de transaction. Restreint le déploiement et l’exécution de contrats intelligents à une liste autorisée de comptes Ethereum. |

Sélectionnez **OK**.

### <a name="monitoring"></a>Surveillance

La surveillance vous permet de configurer une ressource de journal pour votre réseau. L’agent de surveillance collecte et fournit des métriques et journaux d’activité utiles à partir de votre réseau, en offrant la possibilité de vérifier rapidement l’intégrité du réseau ou les problèmes de débogage.

![Azure Monitor](./media/ethereum-poa-deployment/azure-monitor.png)

Paramètre | Description | Valeur d'exemple
----------|-------------|--------------
Surveillance | Option permettant d’activer la surveillance | Activer
Se connecter à des journaux Azure Monitor existants | Option permettant de créer une instance de journaux Azure Monitor ou de rejoindre une instance existante | Création
Emplacement | Région dans laquelle la nouvelle instance est déployée | USA Est
ID de l’espace de travail Log Analytics existant (Se connecter à des journaux Azure Monitor existants = Rejoindre existant)|ID d’espace de travail de l’instance de journaux Azure Monitor existante||N/D
Clé primaire de l’instance Log Analytics existante (Se connecter à des journaux Azure Monitor existants = Rejoindre existant)|Clé primaire utilisée pour se connecter à l’instance de journaux Azure Monitor existante||N/D

Sélectionnez **OK**.

### <a name="summary"></a>Résumé

Cliquez sur le résumé pour passer en revue les entrées spécifiées et exécuter une validation de base avant le déploiement. Avant le déploiement, vous pouvez télécharger le modèle et les paramètres.

Sélectionnez **Créer** pour effectuer le déploiement.

Si le déploiement comprend des passerelles de réseau virtuel, il peut prendre 45 à 50 minutes.

## <a name="deployment-output"></a>Sortie du déploiement

Une fois le déploiement terminé, vous serez en mesure d’accéder aux paramètres nécessaires via le portail Azure.

### <a name="confirmation-email"></a>E-mail de confirmation

Si vous fournissez une adresse e-mail ([Section De base](#basics)), un e-mail contenant des informations de déploiement et des liens vers cette documentation est envoyé.

![e-mail de déploiement](./media/ethereum-poa-deployment/deployment-email.png)

### <a name="portal"></a>Portail

Une fois le déploiement correctement effectué et toutes les ressources configurées, vous serez en mesure de visualiser les paramètres de sortie dans votre groupe de ressources.

1. Accédez à votre groupe de ressources sur le portail.
1. Sélectionnez **Vue d’ensemble > Déploiements**.

    ![Vue d’ensemble du groupe de ressources](./media/ethereum-poa-deployment/resource-group-overview.png)

1. Sélectionnez le déploiement **microsoft-azure-blockchain.azure-blockchain-ether-...** .
1. Sélectionnez la section **Sorties**.

    ![Sorties de déploiement](./media/ethereum-poa-deployment/deployment-outputs.png)

## <a name="growing-the-consortium"></a>Développer le consortium

Pour développer votre consortium, vous devez d’abord vous connecter au réseau physique. Si vous effectuez le déploiement derrière un VPN, consultez la section [Connexion des passerelles de réseau virtuel](#connecting-vnet-gateways) pour configurer la connexion réseau dans le cadre du déploiement du nouveau membre. Une fois votre déploiement terminé, utilisez la [Governance DApp](#governance-dapp) pour devenir administrateur réseau.

### <a name="new-member-deployment"></a>Déploiement du nouveau membre

Partagez les informations suivantes avec le membre participant. Ces informations se trouvent dans l’e-mail reçu après le déploiement ou dans la sortie du déploiement via le portail.

* URL de données du consortium
* Nombre de nœuds que vous avez déployés
* ID de ressource de la passerelle de réseau virtuel (si vous utilisez un VPN)

Le membre à déployer doit utiliser le même modèle de solution de consortium de preuve d’autorité Ethereum lors du déploiement de sa présence sur le réseau et suivre les conseils ci-dessous :

* Sélectionnez **Rejoindre existant**
* Choisissez le même nombre de nœuds validateurs que le reste des membres sur le réseau afin de garantir une représentation équitable
* Utilisez la même adresse Ethereum de l’administrateur
* Utilisez *l’URL de données du consortium* fournie dans *Paramètres Ethereum*
* Si le reste du réseau se trouve derrière un VPN, sélectionnez **Réseau virtuel privé** sous la section Avancé

### <a name="connecting-vnet-gateways"></a>Connexion des passerelles de réseau virtuel

Cette section est requise uniquement si vous avez effectué un déploiement à l’aide d’un réseau virtuel privé. Vous pouvez l'ignorer si vous utilisez des adresses IP publiques.

Dans le cas d’un réseau privé, les différents membres sont connectés via des connexions de passerelle de réseau virtuel. Pour qu’un membre puisse rejoindre le réseau et voir le trafic de transaction, un membre existant doit effectuer une configuration finale sur la passerelle VPN pour accepter la connexion. Les nœuds Ethereum du nouveau membre ne fonctionneront qu’une fois la connexion établie. Pour réduire les risques d'un point de défaillance unique, créez des connexions réseau redondantes.

Une fois que le nouveau membre est déployé, le membre existant doit terminer la connexion bidirectionnelle en configurant une connexion de passerelle de réseau virtuel pour le nouveau membre. Le membre existant a besoin des éléments suivants :

* L’ID de ressource de la passerelle de réseau virtuel du membre qui se connecte. Consultez [Sortie du déploiement](#deployment-output).
* La clé de connexion partagée.

Le membre existant doit exécuter le script PowerShell suivant pour terminer la connexion. Vous pouvez utiliser Azure Cloud Shell, accessible depuis la barre de navigation se trouvant en haute à droite du portail.

![cloud shell](./media/ethereum-poa-deployment/cloud-shell.png)

```Powershell
$MyGatewayResourceId = "<EXISTING_MEMBER_RESOURCEID>"
$OtherGatewayResourceId = "<NEW_MEMBER_RESOURCEID]"
$ConnectionName = "Leader2Member"
$SharedKey = "<NEW_MEMBER_KEY>"

## $myGatewayResourceId tells me what subscription I am in, what ResourceGroup and the VNetGatewayName
$splitValue = $MyGatewayResourceId.Split('/')
$MySubscriptionid = $splitValue[2]
$MyResourceGroup = $splitValue[4]
$MyGatewayName = $splitValue[8]

## $otherGatewayResourceid tells me what the subscription and VNet GatewayName are
$OtherGatewayName = $OtherGatewayResourceId.Split('/')[8]
$Subscription=Select-AzSubscription -SubscriptionId $MySubscriptionid

## create a PSVirtualNetworkGateway instance for the gateway I want to connect to
$OtherGateway=New-Object Microsoft.Azure.Commands.Network.Models.PSVirtualNetworkGateway
$OtherGateway.Name = $OtherGatewayName
$OtherGateway.Id = $OtherGatewayResourceId
$OtherGateway.GatewayType = "Vpn"
$OtherGateway.VpnType = "RouteBased"

## get a PSVirtualNetworkGateway instance for my gateway
$MyGateway = Get-AzVirtualNetworkGateway -Name $MyGatewayName -ResourceGroupName $MyResourceGroup

## create the connection
New-AzVirtualNetworkGatewayConnection -Name $ConnectionName -ResourceGroupName $MyResourceGroup -VirtualNetworkGateway1 $MyGateway -VirtualNetworkGateway2 $OtherGateway -Location $MyGateway.Location -ConnectionType Vnet2Vnet -SharedKey $SharedKey -EnableBgp $True
```

## <a name="service-monitoring"></a>Surveillance des services

Vous pouvez localiser votre portail Azure Monitor en suivant le lien contenu dans l’e-mail de déploiement ou en recherchant le paramètre dans la sortie du déploiement [OMS_PORTAL_URL].

Le portail affiche tout d’abord les statistiques réseau de haut niveau et une vue d’ensemble des nœuds.

![Catégories monitor](./media/ethereum-poa-deployment/monitor-categories.png)

Sélectionnez **Vue d’ensemble des nœuds** pour afficher les statistiques d’infrastructure par nœud.

![Statistiques des nœuds](./media/ethereum-poa-deployment/node-stats.png)

Sélectionnez **Statistiques réseau** pour afficher les statistiques du réseau Ethereum.

![Statistiques réseau](./media/ethereum-poa-deployment/network-stats.png)

### <a name="sample-kusto-queries"></a>Exemples de requêtes Kusto

Vous pouvez interroger les journaux pour examiner les échecs ou configurer des seuils d'alerte. Les requêtes suivantes correspondent à des exemples que vous pouvez exécuter dans l’outil *Recherche dans les journaux* :

Les blocs de liste signalés par plusieurs requêtes de validateur permettent de facilement repérer les branches de chaîne.

```sql
MinedBlock_CL
| summarize DistinctMiners = dcount(BlockMiner_s) by BlockNumber_d, BlockMiner_s
| where DistinctMiners > 1
```

Obtenez le nombre moyen de pairs pour un nœud de validateur spécifié en moyenne sur des compartiments de 5 minutes.

```sql
let PeerCountRegex = @"Syncing with peers: (\d+) active, (\d+) confirmed, (\d+)";
ParityLog_CL
| where Computer == "vl-devn3lgdm-reg1000001"
| project RawData, TimeGenerated
| where RawData matches regex PeerCountRegex
| extend ActivePeers = extract(PeerCountRegex, 1, RawData, typeof(int))
| summarize avg(ActivePeers) by bin(TimeGenerated, 5m)
```

## <a name="ssh-access"></a>Accès SSH

Pour des raisons de sécurité, l’accès au port SSH est refusé par défaut par une règle de sécurité de groupe de réseau. Pour accéder aux instances des machines virtuelles dans le réseau PoA, vous devez modifier la règle de sécurité suivante en sélectionnant le paramètre *Autoriser*

1. Accédez à la section **Vue d’ensemble** du groupe de ressources déployé dans le portail Azure.

    ![vue d’ensemble ssh](./media/ethereum-poa-deployment/ssh-overview.png)

1. Sélectionnez le **groupe de sécurité réseau** pour la région de la machine virtuelle à laquelle vous souhaitez accéder.

    ![ssh nsg](./media/ethereum-poa-deployment/ssh-nsg.png)

1. Sélectionnez la règle **allow-ssh**.

    ![Capture d’écran montrant une fenêtre de vue d’ensemble de la règle ssh-allow sélectionnée.](./media/ethereum-poa-deployment/ssh-allow.png)

1. Cliquez sur **Autoriser** sous **Action**.

    ![ssh enable allow](./media/ethereum-poa-deployment/ssh-enable-allow.png)

1. Sélectionnez **Enregistrer**. L’application des modifications peut prendre quelques minutes.

Vous pouvez vous connecter à distance aux machines virtuelles des nœuds de validateur via SSH avec le nom d’utilisateur administrateur et le mot de passe/la clé SSH fournis. La commande SSH permettant d’accéder au premier nœud de validateur est répertoriée dans la sortie de déploiement de modèle. Par exemple :

``` bash
ssh -p 4000 poaadmin\@leader4vb.eastus.cloudapp.azure.com.
```

Pour obtenir des nœuds de transaction supplémentaires, incrémentez le numéro de port d’une unité.

Si vous avez déployé sur plusieurs régions, modifiez la commande en utilisant le nom DNS ou l’adresse IP de l’équilibreur de charge dans cette région. Pour trouver le nom DNS ou l’adresse IP des autres régions, recherchez la ressource avec la convention d’affectation de noms **\*\*\*\*\*-lbpip-reg\#** et affichez les propriétés de son nom DNS et de son adresse IP.

## <a name="azure-traffic-manager-load-balancing"></a>Équilibrage de charge dans Azure Traffic Manager

Azure Traffic Manager vous permet de réduire les temps d’arrêt et d’améliorer la réactivité du réseau PoA en acheminant le trafic entrant entre plusieurs déploiements dans différentes régions. Les contrôles d’intégrité intégrés et le réacheminement automatique garantissent une haute disponibilité des points de terminaison RPC et de la Governance DApp. Cette fonctionnalité est utile si vous avez déployé dans plusieurs régions et que vous êtes prêt pour la production.

Utilisez Traffic Manager pour améliorer la disponibilité du réseau PoA avec le basculement automatique. Vous pouvez également utiliser Traffic Manager pour augmenter la réactivité de vos réseaux en acheminant les utilisateurs finaux vers l’emplacement Azure avec un minimum de latence réseau.

Si vous décidez de créer un profil Traffic Manager, vous pouvez utiliser le nom DNS du profil pour accéder à votre réseau. Une fois les autres membres du consortium ajoutés au réseau, vous pouvez également utiliser Traffic Manager pour équilibrer la charge entre les validateurs qui ont été déployés.

### <a name="creating-a-traffic-manager-profile"></a>Création d’un profil Traffic Manager

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** en haut à gauche.
1. Recherchez **Profil Traffic Manager**.

    ![Rechercher Azure Traffic Manager](./media/ethereum-poa-deployment/traffic-manager-search.png)

    Donnez un nom unique au profil et sélectionnez le groupe de ressources utilisé pour le déploiement de PoA.

1. Sélectionnez **Créer** pour effectuer le déploiement.

    ![Créer Traffic Manager](./media/ethereum-poa-deployment/traffic-manager-create.png)

1. Une fois déployé, sélectionnez l’instance dans le groupe de ressources. Le nom DNS permettant d’accéder à Traffic Manager se trouve sous l’onglet Vue d’ensemble

    ![Rechercher le DNS Traffic Manager](./media/ethereum-poa-deployment/traffic-manager-dns.png)

1. Sélectionnez l’onglet **Points de terminaison** et cliquez sur le bouton **Ajouter**.
1. Donnez un nom unique au point de terminaison.
1. Pour **Type de ressource cible**, sélectionnez **Adresse IP publique**.
1. Sélectionnez l’adresse IP publique de l'équilibreur de charge de la première région.

    ![acheminement de traffic manager](./media/ethereum-poa-deployment/traffic-manager-routing.png)

Répétez l’opération pour chaque région dans le réseau déployé. Une fois les points de terminaison à l’état **activé**, leur charge et leur région sont automatiquement équilibrées sur le nom DNS de Traffic Manager. Vous pouvez maintenant utiliser ce nom DNS à la place du paramètre [CONSORTIUM_DATA_URL] dans les autres étapes de cet article.

## <a name="data-api"></a>API de données

Chaque membre du consortium héberge les informations nécessaires pour que d’autres utilisateurs puissent se connecter au réseau. Pour faciliter la connectivité, chaque membre héberge un ensemble d’informations de connexion sur le point de terminaison de l’API de données.

Le membre existant fournit le paramètre [CONSORTIUM_DATA_URL] avant le déploiement du membre. Lors du déploiement, un membre rejoignant le consortium récupérera les informations à partir de l’interface JSON au point de terminaison suivant :

`<CONSORTIUM_DATA_URL>/networkinfo`

La réponse contient des informations utiles pour joindre de membres (bloc Genesis, ABI de contrat de l’ensemble de validateurs, bootnodes), ainsi que des informations utiles pour le membre existant (adresses des validateurs). Vous pouvez utiliser cette standardisation pour étendre le consortium à travers les fournisseurs de services cloud. Cette API renvoie une réponse au format JSON avec la structure suivante :

```json
{
  "$id": "",
  "type": "object",
  "definitions": {},
  "$schema": "https://json-schema.org/draft-07/schema#",
  "properties": {
    "majorVersion": {
      "$id": "/properties/majorVersion",
      "type": "integer",
      "title": "This schema’s major version",
      "default": 0,
      "examples": [
        0
      ]
    },
    "minorVersion": {
      "$id": "/properties/minorVersion",
      "type": "integer",
      "title": "This schema’s minor version",
      "default": 0,
      "examples": [
        0
      ]
    },
    "bootnodes": {
      "$id": "/properties/bootnodes",
      "type": "array",
      "items": {
        "$id": "/properties/bootnodes/items",
        "type": "string",
        "title": "This member’s bootnodes",
        "default": "",
        "examples": [
          "enode://a348586f0fb0516c19de75bf54ca930a08f1594b7202020810b72c5f8d90635189d72d8b96f306f08761d576836a6bfce112cfb6ae6a3330588260f79a3d0ecb@10.1.17.5:30300",
          "enode://2d8474289af0bb38e3600a7a481734b2ab19d4eaf719f698fe885fb239f5d33faf217a860b170e2763b67c2f18d91c41272de37ac67386f80d1de57a3d58ddf2@10.1.17.4:30300"
        ]
      }
    },
    "valSetContract": {
      "$id": "/properties/valSetContract",
      "type": "string",
      "title": "The ValidatorSet Contract Source",
      "default": "",
      "examples": [
        "pragma solidity 0.4.21;\n\nimport \"./SafeMath.sol\";\nimport \"./Utils.sol\";\n\ncontract ValidatorSet …"
      ]
    },
    "adminContract": {
      "$id": "/properties/adminContract",
      "type": "string",
      "title": "The AdminSet Contract Source",
      "default": "",
      "examples": [
        "pragma solidity 0.4.21;\nimport \"./SafeMath.sol\";\nimport \"./SimpleValidatorSet.sol\";\nimport \"./Admin.sol\";\n\ncontract AdminValidatorSet is SimpleValidatorSet { …"
      ]
    },
    "adminContractABI": {
      "$id": "/properties/adminContractABI",
      "type": "string",
      "title": "The Admin Contract ABI",
      "default": "",
      "examples": [
        "[{\"constant\":false,\"inputs\":[{\"name\":\"proposedAdminAddress\",\"type\":\"address\"},…"
      ]
    },
    "paritySpec": {
      "$id": "/properties/paritySpec",
      "type": "string",
      "title": "The Parity client spec file",
      "default": "",
      "examples": [
        "\n{\n \"name\": \"PoA\",\n \"engine\": {\n \"authorityRound\": {\n \"params\": {\n \"stepDuration\": \"2\",\n \"validators\" : {\n \"safeContract\": \"0x0000000000000000000000000000000000000006\"\n },\n \"gasLimitBoundDivisor\": \"0x400\",\n \"maximumExtraDataSize\": \"0x2A\",\n \"minGasLimit\": \"0x2FAF080\",\n \"networkID\" : \"0x9a2112\"\n }\n }\n },\n \"params\": {\n \"gasLimitBoundDivisor\": \"0x400\",\n \"maximumExtraDataSize\": \"0x2A\",\n \"minGasLimit\": \"0x2FAF080\",\n \"networkID\" : \"0x9a2112\",\n \"wasmActivationTransition\": \"0x0\"\n },\n \"genesis\": {\n \"seal\": {\n \"authorityRound\": {\n \"step\": \"0x0\",\n \"signature\": \"0x0000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000000\"\n }\n },\n \"difficulty\": \"0x20000\",\n \"gasLimit\": \"0x2FAF080\"\n },\n \"accounts\": {\n \"0x0000000000000000000000000000000000000001\": { \"balance\": \"1\", \"builtin\": { \"name\": \"ecrecover\", \"pricing\": { \"linear\": { \"base\": 3000, \"word\": 0 } } } },\n \"0x0000000000000000000000000000000000000002\": { \"balance\": \"1\", \"builtin\": { \"name\": \"sha256\", \"pricing\": { \"linear\": { \"base\": 60, \"word\": 12 } } } },\n \"0x0000000000000000000000000000000000000003\": { \"balance\": \"1\", \"builtin\": { \"name\": \"ripemd160\", \"pricing\": { \"linear\": { \"base\": 600, \"word\": 120 } } } },\n \"0x0000000000000000000000000000000000000004\": { \"balance\": \"1\", \"builtin\": { \"name\": \"identity\", \"pricing\": { \"linear\": { \"base\": 15, \"word\": 3 } } } },\n \"0x0000000000000000000000000000000000000006\": { \"balance\": \"0\", \"constructor\" : \"…\" }\n }\n}"
      ]
    },
    "errorMessage": {
      "$id": "/properties/errorMessage",
      "type": "string",
      "title": "Error message",
      "default": "",
      "examples": [
        ""
      ]
    },
    "addressList": {
      "$id": "/properties/addressList",
      "type": "object",
      "properties": {
        "addresses": {
          "$id": "/properties/addressList/properties/addresses",
          "type": "array",
          "items": {
            "$id": "/properties/addressList/properties/addresses/items",
            "type": "string",
            "title": "This member’s validator addresses",
            "default": "",
            "examples": [
              "0x00a3cff0dccc0ecb6ae0461045e0e467cff4805f",
              "0x009ce13a7b2532cbd89b2d28cecd75f7cc8c0727"
            ]
          }
        }
      }
    }
  }
}

```

## <a name="governance-dapp"></a>Governance DApp

La décentralisation de la gouvernance est au centre du concept de preuve d’autorité. Dans la mesure où la preuve d’autorité utilise une liste autorisée d’autorités de réseau pour conserver le réseau, il est important de prévoir un mécanisme juste pour apporter des modifications à cette liste d’autorisations. Chaque déploiement est fourni avec un ensemble de contrats intelligents et avec un portail pour permettre la gouvernance de cette liste autorisée. Dès lors qu’une modification proposée atteint la majorité des voix auprès des membres du consortium, la modification est mise en œuvre. Le vote permet d’ajouter de nouveaux participants du consensus ou de supprimer les participants douteux de manière transparente afin d’encourager un réseau honnête.

La Governance DApp est un ensemble de [contrats intelligents](https://github.com/Azure-Samples/blockchain/tree/master/ledger/template/ethereum-on-azure/permissioning-contracts) prédéployés associés à une application web servant à administrer les autorités sur le réseau. Les autorités sont divisées en identités d’administrateur et en nœuds validateurs.
Les administrateurs ont le pouvoir de déléguer la participation au consensus à un ensemble de nœuds validateurs. Les administrateurs peuvent également voter l’adhésion ou l’expulsion d’autres administrateurs sur le réseau.

![Governance DApp](./media/ethereum-poa-deployment/governance-dapp.png)

* **Gouvernance décentralisée :** les modifications apportées aux autorités du réseau sont administrées par le biais d’un vote à la chaîne auprès de certains administrateurs.
* **Délégation de validateur :** les autorités peuvent gérer leurs nœuds de validateur qui sont configurés dans chaque déploiement PoA.
* **Historique des modifications vérifiable :** chaque modification est enregistrée sur la blockchain pour favoriser la transparence et la vérifiabilité.

### <a name="getting-started-with-governance"></a>Bien démarrer avec la gouvernance

Pour effectuer n’importe quel type de transactions via la Governance DApp, vous devez utiliser un portefeuille Ethereum. L’approche la plus simple consiste à utiliser un portefeuille dans le navigateur tel que [MetaMask](https://metamask.io). Cependant, s'agissant de contrats intelligents déployés sur le réseau, vous pouvez également automatiser vos interactions sur le contrat de gouvernance.

Après avoir installé MetaMask, accédez à la Governance DApp dans le navigateur.  Vous pouvez localiser l’URL via le portail Azure dans la sortie du déploiement.  Si aucun portefeuille dans le navigateur n’est installé, vous ne pourrez pas effectuer des actions ; toutefois, vous êtes toujours en mesure de lire l’état de l’administrateur.  

### <a name="becoming-an-admin"></a>Devenir un administrateur

Si vous êtes le premier membre déployé sur le réseau, vous devenez alors automatiquement un administrateur et vos nœuds de parité sont répertoriés comme validateurs. Si vous joignez le réseau, vous devez être élu en tant qu’administrateur par une majorité (plus de 50 %) de l’ensemble d’administrateurs existants. Si vous choisissez de ne pas devenir un administrateur, vos nœuds continuent de synchroniser et de valider la blockchain ; toutefois, ils ne participent pas au processus de création de blocs. Pour démarrer le processus de vote afin de devenir administrateur, cliquez sur **Nommer**, puis entrez vos adresse et alias Ethereum.

![Nommer](./media/ethereum-poa-deployment/governance-dapp-nominate.png)

### <a name="candidates"></a>Candidats

Sélectionnez l’onglet **Candidats** pour afficher l’ensemble d’administrateurs candidats.  Lorsqu'un candidat obtient la majorité des voix auprès des administrateurs actuels, il est promu administrateur.  Pour voter pour un candidat, sélectionnez la ligne, puis **Voter**. Si vous changez d’avis sur un vote, sélectionnez le candidat, puis **Rescind vote**.

![Candidats](./media/ethereum-poa-deployment/governance-dapp-candidates.png)

### <a name="admins"></a>Administrateurs

L’onglet **Administrateurs** affiche l’ensemble actuel des administrateurs et vous permet de voter contre.  Lorsqu'un administrateur perd plus de 50 % de ses soutiens, il est supprimé du réseau en tant qu’administrateur. Tous les nœuds de validateur qui appartiennent à l'administrateur perdent l’état de validateur et deviennent des nœuds de transaction sur le réseau. Un administrateur peut être supprimé pour différentes raisons ; toutefois, il incombe au consortium de se mettre d’accord sur une stratégie à l’avance.

![Administrateurs](./media/ethereum-poa-deployment/governance-dapp-admins.png)

### <a name="validators"></a>Validateurs

Sélectionnez l’onglet **Validateurs** pour afficher les nœuds de parité déployés pour cette instance et leur état (type de nœud). Chaque membre du consortium dispose d'un ensemble différent de validateurs dans cette liste, étant donné que cette vue représente le membre de consortium déployé actuel. S’il s’agit d’une instance qui vient d’être déployée et que vous n’avez pas encore ajouté vos validateurs, l’option **Ajouter des validateurs** s’affiche. L'ajout de validateurs permet de choisir automatiquement un ensemble équilibré au niveau régional de nœuds de parité et de les attribuer à votre ensemble de validateurs. Si vous avez déployé plus de nœuds que la capacité autorisée, les nœuds restants deviennent des nœuds de transaction sur le réseau.

L’adresse de chaque validateur est affectée automatiquement via le [magasin d’identités](#identity-store) dans Azure.  Si un nœud tombe en panne, il renonce à son identité, ce qui permet à un autre nœud dans votre déploiement de prendre sa place. Ce processus garantit que votre participation au consensus est hautement disponible.

![Validateurs](./media/ethereum-poa-deployment/governance-dapp-validators.png)

### <a name="consortium-name"></a>Nom du consortium

Tout administrateur peut mettre à jour le nom du consortium.  Sélectionnez l’icône d’engrenage dans la partie supérieure gauche pour mettre à jour le nom du consortium.

### <a name="account-menu"></a>Menu Compte

Dans la partie supérieure droite figurent votre alias de compte Ethereum et l’icône de votre identité.  Si vous êtes administrateur, vous avez la possibilité de mettre à jour votre alias.

![Compte](./media/ethereum-poa-deployment/governance-dapp-account.png)

## <a name="ethereum-development"></a>Développement Ethereum<a id="tutorials"></a>

Pour compiler, déployer et tester des contrats intelligents, voici quelques options à envisager à des fins de développement Ethereum :
* [Suite Truffle](https://www.trufflesuite.com/docs/truffle/overview) : environnement de développement Ethereum basé sur le client
* [Remix Ethereum](https://remix-ide.readthedocs.io/en/latest/index.html ) : environnement de développement Ethereum local basé sur un navigateur

### <a name="compile-deploy-and-execute-smart-contract"></a>Compiler, déployer et exécuter un contrat intelligent

Dans l’exemple suivant, vous créez un contrat intelligent simple. Vous utilisez Truffle pour compiler et déployer le contrat intelligent sur votre réseau blockchain. Une fois déployé, vous appelez une fonction de contrat intelligent par le biais d’une transaction.

#### <a name="prerequisites"></a>Prérequis

* Installez [Python 2.7.15](https://www.python.org/downloads/release/python-2715/). Python est requis pour Truffle et Web3. Sélectionnez l’option d’installation permettant d'inclure Python dans votre chemin d’accès.
* Installez Truffle v5.0.5 `npm install -g truffle@v5.0.5`. Truffle nécessite que plusieurs outils soient installés, notamment [Node.js](https://nodejs.org) et [Git](https://git-scm.com/). Pour plus d’informations, consultez la [documentation Truffle](https://github.com/trufflesuite/truffle).

### <a name="create-truffle-project"></a>Créer un projet Truffle

Pour pouvoir compiler et déployer un contrat intelligent, vous devez créer un projet Truffle.

1. Ouvrez une invite de commandes ou un interpréteur de commandes.
1. Créez un dossier intitulé `HelloWorld`.
1. Remplacez le répertoire par le dossier `HelloWorld`.
1. Initialisez un nouveau projet Truffle à l’aide de la commande `truffle init`.

    ![Créer un projet Truffle](./media/ethereum-poa-deployment/create-truffle-project.png)

### <a name="add-a-smart-contract"></a>Ajouter un contrat intelligent

Créez vos contrats intelligents dans le sous-répertoire **contracts** de votre projet Truffle.

1. Créez un fichier dans le sous-répertoire nommé `postBox.sol`**contracts** de votre projet Truffle.
1. Ajoutez le code Solidity suivant à **postBox.sol**.

    ```javascript
    pragma solidity ^0.5.0;
    
    contract postBox {
        string message;
        function postMsg(string memory text) public {
            message = text;
        }
        function getMsg() public view returns (string memory) {
            return message;
        }
    }
    ```

### <a name="deploy-smart-contract-using-truffle"></a>Déployer un contrat intelligent à l'aide de Truffle

Les projets Truffle contiennent un fichier de configuration pour les détails de connexion au réseau blockchain. Modifiez le fichier de configuration afin d’y inclure les informations de connexion pour votre réseau.

> [!WARNING]
> N’envoyez jamais votre clé Ethereum privée sur le réseau. Assurez-vous que chaque transaction est d’abord signée en local avant d’être envoyée sur le réseau.

1. Vous devez disposer de l’expression mnémonique correspondant au [compte d’administrateur Ethereum utilisé lors du déploiement de votre réseau blockchain](#ethereum-settings). Si vous avez utilisé MetaMask pour créer le compte, vous pouvez récupérer l'expression mnémonique depuis MetaMask. Sélectionnez l’icône de compte administrateur en haut à droite de l’extension MetaMask, puis **Paramètres > Sécurité et confidentialité > Révéler les mots initiaux**.
1. Remplacez le contenu de `truffle-config.js`dans votre projet Truffle par le contenu suivant. Remplacez l’espace réservé au point de terminaison et les valeurs mnémoniques.

    ```javascript
    const HDWalletProvider = require("truffle-hdwallet-provider");
    const rpc_endpoint = "<Ethereum RPC endpoint>";
    const mnemonic = "Twelve words you can find in MetaMask > Security & Privacy > Reveal Seed Words";

    module.exports = {
      networks: {
        development: {
          host: "localhost",
          port: 8545,
          network_id: "*" // Match any network id
        },
        poa: {
          provider: new HDWalletProvider(mnemonic, rpc_endpoint),
          network_id: 10101010,
          gasPrice : 0
        }
      }
    };
    ```

1. Étant donné que nous utilisons le fournisseur Truffle HD Wallet, installez le module dans votre projet à l’aide de la commande `npm install truffle-hdwallet-provider --save`.

Truffle utilise des scripts de migration pour déployer des contrats intelligents sur un réseau blockchain. Vous devez disposer d’un script de migration pour déployer votre nouveau contrat intelligent.

1. Ajoutez une nouvelle migration afin de déployer le nouveau contrat. Créez un fichier `2_deploy_contracts.js` dans le sous-répertoire **migrations** du projet Truffle.

    ``` javascript
    var postBox = artifacts.require("postBox");
    
    module.exports = deployer => {
        deployer.deploy(postBox);
    };
    ```

1. Effectuez le déploiement sur le réseau PoA à l’aide de la commande de migration Truffle. À l’invite de commandes, dans le répertoire du projet Truffle, exécutez :

    ```javascript
    truffle migrate --network poa
    ```

### <a name="call-a-smart-contract-function"></a>Appeler une fonction de contrat intelligent

Une fois votre contrat intelligent déployé, vous pouvez envoyer une transaction pour appeler une fonction.

1. Dans le répertoire du projet Truffle, créez un fichier nommé `sendtransaction.js`.
1. Ajoutez le contenu suivant à **sendtransaction.js**.

    ``` javascript
    var postBox = artifacts.require("postBox");
    
    module.exports = function(done) {
      console.log("Getting the deployed version of the postBox smart contract")
      postBox.deployed().then(function(instance) {
        console.log("Calling postMsg function for contract ", instance.address);
        return instance.postMsg("Hello, blockchain!");
      }).then(function(result) {
        console.log("Transaction hash: ", result.tx);
        console.log("Request complete");
        done();
      }).catch(function(e) {
        console.log(e);
        done();
      });
    };
    ```

1. Exécutez le script à l’aide de la commande d'exécution Truffle.

    ```javascript
    truffle exec sendtransaction.js --network poa
    ```

    ![Exécuter le script pour appeler la fonction par le biais de la transaction](./media/ethereum-poa-deployment/send-transaction.png)

## <a name="webassembly-wasm-support"></a>Prise en charge de WebAssembly (WASM)

La prise en charge de WebAssembly est déjà activée pour vous sur les réseaux PoA récemment déployés. Elle permet de développer des contrats intelligents dans n’importe quel langage transpilable en Web-Assembly (Rust, C, C++). Pour plus d'informations, consultez les pages suivantes : [Vue d’ensemble de WebAssembly avec Parity](https://openethereum.github.io/WebAssembly-Home.html) et [Didacticiel de Parity Tech](https://github.com/paritytech/pwasm-tutorial)

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="i-notice-there-are-many-transactions-on-the-network-that-i-didnt-send-where-are-these-coming-from"></a>Je remarque un grand nombre de transactions sur le réseau que je n'ai pas envoyées. D’où viennent-elles ?

Il est dangereux de déverrouiller [l’API personnelle](https://web3js.readthedocs.io/en/v1.2.0/web3-eth-personal.html). Les bots écoutent les comptes Ethereum déverrouillés et tentent d’en extraire des fonds. Le bot suppose que ces comptes contiennent real-ether et essaie d’être le premier à permettre l’équilibre. N’activez pas l’API personnelle sur le réseau. Il est préférable de signer au préalable les transactions manuellement à l’aide d’un portefeuille comme MetaMask ou par programmation.

### <a name="how-to-ssh-onto-a-vm"></a>Comment exécuter SSH sur une machine virtuelle ?

Le port SSH n’est pas exposé pour des raisons de sécurité. Suivez [ce guide pour activer le port SSH](#ssh-access).

### <a name="how-do-i-set-up-an-audit-member-or-transaction-nodes"></a>Comment configurer un membre d’audit ou des nœuds de transaction ?

Les nœuds de transaction sont un ensemble de clients Parity homologués avec le réseau, mais qui ne participent pas à un consensus. Ces nœuds peuvent toujours être utilisés pour soumettre des transactions Ethereum et lire l’état du contrat intelligent. Ils fonctionnent bien comme mécanisme de vérifiabilité pour les membres du consortium ne faisant pas autorité sur le réseau. Pour ce faire, suivez les étapes décrites dans [Développer le consortium](#growing-the-consortium).

### <a name="why-are-metamask-transactions-taking-a-long-time"></a>Pourquoi les transactions MetaMask prennent-elles du temps ?

Pour s’assurer que les transactions sont reçues dans le bon ordre, chaque transaction Ethereum est fournie avec un nonce incrémentiel. Si vous avez utilisé un compte dans MetaMask sur un réseau différent, vous devez réinitialiser la valeur du nonce. Cliquez sur l’icône Paramètres (trois barres), Paramètres, Réinitialiser le compte. L’historique des transactions sera effacé et vous pourrez désormais soumettre à nouveau la transaction.

### <a name="do-i-need-to-specify-gas-fee-in-metamask"></a>Dois-je indiquer les frais de gaz dans MetaMask ?

L’Ether n’a aucune utilité dans Proof-of-Authority Consortium. Par conséquent, il est inutile de spécifier les frais de gaz lors de l’envoi des transactions dans MetaMask.

### <a name="what-should-i-do-if-my-deployment-fails-due-to-failure-to-provision-azure-oms"></a>Que dois-je faire si mon déploiement échoue en raison de l’échec de provisionnement d’Azure OMS ?

La supervision est une fonctionnalité facultative. Dans certains cas rares où votre déploiement échoue en raison de l’incapacité à provisionner correctement une ressource Azure Monitor, vous pouvez redéployer sans Azure Monitor.

### <a name="are-public-ip-deployments-compatible-with-private-network-deployments"></a>Les déploiements d’adresses IP publiques sont-ils compatibles avec les déploiements de réseau privé ?

Non. Le peering exige une communication bidirectionnelle, ce qui signifie que l’ensemble du réseau doit être public ou privé.

### <a name="what-is-the-expected-transaction-throughput-of-proof-of-authority"></a>Quel est le débit des transactions attendu de Proof-of-Authority ?

Le débit des transactions dépend fortement des types de transactions et de la topologie du réseau. À l’aide de transactions simples, nous avons testé une moyenne de 400 transactions par seconde avec un réseau déployé dans plusieurs régions.

### <a name="how-do-i-subscribe-to-smart-contract-events"></a>Comment s’abonner aux événements de contrats intelligents ?

Ethereum Proof-of-Authority prend désormais en charge les WebSockets.  Vérifiez votre sortie de déploiement pour localiser l’URL et le port du WebSocket.

## <a name="support-and-feedback"></a>Support et commentaires

Pour vous tenir informé sur Azure Blockchain, visitez le [blog Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/). Vous y trouverez les toutes dernières offres du service Blockchain ainsi que diverses informations de l’équipe technique d’Azure Blockchain.

Pour faire des commentaires sur le produit ou suggérer de nouvelles fonctionnalités, postez votre idée ou votez pour une autre idée sur le [forum de commentaires Azure pour Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Support de la communauté pour les objets blob

Communiquez avec les ingénieurs Microsoft et les experts de la communauté Azure Blockchain.

* [Page de questions Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html). Le support d’ingénierie pour les modèles blockchain est limité aux problèmes de déploiement.
* [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
* [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les solutions Azure Blockchain, consultez la [documentation Azure Blockchain](../index.yml).

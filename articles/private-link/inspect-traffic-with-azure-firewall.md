---
title: Utiliser Pare-feu Azure pour inspecter le trafic destiné à un point de terminaison privé
titleSuffix: Azure Private Link
description: Découvrez comment vous pouvez inspecter le trafic destiné à un point de terminaison privé à l’aide de Pare-feu Azure.
services: private-link
author: jocortems
ms.service: private-link
ms.topic: how-to
ms.date: 09/02/2020
ms.author: allensu
ms.openlocfilehash: 5cbfd90ca65a1fb75c9cbe5602ac2a69741e378f
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96017234"
---
# <a name="use-azure-firewall-to-inspect-traffic-destined-to-a-private-endpoint"></a>Utiliser Pare-feu Azure pour inspecter le trafic destiné à un point de terminaison privé

Le point de terminaison privé Azure est le composant fondamental d’Azure Private Link. Les points de terminaison privés permettent aux ressources Azure déployées dans un réseau virtuel de communiquer en privé avec des ressources de liaisons privées.

Les points de terminaison privés permettent aux ressources d’accéder au service de liaison privée déployé dans un réseau virtuel. L’accès au point de terminaison privé via l’appairage de réseaux virtuels et les connexions réseau locales permettent d’étendre la connectivité.

Vous devrez peut-être inspecter ou bloquer le trafic des clients vers les services exposés via des points de terminaison privés. Effectuez cette inspection à l’aide de [Pare-feu Azure](../firewall/overview.md) ou d’une appliance virtuelle réseau tierce.

Les limites suivantes s'appliquent :

* Les groupes de sécurité réseau (NSG) ne s’appliquent pas aux points de terminaison privés
* Les itinéraires définis par l’utilisateur (UDR) ne s’appliquent pas aux points de terminaison privés
* Une table de routage unique peut être attachée à un sous-réseau
* Une table de routage prend en charge jusqu’à 400 itinéraires

Pare-feu Azure filtre le trafic à l’aide des éléments suivants :

* [Nom de domaine complet dans les règles de réseau](../firewall/fqdn-filtering-network-rules.md) pour les protocoles TCP et UDP
* [Nom de domaine complet dans les règles d’application](../firewall/features.md#application-fqdn-filtering-rules) pour HTTP, HTTPS et MSSQL. 

La plupart des services exposés sur des points de terminaison privés utilisent le protocole HTTPS. L’utilisation de règles d’application par-dessus des règles de réseau est recommandée lors de l’utilisation d’Azure SQL.

> [!NOTE]
> Le filtrage FQDN SQL est pris en charge uniquement en [mode proxy](../azure-sql/database/connectivity-architecture.md#connection-policy) (port 1433). Le mode **proxy** peut entraîner une plus grande latence par rapport au mode de *redirection*. Si vous souhaitez continuer à utiliser le mode de redirection, qui est le mode par défaut pour les clients se connectant dans Azure, vous pouvez filtrer l’accès en utilisant le nom de domaine complet dans les règles de réseau du pare-feu.

## <a name="scenario-1-hub-and-spoke-architecture---dedicated-virtual-network-for-private-endpoints"></a>Scénario 1 : Architecture hub-and-spoke – Réseau virtuel dédié pour les points de terminaison privés

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/hub-and-spoke.png" alt-text="Réseau virtuel dédié pour les points de terminaison privés" border="true":::

Ce scénario est l’architecture la plus extensible pour se connecter en privé à plusieurs services Azure à l’aide de points de terminaison privés. Un itinéraire pointant vers l’espace d’adressage du réseau où sont déployés les points de terminaison privés est créé. Cette configuration réduit les frais administratifs et évite d’atteindre la limite des 400 itinéraires.

Si les réseaux virtuels sont appairés, les connexions d’un réseau virtuel client au pare-feu Azure dans un réseau virtuel hub sont facturées.

Pour plus d’informations sur les frais liés aux connexions à l’aide de réseaux virtuels appairés, consultez la section FAQ de la page [Tarification](https://azure.microsoft.com/pricing/details/private-link/).

>[!NOTE]
> Ce scénario peut être implémenté à l’aide de règles de réseau NVA ou Pare-feu Azure tiers, au lieu de règles d’application.

## <a name="scenario-2-hub-and-spoke-architecture---shared-virtual-network-for-private-endpoints-and-virtual-machines"></a>Scénario 2 : Architecture hub-and-spoke – Réseau virtuel partagé pour les points de terminaison privés et les machines virtuelles

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/shared-spoke.png" alt-text="Points de terminaison privés et machines virtuelles dans le même réseau virtuel" border="true":::

Ce scénario est implémenté dans les cas suivants :

* Il est impossible d’avoir un réseau virtuel dédié pour les points de terminaison privés

* Seuls quelques services sont exposés dans le réseau virtuel à l’aide de points de terminaison privés

Les machines virtuelles ont des itinéraires système /32 qui pointent vers chaque point de terminaison privé. Un itinéraire par point de terminaison privé est configuré pour acheminer le trafic via Pare-feu Azure. 

Les frais administratifs liés au maintien de la table de routage augmentent à mesure que les services sont exposés dans le réseau virtuel. La possibilité d’atteindre la limite d’itinéraires augmente également.

Selon votre architecture globale, il est possible d’atteindre la limite des 400 itinéraires. Nous vous recommandons d’utiliser le scénario 1 dans la mesure du possible.

Si les réseaux virtuels sont appairés, les connexions d’un réseau virtuel client au pare-feu Azure dans un réseau virtuel hub sont facturées.

Pour plus d’informations sur les frais liés aux connexions à l’aide de réseaux virtuels appairés, consultez la section FAQ de la page [Tarification](https://azure.microsoft.com/pricing/details/private-link/).

>[!NOTE]
> Ce scénario peut être implémenté à l’aide de règles de réseau NVA ou Pare-feu Azure tiers, au lieu de règles d’application.

## <a name="scenario-3-single-virtual-network"></a>Scénario 3 : Réseau virtuel unique

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/single-vnet.png" alt-text="Réseau virtuel unique" border="true":::

La mise en œuvre présente certaines limites : une migration vers une architecture hub-and-spoke n’est pas possible. Les mêmes considérations que celles du scénario 2 s’appliquent. Dans ce scénario, les frais d’appairage de réseaux virtuels ne s’appliquent pas.

>[!NOTE]
> Si vous souhaitez implémenter ce scénario à l’aide d’une NVA ou d’un pare-feu Azure tiers, des règles de réseau sont requises pour le trafic SNAT destiné aux points de terminaison privés, au lieu des règles d’application. Dans le cas contraire, la communication entre les machines virtuelles et les points de terminaison privés échoue.

## <a name="scenario-4-on-premises-traffic-to-private-endpoints"></a>Scénario 4 : Trafic local vers des points de terminaison privés

:::image type="content" source="./media/inspect-traffic-using-azure-firewall/on-premises.png" alt-text="Trafic local vers des points de terminaison privés" border="true":::

Vous pouvez implémenter cette architecture si vous avez configuré la connectivité avec votre réseau local à l’aide de l’une des deux méthodes suivantes : 

* [ExpressRoute](..\expressroute\expressroute-introduction.md)
* [VPN site à site](..\vpn-gateway\vpn-gateway-howto-site-to-site-resource-manager-portal.md) 

Si vos besoins en matière de sécurité exigent que le trafic client vers les services exposés via des points de terminaison privés soit acheminé par une appliance de sécurité, déployez ce scénario.

Les mêmes considérations que celles du scénario 2 ci-dessus s’appliquent. Dans ce scénario, il n’y a pas de frais d’appairage de réseaux virtuels. Pour plus d’informations sur la manière de configurer vos serveurs DNS afin de permettre aux charges de travail locales d’accéder à des points de terminaison privés, consultez [Charges de travail locales à l’aide d’un redirecteur DNS](./private-endpoint-dns.md#on-premises-workloads-using-a-dns-forwarder).

>[!NOTE]
> Si vous souhaitez implémenter ce scénario à l’aide d’une NVA ou d’un pare-feu Azure tiers, des règles de réseau sont requises pour le trafic SNAT destiné aux points de terminaison privés, au lieu des règles d’application. Dans le cas contraire, la communication entre les machines virtuelles et les points de terminaison privés échoue.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure.
* Un espace de travail Log Analytics.  

Pour créer un espace de travail si vous n’en avez pas dans votre abonnement, consultez [Créer un espace de travail Log Analytics dans le portail Azure](../azure-monitor/learn/quick-create-workspace.md).


## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-a-vm"></a>Créer une machine virtuelle

Dans cette section, vous allez créer un réseau virtuel et son sous-réseau pour héberger la machine virtuelle utilisée pour accéder à votre ressource de liaison privée. Une base de données Azure SQL est utilisée comme exemple de service.

## <a name="virtual-networks-and-parameters"></a>Réseaux virtuels et paramètres

Créez trois réseaux virtuels et leurs sous-réseaux correspondants pour :

* Contenir le pare-feu Azure utilisé pour restreindre la communication entre la machine virtuelle et le point de terminaison privé.
* Héberger la machine virtuelle qui est utilisée pour accéder à votre ressource de liaison privée.
* Héberger le point de terminaison privé.

Dans les étapes suivantes, remplacez les paramètres du tableau ci-dessous par la valeur indiquée correspondante :

### <a name="azure-firewall-network"></a>Réseau de Pare-feu Azure
| Paramètre                   | Valeur                 |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myAzFwVNet          |
| **\<region-name>**          | États-Unis - partie centrale méridionale      |
| **\<IPv4-address-space>**   | 10.0.0.0/16          |
| **\<subnet-name>**          | AzureFirewallSubnet        |
| **\<subnet-address-range>** | 10.0.0.0/24          |

### <a name="virtual-machine-network"></a>Réseau de machines virtuelles
| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myVMVNet          |
| **\<region-name>**          | États-Unis - partie centrale méridionale      |
| **\<IPv4-address-space>**   | 10.1.0.0/16          |
| **\<subnet-name>**          | VMSubnet      |
| **\<subnet-address-range>** | 10.1.0.0/24          |

### <a name="private-endpoint-network"></a>Réseau de point de terminaison privé
| Paramètre                   | Valeur                 |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroup |
| **\<virtual-network-name>** | myPEVNet         |
| **\<region-name>**          | États-Unis - partie centrale méridionale      |
| **\<IPv4-address-space>**   | 10.2.0.0/16          |
| **\<subnet-name>**          | PrivateEndpointSubnet    |        |
| **\<subnet-address-range>** | 10.2.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

10. Répétez les étapes 1 à 9 pour créer les réseaux virtuels permettant d’héberger la machine virtuelle et les ressources de point de terminaison privés.

### <a name="create-virtual-machine"></a>Créer une machine virtuelle

1. En haut à gauche de l’écran du portail Azure, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**.

2. Dans **Créer une machine virtuelle - Notions de base**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**. Vous avez créé ce groupe de ressources dans la section précédente.  |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **(États-Unis) USA Centre Sud**. |
    | Options de disponibilité | Conservez la valeur par défaut **Aucune redondance d’infrastructure nécessaire**. |
    | Image | Sélectionnez **Ubuntu Server 18.04 LTS – Gen1**. |
    | Taille | Sélectionnez **Standard_B2s**. |
    | **Compte administrateur** |  |
    | Type d'authentification | Sélectionnez **Mot de passe**. |
    | Nom d’utilisateur | Entrez un nom d’utilisateur de votre choix. |
    | Mot de passe | Entrez un mot de passe de votre choix. Le mot de passe doit contenir au moins 12 caractères et satisfaire aux [exigences de complexité définies](../virtual-machines/linux/faq.md?toc=%2fazure%2fvirtual-network%2ftoc.json#what-are-the-password-requirements-when-creating-a-vm).|
    | Confirmer le mot de passe | Retapez le mot de passe. |
    | **Règles des ports d’entrée** |  |
    | Aucun port d’entrée public | Sélectionnez **Aucun**. |
    |||

3. Sélectionnez **Suivant : Disques**.

4. Dans **Créer une machine virtuelle - Disks**, conservez les valeurs par défaut et sélectionnez **Suivant : Mise en réseau**.

5. Dans **Créer une machine virtuelle - Mise en réseau**, sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Réseau virtuel | Sélectionnez **myVMVNet**.  |
    | Subnet | Sélectionnez **VMSubnet (10.1.0.0/24)** .|
    | Adresse IP publique | Conservez la valeur par défaut **(new) myVm-ip**. |
    | Aucun port d’entrée public | Sélectionnez **Autoriser les ports sélectionnés**. |
    | Sélectionner des ports d’entrée | Sélectionnez **SSH**.|
    ||

6. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Vérifier + créer** où Azure valide votre configuration.

7. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

## <a name="deploy-the-firewall"></a>Déployer le pare-feu

1. Dans le menu du Portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

2. Tapez **pare-feu** dans la zone de recherche, puis appuyez sur **Entrée**.

3. Sélectionnez **Pare-feu**, puis **Créer**.

4. Sur la page **Créer un pare-feu**, utilisez le tableau suivant pour configurer le pare-feu :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**.  |
    | **Détails de l’instance** |  |
    | Nom | Entrez **myAzureFirewall**. |
    | Région | Sélectionnez **USA Centre Sud**. |
    | Zone de disponibilité | Conservez la valeur par défaut **Aucun**. |
    | Choisir un réseau virtuel    |    Sélectionnez **Utiliser l’existant**.    |
    | Réseau virtuel    |    Sélectionnez **myAzFwVNet**.    |
    | Adresse IP publique    |    Sélectionnez **Ajouter nouveau** et, dans Nom, entrez **myFirewall-ip**.    |
    | Tunneling forcé    | Conservez la valeur **Désactivé** par défaut.    |
    |||
5. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Vérifier + créer** où Azure valide votre configuration.

6. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

## <a name="enable-firewall-logs"></a>Activer les journaux de pare-feu

Dans cette section, vous allez activer les journaux sur le pare-feu.

1. Dans le portail Azure, sélectionnez **Toutes les ressources** dans le menu de gauche.

2. Sélectionnez le pare-feu **myAzureFirewall** dans la liste des ressources.

3. Sous **Supervision** dans les paramètres du pare-feu, sélectionnez **Paramètres de diagnostic**.

4. Dans les paramètres de diagnostic, sélectionnez **+ Ajouter un paramètre de diagnostic**.

5. Dans **Paramètres de diagnostic**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom du paramètre de diagnostic | Entrez **myDiagSetting**. |
    | Détails de la catégorie | |
    | log | Sélectionnez **AzureFirewallApplicationRule** et **AzureFirewallNetworkRule**. |
    | Détails de la destination | Sélectionnez **Envoyer à Log Analytics**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Espace de travail Log Analytics | Sélectionnez votre espace de travail Log Analytics. |

6. Sélectionnez **Enregistrer**.

## <a name="create-azure-sql-database"></a>Créer une base de données Azure SQL Database

Dans cette section, vous créez une base de données SQL privée.

1. En haut à gauche de l’écran dans le portail Azure, sélectionnez **Créer une ressource** > **Bases de données** > **Base de données SQL**.

2. Dans **Créer une base de données SQL – Concepts de base**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**. Vous avez créé ce groupe de ressources dans la section précédente.|
    | **Détails de la base de données** |  |
    | Nom de la base de données  | Entrez **mydatabase**.  |
    | Serveur | Sélectionnez **Créer**, puis entrez les informations ci-dessous.    |
    | Nom du serveur | Entrez **mydbserver**. Si ce nom est utilisé, entrez un nom unique.   |
    | Connexion d’administrateur serveur | Entrez un nom de votre choix. |
    | Mot de passe    |    Entrez un mot de passe de votre choix.    |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe    |
    | Emplacement    | Sélectionnez **(États-Unis) USA Centre Sud**.    |
    | Vous souhaitez utiliser un pool élastique SQL ?    | Conservez la valeur par défaut **Non**. |
    | Calcul + stockage | Laissez la valeur par défaut **Usage général Gen5, 2 vCores, 32 Go de stockage**. |
    |||

3. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Vérifier + créer** où Azure valide votre configuration.

4. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

## <a name="create-private-endpoint"></a>Créer un point de terminaison privé

Dans cette section, vous allez créer un point de terminaison privé pour la base de données Azure SQL de la section précédente.

1. Dans le portail Azure, sélectionnez **Toutes les ressources** dans le menu de gauche.

2. Sélectionnez le serveur Azure SQL **mydbserver** dans la liste des services.  Si vous avez utilisé un autre nom de serveur, choisissez ce nom.

3. Dans les paramètres du serveur, sélectionnez **Connexions de points de terminaison privés** sous **Sécurité**.

4. Sélectionnez **+ Point de terminaison privé**.

5. Dans **Créer un point de terminaison privé**, entrez ou sélectionnez ces informations dans l’onglet **Général** :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**. |
    | **Détails de l’instance** | |
    | Nom | Entrez **SQLPrivateEndpoint**. |
    | Région | Sélectionnez **(États-Unis) USA Centre Sud**. |

6. Sélectionnez l’onglet **Ressource**, ou sélectionnez **Suivant : Ressource** en bas de la page.

7. Dans l’onglet **Ressource**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Méthode de connexion | Sélectionnez **Se connecter à une ressource Azure dans mon répertoire**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Type de ressource | Sélectionnez **Microsoft.Sql/servers**. |
    | Ressource | Sélectionnez **mydbserver** ou le nom du serveur que vous avez créé à l’étape précédente.
    | Sous-ressource cible | Sélectionnez **sqlServer**. |

8. Sélectionnez l’onglet **Configuration** ou sélectionnez **Suivant : Configuration** en bas de la page.

9. Dans l’onglet **Configuration**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Mise en réseau** | |
    | Réseau virtuel | Sélectionnez **myPEVnet**. |
    | Subnet | Sélectionnez **PrivateEndpointSubnet**. |
    | **Intégration à un DNS privé** | |
    | Intégrer à une zone DNS privée | Sélectionnez **Oui**. |
    | Abonnement | Sélectionnez votre abonnement. |
    | Zones DNS privées | Laissez la valeur par défaut **privatelink.database.windows.net**. |

10. Sélectionnez l’onglet **Vérifier + créer** ou sélectionnez **Vérifier + créer** en bas de la page .

11. Sélectionnez **Create** (Créer).

12. Une fois le point de terminaison créé, sélectionnez **Pare-feu et réseaux virtuels** sous **Sécurité**.

13. Dans **Pare-feu et réseaux virtuels**, sélectionnez **Oui** à côté de l’option **Autoriser les services et les ressources Azure à accéder à ce serveur**.

14. Sélectionnez **Enregistrer**.

## <a name="connect-the-virtual-networks-using-virtual-network-peering"></a>Connecter les réseaux virtuels à l’aide de l’appairage de réseaux virtuels

Dans cette section, nous allons connecter les réseaux virtuels **myVMVNet** et **myPEVNet** à **myAzFwVNet** à l’aide du Peering. Il n’y aura pas de connectivité directe entre **myVMVNet** et **myPEVNet**.

1. Dans la barre de recherche du portail, entrez **myAzFwVNet**.

2. Sélectionnez **Peerings** sous le menu **Paramètres** et sélectionnez **+ Ajouter**.

3. Dans **Ajouter un Peering**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom du Peering de myAzFwVNet à un réseau virtuel distant | Entrez **myAzFwVNet-to-myVMVNet**. |
    | **Détails du Peering** |  |
    | Modèle de déploiement de réseau virtuel  | Conservez la valeur **Resource Manager** par défaut.  |
    | Je connais mon ID de ressource | Laissez la case non activée.    |
    | Abonnement | Sélectionnez votre abonnement.    |
    | Réseau virtuel | Sélectionnez **myVMVNet**. |
    | Nom du Peering du réseau virtuel distant à myAzFwVNet    |    Entrez **myVMVNet-to-myAzFwVNet**.    |
    | **Configuration** | |
    | **Configurer les paramètres d’accès au réseau virtuel** | |
    | Autoriser l’accès au réseau virtuel de myAzFwVNet vers un réseau virtuel distant | Conservez la valeur par défaut **Activé**.    |
    | Autoriser l’accès au réseau virtuel du réseau virtuel distant vers myAzFwVNet    | Conservez la valeur par défaut **Activé**.    |
    | **Configurer les paramètres du trafic transféré** | |
    | Autoriser le trafic transféré du réseau virtuel distant à myAzFwVNet    | Sélectionnez **Enabled**. |
    | Autoriser le trafic transféré de myAzFwVNet à un réseau virtuel distant | Sélectionnez **Enabled**. |
    | **Configurer les paramètres de transit par passerelle** | |
    | Autoriser le transit par passerelle | Laissez la case non activée. |
    |||

4. Sélectionnez **OK**.

5. Sélectionnez **Ajouter**.

6. Dans **Ajouter un Peering**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom du Peering de myAzFwVNet à un réseau virtuel distant | Entrez **myAzFwVNet-to-myPEVNet**. |
    | **Détails du Peering** |  |
    | Modèle de déploiement de réseau virtuel  | Conservez la valeur **Resource Manager** par défaut.  |
    | Je connais mon ID de ressource | Laissez la case non activée.    |
    | Abonnement | Sélectionnez votre abonnement.    |
    | Réseau virtuel | Sélectionnez **myPEVNet**. |
    | Nom du Peering du réseau virtuel distant à myAzFwVNet    |    Entrez **myPEVNet-to-myAzFwVNet**.    |
    | **Configuration** | |
    | **Configurer les paramètres d’accès au réseau virtuel** | |
    | Autoriser l’accès au réseau virtuel de myAzFwVNet vers un réseau virtuel distant | Conservez la valeur par défaut **Activé**.    |
    | Autoriser l’accès au réseau virtuel du réseau virtuel distant vers myAzFwVNet    | Conservez la valeur par défaut **Activé**.    |
    | **Configurer les paramètres du trafic transféré** | |
    | Autoriser le trafic transféré du réseau virtuel distant à myAzFwVNet    | Sélectionnez **Enabled**. |
    | Autoriser le trafic transféré de myAzFwVNet à un réseau virtuel distant | Sélectionnez **Enabled**. |
    | **Configurer les paramètres de transit par passerelle** | |
    | Autoriser le transit par passerelle | Laissez la case non activée. |

7. Sélectionnez **OK**.

## <a name="link-the-virtual-networks-to-the-private-dns-zone"></a>Lier les réseaux virtuels à la zone DNS privée

Dans cette section, nous allons lier les réseaux virtuels **myVMVNet** et **myAzFwVNet** à la zone DNS privée **privatelink.database.windows.net**. Cette zone a été créée lors de la création du point de terminaison privé. 

La liaison est requise pour que la machine virtuelle et le pare-feu résolvent le nom de domaine complet de la base de données à son adresse de point de terminaison privé. Le réseau virtuel **myPEVNet** a été lié automatiquement lors de la création du point de terminaison privé.

>[!NOTE]
>Si vous ne liez pas les réseaux virtuels de la machine virtuelle et du pare-feu à la zone DNS privée, la machine virtuelle et le pare-feu pourront toujours résoudre le nom de domaine complet du serveur SQL. Ils le résoudront à son IP publique.

1. Dans la barre de recherche du portail, entrez **privatelink.database**.

2. Sélectionnez **privatelink.database.windows.net** dans les résultats de la recherche.

3. Sous **Paramètres**, sélectionnez **Liens de réseau virtuel**.

4. Sélectionnez **+ Ajouter**

5. Dans **Ajouter une liaison de réseau virtuel**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de la liaison | Entrez **Link-to-myVMVNet**. |
    | **Détails du réseau virtuel** |  |
    | Je connais l’ID de ressource du réseau virtuel  | Laissez la case non activée.  |
    | Abonnement | Sélectionnez votre abonnement.    |
    | Réseau virtuel | Sélectionnez **myVMVNet**. |
    | **CONFIGURATION** | |
    | Activer l’inscription automatique | Laissez la case non activée.    |


6. Sélectionnez **OK**.

## <a name="configure-an-application-rule-with-sql-fqdn-in-azure-firewall"></a>Configurer une règle d’application avec le nom de domaine complet SQL dans Pare-feu Azure

Dans cette section, configurez une règle d’application pour autoriser la communication entre **myVM** et le point de terminaison privé pour le serveur SQL **mydbserver.database.windows.net**. 

Cette règle autorise la communication via le pare-feu que nous avons créé dans les étapes précédentes.

1. Dans la barre de recherche du portail, entrez **myAzureFirewall**.

2. Sélectionnez **myAzureFirewall** dans les résultats de la recherche.

3. Sélectionnez **Règles** sous **Paramètres** dans la vue d’ensemble **myAzureFirewall**.

4. Sélectionnez l’onglet **Collection de règles d’application**.

5. Sélectionnez **+ Ajouter une collection de règles d’application**.

6. Dans **Ajouter une collection de règles d’application**, entrez ou sélectionnez les informations suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **SQLPrivateEndpoint**. |
    | Priorité | Entrez **100**. |
    | Action | Entrez **Allow**. |
    | **Règles** |  |
    | **Balises FQDN** | |
    | Nom  | Laisser vide.  |
    | Type de source | Entrez l’**adresse IP** par défaut.    |
    | Source | Laisser vide. |
    | Balises FQDN | Conservez la valeur **0 selected** par défaut. |
    | **Noms de domaine complets cibles** | |
    | Nom | Entrez **SQLPrivateEndpoint**.    |
    | Type de source | Entrez l’**adresse IP** par défaut. |
    | Source | Entrez **10.1.0.0/16**. |
    | Protocole : Port | Entrez **mssql:1433**. |
    | Noms de domaine complets cibles | Entrez **mydbserver.database.windows.net**. |
    |||

7. Sélectionnez **Ajouter**.

## <a name="route-traffic-between-the-virtual-machine-and-private-endpoint-through-azure-firewall"></a>Acheminer le trafic entre la machine virtuelle et le point de terminaison privé via Pare-feu Azure

Nous n’avons pas créé d’appairage de réseaux virtuels directement entre les réseaux virtuels **myVMVNet** et **myPEVNet**. La machine virtuelle **myVM** n’a pas d’itinéraire vers le point de terminaison privé que nous avons créé. 

Dans cette section, nous allons créer une table de routage avec un itinéraire personnalisé. 

L’itinéraire envoie le trafic du sous-réseau **myVM** à l’espace d’adressage du réseau virtuel **myPEVNet**, par le biais du pare-feu Azure.

1. Dans le menu du portail Azure ou dans la page **Accueil**, sélectionnez **Créer une ressource**.

2. Saisissez **table de routage** dans la zone de recherche, puis appuyez sur **Entrée**.

3. Sélectionnez **Table de routage**, puis **Créer**.

4. Sur la page **Créer une table de routage**, utilisez le tableau suivant pour configurer la table de routage :

    | Paramètre | Valeur |
    | ------- | ----- |
    | **Détails du projet** | |
    | Abonnement | Sélectionnez votre abonnement. |
    | Resource group | Sélectionnez **myResourceGroup**.  |
    | **Détails de l’instance** |  |
    | Région | Sélectionnez **USA Centre Sud**. |
    | Nom | Entrez **VMsubnet-to-AzureFirewall**. |
    | Propager des itinéraires de passerelle | Sélectionnez **Non**. |

5. Sélectionnez **Revoir + créer**. Vous êtes redirigé vers la page **Vérifier + créer** où Azure valide votre configuration.

6. Lorsque le message **Validation passed** (Validation réussie) apparaît, sélectionnez **Créer**.

7. Une fois le déploiement terminé, sélectionnez **Accéder à la ressource**.

8. Sélectionnez **Routes** sous **Paramètres**.

9. Sélectionnez **Ajouter**.

10. Dans la page **Ajouter un itinéraire**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom de l’itinéraire | Entrez **myVMsubnet-to-privateendpoint**. |
    | Préfixe de l’adresse | Entrez **10.2.0.0/16**.  |
    | Type de tronçon suivant | Sélectionnez **Appliance virtuelle**. |
    | adresse de tronçon suivant | Entrez **10.0.0.4**. |

11. Sélectionnez **OK**.

12. Sous **Paramètres**, sélectionnez **Sous-réseaux**.

13. Sélectionnez **+ Associer**.

14. Dans la page **Associer un sous-réseau**, entrez ou sélectionnez ces informations :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Réseau virtuel | Sélectionnez **myVMVNet**. |
    | Subnet | Sélectionnez **VMSubnet**.  |

15. Sélectionnez **OK**.

## <a name="connect-to-the-virtual-machine-from-your-client-computer"></a>Se connecter à la machine virtuelle à partir de l’ordinateur client

Connectez-vous à la machine virtuelle **myVm** à partir d’Internet comme suit :

1. Dans la barre de recherche du portail, entrez **myVm-ip**.

2. Sélectionnez **myVM-ip** dans les résultats de la recherche.

3. Copiez ou notez la valeur sous **Adresse IP**.

4. Si vous utilisez Windows 10, exécutez la commande suivante à l’aide de PowerShell. Pour les autres versions du client Windows, utilisez un client SSH comme [PuTTY](https://www.putty.org/) :

* Remplacez **username** par le nom d’utilisateur administrateur que vous avez entré lors de la création de la machine virtuelle.

* Remplacez **IPaddress** par l’adresse IP de l’étape précédente.

    ```bash
    ssh username@IPaddress
    ```

5. Entrez le mot de passe que vous avez défini lors de la création de **myVm**.

## <a name="access-sql-server-privately-from-the-virtual-machine"></a>Accéder à SQL Server de manière privée à partir de la machine virtuelle

Dans cette section, vous allez vous connecter de manière privée à SQL Database à l’aide du point de terminaison privé.

1. Entrez `nslookup mydbserver.database.windows.net`
    
    Vous recevrez un message similaire à celui ci-dessous :

    ```bash
    Server:         127.0.0.53
    Address:        127.0.0.53#53

    Non-authoritative answer:
    mydbserver.database.windows.net       canonical name = mydbserver.privatelink.database.windows.net.
    Name:   mydbserver.privatelink.database.windows.net
    Address: 10.2.0.4
    ```

2. Installez les [outils en ligne de commande SQL Server](/sql/linux/quickstart-install-connect-ubuntu?view=sql-server-ver15#tools).

3. Exécutez la commande suivante pour vous connecter à SQL Server. Utilisez l’administrateur de serveur et le mot de passe que vous avez définis lorsque vous avez créé le serveur SQL aux étapes précédentes.

* Remplacez **\<ServerAdmin>** par le nom d’utilisateur administrateur que vous avez entré lors de la création du serveur SQL.

* Remplacez **\<YourPassword>** par le mot de passe administrateur que vous avez entré lors de la création du serveur SQL.

    ```bash
    sqlcmd -S mydbserver.database.windows.net -U '<ServerAdmin>' -P '<YourPassword>'
    ```
4. Une invite de commandes SQL s’affichera en cas de connexion réussie. Entrez **exit** pour quitter l’outil **sqlcmd**.

5. Fermez la connexion à **myVM** en entrant **exit**.

## <a name="validate-the-traffic-in-azure-firewall-logs"></a>Valider le trafic dans les journaux de Pare-feu Azure

1. Dans le portail Azure, sélectionnez **Toutes les ressources**, puis sélectionnez votre espace de travail Log Analytics.

2. Sous **Général** dans la page de l’espace de travail Log Analytics, sélectionnez **Journaux**.

3. Sélectionnez le bouton **Prise en main** bleu.

4. Dans la fenêtre **Exemples de requêtes**, sélectionnez **Pare-feu** sous **Toutes les requêtes**.

5. Sélectionnez le bouton **Exécuter** sous **Données du journal de règles d’application**.

6. Dans le résultat de la requête de journal, vérifiez que **mydbserver.database.windows.net** est répertorié sous **Nom de domaine complet** et que **SQLPrivateEndpoint** est répertorié sous **RuleCollection**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous avez fini d’utiliser les ressources, supprimez le groupe de ressources et toutes les ressources qu’il contient :

1. Entrez **myResourceGroup** dans la zone **Recherche** en haut du portail, puis sélectionnez **myResourceGroup** dans les résultats de la recherche.

1. Sélectionnez **Supprimer le groupe de ressources**.

1. Entrez **myResourceGroup** dans **TAPER LE NOM DU GROUPE DE RESSOURCES** puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez exploré différents scénarios que vous pouvez utiliser pour limiter le trafic entre une machine virtuelle et un point de terminaison privé à l’aide de Pare-feu Azure. 

Vous vous êtes connecté à la machine virtuelle et avez communiqué en toute sécurité avec la base de données via Pare-feu Azure à l’aide de la liaison privée.

Pour plus d’informations sur les points de terminaison privés, consultez [Qu’est-ce qu’un point de terminaison privé Azure ?](private-endpoint-overview.md).
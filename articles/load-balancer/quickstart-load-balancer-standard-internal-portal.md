---
title: 'Démarrage rapide : Créer un équilibreur de charge interne - Portail Azure'
titleSuffix: Azure Load Balancer
description: Ce guide de démarrage rapide montre comment créer un équilibreur de charge interne en utilisant le portail Azure.
services: load-balancer
documentationcenter: na
author: asudbring
manager: KumudD
Customer intent: I want to create a internal load balancer so that I can load balance internal traffic to VMs.
ms.service: load-balancer
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 07/30/2020
ms.author: allensu
ms.custom: mvc
ms.openlocfilehash: 6ac76b3d3cc8fb27734730275836fba0dbfb08fe
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94700305"
---
# <a name="quickstart-create-an-internal-load-balancer-to-load-balance-vms-using-the-azure-portal"></a>Démarrage rapide : Créer un équilibreur de charge interne pour équilibrer la charge de machines virtuelles en utilisant le portail Azure

Découvrez comment bien démarrer avec Azure Load Balancer en utilisant le portail Azure pour créer un équilibreur de charge interne et deux machines virtuelles.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).

---

# <a name="standard-sku"></a>[**Référence Standard**](#tab/option-1-create-internal-load-balancer-standard)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production.  Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

Dans cette section, vous créez un équilibreur de charge qui équilibre la charge des machines virtuelles. 

Quand vous créez un équilibreur de charge interne, un réseau virtuel est configuré comme réseau pour l’équilibreur de charge. 

Une adresse IP privée dans le réseau virtuel est configurée comme front-end (nommé **LoadBalancerFrontend** par défaut) pour l’équilibreur de charge. 

L’adresse IP du front-end peut être **statique** ou **dynamique**.

## <a name="create-the-virtual-network"></a>Créer un réseau virtuel

Dans cette section, vous allez créer un réseau virtuel et un sous-réseau.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel**, ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionnez **CreateIntLBQS-rg**. |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **Europe Ouest** |

3. Sélectionnez l’onglet **Adresses IP**, ou sélectionnez le bouton **Suivant : Adresses IP** au bas de la page.

4. Sous l’onglet **Adresses IP**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Espace d’adressage IPv4 | Entrez **10.1.0.0/16** |

5. Sous **Nom de sous-réseau**, sélectionnez le mot **par défaut**.

6. Dans **Modifier le sous-réseau**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du sous-réseau | Entrez **myBackendSubnet** |
    | Plage d’adresses de sous-réseau | Entrez **10.1.0.0/24** |

7. Sélectionnez **Enregistrer**.

8. Sélectionnez l'onglet **Sécurité** .

9. Sous **BastionHost**, sélectionnez **Activer**. Entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du bastion | Entrez **myBastionHost** |
    | Espace d’adressage AzureBastionSubnet | Entrez **10.1.1.0/24** |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. </br> Pour **Nom**, entrez **myBastionIP**. </br> Sélectionnez **OK**. |


8. Sélectionnez l’onglet **Vérifier + créer**, ou sélectionnez le bouton **Vérifier + créer**.

9. Sélectionnez **Create** (Créer).

## <a name="create-load-balancer"></a>Créer un équilibreur de charge

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Load Balancer**.

2. Dans l’onglet **Fonctions de base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes : 

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **CreateIntLBQS-rg** créé à l’étape précédente.|
    | Nom                   | Entrez **myLoadBalancer**                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | sélectionnez **Interne**.                                        |
    | SKU           | sélectionnez **Standard**. |
    | Réseau virtuel | Sélectionnez **myVNet** créé à l’étape précédente. |
    | Subnet  | Sélectionnez **myBackendSubnet** créé à l’étape précédente. |
    | Affectation d’adresses IP | Sélectionnez **Dynamique**. |
    | Zone de disponibilité | Sélectionnez **Redondant dans une zone**. |

3. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

4. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.   
    
    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/create-standard-internal-load-balancer.png" alt-text="Créer un équilibreur de charge interne standard." border="true":::
 
## <a name="create-load-balancer-resources"></a>Créer les ressources d’équilibreur de charge

Dans cette section, vous configurez :

* Les paramètres d’équilibreur de charge d’un pool d’adresses de back-ends
* Une sonde d’intégrité
* Une règle d’équilibreur de charge

### <a name="create-a-backend-pool"></a>Créer un pool principal

Un pool d’adresses de back-ends contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. 

Créez le pool d’adresses principal **myBackendPool** afin d’inclure des machines virtuelles pour l’équilibrage de charge du trafic Internet.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.

3. Dans la page **Ajouter un pool de backends**, entrez **myBackendPool** comme nom de votre pool principal, puis sélectionnez **Ajouter**.

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

L’équilibreur de charge supervise l’état de votre application avec une sonde d’intégrité. 

La sonde d’intégrité ajoute ou supprime des machines virtuelles dans l’équilibreur de charge en fonction de leur réponse aux contrôles d’intégrité. 

Créez une sonde d’intégrité nommée **myHealthProbe** pour surveiller l’intégrité des machines virtuelles.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Sondes d’intégrité**, puis **Ajouter**.
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHealthProbe**. |
    | Protocol | Sélectionnez **HTTP**. |
    | Port | Entrez **80**.|
    | Intervalle | Entrez **15** pour **l’intervalle** en secondes entre les tentatives de la sonde. |
    | Seuil de défaillance sur le plan de l’intégrité | Sélectionnez **2** pour le **Seuil de défaillance sur le plan de l’intégrité**, soit le nombre d’échecs de sonde consécutifs qui peuvent se produire avant qu’une machine virtuelle soit considérée comme non saine.|
    | | |

3. Laissez les autres valeurs par défaut et sélectionnez **OK**.

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP front-end pour le trafic entrant et le pool d’adresses IP de back-ends pour la réception du trafic. Les ports source et de destination sont définis dans la règle. 

Dans cette section, vous allez créer une règle d’équilibreur de charge :

* A pour nom **myHTTPRule**.
* Se trouve dans le front-end nommé **LoadBalancerFrontEnd**.
* Écoute sur le **Port 80**.
* Dirige le trafic à charge équilibrée vers le back-end nommé **myBackendPool** sur le **Port 80**.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**, puis **Ajouter**.

3. Pour configurer la règle d’équilibrage de charge, utilisez les valeurs suivantes :
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHTTPRule**. |
    | Version de l’adresse IP | Sélectionnez **IPv4** |
    | Adresse IP du serveur frontal | Sélectionnez **LoadBalancerFrontEnd** |
    | Protocol | Sélectionnez **TCP**. |
    | Port | Entrez **80**.|
    | Port principal | Entrez **80**. |
    | Pool principal | Sélectionnez **MyBackendPool**.|
    | Sonde d’intégrité | Sélectionnez **myHealthProbe**. |
    | Délai d’inactivité (minutes) | Déplacez le curseur sur **15 minutes**. |
    | Réinitialisation du protocole TCP | Sélectionnez **Enabled**. |
    | Traduction d’adresses réseau (SNAT) sources sortante | Sélectionnez **(Recommandé) Utiliser des règles de trafic sortant pour fournir aux membres du pool de back-ends l’accès à Internet**. |

4. Laissez les autres valeurs par défaut, puis sélectionnez **OK**.

>[!NOTE]
>Avec cette configuration, les machines virtuelles du pool principal ne disposent pas d’une connectivité Internet sortante. </br> Pour plus d’informations sur la façon de fournir une connectivité sortante, consultez : </br> **[Connexions sortantes dans Azure](load-balancer-outbound-connections.md)**</br> Options pour fournir la connectivité : </br> **[Configuration de l’équilibreur de charge en sortie uniquement](egress-only.md)** </br> **[Qu’est-ce que le service NAT de Réseau virtuel ?](../virtual-network/nat-overview.md)**

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez :

* Créer deux machines virtuelles pour le pool de back-ends de l’équilibreur de charge.
* Installez IIS sur les machines virtuelles pour tester l’équilibreur de charge.

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Dans cette section, vous allez créer deux machines virtuelles (**myVM1** et **myVM2**).

Ces machines virtuelles sont ajoutées au pool de back-ends de l’équilibreur de charge créé auparavant.

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**. 
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **CreateIntLBQS-rg**. |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM1** |
    | Région | Sélectionnez **Europe Ouest** |
    | Options de disponibilité | Sélectionnez **Zones de disponibilité** |
    | Zone de disponibilité | Sélectionnez **1** |
    | Image | Sélectionnez **Windows Server 2019 Datacenter** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Adresse IP publique | Sélectionnez **Aucun** |
    | Groupe de sécurité réseau de la carte réseau | Sélectionnez **Avancé**|
    | Configurer un groupe de sécurité réseau | Sélectionnez **Créer nouveau**. </br> Dans **Créer un groupe de sécurité réseau**, entrez **myNSG** dans **Nom**. </br> Sélectionnez **OK**. |
    | **Équilibrage de charge**  |
    | Placer cette machine virtuelle derrière une solution d’équilibrage de charge existante ? | Sélectionnez **Oui**. |
    | **Paramètres d’équilibrage de charge** |
    | Options d’équilibrage de charge | Sélectionnez **Équilibrage de charge Azure** |
    | Sélectionnez un équilibreur de charge | Sélectionnez **myLoadBalancer**  |
    | Sélectionnez un pool principal | Sélectionnez **myBackendPool** |
   
5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

7. Suivez les étapes 1 à 8 pour créer une machine virtuelle supplémentaire avec les valeurs suivantes et tous les autres paramètres identiques à ceux de **myVM1** :

    | Paramètre | Machine virtuelle 2|
    | ------- | ----- |
    | Nom |  **myVM2** |
    | Zone de disponibilité | **2** |
    | Groupe de sécurité réseau | Sélectionnez le groupe **myNSG** existant|


# <a name="basic-sku"></a>[**Référence De base**](#tab/option-1-create-internal-load-balancer-basic)

>[!NOTE]
>Il est recommandé de disposer d’un équilibreur de charge de référence SKU Standard pour les charges de travail de production.  Pour plus d’informations sur les références SKU, consultez **[Références SKU Azure Load Balancer](skus.md)** .

Dans cette section, vous créez un équilibreur de charge qui équilibre la charge des machines virtuelles. 

Quand vous créez un équilibreur de charge interne, un réseau virtuel est configuré comme réseau pour l’équilibreur de charge. 

Une adresse IP privée dans le réseau virtuel est configurée comme front-end (nommé **LoadBalancerFrontend** par défaut) pour l’équilibreur de charge. 

L’adresse IP du front-end peut être **statique** ou **dynamique**.

## <a name="create-the-virtual-network"></a>Créer un réseau virtuel

Dans cette section, vous allez créer un réseau virtuel et un sous-réseau.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel**, ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionnez **CreateIntLBQS-rg**. |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **Europe Ouest** |

3. Sélectionnez l’onglet **Adresses IP**, ou sélectionnez le bouton **Suivant : Adresses IP** au bas de la page.

4. Sous l’onglet **Adresses IP**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Espace d’adressage IPv4 | Entrez **10.1.0.0/16** |

5. Sous **Nom de sous-réseau**, sélectionnez le mot **par défaut**.

6. Dans **Modifier le sous-réseau**, entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du sous-réseau | Entrez **myBackendSubnet** |
    | Plage d’adresses de sous-réseau | Entrez **10.1.0.0/24** |

7. Sélectionnez **Enregistrer**.

8. Sélectionnez l'onglet **Sécurité** .

9. Sous **BastionHost**, sélectionnez **Activer**. Entrez les informations suivantes :

    | Paramètre            | Valeur                      |
    |--------------------|----------------------------|
    | Nom du bastion | Entrez **myBastionHost** |
    | Espace d’adressage AzureBastionSubnet | Entrez **10.1.1.0/24** |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. </br> Pour **Nom**, entrez **myBastionIP**. </br> Sélectionnez **OK**. |


8. Sélectionnez l’onglet **Vérifier + créer**, ou sélectionnez le bouton **Vérifier + créer**.

9. Sélectionnez **Create** (Créer).

## <a name="create-load-balancer"></a>Créer un équilibreur de charge

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Load Balancer**.

2. Dans l’onglet **Fonctions de base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes : 

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **CreateIntLBQS-rg** créé à l’étape précédente.|
    | Nom                   | Entrez **myLoadBalancer**                                   |
    | Région         | Sélectionnez **Europe Ouest**.                                        |
    | Type          | sélectionnez **Interne**.                                        |
    | SKU           | Sélectionnez **De base** |
    | Réseau virtuel | Sélectionnez **myVNet** créé à l’étape précédente. |
    | Subnet  | Sélectionnez **myBackendSubnet** créé à l’étape précédente. |
    | Affectation d’adresses IP | Sélectionnez **Dynamique**. |

3. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

4. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.   

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/create-basic-internal-load-balancer.png" alt-text="Créer un équilibreur de charge interne de base." border="true":::

## <a name="create-load-balancer-resources"></a>Créer les ressources d’équilibreur de charge

Dans cette section, vous configurez :

* Les paramètres d’équilibreur de charge d’un pool d’adresses de back-ends
* Une sonde d’intégrité
* Une règle d’équilibreur de charge

### <a name="create-a-backend-pool"></a>Créer un pool principal

Un pool d’adresses de back-ends contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. 

Créez le pool d’adresses principal **myBackendPool** afin d’inclure des machines virtuelles pour l’équilibrage de charge du trafic Internet.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.

3. Dans la page **Ajouter un pool de back-ends**, entrez ou sélectionnez :
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **myBackendPool**. |
    | Réseau virtuel | Sélectionnez **myVNet**. |
    | Associé à | Sélectionnez **Machines virtuelles** |

4. Sélectionnez **Ajouter**.

### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

L’équilibreur de charge supervise l’état de votre application avec une sonde d’intégrité. 

La sonde d’intégrité ajoute ou supprime des machines virtuelles dans l’équilibreur de charge en fonction de leur réponse aux contrôles d’intégrité. 

Créez une sonde d’intégrité nommée **myHealthProbe** pour surveiller l’intégrité des machines virtuelles.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Sondes d’intégrité**, puis **Ajouter**.
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHealthProbe**. |
    | Protocol | Sélectionnez **HTTP**. |
    | Port | Entrez **80**.|
    | Path | Entrez **/** |
    | Intervalle | Entrez **15** pour **l’intervalle** en secondes entre les tentatives de la sonde. |
    | Seuil de défaillance sur le plan de l’intégrité | Sélectionnez **2** pour le **Seuil de défaillance sur le plan de l’intégrité**, soit le nombre d’échecs de sonde consécutifs qui peuvent se produire avant qu’une machine virtuelle soit considérée comme non saine.|

3. Sélectionnez **OK**.

### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibrage de charge est utilisée pour définir la distribution du trafic vers les machines virtuelles. Vous définissez la configuration IP front-end pour le trafic entrant et le pool d’adresses IP de back-ends pour la réception du trafic. Les ports source et de destination sont définis dans la règle. 

Dans cette section, vous allez créer une règle d’équilibreur de charge :

* A pour nom **myHTTPRule**.
* Se trouve dans le front-end nommé **LoadBalancerFrontEnd**.
* Écoute sur le **Port 80**.
* Dirige le trafic à charge équilibrée vers le back-end nommé **myBackendPool** sur le **Port 80**.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**, puis **Ajouter**.

3. Pour configurer la règle d’équilibrage de charge, utilisez les valeurs suivantes :
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHTTPRule**. |
    | Version de l’adresse IP | Sélectionnez **IPv4** |
    | Adresse IP du serveur frontal | Sélectionnez **LoadBalancerFrontEnd** |
    | Protocol | Sélectionnez **TCP**. |
    | Port | Entrez **80**.|
    | Port principal | Entrez **80**. |
    | Pool principal | Sélectionnez **MyBackendPool**.|
    | Sonde d’intégrité | Sélectionnez **myHealthProbe**. |
    | Délai d’inactivité (minutes) | Déplacez le curseur sur **15 minutes**. |
 
4. Laissez les autres valeurs par défaut, puis sélectionnez **OK**.

## <a name="create-backend-servers"></a>Créer des serveurs principaux

Dans cette section, vous allez :

* Créer deux machines virtuelles pour le pool de back-ends de l’équilibreur de charge.
* Créez un groupe à haute disponibilité pour les machines virtuelles.
* Installez IIS sur les machines virtuelles pour tester l’équilibreur de charge.

### <a name="create-virtual-machines"></a>Créer des machines virtuelles

Dans cette section, vous allez créer deux machines virtuelles (**myVM1** et **myVM2**).

Les deux machines virtuelles vont être ajoutées à un groupe à haute disponibilité nommé **myAvailabilitySet**.

Ces machines virtuelles sont ajoutées au pool de back-ends de l’équilibreur de charge créé auparavant.

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**. 
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **CreateIntLBQS-rg**. |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM1** |
    | Région | Sélectionnez **Europe Ouest** |
    | Options de disponibilité | Sélectionnez **Groupe à haute disponibilité** |
    | Groupe à haute disponibilité | Sélectionnez **Créer nouveau**. </br> Entrez **myAvailabilitySet** dans **Nom**. </br> Sélectionnez **OK**. |
    | Image | **Windows Server 2019 Datacenter** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | Sélectionnez **myVNet** |
    | Subnet | Sélectionnez **myBackendSubnet** |
    | Adresse IP publique | Sélectionnez **Aucun** |
    | Groupe de sécurité réseau de la carte réseau | Sélectionnez **Avancé**|
    | Configurer un groupe de sécurité réseau | Sélectionnez **Créer nouveau**. </br> Dans **Créer un groupe de sécurité réseau**, entrez **myNSG** dans **Nom**. </br> Sélectionnez **OK**. |
    | **Équilibrage de charge**  |
    | Placer cette machine virtuelle derrière une solution d’équilibrage de charge existante ? | Sélectionnez **Non** |

5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

7. Suivez les étapes 1 à 8 pour créer une machine virtuelle supplémentaire avec les valeurs suivantes et tous les autres paramètres identiques à ceux de **myVM1** :

    | Paramètre | Machine virtuelle 2 |
    | ------- | ----- |
    | Nom |  **myVM2** |
    | Groupe à haute disponibilité| Sélectionnez **myAvailabilitySet** |
    | Groupe de sécurité réseau | Sélectionnez le groupe **myNSG** existant|

### <a name="add-virtual-machines-to-the-backend-pool"></a>Ajouter des machines virtuelles au pool de back-ends

Les machines virtuelles créées lors des étapes précédentes doivent être ajoutées au pool de back-ends de **myLoadBalancer**.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools de back-ends**, puis **myBackendPool**.

3. Sélectionnez **Machines virtuelles** dans **Associé à**.

4. Dans la section **Machines virtuelles**, sélectionnez **+ Ajouter**.

5. Cochez les cases en regard de **myVM1** et **myVM2**.

6. Sélectionnez **Ajouter**.

7. Sélectionnez **Enregistrer**.
---

## <a name="create-test-virtual-machine"></a>Créer une machine virtuelle de test

Dans cette section, vous allez créer une machine virtuelle nommée **myTestVM**.  Cette machine virtuelle sera utilisée pour tester la configuration de l’équilibreur de charge.

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**. 
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **CreateIntLBQS-rg**. |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myTestVM** |
    | Région | Sélectionnez **Europe Ouest** |
    | Options de disponibilité | Sélectionnez **Aucune redondance d’infrastructure requise**. |
    | Image | Sélectionnez **Windows Server 2019 Datacenter** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-|-|
    | **Interface réseau** |  |
    | Réseau virtuel | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Adresse IP publique | Sélectionnez **Aucun**. |
    | Groupe de sécurité réseau de la carte réseau | Sélectionnez **Avancé**|
    | Configurer un groupe de sécurité réseau | Sélectionnez **MyNSG** créé à l’étape précédente.|
       
5. Sélectionnez **Revoir + créer**. 
  
6. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="install-iis"></a>Installer IIS

1. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources** puis, dans la liste de ressources, sélectionnez **myVM1** qui se trouve dans le groupe de ressources **CreateIntLBQS-rg**.

2. Dans la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Bastion**.

4. Entrez le nom d’utilisateur et le mot de passe saisis pendant la création de la machine virtuelle.

5. Sélectionnez **Connecter**.

6. Sur le bureau du serveur, accédez à **Outils d’administration Windows** > **Windows PowerShell**.

7. Dans la fenêtre PowerShell, exécutez les commandes suivantes pour :

    * Installer le serveur IIS
    * Supprimer le fichier iisstart.htm par défaut
    * Ajoutez un nouveau fichier iisstart.htm qui affiche le nom de la machine virtuelle :

   ```powershell
    
    # install IIS server role
    Install-WindowsFeature -name Web-Server -IncludeManagementTools
    
    # remove default htm file
     remove-item  C:\inetpub\wwwroot\iisstart.htm
    
    # Add a new htm file that displays server name
     Add-Content -Path "C:\inetpub\wwwroot\iisstart.htm" -Value $("Hello World from " + $env:computername)
   ```
8. Fermez la session Bastion avec **myVM1**.

9. Répétez les étapes 1 à 6 pour installer IIS et le fichier iisstart.htm mis à jour sur **myVM2**.


## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge

1. Recherchez l’adresse IP privée de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources**, puis sélectionnez **myLoadBalancer**.

2. Notez ou copiez l’adresse en regard de **Adresse IP privée** dans la **Vue d’ensemble** de **myLoadBalancer**.

3. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources** puis, dans la liste des ressources, sélectionnez **myTestVM** qui se trouve dans le groupe de ressources **CreateIntLBQS-rg**.

4. Dans la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Bastion**.

6. Entrez le nom d’utilisateur et le mot de passe saisis pendant la création de la machine virtuelle.

7. Ouvrez **Internet Explorer** sur **myTestVM**.

8. Entrez l’adresse IP de l’étape précédente dans la barre d’adresse du navigateur. La page par défaut du serveur Web IIS s’affiche sur le navigateur.

    :::image type="content" source="./media/quickstart-load-balancer-standard-internal-portal/load-balancer-test.png" alt-text="La capture d’écran présente une fenêtre de navigateur qui affiche la page par défaut, comme prévu." border="true":::
   
Pour voir l’équilibreur de charge répartir le trafic entre les deux machines virtuelles, vous pouvez personnaliser la page par défaut du serveur web IIS de chaque machine virtuelle, puis forcer votre navigateur à s’actualiser à partir de la machine cliente.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, l’équilibreur de charge et toutes les ressources associées. Pour ce faire, sélectionnez le groupe de ressources **CreateIntLBQS-rg** qui contient les ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous :

* Avez créé un équilibreur de charge Azure interne standard ou de base
* Avez attaché 2 machines virtuelles à l’équilibreur de charge.
* Avez configuré la règle de trafic de l’équilibreur de charge, configuré la sonde d’intégrité, puis testé l’équilibreur de charge 

Pour en savoir plus sur Azure Load Balancer, consultez
> [!div class="nextstepaction"]
> [Qu’est-ce que Azure Load Balancer ?](load-balancer-overview.md)
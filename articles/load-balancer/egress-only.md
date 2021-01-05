---
title: Configuration de l’équilibreur de charge en sortie uniquement
titleSuffix: Azure Load Balancer
description: Dans cet article, découvrez comment créer un équilibreur de charge interne avec NAT sortant
services: load-balancer
documentationcenter: na
author: asudbring
ms.custom: seodec18
ms.service: load-balancer
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 08/07/2020
ms.author: allensu
ms.openlocfilehash: ee264a22de5ce094e8a4c1335ace77cbbba49270
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94694991"
---
# <a name="outbound-only-load-balancer-configuration"></a>Configuration de l’équilibreur de charge en sortie uniquement

Utilisez une combinaison d’équilibreurs de charge standard internes et externes pour créer une connectivité sortante pour les machines virtuelles derrière un équilibreur de charge interne. 

Cette configuration fournit une traduction d’adresses réseau (NAT) sortante pour un scénario d’équilibrage de charge interne, ce qui génère une configuration de « sortie uniquement » pour votre pool principal.


<p align="center">
  <img src="./media/egress-only/load-balancer-egress-only.png" alt="Figure depicts a egress only load balancer configuration." width="500" title="Service NAT de Réseau virtuel">
</p>


<!-- 
:::image type="content" source="./media/egress-only/load-balancer-egress-only.png" alt-text="Figure depicts a egress only load balancer configuration" border="true":::
-->


*Figure : Configuration de l’équilibreur de charge en sortie uniquement*

Les étapes requises sont les suivantes :

1. Créez un réseau virtuel avec un hôte bastion.
2. Créez une machine virtuelle avec une adresse IP privée uniquement.
3. Créez des équilibreurs de charge standard internes et publics.
4. Ajoutez des pools principaux aux deux équilibreurs de charge et placez la machine virtuelle dans chaque pool.
5. Connectez-vous à votre machine virtuelle par le biais de l’hôte bastion et :
    1. Testez la connectivité sortante. 
    2. Configurez une règle de trafic sortant sur l’équilibreur de charge public.
    3. Retestez la connectivité sortante.

## <a name="create-virtual-network-and-virtual-machine"></a>Créer un réseau virtuel et une machine virtuelle

Créer un réseau virtuel avec deux sous-réseaux :

* Sous-réseau principal
* Sous-réseau bastion

Créez une machine virtuelle dans le nouveau réseau virtuel.

### <a name="create-the-virtual-network"></a>Créer un réseau virtuel

1. [Connectez-vous](https://portal.azure.com) au portail Azure.

2. En haut à gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Réseau virtuel**, ou recherchez **Réseau virtuel** à partir de la zone de recherche.

2. Dans **Créer un réseau virtuel**, entrez ou sélectionnez les informations suivantes sous l’onglet **Général** :

    | **Paramètre**          | **Valeur**                                                           |
    |------------------|-----------------------------------------------------------------|
    | **Détails du projet**  |                                                                 |
    | Abonnement     | Sélectionner votre abonnement Azure                                  |
    | Groupe de ressources   | Sélectionnez **Créer nouveau**. </br> Entrez **myResourceGroupLB**. </br> Sélectionnez **OK**. |
    | **Détails de l’instance** |                                                                 |
    | Nom             | Entrez **myVNet**                                    |
    | Région           | Sélectionnez **USA Est 2** |

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

### <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Calcul** > **Machine virtuelle**. 
   
2. Dans **Créer une machine virtuelle**, tapez ou sélectionnez les valeurs sous l’onglet **De base** :

    | Paramètre | Valeur                                          |
    |-----------------------|----------------------------------|
    | **Détails du projet** |  |
    | Abonnement | Sélectionner votre abonnement Azure |
    | Groupe de ressources | Sélectionnez **myResourceGroupLB** |
    | **Détails de l’instance** |  |
    | Nom de la machine virtuelle | Entrez **myVM**. |
    | Région | Sélectionnez **USA Est 2** |
    | Options de disponibilité | Sélectionnez **Aucune redondance d’infrastructure requise**. |
    | Image | Sélectionnez **Windows Server 2019 Datacenter** |
    | Instance Azure Spot | Sélectionnez **Non** |
    | Taille | Choisissez la taille de la machine virtuelle ou acceptez le paramètre par défaut |
    | **Compte administrateur** |  |
    | Nom d’utilisateur | Entrez un nom d’utilisateur |
    | Mot de passe | Entrez un mot de passe |
    | Confirmer le mot de passe | Entrez à nouveau le mot de passe |
    | **Règles des ports d’entrée** |  |
    | Aucun port d’entrée public | Sélectionnez **Autoriser les ports sélectionnés** |
    | Sélectionner des ports d’entrée | Sélectionnez **RDP (3389)** |

3. Sélectionnez l'onglet **Mise en réseau** ou choisissez **Suivant : Disques**, puis **Suivant : Mise en réseau**.
  
4. Sous l’onglet Réseau, sélectionnez ou entrez :

    | Paramètre | Valeur |
    |-----|------------|
    | **Interface réseau** |  |
    | Réseau virtuel | **myVNet** |
    | Subnet | **myBackendSubnet** |
    | Adresse IP publique | Sélectionnez **Aucun**. |
    | Groupe de sécurité réseau de la carte réseau | Sélectionnez **Aucun**|
    | Placer cette machine virtuelle derrière une solution d’équilibrage de charge existante ? | Sélectionnez **Non** |
   
5. Sélectionnez l’onglet **Gestion** ou sélectionnez **Suivant** > **Gestion**.

6. Sous l’onglet **Gestion**, sélectionnez ou entrez :
    
    | Paramètre | Valeur |
    |-|-|
    | **Surveillance** |  |
    | Diagnostics de démarrage | Sélectionnez **Désactivé** |
   
7. Sélectionnez **Revoir + créer**. 
  
8. Passez en revue les paramètres, puis sélectionnez **Créer**.

## <a name="create-load-balancers-and-test-connectivity"></a>Créer des équilibreurs de charge et tester la connectivité

Utilisez le portail Azure pour créer :

* Équilibreur de charge interne
* Équilibreur de charge public
 
Ajoutez votre machine virtuelle créée au pool principal de chaque.  Vous allez ensuite établir une configuration pour autoriser uniquement la connectivité sortante de votre machine virtuelle, en effectuant des tests avant et après.

### <a name="create-internal-load-balancer"></a>Créer un équilibrage de charge interne

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Load Balancer**.

2. Dans l’onglet **Fonctions de base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes : 

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **myResourceGroupLB** créé à l’étape précédente.|
    | Nom                   | Entrez **myInternalLoadBalancer**                                   |
    | Région         | Sélectionnez **USA Est**.                                        |
    | Type          | sélectionnez **Interne**.                                        |
    | SKU           | sélectionnez **Standard**. |
    | Réseau virtuel | Sélectionnez **myVNet** créé à l’étape précédente. |
    | Subnet  | Sélectionnez **myBackendSubnet** créé à l’étape précédente. |
    | Affectation d’adresses IP | Sélectionnez **Dynamique**. |

3. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

4. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.   

### <a name="create-public-load-balancer"></a>Créer un équilibreur de charge public

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Load Balancer**.

2. Dans l’onglet **Fonctions de base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes : 

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer**, puis entrez **myResourceGroupLB** dans la zone de texte.|
    | Nom                   | Entrez **myPublicLoadBalancer**                                   |
    | Région         | Sélectionnez **USA Est**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | sélectionnez **Standard**. |
    | Adresse IP publique | Sélectionnez **Créer nouveau**. |
    | Nom de l’adresse IP publique | Entrez **myFrontendIP** dans la zone de texte.|
    | Zone de disponibilité | Sélectionnez **Redondant dans une zone**. |
    | Ajouter une adresse IPv6 publique | Sélectionnez **Non**. |

3. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

4. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.   
   
### <a name="create-internal-backend-address-pool"></a>Créer un pool d’adresses principal interne

Créez le pool d’adresses principal **myInternalBackendPool** :

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myInternalLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.

3. Sur la page **Ajouter un pool principal**, saisissez **myInternalBackendPool** comme nom de votre pool principal.
 
4. Dans **Réseau virtuel**, sélectionnez **myVNet**.

5. Sous **Machines virtuelles**, sélectionnez **Ajouter** et choisissez **myVM**.
 
6. Sélectionnez **Ajouter**.

### <a name="create-public-backend-address-pool"></a>Créer un pool d’adresses principal public

Créez le pool d’adresses principal **myPublicBackendPool** :

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myPublicLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.

3. Sur la page **Ajouter un pool principal**, saisissez **myPublicBackendPool** comme nom de votre pool principal.

4. Dans **Réseau virtuel**, sélectionnez **myVNet**.
 
5. Sous **Machines virtuelles**, sélectionnez **Ajouter** et choisissez **myVM**.
 
6. Sélectionnez **Ajouter**.

### <a name="test-connectivity-before-outbound-rule"></a>Tester la connectivité avant la règle de trafic sortant

1. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources**, puis, dans la liste de ressources, sélectionnez **myVM** qui se trouve dans le groupe de ressources **myResourceGroupLB**.

2. Sur la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Bastion**.

4. Entrez le nom d’utilisateur et le mot de passe saisis lors de la création de la machine virtuelle.

5. Sélectionnez **Connecter**.

6. Ouvrez Internet Explorer.

7. Entrez **https://whatsmyip.org** dans la barre d’adresse.

8. La connexion doit échouer. Par défaut, l’équilibreur de charge public standard [n’autorise pas le trafic sortant sans une règle de trafic sortant définie](load-balancer-overview.md#securebydefault).
 
### <a name="create-a-public-load-balancer-outbound-rule"></a>Créer une règle de trafic sortant d’équilibreur de charge public

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myPublicLoadBalancer** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Règles de trafic sortant**, puis **Ajouter**.

3. Pour configurer les règles de trafic sortant, utilisez les valeurs suivantes :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **myOutboundRule**. |
    | Adresse IP du serveur frontal | Sélectionnez **LoadBalancerFrontEnd**.|
    | Délai d’inactivité (minutes) | Déplacez le curseur sur **15 minutes**.|
    | Réinitialisation du protocole TCP | Sélectionnez **Enabled**.|
    | Pool principal | Sélectionnez **myPublicBackendPool**.| |
    | Allocation de port -> Allocation de port | Sélectionnez **Utiliser le nombre de ports de sortie par défaut** |

4. Sélectionnez **Ajouter**.

### <a name="test-connectivity-after-outbound-rule"></a>Tester la connectivité après la règle de trafic sortant

1. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources**, puis, dans la liste de ressources, sélectionnez **myVM** qui se trouve dans le groupe de ressources **myResourceGroupLB**.

2. Sur la page **Vue d’ensemble**, sélectionnez **Se connecter**, puis **Bastion**.

4. Entrez le nom d’utilisateur et le mot de passe saisis lors de la création de la machine virtuelle.

5. Sélectionnez **Connecter**.

6. Ouvrez Internet Explorer.

7. Entrez **https://whatsmyip.org** dans la barre d’adresse.

8. La connexion doit être établie.

9. L’adresse IP affichée doit être l’adresse IP frontale de **myPublicLoadBalancer**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, les équilibreurs de charge, la machine virtuelle et toutes les ressources associées. 

Pour ce faire, sélectionnez le groupe de ressources **myResourceGroupLB**, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez créé une configuration de « sortie uniquement » avec une combinaison d’équilibreurs de charge publics et internes.  

Cette configuration vous permet d’équilibrer la charge du trafic interne entrant vers votre pool principal tout en empêchant les connexions entrantes publiques.

- En savoir plus sur [Azure Load Balancer](load-balancer-overview.md).
- En savoir plus sur les [connexions sortantes dans Azure](load-balancer-outbound-connections.md).
- [FAQ](load-balancer-faqs.md) sur Azure Load Balancer.
- En savoir plus sur [Azure Bastion](../bastion/bastion-overview.md).
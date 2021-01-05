---
title: Démarrage rapide - Créer un service Private Link avec le portail Azure
titlesuffix: Azure Private Link
description: Découvrez dans ce guide de démarrage rapide comment créer un service Private Link en utilisant le portail Azure
services: private-link
author: malopMSFT
ms.service: private-link
ms.topic: quickstart
ms.date: 02/03/2020
ms.author: allensu
ms.openlocfilehash: 5b7bc8be89068f0d3cf6722c36ae7fd5cc560736
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "96012116"
---
# <a name="quickstart-create-a-private-link-service-by-using-the-azure-portal"></a>Démarrage rapide : Créer un service Private Link en utilisant le portail Azure

Un service Azure Private Link fait référence à votre propre service qui est géré par Private Link. Vous pouvez accorder un accès Private Link au service ou à la ressource qui fonctionne derrière Azure Standard Load Balancer. Les consommateurs de votre service peuvent y accéder en privé à partir de leurs propres réseaux virtuels. Dans ce guide de démarrage rapide, vous découvrez comment créer un service Private Link en utilisant le portail Azure.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au portail Azure sur https://portal.azure.com.

## <a name="create-an-internal-load-balancer"></a>Créer un équilibrage de charge interne

Créez d’abord un réseau virtuel. Ensuite, créez un équilibreur de charge interne à utiliser avec le service Private Link.

## <a name="virtual-network-and-parameters"></a>Réseau virtuel et paramètres

Dans cette section, vous créez un réseau virtuel. Vous créez également le sous-réseau pour héberger l’équilibreur de charge qui accède à votre service Private Link.

Dans cette section, vous allez devoir remplacer les paramètres suivants dans les étapes par les informations ci-dessous :

| Paramètre                   | Valeur                |
|-----------------------------|----------------------|
| **\<resource-group-name>**  | myResourceGroupLB |
| **\<virtual-network-name>** | myVNet          |
| **\<region-name>**          | USA Est 2      |
| **\<IPv4-address-space>**   | 10.3.0.0/16          |
| **\<subnet-name>**          | myBackendSubnet        |
| **\<subnet-address-range>** | 10.3.0.0/24          |

[!INCLUDE [virtual-networks-create-new](../../includes/virtual-networks-create-new.md)]

### <a name="create-a-standard-load-balancer"></a>Créer un équilibreur de charge standard

Utilisez le portail pour créer un équilibreur de charge interne standard. Le nom et l’adresse IP que vous spécifiez sont automatiquement configurés comme front-end de l’équilibreur de charge.

1. En haut à gauche du portail, sélectionnez **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.

1. Sous l’onglet **De base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | **Abonnement**               | Sélectionnez votre abonnement.    |
    | **Groupe de ressources**         | Sélectionnez **myResourceGroupLB** dans la zone.|
    | **Nom**                   | Entrez **myLoadBalancer**.                                   |
    | **Région**         | Sélectionnez **USA Est**.                                        |
    | **Type**          | sélectionnez **Interne**.                                        |
    | **Référence (SKU)**           | Sélectionnez **Standard**.                          |
    | **Réseau virtuel**           | Sélectionnez **myVNet**.                          |
    | **Affectation d’adresses IP**              | Sélectionnez **Statique**.   |
    | **Adresse IP privée**|Entrez une adresse qui se trouve dans l'espace d'adressage de votre réseau virtuel et de votre sous-réseau. Par exemple : 10.3.0.7.  |

1. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

1. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.

### <a name="create-standard-load-balancer-resources"></a>Créer des ressources d’équilibreur de charge standard

Dans cette section, vous configurez les paramètres de l’équilibreur de charge pour un pool d’adresses principales et une sonde d’intégrité. Vous spécifiez également les règles de l’équilibreur de charge.

#### <a name="create-a-back-end-pool"></a>Créer un pool de back-ends

Un pool d’adresses de back-end contient les adresses IP des cartes d’interface réseau virtuelles connectées à l’équilibreur de charge. Ce pool vous permet de distribuer le trafic à vos ressources. Créez le pool d’adresses de back-end appelé **myBackendPool** pour inclure les ressources qui équilibrent la charge du trafic.

1. Dans le menu le plus à gauche, sélectionnez **Tous les services**.
1. Sélectionnez **Toutes les ressources**, puis **myLoadBalancer** dans la liste de ressources.
1. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.
1. Dans la page **Ajouter un pool de backends**, entrez **myBackendPool** comme nom de votre pool de back-ends, puis sélectionnez **Ajouter**.

#### <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Utilisez une sonde d’intégrité pour laisser l’équilibreur de charge superviser l’état des ressources. En fonction des réponses des ressources aux contrôles d’intégrité, la sonde d’intégrité ajoute ou supprime dynamiquement des ressources dans la rotation de l’équilibreur de charge.

Pour créer une sonde d’intégrité en vue de superviser l’intégrité des ressources :

1. Sélectionnez **Toutes les ressources** dans le menu le plus à gauche, puis **myLoadBalancer** dans la liste de ressources.

1. Sous **Paramètres**, sélectionnez **Health probes** (Sondes d’intégrité), puis **Ajouter**.

1. Dans la page **Ajouter une sonde d’intégrité**, entrez ou sélectionnez les valeurs suivantes :

   - **Name** : Entrez **MyHealthProbe**.
   - **Protocole** : Sélectionnez **TCP**.
   - **Port** : Entrez **80**.
   - **Intervalle** : Entrez **15**. Cette valeur est le nombre de secondes entre les tentatives de la sonde.
   - **Seuil de défaillance sur le plan de l’intégrité** : Entrez **2**. Cette valeur est le nombre d’échecs consécutifs de la sonde qui se produisent avant qu’une machine virtuelle soit considérée comme non saine.

1. Sélectionnez **OK**.

#### <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Une règle d’équilibreur de charge définit comment le trafic est distribué aux ressources. La règle définit :

- La configuration IP front-end pour le trafic entrant.
- Le pool d’IP de back-end qui reçoit le trafic.
- Les ports source et de destination requis.

La règle d’équilibreur de charge nommée **myLoadBalancerRule** écoute sur le port 80 dans le front-end **LoadBalancerFrontEnd**. La règle envoie le trafic réseau au pool d’adresses de back-end **myBackEndPool** sur le même port 80.

Pour créer une règle d’équilibreur de charge :

1. Sélectionnez **Toutes les ressources** dans le menu le plus à gauche, puis **myLoadBalancer** dans la liste de ressources.

1. Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**, puis **Ajouter**.

1. Dans la page **Ajouter une règle d’équilibrage de charge**, entrez ou sélectionnez les valeurs suivantes, si elles ne sont pas déjà présentes :

   - **Name** : Entrez **myLoadBalancerRule**.
   - **Adresse IP du serveur frontal :** Entrez **LoadBalancerFrontEnd**.
   - **Protocole** : Sélectionnez **TCP**.
   - **Port** : Entrez **80**.
   - **Port principal** : Entrez **80**.
   - **Pool principal** : Sélectionnez **MyBackendPool**.
   - **Sonde d’intégrité** : Sélectionnez **myHealthProbe**. 

1. Sélectionnez **OK**.

## <a name="create-a-private-link-service"></a>Créer un service Liaison privée

Dans cette section, vous allez créer un service Private Link derrière un équilibreur de charge standard.

1. En haut à gauche de la page du portail Azure, sélectionnez **Créer une ressource** > **Mise en réseau** > **Centre de liaisons privées (préversion)** . Vous pouvez également utiliser la zone de recherche du portail pour rechercher une liaison privée.

1. Dans **Centre de liaisons privées - Vue d’ensemble** > **Exposer votre propre service pour permettre à d’autres utilisateurs de se connecter**, sélectionnez **Démarrer**.

1. Sous **Créer un service de liaison privée - De base**, entrez ou sélectionnez les informations suivantes :

    | Paramètre           | Valeur                                                                        |
    |-------------------|------------------------------------------------------------------------------|
    | Détails du projet :  |                                                                              |
    | **Abonnement**      | Sélectionnez votre abonnement.                                                     |
    | **Groupe de ressources**    | Sélectionnez **myResourceGroupLB**.                                                    |
    | Détails de l’instance : |                                                                              |
    | **Nom**              | Entrez **myPrivateLinkService**. |
    | **Région**            | Sélectionnez **USA Est**.                                                        |

1. Sélectionnez **Suivant : Paramètres sortants**.

1. Sous **Créer un service de liaison privée - Paramètres sortants**, entrez ou sélectionnez les informations suivantes :

    | Paramètre                           | Valeur                                                                           |
    |-----------------------------------|---------------------------------------------------------------------------------|
    | **Équilibreur de charge**                     | Sélectionnez **myLoadBalancer**.                                                           |
    | **Adresse IP front-end de l’équilibreur de charge** | Sélectionnez l’adresse IP front-end de **myLoadBalancer**.                                |
    | **Réseau virtuel NAT source**        | Sélectionnez **myVNet**.                                                                   |
    | **Sous-réseau NAT source**                 | Sélectionnez **myBackendSubnet**.                                                          |
    | **Activer le proxy TCP v2**               | Sélectionnez **OUI** ou **NON** selon que votre application attend ou non un en-tête de proxy TCP v2. |
    | **Paramètres d’adresse IP privée**       | Configurez la méthode d’allocation et l’adresse IP pour chaque adresse IP NAT.                  |

1. Sélectionnez **Suivant : Sécurité des accès**.

1. Sous **Créer un service de liaison privée – Sécurité d’accès**, sélectionnez **Visibilité**, puis choisissez **Contrôle d’accès en fonction du rôle (RBAC) uniquement**.
  
1. Sélectionnez **Suivant : Étiquettes** > **Vérifier + créer** ou choisissez l’onglet **Vérifier + créer** en haut de la page.

1. Passez en revue vos informations, puis sélectionnez **Créer**.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous avez terminé d’utiliser le service Private Link, supprimez le groupe de ressources pour nettoyer les ressources utilisées dans ce démarrage rapide.

1. Entrez **myResourceGroupLB** dans la zone de recherche en haut du portail, puis sélectionnez **myResourceGroupLB** dans les résultats de la recherche.
1. Sélectionnez **Supprimer le groupe de ressources**.
1. Dans **TAPEZ LE NOM DU GROUPE DE RESSOURCES**, entrez **myResourceGroup**.
1. Sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un équilibreur de charge Azure interne et un service Private Link. Vous pouvez aussi apprendre à [créer un point de terminaison privé en utilisant le portail Azure](./create-private-endpoint-portal.md).
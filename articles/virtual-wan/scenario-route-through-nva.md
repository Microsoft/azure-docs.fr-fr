---
title: 'Scénario : Acheminer le trafic via une appliance virtuelle réseau (NVA)'
titleSuffix: Azure Virtual WAN
description: Router le trafic via une NVA
services: virtual-wan
author: cherylmc
ms.service: virtual-wan
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: cherylmc
ms.custom: fasttrack-edit
ms.openlocfilehash: 78ff0440fa83b6bd002cdf4256dc066342b1b390
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92424765"
---
# <a name="scenario-route-traffic-through-an-nva"></a>Scénario : Router le trafic via une NVA

Lorsque vous travaillez avec le routage de hub virtuel Virtual WAN, il existe un certain nombre de scénarios disponibles. Dans ce scénario d’appliance virtuelle réseau, l’objectif est d’acheminer le trafic via une NVA (appliance virtuelle réseau) pour la branche vers le VNet et pour le VNet vers la branche. Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

> [!NOTE]
> Si vous disposez déjà d'une configuration avec des itinéraires antérieurs aux nouvelles fonctionnalités présentées dans [Guide pratique pour configurer le routage de hub virtuel](how-to-virtual-hub-routing.md), suivez les étapes décrites dans les versions suivantes des articles :
>* [Article concernant le portail Azure](virtual-wan-route-table-nva-portal.md)
>* [Article concernant PowerShell](virtual-wan-route-table-nva.md)
>

## <a name="design"></a><a name="design"></a>Conception

Dans ce scénario, nous allons utiliser la convention d’affectation de noms :

* « Réseaux virtuels NVA » pour les réseaux virtuels où les utilisateurs ont déployé une NVA et ont connecté d’autres réseaux virtuels en tant que spokes (VNet 2 et VNet 4 dans la **matrice de connectivité** ci-dessous).
* « Spokes NVA » pour les réseaux virtuels connectés à un réseau virtuel NVA (VNet 5, VNet 6, VNet 7 et VNet 8 dans la **matrice de connectivité** ci-dessous).
* « Réseaux virtuels non-NVA » pour les réseaux virtuels connectés à Azure Virtual WAN qui n’ont pas de NVA ni d’autres réseaux virtuels appairés avec eux (VNet 1 et VNet 3 dans la **matrice de connectivité** ci-dessous).
* « Hubs » pour les hubs Virtual WAN gérés par Microsoft, où les réseaux virtuels NVA sont connectés. Les réseaux virtuels spokes NVA n’ont pas besoin d’être connectés aux hubs Virtual WAN, mais uniquement à aux réseaux virtuels NVA.

La matrice de connectivité suivante résume les flux pris en charge dans ce scénario :

**Matrice de connectivité**

| Du             | Par :|   *Spokes NVA*|*Réseaux virtuels NVA*|*Réseaux virtuels non-NVA*|*Branches*|
|---|---|---|---|---|---|
| **Spokes NVA**   | &#8594; | Sur le réseau virtuel NVA | Peering | Sur le réseau virtuel NVA | Sur le réseau virtuel NVA |
| **Réseaux virtuels NVA**    | &#8594; | Appairage | Direct | Direct | Direct |
| **Réseaux virtuels non-NVA**| &#8594; | Sur le réseau virtuel NVA | Direct | Direct | Direct |
| **Branches**     | &#8594; | Sur le réseau virtuel NVA | Direct | Direct | Direct |

Chacune des cellules de la matrice de connectivité indique comment un réseau virtuel ou une branche (côté « De » du flux, les en-têtes de lignes dans la table) communique avec u réseau virtuel ou une branche de destination (côté « À » du flux, en-têtes de colonne en italique dans la table). « Direct » signifie que la connectivité est fournie en mode natif par Virtual WAN, « Appairage » signifie que la connectivité est assurée par un itinéraire défini par l’utilisateur dans le réseau virtuel, « Sur le réseau virtuel NVA » signifie que la connectivité traverse les appliance virtuelle réseau (NVA) déployées dans le réseau virtuel NVA. Tenez compte des éléments suivants :

* Les spokes NVA ne sont pas gérés par Virtual WAN. Par conséquent, les mécanismes avec lesquels ils communiquent avec d’autres réseaux virtuels ou branches sont gérés par l’utilisateur. La connectivité au réseau virtuel NVA est assurée par un appairage de réseaux virtuels, et un itinéraire par défaut vers 0.0.0.0/0 pointant vers la NVA en tant que tronçon suivant doit couvrir la connectivité à Internet, à d’autres spokes et à des branches
* Les réseaux virtuels NVA connaîtront leurs propres spokes NVA, mais pas les spokes NVA connectés à d’autres réseaux virtuels NVA. Par exemple, dans le Tableau 1, VNet 2 connaît VNet 5 et VNet 6, mais pas les autres spokes tels que VNet 7 et VNet 8. Un itinéraire statique est requis pour injecter les préfixes d’autres spokes dans des réseaux virtuels NVA
* De même, les branches et les réseaux virtuels NVA ne connaissent aucun spoke NVA, étant donné que les spokes NVA ne sont pas connectés aux hubs Virtual WAN. Par conséquent, les itinéraires statiques seront également nécessaires ici.

Compte tenu du fait que les spokes NVA ne sont pas gérés par Virtual WAN, toutes les autres lignes affichent le même modèle de connectivité. Par conséquent, une seule table de routage (celle par défaut) effectue les opérations suivantes :

* Réseaux virtuels (réseaux virtuels non-hub et réseaux virtuels de hub d’utilisateurs) :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **Par défaut**
* Branches :
  * Table de routage associée : **Par défaut**
  * Propagation aux tables de routage : **Par défaut**

Toutefois, dans ce scénario, nous devons réfléchir aux itinéraires statiques à configurer. Chaque itinéraire statique a deux composants, l’un dans le hub Virtual WAN qui indique aux composants Virtual WAN la connexion à utiliser pour chaque spoke, et l’autre dans cette connexion spécifique qui pointe vers l’adresse IP concrète assignée à la NVA (ou à un équilibreur de charge devant plusieurs NVA), comme le montre la **Figure 1**  :

**Figure 1**

:::image type="content" source="media/routing-scenarios/nva/nva-static-concept.png" alt-text="Figure 1":::

Avec cela, les itinéraires statiques dont nous avons besoin dans la table par défaut pour envoyer le trafic vers les spokes NVA derrière le réseau virtuel NVA sont les suivants :

| Description | Table de routage | Itinéraire statique              |
| ----------- | ----------- | ------------------------- |
| Réseau virtuel 2       | Default     | 10.2.0.0/16 -> eastusconn |
| VNet 4       | Default     | 10.4.0.0/16 -> weconn     |

Désormais, le réseau étendu virtuel sait à quelle connexion envoyer les paquets, mais la connexion doit savoir ce qu’il faut faire lors de la réception de ces paquets : c’est là que les tables de routage de connexion sont utilisées. Ici, nous utiliserons les préfixes plus courts (/24 au lieu de /16 qui est plus long), pour nous assurer que ces itinéraires ont la préférence sur les itinéraires importés à partir de réseaux virtuels NVA (VNet 2 et VNet 4) :

| Description | Connexion | Itinéraire statique            |
| ----------- | ---------- | ----------------------- |
| VNet 5       | eastusconn | 10.2.1.0/24 -> 10.2.0.5 |
| VNet 6       | eastusconn | 10.2.2.0/24 -> 10.2.0.5 |
| VNet 7       | weconn     | 10.4.1.0/24 -> 10.4.0.5 |
| VNet 8       | weconn     | 10.4.2.0/24 -> 10.4.0.5 |

Désormais, les réseaux virtuels NVA, les réseaux virtuels non-NVA et les branches savent comment atteindre tous les spokes NVA. Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

## <a name="architecture"></a><a name="architecture"></a>Architecture

Dans la **Figure 2** , il y a deux hubs : **Hub1** et **Hub2** .

* **Hub1** et **Hub2** sont connectés directement aux réseaux virtuels NVA **VNet 2** et **VNet 4** .

* **VNet 5** et **VNet 6** sont appairés avec **VNet 2** .

* **VNet 7** et **VNet 8** sont appairés avec **VNet 4** .

* Les **VNets 5, 6, 7 et 8** sont des spokes indirects, qui ne sont pas directement connectés à un hub virtuel.

**Figure 2**

:::image type="content" source="./media/routing-scenarios/nva/nva.png" alt-text="Figure 1" lightbox="./media/routing-scenarios/nva/nva.png":::

## <a name="scenario-workflow"></a><a name="workflow"></a>Workflow du scénario

Pour configurer le routage via la NVA, voici les étapes à prendre en compte :

1. Identifiez la connexion du réseau virtuel en étoile à la NVA. Dans la **Figure 2** , il y a une **connexion VNet 2 (eastusconn)** et une **connexion VNet 4 (weconn)** .

   Vérifiez que des itinéraires définis par l’utilisateur (UDR) sont configurés :
   * De VNet 5 et VNet 6 à l’IP NVA de VNet 2
   * De VNet 7 et VNet 8 à l’IP NVA de VNet 4 
   
   Vous n’avez pas à connecter les VNets 5, 6, 7 et 8 directement aux hubs virtuels. Assurez-vous que les NSG dans les VNets 5, 6, 7 et 8 autorisent le trafic pour la branche (VPN/ER/P2S) ou aux réseaux virtuels connectés à leurs réseaux virtuels distants. Par exemple, les VNets 5 et 6 doivent garantir que les NSG autorisent le trafic pour les préfixes d’adresses locaux et les VNets 7 et 8 qui sont connectés au Hub 2 distant.

Virtual WAN ne prend pas en charge un scénario où les VNets 5, 6 se connectent au hub virtuel et communiquent via une adresse IP NVA du VNet 2 ; par conséquent, il est nécessaire de connecter les VNets 5, 6 au VNet 2 et de même les VNets 7, 8 au VNet 4.

2. Ajoutez une entrée d’itinéraire statique agrégé pour les VNets 2, 5 et 6 à la table de routage par défaut du Hub 1.

   :::image type="content" source="./media/routing-scenarios/nva/nva-static-expand.png" alt-text="Figure 1":::

3. Configurez un itinéraire statique pour les VNets 5 et 6 dans la connexion de réseau virtuel de VNet 2. Pour configurer la configuration du routage pour une connexion de réseau virtuel, consultez [routage de hub virtuel](how-to-virtual-hub-routing.md#routing-configuration).

4. Ajoutez une entrée d’itinéraire statique agrégé pour les VNet 4, 7 et 8 à la table de routage par défaut de Hub 1.

5. Répétez les étapes 2, 3 et 4 pour la table de routage par défaut de Hub 2.

Cela entraîne des modifications de configuration de routage, comme l’illustre la **Figure 3** ci-dessous.

**Figure 3**

   :::image type="content" source="./media/routing-scenarios/nva/nva-result.png" alt-text="Figure 1" lightbox="./media/routing-scenarios/nva/nva-result.png":::

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Virtual WAN, consultez la [FAQ](virtual-wan-faq.md).
* Pour plus d’informations sur le routage de hub virtuel, consultez [À propos du routage de hub virtuel](about-virtual-hub-routing.md).

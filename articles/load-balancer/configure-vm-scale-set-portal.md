---
title: Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant – Portail Azure
description: Découvrez comment configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide du portail Azure.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: how-to
ms.date: 03/25/2020
ms.openlocfilehash: cb7bfb9ac4b10b807ac186d087b0037953abd559
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91439518"
---
# <a name="configure-a-virtual-machine-scale-set-with-an-existing-azure-load-balancer-using-the-azure-portal"></a>Configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant à l’aide du Portail Azure

Dans cet article, vous allez découvrir comment configurer un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant. 

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure.
- Un équilibreur de charge SKU standard existant dans l’abonnement dans lequel le groupe de machines virtuelles identiques sera déployé.
- Un réseau virtuel Azure pour le groupe de machines virtuelles identiques.

## <a name="sign-in-to-the-azure-portal"></a>Connectez-vous au portail Azure.

Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).



## <a name="deploy-virtual-machine-scale-set-with-existing-load-balancer"></a>Déployer un groupe de machines virtuelles identiques avec un équilibreur de charge existant

Dans cette section, vous allez créer un groupe de machines virtuelles identiques dans le Portail Azure avec un équilibreur de charge Azure existant.

> [!NOTE]
> Les étapes suivantes supposent qu’un réseau virtuel nommé **myVNet** et un équilibreur de charge Azure nommé **myLoadBalancer** ont été déployés précédemment.

1. En haut à gauche de l’écran, cliquez sur **Créer une ressource** > **Calculer** > **Groupe de machines virtuelles identiques** ou recherchez **Groupe de machines virtuelles identiques** dans la barre de recherche du marketplace.

2. Sélectionnez **Create** (Créer).

3. Dans **Créer un groupe de machines virtuelles identiques**, entrez ou sélectionnez ces informations dans l’onglet **Général** :

    | Paramètre                        | Valeur                                                                                                 |
    |--------------------------------|-------------------------------------------------------------------------------------------------------|
    | **Détails du projet**            |                                                                                                       |
    | Abonnement                   | Sélectionner votre abonnement Azure                                                                        |
    | Groupe de ressources                 | Sélectionnez Créer, entrez **myResourceGroup**, puis sélectionnez OK ou un groupe de ressources existant. |
    | **Détails du groupe identique**          |                                                                                                       |
    | Nom du groupe de machines virtuelles identiques | Entrez **myVMSS**                                                                                      |
    | Région                         | Sélectionnez **USA Est 2**                                                                                    |
    | Zone de disponibilité              | Sélectionnez **Aucun**                                                                                       |
    | **Détails de l’instance**           |                                                                                                       |
    | Image                          | Sélectionnez **Ubuntu Server 18.04 LTS**                                                                    |
    | Instance Azure Spot            | Sélectionnez **Non**                                                                                         |
    | Taille                           | Conservez les valeurs par défaut                                                                                      |
    | **Compte administrateur**      |                                                                                                       |
    | Type d'authentification            | Sélectionnez **Mot de passe**                                                                                   |
    | Nom d’utilisateur                       | Entrez votre nom d’utilisateur d’administrateur        |
    | Mot de passe                       | Entrez votre mot de passe d’administrateur    |
    | Confirmer le mot de passe               | Entrez de nouveau votre mot de passe d’administrateur |


    :::image type="content" source="./media/vm-scale-sets/create-vm-scale-set-01.png" alt-text="Capture d’écran montrant l’onglet Concepts de base de Créer un groupe de machines virtuelles identiques." border="true":::

4. Sélectionnez l’onglet **Mise en réseau**.

5. Entrez ou sélectionnez ces informations dans l’onglet **Mise en réseau** :

     Paramètre                           | Valeur                                                    |
    |-----------------------------------|----------------------------------------------------------|
    | **Configuration du réseau virtuel** |                                                          |
    | Réseau virtuel                   | Sélectionnez **myVNet** ou votre réseau virtuel existant      |
    | **Équilibrage de charge**                |                                                          |
    | Utilisez un équilibreur de charge               | Sélectionnez **Oui**.                                           |
    | **Paramètres d’équilibrage de charge**       |                                                          |
    | Options d’équilibrage de charge            | Sélectionnez **Équilibreur de charge Azure**                           |
    | Sélectionnez un équilibreur de charge            | Sélectionnez **myLoadBalancer** ou votre équilibreur de charge existant |
    | Sélectionnez un pool principal             | Sélectionnez **myBackendPool** ou votre pool principal existant.  |

    :::image type="content" source="./media/vm-scale-sets/create-vm-scale-set-02.png" alt-text="Capture d’écran montrant l’onglet Concepts de base de Créer un groupe de machines virtuelles identiques." border="true":::

6. Sélectionnez l’onglet **Gestion**.

7. Sous l’onglet **Gestion**, définissez **Diagnostics de démarrage** sur **Désactivé**.

8. Sélectionnez le bouton bleu **Vérifier + créer**.

9. Passez en revue les paramètres, puis sélectionnez le bouton **Créer**.

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez déployé un groupe de machines virtuelles identiques avec un service Azure Load Balancer existant.  Pour en savoir plus sur les groupes de machines virtuelles identiques et l’équilibreur de charge, consultez :

- [Qu’est-ce que Azure Load Balancer ?](load-balancer-overview.md)
- [Que sont les groupes de machines virtuelles identiques ?](../virtual-machine-scale-sets/overview.md)

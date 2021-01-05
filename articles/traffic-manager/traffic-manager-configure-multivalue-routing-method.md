---
title: Configurer le routage du trafic à valeurs multiples – Azure Traffic Manager
description: Cet article explique comment configurer Traffic Manager pour router le trafic vers des points de terminaison A/AAAA.
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/10/2018
ms.author: duau
ms.openlocfilehash: 85e088dda767a6f6c80ac0a9f6eed84e8802e5ee
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95994925"
---
# <a name="configure-multivalue-routing-method-in-traffic-manager"></a>Configurer la méthode de routage à valeurs multiples dans Traffic Manager

Cet article explique comment configurer la méthode de routage du trafic à valeurs multiples. La méthode de routage du trafic à **valeurs multiples** vous permet de retourner plusieurs points de terminaison sains et améliore la fiabilité de votre application car les clients disposent de plus d’options pour réessayer sans avoir à effectuer une autre recherche DNS. Le routage à valeurs multiples n’est activé que pour les profils dont tous les points de terminaison ont été spécifiés par des adresses IPv4 ou IPv6. Lorsqu’une requête est reçue pour ce profil, les points de terminaison sains sont retournés en fonction du nombre maximal de configurable spécifié. 

>[!NOTE]
> Actuellement, l’ajout de points de terminaison avec adresses IPv4 ou IPv6 est pris en charge uniquement pour les points de terminaison de type **externe** et par conséquent, le routage à valeurs multiples est également pris en charge uniquement pour ces points de terminaison.

## <a name="sign-in-to-azure"></a>Connexion à Azure 

Connectez-vous au portail Azure sur https://portal.azure.com.
## <a name="create-a-resource-group"></a>Créer un groupe de ressources
Créez un groupe de ressources pour le profil Traffic Manager.
1. Sur le panneau de gauche du portail Azure, sélectionnez **Groupes de ressources**.
2. Dans **Groupes de ressources**, en haut de la page, sélectionnez **Ajouter**.
3. Dans **Nom du groupe de ressources**, tapez le nom *myResourceGroupTM1*. Dans **Emplacement du groupe de ressources**, sélectionnez **USA Est**, puis **OK**.

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager
Créez un profil Traffic Manager qui dirige le trafic utilisateur en l’envoyant vers le point de terminaison dont la latence est la plus faible.

1. Dans l’angle supérieur gauche de l’écran, cliquez sur **Créer une ressource** > **Mise en réseau** > **Profil Traffic Manager** > **Créer**.
2. Dans **Créer un profil Traffic Manager**, entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis choisissez **Créer** :
    
    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Name                   | Ce nom doit être unique au sein de la zone trafficmanager.net et affiche le nom DNS, trafficmanager.net, qui est utilisé pour accéder à votre profil Traffic Manager.                                   |
    | Méthode de routage          | Sélectionnez la méthode de routage **Valeurs multiples**.                                       |
    | Subscription            | Sélectionnez votre abonnement.                          |
    | Resource group          | Sélectionnez *myResourceGroupTM1*. |
    | Location                | Ce paramètre fait référence à l’emplacement du groupe de ressources et n’a pas d’impact sur le profil Traffic Manager qui sera déployé globalement.                              |
   |        |           | 
  
   ![Créer un profil Traffic Manager](./media/traffic-manager-multivalue-routing-method/create-traffic-manager-profile.png)

## <a name="add-traffic-manager-endpoints"></a>Ajouter des points de terminaison Traffic Manager

Ajoutez deux adresses IP comme point de terminaison externes au profil Traffic Manager à valeurs multiples que vous avez créé à l’étape précédente.

1. Dans la barre de recherche du portail, recherchez le nom du profil Traffic Manager que vous avez créé dans la section précédente et sélectionnez le profil dans les résultats affichés.
2. Dans **Profil Traffic Manager**, dans la section **Paramètres**, cliquez sur **Points de terminaison**, puis sur **Ajouter**.
3. Entrez ou sélectionnez les informations suivantes, acceptez les valeurs par défaut pour les autres paramètres, puis cliquez sur **OK** :

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Type                    | Point de terminaison externe                                   |
    | Name           | myEndpoint1                                        |
    | Nom de domaine complet (FQDN) ou IP           | Saisir l’adresse IP publique du point de terminaison que vous voulez ajouter à ce profil Traffic Manager                         |
    |        |           |

4. Répétez les étapes 2 et 3 pour ajouter un autre point de terminaison appelé *myEndpoint2*. Dans **Nom de domaine complet (FQDN) ou IP**, saisissez l’adresse IP publique du second point de terminaison.
5. Lorsque l’ajout de deux points de terminaison est terminé, ceux-ci s’affichent dans **Profil Traffic Manager** avec leur état de surveillance défini sur **En ligne**.

   ![Ajouter un point de terminaison Traffic Manager](./media/traffic-manager-multivalue-routing-method/add-endpoint.png)
 
## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur la [méthode de routage du trafic par pondération](traffic-manager-configure-weighted-routing-method.md).
- En savoir plus sur la [méthode de routage prioritaire](traffic-manager-configure-priority-routing-method.md).
- En savoir plus sur la [méthode de routage basé sur les performances](traffic-manager-configure-performance-routing-method.md).
- En savoir plus sur la [méthode de routage géographique](traffic-manager-configure-geographic-routing-method.md).



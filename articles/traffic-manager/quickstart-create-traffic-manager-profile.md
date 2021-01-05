---
title: 'Démarrage rapide : Créer un profil pour la haute disponibilité des applications – Portail Azure – Azure Traffic Manager'
description: Cet article de démarrage rapide décrit comment créer un profil Traffic Manager pour générer une application web hautement disponible à l’aide du portail Azure.
services: traffic-manager
author: duongau
manager: twooley
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: traffic-manager
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/15/2020
ms.author: duau
ms.openlocfilehash: 45489d3500a4a744f2aeb34dc21122d180797133
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92101308"
---
# <a name="quickstart-create-a-traffic-manager-profile-using-the-azure-portal"></a>Démarrage rapide : Créer un profil Traffic Manager à l’aide du portail Azure

Ce démarrage rapide explique comment créer un profil Traffic Manager qui assure une haute disponibilité pour votre application web.

Dans ce démarrage rapide, il est question de deux instances d’une application web. Chacune d’elles s’exécute dans une région Azure distincte. Vous allez créer un profil Traffic Manager en fonction de la [priorité du point de terminaison](traffic-manager-routing-methods.md#priority-traffic-routing-method). Le profil dirige le trafic utilisateur vers le site principal exécutant l’application web. Traffic Manager supervise en permanence l’application web. Si le site principal est indisponible, il assure un basculement automatique vers le site de secours.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) maintenant.

## <a name="prerequisites"></a>Prérequis

Pour ce guide de démarrage rapide, vous aurez besoin de deux instances d’une application web déployée dans deux régions Azure différentes ( *USA Est* et *Europe Ouest* ). Chacune servira de point de terminaison principal et de point de terminaison de basculement à Traffic Manager.

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** . Recherchez **Application web** et sélectionnez **Créer** .

1. Dans **Créer une application web** , tapez ou sélectionnez les valeurs suivantes sous l’onglet **De base**  :

    | Paramètre                 | Valeur |
    | ---                     | --- |
    | Abonnement            | Sélectionnez votre abonnement. |    
    | Resource group          | Sélectionnez **Créer** et entrez *myResourceGroupTM1* dans la zone de texte.|
    | Name                    | Attribuez un **Nom** unique à votre application web. Cet exemple utilise *myWebAppEastUS* . |
    | Publish                 | Sélectionnez **Code** . |
    | Pile d’exécution           | Sélectionnez **ASP.NET V4.7** . |
    | Système d'exploitation        | Sélectionnez **Windows** . |
    | Région                  | Sélectionnez **USA Est** . |
    | Plan Windows            | Sélectionnez **Créer** et entrez *myAppServicePlanEastUS* dans la zone de texte. |
    | SKU et taille            | Sélectionnez **Standard S1 100 ACU au total, 1,75 Go de mémoire** . |
   
1. Sélectionnez l’onglet **Surveillance** ou **Suivant : Supervision** .  Sous **Surveillance** , définissez **Application Insights** > **Activer Application Insights** sur **Non** .

1. Sélectionnez **Examiner et créer** .

1. Passez en revue les paramètres, puis sélectionnez **Créer** .  Quand l’application web est déployé correctement, elle crée un site web par défaut.

1. Suivez les étapes 1 à 6 pour créer une deuxième application web nommée *myWebAppWestEurope* . Le nom pour **Groupe de ressources** est *myResourceGroupTM2* , avec pour **Région** *Europe Ouest* et pour **Plan App Service** le nom **myAppServicePlanWestEurope** . Tous les autres paramètres sont les mêmes que pour *myWebAppEastUS* .

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

Créez un profil Traffic Manager qui dirige le trafic utilisateur en fonction de la priorité du point de terminaison.

1. Dans le coin supérieur gauche de l’écran, sélectionnez **Créer une ressource** . Ensuite, recherchez **Profil Traffic Manager** et sélectionnez **Créer** .
1. Dans **Créer un profil Traffic Manager** , entrez ou sélectionnez ces paramètres :

    | Paramètre | Valeur |
    | --------| ----- |
    | Name | Attribuez un nom unique à votre profil Traffic Manager.|
    | Méthode de routage | Sélectionnez **Priorité** .|
    | Abonnement | Sélectionnez l’abonnement auquel vous souhaitez appliquer le profil Traffic Manager. |
    | Resource group | Sélectionnez *myResourceGroupTM1* .|
    | Location |Ce paramètre fait référence à l’emplacement du groupe de ressources. Il n’a aucun effet sur le profil Traffic Manager qui sera déployé globalement.|

1. Sélectionnez **Create** (Créer).

## <a name="add-traffic-manager-endpoints"></a>Ajouter des points de terminaison Traffic Manager

Ajoutez le site web dans la région *USA Est* en tant que point de terminaison principal pour acheminer tout le trafic utilisateur. Ajoutez le site web dans la région *Europe Ouest* comme point de terminaison de basculement. Quand le point de terminaison principal n’est pas disponible, le trafic est automatiquement routé vers le point de terminaison de basculement.

1. Dans la barre de recherche du portail, entrez le nom du profil Traffic Manager que vous avez créé dans la section précédente.
1. Sélectionnez le profil dans les résultats de recherche.
1. Dans **Profil Traffic Manager** , dans la section **Paramètres** , sélectionnez **Points de terminaison** , puis sélectionnez **Ajouter** .

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-endpoint-menu.png" alt-text="Paramètres des points de terminaison dans le profil Traffic Manager":::

1. Entrez ou sélectionnez ces paramètres :

    | Paramètre | Valeur |
    | ------- | ------|
    | Type | Sélectionnez **Point de terminaison Azure** . |
    | Name | Entrez *myPrimaryEndpoint* . |
    | Type de ressource cible | Sélectionner **App Service** . |
    | Ressource cible | Sélectionnez **Choisir un service d’application** > **USA Est** . |
    | Priority | Sélectionnez **1** . Tout le trafic se dirige vers ce point de terminaison quand il est sain. |

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/add-traffic-manager-endpoint.png" alt-text="Paramètres des points de terminaison dans le profil Traffic Manager":::
    
1. Sélectionnez **OK** .
1. Pour créer un point de terminaison de basculement pour votre deuxième région Azure, répétez les étapes 3 et 4 avec ces paramètres :

    | Paramètre | Valeur |
    | ------- | ------|
    | Type | Sélectionnez **Point de terminaison Azure** . |
    | Name | Entrez *myFailoverEndpoint* . |
    | Type de ressource cible | Sélectionner **App Service** . |
    | Ressource cible | Sélectionnez **Choisir un service d’application** > **Europe Ouest** . |
    | Priority | Sélectionnez **2** . Tout le trafic se dirige vers ce point de terminaison de basculement si le point de terminaison principal n’est pas sain. |

1. Sélectionnez **OK** .

Une fois que vous avez ajouté les deux points de terminaison, il figurent dans **Profil Traffic Manager** . Notez que leur état de supervision est maintenant **En ligne** .

## <a name="test-traffic-manager-profile"></a>Tester le profil Traffic Manager

Dans cette section, vous allez vérifier le nom de domaine de votre profil Traffic Manager. Vous allez aussi configurer le point de terminaison principal pour le rendre indisponible. Enfin, vous allez pouvoir constater que l’application est toujours disponible. Cela est dû au fait que Traffic Manager envoie le trafic au point de terminaison de basculement.

### <a name="check-the-dns-name"></a>Vérifier le nom DNS

1. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous avez créé dans la section précédente.
1. Sélectionnez le profil Traffic Manager. La **Vue d’ensemble** s’affiche.
1. Le **profil Traffic Manager** affiche le nom DNS de votre profil Traffic Manager nouvellement créé.
  
    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-dns-name.png" alt-text="Paramètres des points de terminaison dans le profil Traffic Manager":::

### <a name="view-traffic-manager-in-action"></a>Afficher Traffic Manager en action

1. Dans un navigateur web, entrez le nom DNS de votre profil Traffic Manager pour afficher le site web par défaut de votre application web.

    > [!NOTE]
    > Dans ce scénario de démarrage rapide, toutes les demandes sont routées vers le point de terminaison principal. Il est défini sur **Priorité 1** .

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-test.png" alt-text="Paramètres des points de terminaison dans le profil Traffic Manager":::

1. Pour voir le basculement de Traffic Manager en action, désactivez votre site principal :
    1. Dans la page Profil Traffic Manager, à partir de la **Vue d’ensemble** , sélectionnez **myPrimaryEndpoint** .
    1. Dans *MyPrimaryEndpoint* , sélectionnez **Désactivé** > **Enregistrer** .
    1. Fermez **myPrimaryEndpoint** . Notez que l’état est maintenant **Désactivé** .
1. Copiez le nom DNS de votre profil Traffic Manager de l’étape précédente pour afficher le site web dans une nouvelle session de navigateur web.
1. Vérifiez que l’application web est toujours disponible.

Le point de terminaison principal n’étant pas disponible, vous avez été routé vers le point de terminaison de basculement.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé, supprimez les groupes de ressources, les applications web et toutes les ressources associées. Pour ce faire, sélectionnez individuellement chaque élément dans le tableau de bord, puis sélectionnez **Supprimer** en haut de chaque page.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un profil Traffic Manager. Celui-ci vous permet de diriger le trafic utilisateur pour les applications web à haute disponibilité. Pour plus d’informations sur le routage du trafic, passez aux tutoriels Traffic Manager.

> [!div class="nextstepaction"]
> [Didacticiels Traffic Manager](tutorial-traffic-manager-improve-website-response.md)

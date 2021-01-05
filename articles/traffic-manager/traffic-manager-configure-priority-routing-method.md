---
title: 'Tutoriel : Configurer le routage du trafic prioritaire avec Azure Traffic Manager'
description: Ce tutoriel explique comment configurer la méthode de routage du trafic prioritaire dans Traffic Manager
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/16/2020
ms.author: duau
ms.openlocfilehash: 1835377f4690097c8390957bf7d897242ba7aace
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92208054"
---
# <a name="tutorial-configure-priority-traffic-routing-method-in-traffic-manager"></a>Tutoriel : Configurer la méthode de routage du trafic prioritaire dans Traffic Manager

Ce tutoriel explique comment utiliser Azure Traffic Manager pour router le trafic utilisateur vers des points de terminaison à l’aide de la méthode de routage prioritaire. Avec cette méthode de routage, vous définissez l’ordre de chaque point de terminaison dans la configuration du profil Traffic Manager. Le trafic utilisateur sera routé vers le point de terminaison selon l’ordre configuré. Cette méthode de routage est utile si vous souhaitez configurer le basculement du service. Le point de terminaison principal reçoit le numéro de priorité « 1 », puis traite toutes les requêtes entrantes. Les points de terminaison de priorité moins élevée feront office de sauvegardes.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> - Créer un profil Traffic Manager basé sur le routage prioritaire
> - Ajouter des points de terminaison
> - Configurer la priorité des points de terminaison
> - Utiliser un profil Traffic Manager
> - Supprimer un profil Traffic Manager

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="to-configure-the-priority-traffic-routing-method"></a>Pour configurer une méthode de routage de trafic prioritaire
1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).

1. Sur la gauche, sélectionnez **+ Créer une ressource** . Recherchez **Profil Traffic Manager** , puis sélectionnez **Créer** .

    :::image type="content" source="./media/traffic-manager-priority-routing-method/create-traffic-manager-priority-profile.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Dans la page *Créer un profil Traffic Manager* , configurez les paramètres suivants :

    | Paramètre         | Value                                              |
    | ---             | ---                                                |
    | Name            | Entrez un nom pour votre profil. Ce nom doit être unique dans la zone trafficmanager.net. Pour accéder à votre profil Traffic Manager, vous utilisez le nom DNS `<profilename>.trafficmanager.net`. |    
    | Méthode de routage  | Sélectionnez **Priorité** . |
    | Abonnement    | Sélectionnez votre abonnement. |
    | Resource group   | Utilisez un groupe de ressources existant ou créez-en un où placer ce profil. Si vous choisissez de créer un nouveau groupe de ressources, utilisez la liste déroulante *Emplacement du groupe de ressources* pour spécifier l’emplacement du groupe de ressources. Ce paramètre fait référence à l’emplacement du groupe de ressources et n’a pas d’impact sur le profil Traffic Manager qui est déployé globalement. |

1. Sélectionnez **Créer** pour déployer votre profil Traffic Manager.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/create-traffic-manager-profile-priority.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

## <a name="add-endpoints"></a>Ajouter des points de terminaison

1. Sélectionnez le profil Traffic Manager dans la liste.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/traffic-manager-profile-list.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Sélectionnez **Points de terminaison** sous *Paramètres* , puis sélectionnez **+ Ajouter** pour ajouter un nouveau point de terminaison.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/traffic-manager-add-endpoints.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Sélectionnez ou saisissez les paramètres suivants : 

    | Paramètre                | Valeur                                              |
    | ---                    | ---                                                |
    | Type                   | Sélectionnez le type du point de terminaison. |    
    | Name                   | Attribuez un nom au point de terminaison pour permettre son identification. |
    | Type de ressource cible   | Sélectionnez le type de ressource pour la cible. |
    | Ressource cible        | Sélectionnez la ressource dans la liste. |
    | Priority               | Attribuez un numéro de priorité à ce point de terminaison. 1 correspond à la priorité la plus élevée. |


1. Sélectionnez **Ajouter** pour ajouter le point de terminaison. Répétez les étapes 2 et 3 pour ajouter des points de terminaison supplémentaires. N’oubliez pas de définir un numéro de priorité adapté.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/add-endpoint.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Dans la page **Points de terminaison** , passez en revue l’ordre de priorité des points de terminaison. Lorsque vous configurez la méthode de routage du trafic par **Priorité** , l’ordre des points de terminaison sélectionnés est important. Vérifiez l’ordre de priorité des points de terminaison.  Le point de terminaison principal doit se trouver en haut de la liste. Vérifiez l’ordre dans lequel il est affiché. Toutes les requêtes seront routées vers le premier point de terminaison, et si Traffic Manager détecte que celui-ci n’est pas sain, le trafic bascule automatiquement vers le point de terminaison suivant. 

    :::image type="content" source="./media/traffic-manager-priority-routing-method/endpoints-list.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Pour changer l’ordre de priorité des points de terminaison, sélectionnez un point de terminaison, changez sa priorité, puis sélectionnez **Enregistrer** pour enregistrer les paramètres de point de terminaison.

## <a name="use-the-traffic-manager-profile"></a>Utiliser le profil Traffic Manager

1.  Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous avez créé dans la section précédente, puis sélectionnez le profil Traffic Manager dans les résultats affichés.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/search-traffic-manager-profile.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1.  La page de vue d’ensemble du **profil Traffic Manager** affiche le nom DNS du profil Traffic Manager que vous venez de créer. Celui-ci peut être utilisé par tous les clients (par exemple, en y accédant à l’aide d’un navigateur web) pour être acheminés vers le point de terminaison correct, comme déterminé par le type de routage. Dans ce cas, toutes les requêtes sont routées vers le premier point de terminaison, et si Traffic Manager détecte que celui-ci n’est pas sain, le trafic bascule automatiquement vers le point de terminaison suivant.

    :::image type="content" source="./media/traffic-manager-priority-routing-method/traffic-manager-profile-dns-name.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

1. Une fois le profil Traffic Manager opérationnel, modifiez l’enregistrement DNS sur le serveur DNS faisant autorité, afin de faire pointer votre nom de domaine d’entreprise vers le nom de domaine Traffic Manager.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin du profil Traffic Manager, recherchez le profil, puis sélectionnez **Supprimer le profil** .

:::image type="content" source="./media/traffic-manager-priority-routing-method/traffic-manager-delete-priority-profile.png" alt-text="Créer un profil Traffic Manager avec routage prioritaire":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la méthode de routage prioritaire, consultez :

> [!div class="nextstepaction"]
> [Méthode de routage prioritaire](traffic-manager-routing-methods.md#priority-traffic-routing-method)

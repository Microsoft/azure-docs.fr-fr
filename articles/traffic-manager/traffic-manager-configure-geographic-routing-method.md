---
title: 'Tutoriel : Configurer le routage du trafic géographique avec Azure Traffic Manager'
description: Ce tutoriel explique comment configurer la méthode de routage du trafic géographique avec Azure Traffic Manager
services: traffic-manager
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/15/2020
ms.author: duau
ms.openlocfilehash: 29b3cdde328a994e5806df810db15b529a6da9af
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96188664"
---
# <a name="tutorial-configure-the-geographic-traffic-routing-method-using-traffic-manager"></a>Tutoriel : Configurer la méthode de routage du trafic géographique à l’aide de Traffic Manager

La méthode de routage du trafic géographique vous permet de diriger le trafic vers des points de terminaison spécifiques en fonction de l’emplacement géographique d’où proviennent les demandes. Ce didacticiel vous montre comment créer un profil Traffic Manager avec cette méthode de routage et configurer les points de terminaison pour recevoir le trafic à partir de zones géographiques spécifiques.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Créer un profil Traffic Manager basé sur le routage géographique
> - Configurer un point de terminaison imbriqué
> - Utiliser un profil Traffic Manager
> - Supprimer un profil Traffic Manager

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-a-traffic-manager-profile"></a>Créer un profil Traffic Manager

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).

1. Sur la gauche, sélectionnez **+ Créer une ressource**. Recherchez **Profil Traffic Manager**, puis sélectionnez **Créer**.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/create-traffic-manager.png" alt-text="Créer un profil Traffic Manager":::

1. Dans la page *Créer un profil Traffic Manager*,configurez les paramètres suivants :

    | Paramètre         | Valeur                                              |
    | ---             | ---                                                |
    | Name            | Entrez un nom pour votre profil. Ce nom doit être unique dans la zone trafficmanager.net. Pour accéder à votre profil Traffic Manager, vous utilisez le nom DNS `<profilename>.trafficmanager.net`. |    
    | Méthode de routage  | Sélectionnez **Géographique**. |
    | Abonnement    | Sélectionnez votre abonnement. |
    | Resource group   | Utilisez un groupe de ressources existant ou créez-en un où placer ce profil. Si vous choisissez de créer un nouveau groupe de ressources, utilisez la liste déroulante *Emplacement du groupe de ressources* pour spécifier l’emplacement du groupe de ressources. Ce paramètre fait référence à l’emplacement du groupe de ressources et n’a pas d’impact sur le profil Traffic Manager qui est déployé globalement. |

1. Sélectionnez **Créer** pour déployer votre profil Traffic Manager.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/create-traffic-manager-profile.png" alt-text="Page Créer un profil Traffic Manager":::

## <a name="add-endpoints"></a>Ajouter des points de terminaison

1. Sélectionnez le profil Traffic Manager dans la liste.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/traffic-manager-list-geographic.png" alt-text="Liste des profils Traffic Manager avec routage géographique":::

1. Sélectionnez **Points de terminaison** sous *Paramètres*, puis sélectionnez **+ Ajouter** pour ajouter un nouveau point de terminaison.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/add-geographic-endpoint.png" alt-text="Ajouter des points de terminaison":::

1. Sélectionnez ou saisissez les paramètres suivants : 

    | Paramètre                | Valeur                                              |
    | ---                    | ---                                                |
    | Type                   | Sélectionnez le type du point de terminaison. Pour les profils de routage géographique utilisés en production, nous vous recommandons d’utiliser des types de point de terminaison imbriqués contenant un profil enfant avec plus d’un point de terminaison. Pour plus d’informations, consultez [FAQ sur les méthodes de routage du trafic géographique](traffic-manager-FAQs.md). |    
    | Name                   | Attribuez un nom au point de terminaison pour permettre son identification. |
    | Type de ressource cible   | Sélectionnez le type de ressource pour la cible. |
    | Ressource cible        | Sélectionnez la ressource dans la liste. |

    > [!Note]
    > Certains champs de cette page varient selon le type de point de terminaison que vous ajoutez :
    > 1. Si vous ajoutez un point de terminaison Azure, sélectionnez le **type de ressource cible** et la **cible** en fonction de la ressource vers laquelle vous souhaitez diriger le trafic
    > 1. Si vous ajoutez un point de terminaison **externe**, entrez le **nom de domaine complet (FQDN)** pour votre point de terminaison.
    > 1. Si vous ajoutez un **point de terminaison imbriqué**, sélectionnez la **ressource cible** qui correspond au profil enfant à utiliser et spécifiez le **nombre minimal de points de terminaison enfants**.

1. Dans la section *Géocartographie*, utilisez la liste déroulante pour ajouter les régions à partir desquelles le trafic doit être envoyé vers ce point de terminaison. Vous devez ajouter au moins une région. Il est possible de mapper plusieurs régions.

1. Répétez l’étape précédente pour tous les points de terminaison que vous souhaitez ajouter à ce profil, puis sélectionnez **Enregistrer**.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/traffic-manager-add-endpoint.png" alt-text="Ajouter un point de terminaison Traffic Manager":::

## <a name="use-the-traffic-manager-profile"></a>Utiliser le profil Traffic Manager

1.  Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous avez créé dans la section précédente, puis sélectionnez le profil Traffic Manager dans les résultats affichés.
    
    :::image type="content" source="./media/traffic-manager-geographic-routing-method/search-traffic-manager-profile.png" alt-text="Recherche d’un profil Traffic Manager":::

1. Le **profil Traffic Manager** affiche le nom DNS de votre profil Traffic Manager nouvellement créé. Tous les clients peuvent utiliser ce nom (par exemple, en y accédant à l’aide d’un navigateur web) en vue d’être routés vers le bon point de terminaison, selon le type de routage. Avec le routage géographique, Traffic Manager examine l’adresse IP source de la requête entrante et détermine la région d’où elle provient. Si cette région est mappée à un point de terminaison, le trafic est acheminé vers cet emplacement. Si cette région n’est pas mappée à un point de terminaison, Traffic Manager retourne une réponse NODATA.

    :::image type="content" source="./media/traffic-manager-geographic-routing-method/traffic-manager-geographic-overview.png" alt-text="Vue d’ensemble du routage géographique Traffic Manager":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin du profil Traffic Manager, recherchez le profil, puis sélectionnez **Supprimer le profil**.

:::image type="content" source="./media/traffic-manager-geographic-routing-method/delete-traffic-manager-profile.png" alt-text="Suppression d’un profil Traffic Manager":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la méthode de routage géographique, consultez :

> [!div class="nextstepaction"]
> [Méthode de routage du trafic Geographic (Géographique)](traffic-manager-routing-methods.md#geographic)

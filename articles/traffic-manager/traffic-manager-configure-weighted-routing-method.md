---
title: 'Tutoriel : Configurer le routage du trafic en tourniquet par pondération avec Azure Traffic Manager'
description: Ce tutoriel explique comment équilibrer le trafic en utilisant une méthode en tourniquet (round robin) dans Traffic Manager
services: traffic-manager
documentationcenter: ''
author: duongau
ms.service: traffic-manager
ms.devlang: na
ms.topic: tutorial
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 10/19/2020
ms.author: duau
ms.openlocfilehash: abcfce43b90c7371d5b38aa5b7a6d478e9d6a0dd
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92207824"
---
# <a name="tutorial-configure-the-weighted-traffic-routing-method-in-traffic-manager"></a>Tutoriel : Configurer la méthode de routage du trafic par pondération dans Traffic Manager

Il est courant d'utiliser un ensemble de points de terminaison identiques, comprenant des services cloud et des sites web, et de répartir le trafic à parts égales entre eux. Les étapes suivantes décrivent comment configurer ce type de méthode de routage du trafic.

> [!NOTE]
> Azure Web App propose déjà une fonctionnalité d’équilibrage de charge en tourniquet (round robin) pour les sites web d’une région Azure (comportant potentiellement plusieurs centres de données). Traffic Manager permet de répartir le trafic entre les sites web de différents centres de données.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - Créer un profil Traffic Manager basé sur le routage pondéré
> - Utiliser un profil Traffic Manager
> - Supprimer un profil Traffic Manager

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/).

## <a name="configure-the-weighted-traffic-routing-method"></a>Configurer la méthode de routage du trafic par pondération

1. Dans un navigateur, connectez-vous au [portail Azure](https://portal.azure.com).

1. Dans la barre de recherche du portail, recherchez le nom du **profil Traffic Manager** que vous avez créé dans la section précédente, puis sélectionnez le profil Traffic Manager dans les résultats affichés.

    :::image type="content" source="./media/traffic-manager-weighted-routing-method/search-traffic-manager-weighted-profile.png" alt-text="Recherche du profil Traffic Manager":::

1. Sélectionnez **Configuration** , puis sélectionnez ou saisissez les paramètres suivants :

    | Paramètre         | Value                                              |
    | ---             | ---                                                |
    | Méthode de routage            | Sélectionnez **Pondéré** . |    
    | Durée de vie du DNS (TTL) | Cette valeur contrôle la fréquence à laquelle le serveur de noms de mise en cache locale du client interroge le système Traffic Manager au sujet des entrées DNS mises à jour. Toute modification dans Traffic Manager, telle que le changement de la méthode de routage du trafic ou de la disponibilité des points de terminaison ajoutés, sera actualisée au terme de cette période dans le système global des serveurs DNS. |
    | Protocol    | Sélectionnez un protocole pour la supervision des points de terminaison. *Options : HTTP, HTTPS et TCP* |
    | Port | Spécifiez le numéro de port. |
    | Chemin d’accès | Pour surveiller les points de terminaison, vous devez indiquer un chemin et un nom de fichier. Une barre oblique (« / ») est une entrée valide pour le chemin d’accès relatif. Elle implique que le fichier se trouve dans le répertoire racine (par défaut). |
    | Paramètres d’en-tête personnalisé | Configurez les en-têtes personnalisés au format suivant : host:contoso.com,newheader:newvalue. Le nombre maximal de paires est de 8. S’applique aux protocoles HTTP et HTTPS. S’applique à tous les points de terminaison du profil |
    | Plages de codes d’état attendues (par défaut : 200) | Configurez les plages de codes d’état au format suivant : 200-299,301-301. Le nombre maximal de plages est de 8. S’applique aux protocoles HTTP et HTTPS. S’applique à tous les points de terminaison du profil |
    | Intervalle de sondage | Configurez l’intervalle de temps entre chaque sonde d’intégrité de point de terminaison. Vous pouvez choisir 10 ou 30 secondes. |
    | Nombre d’échecs tolérés | Configurez le nombre d’échecs de sonde d’intégrité tolérés avant le déclenchement d’un échec du point de terminaison. Vous pouvez entrer un nombre entre 0 et 9. | 
    | Délai d’expiration de la sonde | Configurez le délai d’expiration pour les sondes d’intégrité du point de terminaison. Cette valeur doit être au moins égale à 5 et inférieure à la valeur de l’intervalle de sondage. |

1. Sélectionnez **Enregistrer** pour terminer la configuration.

    :::image type="content" source="./media/traffic-manager-weighted-routing-method/traffic-manager-weighted-configuration.png" alt-text="Recherche du profil Traffic Manager"::: 

1. Sélectionnez **Point de terminaison** , puis configurez le poids de chaque point de terminaison. Le poids peut être compris entre 1 et 1 000. Plus le poids est élevé, plus la priorité l’est également.  

    :::image type="content" source="./media/traffic-manager-weighted-routing-method/traffic-manager-configure-endpoints-weighted.png" alt-text="Recherche du profil Traffic Manager"::: 

## <a name="use-the-traffic-manager-profile"></a>Utiliser le profil Traffic Manager

Le **profil Traffic Manager** affiche le nom DNS de votre profil Traffic Manager nouvellement créé. Tous les clients peuvent utiliser ce nom (par exemple, en y accédant à l’aide d’un navigateur web) en vue d’être routés vers le bon point de terminaison, selon le type de routage. Dans ce cas, toutes les requêtes sont routées vers chaque point de terminaison de manière alternée.

:::image type="content" source="./media/traffic-manager-weighted-routing-method/traffic-manager-weighted-overview.png" alt-text="Recherche du profil Traffic Manager"::: 

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin du profil Traffic Manager, recherchez le profil, puis sélectionnez **Supprimer le profil** .

:::image type="content" source="./media/traffic-manager-weighted-routing-method/delete-traffic-manager-weighted-profile.png" alt-text="Recherche du profil Traffic Manager":::

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la méthode de routage pondéré, consultez :

> [!div class="nextstepaction"]
> [Méthode de routage du trafic basée sur la pondération](traffic-manager-routing-methods.md#weighted)
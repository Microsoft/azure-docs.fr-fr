---
title: 'Démarrage rapide : Configurer la haute disponibilité avec Azure Front Door Service - Portail Azure'
description: Ce guide de démarrage rapide montre comment utiliser Azure Front Door Service pour votre application web globale hautement disponible et très performante à l’aide du portail Azure.
services: front-door
documentationcenter: na
author: duongau
manager: KumudD
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/16/2020
ms.author: duau
ms.openlocfilehash: 4846438f8479fe622570aa515a4d8b40cccc57b8
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91252278"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application"></a>Démarrage rapide : Créer une porte d’entrée pour une application web globale hautement disponible

Commencez à utiliser Azure Front Door à l’aide du portail Azure pour configurer la haute disponibilité pour une application web.

Dans ce démarrage rapide, Azure Front Door regroupe deux instances d’une application web qui s’exécutent dans différentes régions Azure. Vous créez une configuration Front Door basée sur des back-ends de pondération et de priorité égales. Cette configuration dirige le trafic vers le site le plus proche qui exécute l’application. Azure Front Door supervise en permanence l’application web. Le service assure le basculement automatique vers le site disponible suivant quand le site le plus proche n’est pas disponible.

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

## <a name="create-two-instances-of-a-web-app"></a>Créer deux instances d’une application web

Ce guide de démarrage rapide demande deux instances d’une application web qui s’exécutent dans différentes régions Azure. Les deux instances de l’application web s’exécutent en mode *Actif/Actif*, donc l’une ou l’autre peut s’occuper du trafic. Cette configuration diffère d’une configuration *Active/Veille* où l’une sert de basculement.

Si vous n’avez pas encore d’application web, utilisez les étapes suivantes pour configurer des exemples d’applications web.

1. Connectez-vous au portail Azure sur https://portal.azure.com.

1. En haut à gauche de l’écran, sélectionnez **Créer une ressource** >  **Application web**.

    :::image type="content" source="media/quickstart-create-front-door/front-door-create-web-app.png" alt-text="Créer une application web dans le portail Azure":::

1. Sous l’onglet **Général** de la page **Application web**, entrez ou sélectionnez les informations suivantes.

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | **Abonnement**               | Sélectionnez votre abonnement. |    
    | **Groupe de ressources**       | Sélectionnez **Créer**, puis entrez *FrontDoorQS_rg1* dans la zone de texte.|
    | **Nom**                   | Attribuez un **Nom** unique à votre application web. Cet exemple utilise *WebAppContoso-1*. |
    | **Publier** | Sélectionnez **Code**. |
    | **Pile d’exécution**         | Sélectionnez **.NET Core 2.1 (LTS)** . |
    | **Système d’exploitation**          | Sélectionnez **Windows**. |
    | **Région**           | Sélectionnez **USA Centre**. |
    | **Plan Windows** | Sélectionnez **Créer** et entrez *myAppServicePlanCentralUS* dans la zone de texte. |
    | **Référence et taille** | Sélectionnez **Standard S1 100 ACU au total, 1,75 Go de mémoire**. |

1. Sélectionnez **Vérifier + Créer**, vérifiez le **Résumé**, puis sélectionnez **Créer**. Le déploiement peut prendre plusieurs minutes.

    :::image type="content" source="media/quickstart-create-front-door/create-web-app.png" alt-text="Créer une application web dans le portail Azure":::

Une fois le déploiement terminé, créez une seconde application web. Utilisez la même procédure avec les mêmes valeurs, à l’exception des valeurs suivantes :

| Paramètre          | Valeur     |
| ---              | ---  |
| **Groupe de ressources**   | Sélectionnez **Créer** et entrez *FrontDoorQS_rg2* |
| **Nom**             | Entrez un nom unique pour votre application web, dans cet exemple, *WebAppContoso-2*  |
| **Région**           | Une autre région, dans cet exemple, *USA Centre Sud* |
| **Plan App Service** > **Plan Windows**         | Sélectionnez **Nouveau** et entrez *myAppServicePlanSouthCentralUS*, puis sélectionnez **OK** |

## <a name="create-a-front-door-for-your-application"></a>Créer une porte d’entrée pour votre application

Configurez Azure Front Door pour diriger le trafic utilisateur selon la plus petite latence entre les serveurs des deux applications web. Pour commencer, ajoutez un hôte front-end pour Azure Front Door.

1. Dans la page d’accueil ou le menu Azure, sélectionnez **Créer une ressource**. Sélectionnez **Réseau** > **Afficher tout** > **Porte d’entrée**.

1. Sous l’onglet **Général** de la page **Créer une porte d’entrée**, entrez ou sélectionnez les informations suivantes, puis sélectionnez **Suivant : Configuration**.

    | Paramètre | Valeur |
    | --- | --- |
    | **Abonnement** | Sélectionnez votre abonnement. |    
    | **Groupe de ressources** | Sélectionnez **Créer**, puis entrez *FrontDoorQS_rg0* dans la zone de texte.|
    | **Emplacement du groupe de ressources** | Sélectionnez **USA Centre**. |

1. Dans **Front-ends/domaines**, sélectionnez **+** pour ouvrir **Ajouter l’hôte front-end**.

1. Pour **Nom d’hôte**, entrez un nom d’hôte global unique. Cet exemple utilise *contoso-frontend*. Sélectionnez **Ajouter**.

    :::image type="content" source="media/quickstart-create-front-door/add-frontend-host-azure-front-door.png" alt-text="Créer une application web dans le portail Azure":::

Ensuite, créez un pool de back-ends qui contient vos deux applications web.

1. Toujours dans **Créer une porte d’entrée**, dans **Pools principaux**, sélectionnez **+** pour ouvrir **Ajouter un pool principal**.

1. Pour **Nom**, entrez *myBackendPool*, puis sélectionnez **Ajouter un backend**.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-backend-pool.png" alt-text="Créer une application web dans le portail Azure":::

1. Dans le panneau **Ajouter un backend**, sélectionnez les informations suivantes, puis sélectionnez **Ajouter**.

    | Paramètre | Valeur |
    | --- | --- |
    | **Type de l’hôte backend** | Sélectionner **App Service**. |   
    | **Abonnement** | Sélectionnez votre abonnement. |    
    | **Nom de l’hôte backend** | Sélectionnez la première application web que vous avez créée. Dans cet exemple, l’application web était *WebAppContoso-1*. |

    **Laissez tous les autres champs sur leurs valeurs par défaut.*

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-a-backend.png" alt-text="Créer une application web dans le portail Azure":::

1. Sélectionnez **Ajouter un pool principal** de nouveau. Sélectionnez les informations suivantes, puis sélectionnez **Ajouter**.

    | Paramètre | Valeur |
    | --- | --- |
    | **Type de l’hôte backend** | Sélectionner **App Service**. |   
    | **Abonnement** | Sélectionnez votre abonnement. |    
    | **Nom de l’hôte backend** | Sélectionnez la deuxième application web que vous avez créée. Dans cet exemple, l’application web était *WebAppContoso-2*. |

    **Laissez tous les autres champs sur leurs valeurs par défaut.*

1. Sélectionnez **Ajouter** dans le panneau **Ajouter un pool de back-ends** pour terminer la configuration du pool de back-ends.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-backend-pool-complete.png" alt-text="Créer une application web dans le portail Azure":::

Enfin, ajoutez une règle de routage. Une règle de routage mappe votre hôte front-end au pool de back-ends. La règle transfère une demande pour `contoso-frontend.azurefd.net` à **myBackendPool**

1. Toujours dans **Créer une porte d’entrée**, dans **Règles de routage**, sélectionnez **+** pour configurer une règle de routage.

1. Dans **Ajouter une règle**, pour **Nom**, entrez *LocationRule*. Acceptez toutes les valeurs par défaut, puis sélectionnez **Ajouter** pour ajouter la règle de routage.

    :::image type="content" source="media/quickstart-create-front-door/front-door-add-a-rule.png" alt-text="Créer une application web dans le portail Azure":::

   >[!WARNING]
   > Vous **devez** vérifier que chaque hôte front-end dans votre porte d’entrée a une règle de routage avec un chemin par défaut (`\*`) associé. En d’autres termes, parmi toutes vos règles de routage, il doit exister au moins une règle de routage pour chacun des hôtes front-end définis dans le chemin par défaut (`\*`). Sinon, le trafic de l’utilisateur final risque de ne pas être routé correctement.

1. Sélectionnez **Vérifier + créer**, puis **Créer**.

    :::image type="content" source="media/quickstart-create-front-door/configuration-azure-front-door.png" alt-text="Créer une application web dans le portail Azure":::

## <a name="view-azure-front-door-in-action"></a>Voir Azure Front Door à l’œuvre

Une fois la porte d’entrée créée, le déploiement global de la configuration prend quelques minutes. Une fois l’opération terminée, accédez à l’hôte front-end que vous avez créé. Dans un navigateur, accédez à `contoso-frontend.azurefd.net`. Votre demande est automatiquement routée vers le serveur le plus proche des serveurs spécifiés dans le pool de back-ends.

Si vous avez créé ces applications dans ce guide de démarrage rapide, vous verrez une page d’informations.

Pour tester le basculement global instantané, essayez les étapes suivantes :

1. Ouvrez un navigateur, comme décrit ci-dessus, puis accédez à l’adresse front-end : `contoso-frontend.azurefd.net`.

1. Dans le portail Azure, recherchez et sélectionnez *App Services*. Faites défiler la liste jusqu’à l’une de vos applications web, **WebAppContoso-1** dans cet exemple.

1. Sélectionnez votre application web, puis **Arrêter** et **Oui** pour vérifier.

1. Actualisez votre navigateur. Vous devriez voir la même page d’informations.

   >[!TIP]
   >Il y a un peu de retard pour ces actions. Vous devrez peut-être actualiser une nouvelle fois.

1. Recherchez l’autre application web, puis arrêtez-la également.

1. Actualisez votre navigateur. Cette fois, vous devriez voir un message d’erreur.

   :::image type="content" source="media/quickstart-create-front-door/web-app-stopped-message.png" alt-text="Créer une application web dans le portail Azure":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Une fois que vous avez terminé, vous pouvez supprimer tous les éléments que vous avez créés. La suppression d’un groupe de ressources supprime également son contenu. Si vous n’envisagez pas d’utiliser cette porte d’entrée, vous devez supprimer les ressources pour éviter des frais inutiles.

1. Dans le portail Azure, sélectionnez et recherchez **Groupes de ressources** ou sélectionnez **Groupes de ressources** dans le menu du portail Azure.

1. Filtrez ou faites défiler la liste pour rechercher un groupe de ressources, par exemple **FrontDoorQS_rg0**.

1. Sélectionnez le groupe de ressources, puis **Supprimer le groupe de ressources**.

   >[!WARNING]
   >Cette action est irréversible.

1. Tapez le nom du groupe de ressources pour vérifier, puis sélectionnez **Supprimer**.

Répétez la procédure pour les deux autres groupes.

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour apprendre à ajouter un domaine personnalisé à votre porte d’entrée.
> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](front-door-custom-domain.md)

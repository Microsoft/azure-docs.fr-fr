---
title: 'Tutoriel : Créer un équilibreur de charge interrégional à l’aide du portail Azure'
titleSuffix: Azure Load Balancer
description: Démarrez avec ce tutoriel pour déployer un équilibreur de charge Azure sur plusieurs régions avec le portail Azure.
author: asudbring
ms.author: allensu
ms.service: load-balancer
ms.topic: tutorial
ms.date: 11/24/2020
ms.openlocfilehash: 7ee203595a796529ae0aefe8b0c52a689ac29968
ms.sourcegitcommit: d6e92295e1f161a547da33999ad66c94cf334563
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96762327"
---
# <a name="tutorial-create-a-cross-region-azure-load-balancer-using-the-azure-portal"></a>Tutoriel : Créer un équilibreur de charge Azure inter-région à l’aide du portail Azure

Un équilibreur de charge interrégional garantit la disponibilité mondiale du service dans plusieurs régions Azure. En cas de défaillance d'une région, le trafic est acheminé vers l'équilibreur de charge régional sain le plus proche.  

Dans ce tutoriel, vous apprenez à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer un équilibreur de charge interrégional.
> * Créer un pool de back-ends contenant deux équilibreurs de charge régionaux.
> * Créez une règle d’équilibreur de charge.
> * Tester l’équilibreur de charge.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

> [!IMPORTANT]
> La réplication Azure Load Balancer entre plusieurs régions est disponible en préversion publique.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge. Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure.
- Deux équilibreurs de charge Azure de SKU **standard** avec des pools de back-ends déployés dans deux régions Azure différentes.
    - Pour plus d’informations sur la création d’un équilibreur de charge standard régional et de machines virtuelles pour les pools de back-ends, consultez [Démarrage rapide : Créer un équilibreur de charge public pour équilibrer la charge de machines virtuelles en utilisant le portail Azure](quickstart-load-balancer-standard-public-portal.md).
        - Ajoutez **-R1** et **-R2** au nom des équilibreurs de charge, des machines virtuelles et des autres ressources dans chaque région. 

## <a name="sign-in-to-azure-portal"></a>Se connecter au portail Azure

[Connectez-vous](https://preview.portal.azure.com) au portail Azure en préversion.

## <a name="create-cross-region-load-balancer"></a>Créer un équilibreur de charge interrégional

Dans cette section, vous allez créer un équilibreur de charge interrégional et une adresse IP publique.

1. Dans la partie supérieure gauche de l’écran, sélectionnez **Créer une ressource > Réseau > Équilibreur de charge** ou tapez **équilibreur de charge** dans la zone de recherche.

2. Dans l’onglet **Fonctions de base** de la page **Créer un équilibreur de charge**, entrez ou sélectionnez les informations suivantes : 

    | Paramètre                 | Valeur                                              |
    | ---                     | ---                                                |
    | Abonnement               | Sélectionnez votre abonnement.    |    
    | Resource group         | Sélectionnez **Créer**, puis entrez **CreateCRLBTutorial-rg** dans la zone de texte.|
    | Nom                   | Entrez **myLoadBalancer-CR**.                                   |
    | Région         | Sélectionnez **USA Ouest**.                                        |
    | Type          | Sélectionnez **Public**.                                        |
    | SKU           | sélectionnez **Standard**. |
    | Niveau           | Sélectionnez **Global**. |
    | Adresse IP publique | Sélectionnez **Créer nouveau**.|
    | Nom de l’adresse IP publique | Tapez **myPublicIP-CR** dans la zone de texte.|
    | Préférence de routage| Sélectionnez **Réseau Microsoft**. |

    > [!NOTE]
    > L’équilibreur de charge interrégional peut uniquement être déployé dans les régions d’accueil suivantes : **USA Est 2, USA Ouest, Europe Ouest, Asie Sud-Est, USA Centre, Europe Nord, Asie Est**. Pour plus d’informations, consultez **https://aka.ms/homeregionforglb**.


3. Acceptez les valeurs par défaut pour les paramètres restants, puis sélectionnez **Vérifier + créer**.

4. Sous l’onglet **Review + create (Vérifier + créer)** , sélectionnez **Créer**.   

    :::image type="content" source="./media/tutorial-cross-region-portal/create-cross-region.png" alt-text="Créer un équilibreur de charge interrégional" border="true":::

## <a name="create-backend-pool"></a>Créer le pool principal

Dans cette section, vous allez ajouter deux équilibreurs de charge standard régionaux au pool de back-ends de l’équilibreur de charge interrégional.

> [!IMPORTANT]
> Pour effectuer ces étapes, vérifiez que deux équilibreurs de charge régionaux avec des pools de back-ends ont été déployés dans votre abonnement.  Pour plus d’informations, consultez **[Démarrage rapide : Créer un équilibreur de charge public pour équilibrer la charge de machines virtuelles en utilisant le portail Azure](quickstart-load-balancer-standard-public-portal.md)** .

Créez le pool d’adresses back-end **myBackendPool-CR** pour inclure les équilibreurs de charge standard régionaux.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer-CR** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis **Ajouter**.

3. Dans la page **Ajouter un pool de back-ends**, tapez **myBackendPool-CR** pour le nom.

4. Sélectionnez **Ajouter**.

4. Sélectionnez **myBackendPool-CR**.

5. Sous **Équilibreurs de charge**, sélectionnez la zone de liste déroulante sous **Équilibreur de charge**.

5. Sélectionnez **myLoadBalancer-R1** ou le nom de votre équilibreur de charge dans la région 1.

6. Sélectionnez la zone de liste déroulante sous **Configuration d’adresse IP frontale**. Entrez **LoadBalancerFrontEnd**.

7. Répétez les étapes 4 à 6 pour ajouter **myLoadBalancer-R2**.

8. Sélectionnez **Ajouter**.

    :::image type="content" source="./media/tutorial-cross-region-portal/add-to-backendpool.png" alt-text="Ajouter des équilibreurs de charge régionaux au pool de back-ends" border="true":::

## <a name="create-a-health-probe"></a>Créer une sonde d’intégrité

Dans cette section, vous allez créer une sonde d’intégrité pour créer la règle d’équilibrage de charge :

* Nommée **myHealthProbe.** .
* Protocole **TCP**.
* Intervalle de **5** secondes.
* Seuil de défaillance sur le plan de l’intégrité défini à **deux** échecs.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer-CR** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Sondes d’intégrité**.

3. Utilisez ces valeurs pour configurer la sonde d’intégrité :

    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHealthProbe**. |
    | Protocol | Sélectionnez **TCP**. |
    | Port | Entrez **80**. |
    | Intervalle | Entrez **5**. |
    | Seuil de défaillance sur le plan de l’intégrité | Entrez **2**. |

4. Sélectionnez **OK**.

    > [!NOTE]
    > L’équilibreur de charge interrégional dispose d’une sonde d’intégrité intégrée. Cette sonde est un espace réservé pour permettre à la création de règle d’équilibrage de charge de fonctionner.  Pour plus d’informations, consultez **[Limitations de l’équilibreur de charge interrégional](cross-region-overview.md#limitations)** .

## <a name="create-a-load-balancer-rule"></a>Créer une règle d’équilibreur de charge

Dans cette section, vous allez créer une règle d’équilibreur de charge :

* A pour nom **myHTTPRule**.
* Se trouve dans le front-end nommé **LoadBalancerFrontEnd**.
* Écoute sur le **Port 80**.
* Dirige le trafic à charge équilibrée vers le back-end nommé **myBackendPool-CR** sur le **Port 80**.

    > [!NOTE]
    > Le port front-end doit correspondre au port back-end et au port front-end des équilibreurs de charge régionaux du pool de back-ends.

1. Sélectionnez **Tous les services** dans le menu de gauche, **Toutes les ressources**, puis **myLoadBalancer-CR** dans la liste des ressources.

2. Sous **Paramètres**, sélectionnez **Règles d’équilibrage de charge**, puis **Ajouter**.

3. Pour configurer la règle d’équilibrage de charge, utilisez les valeurs suivantes :
    
    | Paramètre | Valeur |
    | ------- | ----- |
    | Nom | Entrez **MyHTTPRule**. |
    | Version de l’adresse IP | Sélectionnez **IPv4** |
    | Adresse IP du serveur frontal | Sélectionnez **LoadBalancerFrontEnd** |
    | Protocol | Sélectionnez **TCP**. |
    | Port | Entrez **80**.|
    | Port principal | Entrez **80**. |
    | Pool principal | Sélectionnez **MyBackendPool**.|
    | Sonde d’intégrité | Sélectionnez **myHealthProbe**. |
    | Délai d’inactivité (minutes) | Déplacez le curseur jusqu’à **15**. |
    | Réinitialisation du protocole TCP | Sélectionnez **Enabled**. |

4. Laissez les autres valeurs par défaut, puis sélectionnez **OK**.

    :::image type="content" source="./media/tutorial-cross-region-portal/create-lb-rule.png" alt-text="Créer une règle d’équilibreur de charge" border="true":::

## <a name="test-the-load-balancer"></a>Tester l’équilibreur de charge

Dans cette section, vous allez tester l’équilibreur de charge interrégional. Vous allez vous connecter à l’adresse IP publique dans un navigateur web.  Vous allez arrêter les machines virtuelles dans l’un des pools de back-ends de l’équilibreur de charge régional et observer le basculement.

1. Recherchez l’adresse IP publique de l’équilibreur de charge sur l’écran **Vue d’ensemble**. Sélectionnez **Tous les services** dans le menu de gauche, sélectionnez **Toutes les ressources**, puis **myPublicIP-CR**.

2. Copiez l’adresse IP publique, puis collez-la dans la barre d’adresses de votre navigateur. La page par défaut du serveur Web IIS s’affiche sur le navigateur.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-1.png" alt-text="Tester l’équilibreur de charge" border="true":::

3. Arrêtez les machines virtuelles dans le pool de back-ends de l’un des équilibreurs de charge régionaux.

4. Actualisez le navigateur web et observez le basculement de la connexion vers l’autre équilibreur de charge régional.

    :::image type="content" source="./media/tutorial-cross-region-portal/test-cr-lb-2.png" alt-text="Tester l’équilibreur de charge après le basculement" border="true":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, supprimez le groupe de ressources, l’équilibreur de charge et toutes les ressources associées. 

Pour ce faire, sélectionnez le groupe de ressources **CreateCRLBTutorial-rg** qui contient les ressources, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous allez :

* Vous avez créé un équilibreur de charge interrégional.
* Vous avez ajouté des équilibreurs de charge régionaux au pool de back-ends de l’équilibreur de charge interrégional.
* Vous avez créé une règle d’équilibrage de charge.
* Vous avez testé l’équilibreur de charge.

Pour plus d’informations sur l’équilibreur de charge interrégional, consultez [Équilibreur de charge interrégional (préversion)](cross-region-overview.md).


Passez à l’article suivant pour savoir comment :
> [!div class="nextstepaction"]
> [Équilibrer la charge de machines virtuelles entre des zones de disponibilité](tutorial-load-balancer-standard-public-zone-redundant-portal.md)

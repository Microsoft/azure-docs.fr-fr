---
title: Déployer Traffic Manager pour équilibrer les charges de travail de la solution VMware Azure (AVS)
description: Découvrez comment intégrer Traffic Manager avec la solution VMware Azure (AVS) pour équilibrer les charges de travail d’application sur plusieurs points de terminaison dans différentes régions.
ms.topic: how-to
ms.date: 08/14/2020
ms.openlocfilehash: ed74bb0dfc533abadd50af32afc06c9cb4106193
ms.sourcegitcommit: 642988f1ac17cfd7a72ad38ce38ed7a5c2926b6c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94874280"
---
# <a name="deploy-traffic-manager-to-balance-azure-vmware-solution-avs-workloads"></a>Déployer Traffic Manager pour équilibrer les charges de travail de la solution VMware Azure (AVS)

Cet article vous guide tout au long de l’intégration de Traffic Manager avec la solution VMware Azure (AVS) pour équilibrer les charges de travail d’application sur plusieurs points de terminaison. Nous allons examiner un scénario dans lequel Traffic Manager distribue le trafic entre trois passerelles d’application couvrant plusieurs régions AVS : USA Ouest, Europe Ouest et région locale, USA Est. 

Azure Traffic Manager est un équilibreur de charge du trafic DNS qui vous permet de distribuer le trafic de manière optimale aux services dans les régions Azure globales. Il équilibre la charge du trafic d’application sur les charges de travail Azure en cours d’exécution et les points de terminaison publics externes. Pour plus d’informations sur Traffic Manager, consultez [Qu’est-ce que Traffic Manager ?](../traffic-manager/traffic-manager-overview.md).

Commencez par examiner les [Prérequis](#prerequisites). Ensuite, nous allons parcourir les procédures pour :

> [!div class="checklist"]
> * vérifier la configuration de vos passerelles d’application ;
> * vérifier la configuration du segment NSX-T ;
> * créer votre profil Traffic Manager ;
> * ajouter des points de terminaison externes à votre profil Traffic Manager.

## <a name="topology"></a>Topologie

Comme le montre l’illustration suivante, Azure Traffic Manager fournit un équilibrage de charge pour les applications au niveau du DNS entre les points de terminaison régionaux. Les passerelles d’application ont des membres du pool principal configurés en tant que serveurs IIS, et sont référencées en tant que points de terminaison externes AVS.

La connexion sur le réseau virtuel entre les deux régions de cloud privé AVS, USA Ouest et Europe Ouest, et un serveur local dans la région USA Est, utilise une passerelle ExpressRoute.   

:::image type="content" source="media/traffic-manager/traffic-manager-topology.png" alt-text="Diagramme de l'architecture de l'intégration de Traffic Manager à Azure VMware Solution" lightbox="media/traffic-manager/traffic-manager-topology.png" border="false":::
 
## <a name="prerequisites"></a>Prérequis

- Trois machines virtuelles configurées en tant que serveurs Microsoft IIS s’exécutant dans différentes régions AVS : USA Ouest, Europe Ouest et locale. 

- Une passerelle d’application avec des points de terminaison externes dans les régions USA Ouest, Europe Ouest et locale.

- Hôte avec connectivité Internet pour la vérification. 

## <a name="verify-configuration-of-your-application-gateways"></a>vérifier la configuration de vos passerelles d’application ;

[Azure Application Gateway](https://azure.microsoft.com/services/application-gateway/) est un équilibreur de charge de trafic web de couche 7 qui vous permet de gérer le trafic vers vos applications web. Pour plus d’informations sur Application Gateway, consultez [Qu’est-ce qu’Azure Application Gateway ?](../application-gateway/overview.md). 

Dans ce scénario, trois instances d’Application Gateway sont configurées en tant que points de terminaison AVS externes. Les passerelles d’application disposent de machines virtuelles AVS configurées en tant que membres du pool principal pour équilibrer la charge des demandes de couche 7 entrantes (pour savoir comment configurer Application Gateway avec des machines virtuelles AVS en tant que pools principaux, consultez [Utiliser Application Gateway pour protéger vos applications web sur Azure VMware Solution](protect-azure-vmware-solution-with-application-gateway.md)).  

Les étapes suivantes vérifient la configuration correcte de vos passerelles d’application.

1. Ouvrez la Portail Azure et sélectionnez **Passerelles d’application** pour afficher la liste de vos passerelles d’application actuelles. 

    Pour ce scénario, nous avons configuré trois passerelles d’application :
    - AVS-GW-WUS
    - AVS-GW-EUS (locale)
    - AVS-GW-UEO

    :::image type="content" source="media/traffic-manager/app-gateways-list-1.png" alt-text="Capture d’écran de la page de la passerelle applicative montrant la liste des passerelles applicatives configurées." lightbox="media/traffic-manager/app-gateways-list-1.png":::

2. Sélectionnez l’une des passerelles d’application précédemment déployées. Une fenêtre s’ouvre, qui affiche diverses informations sur la passerelle applicative. Sélectionnez **Pools principaux** pour vérifier la configuration de l’un des pools principaux.

   :::image type="content" source="media/traffic-manager/backend-pool-config.png" alt-text="Capture d’écran de la page de passerelle applicative montrant les détails de la passerelle applicative sélectionnée." lightbox="media/traffic-manager/backend-pool-config.png":::
 
3. Dans ce cas, nous voyons un membre du pool principal de machines virtuelles configuré en tant que serveur web avec l’adresse IP 172.29.1.10.
 
    :::image type="content" source="media/traffic-manager/backend-pool-ip-address.png" alt-text="Capture d’écran de la page Modifier le pool principal avec l’adresse IP cible mise en surbrillance.":::

    Vous pouvez également vérifier la configuration des autres passerelles d’application et des membres du pool principal. 

## <a name="verify-configuration-of-the-nsx-t-segment"></a>Vérifier la configuration du segment NSX-T

Les segments réseau créés dans NSX-T Manager sont utilisés en tant que réseaux pour les machines virtuelles dans vCenter. Pour plus d’informations, consultez le didacticiel [Créer un segment réseau NSX-T dans AVS (Azure VMware Solution)](tutorial-nsx-t-network-segment.md).

Dans notre scénario, un segment NSX-T est configuré dans l’environnement AVS auquel la machine virtuelle membre du pool principal est attachée.

1. Sélectionnez **Segments** pour afficher vos segments configurés. Dans ce cas, nous voyons que le segment Contoso-segment1 est connecté à la passerelle Contoso-T01, un routeur flexible de niveau 1.

    :::image type="content" source="media/traffic-manager/nsx-t-segment-avs.png" alt-text="Capture d’écran montrant des profils de segment dans NSX-T Manager." lightbox="media/traffic-manager/nsx-t-segment-avs.png":::    

2. Sélectionnez **Passerelles de niveau 1** pour afficher la liste de vos passerelles de niveau 1 avec le nombre de segments liés. Sélectionnez le segment lié à Contoso-T01. Une fenêtre s’ouvre, qui affiche l’interface logique configurée sur le routeur de niveau 01. Celui-ci sert de passerelle vers la machine virtuelle du membre du pool principal connecté au segment.

   :::image type="content" source="media/traffic-manager/nsx-t-segment-linked-2.png" alt-text="Capture d’écran montrant l’adresse de la passerelle du segment sélectionné.":::    

3. Dans le client vSphere de machine virtuelle, sélectionnez la machine virtuelle pour afficher ses détails. Notez que son adresse IP correspond à ce que nous avons vu à l’étape 3 de la section précédente : 172.29.1.10.

    :::image type="content" source="media/traffic-manager/nsx-t-vm-details.png" alt-text="Capture d’écran montrant les détails de la machine virtuelle dans le client VSphere." lightbox="media/traffic-manager/nsx-t-vm-details.png":::    

4. Sélectionnez la machine virtuelle, puis cliquez sur **ACTIONS > Modifier les paramètres** pour vérifier la connexion au segment NSX-T.

## <a name="create-your-traffic-manager-profile"></a>Créer votez profil Traffic Manager

1. Connectez-vous au [portail Azure](https://rc.portal.azure.com/#home). Sous **Services Azure > Mise en réseau**, sélectionnez **Profils Traffic Manager**.

2. Sélectionnez **+ Ajouter** pour créer un profil Traffic Manager.
 
3. Fournissez un nom de profil, une méthode de routage (nous utilisons une pondération dans ce scénario ; consultez [Méthodes de routage de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md)), un abonnement et un groupe de ressources, puis sélectionnez **Créer**.

## <a name="add-external-endpoints-into-the-traffic-manager-profile"></a>Ajouter des points de terminaison externes au profil Traffic Manager

1. Sélectionnez le profil Traffic Manager dans le volet des résultats de la recherche, sélectionnez **Points de terminaison** puis **+ Ajouter**.

2. Entrez les informations requises : Type, nom, nom de domaine complet (FQDN) ou IP, et poids (dans ce scénario, nous affectons un poids de 1 à chaque point de terminaison). Sélectionnez **Ajouter**. Cela crée le point de terminaison externe. L’état d’analyse doit être **En ligne**. Répétez les mêmes étapes pour créer deux autres points de terminaison externes, l’un dans une autre région et l’autre local. Une fois créés, les trois point de terminaison s’affichent dans le profil Traffic Manager, et leur état doit être **En ligne**.

3. Sélectionnez **Vue d’ensemble**. Copiez l’URL sous **Nom du DNS**.

   :::image type="content" source="media/traffic-manager/traffic-manager-endpoints.png" alt-text="Capture d’écran montrant une vue d’ensemble de point de terminaison Traffic Manager avec le nom DNS mis en surbrillance." lightbox="media/traffic-manager/traffic-manager-endpoints.png"::: 

4. Collez l’URL du nom DNS dans un navigateur. La capture d’écran suivante montre le trafic dirigé vers la région Europe Ouest.

   :::image type="content" source="media/traffic-manager/traffic-to-west-europe.png" alt-text="Capture d’écran de la fenêtre du navigateur montrant le trafic acheminé vers la région Europe Ouest."::: 

5. Actualisez votre navigateur. La capture d’écran suivante présente le trafic désormais dirigé vers un autre ensemble de membres du pool principal dans la région USA Ouest.

   :::image type="content" source="media/traffic-manager/traffic-to-west-us.png" alt-text="Capture d’écran de la fenêtre du navigateur montrant le trafic acheminé vers la région USA Ouest."::: 

6. Réactualisez votre navigateur. La capture d’écran suivante présente le trafic désormais dirigé vers l’ensemble final de membres du pool principal local.

   :::image type="content" source="media/traffic-manager/traffic-to-on-premises.png" alt-text="Capture d’écran de la fenêtre du navigateur montrant le trafic acheminé vers une région locale.":::

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus :

- [Utilisation d’Azure Application Gateway sur une solution VMware Azure (AVS)](protect-azure-vmware-solution-with-application-gateway.md)
- [Méthodes de routage de Traffic Manager](../traffic-manager/traffic-manager-routing-methods.md)
- [Combinaison de services d’équilibrage de charge dans Azure](../traffic-manager/traffic-manager-load-balancing-azure.md)
- [Mesure des performances de Traffic Manager](../traffic-manager/traffic-manager-performance-considerations.md)

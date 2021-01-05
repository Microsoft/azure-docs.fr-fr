---
title: Utilisation des services d’équilibrage de charge dans Azure | Microsoft Docs
description: 'Ce didacticiel vous montre comment créer un scénario à l’aide du portefeuille d’équilibrage de charge Azure : Traffic Manager, Application Gateway et Load Balancer.'
services: traffic-manager
documentationcenter: ''
author: duongau
manager: kumudD
ms.service: traffic-manager
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 10/27/2016
ms.author: duau
ms.openlocfilehash: 431eaff9da95063648d3e80acb54be9cc5c25bc5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96021688"
---
# <a name="using-load-balancing-services-in-azure"></a>Utilisation des services d’équilibrage de charge dans Azure

## <a name="introduction"></a>Introduction

Microsoft Azure propose plusieurs services destinés à gérer la distribution et l’équilibrage de la charge du trafic réseau. Vous pouvez utiliser ces services individuellement ou combiner leurs méthodes selon vos besoins spécifiques, afin d’élaborer une solution optimale.

Dans ce didacticiel, nous allons tout d’abord définir le cas d’usage d’un client, puis voir comment nous pouvons le rendre plus robuste et plus performant en utilisant la gamme de services d’équilibrage de charge Azure : Traffic Manager, Application Gateway et Load Balancer. Nous donnerons ensuite des instructions détaillées permettant de créer un déploiement géographiquement redondant, qui distribue le trafic à des machines virtuelles et permet de gérer différents types de demandes.

À un niveau conceptuel, chacun de ces services joue un rôle distinct dans la hiérarchie de l’équilibrage de charge.

* **Traffic Manager** assure un équilibrage de charge DNS global. Il examine les demandes DNS entrantes et répond avec un point de terminaison intègre, conforme à la stratégie de routage choisie par le client. Les différentes méthodes de routage disponibles sont les suivantes :
  * Le routage basé sur les performances permet de rediriger le demandeur vers le point de terminaison le plus proche en termes de latence.
  * Le routage par priorité permet de rediriger l’ensemble du trafic vers un point de terminaison, les autres points de terminaison jouant le rôle de points de secours.
  * Le routage du trafic en tourniquet (round robin) pondéré permet de répartir le trafic en fonction de la pondération associée à chaque point de terminaison.
  * Le routage basé sur la géographie permet de répartir le trafic sur des nœuds de point de terminaison d’application en fonction de l’emplacement géographique de l’utilisateur.
  * Le routage basé sur le sous-réseau permet de répartir le trafic sur des nœuds de point de terminaison d’application en fonction du sous-réseau (plage d’adresses IP) de l’utilisateur.
  * Le routage à valeurs multiples vous permet d’envoyer les adresses IP de plusieurs points de terminaison d’application dans une seule réponse DNS.

  Le client se connecte directement au point de terminaison retourné par Traffic Manager. Azure Traffic Manager est capable de détecter un point de terminaison défectueux. Si tel est le cas, il redirige les clients vers une autre instance intègre. Pour plus d’informations sur ce service, consultez la [documentation Azure Traffic Manager](traffic-manager-overview.md).
* **Application Gateway** intègre ADC (Application Delivery Controller) sous la forme d’un service, qui offre diverses fonctionnalités d’équilibrage de charge de couche 7 pour votre application. Elle permet aux clients d’optimiser la productivité de la batterie de serveurs web en déchargeant une terminaison TLS gourmande en ressources de l’UC vers la passerelle d’application. Parmi les autres fonctionnalités de routage de couche 7, citons la distribution en tourniquet (round robin) du trafic entrant, l’affinité de session basée sur les cookies, le routage basé sur le chemin des URL et la possibilité d’héberger plusieurs sites web derrière une seule passerelle Application Gateway. Cette dernière peut être configurée en tant que passerelle accessible sur Internet, passerelle interne uniquement ou une combinaison des deux. La passerelle Application Gateway est une solution Azure entièrement gérée, très évolutive et hautement disponible. Elle fournit un ensemble complet de fonctionnalités de diagnostics et de journalisation, pour une meilleure gérabilité.
* **Load Balancer** fait partie intégrante de la pile Azure SDN, et fournit des services d’équilibrage de charge de couche 4 hautement performants et à faible latence pour tous les protocoles UDP et TCP. Il gère les connexions entrantes et sortantes. Vous pouvez configurer des points de terminaison publics et internes avec équilibrage de charge, et définir des règles de mappage des connexions entrantes aux destinations du pool principal, en utilisant des options d’analyse d’intégrité TCP et HTTP de façon à gérer la disponibilité du service.

## <a name="scenario"></a>Scénario

Dans ce scénario, nous utilisons un site web simple qui affiche deux types de contenu : des images et des pages web affichées dynamiquement. Ce site web doit être géographiquement redondant et doit desservir ses utilisateurs à partir de l’emplacement le plus proche (avec la latence la plus faible). Le développeur de l’application a décidé que les URL correspondant au modèle /images /* seraient desservies par un pool dédié de machines virtuelles différentes du reste de la batterie de serveurs web.

Par ailleurs, le pool de machines virtuelles par défaut affichant le contenu dynamique doit communiquer avec une base de données principale hébergée sur un cluster à haute disponibilité. Le déploiement est entièrement approvisionné via Azure Resource Manager.

L’utilisation de Traffic Manager, d’Application Gateway et de Load Balancer permet à ce site web d’atteindre les objectifs de conception suivants :

* **Géoredondance multiple** : si une région connaît une défaillance, Traffic Manager achemine le trafic en toute transparence vers la région la plus proche sans aucune intervention du propriétaire de l’application.
* **Latence réduite** : comme Azure Traffic Manager dirige automatiquement le client vers la région la plus proche, celui-ci subit une latence plus faible quand il demande le contenu de la page web.
* **Extensibilité indépendante** : comme la charge de travail de l’application web est répartie en fonction du type de contenu, le propriétaire de l’application peut mettre à l’échelle les charges de travail des demandes indépendamment les unes des autres. Application Gateway vérifie que le trafic est acheminé vers les pools appropriés, en fonction des règles spécifiées et de l’intégrité de l’application.
* **Équilibrage de charge interne** : comme Load Balancer se trouve devant le cluster à haute disponibilité, seul le point de terminaison actif et intègre d’une base de données est exposé à l’application. Par ailleurs, un administrateur de base de données peut optimiser la charge de travail en répartissant les réplicas actifs et passifs dans le cluster, indépendamment de l’application frontale. Load Balancer fournit les connexions au cluster à haute disponibilité et fait en sorte que seules les bases de données intègres reçoivent des demandes de connexion.

Le diagramme suivant illustre l’architecture de ce scénario :

![Diagramme de l’architecture d’équilibrage de charge](./media/traffic-manager-load-balancing-azure/scenario-diagram.png)

> [!NOTE]
> Cet exemple est l’une des innombrables configurations possibles offertes par les services d’équilibrage de charge Azure. Les services Azure Traffic Manager, Application Gateway et Load Balancer peuvent être combinés et associés pour répondre au mieux à vos besoins d’équilibrage de charge. Par exemple, s’il n’est pas nécessaire de faire appel au déchargement TLS ou au traitement de la couche 7, Load Balancer peut être utilisé à la place d’Application Gateway.

## <a name="setting-up-the-load-balancing-stack"></a>Configuration de la pile d’équilibrage de charge

### <a name="step-1-create-a-traffic-manager-profile"></a>Étape 1 : Créer un profil Traffic Manager

1. Dans le portail Azure, cliquez sur **Créer une ressource** > **Mise en réseau** > **Profil Traffic Manager** > **Créer**.
2. Entrez les informations de base suivantes :

   * **Name** : donnez un nom de préfixe DNS à votre profil Traffic Manager.
   * **Méthode de routage** : sélectionnez la stratégie de la méthode de routage du trafic. Pour en savoir plus sur les différentes méthodes , voir [À propos des méthodes de routage du trafic de Traffic Manager](traffic-manager-routing-methods.md).
   * **Abonnement**: sélectionnez l’abonnement qui contient le profil.
   * **Groupe de ressources** : sélectionnez le groupe de ressources qui contient le profil. Il peut s’agir d’un groupe de ressources nouveau ou existant.
   * **Emplacement du groupe de ressources** : le service Traffic Manager est global, et non lié à un emplacement. Toutefois, vous devez spécifier une région pour le groupe hébergeant les métadonnées associées au profil Traffic Manager. Cet emplacement n’a aucune incidence sur la disponibilité du profil au moment de l’exécution.

3. Cliquez sur **Créer** pour générer le profil Traffic Manager.

   ![Panneau Créer un profil Traffic Manager](./media/traffic-manager-load-balancing-azure/s1-create-tm-blade.png)

### <a name="step-2-create-the-application-gateways"></a>Étape 2 : créer des passerelles Application Gateway

1. Dans le volet de gauche du portail Azure, cliquez sur **Créer une ressource** > **Mise en réseau** > **Application Gateway**.
2. Indiquez ensuite les informations de base suivantes sur la passerelle Application Gateway :

   * **Name** : Nom de la passerelle Application Gateway.
   * **Taille de la référence** : taille de la passerelle Application Gateway (petite, moyenne ou grande).
   * **Nombre d’instances** : nombre d’instances (valeur comprise entre 2 et 10).
   * **Groupe de ressources** : groupe de ressources qui contient la passerelle Application Gateway. Il peut s’agir d’un groupe existant, ou d’un nouveau groupe.
   * **Emplacement** : région de la passerelle Application Gateway. Il s’agit du même emplacement que celui du groupe de ressources. Cet emplacement est important, dans la mesure où le réseau virtuel et l’adresse IP publique doivent se trouver au même endroit que la passerelle.
3. Cliquez sur **OK**.
4. Définissez le réseau virtuel, le sous-réseau, l’adresse IP frontale et les configurations d’écouteur pour la passerelle Application Gateway. Dans ce scénario, l’adresse IP frontale est de type **Public**. Elle peut ainsi être ajoutée comme point de terminaison au profil Traffic Manager.
5. Configurez l’écouteur avec l’une des options suivantes :
    * Si vous utilisez le protocole HTTP, vous n’avez pas besoin de configurer quoi que ce soit. Cliquez sur **OK**.
    * Une configuration supplémentaire est requise pour l’utilisation du protocole HTTPS. Consultez la section [Créer une passerelle Application Gateway à l’aide du portail](../application-gateway/application-gateway-create-gateway-portal.md), en commençant à l’étape 9. Lorsque vous avez effectué la configuration, cliquez sur **OK**.

#### <a name="configure-url-routing-for-application-gateways"></a>Configurer le routage d’URL pour les passerelles Application Gateway

Au moment de choisir le pool principal, une passerelle Application Gateway configurée avec une règle basée sur le chemin adopte un modèle de chemin de l’URL de la demande en plus du mode de distribution en tourniquet (round robin). Dans ce scénario, nous ajoutons une règle basée sur le chemin pour diriger les URL contenant « /images/\* » vers le pool de serveurs d’images. Pour en savoir plus sur la configuration du routage basé sur le chemin des URL pour une passerelle Application Gateway, voir [Créer une règle basée sur le chemin pour une passerelle Application Gateway](../application-gateway/application-gateway-create-url-route-portal.md).

![Diagramme de couche web Application Gateway](./media/traffic-manager-load-balancing-azure/web-tier-diagram.png)

1. À partir de votre groupe de ressources, accédez à l’instance de la passerelle Application Gateway créée aux étapes de la section précédente.
2. Sous **Paramètres**, sélectionnez **Pools principaux**, puis cliquez sur **Ajouter** pour ajouter les machines virtuelles à associer avec les pools principaux de niveau web.
3. Entrez le nom du pool principal et toutes les adresses IP des machines résidant dans le pool. Dans ce scénario, nous connectons deux pools de serveurs principaux de machines virtuelles.

   ![Ajouter un pool principal, dans Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-add-bepool.png)

4. Dans la section **Paramètres** de la passerelle Application Gateway, sélectionnez **Règles**, puis cliquez sur le bouton **Basé sur le chemin** pour ajouter une règle.

   ![Bouton Basé sur le chemin de la zone Règles de la passerelle Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-add-pathrule.png)

5. Configurez la règle en fournissant les informations décrites ci-après.

   Paramètres de base :

   + **Name** : nom convivial de la règle accessible via le portail.
   + **Écouteur** : écouteur utilisé pour la règle.
   + **Pool principal par défaut** : pool principal à utiliser avec la règle par défaut.
   + **Paramètres HTTP par défaut** : paramètres HTTP à utiliser avec la règle par défaut.

   Règles basées sur le chemin :

   + **Name** : nom convivial de la règle basée sur le chemin.
   + **Chemins** : règle de chemin utilisée pour transférer le trafic.
   + **Pool principal** : pool principal à utiliser avec cette règle.
   + **Paramètre HTTP** : paramètres HTTP à utiliser avec cette règle.

   > [!IMPORTANT]
   > Chemins : pour être valides, les chemins doivent commencer par « / ». Le caractère générique « \* » n’est autorisé qu’à la fin. Exemples valides : /xyz, /xyz\* ou /xyz/\*.

   ![Panneau Ajouter une règle basée sur le chemin d’Application Gateway](./media/traffic-manager-load-balancing-azure/s2-appgw-pathrule-blade.png)

### <a name="step-3-add-application-gateways-to-the-traffic-manager-endpoints"></a>Étape 3 : ajouter des passerelles Application Gateway aux points de terminaison Traffic Manager

Dans ce scénario, Traffic Manager est connecté à des instances Application Gateway (selon la configuration des étapes précédentes) qui résident dans des régions différentes. Maintenant que les passerelles Application Gateway sont configurées, l’étape suivante consiste à les connecter à votre profil Traffic Manager.

1. Ouvrez le profil Traffic Manager. Pour ce faire, consultez le groupe de ressources ou recherchez le nom du profil Traffic Manager dans la zone **Toutes les ressources**.
2. Dans le volet de gauche, sélectionnez **Points de terminaison**, puis cliquez sur **Ajouter** pour ajouter un point de terminaison.

   ![Bouton Ajouter de la zone Points de terminaison Traffic Manager](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint.png)

3. Créez un point de terminaison en entrant les informations suivantes :

   * **Type** : sélectionnez le type de point de terminaison pour l’équilibrage de charge. Dans ce scénario, sélectionnez **Point de terminaison Azure**, car nous le connectons aux instances Application Gateway configurées précédemment.
   * **Name** : saisissez le nom du point de terminaison.
   * **Type de ressource cible** : sélectionnez **Adresse IP publique** et, sous **Ressource cible**, sélectionnez l’adresse IP publique de la passerelle Application Gateway configurée précédemment.

   ![Ajouter un point de terminaison, dans Traffic Manager](./media/traffic-manager-load-balancing-azure/s3-tm-add-endpoint-blade.png)

4. Vous pouvez désormais tester votre configuration en y accédant avec le DNS de votre profil Traffic Manager (dans cet exemple : TrafficManagerScenario.trafficmanager.net). Vous pouvez renvoyer des demandes, mettre en service ou hors service les machines virtuelles et serveurs web créés dans différentes régions, puis modifier les paramètres du profil Traffic Manager pour tester votre configuration.

### <a name="step-4-create-a-load-balancer"></a>Étape 4 : Créer un équilibrage de charge

Dans ce scénario, Load Balancer distribue les connexions de niveau web aux bases de données, au sein d’un cluster à haute disponibilité.

Si votre cluster de bases de données à haute disponibilité utilise SQL Server AlwaysOn, voir [Configurer un ou plusieurs écouteurs de groupe de disponibilité AlwaysOn](../azure-sql/virtual-machines/windows/availability-group-listener-powershell-configure.md) pour obtenir des instructions pas à pas.

Pour en savoir plus sur la configuration d’un équilibrage de charge interne, voir [Créer un équilibreur de charge interne dans le portail Azure](../load-balancer/load-balancer-get-started-ilb-arm-portal.md).

1. Dans le volet de gauche du portail Azure, cliquez sur **Créer une ressource** > **Mise en réseau** > **Équilibreur de charge**.
2. Choisissez un nom pour votre équilibreur de charge.
3. Définissez le **type** sur **Interne** et choisissez le réseau et le sous-réseau virtuels appropriés dans lesquels l’équilibrage de charge doit résider.
4. Sous **Attribution d’adresses IP**, sélectionnez **Dynamique** ou **Statique**.
5. Sous **Groupe de ressources**, choisissez le groupe de ressources de l’équilibrage de charge.
6. Sous **Emplacement**, choisissez la région appropriée pour l’équilibrage de charge.
7. Cliquez sur **Créer** pour générer l’équilibrage de charge.

#### <a name="connect-a-back-end-database-tier-to-the-load-balancer"></a>Connecter un niveau de base de données principal à l’équilibrage de charge

1. À partir de votre groupe de ressources, recherchez l’équilibrage de charge créé lors des étapes précédentes.
2. Sous **Paramètres**, cliquez sur **Pools principaux**, puis cliquez sur **Ajouter** pour ajouter un pool principal.

   ![Ajouter un pool principal, dans Équilibreur de charge](./media/traffic-manager-load-balancing-azure/s4-ilb-add-bepool.png)

3. Entrez le nom du pool principal.
4. Ajoutez des ordinateurs individuels ou un jeu de disponibilité pour le pool principal.

#### <a name="configure-a-probe"></a>Configurer une sonde

1. Dans votre équilibrage de charge, sélectionnez **Paramètres**, puis **Sondes**, et cliquez sur **Ajouter** pour ajouter une sonde.

   ![Ajouter une sonde, dans Équilibreur de charge](./media/traffic-manager-load-balancing-azure/s4-ilb-add-probe.png)

2. Entrez le nom de la sonde.
3. Sélectionnez le **protocole** de la sonde. Pour une base de données, vous choisirez sans doute une sonde TCP plutôt qu’une sonde HTTP. Pour en savoir plus sur les sondes d’équilibrage de charge, voir [Sondes d’équilibreur de charge](../load-balancer/load-balancer-custom-probe-overview.md).
4. Saisissez la valeur **Port** de votre base de données à utiliser au moment d’accéder à la sonde.
5. Sous **Intervalle**, spécifiez la fréquence de sondage de l’application.
6. Sous **Seuil de défaillance sur le plan de l’intégrité**, spécifiez le nombre d’échecs de sonde successifs qui doivent se produire avant que la machine virtuelle principale soit considérée comme défectueuse.
7. Cliquez sur **OK** pour créer la sonde.

#### <a name="configure-the-load-balancing-rules"></a>Configurer des règles d’équilibrage de charge

1. Dans la section **Paramètres** de l’équilibrage de charge, sélectionnez **Règles d’équilibrage de charge**, puis cliquez sur **Ajouter** pour créer une règle.
2. Entrez le **Nom** pour la règle d’équilibrage de charge.
3. Choisissez une valeur pour l’**adresse IP frontale** de l’équilibrage de charge, ainsi que les options **Protocole** et **Port**.
4. Sous **Port principal**, spécifiez le port à utiliser dans le pool principal.
5. Sélectionnez le **pool principal** et la **sonde** créés lors des étapes précédentes pour leur appliquer la règle.
6. Sous **Persistance de session**, choisissez le mode de persistance des sessions.
7. Sous **Délais d’inactivité**, spécifiez le délai d’inactivité, en minutes.
8. Sous **IP flottante**, sélectionnez **Désactivé** ou **Activé**.
9. Cliquez sur **OK** pour créer la règle.

### <a name="step-5-connect-web-tier-vms-to-the-load-balancer"></a>Étape 5 : connecter des machines virtuelles de niveau web à l’équilibreur de charge

Nous allons maintenant configurer l’adresse IP et le port frontal de l’équilibrage de charge dans les applications s’exécutant sur vos machines virtuelles de niveau web pour les connexions de base de données éventuelles. Cette configuration est propre aux applications qui s’exécutent sur ces machines virtuelles. Pour configurer l’adresse IP et le port de destination, consultez la documentation de l’application. Pour trouver l’adresse IP du serveur frontal, dans le portail Azure, accédez au pool d’adresses IP frontales dans le volet **Paramètres d’équilibrage de charge**.

![Volet de navigation Pool d’IP du serveur principal de Load Balancer](./media/traffic-manager-load-balancing-azure/s5-ilb-frontend-ippool.png)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d’ensemble de Traffic Manager](traffic-manager-overview.md)
* [Vue d’ensemble d’Application Gateway](../application-gateway/application-gateway-introduction.md)
* [Vue d’ensemble d’Azure Load Balancer](../load-balancer/load-balancer-overview.md)

---
title: 'Azure ExpressRoute : Configurer NPM pour les circuits'
description: Configurez la surveillance réseau (NPM) basée sur le cloud pour les circuits Azure ExpressRoute. Ce document traite de la surveillance via le peering privé ExpressRoute et le peering Microsoft.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 01/25/2019
ms.author: duau
ms.openlocfilehash: c8127a60a4685a615bc07e21a1efb4dd216c5b8c
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92201050"
---
# <a name="configure-network-performance-monitor-for-expressroute"></a>Configurer Network Performance Monitor pour ExpressRoute

Cet article vous permet de configurer une extension Network Performance Monitor pour superviser ExpressRoute. Network Performance Monitor (NPM) est une solution de supervision réseau basée sur le cloud, qui surveille la connectivité entre les déploiements cloud Azure et les déploiements locaux (succursales, etc.). NPM fait partie des journaux d’Azure Monitor. NPM offre une extension pour ExpressRoute, qui vous permet de surveiller les performances réseau sur des circuits ExpressRoute configurés pour utiliser le peering privé ou le peering Microsoft. Lorsque vous configurez NPM pour ExpressRoute, vous pouvez détecter les problèmes de réseau pour les identifier et les éliminer. Ce service est également disponible pour le cloud Azure Government.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

Vous pouvez :

* Surveiller les pertes et la latence entre différents réseaux virtuels et définir des alertes

* Surveiller tous les chemins d’accès (y compris les chemins d’accès redondants) sur le réseau

* Résoudre les problèmes réseau temporaires et dans le temps qui sont difficiles à répliquer

* Aider à déterminer un segment spécifique sur le réseau qui est responsable de la dégradation des performances

* Obtenir le débit par réseau virtuel (si vous avez des agents installés dans chaque réseau virtuel)

* Vérifier l’état système ExpressRoute à partir d’un point antérieur dans le temps

## <a name="workflow"></a><a name="workflow"></a>Flux de travail

Des agents de surveillance sont installés sur plusieurs serveurs, en local et sur Azure. Les agents communiquent entre eux mais n’envoient pas de données. Ils envoient des paquets de négociation TCP. La communication entre les agents permet à Azure de mapper la topologie réseau et le chemin d’accès que le trafic peut prendre.

1. Créez un espace de travail NPM. Il est identique à un espace de travail Log Analytics.
2. Installez et configurez les agents logiciels (si vous souhaitez uniquement superviser via Microsoft Peering, cette étape est inutile) : 
    * Installez des agents de surveillance sur les serveurs locaux et les machines virtuelles Azure (pour le peering privé).
    * Configurez les paramètres sur les serveurs de l’agent de surveillance pour autoriser les agents de surveillance à communiquer. (Ouvrez les ports du pare-feu, etc.)
3. Configurez des règles de groupe de sécurité réseau pour permettre à l’agent de surveillance installé sur des machines virtuelles Azure de communiquer avec des agents de surveillance locaux.
4. Configurez la supervision : Lancez une détection automatique et gérez les réseaux visibles dans NPM.

Si vous utilisez déjà Network Performance Monitor pour surveiller d’autres objets ou services, et si vous avez déjà un espace de travail dans une des régions prises en charge, vous pouvez passer les étapes 1 et 2 et commencer votre configuration à l’étape 3.

## <a name="step-1-create-a-workspace"></a><a name="configure"></a>Étape 1 : Créer un espace de travail

Créez un espace de travail dans l’abonnement où les réseaux virtuels sont liés aux circuits ExpressRoute.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez l’abonnement avec les réseaux virtuels associés à votre circuit ExpressRoute. Ensuite, recherchez « Network Performance Monitor » dans la liste des services de la **Place de marché**. Dans les résultats, cliquez pour ouvrir la page **Network Performance Monitor**.

   >[!NOTE]
   >Vous pouvez créer un espace de travail ou utiliser un espace de travail existant. Si vous souhaitez utiliser un espace de travail existant, vous devez vérifier qu’il a été migré vers le nouveau langage de requête. [Plus d’informations...](../azure-monitor/log-query/log-query-overview.md)
   >

   ![portail](./media/how-to-npm/3.png)<br><br>
2. En bas de la page **Network Performance Monitor**, cliquez sur **Créer** pour ouvrir la page **Network Performance Monitor - Créer une solution**. Cliquez sur **Espace de travail Log Analytics - Sélectionner un espace de travail** pour ouvrir la page Espaces de travail. Cliquez sur **+ Créer un espace de travail** pour ouvrir la page Espaces de travail.
3. Dans la page **Espace de travail Log Analytics**, sélectionnez **Créer**, puis configurez les paramètres suivants :

   * Espace de travail Log Analytics : entrez un nom pour votre espace de travail.
   * Abonnement : si vous possédez plusieurs abonnements, choisissez celui que vous souhaitez associer au nouvel espace de travail.
   * Groupe de ressources : créez un groupe de ressources ou utilisez un groupe existant.
   * Emplacement : cet emplacement est utilisé pour spécifier l’emplacement du compte de stockage qui est utilisé pour les journaux d’activité de connexion de l’agent.
   * Niveau tarifaire : sélectionnez le niveau tarifaire.
  
     >[!NOTE]
     >Le circuit ExpressRoute peut être n’importe où dans le monde. Il ne doit pas nécessairement se trouver dans la même région que l’espace de travail.
     >
  
     ![espace de travail](./media/how-to-npm/4.png)<br><br>
4. Cliquez sur **OK** pour enregistrer et déployer le modèle de paramètres. Une fois le modèle validé, cliquez sur **Créer** pour déployer l’espace de travail.
5. Une fois l’espace de travail déployé, accédez à la ressource **NetworkMonitoring(name)** que vous avez créée. Validez les paramètres, puis cliquez sur **La solution nécessite une configuration supplémentaire**.

   ![configuration supplémentaire](./media/how-to-npm/5.png)

## <a name="step-2-install-and-configure-agents"></a><a name="agents"></a>Étape 2 : Installer et configurer des agents

### <a name="21-download-the-agent-setup-file"></a><a name="download"></a>2.1 : Télécharger le fichier de configuration de l’agent

1. Accédez à l’onglet **Paramètres communs** de la page **Configuration de Network Performance Monitor** de votre ressource. Cliquez sur l’agent qui correspond au processeur de votre serveur dans la section **Installer les agents Log Analytics**, puis téléchargez le fichier d’installation.
2. Ensuite, copiez et collez **l’ID d’espace de travail** et la **clé primaire** dans le bloc-notes.
3. Dans la section **Configurer les agents Log Analytics pour le monitoring avec le protocole TCP**, téléchargez le script PowerShell. Le script PowerShell vous permet d’ouvrir le port de pare-feu approprié pour les transactions TCP.

   ![Script PowerShell](./media/how-to-npm/7.png)

### <a name="22-install-a-monitoring-agent-on-each-monitoring-server-on-each-vnet-that-you-want-to-monitor"></a><a name="installagent"></a>2.2 : Installer un agent de supervision sur chaque serveur de supervision (sur chaque réseau virtuel que vous voulez superviser)

Nous vous recommandons d’installer au moins deux agents de chaque côté de la connexion ExpressRoute à des fins de redondance (par exemple, localement, réseaux virtuels Azure). L’agent doit être installé sur Windows Server 2008 SP1 ou version ultérieure. La surveillance des circuits ExpressRoute à l’aide du système d’exploitation de bureau Windows et du système d’exploitation Linux n’est pas prise en charge. Pour installer les agents, procédez comme suit :
   
  >[!NOTE]
  >Les agents envoyés (push) par SCOM (notamment [MMA](/previous-versions/system-center/system-center-2012-R2/dn465154(v=sc.12))) risquent de ne pas parvenir à détecter de manière cohérente leur emplacement, s’ils sont hébergés dans Azure. Nous vous recommandons de ne pas utiliser ces agents dans des réseaux virtuels Azure pour surveiller ExpressRoute.
  >

1. Exécutez **Installation** pour installer l’agent sur chaque serveur que vous souhaitez utiliser pour la surveillance ExpressRoute. Le serveur que vous utilisez pour la surveillance peut être une machine virtuelle ou locale et doit avoir accès à Internet. Vous devez installer au moins un agent localement, et un agent sur chaque segment de réseau que vous souhaitez surveiller dans Azure.
2. Sur la page d’**accueil**, cliquez sur **Suivant**.
3. Sur la page **Termes du contrat de licence**, lisez les conditions de licence, puis cliquez sur **J’accepte**.
4. Sur la page **Dossier de destination**, modifiez ou conservez le dossier d’installation par défaut, puis cliquez sur **Suivant**.
5. Dans la page **Options d’installation de l’agent**, vous pouvez choisir de connecter l’agent aux journaux Azure Monitor ou à Operations Manager. Ou bien, vous pouvez ne rien choisir si vous souhaitez configurer l’agent ultérieurement. Une fois vos sélections effectuées, cliquez sur **Suivant**.

   * Si vous choisissez de vous connecter à **Azure Log Analytics**, collez **l’ID de l’espace de travail** et la **Clé de l’espace de travail** (clé primaire) que vous avez copiés dans le bloc-notes de la section précédente. Cliquez ensuite sur **Suivant**.

     ![ID et clé](./media/how-to-npm/8.png)
   * Si vous choisissez de vous connecter à **Operations Manager**, sur la page **Configuration du groupe d’administration**, saisissez le **nom du groupe d’administration**, le **serveur d’administration** et le **port du serveur d’administration**. Cliquez ensuite sur **Suivant**.

     ![Operations Manager](./media/how-to-npm/9.png)
   * Sur la page **Compte d’action d’agent**, choisissez le compte **système local** ou un **compte d’ordinateur de domaine ou local**. Cliquez ensuite sur **Suivant**.

     ![Compte](./media/how-to-npm/10.png)
6. Dans la page **Prêt pour l’installation**, passez en revue vos choix, puis cliquez sur **Installer**.
7. Dans la page **Configuration effectuée**, cliquez sur **Terminer**.
8. Lorsque vous avez terminé, Microsoft Monitoring Agent apparaît dans le Panneau de configuration. Vous pouvez y contrôler votre configuration et vérifier que l’agent est bien connecté aux journaux Azure Monitor. Quand il est connecté, l’agent affiche un message indiquant : **Microsoft Monitoring Agent s’est connecté au service Microsoft Operations Management Suite**.

9. Répétez cette procédure pour chaque réseau virtuel dont vous voulez effectuer la surveillance.

### <a name="23-configure-proxy-settings-optional"></a><a name="proxy"></a>2.3 : Configurer des paramètres de proxy (facultatif)

Si vous utilisez un proxy web pour accéder à Internet, procédez comme suit pour configurer les paramètres de proxy pour Microsoft Monitoring Agent. Effectuez ces étapes pour chaque serveur. Si vous devez configurer plusieurs serveurs, utilisez un script pour automatiser ce processus. Si c’est le cas, consultez [Pour configurer les paramètres de proxy de Microsoft Monitoring Agent à l’aide d’un script](../azure-monitor/platform/agent-windows.md).

Pour configurer les paramètres de proxy de Microsoft Monitoring Agent dans le Panneau de configuration :

1. Ouvrir le **Panneau de configuration**.
2. Ouvrez **Microsoft Monitoring Agent**.
3. Cliquez sur l'onglet **Paramètres de proxy** .
4. Sélectionnez **Utiliser un serveur proxy** et tapez l’URL et le numéro de port, si nécessaire. Si votre serveur proxy requiert une authentification, tapez le nom d'utilisateur et le mot de passe pour accéder au serveur proxy.

   ![proxy](./media/how-to-npm/11.png)

### <a name="24-verify-agent-connectivity"></a><a name="verifyagent"></a>2.4 : Vérifier la connectivité de l’agent

Vous pouvez facilement vérifier si vos agents communiquent.

1. Sur un serveur avec l’agent de surveillance, ouvrez le **Panneau de configuration**.
2. Ouvrez **Microsoft Monitoring Agent**.
3. Cliquez sur l’onglet **Azure Log Analytics**.
4. Dans la colonne **État**, vous devez voir que l’agent est correctement connecté aux journaux Azure Monitor.

   ![status](./media/how-to-npm/12.png)

### <a name="25-open-the-firewall-ports-on-the-monitoring-agent-servers"></a><a name="firewall"></a>2.5 : Ouvrir les ports de pare-feu sur les serveurs de l’agent de supervision

Pour utiliser le protocole TCP, vous devez ouvrir des ports de pare-feu pour veiller à ce que les agents de surveillance puissent communiquer.

Vous pouvez exécuter un script PowerShell pour créer les clés de Registre requises par Network Performance Monitor. Ce script crée également des règles de pare-feu Windows pour autoriser les agents de surveillance à créer des connexions TCP entre eux. Les clés de Registre créées par le script spécifient également s’il faut enregistrer les journaux d’activité de débogage et le chemin d’accès des fichiers journaux. Le script définit également le port TCP de l’agent utilisé pour la communication. Les valeurs de ces clés sont définies automatiquement par le script. Vous ne devez pas modifier manuellement ces clés.

Le port 8084 est ouvert par défaut. Vous pouvez utiliser un port personnalisé en ajoutant le paramètre « portNumber » au script. Toutefois, si vous le faites, vous devez spécifier le même port pour tous les serveurs sur lesquels vous exécutez le script.

>[!NOTE]
>Le script PowerShell « EnableRules » configure des règles de pare-feu Windows uniquement sur le serveur exécutant le script. Si vous avez un pare-feu réseau, vous devez vous assurer qu’il autorise le trafic destiné au port TCP que l’Analyseur de performances réseau utilise.
>
>

Sur les serveurs d’agent, ouvrez une fenêtre PowerShell avec des privilèges administratifs. Exécutez le script PowerShell [EnableRules](https://aka.ms/npmpowershellscript) (que vous avez téléchargé précédemment). N’utilisez pas de paramètres.

![PowerShell_Script](./media/how-to-npm/script.png)

## <a name="step-3-configure-network-security-group-rules"></a><a name="opennsg"></a>Étape 3 : Configurer les règles du groupe de sécurité réseau

Pour surveiller les serveurs d’agent se trouvant dans Azure, vous devez configurer les règles du groupe de sécurité réseau (NSG) pour autoriser le trafic TCP sur un port utilisé par NPM pour les transactions synthétiques. Par défaut, il s’agit du port 8084. Cela permet à un agent de surveillance installé sur une machine virtuelle Azure de communiquer avec un agent de surveillance local.

Pour plus d’informations concernant le groupe de sécurité réseau, consultez [Créer des groupes de sécurité réseau à l’aide du portail Azure](../virtual-network/tutorial-filter-network-traffic.md).

>[!NOTE]
>Vérifiez que vous avez installé les agents (l’agent de serveur local et l’agent de serveur Azure) et que vous avez exécuté le script PowerShell avant de continuer avec cette étape.
>

## <a name="step-4-discover-peering-connections"></a><a name="setupmonitor"></a>Étape 4 : Découvrir les connexions de peering

1. Accédez à la vignette de la vue d’ensemble Network Performance Monitor en vous rendant sur la page **All Resources** (Toutes les ressources), puis en cliquant sur l’espace de travail NPM mis sur liste verte.

   ![espace de travail npm](./media/how-to-npm/npm.png)
2. Cliquez sur la vignette de vue d’ensemble **Network Performance Monitor** pour afficher le tableau de bord. Le tableau de bord contient une page ExpressRoute, qui montre que ExpressRoute est dans un état « non configuré ». Cliquez sur **Installation de la fonctionnalité** pour ouvrir la page de configuration de Network Performance Monitor.

   ![installation de fonctionnalité](./media/how-to-npm/npm2.png)
3. Sur la page de configuration, accédez à l’onglet « Peerings ExpressRoute » situé dans le panneau de gauche. Cliquez ensuite sur **Découvrir maintenant**.

   ![détecter](./media/how-to-npm/13.png)
4. Une fois la détection terminée, vous verrez une liste contenant les éléments suivants :
   * Toutes les connexions de peering Microsoft dans le ou les circuits ExpressRoute associés à cet abonnement.
   * Toutes les connexions de peering privé qui se connectent aux réseaux virtuels associés à cet abonnement.
            
## <a name="step-5-configure-monitors"></a><a name="configmonitor"></a>Étape 5 : Configurer les moniteurs

Dans cette section, vous configurez les analyses. Suivez les étapes pour le type de peering que vous souhaitez surveiller : **peering privé** ou **peering Microsoft**.

### <a name="private-peering"></a>Peering privé

Pour le peering privé, une fois la détection terminée, vous voyez des règles pour un **nom de circuit** et un **nom de réseau virtuel** uniques. Au départ, ces règles sont désactivées.

![rules](./media/how-to-npm/14.png)

1. Cochez la case **Surveiller ce peering**.
2. Cochez la case **Activer le monitoring d’intégrité pour ce peering**.
3. Choisissez les conditions de surveillance. Vous pouvez définir des seuils personnalisés pour la génération d’événements d’intégrité en tapant des valeurs de seuil. Chaque fois que la valeur d’une condition dépasse son seuil sélectionné pour la paire de réseaux/sous-réseaux sélectionnée, un événement d’intégrité est généré.
4. Cliquez sur le bouton AGENTS LOCAUX **Ajouter des agents** pour ajouter les serveurs locaux à partir desquels vous souhaitez surveiller la connexion de peering privé. Assurez-vous de choisir uniquement des agents qui disposent d’une connectivité au point de terminaison de service Microsoft que vous avez spécifié dans la section pour l’étape 2. Les agents locaux doivent être en mesure d’atteindre le point de terminaison à l’aide de la connexion ExpressRoute.
5. Enregistrez les paramètres.
6. Après l’activation des règles et la sélection des valeurs et des agents que vous souhaitez surveiller, vous devez attendre entre 30 minutes et 1 heure pour que les valeurs commencent à s’ajouter et que les vignettes **Surveillance ExpressRoute** deviennent disponibles.

### <a name="microsoft-peering"></a>Peering Microsoft

Pour le peering Microsoft, cliquez sur la ou les connexions de peering Microsoft que vous souhaitez surveiller et configurez les paramètres.

1. Cochez la case **Surveiller ce peering**. 
2. (Facultatif) Vous pouvez modifier le point de terminaison de service Microsoft cible. Par défaut, NPM choisit un point de terminaison de service Microsoft comme cible. NPM surveille la connectivité entre vos serveurs locaux et ce point de terminaison cible via ExpressRoute. 
    * Pour modifier ce point de terminaison cible, cliquez sur le lien **(modifier)** sous **Cible :** , puis sélectionnez un autre point de terminaison de service Microsoft cible dans la liste des URL.
      ![modifier la cible](./media/how-to-npm/edit_target.png)<br>

    * Vous pouvez utiliser une URL ou une adresse IP personnalisée. Cette option est particulièrement utile si vous utilisez le peering Microsoft pour établir une connexion aux services Azure PaaS, par exemple Stockage Azure, les bases de données SQL et les sites web proposés sur des adresses IP publiques. Pour ce faire, cliquez sur le lien **(Utiliser une URL ou adresse IP personnalisée à la place)** en bas de la liste d’URL, puis entrez le point de terminaison public de votre service PaaS Azure qui est connecté via le peering Microsoft ExpressRoute.
    ![URL personnalisée](./media/how-to-npm/custom_url.png)<br>

    * Si vous utilisez ces paramètres facultatifs, assurez-vous que seul le point de terminaison de service Microsoft est sélectionné ici. Le point de terminaison doit être connecté à ExpressRoute et accessible par les agents locaux.
3. Cochez la case **Activer le monitoring d’intégrité pour ce peering**.
4. Choisissez les conditions de surveillance. Vous pouvez définir des seuils personnalisés pour la génération d’événements d’intégrité en tapant des valeurs de seuil. Chaque fois que la valeur d’une condition dépasse son seuil sélectionné pour la paire de réseaux/sous-réseaux sélectionnée, un événement d’intégrité est généré.
5. Cliquez sur le bouton AGENTS LOCAUX **Ajouter des agents** pour ajouter les serveurs locaux à partir desquels vous souhaitez surveiller la connexion de peering Microsoft. Assurez-vous de choisir uniquement des agents qui disposent d’une connectivité aux points de terminaison de service Microsoft que vous avez spécifiés dans la section pour l’étape 2. Les agents locaux doivent être en mesure d’atteindre le point de terminaison à l’aide de la connexion ExpressRoute.
6. Enregistrez les paramètres.
7. Après l’activation des règles et la sélection des valeurs et des agents que vous souhaitez surveiller, vous devez attendre entre 30 minutes et 1 heure pour que les valeurs commencent à s’ajouter et que les vignettes **Surveillance ExpressRoute** deviennent disponibles.

## <a name="step-6-view-monitoring-tiles"></a><a name="explore"></a>Étape 6 : Voir les vignettes de supervision

Une fois que vous voyez les vignettes de surveillance, vos circuits ExpressRoute et ressources de connexion sont surveillés par NPM. Vous pouvez cliquer sur la vignette Peering Microsoft pour explorer au niveau du détail l’intégrité des connexions de peering Microsoft.

![vignettes de surveillance](./media/how-to-npm/15.png)

### <a name="network-performance-monitor-page"></a><a name="dashboard"></a>Page Network Performance Monitor

La page NPM contient une page pour ExpressRoute qui présente une vue d’ensemble de l’intégrité des circuits et des peerings ExpressRoute.

![Capture d’écran montrant un tableau de bord avec une vue d’ensemble de l’intégrité des circuits ExpressRoute et des peerings.](./media/how-to-npm/dashboard.png)

### <a name="list-of-circuits"></a><a name="circuits"></a>Liste des circuits

Pour afficher une liste de tous les circuits ExpressRoute surveillés, cliquez sur la vignette **Circuits ExpressRoute**. Vous pouvez sélectionner un circuit et afficher son état d’intégrité, des graphiques de tendances pour la perte de paquets, l’utilisation de la bande passante et la latence. Les graphiques sont interactifs. Vous pouvez sélectionner une fenêtre de temps personnalisée pour tracer les graphiques. Vous pouvez faire glisser la souris sur une zone sur le graphique pour zoomer et voir des points de données précis.

![circuit_list](./media/how-to-npm/circuits.png)

#### <a name="trend-of-loss-latency-and-throughput"></a><a name="trend"></a>Tendances en matière de perte, de latence et de débit

Les graphiques représentant la bande passante, la latence et la perte sont interactifs. Vous pouvez zoomer sur n’importe quelle section de ces graphiques, à l’aide de contrôles de la souris. Vous pouvez également voir des données de bande passante, de latence et de perte pour d’autres intervalles en cliquant sur l’option **Date/Heure**, située sous le bouton Actions, à gauche.

![tendance](./media/how-to-npm/16.png)

### <a name="peerings-list"></a><a name="peerings"></a>Liste des peerings

Cliquez sur la vignette **Peerings privés** du tableau de bord pour afficher une liste de toutes les connexions à des réseaux virtuels sur le peering privé. Ici, vous pouvez sélectionner une connexion de réseau virtuel et afficher son état d’intégrité, des graphiques de tendances pour la perte de paquets, l’utilisation de la bande passante et la latence.

![liste de circuits](./media/how-to-npm/peerings.png)

### <a name="nodes-view"></a><a name="nodes"></a>Affichage des nœuds

Pour afficher une liste de tous les liens entre les nœuds locaux et les points de terminaison de service des machines virtuelles Azure/Microsoft pour la connexion de peering ExpressRoute choisie, cliquez sur **Afficher les liens de nœud**. Vous pouvez afficher l’état d’intégrité de chaque lien, ainsi que la tendance des pertes et de la latence associée.

![affichage des nœuds](./media/how-to-npm/nodes.png)

### <a name="circuit-topology"></a><a name="topology"></a>Topologie de circuit

Pour afficher la topologie de circuit, cliquez sur la vignette **Topologie**. Vous accédez ainsi à l’affichage de la topologie du circuit ou du peering. Le diagramme de topologie fournit la latence pour chaque segment du réseau. Chaque tronçon de couche 3 est représenté par un nœud du diagramme. En cliquant sur un tronçon, vous pouvez obtenir plus de détails sur le tronçon.

Vous pouvez augmenter le niveau de visibilité pour inclure des sauts locaux en déplaçant le curseur sous **Filtres**. Déplacez le curseur vers la droite ou gauche pour augmenter/diminuer le nombre de sauts dans le graphique de la topologie. La latence pour chaque segment est visible, ce qui permet une isolation plus rapide des segments à latence élevée sur votre réseau.

![filtres](./media/how-to-npm/topology.png)

#### <a name="detailed-topology-view-of-a-circuit"></a>Vue détaillée de la topologie d’un circuit

Cette vue affiche les connexions de réseau virtuel.
![topologie détaillée](./media/how-to-npm/17.png)
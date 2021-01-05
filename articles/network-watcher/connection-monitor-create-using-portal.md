---
title: Créer une instance Moniteur de connexion – Portail Azure
titleSuffix: Azure Network Watcher
description: Cet article explique comment créer un moniteur dans Moniteur de connexion à l’aide du portail Azure.
services: network-watcher
documentationcenter: na
author: vinigam
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 11/23/2020
ms.author: vinigam
ms.openlocfilehash: a6ba9c02b384c1b30977d962659983d600c7b2ac
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95545718"
---
# <a name="create-a-monitor-in-connection-monitor-by-using-the-azure-portal"></a>Créer un moniteur dans Moniteur de connexion à l’aide du portail Azure

Découvrez comment utiliser Moniteur de connexion pour surveiller la communication entre vos ressources. Cet article explique comment créer une analyse à l’aide du portail Azure. Le Moniteur de connexion prend en charge les déploiements cloud hybrides et Azure.


## <a name="before-you-begin"></a>Avant de commencer 

Dans les moniteurs de connexion que vous créez à l’aide de Moniteur de connexion, vous pouvez ajouter des machines locales et des machines virtuelles Azure en tant que sources. Ces moniteurs de connexion peuvent également surveiller la connectivité aux points de terminaison. Les points de terminaison peuvent se trouver sur Azure ou sur toute autre URL ou adresse IP.

Voici quelques définitions pour démarrer :

* **Ressource de moniteur de connexion**. Ressource Azure spécifique à une région. Toutes les entités ci-dessous sont des propriétés d’une ressource de moniteur de connexion.
* **Point de terminaison**. Source ou destination qui participe aux vérifications de la connectivité. Voici quelques exemples de points de terminaison :
    * Machines virtuelles Azure.
    * Réseaux virtuels Azure.
    * Sous-réseaux Azure.
    * Agents locaux.
    * Sous-réseaux locaux.
    * Réseaux personnalisés locaux qui incluent plusieurs sous-réseaux.
    * URL et adresses IP.
* **Configuration de test**. Configuration spécifique à un protocole dans le cadre d’un test. En fonction du protocole que vous choisissez, vous pouvez définir le port, les seuils, la fréquence de test et d’autres paramètres.
* **Groupe de tests**. Groupe contenant les points de terminaison sources, les points de terminaison de destination et les configurations de test. Un moniteur de connexion peut contenir plusieurs groupes de tests.
* **Test**. Combinaison d’un point de terminaison source, d’un point de terminaison de destination et d’une configuration de test. Un test correspond au niveau le plus granulaire auquel les données de surveillance sont disponibles. Les données de surveillance incluent le pourcentage de vérifications qui ont échoué et la durée des boucles.

:::image type="content" source="./media/connection-monitor-2-preview/cm-tg-2.png" alt-text="Diagramme montrant un moniteur de connexion et définissant la relation entre les groupes de tests et les tests.":::


## <a name="create-a-connection-monitor"></a>Créer un moniteur de connexion

Pour créer un moniteur dans Moniteur de connexion à l’aide du portail Azure :

1. Sur la page d'accueil du portail Azure, accédez à **Network Watcher**.
1. Dans le volet gauche, dans la section **Surveillance**, sélectionnez **Moniteur de connexion**.

   Tous les moniteurs de connexion créés dans Moniteur de connexion sont répertoriés. Pour afficher les moniteurs de connexion créés dans le Moniteur de connexion classique, accédez à l’onglet **Moniteur de connexion**.

   :::image type="content" source="./media/connection-monitor-2-preview/cm-resource-view.png" alt-text="Capture d’écran montrant des moniteurs de connexion créés dans Moniteur de connexion.":::
   
    
1. En haut à gauche du tableau de bord **Moniteur de connexion**, sélectionnez **Créer**.

   

1. Sous l'onglet **Informations de base**, entrez les informations relatives à votre moniteur de connexion : 
   * **Nom du moniteur de connexion** : Entrez un nom pour votre moniteur de connexion. Appliquez les règles de nommage standard des ressources Azure.
   * **Abonnement**: Sélectionnez un abonnement pour votre moniteur de connexion.
   * **Région** : Sélectionnez une région pour votre moniteur de connexion. Vous ne pouvez sélectionner que les machines virtuelles sources créées dans cette région.
   * **Configuration de l’espace de travail** : Choisissez un espace de travail personnalisé ou l’espace de travail par défaut. Votre espace de travail contient vos données de surveillance.
       * Pour utiliser l'espace de travail par défaut, cochez la case. 
       * Pour choisir un espace de travail personnalisé, décochez la case. Sélectionnez ensuite l’abonnement et la région de votre espace de travail personnalisé. 

   :::image type="content" source="./media/connection-monitor-2-preview/create-cm-basics.png" alt-text="Capture d’écran montrant l’onglet Informations de base du Moniteur de connexion.":::
   
1. En bas de l'onglet, sélectionnez **Suivant : Groupes de tests**.

1. Ajoutez des sources, des destinations et des configurations de test dans vos groupes de tests. Pour en savoir plus sur la configuration de vos groupes de tests, consultez [Créer des groupes de tests dans Moniteur de connexion](#create-test-groups-in-a-connection-monitor). 

   :::image type="content" source="./media/connection-monitor-2-preview/create-tg.png" alt-text="Capture d’écran montrant l’onglet Groupes de tests dans Moniteur de connexion.":::

1. En bas de l'onglet, sélectionnez **Suivant : Créer des alertes**. Pour en savoir plus sur la création d’alertes, consultez [Créer des alertes dans Moniteur de connexion](#create-alerts-in-connection-monitor).

   :::image type="content" source="./media/connection-monitor-2-preview/create-alert.png" alt-text="Capture d’écran montrant l’onglet Créer une alerte.":::

1. En bas de l'onglet, sélectionnez **Suivant : Vérifier + créer**.

1. Sous l'onglet **Examiner + créer**, passez en revue les informations de base et les groupes de tests avant de créer le moniteur de connexion. Si vous devez modifier le moniteur de connexion, vous pouvez le faire en revenant aux onglets correspondants. 
   :::image type="content" source="./media/connection-monitor-2-preview/review-create-cm.png" alt-text="Capture d’écran montrant l’onglet Vérifier + créer dans Moniteur de connexion.":::
   > [!NOTE] 
   > L’onglet **Vérifier + créer** indique le coût mensuel pendant la phase Moniteur de connexion. Actuellement, rien n’apparaît dans la colonne **Coût actuel/mois**. Lorsque le Moniteur de connexion sera mis à la disposition générale, cette colonne affichera les frais mensuels. 
   > 
   > En revanche, des frais d’ingestion de Log Analytics s’appliquent pendant la phase Moniteur de connexion.

1. Lorsque vous êtes prêt à créer le moniteur de connexion, sélectionnez **Créer** en bas de l'onglet **Examiner + créer**.

Moniteur de connexion crée la ressource de moniteur de connexion en arrière-plan.

## <a name="create-test-groups-in-a-connection-monitor"></a>Créer des groupes de tests dans un moniteur de connexion

Chaque groupe de tests d'un moniteur de connexion comprend des sources et des destinations dont les paramètres réseau sont testés. Les tests concernent le pourcentage de vérifications qui échouent et la durée des boucles sur les configurations de test.

Dans le portail Azure, pour créer un groupe de tests dans un moniteur de connexion, vous spécifiez des valeurs dans les champs suivants :

* **Désactiver le groupe de tests** : Vous pouvez cocher cette case pour désactiver la surveillance de toutes les sources et destinations spécifiées par le groupe de tests. Cette sélection est désactivée par défaut.
* **Nom** : Donnez un nom à votre groupe de tests.
* **Sources** Vous pouvez spécifier des machines virtuelles Azure et des machines locales en tant que sources si des agents y sont installés. Pour en savoir plus sur l’installation d’un agent pour votre source, consultez [Installer des agents de surveillance](./connection-monitor-overview.md#install-monitoring-agents).
   * Pour choisir des agents Azure, sélectionnez l’onglet **Points de terminaison Azure**. Vous ne voyez ici que les machines virtuelles liées à la région que vous avez spécifiée lors de la création du moniteur de connexion. Par défaut, les machines virtuelles sont regroupées au sein de l'abonnement auquel elles appartiennent. Ces groupes sont réduits. 
   
       Vous pouvez passer du niveau **Abonnement** à d’autres niveaux de la hiérarchie :

      **Abonnement** > **Groupes de ressources** > **Réseau virtuel** > **Sous-réseau** > **Machines virtuelles avec agents**

      Vous pouvez également modifier le sélecteur **Grouper par** pour démarrer l’arborescence à partir d’un autre niveau. Par exemple, si vous effectuez un regroupement par réseau virtuel, vous obtenez la liste des machines virtuelles disposant d’agents dans la hiérarchie **Réseau virtuel** > **Sous-réseau** > **Machines virtuelles avec agents**.

       Lorsque vous sélectionnez un réseau virtuel, un sous-réseau ou une machine virtuelle unique, l’ID de ressource correspondant est défini en tant que point de terminaison. Par défaut, toutes les machines virtuelles du réseau virtuel ou du sous-réseau sélectionné avec l’extension Azure Network Watcher participent à la surveillance. Pour réduire l’étendue, sélectionnez des sous-réseaux ou des agents spécifiques ou modifiez la valeur de la propriété d’étendue. 

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-sources.png" alt-text="Capture d’écran montrant le volet Ajouter des sources et l’onglet Points de terminaison Azure dans Moniteur de connexion.":::

   * Pour choisir des agents locaux, sélectionnez l’onglet **Points de terminaison non Azure**. Par défaut, les agents sont regroupés par région dans les espaces de travail. Network Performance Monitor est configuré pour tous ces espaces de travail. 
   
       Si nécessaire, procurez-vous Network Performance Monitor à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/Microsoft.NetworkMonitoringOMS?tab=Overview) pour l'ajouter à votre espace de travail. Pour plus d'informations sur l'ajout de Network Performance Monitor, consultez [Solutions de supervision dans Azure Monitor](../azure-monitor/insights/solutions.md). 
   
       Sous **Créer un moniteur de connexion**, dans l’onglet **Informations de base**, la région par défaut est sélectionnée. Si vous changez la région, vous pouvez choisir des agents des espaces de travail de la nouvelle région. Vous pouvez sélectionner un ou plusieurs agents ou sous-réseaux. Dans la vue **Sous-réseau**, vous pouvez sélectionner des adresses IP spécifiques pour la surveillance. Si vous ajoutez plusieurs sous-réseaux, un réseau local personnalisé nommé **OnPremises_Network_1** sera créé. Vous pouvez également modifier le sélecteur **Grouper par** pour regrouper par agents.

      :::image type="content" source="./media/connection-monitor-2-preview/add-non-azure-sources.png" alt-text="Capture d’écran montrant le volet Ajouter des sources et l’onglet Points de terminaison non Azure dans Moniteur de connexion.":::

   * Pour choisir les points de terminaison récemment utilisés, vous pouvez utiliser l’onglet **Point de terminaison récent** 
   
   * Une fois la configuration des sources terminée, sélectionnez **Terminé** en bas de l’onglet. Vous pouvez toujours modifier des propriétés de base telles que le nom du point de terminaison en sélectionnant le point de terminaison dans la vue **Créer un groupe de tests**. 

* **Destinations** : Vous pouvez surveiller la connectivité à une machine virtuelle Azure, à un ordinateur local ou à un point de terminaison (adresse IP publique, URL ou FQDN) en le spécifiant en tant que destination. Au sein d’un même groupe de tests, vous pouvez ajouter des machines virtuelles Azure, des ordinateurs locaux, des URL Office 365, des URL Dynamics 365 et des points de terminaison personnalisés.

    * Pour choisir des machines virtuelles Azure comme destinations, sélectionnez l’onglet **Points de terminaison Azure**. Par défaut, les machines virtuelles Azure sont regroupées dans une hiérarchie d’abonnement qui se trouve dans la région que vous avez sélectionnée sous **Créer un moniteur de connexion** dans l’onglet **Informations de base**. Vous pouvez changer la région et choisir des machines virtuelles Azure dans la nouvelle région. Vous pouvez ensuite descendre dans la hiérarchie à partir du niveau **Abonnement** à d’autres niveaux de la hiérarchie, comme vous pouvez le faire lorsque vous définissez les points de terminaison Azure sources.

      Vous pouvez sélectionner des réseaux virtuels, des sous-réseaux ou des machines virtuelles uniques, comme vous pouvez le faire lorsque vous définissez les points de terminaison Azure sources. Lorsque vous sélectionnez un réseau virtuel, un sous-réseau ou une machine virtuelle unique, l’ID de ressource correspondant est défini en tant que point de terminaison. Par défaut, toutes les machines virtuelles du réseau virtuel ou du sous-réseau sélectionné avec l’extension Network Watcher participent à la surveillance. Pour réduire l’étendue, sélectionnez des sous-réseaux ou des agents spécifiques ou modifiez la valeur de la propriété d’étendue. 

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-dests1.png" alt-text="<Capture d’écran montrant le volet Ajouter des destinations et l’onglet Points de terminaison Azure.>":::

      :::image type="content" source="./media/connection-monitor-2-preview/add-azure-dests2.png" alt-text="<Capture d’écran montrant le volet Ajouter des destinations au niveau Abonnement.>":::
       
    
    * Pour choisir des agents non Azure comme destinations, sélectionnez l’onglet **Points de terminaison non Azure**. Par défaut, les agents sont regroupés par région dans les espaces de travail. Network Performance Monitor est configuré pour tous ces espaces de travail. 
    
      Si nécessaire, procurez-vous Network Performance Monitor à partir de la Place de marché Azure pour l'ajouter à votre espace de travail. Pour plus d'informations sur l'ajout de Network Performance Monitor, consultez [Solutions de supervision dans Azure Monitor](../azure-monitor/insights/solutions.md). 

      Sous  **Créer un moniteur de connexion**, dans l’onglet  **Informations de base** , la région par défaut est sélectionnée. Si vous changez la région, vous pouvez choisir des agents des espaces de travail de la nouvelle région. Vous pouvez sélectionner un ou plusieurs agents ou sous-réseaux. Dans la vue **Sous-réseau**, vous pouvez sélectionner des adresses IP spécifiques pour la surveillance. Si vous ajoutez plusieurs sous-réseaux, un réseau local personnalisé nommé **OnPremises_Network_1** sera créé.  

      :::image type="content" source="./media/connection-monitor-2-preview/add-non-azure-dest.png" alt-text="Capture d’écran montrant le volet Ajouter des destinations et l’onglet Points de terminaison non Azure.":::
    
    * Pour choisir des points de terminaison publics comme destinations, sélectionnez l’onglet **Adresses externes**. La liste des points de terminaison comprend les URL de test Office 365 et les URL de test Dynamics 365, regroupées par nom. Vous pouvez également choisir des points de terminaison que vous avez créés dans d’autres groupes de tests du même moniteur de connexion. 
    
        Pour ajouter un point de terminaison, sélectionnez **Ajouter un point de terminaison** en haut à droite. Fournissez ensuite un nom de point de terminaison ainsi qu'une URL, une adresse IP ou un FQDN.

       :::image type="content" source="./media/connection-monitor-2-preview/add-endpoints.png" alt-text="Capture d’écran montrant où ajouter des points de terminaison publics comme destinations dans Moniteur de connexion.":::

    * Pour choisir des points de terminaison récemment utilisés, accédez à l’onglet  **Point de terminaison récent** .
    * Une fois les destinations choisies, sélectionnez **Terminé**. Vous pouvez toujours modifier des propriétés de base telles que le nom du point de terminaison en sélectionnant le point de terminaison dans la vue **Créer un groupe de tests**. 

* **Configurations de test** : Vous pouvez ajouter une ou plusieurs configurations de test à un groupe de tests. Créez une nouvelle configuration de test à l’aide de l’onglet **Nouvelle configuration**. Vous pouvez également ajouter une configuration de test à partir d’un autre groupe de tests dans le même moniteur de connexion dans l’onglet **Choisir une existante**.

    * **Nom de la configuration de test** : Donnez un nom à la configuration de test.
    * **Protocole** : Sélectionnez **TCP**, **ICMP** ou **HTTP**. Pour remplacer HTTP par HTTPS, sélectionnez le protocole **HTTP** puis le port **443**.
        * **Créer une configuration de test TCP** : Cette case à cocher n’apparaît que si vous sélectionnez **HTTP** dans la liste **Protocole**. Cochez cette case pour créer une autre configuration de test utilisant les mêmes sources et destinations que celles que vous avez spécifiées ailleurs dans votre configuration. La nouvelle configuration de test est nommée **\<name of test configuration>_networkTestConfig**.
        * **Désactiver traceroute** : Cette case à cocher s’applique lorsque le protocole est TCP ou ICMP. Cochez cette case pour empêcher les sources de découvrir la topologie et la durée des boucles tronçon par tronçon.
    * **Port de destination** : Vous pouvez spécifier le port de destination de votre choix.
        * **Écouter sur le port** : Cette case à cocher s’applique lorsque le protocole est TCP. Cochez cette case pour ouvrir le port TCP choisi s’il n’est pas déjà ouvert. 
    * **Fréquence de test** : Dans cette liste, choisissez la fréquence à laquelle les sources effectueront un test Ping des destinations à l’aide du protocole et du port que vous avez spécifiés. Vous pouvez choisir 30 secondes, 1 minute, 5 minutes, 15 minutes ou 30 minutes. Sélectionnez **personnalisé** pour entrer une autre fréquence qui est comprise entre 30 secondes et 30 minutes. Les sources testeront la connectivité aux destinations en fonction de la valeur que vous avez choisie. Par exemple, si vous sélectionnez 30 secondes, les sources vérifieront la connectivité à la destination au moins une fois toutes les 30 secondes.
    * **Seuil de succès** : Vous pouvez définir des seuils sur les paramètres réseau suivants :
       * **Vérifications ayant échoué** : Définissez le pourcentage de vérifications qui peuvent échouer lorsque les sources vérifient la connectivité aux destinations à l’aide des critères que vous avez spécifiés. Pour le protocole TCP ou ICMP, le pourcentage de vérifications qui ont échoué peut être égal au pourcentage de perte de paquets. Pour le protocole HTTP, cette valeur représente le pourcentage de requêtes HTTP qui n’ont reçu aucune réponse.
       * **Durée aller-retour** : Définissez la durée des boucles en millisecondes pour déterminer combien de temps les sources peuvent prendre pour se connecter à la destination sur la configuration de test.
       
   :::image type="content" source="./media/connection-monitor-2-preview/add-test-config.png" alt-text="Capture d’écran montrant où définir une configuration de test dans Moniteur de connexion.":::
       
## <a name="create-alerts-in-connection-monitor"></a>Créer des alertes dans Moniteur de connexion

Vous pouvez configurer des alertes sur les tests qui échouent en fonction des seuils définis dans les configurations de test.

Dans le portail Azure, pour créer des alertes pour un moniteur de connexion, vous spécifiez des valeurs dans ces champs : 

- **Créer une alerte** : Vous pouvez cocher cette case pour créer une alerte de mesure dans Azure Monitor. Lorsque vous cochez cette case, les autres champs sont activés pour modification. Des frais supplémentaires pour l’alerte seront applicables, en fonction de la [tarification pour les alertes](https://azure.microsoft.com/pricing/details/monitor/). 

- **Étendue** > **Ressource** > **Hiérarchie** : Ces valeurs sont automatiquement remplies, en fonction des valeurs spécifiées dans l’onglet **Informations de base**.

- **Nom de la condition** : L’alerte est créée sur la métrique `Test Result(preview)`. Lorsque le résultat du test du moniteur de connexion, la règle d’alerte est déclenchée. 

- **Nom du groupe d’actions** : Vous pouvez entrer votre adresse e-mail directement ou vous pouvez créer des alertes via des groupes d’actions. Si vous entrez votre adresse e-mail directement, un groupe d’actions nommé **NPM Email ActionGroup** est créé. L’ID d’e-mail est ajouté à ce groupe d’actions. Si vous choisissez d’utiliser des groupes d’actions, vous devez sélectionner un groupe d’actions précédemment créé. Pour en savoir plus sur la création d’un groupe d’actions, consultez [Créer des groupes d’actions dans le portail Azure](../azure-monitor/platform/action-groups.md). Une fois l’alerte créée, vous pouvez [gérer vos alertes](../azure-monitor/platform/alerts-metric.md#view-and-manage-with-azure-portal). 

- **Nom de la règle d’alerte** : Nom du moniteur de connexion.

- **Activer la règle lors de sa création** : Cochez cette case pour activer la règle d’alerte en fonction de la condition. Décochez cette case si vous souhaitez créer la règle sans l’activer. 

:::image type="content" source="./media/connection-monitor-2-preview/create-alert-filled.png" alt-text="Capture d’écran montrant l’onglet Créer une alerte dans Moniteur de connexion.":::

## <a name="scale-limits"></a>Limites de mise à l’échelle

Les moniteurs de connexion présentent les limites de mise à l’échelle suivantes :

* Nombre maximum de moniteurs de connexion par abonnement et par région : 100
* Nombre maximum de groupes de tests par moniteur de connexion : 20
* Nombre maximum de sources et de destinations par moniteur de connexion : 100
* Nombre maximum de configurations de test par moniteur de connexion : 2 via le portail Azure

## <a name="next-steps"></a>Étapes suivantes

* Découvrez [comment analyser les données de surveillance et définir des alertes](./connection-monitor-overview.md#analyze-monitoring-data-and-set-alerts).
* Découvrez [comment diagnostiquer les problèmes dans votre réseau](./connection-monitor-overview.md#diagnose-issues-in-your-network).
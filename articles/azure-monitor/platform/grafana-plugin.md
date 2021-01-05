---
title: Assurer le monitoring des applications et services Azure à l’aide de Grafana
description: Acheminez les données Azure Monitor et Application Insights en vue de les afficher dans Grafana.
ms.subservice: ''
ms.topic: conceptual
ms.date: 11/06/2017
ms.openlocfilehash: 23bba091628eee767fbf292a8a8d772ffab674cb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87073460"
---
# <a name="monitor-your-azure-services-in-grafana"></a>Surveiller vos services Azure dans Grafana
Maintenant, vous pouvez superviser les services et applications Azure à partir de [Grafana](https://grafana.com/) en utilisant le [plug-in Azure Monitor de source de données](https://grafana.com/plugins/grafana-azure-monitor-datasource). Le plug-in rassemble des données de performance d’application collectées par Azure Monitor, y compris divers journaux d’activité et métriques. Vous pouvez ensuite afficher ces données dans votre tableau de bord Grafana.

Utilisez les étapes suivantes pour configurer un serveur Grafana et générer des tableaux de bord pour les métriques et les journaux d’activité depuis Azure Monitor.

## <a name="set-up-a-grafana-server"></a>Configurer un serveur Grafana

### <a name="set-up-grafana-locally"></a>Configurer Grafana localement
Pour configurer un serveur Grafana local, [téléchargez et installez Grafana dans votre environnement local](https://grafana.com/grafana/download). Pour utiliser l’intégration Azure Monitor du plug-in, installez Grafana version 5.3 ou ultérieure.

### <a name="set-up-grafana-on-azure-through-the-azure-marketplace"></a>Configurer Grafana sur Azure par l’intermédiaire de la Place de marché Azure
1. Accédez à la Place de marché Azure et choisissez Grafana, de Grafana Labs.

2. Renseignez les noms et les détails. Créez un groupe de ressources. Effectuez le suivi des valeurs que vous choisissez pour le nom d’utilisateur de la machine virtuelle, le mot de passe de la machine virtuelle et le mot de passe d’administrateur du serveur Grafana.  

3. Choisissez une taille de machine virtuelle et un compte de stockage.

4. Configurez les paramètres de configuration du réseau.

5. Affichez le résumé et sélectionnez **Créer** après avoir accepté les conditions d’utilisation.

6. Lorsque le déploiement est terminé, sélectionnez **Accéder au groupe de ressources**. Une liste de ressources nouvellement créées s’affiche.

    ![Objets du groupe de ressources Grafana](media/grafana-plugin/grafana1.png)

    Si vous sélectionnez le groupe de sécurité réseau (*grafana-nsg* dans ce cas), vous vous apercevez que le port 3000 est utilisé pour accéder au serveur Grafana.

7. Récupérez l’adresse IP publique de votre serveur Grafana, puis revenez à la liste des ressources et sélectionnez **Adresse IP publique**.

## <a name="sign-in-to-grafana"></a>Se connecter à Grafana

1. À l’aide de l’adresse IP de votre serveur, ouvrez la page de connexion *http://\<IP address\>:3000* ou le *\<DNSName>\:3000* dans votre navigateur. 3000 étant le port par défaut, notez que vous avez peut-être sélectionné un port différent pendant la configuration. Une page de connexion doit s’afficher pour le serveur Grafana que vous avez créé.

    ![Écran de connexion Grafana](./media/grafana-plugin/grafana-login-screen.png)

2. Connectez-vous avec le nom d’utilisateur *admin* et le mot de passe d’administrateur du serveur Grafana que vous avez créé précédemment. Si vous utilisez une configuration locale, le mot de passe par défaut doit être *administrateur* ; il vous est demandé de le changer lors de votre première connexion.

## <a name="configure-data-source-plugin"></a>Configurer le plug-in de source de données

Lorsque vous êtes correctement connecté, vous devez voir que le plug-in Azure Monitor de source de données est déjà inclus.

![Grafana comprend le plug-in Azure Monitor](./media/grafana-plugin/grafana-includes-azure-monitor-plugin-dark.png)

1. Sélectionnez **Add data source** (Ajouter une source de données) pour ajouter et configurer la source de données Azure Monitor.

2. Choisissez un nom pour la source de données et sélectionnez **Azure Monitor** comme type dans la liste déroulante.

3. Créer un principal de service - Grafana utilise un principal de service Azure Active Directory pour se connecter à des API Azure Monitor et collecter des données. Vous devez créer un principal de service ou en utiliser un existant pour gérer l’accès à vos ressources Azure.
    * Consultez [ces instructions](../../active-directory/develop/howto-create-service-principal-portal.md) pour créer un principal de service. Copiez et enregistrez votre ID de locataire (ID d’annuaire), ID client (ID d’application) et le secret client (valeur de la clé d’application).
    * Consultez [Affecter l’application à un rôle](../../active-directory/develop/howto-create-service-principal-portal.md) pour assigner le rôle de lecteur à l’application Azure Active Directory dans l’abonnement, le groupe de ressources ou la ressource que vous souhaitez superviser. 
    L’API Log Analytics a besoin du [rôle Lecteur Log Analytics](../../role-based-access-control/built-in-roles.md#log-analytics-reader) qui inclut les autorisations du rôle de lecteur et les lui ajoute.

4. Fournissez les informations de connexion aux API que vous souhaitez utiliser. Vous pouvez vous connecter à l’intégralité de ces API ou à certaines d’entre elles. 
    * Si vous vous connectez aux métriques et aux journaux dans Azure Monitor, vous pouvez réutiliser les mêmes informations d’identification en sélectionnant **Same details as Azure Monitor API (Mêmes détails que l’API Azure Monitor)** .
    * Lors de la configuration du plug-in, vous pouvez indiquer quel cloud Azure vous souhaitez que le plug-in supervise (Public, Azure - US Government, Azure - Allemagne ou Azure - Chine).
    * Si vous utilisez Application Insights, vous pouvez également inclure votre API Application Insights et l’ID de l’application pour collecter les métriques se rapportant à Application Insights. Pour plus d’informations, consultez [Récupération de votre clé API et de votre ID d’application](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID).

        > [!NOTE]
        > Certains champs de source de données sont nommés différemment de leurs paramètres Azure corrélés :
        > * L’ID de locataire est l’ID d’annuaire Azure
        > * L’ID client est l’ID d’application Azure Active Directory
        > * Le secret client est la valeur de la clé de l’application Azure Active Directory

5. Si vous utilisez Application Insights, vous pouvez également inclure votre API Application Insights et l’ID de l’application pour collecter les métriques se rapportant à Application Insights. Pour plus d’informations, consultez [Récupération de votre clé API et de votre ID d’application](https://dev.applicationinsights.io/documentation/Authorization/API-key-and-App-ID).

6. Sélectionnez **Save** (Enregistrer) pour que Grafana teste les informations d’identification de chaque API. Un message similaire à celui qui suit doit s’afficher.  
    ![Grafana - Configuration approuvée d’une source de données](./media/grafana-plugin/grafana-data-source-config-approved-dark.png)

## <a name="build-a-grafana-dashboard"></a>Créer un tableau de bord Grafana

1. Accédez à la page d’accueil de Grafana, puis sélectionnez **New Dashboard** (Nouveau tableau de bord).

2. Dans le nouveau tableau de bord, sélectionnez le **Graph** (Graphe). Vous pouvez essayer d’autres options de création de graphiques, mais cet article utilise *Graph* (Graphe) comme exemple.

3. Un graphe vide s’affiche dans votre tableau de bord. Cliquez sur le titre du volet et sélectionnez **Edit** (Modifier) pour entrer les détails des données à tracer dans ce graphe.
    ![Grafana - Nouveau graphe](./media/grafana-plugin/grafana-new-graph-dark.png)

4. Sélectionnez la source de données Azure Monitor que vous avez configurée.
   * Collecte des métriques Azure Monitor - Sélectionnez **Azure Monitor** dans la liste déroulante du service. Une liste de sélecteurs s’affiche, dans laquelle vous pouvez sélectionner les ressources et le métrique à superviser dans ce graphe. Pour collecter les métriques à partir d’une machine virtuelle, utilisez l’espace de noms **Microsoft.Compute/VirtualMachines**. Après avoir sélectionné les machines virtuelles et les métriques, vous pouvez commencer à voir leurs données s’afficher dans le tableau de bord.
     ![Grafana - Configuration du graphe pour Azure Monitor](./media/grafana-plugin/grafana-graph-config-for-azure-monitor-dark.png)
   * Collecte des données de journal Azure Monitor - Sélectionnez **Azure Log Analytics** dans la liste déroulante du service. Sélectionnez l’espace de travail que vous souhaitez interroger et définissez le texte de la requête. Vous pouvez copier ici n’importe quelle requête de journal que vous avez déjà, ou en créer une nouvelle. Au fur et à mesure que vous tapez votre requête, IntelliSense apparaît en suggérant des options de saisie semi-automatique. Sélectionnez le type de visualisation, **Time series** (Série chronologique) **Table** (Tableau), et exécutez la requête.
    
     > [!NOTE]
     >
     > La requête par défaut fournie avec le plug-in utilise deux macros : "$__timeFilter() et $__interval. 
     > Lorsque vous effectuez un zoom sur une partie d’un graphe, ces macros permettent à Grafana de calculer dynamiquement la plage de temps et le fragment de temps. Vous pouvez supprimer ces macros et utiliser un filtre d’heure standard, tel que *TimeGenerated > ago(1h)* , mais cela signifie que le graphe ne prend plus en charge la fonctionnalité de zoom.
    
     ![Grafana - Configuration du graphe pour Azure Log Analytics](./media/grafana-plugin/grafana-graph-config-for-azure-log-analytics-dark.png)

5. Voici un tableau de bord simple avec deux graphes. Celui qui est situé à gauche affiche le pourcentage UC de deux machines virtuelles. Le graphe de droite montre les transactions dans un compte de Stockage Azure ventilé par le type API de transaction.
    ![Exemple de deux graphes Grafana](media/grafana-plugin/grafana6.png)


## <a name="optional-monitor-your-custom-metrics-in-the-same-grafana-server"></a>Facultatif : Surveiller vos métriques personnalisés dans le même serveur Grafana

Vous pouvez également installer Telegraf et InfluxDB pour collecter et tracer des métriques à la fois personnalisés et basés sur agent dans la même instance Grafana. Il existe de nombreux plug-ins de source de données à utiliser pour rassembler ces métriques dans un tableau de bord.

Vous pouvez également réutiliser cette configuration pour inclure des métriques à partir de votre serveur Prometheus. Utilisez le plug-in Prometheus de source de données dans la galerie des plug-ins de Grafana.

Voici des articles de référence sur l’utilisation de Telegraf, InfluxDB, Prometheus et Docker
 - [Guide pratique pour surveiller les métriques système avec TICK Stack sur Ubuntu 16.04](https://www.digitalocean.com/community/tutorials/how-to-monitor-system-metrics-with-the-tick-stack-on-ubuntu-16-04)

 - [Une solution de supervision pour les hôtes, conteneurs et services en conteneur Docker](https://stefanprodan.com/2016/a-monitoring-solution-for-docker-hosts-containers-and-containerized-services/)

Voici l’image d’un tableau de bord Grafana complet qui comporte des métriques provenant d’Azure Monitor et d’Application Insights.
![Exemple de métriques de Grafana](media/grafana-plugin/grafana8.png)

## <a name="advanced-grafana-features"></a>Fonctionnalités avancées de Grafana

### <a name="variables"></a>Variables
Certaines valeurs de requête peuvent être sélectionnées via les menus déroulants de l’interface utilisateur, et mises à jour dans la requête. Examinez la requête suivante en guise d’exemple :
```
Usage 
| where $__timeFilter(TimeGenerated) 
| summarize total_KBytes=sum(Quantity)*1024 by bin(TimeGenerated, $__interval) 
| sort by TimeGenerated
```

Vous pouvez configurer une variable pour qu’elle liste toutes les valeurs de **Solution** disponibles, puis mette à jour votre requête pour l’utiliser.
Pour créer une variable, cliquez sur le bouton Settings (Paramètres) du tableau de bord situé dans la zone supérieure droite, sélectionnez **Variables**, puis **New** (Nouveau).
Dans la page de la variable, définissez la source de données et la requête à exécuter afin d’obtenir la liste des valeurs.
![Grafana - Configurer une variable](./media/grafana-plugin/grafana-configure-variable-dark.png)

Une fois créée, ajustez la requête pour utiliser la ou les valeurs sélectionnées, afin que vos graphes réagissent en conséquence :
```
Usage 
| where $__timeFilter(TimeGenerated) and Solution in ($Solutions)
| summarize total_KBytes=sum(Quantity)*1024 by bin(TimeGenerated, $__interval) 
| sort by TimeGenerated
```
    
![Grafana - Utiliser des variables](./media/grafana-plugin/grafana-use-variables-dark.png)

### <a name="create-dashboard-playlists"></a>Créer des sélections de tableau de bord

Une des nombreuses fonctionnalités intéressantes de Grafana repose sur la sélection de tableaux de bord. Vous pouvez créer plusieurs tableaux de bord et les ajouter à une liste en configurant un intervalle pour que chaque tableau de bord s’affiche. Sélectionnez **Play** (Lire) pour voir le cycle des tableaux de bord. Vous pouvez les afficher sur un grand écran mural pour fournir un tableau complet à votre groupe.

![Exemple de sélection Grafana](./media/grafana-plugin/grafana7.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous avez configuré un environnement Grafana sur Azure, vous êtes facturé lorsque les machines virtuelles sont en cours d’exécution, que vous les utilisiez ou non. Pour éviter des frais supplémentaires, nettoyez le groupe de ressources créé dans cet article.

1. Dans le menu de gauche du portail Azure, cliquez sur **Groupes de ressources** et sur **Grafana**.
2. Dans la page de votre groupe de ressources, cliquez sur **Supprimer**, tapez **Grafana** dans la zone de texte, puis cliquez sur **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes
* [Vue d’ensemble des métriques d’Azure Monitor](data-platform.md)

---
title: Générer des cartes
description: Cet article décrit comment générer des cartes dans Azure FarmBeats.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 92228c691c323bc0b9621dfc7413d86c5c2669e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84709060"
---
# <a name="generate-maps"></a>Générer des cartes

Vous pouvez générer les cartes suivantes dans Azure FarmBeats à l’aide de l’imagerie satellite et des entrées de données des capteurs. Les cartes vous permettent de voir l’emplacement géographique de votre exploitation agricole et de déterminer l’emplacement approprié de vos appareils.

  - **Cartes d’emplacement des capteurs** : Fournit des recommandations sur le nombre de capteurs à utiliser et où les placer dans l’exploitation agricole.
  - **Carte d’indices satellite** : Indique l’indice de végétation et l’indice de teneur en eau pour une exploitation agricole.
  - **Carte thermique d’humidité du sol** : Indique la distribution de l’humidité du sol en fusionnant les données satellite et les données des capteurs.

## <a name="sensor-placement-map"></a>Cartes d’emplacement des capteurs

Une carte d’emplacement des capteurs FarmBeats vous aide à placer les capteurs d’humidité du sol. La sortie de la carte offre une liste de coordonnées pour le déploiement des capteurs. Les entrées de ces capteurs sont utilisées avec l’imagerie satellite pour générer la carte thermique d’humidité du sol.

Cette carte est obtenue par segmentation de la couverture terrestre, observée à différentes dates tout au long de l’année. Même les sols nus et les bâtiments sont inclus dans la couverture terrestre. Vous pouvez supprimer les capteurs à un emplacement où ils ne sont pas nécessaires. Cette carte est fournie à titre indicatif, et vous pouvez modifier légèrement la position et le nombre des capteurs en fonction de vos connaissances personnelles. L’ajout de capteurs ne fera pas régresser les résultats de la carte thermique de l’humidité du sol, mais la précision de la carte risque de se détériorer si le nombre de capteurs est réduit.

## <a name="before-you-begin"></a>Avant de commencer

Respectez les prérequis suivants avant d’essayer de générer une carte d’emplacement des capteurs :

- La surface de l’exploitation agricole doit être supérieure à une acre.
- Le nombre de scènes Sentinel sans nuage doit être supérieur à six pour la plage de dates sélectionnée.
- Au moins six scènes Sentinel sans nuage doivent avoir un index de végétation par différence normalisée (NDVI, Normalized Difference Vegetation Index) supérieur ou égal à 0,3.

    > [!NOTE]
    > Sentinel n’autorise que deux threads simultanés par utilisateur. Les travaux sont donc mis en file d’attente, et leur exécution peut prendre plus de temps.

### <a name="dependencies-on-sentinel"></a>Dépendances à Sentinel

Les dépendances suivantes se rapportent à Sentinel :

- Nous dépendons des performances de Sentinel pour le téléchargement des images satellite. Vérifiez les [activités](https://scihub.copernicus.eu/twiki/do/view/SciHubNews/WebHome) de maintenance l’état des performances de Sentinel.
- Sentinel n’autorise que deux [threads de téléchargement](https://sentinels.copernicus.eu/web/sentinel/sentinel-data-access/typologies-and-services) simultanés par utilisateur.
- La génération de cartes de précision sera perturbée par [la fréquence de répétitivité et la couverture de Sentinel]( https://sentinel.esa.int/web/sentinel/user-guides/sentinel-2-msi/revisit-coverage).

## <a name="create-a-sensor-placement-map"></a>Créer une carte d’emplacement des capteurs
Cette section décrit en détail les procédures de création de cartes d’emplacement des capteurs.

> [!NOTE]
> Vous pouvez lancer la création d’une carte d’emplacement des capteurs à partir de la page **Cartes** ou du menu déroulant **Générer des cartes de précision** de la page **Détails de l’exploitation agricole**.

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Maps** dans le menu de navigation de gauche.
2. Sélectionnez **Créer des cartes**, puis, dans le menu déroulant, sélectionnez **Emplacement des capteurs**.

    ![Sélectionner Emplacement des capteurs](./media/get-sensor-data-from-sensor-partner/create-maps-drop-down-1.png)

3. Une fois que vous avez sélectionné **Emplacement des capteurs**, la fenêtre **Emplacement des capteurs** s’affiche.

    ![Fenêtre Emplacement des capteurs](./media/get-sensor-data-from-sensor-partner/sensor-placement-1.png)

4. Sélectionnez une ferme dans le menu déroulant **Farm**.
   Pour rechercher et sélectionner votre exploitation agricole, vous pouvez faire défiler la liste déroulante ou entrer le nom de l’exploitation agricole dans la zone de texte.
5. Pour générer une carte pour l’année passée, sélectionnez l’option **Recommandé**.
6. Pour générer une carte pour une plage de dates personnalisée, sélectionnez l’option **Sélectionner une plage de dates**. Entrez les dates de début et de fin pour lesquelles vous souhaitez générer la carte d’emplacement des capteurs.
7. Sélectionnez **Generate Maps** (Générer les cartes).
 Un message de confirmation contenant des informations détaillées sur le travail s’affiche.

  Pour plus d’informations sur l’état du travail, consultez la section **Voir les travaux**. Si le travail a l’état *Échec*, un message d’erreur détaillé s’affiche dans l’info-bulle de l’état *Échec*. Dans ce cas, répétez les étapes précédentes, puis réessayez.

  Si le problème persiste, consultez la section [Détecter un problème](troubleshoot-azure-farmbeats.md) ou contactez le [forum Azure FarmBeats](https://aka.ms/FarmBeatsMSDN) en fournissant les journaux appropriés pour obtenir de l’aide.

### <a name="view-and-download-a-sensor-placement-map"></a>Afficher et télécharger une carte d’emplacement des capteurs

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Maps** dans le menu de navigation de gauche.

    ![Sélectionner Cartes dans le menu de navigation de gauche](./media/get-sensor-data-from-sensor-partner/view-download-sensor-placement-map-1.png)

2. Sélectionnez **Filter** dans le menu de navigation de gauche de la fenêtre.
  La fenêtre **Filter** présente des critères de recherche.

    ![Fenêtre Filtrer](./media/get-sensor-data-from-sensor-partner/view-download-filter-1.png)

3. Dans les menus déroulants, sélectionnez les valeurs **Type**, **Date**et **Nom**. Sélectionnez ensuite **Appliquer** pour rechercher la carte que vous souhaitez afficher.
  La date de création du travail est indiquée au format type_nomferme_AAAA-MM-JJ.
4. Faites défiler la liste des cartes disponibles à l’aide des barres de navigation en bas de la page.
5. Sélectionnez la carte à afficher. Une fenêtre indépendante affiche l’aperçu de la carte sélectionnée.
6. Sélectionnez **Download** pour télécharger le fichier GeoJSON contenant les coordonnées des capteurs.

    ![Aperçu de la carte d’emplacement des capteurs](./media/get-sensor-data-from-sensor-partner/download-sensor-placement-map-1.png)

## <a name="satellite-indices-map"></a>Carte d’indices satellite

Les sections suivantes décrivent les procédures à suivre pour créer et afficher une carte d’indices satellite.

> [!NOTE]
> Vous pouvez lancer la création d’une carte d’indices satellite à partir de la page **Cartes** ou du menu déroulant **Générer des cartes de précision** de la page **Détails de l’exploitation agricole**.

FarmBeats vous offre la possibilité de générer des cartes NDVI, d’indice de végétation amélioré (EVI, Enhanced Vegetation Index) et d’indice de teneur en eau par différence normalisée (NDWI, Normalized Difference Water Index) pour les exploitations agricoles. Ces indices vous permettent d’évaluer la croissance actuelle et antérieure des cultures et les teneurs en eau représentatives du sol.


> [!NOTE]
> Une image Sentinel est requise pour les jours sur lesquels porte la génération de la carte.


## <a name="create-a-satellite-indices-map"></a>Créer une carte d’indices satellite

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Maps** dans le menu de navigation de gauche.
2. Sélectionnez **Créer des cartes**, puis, dans le menu déroulant, sélectionnez **Indices satellite**.

    ![Sélectionner Indices satellite dans le menu déroulant](./media/get-sensor-data-from-sensor-partner/create-maps-drop-down-satellite-indices-1.png)

3. Une fois que vous avez sélectionné **Indices satellite**, la fenêtre **Indices satellite** s’affiche.

    ![Fenêtre Indices satellite](./media/get-sensor-data-from-sensor-partner/satellitte-indices-1.png)

4. Sélectionnez une ferme dans le menu déroulant.
   Pour rechercher et sélectionner votre exploitation agricole, vous pouvez faire défiler la liste déroulante ou entrer le nom de l’exploitation.   
5. Pour générer une carte pour la semaine passée, sélectionnez l’option **Cette semaine**.
6. Pour générer une carte pour une plage de dates personnalisée, sélectionnez l’option **Sélectionner une plage de dates**. Entrez les dates de début et de fin pour lesquelles vous souhaitez générer la carte d’indices satellite.
7. Sélectionnez **Generate Maps** (Générer les cartes).
    Un message de confirmation contenant des informations détaillées sur le travail s’affiche.

    ![Message de confirmation de la carte d’indices satellite](./media/get-sensor-data-from-sensor-partner/successful-satellitte-indices-1.png)

    Pour plus d’informations sur l’état du travail, consultez la section **Voir les travaux**. Si le travail a l’état *Échec*, un message d’erreur détaillé s’affiche dans l’info-bulle de l’état *Échec*. Dans ce cas, répétez les étapes précédentes, puis réessayez.

    Si le problème persiste, consultez la section [Détecter un problème](troubleshoot-azure-farmbeats.md) ou contactez le [forum Azure FarmBeats](https://aka.ms/FarmBeatsMSDN) en fournissant les journaux appropriés pour obtenir de l’aide.

### <a name="view-and-download-a-map"></a>Afficher et télécharger une carte

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Maps** dans le menu de navigation de gauche.

    ![Sélectionner Cartes](./media/get-sensor-data-from-sensor-partner/view-download-sensor-placement-map-1.png)

2. Sélectionnez **Filter** dans le menu de navigation de gauche de la fenêtre. La fenêtre **Filter** présente des critères de recherche.

    ![La fenêtre Filtrer affiche les critères de recherche](./media/get-sensor-data-from-sensor-partner/view-download-filter-1.png)

3. Dans les menus déroulants, sélectionnez les valeurs **Type**, **Date**et **Nom**. Sélectionnez ensuite **Appliquer** pour rechercher la carte que vous souhaitez afficher.
  La date de création du travail est indiquée au format type_nomferme_AAAA-MM-JJ.

4. Faites défiler la liste des cartes disponibles à l’aide des barres de navigation en bas de la page.
5. Pour chaque combinaison de nom et de date (champs **Farm Name** et **Date**), les trois cartes suivantes sont disponibles :
    - NDVI
    - EVI
    - NDWI
6. Sélectionnez la carte à afficher. Une fenêtre indépendante affiche l’aperçu de la carte sélectionnée.
7. Pour sélectionner le format de téléchargement, sélectionnez **Télécharger** dans le menu déroulant. La carte est téléchargée et stockée dans le dossier local de votre ordinateur.

    ![Aperçu de la carte d’indices satellite sélectionnée](./media/get-sensor-data-from-sensor-partner/download-satellite-indices-map-1.png)

## <a name="soil-moisture-heatmap"></a>Carte thermique d’humidité du sol

L’humidité du sol fait référence à l’eau qui est retenue dans les espaces entre les particules de sol. La carte thermique d’humidité du sol vous permet de comprendre les données d’humidité du sol sur votre exploitation agricole à l’aide d’images haute résolution, indépendamment de la profondeur. Pour générer une carte thermique d’humidité du sol précise et exploitable, vous devez déployer des capteurs de façon uniforme. Tous les capteurs doivent provenir du même fournisseur. Les méthodes de mesure de l’humidité du sol et l’étalonnage diffèrent selon les fournisseurs. La carte thermique est générée pour une profondeur spécifique à l’aide des capteurs déployés à cette profondeur.

### <a name="before-you-begin"></a>Avant de commencer

Respectez les prérequis suivants avant d’essayer de générer une carte thermique d’humidité du sol :

- Au moins trois capteurs d’humidité du sol doivent être déployés. N’essayez pas de créer une carte thermique d’humidité du sol avant de déployer les capteurs et de les associer à l’exploitation agricole.
- La génération d’une carte thermique d’humidité du sol dépend de la couverture du chemin d’accès de Sentinel, de la couverture nuageuse et de l’ombre des nuages. Au moins une scène Sentinel sans nuage doit être disponible pour les 120 jours précédant le jour où la carte thermique d’humidité du sol a été demandée.
- Au moins la moitié des capteurs déployés sur l’exploitation agricole doivent être en ligne et avoir un flux de données vers le hub de données.
- Les capteurs fournissant les mesures pour la génération de la carte thermique doivent provenir du même fournisseur.


## <a name="create-a-soil-moisture-heatmap"></a>Créer une carte thermique d’humidité du sol

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Cartes** dans le menu de navigation de gauche pour afficher la page **Cartes**.
2. Sélectionnez **Créer des cartes**, puis, dans le menu déroulant, sélectionnez **Humidité du sol**.

    ![Sélectionner Humidité du sol dans le menu déroulant](./media/get-sensor-data-from-sensor-partner/create-maps-drop-down-soil-moisture-1.png)

3. Une fois que vous avez sélectionné **Humidité du sol**, la fenêtre **Humidité du sol** s’affiche.

    ![Fenêtre Humidité du sol](./media/get-sensor-data-from-sensor-partner/soil-moisture-1.png)

4. Sélectionnez une ferme dans le menu déroulant **Farm**.
   Pour rechercher et sélectionner votre exploitation agricole, vous pouvez faire défiler la liste déroulante ou entrer le nom de l’exploitation dans le menu déroulant **Sélectionner une exploitation agricole**.
5. Dans le menu déroulant **Sélectionner la mesure des capteurs d’humidité du sol**, sélectionnez la mesure (profondeur) des capteurs d’humidité du sol pour laquelle vous souhaitez générer la carte.
Pour trouver la mesure de capteur, accédez à **Capteurs** et sélectionnez un capteur d’humidité du sol. Ensuite, sous la section **Propriétés du capteur**, utilisez la valeur appropriée dans **Nom de la mesure**.
6. Pour générer une carte pour **Aujourd’hui** ou **Cette semaine**, sélectionnez l’une des options.
7. Pour générer une carte pour une plage de dates personnalisée, sélectionnez l’option **Sélectionner une plage de dates**. Entrez les dates de début et de fin pour lesquelles vous souhaitez générer la carte thermique d’humidité du sol.
8. Sélectionnez **Generate Maps** (Générer les cartes).
 Un message de confirmation contenant des informations détaillées sur le travail s’affiche.

   ![Message de confirmation de la carte d’humidité du sol](./media/get-sensor-data-from-sensor-partner/successful-soil-moisture-1.png)

    Pour plus d’informations sur l’état du travail, consultez la section **Voir les travaux**. Si le travail a l’état *Échec*, un message d’erreur détaillé s’affiche dans l’info-bulle de l’état *Échec*. Dans ce cas, répétez les étapes précédentes, puis réessayez.

    Si le problème persiste, consultez la section [Détecter un problème](troubleshoot-azure-farmbeats.md) ou contactez le [forum Azure FarmBeats](https://aka.ms/FarmBeatsMSDN) en fournissant les journaux appropriés pour obtenir de l’aide.

### <a name="view-and-download-a-map"></a>Afficher et télécharger une carte

Effectuez les opérations suivantes.

1. Dans la page d’accueil, sélectionnez **Maps** dans le menu de navigation de gauche.

    ![Accéder à Cartes](./media/get-sensor-data-from-sensor-partner/view-download-sensor-placement-map-1.png)

2. Sélectionnez **Filter** dans le menu de navigation de gauche de la fenêtre. La fenêtre **Filtrer** s’affiche. Vous pouvez y rechercher des cartes.

    ![Sélectionner Filtrer dans le volet de navigation gauche](./media/get-sensor-data-from-sensor-partner/view-download-filter-1.png)

3.  Dans les menus déroulants, sélectionnez les valeurs **Type**, **Date**et **Nom**. Sélectionnez ensuite **Appliquer** pour rechercher la carte que vous souhaitez afficher. La date de création du travail est indiquée au format type_nomferme_AAAA-MM-JJ.
4. Sélectionnez l’icône de **tri** à côté des en-têtes du tableau pour effectuer un tri selon les critères **Exploitation agricole**, **Date**, **Date de création**, **ID de travail** et **Type de travail**.
5. Faites défiler la liste des cartes disponibles à l’aide des boutons de navigation en bas de la page.
6. Sélectionnez la carte à afficher. Une fenêtre indépendante affiche l’aperçu de la carte sélectionnée.
7. Pour sélectionner le format de téléchargement, sélectionnez **Télécharger** dans le menu déroulant. La carte est téléchargée et stockée dans le dossier spécifié.

    ![Aperçu de la carte thermique d’humidité du sol](./media/get-sensor-data-from-sensor-partner/download-soil-moisture-map-1.png)

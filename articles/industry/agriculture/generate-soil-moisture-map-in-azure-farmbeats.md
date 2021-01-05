---
title: Générer une carte thermique d’humidité du sol
description: Décrit comment générer une carte thermique d’humidité du sol dans Azure FarmBeats
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: 2887e7c5fd875942c3d19391267ddbb9b73ccfec
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92165551"
---
# <a name="generate-soil-moisture-heatmap"></a>Générer une carte thermique d’humidité du sol

L’humidité du sol fait référence à l’eau maintenue dans les espaces entre les particules de sol. Une carte thermique d’humidité du sol vous permet de comprendre les données d’humidité de vos fermes à l’aide d’images haute résolution, quelle que soit la profondeur. Pour générer une carte thermique d’humidité du sol précise et exploitable, vous devez déployer des capteurs du même fournisseur de façon uniforme. Les méthodes de mesure de l’humidité du sol et l’étalonnage diffèrent selon les fournisseurs. La carte thermique est générée pour une profondeur spécifique à l’aide des capteurs déployés à cette profondeur.

Cet article décrit la procédure à suivre pour générer une carte thermique d’humidité du sol pour votre ferme avec Azure FarmBeats Accelerator. Dans cet article, vous allez apprendre à :

- [Créer des fermes](#create-a-farm)
- [Attribuer des capteurs aux fermes](#get-soil-moisture-sensor-data-from-partner)
- [Générer une carte thermique d’humidité du sol](#generate-soil-moisture-heatmap)

## <a name="before-you-begin"></a>Avant de commencer

Vérifiez les points suivants :  

- Un abonnement Azure.
- Une instance d’Azure FarmBeats est en cours d’exécution.
- Trois capteurs d’humidité du sol au minimum sont disponibles pour la ferme.

## <a name="create-a-farm"></a>Créer une ferme

Une ferme est une zone d’intérêt géographique pour laquelle vous souhaitez créer une carte thermique d’humidité du sol. Vous pouvez créer une ferme à l’aide de l’[API Farms](https://aka.ms/FarmBeatsDatahubSwagger) ou de l’[interface utilisateur FarmBeats Accelerator](manage-farms-in-azure-farmbeats.md#create-farms).

## <a name="deploy-sensors"></a>Déployer des capteurs

Vous devez déployer physiquement des capteurs d’humidité du sol dans la ferme. Vous pouvez acheter des capteurs d’humidité du sol auprès de nos partenaires agréés [Davis instruments](https://www.davisinstruments.com/product/enviromonitor-gateway/) et [Teralytic](https://teralytic.com/). Vous devez vous organiser avec votre fournisseur de capteurs pour effectuer la mise en place physique dans votre ferme.

## <a name="get-soil-moisture-sensor-data-from-partner"></a>Obtenir les données des capteurs d’humidité du sol auprès du partenaire

Quand les capteurs commencent à effectuer le streaming des données vers le tableau de bord de données du partenaire, vous devez activer la transmission des données vers Azure FarmBeats. Cette opération peut être effectuée à partir de l’application partenaire.

Par exemple, si vous avez acheté des capteurs Davis, vous vous connecterez à votre compte de liaison météo et fournirez les informations d’identification requises pour activer le streaming des données vers Azure FarmBeats. Pour obtenir les informations d’identification requises, suivez les instructions de l’article [Obtenir des données de capteur](get-sensor-data-from-sensor-partner.md#get-sensor-data-from-sensor-partners).

Une fois que vous avez entré vos informations d’identification et sélectionné **Submit** dans l’application partenaire, les données peuvent être transmises vers Azure FarmBeats.

### <a name="assign-soil-moisture-sensors-to-the-farm"></a>Attribuer des capteurs d’humidité du sol à la ferme

Après avoir lié votre compte de capteur à Azure FarmBeats, vous devez attribuer les capteurs d’humidité du sol à la ferme concernée.

1.  Dans la page d’accueil, sélectionnez **Farms** dans le menu. La page de liste **Farms** s’affiche.
2.  Sélectionnez **MyFarm** > **Add Devices** (Ajouter des appareils).
3.  La fenêtre **Add Devices** s’affiche. Sélectionnez un appareil lié aux capteurs d’humidité du sol pour votre ferme.

    ![Capture d’écran montrant l’écran Ajouter des appareils.](./media/get-sensor-data-from-sensor-partner/add-devices-1.png)

4. Sélectionnez **Add Devices** .     

## <a name="generate-soil-moisture-heatmap"></a>Générer une carte thermique d’humidité du sol

Cette étape consiste à créer un travail ou une opération durable qui générera une carte thermique d’humidité du sol pour votre ferme.

1.  Dans la page d’accueil, sélectionnez **Farms** dans le menu de navigation de gauche pour voir la page Farms.
2.  Sélectionnez **MyFarm** .
3.  Dans la page **Farm Details** , sélectionnez **Generate Precision Map** (Générer une carte de précision).
4.  Dans le menu déroulant, sélectionnez **Soil Moisture** (Humidité du sol).
5.  Dans la fenêtre **Soil Moisture** , sélectionnez **This Week** .
6.  Sous **Select Soil Moisture** **Sensor Measure** (Sélectionner la mesure des capteurs d’humidité du sol), entrez la mesure que vous souhaitez utiliser pour la carte.
    Pour trouver la mesure de capteur, sous **Sensors** (Capteurs), sélectionnez un capteur d’humidité du sol. Sous **Sensor Properties** (Propriétés de capteur), utilisez la valeur **Measure Name** (Nom de la mesure).

    ![Capture d’écran montrant l’écran Humidité du sol.](./media/get-sensor-data-from-sensor-partner/soil-moisture-1.png)


7.  Sélectionnez **Generate Maps** (Générer les cartes).
    Un message de confirmation contenant des informations détaillées sur le travail s’affiche. Pour plus d’informations, consultez l’état du travail sous Job Status dans la fenêtre Jobs.

    >[!NOTE]
    > L’exécution du travail prend environ trois à quatre heures.

### <a name="download-the-soil-moisture-heatmap"></a>Télécharger la carte thermique d’humidité du sol

Utiliser les étapes suivantes :

1. Dans la page **Jobs** , vérifiez l’état du travail créé dans le cadre de la dernière procédure sous **Job Status** .
2. Quand le travail présente l’état **Succeeded** (Réussi), cliquez sur **Maps** dans le menu.
3. Recherchez la carte selon le jour de création en utilisant le format <sol-moisture_MyFarm_AAAA-MM-JJ>.
4. Sélectionnez une carte dans la colonne **Name** . Une fenêtre indépendante s’affiche avec un aperçu de la carte sélectionnée.
5. Sélectionnez **Télécharger** . La carte est téléchargée et stockée dans le dossier local de votre ordinateur.

    ![Projet FarmBeats](./media/get-sensor-data-from-sensor-partner/download-soil-moisture-map-1.png)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez généré une carte thermique d’humidité du sol, découvrez comment [générer une carte d’emplacement des capteurs](generate-maps-in-azure-farmbeats.md#sensor-placement-map) et [ingérer des données de télémétrie historiques](ingest-historical-telemetry-data-in-azure-farmbeats.md). 

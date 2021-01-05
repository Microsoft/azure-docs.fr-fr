---
title: Tutoriel - Personnaliser le tableau de bord de l’opérateur dans Azure IoT Central
description: Ce tutoriel montre comment personnaliser le tableau de bord de l’opérateur dans une application IoT Central et gérer les appareils.
services: iot-central
ms.service: iot-central
ms.subservice: iot-central-retail
ms.topic: tutorial
ms.custom:
- iot-storeAnalytics-checkout
- iot-p0-scenario
ms.author: timlt
author: timlt
ms.date: 11/12/2019
ms.openlocfilehash: 5165f7d6e16e0d136dbf8d4507090c78372185cb
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97348048"
---
# <a name="tutorial--customize-the-operator-dashboard-and-manage-devices-in-azure-iot-central"></a>Tutoriel :  Personnaliser le tableau de bord de l’opérateur et gérer les appareils dans Azure IoT Central


Dans ce tutoriel, vous allez découvrir comment vous pouvez, en tant que réalisateur de logiciel, personnaliser le tableau de bord de l’opérateur dans votre application d’analytique en magasin Azure IoT Central. Les opérateurs d’application peuvent utiliser le tableau de bord personnalisé pour exécuter l’application et gérer les appareils connectés.

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> * Modifier le nom du tableau de bord
> * Personnaliser les vignettes d’image sur le tableau de bord
> * Réorganiser les vignettes pour modifier la disposition
> * Ajouter des vignettes de télémétrie pour afficher les conditions
> * Ajouter des vignettes de propriété pour afficher les détails de l’appareil
> * Ajouter des vignettes de commande pour exécuter des commandes

## <a name="prerequisites"></a>Prérequis

Le réalisateur de logiciel doit terminer le tutoriel pour créer l’application d’analytique en magasin Azure IoT Central et ajouter des appareils :

* [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md) (obligatoire)

## <a name="change-the-dashboard-name"></a>Modifier le nom du tableau de bord
Pour personnaliser le tableau de bord de l’opérateur, vous devez modifier le tableau de bord par défaut dans votre application. Vous pouvez aussi créer des tableaux de bord supplémentaires. La première étape pour personnaliser le tableau de bord dans votre application consiste à modifier le nom.

1. Accédez au site web du [gestionnaire d’applications d’Azure IoT Central](https://aka.ms/iotcentral).

1. Ouvrez l’application de surveillance des conditions que vous avez créée à l’aide du tutoriel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md).

1. Sélectionnez **Modifier** dans la barre d’outils du tableau de bord. En mode d’édition, vous pouvez personnaliser l’apparence, la disposition et le contenu du tableau de bord.

    ![Modification du tableau de bord Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/dashboard-edit.png)

1. Si vous le souhaitez, masquez le volet gauche. En masquant le volet gauche, vous disposez d’une plus grande zone de travail pour modifier le tableau de bord.

1. Entrez un nom convivial pour votre tableau de bord dans **Nom du tableau de bord**. Ce tutoriel utilise une société fictive nommée Contoso, et l’exemple de nom du tableau de bord est *Tableau de bord Contoso*. 

1. Sélectionnez **Enregistrer**. Vos modifications sont enregistrées dans le tableau de bord et le mode d’édition est désactivé.

    ![Modification du nom de tableau de bord Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/dashboard-change-name.png)

## <a name="customize-image-tiles-on-the-dashboard"></a>Personnaliser les vignettes d’image sur le tableau de bord
Un tableau de bord d’application Azure IoT Central se compose d’une ou de plusieurs vignettes. Une vignette est un conteneur rectangulaire permettant d’afficher du contenu sur un tableau de bord. Vous associez différents types de contenu à des vignettes que vous faites glisser, déplacez et redimensionnez pour personnaliser la disposition d’un tableau de bord. Il existe plusieurs types de vignettes pour afficher du contenu. Les vignettes d’image contiennent des images, et vous pouvez y ajouter une URL permettant aux utilisateurs de cliquer sur l’image. Les vignettes d’étiquette affichent du texte brut. Les vignettes Markdown contiennent du contenu formaté et vous permettent de définir une image, une URL, un titre et un code Markdown qui est rendu au format HTML. Les vignettes de télémétrie, de propriété ou de commande affichent des données spécifiques à l’appareil. 

Dans cette section, vous apprendrez à personnaliser les vignettes d’image sur le tableau de bord.

Pour personnaliser la vignette d’image qui affiche une image de marque sur le tableau de bord :

1. Sélectionnez **Modifier** dans la barre d’outils du tableau de bord. 

1. Sélectionnez **Configurer** sur la vignette d’image qui affiche l’image de marque Northwind. 

    ![Modification d’une image de marque Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/brand-image-edit.png)

1. Modifiez le **Titre**. Le titre apparaît lorsqu’un utilisateur survole l’image.

1. Sélectionnez **Image**. Une boîte de dialogue s’ouvre et vous permet de télécharger une image personnalisée. 

1. Si vous le souhaitez, spécifiez une URL pour l’image.

1. Sélectionnez **Mettre à jour la configuration**. Le bouton **Mettre à jour la configuration** enregistre les modifications apportées au tableau de bord et laisse le mode d’édition activé.

    ![Enregistrement d’une image de marque Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/brand-image-save.png)

1. Si vous le souhaitez, sélectionnez **Configurer** sur la vignette intitulée **Documentation**, puis spécifiez une URL pour le contenu du support. 

Pour personnaliser la vignette d’image qui affiche une carte des zones de capteurs dans le magasin :

1. Sélectionnez **Configurer** sur la vignette d’image qui affiche la carte des zones du magasin par défaut. 

1. Sélectionnez **Image** et utilisez la boîte de dialogue pour télécharger une image personnalisée d’une carte de zones du magasin. 

1. Sélectionnez **Mettre à jour la configuration**.

    ![Enregistrement d’une carte de magasin Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/store-map-save.png)

    La carte du magasin Contoso en exemple montre quatre zones : deux zones de caisse, une zone pour les vêtements et les soins personnels et une zone pour l’épicerie et le traiteur. Dans ce tutoriel, vous associerez des capteurs à ces zones pour fournir des données de télémétrie.

    ![Zones de magasin Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/store-zones.png)

1. Sélectionnez **Enregistrer**. 

## <a name="arrange-tiles-to-modify-the-layout"></a>Réorganiser les vignettes pour modifier la disposition
Une étape clé de la personnalisation d’un tableau de bord consiste à réorganiser les vignettes pour créer un affichage utile. Les opérateurs d’application utilisent le tableau de bord pour visualiser les données de télémétrie des appareils, gérer des appareils et surveiller les conditions d’un magasin. Azure IoT Central simplifie la tâche du générateur d’applications lors de la création d’un tableau de bord. Le mode d’édition du tableau de bord vous permet d’ajouter, de déplacer, de redimensionner et de supprimer rapidement des vignettes. Le modèle d’application **Analytique en magasin – caisse** simplifie également la création d’un tableau de bord. Il fournit une disposition de tableau de bord fonctionnelle, avec des capteurs connectés et des vignettes affichant les nombres de lignes de caisse et les conditions environnementales.

Dans cette section, vous réorganiserez le tableau de bord dans le modèle d’application **Analytique en magasin – caisse** pour créer une disposition personnalisée.

Pour supprimer les vignettes que vous n’envisagez pas d’utiliser dans votre application :

1. Sélectionnez **Modifier** dans la barre d’outils du tableau de bord. 

1. Sélectionnez **X Supprimer** pour supprimer les vignettes suivantes : **Retour à toutes les zones**, **Accéder au tableau de bord du magasin**, **Temps d’attente** et les trois vignettes associées à **Caisse 3**. Le tableau de bord du magasin Contoso n’utilise pas ces vignettes. 

    ![Suppression de vignettes Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/delete-tiles.png)

1. Faites défiler la page pour afficher les vignettes restantes du tableau de bord.

1. Sélectionnez **X Supprimer** pour supprimer les vignettes suivantes : **Chauffer la zone de caisse**, **Refroidir la zone de de caisse**, **Paramètres des capteurs de présence**, **Paramètres des capteurs du thermostat** et **Conditions environnementales**. 

   ![Suppression des vignettes restantes Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/delete-tiles-2.png)

1. Sélectionnez **Enregistrer**. La suppression des vignettes inutilisées libère de l’espace dans la page d’édition et simplifie l’affichage du tableau de bord pour les opérateurs.

1. Affichez les modifications que vous avez apportées au tableau de bord.

   ![Azure IoT Central après la suppression des vignettes](./media/tutorial-in-store-analytics-customize-dashboard/after-delete-tiles.png)

Après avoir supprimé les vignettes inutilisées, réorganisez les vignettes restantes pour créer une disposition organisée. La nouvelle disposition comprend de l’espace pour les vignettes que vous ajouterez ultérieurement.

Pour réorganiser les vignettes restantes :

1. Sélectionnez **Modifier**.

1. Sélectionnez la vignette **Microprogramme de présence** et faites-la glisser à droite de la vignette de batterie **Présence**.

1. Sélectionnez la vignette **Microprogramme du thermostat** et faites-la glisser à droite de la vignette de batterie **Thermostat**.

1. Sélectionnez **Enregistrer**.

1. Affichez les modifications apportées à votre disposition. 

    ![Vignettes de batterie des microprogrammes Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/firmware-battery-tiles.png)

## <a name="add-telemetry-tiles-to-display-conditions"></a>Ajouter des vignettes de télémétrie pour afficher les conditions
Après avoir personnalisé la disposition du tableau de bord, vous êtes prêt à ajouter des vignettes pour afficher les données de télémétrie. Pour créer une vignette de télémétrie, sélectionnez un modèle d’appareil et une instance d’appareil, puis sélectionnez les données de télémétrie spécifiques à l’appareil devant être affichées dans la vignette. Le modèle d’application **Analytique en magasin – caisse** comprend plusieurs vignettes de télémétrie dans le tableau de bord. Les quatre vignettes dans les deux zones de caisse affichent les données de télémétrie à partir d’une simulation du capteur de présence. La vignette **Fréquentation** affiche les nombres dans les deux zones de caisse. 

Dans cette section, vous ajouterez deux vignettes de télémétrie supplémentaires pour afficher les données de télémétrie de l’environnement provenant des capteurs RuuviTag que vous avez ajoutés à l’aide du tutoriel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md). 

Pour ajouter des vignettes afin d’afficher les données environnementales des capteurs RuuviTag :

1. Sélectionnez **Modifier**.

1. Sélectionnez `RuuviTag` dans la liste **Modèle d’appareil**. 

1. Sélectionnez l’**Instance d’appareil** de l’un des deux capteurs RuuviTag. Dans l’exemple du magasin Contoso, sélectionnez `Zone 1 Ruuvi` pour créer une vignette de télémétrie pour la zone 1. 

1. Sélectionnez `Relative humidity` et `temperature` dans la liste **Données de télémétrie**. Ce sont les éléments de télémétrie qui s’affichent pour chaque zone sur la vignette.

1. Sélectionnez **Combiner**. 

    ![Ajout de la vignette RuuviTag 1 Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/add-zone1-ruuvi.png)

    Une nouvelle vignette apparaît pour afficher les données de télémétrie combinées de l’humidité et de la température pour le capteur sélectionné. 

1. Sélectionnez **Configurer** sur la nouvelle vignette pour le capteur RuuviTag. 

1. Modifiez le **Titre** en *Environnement zone 1*. 

1. Sélectionnez **Mettre à jour la configuration**.

1. Répétez les étapes précédentes afin de créer une vignette pour la deuxième instance de capteur. Modifiez le **Titre** en *Environnement zone 2* puis sélectionnez **Mettre à jour la configuration**.

1. Faites glisser la vignette intitulée **Environnement zone 2** sous la vignette **Microprogramme du thermostat**. 

1. Faites glisser la vignette intitulée **Environnement zone 1** sous la vignette **Fréquentation**. 

1. Sélectionnez **Enregistrer**. Le tableau de bord affiche les données de télémétrie des zones dans les deux nouvelles vignettes.

    ![Toutes les vignettes RuuviTag Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/all-ruuvitag-tiles.png)

Pour modifier la vignette **Fréquentation** afin d’afficher les données de télémétrie pour deux zones de caisse uniquement :

1. Sélectionnez **Modifier**. 

1. Sélectionnez **Configurer** sur la vignette **Fréquentation**.

1. Dans **Données de télémétrie**, sélectionnez **nombre 1**, **nombre 2** et **nombre 3**. 

1. Sélectionnez **Mettre à jour la configuration**. Cette opération efface la configuration existante de la vignette. 

1. Sélectionnez à nouveau **Configurer** sur la vignette **Fréquentation**.

1. Dans **Données de télémétrie**, sélectionnez **nombre 1** et **nombre 2**. 

1. Sélectionnez **Mettre à jour la configuration**. 

1. Sélectionnez **Enregistrer**.  Le tableau de bord mis à jour affiche les nombres pour vos deux zones de caisse uniquement, lesquels sont basés sur la simulation du capteur de présence.

    ![Deux couloirs de fréquentation Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/people-traffic-two-lanes.png)

## <a name="add-property-tiles-to-display-device-details"></a>Ajouter des vignettes de propriété pour afficher les détails de l’appareil
Les opérateurs d’application utilisent le tableau de bord pour gérer les appareils et surveiller l’état. Ajoutez une vignette pour chaque RuuviTag afin de permettre aux opérateurs d’afficher la version du logiciel. 

Pour ajouter une vignette de propriété pour chaque RuuviTag :

1. Sélectionnez **Modifier**.

1. Sélectionnez `RuuviTag` dans la liste **Modèle d’appareil**. 

1. Sélectionnez l’**Instance d’appareil** de l’un des deux capteurs RuuviTag. Dans l’exemple du magasin Contoso, sélectionnez `Zone 1 Ruuvi` pour créer une vignette de télémétrie pour la zone 1. 

1. Sélectionnez **Propriétés > Version du logiciel**.

1. Sélectionnez **Combiner**. 

1. Sélectionnez **Configurer** sur la nouvelle vignette intitulée **Version du logiciel**. 

1. Modifiez le **Titre** en *Version du logiciel de Ruuvi 1*.

1. Sélectionnez **Mettre à jour la configuration**. 

1. Faites glisser la vignette intitulée **Version du logiciel de Ruuv 1** sous la vignette **Environnement zone 1**.

1. Répétez les étapes précédentes afin de créer une vignette de propriété pour la version du logiciel du deuxième capteur RuuviTag. 

1. Sélectionnez **Enregistrer**.  

    ![Vignettes des propriétés RuuviTag Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/add-ruuvi-property-tiles.png)

## <a name="add-command-tiles-to-run-commands"></a>Ajouter des vignettes de commande pour exécuter des commandes
Les opérateurs d’application utilisent également le tableau de bord pour gérer les appareils en exécutant des commandes. Vous pouvez ajouter des vignettes de commande au tableau de bord qui exécuteront des commandes prédéfinies sur un appareil. Dans cette section, vous ajouterez une vignette de commande pour permettre aux opérateurs de redémarrer la passerelle Rigado. 

Pour ajouter une vignette de commande afin de redémarrer la passerelle :

1. Sélectionnez **Modifier**. 

1. Sélectionnez `C500` dans la liste **Modèle d’appareil**. Il s’agit du modèle pour la passerelle Rigado C500. 

1. Sélectionnez l’instance de passerelle dans **Instance d’appareil**.

1. Sélectionnez **Commande > Redémarrer** et faites glisser dans le tableau de bord à côté de la carte du magasin. 

1. Sélectionnez **Enregistrer**. 

1. Affichez votre tableau de bord Contoso terminé. 

    ![Personnalisation complète du tableau de bord Azure IoT Central](./media/tutorial-in-store-analytics-customize-dashboard/completed-dashboard.png)

1. Si vous le souhaitez, sélectionnez la vignette **Redémarrer** pour exécuter la commande de redémarrage sur votre passerelle.

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à :

* Modifier le nom du tableau de bord
* Personnaliser les vignettes d’image sur le tableau de bord
* Réorganiser les vignettes pour modifier la disposition
* Ajouter des vignettes de télémétrie pour afficher les conditions
* Ajouter des vignettes de propriété pour afficher les détails de l’appareil
* Ajouter des vignettes de commande pour exécuter des commandes

Maintenant que vous avez personnalisé le tableau de bord dans votre application d’analytique en magasin Azure IoT Central, nous vous suggérons de réaliser l’étape suivante :

> [!div class="nextstepaction"]
> [Exporter des données et visualiser des aperçus](./tutorial-in-store-analytics-export-data-visualize-insights.md)

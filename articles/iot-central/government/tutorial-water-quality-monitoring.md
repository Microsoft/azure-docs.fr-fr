---
title: 'Tutoriel : Créer une application de suivi de la qualité de l’eau avec Azure IoT Central'
description: 'Tutoriel : Découvrez comment créer une application de suivi de la qualité de l’eau à l’aide de modèles d’application Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: 6e8f9b8b33d391c4e13c6076a8589d336455d8c0
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97590220"
---
# <a name="tutorial-create-a-water-quality-monitoring-application-in-azure-iot-central"></a>Tutoriel : Créer une application de suivi de la qualité de l’eau dans Azure IoT Central



Ce tutoriel vous guide tout au long de la création d’une application de suivi de la qualité de l’eau dans Azure IoT Central. Vous créez l’application à partir du modèle d’application Azure IoT Central **Water quality monitoring** (Suivi de la qualité de l’eau).

Ce didacticiel vous apprend à effectuer les opérations suivantes :

> [!div class="checklist"]
> * Utiliser le modèle **Water quality monitoring** (Suivi de la qualité de l’eau) pour créer une application de suivi de la qualité de l’eau.
> * Explorer et personnaliser le tableau de bord de l’opérateur.
> * Explorer un modèle d’appareil de suivi de la qualité de l’eau.
> * Explorer les appareils simulés.
> * Explorer et configurer les règles.
> * Configurer des travaux.
> * Personnaliser le marquage de l’application à l’aide d’une étiquette blanche.

## <a name="prerequisites"></a>Prérequis

Nous vous recommandons d’avoir un abonnement Azure pour suivre ce tutoriel. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un sur la [page d’inscription à Azure](https://aka.ms/createazuresubscription).

## <a name="create-a-water-quality-monitoring-application-in-azure-iot-central"></a>Créer une application de suivi de la qualité de l’eau dans Azure IoT Central

Dans cette section, vous utilisez le modèle Azure IoT Central **Water quality monitoring** (Suivi de la qualité de l’eau) pour créer une application de suivi de la qualité de l’eau.

1. Accédez à la [page d’accueil d’Azure IoT Central](https://aka.ms/iotcentral).

    Si vous disposez d’un abonnement Azure, connectez-vous avec les informations d’identification que vous utilisez pour y accéder. Sinon, connectez-vous à l’aide d’un compte Microsoft :

    ![Connexion au compte de votre organisation](./media/tutorial-waterqualitymonitoring/sign-in.png)

1. Sélectionnez **Build** (Créer) dans le volet le plus à gauche d’Azure IoT Central et sélectionnez l’onglet **Government** (Service public). Le volet relatif au secteur public affiche plusieurs modèles d’application pour le secteur public.

    ![Modèles d’application pour le secteur public](./media/tutorial-waterqualitymonitoring/iotcentral-government-tab-overview1.png)

1. Sélectionnez le modèle d’application **Water quality monitoring** (Suivi de la qualité de l’eau). Ce modèle d’application comprend un modèle d’appareil pour la qualité de l’eau, des appareils simulés, un tableau de bord d’opérateur et des règles de supervision préconfigurées.

1. Sélectionnez **Créer une application**. Le volet **New application** (Nouvelle application) s’ouvre et affiche les éléments suivants :

    * **Nom de l'application** : Par défaut, le nom de l’application est **Water quality monitoring** suivi d’une chaîne d’ID unique générée par Azure IoT Central. Si vous le souhaitez, vous pouvez entrer un nom d’affichage ou modifier ultérieurement le nom de l’application.
    * **URL** : Vous pouvez entrer l’URL de votre choix ou modifier ultérieurement la valeur de l’URL.
    * Si vous disposez d’un abonnement Azure, entrez vos valeurs de **Répertoire**, **Abonnement Azure** et **Région**. Si vous n’avez pas d’abonnement, vous pouvez activer **7-day free trial** (Version d’évaluation gratuite de 7 jours) et renseigner les coordonnées requises.

    Pour plus d’informations sur les répertoires et les abonnements, consultez le guide de démarrage rapide [Créer une application](../core/quick-deploy-iot-central.md?toc=/azure/iot-central-pnp/toc.json&bc=/azure/iot-central-pnp/breadcrumb/toc.json).

1. Sélectionnez le bouton **Create** (Créer) en bas à gauche de la page.

    ![Page de création d’une application Azure IoT Central](./media/tutorial-waterqualitymonitoring/new-application-waterqualitymonitoring1.png)

    ![Informations de facturation d’une nouvelle application Azure IoT Central](./media/tutorial-waterqualitymonitoring/new-application-waterqualitymonitoring1-billinginfo.png)

Vous avez désormais créé une application de suivi de la qualité de l’eau à l’aide du modèle Azure IoT Central **Water quality monitoring** (Suivi de la qualité de l’eau).

Votre nouvelle application est accompagnée des composants préconfigurés suivants :

* Tableaux de bord d’opérateur
* Modèles d’appareil de suivi de la qualité de l’eau
* Appareils simulés de suivi de la qualité de l’eau
* Règles et travaux
* Marquage à l’aide d’une étiquette blanche

Vous pouvez modifier votre application à tout moment.

Nous allons maintenant explorer l’application et effectuer certaines personnalisations.

## <a name="explore-and-customize-the-operator-dashboard"></a>Explorer et personnaliser le tableau de bord de l’opérateur

Une fois l’application créée, le volet **Wide World water quality dashboard** (Tableau de bord de qualité de l’eau dans le monde) s’ouvre.

   ![Tableau de bord de suivi de la qualité de l’eau](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-dashboard1.png)

En tant que créateur, vous pouvez créer et personnaliser des vues sur le tableau de bord, que les opérateurs pourront utiliser. Avant d’essayer de le personnaliser, commençons par explorer le tableau de bord.

Toutes les données affichées dans le tableau de bord sont basées sur des données d’appareils simulés, qui sont décrites dans la section suivante.

Le tableau de bord inclut les types de vignettes suivants :

* **Vignette d’image Wide World water utility** (Utilitaire Wide World Water) : la première vignette en haut à gauche du tableau de bord est une vignette d’image représentant l’utilitaire fictif « Wide World Water ». Vous pouvez personnaliser cette vignette et fournir votre propre image, ou la supprimer.

* **Vignettes d’indicateur de performance clé de pH moyen** : des vignettes de KPI comme **pH moyen au cours des 30 dernières minutes** figurent en haut du volet de tableau de bord. Vous pouvez personnaliser les vignettes de KPI et définir chacune sur un type et une plage de temps différents.

* **Water monitoring area map** (Carte des zones de suivi de l’eau) : Azure IoT Central utilise Azure Maps, que vous pouvez configurer directement dans votre application pour afficher l’emplacement des appareils. Vous pouvez également mapper les informations d’emplacement de votre application sur votre appareil, puis utiliser Azure Maps pour afficher les informations sur une carte. Survolez la carte et essayez les contrôles.

* **Carte thermique de distribution moyenne du pH** : Vous pouvez sélectionner différents graphiques de visualisation pour afficher les données de télémétrie des appareils de la manière la plus appropriée à votre application.

* **Graphique en courbes des indicateurs de qualité critique** : vous pouvez visualiser les données de télémétrie des appareils sous la forme d’un graphique en courbes sur une plage de temps.  

* **Graphique à barres de concentration des agents chimiques** : vous pouvez visualiser les données de télémétrie des appareils dans un graphique à barres.

* **Bouton d’action** : le tableau de bord comprend une vignette pour les actions qu’un opérateur peut lancer directement à partir du tableau de bord de suivi. La réinitialisation des propriétés d’un appareil est un exemple des actions possibles.

* **Vignettes de liste de propriétés** : le tableau de bord comporte plusieurs vignettes de propriétés qui représentent les informations de seuil, les informations d’intégrité d’appareil et les informations de maintenance.

### <a name="customize-the-dashboard"></a>Personnaliser le tableau de bord

En tant que créateur, vous pouvez personnaliser les vues figurant sur le tableau de bord, que les opérateurs pourront utiliser.

1. Sélectionnez **Edit** (Modifier) pour personnaliser le volet du **Wide World water quality dashboard** (Tableau de bord de qualité de l’eau dans le monde). Vous pouvez personnaliser ce tableau de bord en sélectionnant des commandes dans le menu **Edit** (Modifier). Une fois le tableau de bord en mode d’édition, vous pouvez ajouter de nouvelles vignettes ou configurer les vignettes existantes.

    ![Modification de votre tableau de bord](./media/tutorial-waterqualitymonitoring/edit-dashboard.png)

1. Sélectionnez **+ New** (Nouveau) pour créer un tableau de bord et le configurer. Vous pouvez avoir plusieurs tableaux de bord et naviguer entre eux via le menu Dashboard (Tableau de bord).

## <a name="explore-a-water-quality-monitoring-device-template"></a>Explorer un modèle d’appareil de suivi de la qualité de l’eau

Un modèle d’appareil dans Azure IoT Central définit les fonctionnalités d’un appareil. Les fonctionnalités disponibles sont la télémétrie, les propriétés et les commandes. En tant que créateur, vous pouvez définir des modèles d’appareil dans Azure IoT Central qui représenteront les fonctionnalités des appareils connectés. Vous pouvez également créer des appareils simulés pour tester le modèle et l’application de vos appareils.

L’application de suivi de la qualité de l’eau que vous avez créée est fournie avec un modèle d’appareil de suivi de la qualité de l’eau.

Pour afficher ce modèle d’appareil :

1. Sélectionnez **Device templates** (Modèles d’appareil) dans le volet le plus à gauche de votre application dans Azure IoT Central.
1. Dans la liste des modèles d’appareil, sélectionnez **Water Quality Monitor** (Moniteur de qualité de l’eau). Ce modèle d’appareil s’ouvre.

    ![Modèle d’appareil](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devicetemplate.png)

### <a name="customize-the-device-template"></a>Personnaliser le modèle d’appareil

Exercez-vous à personnaliser les paramètres de modèle d’appareil suivants :

1. Dans le menu du modèle d’appareil, sélectionnez **Customize** (Personnaliser).
1. Accédez au type de télémétrie **Temperature** (Température).
1. Remplacez le nom d’affichage (**Display name**) par **Température signalée**.
1. Changez l’unité de mesure ou définissez **Min value** (Valeur minimale) et **Max value** (Valeur maximale).
1. Sélectionnez **Enregistrer**.

#### <a name="add-a-cloud-property"></a>Ajouter une propriété cloud

1. Dans le menu du modèle d’appareil, sélectionnez **Cloud properties** (Propriétés cloud).
1. Pour ajouter une nouvelle propriété cloud, sélectionnez **+ Add Cloud Property** (Ajouter une propriété cloud). Dans Azure IoT Central, vous pouvez ajouter une propriété qui s’applique à un appareil, mais qui n’est pas censée être envoyée par l’appareil. Un seuil d’alerte spécifique à la zone d’installation, aux informations sur les ressources ou aux informations de maintenance est un exemple d’une telle propriété.
1. Sélectionnez **Enregistrer**.

### <a name="explore-views"></a>Explorer les vues

Le modèle d’appareil de suivi de la qualité de l’eau est fourni avec des vues prédéfinies. Les vues définissent la façon dont les opérateurs verront les données de l’appareil et définiront les propriétés cloud. Explorez les vues et exercez-vous à apporter des modifications.

  ![Vues du modèle d’appareil](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devicetemplate-views.png)

### <a name="publish-the-device-template"></a>Publier le modèle d’appareil

Si vous avez apporté des modifications, veillez à sélectionner **Publish** (Publier) pour publier le modèle d’appareil.

### <a name="create-a-new-device-template"></a>Créer un modèle d’appareil

1. Sélectionnez **+ New** (Nouveau) pour créer un nouveau modèle d’appareil et suivre le processus de création.
1. Créez un modèle d’appareil personnalisé ou choisissez un modèle d’appareil dans le catalogue d’appareils Azure IoT.

## <a name="explore-simulated-devices"></a>Explorer les appareils simulés

L’application de suivi de la qualité de l’eau que vous avez créée à partir du modèle d’application a deux appareils simulés. Ces appareils sont mappés au modèle d’appareil de suivi de la qualité de l’eau.

### <a name="view-the-devices"></a>Afficher les appareils

1. Sélectionnez **Devices** (Appareils) dans le volet le plus à gauche de votre application.

   ![Appareils](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-devices.png)

1. Sélectionnez un appareil simulé.

    ![Sélection de l’appareil 1](./media/tutorial-waterqualitymonitoring/waterqualitymonitor-device1.png)

1. Sous l’onglet **Cloud Properties** (Propriétés cloud), modifiez la valeur **Acidity (pH) threshold** (Seuil d’acidité (pH)) en remplaçant **8** par **9**.
1. Explorez l’onglet **Device Properties** (Propriétés de l’appareil) et l’onglet **Device Dashboard** (Tableau de bord de l’appareil).

> [!NOTE]
> Tous les onglets ont été configurés à partir des **vues du modèle d’appareil**.

### <a name="add-new-devices"></a>Ajouter de nouveaux appareils

Sous l’onglet **Devices** (Appareils), sélectionnez **+ New** (Nouveau) pour ajouter un nouvel appareil.

## <a name="explore-and-configure-rules"></a>Explorer et configurer les règles

Dans Azure IoT Central, vous pouvez créer des règles pour superviser automatiquement les données de télémétrie des appareils. Ces règles déclenchent une action lorsque l’une de leurs conditions est remplie. L’une des actions possibles consiste à envoyer des e-mails de notification. D’autres actions possibles incluent une action Power Automate ou webhook pour envoyer des données à d’autres services.

L’application de suivi de la qualité de l’eau que vous avez créée comporte deux règles préconfigurées.

### <a name="view-rules"></a>Afficher les règles

1. Sélectionnez **Rules** (Règles) dans le volet le plus à gauche de votre application.

   ![Règles](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-rules.png)

1. Sélectionnez **High pH alert** (Alerte de pH élevé), qui est l’une des règles préconfigurées dans l’application.

   ![Règle d’alerte de pH élevé](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-highphalert.png)

   La règle **High pH alert** (Alerte de pH élevé) est configurée pour vérifier si la condition d’acidité (pH) est supérieure à 8.

Ensuite, ajoutez une action de messagerie à la règle :

1. Sélectionnez **+ E-mail**.
1. Dans la zone **Display name** (Nom d’affichage), entrez **Alerte de pH élevé**.
1. Dans la zone **To** (À), entrez l’adresse e-mail associée à votre compte Azure IoT Central.
1. Si vous le souhaitez, entrez une note à inclure dans le texte de l’e-mail.
1. Sélectionnez **Terminé** pour terminer l’action.
1. Sélectionnez **Enregistrer** pour enregistrer et activer la nouvelle règle.

Après quelques minutes, vous devrez recevoir un e-mail quand la condition configurée sera remplie.

> [!NOTE]
> L’application enverra un e-mail chaque fois qu’une condition sera remplie. Sélectionnez **Disable** (Désactiver) pour cesser de recevoir un e-mail automatisé à partir de cette règle.
  
Pour créer une nouvelle règle, sélectionnez **Rules** (Règles) dans le volet le plus à gauche de votre application, puis sélectionnez **+New** (Nouveau).

## <a name="configure-jobs"></a>Configuration des travaux

Les travaux Azure IoT Central vous permettent de déclencher des mises à jour des propriétés d’appareil ou cloud sur plusieurs appareils. Vous pouvez également utiliser des travaux pour déclencher des commandes d’appareil sur plusieurs appareils. Azure IoT Central automatise le flux de travail pour vous.

1. Sélectionnez **Jobs** (Travaux) dans le volet le plus à gauche de votre application.
1. Sélectionnez **+New** (Nouveau) et configurez un ou plusieurs travaux.

## <a name="customize-your-application"></a>Personnaliser votre application

En tant que réalisateur de logiciel, vous pouvez modifier plusieurs paramètres pour personnaliser l’expérience utilisateur dans votre application.

1. Sélectionnez **Administration** > **Personnaliser votre application**.
1. Sous **Logo de l’application**, sélectionnez **Modifier** pour choisir l’image à charger comme logo.
1. Sous **Icône du navigateur**, sélectionnez **Modifier** pour choisir l’image qui apparaîtra dans les onglets du navigateur.
1. Sous **Couleurs du navigateur**, vous pouvez remplacer les valeurs par défaut par des codes couleur hexadécimaux HTML.
1. Sélectionnez **Paramètres** pour modifier la valeur de **Thème**.

   ![Personnaliser votre application](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-customize-your-application1.png)

### <a name="update-the-application-image"></a>Mettre à jour l’image de l’application

1. Sélectionnez **Administration** > **Paramètres d’application**.

1. Utilisez le bouton **Sélectionner une image** pour choisir une image à télécharger en tant qu’image de l’application.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne pensez pas continuer à utiliser cette application, supprimez l’application en procédant comme suit :

1. Ouvrez l’onglet **Administration** dans le volet le plus à gauche de votre application.
1. Sélectionnez **Paramètres d’application**, puis sélectionnez le bouton **Supprimer**.

    ![Suppression de votre application](./media/tutorial-waterqualitymonitoring/waterqualitymonitoring-application-settings-delete-app1.png)

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur 

> [!div class="nextstepaction"]
> [concepts du suivi de la qualité de l’eau](./concepts-waterqualitymonitoring-architecture.md).

---
title: 'Tutoriel : Créer une application de suivi de la consommation d’eau avec Azure IoT Central'
description: 'Tutoriel : Découvrez comment créer une application de suivi de la consommation d’eau à l’aide de modèles d’application Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: 476a0c2d75dcbee5c1ed4d758e0cbc9d4726ff95
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587194"
---
# <a name="tutorial-create-a-water-consumption-monitoring-application-with-azure-iot-central"></a>Tutoriel : Créer une application de suivi de la consommation d’eau avec Azure IoT Central

Ce tutoriel vous montre comment créer une application de suivi de la consommation d’eau Azure IoT Central à partir du modèle d’application Azure IoT Central correspondant.

Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
> * Utiliser le modèle Azure IoT Central de suivi de la consommation d’eau pour créer votre application de suivi de la consommation d’eau
> * Explorer et personnaliser le tableau de bord de l’opérateur
> * Explorer les modèles d’appareils
> * Explorer les appareils simulés.
> * Explorer et configurer les règles.
> * Configurer des travaux.
> * Personnaliser la marque de votre application à l’aide d’une étiquette blanche

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un sur la [page d’inscription à Azure](https://aka.ms/createazuresubscription).

## <a name="create-a-water-consumption-monitoring-app-with-azure-iot-central"></a>Créer une application de suivi de la consommation d’eau avec Azure IoT Central

Dans cette section, vous allez utiliser le modèle Azure IoT Central de suivi de la consommation d’eau pour créer votre application de suivi de la consommation d’eau dans Azure IoT Central.

Pour créer une application Azure IoT Central de suivi de la consommation d’eau :

1. Accédez à la [page d’accueil d’Azure IoT Central](https://aka.ms/iotcentral).

    Si vous disposez d’un abonnement Azure, connectez-vous avec les informations d’identification que vous utilisez pour y accéder. Sinon, connectez-vous à l’aide d’un compte Microsoft.

    ![Entrer votre compte d’entreprise](media/tutorial-waterconsumptionmonitoring/sign-in.png)

1. Sélectionnez **Build** (Créer) dans le volet gauche, puis sélectionnez l’onglet **Government** (Service public). La page **Government** (Secteur public) affiche plusieurs modèles d’application pour le secteur public.

   ![Créer des modèles d’application pour le secteur public](./media/tutorial-waterconsumptionmonitoring/iotcentral-government-tab-overview1.png)

1. Sélectionnez le modèle d’application **Water consumption monitoring** (Suivi de la consommation d’eau).
Ce modèle comprend un exemple de modèle d’appareil pour la consommation d’eau, un appareil simulé, un tableau de bord d’opérateur et des règles de supervision préconfigurées.

1. Sélectionnez **Create app** (Créer une application) pour ouvrir le formulaire de création **New application** (Nouvelle application) avec les champs suivants :
    * **Nom de l'application** : par défaut, le nom de l’application est composé de *Water consumption monitoring* et d’une chaîne d’ID unique générée par Azure IoT Central. Si vous le souhaitez, choisissez un nom d’application convivial. Vous pouvez également modifier le nom de l’application.
    * **URL** : Azure IoT Central génère automatiquement une URL basée sur le nom de l’application. Vous pouvez choisir de mettre à jour l’URL à votre convenance. Vous pouvez également modifier l’URL.
    * Si vous disposez d’un abonnement Azure, entrez les informations concernant l’**annuaire**, l’**abonnement Azure** et la **zone géographique**. Si vous n’avez pas d’abonnement, vous pouvez sélectionner l’option **Essai gratuit de 7 jours** et fournir vos coordonnées.

    Pour plus d’informations sur les annuaires et les abonnements, consultez le guide de démarrage rapide [Créer une application](../core/quick-deploy-iot-central.md).

1. Au bas de la page, sélectionnez **Créer**.

    ![Page Nouvelle application dans Azure IoT Central](./media/tutorial-waterconsumptionmonitoring/new-application-waterconsumptionmonitoring.png)

    ![Page d’informations de facturation Azure IoT Central](./media/tutorial-waterconsumptionmonitoring/new-application-waterconsumptionmonitoring-billinginfo.png)

Vous venez de créer une application de suivi de la consommation d’eau à l’aide du modèle Azure IoT Central de suivi de la consommation d’eau.

L’application de suivi de la consommation d’eau est préconfigurée avec les éléments suivants :

* Exemples de tableaux de bord d’opérateur
* Exemples de modèles prédéfinis de débitmètre et de vanne
* Débitmètre et dispositif de vanne intelligente simulés
* Règles et travaux
* Exemple de personnalisation à l’aide d’une étiquette blanche

Cette application est la vôtre et vous pouvez la modifier à tout moment. Nous allons maintenant explorer l’application et effectuer certaines personnalisations.

## <a name="explore-and-customize-the-operator-dashboard"></a>Explorer et personnaliser le tableau de bord de l’opérateur

Une fois l’application créée, l’exemple **Wide World water consumption dashboard** s’ouvre.

   ![Tableau de bord de suivi de la consommation d’eau](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-dashboardfull.png)

En tant que créateur, vous pouvez créer et personnaliser des vues dans le tableau de bord pour les opérateurs. Avant d’essayer de le personnaliser, commençons par explorer le tableau de bord.

> [!NOTE]
> Toutes les données affichées dans le tableau de bord sont basées sur des données d’appareils simulés, que nous explorerons dans la section suivante.
  
Le tableau de bord est constitué de différents types de vignettes :

* **Vignette d’image Wide World water utility** (Utilitaire Wide World Water) : la première vignette du tableau de bord est une vignette d’image d’un utilitaire fictif Wide World Water. Vous pouvez personnaliser cette vignette en y insérant votre propre image ou en la supprimant.
* **Vignette des KPI de débit moyen d’eau** : la vignette des KPI est configurée pour afficher comme exemple le *débit moyen des 30 dernières minutes*. Vous pouvez personnaliser les vignettes de KPI et définir un type et une plage de temps différents.
* **Vignettes des commandes d’appareil** : il s’agit notamment des vignettes **Close valve** (Fermer la vanne), **Open valve**(Ouvrir la vanne) et **Set valve position** (Régler la position de la vanne). Lorsque vous sélectionnez une commande, vous êtes dirigé vers la page des commandes de l’appareil simulé. Dans Azure IoT Central, une *commande* est un type de *fonctionnalité d’appareil*. Nous aborderons ce concept dans la section « Modèle d’appareil » de ce tutoriel.
* **Carte des zones de distribution de l’eau** : cette carte utilise Azure Maps, que vous pouvez configurer directement dans Azure IoT Central. La vignette de carte affiche l’emplacement des appareils. Essayez les contrôles de la carte, tels que *Zoom avant*, *Zoom arrière* ou *Développer*.

    ![Carte du tableau de bord de suivi de la consommation d’eau](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-dashboard-map.png)

* **Graphique en courbes du débit moyen de l’eau** et **Graphique en courbes des conditions environnementales** : vous pouvez visualiser les données de télémétrie provenant d’un ou de plusieurs appareils sous la forme d’un graphique en courbes représentant les données collectées au cours d’une période donnée.
* **Carte thermique de la pression moyenne des valves** : vous pouvez choisir le type de carte thermique à utiliser pour afficher les données de télémétrie des appareils qui ont été collectées au cours d’une période avec un index de couleurs.
* **Vignette de contenu Réinitialiser les seuils d’alerte** : Vous pouvez inclure des vignettes de contenu d’appel à l’action et incorporer un lien vers une page d’action. Dans ce cas, la vignette Réinitialiser les seuils d’alerte vous dirige vers l’application **Travaux**, dans laquelle vous pouvez mettre à jour les propriétés d’appareil. Nous aborderons cette option dans la section « Configuration des travaux » de ce tutoriel.
* **Vignettes Propriétés** : le tableau de bord affiche les vignettes **Valve operational info** (Informations opérationnelles de la vanne), **Flow alert thresholds** (Seuils d’alerte de débit) et **Maintenance info** (Informations de maintenance).

### <a name="customize-the-dashboard"></a>Personnaliser le tableau de bord

En tant que créateur, vous pouvez personnaliser les vues du tableau de bord pour les opérateurs.

1. Sélectionnez **Edit** (Modifier) pour personnaliser le tableau de bord **Wide World water consumption dashboard**. Vous pouvez personnaliser ce tableau de bord en sélectionnant le menu **Edit** (Modifier). Une fois que le tableau de bord est en mode **Edit**, vous pouvez ajouter de nouvelles vignettes ou le configurer.

     ![Modification du tableau de bord](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-edit-dashboard.png)

1. Sélectionnez **+ New** (Nouveau) pour créer un tableau de bord et le configurer intégralement. Vous pouvez avoir plusieurs tableaux de bord et passer de l’un à l’autre dans le menu Dashboard (Tableau de bord).

## <a name="explore-the-device-template"></a>Explorer le modèle d’appareil

Un modèle d’appareil dans Azure IoT Central définit la fonctionnalité d’un appareil, qui peut être la télémétrie, une propriété ou une commande. En tant que créateur, vous pouvez définir un ou plusieurs modèles d’appareil dans Azure IoT Central qui représenteront la fonctionnalité des appareils que vous connecterez.

L’application de suivi de la consommation d’eau est accompagnée de deux modèles d’appareil de référence représentant un *débitmètre* et une *vanne intelligente*.

Pour afficher ce modèle d’appareil :

1. Sélectionnez **Device templates** (Modèles d’appareil) dans le volet de gauche de votre application dans Azure IoT Central. Dans la liste **Device templates** (Modèles d’appareil), vous pouvez voir deux modèles d’appareil : **Smart Valve** (Vanne intelligente) et **Flow meter** (Débitmètre).

   ![Modèle d’appareil](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate.png)

1. Sélectionnez le modèle d’appareil **Flow meter** (Débitmètre) et familiarisez-vous avec les fonctionnalités de l’appareil.

     ![Modèle d’appareil Flow meter (Débitmètre)](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate-flowmeter.png)

### <a name="customize-the-device-template"></a>Personnaliser le modèle d’appareil

Pour personnaliser le modèle d’appareil :

1. Accédez à **Customize** (Personnaliser) dans le menu **Device templates** (Modèles d’appareils).
1. Recherchez le type de données de télémétrie `Temperature`.
1. Mettez à jour le nom d’affichage (**Display name**) en remplaçant `Temperature` par `Reported temperature`.
1. Mettez à jour l’unité de mesure ou définissez la valeur minimale (**Min value**) et la valeur maximale (**Max value**).
1. Sélectionnez **Save** (Enregistrer) pour enregistrer les modifications.

### <a name="add-a-cloud-property"></a>Ajouter une propriété cloud

1. Accédez à **Cloud Properties** (Propriétés cloud) dans le menu **Device templates** (Modèles d’appareils).
1. Ajoutez une propriété cloud en sélectionnant **+ Add Cloud Property** (Ajouter une propriété cloud).
    Dans Azure IoT Central, vous pouvez ajouter une propriété qui s’applique à l’appareil. Par exemple, une propriété cloud peut être un seuil d’alerte propre à la zone d’installation, aux informations sur les ressources ou à d’autres informations de maintenance.
1. Sélectionnez **Save** (Enregistrer) pour enregistrer les modifications.

### <a name="views"></a>Les vues

Le modèle d’appareil de suivi de la consommation d’eau est fourni avec des vues prédéfinies. Vous pouvez explorer ces vues et y apporter des modifications. Les vues définissent la façon dont les opérateurs voient les données de l’appareil, ainsi que la façon dont ils saisissent les propriétés cloud.

  ![Vues du modèle d’appareil](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devicetemplate-views.png)

### <a name="publish"></a>Publish

Si vous avez apporté des modifications, veillez à **publier** le modèle d’appareil.

### <a name="create-a-new-device-template"></a>Créer un modèle d’appareil

Sélectionnez **+ New** (Nouveau) pour créer un nouveau modèle d’appareil et suivre le processus de création.
Vous pouvez créer un modèle d’appareil personnalisé à partir de zéro ou choisir un modèle d’appareil dans le catalogue d’appareils Azure.

## <a name="explore-simulated-devices"></a>Explorer les appareils simulés

Dans Azure IoT Central, vous pouvez créer des appareils simulés pour tester votre modèle d’appareil et votre application. L’application de suivi de la consommation d’eau comprend deux appareils simulés mappés sur les modèles d’appareil **Flow meter** (Débitmètre) et **Smart Valve** (Vanne intelligente).

### <a name="view-the-devices"></a>Afficher les appareils

1. Sélectionnez **Devices (Appareils)**  > **All devices (Tous les appareils)** dans le volet gauche.

   ![Volet Tous les appareils](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-devices.png)

1. Sélectionnez **Smart Valve 1**.

    ![Smart Valve 1](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitor-device1.png)

1. Sous l’onglet **Commands** (Commandes), vous pouvez voir les trois commandes de l’appareil (**Close valve** (Fermer la vanne), **Open valve** (Ouvrir la vanne) et **Set valve position** (Régler la position de la vanne)), qui sont les fonctionnalités définies dans le modèle d’appareil **Smart Valve** (Vanne intelligente).

1. Explorez l’onglet **Device Properties** (Propriétés de l’appareil) et l’onglet **Device Dashboard** (Tableau de bord de l’appareil).

> [!NOTE]
> Notez que tous les onglets sont configurés à partir des vues de modèle d’appareil.

### <a name="add-new-devices"></a>Ajouter de nouveaux appareils

Ajoutez de nouveaux appareils en sélectionnant **+ New** (Nouveau) sous l’onglet **Devices** (Appareils).

## <a name="explore-and-configure-rules"></a>Explorer et configurer les règles

Dans Azure IoT Central, vous pouvez créer des règles pour superviser automatiquement les données de télémétrie des appareils et déclencher des actions quand une ou plusieurs conditions sont remplies. Ces actions peuvent inclure l’envoi d’e-mails de notification, le déclenchement d’une action Microsoft Power Automate ou une action webhook pour envoyer des données à d’autres services.

L’application de suivi de la consommation d’eau que vous avez créée a trois règles préconfigurées.

### <a name="view-rules"></a>Afficher les règles

1. Dans le volet gauche, sélectionnez **Rules** (Règles).

   ![Volet Règles](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-rules.png)

1. Sélectionnez **High pH alert** (Alerte de pH élevé), qui est l’une des règles préconfigurées dans l’application.

     ![Alerte de pH élevé](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-highflowalert.png)

    La règle `High flow alert` est configurée pour vérifier si la condition `Acidity (pH)` est `greater than` à `Max flow threshold`. Le seuil de débit maximal est une propriété cloud qui est définie dans le modèle d’appareil **Smart Valve** (Vanne intelligente). La valeur de `Max flow threshold` est définie pour chaque instance d’appareil.

À présent, nous allons créer une action de messagerie.

Pour ajouter une action à la règle :

1. Sélectionnez **+ E-mail**.
1. Entrez **Alerte de conteneur plein** comme nom d’affichage (**Display name**) convivial pour l’action.
1. Entrez l’adresse e-mail associée à votre compte Azure IoT Central dans **To** (À).
1. Si vous le souhaitez, entrez une note à inclure dans le texte de l’e-mail.
1. Sélectionnez **Terminé** pour terminer l’action.
1. Sélectionnez **Enregistrer** pour enregistrer et activer la nouvelle règle.

Si la condition configurée est remplie, vous recevrez un e-mail au bout de quelques minutes.

> [!NOTE]
> L’application envoie un e-mail chaque fois qu’une condition est remplie. Sélectionnez **Disable** (Désactiver) pour ne plus recevoir les e-mails qui sont générés par une règle automatisée.
  
Pour créer une nouvelle règle :

* Sélectionnez **+ New** (+ Nouveau) sous l’onglet **Rules** (Règles) situé dans le volet gauche.

## <a name="configure-jobs"></a>Configuration des travaux

Dans Azure IoT Central, les travaux vous permettent de déclencher la mise à jour des propriétés d’appareil ou des propriétés cloud sur plusieurs appareils. Outre les propriétés, vous pouvez également utiliser des travaux pour déclencher des commandes d’appareil sur plusieurs appareils. Azure IoT Central automatise le flux de travail pour vous.

1. Dans le volet gauche, sélectionnez **Jobs** (Travaux).
1. Sélectionnez **+New** (Nouveau) et configurez un ou plusieurs travaux.

## <a name="customize-your-application"></a>Personnaliser votre application

En tant que réalisateur de logiciel, vous pouvez modifier plusieurs paramètres pour personnaliser l’expérience utilisateur dans votre application.

1. Sélectionnez **Administration** > **Personnaliser votre application**.
1. Pour choisir l’image à charger en tant que **logo de l’application**, sélectionnez le bouton **Change** (Modifier).
1. Pour choisir l’image de l’**icône du navigateur** qui s’affichera dans les onglets du navigateur, sélectionnez le bouton **Change** (Modifier).
1. Vous pouvez également remplacer les **couleurs du navigateur** par défaut en ajoutant des codes couleur hexadécimaux HTML.

   ![Choix du logo de l’application, de l’icône du navigateur et des couleurs du navigateur](./media/tutorial-waterconsumptionmonitoring/waterconsumptionmonitoring-customize-your-application.png)

1. Vous pouvez également modifier les images d’application en sélectionnant **Administration** > **Application settings (Paramètres d’application)** . Pour choisir une image à charger en tant qu’image de l’application, sélectionnez le bouton **Select image** (Sélectionner une image).
1. Enfin, vous pouvez également modifier le **thème** en sélectionnant l’icône des **paramètres** située dans l’angle supérieur droit de l’application.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne comptez pas continuer à utiliser cette application, supprimez-la.

1. Sélectionnez **Administration** dans le volet de gauche de votre application Azure IoT Central.
1. Sélectionnez **Application settings** (Paramètres de l’application), puis sélectionnez le bouton **Delete** (Supprimer) en bas de la page.

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur 

> [!div class="nextstepaction"]
> [Concepts du suivi de la consommation d’eau](./concepts-waterconsumptionmonitoring-architecture.md).

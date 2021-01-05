---
title: 'Tutoriel : Créer une application de gestion connectée des déchets avec Azure IoT Central'
description: 'Tutoriel : Découvrez comment créer une application de gestion connectée des déchets à l’aide de modèles d’application Azure IoT Central.'
author: miriambrus
ms.author: miriamb
ms.date: 12/11/2020
ms.topic: tutorial
ms.service: iot-central
services: iot-central
ms.openlocfilehash: 5ecfa3e4c46c17a3b381e9743953c7d97309ea3b
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97587245"
---
# <a name="tutorial-create-a-connected-waste-management-application-in-iot-central"></a>Tutoriel : Créer une application de gestion connectée des déchets dans IoT Central

Ce tutoriel vous guide tout au long de la création d’une application de gestion connectée des déchets Azure IoT Central à partir du modèle d’application IoT Central **Connected waste management** (Gestion connectée des déchets). 

Dans ce didacticiel, vous apprendrez à : 

* Utiliser le modèle Azure IoT Central **Connected waste management** (Gestion connectée des déchets) pour créer votre application de gestion connectée des déchets
* Explorer et personnaliser le tableau de bord de l’opérateur 
* Explorer le modèle d’appareil de conteneur connecté de déchets
* Explorer les appareils simulés
* Explorer et configurer les règles
* Configuration des travaux
* Personnaliser la marque de votre application à l’aide d’une étiquette blanche

## <a name="prerequisites"></a>Prérequis

Pour suivre ce didacticiel, vous avez besoin des éléments suivants :
*  Un abonnement Azure est recommandé. Vous pouvez éventuellement utiliser une version d’évaluation gratuite de 7 jours. Si vous n’avez pas d’abonnement Azure, vous pouvez en créer un sur la [page d’inscription à Azure](https://aka.ms/createazuresubscription).

## <a name="create-connected-waste-management-app-in-iot-central"></a>Créer une application de gestion connectée des déchets dans IoT Central

Dans cette section, vous utilisez le modèle Azure IoT Central **Connected waste management** (Gestion connectée des déchets) pour créer votre application connectée de gestion des déchets dans IoT Central.

Pour créer une application Azure IoT Central de gestion connectée des déchets :  

1. Accédez à la [page d’accueil d’Azure IoT Central](https://aka.ms/iotcentral).

    Si vous disposez d’un abonnement Azure, connectez-vous avec les informations d’identification que vous utilisez pour y accéder. Sinon, connectez-vous à l’aide d’un compte Microsoft :

    ![Entrer votre compte d’entreprise](./media/tutorial-connectedwastemanagement/sign-in.png)

1. Cliquez sur **Build** (Créer) dans le volet gauche et sélectionnez l’onglet **Government** (Service public). La page relative au secteur public affiche plusieurs modèles d’application pour le secteur public.

    ![Créer des modèles d’application pour le secteur public](./media/tutorial-connectedwastemanagement/iotcentral-government-tab-overview.png)

1. Sélectionnez le modèle d’application **Connected Waste Management** (Gestion connectée des déchets). Ce modèle comprend un exemple de modèle d’appareil de conteneur connecté de déchets, un appareil simulé, un tableau de bord d’opérateur et des règles de supervision préconfigurées.    

1. Cliquez sur **Create app** (Créer une application) pour ouvrir le formulaire de création **New application** (Nouvelle application) avec les champs suivants :
    * **Application name** (Nom de l’application). Par défaut, le nom de l’application est *Connected waste management* suivi d’une chaîne d’ID unique générée par IoT Central. Si vous le souhaitez, choisissez un nom d’application convivial. Vous pouvez également modifier ultérieurement le nom de l’application.
    * **URL** – Si vous le souhaitez, vous pouvez spécifier l’URL de votre choix. Vous pouvez également modifier ultérieurement l’URL. 
    * Si vous disposez d’un abonnement Azure, entrez vos informations *Répertoire, Abonnement Azure et Région*. Si vous n’avez pas d’abonnement, vous pouvez activer une **version d’évaluation gratuite de 7 jours** et remplir les coordonnées requises.  

    Pour plus d’informations sur les répertoires et les abonnements, consultez le guide de démarrage rapide [Créer une application](../core/quick-deploy-iot-central.md).

1. Cliquez sur le bouton **Create** (Créer) en bas de la page. 

    ![Azure IoT Central – Page de création d’une application de gestion connectée des déchets](./media/tutorial-connectedwastemanagement/new-application-connectedwastemanagement.png)
    
    ![Création d’une application Azure IoT Central connectée, informations de facturation](./media/tutorial-connectedwastemanagement/new-application-connectedwastemanagement-billinginfo.png)

 
1. Vous avez désormais créé une application de gestion connectée des déchets à l’aide du modèle Azure IoT Central **Gestion connectée des déchets**. 

Félicitations ! L’application que vous venez de créer est accompagnée des éléments préconfigurés suivants :
* Exemples de tableaux de bord d’opérateur
* Exemples de modèles d’appareil préconfigurés de conteneur connecté de déchets
* Conteneur de déchets connectés simulés
* Règles et travaux préconfigurés
* Exemple de marquage à l’aide d’une étiquette blanche 

Cette application est la vôtre et vous pouvez la modifier à tout moment. Nous allons maintenant explorer l’application et effectuer certaines personnalisations.  

## <a name="explore-and-customize-operator-dashboard"></a>Explorer et personnaliser le tableau de bord de l’opérateur 
Après avoir créé l’application, vous accédez au **tableau de bord de gestion connectée des déchets dans le monde**.

   ![Tableau de bord de gestion connectée des déchets](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard1.png)

En tant que créateur, vous pouvez créer et personnaliser des vues dans le tableau de bord pour les opérateurs. Avant d’essayer de le personnaliser, commençons par explorer le tableau de bord. 

>>[!NOTE]
>> Toutes les données affichées dans le tableau de bord sont basées sur des données d’appareils simulés, que nous explorerons dans la section suivante. 

Le tableau de bord est constitué de différents types de vignettes :

* ***Vignette d’image Wide World Waste utility** _ (Utilitaire Wide World Waste) : la première vignette du tableau de bord est une vignette d’image d’un utilitaire fictif de gestion des déchets « Wide World Waste ». Vous pouvez personnaliser cette vignette et fournir votre propre image, ou la supprimer. 

_ ***Vignette d’image Waste bin** _ (Conteneur de déchets) : vous pouvez utiliser des vignettes d’image et de contenu pour créer une représentation visuelle de l’appareil supervisé et y ajouter un texte descriptif. 

_ ***Vignette de KPI de niveau de remplissage** _ : la vignette affiche une valeur signalée par un capteur de _niveau de remplissage* dans un conteneur de déchets. Le capteur *Fill level* (Niveau de remplissage) et d’autres capteurs, comme *Odor meter* (Mesure des odeurs) ou *Weight* (Poids), peuvent être supervisés à distance dans un conteneur de déchets. Un opérateur peut entreprendre une action, comme envoyer un camion de ramassage des ordures. 

* ***Waste monitoring area map** _ (Carte des zones de supervision des déchets) : la carte utilise Azure Maps, que vous pouvez configurer directement dans Azure IoT Central. La vignette de carte affiche l’emplacement des appareils. Essayez de survoler la carte et essayez les contrôles sur la carte, comme le zoom avant, le zoom arrière ou développer.

     ![Carte du tableau de bord de gestion connectée des déchets](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard-map.png)


_ ***Graphique à barres des niveaux de remplissage, d’odeur et de poids** : vous pouvez visualiser les données de télémétrie d’un ou de plusieurs appareils dans un graphique à barres. Vous pouvez également développer le graphique à barres.  

  ![Graphique à barres du tableau de bord de gestion connectée des déchets](./media/tutorial-connectedwastemanagement/connectedwastemanagement-dashboard-barchart.png)


* **Vignette de contenu Field Services** (Services sur le terrain) : le tableau de bord inclut un lien pour l’intégration de services sur le terrain Dynamics 365 à partir de votre application Azure IoT Central. Par exemple, vous pouvez utiliser Field Services (Services sur le terrain) pour créer des tickets d’affectation des services de ramassage des ordures. 


### <a name="customize-dashboard"></a>Personnaliser le tableau de bord 

En tant que créateur, vous pouvez personnaliser les vues du tableau de bord pour les opérateurs. Vous pouvez essayer de procéder comme suit :
1. Cliquez sur **Edit** (Modifier) pour personnaliser le tableau de bord de gestion connectée des déchets dans le monde (**Wide World waste management dashboard**). Vous pouvez personnaliser ce tableau de bord en cliquant sur le menu **Edit** (Modifier). Une fois que le tableau de bord est en **mode d’édition**, vous pouvez ajouter de nouvelles vignettes et les configurer. 

    ![Modification du tableau de bord](./media/tutorial-connectedwastemanagement/edit-dashboard.png)

1. Vous pouvez également cliquer sur **+ New** (Nouveau) pour créer un nouveau tableau de bord et le configurer à partir de zéro. Vous pouvez avoir plusieurs tableaux de bord et naviguer entre eux via le menu Dashboard (Tableau de bord). 

## <a name="explore-connected-waste-bin-device-template"></a>Explorer le modèle d’appareil de conteneur connecté de déchets

Un modèle d’appareil dans Azure IoT Central définit la fonctionnalité d’un appareil, qui peut être la télémétrie, des propriétés ou une commande. En tant que créateur, vous pouvez définir des modèles d’appareil qui représenteront la fonctionnalité des appareils que vous connecterez. 

L’application **Connected waste management** (Gestion connectée des déchets) est fournie avec un exemple de modèle d’appareil de conteneur connecté de déchets.

Pour afficher ce modèle d’appareil :

1. Cliquez sur **Modèles d’appareil** dans le volet gauche de votre application dans IoT Central. 

    ![Capture d’écran montrant la liste des modèles d’appareil dans l’application](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate.png)

1. Dans la liste Modèles d’appareil, vous voyez **Connected Waste Bin** (Conteneur de déchets connecté). Ouvrez-le en cliquant sur son nom.

1. Familiarisez-vous avec les capacités de ce modèle d’appareil. Vous pouvez voir qu’il définit des capteurs comme que *Fill level* (Niveau de remplissage), *Odor meter* (Mesure des odeurs), *Weight* (Poids), *Location* (Emplacement), etc.

   ![Capture d’écran montrant les détails du modèle d’appareil de conteneur connecté de déchets](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate-connectedbin.png)


### <a name="customizing-the-device-template"></a>Personnalisation du modèle d’appareil

Essayez de personnaliser les éléments suivants :
1. Accédez à **Customize** (Personnaliser) via le menu du modèle d’appareil.
1. Recherchez le type de télémétrie `Odor meter`.
1. Mettez à jour le nom d’affichage (**Display name**) en remplaçant `Odor meter` par `Odor level`.
1. Vous pouvez également essayer de mettre à jour l’unité de mesure ou de définir *Min value* (Valeur minimale) et *Max value* (Valeur maximale).
1. Enregistrez (**Save**) les modifications. 

### <a name="add-a-cloud-property"></a>Ajouter une propriété cloud 

1. Accédez à **Cloud properties** (Propriétés cloud) via le menu du modèle d’appareil.
1. Ajoutez une nouvelle propriété cloud en cliquant sur **+ Add Cloud Property** (Ajouter une propriété cloud). Dans IoT Central, vous pouvez ajouter une propriété qui s’applique à l’appareil, mais qui n’est pas censée être envoyée par un appareil. Par exemple, une propriété cloud peut être un seuil d’alerte spécifique à la zone d’installation, aux informations sur les ressources, aux informations de maintenance ou à d’autres informations. 
1. Enregistrez (**Save**) les modifications. 
 
### <a name="views"></a>Les vues 
* Le modèle d’appareil de conteneur connecté de déchets est fourni avec des vues prédéfinies. Vous pouvez explorer ces vues et y apporter des modifications. Les vues définissent la façon dont les opérateurs verront les données de l’appareil et réaliseront également la saisie des propriétés cloud. 

  ![Vues du modèle d’appareil](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devicetemplate-views.png)

### <a name="publish"></a>Publish 

* Si vous avez apporté des modifications, veillez à **publier** le modèle d’appareil. 

### <a name="create-a-new-device-template"></a>Créer un modèle d’appareil 

* Sélectionnez **+ New** (Nouveau) pour créer un nouveau modèle d’appareil et suivre le processus de création. Vous pouvez créer un modèle d’appareil personnalisé à partir de zéro ou choisir un modèle d’appareil dans le catalogue d’appareils Azure. 

## <a name="explore-simulated-devices"></a>Explorer les appareils simulés

Dans IoT Central, vous pouvez créer des appareils simulés pour tester votre modèle d’appareil et votre application. 

L’application **Connected waste management** (Gestion connectée des déchets) possède deux appareils simulés, mappés sur le modèle d’appareil de conteneur connecté de déchets. 

### <a name="to-view-the-devices"></a>Pour afficher ces appareils :

1. Accédez à **Devices** (Appareils) dans le volet gauche IoT Central. 

   ![Appareils](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devices.png)

1. Cliquez sur l’appareil Connected Waste Bin (Conteneur connecté de déchets).  

     ![Appareil 1](./media/tutorial-connectedwastemanagement/connectedwastemanagement-devices-bin1.png)

1. Accédez à l’onglet **Cloud Properties** (Propriétés cloud) et essayez de mettre à jour la valeur `Bin full alert threshold` en remplaçant `95` par `100`. 
* Explorez l’onglet **Device Properties** (Propriétés de l’appareil) et l’onglet **Device Dashboard** (Tableau de bord de l’appareil). 

>> [!NOTE]
>> Tous les onglets ont été configurés à partir des **vues du modèle d’appareil**.

### <a name="add-new-devices"></a>Ajouter de nouveaux appareils

* Vous pouvez ajouter de nouveaux appareils en cliquant sur **+ New** (Nouveau) sous l’onglet **Devices** (Appareils). 

## <a name="explore-and-configure-rules"></a>Explorer et configurer les règles

Dans Azure IoT Central, vous pouvez créer des règles pour superviser automatiquement les données de télémétrie des appareils et déclencher des actions quand une ou plusieurs conditions sont remplies. Ces actions peuvent inclure l’envoi d’e-mails de notification, le déclenchement d’une action Power Automate ou une action webhook pour envoyer des données à d’autres services.

L’application **Connected Waste Management** (Gestion connectée des déchets) possède quatre exemples de règles.

### <a name="to-view-rules"></a>Pour afficher ces règles :
1. Accédez à **Rules** (Règles) dans le volet gauche IoT Central.

   ![Règles](./media/tutorial-connectedwastemanagement/connectedwastemanagement-rules.png)

1. Sélectionnez l’alerte **Bin full alert** (Alerte de conteneur plein).

     ![Alerte de conteneur plein](./media/tutorial-connectedwastemanagement/connectedwastemanagement-binfullalert.png)

 1. L’alerte `Bin full alert` vérifie la **condition** `Fill level is greater than or equal to Bin full alert threshold`.

    `Bin full alert threshold` est une *propriété cloud* définie dans le modèle d’appareil `Connected waste bin`. 

À présent, nous allons créer une action de messagerie.

### <a name="create-an-email-action"></a>Créer une action de messagerie
Pour configurer une action de messagerie dans la liste des actions de la règle :
1. Sélectionnez **+ E-mail**. 
1. Entrez *Alerte de conteneur plein* comme nom d’affichage (**Display name**) convivial pour l’action.
1. Entrez l’adresse e-mail associée à votre compte IoT Central dans **To** (À). 
1. Si vous le souhaitez, entrez une note à inclure dans le texte de l’e-mail.
1. Sélectionnez **Terminé** pour terminer l’action.
1. Sélectionnez **Enregistrer** pour enregistrer et activer la nouvelle règle. 

Vous devrez recevoir un e-mail quand la **condition** configurée sera remplie.

>[!NOTE]
>L’application enverra un e-mail chaque fois qu’une condition sera remplie. Désactivez (**Disable**) la règle pour cesser de recevoir des e-mails à partir de la règle automatisée. 
  
Pour créer une nouvelle règle : 
1. Sélectionnez **+ New** (+ Nouveau) sous **Rules** (Règles) dans le volet gauche.

## <a name="configure-jobs"></a>Configurer des travaux

Dans IoT Central, les travaux vous permettent de déclencher des mises à jour des propriétés d’appareil ou cloud sur plusieurs appareils. Outre les propriétés, vous pouvez également utiliser des travaux pour déclencher des commandes d’appareil sur plusieurs appareils. IoT Central automatisera le flux de travail pour vous. 

1. Accédez à **Jobs** (Travaux) dans le volet gauche. 
1. Cliquez sur **+New** (Nouveau) et configurez un ou plusieurs travaux. 


## <a name="customize-your-application"></a>Personnaliser votre application 

En tant que réalisateur de logiciel, vous pouvez modifier plusieurs paramètres pour personnaliser l’expérience utilisateur dans votre application.

### <a name="to-change-the-application-theme"></a>Pour modifier le thème de l’application :

1. Accédez à **Administration > Personnaliser votre application**.
1. Utilisez le bouton **Modifier** pour choisir une image à télécharger en tant que **Logo de l’application**.
1. Utilisez le bouton **Modifier** pour choisir l’image de l’**icône du navigateur** qui apparaîtra dans les onglets du navigateur.
1. Vous pouvez également remplacer les **couleurs du navigateur** par défaut en ajoutant des codes couleur hexadécimaux HTML.

   ![Personnalisation de votre application dans Azure IoT Central](./media/tutorial-connectedwastemanagement/connectedwastemanagement-customize-your-application.png)

1. Vous pouvez également modifier les images de l’application en accédant à **Administration > Paramètres d’application** et en cliquant sur le bouton **Sélectionner une image** pour choisir une image à charger en tant qu’image de l’application.
1. Enfin, vous pouvez également modifier le **thème** en cliquant sur **Paramètres** sur le titre de l’application.

  
## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne pensez pas continuer à utiliser cette application, supprimez votre application en procédant comme suit :

1. Ouvrez l’onglet Administration dans le volet gauche de votre application IoT Central.
1. Sélectionnez Paramètres de l'application, puis cliquez sur le bouton Supprimer en bas de la page.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus en détail les
 
> [!div class="nextstepaction"]
> [concepts de la gestion connectée des déchets](./concepts-connectedwastemanagement-architecture.md)

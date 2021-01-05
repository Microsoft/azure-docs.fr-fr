---
title: Tutoriel - Créer une application de suivi continu des patients avec Azure IoT Central | Microsoft Docs
description: Dans ce tutoriel, vous allez apprendre à créer une application de suivi continu des patients à l’aide de modèles d’application Azure IoT Central.
author: philmea
ms.author: philmea
ms.date: 09/24/2019
ms.topic: tutorial
ms.service: iot-central
services: iot-central
manager: eliotgra
ms.openlocfilehash: 1967a2fb5adebe01ef4bff8d58f7832bffe95762
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90531268"
---
# <a name="tutorial-deploy-and-walkthrough-a-continuous-patient-monitoring-app-template"></a>Tutoriel : Déployer et parcourir pas à pas un modèle d’application de surveillance continue des patients

Ce tutoriel vous montre comment vous lancer, en votre qualité de générateur de solutions, en déployant un modèle d’application de surveillance continue des patients IoT Central. Vous allez apprendre à déployer et à utiliser le modèle.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer un modèle d’application
> * Parcourir pas à pas le modèle d’application

## <a name="create-an-application-template"></a>Créer un modèle d’application

Accédez au [site web du gestionnaire d’applications Azure IoT Central](https://apps.azureiotcentral.com/). Sélectionnez **Générer** dans la barre de navigation de gauche, puis l’onglet **Santé**.

:::image type="content" source="media/app-manager-health.png" alt-text="Modèle d’application Santé":::

Sélectionnez le bouton **Créer une application** pour commencer à créer votre application, puis connectez-vous avec un compte personnel, professionnel ou scolaire Microsoft. Cela vous amène la page **New application** (Nouvelle application).

![Créer une application Healthcare](media/app-manager-health-create.png)

![Créer une application de santé, informations de facturation](media/app-manager-health-create-billinginfo.png)

Pour créer votre application :

1. Azure IoT Central suggère automatiquement un nom d’application basé sur le modèle que vous avez sélectionné. Vous pouvez accepter ce nom ou entrer le nom d’application convivial de votre choix, par exemple **Surveillance continue des patients**. Azure IoT Central génère aussi un préfixe d’URL unique basé sur le nom de l’application. Si vous le souhaitez, vous pouvez remplacer ce préfixe d’URL par une chaîne plus facile à mémoriser.

2. Vous pouvez choisir de créer l’application avec le plan tarifaire *gratuit* ou avec l’un des plans tarifaires *standard*. Les applications créées avec le plan gratuit sont utilisables gratuitement pendant sept jours et sur cinq appareils avant leur expiration. Vous pouvez transférer une application du plan gratuit vers un plan tarifaire standard à tout moment avant son expiration. Si vous choisissez le plan gratuit, entrez vos coordonnées et indiquez si vous souhaitez recevoir des informations et des conseils de la part de Microsoft. Les applications créées avec un plan tarifaire standard peuvent être utilisées gratuitement sur deux appareils au maximum et nécessitent vos informations d’abonnement Azure pour la facturation.

3. Sélectionnez **Create** (Créer) au bas de la page pour déployer votre application.

## <a name="walk-through-the-application-template"></a>Parcourir pas à pas le modèle d’application

### <a name="dashboards"></a>Tableaux de bord

Après avoir déployé le modèle d’application, vous accédez dans un premier temps au **tableau de bord de surveillance des patients Lamna**. Lamna Healthcare est un ensemble hospitalier fictif qui se compose de deux hôpitaux : Woodgrove Hospital et Burkville Hospital. Dans le tableau de bord de l’opérateur du Woodgrove Hospital, vous pouvez :

* Consulter les données de télémétrie et les propriétés des appareils, comme le **niveau de charge de la batterie** ou le statut de **connectivité**.

* Affichez le **plan d’étage** et l’emplacement de l’appareil Smart Vitals Patch.

* **Reprovisionner** le Smart Vitals Patch pour un nouveau patient.

* Voir un exemple de **tableau de bord de fournisseur** dont pourrait se servir une équipe de soins hospitaliers pour assurer le suivi de leurs patients.

* Changer le **statut de surveillance** de votre appareil pour indiquer s’il est utilisé à l’hôpital ou dans un scénario à distance.

:::image type="content" source="media/lamna-in-patient.png" alt-text="Modèle d’application Santé":::

Vous pouvez aussi sélectionner **Go to remote patient dashboard** (Accéder au tableau de bord de surveillance à distance des patients) pour voir le tableau de bord de l’opérateur du Burkville Hospital. Ce tableau de bord contient un ensemble similaire d’actions, de données de télémétrie et d’informations. Vous pouvez également voir plusieurs appareils en cours d’utilisation et choisir de **mettre à jour le microprogramme** sur chacun.

:::image type="content" source="media/lamna-remote.png" alt-text="Modèle d’application Santé":::

### <a name="device-templates"></a>Modèles d’appareil

Si vous sélectionnez **Modèles d’appareil**, vous voyez les deux types d’appareils dans le modèle :

* **Smart Vitals Patch** : cet appareil représente un patch qui mesure différents signes vitaux. Il sert au suivi des patients à l’intérieur et à l’extérieur de l’hôpital. Si vous sélectionnez le modèle, vous constatez que le patch envoie à la fois des données sur l’appareil comme le niveau de charge de la batterie et la température de l’appareil, et des données médicales des patients comme la fréquence respiratoire et la pression artérielle.

* **Smart Knee Brace** : cet appareil représente une attelle de genou que les patients utilisent à la suite d’une opération de reconstruction du genou. Si vous sélectionnez ce modèle, vous voyez des fonctionnalités comme des données sur l’appareil, l’amplitude de mouvement et la mobilité.

:::image type="content" source="media/smart-vitals-device-template.png" alt-text="Modèle d’application Santé":::

### <a name="device-groups"></a>Groupes d'appareils

Les groupes d’appareils vous permettent de regrouper logiquement un ensemble d’appareils, puis d’effectuer des requêtes ou des opérations dessus.

Si vous sélectionnez l’onglet Groupes d’appareils, vous voyez un groupe d’appareils par défaut pour chaque modèle d’appareil dans l’application. Deux exemples de groupes d’appareils supplémentaires ont aussi été créés, appelés **Provision devices** (Appareils de provisionnement) et **Devices with outdated firmware** (Appareils avec microprogramme obsolète). Vous pouvez utiliser ces exemples de groupes d’appareils comme entrées pour exécuter certains des [travaux](#jobs) dans l’application.

### <a name="rules"></a>Règles

Si vous sélectionnez **Règles**, vous voyez les trois règles dans le modèle :

* **Brace temperature high** (Température élevée de l’attelle) : cette règle se déclenche quand la température de l’attelle de genou Smart Knee Brace est supérieure à 95 &deg;F (35 °C) sur une période de 5 minutes. Utilisez cette règle pour alerter le patient et l’équipe de soins afin qu’elle refroidisse l’appareil à distance.

* **Fall detected** (Chute détectée) : cette règle se déclenche si une chute de patient est détectée. Utilisez cette règle pour configurer une action d’intervention d’équipe opérationnelle afin de secourir le patient qui a chuté.

* **Patch battery low** (Faible niveau de charge du patch) : cette règle se déclenche quand le niveau de charge de la batterie de l’appareil est inférieur à 10 %. Utilisez cette règle pour déclencher une notification à l’intention du patient afin qu’il charge son appareil.

:::image type="content" source="media/brace-temp-rule.png" alt-text="Modèle d’application Santé":::

### <a name="jobs"></a>travaux

Les travaux vous permettent d’exécuter des opérations en bloc sur un ensemble d’appareils, en utilisant des [groupes d’appareils](#device-groups) comme entrée. Le modèle d’application comporte deux exemples de travaux qu’un opérateur peut exécuter :

* **Mettre à jour le microprogramme des genouillères** : Ce travail recherche les appareils dans le groupe d’appareils **Devices with outdated firmware** et exécute une commande pour mettre à jour ces appareils vers la dernière version du microprogramme. Cet exemple de travail suppose que les appareils peuvent gérer une commande **update** et récupérer les fichiers de microprogramme à partir du cloud.  

* **Reprovisionner des appareils** : Vous disposez d’un ensemble d’appareils qui ont été récemment retournés à l’hôpital. Ce travail recherche les appareils dans le groupe d’appareils **Provision devices** et exécute une commande afin de les reprovisionner pour l’ensemble suivant de patients.

### <a name="devices"></a>Périphériques

Sélectionnez l’onglet **Appareils**, puis une instance de **Smart Knee Brace**. Il existe trois vues qui permettent d’explorer les informations sur l’appareil que vous avez sélectionné. Ces vues sont créées et publiées au moment de générer le modèle de votre appareil. Par conséquent, elles sont identiques sur tous les appareils que vous connectez ou simulez.

La vue **Tableau de bord** offre une vue d’ensemble des données de télémétrie et des propriétés de l’appareil destinées aux opérateurs.

L’onglet **Propriétés** vous permet de modifier les propriétés du cloud et de lire/d’écrire les propriétés de l’appareil.

L’onglet **Commandes** vous permet d’exécuter des commandes sur l’appareil.

:::image type="content" source="media/knee-brace-dashboard.png" alt-text="Modèle d’application Santé":::

### <a name="data-export"></a>Exportation de données

L’exportation de données vous permet d’exporter les données de vos appareils en continu vers d’autres services Azure, notamment l’[API Azure pour FHIR](concept-continuous-patient-monitoring-architecture.md#export-to-azure-api-for-fhir).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’envisagez pas de continuer à utiliser cette application, supprimez-la en accédant à **Administration > Application settings** (Administration > Paramètre d’application), puis cliquez sur **Delete** (Supprimer).

:::image type="content" source="media/admin-delete.png" alt-text="Modèle d’application Santé":::

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant pour apprendre à créer un tableau de bord de fournisseur qui se connecte à votre application IoT Central.

> [!div class="nextstepaction"]
> [Créer un tableau de bord de fournisseur](howto-health-data-triage.md)
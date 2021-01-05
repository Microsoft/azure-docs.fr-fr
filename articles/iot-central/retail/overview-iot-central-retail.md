---
title: Création de solutions de vente au détail avec Azure IoT Central | Microsoft Docs
description: Découvrez comment utiliser les modèles d’applications Azure IoT Central pour créer des solutions de logistique connectée, de centre de distribution numérique, de données analytiques en magasin, de supervision des conditions, de paiement, de gestion intelligente des stocks et de vente au détail.
author: avneet723
ms.author: avneets
ms.date: 01/10/2020
ms.topic: overview
ms.service: iot-central
ms.subservice: iot-central-retail
services: iot-central
ms.openlocfilehash: 43bf91c83f66b05c448e52286b34c51a9fd30a2b
ms.sourcegitcommit: 7dacbf3b9ae0652931762bd5c8192a1a3989e701
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92124960"
---
# <a name="building-retail-solutions-with-azure-iot-central"></a>Création de solutions de vente au détail avec Azure IoT Central

Azure IoT Central est une plateforme d’application IoT qui réduit les charges et les coûts associés au développement, à la gestion et à la maintenance de solutions IoT de qualité professionnelle. Le fait de créer avec Azure IoT Central vous offre la possibilité de consacrer du temps, de l’argent et de l’énergie à transformer votre activité avec des données IoT, plutôt qu’à simplement gérer et mettre à jour une infrastructure IoT complexe et en constante évolution.

Cet article décrit plusieurs modèles d’application IoT Central propres à la vente au détail. En tant que créateur de solutions, vous pouvez utiliser ces modèles pour créer des solutions IoT qui optimisent la chaîne logistique, améliorent l’expérience en magasin des clients et permettent de suivre les stocks plus efficacement.

:::image type="content" source="media/overview-iot-central-retail/retail-app-templates.png" alt-text="Présentation d’Azure IoT Retail":::

Les sections suivantes décrivent les fonctionnalités de ces modèles d’applications :

## <a name="connected-logistics"></a>Logistique connectée

Les coûts logistiques mondiaux devraient atteindre 10,6 billions de dollars en 2020. La plus grande partie de ces dépenses est imputable au transport des marchandises, et les transporteurs sont soumis à de fortes contraintes, ainsi qu’à une forte concurrence.

Vous pouvez utiliser des capteurs IoT afin de collecter et de superviser des conditions ambiantes, telles que la température, l’humidité, l’inclinaison, les chocs, la lumière et l’emplacement d’une marchandise. Vous pouvez combiner les données de télémétrie collectées à partir des capteurs et des appareils IoT avec d’autres sources de données, telles que la météo et le trafic dans les systèmes décisionnels basés sur le cloud.

Les avantages d’une solution de logistique connectée sont les suivants :

* Supervision de la cargaison à l’aide d’un suivi et d’un traçage en temps réel 
* Intégrité de la cargaison à l’aide d’une supervision en temps réel des conditions ambiantes
* Sécurité de la cargaison en lien avec le vol, la perte ou les dommages
* Délimitation géographique, optimisation des itinéraires, gestion des flottes et analytique des véhicules
* Prévisions concernant le départ et l’arrivée de la cargaison

Les captures d’écran suivantes montrent le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions :

:::image type="content" source="media/overview-iot-central-retail/connected-logistics-dashboard1.png" alt-text="Présentation d’Azure IoT Retail":::

:::image type="content" source="media/overview-iot-central-retail/connected-logistics-dashboard2.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’informations, consultez le tutoriel [Déployer et découvrir un modèle d’application de logistique connectée](./tutorial-iot-central-connected-logistics.md).

## <a name="digital-distribution-center"></a>Centre de distribution numérique

À mesure que les fabricants et détaillants établissent leur présence dans le monde entier, leurs chaînes logistiques se ramifient et deviennent de plus en plus complexes. Les consommateurs s’attendent à ce que de vastes éventails de produits soient disponibles et à ce que leurs achats leur soient livrés dans un délai d’un ou deux jours après l’achat. Les centres de distribution doivent s’adapter à ces tendances tout en éliminant des inefficacités existantes.

Aujourd’hui, le prélèvement et le conditionnement manuels représentent de 55 à 65 % des coûts des centres de distribution. Le prélèvement et le conditionnement manuels sont généralement plus lents que lorsqu’ils sont automatisés. En outre, la fluctuation rapide des besoins en personnel fait qu’il est encore plus difficile de traiter les volumes élevés des marchandises à expédier. Cette fluctuation saisonnière entraîne un taux élevé de renouvellement du personnel et augmente la probabilité d’erreurs coûteuses.

Des solutions basées sur des caméras compatibles IoT peuvent offrir des avantages en permettant la mise en place d’une boucle de rétroaction numérique. Les données du centre de distribution se transforment en insights actionnables qui, à leur tour, produisent de meilleures données.

Les avantages d’un centre de distribution numérique sont les suivants :

* Supervision à l’aide de caméras des marchandises à mesure qu’elles arrivent et transitent par le système d’acheminement
* Identification automatique des marchandises défectueuses
* Suivi efficace des commandes
* Réduction des coûts, amélioration de la productivité et optimisation de l’utilisation

La capture d’écran suivante montre le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions : 

:::image type="content" source="media/overview-iot-central-retail/digital-distribution-center-dashboard.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’informations, consultez le tutoriel [Déployer et parcourir un modèle d’application du centre de distribution numérique](./tutorial-iot-central-digital-distribution-center.md).

## <a name="in-store-analytics---condition-monitoring"></a>Analytique en magasin – Monitorage des conditions

Pour de nombreux détaillants, les bonnes conditions ambiantes dont bénéficient leurs magasins est ce qui les distingue de leurs concurrents. Les détaillants ont intérêt à maintenir des conditions ambiantes agréables dans leurs magasins pour le bien-être de leurs clients.  

En tant que créateur de solutions, vous pouvez utiliser le modèle d’application « Analytique dans le magasin – Surveillance des conditions » d’IoT Central afin de créer une solution de bout en bout. Le modèle d’application vous permet de connecter et de superviser numériquement l’environnement d’un magasin de détail en utilisant différentes sortes de capteurs. Ces capteurs génèrent des données de télémétrie que vous pouvez convertir en insights métier afin d’aider le détaillant à réduire ses coûts d’exploitation et à fournir une expérience optimale pour ses clients.

Utilisez le modèle d’application pour :

* Connecter différents genres de capteurs IoT à une instance d’application IoT Central
* Surveiller et gérer l’intégrité du réseau de capteurs et les éventuels appareils de passerelle de l’environnement
* Créer des règles personnalisées en lien avec les conditions ambiantes au sein d’un magasin afin de déclencher des alertes à l’intention des responsables magasin
* Convertir les conditions ambiantes du magasin en insights que l’équipe du magasin peut utiliser pour améliorer l’expérience du client
* Exporter les insights agrégés dans des applications métier existantes ou nouvelles afin de fournir des informations pertinentes aux vendeurs

Le modèle d’application est fourni avec un ensemble de modèles d’appareils, et utilise un ensemble d’appareils simulés pour renseigner le tableau de bord. 

La capture d’écran suivante montre le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions : 

:::image type="content" source="media/overview-iot-central-retail/in-store-analytics-condition-dashboard.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’information, consultez le tutoriel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md).

## <a name="in-store-analytics---checkout"></a>Analytique en magasin – Validation de l’achat

Pour certains détaillants, c’est l’expérience de paiement en magasin qui les distingue de la concurrence. Les détaillants veulent offrir une expérience simplifiée de paiement en magasin afin d’inciter les clients à revenir.  

En tant que créateur de solutions, vous pouvez utiliser le modèle d’application « Analytique dans le magasin – Paiement » d’IoT Central afin de créer une solution qui fournit aux vendeurs des insights concernant les caisses. Par exemple, les capteurs peuvent fournir des informations sur la longueur des files d’attente et les temps d’attente moyens pour chaque caisse.

Utilisez le modèle d’application pour :

* Connecter différents genres de capteurs IoT à une instance d’application IoT Central
* Surveiller et gérer l’intégrité du réseau de capteurs et les éventuels appareils de passerelle de l’environnement
* Créer des règles personnalisées concernant le paiement en magasin afin de déclencher des alertes à l’intention des vendeurs
* Convertir les conditions de paiement en magasin en insights, que l’équipe du magasin peut utiliser pour améliorer l’expérience du client
* Exporter les insights agrégés dans des applications métier existantes ou nouvelles afin de fournir des informations pertinentes aux vendeurs

Le modèle d’application est fourni avec un ensemble de modèles d’appareils, et utilise un ensemble d’appareils simulés pour renseigner le tableau de bord avec les données concernant la longueur des files d’attente de chaque caisse. 

La capture d’écran suivante montre le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions : 

:::image type="content" source="media/overview-iot-central-retail/In-Store-Analytics-Checkout-Dashboard.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’information, consultez le tutoriel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md).

## <a name="smart-inventory-management"></a>Gestion intelligente des stocks

Le stock correspond aux marchandises que détient un détaillant. La gestion des stocks est essentielle pour s’assurer que le bon produit se trouve au bon endroit au bon moment. Un détaillant doit trouver un équilibre entre les coûts liés au stockage d’un trop grand nombre d’articles et les coûts liés au fait de ne pas avoir suffisamment d’articles en stock pour répondre à la demande.

Les données IoT obtenues à partir d’étiquettes RFID, de balises et de caméras permettent d’améliorer les processus impliqués dans la gestion des stocks. Vous pouvez combiner les données de télémétrie collectées à partir des capteurs et des appareils IoT avec d’autres sources de données, telles que la météo et le trafic dans les systèmes décisionnels basés sur le cloud.

Les avantages de la gestion des stocks intelligente sont les suivants :

* Réduction du risque de rupture de stock des articles et maintien du niveau de service client souhaité 
* Analyse approfondie et insights sur la précision du stock en quasi temps réel
* Outils permettant de déterminer le niveau de stock adapté pour répondre aux commandes des clients

Ce modèle d’application se concentre sur la connectivité des appareils, ainsi que sur la configuration et la gestion des lecteurs RFID et Bluetooth basse énergie.

La capture d’écran suivante montre le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions :

:::image type="content" source="media/overview-iot-central-retail/smart-inventory-management-dashboard.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’informations, consultez le tutoriel [Déployer et découvrir un modèle de gestion intelligente des stocks IoT pour Azure IoT Central](./tutorial-iot-central-smart-inventory-management.md).

## <a name="micro-fulfillment-center"></a>Centre de microtraitement

En raison d’une concurrence de plus en plus accrue, les détaillants se doivent de répondre le plus rapidement possible à la demande. Une nouvelle tendance est apparue pour répondre à la demande croissante des consommateurs. Celle-ci consiste à héberger un stock proche des clients finaux et des magasins dans lesquels ils se rendent.

Le modèle d’application Centre de microtraitement d’IoT Central permet aux créateurs de solutions de superviser et de gérer tous les aspects de leurs centres de traitement entièrement automatisés. Ce modèle comprend des capteurs pour la supervision des conditions ambiantes, ainsi que des robots de transport pour accélérer le processus de développement de la solution. Ces capteurs collectent des signaux significatifs qui peuvent être convertis en insights métier permettant aux détaillants de réduire leurs coûts d’exploitation et d’offrir des expériences appréciées des clients.

Ce modèle d’application vous permet de : 

- Connecter de façon continue différentes sortes de capteurs IoT comme des robots ou des capteurs de supervision des conditions ambiantes à une instance d’application IoT Central
- Superviser et gérer l’intégrité du réseau de capteurs et les éventuels appareils de passerelle de l’environnement
- Créer des règles personnalisées en lien avec les conditions ambiantes au sein d’un centre de traitement afin de déclencher les alertes nécessaires
- Convertir les conditions ambiantes du centre de traitement en insights exploitables par l’équipe de l’entrepôt
- Exporter des insights agrégés dans des applications métier existantes ou nouvelles à l’intention des vendeurs

La capture d’écran suivante montre le tableau de bord prêt à l’emploi du modèle d’application. Le tableau de bord est entièrement personnalisable afin de mieux répondre à vos besoins en solutions :

:::image type="content" source="media/overview-iot-central-retail/MFC-Dashboard.png" alt-text="Présentation d’Azure IoT Retail":::

Pour plus d’informations, consultez le tutoriel [Déployer et parcourir un modèle d’application du Centre de microtraitement](./tutorial-micro-fulfillment-center.md).

## <a name="video-analytics---object-and-motion-detection"></a>Analytique vidéo - Détection d’objets et de mouvements

Le *modèle d’application d’analytique vidéo pour la détection d’objet et de mouvement Azure IoT Central* vous permet d’expérimenter rapidement la manière de déployer, gérer et surveiller une solution qui utilise des caméras de périphérie intelligentes pour la détection d’objet et de mouvement.

L’application d’analytique vidéo utilise un module d’[analytique vidéo en direct](#live-video-analytics) s’exécutant dans IoT Edge. Le module d’analytique vidéo en direct fournit une plateforme vous permettant de créer des applications vidéo intelligentes qui s’étendent à la périphérie et au cloud. Vous pouvez utiliser la plateforme pour améliorer des solutions IoT, telles que l’application d’analytique vidéo, avec la détection d’objet et de mouvement.

Le modèle d’application comprend quatre tableaux de bord d’application :

* **Bien démarrer** fournit des liens vers des ressources pour vous aider à commencer à utiliser le modèle d’application.
* **Tableau de bord de démonstration** fournit une illustration des types d’informations que vous pouvez afficher à partir de vos caméras connectées.
* **(Exemple) Gestion de caméras réelles** utilise des caméras simulées pour montrer comment vous pouvez gérer vos caméras à partir de l’application.
* **(Exemple) Supervision de caméras réelles** utilise des caméras simulées pour montrer comment vous pouvez superviser vos caméras à partir de l’application.

:::image type="content" source="media/overview-iot-central-retail/live-video-analytics.png" alt-text="Présentation d’Azure IoT Retail":::

Pour en savoir plus sur l’architecture de la solution, consultez [Architecture de l’application d’analytique vidéo](architecture-video-analytics.md).

Pour savoir comment déployer la solution, consultez le tutoriel [Créer une application d’analytique vidéo dans Azure IoT Central](tutorial-video-analytics-deploy.md).

### <a name="live-video-analytics"></a>Analytique vidéo en direct

L’[analytique vidéo en direct](https://github.com/Azure/live-video-analytics) fournit une plateforme pour créer des applications vidéo intelligentes qui s’étendent à la périphérie et au cloud. La plateforme offre la possibilité de capturer, d’enregistrer, d’analyser des vidéos en direct et de publier les résultats, tels qu’une analytique vidéo, dans les services Azure. Les services Azure peuvent s’exécuter dans le cloud ou à la périphérie. La plateforme permet d’améliorer des solutions IoT avec une analytique vidéo.

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à créer une solution de vente au détail :

* Pour bien démarrer, consultez le tutoriel [Créer une application d’analytique en magasin dans Azure IoT Central](./tutorial-in-store-analytics-create-app.md) qui explique comment créer une solution à l’aide de l’un des modèles d’application d’analytique en magasin.
* [Déployer et découvrir un modèle d’application logistique connectée](./tutorial-iot-central-connected-logistics.md)
* [Déployer et parcourir un modèle d’application du Centre de distribution numérique](./tutorial-iot-central-digital-distribution-center.md)
* [Déployer et découvrir un modèle de gestion intelligente des stocks IoT pour Azure IoT Central](./tutorial-iot-central-smart-inventory-management.md)
* [Déployer et parcourir un modèle d’application du Centre de microtraitement](./tutorial-micro-fulfillment-center.md)
* [Déployer et découvrir un modèle d’application d’analytique vidéo](./tutorial-video-analytics-deploy.md)
* Pour plus d’informations sur IoT Central, consultez la [présentation d’IoT Central](../core/overview-iot-central.md).
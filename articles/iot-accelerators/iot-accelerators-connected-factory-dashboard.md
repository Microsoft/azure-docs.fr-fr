---
title: Utiliser le tableau de bord Usine connectée - Azure | Microsoft Docs
description: Cet article explique comment utiliser les fonctionnalités du tableau de bord Usine connectée pour surveiller et gérer vos appareils IoT industriels.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 07/10/2018
ms.author: dobett
ms.openlocfilehash: 797bd97940aa49db87b4ca4dd96f140208cd69b6
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011215"
---
# <a name="use-features-in-the-connected-factory-solution-accelerator-dashboard"></a>Utiliser les fonctionnalités du tableau de bord de l’accélérateur de solution Usine connectée

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Le guide de démarrage rapide [Déployer une solution cloud pour gérer mes appareils IoT industriels](quickstart-connected-factory-deploy.md) vous a montré comment naviguer dans le tableau de bord et répondre aux alarmes. Ce guide pratique vous montre d’autres fonctionnalités du tableau de bord, que vous pouvez utiliser pour suivre et gérer vos appareils IoT industriels.

## <a name="apply-filters"></a>Appliquer des filtres

Vous pouvez filtrer les informations affichées dans le tableau de bord dans le panneau **Emplacements des usines** ou dans le panneau **Alarmes** :

1. Cliquez sur le **Entonnoir** pour afficher la liste des filtres disponibles dans le panneau des emplacements d’usine ou le panneau des alarmes.

1. Le panneau des filtres s’affiche :

    [![Filtres de l’accélérateur de solution d’usine connectée](./media/iot-accelerators-connected-factory-dashboard/filterpanel-inline.png)](./media/iot-accelerators-connected-factory-dashboard/filterpanel-expanded.png#lightbox)

1. Choisissez le filtre nécessaire et cliquez sur **Appliquer**. Il est également possible d’entrer du texte libre dans les champs du filtre.

1. Le filtre est ensuite appliqué. L’icône d’entonnoir supplémentaire indique qu’un filtre est appliqué :

    [![Filtre appliqué pour l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/filterapplied-inline.png)](./media/iot-accelerators-connected-factory-dashboard/filterapplied-expanded.png#lightbox)

    > [!NOTE]
    > Un filtre actif n’affecte pas les valeurs affichées pour les taux de rendement global (TRG) et les indicateurs de performance clés (KPI) : il filtre seulement le contenu de la liste.

1. Pour effacer un filtre, cliquez sur l’entonnoir, puis cliquez sur **Effacer** dans le panneau du filtre.

## <a name="browse-an-opc-ua-server"></a>Parcourir un serveur OPC UA

Quand vous déployez l’accélérateur de solution, vous provisionnez automatiquement un ensemble de serveurs OPC UA simulés auxquels vous pouvez accéder via le tableau de bord. Les serveurs simulés vous permettent d’expérimenter plus facilement l’accélérateur de solution sans avoir à déployer des serveurs réels.

1. Cliquez sur **l’icône de navigateur** dans la barre de navigation du tableau de bord :

    [![Navigateur de serveurs de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/browser-inline.png)](./media/iot-accelerators-connected-factory-dashboard/browser-expanded.png#lightbox)

1. Choisissez un des serveurs dans la liste qui montre les serveurs déployés pour vous dans l’accélérateur de solution :

    [![Liste des serveurs de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/serverlist-inline.png)](./media/iot-accelerators-connected-factory-dashboard/serverlist-expanded.png#lightbox)

1. Cliquez sur **Connecter**. Une boîte de dialogue de sécurité s’affiche. Pour la simulation, vous pouvez cliquer sur **Continuer**.

1. Cliquez sur un des nœuds dans l’arborescence de serveur pour le développer. Les nœuds qui publient des données de télémétrie sont indiqués par une coche :

    [![Arborescence des serveurs de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/servertree-inline.png)](./media/iot-accelerators-connected-factory-dashboard/servertree-expanded.png#lightbox)

1. Cliquez avec le bouton droit sur un élément pour lire, écrire, publier ou appeler ce nœud. Les actions disponibles dépendent de vos autorisations et des attributs du nœud. L’option de lecture affiche un panneau de contexte indiquant la valeur du nœud spécifique. L’option d’écriture affiche un panneau de contexte dans lequel vous pouvez entrer une nouvelle valeur. L’option d’appel affiche un nœud dans lequel vous pouvez entrer les paramètres de l’appel.

## <a name="publish-a-node"></a>Publier un nœud

Lorsque vous parcourez un *serveur OPC UA simulé*, vous pouvez également choisir de publier de nouveaux nœuds. Vous pouvez analyser les données de télémétrie de ces nœuds dans la solution. Ces *serveurs OPC UA simulés* vous permettent d’expérimenter plus facilement l’accélérateur de solution sans déployer des appareils réels :

1. Accédez à un nœud de l’arborescence du navigateur de serveur OPC UA que vous souhaitez publier.

1. Cliquez avec le bouton droit sur le nœud. **Cliquez sur** Publier :

    [![Nœud de la publication de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/publishnode-inline.png)](./media/iot-accelerators-connected-factory-dashboard/publishnode-expanded.png#lightbox)

1. Un panneau de contexte vous indiquant que la publication a réussi s’affiche. Le nœud apparaît dans la vue du niveau poste avec une coche :

    [![Réussite de la publication de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/publishsuccess-inline.png)](./media/iot-accelerators-connected-factory-dashboard/publishsuccess-expanded.png#lightbox)

## <a name="command-and-control"></a>Commande et contrôle

L’usine connectée vous permet de commander et de contrôler vos appareils industriels directement à partir du cloud. Vous pouvez utiliser cette fonctionnalité pour répondre aux alarmes générées par l’appareil. Par exemple, vous pouvez envoyer une commande à l’appareil pour ouvrir une soupape de relâchement de la pression. Vous pouvez rechercher les commandes disponibles dans le nœud **StationCommands** dans l’arborescence du navigateur de serveurs OPC UA. Dans ce scénario, vous ouvrez une soupape de décompression sur le poste d’assembly d’une ligne de production à Munich. Pour utiliser la commande et contrôler la fonctionnalité, vous devez disposer du rôle **Administrateur** pour le déploiement de l’accélérateur de solution :

1. Accédez au nœud **StationCommands** dans l’arborescence du navigateur du serveur OPC UA server pour le poste d’assemblage Munich, ligne de production 0.

1. Choisissez la commande que vous souhaitez utiliser. Cliquez avec le bouton droit sur le nœud **OpenPressureReleaseValve**. Cliquez sur **Appeler** :

    [![Commande d’appel de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/callcommand-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callcommand-expanded.png#lightbox)

1. Un panneau de contexte apparaît, vous informant sur la méthode que vous allez appeler et sur les détails des éventuels paramètres. Cliquez sur **Appeler** :

    [![Paramètres d’appel de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/callpanel-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callpanel-expanded.png#lightbox)

1. Le panneau de contexte est mis à jour pour vous informer que l’appel de méthode a réussi. Vous pouvez vérifier que l’appel a réussi en lisant la valeur du nœud de pression mise à jour suite à l’appel.

    [![Réussite d’appel de l’accélérateur de solution d’usine connectée](./media/iot-accelerators-connected-factory-dashboard/callsuccess-inline.png)](./media/iot-accelerators-connected-factory-dashboard/callsuccess-expanded.png#lightbox)

## <a name="behind-the-scenes"></a>Dans les coulisses

Lorsque vous déployez un accélérateur de solution, le processus de déploiement crée plusieurs ressources dans l’abonnement Azure sélectionné. Vous pouvez voir ces ressources dans le [portail](https://portal.azure.com) Azure. Le processus de déploiement crée un **groupe de ressources** avec un nom basé sur celui que vous avez choisi pour votre accélérateur de solution :

[![Groupe de ressources de l’accélérateur de solution Usine connectée](./media/iot-accelerators-connected-factory-dashboard/resourcegroup-inline.png)](./media/iot-accelerators-connected-factory-dashboard/resourcegroup-expanded.png#lightbox)

Vous pouvez afficher les paramètres de chaque ressource en la sélectionnant dans la liste des ressources dans le groupe de ressources.

Vous pouvez aussi voir le code source de l’accélérateur de solution Usine connectée dans le dépôt GitHub [azure-iot-connected-factory](https://github.com/Azure/azure-iot-connected-factory).

Quand vous avez terminé, vous pouvez supprimer l’accélérateur de solution de votre abonnement Azure sur le site [azureiotsolutions.com](https://www.azureiotsolutions.com/Accelerators#dashboard). Ce site vous permet de supprimer facilement toutes les ressources qui ont été configurées lors de la création de l’accélérateur de solution.

> [!NOTE]
> Pour supprimer tout ce qui concerne l’accélérateur de solution, supprimez ce dernier sur le site [azureiotsolutions.com](https://www.azureiotsolutions.com/Accelerators#dashboard). Ne supprimez pas le groupe de ressources dans le portail.

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez déployé un accélérateur de solution opérationnel, vous pouvez poursuivre la prise en main des accélérateurs de solution IoT en lisant les articles suivants :

* [Configurer l’accélérateur de solution Usine connectée](iot-accelerators-connected-factory-configure.md)
* [Autorisations sur le site azureiotsolutions.com](iot-accelerators-permissions.md)

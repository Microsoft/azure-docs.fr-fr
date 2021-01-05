---
title: Déplacer des applications logiques entre des abonnements, des groupes de ressources ou des régions
description: Migrer des applications logiques ou des comptes d’intégration vers d’autres abonnements Azure, des groupes de ressources ou des emplacements (régions)
services: logic-apps
ms.suite: integration
ms.reviewer: logicappspm
ms.topic: conceptual
ms.date: 04/06/2020
ms.openlocfilehash: aca2c51ff14b99ba41b159cf32e59dc861de7a53
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87826209"
---
# <a name="move-logic-app-resources-to-other-azure-resource-groups-regions-or-subscriptions"></a>Déplacer des ressources d’application logique vers d’autres abonnements, régions ou groupes de ressources Azure

Pour migrer votre application logique ou les ressources associées vers un autre abonnement, région ou groupe de ressources Azure, vous disposez de différentes méthodes pour effectuer ces tâches, telles que le portail Azure, Azure PowerShell, Azure CLI et l’API REST. Avant de déplacer des ressources, passez en revue les considérations suivantes : 

* Vous pouvez déplacer uniquement des [types de ressources d’application logique spécifiques](../azure-resource-manager/management/move-support-resources.md#microsoftlogic) entre des groupes de ressources Azure ou des abonnements.

* Vérifiez les [limites](../logic-apps/logic-apps-limits-and-config.md) du nombre de ressources d’application logique que vous pouvez avoir dans votre abonnement Azure et dans chaque région Azure. Ces limites déterminent si vous pouvez déplacer des types de ressources spécifiques lorsque la région reste la même sur plusieurs abonnements ou groupes de ressources. Par exemple, vous ne pouvez avoir qu’un seul compte d’intégration de niveau gratuit pour chaque région Azure dans chaque abonnement Azure.

* Lorsque vous déplacez des ressources, Azure crée de nouveaux ID de ressource. Par conséquent, assurez-vous d’utiliser les nouveaux ID et de mettre à jour tous les scripts ou outils associés aux ressources déplacées.

* Après avoir migré des applications logiques entre les abonnements, les groupes de ressources ou les régions, vous devez recréer ou autoriser à nouveau toutes les connexions qui nécessitent l’authentification ouverte (OAuth).

* Vous pouvez déplacer un [environnement de service d’intégration (ISE, Integration Service Environment)](connect-virtual-network-vnet-isolated-environment-overview.md) uniquement vers un autre groupe de ressources qui se trouve dans la même région Azure ou dans le même abonnement Azure. Vous ne pouvez pas déplacer un environnement ISE vers un groupe de ressources situé dans une autre région Azure ou dans un autre abonnement Azure. De plus, après ce déplacement, vous devez mettre à jour toutes les références à l’environnement ISE dans vos workflows d’application logique, comptes d’intégration, connexions, etc.

## <a name="prerequisites"></a>Prérequis

* Le même abonnement Azure que celui utilisé pour créer l’application logique ou le compte d’intégration que vous souhaitez déplacer

* Autorisations du propriétaire des ressources pour déplacer et configurer les ressources souhaitées. Apprenez-en plus sur le [Contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../role-based-access-control/built-in-roles.md#owner).

<a name="move-subscription"></a>

## <a name="move-resources-between-subscriptions"></a>Déplacer des ressources entre des abonnements

Pour déplacer une ressource, telle qu’une application logique ou un compte d’intégration, vers un autre abonnement Azure, vous pouvez utiliser l’API Portail Azure, Azure PowerShell, Azure CLI ou REST. Ces étapes couvrent les Portail Azure, que vous pouvez utiliser lorsque la région de la ressource reste la même. Pour plus d’informations, consultez la page [Déplacer des ressources vers un nouveau groupe de ressources ou un abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez la ressource d’application logique que vous souhaitez déplacer.

1. Sur la page **Vue d'ensemble** de la ressource, à côté d’**Abonnement**, sélectionnez le lien **Modifier**.

1. Sur la page **Déplacer les ressources**, sélectionnez la ressource d’application logique et toutes les ressources associées que vous souhaitez déplacer.

1. Dans la liste **Abonnement**, sélectionnez l’abonnement de destination.

1. Dans la liste **Groupe de ressources**, sélectionnez le groupe de ressources de destination. Ou, pour créer un groupe de ressources différent, sélectionnez **Créer un nouveau groupe**.

1. Pour confirmer que vous avez compris que tous les scripts ou outils associés aux ressources déplacées ne fonctionneront pas tant que vous ne les aurez pas mis à jour avec les nouveaux ID de ressource, sélectionnez la zone de confirmation, puis sélectionnez **OK**.

<a name="move-resource-group"></a>

## <a name="move-resources-between-resource-groups"></a>Déplacer des ressources entre des groupes de ressources

Pour déplacer une ressource, telle qu’une application logique, un compte d’intégration ou un [environnement de service d’intégration (ISE)](connect-virtual-network-vnet-isolated-environment-overview.md), vers un autre groupe de ressources Azure, vous pouvez utiliser le portail Azure, Azure PowerShell, Azure CLI ou l’API REST. Ces étapes couvrent les Portail Azure, que vous pouvez utiliser lorsque la région de la ressource reste la même. Pour plus d’informations, consultez la page [Déplacer des ressources vers un nouveau groupe de ressources ou un abonnement](../azure-resource-manager/management/move-resource-group-and-subscription.md).

Avant de déplacer réellement des ressources entre des groupes, vous pouvez tester si vous pouvez déplacer votre ressource vers un autre groupe. Pour plus d’informations, voir [Valider votre opération de déplacement](../azure-resource-manager/management/move-resource-group-and-subscription.md#validate-move).

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez la ressource d’application logique que vous souhaitez déplacer.

1. Sur la page **Vue d'ensemble** de la ressource, à côté de **Groupe de ressources**, sélectionnez le lien **Modifier**.

1. Sur la page **Déplacer les ressources**, sélectionnez la ressource d’application logique et toutes les ressources associées que vous souhaitez déplacer.

1. Dans la liste **Groupe de ressources**, sélectionnez le groupe de ressources de destination. Ou, pour créer un groupe de ressources différent, sélectionnez **Créer un nouveau groupe**.

1. Pour confirmer que vous avez compris que tous les scripts ou outils associés aux ressources déplacées ne fonctionneront pas tant que vous ne les aurez pas mis à jour avec les nouveaux ID de ressource, sélectionnez la zone de confirmation, puis sélectionnez **OK**.

<a name="move-location"></a>

## <a name="move-resources-between-regions"></a>Déplacer des ressources entre des régions

Lorsque vous souhaitez déplacer une application logique vers une autre région, vos options dépendent de la façon dont vous avez créé votre application logique. En fonction de l’option que vous choisissez, vous devez recréer ou réautoriser les connexions dans votre application logique.

* Dans le Portail Azure, recréez l’application logique dans la nouvelle région et reconfigurez les paramètres du flux de travail. Pour gagner du temps, vous pouvez copier la définition de flux de travail sous-jacente et les connexions de l’application source vers l’application de destination. Pour afficher le « code » derrière une application logique, dans la barre d’outils du concepteur d'applications logiques, sélectionnez **Mode code**.

* À l’aide de Visual Studio et des outils d’Azure Logic apps pour Visual Studio, vous pouvez [ouvrir et télécharger votre application logique](../logic-apps/manage-logic-apps-with-visual-studio.md) à partir du portail Azure en tant que [modèle de Azure Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md). Ce modèle est principalement prêt pour le déploiement et inclut les définitions de ressources pour votre application logique, y compris le flux de travail lui-même et les connexions. Le modèle déclare également des paramètres pour les valeurs à utiliser au moment du déploiement. De cette façon, vous pouvez modifier plus facilement l’emplacement et la manière dont vous déployez l’application logique, en fonction de vos besoins. Pour spécifier l’emplacement et d’autres informations nécessaires au déploiement, vous pouvez utiliser un fichier de paramètres distinct.

* Si vous avez créé et déployé votre application logique à l’aide des outils d’intégration continue (CI) et de livraison continue (CD), tels que les Azure Pipelines dans Azure DevOps, vous pouvez déployer votre application dans une autre région à l’aide de ces outils.

Pour plus d’informations sur les modèles de déploiement pour les applications logiques, consultez les rubriques suivantes :

* [Vue d’ensemble : Automatiser le déploiement pour le service Azure Logic Apps à l’aide de modèles Resource Manager](../logic-apps/logic-apps-azure-resource-manager-templates-overview.md)
* [Rechercher, ouvrir et télécharger votre application logique à partir du Portail Azure dans Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md)
* [Créer des modèles Azure Resource Manager pour Azure Logic Apps](../logic-apps/logic-apps-create-azure-resource-manager-templates.md)
* [Déployer des modèles Azure Resource Manager pour Azure Logic Apps](../logic-apps/logic-apps-deploy-azure-resource-manager-templates.md)

### <a name="related-resources"></a>Ressources associées

Certaines ressources Azure, telles que les ressources de passerelle de données locales dans Azure, peuvent exister dans une région qui diffère des applications logiques qui utilisent ces ressources. Toutefois, d’autres ressources Azure, telles que les comptes d’intégration liés, doivent exister dans la même région que vos applications logiques. En fonction de votre scénario, assurez-vous que vos applications logiques peuvent accéder aux ressources que vos applications s'attendent à trouver dans la même région.

Par exemple, pour lier une application logique à un compte d’intégration, les deux ressources doivent exister dans la même région. Dans les scénarios tels que la récupération d’urgence, vous souhaitez généralement que les comptes d’intégration possèdent la même configuration et les mêmes artefacts. Dans d’autres scénarios, vous pouvez avoir besoin de comptes d’intégration avec différentes configurations et artefacts.

Les connecteurs personnalisés dans Azure Logic Apps sont visibles pour les auteurs des connecteurs et les utilisateurs qui ont le même abonnement Azure et le même client Azure Active Directory. Ces connecteurs sont disponibles dans la même région que celle où les applications logiques sont déployées. Pour plus d’informations, consultez [Share a custom connector in your organization](/connectors/custom-connectors/share) (Partager un connecteur personnalisé dans votre organisation).

Le modèle obtenu à partir de Visual Studio comprend uniquement les définitions de ressource pour votre application logique et ses connexions. Par conséquent, si votre application logique utilise d’autres ressources, par exemple, un compte d’intégration et des artefacts B2B, tels que des partenaires, des contrats et des schémas, vous devez exporter le modèle de ce compte d’intégration à l’aide du Portail Azure. Ce modèle comprend les définitions de ressource pour le compte d’intégration et les artefacts. Toutefois, le modèle n’est pas entièrement paramétré. Par conséquent, vous devez paramétrer manuellement les valeurs que vous souhaitez utiliser pour le déploiement.

### <a name="export-templates-for-integration-accounts"></a>Exporter des modèles pour les comptes d’intégration

1. Dans le [portail Azure](https://portal.azure.com), recherchez et ouvrez votre compte d’intégration.

1. Dans le menu du compte d’intégration, sous **Paramètres**, sélectionnez **Modèle d’exportation**.

1. Dans la barre d’outils, sélectionnez **Télécharger**, puis enregistrez le modèle.

1. Ouvrez et modifiez le modèle pour paramétrer les valeurs nécessaires au déploiement.

## <a name="next-steps"></a>Étapes suivantes

[Déplacer des ressources Azure vers de nouveaux groupes de ressources ou abonnements](../azure-resource-manager/management/move-resource-group-and-subscription.md)

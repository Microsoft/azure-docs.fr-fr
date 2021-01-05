---
title: Activer le suivi des modifications et l’inventaire d’Azure Automation à partir d’un compte Automation
description: Cet article explique comment activer le suivi des modifications et l’inventaire à partir d’un compte Automation
services: automation
ms.subservice: change-inventory-management
ms.date: 10/14/2020
ms.topic: conceptual
ms.openlocfilehash: 50188ad5fea0ee34a6896f0045e3bbcbfb553aaa
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92677296"
---
# <a name="enable-change-tracking-and-inventory-from-an-automation-account"></a>Activer Change Tracking et Inventory à partir d’un compte Automation

Cet article explique comment vous pouvez utiliser votre compte Automation pour activer [Suivi des modifications et inventaire](overview.md) pour les machines virtuelles dans votre environnement. Pour activer des machines virtuelles Azure à grande échelle, vous devez activer une machine virtuelle existante à l’aide de la fonctionnalité Suivi des modifications et inventaire.

> [!NOTE]
> Lors de l’activation de la fonctionnalité Suivi des modifications et inventaire, seules certaines régions sont prises en charge pour la liaison d’un espace de travail Log Analytics et d’un compte Automation. Pour obtenir la liste des paires de mappages prises en charge, consultez [Mappage de régions pour un compte Automation et un espace de travail Log Analytics](../how-to/region-mappings.md).

## <a name="prerequisites"></a>Prérequis

* Abonnement Azure. Si vous n’avez pas encore d’abonnement, vous pouvez [activer vos avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer [un compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Compte Automation](../index.yml) pour gérer les machines.
* Une [machine virtuelle](../../virtual-machines/windows/quick-create-portal.md).

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à Azure sur https://portal.azure.com.

## <a name="enable-change-tracking-and-inventory"></a>Activer Change Tracking et Inventory

1. Accédez à votre compte Automation et sélectionnez **Inventorier** ou **Suivi des modifications** sous **Gestion de la configuration**.

2. Choisissez l’espace de travail Log Analytics et un compte Automation, puis cliquez sur **Activer** pour activer la fonctionnalité Suivi des modifications et inventaire. La configuration prend jusqu’à 15 minutes.

    ![Activer Change Tracking et Inventory](media/enable-from-automation-account/enable-feature.png)

## <a name="enable-azure-vms"></a>Activer les machines virtuelles Azure

1. Dans votre compte Automation, sélectionnez **Inventaire** ou **Suivi des modifications** sous **Gestion de la configuration**.

2. Cliquez sur **+ Ajouter des machines virtuelles Azure** et sélectionnez une ou plusieurs machines virtuelles dans la liste. Les machines virtuelles qui ne peuvent pas être activées sont grisées et ne peuvent pas être sélectionnées. Les machines virtuelles Azure peuvent exister dans n’importe quelle région, quel que soit l’emplacement de votre compte Automation. 

3. Cliquez sur **Activer** pour ajouter les machines virtuelles sélectionnées à la recherche enregistrée de groupe d’ordinateurs pour la fonctionnalité. Pour plus d’informations, consultez [Limiter l’étendue du déploiement de Change Tracking et Inventory](manage-scope-configurations.md).

      [ ![Activer les machines virtuelles Azure](./media/enable-from-automation-account/enable-azure-vms.png)](./media/enable-from-automation-account/enable-azure-vms-expanded.png#lightbox)

## <a name="enable-non-azure-vms"></a>Activer les machines virtuelles non-Azure

Les machines qui ne sont pas dans Azure doivent être ajoutées manuellement. Nous vous recommandons d’installer l’agent Log Analytics pour Windows ou Linux en connectant d’abord votre machine à des [serveurs avec Azure Arc](../../azure-arc/servers/overview.md), puis en utilisant Azure Policy pour attribuer la stratégie intégrée [Déployer l’agent Log Analytics sur des machines Azure Arc *Linux* ou *Windows*](../../governance/policy/samples/built-in-policies.md#monitoring). Si vous envisagez également de surveiller les machines avec Azure Monitor pour machines virtuelles, utilisez plutôt l’initiative [Activer Azure Monitor pour machines virtuelles](../../governance/policy/samples/built-in-initiatives.md#monitoring).

1. Dans votre compte Automation, sélectionnez **Inventaire** ou **Suivi des modifications** sous **Gestion de la configuration**.

2. Cliquez sur **Ajouter une machine virtuelle non-Azure**. Une nouvelle fenêtre de navigateur s’ouvre et présente des [instructions d’installation et de configuration de l’agent Log Analytics pour Windows](../../azure-monitor/platform/log-analytics-agent.md) afin de permettre à la machine de commencer à rapporter les opérations de la fonctionnalité Suivi des modifications et inventaire. Si vous activez une machine actuellement gérée par Operations Manager, l’installation d’un nouvel agent n’est pas nécessaire et les informations de l’espace de travail sont entrées dans l’agent existant.

## <a name="enable-machines-in-the-workspace"></a>Activer des machines dans l’espace de travail

Les machines installées manuellement ou les machines déjà connectées à votre espace de travail doivent être ajoutées à Azure Automation pour que la solution Suivi des modifications et inventaire soit activée.

1. Dans votre compte Automation, sélectionnez **Inventaire** ou **Suivi des modifications** sous **Gestion de la configuration**.

2. Sélectionnez **Gérer des machines**. L’option **Gérer les machines** peut être grisée si vous avez précédemment choisi l’option **Activer sur toutes les machines disponibles et futures**

    ![Recherches enregistrées](media/enable-from-automation-account/manage-machines.png)

3. Pour activer le Suivi des modifications et inventaire sur toutes les machines disponibles, sélectionnez **Activer sur toutes les machines disponibles** sur la page **Gérer les machines** . Cette action désactive le contrôle pour ajouter des machines individuellement et ajoute toutes les machines qui sont en rapport avec l’espace de travail à la requête de recherche enregistrée de groupe d’ordinateurs. Quand elle est sélectionnée, cette action désactive l’option **Gérer des machines**.

4. Pour activer la fonctionnalité sur toutes les machines disponibles et les futures machines, sélectionnez **Activer sur toutes les machines disponibles et futures**. Cette option supprime les recherches enregistrées et les configurations d’étendue de l’espace de travail et ouvre la fonctionnalité pour toutes les machines Azure et non-Azure qui envoient des informations à l’espace de travail. Quand cette option est sélectionnée, le bouton **Gérer des machines** est désactivé définitivement, car il n’existe plus aucune configuration d’étendue.

    > [!NOTE]
    > Comme cette option supprime la recherche enregistrée et la configuration d’étendue dans Log Analytics, il est important de supprimer tous les verrous de suppression sur l’espace de travail Log Analytics avant de sélectionner cette option. Si vous ne le faites pas, l’option ne parviendra pas à supprimer les configurations et vous devrez les supprimer manuellement.

5. Le cas échéant, vous pouvez rajouter la configuration d’étendue en ajoutant à nouveau la recherche enregistrée initiale. Pour plus d’informations, consultez [Limiter l’étendue du déploiement de Change Tracking et Inventory](manage-scope-configurations.md).

6. Pour activer la fonctionnalité sur une ou plusieurs machines, sélectionnez **Activer sur les machines sélectionnées** , puis cliquez sur **Ajouter** en regard de chaque machine à activer pour la fonctionnalité. Cette tâche ajoute le nom des machines sélectionnées à la requête de recherche enregistrée de groupe d’ordinateurs pour la fonctionnalité.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur l’utilisation de la fonctionnalité, consultez [Gérer le suivi des modifications](manage-change-tracking.md) et [Gérer l’inventaire](manage-inventory-vms.md).

* Pour résoudre des problèmes généraux liés à la fonctionnalité, consultez [Résoudre les problèmes rencontrés avec la fonctionnalité Suivi des modifications et inventaire](../troubleshoot/change-tracking.md).

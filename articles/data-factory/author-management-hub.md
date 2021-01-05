---
title: Hub de gestion
description: Gérez vos connexions, la configuration du contrôle de code source et les propriétés de création globales dans le hub de gestion d’Azure Data Factory
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: dcstwh
ms.author: weetok
manager: anandsub
ms.date: 06/02/2020
ms.openlocfilehash: aa1246a65dc6971107abe35f4a418b0bbb05641b
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96486128"
---
# <a name="management-hub-in-azure-data-factory"></a>Hub de gestion dans Azure Data Factory

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Le hub de gestion, accessible via l’onglet *Gérer* dans l’expérience utilisateur Azure Data Factory, est un portail qui héberge des actions de gestion globale pour votre fabrique de données. Ici, vous pouvez gérer vos connexions aux banques de données, aux calculs externes, à la configuration du contrôle de code source et aux paramètres de déclencheur.

## <a name="manage-connections"></a>Gérer des connexions

### <a name="linked-services"></a>Services liés

Les services liés définissent les informations de connexion pour Azure Data Factory pour se connecter à des magasins de données et à des environnements de calcul externes. Pour plus d’informations, consultez [Concepts des services liés](concepts-linked-services.md). La création, la modification et la suppression de services liés s’effectuent dans le hub de gestion.

![Gérer des services liés](media/author-management-hub/management-hub-linked-services.png)

### <a name="integration-runtimes"></a>Runtimes d’intégration

Un runtime d’intégration est une infrastructure de calcul qu’Azure Data Factory utilise pour fournir des fonctionnalités d’intégration de données dans différents environnements réseau. Pour plus d’informations, consultez [Concepts de runtime d’intégration](concepts-integration-runtime.md). Le hub de gestion vous permet de créer, supprimer et surveiller vos runtimes d’intégration.

![Runtimes d’intégration](media/author-management-hub/management-hub-integration-runtime.png)

## <a name="manage-source-control"></a>Gérer le contrôle de code source

### <a name="git-configuration"></a>Configuration Git

Affichez et modifiez les paramètres configurés de votre référentiel git dans le hub de gestion. Pour plus d’informations, découvrez le [contrôle de code source dans Azure Data Factory](source-control.md).

![Gérer le dépôt git](media/author-management-hub/management-hub-git.png)

### <a name="parameterization-template"></a>Modèle de paramétrage

Pour remplacer les paramètres modèle Resource Manager générés lors de la publication à partir de la branche de collaboration, vous pouvez générer ou modifier un fichier de paramètres personnalisés. Pour plus d’informations, découvrez comment [utiliser des paramètres personnalisés dans le modèle Resource Manager](continuous-integration-deployment.md#use-custom-parameters-with-the-resource-manager-template). Le modèle de paramétrage est disponible uniquement lorsque vous travaillez dans un dépôt git. Si le fichier *arm-template-parameters-definition.json* n’existe pas dans la branche de travail, la modification du modèle par défaut a pour effet de le générer.

![Gérer les paramètres personnalisés](media/author-management-hub/management-hub-custom-parameters.png)

## <a name="manage-authoring"></a>Gérer la création

### <a name="triggers"></a>Déclencheurs

Les déclencheurs déterminent à quel moment l’exécution d’un pipeline doit être lancée. Actuellement, les déclencheurs peuvent figurer dans une planification d’horloge, opérer à intervalles réguliers ou dépendre d’un événement. Pour plus d’informations, découvrez comme [déclencher une exécution](concepts-pipeline-execution-triggers.md#trigger-execution). Le hub de gestion vous permet de créer, modifier, supprimer ou afficher l’état actuel d’un déclencheur.

![Capture d’écran montrant où créer, modifier, supprimer ou afficher l’état actuel d’un déclencheur.](media/author-management-hub/management-hub-triggers.png)

### <a name="global-parameters"></a>Paramètres globaux

Les paramètres globaux représentent des constantes d’une fabrique de données, qui peuvent être consommées par un pipeline dans n’importe quelle expression. Pour plus d’informations, consultez [Paramètres globaux](author-global-parameters.md).

![Créer des paramètres globaux](media/author-global-parameters/create-global-parameter-3.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [configurer un dépôt git](source-control.md) pour votre ADF



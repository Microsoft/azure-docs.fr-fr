---
title: Création visuelle
description: Découvrez comment utiliser la création visuelle dans Azure Data Factory
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
author: dcstwh
ms.author: weetok
ms.reviewer: ''
manager: anandsub
ms.date: 09/08/2020
ms.openlocfilehash: c3204d33cac9f0944ec2ea2cf43be65c7afa46df
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500136"
---
# <a name="visual-authoring-in-azure-data-factory"></a>Création visuelle dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

L’expérience utilisateur Azure Data Factory vous permet de créer et de déployer visuellement des ressources dans votre fabrique de données sans avoir à écrire du code. Vous pouvez glisser-déposer des activités sur le canevas d’un pipeline, effectuer des séries de tests, déboguer de manière itérative, mais aussi déployer et surveiller les exécutions de votre pipeline.

Actuellement, l’expérience utilisateur Azure Data Factory est prise en charge seulement dans Microsoft Edge et Google Chrome.

## <a name="authoring-canvas"></a>Zone de travail de création

Pour ouvrir la **zone de travail de création**, cliquez sur l’icône de crayon. 

![Zone de travail de création](media/author-visually/authoring-canvas.png)

Ici, vous créez les pipelines, les activités, les jeux de données, les services liés, les flux de données, les déclencheurs et les runtimes d’intégration qui composent votre fabrique. Pour commencer à créer un pipeline à l’aide de la zone de travail de création, consultez [Copier des données à l’aide de l’activité de copie](tutorial-copy-data-portal.md). 

L’expérience de création visuelle par défaut fonctionne directement avec le service Data Factory. L’intégration GitHub ou Azure Repos Git est également prise en charge afin d’autoriser le contrôle du code source et la collaboration pour les utiliser sur vos pipelines Data Factory. Pour en savoir plus sur les différences entre ces expériences de création, consultez [Contrôle de code source dans Azure Data Factory](source-control.md).

### <a name="properties-pane"></a>Propriétés, volet

Pour des ressources de niveau supérieur, telles que des pipelines, des jeux de données et des flux de données, les propriétés de haut niveau sont modifiables dans le volet Propriétés sur le côté droit de la zone de dessin. Le volet Propriétés contient des propriétés telles que le nom, la description, les annotations et d’autres propriétés de haut niveau. Les sous-ressources telles que les activités de pipeline et les transformations de flux de données sont modifiées via le volet en bas de la zone de dessin. 

![Propriétés, volet](media/author-visually/properties-pane.png)

Par défaut, le volet des propriétés s’ouvre uniquement lors de la création de ressource. Pour le modifier, cliquez sur l’icône du volet de propriétés située dans l’angle supérieur droit de la zone de dessin.

### <a name="related-resources"></a>Ressources associées

Dans le volet Propriétés, vous pouvez voir quelles sont les ressources qui dépendent de la ressource sélectionnée en sélectionnant l’onglet **Associées**. Toutes les ressources qui font référence à la ressource actuelle seront répertoriées ici.

![Ressources associées](media/author-visually/related-resources.png)

Par exemple, dans l’image ci-dessus, un pipeline et deux flux de données utilisent le jeu de données actuellement sélectionné.

## <a name="management-hub"></a>Hub de gestion

Le hub de gestion, accessible via l’onglet *Gérer* dans l’expérience utilisateur Azure Data Factory, est un portail qui héberge des actions de gestion globale pour votre fabrique de données. Ici, vous pouvez gérer vos connexions aux banques de données, aux calculs externes, à la configuration du contrôle de code source et aux paramètres de déclencheur. Pour plus d’informations, consultez les capacités du [hub de gestion](author-management-hub.md).

![Gérer des services liés](media/author-management-hub/management-hub-linked-services.png)

## <a name="expressions-and-functions"></a>Expressions et fonctions

Les expressions et les fonctions peuvent être utilisées à la place des valeurs statiques pour spécifier de nombreuses propriétés dans Azure Data Factory.

Pour spécifier une expression pour une valeur de propriété, sélectionnez **Ajouter du contenu dynamique** ou cliquez sur **Alt+P** tout en vous concentrant sur le champ.

![Ajouter du contenu dynamique](media/author-visually/dynamic-content-1.png)

Vous ouvrez ainsi le **Générateur d’expressions Data Factory**, où vous pouvez générer des expressions à partir des variables système prises en charge, de la sortie de l’activité, des fonctions et des variables ou paramètres spécifiés par l’utilisateur. 

![Générateur d’expressions](media/author-visually/dynamic-content-2.png)

Pour plus d’informations sur le langage d’expression, consultez [Expressions et fonctions dans Azure Data Factory](control-flow-expression-language-functions.md).

## <a name="provide-feedback"></a>Fournir des commentaires

Sélectionnez **Feedback** (Commentaire) pour donner votre avis sur les fonctionnalités ou informer Microsoft de problèmes avec l’outil :

![Commentaires](media/author-visually/provide-feedback.png)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur la surveillance et la gestion des pipelines, consultez [Surveiller et gérer les pipelines par programmation](monitor-programmatically.md).

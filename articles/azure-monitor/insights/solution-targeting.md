---
title: Ciblage des solutions de supervision dans Azure Monitor | Microsoft Docs
description: Le ciblage des solutions de supervision vous permet de limiter les solutions de supervision à un ensemble spécifique d'agents.  Cet article décrit comment créer une configuration d’étendue et l’appliquer à une solution.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 04/27/2017
ms.openlocfilehash: 5cecf24f4ba086feba5ab87b5752fd665c540dff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86498676"
---
# <a name="targeting-monitoring-solutions-in-azure-monitor-preview"></a>Ciblage des solutions de supervision dans Azure Monitor (préversion)
Lorsque vous ajoutez une solution de supervision à votre abonnement, par défaut, celle-ci est automatiquement déployée sur tous les agents Windows et Linux connectés à votre espace de travail Log Analytics.  Vous pouvez gérer vos coûts et limiter la quantité de données collectées pour une solution en limitant celle-ci à un ensemble spécifique d’agents.  Cet article explique comment utiliser le **ciblage de solution**, une fonctionnalité qui permet d’appliquer une étendue à vos solutions.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="how-to-target-a-solution"></a>Ciblage d’une solution
Il existe trois étapes au ciblage d’une solution, comme décrit dans les sections suivantes. 


### <a name="1-create-a-computer-group"></a>1. Créer un groupe d’ordinateurs
Vous spécifiez les ordinateurs que vous souhaitez inclure dans une étendue en créant un [groupe d'ordinateurs](../platform/computer-groups.md) dans Azure Monitor.  Le groupe d'ordinateurs peut être basé sur une requête de journal ou importé à partir d'autres sources comme les groupes WSUS ou Active Directory. Comme [décrit ci-dessous](#solutions-and-agents-that-cant-be-targeted), seuls les ordinateurs qui sont directement connectés à Azure Monitor figurent dans l'étendue.

Une fois que le groupe d’ordinateurs est créé dans votre espace de travail, vous devez l’inclure dans une configuration d’étendue qui peut être appliquée à une ou plusieurs solutions.
 
 
### <a name="2-create-a-scope-configuration"></a>2. Création d’une configuration d’étendue
 Une **configuration d’étendue** inclut un ou plusieurs groupes d’ordinateurs et peut être appliquée à une ou plusieurs solutions. 
 
 Créez une configuration d’étendue à l’aide de la procédure suivante.  

 1. Sur le portail Azure, accédez à **Espaces de travail Log Analytics** et sélectionnez votre espace de travail.
 2. Dans les propriétés de l’espace de travail sous **Sources de données de l’espace de travail**, sélectionnez **Configurations d’étendue**.
 3. Cliquez sur **Ajouter** pour créer une configuration d’étendue.
 4. Entrez un **nom** pour la configuration d’étendue.
 5. Cliquez sur **Sélectionner des groupes d’ordinateurs**.
 6. Sélectionnez le groupe d’ordinateurs que vous avez créé et éventuellement d’autres groupes à ajouter à la configuration.  Cliquez sur **Sélectionner**.  
 6. Cliquez sur **OK** pour créer la configuration d’étendue. 


### <a name="3-apply-the-scope-configuration-to-a-solution"></a>3. Appliquez la configuration d’étendue à une solution.
Une fois que vous avez une configuration d’étendue, vous pouvez l’appliquer à une ou plusieurs solutions.  Notez qu’une seule configuration d’étendue peut être utilisée avec plusieurs solutions, mais chaque solution ne peut utiliser qu’une seule configuration d’étendue.

Appliquez une configuration d’étendue à l’aide de la procédure suivante.  

 1. Sur le portail Azure, accédez à **Espaces de travail Log Analytics** et sélectionnez votre espace de travail.
 2. Dans les propriétés de l’espace de travail, sélectionnez **Solutions**.
 3. Cliquez sur la solution à laquelle appliquer une étendue.
 4. Dans les propriétés de la solution sous **Sources de données de l’espace de travail**, sélectionnez **Ciblage de solution**.  Si l’option n’est pas disponible, alors [cette solution ne peut pas être ciblée](#solutions-and-agents-that-cant-be-targeted).
 5. Cliquez sur **Ajouter une configuration d’étendue**.  Si une configuration est déjà appliquée à cette solution, alors cette option n’est pas disponible.  Vous devez supprimer la configuration existante avant d’en ajouter une autre.
 6. Cliquez sur la configuration d’étendue que vous avez créée.
 7. Consultez **l’état** de la configuration pour vous assurer qu’il s’affiche en tant que **Réussi**.  Si l’état indique une erreur, cliquez sur les points de suspension à droite de la configuration et sélectionnez **Modifier la configuration d’étendue** pour apporter des modifications.

## <a name="solutions-and-agents-that-cant-be-targeted"></a>Solutions et agents ne pouvant pas être ciblés
Voici les critères pour les agents et les solutions qui ne peuvent pas être utilisés avec le ciblage de solution.

- Le ciblage de solution s’applique uniquement aux solutions avec déploiement sur les agents.
- Le ciblage de solution s’applique uniquement aux solutions fournies par Microsoft.  Il ne s’applique pas aux solutions [créées par vous-même ou vos partenaires](./solutions.md).
- Vous pouvez uniquement filtrer les agents qui se connectent directement à Azure Monitor.  Les solutions se déploient automatiquement sur les agents (qu’ils soient inclus dans une configuration d’étendue ou non) qui font partie d’un groupe d’administration Operations Manager connecté.

### <a name="exceptions"></a>Exceptions
Le ciblage de solution ne peut pas être utilisé avec les solutions suivantes, même si elles sont conformes aux critères indiqués.

- Évaluation de l’intégrité de l’agent

## <a name="next-steps"></a>Étapes suivantes
- Découvrez-en plus sur les solutions de supervision, notamment sur les solutions qui peuvent être installées dans votre environnement en consultant [Ajout de solutions de supervision Azure Log Analytics dans votre espace de travail](solutions.md).
- Découvrez-en plus sur la création de groupes d'ordinateurs en consultant [Groupes d'ordinateurs dans les requêtes de journal Azure Monitor](../platform/computer-groups.md).

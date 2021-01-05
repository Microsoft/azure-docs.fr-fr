---
title: Concepts et définitions de commandes personnalisées - service Speech
titleSuffix: Azure Cognitive Services
description: Dans cet article, vous découvrirez les concepts et définitions des applications Commandes personnalisées.
services: cognitive-services
author: singhsaumya
manager: yetian
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 06/18/2020
ms.author: sausin
ms.openlocfilehash: 98510132b2341736664dfafa52e9567df95652be
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94561095"
---
# <a name="custom-commands-concepts-and-definitions"></a>Concepts et définitions de commandes personnalisées

Cet article sert de référence pour les concepts et définitions des applications Commandes personnalisées.

## <a name="commands-configuration"></a>Configuration des commandes
Les commandes représentent les blocs de construction de base d’une application Commandes personnalisées. Une commande est un ensemble de configurations requises pour effectuer une tâche spécifique définie par un utilisateur.

### <a name="example-sentences"></a>Exemples de phrases
Les exemples d’énoncés sont des exemples définis que l’utilisateur peut prononcer pour déclencher une commande particulière. Vous devez fournir uniquement un échantillon d’énoncés et non une liste exhaustive. 

### <a name="parameters"></a>Paramètres
Les paramètres constituent des informations requises par les commandes pour effectuer une tâche. Dans des scénarios complexes, les paramètres peuvent également servir à définir des conditions qui déclenchent des actions personnalisées.

### <a name="completion-rules"></a>Règles d'exécution
Les règles d’exécution sont une série de règles à exécuter une fois la commande prête à être remplie, par exemple, lorsque toutes les conditions des règles sont satisfaites.

### <a name="interaction-rules"></a>Règles d'interaction
Les règles d’interaction sont des règles supplémentaires pour traiter des situations plus spécifiques ou complexes. Vous pouvez ajouter d’autres validations ou configurer des fonctionnalités avancées telles que des confirmations ou une correction en une étape. Vous pouvez également créer vos propres règles d’interaction personnalisées.

## <a name="parameters-configuration"></a>Configuration des paramètres

Les paramètres constituent des informations requises par les commandes pour effectuer une tâche. Dans des scénarios complexes, les paramètres peuvent également servir à définir des conditions qui déclenchent des actions personnalisées.

### <a name="name"></a>Nom
Un paramètre est identifié par la propriété de nom. Vous devez toujours attribuer un nom descriptif à un paramètre. Un paramètre peut être référencé dans différentes sections, par exemple, lors de la construction de conditions, de réponses vocales ou d’autres actions.
 
### <a name="isglobal"></a>IsGlobal
Cette case à cocher indique si la portée de ce paramètre est partagée entre toutes les commandes dans l’application. Si un paramètre est global, sa valeur peut éventuellement être fournie à partir de n’importe quelle portée de commande. Une fois qu’une valeur est attribuée, elle peut être référencée à partir de l’une des commandes. 

### <a name="required"></a>Obligatoire
Cette case à cocher indique si une valeur de ce paramètre est requise pour exécuter la commande. Vous devez configurer les réponses pour inviter l’utilisateur à fournir une valeur si un paramètre est marqué comme étant obligatoire.

Notez que si vous avez configuré un **paramètre obligatoire** pour avoir une **valeur par défaut**, le système demande toujours explicitement la valeur du paramètre.

### <a name="type"></a>Type
Les commandes personnalisées prennent en charge les types de paramètres suivants :

* DateTime
* Geography
* Number
* String

Tous ces types de paramètre, à l’exception de Geography, prennent en charge la configuration de la valeur par défaut, que vous pouvez configurer à partir du portail.

### <a name="configuration"></a>Configuration
Configuration est une propriété de paramètre définie uniquement pour le type Chaîne. Les valeurs suivantes sont admises :

* **Aucun**.
* **Accepter une entrée complète** : lorsqu’il est activé, un paramètre accepte tout énoncé d’entrée. Cette option est utile lorsque l’utilisateur a besoin d’un paramètre avec l’énoncé complet. Par exemple, les adresses postales.
* **Accepter les valeurs d’entrée prédéfinies d’un catalogue externe** : cette valeur est utilisée pour configurer un paramètre qui peut accepter une grande variété de valeurs. Un catalogue de ventes en est un exemple. Dans ce cas, le catalogue est hébergé sur un point de terminaison web externe et peut être configuré indépendamment.
* **Accepter les valeurs d’entrée prédéfinies du catalogue interne** : cette valeur est utilisée pour configurer un paramètre qui peut accepter quelques valeurs. Dans ce cas, les valeurs doivent être configurées dans Speech Studio.


### <a name="validation"></a>Validation
Les validations sont des constructions applicables à certains types de paramètres qui vous permettent de configurer des contraintes sur la valeur d’un paramètre. Actuellement, les commandes personnalisées prennent en charge les validations sur les types de paramètres suivants :

* DateTime
* Number

## <a name="rules-configuration"></a>Configuration de règles
Une règle dans Commandes personnalisées est définie par un ensemble de *conditions* qui, lorsqu’elles sont remplies, exécutent un ensemble d’*actions*. Les règles vous permettent également de configurer l’*état après exécution* et des *attentes* pour le tour suivant.

### <a name="types"></a>Types
Les commandes personnalisées prennent en charge les catégories de règles suivantes :

* **Règles d’exécution** : ces règles doivent être exécutées lors de l’exécution de la commande. Toutes les règles configurées dans cette section pour lesquelles les conditions sont vraies seront exécutées. 
* **Règles d’interaction** : ces règles peuvent être utilisées pour configurer d’autres validations personnalisées, des confirmations et une correction en une étape ou pour appliquer toute autre logique de dialogue personnalisée. Les règles d’interaction sont évaluées à chaque étape de traitement et permettent de déclencher des règles d’exécution.

Les différentes actions configurées dans le cadre d’une règle sont exécutées dans l’ordre dans lequel elles apparaissent dans le portail de création.

### <a name="conditions"></a>Conditions
Les conditions sont les exigences qui doivent être satisfaites pour qu’une règle s’exécute. Les conditions de règles se présente sous plusieurs formes :

* **La valeur du paramètre est égale à** : la valeur du paramètre configuré est égale à une valeur spécifique.
* **Aucune valeur de paramètre** : les paramètres configurés ne doivent pas avoir de valeur.
* **Paramètres obligatoires** : le paramètre configuré a une valeur.
* **Tous les paramètres obligatoires** : tous les paramètres qui ont été marqués comme obligatoires ont une valeur.
* **Paramètres mis à jour** : une ou plusieurs valeurs de paramètre ont été mises à jour suite au traitement de l’entrée actuelle (énoncé ou activité).
* **Confirmation réussie** : l’énoncé ou l’activité d’entrée est une confirmation réussie (oui).
* **Confirmation refusée** : l’énoncé ou l’activité d’entrée est une confirmation qui a échoué (non).
* **La commande précédente doit être mise à jour** : cette condition est utilisée dans les instances où vous souhaitez intercepter une confirmation négative avec une mise à jour. En arrière-plan, cette condition est configurée dans le cas suivant : le moteur de dialogue détecte une confirmation négative, l’intention est identique à celle du tour précédent, et l’utilisateur a répondu avec une mise à jour.

### <a name="actions"></a>Actions
* **Envoyer une réponse vocale** : envoyer une réponse vocale au client.
* **Mettre à jour la valeur de paramètre** : mettre à jour la valeur d’un paramètre de commande avec une valeur spécifiée.
* **Effacer la valeur de paramètre** : effacer la valeur du paramètre de commande.
* **Appeler un point de terminaison web** : appeler un point de terminaison web.
* **Envoyer l’activité au client** : envoyer une activité personnalisée au client.

### <a name="expectations"></a>Attentes
Les attentes servent à configurer des conseils pour le traitement de l’entrée utilisateur suivante. Les types suivants sont pris en charge :

* **Attente de confirmation de l’utilisateur** : cette attente spécifie que l’application attend une confirmation (oui/non) pour l’entrée utilisateur suivante.
* **Attente d’entrée de paramètre de l’utilisateur** : cette attente spécifie un ou plusieurs paramètres de commande que l’application attend de l’entrée utilisateur.

### <a name="post-execution-state"></a>État après exécution
L’état après exécution désigne l’état de la boîte de dialogue après traitement de l’entrée actuelle (énoncé ou activité). Défini selon l’un des types suivants :

* **Conserver l’état actuel** : conserver l’état actuel uniquement.
* **Terminer la commande** : terminer la commande sans traiter aucune autre règle de la commande.
* **Exécuter les règles d’exécution** : exécuter toutes les règles d’exécution valides.
* **Attente d’entrée de l’utilisateur** : attendre l’entrée utilisateur suivante.



## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Voir des exemples sur GitHub](https://aka.ms/speech/cc-samples)

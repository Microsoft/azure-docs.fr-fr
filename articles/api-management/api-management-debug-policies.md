---
title: Déboguer les stratégies de Gestion des API Azure dans Visual Studio Code | Microsoft Docs
description: Apprenez à déboguer les stratégies de Gestion des API Azure à l'aide de l'extension Visual Studio Code de Gestion des API Azure
services: api-management
documentationcenter: ''
author: miaojiang
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 09/22/2020
ms.author: apimpm
ms.openlocfilehash: 4eb32243df219d721d7baae80984c45d0fc4cf25
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91338624"
---
# <a name="debug-azure-api-management-policies-in-visual-studio-code"></a>Déboguer les stratégies de Gestion des API Azure dans Visual Studio Code

Les [stratégies](api-management-policies.md) du service Gestion des API Azure offrent de puissantes capacités qui permettent aux éditeurs d'API de traiter les questions transversales telles que l'authentification, l'autorisation, la limitation, la mise en cache et la transformation. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. 

Cet article explique comment déboguer les stratégies de Gestion des API à l'aide de l'[Extension Gestion des API Azure pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 

> [!NOTE]
> Cette fonctionnalité est en version préliminaire publique.

## <a name="prerequisites"></a>Prérequis

Créez une instance de niveau Développeur de Gestion des API en commençant par suivre ce [tutoriel](get-started-create-service-instance.md).

Installez [Visual Studio Code](https://code.visualstudio.com/) et la dernière version de l'[Extension Gestion des API Azure pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 

## <a name="restrictions-and-limitations"></a>Limitations et restrictions

Cette fonctionnalité est uniquement disponible au niveau Développeur de Gestion des API. Chaque instance de Gestion des API ne prend en charge qu'une seule session de débogage à la fois.

## <a name="initiate-a-debugging-session"></a>Lancer une session de débogage

1. Lancez Visual Studio Code.
2. Accédez à l'extension Gestion des API sous les extensions Azure.
3. Recherchez l'instance de Gestion des API à déboguer.
4. Recherchez l'API et l'opération à déboguer.
5. Cliquez avec le bouton droit sur l'opération et sélectionnez **Commencer le débogage de la stratégie**.

À ce stade, l'extension tentera de lancer et d'établir une session de débogage avec la passerelle Gestion des API.

![lancer le débogage](media/api-management-debug-policies/initiate-debugging-session.png)

## <a name="send-a-test-request"></a>Envoyer une requête de test
Une fois la session de débogage établie, l'extension ouvre un nouvel éditeur qui nous permet de créer et d'envoyer une requête HTTP de test à cette opération en utilisant l'[extension Client REST](https://marketplace.visualstudio.com/items?itemName=humao.rest-client).

Vous pouvez constater que l'en-tête **Ocp-Apim-Debug** a déjà été ajouté à la requête. Cet en-tête est obligatoire et la valeur doit être définie sur la clé d'abonnement avec accès complet de niveau de service pour déclencher la fonctionnalité de débogage sur la passerelle Gestion des API.

Modifiez la requête HTTP dans l'éditeur en fonction de votre scénario de test. Cliquez ensuite sur **Envoyer la requête** pour envoyer la requête de test à la passerelle Gestion des API.

![envoyer une requête de test](media/api-management-debug-policies/rest-client.png)

## <a name="debug-policies"></a>Déboguer des stratégies
Une fois la requête HTTP de test envoyée, l'extension ouvrira la fenêtre de débogage présentant les stratégies actuelles de cette opération et s'arrêtera à la première stratégie actuelle. 

![déboguer des stratégies](media/api-management-debug-policies/main-window.png)

Pour suivre le pipeline de stratégies, vous pouvez effectuer un pas à pas détaillé dans les stratégies individuelles ou définir un point d'arrêt au niveau d'une stratégie et passer directement à celle-ci. 

Dans le volet **Variables**, vous pouvez examiner les valeurs des variables créées par le système et créées par l'utilisateur. Dans le volet **Points d'arrêt**, vous pouvez consulter la liste de tous les points d'arrêt qui ont été définis. Dans le volet **Pile des appels**, vous pouvez consulter l'étendue de la stratégie actuelle. 

Si une erreur se produit pendant l'exécution de la stratégie, les détails de celle-ci apparaissent au niveau de la stratégie où elle s'est produite. 

![exceptions](media/api-management-debug-policies/exception.png)

> [!TIP]
> N'oubliez pas de quitter la session de débogage en cliquant sur le bouton **Arrêter** lorsque vous avez terminé.


## <a name="next-steps"></a>Étapes suivantes

+ Découvrez-en plus sur l'[Extension Gestion des API pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement). 
+ Signalez les problèmes dans le [référentiel GitHub](https://github.com/Microsoft/vscode-apimanagement)


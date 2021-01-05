---
title: Tutoriel - Importer et gérer des API - Gestion des API Azure et Visual Studio Code | Microsoft Docs
description: Dans ce tutoriel, découvrez comment importer, tester et gérer des API à l’aide de l’extension Gestion des API Azure pour Visual Studio Code.
ms.service: api-management
author: dlepow
ms.author: apimpm
ms.topic: tutorial
ms.date: 12/10/2020
ms.openlocfilehash: 6cf5c6f716912689b39264ed71f6a7c55f944ad2
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97410040"
---
# <a name="tutorial-use-the-api-management-extension-for-visual-studio-code-to-import-and-manage-apis"></a>Tutoriel : Importer et gérer des API à l’aide de l’extension Gestion des API pour Visual Studio Code

Dans ce tutoriel, vous allez apprendre à effectuer des opérations courantes dans Gestion des API à l’aide de l’extension Gestion des API en préversion pour Visual Studio Code. Vous utiliserez l’environnement familier de Visual Studio Code pour importer, mettre à jour, tester et gérer des API.

Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Importer une API dans Gestion des API
> * Modifier l’API
> * Appliquer des stratégies Gestion des API
> * Tester l’API


:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-result.png" alt-text="API dans l’extension Gestion des API":::

Pour obtenir une présentation des fonctionnalités supplémentaires de Gestion des API, consultez les tutoriels Gestion des API à l’aide du [portail Azure](import-and-publish.md).

## <a name="prerequisites"></a>Prérequis
- Comprendre la [terminologie relative au service Gestion des API Azure](api-management-terminology.md)
- Vérifier que [Visual Studio Code](https://code.visualstudio.com/) et la dernière [extension Gestion des API Azure pour Visual Studio Code (préversion)](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-apimanagement&ssr=false#overview) sont installés
- [Création d'une instance Gestion des API](vscode-create-service-instance.md)

## <a name="import-an-api"></a>Importer une API

L’exemple suivant importe une spécification OpenAPI au format JSON dans Gestion des API. Microsoft fournit l’API back-end utilisée dans cet exemple et l’héberge sur Azure à l’adresse `https://conferenceapi.azurewebsites.net?format=json`.

1. Dans Visual Studio Code, sélectionnez l’icône Azure dans la barre d’activités.
1. Dans le volet de l’Explorateur, développez l’instance Gestion des API que vous avez créée.
1. Cliquez avec le bouton droit sur **API**, puis sélectionnez **Importer à partir d’un lien OpenAPI**. 
1. Quand vous y êtes invité, entrez les valeurs suivantes :
    1. Un **lien OpenAPI** pour du contenu au format JSON. Pour cet exemple : *https://conferenceapi.azurewebsites.net?format=json* .
    Cette URL est le service qui implémente l’exemple d’API. Gestion des API transmet les demandes à cette adresse.
    1. Un **nom d’API** unique dans l’instance Gestion des API, par exemple *demo-conference-api*. Ce nom ne peut contenir que des lettres, des chiffres et des traits d’union. Le premier et le dernier caractères doivent être alphanumériques. Ce nom est utilisé dans le chemin pour appeler l’API.

Une fois l’API importée, elle apparaît dans le volet de l’Explorateur et les opérations d’API disponibles sont listées sous le nœud **Opérations**.

:::image type="content" source="media/visual-studio-code-tutorial/tutorial-api-operations.png" alt-text="API importée dans le volet de l’Explorateur":::

## <a name="edit-the-api"></a>Modifier l’API

Vous pouvez modifier l’API dans Visual Studio Code. Par exemple, modifiez la description JSON Resource Manager de l’API dans la fenêtre de l’éditeur pour supprimer le protocole **http** utilisé pour accéder à l’API. Sélectionnez ensuite **Fichier** > **Enregistrer**.

:::image type="content" source="media/visual-studio-code-tutorial/import-demo-api.png" alt-text="Modifier la description JSON":::

Pour modifier le format OpenAPI, cliquez avec le bouton droit sur le nom de l’API dans le volet de l’Explorateur, puis sélectionnez **Modifier OpenAPI**. Apportez vos modifications, puis sélectionnez **Fichier** > **Enregistrer**.

## <a name="apply-policies-to-the-api"></a>Appliquer des stratégies à l’API 

Gestion des API fournit des [stratégies](api-management-policies.md) que vous pouvez configurer pour vos API. Les stratégies sont un ensemble d'instructions qui sont exécutées dans l'ordre sur demande ou sur réponse d'une API. Les stratégies peuvent être soit globales (c’est-à-dire qu’elles s’appliquent à toutes les API de votre instance Gestion des API), soit délimitées à une API ou à une opération d’API spécifique.

Cette section montre comment appliquer certaines stratégies sortantes courantes à votre API pour transformer la réponse de l’API. Les stratégies de cet exemple modifient les en-têtes de réponse et masquent les URL de back-end d’origine qui apparaissent dans le corps de la réponse.

1. Dans le volet de l’Explorateur, sélectionnez **Stratégie** sous l’API *demo-conference-api* que vous avez importée. Le fichier de stratégie s’ouvre dans la fenêtre de l’éditeur. Ce fichier configure des stratégies pour toutes les opérations dans l’API. 

1. Mettez le fichier à jour avec le contenu suivant dans l’élément `<outbound>` :
    ```html
    [...]
    <outbound>
        <set-header name="Custom" exists-action="override">
            <value>"My custom value"</value>
        </set-header>
        <set-header name="X-Powered-By" exists-action="delete" />
        <redirect-content-urls />
        <base />
    </outbound>
    [...]
    ```

    * La première stratégie `set-header` ajoute un en-tête de réponse personnalisé à des fins de démonstration.
    * La deuxième stratégie `set-header` supprime l’en-tête **X-Powered-By**, s’il existe. Cet en-tête peut révéler l’infrastructure d’application utilisée dans le back-end d’API, et les éditeurs le suppriment souvent.
    * La stratégie `redirect-content-urls` réécrit (masque) les liens dans le corps de la réponse afin qu’ils pointent vers les liens équivalents par l’intermédiaire de la passerelle Gestion des API.
    
1. Enregistrez le fichier . Si vous y êtes invité, sélectionnez **Charger** pour charger le fichier dans le cloud.

## <a name="test-the-api"></a>Tester l’API

### <a name="get-the-subscription-key"></a>Obtenir la clé d’abonnement

Pour tester l’API importée que vous avez importée et les stratégies appliquées, vous avez besoin d’une clé d’abonnement pour votre instance Gestion des API.

1. Dans le volet de l’Explorateur, cliquez avec le bouton droit sur le nom de votre instance Gestion des API.
1. Sélectionnez **Copier la clé d’abonnement**.

    :::image type="content" source="media/visual-studio-code-tutorial/copy-subscription-key.png" alt-text="Copier la clé d’abonnement":::

### <a name="test-an-api-operation"></a>Tester une opération d’API

1. Dans le volet de l’Explorateur, développez le nœud **Opérations** sous l’API *demo-conference-api* que vous avez importée.
1. Sélectionnez une opération, par exemple *GetSpeakers*.
1. Dans la fenêtre de l’éditeur, en regard de **Ocp-Apim-Subscription-Key**, remplacez `{{SubscriptionKey}}` par la clé d’abonnement que vous avez copiée.
1. Sélectionnez **Envoyer une demande**. 

:::image type="content" source="media/visual-studio-code-tutorial/test-api.png" alt-text="Envoyer une demande d’API à partir de Visual Studio Code":::

Quand la demande réussit, le back-end répond avec **200 OK** et des données.

:::image type="content" source="media/visual-studio-code-tutorial/test-api-policies.png" alt-text="Opération de test de l’API":::

Notez les détails suivants dans la réponse :
* L’en-tête **Custom** est ajouté à la réponse.
* L’en-tête **X-Powered-By** n’apparaît pas dans la réponse.
* Les URL au back-end d’API sont redirigées vers la passerelle Gestion des API, dans ce cas `https://apim-hello-world.azure-api.net/demo-conference-api`.

### <a name="trace-the-api-operation"></a>Suivre l’opération d’API

Pour obtenir des informations de suivi détaillées afin de déboguer l’opération d’API, sélectionnez le lien qui s’affiche en regard de **Ocp-APIM-Trace-Location**. 

Le fichier JSON à cet emplacement contient des informations de suivi Inbound, Backend et Outbound afin que vous puissiez déterminer où se produisent les problèmes une fois la demande effectuée.

> [!TIP]
> Quand vous testez des opérations d’API, l’extension Gestion des API autorise un [débogage de stratégie](api-management-debug-policies.md) facultatif (disponible dans le niveau de service Développeur).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, enlevez l’instance Gestion des API en cliquant avec le bouton droit et en sélectionnant **Ouvrir dans le portail** pour [supprimer le service Gestion des API](get-started-create-service-instance.md#clean-up-resources) et son groupe de ressources.

Vous pouvez également sélectionner **Supprimer Gestion des API** pour supprimer uniquement l’instance Gestion des API (cette opération ne supprime pas son groupe de ressources).

:::image type="content" source="media/visual-studio-code-tutorial/vscode-apim-delete.png" alt-text="Supprimer une instance Gestion des API de VS Code":::

## <a name="next-steps"></a>Étapes suivantes

Ce tutoriel a introduit plusieurs fonctionnalités de l’extension Gestion des API pour Visual Studio Code que vous pouvez utiliser pour importer et gérer des API. Vous avez appris à :

> [!div class="checklist"]
> * Importer une API dans Gestion des API
> * Modifier l’API
> * Appliquer des stratégies Gestion des API
> * Tester l’API

L’extension Gestion des API fournit des fonctionnalités supplémentaires pour utiliser vos API. Vous pouvez par exemple [déboguer des stratégies](api-management-debug-policies.md) (disponibles dans le niveau de service Développeur) ou créer et gérer des [valeurs nommées](api-management-howto-properties.md).
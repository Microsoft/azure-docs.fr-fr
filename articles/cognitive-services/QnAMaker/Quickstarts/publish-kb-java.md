---
title: 'Démarrage rapide : Publier une base de connaissances, REST, Java - QnA Maker'
description: Ce démarrage rapide de Java basé sur REST publie votre base de connaissances et crée un point de terminaison qui peut être appelé dans votre application ou bot conversationnel.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.date: 02/08/2020
ROBOTS: NOINDEX,NOFOLLOW
ms.custom: RESTCURL2020FEB27, devx-track-java
ms.topic: how-to
ms.openlocfilehash: 8e2e902e0563e0f4ae8c0c3d0dc795a8260c62db
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351161"
---
# <a name="quickstart-publish-a-knowledge-base-in-qna-maker-using-java"></a>Démarrage rapide : Publier une base de connaissances dans QnA Maker à l’aide de Java

Ce guide de démarrage rapide basé sur REST vous aide à publier votre base de connaissances par programmation. La publication envoie la dernière version de la base de connaissances à un index de Recherche cognitive Azure dédié, et crée un point de terminaison pouvant être appelé dans votre application ou bot conversationnel.

Ce démarrage rapide fait appel aux API QnA Maker :
* [Publier](/rest/api/cognitiveservices/qnamaker/knowledgebase/publish) : cette API ne nécessite aucune information dans le corps de la requête.

## <a name="prerequisites"></a>Prérequis

* [JDK SE](/azure/developer/java/fundamentals/java-jdk-long-term-support) (Kit de développement Java, Édition Standard)
* Cet exemple utilise le [client HTTP](https://hc.apache.org/httpcomponents-client-ga/) Apache à partir de composants HTTP. Vous devez ajouter les bibliothèques clientes HTTP Apache suivantes à votre projet :
    * httpclient-4.5.3.jar
    * httpcore-4.4.6.jar
    * commons-logging-1.2.jar
* [Visual Studio Code](https://code.visualstudio.com/)
* Vous devez disposer d’un [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md). Pour récupérer votre clé et votre point de terminaison (qui incluent le nom de la ressource), sélectionnez **Démarrage rapide** pour votre ressource dans le portail Azure.
* ID de la base de connaissances QnA Maker figurant dans l’URL du paramètre de chaîne de requête `kbid`, comme illustré ci-dessous.

    ![ID de base de connaissances QnA Maker](../media/qnamaker-quickstart-kb/qna-maker-id.png)

    Si vous n’avez pas encore de base de connaissances, vous pouvez créer un exemple à utiliser pour ce démarrage rapide : [Créer une base de connaissances](create-new-kb-csharp.md).

> [!NOTE]
> Les fichiers solution complets sont disponibles dans le [dépôt GitHub **Azure-Samples/cognitive-services-qnamaker-java**](https://github.com/Azure-Samples/cognitive-services-qnamaker-java/tree/master/documentation-samples/quickstarts/publish-knowledge-base).

## <a name="create-a-java-file"></a>Créer un fichier Java

Ouvrez VSCode et créez un fichier nommé `PublishKB.java`.

## <a name="add-the-required-dependencies"></a>Ajouter les dépendances nécessaires

En haut de `PublishKB.java`, au-dessus de la classe, ajoutez les lignes suivantes pour ajouter les dépendances nécessaires au projet :

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/PublishKB.java" id="dependencies":::

## <a name="create-publishkb-class-with-main-method"></a>Créer la classe PublishKB avec la méthode main

Après les dépendances, ajoutez la classe suivante :

```Go
public class PublishKB {

    public static void main(String[] args)
    {
    }
}
```

## <a name="add-required-constants"></a>Ajouter les constantes nécessaires

Dans la méthode **main**, ajoutez les constantes nécessaires pour accéder à QnA Maker. Remplacez les valeurs par les vôtres.

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/PublishKB.java" id="constants":::

## <a name="add-post-request-to-publish-knowledge-base"></a>Ajouter une requête POST pour publier la base de connaissances

Après les constantes nécessaires, ajoutez le code suivant, qui adresse une requête HTTPS à l’API QnA Maker afin de publier une base de connaissances et reçoit la réponse :

:::code language="java" source="~/cognitive-services-quickstart-code/java/QnAMaker/rest/PublishKB.java" id="post":::

L’appel API renvoie un état 204 si la publication réussit sans aucun contenu dans le corps de la réponse. Le code ajoute du contenu pour les réponses 204.

Toutes les autres réponses sont renvoyées inchangées.

## <a name="build-and-run-the-program"></a>Créez et exécutez le projet.

Générez et exécutez le programme à partir de la ligne de commande. Il envoie automatiquement la requête à l’API QnA Maker, puis affiche la réponse dans la fenêtre de console.

1. Générez le fichier :

    ```bash
    javac -cp "lib/*" PublishKB.java
    ```

1. Exécutez le fichier :

    ```bash
    java -cp ".;lib/*" PublishKB
    ```

[!INCLUDE [Clean up files and knowledge base](../../../../includes/cognitive-services-qnamaker-quickstart-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Une fois la base de connaissances publiée, il faut que l’[URL du point de terminaison génère une réponse](./get-answer-from-knowledge-base-java.md).

> [!div class="nextstepaction"]
> [Documentation de référence pour l’API REST QnA Maker (V4)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase)
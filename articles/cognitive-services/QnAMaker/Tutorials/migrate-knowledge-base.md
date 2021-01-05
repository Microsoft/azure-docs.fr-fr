---
title: Migrer des bases de connaissances - QnA Maker
description: La migration d’une base de connaissances nécessite l’exportation d’une base de connaissances, puis l’importation dans une autre.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: how-to
ms.date: 11/09/2020
ms.openlocfilehash: c89ab375cb02824a08ff57e6b5278dd9299126ff
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350923"
---
# <a name="migrate-a-knowledge-base-using-export-import"></a>Migrer une base de connaissances à l’aide des fonctions d’exportation-importation

La migration est le processus de création d’une nouvelle base de connaissances à partir d’une base de connaissances existante. Vous pouvez effectuer cette opération pour plusieurs raisons :

* sauvegarder et restaurer le processus
* Pipeline CI/CD
* déplacer dans d’autres régions

La migration d’une base de connaissances nécessite son exportation d’une base de connaissances existante, puis son importation dans une autre.

> [!NOTE]
> Suivez les instructions ci-dessous pour migrer votre base de connaissances existante vers un nouveau QnA Maker managé (préversion).

## <a name="prerequisites"></a>Prérequis

* Avant de commencer, créez un [compte gratuit](https://azure.microsoft.com/free/cognitive-services/).
* Configurer un nouveau [service QnA Maker](../How-To/set-up-qnamaker-service-azure.md)

## <a name="migrate-a-knowledge-base-from-qna-maker"></a>Migrer une base de connaissances à partir de QnA Maker
1. Connectez-vous au [portail QnA Maker](https://qnamaker.ai).
1. Sélectionnez la base de connaissances d'origine que vous souhaitez migrer.

1. Sur la page **Paramètres**, sélectionnez **Exporter la base de connaissances** pour télécharger un fichier .tsv qui contient le contenu de votre base de connaissances : questions, réponses, métadonnées, invites de suivi et noms des sources de données à partir desquelles elles ont été extraites. Les ID QnA exportés avec les questions et réponses peuvent être utilisés pour mettre à jour une paire QnA spécifique à l’aide de l’[API de mise à jour](/rest/api/cognitiveservices/qnamaker/knowledgebase/update). L’ID QnA d’une paire QnA spécifique reste inchangé après plusieurs opérations d’exportation.

1. Sélectionnez **Créer une base de connaissances** dans le menu supérieur, puis créez une base de connaissances _vide_. Elle est vide car lorsque vous la créez, vous n'y ajoutez ni URL ni fichiers. Ceux-ci sont ajoutés lors de l’étape d’importation, après la création.

    Configurez la base de connaissances. Définissez le nom de la nouvelle base de connaissances uniquement. Les noms dupliqués et les caractères spéciaux sont acceptés.

    Ne sélectionnez rien à l’étape 4, car ces valeurs seront remplacées lorsque vous importerez le fichier.

1. À l’étape 5, sélectionnez **Créer**.

1. Dans cette nouvelle base de connaissances, ouvrez l’onglet **Paramètres**, puis sélectionnez **Importer une base de connaissances**. Cette action importe les questions, les réponses, les métadonnées et les invites de suivi, tout en conservant le nom des sources de données à partir desquelles elles ont été extraites. **Les paires QnA créées dans la nouvelle base de connaissances portent le même ID QnA que celui présent dans le fichier exporté**. Cela vous permet de créer un réplica exact de la base de connaissances.

   > [!div class="mx-imgBorder"]
   > [![Importer une base de connaissances](../media/qnamaker-how-to-migrate-kb/Import.png)](../media/qnamaker-how-to-migrate-kb/Import.png#lightbox)

1. **Testez** la nouvelle base de connaissances à l’aide du panneau de test. Découvrez comment [tester votre base de connaissances](../How-To/test-knowledge-base.md).

1. **Publiez** la base de connaissances et créez un chatbot. Découvrez comment [publier votre base de connaissances](../Quickstarts/create-publish-knowledge-base.md#publish-the-knowledge-base).

## <a name="programmatically-migrate-a-knowledge-base-from-qna-maker"></a>Migrer par programmation une base de connaissances à partir de QnA Maker

Le processus de migration est disponible par programmation à l’aide des API REST suivantes :

**Export**

* [API Télécharger la base de connaissances](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/download)

**Importer**

* [API Remplacer (recharger avec le même ID de base de connaissances)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/replace)
* [API Créer (charger avec un nouvel ID de base de connaissances)](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase/create)


## <a name="chat-logs-and-alterations"></a>Conversations et modifications
Les modifications qui ne respectent pas la casse (synonymes) ne sont pas importées automatiquement. Utilisez les [API V4](/rest/api/cognitiveservices/qnamaker4.0/knowledgebase) pour déplacer les modifications dans nouvelle la base de connaissances.

Il n’existe aucun moyen de migrer les conversations, étant donné que la nouvelle base de connaissances utilise Application Insights pour le stockage des conversations.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Modifier une base de connaissances](../How-To/edit-knowledge-base.md)
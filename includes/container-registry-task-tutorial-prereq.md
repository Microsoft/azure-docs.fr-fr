---
author: dlepow
ms.service: container-registry
ms.topic: include
ms.date: 05/02/2019
ms.author: danlep
ms.openlocfilehash: 79e2ca71d6b8178be63c3429edba89cf3bb523e2
ms.sourcegitcommit: 2e9643d74eb9e1357bc7c6b2bca14dbdd9faa436
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96029985"
---
## <a name="prerequisites"></a>Conditions préalables requises

### <a name="get-sample-code"></a>Obtenir un exemple de code

Pour suivre ce didacticiel, vous devez avoir déjà effectué les étapes du [didacticiel précédent](../articles/container-registry/container-registry-tutorial-quick-task.md) et avoir dupliqué et cloné l’exemple de référentiel. Si ce n’est déjà fait, effectuez les étapes de la section [Conditions préalables](../articles/container-registry/container-registry-tutorial-quick-task.md#prerequisites) du didacticiel précédent avant de continuer.

### <a name="container-registry"></a>Registre de conteneurs

Pour effectuer ce didacticiel, vous devez disposer d’un registre de conteneurs Azure dans votre abonnement Azure. Si vous avez besoin d’un registre, consultez le [didacticiel précédent](../articles/container-registry/container-registry-tutorial-quick-task.md) ou [Démarrage rapide : Créer un registre de conteneurs à l’aide de l’interface de ligne de commande Azure](../articles/container-registry/container-registry-get-started-azure-cli.md).

### <a name="create-a-github-personal-access-token"></a>Créer un jeton d’accès personnel GitHub

Pour déclencher une tâche basée sur une validation dans un dépôt Git, ACR Tasks a besoin d’un jeton d’accès personnel (PAT) pour accéder au dépôt. Si vous ne disposez pas encore de jeton PAT, suivez ces étapes pour en générer un dans GitHub :

1. Accédez à la page de création du jeton d’accès personnel sur GitHub à l’adresse https://github.com/settings/tokens/new.
1. Entrez une brève **description** du jeton, par exemple, « Démo ACR Tasks »
1. Sélectionnez les étendues d’accès au référentiel pour ACR. Pour accéder à un référentiel public comme dans ce didacticiel, sous **référentiel**, cochez les cases **repo:status** et **public_repo**

   ![Capture d’écran de la page de génération du jeton d’accès personnel dans GitHub][build-task-01-new-token]

   > [!NOTE]
   > Pour générer un jeton d’accès personnel pour accéder à un référentiel *privé*, sélectionnez l’étendue pour un contrôle de **référentiel** intégral.

1. Sélectionnez le bouton **Générer un jeton** (vous pouvez être invité à confirmer votre mot de passe).
1. Copiez et enregistrez le jeton généré dans un **emplacement sécurisé** (vous utiliserez ce jeton lorsque vous définirez une tâche dans la section suivante)

   ![Capture d’écran du jeton d’accès personnel généré dans GitHub][build-task-02-generated-token]

<!-- Images -->
[build-task-01-new-token]: ./media/container-registry-task-tutorial-prereq/build-task-01-new-token.png
[build-task-02-generated-token]: ./media/container-registry-task-tutorial-prereq/build-task-02-generated-token.png

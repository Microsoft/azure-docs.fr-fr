---
title: Ajouter et appeler Azure Functions à partir d’Azure Logic Apps
description: Appelez et exécutez du code personnalisé dans Azure Functions à partir de tâches et de flux de travail automatisés dans Azure Logic Apps.
services: logic-apps
ms.suite: integration
ms.reviewer: klam, logicappspm
ms.topic: article
ms.date: 10/01/2019
ms.custom: devx-track-js
ms.openlocfilehash: 75693c57a8d120aad53a15d03ae4054bac8262af
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023055"
---
# <a name="call-azure-functions-from-azure-logic-apps"></a>Appeler des fonctions Azure à partir d’Azure Logic Apps

Quand vous voulez exécuter du code qui effectue une tâche spécifique dans vos applications logiques, vous pouvez créer votre propre fonction à l’aide d’[Azure Functions](../azure-functions/functions-overview.md). Ce service vous permet de créer des fonctions Node.js, C# et F# pour ne pas avoir à générer une application ou une infrastructure complète en vue d’exécuter votre code. Vous pouvez également [appeler des applications logiques à l’intérieur de fonctions Azure](#call-logic-app). Azure Functions fournit une informatique sans serveur dans le cloud et est utile pour effectuer des tâches telles que les suivantes :

* Étendre le comportement de votre application logique avec des fonctions dans Node.js ou C#
* Effectuer des calculs dans le workflow de votre application logique
* Appliquer une mise en forme avancée ou calculer des champs dans vos applications logiques

Pour exécuter des extraits de code sans créer de fonctions Azure, découvrez comment [ajouter et exécuter du code inline](../logic-apps/logic-apps-add-run-inline-code.md).

> [!NOTE]
> L’intégration entre Logic Apps et Azure Functions ne fonctionne actuellement pas avec l’option Emplacements activée.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

* Une application de fonction Azure, qui est un conteneur pour les fonctions Azure, ainsi que votre fonction Azure. Si vous n’avez pas d’application de fonction, [créez-la en premier lieu](../azure-functions/functions-create-first-azure-function.md). Vous pouvez ensuite créer votre fonction soit à l’extérieur de votre application logique dans le portail Azure, soit [à partir de votre application logique](#create-function-designer) dans le Concepteur d’application logique.

* Quand vous travaillez avec des applications logiques, les mêmes exigences s’appliquent aux applications de fonctions et aux fonctions, qu’elles soient nouvelles ou pas :

  * Votre application de fonction et votre application logique doivent utiliser le même abonnement Azure.

  * Les nouvelles applications de fonction doivent utiliser le .NET ou JavaScript comme pile d’exécution. Quand vous ajoutez une nouvelle fonction à des applications de fonction existantes, vous pouvez sélectionner C# ou JavaScript.

  * Votre fonction utilise le modèle **Déclencheur HTTP**.

    Le modèle de déclencheur HTTP peut accepter du contenu ayant le type `application/json` à partir de votre application logique. Quand vous ajoutez une fonction Azure à votre application logique, le Concepteur d’application logique affiche des fonctions personnalisées qui sont créées à partir de ce modèle dans votre abonnement Azure.

  * Votre fonction n’utilise pas de routes personnalisées, sauf si vous avez défini une [définition OpenAPI](../azure-functions/functions-openapi-definition.md) (anciennement appelée [fichier Swagger](https://swagger.io/)).

  * Si vous avez une définition OpenAPI pour votre fonction, le Concepteur Logic Apps vous offre une meilleure expérience quand vous travaillez avec des paramètres de fonction. Pour que votre application logique puisse trouver les fonctions qui ont des définitions OpenAPI et y accéder, [configurez votre application de fonction en suivant ces étapes](#function-swagger).

* L’application logique dans laquelle vous souhaitez ajouter la fonction, y compris un [déclencheur](../logic-apps/logic-apps-overview.md#logic-app-concepts) comme première étape de votre application logique

  Pour que vous puissiez ajouter des actions qui exécutent des fonctions, votre application logique doit démarrer avec un déclencheur. Si vous débutez avec les applications logiques, consultez [Qu’est-ce qu’Azure Logic Apps ?](../logic-apps/logic-apps-overview.md) et [Démarrage rapide : Créer votre première application logique](../logic-apps/quickstart-create-first-logic-app-workflow.md).

<a name="function-swagger"></a>

## <a name="find-functions-that-have-openapi-descriptions"></a>Rechercher des fonctions qui ont des descriptions OpenAPI

Pour bénéficier d’une expérience plus complète quand vous travaillez avec des paramètres de fonction dans le Concepteur Logic Apps, [générez une définition OpenAPI](../azure-functions/functions-openapi-definition.md) (anciennement [fichier Swagger](https://swagger.io/)) pour votre fonction. Pour configurer votre application de fonction afin que votre application logique puisse trouver les fonctions qui ont des descriptions Swagger et y accéder, procédez comme suit :

1. Vérifiez que votre application de fonction est en cours d’exécution.

1. Dans votre application de fonction, configurez le [Partage des ressources Cross-Origin (CORS)](https://en.wikipedia.org/wiki/Cross-origin_resource_sharing) afin que toutes les origines soient autorisées en procédant comme suit :

   1. Dans la liste **Applications de fonction**, sélectionnez votre application de fonction. Dans le volet de droite, sélectionnez **Fonctionnalités de la plateforme** > **CORS**.

      ![Sélectionnez votre application de fonction > « Fonctionnalités de la plateforme » > « CORS »](./media/logic-apps-azure-functions/function-platform-features-cors.png)

   1. Sous **CORS**, ajoutez le caractère générique **`*`** (astérisque), mais supprimez toutes les autres origines de la liste, puis sélectionnez **Enregistrer**.

      ![Définir CORS* sur le caractère générique « * »](./media/logic-apps-azure-functions/function-platform-features-cors-origins.png)

## <a name="access-property-values-inside-http-requests"></a>Accéder aux valeurs de propriétés à l’intérieur des requêtes HTTP

Les fonctions webhook peuvent accepter des requêtes HTTP en tant qu’entrées, et transmettre ces requêtes à d’autres fonctions. Par exemple, bien que Logic Apps ait des [fonctions qui convertissent les valeurs DateTime](../logic-apps/workflow-definition-language-functions-reference.md), cette exemple de fonction JavaScript de base montre comment accéder à une propriété à l’intérieur d’un objet de requête qui est passé à la fonction, et comment effectuer des opérations sur la valeur de cette propriété. Pour accéder aux propriétés à l’intérieur d’objets, cet exemple utilise l’[opérateur point (.)](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Property_accessors) :

```javascript
function convertToDateString(request, response){
   var data = request.body;
   response = {
      body: data.date.ToDateString();
   }
}
```

Voici ce qui se passe à l’intérieur de cette fonction :

1. La fonction crée une variable `data` et assigne l’objet `body` à l’intérieur de l’objet `request` à cette variable. La fonction utilise l’opérateur point (.) pour référencer l’objet `body` à l’intérieur de l’objet `request` :

   ```javascript
   var data = request.body;
   ```

1. La fonction peut maintenant accéder à la propriété `date` par le biais de la variable `data`, et convertir cette valeur de propriété du type DateTime vers le type DateString en appelant la fonction `ToDateString()`. La fonction retourne également le résultat par le biais de la propriété `body` dans la réponse de la fonction :

   ```javascript
   body: data.date.ToDateString();
   ```

Maintenant que vous avez créé votre fonction Azure, suivez les étapes pour découvrir comment [ajouter des fonctions à des applications logiques](#add-function-logic-app).

<a name="create-function-designer"></a>

## <a name="create-functions-inside-logic-apps"></a>Créer des fonctions au sein d’applications logiques

Vous pouvez créer des fonctions Azure directement depuis le flux de travail de votre application logique à l’aide de l’action intégrée Azure Functions dans le Concepteur d’application logique, mais cette méthode est réservée aux fonctions Azure écrites en JavaScript. Pour les autres langages, vous pouvez créer des fonctions Azure via l’expérience Azure Functions du portail Azure. Pour plus d'informations, consultez [Créer votre première fonction à l’aide du portail Azure](../azure-functions/functions-create-first-azure-function.md).

Cela étant, avant de pouvoir créer une fonction Azure, vous devez déjà disposer d’une application de fonction Azure, à savoir d'un conteneur pour vos fonctions. Si vous n’avez pas d’application de fonction, créez-la en premier lieu. Voir [Créer votre première fonction à l’aide du portail Azure](../azure-functions/functions-create-first-azure-function.md).

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Pour créer et ajouter votre fonction, suivez l’étape qui s’applique à votre scénario :

   * Sous la dernière étape du flux de travail de votre application logique, sélectionnez **Nouvelle étape**.

   * Entre les étapes existantes du flux de travail de votre application logique, déplacez votre souris sur la flèche, sélectionnez le signe plus (+), puis **Ajouter une action**.

1. Dans la zone de recherche, entrez le filtre « azure functions ». Dans la liste d’actions, sélectionnez l’action **Choisir une fonction Azure**, par exemple :

   ![Recherchez « Azure Functions »](./media/logic-apps-azure-functions/find-azure-functions-action.png)

1. Dans la liste d’applications de fonction, sélectionnez la vôtre. Une fois la liste des actions ouverte, sélectionnez cette action : **Créer une fonction**

   ![Sélectionnez votre application de fonction](./media/logic-apps-azure-functions/select-function-app-create-function.png)

1. Dans l’éditeur de définition de fonction, définissez votre fonction :

   1. Dans la zone **Nom de la fonction**, fournissez un nom pour votre fonction.

   1. Dans la zone **Code**, ajoutez votre code au modèle de fonction, notamment la réponse et la charge utile qui doivent être retournées à votre application logique une fois l’exécution de votre fonction terminée. Sélectionnez **Créer** lorsque vous avez terminé.

   Par exemple :

   ![Définissez votre fonction](./media/logic-apps-azure-functions/add-code-function-definition.png)

   Dans le code du modèle, *`context` l’objet* fait référence au message envoyé par votre application logique via le champ **Corps de la demande** au cours d’une étape ultérieure. Pour accéder aux propriétés de l’objet `context` depuis l’intérieur de votre fonction, utilisez la syntaxe suivante :

   `context.body.<property-name>`

   Par exemple, pour référencer la propriété `content` à l’intérieur de l’objet `context`, utilisez la syntaxe suivante :

   `context.body.content`

   Le code du modèle inclut également une variable `input`, qui stocke la valeur du paramètre `data` afin que votre fonction puisse effectuer des opérations sur cette valeur. À l’intérieur des fonctions JavaScript, la variable `data` est également un raccourci pour `context.body`.

   > [!NOTE]
   > La propriété `body` s’applique ici à l’objet `context`. Ce n’est pas la même chose que le jeton **Corps** du résultat d’une action, que vous pouvez également transmettre à votre fonction.

1. Dans la zone **Corps de la demande**, spécifiez l’entrée de votre fonction, qui doit être au format JSON (JavaScript Objet Notation).

   Cette entrée est l’*objet de contexte* ou le message que votre application logique envoie à votre fonction. Quand vous cliquez dans le champ **Corps de la demande**, la liste de contenu dynamique s’ouvre afin que vous puissiez sélectionner des jetons pour les sorties issues des étapes précédentes. Cet exemple indique que la charge utile du contexte contient une propriété nommée `content` qui a la valeur du jeton **De** à partir du déclencheur d’e-mail.

   ![Exemple « Corps de la demande » : charge utile d’objet de contexte](./media/logic-apps-azure-functions/function-request-body-example.png)

   Ici, l’objet de contexte n’est pas converti sous forme de chaîne. Le contenu de l’objet est donc ajouté directement à la charge utile JSON. Toutefois, lorsque l’objet de contexte n’est pas un jeton JSON qui renvoie une chaîne, un objet JSON ou un tableau JSON, une erreur se produit. Par conséquent, en utilisant à la place le jeton **Heure de réception**, vous pouvez convertir l’objet de contexte sous forme de chaîne en ajoutant des guillemets doubles.

   ![Caster un objet en tant que chaîne](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

1. Pour spécifier d’autres détails comme la méthode à utiliser, les en-têtes de demande, les paramètres de requête ou l’authentification, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les options souhaitées. Pour l’authentification, vos options varient selon la fonction sélectionnée. Consultez [Activer l’authentification pour les fonctions Azure](#enable-authentication-functions).

<a name="add-function-logic-app"></a>

## <a name="add-existing-functions-to-logic-apps"></a>Ajouter des fonctions existantes à des applications logiques

Pour appeler des fonctions Azure existantes à partir de vos applications logiques, vous pouvez ajouter des fonctions Azure comme toute autre action dans le Concepteur d’applications logiques.

1. Dans le [portail Azure](https://portal.azure.com), ouvrez votre application logique dans le Concepteur d’applications logiques.

1. Sous l’étape à laquelle vous souhaitez ajouter la fonction, sélectionnez **Nouvelle étape**.

1. Sous **Choisir une action**, dans la zone de recherche, entrez « fonctions azure » en guise de filtre. Dans la liste d’actions, sélectionnez l’action **Choisir une fonction Azure**.

   ![Recherchez « Azure Functions »](./media/logic-apps-azure-functions/find-azure-functions-action.png)

1. Dans la liste d’applications de fonction, sélectionnez la vôtre. Une fois la liste de fonctions affichée, sélectionnez votre fonction.

   ![Sélectionnez votre application de fonction et votre fonction Azure](./media/logic-apps-azure-functions/select-function-app-existing-function.png)

   Pour les fonctions qui ont des définitions d’API (description Swagger) et qui sont [configurées pour que votre application logique puisse rechercher ces fonctions et y accéder](#function-swagger), vous pouvez sélectionner **Actions Swagger**.

   ![Sélectionnez votre application de fonction, « Actions Swagger » et votre fonction Azure](./media/logic-apps-azure-functions/select-function-app-existing-function-swagger.png)

1. Dans la zone **Corps de la demande**, spécifiez l’entrée de votre fonction, qui doit être au format JSON (JavaScript Objet Notation).

   Cette entrée est l’*objet de contexte* ou le message que votre application logique envoie à votre fonction. Quand vous cliquez dans le champ **Corps de la demande**, la liste de contenu dynamique s’affiche afin que vous puissiez sélectionner des jetons pour les sorties des étapes précédentes. Cet exemple indique que la charge utile du contexte contient une propriété nommée `content` qui a la valeur du jeton **De** à partir du déclencheur d’e-mail.

   ![Exemple « Corps de la demande » : charge utile d’objet de contexte](./media/logic-apps-azure-functions/function-request-body-example.png)

   Ici, l’objet de contexte n’est pas converti sous forme de chaîne. Le contenu de l’objet est donc ajouté directement à la charge utile JSON. Toutefois, lorsque l’objet de contexte n’est pas un jeton JSON qui renvoie une chaîne, un objet JSON ou un tableau JSON, une erreur se produit. Par conséquent, en utilisant à la place le jeton **Heure de réception**, vous pouvez convertir l’objet de contexte sous forme de chaîne en ajoutant des guillemets doubles :

   ![Caster un objet en tant que chaîne](./media/logic-apps-azure-functions/function-request-body-string-cast-example.png)

1. Pour spécifier d’autres détails comme la méthode à utiliser, les en-têtes de demande, les paramètres de requête ou l’authentification, ouvrez la liste **Ajouter un nouveau paramètre**, puis sélectionnez les options souhaitées. Pour l’authentification, vos options varient selon la fonction sélectionnée. Consultez [Activer l’authentification dans les fonctions Azure](#enable-authentication-functions).

<a name="call-logic-app"></a>

## <a name="call-logic-apps-from-azure-functions"></a>Appeler des applications logiques à partir de fonctions Azure

Pour déclencher une application logique à partir d’une fonction Azure, l’application logique doit démarrer avec un déclencheur fournissant un point de terminaison pouvant être appelé. Par exemple, vous pouvez démarrer l’application logique avec le déclencheur **HTTP**, **Requête**, **Files d’attente Azure** ou **Event Grid**. À partir de l’intérieur de votre fonction, envoyez une requête HTTP POST à l’URL du déclencheur et incluez la charge utile que cette application logique doit traiter. Pour plus d’informations, consultez [Appeler, déclencher ou imbriquer des applications logiques](../logic-apps/logic-apps-http-endpoint.md).

<a name="enable-authentication-functions"></a>

## <a name="enable-authentication-for-azure-functions"></a>Activer l’authentification pour les fonctions Azure

Pour authentifier facilement l’accès à d’autres ressources qui sont protégées par Azure Active Directory (Azure AD) sans avoir à vous connecter ni à fournir des informations d’identification ou des secrets, votre application logique peut utiliser une [identité managée](../active-directory/managed-identities-azure-resources/overview.md) (anciennement appelée MSI ou Managed Service Identity). Azure gère cette identité pour vous et vous aide à sécuriser vos informations d’identification, car vous n’êtes pas obligé de fournir ni de faire pivoter des secrets. En savoir plus sur les [services Azure qui prennent en charge les identités managées pour l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

Si vous configurez votre application logique pour utiliser l’identité affectée par le système ou une identité affectée par l’utilisateur créée manuellement, les fonctions Azure de votre application logique peuvent également utiliser cette même identité pour l’authentification. Pour en savoir plus sur la prise en charge de l’authentification pour les fonctions Azure dans les applications logiques, consultez [Ajouter l’authentification aux appels sortants](../logic-apps/logic-apps-securing-a-logic-app.md#add-authentication-outbound).

Pour configurer et utiliser l’identité managée avec votre fonction, effectuez les étapes suivantes :

1. Activez l’identité managée sur votre application logique, puis configurez l’accès de cette identité à la ressource cible. Consultez [Authentifier l’accès aux ressources Azure avec des identités managées dans Azure Logic Apps](../logic-apps/create-managed-service-identity.md).

1. Pour activer l’authentification dans votre fonction Azure et votre application de fonction, procédez comme suit :

   * [Configurer l’authentification anonyme dans votre fonction](#set-authentication-function-app)
   * [Configurer l’authentification Azure AD dans votre application de fonction](#set-azure-ad-authentication)

<a name="set-authentication-function-app"></a>

### <a name="set-up-anonymous-authentication-in-your-function"></a>Configurer l’authentification anonyme dans votre fonction

Pour utiliser l’identité managée de votre application logique dans votre fonction Azure, vous devez définir le niveau d’authentification de votre fonction à anonyme. Dans le cas contraire, votre application logique génère une erreur « BadRequest ».

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre application de fonction. Ces étapes utilisent « FabrikamFunctionApp » comme exemple d’application de fonction.

1. Sélectionnez **Fonctionnalités de la plateforme** dans le volet de l’application de fonction. Sous **Outils de développement**, sélectionnez **Outils avancés (Kudu)** .

   ![Ouvrir les outils avancés pour Kudu](./media/logic-apps-azure-functions/open-advanced-tools-kudu.png)

1. Dans la barre de titre du site web Kudu, dans le menu **Console de débogage**, sélectionnez **CMD**.

   ![Dans le menu de la console de débogage, sélectionner l’option « CMD »](./media/logic-apps-azure-functions/open-debug-console-kudu.png)

1. Une fois la page suivante affichée, dans la liste des dossiers, sélectionnez **site** > **wwwroot** > *votre-fonction*. Ces étapes utilisent « FabrikamAzureFunction » comme exemple de fonction.

   ![Sélectionnez « Site » > « wwwroot » > votre fonction](./media/logic-apps-azure-functions/select-site-wwwroot-function-folder.png)

1. Ouvre le fichier `function.json` pour modification.

   ![Cliquer sur Modifier pour le fichier « function.json »](./media/logic-apps-azure-functions/edit-function-json-file.png)

1. Dans l’objet `bindings`, vérifiez si la propriété `authLevel` existe. Si la propriété existe, définissez sa valeur sur `anonymous`. Sinon, ajoutez cette propriété et définissez sa valeur.

   ![Ajoutez la propriété « authLevel » et attribuez-lui la valeur « anonymous ».](./media/logic-apps-azure-functions/set-authentication-level-function-app.png)

1. Lorsque vous avez terminé, enregistrez vos paramètres, puis passez à la section suivante.

<a name="set-azure-ad-authentication"></a>

### <a name="set-up-azure-ad-authentication-for-your-function-app"></a>Configurer l’authentification Azure AD pour votre application de fonction

Avant de commencer cette tâche, recherchez et mettez ces valeurs de côté pour une utilisation ultérieure :

* L’ID d’objet généré pour l’identité attribuée par le système qui représente votre application logique

  * Pour générer cet ID d’objet, [activez l’identité attribuée par le système de votre application logique](../logic-apps/create-managed-service-identity.md#azure-portal-system-logic-app).

  * Sinon, pour trouver cet ID d’objet, ouvrez votre application logique dans le concepteur d’application logique. Dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Identité** > **Attribuée par le système**.

* L’ID de répertoire de votre locataire dans Azure Active Directory (Azure AD)

  Pour récupérer l’ID de répertoire de votre locataire, vous pouvez exécuter la commande PowerShell [`Get-AzureAccount`](/powershell/module/servicemanagement/azure.service/get-azureaccount). Ou, dans le portail Azure, procédez comme suit :

  1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre application de fonction.

  1. Recherchez et sélectionnez votre locataire Azure AD. Ces étapes utilisent « Fabrikam » comme exemple de locataire.

  1. Dans le menu du locataire, sous **Gérer**, sélectionnez **Propriétés**.

  1. Copiez l’ID de répertoire de votre locataire, par exemple, puis enregistrez-le pour une utilisation ultérieure.

     ![Rechercher et copier l’ID de répertoire du locataire Azure AD](./media/logic-apps-azure-functions/azure-active-directory-tenant-id.png)

* L’ID de la ressource cible à laquelle vous souhaitez accéder

  * Pour rechercher ces ID de ressource, consultez les [services Azure qui prennent en charge Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

  > [!IMPORTANT]
  > Cet ID de ressource doit correspondre exactement à ce qu’attend Azure AD, notamment les barres obliques de fin obligatoires.

  Cet ID de ressource est également la même valeur que celle que vous utiliserez ultérieurement dans la propriété **Audience** lors de la [configuration de votre action de fonction pour qu’elle utilise l’identité attribuée par le système](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

Vous êtes maintenant prêt à configurer l’authentification Azure AD pour votre application de fonction.

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre application de fonction.

1. Sélectionnez **Fonctionnalités de la plateforme** dans le volet de l’application de fonction. Sous **Mise en réseau**, sélectionnez **Authentification/Autorisation**.

   ![Afficher les paramètres d’authentification et d’autorisation](./media/logic-apps-azure-functions/view-authentication-authorization-settings.png)

1. Définissez le paramètre **Authentification App Service** sur **Activé**. Dans la liste **Action à exécuter quand une demande n’est pas authentifiée**, sélectionnez **Se connecter avec Azure Active Directory**. Sous **Fournisseurs d’authentification**, cliquez sur **Azure Active Directory**.

   ![Activer l’authentification avec Azure AD](./media/logic-apps-azure-functions/turn-on-authentication-azure-active-directory.png)

1. Dans le volet **Paramètres Azure Active Directory**, procédez comme suit :

   1. Définissez le **Mode de gestion** sur **Avancé**.

   1. Dans la propriété **ID client**, entrez l’ID d’objet de l’identité attribuée par le système de votre application logique.

   1. Dans la propriété **URL de l’émetteur**, entrez l’URL `https://sts.windows.net/` et ajoutez l’ID de répertoire de votre locataire Azure AD.

      `https://sts.windows.net/<Azure-AD-tenant-directory-ID>`

   1. Dans la propriété **Audiences de jeton autorisées**, entrez l’ID de la ressource cible à laquelle vous souhaitez accéder.

      Cet ID de ressource est la même valeur que celle que vous utiliserez ultérieurement dans la propriété **Audience** lors de la [configuration de votre action de fonction pour qu’elle utilise l’identité attribuée par le système](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

   À ce stade, votre version ressemble à l’exemple suivant :

   ![Paramètres d’authentification Azure Active Directory](./media/logic-apps-azure-functions/azure-active-directory-authentication-settings.png)

1. Quand vous avez terminé, sélectionnez **OK**.

1. Revenez au concepteur d’application logique et suivez les [étapes pour authentifier l’accès avec l’identité managée](../logic-apps/create-managed-service-identity.md#authenticate-access-with-identity).

## <a name="next-steps"></a>Étapes suivantes

* En savoir plus sur les [connecteurs Logic Apps](../connectors/apis-list.md)

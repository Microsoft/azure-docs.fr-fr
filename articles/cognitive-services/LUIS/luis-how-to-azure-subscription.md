---
title: Utilisation des clés de création et de runtime - LUIS
description: Quand vous utilisez LUIS pour la première fois, vous n’avez pas besoin de créer de clé de création. Quand vous souhaitez publier l’application, puis utiliser votre point de terminaison de runtime, vous devez créer et affecter la clé de runtime à l’application.
services: cognitive-services
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: how-to
ms.date: 09/07/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 19c27dc80f9af013c458663c9c7afb0033683acd
ms.sourcegitcommit: fa807e40d729bf066b9b81c76a0e8c5b1c03b536
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97348065"
---
# <a name="create-luis-resources"></a>Créer des ressources LUIS

Les ressources de création et de runtime de prédiction de requête fournissent une fonctionnalité d’authentification à votre application LUIS (Language Understanding) et à votre point de terminaison de prédiction.

<a name="azure-resources-for-luis"></a>
<a name="programmatic-key" ></a>
<a name="endpoint-key"></a>
<a name="authoring-key"></a>

## <a name="luis-resources"></a>Ressources LUIS

LUIS autorise trois types de ressources Azure et une ressource non-Azure :

|Ressource|Objectif|Service cognitif `kind`|Service cognitif `type`|
|--|--|--|--|
|Ressource de création|Vous permet de créer, de gérer, d’entraîner, de tester et de publier vos applications. [Créez une ressource de création LUIS](luis-how-to-azure-subscription.md#create-luis-resources-in-the-azure-portal) si vous comptez créer des applications LUIS par programmation ou à partir du portail LUIS. Vous devez d’abord [migrer votre compte LUIS](luis-migration-authoring.md#what-is-migration) pour pouvoir lier vos ressources de création Azure à votre application. Vous pouvez contrôler les autorisations d’accès à la ressource de création en attribuant le [rôle Contributeur](#contributions-from-other-authors) aux utilisateurs appropriés. <br><br> Il existe un seul niveau disponible pour la ressource de création LUIS :<br> <ul> <li>**Ressource de création F0 gratuite**, qui vous offre 1 million de transactions de création gratuites et 1 000 demandes de point de terminaison de prédiction de test gratuites par mois. |`LUIS.Authoring`|`Cognitive Services`|
|Ressource de prédiction| Après avoir publié votre application LUIS, utilisez la ressource/clé de prédiction pour interroger les demandes de point de terminaison de prédiction. Créez une ressource de prédiction LUIS avant que votre application cliente ne demande des prédictions au-delà des 1 000 demandes fournies par la ressource de création ou de démarrage. <br><br> Il existe deux niveaux disponibles pour la ressource de prédiction :<br><ul> <li> **Ressource de prédiction F0 gratuite**, qui vous offre 10 000 demandes de point de terminaison de prédiction gratuites par mois.<br> <li> **Ressource de prédiction S0 standard**, qui correspond au niveau payant. [Apprenez-en davantage sur la tarification.](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/)|`LUIS`|`Cognitive Services`|
|Ressource de démarrage/d’essai|Vous permet de créer, de gérer, d’entraîner, de tester et de publier vos applications. Cette ressource est créée par défaut si vous choisissez l’option de ressource de démarrage au moment de votre première connexion à LUIS. La clé de démarrage finira par être dépréciée. Tous les utilisateurs de LUIS devront [migrer leurs comptes](luis-migration-authoring.md#what-is-migration), et lier leurs applications LUIS à une ressource de création. Contrairement à la ressource de création, cette ressource ne vous accorde pas d’autorisations pour le contrôle d’accès en fonction du rôle Azure. <br><br> Tout comme la ressource de création, la ressource de démarrage vous offre 1 million de transactions de création gratuites et 1 000 demandes de point de terminaison de prédiction de test gratuites.|-|Ressource non Azure.|
|[Clé de ressource multiservice Cognitive Services](../cognitive-services-apis-create-account-cli.md?tabs=windows#create-a-cognitive-services-resource)|Requêtes de point de terminaison de prédiction de requête partagées avec LUIS et d’autres services Cognitive Services pris en charge.|`CognitiveServices`|`Cognitive Services`|


> [!Note]
> LUIS fournit deux types de ressource F0 (niveau gratuit) : l’une pour les transactions de création et l’autre pour les transactions de prédiction. Si vous épuisez le quota gratuit de transactions de prédiction, veillez à utiliser la ressource de prédiction F0, qui vous offre 10 000 transactions gratuites par mois, et non la ressource de création, qui vous offre seulement 1 000 transactions de prédiction par mois.

À la fin du processus de création de ressource Azure, [affectez la ressource](#assign-a-resource-to-an-app) à l’application dans le portail LUIS.

> [!important]
> Vous devez créer des applications LUIS dans les [régions](luis-reference-regions.md#publishing-regions) où vous souhaitez effectuer des opérations de publication et d’interrogation.

## <a name="resource-ownership"></a>Propriété des ressources

Une ressource Azure, par exemple une ressource LUIS, appartient à l’abonnement qui contient la ressource.

Pour changer la propriété d’une ressource, vous pouvez effectuer l’une des actions suivantes :
* Transférer la [propriété](../../cost-management-billing/manage/billing-subscription-transfer.md) de votre abonnement.
* Exporter l’application LUIS en tant que fichier, puis importer l’application dans un autre abonnement. L’exportation est disponible dans la page **Mes applications** du portail LUIS.

## <a name="resource-limits"></a>Limites des ressources

### <a name="authoring-key-creation-limits"></a>Limites de création de clés de création

Vous pouvez générer jusqu’à 10 clés de création par région et par abonnement. Les régions de publication sont différentes des régions de création. Veillez à créer une application dans la région de création qui correspond à la région de publication dans laquelle vous souhaitez que votre application cliente soit située. Pour plus d’informations sur la façon dont les régions de création correspondent aux régions de publication, consultez [Régions de création et de publication](luis-reference-regions.md). 

Pour plus d’informations sur les limites de clés, consultez [Limites de clés](luis-limits.md#key-limits).

### <a name="errors-for-key-usage-limits"></a>Erreurs liées aux limites d’utilisation des clés

Les limites d’utilisation dépendent du niveau tarifaire.

Si vous dépassez votre quota de transactions par seconde (TPS), vous recevez une erreur HTTP 429. Si vous dépassez votre quota de transactions par mois (TPM), vous recevez une erreur HTTP 403.


### <a name="reset-an-authoring-key"></a>Réinitialiser une clé de création

Pour les applications de [ressource de création migrées](luis-migration-authoring.md) : Si votre clé de création est compromise, réinitialisez-la dans le portail Azure, dans la page **Clés** de la ressource de création.

Pour les applications qui n’ont pas été migrées : La clé est réinitialisée pour toutes vos applications dans le portail LUIS. Si vous éditez vos applications par le biais des API de création, vous devez changer la valeur de `Ocp-Apim-Subscription-Key` pour la nouvelle clé.

### <a name="regenerate-an-azure-key"></a>Regénérer une clé Azure

Vous pouvez regénérer une clé Azure à partir de la page **Clés** du portail Azure.


<a name="securing-the-endpoint"></a>

## <a name="app-ownership-access-and-security"></a>Propriété, accès et sécurité des applications

Une application est définie par ses ressources Azure, lesquelles sont déterminées par l’abonnement du propriétaire.

Vous pouvez déplacer votre application LUIS. Pour ce faire, utilisez les ressources suivantes via le portail Azure ou Azure CLI :

* [Déplacer une application entre des ressources de création LUIS](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/apps-move-app-to-another-luis-authoring-azure-resource)
* [Déplacer une ressource vers un nouveau groupe de ressources ou un nouvel abonnement](../../azure-resource-manager/management/move-resource-group-and-subscription.md)
* [Déplacer une ressource au sein du même abonnement ou entre des abonnements distincts](../../azure-resource-manager/management/move-limitations/app-service-move-limitations.md)


### <a name="contributions-from-other-authors"></a>Contributions d'autres auteurs

Pour les applications de [ressource de création migrées](luis-migration-authoring.md) : Vous pouvez gérer les _contributeurs_ d’une ressource de création dans le portail Azure à l’aide de la page **Contrôle d’accès (IAM)** . Découvrez [comment ajouter un utilisateur](luis-how-to-collaborate.md) à l’aide de l’adresse e-mail du collaborateur et du rôle Contributeur.

Pour les applications qui n’ont pas encore été migrées : Vous pouvez gérer tous les _collaborateurs_ dans le portail LUIS à partir de la page **Gérer -> Collaborateurs**.

### <a name="query-prediction-access-for-private-and-public-apps"></a>Accès aux prédictions de requête pour les applications privées et publiques

Pour les applications privées, le runtime de prédiction de requête est accessible aux propriétaires et aux contributeurs. Pour les applications publiques, le runtime est accessible à tous les utilisateurs qui disposent de leur propre ressource de runtime Azure [Cognitive Services](../cognitive-services-apis-create-account.md) ou [LUIS](#create-resources-in-the-azure-portal), et de l’ID de l’application publique.

Il n’existe aucun catalogue d’applications publiques.

### <a name="authoring-permissions-and-access"></a>Autorisations et accès à la création
L’accès à une application à partir du portail [LUIS](luis-reference-regions.md#luis-website) ou des [API de création](https://go.microsoft.com/fwlink/?linkid=2092087) est contrôlé par la ressource de création Azure.

Le propriétaire et tous les contributeurs disposent de l'accès requis pour créer l'application.

|L’accès à la création comprend :|Notes|
|--|--|
|Ajouter ou supprimer des clés de point de terminaison||
|Exporter une version||
|Exporter les journaux d’activité du point de terminaison||
|Importer une version||
|Rendre une application publique|Quand une application est publique, elle est interrogeable par toute personne disposant d’une clé de création ou de point de terminaison.|
|Modifier le modèle|
|Publish|
|Passez en revue les énoncés de point de terminaison pour un [apprentissage actif](luis-how-to-review-endpoint-utterances.md)|
|Former|

<a name="prediction-endpoint-runtime-key"></a>

### <a name="prediction-endpoint-runtime-access"></a>Accès à l'environnement d'exécution du point de terminaison de prédiction

L’accès à l’interrogation du point de terminaison de prédiction est contrôlé par un paramètre de la page **Informations sur l’application** dans la section **Gérer**.

|[Point de terminaison privé](#runtime-security-for-private-apps)|[Point de terminaison public](#runtime-security-for-public-apps)|
|:--|:--|
|Accessible au propriétaire et aux contributeurs|Accessible au propriétaire, aux contributeurs et à toute personne qui connaît l’ID de l’application|

Vous pouvez contrôler qui voit votre clé d'exécution LUIS en l'appelant dans un environnement serveur à serveur. Si vous utilisez LUIS à partir d’un bot, la connexion entre ce bot et LUIS est déjà plus sécurisée. Si vous appelez directement le point de terminaison LUIS, vous devez créer une API côté serveur (par exemple une [fonction](https://azure.microsoft.com/services/functions/) Azure) avec un accès contrôlé (via [Azure AD](https://azure.microsoft.com/services/active-directory/), par exemple). Une fois l'API côté serveur appelée et authentifiée, et l'autorisation vérifiée, passez l'appel à LUIS. Cette stratégie n’empêche pas les attaques de l’intercepteur. Toutefois, elle obfusque votre clé et l’URL du point de terminaison à vos utilisateurs, elle vous permet d’effectuer un suivi de l’accès et vous permet d’ajouter une journalisation de la réponse du point de terminaison (par exemple [Application Insights](https://azure.microsoft.com/services/application-insights/)).

### <a name="runtime-security-for-private-apps"></a>Sécurité d'exécution pour les applications privées

Le runtime d’une application privée est uniquement disponible pour les clés suivantes :

|Clé et utilisateur|Explication|
|--|--|
|Clé de création du propriétaire| Jusqu’à 1 000 accès au point de terminaison|
|Clés de création collaborateur/contributeur| Jusqu’à 1 000 accès au point de terminaison|
|N'importe quelle clé attribuée à LUIS par un auteur ou un collaborateur/contributeur|Dépend du niveau d’utilisation de la clé|

### <a name="runtime-security-for-public-apps"></a>Sécurité d'exécution pour les applications publiques

Une fois que votre application est configurée comme étant publique, _toute_ clé de création LUIS ou clé de point de terminaison LUIS valide peut l’interroger, du moment que la clé n’a pas épuisé le quota du point de terminaison.

Un utilisateur qui n’est ni propriétaire, ni contributeur, ne peut pas accéder au runtime d’une application publique s’il ne dispose pas de l’ID de l’application. LUIS n’a pas de marché public, ni d’autres moyens permettant aux utilisateurs de rechercher une application publique.

Une application publique est publiée dans toutes les régions. Ainsi, un utilisateur ayant une clé de ressource LUIS basée sur une région peut accéder à l’application dans toute région associée à la clé de ressource.


### <a name="control-access-to-your-query-prediction-endpoint"></a>Contrôler l’accès à votre point de terminaison de prédiction de requête

Vous pouvez déterminer qui peut voir votre clé d'exécution de point de terminaison de prédiction LUIS en l'appelant dans un environnement serveur à serveur. Si vous utilisez LUIS à partir d’un bot, la connexion entre ce bot et LUIS est déjà plus sécurisée. Si vous appelez directement le point de terminaison LUIS, vous devez créer une API côté serveur (par exemple une [fonction](https://azure.microsoft.com/services/functions/) Azure) avec un accès contrôlé (via [Azure AD](https://azure.microsoft.com/services/active-directory/), par exemple). Lorsque l’API côté serveur est appelée et que l’authentification et l’autorisation sont vérifiées, passez l’appel à LUIS. Cette stratégie n’empêche pas les attaques de l’intercepteur. Toutefois, elle obfusque votre point de terminaison à vos utilisateurs, elle vous permet d’effectuer un suivi de l’accès et vous permet d’ajouter une journalisation de la réponse du point de terminaison (par exemple [Application Insights](https://azure.microsoft.com/services/application-insights/)).

<a name="starter-key"></a>

## <a name="sign-in-to-the-luis-portal-and-begin-authoring"></a>Se connecter au portail LUIS et commencer à créer

1. Connectez-vous au [portail LUIS](https://www.luis.ai) et acceptez les conditions d’utilisation.
1. Commencez à créer votre application LUIS en choisissant votre clé de création LUIS Azure :

   ![Capture d’écran montrant l’écran d’accueil.](./media/luis-how-to-azure-subscription/sign-in-create-resource.png)

1. Une fois le processus de sélection des ressources terminé, [créez une application](luis-how-to-start-new-app.md#create-new-app-in-luis).


<a name="create-azure-resources"></a>
<a name="create-resources-in-the-azure-portal"></a>

[!INCLUDE [Create LUIS resource in Azure portal](includes/create-luis-resource.md)]

### <a name="create-resources-in-the-azure-cli"></a>Créer des ressources dans Azure CLI

Utilisez [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest) pour créer chaque ressource individuellement.

Ressource `kind` :

* Création : `LUIS.Authoring`
* Prédiction : `LUIS`

1. Connectez-vous à Azure CLI :

    ```azurecli
    az login
    ```

    Cette commande ouvre un navigateur pour vous permettre de sélectionner le compte approprié et de fournir l’authentification nécessaire.

1. Créez une ressource de création LUIS dont le genre est `LUIS.Authoring` et le nom `my-luis-authoring-resource`. Créez-la dans le groupe de ressources _existant_ nommé `my-resource-group` pour la région `westus`.

    ```azurecli
    az cognitiveservices account create -n my-luis-authoring-resource -g my-resource-group --kind LUIS.Authoring --sku F0 -l westus --yes
    ```

1. Créez une ressource de point de terminaison de prédiction LUIS dont le genre est `LUIS` et le nom `my-luis-prediction-resource`. Créez-la dans le groupe de ressources _existant_ nommé `my-resource-group` pour la région `westus`. Si vous souhaitez avoir un débit plus élevé que celui du niveau Gratuit, passez de `F0` à `S0`. [Découvrez plus en détail les niveaux tarifaires et le débit.](luis-limits.md#key-limits)

    ```azurecli
    az cognitiveservices account create -n my-luis-prediction-resource -g my-resource-group --kind LUIS --sku F0 -l westus --yes
    ```

    > [!Note]
    > Ces clés ne sont pas utilisées par le portail LUIS tant qu’elles n’ont pas été affectées via la page **Gérer** > **Ressources Azure** du portail LUIS.

<a name="assign-an-authoring-resource-in-the-luis-portal-for-all-apps"></a>

### <a name="assign-resources-in-the-luis-portal"></a>Affecter des ressources dans le portail LUIS

Vous pouvez affecter une ressource de création pour une seule application ou pour toutes les applications dans LUIS. La procédure suivante affecte toutes les applications à une seule ressource de création.

1. Connectez-vous au [portail LUIS](https://www.luis.ai).
1. Dans le coin supérieur droit, sélectionnez votre compte d’utilisateur, puis **Paramètres**.
1. Dans la page **Paramètres utilisateur**, sélectionnez **Ajouter une ressource de création**, puis sélectionnez une ressource de création existante. Sélectionnez **Enregistrer**.

## <a name="assign-a-resource-to-an-app"></a>Affecter une ressource à une application

>[!NOTE]
>Si vous n’avez pas d’abonnement Azure, vous ne pourrez pas affecter ni créer de ressource. Vous devez d’abord créer un [compte gratuit Azure](https://azure.microsoft.com/en-us/free/), puis revenir à LUIS pour créer une ressource à partir du portail.

Vous pouvez utiliser la procédure suivante pour créer une ressource de création ou de prédiction, ou pour l’affecter à une application : 

1. Connectez-vous au [portail LUIS](https://www.luis.ai). Sélectionnez une application dans la liste **My apps** (Mes applications).
1. Accédez à **Gérer** > **Ressources Azure** :

    ![Capture d’écran montrant la page Ressources Azure.](./media/luis-how-to-azure-subscription/manage-azure-resources-prediction.png)

1. Sous l’onglet **Ressource de prédiction** ou **Ressource de création**, sélectionnez le bouton **Ajouter une ressource de prédiction** ou **Ajouter une ressource de création**.
1. Utilisez les champs du formulaire pour trouver la ressource appropriée, puis sélectionnez **Enregistrer**.
1. Si vous n’avez pas de ressource existante, vous pouvez en créer une en sélectionnant **Créer une ressource LUIS ?** au bas de la fenêtre.


### <a name="assign-a-query-prediction-runtime-resource-without-using-the-luis-portal"></a>Affecter une ressource de runtime de prédiction de requête sans utiliser le portail LUIS

Pour les processus automatisés tels que les pipelines CI/CD, vous pouvez être amené à automatiser l’affectation d’une ressource de runtime LUIS à une application LUIS. Pour ce faire, effectuez les étapes suivantes :

1. Obtenez un jeton Azure Resource Manager à partir de ce [site web](https://resources.azure.com/api/token?plaintext=true). Ce jeton a un délai d’expiration. Veillez donc à l’utiliser immédiatement. La requête retourne un jeton Azure Resource Manager.

    ![Capture d’écran montrant le site web qui permet de demander un jeton Azure Resource Manager.](./media/luis-manage-keys/get-arm-token.png)

1. Utilisez le jeton pour demander les ressources de runtime LUIS parmi les différents abonnements. Utilisez l’[API de récupération des comptes LUIS Azure](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be313cec181ae720aa2b26c), à laquelle votre compte d’utilisateur a accès.

    Cette API POST nécessite les valeurs suivantes :

    |En-tête|Valeur|
    |--|--|
    |`Authorization`|La valeur de `Authorization` est `Bearer {token}`. La valeur du jeton doit être précédée du mot `Bearer` et d’un espace.|
    |`Ocp-Apim-Subscription-Key`|Votre clé de création.|

    L’API retourne un tableau d’objets JSON qui représentent vos abonnements LUIS. Les valeurs, qui incluent l’ID d’abonnement, le groupe de ressources et le nom de ressource, sont retournées en tant que `AccountName`. Recherchez dans le tableau l’élément correspondant à la ressource LUIS que vous souhaitez affecter à l’application LUIS.

1. Affectez le jeton à la ressource LUIS à l’aide de l’API d’[affectation de compte LUIS Azure à une application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/5be32228e8473de116325515).

    Cette API POST nécessite les valeurs suivantes :

    |Type|Paramètre|Valeur|
    |--|--|--|
    |En-tête|`Authorization`|La valeur de `Authorization` est `Bearer {token}`. La valeur du jeton doit être précédée du mot `Bearer` et d’un espace.|
    |En-tête|`Ocp-Apim-Subscription-Key`|Votre clé de création.|
    |En-tête|`Content-type`|`application/json`|
    |Querystring|`appid`|L’ID d’application LUIS.
    |body||{"AzureSubscriptionId":"ddda2925-af7f-4b05-9ba1-2155c5fe8a8e",<br>"ResourceGroup": "resourcegroup-2",<br>"AccountName": "luis-uswest-S0-2"}|

    Quand cette API réussit son exécution, elle retourne `201 - created status`.

## <a name="unassign-a-resource"></a>Annuler l’affectation d’une ressource

1. Connectez-vous au [portail LUIS](https://www.luis.ai), puis sélectionnez une application dans la liste **Mes applications**.
1. Accédez à **Gérer** > **Ressources Azure**.
1. Sous l’onglet **Ressource de prédiction** ou **Ressource de création**, sélectionnez le bouton **Annuler l’affectation de la ressource** pour la ressource concernée.

Quand vous annulez l’affectation d’une ressource, elle n’est pas supprimée d’Azure. Elle est uniquement dissociée de LUIS.


## <a name="delete-an-account"></a>Supprimer un compte

Consultez la page [Stockage et suppression de données](luis-concept-data-storage.md#accounts) pour savoir quelles données sont supprimées lorsque vous supprimez votre compte.

## <a name="change-the-pricing-tier"></a>Changer le niveau tarifaire

1.  Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez votre abonnement LUIS :

    ![Capture d’écran montrant un abonnement LUIS dans le portail Azure.](./media/luis-usage-tiers/find.png)
1.  Sélectionnez **Niveau tarifaire** pour voir les niveaux tarifaires disponibles :

    ![Capture d’écran montrant l’élément de menu Niveau tarifaire.](./media/luis-usage-tiers/subscription.png)
1.  Sélectionnez le niveau tarifaire, puis cliquez sur **Sélectionner** pour enregistrer le changement apporté :

    ![Capture d’écran montrant comment sélectionner et enregistrer un niveau tarifaire.](./media/luis-usage-tiers/plans.png)

    Une fois le changement de tarif effectué, une fenêtre indépendante s’affiche pour confirmer la mise à jour du niveau tarifaire :

    ![Capture d’écran montrant la fenêtre indépendante de confirmation de la mise à jour du tarif.](./media/luis-usage-tiers/updated.png)
1. N’oubliez pas d’[affecter cette clé de point de terminaison](#assign-a-resource-to-an-app) sur la page **Publier** et de l’utiliser dans toutes les requêtes de point de terminaison.

## <a name="view-azure-resource-metrics"></a>Voir les métriques de ressources Azure

### <a name="view-a-summary-of-azure-resource-usage"></a>Voir un récapitulatif de l’utilisation des ressources Azure
Vous pouvez voir les informations relatives à l’utilisation de LUIS dans le portail Azure. La page **Vue d’ensemble** affiche un récapitulatif, notamment des informations récentes sur les appels et les erreurs. Si vous effectuez une demande de point de terminaison LUIS, attendez au moins cinq minutes, le temps que le changement soit visible.

![Capture d’écran de la page Vue d’ensemble.](./media/luis-usage-tiers/overview.png)

### <a name="customizing-azure-resource-usage-charts"></a>Personnalisation des graphiques d’utilisation des ressources Azure
La page **Métriques** fournit une vue plus détaillée des données :

![Capture d’écran montrant la page Métriques.](./media/luis-usage-tiers/metrics-default.png)

Vous pouvez configurer vos graphiques de métriques en fonction d’une période et d’un type de métrique spécifiques :

![Capture d’écran montrant un graphique personnalisé.](./media/luis-usage-tiers/metrics-custom.png)

### <a name="total-transactions-threshold-alert"></a>Alerte de seuil de nombre total de transactions
Si vous souhaitez savoir à quel moment vous atteignez un certain seuil de transactions (10 000 transactions, par exemple), vous pouvez créer une alerte :

![Capture d’écran montrant la page des règles d’alerte.](./media/luis-usage-tiers/alert-default.png)

Ajout d’une alerte de métrique pour le **nombre total d’appels** pendant une certaine période. Ajoutez les adresses e-mail de toutes les personnes qui doivent recevoir l’alerte. Ajoutez des Webhooks pour tous les systèmes qui doivent recevoir l’alerte. Vous pouvez également exécuter une application logique lorsque l’alerte est déclenchée.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez [comment utiliser des versions](luis-how-to-manage-versions.md) pour contrôler le cycle de vie de votre application.
* Migrez vers la nouvelle [ressource de création](luis-migration-authoring.md).

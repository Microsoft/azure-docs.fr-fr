---
title: Forum Aux Questions - LUIS
description: Cet article contient des réponses aux questions fréquemment posées sur Language Understanding (LUIS).
ms.service: cognitive-services
ms.subservice: language-understanding
ms.topic: troubleshooting
ms.date: 05/06/2020
ms.openlocfilehash: e803fa393d85f7f0b4849aed96356b7a4e77d83f
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "96000128"
---
# <a name="language-understanding-frequently-asked-questions-faq"></a>Forum aux questions (FAQ) sur Language Understanding

Cet article contient des réponses aux questions fréquemment posées sur Language Understanding (LUIS).

## <a name="whats-new"></a>Nouveautés

[Découvrez-en plus](whats-new.md) sur les nouveautés du service Language Understanding (LUIS).

<a name="luis-authoring"></a>

## <a name="authoring"></a>Création

### <a name="what-are-the-luis-best-practices"></a>Quelles sont les meilleures pratiques de LUIS ?
Commencez par le [Cycle de création](luis-concept-app-iteration.md), puis lisez les [meilleures pratiques](luis-concept-best-practices.md).

### <a name="what-is-the-best-way-to-start-building-my-app-in-luis"></a>Quel est la meilleure manière de commencer à créer mon application dans LUIS ?

La meilleure façon de créer votre application consiste à recourir à un [traitement incrémentiel](luis-concept-app-iteration.md).

### <a name="what-is-a-good-practice-to-model-the-intents-of-my-app-should-i-create-more-specific-or-more-generic-intents"></a>Qu’est une bonne pratique pour modéliser les intentions de mon application ? Dois-je créer des intentions plus spécifiques ou plus génériques ?

Choisissez des intentions qui ne soient ni générales au point de se chevaucher, ni spécifiques au point de rendre difficile pour LUIS de faire la distinction entre des intentions similaires. La création d’intentions spécifiques discriminantes est l’une des meilleures pratiques pour la modélisation de LUIS.

### <a name="is-it-important-to-train-the-none-intent"></a>Est-il important de former l’intention None ?

Oui, il est conseillé de former votre intention **None** avec davantage d’énoncés à mesure que vous ajoutez des étiquettes à d’autres intentions. Un bon rapport est d’ajouter 1 ou 2 étiquettes à l’intention **None** pour chaque dizaine d’étiquettes ajoutées à une intention. Cet rapport augmente la puissance discriminante de LUIS.

### <a name="how-can-i-correct-spelling-mistakes-in-utterances"></a>Comment corriger les fautes d’orthographe dans des énoncés ?

Consultez le didacticiel sur l’[API Vérification orthographique Bing v7](luis-tutorial-bing-spellcheck.md). LUIS applique les limites imposées par l’API Vérification orthographique v7.

### <a name="how-do-i-edit-my-luis-app-programmatically"></a>Comment faire pour modifier mon application LUIS par programmation ?
Pour modifier votre application LUIS par programmation, utilisez l’[API Création](https://go.microsoft.com/fwlink/?linkid=2092087). Pour obtenir des exemples d’appel de l’API Création, voir [Appel de l’API Création de LUIS](./get-started-get-model-rest-apis.md) et [Créer une application LUIS par programmation à l’aide de Node.js](./luis-tutorial-node-import-utterances-csv.md). L’API Création exige que vous utilisiez une [clé de création](luis-how-to-azure-subscription.md#azure-resources-for-luis) plutôt qu’une clé de point de terminaison. La création par programmation permet jusqu’à 1 000 000 d’appels par mois et cinq transactions par seconde. Pour plus d’informations sur les clés à utiliser avec LUIS, voir [Gérer les clés](./luis-how-to-azure-subscription.md).

### <a name="where-is-the-pattern-feature-that-provided-regular-expression-matching"></a>Où est la fonctionnalité Modèle qui assurait la mise en correspondance d’expression régulière ?
La **fonctionnalité Modèle** précédente est actuellement désapprouvée. Elle est remplacée par **[Modèles](luis-concept-patterns.md)** .

### <a name="how-do-i-use-an-entity-to-pull-out-the-correct-data"></a>Comment utiliser une entité pour extraire les données correctes ?
Voir [Entités](luis-concept-entity-types.md) et [Extraction de données](luis-concept-data-extraction.md).

### <a name="should-variations-of-an-example-utterance-include-punctuation"></a>Les variantes d’un exemple d’énoncé incluent-elles des signes de ponctuation ?
Utilisez l’une des solutions suivantes :
* Ignorer la [ponctuation](luis-reference-application-settings.md#punctuation-normalization)
* Ajouter les différentes variantes comme exemples d’énoncés à l’intention
* Ajoutez le modèle de l’exemple d’énoncé avec la [syntaxe pour ignorer](luis-concept-patterns.md#pattern-syntax) la ponctuation.

### <a name="does-luis-currently-support-cortana"></a>LUIS prend-il en charge Cortana ?

Les applications prédéfinies Cortana sont déconseillées depuis 2017. Elles ne sont plus prises en charge.

### <a name="how-do-i-transfer-ownership-of-a-luis-app"></a>Comment faire pour transférer la propriété d’une application LUIS ?
Pour transférer une application LUIS vers un autre abonnement Azure, exportez l’application LUIS et importez-la à l’aide d’un nouveau compte. Mettez à jour l’ID de l’application LUIS dans l’application cliente qui l’appelle. La nouvelle application peut renvoyer des scores LUIS légèrement différents de ceux renvoyés par l’application d’origine.

### <a name="a-prebuilt-entity-is-tagged-in-an-example-utterance-instead-of-my-custom-entity-how-do-i-fix-this"></a>Une entité prédéfinie est marquée dans un énoncé exemple au lieu de mon entité personnalisée. Comment la corriger ?

Dans le portail LUIS, vous pouvez étiqueter le texte de l’entité exacte que vous souhaitez extraire. Si le portail LUIS n’indique pas la prédiction d’entité correcte, vous devrez peut-être ajouter d’autres énoncés et étiqueter l’entité dans le texte ou ajouter une fonctionnalité.

### <a name="i-tried-to-import-an-app-or-version-file-but-i-got-an-error-what-happened"></a>J’ai essayé d’importer un fichier d’application ou de version, mais j’ai obtenu une erreur. Que s’est-il passé ?

Découvrez-en plus sur les [erreurs d’importation de version](luis-how-to-manage-versions.md#import-errors).

<a name="luis-collaborating"></a>

## <a name="collaborating-and-contributing"></a>Collaboration et contribution

### <a name="how-do-i-give-collaborators-access-to-luis-with-azure-active-directory-azure-ad-or-azure-role-based-access-control-azure-rbac"></a>Comment permettre aux collaborateurs d'accéder à LUIS avec Azure Active Directory (Azure AD) ou le contrôle d'accès en fonction du rôle Azure (Azure RBAC) ?

Pour savoir comment octroyer l'accès aux collaborateurs, consultez [Ressources Azure Active Directory](luis-how-to-collaborate.md#azure-active-directory-resources) et [Utilisateur locataire Azure Active Directory](luis-how-to-collaborate.md#azure-active-directory-tenant-user).

<a name="luis-endpoint"></a>

## <a name="endpoint"></a>Point de terminaison

### <a name="i-received-an-http-403-error-status-code-how-do-i-fix-it"></a>J’ai reçu un code d’état d’erreur HTTP 403. Comment la corriger ?

Vous obtenez des codes d’état d’erreur 403 et 429 lorsque vous dépassez le nombre de transactions par seconde ou le nombre de transactions par mois pour votre niveau tarifaire. Augmentez votre niveau tarifaire ou utilisez des conteneurs [Language Understanding](luis-container-howto.md).

Lorsque vous utilisez l’ensemble des 1 000 requêtes de point de terminaison gratuites ou quand vous dépassez le quota de transactions mensuel de votre niveau tarifaire, vous recevez un code d’état d’erreur HTTP 403.

Pour corriger cette erreur, vous devez [modifier votre niveau tarifaire](luis-how-to-azure-subscription.md#change-the-pricing-tier) pour en choisir un de niveau supérieur ou [créer une ressource](get-started-portal-deploy-app.md#create-the-endpoint-resource) et [l’attribuer à votre application](get-started-portal-deploy-app.md#assign-the-resource-key-to-the-luis-app-in-the-luis-portal).

Les solutions pour corriger cette erreur incluent :

* Dans le [portail Microsoft Azure](https://portal.azure.com), sur votre ressource Language Understanding, dans **Gestion des ressources -> Niveau tarifaire**, remplacez votre niveau tarifaire par un niveau TPS supérieur. Vous n’avez aucune action à effectuer dans le portail Language Understanding si votre ressource est déjà attribuée à votre application Language Understanding.
*  Si votre utilisation dépasse le niveau tarifaire le plus élevé, ajoutez plus de ressources Language Understanding avec un équilibreur de charge placé devant celles-ci. Le [conteneur Language Understanding](luis-container-howto.md) avec Kubernetes ou Docker Compose peut vous y aider.

### <a name="i-received-an-http-429-error-status-code-how-do-i-fix-it"></a>J’ai reçu un code d’état d’erreur HTTP 429. Comment la corriger ?

Vous obtenez des codes d’état d’erreur 403 et 429 lorsque vous dépassez le nombre de transactions par seconde ou le nombre de transactions par mois pour votre niveau tarifaire. Augmentez votre niveau tarifaire ou utilisez des conteneurs [Language Understanding](luis-container-howto.md).

Ce code d’état est renvoyé lorsque le nombre de transactions par seconde dépasse votre niveau tarifaire.

Les solutions pour corriger le problème incluent :

* Vous pouvez [augmenter votre niveau tarifaire](luis-how-to-azure-subscription.md#change-the-pricing-tier), si vous n’êtes pas au niveau le plus élevé.
* Si votre utilisation dépasse le niveau tarifaire le plus élevé, ajoutez plus de ressources Language Understanding avec un équilibreur de charge placé devant celles-ci. Le [conteneur Language Understanding](luis-container-howto.md) avec Kubernetes ou Docker Compose peut vous y aider.
* Vous pouvez réguler vos requêtes d’application cliente avec une [stratégie de nouvelles tentatives](/azure/architecture/best-practices/transient-faults#general-guidelines) que vous implémentez lorsque vous recevez ce code d’état.

### <a name="my-endpoint-query-returned-unexpected-results-what-should-i-do"></a>Ma requête de point de terminaison a retourné des résultats inattendus. Que dois-je faire ?

Les résultats inattendus de la prédiction de requête sont basés sur l’état du modèle publié. Pour corriger le modèle, vous devrez peut-être changer le modèle, effectuer une formation et le publier à nouveau.

La correction du modèle commence par l’[entraînement actif](luis-how-to-review-endpoint-utterances.md).

Vous pouvez supprimer un entraînement non déterministe en mettant à jour l’[API des paramètres de la version de l’application](https://westus.dev.cognitive.microsoft.com/docs/services/5890b47c39e2bb17b84a55ff/operations/versions-update-application-version-settings) afin d’utiliser toutes les données d’entraînement.

Pour obtenir d’autres conseils, passez en revue les [bonnes pratiques](luis-concept-best-practices.md).

### <a name="why-does-luis-add-spaces-to-the-query-around-or-in-the-middle-of-words"></a>Pourquoi LUIS ajoute-t-il des espaces à la requête autour ou au milieu de mots ?
LUIS [tokénise](luis-glossary.md#token) l’énoncé en fonction de la [culture](luis-language-support.md#tokenization). Tant la valeur d’origine que la valeur tokénisée sont disponibles pour l’[extraction de données](luis-concept-data-extraction.md#tokenized-entity-returned).

### <a name="how-do-i-create-and-assign-a-luis-endpoint-key"></a>Comment faire pour créer et affecter une clé de point de terminaison LUIS ?
[Créez la clé de point de terminaison](luis-how-to-azure-subscription.md) dans Azure pour votre niveau de [service](https://azure.microsoft.com/pricing/details/cognitive-services/language-understanding-intelligent-services/). [Attribuez la clé](luis-how-to-azure-subscription.md) dans la page **[Ressources Azure](luis-how-to-azure-subscription.md)** . Il n’existe d’API pour cette action. Ensuite, vous devez modifier la requête HTTP adressée au point de terminaison de façon à [utiliser la nouvelle clé de point de terminaison](luis-how-to-azure-subscription.md).

### <a name="how-do-i-interpret-luis-scores"></a>Comment interpréter les scores de LUIS ?
Votre système doit utiliser l’intention de score le plus élevée, quelle qu’en soit la valeur. Par exemple, un score inférieur à 0,5 (moins de 50 %) ne signifie pas nécessairement que la confiance de LUIS est faible. La fourniture de données d’entraînement supplémentaires peut aider à augmenter le [score](luis-concept-prediction-score.md) de l’intention la plus probable.

### <a name="why-dont-i-see-my-endpoint-hits-in-my-apps-dashboard"></a>Pourquoi ne vois-je pas les accès à mon point de terminaison sur le tableau de bord de mon application ?
Le nombre total d’accès de point de terminaison est régulièrement mis à jour sur le tableau de bord de l’application, mais la fréquence de mise à jour est plus élevée pour les métriques associées à la clé du point de terminaison LUIS sur le Portail Azure.

Si vous ne voyez pas les accès de point de terminaison mis à jour sur le tableau de bord, connectez-vous au Portail Azure et trouvez la ressource associée à la clé de votre point de terminaison LUIS, puis ouvrez **Métriques** pour sélectionner la métrique **Nombre total d’appels**. Si la clé de point de terminaison est utilisée pour plusieurs applications LUIS, la métrique du Portail Azure indique le nombre agrégé d’appels provenant de toutes les applications LUIS qui l’utilisent.

### <a name="is-there-a-powershell-command-get-to-the-endpoint-quota"></a>Existe-t-il une commande PowerShell pour connaître le quota de points de terminaison ?

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Vous pouvez utiliser une commande PowerShell pour afficher le quota de points de terminaison :

```powershell
Get-AzCognitiveServicesAccountUsage -ResourceGroupName <your-resource-group> -Name <your-resource-name>
```

### <a name="my-luis-app-was-working-yesterday-but-today-im-getting-403-errors-i-didnt-change-the-app-how-do-i-fix-it"></a>Mon application LUIS fonctionnait hier, et aujourd’hui j’obtiens des erreurs 403. Je n’ai pas modifié l’application. Comment la corriger ?
Suivez ces [instructions](#how-do-i-create-and-assign-a-luis-endpoint-key) pour créer une clé de point de terminaison LUIS et l’affecter à l’application. Ensuite, vous devez modifier la requête HTTP de l’application cliente adressée au point de terminaison de façon à [utiliser la nouvelle clé de point de terminaison](luis-how-to-azure-subscription.md). Si vous avez créé une nouvelle ressource dans une autre région, modifiez également la région de la requête du client HTTP.

### <a name="how-do-i-secure-my-luis-endpoint"></a>Comment faire pour sécuriser mon point de terminaison LUIS ?
Voir [Sécurisation du point de terminaison](luis-how-to-azure-subscription.md#securing-the-endpoint).

## <a name="working-within-luis-limits"></a>Respect des limites de LUIS

### <a name="what-is-the-maximum-number-of-intents-and-entities-that-a-luis-app-can-support"></a>Quel est le nombre maximal d’intentions et d’entités qu’une application LUIS peut prendre en charge ?
Voir la référence sur les [limites](luis-limits.md).

### <a name="i-want-to-build-a-luis-app-with-more-than-the-maximum-number-of-intents-what-should-i-do"></a>Je souhaite créer une application LUIS avec un nombre d’intentions supérieur au nombre maximal. Que dois-je faire ?

Voir [Meilleures pratiques pour les intentions](luis-concept-intent.md#if-you-need-more-than-the-maximum-number-of-intents).

### <a name="i-want-to-build-an-app-in-luis-with-more-than-the-maximum-number-of-entities-what-should-i-do"></a>Je souhaite créer une application dans LUIS avec un nombre d’entités supérieur au nombre maximal. Que dois-je faire ?

Voir [Meilleures pratiques pour les entités](luis-concept-entity-types.md#if-you-need-more-than-the-maximum-number-of-entities).

### <a name="what-are-the-limits-on-the-number-and-size-of-phrase-lists"></a>Quelles sont les limites de nombre et de taille des listes de phrases ?
Pour connaître la longueur maximale d’une [liste de phrases](./luis-concept-feature.md), voir la référence [limites](luis-limits.md).

### <a name="what-are-the-limits-on-example-utterances"></a>Quelles sont les limites applicables aux exemples d’énoncés ?
Voir la référence sur les [limites](luis-limits.md).

## <a name="testing-and-training"></a>Test et apprentissage

### <a name="i-see-some-errors-in-the-batch-testing-pane-for-some-of-the-models-in-my-app-how-can-i-address-this-problem"></a>Je constate des erreurs dans le volet des tests par lot pour certains modèles dans mon application. Comment puis-je résoudre ce problème ?

Les erreurs indiquent l’existence d’incohérences entre vos étiquettes et les prédictions effectuées à partir de vos modèles. Pour résoudre le problème, accomplissez une ou plusieurs des tâches suivantes :
* Pour aider LUIS à améliorer la discrimination entre les intentions, ajoutez des étiquettes.
* Pour aider LUIS à apprendre plus rapidement, ajoutez des fonctionnalités de liste de phrases qui introduisent le vocabulaire spécifique d’un domaine.

Voir le didacticiel [Tests par lot](luis-tutorial-batch-testing.md).

### <a name="when-an-app-is-exported-then-reimported-into-a-new-app-with-a-new-app-id-the-luis-prediction-scores-are-different-why-does-this-happen"></a>Quand une application est exportée, puis réimportée dans une nouvelle application (avec un nouvel ID d’application), les scores de prédiction LUIS sont différents. Pourquoi cela se produit-il ?

Voir [Différences de prédiction entre les copies d’une même application](luis-concept-prediction-score.md#review-intents-with-similar-scores).

### <a name="some-utterances-go-to-the-wrong-intent-after-i-made-changes-to-my-app-the-issue-seems-to-disappear-at-random-how-do-i-fix-it"></a>Après avoir apporté des modifications à mon application, l'intention de certains énoncés est erronée. Le problème semble disparaître de façon aléatoire. Comment la corriger ?

Voir [Entraîner avec toutes les données](luis-how-to-train.md#train-with-all-data).

## <a name="app-publishing"></a>Publication d’application

### <a name="what-is-the-tenant-id-in-the-add-a-key-to-your-app-window"></a>Qu’est-ce que l’ID de locataire dans la fenêtre « Add a key to your app » (Ajouter une clé à votre application) ?
Dans Azure, un locataire représente le client ou l’organisation associé(e) à un service. Cherchez votre ID de locataire sur le portail Azure dans la zone **ID de répertoire** en sélectionnant **Azure Active Directory** > **Gérer** > **Propriétés**.

![ID de locataire sur le portail Azure](./media/luis-manage-keys/luis-assign-key-tenant-id.png)

<a name="why-are-there-more-subscription-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>
<a name="why-are-there-more-endpoint-keys-on-my-apps-publish-page-than-i-assigned-to-the-app"></a>


### <a name="why-are-there-more-endpoint-keys-assigned-to-my-app-than-i-assigned"></a>Pourquoi y a-t-il plus de clés de point de terminaison affectées à mon application que je ne lui en ai affecté ?
Chaque application LUIS a la clé de démarrage/création dans la liste des points de terminaison pour des raisons pratiques. Cette clé autorise uniquement quelques accès aux points de terminaison afin que vous puissiez essayer LUIS.

Si votre application existait avant la disponibilité générale de LUIS, les clés de points de terminaison LUIS dans votre abonnement sont affectées automatiquement. Cela a été conçu pour faciliter la migration pendant la période de disponibilité générale. Les nouvelles clés de points de terminaison LUIS dans le portail Azure ne sont _pas_ affectées automatiquement à LUIS.

## <a name="key-management"></a>Gestion des clés

### <a name="how-do-i-know-what-key-i-need-where-i-get-it-and-what-i-do-with-it"></a>Comment savoir de quelle clé j'ai besoin, où l'obtenir et ce que je dois en faire ?

Pour en savoir plus sur les différences entre la clé de création et la clé de runtime de prédiction, voir [Clés de point de terminaison de création et de prédiction de requête dans LUIS](luis-how-to-azure-subscription.md).

### <a name="i-got-an-error-about-being-out-of-quota-how-do-i-fix-it"></a>J'ai rencontré une erreur de dépassement de quota. Comment la corriger ?

Pour en savoir plus, voir Corriger les codes d’état HTTP [403](#i-received-an-http-403-error-status-code-how-do-i-fix-it) et [429](#i-received-an-http-429-error-status-code-how-do-i-fix-it).

### <a name="i-need-to-handle-more-endpoint-queries-how-do-i-do-that"></a>J'ai besoin de gérer davantage de requêtes de point de terminaison. Que dois-je faire ?

Pour en savoir plus, voir Corriger les codes d’état HTTP [403](#i-received-an-http-403-error-status-code-how-do-i-fix-it) et [429](#i-received-an-http-429-error-status-code-how-do-i-fix-it).

### <a name="i-created-an-authoring-key-but-it-isnt-showing-in-the-luis-portal-what-happened"></a>J’ai créé une clé de création, mais elle n’apparaît pas dans le portail LUIS. Que s’est-il passé ?

Les clés de création sont disponibles dans le portail LUIS après la [migration vers l’expérience de clé de création](luis-migration-authoring.md).

## <a name="app-management"></a>Gestion des applications

### <a name="how-do-i-download-a-log-of-user-utterances"></a>Comment faire pour télécharger un journal d’énoncés d’utilisateurs ?
Par défaut, votre application LUIS journalise les énoncés des utilisateurs. Pour télécharger un journal des énoncés que les utilisateurs envoient à votre application LUIS, accédez à **Mes applications** et sélectionnez l'application. Dans la barre d’outils contextuelle, sélectionnez **Exporter les journaux d’activité du point de terminaison**. Un journal est mis en forme comme un fichier de valeurs séparées par des virgules (CSV).

### <a name="how-can-i-disable-the-logging-of-utterances"></a>Comment puis-je désactiver la journalisation des énoncés ?
Vous pouvez désactiver la journalisation des énoncés des utilisateurs en définissant `log=false` dans l’URL de point de terminaison que votre application cliente utilise pour la interroger LUIS. Toutefois, la désactivation de la journalisation désactive la capacité de votre application LUIS à suggérer des énoncés ou à améliorer les performances sur la base d’un [apprentissage actif](luis-concept-review-endpoint-utterances.md#what-is-active-learning). Si vous définissez `log=false` en raison de problèmes de confidentialité des données, vous ne pouvez pas télécharger un enregistrement de ces énoncés d’utilisateurs à partir de LUIS ou utiliser ces énoncés pour améliorer votre application.

La journalisation est l’unique mode de stockage des énoncés.

### <a name="why-dont-i-want-all-my-endpoint-utterances-logged"></a>Qu’est-ce qui peut justifier que je ne journalise pas tous les énoncés de mon point de terminaison ?
Si vous utilisez votre journal à des fins d’analyse prédictive, ne capturez pas énoncés de test dans votre journal.

## <a name="data-management"></a>Gestion des données

### <a name="can-i-delete-data-from-luis"></a>Puis-je supprimer des données de LUIS ?

* Vous pouvez toujours supprimer des exemples d’énoncés utilisés pour l’apprentissage de LUIS. Si vous supprimez un exemple d’énoncé de votre application LUIS, il est supprimé du service web LUIS et n’est plus disponible pour l’exportation.
* Vous pouvez supprimer des énoncés de la liste des énoncés d’utilisateurs que LUIS suggère dans la page **Review endpoint utterances** (Examiner les énoncés du point de terminaison). La suppression d’énoncés de cette liste a pour effet d’empêcher leur suggestion, mais pas de les supprimer des journaux d’activité.
* Si vous supprimez un compte, toutes les applications sont supprimées, ainsi que leurs exemples d’énoncés et journaux d’activité. Les données sont conservées sur les serveurs pendant 60 jours avant leur suppression définitive.

### <a name="how-does-microsoft-manage-data-i-send-to-luis"></a>Comment Microsoft gère-t-il les données que j’envoie à LUIS ?

Le [Centre de gestion de la confidentialité](https://www.microsoft.com/trustcenter) présente nos engagements et les options disponibles pour la gestion des données et des accès dans Azure Services.

## <a name="language-and-translation-support"></a>Prise en charge des langues et de la traduction

### <a name="i-have-an-app-in-one-language-and-want-to-create-a-parallel-app-in-another-language-what-is-the-easiest-way-to-do-so"></a>Je dispose d’une application dans une langue et souhaite créer une application parallèle dans une autre langue. Quel est le moyen le plus simple de procéder ?
1. Exportez votre application.
2. Traduisez les énoncés étiquetés dans le fichier JSON de l’application exportée vers la langue cible.
3. Vous pouvez modifier les noms des intentions et entités ou les laisser tels quels.
4. Enfin, importez l’application pour obtenir une application LUIS dans la langue cible.

## <a name="app-notification"></a>Notification d’application

### <a name="why-did-i-get-an-email-saying-im-almost-out-of-quota"></a>Pourquoi ai-je reçu un e-mail indiquant que je suis presque hors quota ?
Votre clé authoring/starter n’autorise que 1 000 requêtes sur le point de terminaison par mois. Créez une clé de point de terminaison LUIS (gratuite ou payante) et utilisez-la pour effectuer des requêtes sur le point de terminaison. Si vous effectuez des requêtes sur le point de terminaison à partir d’un bot ou d’une autre application cliente, vous devez y modifier la clé de point de terminaison LUIS.

## <a name="bots"></a>Bots

### <a name="my-luis-bot-isnt-working-what-do-i-do"></a>Mon bot LUIS ne fonctionne pas. Que faire ?

Il faut d’abord constater si le problème est lié à LUIS ou survient en dehors de l’intergiciel LUIS.

#### <a name="resolve-issue-in-luis"></a>Résoudre le problème dans LUIS
Passer le même énoncé à LUIS à partir du [point de terminaison LUIS](luis-get-started-create-app.md#query-the-v2-api-prediction-endpoint). Si vous recevez une erreur, résolvez le problème dans LUIS jusqu'à ce que l’erreur ne soit plus renvoyée. Les erreurs courantes sont les suivantes :

* `Out of call volume quota. Quota will be replenished in <time>.` -Ce problème indique que vous devez passer d’une clé de création à une [clé de point de terminaison](luis-how-to-azure-subscription.md) ou changer les [niveaux de service](luis-how-to-azure-subscription.md#change-the-pricing-tier).

#### <a name="resolve-issue-in-azure-bot-service"></a>Résoudre le problème dans Azure Bot Service

Si vous utilisez Azure Bot Service et que le problème est que le **test dans la discussion Web** retourne `Sorry, my bot code is having an issue`, consultez vos journaux d’activité :

1. Dans le portail Azure, pour votre bot, sélectionnez **Build** dans la section **Gestion du bot**.
1. Ouvrez l’éditeur de code en ligne.
1. Dans la barre de navigation supérieure bleue, sélectionnez le nom du bot (le deuxième élément à droite).
1. Dans la liste déroulante qui s’affiche, sélectionnez **Ouvrir la console Kudu**.
1. Sélectionnez **LogFiles**, puis **Application**. Passez en revue tous les fichiers journaux. Si vous ne voyez pas l’erreur dans le dossier de l’application, passez en revue tous les fichiers journaux sous **LogFiles**.
1. N’oubliez pas de régénérer votre projet si vous utilisez un langage compilé tel que C#.

> [!Tip]
> La console peut également installer des packages.

#### <a name="resolve-issue-while-debugging-on-local-machine-with-bot-framework"></a>Résolvez le problème en déboguant sur un ordinateur local avec Bot Framework.

Pour en savoir plus sur le débogage local d’un bot, consultez [Déboguer un robot](/azure/bot-service/bot-service-debug-bot?view=azure-bot-service-4.0).

## <a name="integrating-luis"></a>Intégration de LUIS

### <a name="where-is-my-luis-app-created-during-the-azure-web-app-bot-subscription-process"></a>Où est l’application LUIS que j’ai créée durant le processus d’abonnement de bot d’application web Azure ?
Si vous choisissez un modèle LUIS, puis le bouton **Sélectionner** dans le volet des modèles, le volet du côté gauche change pour inclure le type de modèle, et vous êtes invité à spécifier la région dans laquelle vous voulez créer le modèle LUIS. Le processus d’abonnement de bot d’application web ne crée cependant pas d’abonnement LUIS.

![Région du bot d’application web de modèle LUIS](./media/luis-faq/web-app-bot-location.png)

### <a name="what-luis-regions-support-bot-framework-speech-priming"></a>Quelles régions LUIS prennent en charge la préparation vocale de Bot Framework ?
La [préparation vocale](/bot-framework/bot-service-manage-speech-priming) est prise en charge uniquement pour les applications LUIS dans l’instance centrale (États-Unis).

## <a name="api-programming-strategies"></a>Stratégies de programmation d'API

### <a name="how-do-i-programmatically-get-the-luis-region-of-a-resource"></a>Comment obtenir par programmation la région LUIS d'une ressource ?

Utilisez l'exemple LUIS pour [rechercher une région](https://github.com/Azure-Samples/cognitive-services-language-understanding/tree/master/documentation-samples/find-region) par programmation via C# ou Node.Js.

## <a name="luis-service"></a>Service LUIS

### <a name="is-language-understanding-luis-available-on-premises-or-in-private-cloud"></a>Language Understanding (LUIS) est-il disponible localement ou dans un cloud privé ?

Oui, vous pouvez utiliser le [conteneur](luis-container-howto.md) LUIS pour ces scénarios si vous disposez de la connectivité nécessaire pour limiter l’utilisation.

## <a name="migrating-to-the-next-version"></a>Migration vers la version suivante

### <a name="how-do-i-migrate-to-preview-v3-api"></a>Comment effectuer une migration pour afficher un aperçu de l’API V3 ?

Voir le [guide de migration d’API v2 vers v3 pour les applications LUIS](luis-migration-api-v3.md)

## <a name="build-2019-conference-announcements"></a>Annonces pour la conférence Build 2019

Les fonctionnalités suivantes ont été publiées lors de la conférence Build 2019 :

* [Aperçu du guide de migration API V3](luis-migration-api-v3.md)
* [Tableau de bord d’analyse amélioré](luis-how-to-use-dashboard.md)
* [Domaines prédéfinis améliorés](luis-reference-prebuilt-domains.md)
* [Entités de liste dynamique](schema-change-prediction-runtime.md#dynamic-lists-passed-in-at-prediction-time)
* [Entités externes](luis-migration-api-v3.md#external-entities-passed-in-at-prediction-time)

Vidéos :

* [Comment utiliser l’intelligence artificielle conversationnelle Azure pour préparer votre entreprise à la prochaine génération](https://www.youtube.com/watch?v=_k97jd-csuk&feature=youtu.be)

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur LUIS, voir les ressources suivantes :
* [Questions de Stack Overflow balisées avec LUIS](https://stackoverflow.com/questions/tagged/luis)
* [Page de questions Microsoft Q&A sur MSDN Language Understanding Intelligent Services (LUIS)](/answers/topics/azure-language-understanding.html)
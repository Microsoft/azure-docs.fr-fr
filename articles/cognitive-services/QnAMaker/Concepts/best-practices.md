---
title: Bonnes pratiques - QnA Maker
description: Suivez ces bonnes pratiques pour améliorer votre base de connaissances et fournir des résultats plus pertinents aux utilisateurs finaux de votre application/bot conversationnel.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: ee8d838ba315c2e261a61699948b71a710341165
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96346356"
---
# <a name="best-practices-of-a-qna-maker-knowledge-base"></a>Bonnes pratiques pour une base de connaissances QnA Maker

Le [cycle de vie du développement d’une base de connaissances](../Concepts/development-lifecycle-knowledge-base.md) vous guide de bout en bout sur la façon de gérer votre base de connaissances. Suivez ces bonnes pratiques pour améliorer votre base de connaissances et fournir des résultats plus pertinents aux utilisateurs finaux de l’application client ou du bot conversationnel.

## <a name="extraction"></a>Extraction

Le service QnA Maker améliore en permanence les algorithmes qui extraient des entités QnA à partir du contenu, et étend la liste des formats HTML et des formats de fichiers pris en charge. Suivez les [instructions](../index.yml) d’extraction de données en fonction de votre type de document.

En général, les pages FAQ doivent être autonomes et ne pas être combinées avec d’autres informations. Les manuels de produits doivent avoir des titres clairs et, si possible, une page d’index.

### <a name="configuring-multi-turn"></a>Configuration de l’extraction multitours

[Créez votre base de connaissances](../how-to/multiturn-conversation.md#create-a-multi-turn-conversation-from-a-documents-structure) avec extraction multitours activée. Si votre base de connaissances prend en charge la hiérarchie de questions, ou doit la prendre en charge, vous pouvez extraire ladite hiérarchie du document, ou la créer une fois le document extrait.

## <a name="creating-good-questions-and-answers"></a>Créer de bonnes questions et réponses

### <a name="good-questions"></a>Bonnes questions

Les meilleures questions sont simples. Identifiez le mot ou l’expression clé de chaque question, puis créez une question simple pour ce mot ou cette expression.

Ajoutez autant de questions alternatives que vous le souhaitez, en restant toutefois sur des modifications simples. Le fait d’ajouter des mots ou des formulations qui ne font pas partie de l’objectif principal de la question n’aide pas QnA Maker à trouver une correspondance.


### <a name="add-relevant-alternative-questions"></a>Ajouter d’autres questions appropriées

Votre utilisateur peut saisir des questions avec un style de texte de type conversation : `How do I add a toner cartridge to my printer?` ou une recherche par mot clé, comme `toner cartridge`. La base de connaissances doit avoir inclure des questions présentant ces deux styles, afin de pouvoir renvoyer la réponse la plus appropriée. Si vous ne savez pas quels mots clés le client saisit, utilisez les données Application Insights pour analyser les requêtes.

### <a name="good-answers"></a>Bonnes réponses

Les réponses les plus appropriées sont simples, mais pas trop. N’utilisez pas de réponses comme `yes` et `no`. Si votre réponse doit comporter des liens vers d’autres sources ou offrir une expérience enrichie avec des contenus multimédias et des liens, utilisez le [balisage des métadonnées](../how-to/edit-knowledge-base.md#add-metadata) pour faire la distinction entre les réponses, puis [envoyez la requête](../how-to/metadata-generateanswer-usage.md#generateanswer-request-configuration) avec des balises de métadonnées dans la propriété `strictFilters` pour obtenir la bonne version de la réponse.

|Réponse|Invites de suivi|
|--|--|
|À l'aide du bouton d'alimentation situé sur le clavier, mettez le Surface Laptop hors tension.|* Combinaisons de touches pour la mise en veille, l’arrêt et le redémarrage.<br>* Comment démarrer à froid un Surface Laptop<br>* Comment changer le BIOS d’un Surface Laptop<br>* Différences entre la mise en veille, l’arrêt et le redémarrage|
|Le service client est disponible par téléphone, Skype et SMS 24 heures sur 24.|Informations de contact - Ventes.<br> * Adresses des bureaux et magasins et horaires des visites.<br> * Accessoires pour Surface Laptop.|

## <a name="chit-chat"></a>Échanges de conversation (chit-chat)
Ajoutez facilement des échanges à votre bot afin de rendre ses conversations plus naturelles et engageantes. Vous pouvez aisément ajouter des jeux de données d’échanges de conversation pour des personnalités prédéfinies lors de la création de votre base de connaissances, et les modifier à tout moment. Découvrez comment [ajouter des échanges de conversation à votre base de connaissances](../How-To/chit-chat-knowledge-base.md).

Les échanges sont pris en charge dans de [nombreuses langues](../how-to/chit-chat-knowledge-base.md#language-support).

### <a name="choosing-a-personality"></a>Choix d’une personnalité
Les échanges de conversation sont pris en charge pour plusieurs personnalités prédéfinies :

|Personnalité |Fichier de jeu de données QnA Maker |
|---------|-----|
|Professional |[qna_chitchat_professional.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_professional.tsv) |
|Convivial |[qna_chitchat_friendly.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_friendly.tsv) |
|Comique |[qna_chitchat_witty.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_witty.tsv) |
|Attentif |[qna_chitchat_caring.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_caring.tsv) |
|Enthousiaste |[qna_chitchat_enthusiastic.tsv](https://qnamakerstore.blob.core.windows.net/qnamakerdata/editorial/qna_chitchat_enthusiastic.tsv) |

Les réponses vont du formel à l’informel, en passant par l’irrévérencieux. Vous devez sélectionner la personnalité qui correspond le plus au ton souhaité pour votre bot. Vous pouvez afficher les [jeux de données](https://github.com/Microsoft/BotBuilder-PersonalityChat/tree/master/CSharp/Datasets), en choisir un qui sert de base pour votre bot, puis personnaliser les réponses.

### <a name="edit-bot-specific-questions"></a>Modifier les questions propres au bot
Il existe certaines questions propres au bot qui font partie du jeu de données d’échanges de conversation et qui ont été remplies avec des réponses génériques. Modifiez ces réponses pour qu’elles reflètent mieux les caractéristiques de votre bot.

Nous vous recommandons de rendre les questions-réponses d’échanges de conversation suivantes plus spécifiques :

* qui êtes-vous ?
* que pouvez-vous faire ?
* Quel âge avez-vous ?
* Qui vous a créé ?
* Hello

### <a name="adding-custom-chit-chat-with-a-metadata-tag"></a>Ajout d’échanges de conversation personnalisés avec une balise de métadonnées

Si vous ajoutez vos propres paires QnA d’échanges de conversation, assurez-vous d’ajouter les métadonnées afin que ces réponses soient envoyées. La paire nom/valeur des métadonnées est `editorial:chitchat`.

## <a name="searching-for-answers"></a>Recherche de réponses

L’API GenerateAnswer utilise des questions et leur réponse pour rechercher les meilleures réponses à la requête d’un utilisateur.

### <a name="searching-questions-only-when-answer-is-not-relevant"></a>Recherche de questions uniquement lorsque la réponse n’est pas pertinente

Utilisez le paramètre [`RankerType=QuestionOnly`](#choosing-ranker-type) si vous ne voulez pas rechercher des réponses.

Voici un exemple : une base de connaissances est un catalogue d’acronymes sous la forme de questions, avec leur forme complète en tant que réponse. La valeur de la réponse ne facilite pas la recherche de la réponse appropriée.

## <a name="rankingscoring"></a>Classement/scoring
Essayez d’optimiser l’utilisation des fonctionnalités de classement que QnA Maker prend en charge. Ces fonctionnalités augmentent la probabilité de retourner une réponse pertinente à une requête utilisateur.

### <a name="choosing-a-threshold"></a>Choix d’un seuil

Le [score de confiance](confidence-score.md) par défaut utilisé comme seuil est de 0, mais vous pouvez [le changer](confidence-score.md#set-threshold) pour votre base de connaissances, en fonction de vos besoins. Chaque base de connaissances étant différente, vous devez procéder à des test et choisir le seuil qui convient le mieux à votre base de connaissances.

### <a name="choosing-ranker-type"></a>Sélection du type d’outil de classement
Par défaut, QnA Maker recherche dans les questions et les réponses. Pour rechercher dans les questions uniquement afin de générer une réponse, utilisez le paramètre `RankerType=QuestionOnly` dans le corps POST de la requête GenerateAnswer.

### <a name="add-alternate-questions"></a>Ajouter des questions alternatives
Les [questions alternatives](../How-To/edit-knowledge-base.md) augmentent la probabilité de trouver une correspondance avec une requête utilisateur. Les questions alternatives sont utiles quand il existe plusieurs manières de poser une question. C’est notamment le cas pour les variantes syntaxiques et lexicales.

|Requête d’origine|Autres requêtes|Change|
|--|--|--|
|Un stationnement est-il disponible ?|Vous avez un parking ?|Variante syntaxique|
 |Salut|Bonjour<br>Ça va ?|Variante lexicale ou argot|

<a name="use-metadata-filters"></a>

### <a name="use-metadata-tags-to-filter-questions-and-answers"></a>Utiliser des balises de métadonnées pour filtrer les questions et les réponses

Les [métadonnées](../How-To/edit-knowledge-base.md) ajoutent à une application cliente la possibilité de savoir qu’elle ne doit pas récupérer toutes les réponses, mais réduire les résultats d’une requête utilisateur en fonction des balises de métadonnées. La réponse de la base de connaissances peut varier en fonction du mot clé de métadonnées, même si la requête est la même. Par exemple, dans le cas d’une chaîne de restaurants, la question *« Où se trouve le parking ? »* peut donner une réponse différente en fonction du lieu du restaurant (ici, les métadonnées sont *Location: Seattle* et *Location: Redmond*.)

### <a name="use-synonyms"></a>Utiliser des synonymes
# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)
Les synonymes en langue anglaise sont pris en charge. Utilisez des variantes de mots (sans respect de la casse) avec [l’API Alterations](/rest/api/cognitiveservices/qnamaker/alterations/replace), afin d’ajouter des synonymes à des mots clés qui prennent différentes formes. Les synonymes sont ajoutés au niveau du service QnA Maker et **partagés par toutes les bases de connaissances du service**.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)
Les synonymes en langue anglaise sont pris en charge. Utilisez des variantes de mots (sans respect de la casse) avec [l’API Alterations](/rest/api/cognitiveservices/qnamaker/alterations/replace), afin d’ajouter des synonymes à des mots clés qui prennent différentes formes. Les synonymes dans QnA Maker managé (préversion) sont **ajoutés par base de connaissances**.

|Mot d’origine|Synonymes|
|--|--|
|buy|purchase<br>net-banking<br>net banking|

---

### <a name="use-distinct-words-to-differentiate-questions"></a>Utiliser des mots distincts pour différencier les questions
L’algorithme de classement de QnA Maker, qui recherche une correspondance entre une requête utilisateur et les questions de la base de connaissances, retourne de meilleurs résultats si chaque question répond à un besoin différent. La répétition du même mot défini entre les questions réduit la probabilité de sélection de la réponse appropriée pour une requête utilisateur qui contient ces mots.

Par exemple, vous pouvez avoir deux entités QnA distinctes avec les questions suivantes :

|Entités QnA|
|--|
|où est l’*emplacement* de stationnement|
|où est *l’emplacement* du distributeur automatique de billets|

Ces deux entités QnA employant des mots très similaires, elles pourraient générer des scores très similaires pour de nombreuses requêtes d’utilisateur utilisant la tournure de phrase *« où est l’emplacement de `<x>` ».* Essayez plutôt de les distinguer clairement avec des requêtes du type *« où est le parking »* et *« où est le distributeur automatique de billets »* , en évitant des mots comme « emplacement » susceptibles de figurer dans de nombreuses questions de la base de connaissances.

## <a name="collaborate"></a>Travailler en collaboration
QnA Maker permet aux utilisateurs de [travailler en collaboration](../index.yml) sur une base de connaissances. Les utilisateurs doivent avoir accès au groupe de ressources Azure QnA Maker pour accéder à la base de connaissances. Certaines organisations souhaitent externaliser la mise à jour et la gestion des bases de connaissances tout en pouvant quand même protéger l’accès à leurs ressources Azure. Ce modèle éditeur-approbateur s’obtient en configurant deux [services QnA Maker](../How-to/set-up-qnamaker-service-azure.md) identiques dans des abonnements différents et en sélectionnant l’un d’entre eux pour le cycle de modification-test. À la fin des tests, le contenu de la base de connaissances est transféré, par un processus [importation-exportation](../Tutorials/migrate-knowledge-base.md), au service QnA Maker de l’approbateur, qui publiera la base de connaissances et mettra à jour le point de terminaison.



## <a name="active-learning"></a>Apprentissage actif

L’[apprentissage actif](../How-to/use-active-learning.md) est le plus à même de suggérer d’autres questions quand il dispose d’un large éventail qualitatif et quantitatif de requêtes utilisateur. Il est important d’autoriser les requêtes utilisateur des applications clientes à participer à la boucle de commentaires sur l’apprentissage actif sans censure. Une fois les questions suggérées dans le portail QnA Maker, vous pouvez **[appliquer un filtre par suggestions](../How-To/improve-knowledge-base.md#accept-an-active-learning-suggestion-in-the-knowledge-base)**, puis vérifier et accepter, ou rejeter, ces suggestions.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Modifier une base de connaissances](../How-to/edit-knowledge-base.md)
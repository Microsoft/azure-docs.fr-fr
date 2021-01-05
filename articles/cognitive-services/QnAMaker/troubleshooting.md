---
title: Résolution des problèmes - QnA Maker
description: La liste organisée des questions fréquentes concernant le service QnA Maker vous aidera à adopter le service plus rapidement et avec de meilleurs résultats.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: troubleshooting
ms.date: 11/09/2020
ms.openlocfilehash: e8b1d985fcb2852df52382e005ec0f0266e23d9d
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96345642"
---
# <a name="troubleshooting-for-qna-maker"></a>Résolution des problèmes pour QnA Maker

La liste organisée des questions fréquentes concernant le service QnA Maker vous aidera à adopter le service plus rapidement et avec de meilleurs résultats.

<a name="how-to-get-the-qnamaker-service-hostname"></a>

## <a name="manage-predictions"></a>Gérer les prédictions

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

<details>
<summary><b>Comment améliorer les performances de débit pour les prédictions de requête ?</b></summary>

**Réponse** : Les problèmes de performances de débit indiquent que vous devez monter en puissance pour votre service d’application et votre recherche cognitive. Envisagez d’ajouter un réplica à votre recherche cognitive pour améliorer les performances.

Apprenez-en davantage sur les [niveaux tarifaires](Concepts/azure-resources.md).
</details>

<details>
<summary><b>Comment obtenir le point de terminaison du service QnAMaker ?</b></summary>

**Réponse** : Le point de terminaison du service QnAMaker est utile pour le débogage quand vous contactez le support technique de QnAMaker ou UserVoice. Le point de terminaison est une URL qui se présente sous la forme suivante : `https://your-resource-name.azurewebsites.net`.

1. Accédez à votre service QnAMaker (groupe de ressources) dans le [portail Azure](https://portal.azure.com).

    ![Groupe de ressources Azure QnAMaker sur le portail Azure](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-resourcegroup.png)

1. Sélectionnez l’App Service associé à la ressource QnA Maker. Les noms sont généralement identiques.

     ![Sélectionner l’App Service QnAMaker](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-appservice.png)

1. L’URL du point de terminaison est disponible dans la section Vue d’ensemble.

    ![Point de terminaison QnAMaker](./media/qnamaker-how-to-troubleshoot/qnamaker-azure-gethostname.png)

</details>

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

<details>
<summary><b>Comment améliorer les performances de débit pour les prédictions de requête ?</b></summary>

**Réponse** : Les problèmes de performances de débit indiquent que vous devez effectuer un scale-up de votre Recherche cognitive. Envisagez d’ajouter un réplica à votre recherche cognitive pour améliorer les performances.

Apprenez-en davantage sur les [niveaux tarifaires](Concepts/azure-resources.md).
</details>

---

## <a name="manage-the-knowledge-base"></a>Gestion de la base de connaissances

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

<details>
<summary><b>J’ai accidentellement supprimé une partie de mon QnA Maker, que dois-je faire ?</b></summary>

**Réponse** : Ne supprimez pas les services Azure créés avec la ressource QnA Maker, tels que la recherche ou l’application web. Ceux-ci sont nécessaires pour que QnA Maker fonctionne. Si vous en supprimez un, QnA Maker cessera de fonctionne correctement.

Toutes les suppressions sont définitives et s’appliquent notamment aux paires questions et réponses, fichiers, URL, questions et réponses personnalisées, bases de connaissances ou ressources Azure. Assurez-vous d’exporter votre base de connaissances à partir de la page **Paramètres** avant de supprimer la moindre de ses parties.

</details>

<details>
<summary><b>Pourquoi mes URL/mes fichiers n’extraient-ils pas les paires de question-réponse ?</b></summary>

**Réponse** : Il est possible que QnA Maker ne puisse pas extraire automatiquement du contenu question-réponse (QnA) à partir des URL de FAQ valides. Dans ce cas, vous pouvez coller le contenu QnA dans un fichier .txt et voir si l’outil peut l’ingérer. Vous pouvez également ajouter manuellement du contenu à votre base de connaissances par le biais du [portail QnA Maker](https://qnamaker.ai).

</details>

<details>
<summary><b>Quelle taille maximale peut avoir une base de connaissances ?</b></summary>

**Réponse** : La taille de la base de connaissances dépend de la référence SKU de Recherche Azure que vous choisissez lors de la création du service QnA Maker. Lisez plus d’informations [ici](./concepts/azure-resources.md).

</details>

<details>
<summary><b>Pourquoi rien ne s’affiche dans la liste déroulante quand j’essaie de créer une base de connaissances ?</b></summary>

**Réponse** : Vous n’avez encore jamais créé de services QnA Maker dans Azure. Lisez [cet article](./How-To/set-up-qnamaker-service-azure.md) pour apprendre à le faire.

</details>

<details>
<summary><b>Comment partager une base de connaissances avec d’autres utilisateurs ?</b></summary>

**Réponse** : Le partage fonctionne au niveau d’un service QnA Maker, autrement dit, toutes les bases de connaissances dans le service seront partagées. Lisez [ici](./index.yml) comment collaborer sur une base de connaissances.

</details>

<details>
<summary><b>Est-il possible de partager une base de connaissances avec un contributeur qui ne se trouve pas dans le même locataire AAD, dans le but de la modifier ?</b></summary>

**Réponse** : Le partage est basé sur le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Si vous pouvez partager _n’importe quelle_ ressource dans Azure avec un autre utilisateur, vous pouvez également partager QnA Maker.

</details>

<details>
<summary><b>Si vous avez un plan App Service avec cinq bases de connaissances QnAMaker. Est-il possible d’attribuer des droits de lecture/d’écriture à cinq utilisateurs différents afin que chacun d’eux ne puisse accéder qu’à une seule base de connaissances QnAMaker ?</b></summary>

**Réponse** : Vous pouvez partager l’ensemble d’un service QnAMaker, mais pas des bases de connaissances individuelles.

</details>

<details>
<summary><b>Comment puis-je modifier le message par défaut si aucune bonne correspondance n’est trouvée ?</b></summary>

**Réponse** : Le message par défaut fait partie des paramètres dans votre App Service.
- Accédez à votre ressource App Service dans le portail Azure

![appservice qnamaker](./media/qnamaker-faq/qnamaker-resource-list-appservice.png)
- Cliquez sur l’option **Paramètres**

![paramètres d’appservice qnamaker](./media/qnamaker-faq/qnamaker-appservice-settings.png)
- Modifiez la valeur du paramètre **DefaultAnswer**
- Redémarrer votre App service

![redémarrer appservice qnamaker](./media/qnamaker-faq/qnamaker-appservice-restart.png)


</details>

<details>
<summary><b>Pourquoi mon lien SharePoint n’est-il pas extrait ?</b></summary>

**Réponse** : Pour plus d’informations, consultez la section [Emplacements des sources de données](./concepts/data-sources-and-content.md#data-source-locations).

</details>

<details>
<summary><b>Les mises à jour apportées à ma base de connaissances ne sont pas reflétées lors de la publication. Pourquoi ?</b></summary>

**Réponse** : Toute opération de modification, qu’elle soit effectuée dans une mise à jour de table, un test ou un paramètre, doit être enregistrée avant d’être publiée. Veillez à cliquer sur le bouton **Enregistrer et entraîner** après chaque opération de modification.

</details>

<details>
<summary><b>La base de connaissances prend-elle en charge les données enrichies ou le contenu multimédia ?</b></summary>

**Réponse** :

#### <a name="multimedia-auto-extraction-for-files-and-urls"></a>Extraction automatique de contenu multimédia pour les fichiers et URL

* URL - Fonctionnalités de conversion HTML en Markdown limitées.
* Fichiers - Non pris en charge

#### <a name="answer-text-in-markdown"></a>Texte de réponse dans Markdown
Une fois que les paires de Q/R sont dans la base de connaissances, vous pouvez modifier le texte Markdown d’une réponse de manière à inclure des liens vers des médias disponibles depuis des URL publiques.


</details>

<details>
<summary><b>QnA Maker prend-il en charge les langues autres que l’anglais ?</b></summary>

**Réponse** : Affichez plus d’informations sur les [langues prises en charge](./overview/language-support.md).

Si vous avez du contenu dans plusieurs langues, veillez à créer un service distinct pour chaque langue.

</details>

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

<details>
<summary><b>Pourquoi mes URL/mes fichiers n’extraient-ils pas les paires de question-réponse ?</b></summary>

**Réponse** : Il est possible que QnA Maker ne puisse pas extraire automatiquement du contenu question-réponse (QnA) à partir des URL de FAQ valides. Dans ce cas, vous pouvez coller le contenu QnA dans un fichier .txt et voir si l’outil peut l’ingérer. Vous pouvez également ajouter manuellement du contenu à votre base de connaissances par le biais du [portail QnA Maker](https://qnamaker.ai).

</details>

<details>
<summary><b>Quelle taille maximale peut avoir une base de connaissances ?</b></summary>

**Réponse** : La taille de la base de connaissances dépend de la référence SKU de Recherche Azure que vous choisissez lors de la création du service QnA Maker. Lisez plus d’informations [ici](./concepts/azure-resources.md).

</details>

<details>
<summary><b>Pourquoi rien ne s’affiche dans la liste déroulante quand j’essaie de créer une base de connaissances ?</b></summary>

**Réponse** : Vous n’avez encore jamais créé de services QnA Maker dans Azure. Lisez [cet article](./How-To/set-up-qnamaker-service-azure.md) pour apprendre à le faire.

</details>

<details>
<summary><b>Comment partager une base de connaissances avec d’autres utilisateurs ?</b></summary>

**Réponse** : Le partage fonctionne au niveau d’un service QnA Maker, autrement dit, toutes les bases de connaissances dans le service seront partagées. Lisez [ici](./index.yml) comment collaborer sur une base de connaissances.

</details>

<details>
<summary><b>Est-il possible de partager une base de connaissances avec un contributeur qui ne se trouve pas dans le même locataire Azure Active Directory, dans le but de la modifier ?</b></summary>

**Réponse** : Le partage est basé sur le contrôle d’accès en fonction du rôle Azure (Azure RBAC). Si vous pouvez partager _n’importe quelle_ ressource dans Azure avec un autre utilisateur, vous pouvez également partager QnA Maker.

</details>

<details>
<summary><b>Est-il possible d’attribuer des droits de lecture/écriture à 5 utilisateurs différents afin que chacun d’eux ne puisse accéder qu’à une seule base de connaissances QnAMaker ?</b></summary>

**Réponse** : Vous pouvez partager l’ensemble d’un service QnAMaker, mais pas des bases de connaissances individuelles.

</details>

<details>
<summary><b>Pourquoi mon lien SharePoint n’est-il pas extrait ?</b></summary>

**Réponse** : Pour plus d’informations, consultez la section [Emplacements des sources de données](./concepts/data-sources-and-content.md#data-source-locations).

</details>

<details>
<summary><b>Les mises à jour apportées à ma base de connaissances ne sont pas reflétées lors de la publication. Pourquoi ?</b></summary>

**Réponse** : Toute opération de modification, qu’elle soit effectuée dans une mise à jour de table, un test ou un paramètre, doit être enregistrée avant d’être publiée. Veillez à cliquer sur le bouton **Enregistrer et entraîner** après chaque opération de modification.

</details>

<details>
<summary><b>La base de connaissances prend-elle en charge les données enrichies ou le contenu multimédia ?</b></summary>

**Réponse** :

#### <a name="multimedia-auto-extraction-for-files-and-urls"></a>Extraction automatique de contenu multimédia pour les fichiers et URL

* URL - Fonctionnalités de conversion HTML en Markdown limitées.
* Fichiers - Non pris en charge

#### <a name="answer-text-in-markdown"></a>Texte de réponse dans Markdown
Une fois que les paires de Q/R sont dans la base de connaissances, vous pouvez modifier le texte Markdown d’une réponse de manière à inclure des liens vers des médias disponibles depuis des URL publiques.


</details>

<details>
<summary><b>QnA Maker prend-il en charge les langues autres que l’anglais ?</b></summary>

**Réponse** : Affichez plus d’informations sur les [langues prises en charge](./overview/language-support.md).

Si vous avez du contenu dans plusieurs langues, veillez à créer un service distinct pour chaque langue.

</details>

---

## <a name="manage-service"></a>Gérer le service

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

<details>
<summary><b>Quand dois-je redémarrer mon instance d’App Service ?</b></summary>

**Réponse** : Actualisez votre instance d’App Service lorsque l’icône d’avertissement apparaît en regard du numéro de version de la base de connaissances dans le tableau **Clés de point de terminaison** de la **page** [Paramètres utilisateur](https://www.qnamaker.ai/UserSettings).

</details>

<details>
<summary><b>J’ai supprimé mon service de recherche existant. Comment puis-je résoudre ce problème ?</b></summary>

**Réponse** : Si vous supprimez un index Recherche cognitive Azure, l’opération est définitive et l’index ne peut pas être récupéré.

</details>

<details>
<summary><b>J’ai supprimé l’index `testkb` de mon service de recherche. Comment puis-je résoudre ce problème ?</b></summary>

**Réponse** : Vos anciennes données ne peuvent pas être récupérées. Créez une ressource QnA Maker, puis recréez votre base de connaissances.

</details>

<details>
<summary><b>Quand dois-je actualiser mes clés de point de terminaison ?</b></summary>

**Réponse** : Actualisez vos clés de point de terminaison si vous pensez qu’elles ont été compromises.

</details>

<details>
<summary><b>Puis-je utiliser la même ressource Recherche cognitive Azure pour des bases de connaissances utilisant plusieurs langues ?</b></summary>

**Réponse** : Pour utiliser plusieurs langues et plusieurs bases de connaissances, l’utilisateur doit créer une ressource QnA Maker pour chaque langue. Cette opération crée un service de recherche Azure distinct par langue. La combinaison de bases de connaissances en différentes langues dans un même service de recherche Azure entraîne une détérioration de la pertinence des résultats.

</details>

<details>
<summary><b>Comment changer le nom de la ressource Recherche cognitive Azure utilisée par QnA Maker ?</b></summary>

**Réponse** : Le nom de la ressource Recherche cognitive Azure est le nom de ressource QnA Maker avec des lettres aléatoires ajoutées à la fin. De ce fait, il est difficile de faire la distinction entre plusieurs ressources de recherche pour QnA Maker. Créez un service de recherche distinct (en le nommant comme vous le souhaitez), puis connectez-le à votre service QnA. La procédure est similaire à celle que vous devez effectuer pour [mettre à niveau une Recherche Azure](How-To/set-up-qnamaker-service-azure.md#upgrade-the-azure-cognitive-search-service).

</details>

<details>
<summary><b>Quand QnA Maker retourne `Runtime core is not initialized,`, comment résoudre le problème ?</b></summary>

**Réponse** : L’espace disque pour votre App Service est peut-être plein. Étapes pour corriger l’espace disque :

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez le service App Service de votre QnA Maker, puis arrêtez le service.
1. Tout en restant sur App Service, sélectionnez **Outils de développement**, **Outils avancés**, puis **OK**. Une nouvelle fenêtre de navigateur s’ouvre.
1. Sélectionnez **Console de débogage**, puis **CMD** pour ouvrir un outil en ligne de commande.
1. Accédez au répertoire _site/wwwroot/Data/QnAMaker/_ .
1. Supprimez tous les dossiers dont le nom commence par `rd`.

    **Ne supprimez pas** les éléments suivants :

    * Fichier KbIdToRankerMappings.txt
    * Fichier EndpointSettings.json
    * Dossier EndpointKeys

1. Démarrez App Service.
1. Accédez à votre base de connaissances pour vérifier qu’elle fonctionne maintenant.

</details>

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)


<details>
<summary><b>J’ai supprimé mon service de recherche existant. Comment puis-je résoudre ce problème ?</b></summary>

**Réponse** : Si vous supprimez un index Recherche cognitive Azure, l’opération est définitive et l’index ne peut pas être récupéré.

</details>

<details>
<summary><b>J’ai supprimé l’index `testkb` de mon service de recherche. Comment puis-je résoudre ce problème ?</b></summary>

**Réponse** : Vos anciennes données ne peuvent pas être récupérées. Créez une ressource QnA Maker, puis recréez votre base de connaissances.

</details>

<details>
<summary><b>Puis-je utiliser la même ressource Recherche cognitive Azure pour des bases de connaissances utilisant plusieurs langues ?</b></summary>

**Réponse** : Pour utiliser plusieurs langues et plusieurs bases de connaissances, l’utilisateur doit créer une ressource QnA Maker pour chaque langue. Cette opération crée un service de recherche Azure distinct par langue. La combinaison de bases de connaissances en différentes langues dans un même service de recherche Azure entraîne une détérioration de la pertinence des résultats.

</details>

<details>
<summary><b>Comment changer le nom de la ressource Recherche cognitive Azure utilisée par QnA Maker ?</b></summary>

**Réponse** : Le nom de la ressource Recherche cognitive Azure est le nom de ressource QnA Maker avec des lettres aléatoires ajoutées à la fin. De ce fait, il est difficile de faire la distinction entre plusieurs ressources de recherche pour QnA Maker. Créez un service de recherche distinct (en le nommant comme vous le souhaitez), puis connectez-le à votre service QnA. La procédure est similaire à celle que vous devez effectuer pour [mettre à niveau une Recherche Azure](How-To/set-up-qnamaker-service-azure.md#upgrade-the-azure-cognitive-search-service).

</details>

---

## <a name="integrate-with-other-services-including-bots"></a>Intégration aux autres services tels que les bots

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

<details>
<summary><b>Dois-je utiliser Bot Framework pour pouvoir utiliser QnA Maker ?</b></summary>

**Réponse** : Non, vous n’avez pas besoin d’utiliser [Bot Framework](https://github.com/Microsoft/botbuilder-dotnet) avec QnA Maker. Toutefois, QnA Maker est proposé parmi plusieurs modèles dans [Azure Bot Service](/azure/bot-service/?preserve-view=true&view=azure-bot-service-4.0). Bot Service permet le développement rapide de bot intelligent via Microsoft Bot Framework et s’exécute dans un environnement serverless.

</details>

<details>
<summary><b>Comment puis-je créer un bot avec QnA Maker ?</b></summary>

**Réponse** : Suivez les instructions de [cette](./Quickstarts/create-publish-knowledge-base.md) documentation pour créer votre bot avec Azure Bot Service.

</details>

<details>
<summary><b>Comment faire pour utiliser une base de connaissances différente avec un Azure Bot Service existant ?</b></summary>

**Réponse** : Vous devez disposer des informations suivantes concernant votre base de connaissances :

* ID de la base de connaissances.
* Nom de sous-domaine personnalisé du point de terminaison publié de la base de connaissances, appelé `host` et présent dans la page **Paramètres** après la publication.
* Clé de point de terminaison publié de la base de connaissances : affiché sur la page **Paramètres** après la publication.

Avec ces informations, accédez au service d’application de votre bot sur le portail Azure. Sous **Paramètres -> Configuration -> paramètre d’application**, modifiez les valeurs suivantes.

La clé de point de terminaison de la base de connaissances est étiquetée `QnAAuthkey` dans le service ABS.

</details>

<details>
<summary><b>Plusieurs applications clientes peuvent-elles partager une même base de connaissances ?</b></summary>

**Réponse** : Oui, la base de connaissances peut être interrogée par un nombre quelconque de clients. Si la réponse de la base de connaissances est lente ou expire, envisagez s’élever niveau du service d’application associé à la base de connaissances.

</details>

<details>
<summary><b>Comment incorporer le service QnA Maker dans mon site web ?</b></summary>

**Réponse** : Procédez comme suit pour incorporer le service QnA Maker en tant que contrôle de conversation web dans votre site web :

1. Créez votre bot de FAQ en suivant les instructions [ici](./Quickstarts/create-publish-knowledge-base.md).
2. Activez la conversation web en suivant [ces](/azure/bot-service/bot-service-channel-connect-webchat) étapes

</details>

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)


<details>
<summary><b>Dois-je utiliser Bot Framework pour pouvoir utiliser QnA Maker ?</b></summary>

**Réponse** : Non, vous n’avez pas besoin d’utiliser [Bot Framework](https://github.com/Microsoft/botbuilder-dotnet) avec QnA Maker. Toutefois, QnA Maker est proposé parmi plusieurs modèles dans [Azure Bot Service](/azure/bot-service/?preserve-view=true&view=azure-bot-service-4.0). Bot Service permet le développement rapide de bot intelligent via Microsoft Bot Framework et s’exécute dans un environnement serverless.

</details>

<details>
<summary><b>Comment puis-je créer un bot avec QnA Maker ?</b></summary>

**Réponse** : Suivez les instructions de [cette](./Quickstarts/create-publish-knowledge-base.md) documentation pour créer votre bot avec Azure Bot Service.

</details>

<details>
<summary><b>Comment faire pour utiliser une base de connaissances différente avec un Azure Bot Service existant ?</b></summary>

**Réponse** : Vous devez disposer des informations suivantes concernant votre base de connaissances :

* ID de la base de connaissances.
* Nom de sous-domaine personnalisé du point de terminaison publié de la base de connaissances, appelé `host` et présent dans la page **Paramètres** après la publication.
* Clé de point de terminaison publié de la base de connaissances : affiché sur la page **Paramètres** après la publication.

Avec ces informations, accédez au service d’application de votre bot sur le portail Azure. Sous **Paramètres -> Configuration -> paramètre d’application**, modifiez les valeurs suivantes.

La clé de point de terminaison de la base de connaissances est étiquetée `QnAAuthkey` dans le service ABS.

</details>

<details>
<summary><b>Plusieurs applications clientes peuvent-elles partager une même base de connaissances ?</b></summary>

**Réponse** : Oui, la base de connaissances peut être interrogée par un nombre quelconque de clients. Si la réponse de la base de connaissances est lente ou expire, envisagez s’élever niveau du service d’application associé à la base de connaissances.

</details>

<details>
<summary><b>Comment incorporer le service QnA Maker dans mon site web ?</b></summary>

**Réponse** : Procédez comme suit pour incorporer le service QnA Maker en tant que contrôle de conversation web dans votre site web :

1. Créez votre bot de FAQ en suivant les instructions [ici](./Quickstarts/create-publish-knowledge-base.md).
2. Activez la conversation web en suivant [ces](/azure/bot-service/bot-service-channel-connect-webchat) étapes

---

## <a name="data-storage"></a>Stockage des données

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

<details>
<summary><b>Où se trouve l’emplacement de stockage et quelles données sont stockées ?</b></summary>

**Réponse** :

Lorsque vous avez créé votre service QnA Maker, vous avez sélectionné une région Azure. Vos bases de connaissances et vos fichiers journaux sont stockés dans cette région.

</details>

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

<details>
<summary><b>Où se trouve l’emplacement de stockage et quelles données sont stockées ?</b></summary>

**Réponse** :

Lorsque vous avez créé votre service QnA Maker, vous avez sélectionné une région Azure. Vos bases de connaissances et vos fichiers journaux sont stockés dans cette région.

</details>

---
---
title: Guide pratique pour tester une base de connaissances - QnA Maker
description: Le test de votre base de connaissances QnA Maker est une partie importante d’un processus itératif afin d’améliorer la précision des réponses retournées. Vous pouvez tester la base de connaissances via une interface de conversation améliorée qui vous permet également d’apporter des modifications.
ms.service: cognitive-services
ms.subservice: qna-maker
ms.topic: conceptual
ms.date: 11/09/2020
ms.openlocfilehash: 273548ec095ce04772438a2d732b914d80d976cc
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96353151"
---
# <a name="test-your-knowledge-base-in-qna-maker"></a>Tester votre base de connaissances dans QnA Maker

Le test de votre base de connaissances QnA Maker est une partie importante d’un processus itératif afin d’améliorer la précision des réponses retournées. Vous pouvez tester la base de connaissances via une interface de conversation améliorée qui vous permet également d’apporter des modifications.

## <a name="interactively-test-in-qna-maker-portal"></a>Tester de manière interactive dans le portail QnA Maker

# <a name="qna-maker-ga-stable-release"></a>[QnA Maker GA (version stable)](#tab/v1)

1. Accédez à votre base de connaissances en sélectionnant son nom dans la page **Mes bases de connaissances**.
1. Pour accéder au panneau déroulant Test, sélectionnez **Test** dans le panneau supérieur de votre application.
1. Entrez une question dans la zone de texte, puis sélectionnez Entrée.
1. La réponse la plus concordante de la base de connaissances est retournée.

### <a name="clear-test-panel"></a>Effacer le panneau Test

Pour effacer toutes les questions de test entrées et leurs résultats de la console de test, sélectionnez **Recommencer** dans l’angle supérieur gauche du panneau Test.

### <a name="close-test-panel"></a>Fermer le panneau Test

Pour fermer le panneau Test, sélectionnez le bouton **Test** à nouveau. Tant que le panneau Test est ouvert, vous ne pouvez pas modifier le contenu de la base de connaissances.

### <a name="inspect-score"></a>Inspecter le score

Vous inspectez les détails du résultat du test dans le panneau Inspecter.

1.  Avec le panneau déroulant Test ouvert, sélectionnez **Inspecter** pour obtenir plus d’informations sur cette réponse.

    ![Inspecter des réponses](../media/qnamaker-how-to-test-knowledge-bases/inspect.png)

2.  Le panneau Inspection s’affiche. Le panneau inclut l’intention de score la plus élevée, ainsi que toutes les entités identifiées. Le panneau affiche le résultat de l’énoncé sélectionné.

### <a name="correct-the-top-scoring-answer"></a>Corriger la réponse avec le score le plus élevé

Si la réponse avec le score le plus élevé est incorrecte, sélectionnez la réponse correcte dans la liste et sélectionnez **Enregistrer et former**.

![Corriger la réponse avec le score le plus élevé](../media/qnamaker-how-to-test-knowledge-bases/choose-answer.png)

### <a name="add-alternate-questions"></a>Ajouter des questions alternatives

Vous pouvez ajouter des formes alternatives d’une question à une réponse donnée. Tapez les réponses alternatives dans la zone de texte et sélectionnez Entrée pour les ajouter. Sélectionnez **Enregistrer et former** pour stocker les mises à jour.

![Ajouter des questions alternatives](../media/qnamaker-how-to-test-knowledge-bases/add-alternate-question.png)

### <a name="add-a-new-answer"></a>Ajouter une nouvelle réponse

Vous pouvez ajouter une nouvelle réponse si les réponses existantes qui ont été mises en correspondance sont incorrectes ou si la réponse n’existe pas dans la base de connaissances (aucune bonne correspondance trouvée dans la base de connaissances).

En bas de la liste de réponses, utilisez la zone de texte pour entrer une nouvelle réponse, et appuyez sur Entrée pour l’ajouter.

Sélectionnez **Enregistrer et former** pour conserver cette réponse. Une nouvelle paire question/réponse a maintenant été ajoutée à votre base de connaissances.

> [!NOTE]
> Toutes les modifications apportées à votre base de connaissances ne sont enregistrées que lorsque vous appuyez sur le bouton **Enregistrer et former**.

# <a name="qna-maker-managed-preview-release"></a>[QnA Maker managé (préversion)](#tab/v2)

1. Accédez à votre base de connaissances en sélectionnant son nom dans la page **Mes bases de connaissances**.
1. Pour accéder au panneau déroulant Test, sélectionnez **Test** dans le panneau supérieur de votre application. 
1. Vous allez voir une case à cocher en haut nommée **Display short answer** (Afficher une réponse brève), qui est sélectionnée par défaut. Cette option permet d’activer la détection de l’étendue des réponses basée sur MRC dans votre volet de test. 
1. Entrez une question dans la zone de texte, puis sélectionnez Entrée. 
1. Pour chaque requête, s’il existe une réponse précise/brève dans le passage de la réponse, ainsi que dans le passage de la réponse le mieux adapté, présent dans la base de connaissances, vous aurez également une réponse brève pour votre requête.
    ![Volet de test activé (version managée)](../media/qnamaker-how-to-test-knowledge-bases/test-pane-with-managed-detail.png)
1. Si vous décochez **Display short answer** (Afficher une réponse brève), seul le passage de la réponse le mieux adapté issu de la base de connaissances est retourné en tant que réponse.

### <a name="clear-test-panel"></a>Effacer le panneau Test

Pour effacer toutes les questions de test entrées et leurs résultats de la console de test, sélectionnez **Recommencer** dans l’angle supérieur gauche du panneau Test.

### <a name="close-test-panel"></a>Fermer le panneau Test

Pour fermer le panneau Test, sélectionnez le bouton **Test** à nouveau. Tant que le panneau Test est ouvert, vous ne pouvez pas modifier le contenu de la base de connaissances.

### <a name="inspect-score"></a>Inspecter le score

Vous inspectez les détails du résultat du test dans le panneau Inspecter.

1.  Avec le panneau déroulant Test ouvert, sélectionnez **Inspecter** pour obtenir plus d’informations sur cette réponse.

    ![Inspecter les réponses (préversion)](../media/qnamaker-how-to-test-knowledge-bases/inspect-with-managed.png)

2.  Le panneau Inspection s’affiche. Le panneau inclut l’intention de score la plus élevée, ainsi que toutes les entités identifiées. Le panneau affiche le résultat de l’énoncé sélectionné.
3. Le panneau indique le score de confiance du passage de la réponse et le score de l’étendue des réponses détectée.

### <a name="correct-the-top-scoring-answer"></a>Corriger la réponse avec le score le plus élevé

Si la réponse avec le score le plus élevé est incorrecte, sélectionnez la réponse correcte dans la liste et sélectionnez **Enregistrer et former**.

![Corriger la réponse avec le score le plus élevé (préversion)](../media/qnamaker-how-to-test-knowledge-bases/choose-answer-managed.png)

### <a name="add-alternate-questions"></a>Ajouter des questions alternatives

Vous pouvez ajouter des formes alternatives d’une question à une réponse donnée. Tapez les réponses alternatives dans la zone de texte et sélectionnez Entrée pour les ajouter. Sélectionnez **Enregistrer et former** pour stocker les mises à jour.

![Ajouter des questions alternatives (préversion)](../media/qnamaker-how-to-test-knowledge-bases/add-alternate-question-with-managed.png)

### <a name="add-a-new-answer"></a>Ajouter une nouvelle réponse

Vous pouvez ajouter une nouvelle réponse si les réponses existantes qui ont été mises en correspondance sont incorrectes ou si la réponse n’existe pas dans la base de connaissances (aucune bonne correspondance trouvée dans la base de connaissances).

En bas de la liste de réponses, utilisez la zone de texte pour entrer une nouvelle réponse, et appuyez sur Entrée pour l’ajouter.

Sélectionnez **Enregistrer et former** pour conserver cette réponse. Une nouvelle paire question/réponse a maintenant été ajoutée à votre base de connaissances.

---

### <a name="test-the-published-knowledge-base"></a>Tester la base de connaissances publiée

Vous pouvez tester la version publiée de la base de connaissances dans le volet de test. Une fois que vous avez publié la base de connaissances, sélectionnez la zone **Base de connaissances publiée** et envoyez une requête pour obtenir des résultats à partir de la base de connaissances publiée.

![Tester par rapport à une base de connaissances publiée](../media/qnamaker-how-to-test-knowledge-bases/test-against-published-knowledge-base.png)

## <a name="batch-test-with-tool"></a>Test par lot avec l’outil

Utilisez l’outil de test par lot lorsque vous souhaitez :

* déterminer la meilleure réponse et le meilleur score pour un ensemble de questions
* valider la réponse attendue pour l’ensemble de questions

### <a name="prerequisites"></a>Prérequis

* Abonnement Azure : [créez-en un gratuitement](https://azure.microsoft.com/free/cognitive-services/)
* [Créez un service QnA Maker](../Quickstarts/create-publish-knowledge-base.md) ou utilisez un service existant, qui utilise la langue anglaise.
* Téléchargez l’[exemple de fichier multitour `.docx`](https://github.com/Azure-Samples/cognitive-services-sample-data-files/blob/master/qna-maker/data-source-formats/multi-turn.docx)
* Téléchargez l’[outil de test par lot](https://aka.ms/qnamakerbatchtestingtool), puis extrayez le fichier exécutable du fichier `.zip`.

### <a name="sign-into-qna-maker-portal"></a>Se connecter au portail QnA Maker

[Connectez-vous](https://www.qnamaker.ai/) au portail QnA Maker.

### <a name="create-a-new-knowledge-base-from-the-multi-turn-sampledocx-file"></a>Créer une base de connaissances à partir de l’exemple de fichier multitour .docx

1. Sélectionnez **Create a knowledge base** (Créer une base de connaissances) dans la barre d’outils.
1. Ignorez l’**étape 1**, car vous devez déjà disposer d’une ressource QnA Maker. Passez à **étape 2** pour sélectionner les informations sur votre ressource existante :
    * ID Azure Active Directory
    * Nom de l’abonnement Azure
    * Nom du service Azure QnA
    * Langue (anglais)
1. Entrez `Multi-turn batch test quickstart` pour le nom de votre base de connaissances.

1. À l’**étape 4**, configurez les paramètres à l’aide du tableau suivant :

    |Paramètre|Valeur|
    |--|--|
    |**Enable multi-turn extraction from URLs, .pdf or .docx files** (Activer l’extraction multitour à partir d’URL, de fichier .pdf ou .docx)|Activé|
    |**Texte de réponse par défaut**| `Batch test - default answer not found.`|
    |**+ Ajouter un fichier**|Sélectionnez la liste des fichiers `.docx` téléchargés dans le cadre des prérequis.|
    |**Échange de conversation (chit-chat)**|Sélectionnez **Professional**.|

1. À l’**étape 5**, sélectionnez **Create your KB** (Créer votre base de connaissances).

    À l’issue du processus de création, le portail affiche la base de connaissances modifiable.

### <a name="save-train-and-publish-knowledge-base"></a>Enregistrer, entraîner et publier la base de connaissances

1. Sélectionnez **Save and Train** (Enregistrer et entraîner) dans la barre d’outils pour enregistrer la base de connaissances.
1. Sélectionnez **Publish** (Publier) dans la barre d’outils, puis sélectionnez à nouveau **Publish** pour publier la base de connaissances. La publication rend la base de connaissances accessible aux requêtes en provenance d’un point de terminaison d’URL public. Une fois la publication terminée, enregistrez les informations d’URL de l’hôte et de clé du point de terminaison figurant dans la page **Publish** (Publier).

    |Données nécessaires| Exemple|
    |--|--|
    |Hôte publié|`https://YOUR-RESOURCE-NAME.azurewebsites.net`|
    |Clé publiée|`XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX` (chaîne de 32 caractères située après `Endpoint`)|
    |ID d'application|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (chaîne de 36 caractères figurant dans `POST`) |

### <a name="create-batch-test-file-with-question-ids"></a>Créer un fichier de test par lot avec des ID de question

Pour pouvoir utiliser l’outil de test par lot, créez un fichier sous le nom `batch-test-data-1.tsv` avec un éditeur de texte. Le fichier doit être au format UTF-8 et contenir les colonnes suivantes, séparées par une tabulation.

|Champs du fichier d'entrée TSV|Notes|Exemple|
|--|--|--|
|ID de base de connaissances|ID de votre base de connaissances figurant dans la page Publish (Publier). Testez plusieurs bases de connaissances dans le même service en une seule opération dans un même fichier en utilisant différents ID de base de connaissances dans un seul fichier.|`xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` (chaîne de 36 caractères figurant dans `POST`) |
|Question|Texte de la question entré par un utilisateur. 1 000 caractères au maximum.|`How do I sign out?`|
|Balises de métadonnées|facultatif|`topic:power` utilise le format `key:value`|
|Meilleur paramètre|facultatif|`25`|
|ID de la réponse attendue|facultatif|`13`|

Pour cette base de connaissances, ajoutez au fichier trois lignes des deux colonnes obligatoires. La première colonne est l’ID de votre base de connaissances et la deuxième doit contenir la liste de questions suivante :

|Colonne 2 – questions|
|--|
|`Use Windows Hello to sign in`|
|`Charge your Surface Pro 4`|
|`Get to know Windows 10`|

La formulation de ces questions étant exactement la même que la base de connaissances, le score de confiance retourné doit être égal à 100.

Ensuite, ajoutez quelques questions similaires, mais pas tout à fait identiques, sur trois lignes supplémentaires, en utilisant le même ID de base de connaissances :

|Colonne 2 – questions|
|--|
|`What is Windows Hello?`|
|`How do I charge the laptop?`|
|`What features are in Windows 10?`|

> [!CAUTION]
> Veillez à ce que chaque colonne soit séparée par une tabulation uniquement. Des espaces de début ou de fin sont ajoutés aux données de colonne et conduisent le programme à lever des exceptions quand le type ou la taille sont incorrects.

Quand il est ouvert dans Excel, le fichier de test par lot se présente comme dans l’image suivante. L’ID de base de connaissances a été remplacé par `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` pour des raisons de sécurité. Pour votre propre test par lot, veillez à ce que la colonne affiche l’ID de votre base de connaissances.

> [!div class="mx-imgBorder"]
> ![Première version en entrée du fichier .tsv du test par lot](../media/batch-test/batch-test-1-input.png)

### <a name="test-the-batch-file"></a>Tester le fichier par lot

Exécutez le programme de test par lot en utilisant le format CLI suivant sur la ligne de commande.

Remplacez `YOUR-RESOURCE-NAME` et `ENDPOINT-KEY` par vos propres valeurs pour le nom du service et la clé du point de terminaison. Ces valeurs se trouvent dans la page **Settings** (Paramètres) du portail QnA Maker.

```console
batchtesting.exe batch-test-data-1.tsv https://YOUR-RESOURCE-NAME.azurewebsites.net ENDPOINT-KEY out.tsv
```
Le test se termine et génère le fichier `out.tsv` :

> [!div class="mx-imgBorder"]
> ![Première version en sortie du fichier .tsv du test par lot](../media/batch-test/batch-test-1-output.png)

L’ID de base de connaissances a été remplacé par `xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx` pour des raisons de sécurité. Pour votre propre test par lot, la colonne affiche l’ID de votre base de connaissances.

Dans la quatrième colonne, le score de confiance en sortie du test, les trois premières questions ont retourné un score de 100 comme prévu, car chaque question est exactement la même que dans la base de connaissances. Les trois dernières questions, du fait de leur nouvelle formulation, ne retournent pas le score de confiance 100. Pour accroître le score à la fois pour le test et pour vos utilisateurs, vous devez ajouter des questions supplémentaires à la base de connaissances.

### <a name="testing-with-the-optional-fields"></a>Test avec les champs facultatifs

Une fois que vous avez compris le format et le processus, vous pouvez générer un fichier de test de façon à l’exécuter sur votre base de connaissances à partir d’une source de données (p.ex., des journaux de conversations).

Sachant que la source de données et le processus sont automatisés, le fichier de test peut être exécuté plusieurs fois avec des paramètres différents pour trouver les bonnes valeurs.

Par exemple, si vous disposez d’un journal de conversation et que vous souhaitez identifier le texte du journal de conversation qui s’applique aux différents champs de métadonnées, créez un fichier de test et définissez les champs de métadonnées pour chaque ligne. Exécutez le test, puis examinez les lignes qui correspondent aux métadonnées. En règle générale, les correspondances sont positives, mais il est préférable de passer en revue les résultats pour vérifier qu’il n’y a pas de faux positifs. Un faux positif est une ligne qui correspond aux métadonnées, mais qui, au vu du texte, ne devrait pas correspondre.

### <a name="using-optional-fields-in-the-input-batch-test-file"></a>Utilisation de champs facultatifs dans le fichier de test par lot d’entrée

Aidez-vous du tableau suivant pour savoir comment trouver les valeurs de champs pour les données facultatives.

|Numéro de colonne|Colonne facultative|Emplacement des données|
|--|--|--|
|3|metadata|Exportez la base de connaissances existante pour les paires `key:value` existantes.|
|4|top|La valeur par défaut `25` est recommandée.|
|5|Question and answer set ID|Exportez la base de connaissances existante pour obtenir les valeurs d’ID. Notez aussi que les ID ont été retournés dans le fichier de sortie.|

### <a name="add-metadata-to-the-knowledge-base"></a>Ajouter des métadonnées à la base de connaissances

1. Sur le portail QnA, dans la page **Edit** (Modifier), ajoutez les métadonnées `topic:power` aux questions suivantes :

    |Questions|
    |--|
    |Charge your Surface Pro 4 (Charger votre Surface Pro 4)|
    |Check the battery level (Vérifiez le niveau de la batterie)|

    Les métadonnées sont définies pour deux paires de Q/R.

    > [!TIP]
    > Pour voir les métadonnées et les ID de questions-réponses de chaque série, exportez la base de connaissances. Sélectionnez la page **Settings** (Paramètres), puis **Export** (Exporter) en tant que fichier `.xls`. Recherchez ce fichier téléchargé et ouvrez-le dans Excel pour examiner les métadonnées et les ID.

1. Sélectionnez **Save and train** (Enregistrer et entraîner) puis, dans la page **Publish** (Publier), cliquez sur le bouton **Publish** (Publier). Ces actions rendent la modification accessible au test par lot. Téléchargez la base de connaissances à partir de la page **Settings** (Paramètres).

    Le fichier téléchargé présente le bon format pour les métadonnées ainsi que l’ID de série de questions et réponses approprié. Utiliser ces champs dans la section suivante

    > [!div class="mx-imgBorder"]
    > ![Base de connaissances exportée avec les métadonnées](../media/batch-test/exported-knowledge-base-with-metadata.png)

### <a name="create-a-second-batch-test"></a>Créer un deuxième test par lot

Il existe deux scénarios principaux pour le test par lot :
* **Traiter des fichiers de journaux de conversations** – Déterminez la meilleure réponse pour une question qui n’a pas encore été vue – Le cas le plus courant est lorsque vous devez traiter un fichier journal de requêtes, comme les questions utilisateur d’un bot conversationnel. Créez un test de fichier par lot, avec uniquement les colonnes obligatoires. Le test retourne la meilleure réponse pour chaque question. Cela ne signifie pas que la meilleure réponse est la bonne réponse. Une fois que vous avez terminé ce test, passez au test de validation.
* **Test de validation** – Validez la réponse attendue. Pour ce test, l’ensemble des questions et des réponses attendues correspondantes figurant dans le test par lot doivent avoir été validées. Cela peut nécessiter un traitement manuel.

La procédure suivante suppose que le scénario consiste à traiter les journaux de conversations avec

1. Créez un fichier de test par lot pour inclure des données facultatives, `batch-test-data-2.tsv`. Ajoutez les six lignes du fichier d’entrée de test par lot d’origine, puis ajoutez les colonnes medatada, top et QnA pair ID pour chaque ligne.

    Pour simuler le processus automatisé de vérification de texte nouveau dans les journaux de conversations par rapport à la base de connaissances, attribuez aux métadonnées de chaque colonne la même valeur : `topic:power`.

    > [!div class="mx-imgBorder"]
    > ![Deuxième version en entrée du fichier .tsv du test par lot](../media/batch-test/batch-test-2-input.png)

1. Réexécutez le test en changeant les noms de fichiers d’entrée et de sortie pour indiquer qu’il s’agit du deuxième test.

    > [!div class="mx-imgBorder"]
    > ![Deuxième version en sortie du fichier .tsv du test par lot](../media/batch-test/batch-test-2-output.png)

### <a name="test-results-and-an-automated-test-system"></a>Résultats du test et système de test automatisé

Ce fichier de sortie de test peut être analysé dans le cadre d’un pipeline de test continu automatisé.

Cette sortie de test spécifique doit être interprétée comme suit : chaque ligne a été filtrée avec des métadonnées, et comme toutes les lignes ne correspondaient pas aux métadonnées de la base de connaissances, la réponse par défaut retournée pour les lignes sans correspondance a été « no good match found in kb » (aucune correspondance exacte trouvée dans la base de connaissances). Parmi les lignes qui correspondaient, l’ID de question-réponse et le score étaient retournés.

Toutes les lignes ont retourné l’étiquette « incorrect », car aucune ligne ne correspondait à l’ID de réponse attendu.

Ces résultats montrent que vous pouvez prendre un journal de conversation et utiliser le texte comme requête de chaque ligne. Sans rien connaître des données, les résultats vous en disent long sur les données que vous pouvez utiliser par la suite :

* meta-data
* QnA ID
* score

Pour le test, était-il judicieux de filtrer avec les métadonnées ? Oui et non. Le système de test doit créer des fichiers de test pour chaque paire de métadonnées ainsi qu’un test sans paires de métadonnées.

### <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne souhaitez pas continuer à tester la base de connaissances, supprimez l’outil de fichier par lot et les fichiers de test.

Si vous ne prévoyez pas de réutiliser cette base de connaissances, supprimez-la en effectuant les étapes suivantes :

1. Sur le portail QnA Maker, sélectionnez **My Knowledge bases**  (Mes bases de connaissances) dans le menu du haut.
1. Dans la liste des bases de connaissances, sélectionnez l’icône **Delete** (Supprimer) sur la ligne de la base de connaissances de ce guide de démarrage rapide.

[La documentation de référence sur l’outil](../reference-tsv-format-batch-testing.md) comprend ce qui suit :

* exemple de ligne de commande de l’outil
* format des fichiers d’entrée et de sortie TSV

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Publier une base de connaissances](../quickstarts/create-publish-knowledge-base.md)
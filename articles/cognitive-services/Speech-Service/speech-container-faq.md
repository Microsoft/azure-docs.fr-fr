---
title: Questions fréquentes (FAQ) sur les conteneurs de service Speech
titleSuffix: Azure Cognitive Services
description: Installez et exécutez des conteneurs Speech. La reconnaissance vocale transcrit en temps réel des flux audio en texte que vos applications, outils ou appareils peuvent utiliser ou afficher. La synthèse vocale convertit le texte d’entrée en parole synthétisée quasi humaine.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 11/12/2020
ms.author: aahi
ms.custom: devx-track-csharp
ms.openlocfilehash: a657f43ef2d889cad1608d34e9235b1d5e7cb576
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95894148"
---
# <a name="speech-service-containers-frequently-asked-questions-faq"></a>Questions fréquentes (FAQ) sur les conteneurs de service Speech

Lorsque vous utilisez le service Speech avec des conteneurs, consultez cette collection de questions fréquemment posées avant de contacter le support. Cet article regroupe des questions variant du niveau général au niveau technique. Pour développer une réponse, cliquez sur la question.

## <a name="general-questions"></a>Questions générales

<details>
<summary>
<b>Comment fonctionnent les conteneurs Speech et comment les configurer ?</b>
</summary>

**Réponse :** Quand vous configurez le cluster de production, plusieurs éléments sont à prendre en compte. Tout d’abord, la configuration d’un seul langage et de plusieurs conteneurs sur la même machine ne doit pas être un gros problème. Si vous rencontrez des problèmes, il s’agit peut-être d’un problème matériel. Nous allons tout d’abord examiner la ressource, autrement dit, les spécifications de l’UC et de la mémoire.

Prenons le conteneur `ja-JP` et le modèle le plus récent. Le modèle acoustique est le plus exigeant au niveau du processeur, tandis que le modèle de langage monopolise la plus grande partie de la mémoire. Lorsque nous avons testé l’utilisation, il a fallu environ 0,6 cœur de processeur pour traiter une seule requête de reconnaissance vocale lorsque l’audio est transmis en temps réel (par exemple, à partir du microphone). Si vous envoyez du contenu audio plus rapidement que le temps réel (par exemple, à partir d’un fichier), cette utilisation peut doubler (1,2 cœur). Dans le même temps, la mémoire indiquée ci-dessous est utilisée pour décoder la parole. Cela ne prend *pas* en compte la taille réelle complète du modèle de langage, qui résidera dans le cache de fichiers. Pour `ja-JP`, ce sont 2 Go supplémentaires ; pour `en-US`, cela peut être encore plus (6-7 Go).

Si vous avez une machine sur laquelle la mémoire est limitée et que vous essayez de déployer plusieurs langages sur ce dernier, il est possible que le cache de fichiers soit plein et que le système d’exploitation soit contraint de paginer les modèles. Pour une transcription en cours d’exécution, cela peut être désastreux et entraîner des ralentissements et d’autres implications sur les performances.

En outre, nous avons des packages de fichiers exécutables pour les machines avec le jeu d’instructions [Advanced Vector Extension (AVX2)](speech-container-howto.md#advanced-vector-extension-support). Une machine avec le jeu d’instructions AVX512 nécessite la génération de code pour cette cible, et le démarrage de 10 conteneurs pour 10 langages peut épuiser temporairement l’UC. Un message similaire à celui-ci apparaîtra dans les journaux d’activité Docker :

```console
2020-01-16 16:46:54.981118943 
[W:onnxruntime:Default, tvm_utils.cc:276 LoadTVMPackedFuncFromCache]
Cannot find Scan4_llvm__mcpu_skylake_avx512 in cache, using JIT...
```

Vous pouvez définir le nombre de décodeurs souhaités à l’intérieur d’un *seul* conteneur à l’aide de la variable `DECODER MAX_COUNT`. Donc, fondamentalement, nous devons commencer par votre référence SKU (UC/mémoire), et de là nous pouvons vous suggérer comment en tirer le meilleur. Un excellent point de départ consiste à faire référence aux spécifications de ressources de machine hôte recommandées.

<br>
</details>

<details>
<summary>
<b>Pouvez-vous aider à planifier la capacité et à estimer les coûts des conteneurs de reconnaissance vocale locaux ?</b>
</summary>

**Réponse :** Pour la capacité de conteneur en mode de traitement par lots, chaque décodeur peut gérer x2 à x3 en temps réel, avec deux cœurs de processeur, pour une reconnaissance unique. Nous vous déconseillons de conserver plus de deux reconnaissances simultanées par instance de conteneur, mais vous recommandons d’exécuter davantage d’instances de conteneurs pour des raisons de fiabilité/disponibilité, derrière un équilibreur de charge.

Toutefois, chaque instance de conteneur peut s’exécuter avec plus de décodeurs. Par exemple, nous pouvons peut-être configurer 7 décodeurs par instance de conteneur sur une machine à huit cœurs (à plus de 2x chacun), ce qui génère un débit x15. Il existe un paramètre `DECODER_MAX_COUNT` à connaître. Dans le cas extrême, des problèmes de fiabilité et de latence surviennent, avec un débit augmenté de manière significative. Pour un microphone, cela sera à x1 en temps réel. L’utilisation globale devrait être d’environ un cœur pour une reconnaissance unique.

Pour le scénario de traitement de 1k heures/jour en mode de traitement par lots, dans un cas extrême, 3 machines virtuelles pourraient gérer cela dans les 24 heures, mais sans garantie. Pour gérer les jours de pointe, le basculement, la mise à jour et pour fournir le minimum en matière de sauvegarde/plan de continuité des activités, nous vous recommandons 4-5 machines plutôt que 3 par cluster, et avec au moins 2 clusters.

Pour le matériel, nous utilisons des machines virtuelles Azure standard `DS13_v2` en tant que référence (chaque noyau doit avoir 2,6 GHz ou plus, avec le jeu d’instructions AVX2 activé).

| Instance  | Processeurs virtuels | RAM    | Stockage temporaire | Paiement à l’utilisation avec AHB | Réservation de 1 an avec AHB (% d’économies) | Réservation de 3 ans avec AHB (% d’économies) |
|-----------|---------|--------|--------------|------------------------|-------------------------------------|--------------------------------------|
| `DS13 v2` | 8       | 56 Gio | 112 Gio      | 0,598 $/heure            | 0,3528 $/heure (~41 %)                 | 0,2333 $/heure (~61%)                  |

En fonction de la référence de conception (deux clusters de 5 machines virtuelles pour gérer le traitement par lots de 1k heures/jour), le coût matériel sur 1 an sera de :

> 2 (clusters) * 5 (machines virtuelles par cluster) * 0,3528 $/heure * 365 (jours) * 24 (heures) = 31k $/an

Lors du mappage à la machine physique, une estimation générale est de 1 processeur virtuel = 1 cœur d’UC physique. En réalité, 1 processeur virtuel est plus puissant qu’un seul cœur.

En local, tous ces facteurs supplémentaires entrent en jeu :

- Le type de l’UC physique et le nombre de cœurs qu’elle contient
- Le nombre d’UC s’exécutant ensemble sur la même boîte/machine
- Configuration des machines virtuelles
- Utilisation de l’hyper-threading/du multi-threading
- Méthode de partage de la mémoire
- Système d’exploitation, etc.

Normalement, ce n’est pas aussi bien réglé que l’environnement Azure. En tenant compte des autres charges, je dirais qu’une estimation sûre est de 10 cœurs d’UC physiques = 8 processeurs virtuels Azure. Cela étant, les processeurs populaires disposent seulement de huit cœurs. Avec le déploiement local, le coût est plus élevé que l’utilisation de machines virtuelles Azure. Tenez également compte du taux d’amortissement.

Le coût du service est le même que celui du service en ligne : 1 $/heure pour la reconnaissance vocale. Le coût du service Speech est de :

> 1 $ * 1000 * 365 = 365k $

Le coût de maintenance payé à Microsoft dépend du niveau de service et du contenu du service. Il varie de 29,99 $/mois pour le niveau de base à des centaines de milliers de dollars si un entretien sur site est impliqué. Un chiffre approximatif est de 300 $/heure pour l’entretien/la maintenance. Le coût du personnel n’est pas inclus. D’autres coûts d’infrastructure (tels que le stockage, les réseaux et les équilibrages de charge) ne sont pas inclus.

<br>
</details>

<details>
<summary>
<b>Pourquoi la ponctuation manque-t-elle dans la transcription ?</b>
</summary>

**Réponse :** Vous devez configurer `speech_recognition_language=<YOUR_LANGUAGE>` explicitement dans la demande si vous utilisez le client Carbon.

Par exemple :

```python
if not recognize_once(
    speechsdk.SpeechRecognizer(
        speech_config=speechsdk.SpeechConfig(
            endpoint=template.format("interactive"),
            speech_recognition_language="ja-JP"),
            audio_config=audio_config)):

    print("Failed interactive endpoint")
    exit(1)
```
Voici la sortie :

```cmd
RECOGNIZED: SpeechRecognitionResult(
    result_id=2111117c8700404a84f521b7b805c4e7, 
    text="まだ早いまだ早いは猫である名前はまだないどこで生まれたかとんと見当を検討をなつかぬ。
    何でも薄暗いじめじめした所でながら泣いていた事だけは記憶している。
    まだは今ここで初めて人間と言うものを見た。
    しかも後で聞くと、それは書生という人間中で一番同額同額。",
    reason=ResultReason.RecognizedSpeech)
```

<br>
</details>

<details>
<summary>
<b>Puis-je utiliser à la fois un modèle acoustique et un modèle de langage personnalisés avec le conteneur Speech ?</b>
</summary>

Nous sommes uniquement en mesure de passer un ID de modèle, soit un modèle de langage personnalisé soit un modèle acoustique personnalisé.

**Réponse :** La décision de ne *pas* prendre en charge à la fois le modèle de langage personnalisé et le modèle acoustique personnalisé. Cela reste en vigueur jusqu’à ce qu’un identificateur unifié soit créé pour réduire les interruptions d’API. Malheureusement, cela n’est pas pris en charge pour le moment.

<br>
</details>

<details>
<summary>
<b>Pouvez-vous expliquer ces erreurs du conteneur de reconnaissance vocale personnalisé ?</b>
</summary>

**Erreur 1 :**

```cmd
Failed to fetch manifest: Status: 400 Bad Request Body:
{
    "code": "InvalidModel",
    "message": "The specified model is not supported for endpoint manifests."
}
```

**Réponse 1 :** Si vous formez avec le dernier modèle personnalisé, nous ne prenons pas cela en charge. Si vous formez avec une version antérieure, il doit être possible de l’utiliser. Nous travaillons toujours sur la prise en charge des dernières versions.

Essentiellement, les conteneurs personnalisés ne prennent pas en charge les modèles acoustiques basés sur Halide ou ONNX (le comportement par défaut dans le portail de formation personnalisée). Cela est dû au fait que les modèles personnalisés ne sont pas chiffrés et que nous ne souhaitons pas exposer les modèles ONNX ; toutefois, les modèles de langage ne posent pas de problème. Le client devra sélectionner explicitement un ancien modèle hors ONNX pour la formation personnalisée. La précision n’est pas affectée. La taille du modèle peut être supérieure (de 100 Mo).

> Modèle de support > 20190220 (v4.5 unifiée)

**Erreur 2 :**

```cmd
HTTPAPI result code = HTTPAPI_OK.
HTTP status code = 400.
Reason:  Synthesis failed.
StatusCode: InvalidArgument,
Details: Voice does not match.
```

**Réponse 2 :** Vous devez fournir le bon nom de voix dans la requête, en respectant la casse. Reportez-vous au mappage de nom de service complet. Vous devez utiliser `en-US-JessaRUS`, car `en-US-JessaNeural` n’est pas disponible pour le moment dans la version conteneur de la synthèse vocale.

**Erreur 3 :**

```json
{
    "code": "InvalidProductId",
    "message": "The subscription SKU \"CognitiveServices.S0\" is not supported in this service instance."
}
```

**Réponse 3 :** Vous pouvez créer une ressource Speech, mais pas une ressource Cognitive Services.


<br>
</details>

<details>
<summary>
<b>Quels protocoles d’API sont pris en charge, REST ou WS ?</b>
</summary>

**Réponse :** Pour les conteneurs de reconnaissance vocale et de synthèse vocale, nous ne prenons actuellement en charge que le protocole WebSocket. Le kit de développement logiciel (SDK) prend uniquement en charge les appels WS, mais pas REST. Nous prévoyons d’ajouter la prise en charge REST, mais n’avons pas d’estimation de date pour le moment. Reportez-vous toujours à la documentation officielle. Consultez [Points de terminaison de prédiction de requête](speech-container-howto.md#query-the-containers-prediction-endpoint).

<br>
</details>

<details>
<summary>
<b>CentOS est-il pris en charge pour les conteneurs Speech?</b>
</summary>

**Réponse :** CentOS 7 n’est pas encore pris en charge par le kit de développement logiciel (SDK) Python, et Ubuntu 19.04 n’est pas pris en charge.

Le package Python du kit SDK Speech est disponible pour les systèmes d’exploitation suivants :
- **Windows** : x64 et x86
- **Mac** : macOS X version 10.12 ou ultérieure
- **Linux** : Ubuntu 16.04, Ubuntu 18.04, Debian 9 sur x64

Pour plus d’informations sur la configuration de l’environnement, consultez [Configuration de la plateforme Python](quickstarts/setup-platform.md?pivots=programming-language-python). Pour le moment, Ubuntu 18.04 est la version recommandée.

<br>
</details>

<details>
<summary>
<b>Pourquoi est-ce que j’obtiens des erreurs quand j’essaie d’appeler des points de terminaison de prédiction LUIS ?</b>
</summary>

J’utilise le conteneur LUIS dans un déploiement IoT Edge et j’essaie d’appeler le point de terminaison de prédiction LUIS à partir d’un autre conteneur. Le conteneur LUIS est à l’écoute sur le port 5001, et l’URL que j’utilise est la suivante :

```csharp
var luisEndpoint =
    $"ws://192.168.1.91:5001/luis/prediction/v3.0/apps/{luisAppId}/slots/production/predict";
var config = SpeechConfig.FromEndpoint(new Uri(luisEndpoint));
```

L’erreur que j’obtiens est la suivante :

```cmd
WebSocket Upgrade failed with HTTP status code: 404 SessionId: 3cfe2509ef4e49919e594abf639ccfeb
```

Je vois la requête dans les journaux du conteneur LUIS et le message indique :

```cmd
The request path /luis//predict" does not match a supported file type.
```

Qu’est-ce que cela signifie ? Qu’ai-je manqué ? Je suivais l’exemple du kit de développement logiciel (SDK) Speech, [ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk). Le scénario est que nous détectons l’audio directement à partir du microphone du PC et que nous essayons de déterminer l’intention, en fonction de l’application LUIS que nous avons formée. L’exemple que j’ai lié fait exactement cela. Il fonctionne bien avec le service cloud LUIS. L’utilisation du kit de développement logiciel (SDK) Speech semblait nous éviter d’avoir à effectuer un appel explicite distinct à l’API de reconnaissance vocale, puis un deuxième appel à LUIS.

Donc, tout ce que je tente de faire est de passer du scénario d’utilisation de LUIS dans le cloud à l’utilisation du conteneur LUIS. Je ne comprends pas pourquoi le kit de développement logiciel (SDK) Speech fonctionne pour l’un, mais pas pour l’autre.

**Réponse :** Le kit de développement logiciel (SDK) Speech ne doit pas être utilisé avec un conteneur LUIS. Pour utiliser le conteneur LUIS, vous devez utiliser le kit de développement logiciel (SDK) LUIS ou l’API REST LUIS. Le kit de développement logiciel (SDK) Speech doit être utilisé avec un conteneur vocal.

Un cloud est différent d’un conteneur. Un cloud peut être composé de plusieurs conteneurs agrégés (parfois appelés microservices). Il y a donc un conteneur LUIS, et un conteneur Speech : deux conteneurs distincts. Le conteneur Speech n’effectue que la reconnaissance vocale. Le conteneur LUIS ne fait que l’interprétation du langage. Dans le cloud, étant donné que les deux conteneurs sont déployés et qu’il s’agit d’une solution non optimale qu’un client distant accède au cloud, fasse la reconnaissance vocale, revienne, accède à nouveau au cloud et passe à LUIS, nous fournissons une fonctionnalité qui permet au client d’accéder à Speech, de rester dans le cloud, d’aller à LUIS et enfin de revenir au client. Par conséquent, même dans ce scénario, le kit de développement logiciel (SDK) Speech passe au conteneur cloud vocal avec l’audio, puis le conteneur de cloud Speech parle au conteneur cloud LUIS avec du texte. Le conteneur LUIS n’a pas de concept d’acceptation de l’audio (il n’est pas judicieux de faire en sorte que le conteneur LUIS accepte la diffusion d’audio, car LUIS est un service textuel). En local, nous n’avons aucune certitude que notre client a déployé les deux conteneurs, et nous ne prévoyons pas l’orchestration entre les conteneurs locaux de nos clients. Si les deux conteneurs sont déployés en local, étant donné qu’ils sont plus locaux pour le client, il n’y a pas de problème à effectuer d’abord la reconnaissance vocale, revenir au client et passer le texte à LUIS.

<br>
</details>

<details>
<summary>
<b>Pourquoi est-ce que nous obtenons des erreurs avec macOS, le conteneur Speech et le kit de développement logiciel (SDK) Python ?</b>
</summary>

Lorsque nous envoyons un fichier *.wav* à transcrire, le résultat revient avec :

```cmd
recognition is running....
Speech Recognition canceled: CancellationReason.Error
Error details: Timeout: no recognition result received.
When creating a websocket connection from the browser a test, we get:
wb = new WebSocket("ws://localhost:5000/speech/recognition/dictation/cognitiveservices/v1")
WebSocket
{
    url: "ws://localhost:5000/speech/recognition/dictation/cognitiveservices/v1",
    readyState: 0,
    bufferedAmount: 0,
    onopen: null,
    onerror: null,
    ...
}
```

Nous savons que le WebSocket est configuré correctement.

**Réponse :** Si tel est le cas, consultez [ce problème GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/310). Nous avons une solution de contournement [proposée ici](https://github.com/Azure-Samples/cognitive-services-speech-sdk/issues/310#issuecomment-527542722).

Carbon a résolu cela dans la version 1.8.


<br>
</details>

<details>
<summary>
<b>Quelles sont les différences entre les points de terminaison de conteneur Speech ?</b>
</summary>

Pouvez-vous nous aider à remplir les métriques de test suivantes, y compris les fonctions à tester, et comment tester le kit de développement logiciel (SDK) et les API REST ? En particulier, les différences entre « interactif » et « conversation », que je ne vois pas dans les documents/exemples existants.

| Point de terminaison                                                | Test fonctionnel                                                   | Kit SDK | API REST |
|---------------------------------------------------------|-------------------------------------------------------------------|-----|----------|
| `/speech/synthesize/cognitiveservices/v1`               | Synthétiser du texte (conversion de texte par synthèse vocale)                                  |     | Oui      |
| `/speech/recognition/dictation/cognitiveservices/v1`    | Point de terminaison de WebSocket V1 de dictée local Cognitive Services local        | Oui | Non       |
| `/speech/recognition/interactive/cognitiveservices/v1`  | Le point de terminaison WebSocket interactif v1 Cognitive Services local  |     |          |
| `/speech/recognition/conversation/cognitiveservices/v1` | Le point de terminaison WebSocket de conversation v1 Cognitive Services local |     |          |

**Réponse :** Il s’agit d’une fusion des éléments suivants :
- Les personnes qui essaient le point de terminaison de dictée pour les conteneurs (je ne sais pas comment elles ont obtenu cette URL)
- Le point de terminaison de 1<sup>re</sup> partie est celui dans un conteneur.
- Le point de terminaison de 1<sup>re</sup> partie qui retourne des messages speech.fragment à la place des messages `speech.hypothesis` que les points de terminaison de tiers<sup></sup> retournent pour le point de terminaison de dictée.
- Les démarrages rapides Carbon utilisent tous `RecognizeOnce` (mode interactif)
- Carbon a une assertion pour les messages `speech.fragment`, nécessitant qu’ils ne soient pas retournés en mode interactif.
- Les assertions sont déclenchées par Carbon dans les versions de production (avec arrêt du processus).

La solution de contournement consiste à utiliser la reconnaissance continue dans votre code ou à vous connecter (plus rapidement) aux points de terminaison interactifs ou continus dans le conteneur.
Pour votre code, définissez le point de terminaison sur `host:port`/speech/recognition/interactive/cognitiveservices/v1

Pour les différents modes, consultez les modes Speech, voir ci-dessous :

[!INCLUDE [speech-modes](includes/speech-modes.md)]

Le correctif approprié sera disponible avec le SDK 1.8, qui disposera de la prise en charge locale (le bon point de terminaison sera choisi, donc cela ne fonctionnera pas moins bien que le service en ligne). En attendant, il existe un échantillon pour la reconnaissance continue, alors pourquoi ne pas le proposer ?

https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/6805d96bf69d9e95c9137fe129bc5d81e35f6309/samples/python/console/speech_sample.py#L196

<br>
</details>

<details>
<summary>
<b>Quel mode dois-je utiliser pour différents fichiers audio ?</b>
</summary>

**Réponse :** Voici un [démarrage rapide avec Python](./get-started-speech-to-text.md?pivots=programming-language-python). Vous pouvez trouver les autres langages liés sur le site de documentation.

Juste pour clarifier l’interactivité, la conversation et la dictée : il s’agit d’un moyen avancé de spécifier la façon dont notre service gère la demande de reconnaissance vocale. Malheureusement, pour les conteneurs locaux, nous devons spécifier l’URI complet (puisqu’il s’agit d’un ordinateur local), donc ces informations sont divulguées lors de l’abstraction. Nous travaillons avec l’équipe du kit de développement logiciel (SDK) pour le rendre plus utilisable à l’avenir.

<br>
</details>

<details>
<summary>
<b>Comment pouvons-nous obtenir une mesure approximative des transactions/seconde/noyau ?</b>
</summary>

**Réponse :** Voici quelques-uns des chiffres approximatifs à attendre du modèle existant (qui sera modifié pour le meilleur dans celui que nous fournirons lors de la disponibilité générale) :

- Pour les fichiers, la limitation sera de x2 dans le kit de développement logiciel (SDK) Speech. Les cinq premières secondes de l’audio ne sont pas limitées. Le décodeur est en mesure d’effectuer x3 en temps réel. Pour ce faire, l’utilisation globale du processeur sera proche de 2 cœurs pour une reconnaissance unique.
- Pour un microphone, cela sera à x1 en temps réel. L’utilisation globale doit être d’environ 1 cœur pour une reconnaissance unique.

Cela peut être vérifié à partir des journaux d’activité Docker. Nous allons en fait consigner la ligne avec les statistiques de session et d’expression/énoncé, et cela comprend les chiffres de RTF.


<br>
</details>

<details>
<summary>
<b>Est-il courant de fractionner des fichiers audio pour l’utilisation des conteneurs Speech ?</b>
</summary>

Mon plan actuel consiste à prendre un fichier audio existant et à le fractionner en segments de 10 secondes et à les envoyer par le biais du conteneur. S’agit-il d’un scénario acceptable ?  Existe-t-il un meilleur moyen de traiter des fichiers audio plus volumineux avec le conteneur ?

**Réponse :** Il vous suffit d’utiliser le kit de développement logiciel (SDK) Speech et de lui attribuer le fichier, il fera le nécessaire. Pourquoi devez-vous segmenter le fichier ?


<br>
</details>

<details>
<summary>
<b>Comment faire en sorte que plusieurs conteneurs s’exécutent sur le même hôte ?</b>
</summary>

Le document indique d’exposer un autre port, ce que je fais, mais le conteneur LUIS écoute toujours sur le port 5000 ?

**Réponse :** Essayez `-p <outside_unique_port>:5000`. Par exemple : `-p 5001:5000`.


<br>
</details>

## <a name="technical-questions"></a>Questions techniques

<details>
<summary>
<b>Comment puis-je faire pour que des API ne fonctionnant pas par lots gèrent de l’audio de &lt;15 secondes ?</b>
</summary>

**Réponse :** `RecognizeOnce()` En mode interactif, ne traite que 15 secondes d’audio, car le mode est destiné aux commandes vocales où les énoncés sont censés être courts. Si vous utilisez `StartContinuousRecognition()` pour la dictée ou la conversation, il n’y a pas de limite de 15 secondes.


<br>
</details>

<details>
<summary>
<b>Quelles sont les ressources recommandées (processeur et RAM) pour 50 requêtes simultanées ?</b>
</summary>

Combien de requêtes simultanées puis-je gérer avec 4 cœurs et 4 Go de RAM ? Si nous devons répondre à, par exemple, 50 requêtes simultanées, combien de cœurs et de RAM sont recommandés ?

**Réponse :** En temps réel, 8 avec notre version `en-US` la plus récente. Nous vous recommandons d’utiliser davantage de conteneurs Docker au-delà de 6 requêtes simultanées. Cela devient très compliqué au-delà de 16 cœurs et sensible au nœud d’accès mémoire non uniforme (NUMA). Le tableau suivant indique l’allocation de ressources minimale et recommandée pour chaque conteneur Speech.

# <a name="speech-to-text"></a>[Reconnaissance vocale](#tab/stt)

| Conteneur      | Minimum             | Recommandé         |
|----------------|---------------------|---------------------|
| Reconnaissance vocale | 2 cœurs, 2 Go de mémoire | 4 cœurs, 4 Go de mémoire |

# <a name="custom-speech-to-text"></a>[Reconnaissance vocale personnalisée](#tab/cstt)

| Conteneur             | Minimum             | Recommandé         |
|-----------------------|---------------------|---------------------|
| Reconnaissance vocale personnalisée | 2 cœurs, 2 Go de mémoire | 4 cœurs, 4 Go de mémoire |

# <a name="text-to-speech"></a>[Synthèse vocale](#tab/tts)

| Conteneur      | Minimum             | Recommandé         |
|----------------|---------------------|---------------------|
| Synthèse vocale | 1 cœur, 2 Go de mémoire | 2 cœur, 3 Go de mémoire |

# <a name="custom-text-to-speech"></a>[Synthèse vocale personnalisée](#tab/ctts)

| Conteneur             | Minimum             | Recommandé         |
|-----------------------|---------------------|---------------------|
| Synthèse vocale personnalisée | 1 cœur, 2 Go de mémoire | 2 cœur, 3 Go de mémoire |

**_

- Chaque cœur doit être cadencé à au moins 2,6 GHz.
- Pour les fichiers, la limitation sera dans le kit de développement logiciel (SDK) Speech, à x2 (les 5 premières secondes de l’audio ne sont pas limitées).
- Le décodeur peut effectuer environ x2 à x3 en temps réel. Pour ce faire, l’utilisation globale du processeur sera proche de deux cœurs pour une reconnaissance unique. C’est pourquoi nous vous déconseillons de conserver plus de deux connexions actives par instance de conteneur. La solution extrême serait de placer environ 10 décodeurs à x2 en temps réel dans une machine à huit cœurs comme `DS13_V2`. Pour le conteneur version 1.3 et versions ultérieures, il existe un paramètre que vous pouvez essayer de définir, `DECODER_MAX_COUNT=20`.
- Pour le microphone, cela sera x1 en temps réel. L’utilisation globale devrait être d’environ un cœur pour une reconnaissance unique.

Prenez en compte le nombre total d’heures d’audio dont vous disposez. Si le nombre est élevé, pour améliorer la fiabilité et la disponibilité, nous vous suggérons d’exécuter davantage d’instances de conteneurs, soit sur une seule boîte, soit sur plusieurs boîtes, derrière un équilibreur de charge. L’orchestration peut être effectuée à l’aide de Kubernetes (K8S) et Helm, ou avec Dockr Compose.

Par exemple, pour gérer 1000 heures/24 heures, nous avons essayé de configurer 3-4 machines virtuelles, avec 10 instances/décodeurs par machine virtuelle.

<br>
</details>

<details>
<summary>
<b>Le conteneur Speech prend-il en charge la ponctuation ?</b>
</summary>

_ *Réponse :* * Nous voyons que la mise en majuscules (ITN) est disponible dans le conteneur local. La ponctuation dépend de la langue et n’est pas prise en charge pour certaines langues, dont le chinois et le japonais.

Nous *prenons en charge* la ponctuation implicite et de base pour les conteneurs existants, mais elle est définie sur `off` par défaut. Cela signifie que vous pouvez récupérer le caractère `.` dans votre exemple, mais pas le caractère `。`. Pour activer cette logique implicite, voici un exemple de la façon de le faire dans Python à l’aide de notre kit de développement logiciel (SDK) Speech (cela se ferait de façon similaire dans les autres langages) :

```python
speech_config.set_service_property(
    name='punctuation',
    value='implicit',
    channel=speechsdk.ServicePropertyChannel.UriQueryParameter
)
```

<br>
</details>

<details>
<summary>
<b>Pourquoi est-ce que j’obtiens des erreurs 404 lors de la tentative de POST dans un conteneur de reconnaissance vocale ?</b>
</summary>

Voici un exemple de POST HTTP :

```http
POST /speech/recognition/conversation/cognitiveservices/v1?language=en-US&format=detailed HTTP/1.1
Accept: application/json;text/xml
Content-Type: audio/wav; codecs=audio/pcm; samplerate=16000
Transfer-Encoding: chunked
User-Agent: PostmanRuntime/7.18.0
Cache-Control: no-cache
Postman-Token: xxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx
Host: 10.0.75.2:5000
Accept-Encoding: gzip, deflate
Content-Length: 360044
Connection: keep-alive
HTTP/1.1 404 Not Found
Date: Tue, 22 Oct 2019 15:42:56 GMT
Server: Kestrel
Content-Length: 0
```

**Réponse :** Nous ne prenons pas en charge l’API REST dans un conteneur de reconnaissance vocale, seulement les WebSockets via le kit de développement logiciel (SDK) Speech. Reportez-vous toujours à la documentation officielle. Consultez [Points de terminaison de prédiction de requête](speech-container-howto.md#query-the-containers-prediction-endpoint).

<br>
</details>


<details>
<summary>
<b> Pourquoi le conteneur s’exécute-t-il en tant qu’utilisateur non-racine ? Quels sont les problèmes qui peuvent en résulter ?</b>
</summary>

**Réponse :** Notez que l’utilisateur par défaut à l’intérieur du conteneur est un utilisateur non-racine. Cela fournit une protection contre les processus qui s’échappent du conteneur et obtiennent des réaffectations d’autorisations sur le nœud hôte. Par défaut, certaines plateformes, par exemple OpenShift Container Platform, le font déjà en exécutant des conteneurs à l’aide d’un identifiant utilisateur affecté arbitrairement. Pour ces plateformes, l’utilisateur non-racine doit disposer d’autorisations d’accès en écriture sur les volumes mappés de manière externe qui nécessitent des écritures. Cela s’applique, par exemple, à un dossier de journalisation ou un dossier de téléchargement de modèle personnalisé.
<br>
</details>

<details>
<summary>
<b>Quand vous utilisez le service de reconnaissance vocale, pourquoi cette erreur s’affiche-t-elle ?</b>
</summary>

```cmd
Error in STT call for file 9136835610040002161_413008000252496:
{
    "reason": "ResultReason.Canceled",
    "error_details": "Due to service inactivity the client buffer size exceeded. Resetting the buffer. SessionId: xxxxx..."
}
```

**Réponse :** Cela se produit généralement lorsque vous envoyez de l’audio plus rapidement que le conteneur Speech peut en accepter. Le tampon du client se remplit et l’annulation est déclenchée. Vous devez contrôler la simultanéité et le RTF à partir duquel vous envoyez l’audio.

<br>
</details>

<details>
<summary>
<b>Pouvez-vous expliquer ces erreurs de conteneur de synthèse vocale à parti des exemples C++ ?</b>
</summary>

**Réponse :** Si la version du conteneur est antérieure à 1.3, ce code doit être utilisé :

```cpp
const auto endpoint = "http://localhost:5000/speech/synthesize/cognitiveservices/v1";
auto config = SpeechConfig::FromEndpoint(endpoint);
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text1}}}").get();
```

Les conteneurs plus anciens n’ont pas le point de terminaison requis pour que Carbon fonctionne avec l’API `FromHost`. Si les conteneurs sont utilisés pour la version 1.3, le code suivant doit être utilisé :

```cpp
const auto host = "http://localhost:5000";
auto config = SpeechConfig::FromHost(host);
config->SetSpeechSynthesisVoiceName(
    "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)");
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text1}}}").get();
```

Voici un exemple d’utilisation de l’API `FromEndpoint` :

```cpp
const auto endpoint = "http://localhost:5000/cognitiveservices/v1";
auto config = SpeechConfig::FromEndpoint(endpoint);
config->SetSpeechSynthesisVoiceName(
    "Microsoft Server Speech Text to Speech Voice (en-US, AriaRUS)");
auto synthesizer = SpeechSynthesizer::FromConfig(config);
auto result = synthesizer->SpeakTextAsync("{{{text2}}}").get();
```

 La fonction `SetSpeechSynthesisVoiceName` est appelée, car les conteneurs avec un moteur de synthèse vocale mis à jour requièrent le nom de voix.

<br>
</details>

<details>
<summary>
<b>Comment puis-je utiliser la v1.7 du kit de développement logiciel (SDK) Speech avec un conteneur Speech ?</b>
</summary>

**Réponse :** Il existe trois points de terminaison sur le conteneur Speech pour différentes utilisations, qui sont définis en tant que modes Speech. Voir ci-dessous :

[!INCLUDE [speech-modes](includes/speech-modes.md)]

Ils sont destinés à des fins différentes et sont utilisés différemment.

[Exemples](https://github.com/Azure-Samples/cognitive-services-speech-sdk/blob/master/samples/python/console/speech_sample.py) Python :
- Pour une reconnaissance unique (mode interactif) avec un point de terminaison personnalisé (autrement dit, `SpeechConfig` avec un paramètre de point de terminaison), consultez `speech_recognize_once_from_file_with_custom_endpoint_parameters()`.
- Pour la reconnaissance continue (mode conversation) et simplement modifier pour utiliser un point de terminaison personnalisé comme indiqué ci-dessus, consultez `speech_recognize_continuous_from_file()`.
- Pour activer la dictée dans des exemples comme ci-dessus (uniquement si vous en avez vraiment besoin), juste après avoir créé `speech_config`, ajoutez le code `speech_config.enable_dictation()`.

Dans C#, pour activer la dictée, appelez la fonction `SpeechConfig.EnableDictation()`.

### <a name="fromendpoint-apis"></a>API `FromEndpoint`
| Langage | Détails de l’API |
|----------|:------------|
| C++ | <a href="https://docs.microsoft.com/en-us/cpp/cognitive-services/speech/speechconfig#fromendpoint" target="_blank">`SpeechConfig::FromEndpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| C# | <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.fromendpoint?view=azure-dotnet" target="_blank">`SpeechConfig.FromEndpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Java | <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.fromendpoint" target="_blank">`SpeechConfig.fromendpoint` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Objective-C | <a href="https://docs.microsoft.com/en-us/objectivec/cognitive-services/speech/spxspeechconfiguration#initwithendpoint" target="_blank">`SPXSpeechConfiguration:initWithEndpoint;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Python | <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python" target="_blank">`SpeechConfig;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| JavaScript | Non pris en charge actuellement, ni de prise en charge prévue. |

<br>
</details>

<details>
<summary>
<b>Comment puis-je utiliser la v1.8 du kit de développement logiciel (SDK) Speech avec un conteneur Speech ?</b>
</summary>

**Réponse :** Il existe une nouvelle API `FromHost`. Elle ne remplace pas les API existantes. Elle ajoute simplement une autre façon de créer une configuration vocale à l’aide d’un hôte personnalisé.

### <a name="fromhost-apis"></a>API `FromHost`

| Langage | Détails de l’API |
|--|:-|
| C# | <a href="https://docs.microsoft.com/dotnet/api/microsoft.cognitiveservices.speech.speechconfig.fromhost?view=azure-dotnet" target="_blank">`SpeechConfig.FromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| C++ | <a href="https://docs.microsoft.com/en-us/cpp/cognitive-services/speech/speechconfig#fromhost" target="_blank">`SpeechConfig::FromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Java | <a href="https://docs.microsoft.com/java/api/com.microsoft.cognitiveservices.speech.speechconfig.fromhost" target="_blank">`SpeechConfig.fromHost` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Objective-C | <a href="https://docs.microsoft.com/en-us/objectivec/cognitive-services/speech/spxspeechconfiguration#initwithhost" target="_blank">`SPXSpeechConfiguration:initWithHost;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| Python | <a href="https://docs.microsoft.com/python/api/azure-cognitiveservices-speech/azure.cognitiveservices.speech.speechconfig?view=azure-python" target="_blank">`SpeechConfig;` <span class="docon docon-navigate-external x-hidden-focus"></span></a> |
| JavaScript | Non prise en charge pour le moment |

> Paramètres : hôte (obligatoire), clé d’abonnement (facultative si vous pouvez utiliser le service sans celui-ci).

Le format de l’hôte est `protocol://hostname:port`, où `:port` est facultatif (voir ci-dessous) :
- Si le conteneur s’exécute localement, le nom d’hôte est `localhost`.
- Si le conteneur s’exécute sur un serveur distant, utilisez le nom d’hôte ou l’adresse IPv4 de ce serveur.

Exemples de paramètres d’hôte pour la reconnaissance vocale :
- `ws://localhost:5000` : connexion non sécurisée à un conteneur local à l’aide du port 5000
- `ws://some.host.com:5000` : connexion non sécurisée à un conteneur s’exécutant sur un serveur distant

Exemples de Python de ci-dessus, mais utilisez le paramètre `host` au lieu de `endpoint` :

```python
speech_config = speechsdk.SpeechConfig(host="ws://localhost:5000")
```

<br>
</details>

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Conteneurs Cognitive Services](speech-container-howto.md)
---
title: Transcription de centre d’appels - Service Speech
titleSuffix: Azure Cognitive Services
description: Un scénario courant pour la reconnaissance vocale est la transcription de grands volumes de données de téléphonie provenant de différents systèmes, tels que la réponse vocale interactive (RVI). À l’aide du service Speech et du modèle vocal unifié, une entreprise peut obtenir des transcriptions de qualité, avec des systèmes de capture audio.
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 19d4cc388494e149b7f258a8e9f154041a3dd070
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95021964"
---
# <a name="speech-service-for-telephony-data"></a>Service Speech pour les données de téléphonie

Les données de téléphonie qui sont générées par l’intermédiaire de lignes fixes, de téléphones mobiles et de systèmes radios sont généralement de basse qualité, avec une bande étroite de l’ordre de 8 KHz, ce qui crée des difficultés lors de la conversion de parole en texte. Les derniers modèles de reconnaissance vocale du service Speech excellent à retranscrire ces données de téléphonie, même quand celles-ci sont difficiles à comprendre pour un humain. Ces modèles sont entraînés avec de grands volumes de données de téléphonie et offrent une précision sans égal sur le marché de la reconnaissance, même dans des environnements bruyants.

Un scénario courant pour la reconnaissance vocale est la transcription de grands volumes de données de téléphonie pouvant provenir de différents systèmes, tels que la réponse vocale interactive (RVI). Les données audio que ces systèmes fournissent peuvent être au format stéréo ou mono, à l’état brut avec un post-traitement du signal nul ou quasi nul. À l’aide du service Speech et du modèle vocal unifié, une entreprise peut obtenir des transcriptions de qualité, quels que soient les systèmes utilisés pour la capture audio.

Vous pouvez utiliser les données de téléphonie pour mieux comprendre les besoins de vos clients, identifier de nouvelles opportunités de marché ou évaluer les performances des agents de centre d’appels. Une fois les données transcrites, une entreprise peut utiliser le résultat à des fins comme l’amélioration des données de télémétrie, l’identification des expressions clés ou l’analyse du sentiment des clients.

Les technologies décrites dans cette page sont utilisées par Microsoft en interne pour différents services de traitement d’appel du support, à la fois en temps réel et en mode de traitement par lot.

Nous allons passer en revue quelques-unes de ces technologies et les fonctionnalités associées offertes par le service Speech.

> [!IMPORTANT]
> Le modèle unifié du service Speech est entraîné avec des données diverses et offre une même solution modèle à divers scénarios d’analytique allant de la dictée à la téléphonie.

## <a name="azure-technology-for-call-centers"></a>Technologie Azure pour les centres d’appels

Au-delà de leur aspect fonctionnel, les fonctionnalités du service Speech servent principalement, dans le cadre du centre d’appels, à améliorer l’expérience client. Trois domaines ressortent clairement à cet égard :

- Analytique post-appel, autrement dit, le traitement par lot des enregistrements d’appel à l’issue de l’appel.
- Analytique en temps réel, à savoir le traitement du signal audio pour extraire divers insights pendant que l’appel a lieu (le sentiment étant un cas d’usage important).
- Assistants vocaux (bots), qui pilotent le dialogue entre le client et le bot afin de tenter de résoudre le problème du client sans intervention de l’agent ou qui font office d’application de protocoles d’intelligence artificielle (IA) pour aider l’agent.

L’image ci-dessous représente une architecture classique de l’implémentation d’un scénario en mode de traitement par lot. ![Architecture de la transcription de centre d’appels](media/scenarios/call-center-transcription-architecture.png)

## <a name="speech-analytics-technology-components"></a>Composants de la technologie Speech Analytics

Que le domaine soit l’analytique post-appel ou en temps réel, Azure offre un ensemble de technologies matures et émergentes qui permettent d’améliorer l’expérience client.

### <a name="speech-to-text-stt"></a>Reconnaissance vocale

La [reconnaissance vocale](speech-to-text.md) est la fonctionnalité la plus recherchée dans toute solution de centre d’appels. Étant donné que la plupart des processus d’analytique en aval s’appuient sur du texte retranscrit, le rapport d’erreurs Windows (_WER_) est primordial. Un des principaux défis dans une transcription de centre d’appels est le bruit qui prévaut dans le centre d’appels (tel que le bruit de fond produit par les conversations des autres agents), la grande variété de dialectes ou de langues locales ainsi que la faible qualité du signal téléphonique réel. WER étant étroitement lié à la manière dont les modèles acoustiques et linguistiques sont entraînés pour une langue locale donnée, il est important que vous puissiez personnaliser le modèle en fonction de votre langue locale. Nos derniers modèles unifiés version 4.x sont la solution à la latence et à la précision de la transcription. Entraînés avec des dizaines de milliers d’heures de données acoustiques et des milliards d’informations lexicales, les modèles unifiés sont les modèles les plus précis du marché pour la transcription des données de centre d’appels.

### <a name="sentiment"></a>Sentiments

L’évaluation de la qualité de l’expérience du client est un des aspects les plus importants de la technologie Speech Analytics quand celle-ci est appliquée à l’espace du centre d’appels. Notre [API de transcription par lot](batch-transcription.md) offre l’analyse des sentiments par énoncé. Vous pouvez agréger l’ensemble des valeurs obtenues dans le cadre d’une transcription d’appel pour déterminer le sentiment de l’appel de vos agents et du client.

### <a name="silence-non-talk"></a>Silence (absence d’énoncé)

Il n’est pas rare que 35 % de la durée d’un appel de support soit dépourvue d’énoncé. Voici quelques scénarios avec absence d’énoncé : un agent recherche l’historique des incidents d’un client, un agent utilise des outils qui lui permettent d’accéder au bureau du client et d’effectuer des fonctions, un client attend que l’appel soit transféré, etc. Il est primordial d’évaluer avec précision les silences qui se produisent dans un appel car de nombreuses sensibilités des clients entourent ces types de scénarios.

### <a name="translation"></a>Traduction

Certaines entreprises expérimentent la fourniture de transcriptions traduites à partir d’appels de support en langues étrangères afin que les responsables des prestations puissent comprendre l’expérience de leurs clients à l’échelle mondiale. Nos capacités de [traduction](./speech-translation.md) sont inégalées. Nous pouvons traduire des données audio dans un format audio ou dans un format texte pour de nombreuses langues locales.

### <a name="text-to-speech"></a>Synthèse vocale

La [synthèse vocale](text-to-speech.md) est un autre aspect important de l’implémentation de bots qui interagissent avec les clients. Le processus est généralement le suivant : le client parle, sa voix est transcrite en texte, le texte est analysé afin que les intentions en soient dégagées, une réponse est synthétisée selon l’intention reconnue, puis un élément est présenté au client ou une réponse vocale synthétisée est générée. Bien entendu, tout ceci se produit rapidement ; une faible latence est donc un composant important de la réussite de ces systèmes.

Notre latence de bout en bout est considérablement faible pour les différentes technologies impliquées telles que la [reconnaissance vocale](speech-to-text.md), [LUIS](https://azure.microsoft.com/services/cognitive-services/language-understanding-intelligent-service/), [Bot Framework](https://dev.botframework.com/) et la [synthèse vocale](text-to-speech.md).

En outre, nos nouvelles voix ressemblent à si méprendre à des voix humaines. Vous pouvez utiliser nos voix afin de doter votre bot d’une personnalité unique.

### <a name="search"></a>Recherche

Un autre élément de base de l’analytique consiste à identifier les interactions où un événement ou une expérience spécifique se sont produits. Deux approches sont généralement possibles à cette fin : une recherche ad hoc où l’utilisateur tape simplement une expression, à laquelle le système répond, ou une requête plus structurée, où un analyste peut créer un jeu de déclarations logiques qui identifient un scénario dans un appel, chaque appel pouvant ensuite être indexé par rapport à cet ensemble de requêtes. Un bon exemple de recherche est la déclaration de conformité classique : « Cet appel est enregistré pour des raisons de qualité ». En effet, de nombreuses entreprises veulent s’assurer que leurs agents indiquent cet avertissement aux clients avant que l’appel ne soit effectivement enregistré. La plupart des systèmes d’analytique ont la possibilité de dégager une tendance des comportements trouvés par les algorithmes de recherche et de requête, cette création de rapports de tendances étant finalement une des fonctions les plus importantes d’un système d’analytique. Par le biais du [Répertoire Cognitive Services](https://azure.microsoft.com/services/cognitive-services/directory/search/), votre solution de bout en bout peut être considérablement améliorée avec des fonctions d’indexation et de recherche.

### <a name="key-phrase-extraction"></a>Extraction d’expressions clés

Ce domaine correspond à une des applications d’analytique les plus complexes, qui bénéficie de l’application de l’intelligence artificielle et du Machine Learning. Le principal scénario dans ce cas consiste à déduire l’intention du client. Pourquoi le client appelle-t-il ? Quel est le problème du client ? Pourquoi le client a-t-il eu une expérience négative ? Notre [service d’analyse de texte](https://azure.microsoft.com/services/cognitive-services/text-analytics/) met à votre disposition un ensemble d’analytiques que vous pouvez exploiter immédiatement pour effectuer une mise à niveau rapide de votre solution de bout en bout afin d’extraire les mots clés ou expressions importants.

Voyons maintenant un peu plus en détail le traitement par lot et les pipelines en temps réel pour la reconnaissance vocale.

## <a name="batch-transcription-of-call-center-data"></a>Transcription par lot des données de centre d’appels

Pour la transcription en bloc des données audio, nous avons développé l’[API de transcription par lot](batch-transcription.md). L’API de transcription par lot a été développée pour la transcription de grandes quantités de données audio de façon asynchrone. En ce qui concerne la transcription des données de centre d’appels, notre solution s’appuie sur les socles suivants :

- **Précision** : avec les modèles unifiés de quatrième génération, nous proposons une qualité de transcription inégalée.
- **Latence** : nous sommes conscients que quand des transcriptions en bloc sont effectuées, elles sont nécessaires rapidement. Les travaux de transcription lancés par le biais de l’[API de transcription par lot](batch-transcription.md) sont mis en attente immédiatement et, une fois l’exécution lancée, ils sont effectués plus rapidement que la transcription en temps réel.
- **Sécurité** : nous sommes conscients que les appels peuvent contenir des données sensibles. Soyez assuré que la sécurité est une de nos priorités les plus élevées. Notre service a obtenu les certifications ISO, SOC, HIPAA et PCI.

Les centres d’appels génèrent quotidiennement de grands volumes de données audio. Si votre entreprise stocke des données de téléphonie dans un emplacement central, tel que Stockage Azure, vous pouvez utiliser [l’API de transcription par lot](batch-transcription.md) pour demander et recevoir des transcriptions de façon asynchrone.

Une solution classique utilise les services suivants :

- Le service Speech sert à transcrire des données obtenues par reconnaissance vocale. Pour pouvoir utiliser l’API de transcription par lot, vous avez besoin d’un abonnement standard (S0) pour le service Speech. Les abonnements gratuits (F0) ne fonctionnent pas.
- [Stockage Azure](https://azure.microsoft.com/services/storage/) est utilisé pour stocker les données de téléphonie et les transcriptions retournées par l’API de transcription par lot. Ce compte de stockage doit utiliser des notifications, en particulier quand de nouveaux fichiers sont ajoutés. Ces notifications permettent de déclencher le processus de transcription.
- [Azure Functions](../../azure-functions/index.yml) sert à créer l’URI des signatures d’accès partagé (SAP) pour chaque enregistrement et à déclencher la requête HTTP POST pour démarrer une transcription. En outre, Azure Functions permet de créer des requêtes pour récupérer et supprimer des transcriptions à l’aide de l’API de transcription par lot.

En interne, nous utilisons les technologies ci-dessus pour prendre en charge les appels des clients de Microsoft en mode de traitement par lot.
:::image type="content" source="media/scenarios/call-center-batch-pipeline.png" alt-text="Technologies utilisées pour prendre en charge les appels des clients Microsoft en mode de traitement par lot.":::

## <a name="real-time-transcription-for-call-center-data"></a>Transcription en temps réel des données de centre d’appels

Certaines entreprises sont amenées à transcrire des conversations en temps réel. La transcription en temps réel permet d’identifier les mots clés et de déclencher des recherches de contenu et de ressources pertinents pour la conversation, à des fins de supervision des sentiments, d’améliorer l’accessibilité ou de fournir des traductions pour les clients et les agents qui ne sont pas des locuteurs natifs.

Pour les scénarios qui nécessitent une transcription en temps réel, nous vous recommandons d’utiliser le [SDK Speech](speech-sdk.md). La reconnaissance vocale est disponible dans [plus de 20 langues](language-support.md), tandis que le SDK est disponible en C++, C#, Java, Python, Node.js, Objective-C et JavaScript. Vous trouverez des exemples dans chaque langage sur [GitHub](https://github.com/Azure-Samples/cognitive-services-speech-sdk). Pour les dernières actualités et mises à jour, consultez [Notes de publication](releasenotes.md).

En interne, nous utilisons les technologies ci-dessus pour analyser en temps réel les appels des clients de Microsoft, comme l’illustre le diagramme suivant.

![Architecture du traitement par lot](media/scenarios/call-center-reatime-pipeline.png)

## <a name="a-word-on-ivrs"></a>Un mot sur les RVI

Vous pouvez facilement intégrer le service Speech à une solution à l’aide du [SDK Speech](speech-sdk.md) ou de l’[API REST](./overview.md#reference-docs). Toutefois, la transcription de centre d’appels peut nécessiter des technologies supplémentaires. En règle générale, une connexion entre un système RVI et Azure est requise. Même si nous n’offrons pas ces composants, voici la description de ce qu’implique une connexion à un système RVI.

Plusieurs produits de service RVI ou de téléphonie (comme Genesys ou AudioCodes) offrent des fonctionnalités d’intégration qui peuvent être exploitées pour permettre au trafic audio entrant et sortant d’accéder à un service Azure. Fondamentalement, un service Azure personnalisé peut fournir une interface spécifique pour définir des sessions d’appel téléphonique (comme un démarrage d’appel ou une fin d’appel) et exposer une API WebSocket pour recevoir les données audio de flux entrant utilisées avec le service Speech. Les réponses sortantes, telles que la transcription de conversation ou les connexions avec le Bot Framework, peuvent être synthétisées avec le service de synthèse vocale de Microsoft et retournées au système RVI à des fins de lecture.

Un autre scénario est l’intégration directe avec le protocole SIP (Session Initiation Protocol). Un service Azure se connecte à un serveur SIP, obtenant ainsi un flux entrant et un flux sortant, qui est utilisé pour les phases de reconnaissance vocale et de synthèse vocale. Pour vous connecter à un serveur SIP, vous disposez d’offres de logiciels commerciaux, tels que le SDK Ozeki ou l’[API Appel et réunions de Teams](/graph/api/resources/communications-api-overview) (en version bêta), qui sont conçues pour prendre en charge ce type de scénario pour les appels audio.

## <a name="customize-existing-experiences"></a>Personnaliser les expériences existantes

 Le service Speech fonctionne bien avec des modèles prédéfinis. Toutefois, vous pouvez avoir envie de personnaliser et d’ajuster l’expérience selon votre produit ou environnement. Les options de personnalisation vont de l’ajustement du modèle acoustique aux polices de voix uniques pour votre marque. Une fois que vous avez créé un modèle personnalisé, vous pouvez l’utiliser avec toutes les fonctionnalités du service Speech en temps réel ou en mode de traitement par lot.

| Service Speech | Modèle | Description |
| -------------- | ----- | ----------- |
| Reconnaissance vocale | [Modèle acoustique](./how-to-custom-speech-train-model.md) | Créez un modèle acoustique personnalisé pour des applications, outils ou appareils utilisés dans des environnements particuliers, tels qu’une voiture ou un atelier, avec des conditions d’enregistrement spécifique. Par exemple, vous pouvez adapter l’enregistrement à des accent régionaux, à des bruits de fond spécifiques ou à l’utilisation d’un microphone particulier. |
|                | [Modèle de langage](./how-to-custom-speech-train-model.md) | Créez un modèle de langage personnalisé afin d’améliorer la transcription du vocabulaire ou de la grammaire spécifiques d’un secteur, par exemple, la terminologie médicale ou le jargon informatique. |
|                | [Modèle de prononciation](./how-to-custom-speech-train-model.md) | Avec un modèle de prononciation personnalisé, vous pouvez définir la forme phonétique et écrite d’un mot. Cela peut être utile pour traiter les termes personnalisés que sont notamment les noms et les acronymes. Pour commencer, vous n’avez besoin que d’un fichier de prononciation, c’est-à-dire un simple fichier `.txt`. |
| Synthèse vocale | [Police de la voix](./how-to-custom-voice-create-voice.md) | Les polices de voix personnalisées vous permettent de créer une voix unique reconnaissable entre toutes pour votre marque. Il suffit d’une petite quantité de données pour commencer. Plus vous fournirez de données, plus naturelle et humaine sonnera votre police de voix. |

## <a name="sample-code"></a>Exemple de code

Un exemple de code est disponible sur GitHub pour chacune des fonctionnalités du service Speech. Ces exemples couvrent des scénarios courants tels que la lecture du signal audio d’un fichier ou d’un flux, la reconnaissance continue et ponctuelle, et l’utilisation de modèles personnalisés. Pour voir les exemples SDK et REST, suivez ces liens :

- [Exemples de reconnaissance vocale et de traduction vocale (SDK)](https://github.com/Azure-Samples/cognitive-services-speech-sdk)
- [Exemples de transcription par lot (REST)](https://github.com/Azure-Samples/cognitive-services-speech-sdk/tree/master/samples/batch)
- [Exemples de synthèse vocale (REST)](https://github.com/Azure-Samples/Cognitive-Speech-TTS)

## <a name="reference-docs"></a>Documents de référence

- [Kit de développement logiciel (SDK) de reconnaissance vocale](./speech-sdk.md)
- [SDK Speech Devices](speech-devices-sdk.md)
- [API REST : Reconnaissance vocale](rest-speech-to-text.md)
- [API REST : Synthèse vocale](rest-text-to-speech.md)
- [API REST : Transcription et personnalisation par lot](https://westus.dev.cognitive.microsoft.com/docs/services/speech-to-text-api-v3-0)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Obtenir gratuitement une clé d’abonnement au service Speech](overview.md#try-the-speech-service-for-free)
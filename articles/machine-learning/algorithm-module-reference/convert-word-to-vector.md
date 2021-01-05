---
title: 'Vectoriser des mots : sur le module Modifier les métadonnées'
titleSuffix: Azure Machine Learning
description: Découvrez comment utiliser trois modèles Word2Vec fournis pour extraire un vocabulaire et les imbrications de mots correspondantes d’un corpus de texte.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: reference
author: likebupt
ms.author: keli19
ms.date: 05/19/2020
ms.openlocfilehash: 5fad3e4862b0c40c9edd00a5b9d47b245e529396
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91536730"
---
# <a name="convert-word-to-vector-module"></a>Module Vectoriser des mots

Cet article décrit comment utiliser le module Vectoriser des mots dans le concepteur Azure Machine Learning pour effectuer les tâches suivantes :

- Appliquer différents modèles Word2Vec (Word2Vec, FastText, modèle préentraîné GloVe) sur le corpus de texte spécifié comme entrée.
- Générer du vocabulaire avec des plongements lexicaux.

Ce module utilise la bibliothèque Gensim. Pour plus d’informations sur Gensim, consultez son [site web officiel](https://radimrehurek.com/gensim/apiref.html) qui comprend des tutoriels et une explication des algorithmes.

### <a name="more-about-converting-words-to-vectors"></a>En savoir plus sur la vectorisation de mots

La conversion de mots en vecteurs, ou vectorisation de mots, est un processus de traitement du langage naturel. Ce processus utilise des modèles de langage pour mapper des mots à un espace vectoriel. Un espace vectoriel représente chaque mot par un vecteur de nombres réels. Il permet aussi à des mots avec des significations similaires d’avoir des représentations similaires.

Utilisez des plongements lexicaux comme entrées initiales pour accomplir en aval des tâches de traitement du langage naturel, comme la classification de texte et l’analyse des sentiments.

Il existe plusieurs technologies de plongement lexical. Dans ce module, nous avons implémenté trois méthodes largement utilisées : les deux premières, Word2Vec et FastText, sont des modèles d’entraînement en ligne ; la troisième, glove-wiki-gigaword-100, est un modèle préentraîné. 

Les modèles d’entraînement en ligne sont entraînés sur vos données d’entrée. Les modèles préentraînés sont entraînés hors connexion sur des corpus de texte plus importants, comme Wikipédia ou Google Actualités, qui contiennent environ 100 milliards mots. Le plongement lexical reste alors constant pendant la vectorisation de texte. Les modèles préentraînés offrent des avantages tels qu’un temps d’entraînement réduit, un meilleur encodage des vecteurs de mots et de meilleures performances globales.

Voici quelques informations sur les méthodes :

+ Word2Vec est l’une des techniques les plus populaires pour apprendre les plongements lexicaux à l’aide d’un réseau neuronal superficiel. La théorie est abordée dans ce document, téléchargeable au format PDF : [Efficient Estimation of Word Representations in Vector Space](https://arxiv.org/pdf/1301.3781.pdf). L’implémentation décrite dans ce module est basée sur la [bibliothèque Gensim pour Word2Vec](https://radimrehurek.com/gensim/models/word2vec.html).

+ La théorie de FastText est exposée dans un document disponible en téléchargement au format PDF : [Enriching Word Vectors with Subword Information](https://arxiv.org/pdf/1607.04606.pdf). L’implémentation décrite dans ce module est basée sur la [bibliothèque Gensim pour FastText](https://radimrehurek.com/gensim/models/fasttext.html).

+ Le modèle préentraîné GloVe est glove-wiki-gigaword-100. Il s’agit d’une collection de vecteurs préentraînés basés sur un corpus de texte Wikipédia qui contient 5,6 milliards de jetons et 400 000 mots de vocabulaire sans casse. Un téléchargement au format PDF est disponible : [GloVe : Global Vectors for Word Representation](https://nlp.stanford.edu/pubs/glove.pdf).

## <a name="how-to-configure-convert-word-to-vector"></a>Comment configurer la fonctionnalité Vectoriser du texte

Ce module nécessite un jeu de données contenant une colonne de texte. Il est préférable d’utiliser du texte prétraité.

1. Ajoutez le module **Vectoriser du texte** à votre pipeline.

2. Comme entrée du module, fournissez un jeu de données contenant une ou plusieurs colonnes de texte.

3. Pour **Colonne cible**, choisissez une seule colonne contenant du texte à traiter.

    Étant donné que ce module crée du vocabulaire à partir de texte, le contenu des colonnes diffère, ce qui conduit à des contenus de vocabulaire différents. C’est pourquoi le module n’accepte qu’une colonne cible.

4. Pour **Stratégie Word2Vec**, choisissez **Modèle anglais préentraîné GloVe**, **Gensim Word2Vec** ou **Gensim FastText**.

5. Si **Stratégie Word2Vec** est **Gensim Word2Vec** ou **Gensim FastText** :

    + Pour **Algorithme d’entraînement Word2Vec**, choisissez **Skip_gram** ou **CBOW**. La différence est expliquée dans le [document d’origine au format PDF](https://arxiv.org/pdf/1301.3781.pdf).

        La méthode par défaut est **Skip_gram**.

    + Pour **Longueur du champ lexical**, spécifiez la dimensionnalité des vecteurs de mots. Correspond au paramètre `size` dans Gensim.

        La valeur de plongement lexical par défaut est 100.

    + Pour **Taille de la fenêtre de contexte**, spécifiez la distance maximale entre le mot prédit et le mot actuel. Correspond au paramètre `window` dans Gensim.

        La taille de fenêtre par défaut est 5.

    + Pour **Nombre d’époques**, spécifiez le nombre d’époques (itérations) sur le corpus. Correspond au paramètre `iter` dans Gensim.

        Le nombre d’époques par défaut est 5.

6. Pour **Taille maximale du vocabulaire**, spécifiez le nombre maximal de mots dans le vocabulaire généré.

    S’il y a plus de mots uniques que la taille maximale, élaguez les mots peu fréquents.

    La taille du vocabulaire par défaut est 10 000.

7. Pour **Nombre minimal de mots**, indiquez un nombre minimal de mots. Le module ignore tous les mots dont la fréquence est inférieure à cette valeur.

    La valeur par défaut est 5.

8. Envoyez le pipeline.

## <a name="examples"></a>Exemples

Le module a une sortie :

+ **Vocabulaire avec imbrications** : contient le vocabulaire généré, ainsi que le plongement lexical de chaque mot. Une dimension occupe une colonne.

L’exemple suivant montre le fonctionnement du module Vectoriser des mots. Il utilise le module Vectoriser des mots avec les paramètres par défaut sur le jeu de données Wikipédia SP 500 prétraité.

### <a name="source-dataset"></a>Jeu de données source

Le jeu de données contient une colonne de catégorie ainsi que le texte intégral extrait de Wikipédia. Le tableau suivant montre quelques exemples représentatifs.

|Texte|
|----------|
|nasdaq 100 component s p 500 component foundation founder location city apple campus 1 infinite loop street infinite loop cupertino california cupertino california location country united states...|
|br nasdaq 100 nasdaq 100 component br s p 500 s p 500 component industry computer software foundation br founder charles geschke br john warnock location adobe systems...|
|s p 500 s p 500 component industry automotive industry automotive predecessor general motors corporation 1908 2009 successor...|
|s p 500 s p 500 component industry conglomerate company conglomerate foundation founder location city fairfield connecticut fairfield connecticut location country usa area...|
|br s p 500 s p 500 component foundation 1903 founder william s harley br arthur davidson harley davidson founder arthur davidson br walter davidson br william a davidson location...|

### <a name="output-vocabulary-with-embeddings"></a>Vocabulaire de sortie avec les imbrications

Le tableau suivant contient la sortie de ce module obtenue en utilisant le jeu de données Wikipédia SP 500 en entrée. La colonne la plus à gauche indique le vocabulaire. Son vecteur de plongement lexical est représenté par les valeurs des colonnes restantes dans la même ligne.

|Vocabulaire|Dimension d’imbrication 0|Dimension d’imbrication 1|Dimension d’imbrication 2|Dimension d’imbrication 3|Dimension d’imbrication 4|Dimension d’imbrication 5|...|Dimension d’imbrication 99|
|-------------|-------------|-------------|-------------|-------------|-------------|-------------|-------------|-------------|
|nasdaq|-0.375865|0.609234|0.812797|-0.002236|0.319071|-0.591986|...|0.364276
|component|0.081302|0.40001|0.121803|0.108181|0.043651|-0.091452|...|0.636587
|s|-0.34355|-0.037092|-0.012167|0.151542|0.601019|0.084501|...|0.149419
|p|-0.133407|0.073244|0.170396|0.326706|0.213463|-0.700355|...|0.530901
foundation|-0.166819|0.10883|-0.07933|-0.073753|0.262137|0.045725|...|0.27487
founder|-0.297408|0.493067|0.316709|-0.031651|0.455416|-0.284208|...|0.22798
location|-0.375213|0.461229|0.310698|0.213465|0.200092|0.314288|...|0.14228
city|-0.460828|0.505516|-0.074294|-0.00639|0.116545|0.494368|...|-0.2403
apple|0.05779|0.672657|0.597267|-0.898889|0.099901|0.11833|...|0.4636
campus|-0.281835|0.29312|0.106966|-0.031385|0.100777|-0.061452|...|0.05978
infinite|-0.263074|0.245753|0.07058|-0.164666|0.162857|-0.027345|...|-0.0525
loop|-0.391421|0.52366|0.141503|-0.105423|0.084503|-0.018424|...|-0.0521

Dans cet exemple, nous avons utilisé la valeur par défaut **Gensim Word2Vec** comme **Stratégie de Word2Vec**, et l’**Algorithme d’entraînement** est **Skip-gram**. Comme la **longueur du plongement lexical** est 100, nous avons 100 colonnes de plongement lexical.

## <a name="technical-notes"></a>Notes techniques

Cette section contient des détails, des conseils et des réponses à des questions fréquentes.

+ Différence entre des modèles d’entraînement en ligne et un modèle préentraîné :

    Dans ce module Vectoriser des mots, nous avons fourni trois stratégies différentes : deux modèles d’entraînement en ligne et un modèle préentraîné. Les modèles d’entraînement en ligne utilisent votre jeu de données d’entrée comme données d’entraînement, et génèrent du vocabulaire et des vecteurs de mots pendant l’entraînement. Le modèle préentraîné est déjà entraîné par un corpus de texte bien plus volumineux, comme du texte sur Wikipédia ou Twitter. Le modèle préentraîné est en fait une collection de paires mot/plongement lexical.  

    Le modèle préentraîné GloVe résume un vocabulaire à partir du jeu de données d’entrée et génère un vecteur de plongement lexical pour chaque mot à partir du modèle préentraîné. Sans entraînement en ligne, l’utilisation d’un modèle préentraîné peut réduire la durée de l’entraînement. Il offre de meilleures performances, en particulier quand le jeu de données d’entrée est relativement petit.

+ Taille du plongement lexical :

    En général, la longueur du plongement lexical d’un mot est de quelques centaines. Par exemple, 100, 200, 300. Un petit plongement lexical signifie un petit espace vectoriel, ce qui pourrait entraîner des collisions de plongements lexicaux.  

    La longueur des plongements lexicaux est fixe pour les modèles préentraînés. Dans cet exemple, la taille du plongement lexical de glove-wiki-gigaword-100 est 100.


## <a name="next-steps"></a>Étapes suivantes

Consultez [l’ensemble des modules disponibles](module-reference.md) pour Azure Machine Learning. 

Pour obtenir la liste des erreurs spécifiques aux modules du concepteur, consultez les [codes d’erreur du Machine Learning](designer-error-codes.md).

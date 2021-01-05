---
title: Modéliser et partitionner des données sur Azure Cosmos DB à l’aide d’un exemple concret
description: Découvrez comment modéliser et partitionner un exemple concret à l’aide de l’API de base d’Azure Cosmos DB
author: ThomasWeiss
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 05/23/2019
ms.author: thweiss
ms.custom: devx-track-js
ms.openlocfilehash: c3cdc0a9fb9fa236fae37a52194f446278a42f72
ms.sourcegitcommit: 9706bee6962f673f14c2dc9366fde59012549649
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94616244"
---
# <a name="how-to-model-and-partition-data-on-azure-cosmos-db-using-a-real-world-example"></a>Guide pratique pour modéliser et partitionner des données sur Azure Cosmos DB à l’aide d’un exemple concret
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article s’appuie sur plusieurs concepts Azure Cosmos DB tels que la [modélisation des données](modeling-data.md), le [partitionnement](partitioning-overview.md) et le [débit provisionné](request-units.md) pour illustrer comment aborder un exercice concret de conception de données.

Si vous travaillez habituellement avec des bases de données relationnelles, vous avez probablement développé des habitudes et des intuitions sur la façon de concevoir un modèle de données. En raison des contraintes spécifiques, mais également des atouts uniques d’Azure Cosmos DB, la plupart de ces bonnes pratiques ne se traduisent pas correctement et peuvent vous faire dériver vers des solutions non optimales. L’objectif de cet article est de vous guider tout au long du processus de modélisation d’un cas d’utilisation concret sur Azure Cosmos DB, de la modélisation des éléments à la colocalisation d’entités et au partitionnement de conteneur.

[Téléchargez ou affichez un code source généré par la communauté](https://github.com/jwidmer/AzureCosmosDbBlogExample) qui illustre les concepts de cet article. Cet exemple de code a été fourni par un contributeur de la communauté et l’équipe Azure Cosmos DB ne prend pas en charge sa maintenance.

## <a name="the-scenario"></a>Scénario

Pour cet exercice, nous allons prendre en compte le domaine d’une plateforme de création de blogs où des *utilisateurs* peuvent créer des *publications*. Les utilisateurs peuvent également ajouter des mentions *j’aime* et des *commentaires* à ces publications.

> [!TIP]
> Nous avons mis en *italique* certains mots pour identifier les types de « choses » que notre modèle devra manipuler.

Ajout d’exigences supplémentaires à notre spécification :

- Une première page affiche un flux de publications récemment créées,
- Nous pouvons extraire toutes les publications d’un utilisateur, tous les commentaires d’une publication et toutes les mentions j’aime d’une publication,
- Les publications sont retournées avec le nom d’utilisateur de leur auteur et un décompte des commentaires et des mentions j’aime associés,
- Les commentaires et mentions j’aime sont également retournés avec le nom d’utilisateur des utilisateurs qui les ont créés,
- Lorsqu’elles sont affichées sous forme de listes, les publications doivent uniquement présenter un résumé tronqué de leur contenu.

## <a name="identify-the-main-access-patterns"></a>Identifier les modèles d’accès principaux

Pour commencer, nous donnons une structure à notre spécification initiale en identifiant les modèles d’accès de notre solution. Lors de la conception d’un modèle de données pour Azure Cosmos DB, il est important de comprendre quelles demandes le modèle devra traiter pour vous assurer que le modèle traitera efficacement ces demandes.

Pour faciliter le processus global à suivre, nous allons classer ces différentes demandes comme des commandes ou des requêtes, empruntant certains termes au principe [CQRS](https://en.wikipedia.org/wiki/Command%E2%80%93query_separation#Command_query_responsibility_segregation) dans lequel les commandes sont des demandes d’écriture (autrement dit, des intentions de mise à jour du système) et les requêtes sont des demandes de lecture seule.

Voici la liste des demandes que notre plateforme devra exposer :

- **[C1]** Créer/modifier un utilisateur
- **[Q1]** Récupérer un utilisateur
- **[C2]** Créer/modifier une publication
- **[Q2]** Récupérer une publication
- **[Q3]** Lister les publications d’un utilisateur sous forme abrégée
- **[C3]** Créer un commentaire
- **[Q4]** Lister les commentaires d’une publication
- **[C4]** Ajouter une mention « j’aime » à une publication
- **[Q5]** Lister les mentions « j’aime » d’une publication
- **[Q6]** Lister les *x* publications les plus récentes créées sous forme abrégée (flux)

À ce stade, nous n’avons pas pensé aux détails de ce que contiendra chaque entité (utilisateur, publication, etc.). Cette étape est généralement parmi les premières considérées lors de la conception par rapport à un magasin relationnel, parce que nous devons déterminer comment ces entités se traduiront en termes de tables, de colonnes, de clés étrangères, etc. Cela est moins préoccupant avec une base de données de documents qui n’impose aucun schéma en écriture.

La principale raison pour laquelle il est important d’identifier nos modèles d’accès dès le début tient au fait que cette liste de demandes sera notre suite de tests. Chaque fois que nous itérerons sur notre modèle de données, nous examinerons chaque demande et vérifierons ses performances et l’extensibilité.

## <a name="v1-a-first-version"></a>V1 : Première version

Nous commençons avec deux conteneurs : `users` et `posts`.

### <a name="users-container"></a>Conteneur d’utilisateurs

Ce conteneur stocke uniquement des éléments utilisateur :

```json
{
    "id": "<user-id>",
    "username": "<username>"
}
```

Nous partitionnons ce conteneur par `id`, ce qui signifie que chaque partition logique figurant dans ce conteneur contiendra un seul élément.

### <a name="posts-container"></a>Conteneur de publications

Ce conteneur héberge les publications, les commentaires et les mentions « j’aime » :

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "title": "<post-title>",
    "content": "<post-content>",
    "creationDate": "<post-creation-date>"
}

{
    "id": "<comment-id>",
    "type": "comment",
    "postId": "<post-id>",
    "userId": "<comment-author-id>",
    "content": "<comment-content>",
    "creationDate": "<comment-creation-date>"
}

{
    "id": "<like-id>",
    "type": "like",
    "postId": "<post-id>",
    "userId": "<liker-id>",
    "creationDate": "<like-creation-date>"
}
```

Nous partitionnons ce conteneur par `postId`, ce qui signifie que chaque partition logique dans ce conteneur contiendra une seule publication, ainsi que tous les commentaires et toutes les mentions « j’aime » relatifs à cette publication.

Notez que nous avons introduit une propriété `type` dans les éléments stockés dans ce conteneur pour faire la distinction entre les trois types d’entités que ce conteneur héberge.

En outre, nous avons choisi de référencer les données associées au lieu de les incorporer (consultez [cette section](modeling-data.md) pour plus d’informations sur ces concepts), car :

- aucune limite supérieure ne s’applique au nombre de publications qu’un utilisateur peut créer,
- les publications peuvent être d’une longueur arbitraire,
- aucune limite supérieure ne s’applique au nombre de commentaires et de mentions « j’aime » qu’une publication peut avoir,
- nous voulons être en mesure d’ajouter un commentaire ou une mention « j’aime » à une publication sans avoir à mettre à jour la publication proprement dite.

## <a name="how-well-does-our-model-perform"></a>Comment fonctionne notre modèle ?

Il est maintenant temps d’évaluer les performances et l’extensibilité de notre première version. Pour chacune des demandes identifiées précédemment, nous mesurons la latence et le nombre d’unités de requête que le modèle consomme. Cette mesure est effectuée sur un jeu de données factice contenant 100 000 utilisateurs avec entre 5 et 50 publications par utilisateur et jusqu'à 25 commentaires et 100 mentions « j’aime » par publication.

### <a name="c1-createedit-a-user"></a>[C1] Créer/modifier un utilisateur

Cette demande est simple à implémenter car il suffit de créer ou de mettre à jour un élément dans le conteneur `users`. Les demandes seront correctement réparties sur toutes les partitions grâce à la clé de partition `id`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C1.png" alt-text="Écriture d’un seul élément dans le conteneur d’utilisateurs" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 7 ms | 5,71 RU | ✅ |

### <a name="q1-retrieve-a-user"></a>[Q1] Récupérer un utilisateur

La récupération d’un utilisateur s’effectue en lisant l’élément correspondant à partir du conteneur `users`.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q1.png" alt-text="Récupération d’un seul élément dans le conteneur d’utilisateurs" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 2 ms | 1 unité de requête | ✅ |

### <a name="c2-createedit-a-post"></a>[C2] Créer/modifier une publication

Similaire à **[C1]** , il suffit d’écrire dans le conteneur `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Écriture d’un seul élément dans le conteneur de publications" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 9 ms | 8,76 RU | ✅ |

### <a name="q2-retrieve-a-post"></a>[Q2] Récupérer une publication

Nous commençons par extraire le document correspondant à partir du conteneur `posts`. Mais ce n’est pas suffisant. Conformément à notre spécification, nous devons également agréger le nom d’utilisateur de l’auteur de la publication et les nombres de commentaires et de mentions « j’aime » que possède la publication, ce qui nécessite l’exécution de 3 requêtes SQL supplémentaires.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q2.png" alt-text="Récupération d’une publication et agrégation de données supplémentaires" border="false":::

Chaque requête supplémentaire est filtrée en fonction de la clé de partition de son conteneur respectif, ce qui est exactement ce que nous voulons pour optimiser les performances et l’extensibilité. Mais nous devons finalement effectuer quatre opérations pour retourner une publication unique, donc nous améliorerons cela dans une prochaine itération.

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 9 ms | 19,54 RU | ⚠ |

### <a name="q3-list-a-users-posts-in-short-form"></a>[Q3] Lister les publications d’un utilisateur sous forme abrégée

Tout d’abord, nous devons récupérer les publications souhaitées à l’aide d’une requête SQL qui extrait les publications correspondant à cet utilisateur particulier. Mais nous devons également émettre des requêtes supplémentaires pour agréger le nom d’utilisateur de l’auteur et les nombres de commentaires et de mentions « j’aime ».

:::image type="content" source="./media/how-to-model-partition-example/V1-Q3.png" alt-text="Récupération de toutes les publications d’un utilisateur et agrégation des données supplémentaires" border="false":::

Cette implémentation présente de nombreux inconvénients :

- les requêtes d’agrégation des nombres de commentaires et de mentions « j’aime » doivent être émises pour chaque publication retournée par la première requête,
- la requête principale ne filtre pas les données sur la clé de partition du conteneur `posts`, ce qui conduit à une distribution ramifiée et à une analyse de partition sur le conteneur.

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 130 ms | 619,41 RU | ⚠ |

### <a name="c3-create-a-comment"></a>[C3] Créer un commentaire

Un commentaire est créé en écrivant l’élément correspondant dans le conteneur `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Écriture d’un seul élément dans le conteneur de publications" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 7 ms | 8,57 RU | ✅ |

### <a name="q4-list-a-posts-comments"></a>[Q4] Lister les commentaires d’une publication

Nous commençons avec une requête qui extrait tous les commentaires pour cette publication et une fois encore, nous devons également agréger les noms d’utilisateur séparément pour chaque commentaire.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q4.png" alt-text="Récupération de tous les commentaires d’une publication et agrégation des données supplémentaires" border="false":::

La requête principale filtre les données sur la clé de partition du conteneur, mais l’agrégation séparée des noms d’utilisateur pénalise les performances globales. Nous améliorerons cela par la suite.

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 23 ms | 27,72 RU | ⚠ |

### <a name="c4-like-a-post"></a>[C4] Ajouter une mention « j’aime » à une publication

Tout comme pour **[C3]** , nous créons l’élément correspondant dans le conteneur `posts`.

:::image type="content" source="./media/how-to-model-partition-example/V1-C2.png" alt-text="Écriture d’un seul élément dans le conteneur de publications" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 6 ms | 7,05 RU | ✅ |

### <a name="q5-list-a-posts-likes"></a>[Q5] Lister les mentions « j’aime » d’une publication

Tout comme pour **[Q4]** , nous interrogeons les mentions « j’aime » de la publication, puis agrégeons leurs noms d’utilisateur.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q5.png" alt-text="Récupération de toutes les mentions « j’aime » d’une publication et agrégation des données supplémentaires" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 59 ms | 58,92 RU | ⚠ |

### <a name="q6-list-the-x-most-recent-posts-created-in-short-form-feed"></a>[Q6] Lister les x publications les plus récentes créées sous forme abrégée (flux)

Nous extrayons les publications les plus récentes en interrogeant le conteneur `posts` trié dans l’ordre décroissant de la date de création, puis agrégeons les noms d’utilisateur et les nombres de commentaires et de mentions « j’aime » pour chaque publication.

:::image type="content" source="./media/how-to-model-partition-example/V1-Q6.png" alt-text="Récupération des publications les plus récentes et agrégation des données supplémentaires" border="false":::

Une fois encore, notre requête initiale ne filtre pas les données sur la clé de partition du conteneur `posts`, ce qui déclenche une distribution ramifiée coûteuse. La situation est encore pire ici, car nous ciblons un jeu de résultats beaucoup plus grand et trions les résultats avec une clause `ORDER BY`, ce qui rend le processus plus coûteux en termes d’unités de requête.

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 306 ms | 2 063,54 RU | ⚠ |

## <a name="reflecting-on-the-performance-of-v1"></a>Réflexion sur les performances de V1

L’examen des problèmes de performances auxquels nous avons été confrontés dans la section précédente permet d’identifier deux classes principales de problèmes :

- certaines demandes nécessitent l’exécution de plusieurs requêtes afin de collecter toutes les données à retourner,
- certaines requêtes ne filtrent pas les données sur la clé de partition des conteneurs qu’elles ciblent, ce qui conduit à une distribution ramifiée qui nuit à l’extensibilité.

Nous allons résoudre chacun de ces problèmes, en commençant par le premier.

## <a name="v2-introducing-denormalization-to-optimize-read-queries"></a>V2 : Introduction d’une dénormalisation pour optimiser les requêtes de lecture

La raison pour laquelle nous devons émettre des demandes supplémentaires dans certains cas tient au fait que les résultats de la demande initiale ne contiennent pas toutes les données à retourner. Dans le cas d’un magasin de données non relationnel comme Azure Cosmos DB, ce genre de problème se résout généralement en dénormalisant les données dans le jeu de données.

Dans notre exemple, nous allons modifier les éléments de publication pour ajouter le nom d’utilisateur de l’auteur de la publication, le nombre de commentaires et le nombre de mentions « j’aime » :

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

Nous allons également modifier chaque élément de commentaire et de mention « j’aime » pour ajouter le nom d’utilisateur de l’utilisateur qui l’a créé :

```json
{
    "id": "<comment-id>",
    "type": "comment",
    "postId": "<post-id>",
    "userId": "<comment-author-id>",
    "userUsername": "<comment-author-username>",
    "content": "<comment-content>",
    "creationDate": "<comment-creation-date>"
}

{
    "id": "<like-id>",
    "type": "like",
    "postId": "<post-id>",
    "userId": "<liker-id>",
    "userUsername": "<liker-username>",
    "creationDate": "<like-creation-date>"
}
```

### <a name="denormalizing-comment-and-like-counts"></a>Dénormalisation des nombres de commentaires et de mentions « j’aime »

Notre objectif est que, chaque fois que nous ajoutons un commentaire ou une mention « j’aime », nous incrémentions également `commentCount` ou `likeCount` dans la publication correspondante. Comme notre conteneur `posts` est partitionné par `postId`, le nouvel élément (commentaire ou mention « j’aime ») et la publication correspondante figurent dans la même partition logique. Par conséquent, nous pouvons utiliser une [procédure stockée](stored-procedures-triggers-udfs.md) pour effectuer cette opération.

À présent, lors de la création d’un commentaire ( **[C3]** ), au lieu d’ajouter simplement un nouvel élément dans le conteneur `posts`, nous appelons la procédure stockée suivante sur ce conteneur :

```javascript
function createComment(postId, comment) {
  var collection = getContext().getCollection();

  collection.readDocument(
    `${collection.getAltLink()}/docs/${postId}`,
    function (err, post) {
      if (err) throw err;

      post.commentCount++;
      collection.replaceDocument(
        post._self,
        post,
        function (err) {
          if (err) throw err;

          comment.postId = postId;
          collection.createDocument(
            collection.getSelfLink(),
            comment
          );
        }
      );
    })
}
```

Cette procédure stockée accepte l’ID de la publication et le corps du nouveau commentaire en tant que paramètres, puis :

- récupère la publication
- incrémente `commentCount`
- remplace la publication
- ajoute le nouveau commentaire

Comme les procédures stockées sont exécutées en tant que transactions atomiques, la valeur de `commentCount` et le nombre réel de commentaires seront toujours synchronisés.

Bien entendu, nous appelons une procédure stockée similaire lors de l’ajout de nouvelles mentions « j’aime » pour incrémenter `likeCount`.

### <a name="denormalizing-usernames"></a>Dénormalisation des noms d’utilisateur

Les noms d’utilisateur requièrent une approche différente, car les utilisateurs figurent non seulement dans des partitions différentes, mais aussi dans un autre conteneur. Lorsque nous devons dénormaliser des données dans des partitions et des conteneurs, nous pouvons utiliser le [flux de modification](change-feed.md) du conteneur source.

Dans notre exemple, nous utilisons le flux de modification du conteneur `users` pour réagir chaque fois que les utilisateurs mettent à jour leurs noms d’utilisateur. Lorsque cela se produit, nous propageons la modification en appelant une autre procédure stockée sur le conteneur `posts` :

:::image type="content" source="./media/how-to-model-partition-example/denormalization-1.png" alt-text="Dénormalisation des noms d’utilisateur dans le conteneur de publications" border="false":::

```javascript
function updateUsernames(userId, username) {
  var collection = getContext().getCollection();
  
  collection.queryDocuments(
    collection.getSelfLink(),
    `SELECT * FROM p WHERE p.userId = '${userId}'`,
    function (err, results) {
      if (err) throw err;

      for (var i in results) {
        var doc = results[i];
        doc.userUsername = username;

        collection.upsertDocument(
          collection.getSelfLink(),
          doc);
      }
    });
}
```

Cette procédure stockée accepte l’ID et le nouveau nom d’utilisateur de l’utilisateur en tant que paramètres, puis :

- extrait tous les éléments correspondant à `userId` (éventuellement des publications, des commentaires ou des mentions « j’aime »)
- pour chacun de ces éléments
  - remplace `userUsername`
  - remplace l’élément

> [!IMPORTANT]
> Cette opération est coûteuse, car elle requiert l’exécution de cette procédure stockée sur chaque partition du conteneur `posts`. Nous supposons que la plupart des utilisateurs choisissent un nom d’utilisateur approprié lors de leur inscription et qu’ils n’en changeront jamais, de sorte que cette mise à jour s’exécutera très rarement.

## <a name="what-are-the-performance-gains-of-v2"></a>Quels sont les gains de performances de V2 ?

### <a name="q2-retrieve-a-post"></a>[Q2] Récupérer une publication

Maintenant que notre dénormalisation est en place, il nous suffit d’extraire un seul élément pour traiter cette demande.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q2.png" alt-text="Récupération d’un seul élément dans le conteneur de publications" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 2 ms | 1 unité de requête | ✅ |

### <a name="q4-list-a-posts-comments"></a>[Q4] Lister les commentaires d’une publication

Ici encore, nous pouvons faire l’économie des demandes supplémentaires qui extrayaient les noms d’utilisateur et obtenir au final une seule requête qui filtre les données sur la clé de partition.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q4.png" alt-text="Récupération de tous les commentaires d’une publication" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 4 ms | 7,72 RU | ✅ |

### <a name="q5-list-a-posts-likes"></a>[Q5] Lister les mentions « j’aime » d’une publication

La situation est exactement la même lors de l’énumération des mentions « j’aime ».

:::image type="content" source="./media/how-to-model-partition-example/V2-Q5.png" alt-text="Récupération de toutes les mentions « j’aime » d’une publication" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 4 ms | 8,92 RU | ✅ |

## <a name="v3-making-sure-all-requests-are-scalable"></a>V3 : S’assurer que toutes les demandes sont évolutives

En examinant les améliorations de nos performances globales, nous constatons qu’il reste deux demandes que nous n’avons pas complètement optimisées : **[Q3]** et **[Q6]** . Ce sont les demandes impliquant des requêtes qui ne filtrent pas les données sur la clé de partition des conteneurs ciblés.

### <a name="q3-list-a-users-posts-in-short-form"></a>[Q3] Lister les publications d’un utilisateur sous forme abrégée

Cette demande bénéficie déjà des améliorations introduites dans V2, ce qui permet de faire l’économie de requêtes supplémentaires.

:::image type="content" source="./media/how-to-model-partition-example/V2-Q3.png" alt-text="Diagramme qui affiche la requête permettant de répertorier les publications d’un utilisateur sous forme abrégée." border="false":::

Mais la requête restante ne filtre toujours pas les données sur la clé de partition du conteneur `posts`.

La façon d’interpréter cette situation est en fait simple :

1. Cette demande *doit* filtrer les données sur `userId`, car nous voulons extraire toutes les publications d’un utilisateur particulier
1. Elle n’est pas efficace parce qu’elle est exécutée sur le conteneur `posts`, qui n’est pas partitionné par `userId`
1. De toute évidence, nous pouvons résoudre le problème de performances en exécutant cette demande sur un conteneur qui *est* partitionné par `userId`
1. Il s’avère que nous possédons déjà un conteneur de ce type : le conteneur `users`.

Nous allons donc introduire un deuxième niveau de dénormalisation en dupliquant toutes les publications dans le conteneur `users`. Cela nous fournit une copie de nos publications, désormais partitionnées selon une dimension différente qui améliore considérablement l’efficacité de leur récupération par leur `userId`.

Le conteneur `users` contient maintenant 2 types d’éléments :

```json
{
    "id": "<user-id>",
    "type": "user",
    "userId": "<user-id>",
    "username": "<username>"
}

{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

Notez les points suivants :

- Nous avons introduit un champ `type` dans l’élément d’utilisateur afin de distinguer les utilisateurs et les publications.
- Nous avons également ajouté un champ `userId` dans l’élément d’utilisateur, qui est redondant avec le champ `id` mais qui est requis, car le conteneur `users` est maintenant partitionné par `userId` (et non plus par `id`).

Pour réaliser cette dénormalisation, nous utilisons une fois encore le flux de modification. Cette fois-ci, nous réagissons sur le flux de modification du conteneur `posts` pour distribuer toute publication nouvelle ou mise à jour vers le conteneur `users`. Et comme l’énumération des publications ne nécessite pas de retourner leur contenu complet, nous pouvons les tronquer dans ce processus.

:::image type="content" source="./media/how-to-model-partition-example/denormalization-2.png" alt-text="Dénormalisation des publications dans le conteneur des utilisateurs" border="false":::

Maintenant, nous pouvons router notre requête vers le conteneur `users` et filtrer les données sur la clé de partition du conteneur.

:::image type="content" source="./media/how-to-model-partition-example/V3-Q3.png" alt-text="Récupération de toutes les publications d’un utilisateur" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 4 ms | 6,46 RU | ✅ |

### <a name="q6-list-the-x-most-recent-posts-created-in-short-form-feed"></a>[Q6] Lister les x publications les plus récentes créées sous forme abrégée (flux)

Nous devons faire face à une situation similaire ici : même après avoir fait l’économie des requêtes supplémentaires rendues inutiles par la dénormalisation introduite dans V2, la requête restante ne filtre pas les données sur la clé de partition du conteneur :

:::image type="content" source="./media/how-to-model-partition-example/V2-Q6.png" alt-text="Diagramme qui montre la requête pour lister les x publications les plus récentes créées sous forme abrégée." border="false":::

En suivant la même approche, l’optimisation des performances et de l’extensibilité de cette demande exige que cette dernière s’applique à une seule partition. Cela est concevable, car nous n’avons à retourner qu’un nombre limité d’éléments. Pour remplir la page d’accueil de notre plateforme de création de blogs, il nous suffit d’obtenir les 100 publications les plus récentes, sans avoir à paginer le jeu de données complet.

Ainsi, pour optimiser cette dernière demande, nous allons introduire un troisième conteneur dans notre conception, entièrement dédié au traitement de cette demande. Nous allons dénormaliser nos publications dans ce nouveau conteneur `feed` :

```json
{
    "id": "<post-id>",
    "type": "post",
    "postId": "<post-id>",
    "userId": "<post-author-id>",
    "userUsername": "<post-author-username>",
    "title": "<post-title>",
    "content": "<post-content>",
    "commentCount": <count-of-comments>,
    "likeCount": <count-of-likes>,
    "creationDate": "<post-creation-date>"
}
```

Ce conteneur est partitionné par `type`, lequel sera toujours `post` dans nos éléments. Cela garantit que tous les éléments figurant dans ce conteneur seront placés dans la même partition.

Pour réaliser cette dénormalisation, il nous suffit de raccorder le pipeline de flux de modification que nous avons précédemment introduit pour distribuer les publications vers ce nouveau conteneur. Il est important de garder à l’esprit qu’il faut s’assurer de ne stocker que les 100 publications les plus récentes. Sinon, le contenu du conteneur peut croître au-delà de la taille maximale d’une partition. Pour cela, il convient d’appeler un [post-déclencheur](stored-procedures-triggers-udfs.md#triggers) chaque fois qu’un document est ajouté dans le conteneur :

:::image type="content" source="./media/how-to-model-partition-example/denormalization-3.png" alt-text="Dénormalisation des publications dans le conteneur de flux" border="false":::

Voici le corps du post-déclencheur qui tronque la collection :

```javascript
function truncateFeed() {
  const maxDocs = 100;
  var context = getContext();
  var collection = context.getCollection();

  collection.queryDocuments(
    collection.getSelfLink(),
    "SELECT VALUE COUNT(1) FROM f",
    function (err, results) {
      if (err) throw err;

      processCountResults(results);
    });

  function processCountResults(results) {
    // + 1 because the query didn't count the newly inserted doc
    if ((results[0] + 1) > maxDocs) {
      var docsToRemove = results[0] + 1 - maxDocs;
      collection.queryDocuments(
        collection.getSelfLink(),
        `SELECT TOP ${docsToRemove} * FROM f ORDER BY f.creationDate`,
        function (err, results) {
          if (err) throw err;

          processDocsToRemove(results, 0);
        });
    }
  }

  function processDocsToRemove(results, index) {
    var doc = results[index];
    if (doc) {
      collection.deleteDocument(
        doc._self,
        function (err) {
          if (err) throw err;

          processDocsToRemove(results, index + 1);
        });
    }
  }
}
```

L’étape finale consiste à rediriger la requête vers le nouveau conteneur `feed` :

:::image type="content" source="./media/how-to-model-partition-example/V3-Q6.png" alt-text="Récupération des publications les plus récentes" border="false":::

| **Latence** | **Frais en RU (unités de requête)** | **Performances** |
| --- | --- | --- |
| 9 ms | 16,97 RU | ✅ |

## <a name="conclusion"></a>Conclusion

Penchons-nous sur les améliorations globales des performances et de l’extensibilité que nous avons introduites au fil des versions de notre conception.

| | V1 | V2 | V3 |
| --- | --- | --- | --- |
| **[C1]** | 7 ms / 5,71 RU | 7 ms / 5,71 RU | 7 ms / 5,71 RU |
| **[Q1]** | 2 ms / 1 RU | 2 ms / 1 RU | 2 ms / 1 RU |
| **[C2]** | 9 ms / 8,76 RU | 9 ms / 8,76 RU | 9 ms / 8,76 RU |
| **[Q2]** | 9 ms / 19,54 RU | 2 ms / 1 RU | 2 ms / 1 RU |
| **[Q3]** | 130 ms / 619,41 RU | 28 ms / 201,54 RU | 4 ms / 6,46 RU |
| **[C3]** | 7 ms / 8,57 RU | 7 ms / 15,27 RU | 7 ms / 15,27 RU |
| **[Q4]** | 23 ms / 27,72 RU | 4 ms / 7,72 RU | 4 ms / 7,72 RU |
| **[C4]** | 6 ms / 7,05 RU | 7 ms / 14,67 RU | 7 ms / 14,67 RU |
| **[Q5]** | 59 ms / 58,92 RU | 4 ms / 8,92 RU | 4 ms / 8,92 RU |
| **[Q6]** | 306 ms / 2 063,54 RU | 83 ms / 532,33 RU | 9 ms / 16,97 RU |

### <a name="we-have-optimized-a-read-heavy-scenario"></a>Nous avons optimisé un scénario nécessitant beaucoup de lectures

Vous avez sans doute remarqué que nous avons concentré nos efforts sur l’amélioration des performances des demandes de lecture (requêtes) au détriment des demandes d’écriture (commandes). Dans de nombreux cas, les opérations d’écriture déclenchent désormais une dénormalisation ultérieure via les flux de modification, ce qui les rend plus coûteuses en termes de calculs et plus longues à matérialiser.

Cela est justifié par le fait qu’une plateforme de création de blogs (telle que la plupart des applications de réseaux sociaux) nécessite beaucoup de lectures, ce qui signifie que la quantité de demandes de lecture qu’elle doit traiter est généralement supérieure de plusieurs ordres de grandeur à la quantité de demandes d’écriture. Il est donc logique d’augmenter le coût d’exécution des demandes d’écriture pour réduire celui des demandes de lecture et les rendre plus performantes.

Si nous examinons l’optimisation la plus extrême que nous avons réalisée, **[Q6]** est passée de plus de 2 000 RU à seulement 17 RU. Nous avons obtenu cela en dénormalisant les publications pour un coût d’environ 10 unités de requête par élément. Comme nous traiterions beaucoup plus de demandes de flux que de création ou de mises à jour de publications, le coût de cette dénormalisation est négligeable au vu de l’économie globale réalisée.

### <a name="denormalization-can-be-applied-incrementally"></a>La dénormalisation peut être appliquée de façon incrémentielle

Les améliorations d’extensibilité que nous avons explorées dans cet article impliquent la dénormalisation et la duplication des données sur l’ensemble du jeu de données. Il convient de noter que ces optimisations ne sont pas tenues d’être mises en place le premier jour. Les requêtes qui filtrent les données sur les clés de partition présentent de meilleures performances à grande échelle, mais les requêtes entre partitions peuvent être totalement acceptables si elles sont appelées rarement ou sur un jeu de données limité. Si vous créez un simple prototype ou lancez un produit avec une base d’utilisateurs réduite et contrôlée, vous pouvez probablement garder ces améliorations pour plus tard. L’important est alors de [surveiller](use-metrics.md) les performances de votre modèle pour que vous puissiez décider si et quand il est temps de les introduire.

Le flux de modification que nous utilisons pour distribuer des mises à jour aux autres conteneurs stocke toutes ces mises à jour de manière permanente. Cela permet de demander toutes les mises à jour depuis la création du conteneur et d’amorcer les vues dénormalisées comme une opération ponctuelle de mise à jour, même si votre système a déjà une grande quantité de données.

## <a name="next-steps"></a>Étapes suivantes

Après cette introduction à la modélisation et au partitionnement des données pratiques, vous pouvez consulter les articles suivants pour passer en revue les concepts que nous avons abordés :

- [Utiliser des bases de données, des conteneurs et des éléments](account-databases-containers-items.md)
- [Partitioning in Azure Cosmos DB](partitioning-overview.md) (Partitionnement dans Azure Cosmos DB)
- [Flux de modification dans Azure Cosmos DB](change-feed.md)

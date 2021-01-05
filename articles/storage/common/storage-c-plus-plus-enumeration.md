---
title: Répertorier les ressources Stockage Azure avec la bibliothèque de client C++
description: Apprenez à utiliser les API de listage de la bibliothèque cliente Microsoft Azure Storage pour C++ pour énumérer conteneurs, objets blob, files d'attente, tables et autres entités.
author: mhopkins-msft
ms.author: mhopkins
ms.date: 01/23/2017
ms.service: storage
ms.subservice: common
ms.topic: how-to
ms.reviewer: dineshm
ms.openlocfilehash: 8c8e2d2ddf6899e62bc95bc1e52c84eccdc3a91e
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92784096"
---
# <a name="list-azure-storage-resources-in-c"></a>Listage des ressources Azure Storage en C++

Les opérations de listage sont essentielles dans de nombreux scénarios de développement avec Azure Storage. Cet article explique comment énumérer de façon optimale les objets d’Azure Storage à l’aide des API de listage fournies par la bibliothèque cliente Microsoft Azure Storage pour C++.

> [!NOTE]
> Ce guide cible la bibliothèque cliente Stockage Azure C++ version 2.x, qui est disponible par le biais de [NuGet](https://www.nuget.org/packages/wastorage) ou [GitHub](https://github.com/Azure/azure-storage-cpp).

La bibliothèque cliente Storage propose diverses méthodes pour lister ou interroger les objets présents dans Azure Storage. Cet article traite les scénarios suivants :

* liste les conteneurs présents dans un compte ;
* lister les objets blob présents dans un conteneur ou un répertoire d'objets blob virtuel ;
* lister les files d'attente contenues dans un compte ;
* lister les tables contenues dans un compte ;
* interroger les entités d’une table.

Chacune de ces méthodes est présentée en utilisant différentes surcharges qui varient en fonction des scénarios.

## <a name="asynchronous-versus-synchronous"></a>Opérations asynchrones/synchrones

Sachant que la bibliothèque cliente de stockage pour C++ s’appuie sur la [bibliothèque REST C++](https://github.com/Microsoft/cpprestsdk), par nature, les opérations asynchrones sont prises en charge en utilisant [pplx::task](https://microsoft.github.io/cpprestsdk/classpplx_1_1task.html). Par exemple :

```cpp
pplx::task<list_blob_item_segment> list_blobs_segmented_async(continuation_token& token) const;
```

Les opérations synchrones englobent les opérations asynchrones correspondantes :

```cpp
list_blob_item_segment list_blobs_segmented(const continuation_token& token) const
{
    return list_blobs_segmented_async(token).get();
}
```

Si vous travaillez avec plusieurs applications ou services de threading, nous vous recommandons d’utiliser directement les API asynchrones au lieu de créer un thread pour appeler des API, ce qui nuit considérablement aux performances.

## <a name="segmented-listing"></a>Listage segmenté

Du fait de son échelle, le stockage cloud nécessite un listage segmenté. Par exemple, un conteneur d’objets blob Azure peut contenir plus d’un million d’objets blob et une table Azure plus d’un milliard d'entités. Il ne s’agit pas là de chiffres théoriques, mais bien de cas d'utilisation réels de clients.

Il est donc impossible de lister tous les objets dans une même réponse. En revanche, il est possible de lister des objets en utilisant la pagination. Chaque API de listage dispose d’une surcharge *segmentée* .

La réponse à une opération de listage segmenté comporte les éléments suivants :

* *_segment* , qui contient le jeu de résultats retourné pour un seul appel à l'API de listage ;
* *continuation_token* (jeton de liaison), qui est transmis à l’appel suivant pour obtenir la page de résultats suivante. Quand il n’y a plus de résultats à retourner, le jeton de liaison prend la valeur null.

Par exemple, un appel type destiné à lister tous les objets blob présents dans un conteneur peut ressembler à l'extrait de code suivant. Le code est disponible dans nos [exemples](https://github.com/Azure/azure-storage-cpp/blob/master/Microsoft.WindowsAzure.Storage/samples/BlobsGettingStarted.cpp):

```cpp
// List blobs in the blob container
azure::storage::continuation_token token;
do
{
    azure::storage::list_blob_item_segment segment = container.list_blobs_segmented(token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}

    token = segment.continuation_token();
}
while (!token.empty());
```

Notez que le nombre de résultats retournés dans une page peut être contrôlé par le paramètre *max_results* au niveau de la surcharge de chaque API, par exemple :

```cpp
list_blob_item_segment list_blobs_segmented(const utility::string_t& prefix, bool use_flat_blob_listing,
    blob_listing_details::values includes, int max_results, const continuation_token& token,
    const blob_request_options& options, operation_context context)
```

Si vous ne spécifiez pas le paramètre *max_results* , le nombre de résultats retournés dans une seule page peut atteindre 5 000 résultats, soit la valeur maximale par défaut.

Sachez aussi qu’une requête au niveau du stockage d’une table Azure peut retourner aucun enregistrement ou moins d’enregistrements que la valeur du paramètre *max_results* que vous avez spécifiée, même si le jeton de liaison n’est pas vide. L’une des raisons possibles à cela est que la requête n’a pas pu aboutir dans un délai de cinq secondes. Tant que le jeton de liaison n'est pas vide, la requête doit se poursuivre et votre code ne doit pas présumer la taille des résultats du segment.

Le modèle de codage recommandé dans la plupart des scénarios est le listing segmenté, qui indique la progression explicite de l’opération de listing ou d'interrogation et la façon dont le service répond à chaque demande. Un contrôle de niveau inférieur de la progression du listage peut aider à contrôler la mémoire et les performances, en particulier pour les applications ou services C++.

## <a name="greedy-listing"></a>Listage vorace

Les versions antérieures de la bibliothèque cliente Storage pour C++ (versions 0.5.0 préliminaire et antérieures) comprenaient des API de listage non segmenté pour les tables et les files d'attente, comme dans l'exemple suivant :

```cpp
std::vector<cloud_table> list_tables(const utility::string_t& prefix) const;
std::vector<table_entity> execute_query(const table_query& query) const;
std::vector<cloud_queue> list_queues() const;
```

Ces méthodes étaient implémentées en tant que wrappers d'API segmentées. À chaque réponse de listage segmenté, le code ajoutait les résultats à un vecteur et retournait tous les résultats après l’analyse complète des conteneurs.

Autant cette approche pouvait fonctionner quand le compte de stockage ou la table contenait peu d'objets, autant les besoins en mémoire pouvaient croître sans limite à mesure que le nombre d’objets augmentait, car tous les résultats restaient en mémoire. De plus, l'appelant ne disposait pas d'informations concernant la progression des opérations de listage, qui pouvaient prendre beaucoup de temps.

Ces API de listage, intégrées au Kit de développement logiciel (SDK), n'existent pas dans l'environnement C#, Java ou JavaScript Node.js. Pour éviter les problèmes potentiels liés à l'utilisation de ces API voraces, nous les avons retirées de la version 0.6.0 préliminaire.

Si votre code appelle ces API voraces :

```cpp
std::vector<azure::storage::table_entity> entities = table.execute_query(query);
for (auto it = entities.cbegin(); it != entities.cend(); ++it)
{
    process_entity(*it);
}
```

Vous avez tout intérêt à modifier votre code de façon à utiliser les API de listage segmenté :

```cpp
azure::storage::continuation_token token;
do
{
    azure::storage::table_query_segment segment = table.execute_query_segmented(query, token);
    for (auto it = segment.results().cbegin(); it != segment.results().cend(); ++it)
    {
        process_entity(*it);
    }

    token = segment.continuation_token();
} while (!token.empty());
```

En spécifiant le paramètre *max_results* du segment, vous pouvez établir un juste équilibre entre le nombre de demandes et l’utilisation de mémoire de façon à répondre à des considérations de performance pour votre application.

Par ailleurs, si vous utilisez les API de listage segmenté, mais que vous stockez les données dans une collection locale, méthode qui s’avère « vorace », nous vous recommandons vivement de refactoriser votre code pour une gestion soigneuse et adaptée du stockage des données dans une collection locale.

## <a name="lazy-listing"></a>Listage paresseux

Même si le listage vorace peut poser des problèmes, il est utile s’il n’y a pas trop d’objets dans le conteneur.

Si vous faites aussi appel à des SDK C# ou Oracle Java, vous devez connaître le modèle de programmation « énumérable », qui offre un mode de listage « paresseux », qui ne va chercher certaines données périphériques que s’il y est contraint. En C++, le modèle basé sur un itérateur offre une approche similaire.

Une API de listage paresseux type, utilisant **list_blobs** en guise d’exemple, se présente comme ceci :

```cpp
list_blob_item_iterator list_blobs() const;
```

Un extrait de code type qui utilise le modèle de listage paresseux peut se présenter comme ceci :

```cpp
// List blobs in the blob container
azure::storage::list_blob_item_iterator end_of_results;
for (auto it = container.list_blobs(); it != end_of_results; ++it)
{
    if (it->is_blob())
    {
        process_blob(it->as_blob());
    }
    else
    {
        process_directory(it->as_directory());
    }
}
```

Notez que le listage paresseux est disponible uniquement en mode synchrone.

Par rapport au listage vorace, le listage paresseux ne va chercher les données qu’en cas de nécessité. En réalité, il ne va chercher les données d’Azure Storage qu’à partir du moment où l'itérateur suivant se déplace dans le segment suivant. Par conséquent, l'utilisation de mémoire étant contrôlée et limitée par la taille, l'opération est rapide.

Les API de listage paresseux sont incluses dans la bibliothèque cliente Storage pour C++ de version 2.2.0.

## <a name="conclusion"></a>Conclusion

Dans cet article, nous nous sommes intéressés à différentes surcharges pour API de listage pour divers objets de la bibliothèque cliente Storage pour C++. Pour résumer :

* Les API asynchrones sont vivement recommandées dans divers scénarios de threading.
* Le listage segmenté est recommandé dans la plupart des scénarios.
* Le listage paresseux est fourni dans la bibliothèque sous forme de wrapper, qui s’avère pratique dans les scénarios synchrones.
* Le listage vorace est déconseillé et a été retiré de la bibliothèque.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d'informations sur Azure Storage et la bibliothèque cliente pour C++, consultez les ressources suivantes :

* [Utilisation du stockage d'objets blob à partir de C++](../blobs/storage-c-plus-plus-how-to-use-blobs.md)
* [Utilisation du stockage de tables à partir de C++](../../cosmos-db/table-storage-how-to-use-c-plus.md)
* [Utilisation du service de stockage de files d'attente à partir de C++](../queues/storage-c-plus-plus-how-to-use-queues.md)
* [Documentation sur les API de la bibliothèque cliente Azure Storage pour C++.](https://azure.github.io/azure-storage-cpp/)
* [Blog de l'équipe Azure Storage](/archive/blogs/windowsazurestorage/)
* [Documentation d’Azure Storage](https://azure.microsoft.com/documentation/services/storage/)
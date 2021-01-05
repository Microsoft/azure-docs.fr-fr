---
title: Modélisation de relations dans la conception de stockage de table Azure | Microsoft Docs
description: Découvrez le processus de modélisation quand vous concevez votre solution de stockage Table Azure. En savoir plus sur les relations de type un à plusieurs, un à un et héritage.
services: storage
ms.service: storage
author: tamram
ms.author: tamram
ms.topic: article
ms.date: 04/23/2018
ms.subservice: tables
ms.openlocfilehash: 3023b478ef7a4aaf6d9239e997bdf63282b56210
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88271190"
---
# <a name="modeling-relationships"></a>Modélisation des relations
Cet article décrit le processus de modélisation pour vous aider à concevoir vos solutions de Stockage Table Azure.

La création de modèles de domaine est une étape importante pour concevoir des systèmes complexes. En règle générale, le processus de modélisation permet d’identifier les entités et les relations entre elles, pour mieux comprendre le domaine de l’entreprise et concevoir votre système. Cette section se concentre sur la manière dont vous pouvez traduire certains des types de relations courantes découverts dans les modèles du domaine pour les conceptions du service de Table. Le processus de mappage d’un modèle de données logique vers un modèle de données NoSQL physique est différent de celui utilisé pour la conception d’une base de données relationnelle. La conception de bases de données relationnelles suppose généralement un processus de normalisation des données optimisé pour réduire la redondance, ainsi qu'une fonctionnalité de requête déclarative qui résume la façon dont fonctionne l'implémentation de la base de données.  

## <a name="one-to-many-relationships"></a>Relations un-à-plusieurs
Les relations un-à-plusieurs entre des objets de domaine d’entreprise se produisent fréquemment : par exemple, un service a de nombreux employés. Il existe plusieurs méthodes pour implémenter des relations un-à-plusieurs dans le service de Table. Chacune d'elles a des inconvénients et des avantages qui peuvent être pertinents pour un scénario particulier.  

Prenons l'exemple d'une grande entreprise multinationale ayant des dizaines de milliers d'entités relatives aux services (department) et aux employés (employee) où chaque service a de nombreux employés et chaque employé est associé à un service spécifique. Une approche consiste à stocker séparément les entités relatives aux services (department) et aux employés (employee), comme ceci :  


![Stocker des entités de service et d’employé distinctes](media/storage-table-design-guide/storage-table-design-IMAGE01.png)

Cet exemple montre une relation un-à-plusieurs implicite entre les types selon la valeur de **PartitionKey** . Chaque service peut avoir de nombreux employés.  

Cet exemple montre également une entité de service (department) et ses entités d'employés (employee) dans la même partition. Vous pouvez choisir d'utiliser des partitions, des tables ou même des comptes de stockage différents pour les différents types d'entité.  

Une autre approche consiste à dénormaliser vos données et à stocker uniquement les entités d'employés (employee) avec les données de services (department) dénormalisées, comme indiqué dans l'exemple suivant. Cette approche dénormalisée n'est peut-être pas la meilleure pour ce scénario si vous avez besoin de modifier les détails d'un responsable de service, car pour ce faire, vous devez mettre à jour chaque employé du service.  

![Entité d’employé](media/storage-table-design-guide/storage-table-design-IMAGE02.png)

Pour plus d’informations, consultez la section [Modèle de dénormalisation](table-storage-design-patterns.md#denormalization-pattern) plus loin dans ce guide.  

Le tableau suivant récapitule les avantages et les inconvénients de chacune des approches décrites ci-dessus pour le stockage des entités d'employés et de services qui partagent une relation un-à-plusieurs. Vous devez également envisager la fréquence à laquelle vous pensez effectuer diverses opérations : il peut être raisonnable d'avoir une conception qui inclut une opération coûteuse si cette opération ne se produit que rarement.  

<table>
<tr>
<th>Approche</th>
<th>Avantages</th>
<th>Inconvénients</th>
</tr>
<tr>
<td>Séparation des types d'entités, même partition, même table</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service en une seule opération.</li>
<li>Vous pouvez utiliser une transaction de groupe d’entités (Entity Group Transaction*, EGT) pour maintenir la cohérence si vous avez besoin de modifier une entité de service à chaque mise à jour/insertion/suppression d’une entité d’employé. Par exemple, si vous conservez un nombre d’employés de service pour chaque service.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer une entité d'employé et de service pour certaines activités du client.</li>
<li>Les opérations de stockage se produisent dans la même partition. Si les volumes de transactions sont élevés, cela peut entraîner une zone sensible.</li>
<li>Vous ne pouvez pas déplacer un employé vers un nouveau service à l'aide d'une EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Types d'entités distincts, différentes partitions ou différentes tables ou différents comptes de stockage</td>
<td>
<ul>
<li>Vous pouvez mettre à jour une entité de service ou une entité d'employé avec une seule opération.</li>
<li>Si les volumes de transactions sont élevés, ceci peut permettre de répartir la charge sur plusieurs partitions.</li>
</ul>
</td>
<td>
<ul>
<li>Vous devrez peut-être récupérer une entité d'employé et de service pour certaines activités du client.</li>
<li>Vous ne pouvez pas utiliser des EGT pour maintenir la cohérence lors d'une mise à jour/insertion/suppression d'employé et d'une mise à jour de service. Par exemple, la mise à jour d'un nombre d'employés dans une entité de service.</li>
<li>Vous ne pouvez pas déplacer un employé vers un nouveau service à l'aide d'une EGT.</li>
</ul>
</td>
</tr>
<tr>
<td>Dénormaliser en type d'entité unique</td>
<td>
<ul>
<li>Vous pouvez récupérer toutes les informations dont vous avez besoin en utilisant une seule demande.</li>
</ul>
</td>
<td>
<ul>
<li>Il peut être coûteux de maintenir la cohérence si vous devez mettre à jour les informations d'un service (cela vous oblige à mettre à jour tous les employés d'un service).</li>
</ul>
</td>
</tr>
</table>

*Pour plus d’informations, consultez [Transactions de groupe d’entités](table-storage-design.md#entity-group-transactions)  


Votre choix entre ces options, ainsi que la détermination des avantages et des inconvénients les plus significatifs, tout cela dépend de votre scénario d'application. Par exemple, quelle sera la fréquence de modification des entités de service ; toutes les requêtes de vos employés ont-elles besoin d'informations de services supplémentaires ; êtes-vous proche des limites d'évolutivité de vos partitions ou de votre compte de stockage ?  

## <a name="one-to-one-relationships"></a>Relations un à un
Les modèles de domaines peuvent inclure des relations un à un entre les entités. Si vous devez implémenter une relation un à un dans le service de Table, vous devez également choisir comment lier les deux entités associées lorsque vous avez besoin de récupérer les deux. Ce lien peut être implicite s’il est basé sur une convention dans les valeurs de clé, ou explicite si l’on stocke un lien sous forme de valeurs de **PartitionKey** et de **RowKey** dans chaque entité et son entité associée. Pour savoir quand stocker les entités associées dans la même partition, consultez la section [Relations un à plusieurs](#one-to-many-relationships).  

Certaines considérations sur l’implémentation peuvent vous conduire à implémenter des relations un à un dans le service de Table :  

* Gestion des entités volumineuses (pour plus d’informations, consultez [Modèle d’entités volumineuses](table-storage-design-patterns.md#large-entities-pattern))  
* L’implémentation de contrôles d’accès (pour plus d’informations, consultez Contrôle d’accès avec des signatures d’accès partagé).  

## <a name="join-in-the-client"></a>Joindre le client
Bien qu'il existe des façons de modéliser des relations dans le service de Table, n'oubliez pas que les deux principaux motifs pour utiliser le service de Table sont l'évolutivité et les performances. Si vous devez modéliser plusieurs relations pouvant compromettre les performances et la scalabilité de votre solution, demandez-vous s’il est nécessaire de générer toutes les relations de données dans votre conception de table. Vous pouvez peut-être simplifier la conception et améliorer l'évolutivité et les performances de votre solution si vous laissez votre application cliente effectuer les jointures nécessaires.  

Par exemple, si vous avez des petites tables qui contiennent des données qui ne changent pas souvent, vous pouvez récupérer ces données une fois et les mettre en cache sur le client. Cela peut éviter des allers-retours répétés pour récupérer les mêmes données. Dans les exemples que nous avons vus dans ce guide, l'ensemble des services d'une petite entreprise sera réduit et ne risque pas d'être modifié très souvent, ce qui en fait un bon candidat pour l'emploi de données que l'application cliente peut télécharger une fois et mettre en cache en tant que données de recherche.  

## <a name="inheritance-relationships"></a>Relations d'héritage
Si votre application cliente utilise un ensemble de classes qui font partie d'une relation d'héritage pour représenter des entités métier, vous pouvez facilement conserver ces entités dans le service de Table. Par exemple, l’ensemble de classes suivant peut être défini dans votre application cliente, **Person** étant une classe abstraite.

![Classe abstraite Person](media/storage-table-design-guide/storage-table-design-IMAGE03.png)

Vous pouvez conserver les instances de deux classes concrètes dans le service de Table à l'aide d'une seule table Person à l'aide d'entités qui ressemblent à ceci :  

![Table Person](media/storage-table-design-guide/storage-table-design-IMAGE04.png)

Pour plus d’informations sur l’utilisation de plusieurs types d’entités dans la même table dans le code client, consultez la section Utilisation des types d’entités hétérogènes plus loin dans ce guide. Vous y trouverez des exemples montrant comment reconnaître le type d'entité dans le code client.  


## <a name="next-steps"></a>Étapes suivantes

- [Modèles de conception de table](table-storage-design-patterns.md)
- [Conception pour l’interrogation](table-storage-design-for-query.md)
- [Chiffrer des données de table](table-storage-design-encrypt-data.md)
- [Conception pour la modification de données](table-storage-design-for-modification.md)

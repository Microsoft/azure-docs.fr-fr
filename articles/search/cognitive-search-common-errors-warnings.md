---
title: Erreurs et avertissements de l’indexeur
titleSuffix: Azure Cognitive Search
description: Cet article fournit des informations et des solutions aux erreurs et aux avertissements courants que vous pourriez rencontrer lors de l’enrichissement de l’IA dans Recherche cognitive Azure.
manager: nitinme
author: nitinme
ms.author: nitinme
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 09/23/2020
ms.openlocfilehash: c65c9265d8eb4a06ea354c6d753cc6ed847eb6db
ms.sourcegitcommit: a4533b9d3d4cd6bb6faf92dd91c2c3e1f98ab86a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/22/2020
ms.locfileid: "97724287"
---
# <a name="troubleshooting-common-indexer-errors-and-warnings-in-azure-cognitive-search"></a>Résoudre les erreurs et les avertissements courants de l’indexeur dans la Recherche cognitive Azure

Cet article fournit des informations et des solutions pour les erreurs et avertissements courants que vous pourriez rencontrer pendant l’indexation et l’enrichissement par IA dans Recherche cognitive Azure.

L’indexation s’arrête quand le nombre d’erreurs dépasse la valeur de ['maxFailedItems'](cognitive-search-concept-troubleshooting.md#tip-3-see-what-works-even-if-there-are-some-failures). 

Si vous souhaitez que les indexeurs ignorent ces erreurs (et sautent les « documents en échec »), envisagez de mettre à jour `maxFailedItems` et `maxFailedItemsPerBatch` comme décrit [ici](/rest/api/searchservice/create-indexer#general-parameters-for-all-indexers).

> [!NOTE]
> Chaque document en échec et la clé de document correspondante (quand elle est disponible) sont présentées comme une erreur dans l’état d’exécution de l’indexeur. Vous pouvez utiliser l’[api index](/rest/api/searchservice/addupdate-or-delete-documents) pour charger manuellement les documents à un moment ultérieur si vous avez défini l’indexeur de façon à tolérer les échecs.

Les informations d’erreur de cet article peuvent vous aider à résoudre les erreurs et à poursuivre l’indexation.

L’indexation des avertissements ne s’arrête pas, mais certaines conditions pourraient entraîner des résultats inattendus. Vos données et votre scénario conditionnent les mesures que vous devriez prendre.

À partir de la version d’API `2019-05-06`, les erreurs et les avertissements de l’indexeur au niveau de l’élément sont structurés de manière à fournir une clarté accrue sur les causes et la marche à suivre. Ils contiennent les propriétés suivantes :

| Propriété | Description | Exemple |
| --- | --- | --- |
| key | ID du document concerné par l’erreur ou l’avertissement. | https:\//coromsearch.blob.core.windows.net/jfk-1k/docid-32112954.pdf |
| name | Nom de l’opération décrivant l’emplacement où l’erreur ou l’avertissement s’est produit. Il est généré selon la structure suivante : [catégorie].[sous-catégorie].[ressourceType].[ressourceNom] | DocumentExtraction.azureblob.myBlobContainerName Enrichment.WebApiSkill.mySkillName Projection.SearchIndex.OutputFieldMapping.myOutputFieldName Projection.SearchIndex.MergeOrUpload.myIndexName Projection.KnowledgeStore.Table.myTableName |
| message | Description de haut niveau de l’erreur ou de l’avertissement. | Impossible d’exécuter la compétence en raison de l’échec de la requête de l’API web. |
| details | Tous les détails supplémentaires qui peuvent être utiles pour diagnostiquer le problème, tels que la réponse WebApi en cas d’échec de l’exécution d’une compétence personnalisée. | `link-cryptonyms-list - Error processing the request record : System.ArgumentNullException: Value cannot be null. Parameter name: source at System.Linq.Enumerable.All[TSource](IEnumerable`1 source, Func`2 predicate) at Microsoft.CognitiveSearch.WebApiSkills.JfkWebApiSkills.` …reste de l’arborescence des appels de procédure… |
| documentationLink | Lien vers la documentation appropriée contenant des informations détaillées pour déboguer et résoudre le problème. Ce lien mène souvent sur cette page, à l’une des sections ci-dessous. | https://go.microsoft.com/fwlink/?linkid=2106475 |

<a name="could-not-read-document"></a>

## <a name="error-could-not-read-document"></a>Erreur : Impossible de lire le document

L'indexeur n'a pas pu lire le document à partir de la source de données. Cela peut se produire si :

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Types de champs incohérents dans différents documents | « Le type de valeur ne correspond pas au type de colonne. Impossible de stocker `'{47.6,-122.1}'` dans la colonne des auteurs.  Le type attendu est JArray. »  « Erreur lors de la conversion du type de données nvarchar en float. »  « Échec de la conversion lors de la conversion de la valeur nvarchar "12 mois" en type de données int. »  « Une erreur de dépassement arithmétique s'est produite lors de la conversion de l'expression en type de données int. » | Assurez-vous que le type de chaque champ est identique dans les différents documents. Par exemple, si le champ `'startTime'` du premier document est un champ DateTime et une chaîne de caractères dans le second document, cette erreur s’affichera. |
| erreurs provenant du service sous-jacent de la source de données | (de Cosmos DB) `{"Errors":["Request rate is large"]}` | Vérifiez l’intégrité de votre instance de stockage. Vous devrez peut-être ajuster votre mise à l'échelle/partitionnement. |
| problèmes temporaires | Une erreur de niveau transport s’est produite lors de la réception des résultats à partir du serveur. (fournisseur : Fournisseur TCP, erreur : 0 - Une connexion existante a été fermée de force par l'hôte distant | Certains problèmes de connectivité inattendus peuvent survenir. Essayez d'exécuter ultérieurement le document dans votre indexeur. |

<a name="could-not-extract-document-content"></a>

## <a name="error-could-not-extract-content-or-metadata-from-your-document"></a>Erreur : Impossible d’extraire le contenu ou les métadonnées de votre document
L’indexeur avec une source de données Blob n’a pas pu extraire le contenu ou les métadonnées du document (par exemple, un fichier PDF). Cela peut se produire si :

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| l’objet Blob dépasse la limite de taille | Le document affiche une taille de `'134217728'`, ce qui est supérieur à la taille maximale de `'150441598'` pour l’extraction de document correspondant à votre niveau de service actuel. | [Erreurs d’indexation des objets Blob](search-howto-indexing-azure-blob-storage.md#DealingWithErrors) |
| L’objet Blob a un type de contenu non pris en charge | Le document contient un type de contenu `'image/png'` non pris en charge | [Erreurs d’indexation des objets Blob](search-howto-indexing-azure-blob-storage.md#DealingWithErrors) |
| L’objet Blob est chiffré | Le document n'a pas pu être traité car il est peut-être chiffré ou protégé par un mot de passe. | Vous pouvez ignorer le blob grâce aux [paramètres de blob](search-howto-indexing-azure-blob-storage.md#PartsOfBlobToIndex). |
| problèmes temporaires | « Erreur de traitement des objets blob : La demande a été abandonnée : La demande a été annulée. » « Le document a expiré pendant le traitement ». | Certains problèmes de connectivité inattendus peuvent survenir. Essayez d'exécuter ultérieurement le document dans votre indexeur. |

<a name="could-not-parse-document"></a>

## <a name="error-could-not-parse-document"></a>Erreur : Impossible d’analyser le document
L’indexeur a lu le document à partir de la source de données, mais un problème est survenu lors de la conversion du contenu du document au schéma de mappage de champs spécifié. Cela peut se produire si :

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| La clé du document est manquante | La clé du document ne peut pas être manquante ou vide | Vérifiez que tous les documents contiennent des clés de documents valides. La clé de document est déterminée en définissant la propriété « clé » dans le cadre de la [définition d’index](/rest/api/searchservice/create-index#request-body). Les indexeurs émettent cette erreur quand la propriété marquée en tant que « clé » est introuvable dans un document particulier. |
| La clé du document n’est pas valide | La clé du document ne peut pas contenir plus de 1024 caractères | Modifiez la clé du document pour répondre aux exigences de validation. |
| Impossible d'appliquer le mappage de champs à un champ | Impossible d'appliquer la fonction de mappage `'functionName'` au champ `'fieldName'`. Le tableau ne peut pas être null. Nom du paramètre : octets | Vérifiez soigneusement les [mappages de champs](search-indexer-field-mappings.md) définis sur l'indexeur, puis comparez ces valeurs avec les données du champ spécifié du document en échec. Il peut être nécessaire de modifier les mappages de champs ou les données du document. |
| Impossible de lire la valeur du champ | Impossible de lire la valeur de la colonne `'fieldName'` à l'index `'fieldIndex'`. Une erreur de niveau transport s’est produite lors de la réception des résultats à partir du serveur. (fournisseur : Fournisseur TCP, erreur : 0 - Une connexion existante a été fermée de force par l'hôte distant.) | Ces erreurs sont généralement dues à des problèmes de connectivité inattendus avec le service sous-jacent de la source de données. Essayez d'exécuter ultérieurement le document dans votre indexeur. |

<a name="Could not map output field '`xyz`' to search index due to deserialization problem while applying mapping function '`abc`'"></a>

## <a name="error-could-not-map-output-field-xyz-to-search-index-due-to-deserialization-problem-while-applying-mapping-function-abc"></a>Erreur : Impossible de mapper le champ de sortie « `xyz` » avec l’index de recherche en raison d’un problème de désérialisation lors de l’application de la fonction de mappage « `abc` »
Le mappage de sortie a peut-être échoué, car les données de sortie sont dans un format incorrect pour la fonction de mappage que vous utilisez. Par exemple, l’application de la fonction de mappage Base64Encode sur des données binaires génère cette erreur. Pour résoudre le problème, réexécutez l’indexeur sans spécifier la fonction de mappage ou assurez-vous que la fonction de mappage est compatible avec le type de données du champ de sortie. Pour plus d’informations, consultez [Mappage de champs de sortie](cognitive-search-output-field-mapping.md).

<a name="could-not-execute-skill"></a>

## <a name="error-could-not-execute-skill"></a>Erreur : Impossible d’exécuter une compétence
L’indexeur n’a pas pu exécuter une compétence dans l’ensemble de compétences.

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Problèmes de connectivité temporaires | Une erreur temporaire s’est produite. Veuillez réessayer plus tard. | Certains problèmes de connectivité inattendus peuvent survenir. Essayez d'exécuter ultérieurement le document dans votre indexeur. |
| Bogue potentiel du produit | Une erreur inattendue s’est produite. | Cela indique une classe de défaillance inconnue et peut signifier qu’il existe un bogue de produit. Pour obtenir de l’aide, créez un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support). |
| Une compétence a rencontré une erreur lors de son exécution | (à partir de Compétence Fusion) Une ou plusieurs valeurs de décalage n’étaient pas valides. Il n’a pas été possible de les analyser. Des éléments ont été insérés à la fin du texte | Utilisez les informations contenues dans le message d’erreur pour résoudre le problème. La résolution de ce type de défaillance nécessite une action. |

<a name="could-not-execute-skill-because-the-web-api-request-failed"></a>

## <a name="error-could-not-execute-skill-because-the-web-api-request-failed"></a>Erreur : Impossible d’exécuter la compétence en raison de l’échec de la requête de l’API web
L’exécution de la compétence a échoué parce que l’appel de l’API web a échoué. En règle générale, cette classe de défaillance se produit lorsque des compétences personnalisées sont utilisées. Dans ce cas, vous devez déboguer votre code personnalisé pour résoudre le problème. Si au contraire l’échec provient d’une compétence intégrée, consultez le message d’erreur dans lequel vous trouverez peut-être de l’aide pour résoudre le problème.

Lors du débogage de ce problème, prêtez attention aux éventuels [avertissements d’entrée de compétence](#warning-skill-input-was-invalid) pour cette compétence. Le point de terminaison de votre API web peut échouer, car l’indexeur lui transmet une entrée inattendue.

<a name="could-not-execute-skill-because-web-api-skill-response-is-invalid"></a>

## <a name="error-could-not-execute-skill-because-web-api-skill-response-is-invalid"></a>Erreur : Impossible d’exécuter la compétence parce que la réponse concernant la compétence de l’API web n’est pas valide
L’exécution de la compétence a échoué parce que l’API web a retourné une réponse non valide. En règle générale, cette classe de défaillance se produit lorsque des compétences personnalisées sont utilisées. Dans ce cas, vous devez déboguer votre code personnalisé pour résoudre le problème. Si au contraire l’échec est dû à une compétence intégrée, créez un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support) pour obtenir de l’aide.

<a name="skill-did-not-execute-within-the-time-limit"></a>

## <a name="error-skill-did-not-execute-within-the-time-limit"></a>Erreur : La compétence n'a pas été exécutée dans le délai imparti
Il existe deux cas dans lesquels vous pouvez rencontrer ce message d'erreur, chacun d'entre eux devant être traité différemment. Veuillez suivre les instructions ci-dessous en fonction de la compétence qui a retourné cette erreur.

### <a name="built-in-cognitive-service-skills"></a>Compétences de service cognitives intégrées
Bon nombre des compétences cognitives intégrées, notamment la détection de la langue, la reconnaissance d'entités ou la reconnaissance optique de caractères (OCR), s’appuient sur un point de terminaison Cognitive Service API. Parfois, des problèmes temporaires peuvent survenir avec ces points de terminaison, entraînant l’expiration d’une demande. Il n’existe aucun remède pour ces problèmes temporaires, si ce n'est d'attendre et de réessayer. Pour résoudre le problème, réglez votre indexeur afin de l’[exécuter selon une planification](search-howto-schedule-indexers.md). L'indexation planifiée reprend là où elle s'était arrêtée. Si les problèmes temporaires ont été résolus, l'indexation et le traitement des compétences cognitives devraient reprendre à la prochaine exécution planifiée.

Si vous continuez à voir cette erreur sur le même document pour une compétence cognitive intégrée, veuillez remplir un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support) pour obtenir de l’aide, car cela n’est pas prévu.

### <a name="custom-skills"></a>Compétences personnalisées
Si vous rencontrez une erreur d’expiration d’une compétence personnalisée que vous avez créée, vous pouvez essayer plusieurs solutions. Tout d'abord, passez en revue votre compétence personnalisée et vérifiez qu'elle ne se retrouve pas bloquée dans une boucle infinie et qu'elle renvoie un résultat constant. Une fois ce point vérifié, déterminez le délai d'exécution de votre compétence. Si vous n'avez pas explicitement défini une valeur `timeout` dans votre définition de compétence personnalisée, la valeur par défaut `timeout` est 30 secondes. Si 30 secondes ne suffisent pas à l’exécution de votre compétence, vous pouvez spécifier une valeur plus élevée `timeout` dans la définition de votre compétence personnalisée. Voici un exemple de définition de compétence personnalisée où le délai d’expiration est fixé à 90 secondes :

```json
  {
        "@odata.type": "#Microsoft.Skills.Custom.WebApiSkill",
        "uri": "<your custom skill uri>",
        "batchSize": 1,
        "timeout": "PT90S",
        "context": "/document",
        "inputs": [
          {
            "name": "input",
            "source": "/document/content"
          }
        ],
        "outputs": [
          {
            "name": "output",
            "targetName": "output"
          }
        ]
      }
```

La valeur maximale que vous pouvez définir pour le paramètre `timeout` est de 230 secondes.  Si votre compétence personnalisée ne peut pas s'exécuter de façon constante en 230 secondes, vous pouvez réduire la valeur `batchSize` de votre compétence personnalisée afin qu'elle ait moins de documents à traiter en une seule exécution.  Si vous avez déjà réglé votre valeur `batchSize` sur 1, vous devrez réécrire la compétence pour pouvoir l'exécuter en moins de 230 secondes ou la diviser en plusieurs compétences personnalisées de sorte que le délai d'exécution d'une seule compétence personnalisée soit au maximum de 230 secondes. Pour plus d'informations, consultez la [documentation sur les compétences personnalisées](cognitive-search-custom-skill-web-api.md).

<a name="could-not-mergeorupload--delete-document-to-the-search-index"></a>

## <a name="error-could-not-mergeorupload--delete-document-to-the-search-index"></a>Erreur : Impossible de « `MergeOrUpload` » | « `Delete` » le document dans l’index de recherche

Le document a été lu et traité, mais l’indexeur n’a pas pu l’ajouter à l’index de recherche. Cela peut se produire si :

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Un champ contient un terme trop grand | Votre document contient un terme qui dépasse la [limite de 32 Ko](search-limits-quotas-capacity.md#api-request-limits) | Vous pouvez éviter cette restriction en vérifiant que le champ n’est pas configuré comme étant filtrable, à choix multiple ou triable.
| Le document est trop volumineux pour être indexé | Un document est plus volumineux que la [taille maximale de demande d’API](search-limits-quotas-capacity.md#api-request-limits) | [Indexer les jeux de données volumineux](search-howto-large-index.md)
| Le document contient un trop grand nombre d’objets dans la collection | Une collection dans votre document dépasse la [limite maximale d’éléments pour toutes collections complexes](search-limits-quotas-capacity.md#index-limits) « Le document avec la clé `'1000052'` contient `'4303'` objets dans des collections (tableaux JSON). `'3000'` objets maximum sont autorisés dans des collections sur l’ensemble du document. Supprimez des objets des collections et réessayez d’indexer le document. » | Nous vous recommandons de réduire la taille de la collection complexe dans le document en dessous de la limite et d’éviter une utilisation intensive du stockage.
| Difficultés à se connecter à l’index cible (qui persiste après les nouvelles tentatives), car le service est soumis à une autre charge, par exemple l’interrogation ou l’indexation. | Échec de l’établissement d’une connexion pour mettre à jour l’index. Le service de recherche est soumis à une charge importante. | [Effectuer le scale-up de votre service de recherche](search-capacity-planning.md)
| Un correctif est appliqué au service de recherche en vue de sa mise à jour ou fait l’objet d’une reconfiguration de topologie. | Échec de l’établissement d’une connexion pour mettre à jour l’index. Le service de recherche est actuellement inopérant/Le service de recherche est en cours de transition. | Configurer le service avec au moins trois réplicas pour une disponibilité de 99,9 % selon la [documentation SLA](https://azure.microsoft.com/support/legal/sla/search/v1_0/)
| Échec dans la ressource de calcul/réseau sous-jacente (rare) | Échec de l’établissement d’une connexion pour mettre à jour l’index. Une erreur inconnue s’est produite. | Configurer les indexeurs pour une [exécution selon une planification](search-howto-schedule-indexers.md) pour récupérer d’un état d’échec.
| Une requête d’indexation envoyée à l’index cible n’a pas reçu d’accusé de réception pendant la période définie en raison de problèmes réseau. | Impossible d’établir la connexion à l’index de recherche en temps opportun. | Configurer les indexeurs pour une [exécution selon une planification](search-howto-schedule-indexers.md) pour récupérer d’un état d’échec. Essayez également de réduire la [taille du lot](/rest/api/searchservice/create-indexer#parameters) de l’indexeur si cet état d’erreur persiste.

<a name="could-not-index-document-because-the-indexer-data-to-index-was-invalid"></a>

## <a name="error-could-not-index-document-because-some-of-the-documents-data-was-not-valid"></a>Erreur : impossible d’indexer le document parce que certaines données de celui-ci n’étaient pas valides

L’indexeur a lu et traité le document mais, en raison d’une discordance de configuration des champs d’index et des données extraites et traitées par l’indexeur, il n’a pas été possible d’ajouter les données à l’index de recherche. Cela peut se produire si :

| Motif | Détails/Exemple
| --- | ---
| Le type de données du ou des champs extraits par l’indexeur est incompatible avec le modèle de données du champ d’index cible correspondant. | Le champ de données « _data_ » dans le document avec la clé « 888 » a une valeur non valide de type « Edm.String ». Le type attendu était « collection (EDM. String) ». |
| Échec d’extraction d’une entité JSON à partir d’une valeur de chaîne. | Impossible d’analyser la valeur de type « Edm.String » du champ « _data_ » en tant qu’objet JSON. Erreur : « Après analyse d’une valeur, un caractère inattendu a été rencontré : «’. Chemin d’accès « _path_ », ligne 1, position 3162. » |
| Échec d’extraction d’une collection d’entités JSON d’une valeur de chaîne.  | Impossible d’analyser la valeur de type « Edm.String » du champ « _data_ » sous la forme d’un tableau JSON. Erreur : « Après analyse d’une valeur, un caractère inattendu a été rencontré : «’. Chemin d’accès « [0] », ligne 1, position 27. » |
| Un type inconnu a été découvert dans le document source. | Le type inconnu « _unknown_ » ne peut pas être indexé |
| Une notation incompatible pour les points géographiques a été utilisée dans le document source. | Les littéraux de chaîne WKT POINT ne sont pas pris en charge. Utilisez des littéraux de points GeoJson à la place |

Dans tous ces cas, consultez les [types de données pris en charge ](/rest/api/searchservice/supported-data-types) et le [mappage des types de données pour les indexeurs](/rest/api/searchservice/data-type-map-for-indexers-in-azure-search) pour vous assurer de générer le schéma d’index correctement et de définir les [mappages de champs d’indexeur](search-indexer-field-mappings.md) appropriés. Le message d’erreur comprendra des détails qui peuvent aider à identifier la source de l’incompatibilité.

## <a name="error-integrated-change-tracking-policy-cannot-be-used-because-table-has-a-composite-primary-key"></a>Erreur : Impossible d’utiliser la stratégie de suivi des modifications intégré car la table contient une clé primaire composite

Cela s’applique aux tables SQL, et se produit généralement lorsque la clé est définie en tant que clé composite ou, lorsque la table a défini un index cluster unique (comme avec l’index SQL, mais pas l’index Azure Search). Ceci est principalement dû au fait que l’attribut de clé est transformé en une clé primaire composite dans le cas d’un [index cluster unique](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described). Dans ce cas, vérifiez que votre table SQL n’a pas d’index cluster unique, ou que vous mappez le champ clé à un champ pour lequel les valeurs en double sont impossibles.

<a name="could-not-process-document-within-indexer-max-run-time"></a>

## <a name="error-could-not-process-document-within-indexer-max-run-time"></a>Erreur : Impossible de traiter le document en respectant le temps d’exécution max. de l’indexeur

Cette erreur se produit lorsque l’indexeur ne peut pas terminer le traitement d’un document unique à partir de la source de données en respectant le temps d’exécution autorisé. [Le temps d’exécution maximal](search-limits-quotas-capacity.md#indexer-limits) est plus court quand des ensembles de compétences sont utilisés. Lorsque cette erreur se produit, si maxFailedItems est défini sur une valeur autre que 0, l’indexeur ignore le document pour les prochaines exécutions, de façon que l’indexation puisse progresser. Si vous ne pouvez pas vous permettre d’ignorer un document, ou si vous voyez cette erreur en permanence, pensez à diviser les documents en documents plus petits pour qu’une seule exécution de l’indexeur puisse traiter une partie de ces derniers.

<a name="could-not-project-document></a>

## <a name="error-could-not-project-document"></a>Erreur : Impossible de projeter le document

Cette erreur se produit lorsque l’indexeur tente de [projeter des données dans une base de connaissances](knowledge-store-projection-overview.md) et échoue.  Cet échec peut être systématique et réparable, ou il peut s’agir d’un échec temporaire au niveau du récepteur de sortie de projection. Dans ce cas, vous devrez attendre un peu avant de réessayer.  Voici certains états d’échecs connus et des solutions possibles.

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Impossible de mettre à jour l’objet blob de projection `'blobUri'` dans le conteneur `'containerName'` |Le conteneur spécifié n’existe pas. | L’indexeur vérifie si le conteneur spécifié a déjà été créé et le crée si nécessaire, mais il n’effectue cette vérification qu’une seule fois par exécution de l’indexeur. Cette erreur signifie que quelque chose a supprimé le conteneur après cette étape.  Pour résoudre cette erreur, essayez ceci : ne modifiez pas les informations de votre compte de stockage, attendez que l’exécution de l’indexeur se termine, puis réexécutez-le. |
| Impossible de mettre à jour l’objet blob de projection `'blobUri'` dans le conteneur `'containerName'` |Impossible d’écrire les données dans la connexion de transport : une connexion existante a dû être fermée par l’hôte distant. | Il s’agit d’un échec temporaire au niveau du stockage Azure. Pour résoudre le problème, vous devez réexécuter l’indexeur. Si vous rencontrez cette erreur de manière systématique, créez un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support) afin que nous examinions le problème plus en détail.  |
| Impossible de mettre à jour la ligne `'projectionRow'` dans la table `'tableName'` | Le serveur est occupé. | Il s’agit d’un échec temporaire au niveau du stockage Azure. Pour résoudre le problème, vous devez réexécuter l’indexeur. Si vous rencontrez cette erreur de manière systématique, créez un [ticket de support](https://ms.portal.azure.com/#create/Microsoft.Support) afin que nous examinions le problème plus en détail.  |

<a name="could-not-execute-skill-because-a-skill-input-was-invalid"></a>

## <a name="warning-skill-input-was-invalid"></a>Avertissement : Entrée de compétence non valide
Il manque une entrée à la compétence, un type est incorrect ou non valide. Le message d’avertissement indique l’impact :
1) Impossible d’exécuter une compétence
2) Compétence exécutée mais susceptible d’engendrer des résultats inattendus

Les compétences cognitives nécessitaient des entrées et des entrées facultatives. Par exemple, la [compétence d’extraction d’expression clé](cognitive-search-skill-keyphrases.md) a deux entrées obligatoires, `text` et `languageCode`, et aucune entrée facultative. Les entrées de compétences personnalisées sont toutes considérées comme des entrées facultatives.

S’il manque des entrées obligatoires ou si une entrée n’est pas du bon type, la compétence est ignorée et génère un avertissement. Les compétences ignorées ne génèrent aucun résultat. Par conséquent, si d’autres compétences utilisent des résultats de la compétence ignorée, elles peuvent générer des avertissements supplémentaires.

S’il manque une entrée facultative, la compétence est quand même exécutée, mais elle risque de produire une sortie inattendue en raison de l’entrée manquante.

Dans les deux cas, cet avertissement peut être attendu en raison de la forme de vos données. Par exemple, si vous avez un document contenant des informations sur des personnes avec les champs `firstName`, `middleName` et `lastName`, certains documents n’ont peut-être pas d’entrée pour `middleName`. Si vous passez `middleName` en tant qu’entrée à une compétence dans le pipeline, alors il est attendu que cette entrée de compétence soit parfois manquante. Vous avez besoin d’évaluer vos données et votre scénario pour déterminer si une action est nécessaire suite à cet avertissement.

Si vous souhaitez fournir une valeur par défaut en cas d’entrée manquante, vous pouvez utiliser la [compétence conditionnelle](cognitive-search-skill-conditional.md) pour générer une valeur par défaut, puis utiliser la sortie de la [compétence conditionnelle](cognitive-search-skill-conditional.md) en tant qu’entrée de compétence.


```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == null" },
        { "name": "whenTrue", "source": "= 'en'" },
        { "name": "whenFalse", "source": "= $(/document/language)" }
    ],
    "outputs": [ { "name": "output", "targetName": "languageWithDefault" } ]
}
```

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Le type de l’entrée de compétence est incorrect | « L’entrée de compétence requise n’est pas du type attendu `String`. Nom : `text`, source : `/document/merged_content`. »  « L’entrée de compétence requise n’est pas au format attendu. Nom : `text`, source : `/document/merged_content`. »  « Impossible d’effectuer une itération sur le non-tableau `/document/normalized_images/0/imageCelebrities/0/detail/celebrities` ».  « Impossible de sélectionner `0` dans le non-tableau `/document/normalized_images/0/imageCelebrities/0/detail/celebrities` » | Certaines compétences attendent des entrées de types particuliers. Par exemple, la [compétence Sentiment](cognitive-search-skill-sentiment.md) attend que `text` soit une chaîne. Si l’entrée spécifie une valeur autre qu’une chaîne, la compétence ne s’exécute pas et ne génère aucune sortie. Assurez-vous que le jeu de données contient des valeurs d’entrée de type uniforme, ou utilisez une [compétence d’API web personnalisée](cognitive-search-custom-skill-web-api.md) pour prétraiter l’entrée. Si vous répétez la compétence sur un tableau, vérifiez que le contexte et l’entrée de la compétence ont `*` aux bons emplacements. Généralement, le contexte et la source d’entrée doivent se terminer par `*` pour des tableaux. |
| Une entrée de compétence est manquante | « L’entrée de compétence requise est manquante. Nom : `text`, source : `/document/merged_content` » « Valeur manquante `/document/normalized_images/0/imageTags`. »  « Impossible de sélectionner `0` dans un tableau `/document/pages` de longueur `0`. » | Si tous les documents reçoivent cet avertissement, il est hautement probable que les chemins d’entrée contiennent une faute de frappe et vous devriez vérifier attentivement la casse du nom de propriété, un signe `*` manquant ou en trop dans le chemin. Vérifiez aussi que les documents de la source de données définissent les entrées nécessaires. |
| L’entré du code de la langue de la compétence n’est pas valide | L’entrée de compétence `languageCode` a les codes de langue suivants `X,Y,Z`, dont au moins un n’est pas valide. | Pour plus de détails, voir [ci-dessous](cognitive-search-common-errors-warnings.md#skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid) |

<a name="skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid"></a>

## <a name="warning--skill-input-languagecode-has-the-following-language-codes-xyz-at-least-one-of-which-is-invalid"></a>Avertissement :  L'entrée de compétence 'languageCode' contient les codes de langue suivants 'X,Y,Z', dont au moins un élément n’est pas valide.
Une ou plusieurs des valeurs passées dans l'entrée optionnelle `languageCode` d'une compétence en aval n'est pas prise en charge. Cela peut se produire si vous passez la sortie de [LanguageDetectionSkill](cognitive-search-skill-language-detection.md) à des compétences ultérieures, et que la sortie comporte plus de langues que celles prises en charge dans ces compétences en aval.

Si vous savez que votre jeu de données utilise une seule langue, vous devriez supprimer [LanguageDetectionSkill](cognitive-search-skill-language-detection.md) et l’entrée de compétence `languageCode`, puis utiliser plutôt `defaultLanguageCode` pour ce paramètre de compétence, en supposant que la langue est prise en charge pour cette compétence.

Si vous savez que votre jeu de données contient plusieurs langues et que vous avez donc besoin de [LanguageDetectionSkill](cognitive-search-skill-language-detection.md) et de l’entrée `languageCode`, ajoutez un paramètre [ConditionalSkill](cognitive-search-skill-conditional.md) pour filtrer le texte selon les langues non prises en charge avant de passer le texte à la compétence en aval.  Voici un exemple de résultat avec le paramètre EntityRecognitionSkill :

```json
{
    "@odata.type": "#Microsoft.Skills.Util.ConditionalSkill",
    "context": "/document",
    "inputs": [
        { "name": "condition", "source": "= $(/document/language) == 'de' || $(/document/language) == 'en' || $(/document/language) == 'es' || $(/document/language) == 'fr' || $(/document/language) == 'it'" },
        { "name": "whenTrue", "source": "/document/content" },
        { "name": "whenFalse", "source": "= null" }
    ],
    "outputs": [ { "name": "output", "targetName": "supportedByEntityRecognitionSkill" } ]
}
```

Voici quelques références pour les langues actuellement prises en charge pour chacune des compétences qui peuvent générer ce message d'erreur :
* [Langues prises en charge pour l'analyse de texte](../cognitive-services/text-analytics/language-support.md) (pour [KeyPhraseExtractionSkill](cognitive-search-skill-keyphrases.md), [EntityRecognitionSkill](cognitive-search-skill-entity-recognition.md), [SentimentSkill](cognitive-search-skill-sentiment.md) et [PIIDetectionSkill](cognitive-search-skill-pii-detection.md))
* [Langues prises en charge par le traducteur](../cognitive-services/translator/language-support.md) (pour [Text TranslationSkill](cognitive-search-skill-text-translation.md))
* [Text SplitSkill](cognitive-search-skill-textsplit.md) - langues prises en charge : `da, de, en, es, fi, fr, it, ko, pt`

<a name="skill-input-was-truncated"></a>

## <a name="warning-skill-input-was-truncated"></a>Avertissement : L'entrée de la compétence a été tronquée
Les compétences cognitives comportent des limites quant à la longueur du texte qui peut être analysé en une seule fois. Si la saisie de texte pour ces compétences dépasse cette limite, nous tronquerons le texte pour respecter la limite, puis appliquerons un enrichissement à ce texte tronqué. Cela signifie que la compétence est exécutée, mais pas sur toutes vos données.

Dans l'exemple LanguageDetectionSkill ci-dessous, le champ de saisie `'text'` peut déclencher cet avertissement s'il dépasse la limite de caractères. Vous trouverez les limites de saisie des compétences dans la [documentation sur les compétences](cognitive-search-predefined-skills.md).

```json
 {
    "@odata.type": "#Microsoft.Skills.Text.LanguageDetectionSkill",
    "inputs": [
      {
        "name": "text",
        "source": "/document/text"
      }
    ],
    "outputs": [...]
  }
```

Si vous voulez vous assurer que tout le texte est analysé, pensez à utiliser la [compétence Split](cognitive-search-skill-textsplit.md).

<a name="web-api-skill-response-contains-warnings"></a>

## <a name="warning-web-api-skill-response-contains-warnings"></a>Avertissement : La réponse de compétence de l’API web contient des avertissements
Indexer a pu exécuter une compétence dans l’ensemble de compétences, mais la réponse à la demande de l’API web indique qu’il y a eu des avertissements durant l’exécution. Examinez les avertissements pour comprendre l’impact sur vos données et déterminer si une action est requise ou non.

<a name="the-current-indexer-configuration-does-not-support-incremental-progress"></a>

## <a name="warning-the-current-indexer-configuration-does-not-support-incremental-progress"></a>Avertissement : La configuration actuelle de l’indexeur ne prend pas en charge la progression incrémentielle

Cet avertissement s’affiche uniquement pour des sources de données Cosmos DB.

Dans le cas où l’exécution de l’indexeur est interrompue par des échecs passagers ou un dépassement du délai d’exécution, une progression incrémentielle pendant une indexation veille à ce que l’indexeur puisse reprendre là il en était lors de sa dernière exécution, afin de ne pas avoir à tout réindexer depuis le début. Ceci est particulièrement important lors de l’indexation de grandes collections.

La possibilité de reprendre un travail d’indexation inachevé dépend de la disponibilité de documents ordonnés par la colonne `_ts`. L’indexeur utilise l’horodatage pour déterminer le prochain document à récupérer. Si la colonne `_ts` est manquante ou si l’indexeur ne peut pas déterminer si une requête personnalisée est ordonnée par celle-ci, l’indexeur commence au début, cet avertissement s’affiche.

Il est possible de modifier ce comportement en activant la progression incrémentielle et en supprimant l’avertissement à l’aide de la propriété de configuration `assumeOrderByHighWatermarkColumn`.

Pour plus d’informations, consultez [Progression incrémentielle et requêtes personnalisées](search-howto-index-cosmosdb.md#IncrementalProgress).

<a name="some-data-was-lost-during projection-row-x-in-table-y-has-string-property-z-which-was-too-long"></a>

## <a name="warning-some-data-was-lost-during-projection-row-x-in-table-y-has-string-property-z-which-was-too-long"></a>Avertissement : Certaines données ont été perdues pendant la projection. La ligne« X » dans la table « Y » présente la propriété de chaîne « Z » qui était trop longue.

Le [service Stockage Table](https://azure.microsoft.com/services/storage/tables) impose des limites sur la taille des [propriétés d’entité](/rest/api/storageservices/understanding-the-table-service-data-model#property-types). Les chaînes peuvent comporter 32 000 caractères ou moins. Si une ligne présentant une propriété de chaîne de plus de 32 000 caractères est en cours de projection, seuls les 32 000 premiers caractères sont conservés. Pour contourner ce problème, évitez de projeter des lignes comportant des propriétés de chaîne de plus de 32 000 caractères.

<a name="truncated-extracted-text-to-x-characters"></a>

## <a name="warning-truncated-extracted-text-to-x-characters"></a>Avertissement : Texte extrait tronqué à X caractères
Les indexeurs limitent la quantité de texte qui peut être extraite d’un document. Cette limite dépend du niveau tarifaire : 32 000 caractères pour le niveau Gratuit, 64 000 pour le niveau De base, 4 millions pour le niveau Standard, 8 millions pour le niveau Standard S2 et 16 millions pour le niveau Standard S3. Le texte qui a été tronqué ne sera pas indexé. Pour éviter cet avertissement, essayez en scindant les documents avec de grandes quantités de texte en plusieurs documents plus petits. 

Pour plus d’informations, consultez [Limites des indexeurs](search-limits-quotas-capacity.md#indexer-limits).

<a name="could-not-map-output-field-x-to-search-index"></a>

## <a name="warning-could-not-map-output-field-x-to-search-index"></a>Avertissement : Impossible de mapper le champ de sortie « X » à l’index de recherche
Les mappages de champs de sortie qui font référence à des données inexistantes/null génèrent des avertissements pour chaque document et créent un champ d’index vide. Pour contourner ce problème, vérifiez que les chemins sources de mappage de champs de sortie sont corrects ou définissez une valeur par défaut à l’aide de la [compétence conditionnelle](cognitive-search-skill-conditional.md#sample-skill-definition-2-set-a-default-value-for-a-value-that-doesnt-exist). Pour plus d’informations, consultez [Mappage de champs de sortie](cognitive-search-output-field-mapping.md).

| Motif | Détails/Exemple | Résolution |
| --- | --- | --- |
| Impossible d’effectuer une itération sur le non-tableau | « Impossible d’effectuer une itération sur le non-tableau `/document/normalized_images/0/imageCelebrities/0/detail/celebrities` ». | Cette erreur se produit lorsque la sortie n’est pas un tableau. Si vous pensez que la sortie doit être un tableau, recherchez les erreurs dans le chemin d’accès du champ de source de sortie indiqué. Par exemple, vous pouvez avoir un `*` manquant ou supplémentaire dans le nom du champ source. Il est également possible que l’entrée de cette qualification soit Null, ce qui se traduit par un tableau vide. Trouvez des détails similaires dans la section [Entrée de compétence non valide](cognitive-search-common-errors-warnings.md#warning-skill-input-was-invalid).    |
| Impossible de sélectionner `0` dans le non-tableau | « Impossible de sélectionner `0` dans le non-tableau `/document/pages` ». | Cela peut se produire si la sortie des compétences ne produit pas de tableau et que le nom du champ de source de sortie a un index de tableau ou `*` dans son chemin d’accès. Vérifiez les chemins d’accès fournis dans les noms de champs de source de sortie et la valeur de champ pour le nom de champ indiqué. Trouvez des détails similaires dans la section [Entrée de compétence non valide](cognitive-search-common-errors-warnings.md#warning-skill-input-was-invalid).  |

<a name="the-data-change-detection-policy-is-configured-to-use-key-column-x"></a>

## <a name="warning-the-data-change-detection-policy-is-configured-to-use-key-column-x"></a>Avertissement : La stratégie de détection des modifications de données est configurée pour utiliser la colonne clé « X ».
Pour détecter les modifications, les [stratégies de détection des modifications de données](/rest/api/searchservice/create-data-source#data-change-detection-policies) ont des exigences spécifiques pour les colonnes qu’elles utilisent. L’une de ces exigences est que la colonne est mise à jour chaque fois que l’élément source est modifié. Une autre exigence est que la nouvelle valeur de cette colonne est supérieure à la valeur précédente. Les colonnes clés ne respectent pas cette exigence, car elles ne changent pas à chaque mise à jour. Pour contourner ce problème, sélectionnez une autre colonne pour la stratégie de détection des modifications.

<a name="document-text-appears-to-be-utf-16-encoded-but-is-missing-a-byte-order-mark"></a>

## <a name="warning-document-text-appears-to-be-utf-16-encoded-but-is-missing-a-byte-order-mark"></a>Avertissement : Le texte du document semble être encodé en UTF-16, mais il manque une marque d’ordre d’octet

Les [modes d’analyse de l’indexeur](/rest/api/searchservice/create-indexer#blob-configuration-parameters) doivent connaître la méthode d’encodage du texte avant de pouvoir l’analyser. Les deux méthodes les plus courantes pour encoder du texte sont UTF-16 et UTF-8. UTF-8 est un encodage de longueur variable où chaque caractère est compris entre 1 et 4 octets. UTF-16 est un encodage de longueur fixe où chaque caractère a une longueur de 2 octets. UTF-16 a deux variantes différentes, « avec primauté des octets de poids fort (big-endian) » et « mode Little Endian ». L’encodage de texte est déterminé par une « marque d’ordre d’octet », une série d’octets avant le texte.

| Encodage | Marque d’ordre d’octet |
| --- | --- |
| UTF-16 avec primauté des octets de poids fort (big-endian) | 0xFE 0xFF |
| UTF-16 mode Little Endian | 0xFF 0xFE |
| UTF-8 | 0xEF 0xBB 0xBF |

Si aucune marque d’ordre d’octet n’est présente, le texte est supposé être encodé au format UTF-8.

Pour contourner cet avertissement, déterminez l’encodage de texte pour ce blob et ajoutez la marque d’ordre d’octet appropriée.

<a name="cosmos-db-collection-has-a-lazy-indexing-policy"></a>

## <a name="warning-cosmos-db-collection-x-has-a-lazy-indexing-policy-some-data-may-be-lost"></a>Avertissement : La collection « X » de Cosmos DB a une stratégie d’indexation différée. Certaines données peuvent être perdues.

Les collections dotées de stratégies d’indexation [différée](../cosmos-db/index-policy.md#indexing-mode) ne peuvent pas être interrogées de manière cohérente, ce qui entraîne une absence de données dans votre indexeur. Pour contourner cet avertissement, modifiez votre stratégie d’indexation sur Cohérent.

## <a name="warning-the-document-contains-very-long-words-longer-than-64-characters-these-words-may-result-in-truncated-andor-unreliable-model-predictions"></a>Avertissement : Le document contient des mots très longs (de plus de 64 caractères). Ces mots peuvent donner lieu à des prédictions de modèle tronquées et/ou non fiables.

Cet avertissement provient du service Analyse de texte.  Dans certains cas, vous pouvez sans problème ignorer cet avertissement, par exemple quand votre document contient une URL longue (qui n’est sans doute pas une phrase clé, un sentiment de conduite, etc.).  Sachez que lorsqu’un mot dépasse 64 caractères, il est tronqué à 64 caractères, ce qui peut impacter les prédictions de modèle.
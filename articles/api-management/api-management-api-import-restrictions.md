---
title: Restrictions et détails de la prise en charge des formats d’API
titleSuffix: Azure API Management
description: Détails des problèmes connus et des restrictions relatifs à la prise en charge des formats Open API, WSDL ou WADL dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: vladvino
manager: vlvinogr
editor: ''
ms.assetid: 7a5a63f0-3e72-49d3-a28c-1bb23ab495e2
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 01/02/2020
ms.author: apimpm
ms.openlocfilehash: 6a53cc2b2ec6d46b4bde54af58b4e5542ff6cf79
ms.sourcegitcommit: 50802bffd56155f3b01bfb4ed009b70045131750
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91932632"
---
# <a name="api-import-restrictions-and-known-issues"></a>Restrictions et problèmes connus relatifs à l’importation d’API

## <a name="about-this-list"></a>À propos de cette liste

Quand vous importez une API, vous pouvez rencontrer certaines restrictions ou identifier des problèmes que vous devez résoudre au préalable. Cet article documente ces restrictions et problèmes connus, organisés par format d’importation de l’API. Il décrit également le fonctionnement de l’exportation OpenAPI.

## <a name="openapiswagger-import-limitations"></a><a name="open-api"> </a>Limitations relatives à l’importation OpenAPI/Swagger

Si vous recevez des erreurs durant l’importation de votre document OpenAPI, vérifiez que vous l’avez validé au préalable. Vous pouvez le faire avec le concepteur dans le portail Azure (Conception - Front End - Éditeur de spécification OpenAPI) ou à l’aide d’un outil tiers comme <a href="https://editor.swagger.io">Swagger Editor</a>.

### <a name="general"></a><a name="open-api-general"> </a>Généralités

-   Les paramètres requis pour le chemin d'accès et la requête doivent avoir des noms uniques. (Dans OpenAPI, un nom de paramètre doit être unique seulement dans un emplacement, par exemple, pour un chemin, une requête, un en-tête. Toutefois, dans Gestion des API nous autorisons la discrimination des opérations dans les paramètres de chemin et de requête (non pris en charge par OpenAPI). C’est pourquoi nous demandons à ce que les noms de paramètre soient uniques dans l’ensemble du modèle d’URL.)
-   Les pointeurs `\$ref` ne peuvent pas référencer des fichiers externes.
-   `x-ms-paths` et `x-servers` sont les seules extensions prises en charge.
-   Les extensions personnalisées sont ignorées à l’importation et ne sont pas enregistrées ni conservées pour l’exportation.
-   `Recursion` - Gestion des API ne prend pas en charge les définitions spécifiées de manière récursive (par exemple, les schémas qui se référencent eux-mêmes).
-   L’URL de fichier source (si disponible) est appliquée aux URL de serveur relatives.
-   Les définitions de sécurité sont ignorées.
-   Les définitions de schéma Inline pour les opérations d’API ne sont pas prises en charge. Les définitions de schéma sont définies dans l’étendue de l’API et peuvent être référencées dans les étendues de la réponse ou de la réponse des opérations de l’API.
-   Un paramètre d’URL défini doit faire partie du modèle d’URL.
-   L’objet `server` n’est pas pris en charge au niveau des opérations de l’API.
-   Le mot clé `Produces`, qui décrit les types MIME retournés par une API, n’est pas pris en charge. 

### <a name="openapi-version-2"></a><a name="open-api-v2"> </a>OpenAPI version 2

-   Seul le format JSON est pris en charge.

### <a name="openapi-version-3"></a><a name="open-api-v3"> </a>OpenAPI version 3

-   Si de nombreux `servers` sont spécifiés, Gestion des API tente de sélectionner la première URL HTTPs. En cas d’absence d’URL HTTPs, elle sélectionne la première URL HTTP. En cas d’absence d’URL HTTP, l’URL de serveur est vide.
-   `Examples` n’est pas pris en charge, contrairement à `example`.

## <a name="openapi-import-update-and-export-mechanisms"></a>Mécanismes d’importation, de mise à jour et d’exportation OpenAPI

### <a name="general"></a><a name="open-import-export-general"> </a>Général

-   Les définitions d’API exportées à partir du service Gestion des API sont principalement destinées aux applications externes à ce service et qui doivent appeler l’API hébergée dans le service Gestion des API. Les définitions d’API exportées ne sont pas destinées à être à nouveau importées dans le même service Gestion des API ou dans un service différent. Pour la gestion de la configuration des définitions d’API dans différents services/environnements, consultez la documentation relative à l’utilisation du service Gestion des API avec Git. 

### <a name="add-new-api-via-openapi-import"></a>Ajouter une nouvelle API via l’importation OpenAPI

Pour chaque opération trouvée dans le document OpenAPI, une nouvelle opération est créée avec le nom de ressource Azure et le nom d’affichage définis sur `operationId` et `summary` respectivement. La valeur `operationId` est normalisée en suivant les règles décrites ci-dessous. La valeur `summary` est importée telle quelle et sa longueur est limitée à 300 caractères.

Si `operationId` n’est pas spécifié (c’est-à-dire qu’il n’est pas présent, `null` ou vide), la valeur du nom de ressource Azure sera générée en combinant la méthode HTTP et le modèle de chemin d’accès, par exemple, `get-foo`.

Si `summary` n’est pas spécifié (c’est-à-dire qu’il n’est pas présent, `null` ou vide), la valeur `display name` est définie sur `operationId`. Si `operationId` n’est pas spécifié, la valeur du nom d’affichage sera générée en combinant la méthode HTTP et le modèle de chemin d’accès, par exemple, `Get - /foo`.

### <a name="update-an-existing-api-via-openapi-import"></a>Mettre à jour une API existante via l’importation OpenAPI

Pendant l’importation, l’API existante est modifiée pour correspondre à l’API décrite dans le document OpenAPI. Chaque opération dans le document OpenAPI est mise en correspondance avec une opération existante en comparant sa valeur `operationId` au nom de ressource Azure de l’opération existante.

Si une correspondance est trouvée, les propriétés de l’opération existante sont mises à jour « sur place ».

Si aucune correspondance n’est trouvée, une nouvelle opération est créée à l’aide des règles décrites dans la section ci-dessus. Pour chaque nouvelle opération, l’importation tente de copier les stratégies à partir d’une opération existante avec la même méthode HTTP et le même modèle de chemin d’accès.

Toutes les opérations sans correspondance existantes sont supprimées.

Pour que l’importation soit plus prévisible, suivez les instructions ci-dessous :

- Veillez à spécifier la propriété `operationId` pour chaque opération.
- Évitez de modifier `operationId` après l’importation initiale.
- Ne modifiez jamais `operationId` et la méthode HTTP ou le modèle de chemin d’accès en même temps.

### <a name="export-api-as-openapi"></a>Exporter l’API comme OpenAPI

Pour chaque opération, son nom de ressource Azure est exporté en tant que `operationId` et le nom d’affichage est exporté en tant que `summary`.
Règles de normalisation pour operationId

- Convertit les caractères en minuscules.
- Remplacez chaque séquence de caractères non alphanumériques par un tiret unique, par exemple, `GET-/foo/{bar}?buzz={quix}` sera transformé en `get-foo-bar-buzz-quix-`.
- Supprimez les tirets des deux côtés, par exemple, `get-foo-bar-buzz-quix-` devient `get-foo-bar-buzz-quix`
- Tronquez pour obtenir 76 caractères, quatre caractères de moins que la limite maximale pour un nom de ressource.
- Utilisez les quatre caractères restants pour un suffixe de déduplication, si nécessaire, sous la forme `-1, -2, ..., -999`.


## <a name="wsdl"></a><a name="wsdl"> </a>WSDL

Les fichiers WSDL permettent de créer des requêtes SOAP directes et des API SOAP à REST.

-   **Liaisons SOAP** : seules les liaisons SOAP avec encodage de style « document » et « littéral » sont prises en charge. Les encodages SOAP ou de style « rpc » ne sont pas pris en charge.
-   **WSDL:Import** : Cet attribut n’est pas pris en charge. Les clients doivent fusionner les importations dans un seul document.
-   **Messages en plusieurs parties** : Ces types de messages ne sont pas pris en charge.
-   **WCF wsHttpBinding** : Les services SOAP créés avec Windows Communication Foundation doivent utiliser basicHttpBinding. wsHttpBinding n’est pas pris en charge.
-   **MTOM** : les services utilisant MTOM <em>peuvent</em> fonctionner. Aucune prise en charge officielle n’est disponible pour l’instant.
-   **Récursivité** : les types qui sont définis de manière récursive (par exemple, qui font référence à leur propre tableau) ne sont pas pris en charge par APIM.
-   **Plusieurs espaces de noms** : plusieurs espaces de noms peuvent être utilisés dans un schéma, mais seul l’espace de noms cible peut être utilisé pour définir des parties de message. Les espaces de noms autres que la cible, qui permettent de définir d’autres éléments d’entrée ou de sortie, ne sont pas conservés. Même si un tel document WSDL peut être importé, lors de l’exportation, toutes les parties de message auront l’espace de noms cible du WSDL.
-   **Tableaux** : la transformation SOAP à REST prend uniquement en charge les tableaux encapsulés indiqués dans l’exemple ci-dessous :

```xml
    <complexType name="arrayTypeName">
        <sequence>
            <element name="arrayElementValue" type="arrayElementType" minOccurs="0" maxOccurs="unbounded"/>
        </sequence>
    </complexType>
    <complexType name="typeName">
        <sequence>
            <element name="element1" type="someTypeName" minOccurs="1" maxOccurs="1"/>
            <element name="element2" type="someOtherTypeName" minOccurs="0" maxOccurs="1" nillable="true"/>
            <element name="arrayElement" type="arrayTypeName" minOccurs="1" maxOccurs="1"/>
        </sequence>
    </complexType>
```

## <a name="wadl"></a><a name="wadl"> </a>WADL

Il n’existe aucun problème connu relatif à l’importation WADL.

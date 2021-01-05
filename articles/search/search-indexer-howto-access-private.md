---
title: Connexions d’indexeurs via un point de terminaison privé
titleSuffix: Azure Cognitive Search
description: Configurez les connexions de l’indexeur pour accéder au contenu à partir d’autres ressources Azure protégées par un point de terminaison privé.
manager: nitinme
author: arv100kri
ms.author: arjagann
ms.service: cognitive-search
ms.topic: conceptual
ms.date: 10/14/2020
ms.openlocfilehash: ff8aa6688d8a838fa2e06d2eef546025cdd9213f
ms.sourcegitcommit: f88074c00f13bcb52eaa5416c61adc1259826ce7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92340051"
---
# <a name="make-indexer-connections-through-a-private-endpoint"></a>Établir des connexions d’indexeurs via un point de terminaison privé

De nombreuses ressources Azure, telles que les comptes de stockage Azure, peuvent être configurées pour accepter les connexions à partir d’une liste de réseaux virtuels et refuser les connexions externes qui proviennent d’un réseau public. Si vous utilisez un indexeur pour indexer des données dans Recherche cognitive Azure et que votre source de données se trouve sur un réseau privé, vous pouvez créer une [connexion de point de terminaison privée](../private-link/private-endpoint-overview.md) sortante pour atteindre les données.

Cette méthode de connexion de l’indexeur est soumise aux deux conditions suivantes :

+ La ressource Azure qui fournit le contenu ou le code doit être inscrite au préalable auprès du [service de liaison privée Azure](https://azure.microsoft.com/services/private-link/).

+ Le service de Recherche cognitive Azure doit avoir le niveau de service De base ou supérieur. Cette fonctionnalité n’est pas disponible sur le niveau gratuit. En outre, si votre indexeur dispose d’un ensemble de compétences, le niveau doit être Standard 2 (S2) ou supérieur. Pour plus d’informations, consultez [Limites du Service](search-limits-quotas-capacity.md#shared-private-link-resource-limits).

## <a name="shared-private-link-resources-management-apis"></a>API de gestion des ressources de liaison privée partagées

Les points de terminaison privés des ressources privées créées par les API Recherche cognitive Azure sont appelés *ressources de liaison privée partagées* . En effet, vous « partagez » l’accès à une ressource, un compte de stockage par exemple, qui a été intégrée au [service Azure Private Link](https://azure.microsoft.com/services/private-link/).

Grâce à son API REST de gestion, Recherche cognitive Azure propose l’opération [CreateOrUpdate](/rest/api/searchmanagement/sharedprivatelinkresources/createorupdate) que vous pouvez utiliser pour configurer l’accès à partir d’un indexeur Recherche cognitive Azure.

Vous pouvez créer des connexions de point de terminaison privé à certaines ressources uniquement à l’aide de la préversion de l’API de gestion de Recherche (version *2020-08-01-Preview* ou ultérieure), qui est désignée *préversion* dans le tableau suivant. Les ressources ne portant pas la mention *préversion* peuvent être créées soit depuis la préversion, soit depuis la version de l’API mise à la disposition générale ( *2020-08-01* ou version ultérieure).

Le tableau suivant dresse la liste des ressources Azure pour lesquelles des points de terminaison privés sortants peuvent être créés à partir du service Recherche cognitive Azure. Pour créer une ressource de liaison privée partagée, entrez les valeurs d’ **ID de groupe** exactement telles qu’elles sont écrites dans l’API. Les valeurs respectent la casse.

| Ressource Azure | ID de groupe |
| --- | --- |
| Stockage Azure - Blob (ou) ADLS Gen 2 | `blob`|
| Stockage Azure - Tables | `table`|
| Azure Cosmos DB - API SQL | `Sql`|
| Azure SQL Database | `sqlServer`|
| Azure Database pour MySQL (préversion) | `mysqlServer`|
| Azure Key Vault | `vault` |
| Azure Functions (préversion) | `sites` |

Vous pouvez également envoyer une requête à propos des ressources Azure pour lesquelles les connexions par point de terminaison privé sortant sont prises en charge en utilisant [la liste des API prises en charge](/rest/api/searchmanagement/privatelinkresources/listsupported).

Dans le reste de cet article, une combinaison d’API [ARMClient](https://github.com/projectkudu/ARMClient) et [Postman](https://www.postman.com/) est utilisée pour illustrer les appels d’API REST.

> [!NOTE]
> Les exemples de cet article se basent sur les hypothèses suivantes :
> * le service de recherche s’appelle _contoso-search_ et se trouve dans le groupe de ressources _contoso_ d’un abonnement dont l’ID est _00000000-0000-0000-0000-000000000000_ . 
> * L’ID de ressource de ce service de recherche est _/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search_ .

Les autres exemples montrent comment le service _contoso-search_ peut être configuré de manière à ce que ses indexeurs puissent accéder aux données à partir du compte de stockage sécurisé _/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage_ .

## <a name="secure-your-storage-account"></a>Sécuriser votre compte de stockage

Configurez le compte de stockage afin de [n'autoriser l'accès qu'à partir de sous-réseaux spécifiques](../storage/common/storage-network-security.md#grant-access-from-a-virtual-network). Sur le portail Azure, si vous sélectionnez cette option sans renseigner le reste, cela signifie qu’aucun trafic provenant d’un réseau virtuel n’est autorisé.

   ![Capture d’écran du volet « Pare-feux et réseaux virtuels » mettant en avant l’option permettant d’autoriser l’accès aux réseaux sélectionnés. ](media\search-indexer-howto-secure-access\storage-firewall-noaccess.png)

> [!NOTE]
> Vous pouvez utiliser l’[approche Service Microsoft approuvé](../storage/common/storage-network-security.md#trusted-microsoft-services) pour contourner les restrictions de réseau virtuel ou d’adresse IP sur un compte de stockage. Vous pouvez également autoriser le service de recherche à accéder aux données dans le compte de stockage. Pour ce faire, consultez [Accès de l’indexeur au stockage Azure avec l’exception de service approuvé](search-indexer-howto-access-trusted-service-exception.md). 
>
> Toutefois, avec cette approche, la communication entre le service Recherche cognitive Azure et votre compte de stockage s’effectue via l’adresse IP publique du compte de stockage, sur le réseau principal sécurisé de Microsoft.

### <a name="step-1-create-a-shared-private-link-resource-to-the-storage-account"></a>Étape 1 : Créer une ressource de liaison privée partagée avec le compte de stockage

Pour demander au service Recherche cognitive Azure d’établir une connexion par point de terminaison privé sortant au compte de stockage, procédez à l’appel d’API suivant : 

`armclient PUT https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe?api-version=2020-08-01 create-pe.json`

Le contenu du fichier *create-pe.json* , qui représente le corps de la requête adressée à l’API, est le suivant :

```json
{
      "name": "blob-pe",
      "properties": {
        "privateLinkResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage",
        "groupId": "blob",
        "requestMessage": "please approve"
      }
}
```

Une réponse `202 Accepted` est retournée en cas de réussite. Le processus de création d’un point de terminaison privé sortant est une opération longue (asynchrone). Il implique le déploiement des ressources suivantes :

* un point de terminaison privé avec une adresse IP privée présentant l’état `"Pending"`. L’adresse IP privée est obtenue à partir de l’espace d’adressage alloué au réseau virtuel de l’environnement d’exécution de l’indexeur privé spécifique au service de recherche. Une fois le point de terminaison privé approuvé, toute communication entre le service Recherche cognitive Azure et le compte de stockage provient de l'adresse IP privée et d'un canal de liaison privé sécurisé.

* Une zone DNS privée pour le type de ressource, sur la base du `groupId`. En déployant cette ressource, vous êtes assuré que toute recherche DNS sur la ressource privée utilisera l’adresse IP associée au point de terminaison privé.

Veillez à spécifier le `groupId` qui convient au type de ressource pour lequel vous créez le point de terminaison privé. Toute incompatibilité générera un message d'échec.

Comme toutes les opérations Azure asynchrones, l’appel de `PUT` ​​renvoie une valeur d’en-tête `Azure-AsyncOperation` semblable à la suivante :

`"Azure-AsyncOperation": "https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe/operationStatuses/08586060559526078782?api-version=2020-08-01"`

Vous pouvez régulièrement interroger cet URI pour obtenir l’état de l’opération. Avant de continuer, nous vous recommandons d’attendre que l’état de l’opération de ressource de liaison privée partagée ait atteint un état terminal (autrement dit, l’opération est *réussie* ).

`armclient GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe/operationStatuses/08586060559526078782?api-version=2020-08-01"`

```json
{
    "status": "running" | "succeeded" | "failed"
}
```

### <a name="step-2a-approve-the-private-endpoint-connection-for-the-storage-account"></a>Étape 2a : Approuver la connexion par point de terminaison privé pour le compte de stockage

> [!NOTE]
> Dans cette section, vous utilisez le portail Azure pour suivre le processus d’approbation d’un point de terminaison privé vers le stockage. Vous pouvez également utiliser l’[API REST](/rest/api/storagerp/privateendpointconnections) disponible via le fournisseur de ressources de stockage.
>
> D’autres fournisseurs, tels qu’Azure Cosmos DB ou Azure SQL Server, offrent des API de fournisseur de ressources de stockage similaires pour la gestion des connexions de point de terminaison privé.

1. Dans le portail Azure, sélectionnez l’onglet **Connexions de point de terminaison privé** de votre compte de stockage. Une fois l’opération asynchrone réussie, une requête de connexion par point de terminaison privé doit y figurer, avec le message de demande d’appel d’API précédent.

   ![Capture d’écran du portail Azure, affichant le volet « Connexions de point de terminaison privé ».](media\search-indexer-howto-secure-access\storage-privateendpoint-approval.png)

1. Sélectionnez le point de terminaison privé créé par Recherche cognitive Azure. Dans la colonne **point de terminaison privé** , identifiez la connexion de point de terminaison privé par le nom spécifié dans l’API précédente, sélectionnez **Approuver** , puis entrez un message approprié. Le contenu du message n’a pas d’importance. 

   Assurez-vous que la connexion de points de terminaison privés s’affiche comme indiqué dans la capture d’écran suivante. La mise à jour de l’état dans le portail peut prendre une à deux minutes.

   ![Capture d’écran du portail Azure, affichant l’état « Approuvé » dans le volet « Connexions de point de terminaison privé ».](media\search-indexer-howto-secure-access\storage-privateendpoint-after-approval.png)

Dès l’approbation de la requête de connexion de points de terminaison privés, le trafic *peut* transiter par le point de terminaison privé. Une fois le point de terminaison privé approuvé, le service Recherche cognitive Azure crée les mappages de zone DNS nécessaires dans la zone DNS créée pour cela.

### <a name="step-2b-query-the-status-of-the-shared-private-link-resource"></a>Étape 2b : Interroger l'état de la ressource de liaison privée partagée

Pour vérifier que la ressource de liaison privée partagée a bien été mise à jour à l’issue de l’approbation, accédez à son état en utilisant l’[API GET](/rest/api/searchmanagement/sharedprivatelinkresources/get).

`armclient GET https://management.azure.com/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Search/searchServices/contoso-search/sharedPrivateLinkResources/blob-pe?api-version=2020-08-01`

Si la propriété `properties.provisioningState` de la ressource est `Succeeded` tandis que sa propriété `properties.status` est `Approved`, cela signifie que la ressource de liaison privée partagée est fonctionnelle et que l’indexeur peut être configuré pour communiquer via le point de terminaison privé.

```json
{
      "name": "blob-pe",
      "properties": {
        "privateLinkResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/contoso/providers/Microsoft.Storage/storageAccounts/contoso-storage",
        "groupId": "blob",
        "requestMessage": "please approve",
        "status": "Approved",
        "resourceRegion": null,
        "provisioningState": "Succeeded"
      }
}

```

### <a name="step-3-configure-the-indexer-to-run-in-the-private-environment"></a>Étape 3 : Configurer l’indexeur pour qu’il s’exécute dans l’environnement privé

> [!NOTE]
> Vous pouvez effectuer cette opération avant même que la connexion de points de terminaison privés ne soit approuvée. Tant que la connexion de points de terminaison privés n’est pas approuvée, tout indexeur qui tente de communiquer avec une ressource sécurisée (par exemple, le compte de stockage) est en état d’échec temporaire. Il est alors impossible de créer de nouveaux indexeurs. Dès que la connexion de points de terminaison privés est approuvée, les indexeurs ont accès au compte de stockage privé.

1. [Créez une source de données](/rest/api/searchservice/create-data-source) qui pointe vers le compte de stockage sécurisé et un conteneur approprié au sein du compte de stockage. La capture d’écran suivante montre cette requête dans Postman.

   ![Capture d’écran montrant la création d’une source de données sur l’interface utilisateur de Postman.](media\search-indexer-howto-secure-access\create-ds.png )

1. De la même manière, vous pouvez [créer un index](/rest/api/searchservice/create-index) et, si vous le souhaitez, [créer un ensemble de compétences](/rest/api/searchservice/create-skillset) à l’aide de l’API REST.

1. [Créez un indexeur](/rest/api/searchservice/create-indexer) qui pointe vers la source de données, l’index et l’ensemble de compétences que vous avez créés à l’étape précédente. En outre, forcez l’exécution de l’indexeur dans l’environnement d’exécution privé en définissant la propriété de configuration de l’indexeur `executionEnvironment` sur `private`.

   ![Capture d’écran montrant la création d’un indexeur sur l’interface utilisateur de Postman.](media\search-indexer-howto-secure-access\create-idr.png)

   Une fois l’indexeur créé avec succès, l’indexation du contenu du compte de stockage via la connexion de point de terminaison privé doit commencer. Vous pouvez surveiller l’état de l’indexeur à l’aide de l’[API d’état de l’indexeur](/rest/api/searchservice/get-indexer-status).

> [!NOTE]
> Si vous disposez déjà d’indexeurs, vous pouvez les mettre à jour via l’[API PUT](/rest/api/searchservice/create-indexer) en configurant `executionEnvironment` sur `private`.

## <a name="troubleshooting"></a>Dépannage

- Si la création de votre indexeur échoue avec un message d’erreur de type « Les informations d’identification de la source de données ne sont pas valides », cela signifie que la connexion de point de terminaison privé n’a pas été *approuvée* ou qu’elle n’est pas fonctionnelle. Pour résoudre ce problème : 
  * Utiliser l’[API GET](/rest/api/searchmanagement/sharedprivatelinkresources/get) pour accéder à l’état de la ressource de liaison privée partagée. Si l’état est *approuvé* , vérifiez la propriété `properties.provisioningState` de la ressource. Si l’état est `Incomplete`, cela signifie que certaines des dépendances sous-jacentes de la ressource n’ont pas pu être configurées. La réémission de la requête `PUT` pour recréer la ressource de liaison privée partagée doit résoudre le problème. Une réapprobation peut être nécessaire. Vérifiez à nouveau l’état de la ressource pour vous assurer que le problème est résolu.

- Si vous créez un indexeur sans que sa propriété `executionEnvironment` ne soit définie, la création peut être un succès, mais l’historique des exécutions de cet indexeur indiquera que celles-ci ont échoué. Pour résoudre ce problème :
   * [Mettre à jour l’indexeur](/rest/api/searchservice/update-indexer) pour spécifier l’environnement d’exécution.

- Si vous avez créé indexeur sans que sa propriété `executionEnvironment` ne soit définie et qu’il s’exécute correctement, cela signifie que le service Recherche cognitive Azure a décidé que son environnement d’exécution est l’environnement *privé* spécifique au service de recherche. Cela peut varier en fonction des ressources consommées par l’indexeur, de la charge sur le service de recherche et d’autres facteurs. L’échec peut survenir ultérieurement. Pour résoudre ce problème :
  * Nous vous recommandons vivement de définir la propriété `executionEnvironment` sur `private` pour vous assurer qu’elle n’échouera pas à l’avenir.

Les [quotas et limites](search-limits-quotas-capacity.md) déterminent le nombre de ressources de liaison privée partagée qui peuvent être créées et qui dépendent de la référence SKU du service de recherche.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les points de terminaison privés :

- [Présentation des points de terminaison privés](../private-link/private-endpoint-overview.md)
- [Configurations DNS nécessaires pour les points de terminaison privés](../private-link/private-endpoint-dns.md)

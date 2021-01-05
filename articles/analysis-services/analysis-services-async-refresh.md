---
title: Actualisation asynchrone pour les modèles Azure Analysis Services | Microsoft Docs
description: Décrit comment utiliser l’API REST Azure Analysis Services pour coder l’actualisation asynchrone des données du modèle.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: conceptual
ms.date: 04/15/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions
ms.openlocfilehash: e9fd20fd42e9fe1eb0e98766798e5c759c974c97
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92013897"
---
# <a name="asynchronous-refresh-with-the-rest-api"></a>Actualisation asynchrone avec l’API REST

En utilisant n’importe quel langage de programmation qui prend en charge les appels REST, vous pouvez effectuer des opérations d’actualisation des données asynchrones sur vos modèles tabulaires Azure Analysis Services. Cela inclut la synchronisation des réplicas en lecture seule pour la montée en puissance des requêtes. 

Les opérations d’actualisation des données peuvent prendre un certain temps en fonction de plusieurs facteurs, notamment le volume de données, le niveau d’optimisation à l’aide de partitions, etc. Ces opérations sont généralement appelées avec des méthodes existantes, telles que [TOM](/analysis-services/tom/introduction-to-the-tabular-object-model-tom-in-analysis-services-amo) (modèle d’objet tabulaire),les cmdlets [PowerShell](/analysis-services/powershell/analysis-services-powershell-reference) ou [TMSL](/analysis-services/tmsl/tabular-model-scripting-language-tmsl-reference) (langage de script de modèle tabulaire). Toutefois, ces méthodes requièrent souvent des connexions HTTP non fiables à longue durée d’exécution.

L’API REST pour Azure Analysis Services permet de réaliser de manière asynchrone des opérations d’actualisation de données. À l’aide de l’API REST, les connexions HTTP à longue durée d’exécution des applications clientes ne sont pas nécessaires. Il existe également d’autres fonctionnalités intégrées à des fins de fiabilité, telles que les tentatives automatiques et les validations par lot.

## <a name="base-url"></a>URL de base

L’URL de base suit ce format :

```
https://<rollout>.asazure.windows.net/servers/<serverName>/models/<resource>/
```

Prenons l’exemple d’un modèle nommé AdventureWorks sur un serveur nommé `myserver`, situé dans la région Azure USA Ouest. Le nom du serveur est :

```
asazure://westus.asazure.windows.net/myserver 
```

L’URL de base pour ce nom de serveur est la suivante :

```
https://westus.asazure.windows.net/servers/myserver/models/AdventureWorks/ 
```

Avec l’URL de base, il est possible d’ajouter des ressources et des opérations en fonction des paramètres suivants : 

![Actualisation asynchrone](./media/analysis-services-async-refresh/aas-async-refresh-flow.png)

- Tout ce qui se termine par **s** est une collection.
- Tout ce qui se termine par **()** est une fonction.
- Tout autre élément est un objet ou une ressource.

Par exemple, vous pouvez utiliser le verbe POST sur la collection Refreshes pour effectuer une opération d’actualisation :

```
https://westus.asazure.windows.net/servers/myserver/models/AdventureWorks/refreshes
```

## <a name="authentication"></a>Authentification

Tous les appels doivent être authentifiés avec un jeton d’Azure Active Directory (OAuth 2) valide dans l’en-tête d’autorisation et doivent respecter les conditions suivantes :

- Le jeton doit être un jeton d’utilisateur ou un principal de service d’application.
- L’audience appropriée pour le jeton doit avoir la valeur `https://*.asazure.windows.net`.
- L’utilisateur ou l’application doit avoir des autorisations suffisantes sur le serveur ou le modèle pour effectuer l’appel requis. Le niveau d’autorisation est déterminé par les rôles au sein du modèle ou du groupe d’administration sur le serveur.

    > [!IMPORTANT]
    > Actuellement, les autorisations du rôle **administrateur du serveur** sont nécessaires.

## <a name="post-refreshes"></a>POST /refreshes

Pour effectuer une opération d’actualisation, utilisez le verbe POST sur la collection /refreshes pour ajouter un nouvel élément d’actualisation à la collection. L’en-tête d’emplacement dans la réponse inclut l’ID de l’actualisation. L’application cliente peut se déconnecter et vérifier l’état ultérieurement si nécessaire, car l’opération est asynchrone.

Une seule opération d’actualisation est acceptée à la fois pour un modèle. Si une opération d’actualisation est en cours d’exécution et qu’une autre est soumise, le code d’état HTTP Conflit 409 est renvoyé.

Le corps doit ressembler à ceci :

```
{
    "Type": "Full",
    "CommitMode": "transactional",
    "MaxParallelism": 2,
    "RetryCount": 2,
    "Objects": [
        {
            "table": "DimCustomer",
            "partition": "DimCustomer"
        },
        {
            "table": "DimDate"
        }
    ]
}
```

### <a name="parameters"></a>Paramètres

La spécification des paramètres n’est pas nécessaire. La valeur par défaut est appliquée.

| Nom             | Type  | Description  |Default  |
|------------------|-------|--------------|---------|
| `Type`           | Enum  | Type de traitement à effectuer. Les types sont alignés sur les types de [commande d’actualisation](/analysis-services/tmsl/refresh-command-tmsl) TMSL : full, clearValues, calculate, dataOnly, automatic et defragment. Le type add n’est pas pris en charge.      |   automatique      |
| `CommitMode`     | Enum  | Détermine si les objets doivent être validés par lot ou uniquement à la fin. Modes inclus : default, transactional, partialBatch.  |  transactional       |
| `MaxParallelism` | Int   | Cette valeur détermine le nombre maximal de threads sur lesquels exécuter des commandes de traitement en parallèle. Elle s’aligne sur la propriété MaxParallelism qui peut être définie dans la [commande de séquence](/analysis-services/tmsl/sequence-command-tmsl) TMSL ou par d’autres méthodes.       | 10        |
| `RetryCount`     | Int   | Indique le nombre de tentatives de l’opération avant échec.      |     0    |
| `Objects`        | Array | Tableau d’objets à traiter. Chaque objet inclut « table » lors du traitement de la table entière, ou « table » et « partition » lors du traitement d’une partition. Si aucun objet n’est spécifié, l’ensemble du modèle est actualisé. |   Traitement de l’ensemble du modèle      |

CommitMode équivaut à partialBatch. Il est utilisé lors du chargement initial de jeux de données volumineux qui peuvent prendre des heures. Si l’opération d’actualisation échoue après la validation correcte d’un ou plusieurs lots, les lots validés restent validés (la validation n’est pas annulée).

> [!NOTE]
> Au moment de l’écriture, la taille de lot est la valeur MaxParallelism, mais cette valeur peut changer.

### <a name="status-values"></a>Valeurs d’état

|Valeur d’état  |Description  |
|---------|---------|
|`notStarted`    |   Opération pas encore commencée.      |
|`inProgress`     |   Opération en cours.      |
|`timedOut`     |    Délai de l’opération expiré conformément au délai d’expiration spécifié par l’utilisateur.     |
|`cancelled`     |   Opération annulée par l’utilisateur ou le système.      |
|`failed`     |   Échec de l’opération.      |
|`succeeded`      |   Opération réussie.      |

## <a name="get-refreshesrefreshid"></a>GET /refreshes/\<refreshId>

Pour vérifier l’état d’une opération d’actualisation, utilisez le verbe GET sur l’ID de l’actualisation. Voici un exemple de corps de réponse. Si l’opération est en cours, `inProgress` est retourné dans l’état.

```
{
    "startTime": "2017-12-07T02:06:57.1838734Z",
    "endTime": "2017-12-07T02:07:00.4929675Z",
    "type": "full",
    "status": "succeeded",
    "currentRefreshType": "full",
    "objects": [
        {
            "table": "DimCustomer",
            "partition": "DimCustomer",
            "status": "succeeded"
        },
        {
            "table": "DimDate",
            "partition": "DimDate",
            "status": "succeeded"
        }
    ]
}
```

## <a name="get-refreshes"></a>GET /refreshes

Pour obtenir la liste des opérations d’actualisation historiques pour un modèle, utilisez le verbe GET sur la collection /refreshes. Voici un exemple de corps de réponse. 

> [!NOTE]
> Au moment de l’écriture, les opérations d’actualisation des 30 derniers jours sont stockées et renvoyées, mais ce nombre peut être modifié.

```
[
    {
        "refreshId": "1344a272-7893-4afa-a4b3-3fb87222fdac",
        "startTime": "2017-12-07T02:06:57.1838734Z",
        "endTime": "2017-12-07T02:07:00.4929675Z",
        "status": "succeeded"
    },
    {
        "refreshId": "474fc5a0-3d69-4c5d-adb4-8a846fa5580b",
        "startTime": "2017-12-07T01:05:54.157324Z",
        "endTime": "2017-12-07T01:05:57.353371Z",
        "status": "succeeded"
    }
]
```

## <a name="delete-refreshesrefreshid"></a>DELETE /refreshes/\<refreshId>

Pour annuler une opération d’actualisation en cours, utilisez le verbe DELETE sur l’ID de l’actualisation.

## <a name="post-sync"></a>POST /sync

Après l’exécution d’opérations d’actualisation, il peut être nécessaire de synchroniser les nouvelles données avec des réplicas pour la montée en puissance des requêtes. Pour effectuer une opération de synchronisation pour un modèle, utilisez le verbe POST sur la fonction /sync. L’en-tête d’emplacement dans la réponse inclut l’ID de l’opération de synchronisation.

## <a name="get-sync-status"></a>GET /sync status

Pour vérifier l’état d’une opération de synchronisation, utilisez le verbe GET en transmettant l’ID de l’opération en tant que paramètre. Voici un exemple de corps de réponse :

```
{
    "operationId": "cd5e16c6-6d4e-4347-86a0-762bdf5b4875",
    "database": "AdventureWorks2",
    "UpdatedAt": "2017-12-09T02:44:26.18",
    "StartedAt": "2017-12-09T02:44:20.743",
    "syncstate": 2,
    "details": null
}
```

Valeurs possibles de `syncstate` :

- 0 : Réplication. Les fichiers de base de données sont en cours de réplication vers un dossier cible.
- 1 : Réalimentation. La base de données est en cours réalimentation sur une ou plusieurs instances de serveur en lecture seule.
- 2 : Terminé. L’opération de synchronisation est terminée.
- 3 : Échec. L’opération de synchronisation a échoué.
- 4 : Finalisation. L’opération de synchronisation est terminée, mais des étapes de nettoyage sont en cours.

## <a name="code-sample"></a>Exemple de code

Voici un exemple de code C# pour vous aider à démarrer, [RestApiSample sur GitHub](https://github.com/Microsoft/Analysis-Services/tree/master/RestApiSample).

### <a name="to-use-the-code-sample"></a>Pour utiliser l’exemple de code

1.    Clonez ou téléchargez le référentiel. Ouvrez la solution RestApiSample.
2.    Recherchez la ligne **client.BaseAddress = ...** et indiquez votre [URL de base](#base-url).

L’exemple de code utilise l’authentification d’un [principal de service](#service-principal).

### <a name="service-principal"></a>Principal du service

Consultez [Créer un principal du service – Portail Azure](../active-directory/develop/howto-create-service-principal-portal.md) et [Ajouter un principal du service au rôle d’administrateur du serveur](analysis-services-addservprinc-admins.md) pour plus d’informations sur la configuration du principal du service et l’attribution des autorisations nécessaires dans Azure AS. Après cette procédure, suivez les étapes complémentaires suivantes :

1.    Dans l’exemple de code, recherchez **string authority = …** , puis remplacez **common** par l’ID de locataire de votre organisation.
2.    Ajoutez ou enlevez des marques de commentaire pour que la classe ClientCredential soit utilisée pour instancier l’objet d’informations d’identification. Vérifiez que les valeurs \<App ID> et \<App Key> sont accessibles de manière sécurisée, ou utilisez l’authentification par certificat pour les principaux de service.
3.    Exécutez l’exemple.


## <a name="see-also"></a>Voir aussi

[Exemples](analysis-services-samples.md)   
[REST API](/rest/api/analysisservices/servers)
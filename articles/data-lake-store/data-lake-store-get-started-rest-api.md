---
title: Gérer un compte Azure Data Lake Storage Gen1 avec REST
description: Utilisez l’API REST WebHDFS pour effectuer des opérations de gestion de compte sur un compte Azure Data Lake Storage Gen1.
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 13467a51b2a06dbc0ca0ec5eadd139fde8b82ad0
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92103490"
---
# <a name="account-management-operations-on-azure-data-lake-storage-gen1-using-rest-api"></a>Opérations de gestion des comptes sur Azure Data Lake Storage Gen1 à l’aide de l’API REST
> [!div class="op_single_selector"]
> * [Kit de développement logiciel (SDK) .NET](data-lake-store-get-started-net-sdk.md)
> * [REST API](data-lake-store-get-started-rest-api.md)
> * [Python](data-lake-store-get-started-python.md)
>
>

Dans cet article, vous allez découvrir comment réaliser des opérations de gestion des comptes sur Azure Data Lake Storage Gen1 avec l’API REST. Ces opérations incluent la création ou la suppression d’un compte Data Lake Storage Gen1, etc. Pour obtenir des instructions sur l’exécution des opérations du système de fichiers sur Data Lake Storage Gen1 à l’aide de l’API REST, consultez la section relative aux [opérations du système de fichiers sur Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-data-operations-rest-api.md).

## <a name="prerequisites"></a>Prérequis
* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **[cURL](https://curl.haxx.se/)** . Cet article utilise cURL pour montrer comment effectuer des appels d’API REST sur un compte Data Lake Storage Gen1.

## <a name="how-do-i-authenticate-using-azure-active-directory"></a>Comment s’authentifier à l’aide d’Azure Active Directory ?
Vous avez le choix entre deux méthodes pour vous authentifier à l’aide d’Azure Active Directory :

* Pour en savoir plus sur l’authentification des utilisateurs accédant à votre application (interactive), consultez la section relative à l’[authentification de l’utilisateur avec Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-rest-api.md).
* Pour en savoir plus sur l’authentification de service à service pour votre application (non interactive), consultez la section relative à l’[authentification de service à service avec Data Lake Storage Gen1 à l’aide du kit de développement logiciel (SDK) .NET](data-lake-store-service-to-service-authenticate-rest-api.md).


## <a name="create-a-data-lake-storage-gen1-account"></a>Créer un compte Data Lake Storage Gen1
Cette opération est basée sur l’appel d’API REST défini [ici](/rest/api/datalakestore/accounts/create).

Utilisez la commande cURL suivante. Remplacez **\<yourstoragegen1name>** par le nom de votre Data Lake Storage Gen1.

```console
curl -i -X PUT -H "Authorization: Bearer <REDACTED>" -H "Content-Type: application/json" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/<yourstoragegen1name>?api-version=2015-10-01-preview -d@"C:\temp\input.json"
```

Dans la commande ci-dessus, remplacez \<`REDACTED`\> par le jeton d’autorisation que vous avez récupéré précédemment. La charge utile de la demande pour cette commande est contenue dans le fichier **input.json** fourni pour le paramètre `-d` ci-dessus. Le contenu du fichier input.json ressemble à l’extrait de code suivant :

```json
{
"location": "eastus2",
"tags": {
    "department": "finance"
    },
"properties": {}
}
```

## <a name="delete-a-data-lake-storage-gen1-account"></a>Supprimer un compte Data Lake Storage Gen1
Cette opération est basée sur l’appel d’API REST défini [ici](/rest/api/datalakestore/accounts/delete).

Utilisez la commande cURL suivante pour supprimer un compte Data Lake Storage Gen1. Remplacez **\<yourstoragegen1name>** par votre nom de compte Data Lake Storage Gen1.

```console
curl -i -X DELETE -H "Authorization: Bearer <REDACTED>" https://management.azure.com/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}/providers/Microsoft.DataLakeStore/accounts/<yourstoragegen1name>?api-version=2015-10-01-preview
```

Un résultat similaire à l’extrait de code suivant doit s’afficher :

```output
HTTP/1.1 200 OK
...
...
```

## <a name="next-steps"></a>Étapes suivantes
* [Opérations de système de fichiers dans Data Lake Storage Gen1 à l’aide de l’API REST](data-lake-store-data-operations-rest-api.md).

## <a name="see-also"></a>Voir aussi
* [Référence relative à l’API REST d’Azure Data Lake Storage Gen1](/rest/api/datalakestore/)
* [Ouvrir des applications Big Data open source compatibles avec Azure Data Lake Storage Gen1](data-lake-store-compatible-oss-other-applications.md)
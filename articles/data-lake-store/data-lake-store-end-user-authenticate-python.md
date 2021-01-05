---
title: Authentification des utilisateurs finaux – Python avec Data Lake Storage Gen1 – Azure
description: Découvrez comment authentifier les utilisateurs finaux auprès de Data Lake Storage Gen1 à l’aide d’Azure Active Directory et de Python
author: twooley
ms.service: data-lake-store
ms.topic: how-to
ms.date: 05/29/2018
ms.author: twooley
ms.custom: has-adal-ref, devx-track-python
ms.openlocfilehash: c7ee58dec4e8bc04eb07695378e0d4dc973b6705
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92109083"
---
# <a name="end-user-authentication-with-azure-data-lake-storage-gen1-using-python"></a>Authentification des utilisateurs finaux auprès d’Azure Data Lake Storage Gen1 à l’aide de Python
> [!div class="op_single_selector"]
> * [À l’aide de Java](data-lake-store-end-user-authenticate-java-sdk.md)
> * [Utilisation du kit de développement logiciel (SDK) .NET](data-lake-store-end-user-authenticate-net-sdk.md)
> * [Utilisation de Python](data-lake-store-end-user-authenticate-python.md)
> * [Utilisation de l'API REST](data-lake-store-end-user-authenticate-rest-api.md)
>
>

Dans cet article, vous allez apprendre à utiliser le Kit de développement logiciel (SDK) Python pour authentifier les utilisateurs finaux auprès d’Azure Data Lake Storage Gen1. L’authentification des utilisateurs finals se subdivise en deux catégories :

* Authentification des utilisateurs finals sans authentification multifacteur
* Authentification des utilisateurs finals avec authentification multifacteur

Les deux options sont décrites dans cet article. Pour plus d’informations sur l’authentification de service à service auprès de Data Lake Storage Gen1 à l’aide de Python, consultez [Authentification de service à service auprès de Data Lake Storage Gen1 à l’aide de Python](data-lake-store-service-to-service-authenticate-python.md).

## <a name="prerequisites"></a>Prérequis

* **Python** Pour télécharger Python, accédez [ici](https://www.python.org/downloads/). Cet article utilise Python 3.6.2.

* **Un abonnement Azure**. Consultez la page [Obtention d’un essai gratuit d’Azure](https://azure.microsoft.com/pricing/free-trial/).

* **Créez une application « native » Azure Active Directory**. Vous devez avoir suivi la procédure dans [End-user authentication with Data Lake Storage Gen1 using Azure Active Directory](data-lake-store-end-user-authenticate-using-active-directory.md) (Authentification des utilisateurs finaux avec Data Lake Storage Gen1 à l’aide d’Azure Active Directory).

## <a name="install-the-modules"></a>Installer les modules

Pour utiliser Data Lake Storage Gen1 avec Python, vous devez installer trois modules.

* Le module `azure-mgmt-resource`, qui inclut des modules Azure pour Active Directory, etc.
* Le module `azure-mgmt-datalake-store`, qui inclut les opérations de gestion du compte Azure Data Lake Storage Gen1. Pour plus d’informations sur ce module, consultez les [informations de référence sur les modules de gestion Azure Data Lake Storage Gen1](/python/api/azure-mgmt-datalake-store/).
* Le module `azure-datalake-store`, qui inclut les opérations de gestion du système de fichiers Azure Data Lake Storage Gen1. Pour plus d’informations sur ce module, consultez les [informations de référence sur les modules du système de fichiers azure-datalake-store](/python/api/azure-datalake-store/azure.datalake.store.core/).

Utilisez les commandes suivantes pour installer les modules.

```console
pip install azure-mgmt-resource
pip install azure-mgmt-datalake-store
pip install azure-datalake-store
```

## <a name="create-a-new-python-application"></a>Créer une application Python

1. Utilisez l’IDE de votre choix et créez une application Python, par exemple, **mysample.py**.

2. Ajoutez l’extrait de code suivant pour importer les modules requis

    ```
    ## Use this for Azure AD authentication
    from msrestazure.azure_active_directory import AADTokenCredentials

    ## Required for Azure Data Lake Storage Gen1 account management
    from azure.mgmt.datalake.store import DataLakeStoreAccountManagementClient
    from azure.mgmt.datalake.store.models import DataLakeStoreAccount

    ## Required for Azure Data Lake Storage Gen1 filesystem management
    from azure.datalake.store import core, lib, multithread

    # Common Azure imports
    import adal
    from azure.mgmt.resource.resources import ResourceManagementClient
    from azure.mgmt.resource.resources.models import ResourceGroup

    ## Use these as needed for your application
    import logging, pprint, uuid, time
    ```

3. Enregistrez les modifications apportées à mysample.py.

## <a name="end-user-authentication-with-multi-factor-authentication"></a>Authentification des utilisateurs finals avec authentification multifacteur

### <a name="for-account-management"></a>Pour la gestion du compte

Utilisez l’extrait de code suivant pour l’authentification auprès d’Azure AD dans le cas d’opérations de gestion d’un compte Data Lake Storage Gen1. L’extrait de code suivant peut être utilisé pour authentifier votre application à l’aide de l’authentification multifacteur. Indiquez les valeurs ci-dessous pour une application **native** Azure AD déjà présente.

```python
authority_host_url = "https://login.microsoftonline.com"
tenant = "FILL-IN-HERE"
authority_url = authority_host_url + '/' + tenant
client_id = 'FILL-IN-HERE'
redirect = 'urn:ietf:wg:oauth:2.0:oob'
RESOURCE = 'https://management.core.windows.net/'

context = adal.AuthenticationContext(authority_url)
code = context.acquire_user_code(RESOURCE, client_id)
print(code['message'])
mgmt_token = context.acquire_token_with_device_code(RESOURCE, code, client_id)
armCreds = AADTokenCredentials(mgmt_token, client_id, resource = RESOURCE)
```

### <a name="for-filesystem-operations"></a>Pour les opérations du système de fichiers

Utilisez cet extrait de code pour l’authentification auprès d’Azure AD dans le cas d’opérations de gestion du système de fichiers sur un compte Data Lake Storage Gen1. L’extrait de code suivant peut être utilisé pour authentifier votre application à l’aide de l’authentification multifacteur. Indiquez les valeurs ci-dessous pour une application **native** Azure AD déjà présente.

```console
adlCreds = lib.auth(tenant_id='FILL-IN-HERE', resource = 'https://datalake.azure.net/')
```

## <a name="end-user-authentication-without-multi-factor-authentication"></a>Authentification des utilisateurs finals sans authentification multifacteur

Cette option est déconseillée. Pour plus d’informations, consultez la page [Authentification Azure à l’aide du SDK Python](/azure/python/python-sdk-azure-authenticate).

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez appris à vous servir de l’authentification des utilisateurs finaux auprès d’Azure Data Lake Storage Gen1 avec Python. Vous pouvez maintenant consulter les articles suivants, qui expliquent comment utiliser Python pour travailler avec Azure Data Lake Storage Gen1.

* [Opérations de gestion du compte sur Data Lake Storage Gen1 à l’aide de Python](data-lake-store-get-started-python.md)
* [Opérations sur les données dans Data Lake Storage Gen1 à l’aide de Python](data-lake-store-data-operations-python.md)
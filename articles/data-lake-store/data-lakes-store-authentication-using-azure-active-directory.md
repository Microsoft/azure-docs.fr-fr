---
title: Authentification – Data Lake Storage Gen1 avec Azure AD
description: Découvrez comment s’authentifier auprès d’Azure Data Lake Storage Gen1 à l’aide d’Azure Active Directory.
author: twooley
ms.service: data-lake-store
ms.topic: conceptual
ms.date: 05/29/2018
ms.author: twooley
ms.openlocfilehash: 6fc09f9145b7a1652b621ed38a8bf9af7c4c82a8
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92106567"
---
# <a name="authentication-with-azure-data-lake-storage-gen1-using-azure-active-directory"></a>Authentification auprès d’Azure Data Lake Storage Gen1 à l’aide d’Azure Active Directory

Azure Data Lake Storage Gen1 utilise Azure Active Directory pour l’authentification. Avant de créer une application qui fonctionne avec Data Lake Storage Gen1, vous devez déterminer la manière dont vous voulez authentifier votre application auprès d’Azure Active Directory (Azure AD).

## <a name="authentication-options"></a>Options d’authentification

* **Authentification de l’utilisateur final** : les informations d’identification Azure d’un utilisateur final sont utilisées pour l’authentification auprès de Data Lake Storage Gen1. L’application que vous créez pour fonctionner avec Data Lake Storage Gen1 vous invite à entrer ces informations d’identification utilisateur. Ainsi, ce mécanisme d’authentification est *interactif* et l’application s’exécute dans le contexte de l’utilisateur connecté. Pour plus d’informations et pour obtenir des instructions, consultez [Authentification de l’utilisateur final pour Data Lake Storage Gen1](data-lake-store-end-user-authenticate-using-active-directory.md).

* **Authentification de service à service** : utilisez cette option si vous souhaitez qu’une application s’authentifie auprès de Data Lake Storage Gen1. Dans ce cas, vous créez une application Azure Active Directory (AD) et vous utilisez la clé de l’application Azure AD pour vous authentifier auprès de Data Lake Storage Gen1. Ce mécanisme d’authentification est *non interactif*. Pour plus d’informations et pour obtenir des instructions, consultez [Authentification de service à service pour Data Lake Storage Gen1](data-lake-store-service-to-service-authenticate-using-active-directory.md).

Le tableau suivant illustre comment les mécanismes d’authentification de l’utilisateur final et de service à service sont pris en charge pour Data Lake Storage Gen1. Voici comment lire le tableau.

* Le symbole ✔* indique que l’option d’authentification est prise en charge et fournit un lien vers un article qui montre comment utiliser l’option d’authentification. 
* Le symbole ✔indique que l’option d’authentification est prise en charge. 
* Les cellules vides indiquent que l’option d’authentification n’est pas prise en charge.


|Utilisez cette option d’authentification avec...                   |.NET         |Java     |PowerShell |Azure CLI | Python   |REST     |
|:---------------------------------------------|:------------|:--------|:----------|:-------------|:---------|:--------|
|Utilisateur final (sans MFA\*\*)                        |   ✔ |    ✔    |    ✔      |       ✔      |    **[✔*](data-lake-store-end-user-authenticate-python.md#end-user-authentication-without-multi-factor-authentication)** (déprécié)     |    **[✔*](data-lake-store-end-user-authenticate-rest-api.md)**    |
|Utilisateur final (avec MFA)                           |    **[✔*](data-lake-store-end-user-authenticate-net-sdk.md)**        |    **[✔*](data-lake-store-end-user-authenticate-java-sdk.md)**     |    ✔      |       **[✔*](data-lake-store-get-started-cli-2.0.md)**      |    **[✔*](data-lake-store-end-user-authenticate-python.md#end-user-authentication-with-multi-factor-authentication)**     |    ✔    |
|Service à service (à l’aide de la clé de client)         |    **[✔*](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-client-secret)** |    **[✔*](data-lake-store-service-to-service-authenticate-java.md)**    |    ✔      |       ✔      |    **[✔*](data-lake-store-service-to-service-authenticate-python.md#service-to-service-authentication-with-client-secret-for-account-management)**     |    **[✔*](data-lake-store-service-to-service-authenticate-rest-api.md)**    |
|Service à service (à l’aide du certificat client) |    **[✔*](data-lake-store-service-to-service-authenticate-net-sdk.md#service-to-service-authentication-with-certificate)**        |    ✔    |    ✔      |       ✔      |    ✔     |    ✔    |

<i>* Cliquez sur le symbole <b>✔\*</b>. Il s’agit d’un lien.</i><br>
<i>** MFA signifie Multi-Factor Authentication (authentification multifacteur)</i>

Pour plus d’informations sur la façon d’utiliser Azure Active Directory pour l’authentification, consultez [Scénarios d’authentification Azure Active Directory](../active-directory/develop/authentication-vs-authorization.md).

## <a name="next-steps"></a>Étapes suivantes

* [Authentification de l’utilisateur final](data-lake-store-end-user-authenticate-using-active-directory.md)
* [Authentification de service à service](data-lake-store-service-to-service-authenticate-using-active-directory.md)
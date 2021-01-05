---
title: Créer une Function App à l’aide du Portail Azure
description: Créez une application de fonction dans Azure à l’aide du portail.
ms.topic: how-to
ms.date: 08/29/2019
ms.custom: mvc
ms.openlocfilehash: 001b4e4f0ea7fbacd232b2a87abfe353f34919bd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80985012"
---
# <a name="create-a-function-app-from-the-azure-portal"></a>Créer une Function App à l’aide du Portail Azure

Cette rubrique vous montre comment utiliser Azure Functions pour créer une application de fonction dans le portail Azure. Une Function App est un conteneur qui héberge l’exécution de fonctions individuelles. 

## <a name="create-a-function-app"></a>Créer une application de fonction

[!INCLUDE [functions-create-function-app-portal](../../includes/functions-create-function-app-portal.md)]

Une fois la Function App créée, vous pouvez créer des fonctions individuelles à l’aide d’un ou plusieurs langages [en utilisant le portail](functions-create-first-azure-function.md#create-function), le [déploiement continu](functions-continuous-deployment.md) ou le [téléchargement par FTP](https://github.com/projectkudu/kudu/wiki/Accessing-files-via-ftp).

## <a name="service-plans"></a>Plans de service

Azure Functions a trois plans de service différents : le plan Consommation, le plan Premium et le plan Dedicated (App Service). Vous devez choisir votre plan de service lors de la création de votre application de fonction. Il ne peut pas être changé par la suite. Pour plus d’informations, consultez [Choisir un plan d’hébergement Azure Functions](functions-scale.md).

Si vous prévoyez d’exécuter des fonctions JavaScript sur un plan Dedicated (App Service), vous devez choisir un plan avec moins de cœurs. Pour plus d’informations, consultez les [informations de référence sur JavaScript pour Functions](functions-reference-node.md#choose-single-vcpu-app-service-plans).

<a name="storage-account-requirements"></a>

## <a name="storage-account-requirements"></a>Conditions requises pour le compte de stockage

Quand vous créez une application de fonction, vous devez créer un compte de stockage Azure à usage général qui prend en charge le stockage Blob, File d’attente et Table, ou établir un lien vers un compte de ce type. En interne, Azure Functions utilise le stockage pour les opérations telles que la gestion des déclencheurs et la journalisation des exécutions de fonctions. Certains comptes de stockage ne prennent pas en charge les files d’attente et les tables, comme les comptes de stockage Blob uniquement, le stockage Premium Azure et les comptes de stockage à usage général avec la réplication ZRS. Ces comptes sont filtrés à partir du panneau du compte de stockage lors de la création d’une Function App.

>[!NOTE]
>Lorsque vous utilisez le plan d’hébergement de la consommation, les fichiers de code de fonction et de configuration de liaison sont stockés sur le Stockage Fichier Azure dans le compte de stockage principal. Lorsque vous supprimez le compte de stockage principal, ce contenu est supprimé et ne peut pas être récupéré.

Pour en savoir plus sur les types de compte de stockage, consultez [Présentation des services Stockage Azure](../storage/common/storage-introduction.md#core-storage-services). 

## <a name="next-steps"></a>Étapes suivantes

Si le portail Azure permet de créer et essayer facilement Functions, nous vous recommandons le [développement local](functions-develop-local.md). Après avoir créé une Function App dans le portail, vous devez toujours ajouter une fonction. 

> [!div class="nextstepaction"]
> [Ajouter une fonction déclenchée via HTTP](functions-create-first-azure-function.md#create-function)

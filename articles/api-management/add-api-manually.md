---
title: Ajouter une API manuellement à l’aide du portail Azure | Microsoft Docs
description: Ce didacticiel vous montre comment utiliser le service Gestion des API (APIM) pour ajouter une API manuellement.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cfowler
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: tutorial
ms.date: 04/20/2020
ms.author: apimpm
ms.openlocfilehash: 39a3b9d7dd9efbda93de0b5d7c5f9938922d0012
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183820"
---
# <a name="add-an-api-manually"></a>Ajouter une API manuellement

Les étapes décrites dans cet article montrent comment utiliser le portail Azure pour ajouter une API manuellement à l’instance de Gestion des API (APIM). Un scénario courant lorsque vous souhaitez créer une API vide et la définir manuellement est lorsque vous souhaitez simuler l’API. Pour plus d’informations sur la simulation d’une API, consultez [Mock API responses](mock-api-responses.md) (Simuler des réponses d’API).

Si vous souhaitez importer une API existante, consultez la section [Rubriques connexes](#related-topics).

Dans cet article, nous créons une API vide et spécifions [httpbin.org](https://httpbin.org) (un service de test public) en tant qu’API de serveur principal.

## <a name="prerequisites"></a>Prérequis

Suivez ce guide de démarrage rapide : [Créer une instance du service Gestion des API Azure](get-started-create-service-instance.md)

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-navigate-to-instance.md)]

## <a name="create-an-api"></a>Création d'une API

1. À partir du portail Azure, accédez à votre service Gestion des API, puis sélectionnez **API** dans le menu.
2. Dans le menu de gauche, cliquez sur **Ajouter l’API**.
3. Sélectionnez **API vide** dans la liste.  
    ![API vide](media/add-api-manually/blank-api.png)  
4. Entrez des paramètres pour l’API. Les paramètres sont expliqués dans le tutoriel [Importer et publier votre première API](import-and-publish.md#import-and-publish-a-backend-api).
5. Sélectionnez **Create** (Créer).

À ce stade, vous ne disposez d’aucune opération dans Gestion des API qui correspond aux opérations dans votre API de serveur principal. Si vous appelez une opération qui est exposée via le serveur principal, mais pas par l’intermédiaire de Gestion des API, vous obtenez une erreur **404**.

>[!NOTE] 
> Par défaut, lorsque vous ajoutez une API, même si elle est connectée à un service de serveur principal, APIM n’exposera pas d’opérations tant que vous ne les aurez pas autorisées. Pour autoriser une opération de votre service de serveur principal, créez une opération APIM qui correspond à l’opération de serveur principal.

## <a name="add-and-test-an-operation"></a>Ajouter et tester une opération

Cette section montre comment ajouter une opération « /get » afin de la mapper sur l’opération « http://httpbin.org/get  » du serveur backend.

### <a name="add-an-operation"></a>Ajout d’une opération

1. Sélectionnez l’API que vous avez créée à l’étape précédente.
2. Cliquez sur **+ Ajouter une opération**.
3. Dans **URL**, sélectionnez **GET** et entrez « */get* » dans la ressource.
4. Entrez « *FetchData* » pour **Nom complet**.
5. Sélectionnez **Enregistrer**.

### <a name="test-an-operation"></a>Tester une opération

Testez l’opération dans le portail Azure. Vous pouvez également la tester dans le **portail des développeurs**.

1. Sélectionnez l’onglet **Test**.
2. Sélectionnez **FetchData**.
3. Appuyez sur **Envoyer**.

La réponse que l’opération « http://httpbin.org/get  » génère s’affiche. Si vous souhaitez transformer vos opérations, consultez [Transform and protect your API](transform-api.md) (Transformer et protéger votre API).

## <a name="add-and-test-a-parameterized-operation"></a>Ajouter et tester une opération paramétrable

Cette section montre comment ajouter une opération qui accepte un paramètre. Dans ce cas, nous mappons l’opération sur « http://httpbin.org/status/200  ».

### <a name="add-the-operation"></a>Ajouter l’opération

1. Sélectionnez l’API que vous avez créée à l’étape précédente.
2. Cliquez sur **+ Ajouter une opération**.
3. Dans **URL**, sélectionnez **GET** et entrez « */status/{code}* » dans la ressource. Si vous le souhaitez, vous pouvez fournir des informations associées à ce paramètre. Par exemple, entrez « *Nombre* » pour **TYPE**, « *200* » (par défaut) pour **VALEURS**.
4. Entrez « GetStatus » pour **Nom complet**.
5. Sélectionnez **Enregistrer**.

### <a name="test-the-operation"></a>Tester l’opération 

Testez l’opération dans le portail Azure.  Vous pouvez également la tester dans le **portail des développeurs**.

1. Sélectionnez l’onglet **Test**.
2. Sélectionnez **GetStatus**. Par défaut, la valeur de code est définie sur « *200* ». Vous pouvez la modifier pour tester d’autres valeurs. Par exemple, tapez « *418* ».
3. Appuyez sur **Envoyer**.

    La réponse que l’opération « http://httpbin.org/status/200  » génère s’affiche. Si vous souhaitez transformer vos opérations, consultez [Transform and protect your API](transform-api.md) (Transformer et protéger votre API).

[!INCLUDE [api-management-navigate-to-instance.md](../../includes/api-management-append-apis.md)]

[!INCLUDE [api-management-define-api-topics.md](../../includes/api-management-define-api-topics.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)

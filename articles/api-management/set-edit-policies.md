---
title: Comment définir ou modifier des stratégies dans Gestion des API Azure | Microsoft Docs
description: Découvrez comment définir ou modifier des stratégies Gestion des API Azure. Ces stratégies sont des documents XML qui décrivent une séquence d’instructions entrantes et sortantes.
services: api-management
documentationcenter: ''
author: mikebudzynski
manager: cflower
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 11/01/2018
ms.author: apimpm
ms.openlocfilehash: a737fada3e019029967bc752aaa0dedc354fa880
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92078321"
---
# <a name="how-to-set-or-edit-azure-api-management-policies"></a>Comment définir ou modifier des stratégies dans Gestion des API Azure

La définition de la stratégie est un document XML qui décrit une séquence d’instructions entrantes et sortantes. Le code XML peut être modifié directement dans la fenêtre de définition. Vous pouvez également sélectionner une stratégie prédéfinie dans la liste fournie à droite de la fenêtre de la stratégie. Les instructions applicables à l’étendue actuelle sont activées et mises en surbrillance. Lorsque vous cliquez sur une instruction active, le code XML correspondant est inséré à l’emplacement du curseur dans la fenêtre de définition. 

Pour des informations détaillées sur les stratégies, consultez la page [Stratégies dans Gestion des API Azure](api-management-howto-policies.md).

## <a name="set-or-edit-a-policy"></a>Définir ou modifier une stratégie

Pour définir ou modifier une stratégie, procédez comme suit :

1. Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com).
2. Accédez à votre instance APIM.
3. Cliquez sur l’onglet **API**.

    ![Modifier la stratégie](./media/set-edit-policies/code-editor.png)

4. Sélectionnez l’une des API que vous avez importées précédemment.
5. Sélectionnez l’onglet **Conception**.
6. Sélectionnez une opération à laquelle vous souhaitez appliquer la stratégie. Si vous souhaitez appliquer la stratégie à toutes les opérations, sélectionnez **Toutes les opérations**.
7. Sélectionnez l’icône **</>** (éditeur de code) dans la section **Traitement entrant** ou **Traitement sortant**.
8. Collez le code de la stratégie de votre choix dans l’une des zones appropriées.

    ```xml
    <policies>
        <inbound>
            <base />
        </inbound>
        <backend>
            <base />
        </backend>
        <outbound>
            <base />
        </outbound>
        <on-error>
            <base />
        </on-error>
    </policies>
    ```
 
## <a name="configure-scope"></a>Configurer l’étendue

Les stratégies peuvent être configurées de façon globale, ou bien au niveau de l’étendue d’un produit, d’une API ou d’une opération. Pour configurer une stratégie, vous devez d’abord sélectionner l’étendue à laquelle elle doit s’appliquer.

Les étendues de stratégie sont évaluées dans l’ordre suivant :

1. Étendue globale
2. Étendue produit
3. Étendue API
4. Étendue opération

Les instructions dans les stratégies sont évaluées en fonction de l’emplacement de l’élément `base`, s’il est présent. Une stratégie globale n’a aucune stratégie parente et l’utilisation de l’élément `<base>` n’a aucun effet.

Pour afficher les stratégies dans l'étendue actuelle dans l'éditeur de stratégie, cliquez sur **Recalculer la stratégie en vigueur pour l'étendue sélectionnée**.

### <a name="global-scope"></a>Étendue globale

L’étendue globale est configurée pour **toutes les API** dans votre instance APIM.

1. Connectez-vous au [portail Azure](https://portal.azure.com/) et accédez à votre instance APIM.
2. Cliquez sur **Toutes les API**.

    ![Étendue globale](./media/api-management-howto-policies/global-scope.png)

3. Cliquez sur l’icône de triangle.
4. Sélectionnez **Éditeur de Code**.
5. Ajoutez ou modifiez des stratégies.
6. Appuyez sur **Enregistrer**. 

    Les modifications sont propagées immédiatement dans la passerelle de Gestion des API.

### <a name="product-scope"></a>Étendue produit

L’étendue du produit est configurée pour le produit sélectionné.

1. Cliquez sur **Produits**.

    ![Étendue produit](./media/api-management-howto-policies/product-scope.png)

2. Sélectionnez le produit auquel vous souhaitez appliquer des stratégies.
3. Cliquez sur **Stratégies**.
4. Ajoutez ou modifiez des stratégies.
5. Appuyez sur **Enregistrer**. 

### <a name="api-scope"></a>Étendue API

L’étendue de l’API est configurée pour **toutes les opérations** de l’API sélectionnée.

1. Sélectionnez **l’API** à laquelle vous souhaitez appliquer des stratégies.

    ![Étendue API](./media/api-management-howto-policies/api-scope.png)

2. Sélectionnez **Toutes les opérations**.
3. Cliquez sur l’icône de triangle.
4. Sélectionnez **Éditeur de Code**.
5. Ajoutez ou modifiez des stratégies.
6. Appuyez sur **Enregistrer**. 

### <a name="operation-scope"></a>Étendue opération 

L’étendue de l’opération est configurée pour l’opération sélectionnée.

1. Sélectionnez une **API**.
2. Sélectionnez l’opération à laquelle vous souhaitez appliquer des stratégies.

    ![Étendue opération](./media/api-management-howto-policies/operation-scope.png)

3. Cliquez sur l’icône de triangle.
4. Sélectionnez **Éditeur de Code**.
5. Ajoutez ou modifiez des stratégies.
6. Appuyez sur **Enregistrer**. 

## <a name="next-steps"></a>Étapes suivantes

Consultez les rubriques associées suivantes :

+ [Transform and protect your API](transform-api.md) (Transformer et protéger votre API)
+ [Référence de stratégie](./api-management-policies.md) pour obtenir la liste complète des instructions et des paramètres de stratégie
+ [Exemples de stratégie](./policy-reference.md)
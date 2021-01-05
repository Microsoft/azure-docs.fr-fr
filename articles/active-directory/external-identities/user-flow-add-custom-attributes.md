---
title: Ajouter des attributs personnalisés aux flux d’inscription en libre-service - Azure AD
description: Découvrez la personnalisation des attributs de vos flux utilisateur d’inscription en libre-service.
services: active-directory
author: msmimart
manager: celestedg
ms.service: active-directory
ms.subservice: B2B
ms.topic: how-to
ms.date: 06/16/2020
ms.author: mimart
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: e64ab70fed13d4ca907b2bfb3aa448acdedc39e9
ms.sourcegitcommit: 9b8425300745ffe8d9b7fbe3c04199550d30e003
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92441451"
---
# <a name="define-custom-attributes-for-user-flows-preview"></a>Définir des attributs personnalisés pour les flux utilisateurs (préversion)

> [!NOTE]
> La fonctionnalité des attributs utilisateur personnalisés est une fonctionnalité en préversion publique d’Azure Active Directory. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Pour chaque application, vous pouvez avoir des exigences différentes concernant les informations que vous souhaitez collecter lors de l’inscription. Azure AD est fourni avec un ensemble intégré d’informations stockées dans des attributs, comme le prénom, le nom, la localité et le code postal. Avec Azure AD, vous pouvez étendre l’ensemble des attributs stockés sur un compte invité lorsque l’utilisateur externe s’inscrit via un flux utilisateur.

Vous pouvez créer des attributs personnalisés dans le portail Azure et les utiliser dans vos flux utilisateur d’inscription en libre-service. Vous pouvez également lire et écrire ces attributs à l’aide de [l’API Microsoft Graph](../../active-directory-b2c/manage-user-accounts-graph-api.md). L’API Microsoft Graph prend en charge la création et la mise à jour d’un utilisateur avec des attributs d’extension. Dans l’API Graph, les attributs d’extension sont nommés à l’aide de la convention `extension_<extensions-app-id>_attributename`. Par exemple :

```JSON
"extension_831374b3bd5041bfaa54263ec9e050fc_loyaltyNumber": "212342"
```

Cette valeur `<extensions-app-id>` est spécifique à votre locataire. Pour trouver cet identificateur, accédez à Azure Active Directory > Inscriptions d’applications > Toutes les applications. Recherchez l’application qui commence par « aad-extensions-app », puis sélectionnez-la. Sur la page Vue d’ensemble, notez l’ID d’application (client).

## <a name="create-a-custom-attribute"></a>Création d’un attribut personnalisé

1. Connectez-vous au [Portail Azure](https://portal.azure.com) en tant qu’administrateur Azure AD.
2. Sous **Services Azure** , sélectionnez **Azure Active Directory** .
3. Dans le menu de gauche, sélectionnez **Identités externes** .
4. Sélectionnez **Attributs utilisateur personnalisés (préversion)** . Les attributs utilisateur disponibles sont répertoriés.

   ![Sélectionner les attributs utilisateur pour l’inscription](media/user-flow-add-custom-attributes/user-attributes.png)

5. Pour ajouter un attribut, sélectionnez **Ajouter** .
6. Dans le volet **Ajouter un attribut** , entrez les valeurs suivantes :

   - **Nom**  : fournissez un nom pour l’attribut personnalisé (par exemple, « ShoeSize »).
   - **Type de données**  : choisissez un type de données ( **chaîne** , **booléen** ou **entier** ).
   - **Description**  : vous pouvez entrez une description de l’attribut personnalisé à usage interne. Cette description n’est pas visible par l’utilisateur.

   ![Ajouter un attribut](media/user-flow-add-custom-attributes/add-an-attribute.png)

7. Sélectionnez **Create** (Créer).

L’attribut personnalisé est maintenant disponible dans la liste des attributs utilisateur, et pour une utilisation dans vos flux utilisateur. Un attribut personnalisé est créé lors de sa première utilisation dans un flux utilisateur, non quand vous l’ajoutez à la liste des attributs utilisateur.

Une fois que vous avez créé un utilisateur à l’aide d’un flux utilisateur qui utilise l’attribut personnalisé nouvellement créé, l’objet peut être interrogé dans l’[Explorateur Microsoft Graph](https://developer.microsoft.com/graph/graph-explorer). Vous devez maintenant voir **ShoeSize** dans la liste d’attributs collectés lors de l’inscription sur l’objet utilisateur. Vous pouvez appeler l’API Graph à partir de votre application pour récupérer les données de cet attribut après qu’il a été ajouté à l’objet utilisateur.

## <a name="next-steps"></a>Étapes suivantes

[Ajouter un flux utilisateur d’inscription en libre-service à une application](self-service-sign-up-user-flow.md)
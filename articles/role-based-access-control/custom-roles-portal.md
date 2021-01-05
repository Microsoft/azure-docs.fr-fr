---
title: Créer ou mettre à jour des rôles personnalisés Azure avec le portail Azure - Azure RBAC
description: Découvrez comment créer des rôles personnalisés Azure à l’aide du portail Azure et du contrôle d’accès en fonction du rôle Azure (Azure RBAC). Cet article indique également comment répertorier, créer, mettre à jour et supprimer des rôles personnalisés.
services: active-directory
documentationcenter: ''
author: rolyon
manager: mtillman
ms.service: role-based-access-control
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: identity
ms.date: 07/13/2020
ms.author: rolyon
ms.openlocfilehash: 9ab6561b1dc0e8b977fc69d259c5f26366f355ce
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92481327"
---
# <a name="create-or-update-azure-custom-roles-using-the-azure-portal"></a>Créer ou mettre à jour des rôles personnalisés Azure à l’aide du portail Azure

Si les [rôles intégrés Azure](built-in-roles.md) ne répondent pas aux besoins spécifiques de votre organisation, vous pouvez créer vos propres rôles personnalisés Azure. Tout comme les rôles intégrés, vous pouvez attribuer des rôles personnalisés à des utilisateurs, à des groupes et à des principaux de service dans des étendues de abonnement et de groupe de ressources. Les rôles personnalisés sont stockés dans un annuaire Azure Active Directory et peuvent être partagés entre des abonnements. Chaque annuaire peut avoir jusqu’à 5 000 rôles personnalisés. Vous pouvez créer des rôles personnalisés à l’aide du portail Azure, d’Azure PowerShell, d’Azure CLI ou de l’API REST. Cet article explique comment créer des rôles personnalisés à l’aide du portail Azure.

## <a name="prerequisites"></a>Prérequis

Pour créer des rôles personnalisés, vous avez besoin des éléments suivants :

- autorisations nécessaires pour créer des rôles personnalisés, par exemple, [Propriétaire](built-in-roles.md#owner) ou [Administrateur de l’accès utilisateur](built-in-roles.md#user-access-administrator) ;

## <a name="step-1-determine-the-permissions-you-need"></a>Étape 1 : Déterminer les autorisations nécessaires

Azure dispose de plusieurs milliers d’autorisations que vous pouvez inclure dans votre rôle personnalisé. Voici quatre façons de déterminer les autorisations à ajouter à votre rôle personnalisé :

| Méthode | Description |
| --- | --- |
| Examiner des rôles existants | Vous pouvez examiner des rôles existants pour voir quelles autorisations sont utilisées. Pour plus d’informations, voir [Rôles intégrés Azure](built-in-roles.md). |
| Rechercher des autorisations par mot clé | Lorsque vous créez un rôle personnalisé à l’aide du portail Azure, vous pouvez rechercher des autorisations par mot clé. Par exemple, vous pouvez rechercher des autorisations d’ *ordinateur virtuel* ou de *facturation*. Cette fonctionnalité de recherche est décrite plus loin à l’[Étape 4 : Autorisations](#step-4-permissions). |
| Télécharger toutes les autorisations | Lorsque vous créez un rôle personnalisé à l’aide du portail Azure, vous pouvez télécharger toutes les autorisations en tant que fichier CSV, puis Rechercher celui-ci. Dans le volet **Ajouter des autorisations** , cliquez sur le bouton **Télécharger toutes les autorisations** pour les télécharger toutes. Pour plus d’informations sur le volet Ajouter des autorisations, voir l’[Étape 4 : Autorisations](#step-4-permissions). |
| Afficher les autorisations dans les documents | Vous pouvez afficher les autorisations disponibles dans les [opérations de fournisseur de ressources Azure Resource Manager](resource-provider-operations.md). |

## <a name="step-2-choose-how-to-start"></a>Étape 2 : Choisir comment commencer

Vous pouvez commencer à créer un rôle personnalisé de trois façons. Vous pouvez cloner un rôle existant, démarrer à partir de zéro ou démarrer avec un fichier JSON. Le moyen le plus simple consiste à rechercher un rôle existant disposant de la plupart des autorisations dont vous avez besoin, puis à le cloner et le modifier pour votre scénario. 

### <a name="clone-a-role"></a>Cloner un rôle

Si un rôle existant ne dispose pas de toutes les autorisations dont vous avez besoin, vous pouvez le cloner, puis modifier les autorisations. Pour démarrer le clonage d’un rôle, procédez comme suit.

1. Dans le portail Azure, ouvrez un abonnement ou un groupe de ressources dans lequel vous souhaitez que le rôle personnalisé soit attribuable, puis ouvrez **Contrôle d’accès (IAM)** .

    La capture d’écran suivante montre la page Contrôle d’accès (IAM) ouverte pour un abonnement.

    ![Page Contrôle d’accès (IAM) pour un abonnement](./media/custom-roles-portal/access-control-subscription.png)

1. Cliquez sur l’onglet **Rôles** pour afficher une liste de tous les rôles intégrés et personnalisés.

1. Recherchez un rôle à cloner, tel que le rôle Lecteur de facturation.

1. À la fin de la ligne, cliquez sur les points de suspension ( **...** ), puis sur **Cloner**.

    ![Menu contextuel Cloner](./media/custom-roles-portal/clone-menu.png)

    Cette action ouvre l’éditeur de rôles personnalisés avec l’option **Cloner un rôle** sélectionnée.

1. Passez à l’[Étape 3 : Paramètres de base](#step-3-basics).

### <a name="start-from-scratch"></a>Commencer à partir de zéro

Si vous préférez, vous pouvez suivre ces étapes pour démarrer un rôle personnalisé à partir de rien.

1. Dans le portail Azure, ouvrez un abonnement ou un groupe de ressources dans lequel vous souhaitez que le rôle personnalisé soit attribuable, puis ouvrez **Contrôle d’accès (IAM)** .

1. Cliquez sur **Ajouter** , puis sur **Ajouter un rôle personnalisé**.

    ![Menu Ajouter un rôle personnalisé](./media/custom-roles-portal/add-custom-role-menu.png)

    Cette action ouvre l’éditeur de rôles personnalisés avec l’option **Commencer à partir de zéro** sélectionnée.

1. Passez à l’[Étape 3 : Paramètres de base](#step-3-basics).

### <a name="start-from-json"></a>Démarrer à partir d’un JSON

Si vous préférez, vous pouvez spécifier la plupart des valeurs de votre rôle personnalisé dans un fichier JSON. Vous pouvez ouvrir le fichier dans l’éditeur de rôles personnalisés, apporter des modifications supplémentaires, puis créer le rôle personnalisé. Pour commencer avec un fichier JSON, procédez comme suit.

1. Créez un fichier JSON au format suivant :

    ```json
    {
        "properties": {
            "roleName": "",
            "description": "",
            "assignableScopes": [],
            "permissions": [
                {
                    "actions": [],
                    "notActions": [],
                    "dataActions": [],
                    "notDataActions": []
                }
            ]
        }
    }
    ```

1. Dans le fichier JSON, spécifiez les valeurs des différentes propriétés. Voici un exemple avec des valeurs ajoutées. Pour obtenir des informations sur les différentes propriétés, consultez [Comprendre les définitions de rôle Azure](role-definitions.md).

    ```json
    {
        "properties": {
            "roleName": "Billing Reader Plus",
            "description": "Read billing data and download invoices",
            "assignableScopes": [
                "/subscriptions/11111111-1111-1111-1111-111111111111"
            ],
            "permissions": [
                {
                    "actions": [
                        "Microsoft.Authorization/*/read",
                        "Microsoft.Billing/*/read",
                        "Microsoft.Commerce/*/read",
                        "Microsoft.Consumption/*/read",
                        "Microsoft.Management/managementGroups/read",
                        "Microsoft.CostManagement/*/read",
                        "Microsoft.Support/*"
                    ],
                    "notActions": [],
                    "dataActions": [],
                    "notDataActions": []
                }
            ]
        }
    }
    ```
    
1. Dans le portail Azure, ouvrez la page **Contrôle d’accès (IAM)** .

1. Cliquez sur **Ajouter** , puis sur **Ajouter un rôle personnalisé**.

    ![Menu Ajouter un rôle personnalisé](./media/custom-roles-portal/add-custom-role-menu.png)

    Cette action ouvre l’éditeur de rôles personnalisés.

1. Sous l’onglet Paramètres de base, dans **Autorisations de base** , sélectionnez **Démarrer à partir d’un JSON**.

1. En regard de la zone Sélectionner un fichier, cliquez sur le bouton de dossier pour ouvrir la boîte de dialogue Ouvrir.

1. Sélectionnez votre fichier JSON, puis cliquez sur **Ouvrir**.

1. Passez à l’[étape 3 : Paramètres de base](#step-3-basics).

## <a name="step-3-basics"></a>Étape 3 : Concepts de base

Sous l’onglet **Paramètres de base** , spécifiez le nom, la description et les autorisations de base pour votre rôle personnalisé.

1. Dans la zone **Nom de rôle personnalisé** , spécifiez un nom pour le rôle personnalisé. Le nom doit être unique pour l’annuaire Azure AD. Il peut inclure des lettres, des chiffres, des espaces et des caractères spéciaux.

1. Dans la zone **Description** , entrez une description facultative pour le rôle personnalisé. Celle-ci deviendra l’info-bulle pour le rôle personnalisé.

    L’option **Autorisations de base** devrait déjà être définie suite à l’étape précédente, mais vous pouvez la modifier.

    ![Onglet Paramètres de base avec des valeurs spécifiées](./media/custom-roles-portal/basics-values.png)

## <a name="step-4-permissions"></a>Étape 4 : Autorisations

Sous l’onglet **Autorisations** , vous spécifiez les autorisations pour votre rôle personnalisé. Selon que vous avez cloné un rôle ou avez démarré avec un fichier JSON, il se peut que l’onglet Autorisations répertorie déjà certaines autorisations.

![Onglet Autorisations pour la création d’un rôle personnalisé](./media/custom-roles-portal/permissions.png)

### <a name="add-or-remove-permissions"></a>Ajouter ou supprimer des autorisations

Pour ajouter ou supprimer des autorisations pour votre rôle personnalisé, procédez comme suit.

1. Pour ajouter des autorisations, cliquez sur **Ajouter des autorisations** pour ouvrir le volet Ajouter des autorisations.

    Ce volet répertorie toutes les autorisations disponibles regroupées par catégories dans un format de carte. Chaque catégorie représente un *fournisseur de ressources* , à savoir un service fournissant des ressources Azure.

1. Dans la zone **Rechercher une autorisation** , tapez une chaîne pour rechercher des autorisations. Par exemple, recherchez *facture* pour trouver des autorisations liées à la facture.

    Une liste de cartes de fournisseur de ressources s’affiche en fonction de votre chaîne de recherche. Pour obtenir une liste qui mappe des fournisseurs de ressources à des services Azure, voir [Fournisseurs de ressources pour les services Azure](../azure-resource-manager/management/azure-services-resource-providers.md).

    ![Volet Ajouter des autorisations avec fournisseur de ressources](./media/custom-roles-portal/add-permissions-provider.png)

1. Cliquez sur une carte de fournisseur de ressources susceptible d’avoir les autorisations que vous souhaitez ajouter à votre rôle personnalisé, par exemple **Facturation Microsoft**.

    Une liste des autorisations de gestion pour ce fournisseur de ressources s’affiche en fonction de votre chaîne de recherche.

    ![Liste Ajouter des autorisations](./media/custom-roles-portal/add-permissions-list.png)

1. Si vous recherchez des autorisations qui s’appliquent au plan de données, cliquez sur **Actions de données**. Dans le cas contraire, laissez le bouton bascule des actions positionné sur **Actions** pour répertorier les autorisations qui s’appliquent au plan de gestion. Pour plus d’informations sur les différences entre le plan de gestion et le plan de données, voir [Opérations de gestion et sur les données](role-definitions.md#management-and-data-operations).

1. Si nécessaire, mettez à jour la chaîne de recherche pour affiner votre recherche.

1. Une fois que vous avez trouvé une ou plusieurs autorisations à ajouter à votre rôle personnalisé, ajoutez une coche en regard des autorisations. Par exemple, ajoutez une coche en regard de **Autre : Télécharger la facture** pour ajouter l’autorisation de télécharger des factures.

1. Cliquez sur **Ajouter** pour ajouter l’autorisation à votre liste d’autorisations.

    L’autorisation est ajoutée en tant que `Actions` ou `DataActions`.

    ![Autorisation ajoutée](./media/custom-roles-portal/permissions-list-add.png)

1. Pour supprimer des autorisations, cliquez sur l’icône Supprimer à la fin de la ligne. Dans cet exemple, étant donné qu’un utilisateur n’a pas besoin de pouvoir créer des tickets de support, l’autorisation `Microsoft.Support/*` peut être supprimée.

### <a name="add-wildcard-permissions"></a>Ajouter des autorisations génériques

Selon la façon dont vous avez choisi de commencer, il se peut que votre liste d’autorisations contienne des autorisations avec des caractères génériques (`*`). Un caractère générique (`*`) étend une autorisation à tout ce qui correspond à la chaîne d’action que vous fournissez. Par exemple, la chaîne de caractères génériques suivante ajoute toutes les autorisations relatives à Azure Cost Management et aux exportations. Cela inclut également toutes les futures autorisations d’exportation qui pourraient être ajoutées.

```
Microsoft.CostManagement/exports/*
```

Si vous souhaitez ajouter une nouvelle autorisation générique, vous ne pouvez pas le faire à l’aide du volet **Ajouter des autorisations**. Pour ajouter une autorisation générique, vous devez procéder manuellement à l’aide de l’onglet **JSON**. Pour plus d’informations, consultez [Étape 6 : JSON](#step-6-json).

### <a name="exclude-permissions"></a>Exclure les autorisations

Si votre rôle a une autorisation générique (`*`) et que vous souhaitez exclure ou soustraire des autorisations spécifiques de cette autorisation générique, vous pouvez les exclure. Par exemple, supposons que vous disposez de l’autorisation générique suivante :

```
Microsoft.CostManagement/exports/*
```

Si vous ne souhaitez pas autoriser la suppression d’une exportation, vous pouvez exclure l’autorisation de suppression suivante :

```
Microsoft.CostManagement/exports/delete
```

Lorsque vous excluez une autorisation, celle-ci est ajoutée en tant que `NotActions` ou `NotDataActions`. Les autorisations de gestion effectives sont calculées en ajoutant toutes les `Actions`, puis en soustrayant toutes les `NotActions`. Les autorisations de données effectives sont calculées en ajoutant toutes les `DataActions`, puis en soustrayant toutes les `NotDataActions`.

> [!NOTE]
> Une exclusion d’autorisation n’est pas la même chose qu’un refus. L’exclusion d’autorisations est simplement un moyen pratique de soustraire des autorisations d’une autorisation générique.

1. Pour exclure ou soustraire une autorisation d’une autorisation générique accordée, cliquez sur **Exclure les autorisations** pour ouvrir le volet Exclure les autorisations.

    Dans ce volet, vous spécifiez les autorisations de gestion ou de données qui sont exclues ou soustraites.

1. Une fois que vous avez trouvé une ou plusieurs autorisations à exclure, ajoutez une coche en regard des autorisations, puis cliquez sur le bouton **Ajouter**.

    ![Volet Exclure les autorisations avec une autorisation sélectionnée](./media/custom-roles-portal/exclude-permissions-select.png)

    L’autorisation est ajoutée en tant que `NotActions` ou `NotDataActions`.

    ![Autorisation exclue](./media/custom-roles-portal/exclude-permissions-list-add.png)

## <a name="step-5-assignable-scopes"></a>Étape 5 : Étendues attribuables

Sous l’onglet **Étendues attribuables** , spécifiez l’emplacement où votre rôle personnalisé est disponible pour affectation, par exemple, un abonnement ou un groupe de ressources. Selon la façon dont vous avez choisi de commencer, cet onglet peut répertorier l’étendue dans laquelle vous avez ouvert la page Contrôle d’accès (IAM). La définition de l’étendue attribuable sur l’étendue racine (« / ») n’est pas prise en charge. Actuellement, vous ne pouvez pas ajouter un groupe d’administration en tant qu’étendue attribuable.

1. Cliquez sur **Ajouter des étendues attribuables** pour ouvrir le volet Ajouter des étendues attribuables.

    ![Onglet Étendues attribuables](./media/custom-roles-portal/assignable-scopes.png)

1. Cliquez sur une ou plusieurs étendues à utiliser, généralement votre abonnement.

    ![Ajouter des étendues attribuables](./media/custom-roles-portal/add-assignable-scopes.png)

1. Cliquez sur le bouton **Ajouter** pour ajouter votre étendue attribuable.

## <a name="step-6-json"></a>Étape 6 : JSON

Sous l’onglet **JSON** , vous voyez votre rôle personnalisé au format JSON. Si vous le souhaitez, vous pouvez modifier directement le JSON. Si vous souhaitez ajouter une autorisation générique (`*`), vous devez utiliser cet onglet.

1. Pour modifier le JSON, cliquez sur **Modifier**.

    ![Onglet JSON présentant un rôle personnalisé](./media/custom-roles-portal/json.png)

1. Modifiez le JSON.

    Si le JSON n’est pas mis en forme correctement, une ligne en escalier rouge et un indicateur apparaissent dans la marge verticale.

1. Une fois la modification terminée, cliquez sur **Enregistrer**.

## <a name="step-7-review--create"></a>Étape 7 : Vérifier + créer

Sous l’onglet **Vérifier + créer** , vous pouvez vérifier les paramètres de votre rôle personnalisé.

1. Vérifiez les paramètres de votre rôle personnalisé.

    ![Onglet Vérifier + créer](./media/custom-roles-portal/review-create.png)

1. Cliquez sur **Créer** pour créer votre rôle personnalisé.

    Après quelques instants, une zone de message s’affiche, indiquant que votre rôle personnalisé a été créé avec succès.

    ![Créer un message de rôle personnalisé](./media/custom-roles-portal/custom-role-success.png)

    Si des erreurs sont détectées, un message s’affiche.

    ![Erreur Vérifier + créer](./media/custom-roles-portal/review-create-error.png)

1. Affichez votre nouveau rôle personnalisé dans la liste **Rôles**. Si vous ne voyez pas votre rôle personnalisé, cliquez sur **Actualiser**.

     Plusieurs minutes peuvent s’écouler avant que votre rôle personnalisé s’affiche partout.

## <a name="list-custom-roles"></a>Répertorier les rôles personnalisés

Pour afficher vos rôles personnalisés, procédez comme suit.

1. Ouvrez un abonnement ou un groupe de ressources, puis ouvrez **Contrôle d’accès (IAM)** .

1. Cliquez sur l’onglet **Rôles** pour afficher une liste de tous les rôles intégrés et personnalisés.

1. Dans la liste **Type** , sélectionnez **CustomRole** pour voir vos rôles personnalisés.

    Si vous venez de créer votre rôle personnalisé et ne le voyez pas dans la liste, cliquez sur **Actualiser**.

    ![Liste Rôle personnalisé](./media/custom-roles-portal/custom-role-list.png)

## <a name="update-a-custom-role"></a>Mettre à jour un rôle personnalisé

1. Comme décrit précédemment dans cet article, ouvrez votre liste de rôles personnalisés.

1. Cliquez sur les points de suspension ( **...** ) pour le rôle personnalisé à mettre à jour, puis cliquez sur **Modifier**. Notez que vous ne pouvez pas mettre à jour des rôles intégrés.

    Le rôle personnalisé s’ouvre dans l’éditeur.

    ![Menu Rôle personnalisé](./media/custom-roles-portal/edit-menu.png)

1. Utilisez les différents onglets pour mettre à jour le rôle personnalisé.

1. Une fois vos modifications apportées, cliquez sur l’onglet **Vérifier + créer** pour passer en revue vos modifications.

1. Cliquez sur le bouton **Mettre à jour** pour mettre à jour votre rôle personnalisé.

## <a name="delete-a-custom-role"></a>Supprimer un rôle personnalisé

1. Comme décrit précédemment dans cet article, ouvrez votre liste de rôles personnalisés.

1. Supprimez toutes les attributions de rôles qui utilisent le rôle personnalisé.

1. Cliquez sur les points de suspension ( **...** ) pour le rôle personnalisé à supprimer, puis cliquez sur **Supprimer**.

    ![Capture d’écran montrant une liste de rôles personnalisés qui peuvent être sélectionnés pour être supprimés.](./media/custom-roles-portal/delete-menu.png)

    Plusieurs minutes peuvent s’écouler avant que votre rôle personnalisé soit complètement supprimé.

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Créer un rôle personnalisé Azure à l’aide d’Azure PowerShell](tutorial-custom-role-powershell.md)
- [Rôle personnalisés Azure](custom-roles.md)
- [Opérations du fournisseur de ressources Azure Resource Manager](resource-provider-operations.md)

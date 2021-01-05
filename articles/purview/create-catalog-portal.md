---
title: 'Démarrage rapide : Créer un compte Azure Purview dans le portail Azure (préversion)'
description: Ce guide de démarrage rapide explique comment créer un compte Azure Purview et configurer des autorisations pour commencer à l’utiliser.
author: nayenama
ms.author: nayenama
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: quickstart
ms.date: 10/23/2020
ms.openlocfilehash: fe83425230a157be245e3aa1ad088c32596ea5ed
ms.sourcegitcommit: b6267bc931ef1a4bd33d67ba76895e14b9d0c661
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/19/2020
ms.locfileid: "97693371"
---
# <a name="quickstart-create-an-azure-purview-account-in-the-azure-portal"></a>Démarrage rapide : Créer un compte Azure Purview dans le portail Azure

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Dans ce guide de démarrage rapide, vous créez un compte Azure Purview.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Votre propre [locataire Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).

* Votre compte doit avoir l’autorisation de créer des ressources dans l’abonnement.

* Si **Azure Policy** empêche toutes les applications de créer un **compte de stockage** et un **espace de noms EventHub**, vous devez introduire une exception de stratégie au moyen d’une étiquette pouvant être renseignée lors du processus de création d’un compte Purview. La raison principale vient de ce que, pour chaque compte Purview créé, un groupe de ressources managé doit être créé, et dans ce groupe de ressources, un compte de stockage et un espace de noms EventHub.

    > [!important]
    > Vous n’êtes pas obligé de suivre cette étape si vous ne disposez pas d’Azure Policy, ou si aucune stratégie Azure existante ne bloque la création du **compte de stockage** et de l’**espace de noms EventHub**.

    1. Accédez au portail Azure et recherchez **Stratégie**.
    1. Suivez [Créer une définition de stratégie personnalisée](https://docs.microsoft.com/azure/governance/policy/tutorials/create-custom-policy-definition) ou modifiez une stratégie existante pour ajouter deux exceptions avec l’opérateur `not` et l’étiquette `resourceBypass` :

        ```json
        {
          "mode": "All",
          "policyRule": {
            "if": {
              "anyOf": [
              {
                "allOf": [
                {
                  "field": "type",
                  "equals": "Microsoft.Storage/storageAccounts"
                },
                {
                  "not": {
                    "field": "tags['<resourceBypass>']",
                    "exists": true
                  }
                }]
              },
              {
                "allOf": [
                {
                  "field": "type",
                  "equals": "Microsoft.EventHub/namespaces"
                },
                {
                  "not": {
                    "field": "tags['<resourceBypass>']",
                    "exists": true
                  }
                }]
              }]
            },
            "then": {
              "effect": "deny"
            }
          },
          "parameters": {}
        }
        ```
        
        > [!Note]
        > Tant que la stratégie détecte l’étiquette, celle-ci peut être constituée de n’importe quel texte à côté de `resourceBypass` ; c’est à vous de définir sa valeur lors de la création de Purview aux étapes suivantes.

        :::image type="content" source="./media/create-catalog-portal/policy-definition.png" alt-text="Capture d’écran illustrant la création d’une définition de stratégie.":::

    1. [Créez une affectation de stratégie](https://docs.microsoft.com/azure/governance/policy/assign-policy-portal) à l’aide de la stratégie personnalisée créée.

        [ ![Capture d’écran illustrant la création d’une affectation de stratégie](./media/create-catalog-portal/policy-assignment.png)](./media/create-catalog-portal/policy-assignment.png#lightbox)

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

## <a name="configure-your-subscription"></a>Configurer votre abonnement

Si nécessaire, suivez ces étapes qui permettent de configurer votre abonnement en vue d’autoriser l’exécution d’Azure Purview dans votre abonnement :

   1. Dans le portail Azure, recherchez et sélectionnez **Abonnements**.

   1. Dans la liste des abonnements, sélectionnez l’abonnement que vous souhaitez utiliser. L’autorisation d’accès administratif pour l’abonnement est nécessaire.

      :::image type="content" source="./media/create-catalog-portal/select-subscription.png" alt-text="Capture d’écran illustrant la sélection d’un abonnement dans le portail Azure.":::

   1. Sur votre abonnement, sélectionnez **Fournisseurs de ressources**. Dans le volet **Fournisseurs de ressources**, recherchez et inscrivez les trois fournisseurs de ressources : 
       1. **Microsoft.Purview**
       1. **Microsoft.Storage**
       1. **Microsoft.EventHub** 
      
      S’ils ne sont pas inscrits, inscrivez-les en sélectionnant **Inscrire**.

      :::image type="content" source="./media/create-catalog-portal/register-purview-resource-provider.png" alt-text="Capture d’écran illustrant l’inscription du fournisseur de ressources Microsoft.Azure Purview dans le portail Azure.":::

## <a name="create-an-azure-purview-account-instance"></a>Créer une instance de compte Azure Purview

1. Accédez à la page **Comptes Purview** dans le portail Azure et sélectionnez **Ajouter** pour créer un compte Azure Purview. Vous pouvez également accéder à la Place de marché pour rechercher **Comptes Purview**, puis sélectionner **Créer**. Notez que vous ne pouvez ajouter qu’un seul compte Azure Purview à la fois.

   :::image type="content" source="./media/create-catalog-portal/add-purview-instance.png" alt-text="Capture d’écran illustrant la création d’une instance de compte Azure Purview dans le portail Azure.":::

1. Dans l’onglet **De base**, effectuez les actions suivantes :
    1. Sélectionnez un **groupe de ressources**.
    1. Entrez le **Nom du compte Purview** pour votre catalogue. Les espaces et les symboles ne sont pas autorisés.
    1. Choisissez un **Emplacement**, puis sélectionnez **Suivant : Configuration**.
1. Sous l’onglet **Configuration**, sélectionnez la **Taille de la plateforme** souhaitée, les valeurs autorisées étant 4 et 16 unités de capacité (CU). Sélectionnez **Suivant : Balises**.
1. Le cas échéant, sous l’onglet **Étiquettes**, vous pouvez ajouter une ou plusieurs étiquettes. Ces étiquettes sont destinées uniquement à être utilisées dans le portail Azure, et non dans Azure Purview. 

    > [!Note] 
    > Si vous utilisez **Azure Policy** et qu’il vous faut ajouter une exception, comme dans les **Prérequis**, vous devez choisir l’étiquette appropriée. Par exemple, vous pouvez ajouter l’étiquette `resourceBypass` : :::image type="content" source="./media/create-catalog-portal/add-purview-tag.png" alt-text="Ajout d’une étiquette au compte Purview.":::

1. Sélectionnez **Vérifier + créer**, puis **Créer**. La création prend quelques minutes. L’instance de compte Azure Purview nouvellement créée apparaît dans la liste de la page de vos **Comptes Purview**.
1. Lorsque le provisionnement du nouveau compte est terminé, sélectionnez **Accéder à la ressource**.

    > [!Note]
    > Si le provisionnement a échoué avec l’état `Conflict`, cela signifie qu’une stratégie Azure empêche Purview de créer un **compte de stockage** et un **espace de noms EventHub**. Vous devez suivre les étapes de la section **Prérequis** pour ajouter des exceptions.
    > :::image type="content" source="./media/create-catalog-portal/purview-conflict-error.png" alt-text="Message d’erreur de conflit Purview":::

1. Sélectionnez **Lancer le compte Purview**.

   :::image type="content" source="./media/use-purview-studio/launch-from-portal.png" alt-text="Capture d’écran de la sélection permettant de lancer le catalogue du compte Azure Purview.":::

## <a name="add-a-security-principal-to-a-data-plane-role"></a>Ajouter un principal de sécurité à un rôle de plan de données

Pour que vous ou votre équipe puissiez commencer à utiliser Azure Purview, un ou plusieurs principaux de sécurité doivent être ajoutés à l’un des rôles de plan de données prédéfinis : **Lecteur de données Purview**, **Conservateur de données Purview** ou **Administrateur de sources de données Purview**. Pour plus d’informations sur les autorisations du catalogue de données Azure Purview, consultez [Autorisations du catalogue](catalog-permissions.md).

Pour ajouter un principal de sécurité au rôle de plan de données **Conservateur de données Purview** dans un compte Azure Purview :

1. Accédez à la page [**Comptes Purview**](https://aka.ms/purviewportal) dans le portail Azure.

1. Sélectionnez le compte Azure Purview que vous souhaitez modifier.

1. Sur la page **Compte Purview**, sélectionnez l’onglet **Contrôle d’accès (IAM)** .

1. Cliquez sur **+ Ajouter**

Si, après avoir cliqué sur Ajouter, deux options s’affichent marquées (désactivées), cela signifie que vous ne détenez pas les autorisations appropriées pour ajouter un utilisateur à un rôle de plan de données dans le compte Azure Purview. Vous devez trouver un propriétaire, un administrateur de l’accès utilisateur ou une autre personne disposant de l’autorité d’affectation de rôle sur votre compte Azure Purview. Vous pouvez rechercher les personnes ad hoc en sélectionnant l’onglet **Attributions de rôles**, puis en faisant défiler la liste pour trouver le propriétaire ou l’administrateur de l’accès utilisateur, et contacter ces personnes.

1. Sélectionnez **Ajouter une attribution de rôle**.

1. Pour le rôle, indiquez **Rôle Conservateur de données Purview** ou **Rôle Administrateur de sources de données Purview**, en fonction de l’utilisation qui sera faite du principal de sécurité (pour plus d’informations, consultez [Autorisations du catalogue](catalog-permissions.md) et [Objets application et principal du service dans Azure Active Directory](https://docs.microsoft.com/azure/active-directory/develop/app-objects-and-service-principals)).

1. Pour **Attribuer l’accès à**, laissez la valeur par défaut **Utilisateur, groupe ou principal de service**.

1. Pour **Sélectionner**, entrez le nom de l’utilisateur, du principal de service ou du groupe Azure Active Directory que vous souhaitez affecter, puis cliquez sur son nom dans le volet des résultats.

1. Cliquez sur **Save**(Enregistrer).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez plus besoin de ce compte Azure Purview, supprimez-le en procédant de la façon suivante :

1. Accédez à la page **Comptes Purview** dans le portail Azure.

2. Sélectionnez le compte Azure Purview que vous avez créé au début de ce guide de démarrage rapide. Sélectionnez **Supprimer**, entrez le nom du compte, puis sélectionnez **Supprimer**.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment créer un compte Azure Purview.

Passez à l’article suivant pour savoir comment autoriser les utilisateurs à accéder à votre compte Azure Purview. 

> [!div class="nextstepaction"]
> [Ajouter des utilisateurs à votre compte Azure Purview](catalog-permissions.md)

---
title: Démarrage rapide - Créer un budget avec un modèle Azure Resource Manager
description: Démarrage rapide montrant comment créer un budget avec un modèle Azure Resource Manager.
author: bandersmsft
ms.author: banders
tags: azure-resource-manager
ms.service: cost-management-billing
ms.subservice: cost-management
ms.topic: quickstart
ms.date: 07/28/2020
ms.custom: subject-armqs, devx-track-azurecli
ms.openlocfilehash: 7d93bd757a39247302a6bc09009a1a814425c32f
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92745372"
---
# <a name="quickstart-create-a-budget-with-an-arm-template"></a>Démarrage rapide : Créer un budget avec un modèle ARM

Les budgets dans Cost Management vous aident à planifier et à suivre la comptabilité de l’organisation. Avec les budgets, vous pouvez prendre en compte les services Azure que vous consommez ou auxquels vous vous abonnez pendant une période spécifique. Ils vous permettent d’informer les autres utilisateurs de leurs dépenses pour gérer les coûts de manière proactive, ainsi que pour superviser la progression des dépenses. En cas de dépassement des seuils budgétaires que vous avez créés, des notifications sont déclenchées. Aucune de vos ressources n’est affectée et votre consommation n’est pas arrêtée. Vous pouvez utiliser des budgets pour comparer et suivre les dépenses lors de l’analyse des coûts. Ce guide de démarrage rapide vous montre comment créer un budget à l’aide d’un modèle Azure Resource Manager (modèle ARM).

[!INCLUDE [About Azure Resource Manager](../../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure** . Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fcreate-budget%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous disposez d’un nouvel abonnement, vous ne pouvez pas créer un budget ou utiliser les autres fonctionnalités de Cost Management tout de suite. Vous risquez de devoir attendre jusqu’à 48 heures avant de pouvoir utiliser toutes les fonctionnalités de Cost Management.

Les budgets sont pris en charge pour les types suivants d’étendues et de types de comptes Azure :

- Étendues de contrôle d’accès en fonction du rôle Azure (Azure RBAC)
    - Groupes d’administration
    - Abonnement
- Étendues Contrat Entreprise
    - Compte de facturation
    - department
    - Compte d’inscription
- Contrats individuels
    - Compte de facturation
- Étendues Contrat client Microsoft
    - Compte de facturation
    - Profil de facturation
    - Section de facture
    - Customer
- Étendues AWS
    - Compte externe
    - Abonnement externe

Pour afficher les budgets, vous devez au minimum disposer d'un accès en lecture à votre compte Azure.

Dans le cadre des abonnements Azure EA, vous devez disposer d'un accès en lecture pour afficher les budgets. Pour créer et gérer des budgets, vous devez disposer d’une autorisation de contributeur.

Les autorisations, ou étendues, Azure suivantes sont prises en charge par abonnement aux budgets par utilisateur et par groupe. Pour plus d’informations sur les étendues, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).

- Propriétaire : peut créer, modifier ou supprimer des budgets pour un abonnement.
- Contributeur et Contributeur Cost Management : peut créer, modifier ou supprimer ses propres budgets. Peut modifier le montant des budgets créés par d’autres utilisateurs.
- Lecteur et Lecteur Cost Management : peut afficher les budgets pour lesquels ils disposent des autorisations adéquates.

Pour plus d’informations sur l’affectation d’une autorisation d’accès aux données Cost Management, consultez [Affecter une autorisation d’accès aux données Cost Management](assign-access-acm-data.md).

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/create-budget).

:::code language="json" source="~/quickstart-templates/create-budget/azuredeploy.json" :::

Une seule ressource Azure est définie dans le modèle :

* [Microsoft.Consumption/budgets](/azure/templates/microsoft.consumption/budgets) : Créer un budget Azure.

## <a name="deploy-the-template"></a>Déployer le modèle

1. Cliquez sur l’image ci-après pour vous connecter à Azure et ouvrir un modèle. Le modèle crée un budget.

   [![Déployer sur Azure](../../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2Fcreate-budget%2Fazuredeploy.json)

2. Sélectionnez ou entrez les valeurs suivantes.

   :::image type="content" source="./media/quick-create-budget-template/create-budget-using-template-portal.png" alt-text="Modèle Resource Manager, Créer un budget, portail de déploiement]" lightbox="./media/quick-create-budget-template/create-budget-using-template-portal.png" :::
   
    * **Abonnement** : sélectionnez un abonnement Azure.
    * **Groupe de ressources**  : si nécessaire, sélectionnez un groupe de ressources ou **créez-en un** .
    * **Région** : sélectionnez une région Azure. Par exemple, **USA Centre** .
    * **Nom du budget**  : entrez le nom pour le budget. Il doit être unique au sein d’un groupe de ressources. Seuls les caractères alphanumériques, le trait de soulignement et les traits d’union sont autorisés.
    * **Montant**  : entrez le montant total du coût à suivre avec le budget.
    * **Fragment de temps**  : entrez la durée couverte par un budget. Les valeurs autorisées sont Mensuel, Trimestriel ou Annuel. Le budget se réinitialise à la fin du fragment de temps.
    * **Date de début**  : entrez la date de début avec le premier jour du mois au format AAAA-MM-JJ. La date de début ultérieure ne doit pas être supérieure à trois mois à compter du jour actuel. Vous pouvez spécifier une date de début passée avec la période Fragment de temps.
    * **Date de fin**  : entrez la date de fin du budget au format AAAA-MM-JJ. 
    * **Premier seuil**  : entrez une valeur de seuil pour la première notification. Une notification est envoyée quand le coût dépasse le seuil. Il est toujours exprimé en pourcentage et doit être compris entre 0 et 1 000.
    * **Deuxième seuil**  : entrez une valeur de seuil pour la deuxième notification. Une notification est envoyée quand le coût dépasse le seuil. Il est toujours exprimé en pourcentage et doit être compris entre 0 et 1 000.
    * **Rôles de contact**  : entrez une liste de rôles de contact auxquels envoyer la notification de budget quand le seuil est dépassé. Les valeurs par défaut sont Propriétaire, Contributeur et Lecteur. Le format attendu est `["Owner","Contributor","Reader"]`.
    * **E-mails de contact**  : entrez une liste d’adresses e-mail auxquelles envoyer la notification de budget quand un seuil est dépassé. Le format attendu est `["user1@domain.com","user2@domain.com"]`.
    * **Groupes de contacts**  : entrez une liste d’ID de ressources de groupes d’actions, sous la forme d’URI de ressource complets, auxquels envoyer la notification de budget quand le seuil est dépassé. Elle accepte un tableau de chaînes. Le format attendu est `["action group resource ID1","action group resource ID2"]`. Si vous ne voulez pas utiliser de groupes d’actions, entrez `[]`.
    * **Valeurs de filtre de groupe de ressources** Entrez une liste de noms de groupes de ressources à filtrer. Le format attendu est `["Resource Group Name1","Resource Group Name2"]`. Si vous ne voulez pas appliquer de filtre, entrez `[]`. 
    * **Valeurs de filtre de catégorie de compteur** Entrez une liste de catégories de compteurs de service Azure. Le format attendu est `["Meter Category1","Meter Category2"]`. Si vous ne voulez pas appliquer de filtre, entrez `[]`.
   
3. En fonction de votre type d’abonnement Azure, effectuez l’une des actions suivantes :
   - Sélectionnez **Revoir + créer** .
   - Prenez connaissance des conditions générales, cochez la case **J’accepte les termes et conditions mentionnés ci-dessus** , puis cochez sélectionnez **Acheter** .

4. Si vous avez sélectionné **Vérifier + créer** , votre modèle est validé. Sélectionnez **Create** (Créer).  

   ![Modèle Resource Manager, budget, notification du portail de déploiement](./media/quick-create-budget-template/resource-manager-template-portal-deployment-notification.png)

Le portail Azure est utilisé pour déployer le modèle. Outre le portail Azure, vous pouvez également utiliser Azure PowerShell, Azure CLI et l’API REST. Pour découvrir d’autres modèles de déploiement, consultez [Déployer des modèles](../../azure-resource-manager/templates/deploy-powershell.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

Vous pouvez utiliser le portail Azure pour vérifier que le budget est créé en accédant à **Cost Management + Facturation**  > sélectionnez une étendue > **Budgets** . Vous pouvez également utiliser les scripts Azure CLI ou Azure PowerShell suivants pour voir le budget.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
az consumption budget list
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
Get-AzConsumptionBudget
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’avez plus besoin d’un budget, supprimez-le à l’aide de l’une des méthodes suivantes :

### <a name="azure-portal"></a>Portail Azure

Accédez à **Cost Management + Facturation** > sélectionnez une étendue de facturation > **Budgets** > sélectionnez un budget > puis sélectionnez **Supprimer le budget** .

### <a name="command-line"></a>Ligne de commande

Vous pouvez supprimer le budget à l’aide d’Azure CLI ou d’Azure PowerShell.

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the budget name:" &&
read budgetName &&
az consumption budget delete --budget-name $budgetName &&
echo "Press [ENTER] to continue ..."
```

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$budgetName = Read-Host -Prompt "Enter the budget name"
Remove-AzConsumptionBudget -Name $budgetName
Write-Host "Press [ENTER] to continue..."
```

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un budget Azure pour le déploiement. Pour en savoir plus sur Azure Cost Management + Facturation et Azure Resource Manager, passez aux articles ci-dessous.

- Lire la vue d’ensemble d’[Azure Cost Management + Facturation](../cost-management-billing-overview.md)
- [Créer des budgets](tutorial-acm-create-budgets.md) dans le portail Azure
- En savoir plus sur [Azure Resource Manager](../../azure-resource-manager/management/overview.md)

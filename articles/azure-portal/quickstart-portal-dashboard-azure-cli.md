---
title: Créer un tableau de bord sur le portail Azure avec Azure CLI
description: 'Démarrage rapide : Découvrez comment créer un tableau de bord dans le portail Azure en utilisant Azure CLI. Un tableau de bord est une vue ciblée et organisée de vos ressources cloud.'
ms.topic: quickstart
ms.custom: devx-track-azurepowershell
ms.date: 12/4/2020
ms.openlocfilehash: ede915df5cd2967c3b6b700bcb9174c89af8f233
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96745653"
---
# <a name="quickstart-create-an-azure-portal-dashboard-with-azure-cli"></a>Démarrage rapide : Créer un tableau de bord sur le portail Azure avec Azure CLI

Dans le portail Azure, un tableau de bord est une vue ciblée et organisée de vos ressources cloud.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Si vous avez plusieurs abonnements Azure, choisissez celui dans lequel les ressources doivent être facturées.
Pour sélectionner un abonnement, utilisez la commande [az account set](/cli/azure/account#az_account_set) :

  ```azurecli
  az account set --subscription 00000000-0000-0000-0000-000000000000
  ```

- Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) à l’aide de la commande [az group create](/cli/azure/group#az_group_create) ou utilisez un groupe de ressources existant :

  ```azurecli
  az group create --name myResourceGroup --location centralus
  ```

   Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

## <a name="create-a-virtual-machine"></a>Créer une machine virtuelle

Créez une machine virtuelle à l’aide de la commande [az vm create](/cli/azure/vm#az_vm_create) :

```azurecli
az vm create --resource-group myResourceGroup --name SimpleWinVM --image win2016datacenter \
   --admin-username azureuser --admin-password 1StrongPassword$
```

> [!Note]
> Le mot de passe doit être complexe.
> Il s’agit d’un nouveau nom d’utilisateur et d’un nouveau mot de passe.
> Ce n’est pas, par exemple, le compte que vous utilisez pour vous connecter à Azure.
> Pour plus d’informations, consultez les [exigences relatives aux noms d’utilisateur](../virtual-machines/windows/faq.md#what-are-the-username-requirements-when-creating-a-vm) et les [exigences relatives aux mots de passe](../virtual-machines/windows/faq.md#what-are-the-password-requirements-when-creating-a-vm).

Le déploiement démarre et se termine au bout de quelques minutes.
Une fois le déploiement terminé, passez à la section suivante.

## <a name="download-the-dashboard-template"></a>Télécharger le modèle de tableau de bord

Étant donné que les tableaux de bord Azure sont des ressources, ils peuvent être représentés au format JSON.
Pour plus d’informations, consultez [Structure des tableaux de bord Azure](./azure-portal-dashboards-structure.md).

Téléchargez le fichier suivant : [portal-dashboard-template-testvm.json](https://raw.githubusercontent.com/Azure/azure-docs-powershell-samples/master/azure-portal/portal-dashboard-template-testvm.json).

Personnalisez le modèle téléchargé en remplaçant les valeurs suivantes par les vôtres :

* `<subscriptionID>` : Votre abonnement
* `<rgName>` : Groupe de ressources, par exemple `myResourceGroup`
* `<vmName>` : Nom de la machine virtuelle, par exemple `SimpleWinVM`
* `<dashboardTitle>` : Titre du tableau de bord, par exemple `Simple VM Dashboard`
* `<location>` : Votre région Azure, par exemple `centralus`

Pour plus d’informations, consultez [Référence sur le modèle de tableaux de bord Microsoft.Portal](/azure/templates/microsoft.portal/dashboards).

## <a name="deploy-the-dashboard-template"></a>Déployer le modèle de tableau de bord

Vous pouvez maintenant déployer le modèle à partir d’Azure CLI.

1. Exécutez la commande [az portal dashboard create](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_create) pour déployer le modèle :

   ```azurecli
   az portal dashboard create --resource-group myResourceGroup --name 'Simple VM Dashboard' \
      --input-path portal-dashboard-template-testvm.json --location centralus
   ```

1. Vérifiez que le tableau de bord a bien été créé en exécutant la commande [az portal dashboard show](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_show) :

   ```azurecli
   az portal dashboard show --resource-group myResourceGroup --name 'Simple VM Dashboard'
   ```

Pour voir tous les tableaux de bord de l’abonnement actif, utilisez [az portal dashboard list](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_list) :

```azurecli
az portal dashboard list
```

Vous pouvez aussi voir tous les tableaux de bord d’un groupe de ressources :

```azurecli
az portal dashboard list --resource-group myResourceGroup
```

Vous pouvez mettre à jour un tableau de bord à l’aide de la commande [az portal dashboard update](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_update) :

```azurecli
az portal dashboard update --resource-group myResourceGroup --name 'Simple VM Dashboard' \
   --input-path portal-dashboard-template-testvm.json --location centralus
```

Vérifiez que vous pouvez voir les données sur la machine virtuelle à partir du portail Azure.

1. Dans le portail Azure, sélectionnez **Tableau de bord**.

   ![Navigation du portail Azure jusqu’au tableau de bord](media/quickstart-portal-dashboard-powershell/navigate-to-dashboards.png)

1. Dans la page Tableau de bord, sélectionnez **Tableau de bord Machine virtuelle simple**.

   ![Accéder au tableau de bord Machine virtuelle simple](media/quickstart-portal-dashboard-powershell/select-simple-vm-dashboard.png)

1. Vérifiez le tableau de bord. Vous pouvez voir qu’une partie du contenu est statique, mais il y a également des graphiques qui indiquent les performances de la machine virtuelle.

   ![Examiner le tableau de bord Machine virtuelle simple](media/quickstart-portal-dashboard-powershell/review-simple-vm-dashboard.png)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer la machine virtuelle et le tableau de bord associé, supprimez le groupe de ressources qui les contient.

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurecli
az group delete --name myResourceGroup
```

Pour supprimer uniquement le tableau de bord, utilisez la commande [az portal dashboard delete](/cli/azure/ext/portal/portal/dashboard#ext_portal_az_portal_dashboard_delete) :

```azurecli
az portal dashboard delete --resource-group myResourceGroup --name "Simple VM Dashboard"
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la prise en charge d’Azure CLI pour les tableaux de bord, consultez [az portal dashboard](/cli/azure/ext/portal/portal/dashboard).

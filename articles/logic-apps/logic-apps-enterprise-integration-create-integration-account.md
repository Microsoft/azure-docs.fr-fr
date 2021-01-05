---
title: Créer ou gérer des comptes d’intégration B2B
description: Créer, lier et gérer des comptes d’intégration pour l’intégration d’entreprise avec Azure Logic Apps
services: logic-apps
ms.suite: integration
author: divyaswarnkar
ms.author: divswa
ms.reviewer: estfan, logicappspm
ms.topic: conceptual
ms.date: 11/04/2020
ms.openlocfilehash: 191832ab227e854b40938183e335c1b6ea52199c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96000049"
---
# <a name="create-and-manage-integration-accounts-for-b2b-enterprise-integrations-in-azure-logic-apps"></a>Créer et gérer des comptes d’intégration pour l’intégration d’entreprise B2B dans Azure Logic Apps

Avant de pouvoir créer des [solutions d’intégration d’entreprise et B2B](../logic-apps/logic-apps-enterprise-integration-overview.md) à l’aide d’[Azure Logic Apps](../logic-apps/logic-apps-overview.md), vous devez créer un compte d’intégration, qui est une ressource Azure distincte offrant une solution sécurisée et évolutive ainsi qu’un conteneur gérable pour les artefacts d’intégration que vous définissez et utilisez avec vos flux de travail d’application logique.

Par exemple, vous pouvez créer, stocker et gérer des artefacts B2B, tels que les partenaires commerciaux, les contrats, les mappages, les schémas, les certificats et les configurations de lots. En outre, pour que votre application logique puisse utiliser ces artefacts et les connecteurs Logic Apps B2B, vous devez [associer votre compte d’intégration](#link-account) à votre application logique. Votre compte d’intégration et votre application logique doivent tous deux se trouver dans le *même* emplacement ou région.

> [!TIP]
> Pour créer un compte d’intégration dans un [environnement de service d'intégration](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), consultez [Créer des comptes d’intégration dans un environnement de service d’intégration](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment).

Cette rubrique explique comment effectuer ces tâches :

* Créez votre compte d’intégration.
* Liez votre compte d’intégration à une application logique.
* Modifiez le niveau tarifaire de votre compte d’intégration.
* Supprimez la liaison de votre compte d’intégration à partir d’une application logique.
* Déplacez votre compte d’intégration vers un autre groupe de ressources ou abonnement Azure.
* Supprimez votre compte d’intégration.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [inscrivez-vous pour bénéficier d’un compte Azure gratuit](https://azure.microsoft.com/free/).

## <a name="create-integration-account"></a>Créer un compte d’intégration

### <a name="portal"></a>[Portail](#tab/azure-portal)

Pour cette tâche, vous pouvez utiliser le Portail Azure en suivant les étapes de cette section [, Azure PowerShell](/powershell/module/Az.LogicApp/New-AzIntegrationAccount)ou [Azure CLI](/cli/azure/resource#az-resource-create).

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans le menu principal d’Azure, choisissez **Créer une ressource**. Dans la zone de recherche, entrez « compte intégration » comme filtre, puis sélectionnez **Compte d’intégration**.

   ![Créer un nouveau compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/create-integration-account.png)

1. Sous **Compte d’intégration**, sélectionnez **Créer**.

   ![Choisissez « Ajouter » pour créer un compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/add-integration-account.png)

1. Fournissez ces informations sur votre compte d’intégration :

   ![Fournir des détails sur le compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-details.png)

   | Propriété | Obligatoire | Valeur | Description |
   |----------|----------|-------|-------------|
   | **Nom** | Oui | <*integration-account-name*> | Nom de votre compte d’intégration, qui peut contenir uniquement des lettres, des chiffres, des traits d’union (`-`) des traits de soulignement (`_`), des parenthèses (`(`, `)`) et des points (`.`). Cet exemple utilise « Fabrikam-Integration ». |
   | **Abonnement** | Oui | <*Azure-subscription-name*> | Nom de votre abonnement Azure. |
   | **Groupe de ressources** | Oui | <*nom-groupe-de-ressources-Azure*> | Le nom du [groupe de ressources Azure](../azure-resource-manager/management/overview.md) à utiliser pour organiser les ressources connexes. Pour cet exemple, créez un nouveau groupe de ressources nommé « FabrikamIntegration-RG ». |
   | **Niveau tarifaire** | Oui | <*pricing-level*> | Niveau tarifaire pour le compte d’intégration, que vous pouvez modifier par la suite. Dans cet exemple, sélectionnez **Gratuit**. Pour plus d’informations, consultez les rubriques suivantes : <p>- [Modèle de tarification de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts) <p>- [Limites et configuration de Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits) <p>- [Tarification de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/) |
   | **Lieu** | Oui | <*Azure-region*> | Région dans laquelle stocker les métadonnées de votre compte d’intégration. Sélectionnez l’emplacement de votre application logique ou créez vos applications logiques au même emplacement que votre compte d’intégration. Pour cet exemple, utilisez « USA Ouest ». <p>**Remarque** : Pour créer un compte d’intégration dans [un Environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), sélectionnez cet ISE comme emplacement. Pour plus d’informations, consultez [Créer des comptes d’intégration dans un environnement ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment). |
   | **Log Analytics** | Non | Off, On | Conservez le paramètre **Off** pour cet exemple. |
   |||||

1. Quand vous avez terminé, sélectionnez **Créer**.

   Une fois le déploiement terminé, Azure ouvre votre compte d’intégration.

   ![Azure ouvre le compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-created.png)

1. Pour que votre application logique puisse utiliser votre compte d’intégration, suivez les étapes suivantes pour lier ensemble votre compte d’intégration et votre application logique.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Vous pouvez créer un compte d’intégration à l’aide des commandes Azure CLI de cette section.

[!INCLUDE [azure-cli-prepare-your-environment-h3.md](../../includes/azure-cli-prepare-your-environment-h3.md)]

### <a name="create-an-integration-account"></a>Création d’un compte d’intégration

Utilisez ces commandes pour créer un compte d’intégration.

1. Pour ajouter l’extension [az logic integration-account](/cli/azure/ext/logic/logic/integration-account), utilisez la commande [az extension add](/cli/azure/extension#az_extension_add) :

   ```azurecli
   az extension add –-name logic
   ```

1. Pour créer un groupe de ressources ou utiliser un groupe de ressources existant, exécutez la commande [az group create](/cli/azure/group#az_group_create) :

   ```azurecli
   az group create --name myresourcegroup --location westus
   ```

   Pour répertorier les comptes d’intégration d’un groupe de ressources, utilisez la commande [az logic integration-account list](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_list) :

   ```azurecli
   az logic integration-account list --resource-group myresourcegroup
   ```

1. Pour créer un compte d’intégration, exécutez la commande [az logic integration-account create](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_create) :

   ```azurecli
   az logic integration-account create --resource-group myresourcegroup \
       --name integration_account_01 --location westus --sku name=Standard
   ```

   Nom de votre compte d’intégration, qui peut contenir uniquement des lettres, des chiffres, des traits d’union [-] des traits de soulignement [ _ ], des parenthèses [(, )] et des points [.].

   > [!TIP]
   > Pour créer un compte d’intégration dans [un Environnement de service d’intégration (ISE)](../logic-apps/connect-virtual-network-vnet-isolated-environment-overview.md), sélectionnez cet ISE comme emplacement. Pour plus d’informations, consultez [Créer des comptes d’intégration dans un environnement ISE](../logic-apps/add-artifacts-integration-service-environment-ise.md#create-integration-account-environment).

   Pour afficher un compte d’intégration spécifique, utilisez la commande [az logic integration-account show](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_show) :

   ```azurecli
   az logic integration-account show --name integration_account_01 --resource-group myresourcegroup
   ```

   Vous pouvez modifier votre SKU ou le niveau tarifaire à l’aide de la commande [az logic integration-account update](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_update) :

   ```azurecli
   az logic integration-account update --sku name=Basic --name integration_account_01 \
       --resource-group myresourcegroup
   ```

   Pour plus d’informations sur la tarification, consultez ces ressources :

   * [Modèle de tarification de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts)
   * [Limites et configuration de Logic Apps](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits)
   * [Tarification de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/)

Pour importer un compte d’intégration à l’aide d’un fichier JSON, utilisez la commande [az logic integration-account import](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_import) :

```azurecli
az logic integration-account import --name integration_account_01 \
    --resource-group myresourcegroup --input-path integration.json
```

Vous pouvez supprimer un compte d’intégration à l’aide de la commande [az logic integration-account delete](/cli/azure/ext/logic/logic/integration-account#ext_logic_az_logic_integration_account_delete) :

```azurecli
az logic integration-account delete --name integration_account_01 --resource-group myresourcegroup
```

Pour que votre application logique puisse utiliser votre compte d’intégration, liez ensemble votre compte d’intégration et votre application logique. La section suivante décrit la liaison.

---
<a name="link-account"></a>

## <a name="link-to-logic-app"></a>Créer un lien vers l’application logique

Pour donner à vos applications logiques l’accès à un compte d’intégration contenant vos artefacts B2B, vous devez d’abord lier votre compte d’intégration à votre application logique. L’application logique et le compte d’intégration doivent être présents dans la même région. Pour effectuer cette tâche, vous pouvez utiliser le portail Azure. Si vous utilisez Visual Studio et que votre application logique se trouve dans un [projet Azure Resource Group](../azure-resource-manager/templates/create-visual-studio-deployment-project.md), vous pouvez [lier votre application logique à un compte d’intégration avec Visual Studio](../logic-apps/manage-logic-apps-with-visual-studio.md#link-integration-account).

1. Dans le portail Azure, recherchez et sélectionnez votre application logique.

1. Dans le [Portail Azure](https://portal.azure.com), ouvrez une application logique existante ou créez une nouvelle application logique.

1. Dans le menu de votre application logique, sous **Paramètres**, sélectionnez **Paramètres de flux de travail**. Sous **Compte d’intégration**, ouvrez la liste **Sélectionner un compte d’intégration**. Sélectionnez le compte d’intégration à lier à votre application logique.

   ![Sélectionnez votre compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/select-integration-account.png)

1. Pour terminer la liaison, sélectionnez **Enregistrer**.

   ![Capture d’écran montrant où sélectionner Enregistrer pour choisir votre compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/save-link.png)

   Une fois votre compte d’intégration correctement associé, Azure affiche un message de confirmation.

   ![Azure confirme la réussite de la liaison](./media/logic-apps-enterprise-integration-create-integration-account/link-confirmation.png)

Désormais, votre application logique peut utiliser les artefacts de votre compte d’intégration en plus des connecteurs B2B, tels que la validation XML et l’encodage ou le décodage de fichiers plats.  

<a name="change-pricing-tier"></a>

## <a name="change-pricing-tier"></a>Changer le niveau tarifaire

Pour augmenter les [limites](../logic-apps/logic-apps-limits-and-config.md#integration-account-limits) d’un compte d’intégration, vous pouvez [procéder à une mise à niveau vers un niveau tarifaire supérieur](#upgrade-pricing-tier), le cas échéant. Par exemple, vous pouvez effectuer une mise à niveau à partir du niveau gratuit vers le niveau De base ou Standard. Vous pouvez également [passer à un niveau inférieur](#downgrade-pricing-tier), le cas échéant. Pour plus d’informations sur les prix, consultez les rubriques suivantes :

* [Tarification de Logic Apps](https://azure.microsoft.com/pricing/details/logic-apps/)
* [Modèle de tarification de Logic Apps](../logic-apps/logic-apps-pricing.md#integration-accounts)

<a name="upgrade-pricing-tier"></a>

### <a name="upgrade-pricing-tier"></a>Mettre à niveau le niveau tarifaire

Pour effectuer cette modification, vous pouvez utiliser le portail Azure ou l’interface de ligne de commande Azure.

#### <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche principale d’Azure, entrez « comptes intégration » comme filtre, puis sélectionnez **Comptes d’intégration**.

   ![Chercher le compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure affiche tous les comptes d’intégration dans vos abonnements Azure.

1. Sous **Comptes d’intégration**, sélectionnez le compte d’intégration à déplacer. Dans le menu de votre compte d’intégration, sélectionnez **Vue d’ensemble**.

   ![Dans le menu du compte d’intégration, sélectionnez « Vue d’ensemble »](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. Dans le volet Vue d’ensemble , sélectionnez **Mettre à niveau le niveau tarifaire**, ce qui répertorie tous les niveaux supérieurs disponibles. Lorsque vous sélectionnez un niveau, la modification prend effet immédiatement.

<a name="upgrade-tier-azure-cli"></a>

#### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

1. Si ce n’est pas déjà fait, [installez les composants requis de l’interface de ligne de commande Azure](/cli/azure/get-started-with-azure-cli).

1. Dans le portail Azure, ouvrez l’environnement [Azure Cloud Shell](../cloud-shell/overview.md).

   ![Ouvrir Azure Cloud Shell](./media/logic-apps-enterprise-integration-create-integration-account/open-azure-cloud-shell-window.png)

1. À l’invite de commandes, entrez la commande [**az resource**](/cli/azure/resource#az-resource-update), puis définissez `skuName` sur le niveau le plus élevé souhaité.

   ```azurecli
   az resource update --resource-group {ResourceGroupName} --resource-type Microsoft.Logic/integrationAccounts --name {IntegrationAccountName} --subscription {AzureSubscriptionID} --set sku.name={SkuName}
   ```
  
   Par exemple, si vous avez le niveau De base, vous pouvez définir `skuName` sur `Standard` :

   ```azurecli
   az resource update --resource-group FabrikamIntegration-RG --resource-type Microsoft.Logic/integrationAccounts --name Fabrikam-Integration --subscription XXXXXXXXXXXXXXXXX --set sku.name=Standard
   ```

---

<a name="downgrade-pricing-tier"></a>

### <a name="downgrade-pricing-tier"></a>Passage à un niveau tarifaire inférieur

Pour effectuer cette modification, utilisez l’[interface de ligne de commande Azure](/cli/azure/get-started-with-azure-cli).

1. Si ce n’est pas déjà fait, [installez les composants requis de l’interface de ligne de commande Azure](/cli/azure/get-started-with-azure-cli).

1. Dans le portail Azure, ouvrez l’environnement [Azure Cloud Shell](../cloud-shell/overview.md).

   ![Ouvrir Azure Cloud Shell](./media/logic-apps-enterprise-integration-create-integration-account/open-azure-cloud-shell-window.png)

1. À l’invite de commandes, entrez la commande [**az resource**](/cli/azure/resource#az-resource-update), puis définissez `skuName` sur le niveau le plus bas souhaité.

   ```azurecli
   az resource update --resource-group <resourceGroupName> --resource-type Microsoft.Logic/integrationAccounts --name <integrationAccountName> --subscription <AzureSubscriptionID> --set sku.name=<skuName>
   ```
  
   Par exemple, si vous avez le niveau Standard, vous pouvez définir `skuName` sur `Basic` :

   ```azurecli
   az resource update --resource-group FabrikamIntegration-RG --resource-type Microsoft.Logic/integrationAccounts --name Fabrikam-Integration --subscription XXXXXXXXXXXXXXXXX --set sku.name=Basic
   ```

## <a name="unlink-from-logic-app"></a>Dissocier de l’application logique

Si vous voulez lier votre application logique à un autre compte d’intégration, ou ne plus utiliser un compte d’intégration avec votre application logique, supprimez la liaison à l’aide d’Azure Resource Explorer.

1. Ouvrez la fenêtre de votre navigateur, puis accédez à [Azure Resource Explorer (https://resources.azure.com)](https://resources.azure.com). Connectez-vous avec les mêmes informations d’identification de votre compte Azure.

   ![Azure Resource Explorer](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer.png)

1. Dans la zone de recherche, entrez le nom de l’application de votre logique, de façon à pouvoir rechercher et sélectionner votre application logique.

   ![Rechercher et sélectionner l’application logique](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-find-logic-app.png)

1. Dans la barre de titre de l’explorateur, sélectionnez **Lecture/écriture**.

   ![Activer le mode « Lecture/écriture »](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-select-read-write.png)

1. Sous l’onglet **Données**, sélectionnez **Modifier**.

   ![Sous l’onglet « Données », sélectionnez « Modifier »](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-select-edit.png)

1. Dans l’éditeur, recherchez l’objet `integrationAccount`, puis supprimez-le. Son format est le suivant :

   ```json
   {
      // <other-attributes>
      "integrationAccount": {
         "name": "<integration-account-name>",
         "id": "<integration-account-resource-ID>",
         "type": "Microsoft.Logic/integrationAccounts"  
   },
   ```

   Par exemple :

   ![Rechercher l’objet « integrationAccount »](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-delete-integration-account.png)

1. Sous l’onglet **Données**, sélectionnez **Put** pour enregistrer vos modifications.

   ![Pour enregistrer les modifications, sélectionnez « Put »](./media/logic-apps-enterprise-integration-create-integration-account/resource-explorer-save-changes.png)

1. Dans le Portail Azure, recherchez et sélectionnez votre application logique. Sous les **Paramètres de flux de travail** de votre application, vérifiez que la propriété **Compte d’intégration** apparaît désormais vide.

   ![Vérifier que le compte d’intégration n’est pas lié](./media/logic-apps-enterprise-integration-create-integration-account/unlinked-account.png)

## <a name="move-integration-account"></a>Déplacer un compte d’intégration

Vous pouvez déplacer votre compte d’intégration vers un autre groupe de ressources Azure ou un abonnement Azure. Lorsque vous déplacez des ressources, Azure crée de nouveaux ID de ressource. Veillez donc à utiliser les nouveaux ID et à mettre à jour tous les scripts ou outils associés aux ressources déplacées. Si vous souhaitez modifier l’abonnement, vous devez également spécifier un groupe de ressources nouveau ou existant.

Pour cette tâche, vous pouvez utiliser le Portail Azure en suivant les étapes décrites dans cette section ou l’[interface de ligne de commande Azure](/cli/azure/resource#az-resource-move).

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche principale d’Azure, entrez « comptes intégration » comme filtre, puis sélectionnez **Comptes d’intégration**.

   ![Chercher le compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure affiche tous les comptes d’intégration dans vos abonnements Azure.

1. Sous **Comptes d’intégration**, sélectionnez le compte d’intégration à déplacer. Dans le menu de votre compte d’intégration, sélectionnez **Vue d’ensemble**.

   ![Dans le menu du compte d’intégration, sélectionnez « Vue d’ensemble »](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. En regard du **Groupe de ressources** ou du **Nom d’abonnement**, sélectionnez **Modifier**.

   ![Modifier le groupe de ressources ou l’abonnement](./media/logic-apps-enterprise-integration-create-integration-account/change-resource-group-subscription.png)

1. Sélectionnez les ressources associées que vous souhaitez également déplacer.

1. En fonction de votre sélection, procédez comme suit pour modifier le groupe de ressources ou l’abonnement :

   * Groupe de ressources : Dans la liste **Groupe de ressources**, sélectionnez le groupe de ressources de destination. Ou, pour créer un groupe de ressources différent, sélectionnez **Créer un groupe de ressources**.

   * Abonnement : Dans la liste **Abonnement**, sélectionnez l’abonnement de destination. Dans la liste **Groupe de ressources**, sélectionnez le groupe de ressources de destination. Ou, pour créer un groupe de ressources différent, sélectionnez **Créer un groupe de ressources**.

1. Pour confirmer que tous les scripts ou outils associés aux ressources déplacées ne fonctionneront pas tant que vous ne les aurez pas mis à jour avec les nouveaux ID de ressource, sélectionnez la zone de confirmation, puis sélectionnez **OK**.

1. Une fois que vous avez terminé, veillez à mettre à jour tous les scripts avec les nouveaux ID de ressource pour vos ressources déplacées.  

## <a name="delete-integration-account"></a>Supprimer un compte d’intégration

Pour cette tâche, vous pouvez utiliser le Portail Azure en suivant les étapes de cette section, l’[interface de ligne de commande Azure](/cli/azure/resource#az-resource-delete) ou [Azure PowerShell](/powershell/module/az.logicapp/remove-azintegrationaccount).

1. Connectez-vous au [portail Azure](https://portal.azure.com) avec les informations d’identification de votre compte Azure.

1. Dans la zone de recherche principale d’Azure, entrez « comptes intégration » comme filtre, puis sélectionnez **Comptes d’intégration**.

   ![Chercher le compte d’intégration](./media/logic-apps-enterprise-integration-create-integration-account/find-integration-account.png)

   Azure affiche tous les comptes d’intégration dans vos abonnements Azure.

1. Sous **Comptes d’intégration**, sélectionnez le compte d’intégration à supprimer. Dans le menu de votre compte d’intégration, sélectionnez **Vue d’ensemble**.

   ![Dans le menu du compte d’intégration, sélectionnez « Vue d’ensemble »](./media/logic-apps-enterprise-integration-create-integration-account/integration-account-overview.png)

1. Dans le volet Vue d’ensemble, sélectionnez **Supprimer**.

   ![Dans le volet « Vue d’ensemble », sélectionnez « Supprimer »](./media/logic-apps-enterprise-integration-create-integration-account/delete-integration-account.png)

1. Pour confirmer que vous souhaitez supprimer votre compte d’intégration, sélectionnez **Oui**.

   ![Pour confirmer la suppression, sélectionnez « Oui »](./media/logic-apps-enterprise-integration-create-integration-account/confirm-delete.png)

## <a name="next-steps"></a>Étapes suivantes

* [Créer des partenaires commerciaux dans votre compte d’intégration](../logic-apps/logic-apps-enterprise-integration-partners.md)
* [Créer des accords entre partenaires dans votre compte d’intégration](../logic-apps/logic-apps-enterprise-integration-agreements.md)

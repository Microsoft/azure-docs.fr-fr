---
title: 'Tutoriel : Créer et gérer des données exportées depuis Azure Cost Management'
description: Cet article vous montre comment créer et gérer des données Azure Cost Management exportées pour les utiliser dans des systèmes externes.
author: bandersmsft
ms.author: banders
ms.date: 12/7/2020
ms.topic: tutorial
ms.service: cost-management-billing
ms.subservice: cost-management
ms.reviewer: adwise
ms.custom: seodec18, devx-track-azurepowershell
ms.openlocfilehash: e3c1fa071cd23b871f754e89d6f17eb2cc44b394
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400350"
---
# <a name="tutorial-create-and-manage-exported-data"></a>Tutoriel : Créer et gérer des données exportées

Si vous avez lu le tutoriel Analyse du coût, vous êtes familiarisé avec le téléchargement manuel de vos données Cost Management. Cependant, vous pouvez créer une tâche récurrente qui exporte automatiquement sur une base quotidienne, hebdomadaire ou mensuelle vos données Cost Management dans un stockage Azure. Les données exportées sont au format CSV, et elles contiennent toutes les informations collectées par Cost Management. Vous pouvez ensuite utiliser les données exportées dans Stockage Azure avec des systèmes externes et les combiner avec vos propres données personnalisées. Vous pouvez aussi utiliser vos données exportées dans un système externe, comme un tableau de bord ou un autre système financier.

Regardez la vidéo [Comment planifier des exportations à des fins de stockage avec Azure Cost Management](https://www.youtube.com/watch?v=rWa_xI1aRzo) pour créer une exportation planifiée de votre données de coût Azure vers Stockage Azure. Pour regarder d’autres vidéos, consultez la [chaîne YouTube relative à Cost Management](https://www.youtube.com/c/AzureCostManagement).

>[!VIDEO https://www.youtube.com/embed/rWa_xI1aRzo]

Les exemples de ce tutoriel montrent comment exporter vos données de gestion des coûts, puis comment vérifier que les données ont été exportées correctement.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Créer une exportation quotidienne
> * Vérifier que les données sont collectées

## <a name="prerequisites"></a>Prérequis
L’exportation des données est disponible pour divers types de comptes Azure, notamment pour les clients [Contrat Entreprise (EA)](https://azure.microsoft.com/pricing/enterprise-agreement/) et [Contrat client Microsoft](get-started-partners.md). Pour accéder à la liste complète des types de comptes pris en charge, voir [Comprendre les données de Cost Management](understand-cost-mgt-data.md). Les autorisations Azure suivantes, ou étendues, sont prises en charge par abonnement pour l’exportation de données par utilisateur et par groupe. Pour plus d’informations sur les étendues, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).

- Propriétaire : peut créer, modifier ou supprimer des exportations planifiées pour un abonnement.
- Contributeur : peut créer, modifier ou supprimer ses propres exportations planifiées. Peut modifier le nom d’exportations planifiées créées par d’autres utilisateurs.
- Lecteur : peut planifier des exportations pour lesquelles il dispose des autorisations adéquates.

Pour les comptes Stockage Azure :
- Des autorisations d’écriture sont requises pour la modification du compte de stockage configuré, quelles que soient les autorisations sur l’exportation.
- Votre compte de stockage Azure doit être configuré pour le stockage d’objets blob ou de fichiers.

Si vous disposez d’un nouvel abonnement, vous ne pouvez pas utiliser les fonctionnalités de Cost Management tout de suite. Vous risquez de devoir attendre jusqu’à 48 heures avant de pouvoir utiliser toutes les fonctionnalités de Cost Management.

## <a name="sign-in-to-azure"></a>Connexion à Azure
Connectez-vous au portail Azure sur [https://portal.azure.com](https://portal.azure.com/).

## <a name="create-a-daily-export"></a>Créer une exportation quotidienne

### <a name="portal"></a>[Portail](#tab/azure-portal)

Pour créer, afficher ou planifier une exportation de données, ouvrez l’étendue souhaitée dans le portail Azure et sélectionnez **Analyse du coût** dans le menu. Par exemple, accédez à **Abonnements**, sélectionnez un abonnement dans la liste, puis sélectionnez **Analyse du coût** dans le menu. En haut de la page Analyse des coûts, sélectionnez **Paramètres**, puis **Exportations**.

> [!NOTE]
> - Vous pouvez non seulement créer des exportations sur des abonnements, mais aussi sur des groupes de ressources, des groupes d'administration, des services et des inscriptions. Pour plus d’informations sur les étendues, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).
>- Quand vous êtes connecté en tant que partenaire dans l’étendue du compte de facturation ou sur le locataire d’un client, vous pouvez exporter des données vers un compte de stockage Azure lié à votre compte de stockage partenaire. Toutefois, vous devez disposer d’un abonnement actif dans votre locataire CSP.

1. Sélectionnez **Ajouter** et entrez un nom pour l'exportation.
1. Pour la **Métrique**, effectuez une sélection :
    - **Coût réel (utilisation et achats)**  : sélectionnez cette option pour exporter l'utilisation standard et les achats.
    - **Coût amorti (utilisation et achats)**  : sélectionnez cette option pour exporter les coûts amortis pour les achats tels que les réservations Azure.
1. Pour le **Type d'exportation**, effectuez une sélection :
    - **Exportation quotidienne des coûts en cumul mensuel à ce jour** : fournit quotidiennement un nouveau fichier d'exportation pour vos coûts en cumul mensuel à ce jour. Les dernières données sont agrégées avec les exportations quotidiennes précédentes.
    - **Exportation hebdomadaire des coûts pour les sept derniers jours** : crée une exportation hebdomadaire de vos coûts pour les sept derniers jours à compter de la date de début d’exportation sélectionnée.
    - **Exportation mensuelle des coûts du mois précédent** : fournit une exportation des coûts du mois précédent par rapport au mois en cours. Ensuite, le calendrier exécute une exportation le cinquième jour de chaque nouveau mois avec vos coûts des mois précédents.
    - **Exportation unique** : vous permet de choisir une plage de dates pour les données historiques à exporter vers le service Stockage Blob Azure. Vous pouvez exporter un maximum de 90 jours de coûts historiques à partir du jour de votre choix. Cette exportation s'exécute immédiatement et est disponible sur votre compte de stockage dans les deux heures.
        En fonction du type d'exportation, choisissez une date de début ou une date au format **De** et **À**.
1. Spécifiez l'abonnement associé à votre compte de stockage Azure, puis sélectionnez un groupe de ressources ou créez-en un.
1. Sélectionnez le nom du compte de stockage ou créez-en un.
1. Sélectionnez l'emplacement (région Azure).
1. Spécifiez le conteneur de stockage et le chemin du répertoire que vous souhaitez utiliser pour le fichier d’exportation.
    :::image type="content" source="./media/tutorial-export-acm-data/basics_exports.png" alt-text="Nouvel exemple d'exportation" lightbox="./media/tutorial-export-acm-data/basics_exports.png":::
1. Vérifiez vos informations d’exportation, puis sélectionnez **Créer**.

Votre nouvelle exportation apparaît dans la liste des exportations. Par défaut, les nouvelles exportations sont activées. Si vous voulez désactiver ou supprimer une exportation planifiée, sélectionnez n’importe quel élément de la liste, puis sélectionnez **Désactiver** ou **Supprimer**.

Au départ, l'exportation peut prendre 12 à 24 heures. Mais l'affichage des données dans les fichiers exportés peut prendre plus de temps.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Commencez par préparer votre environnement pour Azure CLI :

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../../includes/azure-cli-prepare-your-environment-no-header.md)]

1. Une fois connecté, pour voir vos exportations en cours, utilisez la commande [az costmanagement export list](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_list) :

   ```azurecli
   az costmanagement export list --scope "subscriptions/00000000-0000-0000-0000-000000000000"
   ```

   >[!NOTE]
   >
   >* Vous pouvez créer des exportations non seulement pour des abonnements, mais aussi pour des groupes de ressources et des groupes d’administration. Pour plus d’informations sur les étendues, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).
   >* Quand vous êtes connecté en tant que partenaire dans l’étendue du compte de facturation ou sur le locataire d’un client, vous pouvez exporter des données vers un compte de stockage Azure lié à votre compte de stockage partenaire. Toutefois, vous devez disposer d’un abonnement actif dans votre locataire CSP.

1. Créez un groupe de ressources ou utilisez un groupe existant. Pour créer un groupe de ressources, utilisez la commande [az group create](/cli/azure/group#az_group_create) :

   ```azurecli
   az group create --name TreyNetwork --location "East US"
   ```

1. Créez un compte de stockage pour recevoir les exportations, ou utilisez un compte de stockage existant. Pour créer un compte de stockage, utilisez la commande [az storage account create](/cli/azure/storage/account#az_storage_account_create) :

   ```azurecli
   az storage account create --resource-group TreyNetwork --name cmdemo
   ```

1. Exécutez la commande [az costmanagement export create](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_create) pour créer l’exportation :

   ```azurecli
   az costmanagement export create --name DemoExport --type ActualCost \
   --scope "subscriptions/00000000-0000-0000-0000-000000000000" --storage-account-id cmdemo \
   --storage-container democontainer --timeframe MonthToDate --recurrence Daily \
   --recurrence-period from="2020-06-01T00:00:00Z" to="2020-10-31T00:00:00Z" \
   --schedule-status Active --storage-directory demodirectory
   ```

   Pour le paramètre **--type**, vous pouvez choisir `ActualCost`, `AmortizedCost` ou `Usage`.

   Cet exemple utilise `MonthToDate`. L’exportation crée un fichier d’exportation tous les jours pour les coûts du mois en cours. Les dernières données sont ajoutées aux exportations quotidiennes précédentes pour le mois en question.

1. Pour afficher les détails de votre opération d’exportation, utilisez la commande [az costmanagement export show](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_show) :

   ```azurecli
   az costmanagement export show --name DemoExport \
      --scope "subscriptions/00000000-0000-0000-0000-000000000000"
   ```

1. Pour mettre une exportation à jour, exécutez la commande [az costmanagement export update](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_update) :

   ```azurecli
   az costmanagement export update --name DemoExport
      --scope "subscriptions/00000000-0000-0000-0000-000000000000" --storage-directory demodirectory02
   ```

   Cet exemple change le répertoire de sortie.

>[!NOTE]
>Au départ, l'exportation peut prendre 12 à 24 heures. Mais l’affichage des données dans les fichiers exportés peut prendre plus de temps.

Vous pouvez supprimer une exportation à l’aide de la commande [az costmanagement export delete](/cli/azure/ext/costmanagement/costmanagement/export#ext_costmanagement_az_costmanagement_export_delete) :

```azurecli
az costmanagement export delete --name DemoExport --scope "subscriptions/00000000-0000-0000-0000-000000000000"
```

### <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

Commencez par préparer votre environnement pour Azure PowerShell :

[!INCLUDE [azure-powershell-requirements-no-header.md](../../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Tant que le module PowerShell **Az.CostManagement** est en préversion, vous devez l’installer séparément à l’aide de l’applet de commande `Install-Module`. Une fois que ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module PowerShell Az et sera disponible par défaut dans Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.CostManagement
  ```

1. Une fois connecté, pour voir vos exportations en cours, utilisez l’applet de commande [Get-AzCostManagementExport](/powershell/module/Az.CostManagement/get-azcostmanagementexport) :

   ```azurepowershell-interactive
   Get-AzCostManagementExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
   ```

   >[!NOTE]
   >
   >* Vous pouvez créer des exportations non seulement pour des abonnements, mais aussi pour des groupes de ressources et des groupes d’administration. Pour plus d’informations sur les étendues, consultez [Comprendre et utiliser les étendues](understand-work-scopes.md).
   >* Quand vous êtes connecté en tant que partenaire dans l’étendue du compte de facturation ou sur le locataire d’un client, vous pouvez exporter des données vers un compte de stockage Azure lié à votre compte de stockage partenaire. Toutefois, vous devez disposer d’un abonnement actif dans votre locataire CSP.

1. Créez un groupe de ressources ou utilisez un groupe existant. Pour créer un groupe de ressources, utilisez l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) :

   ```azurepowershell-interactive
   New-AzResourceGroup -Name TreyNetwork -Location eastus
   ```

1. Créez un compte de stockage pour recevoir les exportations, ou utilisez un compte de stockage existant. Pour créer un compte de stockage, utilisez l’applet de commande [New-AzStorageAccount](/powershell/module/az.storage/new-azstorageaccount) :

   ```azurepowershell-interactive
   New-AzStorageAccount -ResourceGroupName TreyNetwork -AccountName cmdemo -SkuName Standard_RAGRS -Location eastus
   ```

1. Exécutez l’applet de commande [New-AzCostManagementExport](/powershell/module/Az.CostManagement/new-azcostmanagementexport) pour créer l’exportation :

   ```azurepowershell-interactive
   $Params = @{
     Name = 'DemoExport'
     DefinitionType = 'ActualCost'
     Scope = 'subscriptions/00000000-0000-0000-0000-000000000000'
     DestinationResourceId = '/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/treynetwork/providers/Microsoft.Storage/storageAccounts/cmdemo'
     DestinationContainer = 'democontainer'
     DefinitionTimeframe = 'MonthToDate'
     ScheduleRecurrence = 'Daily'
     RecurrencePeriodFrom = '2020-06-01T00:00:00Z'
     RecurrencePeriodTo = '2020-10-31T00:00:00Z'
     ScheduleStatus = 'Active'
     DestinationRootFolderPath = 'demodirectory'
     Format = 'Csv'
   }
   New-AzCostManagementExport @Params
   ```

   Pour le paramètre **DefinitionType**, vous pouvez choisir `ActualCost`, `AmortizedCost` ou `Usage`.

   Cet exemple utilise `MonthToDate`. L’exportation crée un fichier d’exportation tous les jours pour les coûts du mois en cours. Les dernières données sont ajoutées aux exportations quotidiennes précédentes pour le mois en question.

1. Pour afficher les détails de votre opération d’exportation, utilisez l’applet de commande `Get-AzCostManagementExport` :

   ```azurepowershell-interactive
   Get-AzCostManagementExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
   ```

1. Mettez à jour une exportation en utilisant l’applet de commande [Update-AzCostManagementExport](/powershell/module/Az.CostManagement/update-azcostmanagementexport) :

   ```azurepowershell-interactive
   Update-AzCostManagementExport -Name DemoExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000' -DestinationRootFolderPath demodirectory02
   ```

   Cet exemple change le répertoire de sortie.

>[!NOTE]
>Au départ, l'exportation peut prendre 12 à 24 heures. Mais l’affichage des données dans les fichiers exportés peut prendre plus de temps.

Vous pouvez supprimer une exportation au moyen de l’applet de commande [Remove-AzCostManagementExport](/powershell/module/Az.CostManagement/remove-azcostmanagementexport) :

```azurepowershell-interactive
Remove-AzCostManagementExport -Name DemoExport -Scope 'subscriptions/00000000-0000-0000-0000-000000000000'
```

---

### <a name="export-schedule"></a>Planification des exportations

Les exportations planifiées dépendent de l’heure et du jour de la semaine de la création initiale des exportations. Quand vous créez une exportation planifiée, chacune de ses occurrences suivantes s’exécute à la même fréquence. Par exemple, pour une exportation des coûts en cumul mensuel à ce jour définie sur une fréquence quotidienne, l'exportation s'exécute tous les jours. De même, pour une exportation hebdomadaire, l’exportation s’exécute toutes les semaines le même jour que celui planifié. Le temps de remise exact de l'exportation n'est pas garanti et les données exportées sont disponibles dans un délai de quatre heures.

Chaque exportation crée un fichier, ce qui signifie que les exportations antérieures ne sont pas écrasées.

#### <a name="create-an-export-for-multiple-subscriptions"></a>Création d’une exportation pour plusieurs abonnements

Si vous possédez un Accord Entreprise, vous pouvez utiliser un groupe d’administration pour agréger les informations sur les coûts d’abonnement dans un seul conteneur. Vous pouvez ensuite exporter les données de gestion des coûts pour le groupe d’administration.

Les exportations pour les groupes d’administration d’autres types d’abonnement ne sont pas prises en charge.

1. Si vous n'avez pas encore créé de groupe d'administration, créez-en un et attribuez-lui des abonnements.
1. Dans l'analyse des coûts, définissez l'étendue de votre groupe d'administration et sélectionnez **Sélectionner ce groupe d'administration**.
    :::image type="content" source="./media/tutorial-export-acm-data/management-group-scope.png" alt-text="Exemple illustrant l'option Sélectionner ce groupe d'administration" lightbox="./media/tutorial-export-acm-data/management-group-scope.png":::
1. Créez une exportation selon l’étendue pour obtenir les données de gestion des coûts pour les abonnements dans le groupe d’administration.
    :::image type="content" source="./media/tutorial-export-acm-data/new-export-management-group-scope.png" alt-text="Exemple illustrant l'option Créer une exportation avec l'étendue Groupe d'administration":::

## <a name="verify-that-data-is-collected"></a>Vérifier que les données sont collectées

Vous pouvez facilement vérifier que vos données Cost Management sont collectées et visualiser le fichier CSV exporté avec l’Explorateur Stockage Azure.

Dans la liste des exportations, sélectionnez le nom du compte de stockage. Dans la page du compte de stockage, sélectionnez Ouvrir dans l’Explorateur. Si vous voyez une boîte de confirmation, sélectionnez **Oui** pour ouvrir le fichier dans l’Explorateur Stockage Azure.

![Page de compte de stockage affichant des exemples d’informations et un lien pour les ouvrir dans l’Explorateur](./media/tutorial-export-acm-data/storage-account-page.png)

Dans l’Explorateur Stockage, accédez au conteneur que vous voulez ouvrir, puis sélectionnez le dossier correspondant au mois en cours. Une liste de fichiers CSV s’affiche. Sélectionnez-en un, puis sélectionnez **Ouvrir**.

![Exemples d’informations affichées dans l’Explorateur de stockage](./media/tutorial-export-acm-data/storage-explorer.png)

Le fichier s’ouvre avec le programme ou l’application configuré pour ouvrir les fichiers avec l’extension CSV. Voici un exemple dans Excel.

![Exemples de données CSV exportées affichées dans Excel](./media/tutorial-export-acm-data/example-export-data.png)

### <a name="download-an-exported-csv-data-file"></a>Télécharger un fichier de données CSV exporté

Vous pouvez également télécharger le fichier CSV exporté dans le Portail Azure. Les étapes suivantes expliquent comment le trouver dans l’analyse des coûts.

1. Dans l’analyse des coûts, sélectionnez **Paramètres**, puis **Exportations**.
1. Dans la liste des exportations, sélectionnez le compte de stockage pour une exportation.
1. Dans votre compte de stockage, sélectionnez **Conteneurs**.
1. Dans la liste des conteneurs, sélectionnez le conteneur souhaité.
1. Parcourez les répertoires et les objets BLOB de stockage jusqu’à la date de votre choix.
1. Sélectionnez le fichier CSV, puis **Télécharger**.

[![Exemple de téléchargement d’exportation](./media/tutorial-export-acm-data/download-export.png)](./media/tutorial-export-acm-data/download-export.png#lightbox)

## <a name="view-export-run-history"></a>Consulter l'historique des exécutions des exportations

Vous pouvez consulter l'historique d'exécution de votre exportation planifiée en sélectionnant une exportation individuelle sur la page répertoriant les exportations. La page contenant la liste des exportations vous permet également d'accéder rapidement à la durée d'exécution de vos exportations précédentes et de savoir quand la prochaine exportation aura lieu. Voici un exemple illustrant l'historique des exécutions.

:::image type="content" source="./media/tutorial-export-acm-data/run-history.png" alt-text="Capture d’écran montrant le volet Exportations.":::

Sélectionnez une exportation pour afficher l'historique des exécutions de celle-ci.

:::image type="content" source="./media/tutorial-export-acm-data/single-export-run-history.png" alt-text="Capture d’écran montrant l’historique des exécutions d’une exportation.":::

## <a name="access-exported-data-from-other-systems"></a>Accéder à des données exportées à partir d’autres systèmes

Un des objectifs de l’exportation de vos données Cost Management est d’accéder à ces données à partir de systèmes externes. Vous pouvez utiliser un système de tableau de bord ou un autre système financier. Ces systèmes peuvent grandement varier : montrer un exemple ne serait donc pas pratique.  Vous pouvez cependant découvrir comment accéder à vos données à partir de vos applications dans [Introduction à Stockage Azure](../../storage/common/storage-introduction.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer une exportation quotidienne
> * Vérifier que les données sont collectées

Passez au tutoriel suivant pour optimiser et améliorer l’efficacité en identifiant les ressources inactives et sous-utilisées.

> [!div class="nextstepaction"]
> [Consulter des recommandations d’optimisation et agir en fonction](tutorial-acm-opt-recommendations.md)

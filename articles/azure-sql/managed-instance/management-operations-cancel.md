---
title: Annuler des opérations de gestion
titleSuffix: Azure SQL Managed Instance
description: Découvrez comment annuler des opérations de gestion Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, bonova, MashaMSFT
ms.date: 09/03/2020
ms.openlocfilehash: 342491178d55dacbdc68e6c9042623d381dff898
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861542"
---
# <a name="canceling-azure-sql-managed-instance-management-operations"></a>Annulation d’opérations de gestion Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance permet d’annuler certaines [opérations de gestion](management-operations-overview.md), par exemple lorsque vous déployez une nouvelle instance managée ou mettez à jour les propriétés d’une instance. 

## <a name="overview"></a>Vue d’ensemble

 Toutes les opérations de gestion peuvent être classées comme suit :

- Déploiement d’instance (création d’une nouvelle instance)
- Mise à jour d’instance (modification des propriétés, par exemple, vCore ou stockage réservé)
- Suppression d’instance

Vous pouvez [surveiller la progression et l’état des opérations de gestion](management-operations-monitor.md) et annuler certaines d’entre elles, si nécessaire. 

Le tableau suivant récapitule les opérations de gestion, que vous puissiez les annuler ou non, et leur durée globale standard :

Category  |Opération  |Annulable  |Durée d’annulation estimée  |
|---------|---------|---------|---------|
|Déploiement |Création d’instance |Oui |90 % des opérations ne prennent pas plus de cinq minutes. |
|Update |Scale-up ou scale-down du stockage d’instance (Usage général) |Non |  |
|Update |Scale-up et scale-down du stockage d’instance (Critique pour l’entreprise) |Oui |90 % des opérations ne prennent pas plus de cinq minutes. |
|Update |Scale-up et scale-down de la capacité de calcul des instances (vCores) (Usage général) |Oui |90 % des opérations ne prennent pas plus de cinq minutes. |
|Update |Scale-up et scale-down de la capacité de calcul des instances (vCores) (Critique pour l’entreprise) |Oui |90 % des opérations ne prennent pas plus de cinq minutes. |
|Update |Modification du niveau de service de l’instance (Usage général vers Critique pour l’entreprise et vice versa) |Oui |90 % des opérations ne prennent pas plus de cinq minutes. |
|DELETE |Suppression d’instance |Non |  |
|DELETE |Suppression de cluster virtuel (en tant qu’opération lancée par l’utilisateur) |Non |  |

## <a name="cancel-management-operation"></a>Annuler une opération de gestion

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour annuler des opérations de gestion à l’aide du portail Azure, procédez comme suit :

1. Accédez au [portail Azure](https://portal.azure.com).
1. Accédez au panneau **Vue d’ensemble** de votre instance managée SQL. 
1. Sélectionnez la zone **Notification** en regard de l’opération en cours pour ouvrir la page **Opération en cours**. 

   :::image type="content" source="media/management-operations-cancel/open-ongoing-operation.png" alt-text="Sélectionnez la zone de l’opération en cours pour ouvrir la page Opération en cours.":::

1. Sélectionnez **Annuler l’opération** en bas de la page. 

   :::image type="content" source="media/management-operations-cancel/cancel-operation.png" alt-text="Sélectionnez Annuler pour annuler l’opération.":::

1. Confirmez que vous voulez annuler l’opération. 


Si la demande d’annulation aboutit, l’opération de gestion est annulée et entraîne un échec. Vous recevrez une notification indiquant que l’annulation a réussi ou échoué.

![Résultat d’opération d’annulation](./media/management-operations-cancel/canceling-operation-result.png)


Si la demande d’annulation échoue ou que le bouton Annuler n’est pas actif, cela signifie que l’opération de gestion est passée dans un état non annulable et qu’elle prendra bientôt fin.  L’opération de gestion poursuit son exécution jusqu’à ce qu’elle soit terminée.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Si Azure PowerShell n’est pas encore installé, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

Pour annuler une opération de gestion, vous devez spécifier son nom. Par conséquent, commencez par utiliser la commande get pour récupérer la liste des opérations, puis annulez l’opération spécifique.

```powershell-interactive
$managedInstance = "<Your-instance-name>"
$resourceGroup = "<Your-resource-group-name>"

$managementOperations = Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup

foreach ($mo in $managementOperations ) {
    if($mo.State -eq "InProgress" -and $mo.IsCancellable){
        $cancelRequest = Stop-AzSqlInstanceOperation -ResourceGroupName $resourceGroup -ManagedInstanceName $managedInstance -Name $mo.Name
        Get-AzSqlInstanceOperation -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup -Name $mo.Name
    }
}
```

Pour une explication détaillée des commandes, consultez [Get-AzSqlInstanceOperation](/powershell/module/az.sql/get-azsqlinstanceoperation) et [Stop-AzSqlInstanceOperation](/powershell/module/az.sql/stop-azsqlinstanceoperation).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Si Azure CLI n’est pas encore installé, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

Pour annuler l’opération de gestion, vous devez spécifier son nom. Par conséquent, commencez par utiliser la commande get pour récupérer la liste des opérations, puis annulez l’opération spécifique.

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName |
   --query "[?state=='InProgress' && isCancellable].{Name: name}" -o tsv |
while read -r operationName; do

az sql mi op cancel -g yourResourceGroupName --mi yourInstanceName -n $operationName
done
```

Pour une explication détaillée des commandes, consultez [az sql mi op](/cli/azure/sql/mi/op).

---

## <a name="canceled-deployment-request"></a>Demande de déploiement annulée

Avec la version d’API 2020-02-02, dès que la demande de création d’instance est acceptée, l’instance commence à exister en tant que ressource, quelle que soit la progression du processus de déploiement (l’état de l’instance managée est **Provisionnement**). Si vous annulez la demande de déploiement de l’instance (création d’une nouvelle instance), l’instance managée passe de l’état **Provisionnement** à **FailedToCreate**.

Les instances dont la création a échoué sont toujours présentes en tant que ressource et : 

- Ne sont pas facturées
- Ne sont pas comptabilisées pour les limites de ressources (quota vCore ou sous-réseau)


> [!NOTE]
> Pour réduire au maximum le bruit dans la liste des ressources ou des instances managées, supprimez les instances qui n’ont pas réussi à déployer ou les instances avec des déploiements annulés. 


## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer votre première instance managée, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
- Pour une liste des fonctionnalités et leur comparaison, consultez [Fonctionnalités SQL courantes](../database/features-comparison.md).
- Pour plus d’informations sur la configuration du réseau virtuel, consultez [Configuration de réseau virtuel SQL Managed Instance](connectivity-architecture-overview.md).
- Pour obtenir un guide de démarrage rapide qui crée une instance managée et restaure une base de données à partir d’un fichier de sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour accéder à un tutoriel expliquant comment utiliser Azure Database Migration Service pour la migration, consultez [Migration SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).

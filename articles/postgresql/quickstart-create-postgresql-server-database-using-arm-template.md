---
title: 'Démarrage rapide : Création d’un serveur Azure Database pour PostgreSQL - Modèle ARM'
description: Dans ce guide de démarrage rapide, découvrez comment créer un serveur unique Azure Database pour PostgreSQL à l’aide d’un modèle Azure Resource Manager.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: quickstart
ms.custom: subject-armqs
ms.date: 05/14/2020
ms.openlocfilehash: 9b022f83ed2a4e3a23165cc6bda298a53c008c7c
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93331639"
---
# <a name="quickstart-use-an-arm-template-to-create-an-azure-database-for-postgresql---single-server"></a>Démarrage rapide : Création d’un serveur Azure Database pour PostgreSQL à l’aide d’un modèle ARM – Serveur unique

Azure Database pour PostgreSQL est un service administré que vous utilisez pour exécuter, gérer et mettre à l’échelle des bases de données PostgreSQL hautement disponibles dans le cloud. Ce guide de démarrage rapide utilise un modèle Azure Resource Manager (modèle ARM) pour créer un serveur Azure Database pour PostgreSQL dans le portail Azure, PowerShell ou Azure CLI.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer sur Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-managed-postgresql-with-vnet%2fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/azure-portal)

Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).
* Si vous souhaitez exécuter le code localement, [Azure PowerShell](/powershell/azure/).

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

* Compte Azure avec un abonnement actif. [Créez-en un gratuitement](https://azure.microsoft.com/free/).
* Si vous souhaitez exécuter le code localement, [Azure CLI](/cli/azure/).

---

## <a name="review-the-template"></a>Vérifier le modèle

Un serveur Azure Database pour PostgreSQL est créé. Il contient un ensemble configuré de ressources de calcul et de stockage. Pour plus d’informations, consultez [Niveaux tarifaires dans Azure Database pour PostgreSQL – Serveur unique](concepts-pricing-tiers.md). Vous créez ce serveur dans un [groupe de ressources Azure](../azure-resource-manager/management/overview.md).

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-managed-postgresql-with-vnet/).

:::code language="json" source="~/quickstart-templates/101-managed-postgresql-with-vnet/azuredeploy.json":::

Le modèle définit cinq ressources Azure :

* [**Microsoft.Network/virtualNetworks**](/azure/templates/microsoft.network/virtualnetworks)
* [**Microsoft.Network/virtualNetworks/subnets**](/azure/templates/microsoft.network/virtualnetworks/subnets)
* [**Microsoft.DBforPostgreSQL/servers**](/azure/templates/microsoft.dbforpostgresql/servers)
* [**Microsoft.DBforPostgreSQL/servers/virtualNetworkRules**](/azure/templates/microsoft.dbforpostgresql/servers/virtualnetworkrules)
* [**Microsoft.DBforPostgreSQL/servers/firewallRules**](/azure/templates/microsoft.dbforpostgresql/servers/firewallrules)

Pour voir d’autres exemples de modèles Azure Database pour PostgreSQL, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Dbforpostgresql&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sélectionnez le lien suivant pour déployer le modèle de serveur Azure Database pour PostgreSQL sur le Portail Azure :

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer sur Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3a%2f%2fraw.githubusercontent.com%2fAzure%2fazure-quickstart-templates%2fmaster%2f101-managed-postgresql-with-vnet%2fazuredeploy.json)

Sur la page **Déployer Azure Database pour PostgreSQL avec un réseau virtuel**  :

1. Pour **Groupe de ressources** , sélectionnez **Créer** , entrez un nom pour le nouveau groupe de ressources, puis sélectionnez **OK**.

2. Si vous avez créé un groupe de ressources, sélectionnez une **Localisation** pour le groupe de ressources et le nouveau serveur.

3. Entrez un **Nom de serveur** , une **Connexion de l’administrateur** et un **Mot de passe de connexion de l’administrateur**.

    :::image type="content" source="./media/quickstart-create-postgresql-server-database-using-arm-template/deploy-azure-database-for-postgresql-with-vnet.png" alt-text="Fenêtre Déployer Azure Database pour PostgreSQL avec un réseau virtuel, modèle de démarrage rapide Azure, Portail Azure":::

4. Changez les autres paramètres par défaut si vous le souhaitez :

    * **Abonnement**  : sélectionnez l’abonnement Azure que vous souhaitez utiliser pour le serveur.
    * **Capacité de la référence**  : capacité vCore, qui peut être *2* (valeur par défaut), *4* , *8* , *16* , *32* ou *64*.
    * **Nom de la référence**  : préfixe du niveau de référence, famille de références et capacité de référence, joints par des traits de soulignement, par exemple, *B_Gen5_1* , *GP_Gen5_2* (valeur par défaut) ou *MO_Gen5_32*.
    * **Taille de la référence SKU (Mo)**  : taille de stockage, en mégaoctets, du serveur Azure Database pour PostgreSQL (par défaut, *51200* ).
    * **Niveau de référence SKU**  : niveau de déploiement, tel que *Basic* , *GeneralPurpose* (valeur par défaut) ou *MemoryOptimized*.
    * **Famille de références**  : *Gen4* ou *Gen5* (valeur par défaut), qui indique la génération du matériel pour le déploiement du serveur.
    * **Version de PostgreSQL**  : version du serveur PostgreSQL à déployer, par exemple *9.5* , *9.6* , *10* ou *11* (par défaut).
    * **Nombre de jours de conservation de la sauvegarde**  : période souhaitée pour la conservation des sauvegardes géoredondantes, en jours (par défaut, *7* ).
    * **Sauvegarde géoredondante**  : *Activé* ou *Désactivé* (valeur par défaut), en fonction des exigences de géo-reprise d’activité après sinistre.
    * **Nom du réseau virtuel**  : nom du réseau virtuel (par défaut, *azure_postgresql_vnet* ).
    * **Nom du sous-réseau**  : nom du sous-réseau (par défaut, *azure_postgresql_subnet* ).
    * **Nom de la règle de réseau virtuel**  : nom de la règle de réseau virtuel autorisant le sous-réseau (par défaut, *AllowSubnet* ).
    * **Préfixe d’adresse de réseau virtuel**  : préfixe d’adresse du réseau virtuel (par défaut, *10.0.0.0/16* ).
    * **Préfixe de sous-réseau**  : préfixe de l’adresse du sous-réseau (par défaut, *10.0.0.0/16* ).

5. Lisez les conditions générales, puis cochez la case **J’accepte les conditions générales mentionnées ci-dessus**.

6. Sélectionnez **Achat**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Utilisez le code interactif suivant pour créer un serveur Azure Database pour PostgreSQL à l’aide du modèle. Le code vous invite à entrer le nom du nouveau serveur, le nom et la localisation d’un nouveau groupe de ressources ainsi qu’un nom et un mot de passe de compte d’administrateur.

Pour exécuter le code dans Azure Cloud Shell, sélectionnez **Essayer** dans le coin supérieur de n’importe quel bloc de code.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter a name for the new Azure Database for PostgreSQL server"
$resourceGroupName = Read-Host -Prompt "Enter a name for the new resource group where the server will exist"
$location = Read-Host -Prompt "Enter an Azure region (for example, centralus) for the resource group"
$adminUser = Read-Host -Prompt "Enter the Azure Database for PostgreSQL server's administrator account name"
$adminPassword = Read-Host -Prompt "Enter the administrator password" -AsSecureString

New-AzResourceGroup -Name $resourceGroupName -Location $location # Use this command when you need to create a new resource group for your deployment
New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName `
    -TemplateUri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-managed-postgresql-with-vnet/azuredeploy.json `
    -serverName $serverName `
    -administratorLogin $adminUser `
    -administratorLoginPassword $adminPassword

Read-Host -Prompt "Press [ENTER] to continue: "
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Utilisez le code interactif suivant pour créer un serveur Azure Database pour PostgreSQL à l’aide du modèle. Le code vous invite à entrer le nom du nouveau serveur, le nom et la localisation d’un nouveau groupe de ressources ainsi qu’un nom et un mot de passe de compte d’administrateur.

Pour exécuter le code dans Azure Cloud Shell, sélectionnez **Essayer** dans le coin supérieur de n’importe quel bloc de code.

```azurecli-interactive
read -p "Enter a name for the new Azure Database for PostgreSQL server:" serverName &&
read -p "Enter a name for the new resource group where the server will exist:" resourceGroupName &&
read -p "Enter an Azure region (for example, centralus) for the resource group:" location &&
read -p "Enter the Azure Database for PostgreSQL server's administrator account name:" adminUser &&
read -p "Enter the administrator password:" adminPassword &&
params='serverName='$serverName' administratorLogin='$adminUser' administratorLoginPassword='$adminPassword &&
az group create --name $resourceGroupName --location $location &&
az deployment group create --resource-group $resourceGroupName --parameters $params --template-uri https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-managed-postgresql-with-vnet/azuredeploy.json &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="review-deployed-resources"></a>Vérifier les ressources déployées

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour obtenir une vue d’ensemble de votre nouveau serveur Azure Database pour PostgreSQL, procédez ainsi :

1. Sur le [Portail Azure](https://portal.azure.com), recherchez et sélectionnez **Serveurs Azure Database pour PostgreSQL**.

2. Dans la liste des bases de données, sélectionnez votre nouveau serveur. La page **Vue d’ensemble** de votre nouveau serveur Azure Database pour PostgreSQL s’affiche.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

Exécutez le code interactif suivant pour afficher les détails de votre serveur Azure Database pour PostgreSQL. Vous devrez entrer le nom du nouveau serveur.

```azurepowershell-interactive
$serverName = Read-Host -Prompt "Enter the name of your Azure Database for PostgreSQL server"
Get-AzResource -ResourceType "Microsoft.DbForPostgreSQL/servers" -Name $serverName | ft
Read-Host -Prompt "Press [ENTER] to continue: "
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

Exécutez le code interactif suivant pour afficher les détails de votre serveur Azure Database pour PostgreSQL. Vous devrez entrer le nom et le groupe de ressources du nouveau serveur.

```azurecli-interactive
read -p "Enter your Azure Database for PostgreSQL server name: " serverName &&
read -p "Enter the resource group where the Azure Database for PostgreSQL server exists: " resourcegroupName &&
az resource show --resource-group $resourcegroupName --name $serverName --resource-type "Microsoft.DbForPostgreSQL/servers" &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Groupes de ressources**.

2. Dans la liste des groupes de ressources, choisissez le nom de votre groupe de ressources.

3. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.

4. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Read-Host -Prompt "Press [ENTER] to continue: "
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
read -p "Enter the Resource Group name: " resourceGroupName &&
az group delete --name $resourceGroupName &&
read -p "Press [ENTER] to continue: "
```

---

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir un tutoriel pas à pas vous guidant tout au long du processus de création d’un modèle, consultez :

> [!div class="nextstepaction"]
> [Tutoriel : Créer et déployer votre premier modèle Resource Manager](../azure-resource-manager/templates/template-tutorial-create-first-template.md)

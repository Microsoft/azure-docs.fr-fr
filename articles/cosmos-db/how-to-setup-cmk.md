---
title: Configurer des clés gérées par le client pour votre compte Azure Cosmos DB
description: Découvrez comment configurer des clés gérées par le client pour votre compte Azure Cosmos DB avec Azure Key Vault
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: how-to
ms.date: 08/05/2020
ms.author: thweiss
ms.openlocfilehash: e87f6f158265fd8ac210a0a071e35b0bb77df4d9
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96338268"
---
# <a name="configure-customer-managed-keys-for-your-azure-cosmos-account-with-azure-key-vault"></a>Configurer des clés gérées par le client pour votre compte Azure Cosmos avec Azure Key Vault
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

Les données stockées dans votre compte Azure Cosmos sont chiffrées automatiquement et de façon fluide avec des clés gérées par Microsoft (**clés gérées par le service**). Vous pouvez éventuellement choisir d’ajouter une deuxième couche de chiffrement avec des clés gérées par vos soins (**clés gérées par le client**).

:::image type="content" source="./media/how-to-setup-cmk/cmk-intro.png" alt-text="Couches de chiffrement autour des données du client":::

Vous devez stocker les clés gérées par le client dans [Azure Key Vault](../key-vault/general/overview.md) et fournir une clé pour chaque compte Azure Cosmos activé avec des clés gérées par le client. Cette clé est utilisée pour chiffrer toutes les données stockées dans ce compte.

> [!NOTE]
> Actuellement, les clés gérées par le client sont uniquement disponibles pour les nouveaux comptes Azure Cosmos. Vous devez les configurer lors de la création du compte.

## <a name="register-the-azure-cosmos-db-resource-provider-for-your-azure-subscription"></a><a id="register-resource-provider"></a> Inscrire le fournisseur de ressources Azure Cosmos DB dans l’abonnement Azure

1. Connectez-vous au [Portail Azure](https://portal.azure.com/), accédez à votre abonnement Azure et sélectionnez **Fournisseurs de ressources** sous l’onglet **Paramètres** :

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp.png" alt-text="Entrée Fournisseurs de ressources dans le menu de gauche":::

1. Recherchez le fournisseur de ressources **Microsoft.DocumentDB**. Vérifiez si le fournisseur de ressources est déjà marqué comme inscrit. Si ce n’est pas le cas, choisissez le fournisseur de ressources et sélectionnez **Inscrire** :

   :::image type="content" source="./media/how-to-setup-cmk/portal-rp-register.png" alt-text="Inscription du fournisseur de ressources Microsoft.DocumentDB":::

## <a name="configure-your-azure-key-vault-instance"></a>Configurer votre instance Azure Key Vault

L’utilisation de clés gérées par le client avec Azure Cosmos DB vous oblige à définir deux propriétés sur l’instance Azure Key Vault que vous prévoyez d’utiliser pour héberger vos clés de chiffrement : **Suppression réversible** et **Protection de purge**.

Si vous créez une instance Azure Key Vault, activez ces propriétés lors de la création :

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-prop.png" alt-text="Activation de la suppression réversible et de la protection de purge pour une nouvelle instance Azure Key Vault":::

Si vous utilisez une instance Azure Key Vault existante, vous pouvez vérifier que ces propriétés sont activées en examinant la section **Propriétés** dans le Portail Azure. Si une de ces propriétés n’est pas activée, consultez les sections intitulées « Activation de la suppression réversible » et « Activation de la protection de purge » dans l’un des articles suivants :

- [Guide pratique pour utiliser la suppression réversible avec Power​Shell](../key-vault/general/key-vault-recovery.md)
- [Guide pratique pour utiliser la suppression réversible avec Azure CLI](../key-vault/general/key-vault-recovery.md)

## <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Ajouter une stratégie d’accès à votre instance Azure Key Vault

1. Dans le Portail Azure, accédez à l’instance Azure Key Vault que vous prévoyez d’utiliser pour héberger vos clés de chiffrement. Sélectionnez **Stratégies d’accès** dans le menu de gauche :

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-ap.png" alt-text="Stratégies d’accès dans le menu de gauche":::

1. Sélectionnez **+ Ajouter une stratégie d’accès**.

1. Dans le menu déroulant **Autorisations de clé**, sélectionnez les autorisations **Get**, **Unwrap Key** et **Wrap Key** :

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap-perm2.png" alt-text="Sélection des autorisations appropriées":::

1. Sous **Sélectionner le principal**, sélectionnez **Aucune sélection**. Ensuite, recherchez le principal **Azure Cosmos DB** et sélectionnez-le. Pour faciliter la recherche, vous pouvez également effectuer une recherche par ID de principal : `a232010e-820c-4083-83bb-3ace5fc29d0b` pour toute région Azure, à l’exception des régions Azure Government où l’ID principal est `57506a73-e302-42a9-b869-6f12d9ec29e9`. Enfin, choisissez **Sélectionner** en bas. Si le principal **Azure Cosmos DB** ne figure pas dans la liste, vous devrez peut-être réinscrire le fournisseur de ressources **Microsoft.DocumentDB** comme cela est décrit dans la section [Inscrire le fournisseur de ressources](#register-resource-provider) de cet article.

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-add-ap.png" alt-text="Sélectionnez le principal Azure Cosmos DBl":::

1. Sélectionnez **Ajouter** pour ajouter la nouvelle stratégie d’accès.

1. Sélectionnez **Enregistrer** sur l’instance Key Vault pour enregistrer toutes les modifications.

## <a name="generate-a-key-in-azure-key-vault"></a>Générer une clé dans Azure Key Vault

1. Dans le Portail Azure, accédez à l’instance Azure Key Vault que vous prévoyez d’utiliser pour héberger vos clés de chiffrement. Ensuite, sélectionnez **Clés** dans le menu de gauche :

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keys.png" alt-text="Entrée Clés dans le menu de gauche":::

1. Sélectionnez **Générer/importer**, attribuez un nom à la nouvelle clé, puis sélectionnez une taille de clé RSA. Un minimum de 3072 est recommandé pour une sécurité optimale. Sélectionnez ensuite **Créer** :

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-gen.png" alt-text="Créer une nouvelle clé":::

1. Une fois la clé créée, sélectionnez la clé que vous venez de créer puis sa version actuelle.

1. Copiez l’**identificateur de clé** de la clé à l’exception de la partie qui suit la dernière barre oblique :

   :::image type="content" source="./media/how-to-setup-cmk/portal-akv-keyid.png" alt-text="Copie de l’identificateur de clé de la clé":::

## <a name="create-a-new-azure-cosmos-account"></a>Créer un compte Azure Cosmos

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Lorsque vous créez un compte Azure Cosmos DB à partir du Portail Azure, choisissez **Clé gérée par le client** à l’étape **Chiffrement**. Dans le champ **URI de clé**, collez l’identificateur URI/clé de la clé Azure Key Vault que vous avez copiée à partir de l’étape précédente :

:::image type="content" source="./media/how-to-setup-cmk/portal-cosmos-enc.png" alt-text="Définition des paramètres CMK dans le Portail Azure":::

### <a name="using-azure-powershell"></a><a id="using-powershell"></a> Utilisation d’Azure PowerShell

Lorsque vous créez un compte Azure Cosmos DB avec PowerShell :

- Transmettez l’URI de la clé Azure Key Vault copiée précédemment sous la propriété **keyVaultKeyUri** dans **PropertyObject**.

- Utilisez **2019-12-12** ou une version ultérieure comme version de l’API.

> [!IMPORTANT]
> Vous devez définir explicitement la propriété `locations` pour que le compte soit créé avec les clés gérées par le client.

```powershell
$resourceGroupName = "myResourceGroup"
$accountLocation = "West US 2"
$accountName = "mycosmosaccount"

$failoverLocations = @(
    @{ "locationName"="West US 2"; "failoverPriority"=0 }
)

$CosmosDBProperties = @{
    "databaseAccountOfferType"="Standard";
    "locations"=$failoverLocations;
    "keyVaultKeyUri" = "https://<my-vault>.vault.azure.net/keys/<my-key>";
}

New-AzResource -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    -ApiVersion "2019-12-12" -ResourceGroupName $resourceGroupName `
    -Location $accountLocation -Name $accountName -PropertyObject $CosmosDBProperties
```

Une fois le compte créé, vous pouvez vérifier si les clés gérées par le client ont été activées en extrayant l’URI de la clé Azure Key Vault :

```powershell
Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
    -ResourceType "Microsoft.DocumentDb/databaseAccounts" `
    | Select-Object -ExpandProperty Properties `
    | Select-Object -ExpandProperty keyVaultKeyUri
```

### <a name="using-an-azure-resource-manager-template"></a>Utilisation d’un modèle Azure Resource Manager

Lorsque vous créez un compte Azure Cosmos via un modèle Azure Resource Manager :

- Transmettez l’URI de la clé Azure Key Vault copiée précédemment sous la propriété **keyVaultKeyUri** dans l’objet **properties**.

- Utilisez **2019-12-12** ou une version ultérieure comme version de l’API.

> [!IMPORTANT]
> Vous devez définir explicitement la propriété `locations` pour que le compte soit créé avec les clés gérées par le client.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "accountName": {
            "type": "string"
        },
        "location": {
            "type": "string"
        },
        "keyVaultKeyUri": {
            "type": "string"
        }
    },
    "resources": 
    [
        {
            "type": "Microsoft.DocumentDB/databaseAccounts",
            "name": "[parameters('accountName')]",
            "apiVersion": "2019-12-12",
            "kind": "GlobalDocumentDB",
            "location": "[parameters('location')]",
            "properties": {
                "locations": [ 
                    {
                        "locationName": "[parameters('location')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    }
                ],
                "databaseAccountOfferType": "Standard",
                "keyVaultKeyUri": "[parameters('keyVaultKeyUri')]"
            }
        }
    ]
}
```

Déployez le modèle avec le script PowerShell suivant :

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "mycosmosaccount"
$accountLocation = "West US 2"
$keyVaultKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-key>"

New-AzResourceGroupDeployment `
    -ResourceGroupName $resourceGroupName `
    -TemplateFile "deploy.json" `
    -accountName $accountName `
    -location $accountLocation `
    -keyVaultKeyUri $keyVaultKeyUri
```

### <a name="using-the-azure-cli"></a><a id="using-azure-cli"></a> Utilisation de l’interface de ligne de commande Azure (CLI)

Quand vous créez un compte Azure Cosmos à l’aide d’Azure CLI, transmettez l’URI de la clé Azure Key Vault que vous avez copiée sous le paramètre `--key-uri`.

```azurecli-interactive
resourceGroupName='myResourceGroup'
accountName='mycosmosaccount'
keyVaultKeyUri = 'https://<my-vault>.vault.azure.net/keys/<my-key>'

az cosmosdb create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' failoverPriority=0 isZoneRedundant=False \
    --key-uri $keyVaultKeyUri
```

Une fois le compte créé, vous pouvez vérifier si les clés gérées par le client ont été activées en extrayant l’URI de la clé Azure Key Vault :

```azurecli-interactive
az cosmosdb show \
    -n $accountName \
    -g $resourceGroupName \
    --query keyVaultKeyUri
```

## <a name="key-rotation"></a>Rotation des clés

La rotation de la clé gérée par le client utilisée par votre compte Azure Cosmos peut être effectuée de deux manières.

- Créez une nouvelle version de la clé actuellement utilisée à partir d’Azure Key Vault :

  :::image type="content" source="./media/how-to-setup-cmk/portal-akv-rot.png" alt-text="Créez une nouvelle version de la clé":::

- Remplacez la clé actuellement utilisée par une toute autre, en mettant à jour l’URI de la clé de votre compte. Dans le portail Azure, accédez à votre compte Azure Cosmos, puis sélectionnez **Chiffrement des données** dans le menu de gauche :

    :::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Entrée du menu Chiffrement des données":::

    Remplacez ensuite l’**URI de la clé** par la nouvelle clé que vous souhaitez utiliser, puis sélectionnez **Enregistrer** :

    :::image type="content" source="./media/how-to-setup-cmk/portal-key-swap.png" alt-text="Mettre à jour l’URI de la clé":::

    Voici comment obtenir le même résultat dans PowerShell :

    ```powershell
    $resourceGroupName = "myResourceGroup"
    $accountName = "mycosmosaccount"
    $newKeyUri = "https://<my-vault>.vault.azure.net/keys/<my-new-key>"
    
    $account = Get-AzResource -ResourceGroupName $resourceGroupName -Name $accountName `
        -ResourceType "Microsoft.DocumentDb/databaseAccounts"
    
    $account.Properties.keyVaultKeyUri = $newKeyUri
    
    $account | Set-AzResource -Force
    ```

La clé ou la version de clé précédente peut être désactivée après 24 heures ou une fois que les [journaux d’audit Azure Key Vault](../key-vault/general/logging.md) n’affichent plus d’activité Azure Cosmos DB sur cette clé ou version de clé.
    
## <a name="error-handling"></a>Gestion des erreurs

Lorsque vous utilisez des clés gérées par le client (CMK) dans Azure Cosmos DB, en cas d’erreur, Azure Cosmos DB retourne les détails de l’erreur avec un code de sous-état HTTP dans la réponse. Vous pouvez utiliser ce code de sous-état pour déboguer la cause racine du problème. Pour obtenir la liste des codes de sous-état HTTP pris en charge, consultez l’article [Codes d’état HTTP pour Azure Cosmos DB](/rest/api/cosmos-db/http-status-codes-for-cosmosdb).

## <a name="frequently-asked-questions"></a>Forum aux questions

### <a name="is-there-an-additional-charge-to-enable-customer-managed-keys"></a>Des frais supplémentaires sont-ils facturés pour les clés gérées par le client ?

Non, il n’y a aucun frais pour activer cette fonctionnalité.

### <a name="how-do-customer-managed-keys-impact-capacity-planning"></a>Comment les clés gérées par le client affectent-elles la planification de la capacité ?

Lorsque vous utilisez des clés gérées par le client, les [unités de requête](./request-units.md) consommées par vos opérations de base de données connaissent une augmentation afin de refléter le traitement supplémentaire nécessaire au chiffrement et déchiffrement de vos données. Cela peut entraîner une utilisation légèrement supérieure de la capacité approvisionnée. Consultez le tableau ci-dessous pour en voir des conseils :

| Type d'opération | Augmentation des RU |
|---|---|
| Lectures de points (extraction d’éléments par ID) | +5 % par opération |
| Toute opération d’écriture | +6 % par opération<br/>environ + 0,06 RU par propriété indexée |
| Requêtes, lecture de flux de modification ou flux de conflits | +15 % par opération |

### <a name="what-data-gets-encrypted-with-the-customer-managed-keys"></a>Quelles données sont-elles chiffrées avec les clés gérées par le client ?

Toutes les données stockées dans votre compte Azure Cosmos sont chiffrées avec les clés gérées par le client, à l’exception des métadonnées suivantes :

- Les noms de vos [comptes, bases de données et conteneurs](./account-databases-containers-items.md#elements-in-an-azure-cosmos-account) Azure Cosmos DB

- Les noms de vos [procédures stockées](./stored-procedures-triggers-udfs.md)

- Les chemins de propriété déclarés dans vos [stratégies d’indexation](./index-policy.md)

- Les valeurs de [clés de partition](./partitioning-overview.md) de vos conteneurs

### <a name="are-customer-managed-keys-supported-for-existing-azure-cosmos-accounts"></a>Les clés gérées par le client sont-elles prises en charge pour les comptes Azure Cosmos existants ?

Cette fonctionnalité n’est actuellement disponible que pour les nouveaux comptes.

### <a name="is-there-a-plan-to-support-finer-granularity-than-account-level-keys"></a>Existe-t-il un plan pour prendre en charge une plus grande précision que celle des clés au niveau du compte ?

Actuellement non, mais les clés au niveau du conteneur sont prises en compte.

### <a name="how-can-i-tell-if-customer-managed-keys-are-enabled-on-my-azure-cosmos-account"></a>Comment puis-je savoir si les clés gérées par le client sont activées sur mon compte Azure Cosmos ?

Dans le portail Azure, accédez à votre compte Azure Cosmos, puis recherchez l’entrée **Chiffrement des données** dans le menu de gauche. Si cette entrée existe, les clés gérées par le client sont activées sur votre compte :

:::image type="content" source="./media/how-to-setup-cmk/portal-data-encryption.png" alt-text="Entrée du menu Chiffrement des données":::

Vous pouvez également extraire par programme les détails de votre compte Azure Cosmos et rechercher la propriété `keyVaultKeyUri`. Reportez-vous aux informations plus haut pour savoir comment procéder dans [PowerShell](#using-powershell) et avec [Azure CLI](#using-azure-cli).

### <a name="how-do-customer-managed-keys-affect-a-backup"></a>Comment les clés gérées par le client influent-elles sur une sauvegarde ?

Azure Cosmos DB prend des [sauvegardes régulières et automatiques](./online-backup-and-restore.md) des données stockées dans votre compte. Cette opération sauvegarde les données chiffrées. Pour utiliser la sauvegarde restaurée, la clé de chiffrement que vous avez utilisée au moment de la sauvegarde est requise. Cela signifie qu’aucune révocation n’a été effectuée et que la version de la clé qui a été utilisée au moment de la sauvegarde est toujours activée.

### <a name="how-do-i-revoke-an-encryption-key"></a>Comment révoquer une clé de chiffrement ?

La révocation de la clé s’effectue en désactivant la dernière version de la clé :

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev2.png" alt-text="Désactiver la version d’une clé":::

Sinon, pour révoquer toutes les clés d’une instance Azure Key Vault, vous pouvez supprimer la stratégie d’accès accordée au principal Azure Cosmos DB :

:::image type="content" source="./media/how-to-setup-cmk/portal-akv-rev.png" alt-text="Suppression de la stratégie d’accès pour le principal Azure Cosmos DB":::

### <a name="what-operations-are-available-after-a-customer-managed-key-is-revoked"></a>Quelles opérations sont-elles disponibles après la révocation d’une clé gérée par le client ?

La seule opération possible lorsque la clé de chiffrement a été révoquée est la suppression du compte.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez plus en détail le [chiffrement des données dans Azure Cosmos DB](./database-encryption-at-rest.md).
- Obtenez une vue d’ensemble de l’[accès sécurisé aux données dans Cosmos DB](secure-access-to-data.md).
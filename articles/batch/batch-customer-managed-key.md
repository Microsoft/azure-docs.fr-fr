---
title: Configurer des clés managées par le client pour votre compte Azure Batch avec Azure Key Vault et l’identité managée
description: En savoir plus sur le chiffrement des données Batch à l’aide de clés
author: pkshultz
ms.topic: how-to
ms.date: 07/17/2020
ms.author: peshultz
ms.openlocfilehash: 404103caf376b792d363996664a69f655d5bd202
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96326010"
---
# <a name="configure-customer-managed-keys-for-your-azure-batch-account-with-azure-key-vault-and-managed-identity"></a>Configurer des clés managées par le client pour votre compte Azure Batch avec Azure Key Vault et l’identité managée

Par défaut Azure Batch utilise des clés managées par la plate-forme pour chiffrer toutes les données client stockées au sein d’Azure Batch Service, telles que les certificats ou les métadonnées de travail/tâche. Si vous le souhaitez, vous pouvez utiliser vos propres clés, c’est-à-dire des clés managées par le client, pour chiffrer les données stockées dans Azure Batch.

Les clés que vous fournissez doivent être générées dans [Azure Key Vault](../key-vault/general/basic-concepts.md), et les comptes Batch que vous souhaitez configurer avec les clés managées par le client doivent être activés avec l’[identité managée Azure](../active-directory/managed-identities-azure-resources/overview.md).

> [!IMPORTANT]
> Dans Azure Batch, la prise en charge des clés gérées par le client est actuellement disponible en préversion publique pour les régions Europe Ouest, Europe Nord, Suisse Nord, USA Centre, USA Centre Sud, USA Centre-Ouest, USA Est, USA Est 2, USA Ouest 2, US Gov Virginie et US Gov Arizona.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

## <a name="create-a-batch-account-with-system-assigned-managed-identity"></a>Créer un compte Batch avec identité managée affectée par le système

### <a name="azure-portal"></a>Portail Azure

Dans le [Portail Azure](https://portal.azure.com/), lorsque vous créez des comptes Batch, sélectionnez **Système affecté** dans le type d’identité sous l’onglet **Avancé**.

![Nouveau compte Batch avec type d’identité affectée par le système](./media/batch-customer-managed-key/create-batch-account.png)

Une fois le compte créé, vous trouverez un GUID unique dans le champ **ID du principal d’identité** dans la section **Propriété**. Le **type d’identité** affichera `SystemAssigned`.

![GUID unique dans le champ ID du principal d’identité](./media/batch-customer-managed-key/linked-batch-principal.png)
 
### <a name="azure-cli"></a>Azure CLI

Lorsque vous créez un nouveau compte Batch, précisez `SystemAssigned` pour le paramètre `--identity`.

```powershell
resourceGroupName='myResourceGroup'
accountName='mybatchaccount'

az batch account create \
    -n $accountName \
    -g $resourceGroupName \
    --locations regionName='West US 2' \
    --identity 'SystemAssigned'
```

Une fois le compte créé, vous pouvez vérifier que l’identité managée affectée par le système a été activée sur ce compte. Veillez à noter le `PrincipalId`, car cette valeur sera nécessaire pour accorder à ce compte Batch l’accès à Key Vault.

```powershell
az batch account show \
    -n $accountName \
    -g $resourceGroupName \
    --query identity
```

> [!NOTE]
> L’identité managée affectée par le système créée dans un compte Batch est utilisée uniquement pour récupérer des clés gérées par le client dans le coffre de clés. Cette identité n’est pas disponible sur les pools Batch.

## <a name="configure-your-azure-key-vault-instance"></a>Configurer votre instance Azure Key Vault

### <a name="create-an-azure-key-vault"></a>Créer un Azure Key Vault

Lors de la création d’une instance Azure Key Vault avec des clés managées par le client pour Azure Batch, assurez-vous que la **suppression réversible** et la **protection de purge** sont activées.

![Écran de création Key Vault](./media/batch-customer-managed-key/create-key-vault.png)

### <a name="add-an-access-policy-to-your-azure-key-vault-instance"></a>Ajouter une stratégie d’accès à votre instance Azure Key Vault

Dans le Portail Azure, une fois Key Vault créé, ajoutez l’accès au compte Batch à l’aide de l’identité managée dans la **stratégie d’accès** sous **Paramètres**. Sous **Autorisations de clé**, sélectionnez **Obtenir**, **Inclure la clé** et **Ne pas inclure la clé**. 

![Ajouter une stratégie d’accès](./media/batch-customer-managed-key/key-permissions.png)

Dans le champ **Sélectionner** sous **Principal**, renseignez le `principalId` que vous avez noté précédemment ou le nom du compte Batch.

![Entrez l’ID du principal](./media/batch-customer-managed-key/principal-id.png)

### <a name="generate-a-key-in-azure-key-vault"></a>Générer une clé dans Azure Key Vault

Dans le Portail Azure, accédez à l’instance Key Vault dans la section **Clé**, puis **sélectionnez Générer/importer**. Choisissez le **type de clé** `RSA` et une **taille de clé RSA** d’au moins `2048` bits. Les types de clés `EC` ne sont actuellement pas pris en charge en tant que clé gérée par le client sur un compte Batch.

![Créer une clé](./media/batch-customer-managed-key/create-key.png)

Une fois que la clé a été créée, cliquez sur la nouvelle clé et la version actuelle, copiez l’**identificateur de clé** dans la section **Propriétés**.  Assurez-vous que sous **Opérations autorisées**, les options **Inclure la clé** et **Ne pas inclure la clé** sont activées.

## <a name="enable-customer-managed-keys-on-azure-batch-account"></a>Activer des clés managées par le client sur un compte Azure Batch

### <a name="azure-portal"></a>Portail Azure

Dans le [Portail Azure](https://portal.azure.com/), accédez à la page compte Batch. Dans la section **Chiffrement**, activez **Clé managée par le client**. Vous pouvez utiliser directement l’identificateur de clé, ou sélectionner le coffre de clés, puis cliquer sur **Sélectionner un coffre de clés et une clé**.

![Dans la section Chiffrement, activez Clé managée par le client](./media/batch-customer-managed-key/encryption-page.png)

### <a name="azure-cli"></a>Azure CLI

Une fois que le compte Batch a été créé avec une identité managée affectée par le système et que l’accès à Key Vault a été accordé, mettez à jour le compte Batch avec l’URL `{Key Identifier}` sous le paramètre `keyVaultProperties`. Définissez également **encryption_key_source** sur `Microsoft.KeyVault`.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_source Microsoft.KeyVault \
    --encryption_key_identifier {YourKeyIdentifier} 
```

## <a name="update-the-customer-managed-key-version"></a>Mettez à jour la version de la clé managée par le client

Lors de la création d’une nouvelle version d’une clé, mettez à jour le compte Batch afin qu’il utilise cette nouvelle version. Procédez comme suit :

1. Accédez à votre compte Batch dans le Portail Azure et affichez les paramètres de chiffrement.
2. Saisissez l’URI de la nouvelle version de clé. Vous pouvez également sélectionner à nouveau le coffre de clés et la clé pour mettre à jour la version.
3. Enregistrez vos modifications.

Vous pouvez également utiliser Azure CLI pour mettre à jour la version.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_identifier {YourKeyIdentifierWithNewVersion} 
```
## <a name="use-a-different-key-for-batch-encryption"></a>Utiliser une clé différente pour le chiffrement Batch

Pour modifier la clé utilisée pour le chiffrement Batch, procédez comme suit :

1. Accédez à votre compte Batch et affichez les paramètres de chiffrement.
2. Entrez l’URI de la nouvelle clé. Vous pouvez également sélectionner le coffre de clés et choisir une nouvelle clé.
3. Enregistrez vos modifications.

Vous pouvez également utiliser Azure CLI pour vous servir d’une clé différente.

```powershell
az batch account set \
    -n $accountName \
    -g $resourceGroupName \
    --encryption_key_identifier {YourNewKeyIdentifier} 
```
## <a name="frequently-asked-questions"></a>Forum aux questions
  * **Les clés managées par le client sont-elles prises en charge pour les comptes Batch existants ?** Non. Les clés managées par le client sont uniquement prises en charge pour les nouveaux comptes Batch.
  * **Puis-je sélectionner des tailles de clé RSA supérieures à 2048 bits ?** Oui, les tailles de clé RSA de `3072` et `4096` bits sont également prises en charge.
  * **Quelles sont les opérations disponibles après la révocation d’une clé managée par le client ?** La seule opération autorisée est la suppression de compte si Batch perd l’accès à la clé managée par le client.
  * **Comment puis-je restaurer l’accès à mon compte Batch si je supprime accidentellement la clé de Key Vault ?** Étant donné que la protection de purge et la suppression réversible sont activées, vous pouvez restaurer les clés existantes. Pour plus d’informations, consultez [Restaurer un Azure Key Vault](../key-vault/general/key-vault-recovery.md).
  * **Puis-je désactiver les clés managées par le client ?** Vous pouvez à tout moment définir le type de chiffrement du compte Batch sur « clé managée par Microsoft ». Ensuite, vous êtes libre de supprimer ou de modifier la clé.
  * **Comment faire pivoter mes clés ?** Les clés managées par le client ne pivotent pas automatiquement. Pour faire pivoter la clé, mettez à jour l’identificateur de clé auquel le compte est associé.
  * **Après la restauration de l’accès, combien de temps faut-il pour que le compte Batch fonctionne à nouveau ?** Cela peut prendre jusqu’à 10 minutes pour que le compte soit à nouveau accessible après la restauration.
  * **Qu’arrive-t-il à mes ressources lorsque le compte Batch est indisponible ?** Les pools en cours d’exécution continuent à s’exécuter lorsque l’accès Batch aux clés managées par le client est perdu. Toutefois, les nœuds passent à l’état indisponible et les tâches cessent de s’exécuter (elles sont replacées dans la file d’attente). Une fois l’accès restauré, les nœuds sont à nouveau disponibles et les tâches redémarrent.
  * **Ce mécanisme de chiffrement s’applique-t-il aux disques de machine virtuelle dans un pool Batch ?** Non. Pour les pools de configuration de service Cloud, aucun chiffrement n’est appliqué pour le système d’exploitation et le disque temporaire. Pour les pools de configuration de machine virtuelle, le système d’exploitation et les disques de données spécifiés sont chiffrés par défaut avec une clé managée par la plate-forme Microsoft. Actuellement, vous ne pouvez pas spécifier votre propre clé pour ces disques. Pour chiffrer le disque temporaire des machines virtuelles pour un pool Batch avec une clé managée par la plate-forme Microsoft, vous devez activer la propriété [diskEncryptionConfiguration](/rest/api/batchservice/pool/add#diskencryptionconfiguration) dans votre pool de [configuration de machine virtuelle](/rest/api/batchservice/pool/add#virtualmachineconfiguration). Pour les environnements très sensibles, nous vous recommandons d’activer le chiffrement de disque temporaire et d’éviter de stocker des données sensibles sur le système d’exploitation et les disques de données. Pour plus d’informations, consultez [Créer un pool avec le chiffrement de disque activé](./disk-encryption.md).
  * **L’identité managée affectée par le système sur le compte Batch est-elle disponible sur les nœuds de calcul ?** Non. Cette identité managée est actuellement utilisée uniquement pour accéder au coffre de clés Azure pour la clé gérée par le client.

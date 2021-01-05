---
title: Gérer une instance Azure Blockchain Service à l’aide d’Azure CLI
description: Comment gérer une instance Azure Blockchain Service avec Azure CLI
ms.date: 07/23/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: 36b012c486c0c7d3303a81998e88f1605999c899
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87170853"
---
# <a name="manage-azure-blockchain-service-using-azure-cli"></a>Gérer une instance Azure Blockchain Service à l’aide d’Azure CLI

En plus du portail Azure, vous pouvez utiliser Azure CLI pour gérer les membres blockchain et les nœuds de transaction pour votre service Azure Blockchain.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

Si vous préférez installer et utiliser l’interface de ligne de commande localement, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="prepare-your-environment"></a>Préparation de votre environnement

1. Connectez-vous.

    Connectez-vous à l’aide de la commande [az login](/cli/azure/reference-index#az-login) si vous utilisez une installation locale de l’interface CLI.

    ```azurecli
    az login
    ```

    Suivez les étapes affichées dans votre terminal pour effectuer le processus d’authentification.

1. Installez l’extension Azure CLI.

    Quand vous utilisez des références d’extension pour l’interface Azure CLI, vous devez d’abord installer l’extension.  Les extensions Azure CLI vous donnent accès à des commandes expérimentales et en préversion qui ne sont pas encore offertes par l’interface CLI principale.  Pour en savoir plus sur les extensions, notamment sur la mise à jour et la désinstallation, consultez [Utiliser des extensions avec Azure CLI](/cli/azure/azure-cli-extensions-overview).

    Installez l’[extension pour Azure Blockchain Service](/cli/azure/ext/blockchain/blockchain) en exécutant la commande suivante :

    ```azurecli-interactive
    az extension add --name blockchain
    ```

## <a name="create-blockchain-member"></a>Créer un membre blockchain

Dans Azure Blockchain Service, l’exemple [crée un membre blockchain](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-create) qui exécute le protocole de registre Quorum dans un nouveau consortium.

```azurecli
az blockchain member create \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            --location <myBlockchainLocation> \
                            --password <strongMemberAccountPassword> \
                            --protocol "Quorum" \
                            --consortium <myConsortiumName> \
                            --consortium-management-account-password <strongConsortiumManagementPassword> \
                            --sku <skuName>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service sont créées. |
| **name** | Nom unique qui identifie le membre blockchain Azure Blockchain Service. Le nom est utilisé pour l’adresse de point de terminaison public. Par exemple : `myblockchainmember.blockchain.azure.com`. |
| **location** | Région Azure dans laquelle le membre blockchain est créé. Par exemple : `eastus`. Choisissez l’emplacement le plus proche de vos utilisateurs ou de vos autres applications Azure. Les fonctionnalités peuvent ne pas être disponibles dans certaines régions. |
| **mot de passe** | Le mot de passe du nœud de transaction par défaut du membre. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction par défaut du membre blockchain. Le mot de passe doit remplir trois des quatre conditions suivantes : il doit comprendre entre 12 et 72 caractères, 1 caractère minuscule, 1 caractère majuscule, 1 chiffre et 1 caractère spécial qui ne soit pas le signe dièze (#), le symbole de pourcentage (%), une virgule (,), une étoile (*), un guillemet inverse (\`), un guillemet double ("), un guillemet unique ('), un tiret (-) ou un point-virgule (;)|
| **protocol** | Protocole Blockchain. Actuellement, le protocole *Quorum* est pris en charge. |
| **consortium** | Nom du consortium à rejoindre ou créer. Pour plus d’informations sur les consortiums, consultez [Consortium Azure Blockchain Service](consortium.md). |
| **consortium-management-account-password** | Le mot de passe du compte consortium est également appelé mot de passe du compte du membre. Le mot de passe du compte du membre sert à chiffrer la clé privée du compte Ethereum créé pour votre membre. Vous utilisez le compte du membre et le mot de passe du compte du membre pour la gestion du consortium. |
| **sku** | Niveau de service. *Standard* ou *De base*. Utilisez le niveau *De base* pour le développement, les tests et les preuves de concept. Utilisez le niveau *Standard* pour les déploiements en production. Vous devez également utiliser le niveau *Standard* si vous utilisez Blockchain Data Manager, ou si vous envoyez un volume élevé de transactions privées. Le passage du niveau tarifaire de base au niveau standard (ou inversement) après la création du membre n'est pas pris en charge. |

## <a name="change-blockchain-member-passwords-or-firewall-rules"></a>Modifier les mots de passe ou les règles de pare-feu du membre blockchain

L’exemple [met à jour le mot de passe d’un membre blockchain](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-update), le mot de passe de la gestion du consortium et la règle de pare-feu.

```azurecli
az blockchain member update \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName> \
                     --password <strongMemberAccountPassword> \
                     --consortium-management-account-password <strongConsortiumManagementPassword> \
                     --firewall-rules <firewallRules>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service sont créées. |
| **name** | Nom qui identifie votre membre Azure Blockchain Service. |
| **mot de passe** | Le mot de passe du nœud de transaction par défaut du membre. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction par défaut du membre blockchain. Le mot de passe doit remplir trois des quatre conditions suivantes : il doit comprendre entre 12 et 72 caractères, 1 caractère minuscule, 1 caractère majuscule, 1 chiffre et 1 caractère spécial qui ne soit pas le signe dièze (#), le symbole de pourcentage (%), une virgule (,), une étoile (*), un guillemet inverse (\`), un guillemet double ("), un guillemet unique ('), un tiret (-) ou un point-virgule (;)|
| **consortium-management-account-password** | Le mot de passe du compte consortium est également appelé mot de passe du compte du membre. Le mot de passe du compte du membre sert à chiffrer la clé privée du compte Ethereum créé pour votre membre. Vous utilisez le compte du membre et le mot de passe du compte du membre pour la gestion du consortium. |
| **firewall-rules** | Adresse IP de début et de fin pour la liste d’adresses IP autorisées. |

## <a name="create-transaction-node"></a>Créer un nœud de transaction

[Créez un nœud de transaction](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-create) à l’intérieur d’un membre blockchain existant. L’ajout de nœuds de transaction vous permet d’augmenter l’isolation à des fins de sécurité et de répartir la charge. Par exemple, vous pouvez disposer d’un point de terminaison de nœud de transaction pour différentes applications clientes.

```azurecli
az blockchain transaction-node create \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service sont créées. |
| **location** | Région Azure du membre blockchain. |
| **member-name** | Nom qui identifie votre membre Azure Blockchain Service. |
| **mot de passe** | Mot de passe du nœud de transaction. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction. Le mot de passe doit remplir trois des quatre conditions suivantes : il doit comprendre entre 12 et 72 caractères, 1 caractère minuscule, 1 caractère majuscule, 1 chiffre et 1 caractère spécial qui ne soit pas le signe dièze (#), le symbole de pourcentage (%), une virgule (,), une étoile (*), un guillemet inverse (\`), un guillemet double ("), un guillemet unique ('), un tiret (-) ou un point-virgule (;)|
| **name** | Nom du nœud de transaction. |

## <a name="change-transaction-node-password"></a>Modifier le mot de passe du nœud de transaction

L’exemple [met à jour le mot de passe d’un nœud de transaction](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-update).

```azurecli
az blockchain transaction-node update \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --password <strongTransactionNodePassword> \
                     --name <myTransactionNodeName>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service existent. |
| **member-name** | Nom qui identifie votre membre Azure Blockchain Service. |
| **mot de passe** | Mot de passe du nœud de transaction. Utilisez le mot de passe pour l’authentification de base lorsque vous vous connectez au point de terminaison public du nœud de transaction. Le mot de passe doit remplir trois des quatre conditions suivantes : il doit comprendre entre 12 et 72 caractères, 1 caractère minuscule, 1 caractère majuscule, 1 chiffre et 1 caractère spécial qui ne soit pas le signe dièze (#), le symbole de pourcentage (%), une virgule (,), une étoile (*), un guillemet inverse (\`), un guillemet double ("), un guillemet unique ('), un tiret (-) ou un point-virgule (;)|
| **name** | Nom du nœud de transaction. |

## <a name="list-api-keys"></a>Répertorier les clés API

Les clés API peuvent être utilisées pour l'accès au nœud comme avec le nom d'utilisateur et le mot de passe. Il existe deux clés API pour prendre en charge la rotation des clés. Utilisez la commande suivante pour [répertorier vos clés API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-list-api-key).

```azurecli
az blockchain member list-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service existent. |
| **name** | Nom du membre blockchain Azure Blockchain Service. |

## <a name="regenerate-api-keys"></a>Régénérer les clés API

Utilisez la commande suivante pour [régénérer vos clés API](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-transaction-node-regenerate-api-key).

```azurecli
az blockchain member regenerate-api-key \
                            --resource-group <myResourceGroup> \
                            --name <myMemberName> \
                            [--key-name {<keyValue1>, <keyValue2>}]
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service existent. |
| **name** | Nom du membre Azure Blockchain Service. |
| **keyName** | Remplace \<keyValue\> par key1, key2 ou les deux. |

## <a name="delete-a-transaction-node"></a>Supprimer un nœud de transaction

L’exemple [supprime le nœud de transaction d’un membre blockchain](/cli/azure/ext/blockchain/blockchain/transaction-node#ext-blockchain-az-blockchain-transaction-node-delete).

```azurecli
az blockchain transaction-node delete \
                     --resource-group <myResourceGroup> \
                     --member-name <myMemberName> \
                     --name <myTransactionNode>
```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service existent. |
| **member-name** | Nom du membre Azure Blockchain Service qui inclut également le nom du nœud de transaction à supprimer. |
| **name** | Nom du nœud de transaction à supprimer. |

## <a name="delete-a-blockchain-member"></a>Supprimer un membre blockchain

L’exemple [supprime un membre blockchain](/cli/azure/ext/blockchain/blockchain/member#ext-blockchain-az-blockchain-member-delete).

```azurecli
az blockchain member delete \
                     --resource-group <myResourceGroup> \
                     --name <myMemberName>

```

| Paramètre | Description |
|---------|-------------|
| **resource-group** | Nom du groupe de ressources dans lequel les ressources Azure Blockchain Service existent. |
| **name** | Nom du membre Azure Blockchain Service à supprimer. |

## <a name="azure-active-directory"></a>Azure Active Directory

### <a name="grant-access-for-azure-ad-user"></a>Accorder l’accès à un utilisateur Azure AD

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee <assignee> \
                            --scope /subscriptions/<subId>/resourceGroups/<groupName>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>
```

| Paramètre | Description |
|---------|-------------|
| **role** | Nom du rôle Azure AD. |
| **assignee** | ID d'utilisateur Azure AD. Par exemple : `user@contoso.com` |
| **scope** | Étendue de l’attribution de rôle. Peut être un membre blockchain du nœud de la transaction. |

**Exemple :**

Accorder l’accès au nœud pour l’utilisateur Azure AD au **membre** blockchain :

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

**Exemple :**

Accorder l’accès au nœud pour l’utilisateur Azure AD au **nœud de transaction** blockchain :

```azurecli
az role assignment create \
                            --role 'MyRole' \
                            --assignee user@contoso.com \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1/transactionNodes/contosoTransactionNode1
```

### <a name="grant-node-access-for-azure-ad-group-or-application-role"></a>Accorder l’accès au nœud au groupe Azure AD ou au rôle d'application

```azurecli
az role assignment create \
                            --role <role> \
                            --assignee-object-id <assignee_object_id>
```

| Paramètre | Description |
|---------|-------------|
| **role** | Nom du rôle Azure AD. |
| **assignee-object-id** | ID de groupe ou ID d’application Azure AD. |
| **scope** | Étendue de l’attribution de rôle. Peut être un membre blockchain du nœud de la transaction. |

**Exemple :**

Accorder l’accès au nœud au **rôle d’application**

```azurecli
az role assignment create \
                            --role 'myRole' \
                            --assignee-object-id 22222222-2222-2222-2222-222222222222 \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/contosoResourceGroup/providers/Microsoft.Blockchain/blockchainMembers/contosoMember1
```

### <a name="remove-azure-ad-node-access"></a>Supprimer l’accès au nœud Azure AD

```azurecli
az role assignment delete \
                            --role <myRole> \
                            --assignee <assignee> \
                            --scope /subscriptions/mySubscriptionId/resourceGroups/<myResourceGroup>/providers/Microsoft.Blockchain/blockchainMembers/<myMemberName>/transactionNodes/<myTransactionNode>
```

| Paramètre | Description |
|---------|-------------|
| **role** | Nom du rôle Azure AD. |
| **assignee** | ID d'utilisateur Azure AD. Par exemple : `user@contoso.com` |
| **scope** | Étendue de l’attribution de rôle. Peut être un membre blockchain du nœud de la transaction. |

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [configurer des nœuds de transaction Azure Blockchain Service avec le portail Azure](configure-transaction-nodes.md).

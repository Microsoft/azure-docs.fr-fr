---
title: Configurer les nœuds de transaction Azure Blockchain Service
description: Configuration des nœuds de transaction Azure Blockchain Service
ms.date: 11/20/2019
ms.topic: how-to
ms.reviewer: janders
ms.openlocfilehash: 50f9b2aa0f5e1a1b1095b6c799ce0706b6e541de
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92150875"
---
# <a name="configure-azure-blockchain-service-transaction-nodes"></a>Configurer les nœuds de transaction Azure Blockchain Service

Les nœuds de transaction permettent d’envoyer des transactions blockchain à Azure Blockchain Service par le biais d’un point de terminaison public. Le nœud de transaction par défaut contient la clé privée du compte Ethereum inscrit sur la blockchain et n’est pas supprimable en tant que tel.

Pour visualiser les détails du nœud de transaction par défaut :

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Accédez au membre Azure Blockchain Service. Sélectionnez **Nœuds de transaction**.

    ![Sélectionner le nœud de transaction par défaut](./media/configure-transaction-nodes/nodes.png)

    Les détails de la vue d’ensemble comprennent les adresses de point de terminaison public et la clé publique.

## <a name="create-transaction-node"></a>Créer un nœud de transaction

Vous pouvez ajouter jusqu’à neuf nœuds de transaction supplémentaires à votre membre blockchain, ce qui totalise dix nœuds de transaction. L’ajout de nœuds de transaction vous permet d’augmenter l’extensibilité ou de répartir la charge. Par exemple, vous pouvez disposer d’un point de terminaison de nœud de transaction pour différentes applications clientes.

Pour ajouter un nœud de transaction :

1. Dans le Portail Azure, accédez à votre membre Azure Blockchain Service et sélectionnez **Nœuds de transaction > Ajouter**.
1. Renseignez les paramètres du nouveau nœud de transaction.

    ![Ajout d’un nœud de transaction](./media/configure-transaction-nodes/add-node.png)

    | Paramètre | Description |
    |---------|-------------|
    | Nom | Nom du nœud de transaction. Le nom est utilisé pour créer l’adresse DNS du point de terminaison du nœud de transaction. Par exemple : `newnode-myblockchainmember.blockchain.azure.com`. Une fois créé, le nom du nœud n’est plus modifiable. |
    | Mot de passe | Définissez un mot de passe fort. Utilisez le mot de passe pour accéder au point de terminaison du nœud de transaction avec une authentification de base.

1. Sélectionnez **Create** (Créer).

    Le provisionnement d’un nouveau nœud de transaction prend environ 10 minutes. L’ajout de nœuds de transaction entraîne des frais. Pour plus d’informations sur les coûts, consultez la page [Tarification Service Azure Blockchain](https://aka.ms/ABSPricing).

## <a name="endpoints"></a>Points de terminaison

Les nœuds de transaction comportent un nom DNS unique et des points de terminaison publics.

Pour afficher les détails du point de terminaison d’un nœud de transaction :

1. Dans le Portail Azure, accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service et sélectionnez **Vue d’ensemble**.

    ![La capture d’écran montre la vue d’ensemble des nœuds de transaction pour un membre de blockchain.](./media/configure-transaction-nodes/endpoints.png)

Les points de terminaison de nœud de transaction sont sécurisés et nécessitent une authentification. Vous pouvez vous connecter à un point de terminaison de transaction à l’aide de l’authentification Azure AD, de l’authentification de base HTTPS et d’une clé d’accès sur HTTPS ou Websocket par le biais du protocole TLS.

### <a name="azure-active-directory-access-control"></a>Contrôle d’accès Azure Active Directory

Les points de terminaison de nœud de transaction Azure Blockchain Service prennent en charge l’authentification Azure Active Directory (Azure AD). Vous pouvez octroyer un accès à votre point de terminaison à un utilisateur, un groupe et un principal de service Azure AD.

Pour accorder un contrôle d’accès Azure AD à votre point de terminaison :

1. Dans le Portail Azure, accédez à votre membre Azure Blockchain Service et sélectionnez **Nœuds de transaction > Contrôle d’accès (IAM) > Ajouter > Ajouter une attribution de rôle**.
1. Créez une attribution de rôle pour un utilisateur, un groupe ou un principal de service (rôles d’application).

    ![Ajout d’un rôle IAM](./media/configure-transaction-nodes/add-role.png)

    | Paramètre | Action |
    |---------|-------------|
    | Role | Sélectionnez **Propriétaire**, **Contributeur** ou **Lecteur**.
    | Attribuer l’accès à | Sélectionnez **Utilisateur, groupe ou principal du service Azure AD**.
    | Sélectionnez | Recherchez l’utilisateur, le groupe ou le principal de service que vous souhaitez ajouter.

1. Sélectionnez **Enregistrer** pour ajouter l’attribution de rôle.

Pour plus d’informations sur le contrôle d’accès Azure AD, consultez [Ajouter ou supprimer des attributions de rôles Azure à l’aide du portail Azure](../../role-based-access-control/role-assignments-portal.md).

Pour plus d’informations sur la connexion à l’aide de l’authentification Azure AD, consultez l’article [Connect to your node using AAD authentication](configure-aad.md) (Se connecter à votre nœud à l’aide de l’authentification AAD).

### <a name="basic-authentication"></a>Authentification de base

Pour l’authentification de base HTTPS, les informations d’identification de nom d’utilisateur et de mot de passe sont transmises dans l’en-tête HTTPS de la requête au point de terminaison.

Vous pouvez afficher les détails du point de terminaison de l’authentification de base d’un nœud de transaction dans le Portail Azure. Accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service, puis sélectionnez **Authentification de base** dans les paramètres.

![Authentification de base](./media/configure-transaction-nodes/basic.png)

Le nom d’utilisateur correspond au nom de votre nœud et n’est pas modifiable.

Pour utiliser l’URL, remplacez \<password\> par le mot de passe défini lors de l’approvisionnement du nœud. Vous pouvez mettre à jour le mot de passe en sélectionnant **Réinitialiser le mot de passe**.

### <a name="access-keys"></a>Clés d'accès

Pour l’authentification par clé d’accès, la clé d’accès figure dans l’URL du point de terminaison. Lorsque le nœud de transaction est approvisionné, deux clés d’accès sont générées. Vous pouvez utiliser l’une ou l’autre clé pour l’authentification. Deux clés vous permettent de modifier et de faire pivoter des clés.

Vous pouvez visualiser les détails de la clé d’accès d’un nœud de transaction et copier les adresses de point de terminaison qui contiennent les clés d’accès. Accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service, puis sélectionnez **Clés d’accès** dans les paramètres.

### <a name="firewall-rules"></a>Règles de pare-feu

Les règles de pare-feu vous permettent de limiter les adresses IP qui peuvent tenter de s’authentifier auprès de votre nœud de transaction.  Si aucune règle de pare-feu n’est configurée pour votre nœud de transaction, ce dernier n’est accessible par aucun tiers.  

Pour visualiser les règles de pare-feu d’un nœud de transaction, accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service, puis sélectionnez **Règles de pare-feu** dans les paramètres.

Vous pouvez ajouter des règles de pare-feu en entrant un nom de règle, une adresse IP de début et une adresse IP de fin dans la grille **Règles de pare-feu**.

![Règles de pare-feu](./media/configure-transaction-nodes/firewall-rules.png)

Pour activer :

* **Adresse IP unique :** configurez la même adresse pour les adresses IP de début et de fin.
* **Plage d’adresses IP :** configurez la plage d’adresses IP de début et de fin. Par exemple, une plage commençant par 10.221.34.0 et se terminant par 10.221.34.255 activerait la totalité du sous-réseau 10.221.34.xxx.
* **Autoriser toutes les adresses IP :** configurez l’adresse IP de début sur 0.0.0.0 et l’adresse IP de fin sur 255.255.255.255.

## <a name="connection-strings"></a>Chaînes de connexion

La syntaxe des chaînes de connexion pour votre nœud de transaction est fournie pour l’authentification de base ou pour l’utilisation de clés d’accès. Les chaînes de connexion incluant les clés d’accès sur HTTPS et WebSocket sont spécifiées.

Vous pouvez visualiser les chaînes de connexion d’un nœud de transaction et copier les adresses de point de terminaison. Accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service, puis sélectionnez **Chaînes de connexion** dans les paramètres.

![Chaînes de connexion](./media/configure-transaction-nodes/connection-strings.png)

## <a name="sample-code"></a>Exemple de code

Un exemple de code est fourni pour vous aider à vous connecter rapidement à votre nœud de transaction par le biais de Web3, Nethereum, Web3js et Truffle.

Vous pouvez visualiser l’exemple de code de connexion d’un nœud de transaction et le copier afin de l’utiliser avec les outils de développement les plus courants. Accédez à l’un des nœuds de transaction de votre membre Azure Blockchain Service, puis sélectionnez **Exemple de code** dans les paramètres.

Choisissez l’onglet Web3, Nethereum, Truffle ou Web3j pour afficher l’exemple de code que vous souhaitez utiliser.

![Exemple de code](./media/configure-transaction-nodes/sample-code.png)

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configure transaction nodes using Azure CLI](manage-cli.md) (Configurer les nœuds de transaction à l’aide d’Azure CLI)

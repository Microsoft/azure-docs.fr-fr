---
title: Configurer une instance et l’authentification (CLI)
titleSuffix: Azure Digital Twins
description: Découvrez comment configurer une instance du service Azure Digital Twins à l’aide de l’interface CLI
author: baanders
ms.author: baanders
ms.date: 7/23/2020
ms.topic: how-to
ms.service: digital-twins
ms.openlocfilehash: 26302fa67394e6c3122b159866c3814fb5677ba6
ms.sourcegitcommit: d6a739ff99b2ba9f7705993cf23d4c668235719f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92494968"
---
# <a name="set-up-an-azure-digital-twins-instance-and-authentication-cli"></a>Configurer une instance Azure Digital Twins et l’authentification (interface CLI)

[!INCLUDE [digital-twins-setup-selector.md](../../includes/digital-twins-setup-selector.md)]

Cet article explique comment **configurer une nouvelle instance Azure Digital Twins** , notamment la création de l’instance et la configuration de l’authentification. À l’issue de cet article, vous aurez une instance Azure Digital Twins prête pour la programmation.

Cette version de cet article suit ces étapes manuellement, une par une, à l’aide de l’interface CLI.
* Pour suivre ces étapes manuellement à l’aide du portail Azure, consultez la version de cet article relative au portail : [*Guide pratique : Configurer une instance et l’authentification (portail)*](how-to-set-up-instance-portal.md).
* Pour exécuter une configuration automatisée à l’aide d’un exemple de script de déploiement, consultez la version avec script de cet article : [*Guide pratique : Configurer une instance et l’authentification (procédure scriptée)*](how-to-set-up-instance-scripted.md).

[!INCLUDE [digital-twins-setup-steps.md](../../includes/digital-twins-setup-steps.md)]
[!INCLUDE [digital-twins-setup-permissions.md](../../includes/digital-twins-setup-permissions.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="set-up-cloud-shell-session"></a>Configurer une session Cloud Shell
[!INCLUDE [Cloud Shell for Azure Digital Twins](../../includes/digital-twins-cloud-shell.md)]

## <a name="create-the-azure-digital-twins-instance"></a>Créer l’instance Azure Digital Twins

Dans cette section, vous allez **créer une nouvelle instance d’Azure Digital Twins** à l’aide de la commande Cloud Shell. Vous devrez fournir les éléments suivants :
* Un groupe de ressources dans lequel la déployer. Si vous n’avez pas encore choisi un groupe de ressources existant, vous pouvez en créer un maintenant avec cette commande :
    ```azurecli-interactive
    az group create --location <region> --name <name-for-your-resource-group>
    ```
* Une région pour le déploiement. Pour connaître les régions qui prennent en charge Azure Digital Twins, consultez [*Produits Azure disponibles par région*](https://azure.microsoft.com/global-infrastructure/services/?products=digital-twins).
* Un nom pour votre instance. Le nom de la nouvelle instance doit être unique dans la région pour votre abonnement (ce qui signifie que si votre abonnement a une autre instance Azure Digital Twins dans cette région, qui utilise déjà le nom que vous choisissez, vous devrez choisir un autre nom).

Utilisez ces valeurs dans la commande suivante pour créer l’instance :

```azurecli-interactive
az dt create --dt-name <name-for-your-Azure-Digital-Twins-instance> -g <your-resource-group> -l <region>
```

### <a name="verify-success-and-collect-important-values"></a>Vérifier la réussite de l’exécution et collecter les valeurs importantes

Si l’instance a été créée avec succès, le résultat dans Cloud Shell ressemble à ceci, générant des informations sur la ressource que vous avez créée :

:::image type="content" source="media/how-to-set-up-instance/cloud-shell/create-instance.png" alt-text="Fenêtre Commande montrant la création réussie d’un groupe de ressources et d’une instance Azure Digital Twins":::

Notez les valeurs *hostName* , *name* et *resourceGroup* de l’instance Azure Digital Twins fournies dans la sortie. Il s’agit de toutes les valeurs importantes dont vous pouvez avoir besoin quand vous continuez à travailler avec votre instance Azure Digital Twins pour configurer l’authentification et les ressources Azure associées. Si d’autres utilisateurs doivent programmer pour l’instance, vous devez partager ces valeurs avec eux.

> [!TIP]
> Vous pouvez afficher ces propriétés, ainsi que toutes les propriétés de votre instance, à tout moment en exécutant `az dt show --dt-name <your-Azure-Digital-Twins-instance>`.

Vous disposez maintenant d’une instance Azure Digital Twins opérationnelle. Ensuite, vous allez accorder les autorisations utilisateur Azure appropriées pour la gérer.

## <a name="set-up-user-access-permissions"></a>Configurer les autorisations d’accès utilisateur

[!INCLUDE [digital-twins-setup-role-assignment.md](../../includes/digital-twins-setup-role-assignment.md)]

Utilisez la commande suivante pour attribuer le rôle (doit être exécutée par un utilisateur avec les [autorisations suffisantes](#prerequisites-permission-requirements) dans l’abonnement Azure). La commande nécessite de passer le *nom d’utilisateur principal* sur le compte Azure AD de l’utilisateur auquel le rôle doit être attribué. Dans la plupart des cas, cela correspond à l’adresse e-mail de l’utilisateur sur le compte Azure AD.

```azurecli-interactive
az dt role-assignment create --dt-name <your-Azure-Digital-Twins-instance> --assignee "<Azure-AD-user-principal-name-of-user-to-assign>" --role "Azure Digital Twins Data Owner"
```

Les résultats de cette commande décrivent l’attribution de rôle que vous avez créée.

> [!NOTE]
> Si cette commande renvoie une erreur indiquant que l’interface CLI **ne trouve pas l’utilisateur ou le principal du service dans la base de données de graphes**  :
>
> Attribuez le rôle en utilisant à la place l’ *ID d’objet* de l’utilisateur. Cela peut se produire pour les utilisateurs disposant de [comptes Microsoft (MSA)](https://account.microsoft.com/account) personnels. 
>
> Utilisez la [page du portail Azure des utilisateurs Azure Active Directory](https://portal.azure.com/#blade/Microsoft_AAD_IAM/UsersManagementMenuBlade/AllUsers) pour sélectionner le compte d’utilisateur et afficher ses détails. Copiez la valeur *ObjectID* de l’utilisateur :
>
> :::image type="content" source="media/includes/user-id.png" alt-text="Fenêtre Commande montrant la création réussie d’un groupe de ressources et d’une instance Azure Digital Twins" lightbox="media/includes/user-id.png":::
>
> Ensuite, répétez la commande 'role assignment list' en utilisant l’ *ID d’objet* de l’utilisateur pour le paramètre `assignee` ci-dessus.

### <a name="verify-success"></a>Vérifier la réussite de l’exécution

[!INCLUDE [digital-twins-setup-verify-role-assignment.md](../../includes/digital-twins-setup-verify-role-assignment.md)]

Vous disposez maintenant d’une instance Azure Digital Twins prête à l’emploi et des autorisations pour la gérer.

## <a name="next-steps"></a>Étapes suivantes

Testez les appels d’API REST individuels sur votre instance à l’aide des commandes CLI d’Azure Digital Twins : 
* [az dt reference](/cli/azure/ext/azure-iot/dt?preserve-view=true&view=azure-cli-latest)
* [*Guide pratique : Utiliser l’interface CLI d’Azure Digital Twins*](how-to-use-cli.md)

Vous pouvez également découvrir comment connecter une application cliente à votre instance avec un code d’authentification :
* [*Guide pratique : Écrire le code d’authentification de l’application*](how-to-authenticate-client.md)
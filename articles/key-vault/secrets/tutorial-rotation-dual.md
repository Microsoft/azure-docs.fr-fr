---
title: Tutoriel sur la rotation des ressources avec deux jeux d’informations d’identification
description: Utilisez ce tutoriel pour découvrir comment automatiser la rotation d’un secret pour des ressources qui utilisent deux jeux d’informations d’authentification.
services: key-vault
author: msmbaldwin
manager: rkarlin
tags: rotation
ms.service: key-vault
ms.subservice: secrets
ms.topic: tutorial
ms.date: 06/22/2020
ms.author: jalichwa
ms.openlocfilehash: 72541b8d8f8d8865c680c36f7f84cd91a4ce8ba2
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96903324"
---
# <a name="automate-the-rotation-of-a-secret-for-resources-that-have-two-sets-of-authentication-credentials"></a>Automatiser la permutation d’un secret pour des ressources qui ont deux jeux d’informations d’authentification

La meilleure façon de s’authentifier auprès des services Azure consiste à utiliser une [identité managée](../general/authentication.md). Il existe toutefois des scénarios dans lesquels cette approche est inappropriée. Dans ces cas-là, des clés d’accès ou des mots de passe sont utilisés. Vous devez fréquemment permuter les clés d’accès et les mots de passe.

Ce tutoriel montre comment automatiser la rotation régulière des secrets pour les bases de données et les services qui utilisent deux jeux d’informations d’authentification. Plus précisément, ce tutoriel montre comment permuter les clés de compte de stockage Azure stockées dans Azure Key Vault en tant que secrets. Vous utiliserez une fonction déclenchée par une notification Azure Event Grid. 

> [!NOTE]
> Les clés de compte de stockage peuvent être gérées automatiquement dans Key Vault si vous fournissez des jetons de signature d’accès partagé pour l’accès délégué au compte de stockage. Il existe des services qui nécessitent des chaînes de connexion de compte de stockage avec des clés d’accès. Pour ce scénario, nous vous recommandons cette solution.

Voici la solution de permutation décrite dans ce tutoriel : 

![Diagramme illustrant la solution de permutation](../media/secrets/rotation-dual/rotation-diagram.png)

Dans cette solution, Azure Key Vault stocke les clés d’accès individuelles de compte de stockage sous la forme de versions du même secret en alternant entre la clé primaire et la clé secondaire dans les versions qui se suivent. Quand une clé d’accès est stockée dans la dernière version du secret, la clé secondaire est regénérée et ajoutée à Key Vault en tant que nouvelle dernière version du secret. La solution fournit le cycle de permutation complet de l’application pour actualiser jusqu’à la clé regénérée la plus récente. 

1. Trente jours avant la date d’expiration d’un secret, Key Vault publie l’événement d’expiration proche sur Event Grid.
1. Event Grid vérifie les abonnements aux événements et utilise HTTP POST pour appeler le point de terminaison de l’application de fonction qui est abonné à l’événement.
1. L’application de fonction identifie la clé secondaire (pas la dernière) et appelle le compte de stockage pour la regénérer.
1. L’application de fonction ajoute la nouvelle clé regénérée à Azure Key Vault en tant que nouvelle version du secret.

## <a name="prerequisites"></a>Prérequis
* Un abonnement Azure. [Créez-en un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Azure Key Vault.
* Deux comptes de stockage Azure.

Vous pouvez utiliser ce lien de déploiement si vous n’avez pas de coffre de clés ou de comptes de stockage existants :

[![Lien intitulé Deploy to Azure](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FInitial-Setup%2Fazuredeploy.json)

1. Sous **Groupe de ressources**, sélectionnez **Créer**. Nommez le groupe **akvrotation**, puis sélectionnez **OK**.
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

    ![Capture d’écran montrant comment créer un groupe de ressources.](../media/secrets/rotation-dual/dual-rotation-1.png)

Vous disposez maintenant d’un coffre de clés et de deux comptes de stockage. Vous pouvez vérifier cette configuration dans l’interface de ligne de commande (CLI) Azure en exécutant cette commande :

```azurecli
az resource list -o table -g akvrotation
```

Le résultat doit ressembler à la sortie suivante :

```console
Name                     ResourceGroup         Location    Type                               Status
-----------------------  --------------------  ----------  ---------------------------------  --------
akvrotation-kv         akvrotation      eastus      Microsoft.KeyVault/vaults
akvrotationstorage     akvrotation      eastus      Microsoft.Storage/storageAccounts
akvrotationstorage2    akvrotation      eastus      Microsoft.Storage/storageAccounts
```

## <a name="create-and-deploy-the-key-rotation-function"></a>Créer et déployer la fonction de permutation de clé

Ensuite, vous allez créer une application de fonction avec une identité managée par le système en plus d’autres composants nécessaires. Vous allez également déployer la fonction de permutation pour les clés de compte de stockage.

La fonction de permutation de l’application de fonction nécessite les composants et la configuration suivants :
- Un plan Azure App Service
- Un compte de stockage pour gérer les déclencheurs de l’application de fonction
- Une stratégie d’accès pour accéder aux secrets dans Key Vault
- Le rôle de service d’opérateur de clé de compte de stockage affecté à l’application de fonction afin qu’elle puisse accéder aux clés d’accès de compte de stockage
- Une fonction de permutation de clé avec un déclencheur d’événement et un déclencheur HTTP (permutation à la demande)
- Un abonnement aux événements Event Grid pour l’événement **SecretNearExpiry**

1. Sélectionnez le lien de déploiement d’un modèle Azure : 

   [![Lien de déploiement d’un modèle Azure.](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FFunction%2Fazuredeploy.json)

1. Dans la liste **Groupe de ressources**, sélectionnez **akvrotation**.
1. Dans la zone **Groupe de ressources du compte de stockage**, entrez le nom du groupe de ressources dans lequel se trouve votre compte de stockage. Conservez la valeur par défaut **[resourceGroup().name]** si votre compte de stockage se trouve déjà dans le même groupe de ressources que celui dans lequel vous allez déployer la fonction de permutation de clés.
1. Dans la zone **Nom du compte de stockage**, entrez le nom du compte de stockage qui contient les clés d’accès à permuter.
1. Dans la zone **Groupe de ressources du coffre de clés**, entrez le nom du groupe de ressources dans lequel se trouve votre coffre de clés. Conservez la valeur par défaut **[resourceGroup().name]** si votre coffre de clés existe déjà dans le même groupe de ressources que celui dans lequel vous allez déployer la fonction de permutation de clés.
1. Dans la zone **Nom du coffre de clés**, entrez le nom du coffre de clés.
1. Dans la zone **Nom de l’application de fonction**, entrez le nom de l’application de fonction.
1. Dans la zone **Nom du secret**, entrez le nom du secret dans lequel vous allez stocker les clés d’accès.
1. Dans la zone **URL du dépôt**, entrez l’emplacement GitHub du code de la fonction : **https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell.git** .
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

   ![Capture d’écran montrant comment créer le premier compte de stockage.](../media/secrets/rotation-dual/dual-rotation-2.png)

Après avoir effectué les étapes précédentes, vous disposez d’un compte de stockage, d’une batterie de serveurs, d’une application de fonction et d’Application Insights. Une fois le déploiement terminé, la page suivante s’affiche : ![Capture d’écran montrant la page Votre déploiement a été effectué.](../media/secrets/rotation-dual/dual-rotation-3.png)
> [!NOTE]
> En cas de problème, vous pouvez sélectionner **Redéployer** pour terminer le déploiement des composants.


Vous trouverez des modèles de déploiement et du code pour la fonction de permutation sur [GitHub](https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell).

## <a name="add-the-storage-account-access-keys-to-key-vault"></a>Ajouter les clés d’accès de compte de stockage dans Key Vault

Commencez par définir votre stratégie d’accès pour accorder des autorisations **Gérer les secrets** aux utilisateurs :

```azurecli
az keyvault set-policy --upn <email-address-of-user> --name akvrotation-kv --secret-permissions set delete get list
```

Vous pouvez maintenant créer un secret ayant pour valeur une clé d’accès de compte de stockage. Vous avez également besoin de l’ID de ressource du compte de stockage, de la période de validité du secret et de l’ID de clé à ajouter au secret, afin que la fonction de permutation puisse regénérer la clé dans le compte de stockage.

Déterminez l’ID de ressource du compte de stockage. Vous pouvez trouver cette valeur dans la propriété `id`.
```azurecli
az storage account show -n akvrotationstorage
```

Listez les clés d’accès de compte de stockage afin de pouvoir récupérer les valeurs des clés :

```azurecli
az storage account keys list -n akvrotationstorage 
```

Exécutez cette commande, en utilisant les valeurs récupérées pour `key1Value` et `storageAccountResourceId` :

```azurecli
$tomorrowDate = (get-date).AddDays(+1).ToString("yyy-MM-ddThh:mm:ssZ")
az keyvault secret set --name storageKey --vault-name akvrotation-kv --value <key1Value> --tags "CredentialId=key1" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```

Si vous créez un secret avec une date d’expiration proche, un événement `SecretNearExpiry` est publié dans un délai de plusieurs minutes. Cet événement déclenche à son tour la fonction pour permuter le secret.

Vous pouvez vérifier que les clés d’accès ont été regénérées en récupérant la clé de compte de stockage et le secret Key Vault et en les comparant.

Utilisez cette commande pour obtenir des informations sur le secret :
```azurecli
az keyvault secret show --vault-name akvrotation-kv --name storageKey
```
Remarquez que `CredentialId` est mis à jour vers l’autre `keyName` et que `value` est regénérée : ![Capture d’écran montrant la sortie de la commande az keyvault secret show pour le premier compte de stockage.](../media/secrets/rotation-dual/dual-rotation-4.png)

Récupérez les clés d’accès pour comparer les valeurs :
```azurecli
az storage account keys list -n akvrotationstorage 
```
![Capture d’écran montrant la sortie de la commande az storage account keys list pour le premier compte de stockage.](../media/secrets/rotation-dual/dual-rotation-5.png)

## <a name="add-storage-accounts-for-rotation"></a>Ajouter des comptes de stockage pour la permutation

Vous pouvez réutiliser la même application de fonction pour permuter les clés de plusieurs comptes de stockage. 

Pour ajouter des clés de compte de stockage à une fonction existante en vue d’une permutation, vous avez besoin des éléments suivants :
- Le rôle de service d’opérateur de clé de compte de stockage affecté à l’application de fonction afin qu’elle puisse accéder aux clés d’accès de compte de stockage
- Un abonnement aux événements Event Grid pour l’événement **SecretNearExpiry**

1. Sélectionnez le lien de déploiement d’un modèle Azure : 

   [![Lien de déploiement d’un modèle Azure.](https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/1-CONTRIBUTION-GUIDE/images/deploytoazure.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fjlichwa%2FKeyVault-Rotation-StorageAccountKey-PowerShell%2Fmaster%2Farm-templates%2FAdd-Event-Subscriptions%2Fazuredeploy.json)

1. Dans la liste **Groupe de ressources**, sélectionnez **akvrotation**.
1. Dans la zone **Nom du compte de stockage**, entrez le nom du compte de stockage qui contient les clés d’accès à permuter.
1. Dans la zone **Nom du coffre de clés**, entrez le nom du coffre de clés.
1. Dans la zone **Nom de l’application de fonction**, entrez le nom de l’application de fonction.
1. Dans la zone **Nom du secret**, entrez le nom du secret dans lequel vous allez stocker les clés d’accès.
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

   ![Capture d’écran montrant comment créer un compte de stockage supplémentaire.](../media/secrets/rotation-dual/dual-rotation-7.png)

### <a name="add-another-storage-account-access-key-to-key-vault"></a>Ajouter une autre clé d’accès de compte de stockage dans Key Vault

Déterminez l’ID de ressource du compte de stockage. Vous pouvez trouver cette valeur dans la propriété `id`.
```azurecli
az storage account show -n akvrotationstorage2
```

Listez les clés d’accès de compte de stockage afin de pouvoir récupérer la valeur key2 :

```azurecli
az storage account keys list -n akvrotationstorage2 
```

Exécutez cette commande, en utilisant les valeurs récupérées pour `key2Value` et `storageAccountResourceId` :

```azurecli
tomorrowDate=`date -d tomorrow -Iseconds -u | awk -F'+' '{print $1"Z"}'`
az keyvault secret set --name storageKey2 --vault-name akvrotation-kv --value <key2Value> --tags "CredentialId=key2" "ProviderAddress=<storageAccountResourceId>" "ValidityPeriodDays=60" --expires $tomorrowDate
```

Utilisez cette commande pour obtenir des informations sur le secret :
```azurecli
az keyvault secret show --vault-name akvrotation-kv --name storageKey2
```
Remarquez que `CredentialId` est mis à jour vers l’autre `keyName` et que `value` est regénérée : ![Capture d’écran montrant la sortie de la commande az keyvault secret show pour le second compte de stockage.](../media/secrets/rotation-dual/dual-rotation-8.png)

Récupérez les clés d’accès pour comparer les valeurs :
```azurecli
az storage account keys list -n akvrotationstorage 
```
![Capture d’écran montrant la sortie de la commande az storage account keys list pour le second compte de stockage.](../media/secrets/rotation-dual/dual-rotation-9.png)

## <a name="key-vault-dual-credential-rotation-functions"></a>Fonctions de permutation des informations d’identification doubles Key Vault

- [Compte de stockage](https://github.com/jlichwa/KeyVault-Rotation-StorageAccountKey-PowerShell)
- [Cache Redis](https://github.com/jlichwa/KeyVault-Rotation-RedisCacheKey-PowerShell)

## <a name="next-steps"></a>Étapes suivantes
- Présentation : [Supervision de Key Vault avec Azure Event Grid](../general/event-grid-overview.md)
- Procédure : [Créer votre première fonction sur le Portail Azure](../../azure-functions/functions-create-first-azure-function.md)
- Procédure : [Recevoir un e-mail en cas de changement d’un secret Key Vault](../general/event-grid-logicapps.md)
- Référence : [Schéma des événements Azure Event Grid pour Azure Key Vault](../../event-grid/event-schema-key-vault.md)

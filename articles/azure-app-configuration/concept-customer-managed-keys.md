---
title: Utiliser des clés gérées par le client pour chiffrer vos données de configuration
description: Chiffrer vos données de configuration avec des clés gérées par le client
author: AlexandraKemperMS
ms.author: alkemper
ms.date: 07/28/2020
ms.topic: conceptual
ms.service: azure-app-configuration
ms.openlocfilehash: 46a1e99a7bc75efa85fcb5eff649c14af5abd17b
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96930498"
---
# <a name="use-customer-managed-keys-to-encrypt-your-app-configuration-data"></a>Utiliser des clés gérées par le client pour chiffrer vos données App Configuration
Azure App Configuration [chiffre les informations sensibles au repos](../security/fundamentals/encryption-atrest.md). L’utilisation de clés gérées par le client offre une protection améliorée des données en vous permettant de gérer vos clés de chiffrement.  Quand un chiffrement avec des clés gérées est utilisé, toutes les informations sensibles dans App Configuration sont chiffrées avec une clé Azure Key Vault fournie par l’utilisateur.  Ceci donne la possibilité d’effectuer une rotation de la clé de chiffrement à la demande.  Ceci permet aussi de révoquer l’accès d’Azure App Configuration à des informations sensibles en révoquant l’accès à la clé par l’instance App Configuration.

## <a name="overview"></a>Vue d’ensemble 
Azure App Configuration chiffre les informations sensibles au repos en utilisant une clé de chiffrement AES 256 bits fournie par Microsoft. Chaque instance App Configuration a sa propre clé de chiffrement gérée par le service et utilisée pour chiffrer les informations sensibles. Les informations sensibles incluent les valeurs qui se trouvent dans les paires clé-valeur.  Quand la fonctionnalité de clé gérée par le client est activée, App Configuration utilise une identité managée affectée à l’instance App Configuration pour s’authentifier auprès d’Azure Active Directory. L’identité managée appelle ensuite Azure Key Vault et encapsule la clé de chiffrement de l’instance App Configuration. La clé de chiffrement encapsulée est ensuite stockée et la clé de chiffrement désencapsulée est mise en cache pour une heure dans App Configuration. App Configuration actualise toutes les heures la version désencapsulée de la clé de chiffrement de l’instance App Configuration. Ceci garantit la disponibilité dans des conditions de fonctionnement normales. 

>[!IMPORTANT]
> Si l’identité affectée à l’instance App Configuration n’est plus autorisée à désencapsuler la clé de chiffrement de l’instance, ou si la clé gérée est supprimée définitivement, il n’est plus possible de déchiffrer les informations sensibles stockées dans l’instance App Configuration. L’utilisation de la fonction de [suppression réversible](../key-vault/general/soft-delete-overview.md) d’Azure Key Vault réduit le risque de suppression accidentelle de votre clé de chiffrement.

Quand des utilisateurs activent la fonctionnalité de clé gérée par le client sur leur instance Azure App Configuration, ils contrôlent la capacité du service à accéder à leurs informations sensibles. La clé gérée sert de clé de chiffrement racine. Un utilisateur peut révoquer l’accès de son instance App Configuration à sa clé gérée en changeant sa stratégie d’accès au coffre de clés. Quand cet accès est révoqué, App Configuration perd la capacité à déchiffrer les données utilisateur dans un délai d’une heure. À ce stade, l’instance App Configuration interdit toutes les tentatives d’accès. Cette situation est récupérable en accordant à nouveau au service l’accès à la clé gérée.  Dans un délai d’une heure, App Configuration est en mesure de déchiffrer les données utilisateur et de fonctionner dans des conditions normales.

>[!NOTE]
>Toutes les données Azure App Configuration sont stockées pendant jusqu’à 24 heures dans une sauvegarde isolée. Ceci inclut la clé de chiffrement désencapsulée. Ces données ne sont pas immédiatement disponibles pour le service ou l’équipe du service. Dans le cas d’une restauration d’urgence, Azure App Configuration s’autorévoque des données de clé gérée.

## <a name="requirements"></a>Spécifications
Les composants suivants sont nécessaires pour activer la fonctionnalité de clé gérée par le client pour la Azure App Configuration :
- Instance Azure App Configuration de niveau standard
- Azure Key Vault avec les fonctionnalités de suppression réversible et de protection contre le vidage activées
- Une clé RSA ou RSA-HSM au sein du coffre de clés
    - La clé ne doit pas avoir expiré, elle doit être activée, et les fonctionnalités d’encapsulation et de désencapsulation doivent être activées

Une fois ces ressources configurées, il reste deux étapes pour autoriser Azure App Configuration à utiliser la clé Key Vault :
1. Affecter une identité managée à l’instance Azure App Configuration
2. Accordez les autorisations `GET`, `WRAP`et `UNWRAP` à l’identité dans la stratégie d’accès du coffre de clés cible.

## <a name="enable-customer-managed-key-encryption-for-your-azure-app-configuration-instance"></a>Activer le chiffrement avec une clé géré par le client pour votre instance Azure App Configuration
Pour commencer, vous avec besoin d’une instance Azure App Configuration correctement configurée. Si vous n’avez pas encore d’instance Azure App Configuration disponible, suivez un de ces guides de démarrage rapide pour en configurer une :
- [Créer une application ASP.NET Core avec Azure App Configuration](quickstart-aspnet-core-app.md)
- [Créer une application .NET Core avec Azure App Configuration](quickstart-dotnet-core-app.md)
- [Créer une application .NET Framework avec Azure App Configuration](quickstart-dotnet-app.md)
- [Créer une application Java Spring avec Azure App Configuration](quickstart-java-spring-app.md)

>[!TIP]
> Azure Cloud Shell est un interpréteur de commandes interactif gratuit qui vous permet d’exécuter les instructions de ligne de commande de cet article.  Les outils Azure les plus courants sont préinstallés, y compris le SDK .NET Core. Si vous êtes connecté à votre abonnement Azure, lancez [Azure Cloud Shell](https://shell.azure.com) à partir de shell.azure.com.  Pour en savoir plus sur Azure Cloud Shell, consultez [notre documentation](../cloud-shell/overview.md).

### <a name="create-and-configure-an-azure-key-vault"></a>Créer et configurer un coffre Azure Key Vault
1. Créez un coffre Azure Key Vault avec Azure CLI.  Notez que `vault-name` et `resource-group-name` sont fournis par l’utilisateur et doivent être uniques.  Dans ces exemples, nous utilisons `contoso-vault` et `contoso-resource-group`.

    ```azurecli
    az keyvault create --name contoso-vault --resource-group contoso-resource-group
    ```
    
1. Activez la suppression réversible et la protection contre le vidage pour le coffre de clés. Remplacez les espaces réservés par les noms du coffre de clés (`contoso-vault`) et du groupe de ressources (`contoso-resource-group`) créés à l’étape 1.

    ```azurecli
    az keyvault update --name contoso-vault --resource-group contoso-resource-group --enable-purge-protection --enable-soft-delete
    ```
    
1. Créez une clé Key Vault. Spécifiez un `key-name` unique pour cette clé et remplacez l’espace réservé par le nom du coffre de clés (`contoso-vault`) créé à l’étape 1. Spécifiez si vous préférez un chiffrement `RSA` ou `RSA-HSM`.

    ```azurecli
    az keyvault key create --name key-name --kty {RSA or RSA-HSM} --vault-name contoso-vault
    ```
    
    La sortie de cette commande montre l’ID de clé (« kid ») pour la clé générée.  Notez l’ID de clé pour pouvoir l’utiliser plus tard dans cet exemple.  Cet ID a la forme : `https://{my key vault}.vault.azure.net/keys/{key-name}/{Key version}`.  L’ID de clé a trois composants importants :
    1. URI du coffre de clés : `https://{my key vault}.vault.azure.net
    1. Nom de la clé du coffre de clés : {key-name}
    1. Version de la clé du coffre de clés : {Key version}

1. Créez une identité managée affectée par le système avec Azure CLI, en utilisant le nom de votre instance App Configuration et le nom groupe de ressources des étapes précédentes. L’identité managée sera utilisée pour accéder à la clé gérée. Nous utilisons `contoso-app-config` pour illustrer le nom d’une instance App Configuration :
    
    ```azurecli
    az appconfig identity assign --name contoso-app-config --resource-group contoso-resource-group --identities [system]
    ```
    
    La sortie de cette commande comprend l’ID du principal (« principalId ») et l’ID de locataire (« tenandId ») de l’identité affectée par le système.  Ces ID seront utilisés pour accorder à l’identité l’accès à la clé gérée.

    ```json
    {
    "principalId": {Principal Id},
    "tenantId": {Tenant Id},
    "type": "SystemAssigned",
    "userAssignedIdentities": null
    }
    ```

1. L’identité managée de l’instance Azure App Configuration doit accéder à la clé pour effectuer la validation des clés, le chiffrement et le déchiffrement. L’ensemble d’actions spécifique auquel il doit accéder inclut : `GET`, `WRAP` et `UNWRAP` pour les clés.  L’octroi de l’accès nécessite l’ID du principal de l’identité managée de l’instance App Configuration. Cette valeur a été obtenue à l’étape précédente. Elle figure ci-dessous en tant que `contoso-principalId`. Accordez l’autorisation à la clé gérée en utilisant la ligne de commande :

    ```azurecli
    az keyvault set-policy -n contoso-vault --object-id contoso-principalId --key-permissions get wrapKey unwrapKey
    ```

1. Une fois que l’instance Azure App Configuration peut accéder à la clé gérée, nous pouvons activer la fonctionnalité de clé gérée par le client dans le service en utilisant Azure CLI. Rappelez-vous des propriétés suivantes enregistrées lors des étapes de création de la clé : `key name` `key vault URI`.

    ```azurecli
    az appconfig update -g contoso-resource-group -n contoso-app-config --encryption-key-name key-name --encryption-key-version key-version --encryption-key-vault key-vault-Uri
    ```

Votre instance Azure App Configuration est maintenant configurée pour utiliser une clé gérée par le client stockée dans Azure Key Vault.

## <a name="next-steps"></a>Étapes suivantes
Dans cet article, vous avez configuré votre instance Azure App Configuration pour utiliser une clé gérée par le client pour le chiffrement.  Découvrez comment [intégrer votre service aux identités managées Azure](howto-integrate-azure-managed-service-identity.md).

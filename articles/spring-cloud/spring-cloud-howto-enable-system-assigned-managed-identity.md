---
title: Activer une identité managée affectée par le système pour une application Azure Spring Cloud
description: Activer une identité managée affectée par le système pour une application
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: how-to
ms.date: 05/13/2020
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 2fbdd46b872c4c70999355d457045f4aac8aa34c
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92742804"
---
# <a name="how-to-enable-system-assigned-managed-identity-for-azure-spring-cloud-application"></a>Activer une identité managée affectée par le système pour une application Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java ✔️ C#

Les identités managées pour les ressources Azure fournissent une identité automatiquement managée dans Azure Active Directory à une ressource Azure telle que votre application Azure Spring Cloud. Vous pouvez utiliser cette identité pour vous authentifier sur n’importe quel service prenant en charge l’authentification Azure AD, sans avoir d’informations d’identification dans votre code.

Cet article explique comment activer et désactiver les identités managées affectées par le système pour une application Azure Spring Cloud, à l'aide du portail Azure et de l'interface CLI (disponible à partir de la version 0.2.4).

## <a name="prerequisites"></a>Prérequis
Si nécessaire, consultez la [section de présentation](../active-directory/managed-identities-azure-resources/overview.md) afin de vous familiariser avec les identités managées pour les ressources Azure.
Une instance d'Azure Spring Cloud doit être déployée. Suivez le [Guide de démarrage Déployer à l'aide d'Azure CLI](spring-cloud-quickstart.md).

## <a name="add-a-system-assigned-identity"></a>Ajouter une identité affectée par le système
La création d'une application avec une identité affectée par le système requiert la définition d'une propriété supplémentaire sur cette application.

### <a name="using-azure-portal"></a>En passant par le portail Azure
Pour configurer une identité managée à partir du [portail Azure](https://portal.azure.com/), commencez par créer une application, puis activez la fonctionnalité.

1. Créez une application dans le portail, comme vous le feriez normalement. Accédez-y dans le portail.
2. Faites défiler l'écran jusqu'au groupe **Paramètres** dans le volet de navigation gauche.
3. Sélectionnez **Identité** .
4. Dans l’onglet **Attribuée par le système** , définissez **État** sur *Activé* . Cliquez sur **Enregistrer** .

 ![Identité managée sur le portail](./media/spring-cloud-managed-identity/identity-1.png)

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure
Vous pouvez activer une identité managée affectée par le système pendant la création d'une application ou sur une application existante.

**Activer une identité managée affectée par le système lors de la création d'une application**

L'exemple suivant crée une application nommée *app_name* avec une identité managée affectée par le système, comme le demande le paramètre `--assign-identity`.

```azurecli
az spring-cloud app create -n app_name -s service_name -g resource_group_name --assign-identity
```

**Activer une identité managée affectée par le système sur une application existante**  Utilisez la commande `az spring-cloud app identity assign` pour activer l'identité affectée par le système sur une application existante.

```azurecli
az spring-cloud app identity assign -n app_name -s service_name -g resource_group_name
```

## <a name="obtain-tokens-for-azure-resources"></a>Obtenir des jetons pour les ressources Azure
Une application peut utiliser son identité managée pour obtenir des jetons afin d’accéder à d’autres ressources protégées par Azure Active Directory, comme Azure Key Vault. Ces jetons représentent l'application qui accède à la ressource, et non un utilisateur spécifique de l'application.

Vous pouvez être amené à [configurer la ressource cible pour autoriser l'accès à partir de votre application](../active-directory/managed-identities-azure-resources/howto-assign-access-portal.md). Par exemple, si vous demandez un jeton pour accéder à Key Vault, vérifiez que vous avez ajouté une stratégie d'accès qui inclut l'identité de votre application. Si tel n’est pas le cas, vos appels au coffre de clés sont rejetés, même s’ils incluent le jeton. Pour en savoir plus sur les ressources qui prennent en charge les jetons Azure Active Directory, consultez [Services Azure prenant en charge l’authentification Azure AD](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-azure-ad-authentication).

Azure Spring Cloud partage le même point de terminaison pour l'acquisition de jetons auprès d'Azure Virtual Machine. Nous vous recommandons d’utiliser le SDK Java ou des instances Spring Boot Starters pour acquérir un jeton.  Consultez [Utiliser le jeton de la machine virtuelle](../active-directory/managed-identities-azure-resources/how-to-use-vm-token.md) pour accéder à des exemples de code et de script ainsi qu'à des conseils sur des sujets importants tels que la gestion des erreurs HTTP et de l'expiration des jetons.

Recommandé : utilisez le kit de développement logiciel (SDK) Java ou des « starters » Spring Boot pour obtenir des jetons.  Consultez les exemples fournis dans [Étapes suivantes](#next-steps).

## <a name="disable-system-assigned-identity-from-an-app"></a>Désactiver l'identité affectée par le système d'une application
Si vous supprimez une identité affectée par le système, vous la supprimez également d'Azure AD. La suppression de la ressource d'application supprime automatiquement les identités affectées par le système d'Azure AD.

### <a name="using-azure-portal"></a>En passant par le portail Azure
Pour supprimer l'identité managée affectée par le système d'une application qui n'en a plus besoin :

1. Connectez-vous au [portail Azure](https://portal.azure.com/) à l'aide d'un compte associé à l'abonnement Azure qui contient l'instance d'Azure Spring Cloud.
1. Accédez à la machine virtuelle souhaitée et sélectionnez **Identité** .
1. Sous **Affectée par le système**/**État** , sélectionnez **Désactivé** , puis cliquez sur **Enregistrer**  :

 ![Identité managée](./media/spring-cloud-managed-identity/remove-identity.png)

### <a name="using-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure
Pour supprimer l'identité managée affectée par le système d'une application qui n'en a plus besoin, utilisez la commande suivante :
```azurecli
az spring-cloud app identity remove -n app_name -s service_name -g resource_group_name
```

## <a name="next-steps"></a>Étapes suivantes

* [Accéder à Azure Key Vault avec des identités managées dans un « starter » Spring Boot](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/spring/azure-spring-boot-starter-keyvault-secrets/README.md#use-msi--managed-identities)
* [En savoir plus sur les identités managées pour les ressources Azure](https://github.com/MicrosoftDocs/azure-docs/blob/master/articles/active-directory/managed-identities-azure-resources/overview.md)
* [Utiliser des identités managées avec le kit de développement logiciel (SDK) Java](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples)

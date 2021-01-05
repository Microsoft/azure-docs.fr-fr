---
title: Options d’authentification du Registre
description: Options d’authentification pour un registre de conteneurs Azure privé, y compris la connexion auprès d’une identité Azure Active Directory, au moyen de principaux de service et à l’aide d’informations d’identification d’administrateur facultatives.
ms.topic: article
ms.date: 01/30/2020
ms.openlocfilehash: 5315c11e0f1e2c859384e3783ae4be5d709adb42
ms.sourcegitcommit: dbe434f45f9d0f9d298076bf8c08672ceca416c6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/17/2020
ms.locfileid: "92148557"
---
# <a name="authenticate-with-an-azure-container-registry"></a>Authentification avec un registre de conteneurs Azure

Plusieurs méthodes permettent de s’authentifier avec un registre de conteneurs Azure, chacune d’elles étant applicable à un ou plusieurs scénarios d’usage du registre.

Les méthodes recommandées incluent l’authentification à un registre par le biais d’une [connexion individuelle](#individual-login-with-azure-ad), sinon vos applications et orchestrateurs de conteneurs peuvent procéder à une authentification sans assistance, ou « headless » (administrée à distance), à l’aide d’un [principal de service](#service-principal) Azure Active Directory (Azure AD).

## <a name="authentication-options"></a>Options d’authentification

Le tableau suivant liste les méthodes d’authentification disponibles et les scénarios typiques. Pour plus d’informations, suivez les liens.

| Méthode                               | Comment s’authentifier                                           | Scénarios                                                            | Contrôle d’accès en fonction du rôle Azure (Azure RBAC)                             | Limites                                |
|---------------------------------------|-------------------------------------------------------|---------------------------------------------------------------------|----------------------------------|--------------------------------------------|
| [Identité AD individuelle](#individual-login-with-azure-ad)                 | `az acr login` dans Azure CLI                              | Push/pull interactif par des développeurs ou des testeurs                                    | Oui                              | Le jeton AD doit être renouvelé toutes les 3 heures     |
| [Principal de service AD](#service-principal)                   | `docker login`<br/><br/>`az acr login` dans Azure CLI<br/><br/> Paramètres de connexion au registre dans les API ou les outils<br/><br/> [Secret d’extraction Kubernetes](container-registry-auth-kubernetes.md)                                               | Push sans assistance à partir d’un pipeline CI/CD<br/><br/> Pull sans assistance vers Azure ou des services externes  | Oui                              | Le mot de passe SP par défaut est valable 1 an       |                                                           
| [Intégrer à AKS](../aks/cluster-container-registry-integration.md?toc=/azure/container-registry/toc.json&bc=/azure/container-registry/breadcrumb/toc.json)                     | Attacher le registre quand un cluster AKS est créé ou mis à jour  | Pull sans assistance vers un cluster AKS                                                  | Non, accès pull uniquement             | Disponible uniquement avec le cluster AKS            |
| [Identité managée pour les ressources Azure](container-registry-authentication-managed-identity.md)   | `docker login`<br/><br/> `az acr login` dans Azure CLI                                       | Push sans assistance à partir d’un pipeline CI/CD Azure<br/><br/> Pull sans assistance vers les services Azure<br/><br/>   | Oui                              | À utiliser uniquement à partir de services Azure [prenant en charge les identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/services-support-managed-identities.md#azure-services-that-support-managed-identities-for-azure-resources)              |
| [Utilisateur administrateur](#admin-account)                             | `docker login`                                          | Push/pull interactif par un développeur ou un testeur individuel<br/><br/>Déploiement du portail de l’image à partir du registre vers Azure App Service ou Azure Container Instances                      | Non, toujours un accès pull et push  | Compte unique par registre, non recommandé pour plusieurs utilisateurs         |
| [Jeton d’accès délimité au dépôt](container-registry-repository-scoped-permissions.md)                | `docker login`<br/><br/>`az acr login` dans Azure CLI   | Push/pull interactif dans le dépôt par un développeur ou un testeur individuel<br/><br/> Push/pull sans assistance vers le dépôt par un système individuel ou un périphérique externe                  | Oui                              | Pas actuellement intégré à l’identité AD  |

## <a name="individual-login-with-azure-ad"></a>Connexion individuelle avec Azure AD

Si vous utilisez directement votre registre, par exemple si vous extrayez des images et en envoyez depuis une station de travail de développement vers un registre que vous avez créé, authentifiez-vous à l’aide de votre identité Azure individuelle. Connectez-vous à [Azure CLI](/cli/azure/install-azure-cli) avec [az login](/cli/azure/reference-index#az-login), puis exécutez la commande [az acr login](/cli/azure/acr#az-acr-login) :

```azurecli
az login
az acr login --name <acrName>
```

Si vous vous connectez avec `az acr login`, l’interface CLI utilise le jeton créé lorsque vous avez exécuté `az login` pour authentifier en toute transparence votre session avec votre registre. Pour terminer le flux d’authentification, le client Docker doit être installé et en cours d’exécution dans votre environnement. `az acr login` utilise le client Docker pour définir un jeton Azure Active Directory dans le fichier `docker.config`. Lorsque vous vous connectez à l'aide de cette méthode, vos informations d'identification sont mises en cache, et les commandes `docker` suivantes de votre session ne nécessitent ni nom d'utilisateur ni mot de passe.

> [!TIP]
> Utilisez également `az acr login` pour authentifier une identité individuelle quand vous souhaitez envoyer (push) ou tirer (pull) des artefacts autres que des images Docker dans votre registre, par exemple des [artefacts OCI](container-registry-oci-artifacts.md).  

Pour l’accès au registre, le jeton utilisé par `az acr login` est valable **trois heures**. Nous vous recommandons donc de toujours vous connecter au registre avant d’exécuter une commande `docker`. Si votre jeton arrive à expiration, vous pouvez l’actualiser en utilisant de nouveau la commande `az acr login` pour la réauthentification. 

L’utilisation de `az acr login` avec des identités Azure fournit un [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/role-assignments-portal.md). Pour certains scénarios, vous souhaiterez peut-être vous connecter à un registre avec votre propre identité dans Azure AD ou configurer d’autres utilisateurs Azure avec des [autorisations et des rôles Azure](container-registry-roles.md) spécifiques. Pour les scénarios entre les services ou pour gérer les besoins d’un groupe de travail ou d’un workflow de développement où vous ne souhaitez pas gérer l’accès individuel, vous pouvez également vous connecter avec une [identité managée pour les ressources Azure](container-registry-authentication-managed-identity.md).

### <a name="az-acr-login-with---expose-token"></a>az acr login with --expose-token

Dans certains cas, vous devrez peut-être vous authentifier avec `az acr login` lorsque le démon Docker n’est pas en cours d’exécution dans votre environnement. Par exemple, vous devrez peut-être exécuter `az acr login` dans un script dans Azure Cloud Shell, qui fournit l’interface de commande Docker, mais n’exécute pas le démon Docker.

Pour ce scénario, exécutez d’abord `az acr login` avec le paramètre `--expose-token`. Cette option expose un jeton d’accès au lieu de se connecter par le biais de l’interface de commande Docker.

```azurecli
az acr login --name <acrName> --expose-token
```

La sortie affiche le jeton d’accès, abrégé ici :

```console
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs[...]24V7wA",
  "loginServer": "myregistry.azurecr.io"
}
``` 

Ensuite, exécutez `docker login`, en transmettant `00000000-0000-0000-0000-000000000000` comme nom d’utilisateur et en utilisant le jeton d’accès comme mot de passe :

```console
docker login myregistry.azurecr.io --username 00000000-0000-0000-0000-000000000000 --password eyJhbGciOiJSUzI1NiIs[...]24V7wA
```

## <a name="service-principal"></a>Principal du service

Si vous affectez un [principal du service](../active-directory/develop/app-objects-and-service-principals.md) à votre registre, votre application ou service peut l’utiliser pour une authentification headless (administrée à distance). Les principaux du service autorisent le [contrôle d’accès en fonction du rôle (Azure RBAC)](../role-based-access-control/role-assignments-portal.md) à un registre, et vous pouvez affecter plusieurs principaux du service à un registre. Plusieurs principaux du service vous permettent de définir différents accès pour plusieurs applications.

Les rôles disponibles pour un registre de conteneurs sont les suivants :

* **AcrPull** : extraction

* **AcrPush**: extraction (pull) et envoi (push)

* **Propriétaire** : extraction, envoi et attribution de rôles à d’autres utilisateurs

Pour obtenir la liste complète des rôles, consultez [Autorisations et rôles Azure Container Registry](container-registry-roles.md).

Pour que les scripts CLI créent un principal de service pour l’authentification avec un registre de conteneurs Azure, et pour obtenir une aide supplémentaire, consultez [Authentification Azure Container Registry avec des principaux de service](container-registry-auth-service-principal.md).

## <a name="admin-account"></a>Compte d’administrateur

Chaque registre de conteneurs inclut un compte d’utilisateur administrateur, qui est désactivé par défaut. Vous pouvez activer le compte d'utilisateur administrateur et gérer ses informations d'identification à partir du portail Azure, ou par le biais de l'interface Azure CLI ou d'autres outils Azure. Le compte administrateur dispose des autorisations complètes sur le registre.

Le compte administrateur est actuellement requis pour certains scénarios de déploiement d’une image à partir d’un registre de conteneurs vers certains services Azure. Par exemple, le compte d’administrateur est nécessaire lorsque vous déployez une image conteneur dans le portail à partir d’un registre directement vers [Azure Container Instances](../container-instances/container-instances-using-azure-container-registry.md#deploy-with-azure-portal) ou [Azure Web Apps pour conteneurs](container-registry-tutorial-deploy-app.md).

> [!IMPORTANT]
> Le compte d’administrateur est conçu pour permettre à un seul utilisateur d’accéder au registre, principalement à des fins de test. Nous ne recommandons pas de partager les informations d’identification du compte d’administrateur avec plusieurs utilisateurs. Tous les utilisateurs qui s’authentifient avec le compte d’administrateur apparaissent sous la forme d’un seul utilisateur avec un accès par envoi et par extraction au registre. La modification ou la désactivation de ce compte désactive l’accès au registre pour tous les utilisateurs qui utilisent ces informations d’identification. Une identité individuelle est recommandée pour les utilisateurs et principaux du service pour les scénarios sans périphérique de contrôle.
>

Le compte d’administrateur reçoit deux mots de passe qui peuvent être régénérés. Deux mots de passe vous permettent de maintenir la connexion au registre en utilisant un mot de passe tandis que vous régénérez l’autre. Si le compte d'administrateur est activé, vous pouvez transmettre le nom d'utilisateur et l'un ou l'autre des mots de passe à la commande `docker login` lorsque vous y êtes invité pour une authentification de base auprès du registre. Par exemple :

```
docker login myregistry.azurecr.io 
```

Pour connaître les meilleures pratiques de gestion des informations d'identification, consultez la référence de la commande [docker login](https://docs.docker.com/engine/reference/commandline/login/).

Pour activer l’utilisateur administrateur pour un registre existant, vous pouvez utiliser le paramètre `--admin-enabled` de la commande [az acr update](/cli/azure/acr#az-acr-update) dans Azure CLI :

```azurecli
az acr update -n <acrName> --admin-enabled true
```

Pour activer l’utilisateur administrateur dans le portail Azure, accédez au registre, sélectionnez **Clés d’accès** sous **PARAMÈTRES**, puis **Activer** sous **Utilisateur administrateur**.

![Activer l’IU de l’utilisateur administrateur dans le portail Azure][auth-portal-01]

## <a name="next-steps"></a>Étapes suivantes

* [Push your first image using the Azure CLI (Transmettre votre première image à l’aide d’Azure CLI)](container-registry-get-started-azure-cli.md)

<!-- IMAGES -->
[auth-portal-01]: ./media/container-registry-authentication/auth-portal-01.png

---
title: Configurer le service Gestion des API à l’aide de Git - Azure | Microsoft Docs
description: Découvrez comment enregistrer et configurer votre configuration du service Gestion des API à l’aide de Git
services: api-management
documentationcenter: ''
author: vladvino
manager: erikre
editor: mattfarm
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 03/12/2019
ms.author: apimpm
ms.openlocfilehash: 18cc42c3447de733447c27db52a9a6d664539464
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89400351"
---
# <a name="how-to-save-and-configure-your-api-management-service-configuration-using-git"></a>Comment enregistrer et configurer votre configuration du service Gestion des API à l’aide de Git

Chaque instance du service Gestion des API gère une base de données de configuration qui contient des informations sur la configuration et les métadonnées de cette instance de service. Vous pouvez modifier l’instance de service en ajustant un paramètre dans le portail Azure, en utilisant une applet de commande PowerShell ou en effectuant un appel API REST. Outre ces méthodes, vous pouvez gérer votre configuration d’instance de service à l’aide de Git, notamment dans le cadre des scénarios de gestion de service suivants :

* Contrôle de version de la configuration : téléchargez et stockez différentes versions de votre configuration de service
* Modifications en bloc de la configuration : apportez des modifications à plusieurs parties de votre configuration de service dans votre dépôt local et intégrez les modifications au serveur en une seule opération
* Chaîne d’outils et flux de travail Git familiers : utilisez les flux de travail et les outils Git qui vous sont déjà familiers

Le diagramme suivant montre une vue d’ensemble des différentes façons de configurer votre instance du service Gestion des API.

![Configurer avec Git][api-management-git-configure]

Quand vous apportez des changements à votre service en utilisant le portail Azure, les applets de commande PowerShell ou l’API REST, vous gérez votre base de données de configuration de service à l’aide du point de terminaison `https://{name}.management.azure-api.net`, comme indiqué sur le côté droit du diagramme. Le côté gauche du diagramme illustre comment vous pouvez gérer votre configuration de service à l’aide de Git et du dépôt Git pour votre service situé à l’adresse `https://{name}.scm.azure-api.net`.

Les étapes suivantes fournissent une vue d’ensemble de la gestion de votre instance du service Gestion des API à l’aide de Git.

1. Accéder à la configuration de Git dans votre service
2. Enregistrer votre base de données de configuration de service dans votre dépôt Git
3. Cloner le dépôt Git sur votre ordinateur local
4. Récupérer le dernier dépôt sur votre ordinateur local, puis valider les modifications et les transférer vers votre dépôt
5. Déployer les modifications depuis votre dépôt dans votre base de données de configuration de service

Cet article décrit comment activer et utiliser Git pour gérer votre configuration de service et fournit une référence pour les fichiers et dossiers dans le dépôt Git.

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="access-git-configuration-in-your-service"></a>Accéder à la configuration de Git dans votre service

Pour afficher et configurer vos paramètres de configuration Git, vous pouvez cliquer sur le menu **Déploiement et infrastructure** et accéder à l’onglet **Dépôt**.

![Activer GIT][api-management-enable-git]

> [!IMPORTANT]
> Les secrets qui ne sont pas définis comme valeurs nommées sont stockés dans le dépôt et restent dans son historique jusqu’à ce que vous désactiviez et réactiviez l’accès à Git. Les valeurs nommées offrent un emplacement sécurisé pour gérer les valeurs de chaîne constante, notamment les secrets, dans toutes les stratégies et configurations d’API ; vous n’êtes donc pas obligé de les stocker directement dans les déclarations de votre stratégie. Pour plus d’informations, consultez [Guide pratique pour utiliser des valeurs nommées dans les stratégies Gestion des API Azure](api-management-howto-properties.md).
>
>

Pour plus d’informations sur l’activation ou la désactivation de l’accès à Git en utilisant l’API REST, consultez [Activer ou désactiver l’accès à Git à l’aide de l’API REST](/rest/api/apimanagement/2019-12-01/tenantaccess?EnableGit).

## <a name="to-save-the-service-configuration-to-the-git-repository"></a>Pour enregistrer la configuration du service dans le dépôt Git

La première étape avant le clonage du dépôt consiste à enregistrer l’état actuel de la configuration du service dans le dépôt. Cliquez sur **Enregistrer dans le dépôt**.

Apportez les modifications souhaitées dans l’écran de confirmation, puis cliquez sur **Enregistrer** pour les enregistrer.

Après quelques instants, la configuration est enregistrée, et l’état de configuration du dépôt est affiché, y compris la date et l’heure de la dernière modification de la configuration et de la dernière synchronisation entre la configuration du service et le dépôt.

Une fois la configuration enregistrée dans le dépôt, elle peut être clonée.

Pour plus d’informations sur l’exécution de cette opération avec l’API REST, consultez [Valider l’instantané de configuration à l’aide de l’API REST](/rest/api/apimanagement/2019-12-01/tenantaccess?CommitSnapshot).

## <a name="to-clone-the-repository-to-your-local-machine"></a>Pour cloner le dépôt sur votre ordinateur local

Pour cloner un dépôt, vous avez besoin de l’URL de votre dépôt, d’un nom d’utilisateur et d’un mot de passe. Pour obtenir le nom d’utilisateur et d’autres informations d’identification, cliquez sur **Informations d’identification d’accès** dans la partie supérieure de la page.

Pour générer un mot de passe, vérifiez d’abord que le champ **Expiration** est défini sur la date et l’heure d’expiration souhaitées, puis cliquez sur **Générer**.

> [!IMPORTANT]
> Notez ce mot de passe. Une fois que vous quittez cette page, le mot de passe ne s’affiche plus.
>

Les exemples suivants utilisent l’outil Git Bash de [Git pour Windows](https://www.git-scm.com/downloads) , mais vous pouvez utiliser n’importe quel outil Git auquel vous êtes habitué.

Ouvrez votre outil Git dans le dossier de votre choix et exécutez la commande suivante pour cloner le référentiel Git sur votre ordinateur local, à l'aide de la commande fournie par le portail Azure.

```
git clone https://{name}.scm.azure-api.net/
```

Quand vous y êtes invité, fournissez le nom d’utilisateur et le mot de passe.

En cas d’erreur, essayez de modifier votre commande `git clone` pour y inclure le nom d’utilisateur et le mot de passe, comme illustré dans l’exemple suivant.

```
git clone https://username:password@{name}.scm.azure-api.net/
```

En cas d’erreur, essayez d’appliquer un encodage URL à la partie mot de passe de la commande. Pour effectuer cette opération rapidement, vous pouvez ouvrir Visual Studio et exécuter la commande ci-dessous dans la **Fenêtre Exécution**. Pour ouvrir la **Fenêtre Exécution**, ouvrez une solution ou un projet dans Visual Studio (ou créez une application console vide), puis choisissez **Fenêtres**, **Exécution** dans le menu **Déboguer**.

```
?System.Net.WebUtility.UrlEncode("password from the Azure portal")
```

Pour construire la commande git, utilisez le mot de passe codé, avec votre nom d’utilisateur et l’emplacement du dépôt.

```
git clone https://username:url encoded password@{name}.scm.azure-api.net/
```

Une fois le référentiel cloné, vous pouvez l’afficher et l’utiliser dans votre système de fichiers local. Pour plus d’informations, consultez [Référence de la structure des fichiers et des dossiers du dépôt Git local](#file-and-folder-structure-reference-of-local-git-repository).

## <a name="to-update-your-local-repository-with-the-most-current-service-instance-configuration"></a>Pour mettre à jour votre dépôt local avec la dernière configuration de l’instance du service

Si vous apportez des changements à votre instance de service Gestion des API dans le portail Azure ou à l’aide de l’API REST, vous devez enregistrer ces changements dans le dépôt pour pouvoir mettre à jour votre dépôt local avec les derniers changements. Pour ce faire, cliquez sur **Enregistrer dans le dépôt** sous l’onglet **Dépôt** dans le portail Azure, puis exécutez la commande suivante dans votre dépôt local.

```
git pull
```

Avant d’exécuter `git pull` , vérifiez que vous vous trouvez dans le dossier de votre dépôt local. Si vous venez d’exécuter la commande `git clone` , vous devez accéder au répertoire de votre dépôt en exécutant une commande semblable à la suivante.

```
cd {name}.scm.azure-api.net/
```

## <a name="to-push-changes-from-your-local-repo-to-the-server-repo"></a>Pour transférer les modifications depuis votre dépôt local vers le dépôt du serveur
Pour transférer les modifications depuis votre dépôt local vers le dépôt du serveur, vous devez valider vos modifications, puis les transférer vers le dépôt du serveur. Pour valider vos modifications, ouvrez votre outil de commande Git, accédez au répertoire de votre dépôt local et exécutez les commandes suivantes.

```
git add --all
git commit -m "Description of your changes"
```

Pour transférer toutes les validations vers le serveur, exécutez la commande suivante.

```
git push
```

## <a name="to-deploy-any-service-configuration-changes-to-the-api-management-service-instance"></a>Pour déployer les modifications de configuration de service sur l’instance du service Gestion des API

Une fois vos modifications locales validées et transférées vers le dépôt du serveur, vous pouvez les déployer sur votre instance du service Gestion des API.

Pour plus d’informations sur l’exécution de cette opération en utilisant l’API REST, consultez [Déployer les modifications Git dans votre base de données de configuration à l’aide de l’API REST](/rest/api/apimanagement/2019-12-01/tenantconfiguration).

## <a name="file-and-folder-structure-reference-of-local-git-repository"></a>Référence de la structure des fichiers et des dossiers du dépôt Git local

Les fichiers et dossiers figurant dans le référentiel Git local contiennent les informations de configuration relatives à l'instance de service.

| Élément | Description |
| --- | --- |
| Dossier api-management racine |Contient la configuration de niveau supérieur pour l’instance de service |
| Dossier apis |Contient la configuration des API dans l’instance de service |
| Dossier groups |Contient la configuration des groupes dans l’instance de service |
| Dossier policies |Contient les stratégies dans l’instance de service |
| Dossier portalStyles |Contient la configuration des personnalisations du portail des développeurs dans l’instance de service |
| Dossier products |Contient la configuration des produits dans l’instance de service |
| Dossier templates |Contient la configuration des modèles d’e-mail dans l’instance de service |

Chaque dossier peut contenir un ou plusieurs fichiers et, dans certains cas, un ou plusieurs dossiers, par exemple un dossier par API, produit ou groupe. Les fichiers dans chaque dossier sont spécifiques du type d’entité décrit par le nom du dossier.

| Type de fichier | Objectif |
| --- | --- |
| json |Informations de configuration sur l’entité concernée |
| html |Descriptions de l’entité, souvent affichées dans le portail des développeurs |
| Xml |Policy statements |
| css |Feuilles de style pour la personnalisation du portail des développeurs |

Ces fichiers peuvent être créés, supprimés, modifiés et gérés dans votre système de fichiers local, et les modifications peuvent être redéployées sur votre instance du service Gestion des API.

> [!NOTE]
> Les entités suivantes ne se trouvent pas dans le dépôt Git et ne peuvent pas être configurées à l’aide de Git.
>
> * [Utilisateurs](/rest/api/apimanagement/2019-12-01/user)
> * [Abonnements](/rest/api/apimanagement/2019-12-01/subscription)
> * Valeurs nommées
> * Entités du portail des développeur autres que les styles
>

### <a name="root-api-management-folder"></a>Dossier api-management racine
Le dossier `api-management` racine contient un fichier `configuration.json` qui renferme des informations de premier niveau sur l’instance de service dans le format suivant.

```json
{
  "settings": {
    "RegistrationEnabled": "True",
    "UserRegistrationTerms": null,
    "UserRegistrationTermsEnabled": "False",
    "UserRegistrationTermsConsentRequired": "False",
    "DelegationEnabled": "False",
    "DelegationUrl": "",
    "DelegatedSubscriptionEnabled": "False",
    "DelegationValidationKey": "",
    "RequireUserSigninEnabled": "false"
  },
  "$ref-policy": "api-management/policies/global.xml"
}
```

Les quatre premiers paramètres (`RegistrationEnabled`, `UserRegistrationTerms`, `UserRegistrationTermsEnabled` et `UserRegistrationTermsConsentRequired`) correspondent aux paramètres suivants, disponibles dans l’onglet **Identités** de la section **Portail des développeurs**.

| Paramètre d’identité | Correspond à |
| --- | --- |
| RegistrationEnabled |Présence du fournisseur d’identité **Nom d’utilisateur et mot de passe** |
| UserRegistrationTerms |**Conditions d’utilisation liées à l’inscription de l’utilisateur** |
| UserRegistrationTermsEnabled |**Afficher les conditions d’utilisation dans la page d’abonnement** |
| UserRegistrationTermsConsentRequired |**Exiger le consentement** |
| RequireUserSigninEnabled |**Rediriger les utilisateurs anonymes vers la page de connexion** |

Les quatre paramètres qui suivent (`DelegationEnabled`, `DelegationUrl`, `DelegatedSubscriptionEnabled` et `DelegationValidationKey`) correspondent aux paramètres suivants, disponibles dans l’onglet **Délégation** de la section **Portail des développeurs**.

| Paramètre de délégation | Correspond à |
| --- | --- |
| DelegationEnabled |Case à cocher **Déléguer la connexion et l’inscription** |
| DelegationUrl |**URL de point de terminaison de la délégation** |
| DelegatedSubscriptionEnabled |**Déléguer l’abonnement au produit** |
| DelegationValidationKey |**Déléguer la clé de validation** |

Le dernier paramètre, `$ref-policy`, correspond au fichier d’instructions de stratégie globale pour l’instance de service.

### <a name="apis-folder"></a>Dossier apis
Le dossier `apis` contient un dossier pour chaque API dans l’instance de service, qui renferme les éléments suivants.

* `apis\<api name>\configuration.json` : cet élément représente la configuration de l’API et contient des informations sur l’URL du service principal et sur les opérations. Vous pouvez obtenir les mêmes informations en appelant l’opération [Obtenir une API spécifique](/rest/api/apimanagement/2019-12-01/apis/get) avec `export=true` au format `application/json`.
* `apis\<api name>\api.description.html` : cet élément décrit l’API et correspond à la propriété `description` de [l’entité API](/java/api/com.microsoft.azure.storage.table.entityproperty).
* `apis\<api name>\operations\` : ce dossier contient des fichiers `<operation name>.description.html` correspondant aux opérations dans l’API. Chaque fichier contient la description d’une opération unique dans l’API, qui correspond à la propriété `description` de [l’entité Operation](/rest/api/visualstudio/operations/list#operationproperties) dans l’API REST.

### <a name="groups-folder"></a>Dossier groups
Le dossier `groups` contient un dossier pour chaque groupe défini dans l’instance de service.

* `groups\<group name>\configuration.json` : cet élément correspond à la configuration du groupe. Vous pouvez obtenir les mêmes informations en appelant l’opération [Obtenir un groupe spécifique](/rest/api/apimanagement/2019-12-01/group/get) .
* `groups\<group name>\description.html` : cet élément décrit le groupe et correspond à la propriété `description` de [l’entité groupe](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-group-entity).

### <a name="policies-folder"></a>Dossier policies
Le dossier `policies` contient les instructions de stratégie pour votre instance de service.

* `policies\global.xml` : contient les stratégies définies dans l’étendue globale de votre instance de service.
* `policies\apis\<api name>\` : si des stratégies sont définies à l’échelle de l’API, elles se trouvent dans ce dossier.
* `policies\apis\<api name>\<operation name>\` : si des stratégies sont définies à l’échelle des opérations, elles se trouvent dans ce dossier, qui contient des fichiers `<operation name>.xml` correspondant aux instructions de stratégie pour chaque opération.
* `policies\products\` : si des stratégies sont définies à l’échelle des produits, elles se trouvent dans ce dossier, qui contient des fichiers `<product name>.xml` correspondant aux instructions de stratégie pour chaque produit.

### <a name="portalstyles-folder"></a>Dossier portalStyles
Le dossier `portalStyles` contient la configuration et les feuilles de style pour les personnalisations du portail des développeurs pour l’instance de service.

* `portalStyles\configuration.json` : contient les noms des feuilles de style utilisées par le portail des développeurs.
* `portalStyles\<style name>.css` : chaque fichier `<style name>.css` contient des styles pour le portail des développeurs (`Preview.css` et `Production.css` par défaut).

### <a name="products-folder"></a>Dossier products
Le dossier `products` contient un dossier pour chaque produit défini dans l’instance de service.

* `products\<product name>\configuration.json` : cet élément correspond à la configuration du produit. Vous pouvez obtenir les mêmes informations en appelant l’opération [Obtenir un produit spécifique](/rest/api/apimanagement/2019-12-01/product/get) .
* `products\<product name>\product.description.html` : cet élément décrit le produit et correspond à la propriété `description` de [l’entité produit](/rest/api/apimanagement/apimanagementrest/azure-api-management-rest-api-product-entity) dans l’API REST.

### <a name="templates"></a>modèles
Le dossier `templates` contient la configuration des [modèles d’e-mail](api-management-howto-configure-notifications.md) dans l’instance de service.

* `<template name>\configuration.json` : cet élément correspond à la configuration du modèle d’e-mail.
* `<template name>\body.html` : il s’agit du corps du modèle d’e-mail.

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur d’autres méthodes pour gérer votre instance de service, consultez les sources suivantes :

* Gérer votre instance de service à l’aide des applets de commande PowerShell suivante :
  * [Référence sur les applets de commande PowerShell de déploiement des services](/powershell/module/wds)
  * [Référence sur les applets de commande PowerShell de gestion des services](/powershell/azure/servicemanagement/overview)
* Gérer votre instance de service à l’aide de l’API REST
  * [Référence de l’API REST Gestion des API](/rest/api/apimanagement/)


[api-management-enable-git]: ./media/api-management-configuration-repository-git/api-management-enable-git.png
[api-management-git-enabled]: ./media/api-management-configuration-repository-git/api-management-git-enabled.png
[api-management-save-configuration]: ./media/api-management-configuration-repository-git/api-management-save-configuration.png
[api-management-save-configuration-confirm]: ./media/api-management-configuration-repository-git/api-management-save-configuration-confirm.png
[api-management-configuration-status]: ./media/api-management-configuration-repository-git/api-management-configuration-status.png
[api-management-configuration-git-clone]: ./media/api-management-configuration-repository-git/api-management-configuration-git-clone.png
[api-management-generate-password]: ./media/api-management-configuration-repository-git/api-management-generate-password.png
[api-management-password]: ./media/api-management-configuration-repository-git/api-management-password.png
[api-management-git-configure]: ./media/api-management-configuration-repository-git/api-management-git-configure.png
[api-management-configuration-deploy]: ./media/api-management-configuration-repository-git/api-management-configuration-deploy.png
[api-management-identity-settings]: ./media/api-management-configuration-repository-git/api-management-identity-settings.png
[api-management-delegation-settings]: ./media/api-management-configuration-repository-git/api-management-delegation-settings.png
[api-management-git-icon-enable]: ./media/api-management-configuration-repository-git/api-management-git-icon-enable.png

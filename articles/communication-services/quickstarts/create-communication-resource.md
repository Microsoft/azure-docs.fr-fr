---
title: 'Démarrage rapide : Créer et gérer des ressources dans Azure Communication Services'
titleSuffix: An Azure Communication Services quickstart
description: Dans ce guide de démarrage rapide, vous allez découvrir comment créer et gérer votre première ressource Azure Communication Services.
author: mikben
manager: jken
services: azure-communication-services
ms.author: mikben
ms.date: 09/30/2020
ms.topic: overview
ms.service: azure-communication-services
zone_pivot_groups: acs-plat-azp-net
ms.openlocfilehash: a93ac3b5d988be33c0f27726a75b1006f990d1da
ms.sourcegitcommit: 230d5656b525a2c6a6717525b68a10135c568d67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94886109"
---
# <a name="quickstart-create-and-manage-communication-services-resources"></a>Démarrage rapide : Créer et gérer des ressources Communication Services

[!INCLUDE [Public Preview Notice](../includes/public-preview-include.md)]

Commencez avec Azure Communication Services en provisionnant votre première ressource Communication Services. Les ressources Communication Services peuvent être provisionnées via le portail Azure ou avec la bibliothèque de client .NET Management. La bibliothèque de client Management vous permet de créer, configurer, mettre à jour et supprimer votre ressource et vos interfaces à l’aide d’[Azure Resource Manager](../../azure-resource-manager/management/overview.md), le service de gestion et de déploiement d’Azure. Toutes les fonctionnalités disponibles dans les bibliothèques de client sont accessibles à partir du portail Azure. 

> [!WARNING]
> Notez que Communication Services est disponible uniquement aux États-Unis pendant la préversion publique. Notez également que les ressources Communication ne peuvent pas être transférées vers un autre abonnement durant cette période de préversion publique.

::: zone pivot="platform-azp"
[!INCLUDE [Azure portal](./includes/create-resource-azp.md)]
::: zone-end

::: zone pivot="platform-net"
[!INCLUDE [.NET](./includes/create-resource-net.md)]
::: zone-end

## <a name="access-your-connection-strings-and-service-endpoints"></a>Accéder à vos chaînes de connexion et points de terminaison de service

Les chaînes de connexion sont utilisées par les bibliothèques de client Communication Services pour se connecter et s’authentifier auprès d’Azure. Vous pouvez accéder à vos chaînes de connexion et points de terminaison de service Communication Services à partir du portail Azure, ou programmatiquement à l’aide des API Azure Resource Manager. 

Une fois que vous avez accédé à votre ressource Communication Services, sélectionnez **Clés** dans le menu de navigation et copiez les valeurs **Chaîne de connexion** ou **Point de terminaison** à utiliser par les bibliothèques de client Communication Services. Notez que vous avez accès aux clés primaires et secondaires. Cela peut être utile dans les scénarios où vous souhaitez accorder à un environnement tiers ou de préproduction un accès temporaire à vos ressources Communication Services.

:::image type="content" source="./media/key.png" alt-text="Capture d’écran de la page Clés dans Communication Services.":::

## <a name="store-your-connection-string"></a>Stocker votre chaîne de connexion

Les bibliothèques de client Communication Services utilisent des chaînes de connexion pour autoriser les demandes adressées à Communication Services. Plusieurs options vous permettant de stocker votre chaîne de connexion s’offrent à vous :

* Une application s’exécutant sur le bureau ou sur un appareil peut stocker la chaîne de connexion dans un fichier **app.config** ou **web.config**. Ajoutez la chaîne de connexion dans la section **AppSettings** de ces fichiers.
* Une application qui s’exécute dans Azure App Service peut stocker la chaîne de connexion dans les [paramètres d’application d’App Service](../../app-service/configure-common.md). Ajoutez la chaîne de connexion dans la section **Chaînes de connexion** sous l’onglet Paramètres d’application dans le portail.
* Vous pouvez stocker votre chaîne de connexion dans [Azure Key Vault](../../data-factory/store-credentials-in-key-vault.md).
* Si vous exécutez votre application localement, vous voudrez peut-être stocker votre chaîne de connexion dans une variable d’environnement.

### <a name="store-your-connection-string-in-an-environment-variable"></a>Stocker votre chaîne de connexion dans une variable d’environnement

Pour configurer une variable d’environnement, ouvrez une fenêtre de console, puis sélectionnez votre système d’exploitation dans les onglets ci-dessous. Remplacez `<yourconnectionstring>` par votre chaîne de connexion.

#### <a name="windows"></a>[Windows](#tab/windows)

Ouvrez une fenêtre de console et entrez la commande suivante :

```console
setx COMMUNICATION_SERVICES_CONNECTION_STRING "<yourconnectionstring>"
```

Après avoir ajouté la variable d’environnement, vous devrez peut-être redémarrer tous les programmes en cours d’exécution qui devront la lire, y compris la fenêtre de console. Par exemple, si vous utilisez Visual Studio comme éditeur, redémarrez Visual Studio avant d’exécuter l’exemple.

#### <a name="macos"></a>[macOS](#tab/unix)

Modifiez le fichier **.zshrc** pour y ajouter la variable d’environnement :

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

Après avoir ajouté la variable d’environnement, exécutez `source ~/.zshrc` depuis la fenêtre de console pour appliquer les changements. Si vous avez créé la variable d’environnement avec votre IDE ouvert, vous devrez peut-être fermer et rouvrir l’éditeur, l’IDE ou l’interpréteur de commandes pour accéder à la variable.

#### <a name="linux"></a>[Linux](#tab/linux)

Modifiez le fichier **.bash_profile** afin d’y ajouter la variable d’environnement :

```bash
export COMMUNICATION_SERVICES_CONNECTION_STRING="<yourconnectionstring>"
```

Après avoir ajouté la variable d’environnement, exécutez `source ~/.bash_profile` depuis la fenêtre de console pour appliquer les changements. Si vous avez créé la variable d’environnement avec votre IDE ouvert, vous devrez peut-être fermer et rouvrir l’éditeur, l’IDE ou l’interpréteur de commandes pour accéder à la variable.

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez nettoyer et supprimer un abonnement Communication Services, vous pouvez supprimer la ressource ou le groupe de ressources. La suppression du groupe de ressources efface également les autres ressources qui y sont associées.

Si des numéros de téléphone sont encore associés à votre ressource au moment de la suppression de la ressource, ils sont automatiquement libérés de votre ressource en même temps. 

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez découvert comment :

> [!div class="checklist"]
> * Créer une ressource Communication Services
> * Configurer la zone géographique et des étiquettes pour la ressource
> * Accéder aux clés de cette ressource
> * Supprimer la ressource

> [!div class="nextstepaction"]
> [Créer vos premiers jetons d’accès utilisateur](access-tokens.md)
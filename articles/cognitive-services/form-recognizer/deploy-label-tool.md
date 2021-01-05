---
title: Guide pratique pour déployer l’outil d’étiquetage des exemples Form Recognizer
titleSuffix: Azure Cognitive Services
description: Découvrez les différentes façons de déployer l’outil d’étiquetage des exemples Form Recognizer pour faciliter l’apprentissage supervisé.
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.subservice: forms-recognizer
ms.topic: how-to
ms.date: 04/14/2020
ms.author: pafarley
ms.openlocfilehash: 985f9e741a1491a812c1d2b20de96381f8af3fa4
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359844"
---
# <a name="deploy-the-sample-labeling-tool"></a>Déployer l’outil d’étiquetage des exemples

L’outil d’étiquetage des exemples Form Recognizer est une application qui fournit une interface utilisateur simple, que vous pouvez utiliser pour étiqueter manuellement des formulaires (documents) à des fins d’apprentissage supervisé. Dans cet article, nous allons vous fournir des liens et des instructions qui vous apprendront à :

* [Exécuter l’outil d’étiquetage des exemples localement](#run-the-sample-labeling-tool-locally)
* [Déployer l’outil d’étiquetage des exemples dans une Azure Container Instances (ACI)](#deploy-with-azure-container-instances-aci)
* [Utiliser et contribuer à l’outil open source OCR Form Labeling Tool](#open-source-on-github)

## <a name="run-the-sample-labeling-tool-locally"></a>Exécuter l’outil d’étiquetage des exemples localement

Le moyen le plus rapide de commencer à étiqueter des données consiste à exécuter l’outil d’étiquetage des exemples localement. Le guide de démarrage rapide suivant utilise l’API REST Form Recognizer et l’outil d’étiquetage des exemples pour entraîner un modèle personnalisé avec des données étiquetées manuellement. 

* [Démarrage rapide : Étiqueter des formulaires, entraîner un modèle et analyser un formulaire à l’aide de l’outil d’étiquetage des exemples](./quickstarts/label-tool.md).

## <a name="deploy-with-azure-container-instances-aci"></a>Déployer avec Azure Container Instances (ACI)

Avant de commencer, il est important de noter qu’il existe deux façons de déployer l’outil d’étiquetage des exemples sur Azure Container Instance (ACI). Les deux options sont utilisées pour exécuter l’outil d’étiquetage des exemples avec ACI : 

* [À l’aide du portail Azure](#azure-portal)
* [Utilisation de l’interface de ligne de commande Azure (CLI)](#azure-cli)

### <a name="azure-portal"></a>Portail Azure

Suivez ces étapes pour créer une nouvelle ressource à partir du portail Azure : 

1. Connectez-vous au [portail Azure](https://portal.azure.com/signin/index/).
2. Sélectionnez **Créer une ressource**. 
3. Ensuite, sélectionnez **Application web**. 

   > [!div class="mx-imgBorder"]
   > ![Sélectionner l’application web](./media/quickstarts/formre-create-web-app.png)
   
4. Tout d’abord, assurez-vous que l’onglet **De base** est sélectionné. À présent, vous allez avoir besoin de fournir des informations : 

   > [!div class="mx-imgBorder"]
   > ![Sélectionner De base](./media/quickstarts/formre-select-basics.png)
   * Abonnement : sélectionnez un abonnement Azure existant
   * Groupe de ressources : vous pouvez réutiliser un groupe de ressources existant ou en créer un pour ce projet. Nous vous recommandons de créer un groupe de ressources.
   * Nom : donnez un nom à votre application web. 
   * Publier : sélectionnez **Conteneur Docker**
   * Système d’exploitation : sélectionnez **Linux**
   * Région : choisissez une région pertinente pour vous.
   * Plan Linux : sélectionnez un niveau tarifaire/plan pour votre service d’application. 

   > [!div class="mx-imgBorder"]
   > ![Configurer votre application web](./media/quickstarts/formre-select-docker-linux.png)

5. Ensuite, sélectionnez l’onglet **Docker**. 

   > [!div class="mx-imgBorder"]
   > ![Sélectionner Docker](./media/quickstarts/formre-select-docker.png)

6. Nous allons maintenant configurer votre conteneur Docker. Sauf indication contraire, tous les champs sont obligatoires :

    # <a name="v20"></a>[v2.0](#tab/v2-0)  
   * Options : sélectionnez **Conteneur unique**
   * Source d’image : sélectionnez **Registre privé** 
   * URL du serveur : définissez cette valeur sur `https://mcr.microsoft.com`
   * Nom d’utilisateur (facultatif) : créez un nom d’utilisateur. 
   * Mot de passe (facultatif) : créez un mot de passe sécurisé dont vous vous souviendrez.
   * Image et étiquette : définissez cette valeur sur `mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest`
   * Déploiement continu : définissez cette valeur sur **Activé** si vous souhaitez recevoir des mises à jour automatiques lorsque l’équipe de développement apporte des modifications à l’outil d’étiquetage des exemples.
   * Commande de démarrage : définissez cette option sur `./run.sh eula=accept`

    # <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1) 
   * Options : sélectionnez **Conteneur unique**
   * Source d’image : sélectionnez **Registre privé** 
   * URL du serveur : définissez cette valeur sur `https://mcr.microsoft.com`
   * Nom d’utilisateur (facultatif) : créez un nom d’utilisateur. 
   * Mot de passe (facultatif) : créez un mot de passe sécurisé dont vous vous souviendrez.
   * Image et étiquette : définissez cette valeur sur `mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-preview`
   * Déploiement continu : définissez cette valeur sur **Activé** si vous souhaitez recevoir des mises à jour automatiques lorsque l’équipe de développement apporte des modifications à l’outil d’étiquetage des exemples.
   * Commande de démarrage : définissez cette option sur `./run.sh eula=accept`
    
    ---

   > [!div class="mx-imgBorder"]
   > ![Configurer Docker](./media/quickstarts/formre-configure-docker.png)

7. Vous avez terminé. Ensuite, sélectionnez **Vérifier + créer**, puis **Créer** pour déployer votre application web. Lorsque vous avez terminé, vous pouvez accéder à votre application web à l’URL fournie dans la **Vue d’ensemble** de votre ressource.

> [!NOTE]
> Lorsque vous créez votre application web, vous pouvez également configurer l’autorisation/l’authentification. Cela n’est pas nécessaire au départ. 

> [!IMPORTANT]
> Vous devrez peut-être activer TLS pour votre application web afin de l’afficher à son adresse `https`. Suivez les instructions figurant dans [Activer un point de terminaison TLS](https://docs.microsoft.com/azure/container-instances/container-instances-container-group-ssl) pour configurer un conteneur sidecar qui active TLS/SSL pour votre application web.

### <a name="azure-cli"></a>Azure CLI

En guise d’alternative à l’utilisation du portail Azure, vous pouvez créer une ressource à l’aide d’Azure CLI. Avant de continuer, vous devez installer [Azure CLI](/cli/azure/install-azure-cli). Vous pouvez ignorer cette étape si vous utilisez déjà Azure CLI. 

Voici quelques éléments que vous devez savoir à propos de cette commande :

* `DNS_NAME_LABEL=aci-demo-$RANDOM` génère un nom DNS aléatoire. 
* Cet exemple suppose que vous disposez d’un groupe de ressources que vous pouvez utiliser pour créer une ressource. Remplacez `<resource_group_name>` par un groupe de ressources valide associé à votre abonnement. 
* Vous devez spécifier l’emplacement auquel vous souhaitez créer la ressource. Remplacez `<region name>` par la région de votre choix pour l’application web. 
* Cette commande accepte automatiquement le CLUF.

À partir d’Azure CLI, exécutez cette commande pour créer une ressource d’application web pour l’outil d’étiquetage des exemples : 


# <a name="v20"></a>[v2.0](#tab/v2-0)   
```azurecli
DNS_NAME_LABEL=aci-demo-$RANDOM

az container create \
  --resource-group <resource_group_name> \
  --name <name> \
  --image mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool \
  --ports 3000 \
  --dns-name-label $DNS_NAME_LABEL \
  --location <region name> \
  --cpu 2 \
  --memory 8 \
  --command-line "./run.sh eula=accept"
``` 
# <a name="v21-preview"></a>[v2.1 (préversion)](#tab/v2-1)    
```azurecli
DNS_NAME_LABEL=aci-demo-$RANDOM

az container create \
  --resource-group <resource_group_name> \
  --name <name> \
  --image mcr.microsoft.com/azure-cognitive-services/custom-form/labeltool:latest-preview \
  --ports 3000 \
  --dns-name-label $DNS_NAME_LABEL \
  --location <region name> \
  --cpu 2 \
  --memory 8 \
  --command-line "./run.sh eula=accept"
```

---

### <a name="connect-to-azure-ad-for-authorization"></a>Se connecter à Azure AD pour l’autorisation

Nous vous recommandons de connecter votre application web à Azure Active Directory. Cela permet de s’assurer que seuls les utilisateurs disposant d’informations d’identification valides peuvent se connecter à et utiliser votre application web. Suivez les instructions qui figurent dans [Configurer votre application App Service ](../../app-service/configure-authentication-provider-aad.md) pour vous connecter à Azure Active Directory.

## <a name="open-source-on-github"></a>Open source sur GitHub

OCR Form Labeling Tool est également disponible en tant que projet open source sur GitHub. Cet outil est une application web créée à l’aide de React + Redux, et il est écrit en TypeScript. Pour en savoir plus ou pour contribuer, consultez [OCR Form Labeling Tool](https://github.com/microsoft/OCR-Form-Tools/blob/master/README.md).

## <a name="next-steps"></a>Étapes suivantes

Utilisez le guide de démarrage rapide [Effectuer l’entraînement avec des étiquettes](./quickstarts/label-tool.md) afin d’apprendre à utiliser l’outil pour étiqueter manuellement les données d’entraînement et effectuer un apprentissage supervisé.

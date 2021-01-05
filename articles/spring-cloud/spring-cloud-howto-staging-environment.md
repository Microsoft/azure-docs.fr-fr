---
title: Configurer un environnement de préproduction dans Azure Spring Cloud | Microsoft Docs
description: Découvrir comment utiliser le déploiement bleu-vert avec Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 02/03/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: 72cf5553bec5985ba0310b4a347b0d2c60da6924
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92090707"
---
# <a name="set-up-a-staging-environment-in-azure-spring-cloud"></a>Configurer un environnement intermédiaire dans Azure Spring Cloud

**Cet article s’applique à :** ✔️ Java

Cet article explique comment configurer un déploiement intermédiaire à l’aide du modèle de déploiement bleu-vert dans Azure Spring Cloud. Le déploiement bleu/vert est un modèle de livraison continue Azure DevOps qui consiste à garder en service une version existante (bleue) pendant le déploiement d’une nouvelle version (verte). Cet article vous montre également comment faire passer ce déploiement intermédiaire en production sans modifier directement le déploiement de production.

## <a name="prerequisites"></a>Prérequis

Cet article part du principe que vous avez déjà déployé l’application PiggyMetrics à partir de notre [tutoriel sur le lancement d’une application Azure Spring Cloud](./spring-cloud-quickstart.md). PiggyMetrics comprend trois applications : « Gateway », « account-service » et « auth-service ».  

Si vous voulez utiliser une autre application pour cet exemple, vous devez effectuer un changement simple dans une partie publique de l’application.  Ce changement permet de différencier votre déploiement de préproduction de la production.

>[!TIP]
> Azure Cloud Shell est un interpréteur de commandes interactif gratuit, qui vous permet d’exécuter les instructions de cet article.  Les outils Azure les plus courants sont préinstallés, notamment les dernières versions de Git, JDK, Maven et Azure CLI. Si vous êtes connecté à votre abonnement Azure, lancez votre [Azure Cloud Shell](https://shell.azure.com).  Pour en savoir plus, consultez [Vue d’ensemble d’Azure Cloud Shell](../cloud-shell/overview.md).

Pour configurer un environnement intermédiaire dans Azure Spring Cloud, suivez les instructions des sections suivantes.

## <a name="install-the-azure-cli-extension"></a>Installer l’extension Azure CLI

Installez l’extension Azure Spring Cloud pour Azure CLI à l’aide de la commande suivante :

```azurecli
az extension add --name spring-cloud
```
    
## <a name="view-all-deployments"></a>Afficher tous les déploiements

Accédez à votre instance de service dans le portail Azure et sélectionnez **Gestion du déploiement** pour afficher tous les déploiements. Vous pouvez sélectionner chaque déploiement pour obtenir des détails supplémentaires.

## <a name="create-a-staging-deployment"></a>Créer un déploiement intermédiaire

1. Dans votre environnement de développement local, apportez une petite modification à l’application passerelle PiggyMetrics. Par exemple, modifiez la couleur dans le fichier *gateway/src/main/resources/static/css/launch.css*. Vous pourrez ainsi différencier facilement les deux déploiements. Exécutez la commande suivante pour générer le package jar : 

    ```console
    mvn clean package
    ```

1. Créez un déploiement avec Azure CLI et donnez-lui le nom de déploiement intermédiaire « green ».

    ```azurecli
    az spring-cloud app deployment create -g <resource-group-name> -s <service-instance-name> --app gateway -n green --jar-path gateway/target/gateway.jar
    ```

1. Une fois le déploiement terminé, accédez à la page de la passerelle à partir du **Tableau de bord de l’application** et examinez toutes vos instances sous l’onglet **Instances d’application** à gauche.
  
> [!NOTE]
> L’état de découverte est *OUT_OF_SERVICE* pour que le trafic ne soit pas acheminé vers ce déploiement avant la fin de la vérification.

## <a name="verify-the-staging-deployment"></a>Vérifier le déploiement de préproduction

1. Revenez à la page **Gestion du déploiement** et sélectionnez votre nouveau déploiement. L’état du déploiement doit indiquer *En cours d’exécution*. Le bouton **Attribuer/supprimer un domaine** doit apparaître en grisé, car l’environnement est un environnement intermédiaire.

1. Dans le volet **Vue d’ensemble**, vous devez voir un **Point de terminaison de test**. Copiez-le et collez-le dans une nouvelle fenêtre de navigateur, et la nouvelle page PiggyMetrics doit s’afficher.

>[!TIP]
> * Vérifiez que votre point de terminaison de test se termine par une barre oblique (/) pour garantir le bon chargement du fichier CSS.  
> * Si votre navigateur vous demande d’entrer des informations d’identification de connexion pour afficher la page, utilisez [URL Decode](https://www.urldecoder.org/) pour décoder votre point de terminaison de test. URL Decode retourne une URL au format « https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green ».  Utilisez-le pour accéder à votre point de terminaison.

>[!NOTE]    
> Les paramètres du serveur de configuration s’appliquent à votre environnement intermédiaire et à la production. Par exemple, si vous définissez *somepath* comme chemin du contexte (`server.servlet.context-path`) de votre passerelle d’application dans le serveur de configuration, le chemin de votre déploiement vert devient : « https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/gateway/green/somepath/... ».
 
 Si vous visitez votre passerelle d’application publique à ce stade, vous devriez voir l’ancienne page sans votre nouveau changement.
    
## <a name="set-the-green-deployment-as-the-production-environment"></a>Définir le déploiement vert comme environnement de production

1. Après avoir vérifié votre changement dans votre environnement intermédiaire, vous pouvez l’envoyer (push) en production. Revenez à **Gestion du déploiement** et cochez la case d’application **gateway**.

2. Sélectionnez **Définir le déploiement**.
3. Dans la liste **Déploiement de production**, sélectionnez **Vert**, puis **Appliquer**.
4. Accédez à la page **Vue d’ensemble** de votre application de passerelle. Si vous avez déjà attribué un domaine à votre application de passerelle, l’URL apparaît dans le volet **Vue d’ensemble**. Pour afficher la page PiggyMetrics modifiée, sélectionnez l’URL, puis accédez au site.

>[!NOTE]
> Une fois le déploiement vert défini comme environnement de production, le déploiement précédent devient le déploiement intermédiaire.

## <a name="modify-the-staging-deployment"></a>Modifier le déploiement de préproduction

Si vous n’êtes pas satisfait de votre changement, vous pouvez modifier le code de votre application, générer un nouveau package jar et le charger dans votre déploiement Vert à l’aide d’Azure CLI.

```azurecli
az spring-cloud app deploy  -g <resource-group-name> -s <service-instance-name> -n gateway -d green --jar-path gateway.jar
```

## <a name="delete-the-staging-deployment"></a>Supprimer le déploiement intermédiaire

Pour supprimer votre déploiement intermédiaire du port Azure, accédez à la page de votre déploiement intermédiaire et sélectionnez le bouton **Supprimer**.

Vous pouvez également supprimer votre déploiement intermédiaire à partir d’Azure CLI en exécutant la commande suivante :

```azurecli
az spring-cloud app deployment delete -n <staging-deployment-name> -g <resource-group-name> -s <service-instance-name> --app gateway
```

## <a name="next-steps"></a>Étapes suivantes

* [Démarrage rapide : Déployer votre première application Azure Spring Cloud](spring-cloud-quickstart.md)
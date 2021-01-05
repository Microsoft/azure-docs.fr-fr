---
title: Gérer les applications Azure Service Fabric à l’aide de sfctl
description: Découvrez comment déployer et supprimer des applications à partir d’un cluster Azure Service Fabric à l’aide d’Azure Service Fabric CLI
author: Christina-Kang
ms.topic: conceptual
ms.date: 07/31/2018
ms.author: bikang
ms.openlocfilehash: 7d361d44c349bc7a6e3c041f78d00ad66182fa15
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84711032"
---
# <a name="manage-an-azure-service-fabric-application-by-using-azure-service-fabric-cli-sfctl"></a>Gérer une application Azure Service Fabric à l’aide d’Azure Service Fabric CLI (sfctl)

Découvrez comment créer et supprimer des applications qui s’exécutent dans un cluster Azure Service Fabric.

## <a name="prerequisites"></a>Conditions préalables requises

* Installez Service Fabric CLI. Sélectionnez ensuite votre cluster Service Fabric. Pour plus d’informations, consultez [Prise en main de Service Fabric CLI](service-fabric-cli.md).

* Vous devez également avoir un package d’application Service Fabric prêt à être déployé. Pour plus d’informations sur la création et l’empaquetage d’une application, consultez la documentation dédiée au [modèle d’application Service Fabric](service-fabric-application-model.md).

## <a name="overview"></a>Vue d’ensemble

Pour déployer une nouvelle application, suivez les étapes ci-dessous :

1. Téléchargez un package d’application dans le magasin d’images Service Fabric.
2. Approvisionnez un type d’application.
3. Supprimez le contenu du magasin d’images.
4. Spécifiez et créez une application.
5. Spécifiez et créez des services.

Pour supprimer une application existante, procédez comme suit :

1. Supprimez l’application.
2. Annulez l’approvisionnement du type d’application associé.

## <a name="deploy-a-new-application"></a>Déployer une nouvelle application

Pour déployer une nouvelle application, suivez les étapes ci-dessous :

### <a name="upload-a-new-application-package-to-the-image-store"></a>Charger un nouveau package d’application dans le magasin d’images

Avant de créer une application, vous devez charger le package d’application dans le magasin d’images Service Fabric.

Par exemple, si votre package d’application se trouve dans le répertoire `app_package_dir`, utilisez les commandes suivantes pour charger le répertoire :

```shell
sfctl application upload --path ~/app_package_dir
```

Pour les packages d’application volumineux, vous pouvez spécifier l’option `--show-progress` pour afficher la progression du chargement.

### <a name="provision-the-application-type"></a>Approvisionner le type d’application

Lorsque le chargement est terminé, approvisionnez l’application. Pour approvisionner l’application, utilisez la commande suivante :

```shell
sfctl application provision --application-type-build-path app_package_dir
```

La valeur de `application-type-build-path` est le nom du répertoire où vous avez chargé le package d’application.

### <a name="delete-the-application-package"></a>Supprimer le package d’application

Nous vous recommandons de supprimer le package d’application une fois que l’application est inscrite.  La suppression de packages d’application du magasin d’images libère des ressources système.  La conservation des packages d’application inutilisés consomme du stockage sur disque et affecte le niveau de performance des applications. 

Pour supprimer le package d’application du magasin d’images, utilisez la commande suivante :

```shell
sfctl store delete --content-path app_package_dir
```

`content-path` doit être le nom du répertoire que vous avez chargé lors de la création de l’application.

### <a name="create-an-application-from-an-application-type"></a>Créer une application à partir d’un type d’application

Une fois que vous avez approvisionné l’application, utilisez la commande suivante pour nommer et créer votre application :

```shell
sfctl application create --app-name fabric:/TestApp --app-type TestAppType --app-version 1.0
```

`app-name` est le nom que vous souhaitez utiliser pour l’instance de l’application. Vous pouvez obtenir des paramètres supplémentaires à partir du manifeste de l’application déjà approvisionné.

Le nom de l’application doit commencer par le préfixe `fabric:/`.

### <a name="create-services-for-the-new-application"></a>Créer des services pour la nouvelle application

Après avoir créé une application, créez des services à partir de l’application. Dans l’exemple suivant, nous créons un service sans état à partir de notre application. Les services que vous pouvez créer à partir d’une application sont définis dans un manifeste de service placé dans le package d’application déjà approvisionné.

```shell
sfctl service create --app-id TestApp --name fabric:/TestApp/TestSvc --service-type TestServiceType \
--stateless --instance-count 1 --singleton-scheme
```

## <a name="verify-application-deployment-and-health"></a>Vérifier le déploiement et l’intégrité de l’application

Pour vérifier que tout est correct, utilisez les commandes de contrôle d’intégrité suivantes :

```shell
sfctl application list
sfctl service list --application-id TestApp
```

Pour vérifier l’intégrité du service, utilisez des commandes similaires afin de récupérer des informations sur l’intégrité du service et de l’application :

```shell
sfctl application health --application-id TestApp
sfctl service health --service-id TestApp/TestSvc
```

Les applications et services intègres ont une valeur `HealthState` égale à `Ok`.

## <a name="remove-an-existing-application"></a>Supprimer une application existante

Pour supprimer une application, suivez les étapes ci-dessous :

### <a name="delete-the-application"></a>Supprimer l’application

Pour supprimer l’application, utilisez la commande suivante :

```shell
sfctl application delete --application-id TestEdApp
```

### <a name="unprovision-the-application-type"></a>Annuler l’approvisionnement de l’application

Après avoir supprimé l’application, vous pouvez annuler l’approvisionnement du type d’application s’il n’est plus nécessaire. Pour annuler l’approvisionnement du type d’application, utilisez la commande suivante :

```shell
sfctl application unprovision --application-type-name TestAppType --application-type-version 1.0
```

Le nom et la version du type doivent correspondre au nom et à la version du manifeste de l’application précédemment approvisionné.

## <a name="upgrade-application"></a>Mettre à niveau l’application

Après avoir créé votre application, vous pouvez répéter la même procédure pour configurer une deuxième version de votre application. Avec une mise à niveau d’application Service Fabric, vous pouvez ensuite exécuter la deuxième version de l’application. Pour plus d’informations, consultez la documentation [Mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md).

Pour effectuer une mise à niveau, configurez d’abord la version suivante de l’application à l’aide des mêmes commandes que précédemment :

```shell
sfctl application upload --path ~/app_package_dir_2
sfctl application provision --application-type-build-path app_package_dir_2
sfctl store delete --content-path app_package_dir_2
```

Il est ensuite recommandé d’effectuer une mise à niveau automatique surveillée et de lancer cette dernière en exécutant la commande suivante :

```shell
sfctl application upgrade --app-id TestApp --app-version 2.0.0 --parameters "{\"test\":\"value\"}" --mode Monitored
```

Les mises à niveau remplacent les paramètres existants par ce qui est spécifié. Les paramètres de l’application doivent être transmis en tant qu’arguments à la commande de mise à niveau, si nécessaire. Les paramètres de l’application doivent être codés en tant qu’objet JSON.

Pour récupérer les paramètres spécifiés antérieurement, vous pouvez utiliser la commande `sfctl application info`.

Lorsqu’une mise à niveau de l’application est en cours, l’état peut être récupéré à l’aide de la commande `sfctl application upgrade-status`.

Enfin, si une mise à niveau est en cours et doit être annulée, vous pouvez utiliser la commande `sfctl application upgrade-rollback` pour restaurer la mise à niveau.

## <a name="next-steps"></a>Étapes suivantes

* [Concepts de base Service Fabric CLI](service-fabric-cli.md)
* [Prise en main de Service Fabric sur Linux](service-fabric-get-started-linux.md)
* [Lancement d’une mise à niveau des applications Service Fabric](service-fabric-application-upgrade.md)

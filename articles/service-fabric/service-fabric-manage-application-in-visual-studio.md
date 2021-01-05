---
title: Gérer des applications dans Visual Studio
description: Utilisez Visual Studio pour créer, développer, packager, déployer et déboguer vos applications et services Azure Service Fabric.
ms.topic: conceptual
ms.date: 03/26/2018
ms.openlocfilehash: 1fd9295a0dead180767febcc5339ef0a25cb1e48
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574545"
---
# <a name="use-visual-studio-to-simplify-writing-and-managing-your-service-fabric-applications"></a>Utilisation de Visual Studio pour simplifier l'écriture et la gestion des applications Service Fabric
Vous pouvez gérer vos applications et services Azure Service Fabric via Visual Studio. Après avoir [configuré votre environnement de développement](service-fabric-get-started.md), vous pouvez utiliser Visual Studio pour créer des applications Service Fabric, ajouter des services ou empaqueter, enregistrer et déployer des applications dans votre cluster de développement local.

## <a name="deploy-your-service-fabric-application"></a>Déploiement de votre application Service Fabric
Par défaut, le déploiement d’une application regroupe les étapes suivantes en une simple opération :

1. Création du package d'application
2. Téléchargement du package d'application dans le magasin d'images
3. Enregistrement du type d'application
4. Suppression des instances d'application en cours d'exécution
5. Création d’une instance d’application

Dans Visual Studio, appuyer sur **F5** permet de déployer votre application et d’attacher le débogueur à toutes les instances de l’application. Vous pouvez utiliser **Ctrl + F5** pour déployer une application sans débogage ou la publier sur un cluster local ou distant à l’aide du profil de publication.

### <a name="application-debug-mode"></a>Mode de débogage d’application
Visual Studio fournit une propriété appelée **Mode de débogage de l’application**, qui contrôle la façon dont vous souhaitez que Visual Studio gère le déploiement de l’application dans le cadre du débogage.

#### <a name="to-set-the-application-debug-mode-property"></a>Pour définir la propriété Mode de débogage d’application
1. Dans le menu contextuel du projet d’application Service Fabric (*.sfproj), cliquez sur **Propriétés** (ou appuyez sur la touche **F4**).
2. Dans la fenêtre **Propriétés**, définissez la propriété **Mode de débogage d’application**.

![Définir la propriété Mode de débogage d’application][debugmodeproperty]

#### <a name="application-debug-modes"></a>Modes de débogage de l’application

1. **Actualiser l’application** Ce mode vous permet de modifier et de déboguer rapidement votre code, et prend en charge la modification des fichiers web statiques pendant le débogage. Ce mode fonctionne uniquement si votre cluster de développement local est en mode 1 nœud. Il s’agit du Mode de débogage d’application par défaut.
2. **Supprimer l’application** entraîne la suppression de l’application lorsque la session de débogage se termine.
3. **Mise à niveau automatique** L’application continue à s’exécuter lorsque la session de débogage se termine. La session de débogage suivante traitera le déploiement comme une mise à niveau. Le processus de mise à niveau conserve les données que vous avez saisies au cours de la précédente session de débogage.
4. **Conserver l’application** L’application continue à s’exécuter dans le cluster lorsque la session de débogage se termine. Au début de la prochaine session de débogage, l’application sera supprimée.

Avec l’option **Mise à niveau automatique**, les données sont conservées en appliquant les fonctionnalités de mise à niveau d’application de Service Fabric. Pour plus d’informations sur la mise à niveau des applications et sur la façon d’effectuer une mise à niveau dans un environnement réel, consultez [Mise à niveau d’application Service Fabric](service-fabric-application-upgrade.md).

## <a name="add-a-service-to-your-service-fabric-application"></a>Ajouter un service à votre application Service Fabric
Vous pouvez ajouter de nouveaux services à votre application pour étendre ses fonctionnalités. Pour garantir que le service est inclus dans votre package d’application, ajoutez-le via l’élément de menu **Nouveau service Service Fabric…** .

![Ajouter un service Service Fabric][newservice]

Sélectionnez un type de projet Service Fabric à ajouter à votre application et spécifiez un nom pour le service.  Consultez [Choix d’une infrastructure pour votre service](service-fabric-choose-framework.md) pour vous aider à choisir le type de service à utiliser.

![Sélectionner un type de projet de service Service Fabric à ajouter à votre application][addserviceproject]

Le nouveau service est ajouté à votre solution et au package d’application existant. Les références de service et une instance de service par défaut seront ajoutées au manifeste de l’application, provoquant la création et le démarrage du service la prochaine fois que vous déployez l’application.

![Le nouveau service est ajouté à votre manifeste de l’application][newserviceapplicationmanifest]

## <a name="package-your-service-fabric-application"></a>Empaquetage de votre application Service Fabric
Pour déployer l’application et ses services dans un cluster, vous devez créer un package d’application.  Le package organise le manifeste de l’application, les manifestes de service et les autres fichiers nécessaires dans une disposition spécifique.  Visual Studio configure et gère le package dans le dossier du projet d'application, dans le répertoire « pkg ».  Cliquez sur **Package** dans le menu contextuel **Application** pour créer ou mettre à jour le package d’application.

## <a name="remove-applications-and-application-types-using-cloud-explorer"></a>Supprimer des applications et des types d’applications à l’aide de Cloud Explorer
Vous pouvez effectuer des opérations de gestion de cluster de base à partir de Visual Studio à l’aide de Cloud Explorer, que vous pouvez lancer à partir du menu **Affichage** . Par exemple, vous pouvez supprimer des applications et annuler la mise en service de types d’applications sur des clusters locaux ou distants.

![Supprimer une application][removeapplication]

> [!TIP]
> Pour obtenir des fonctionnalités de gestion de clusters enrichies, voir [Visualisation de votre cluster à l’aide de l’outil Service Fabric Explorer](service-fabric-visualizing-your-cluster.md).
>
>

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## <a name="next-steps"></a>Étapes suivantes
* [Modèle d'application Service Fabric](service-fabric-application-model.md)
* [Déploiement d'applications Service Fabric](service-fabric-deploy-remove-applications.md)
* [Gestion des paramètres d’application pour plusieurs environnements](service-fabric-manage-multiple-environment-app-configuration.md)
* [Débogage de votre application Service Fabric](service-fabric-debugging-your-application.md)
* [Visualisation de votre cluster à l’aide de l’explorateur Service Fabric](service-fabric-visualizing-your-cluster.md)

<!--Image references-->
[addserviceproject]:./media/service-fabric-manage-application-in-visual-studio/addserviceproject.png
[manageservicefabric]: ./media/service-fabric-manage-application-in-visual-studio/manageservicefabric.png
[newservice]:./media/service-fabric-manage-application-in-visual-studio/newservice.png
[newserviceapplicationmanifest]:./media/service-fabric-manage-application-in-visual-studio/newserviceapplicationmanifest.png
[debugmodeproperty]:./media/service-fabric-manage-application-in-visual-studio/debugmodeproperty.png
[removeapplication]:./media/service-fabric-manage-application-in-visual-studio/removeapplication.png

---
title: Guide pratique pour déployer un module OPC Twin dans un projet Azure existant | Microsoft Docs
description: Cet article explique comment déployer un module OPC Twin dans un projet existant. Vous pouvez également découvrir comment détecter un problème de déploiement.
author: dominicbetts
ms.author: dobett
ms.date: 11/26/2018
ms.topic: conceptual
ms.service: industrial-iot
services: iot-industrialiot
manager: philmea
ms.openlocfilehash: 0a4866db463dcf4ff17b2f7b25806806a700e8fa
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92071640"
---
# <a name="deploy-opc-twin-to-an-existing-project"></a>Déployer OPC Twin dans un projet existant

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Le module OPC Twin s’exécute sur IoT Edge et fournit plusieurs services de périphérie aux services OPC Twin et Registre.

Avec son module IoT Edge, le microservice OPC Twin facilite la communication entre les opérateurs en usine et les appareils industriels du serveur OPC UA. Il expose des services OPC UA (Parcourir, Lire, Écrire et Exécuter) par le biais de ses API REST. 

Le microservice Registre d’appareil OPC UA fournit l’accès aux applications OPC UA inscrites et à leurs points de terminaison. Les opérateurs et les administrateurs peuvent inscrire de nouvelles applications OPC UA et en désinscrire, et parcourir les applications existantes, y compris leurs points de terminaison. En plus de gérer les applications et les points de terminaison associés, le service de Registre liste également les modules IoT Edge OPC Twin inscrits. L'API de service vous permet de contrôler les fonctionnalités du module de périphérie, par exemple, le démarrage ou l'arrêt de la découverte des serveurs (services d'analyse), ou l'activation de nouveaux jumeaux de points de terminaison qui sont accessibles via le microservice OPC Twin.

Le cœur du module est l’identité Superviseur. Le superviseur gère les jumeaux de point de terminaison, qui correspondent aux points de terminaison du serveur OPC UA activés avec l’API du Registre OPC UA correspondante. Les jumeaux de point de terminaison traduisent le code JSON OPC UA reçu du microservice OPC Twin exécuté dans le cloud en messages binaires OPC UA, lesquels sont ensuite envoyés via un canal sécurisé avec état au point de terminaison managé. Le superviseur fournit également des services de découverte qui envoient les événements de découverte d’appareil au service d’intégration d’appareil OPC UA en vue de leur traitement, ces événements donnant lieu à des mises à jour dans le Registre OPC UA.  Cet article vous montre comment déployer le module OPC Twin dans un projet existant.

> [!NOTE]
> Pour obtenir des informations détaillées et des instructions sur le déploiement, consultez le [dépôt](https://github.com/Azure/azure-iiot-opc-twin-module) GitHub.

## <a name="prerequisites"></a>Prérequis

Vérifiez que PowerShell et les extensions [AzureRM PowerShell](/powershell/azure/azurerm/install-azurerm-ps) sont installés. Si ce n’est déjà fait, clonez ce dépôt GitHub. Exécutez les commandes suivantes dans PowerShell :

```powershell
git clone --recursive https://github.com/Azure/azure-iiot-components.git
cd azure-iiot-components
```

## <a name="deploy-industrial-iot-services-to-azure"></a>Déployer des services IoT industriels dans Azure

1. Dans votre session PowerShell, exécutez :

    ```powershell
    set-executionpolicy -ExecutionPolicy Unrestricted -Scope Process
    .\deploy.cmd
    ```

2. Suivez les invites pour nommer le groupe de ressources du déploiement et nommer le site web.   Le script déploie les microservices et leurs dépendances avec la plateforme Azure dans le groupe de ressources de votre abonnement Azure.  Le script inscrit également une application dans votre locataire Azure Active Directory (AAD) pour prendre en charge l’authentification OAUTH.  Le déploiement prend plusieurs minutes.  Une fois que la solution a été déployée, vous voyez une sortie similaire à celle-ci :

   ![Déployer un service IoT industriel avec OPC Twin dans un projet existant](media/howto-opc-twin-deploy-existing/opc-twin-deploy-existing1.png)

   La sortie inclut l’URL du point de terminaison public. 

3. À la fin du script, choisissez si vous souhaitez enregistrer le fichier `.env`.  Vous avez besoin du fichier d'environnement `.env` si vous prévoyez de vous connecter au point de terminaison cloud à l'aide de certains outils comme la console, ou de déployer des modules de développement et de débogage.

## <a name="troubleshooting-deployment-failures"></a>Résoudre les problèmes de déploiement

### <a name="resource-group-name"></a>Nom de groupe ressources

Veillez à utiliser un nom de groupe de ressources court et simple.  Le nom est également utilisé pour nommer les ressources : il doit donc être conforme aux spécifications de nommage des ressources.  

### <a name="website-name-already-in-use"></a>Nom du site web déjà utilisé

Le nom du site web est peut-être déjà utilisé ailleurs.  Si vous rencontrez cette erreur, choisissez un nom d’application différent.

### <a name="azure-active-directory-aad-registration"></a>Inscription dans Azure Active Directory (AAD)

Le script de déploiement essaie d’inscrire deux applications AAD dans Azure Active Directory.  En fonction de vos droits sur le locataire AAD sélectionné, le déploiement peut échouer. Nous avons deux options :

1. Si vous avez choisi un locataire AAD dans une liste de locataires, redémarrez le script et choisissez un autre locataire dans la liste.
2. Vous pouvez aussi déployer un locataire AAD privé dans un autre abonnement, redémarrer le script et choisir de l’utiliser.

> [!WARNING]
> Ne continuez JAMAIS sans authentification.  Si vous choisissez un déploiement sans authentification, tout le monde pourra accéder à vos points de terminaison OPC Twin à partir d’un point d’accès Internet non authentifié.   Vous pouvez toujours choisir l’[option de déploiement « local »](howto-opc-twin-deploy-dependencies.md) pour essayer.

## <a name="deploy-an-all-in-one-industrial-iot-services-demo"></a>Déployer une démonstration tout-en-un de services IoT industriels

Au lieu de juste déployer les services et les dépendances, vous pouvez aussi déployer une démonstration tout-en-un.  La démonstration tout-en-un contient trois serveurs OPC UA, le module OPC Twin, tous les microservices et un exemple d'application web.  Cet exemple sert à des fins de démonstration.

1. Assurez-vous d’avoir un clone du dépôt (voir ci-dessus). Ouvrez une invite PowerShell à la racine du dépôt et exécutez :

    ```powershell
    set-executionpolicy -ExecutionPolicy Unrestricted -Scope Process
    .\deploy -type demo
    ```

2. Suivez les invites pour renommer le groupe de ressources et nommer le site web.  Une fois déployé, le script affiche l’URL du point de terminaison de l’application web.

## <a name="deployment-script-options"></a>Options du script de déploiement

Le script utilise les paramètres suivants :

```powershell
-type
```

Le type de déploiement (machine virtuelle, local, démonstration)

```powershell
-resourceGroupName
```

Peut être le nom d’un groupe de ressources existant ou nouveau.

```powershell
-subscriptionId
```

(Facultatif) L’ID de l’abonnement où déployer les ressources.

```powershell
-subscriptionName
```

Ou le nom de l’abonnement.

```powershell
-resourceGroupLocation
```

(Facultatif) L’emplacement du groupe de ressources. S’il est spécifié, le script essaiera de créer un groupe de ressources à cet emplacement.

```powershell
-aadApplicationName
```

Un nom sous lequel inscrire l’application AAD.

```powershell
-tenantId
```

Le client AAD à utiliser.

```powershell
-credentials
```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment déployer OPC Twin dans un projet existant, voici l’étape suivante suggérée :

> [!div class="nextstepaction"]
> [Sécuriser les communications du client OPC UA et d'OPC UA PLC](howto-opc-vault-secure.md)
---
title: Création et déploiement d’un service cloud | Microsoft Docs
description: Découvrez comment utiliser la méthode Création rapide pour créer un service cloud et comment utiliser Charger pour charger et déployer un package de services cloud dans Azure.
services: cloud-services
documentationcenter: ''
author: tgore03
ms.service: cloud-services
ms.topic: article
ms.date: 05/18/2017
ms.author: tagore
ms.openlocfilehash: 66938975784f1de2abdc0ac22e62aaca82279f86
ms.sourcegitcommit: 419c8c8061c0ff6dc12c66ad6eda1b266d2f40bd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/18/2020
ms.locfileid: "92164162"
---
# <a name="how-to-create-and-deploy-a-cloud-service"></a>Création et déploiement d’un service cloud
Le portail Azure vous permet de créer et de déployer un service cloud de deux manières : *Création rapide* et *Création personnalisée* .

Cet article explique comment utiliser la méthode Quick Create pour créer un service cloud et comment utiliser ensuite **Upload** pour télécharger et déployer un package de service cloud dans Azure. Si vous utilisez cette méthode, le portail Azure met à votre disposition tous les liens nécessaires pour remplir les conditions requises au fur et à mesure. Si vous êtes prêt à déployer votre service cloud lorsque vous le créez, vous pouvez effectuer ces deux opérations en même temps à l'aide de Création personnalisée.

> [!NOTE]
> Si vous prévoyez de publier votre service cloud à partir d’Azure DevOps, utilisez Création rapide, puis configurez la publication d’Azure DevOps à partir du Démarrage rapide Azure ou du tableau de bord. Pour plus d’informations, voir [Livraison continue à Azure à l’aide d’Azure DevOps][TFSTutorialForCloudService] ou l’aide de la page **Démarrage rapide** .
>
>

## <a name="concepts"></a>Concepts
Trois composants sont nécessaires pour déployer une application en tant que service cloud dans Azure :

* **Définition de service**  
  Le fichier de définition de service cloud (.csdef) définit le modèle de service, notamment le nombre de rôles.
* **Configuration de service**  
  Le fichier de configuration de service cloud (.cscfg) contient les paramètres de configuration du service cloud et des différents rôles, notamment le nombre d’instances de rôle.
* **Package de service**  
  Le package de service (.cspkg) contient le code d’application, les configurations et le fichier de définition de service.

Pour plus d’informations sur ces composants et sur la création d’un package, cliquez [ici](cloud-services-model-and-package.md).

## <a name="prepare-your-app"></a>Préparation de votre application
Avant de déployer un service cloud, vous devez créer le package de service cloud (.cspkg) à partir du code de l'application, ainsi que le fichier de configuration de service cloud (.cscfg). Le Kit de développement logiciel (SDK) Azure fournit les outils nécessaires à la préparation des fichiers de déploiement. Vous pouvez installer le Kit de développement logiciel (SDK) depuis la page des [téléchargements Azure](https://azure.microsoft.com/downloads/) , dans le langage souhaité pour le développement de votre code.

Trois fonctions du service cloud nécessitent une configuration spécifique avant d'exporter le package de service :

* Si vous souhaitez déployer un service cloud qui utilise le protocole TLS (Transport Layer Security), anciennement SSL (Secure Sockets Layer), pour le chiffrement des données, [configurez votre application](cloud-services-configure-ssl-certificate-portal.md#modify) pour TLS.
* Si vous voulez configurer les connexions Bureau à distance aux instances de rôle, [configurez les rôles](cloud-services-role-enable-remote-desktop-new-portal.md) pour le Bureau à distance.
* Si vous voulez configurer la surveillance détaillée pour votre service cloud, activez Diagnostics Azure pour le service cloud. *Surveillance minimale* (niveau de surveillance par défaut) utilise des compteurs de performances récupérés sur le système d'exploitation hôte des instances de rôle (machine virtuelle). *surveillance détaillée* recueille des mesures supplémentaires sur les données de performances dans les instances de rôle, afin de permettre une analyse plus fine des problèmes qui surviennent au cours du traitement de l'application. Pour savoir comment activer Diagnostics Azure, consultez la page [Activation des diagnostics dans Azure](cloud-services-dotnet-diagnostics.md).

Pour créer un service cloud avec des déploiements de rôles web ou de rôles de travail, vous devez [créer le package de service](cloud-services-model-and-package.md#servicepackagecspkg).

## <a name="before-you-begin"></a>Avant de commencer
* Si vous n'avez pas installé le Kit de développement logiciel (SDK), cliquez sur **Install Azure SDK** pour ouvrir la page des [téléchargements Azure](https://azure.microsoft.com/downloads/), puis téléchargez le Kit de développement logiciel dans le langage souhaité pour le développement de votre code. (Vous pourrez le faire ultérieurement.)
* Si des instances de rôle nécessitent des certificats, créez ces certificats. Les services cloud requièrent un fichier .pfx avec une clé privée. Vous pouvez charger les certificats sur Azure quand vous créez et déployez le service cloud.

## <a name="create-and-deploy"></a>Création et déploiement
1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Cliquez sur **Créer une ressource > Compute** , faites défiler la page vers le bas, puis cliquez sur **Service cloud** .

    ![Capture d’écran avec l’option Service cloud sous Créer une ressource > Calcul en évidence.](media/cloud-services-how-to-create-deploy-portal/create-cloud-service.png)
3. Dans le nouveau volet **Service cloud** , entrez une valeur pour le **nom DNS** .
4. Créez un **groupe de ressources** ou sélectionnez-en un.
5. Sélectionnez un **emplacement** .
6. Cliquez sur **Package** . Cette opération ouvre le volet **Charger un package** . Renseignez les champs obligatoires. Si l’un de vos rôles contient une seule et même instance, vérifiez que l’option **Déployer même si un ou plusieurs rôles ne contiennent qu’une seule et même instance** est sélectionnée.
7. Vérifiez que l’option **Démarrer le déploiement** est sélectionnée.
8. Cliquez sur **OK** pour fermer le volet **Charger un package** .
9. Si vous n’avez aucun certificat à ajouter, cliquez sur **Créer** .

    ![Publier votre service cloud](media/cloud-services-how-to-create-deploy-portal/select-package.png)

## <a name="upload-a-certificate"></a>Téléchargement d'un certificat
Si votre package de déploiement a été [configuré pour utiliser des certificats](cloud-services-configure-ssl-certificate-portal.md#modify), vous pouvez charger le certificat maintenant.

1. Sélectionnez **Certificats** et, dans le volet **Ajouter des certificats** , sélectionnez le fichier .pfx du certificat TLS/SSL et indiquez le **mot de passe** pour le certificat.
2. Cliquez sur **Joindre un certificat** , puis sur **OK** dans le volet **Ajouter des certificats** .
3. Cliquez sur **Créer** dans le volet **Service cloud** . Lorsque le déploiement atteint l'état **Ready** , vous pouvez passer aux étapes suivantes.

    ![Capture d’écran mettant en évidence le processus de chargement d’un certificat.](media/cloud-services-how-to-create-deploy-portal/attach-cert.png)

## <a name="verify-your-deployment-completed-successfully"></a>Vérifier la réussite du déploiement
1. Cliquez sur l’instance de service cloud.

    L'état présente le service comme étant **En cours d'exécution** .
2. Sous **Bases** , cliquez sur l’ **URL du site** pour ouvrir le service cloud dans un navigateur web.

    ![CloudServices_QuickGlance](./media/cloud-services-how-to-create-deploy-portal/running.png)

[TFSTutorialForCloudService]: ./cloud-services-choose-me.md

## <a name="next-steps"></a>Étapes suivantes
* [Configuration générale de votre service cloud](cloud-services-how-to-configure-portal.md).
* Configurez un [nom de domaine personnalisé](cloud-services-custom-domain-name-portal.md).
* [Gérez votre service cloud](cloud-services-how-to-manage-portal.md).
* Configurez des [certificats TLS/SSL](cloud-services-configure-ssl-certificate-portal.md).
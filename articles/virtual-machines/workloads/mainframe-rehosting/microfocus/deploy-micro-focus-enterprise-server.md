---
title: Déployer Micro Focus Enterprise Server 5.0 sur AKS | Microsoft Docs
description: Réhébergez vos charges de travail mainframe IBM z/OS en utilisant l’environnement de développement et de test Micro Focus sur des machines virtuelles Azure.
services: virtual-machines-linux
documentationcenter: ''
author: maggsl
ms.author: edprice
manager: edprice
editor: edprice
ms.topic: conceptual
ms.date: 06/29/2020
tags: ''
keywords: ''
ms.service: multiple
ms.openlocfilehash: 6780942d922f885c7afebd8e64f4f28654c3800e
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87042545"
---
# <a name="deploy-micro-focus-enterprise-server-50-to-aks"></a>Déployer Micro Focus Enterprise Server 5.0 sur AKS

Dans un autre [article](./run-enterprise-server-container.md), j’ai décrit les étapes permettant d’exécuter Micro Focus Enterprise Server 5.0 dans un conteneur Docker. J’aimerais donc maintenant vous montrer comment aller plus loin et déployer l’image Docker que vous avez créée dans Azure Kubernetes service (AKS).

Azure Kubernetes Service est un service d’orchestration managé basé sur Kubernetes. Il vous permet de déployer, de mettre à l’échelle et de gérer des conteneurs Docker (et autres applications basées sur des conteneurs) sur un cluster d’hôtes de conteneur.

Ce processus comprend trois étapes. Vous devez :

1.  Créer un registre de conteneurs Azure Container Registry pour stocker votre image Docker.

2.  Créer un cluster Azure Kubernetes pour exécuter l’image Docker.

3.  Exécutez l'application.

Cela vous permet de mettre à l’échelle les charges de travail de modernisation de mainframe dans Azure, en tirant parti de la plateforme Cloud.

Vous êtes prêt ? Allons-y !

## <a name="create-the-azure-container-registry"></a>Créer le registre Azure Container Registry

Dans le portail Azure, sélectionnez **Créer une ressource** dans le coin supérieur gauche. Dans le tableau de bord Marketplace, sélectionnez **Conteneurs,** puis **Registre de conteneurs**. Vous accédez au volet **Créer un registre de conteneurs** dans lequel vous devez renseigner le **nom du registre**, l’**abonnement Azure**, le **groupe de ressources** et l’**emplacement**. Le **nom du registre** doit être résolu et doit donc être unique. Sélectionnez le **groupe de ressources** que vous avez utilisé dans le billet de blog précédent et le même **emplacement** correspondant. Sélectionnez **Activer** pour **Utilisateur administrateur** et **De base** pour la **SKU**. Une fois que tous les éléments sont renseignés, sélectionnez **Créer**.

![Créer une interface de registre de conteneurs](media/deploy-image-1.png)

Une fois le registre déployé, sélectionnez **Accéder à la ressource**. Vous arrivez au panneau principal d’Azure Container Registry. Une fonctionnalité intéressante ici est l’option de menu **Démarrage rapide**. Sélectionnez-le et vous verrez des instructions sur ce qui doit être fait pour envoyer et extraire des images du registre. Allons-y :

1.  **Installez Docker** : vous n’avez pas besoin de vous en préoccuper, car c’est déjà fait.

2.  **Exécutez l’image de base « Hello – World »**  : là encore, ce n’est pas nécessaire, mais n’hésitez pas à tester cette opération.

3.  **Connectez-vous à votre registre de conteneurs** : vous devez le faire à partir de la machine virtuelle. Copiez la commande dans le Presse-papiers ou le Bloc-notes.

    Pour le registre que j’ai créé, la commande est : **dockr login acrmf50.azurecr.io**

4.  **Transmettre au registre** : vous devez effectuer cette opération pour l’image Micro Focus et quand vous êtes connecté à la machine virtuelle.

5.  **Extraire de votre registre** : cela ne s’applique pas à cette procédure pas à pas, mais il est utile de le savoir si vous devez exécuter une autre image Docker.

Avant de quitter le portail, vous devez obtenir les informations d’identification du registre afin de pouvoir vous y connecter. Quittez le panneau **Démarrage rapide** et sélectionnez **Clés d’accès** dans le menu Registre. Copiez le **nom d’utilisateur** et l’un des **mots de passe** (il y en a deux) dans le Presse-papiers ou le Bloc-notes. Vous en aurez besoin ultérieurement pour vous connecter.

Maintenant que vous savez ce que vous devez faire, connectez-vous à la machine virtuelle.

## <a name="rdp-to-the-virtual-machine-you-used-to-create-the-docker-image"></a>Utilisez une session de bureau à distance vers la machine virtuelle que vous avez utilisée pour créer l’image Docker.

Étant donné que vous avez déjà créé l’image Docker sur un serveur Windows 2016, vous devez vous connecter à cette machine virtuelle. À partir de cette machine virtuelle, vous pouvez envoyer l’image vers le registre Azure Container Registry que vous venez de créer. Accédez à la machine virtuelle dans le Portail Azure, puis sélectionnez **Vue d’ensemble**, puis **Connecter**. Cela vous permet de vous connecter à la machine virtuelle via le protocole RDP (Remote Desktop Protocol). Vous devez utiliser les informations d’identification que vous avez utilisées lors de la création de la machine virtuelle.

## <a name="log-in-and-push-the-image-to-the-registry"></a>Connectez-vous et envoyez l’image vers le registre

Une fois que vous êtes connecté, ouvrez une invite de commandes et lancez les commandes Docker suivantes :

-   **docker images** : affiche une liste des images actuellement installées sur la machine virtuelle. Notez le **Microfocus/es-acctdemo**, car il s’agit de celui que vous utiliserez.

-   **docker login acrmf50.azurecr.io** : le format correct est ici *docker login \<registry name\>* . Remplacez le nom que vous avez utilisé lors de la création du registre.

    -   Vous aurez besoin du **nom d’utilisateur** et du **mot de passe** que vous avez copiés sur le Portail Azure. Un résultat similaire à cette image doit s’afficher.

    ![Capture d’écran d’une invite de commandes Administrateur](media/deploy-image-2.png)

-   **docker tag microfocus/es-acctdemo acrmf50.azurecr.io/es-acctdemo** : il s’agit de l’image appropriée avec le nom du référentiel. **REMARQUE** : Si le nom \<microfocus/es-acctdemo\> ne fonctionne pas, essayez d’utiliser l’ID d’image complète. Une fois que vous avez exécuté la commande, tapez **docker images –no-trunc**. Vous devriez voir quelque chose de similaire à ceci. Notez que l’image est correctement étiquetée.

    ![Écran Invite de commandes Administrateur](media/deploy-image-3.png)

-   **docker push acrmf50.azurecr.io/es-acctdemo** : ceci déclenche la transmission vers votre référentiel. Étant donné que la taille est de 15 Go, l’exécution prend quelques minutes. Si tout se passe bien, vous devriez voir une image ressemblant à celle-ci.

    ![Écran Invite de commandes Administrateur](media/deploy-image-4.png)

Revenez maintenant au Portail Azure, en particulier dans le **Référentiel**. Dans le menu du **référentiel**, sélectionnez **Référentiels**, et vous devriez voir **es-acctdemo** dans la liste. Créez maintenant le cluster AKS.

## <a name="create-the-azure-kubernetes-aks-cluster"></a>Créer le cluster Azure Kubernetes Service (AKS)

Dans le Portail Azure, sélectionnez **Créer une ressource**, puis **Conteneurs/Kubernetes service** dans le menu **Marketplace**. Ensuite, vous devez remplir le panneau **Créer un cluster Kubernetes**. Veillez à tenir le cluster dans la même région et le même groupe de ressources que celui que vous avez utilisé. Vous pouvez accepter les autres valeurs par défaut, à l’exception du **nombre de nœuds**, qui doit uniquement être 1. Quand vous avez terminé, sélectionnez **Vérifier + créer**.

![Écran Créer un cluster Kubernetes](media/deploy-image-5.png)

Une fois l’opération terminée, le déploiement placera les artefacts du **service Kubernetes** dans le **groupe de ressources** que vous avez sélectionné dans le panneau. Toutefois, le cluster réel verra son propre groupe de ressources créé pendant le déploiement. Si vous sélectionnez **Groupes de ressources** dans le menu de gauche, vous pouvez le trouver en fonction de la convention d’affectation de noms. Voici une image de la mienne : c’est la dernière de la liste.

![Capture d’écran des groupes de ressources](media/deploy-image-6.png)

**Exécuter l’image**

Il est maintenant temps d’extraire l’image et de l’exécuter dans AKS. Pour cela, le moyen le plus simple consiste à utiliser Cloud Shell dans le portail Azure. Sélectionnez l’icône en haut à droite du portail Azure. Notez que pour cette procédure pas à pas, j’utilise l’interpréteur de commandes Bash.

![Capture d’écran de l’icône Cloud Shell](media/deploy-image-7.png)

Une fois l’interpréteur de commandes chargé, tapez la commande suivante :

**kubectl run es-acctdemo --image acrmf50.azurecr.io/es-acctdemo --port=9040**

Cette opération extrait l’image du référentiel **acrmf50.azurecr.io** et la charge dans AKS. Elle exécute ensuite l’image avec le port 9040 ouvert. Vous vous souvenez peut-être qu’il s’agit du port que vous aviez ouvert pour votre image Docker. Vous en avez besoin pour accéder à Entreprise Server.

Kubernetes doit répondre par un message indiquant que le déploiement a été créé.

![Capture d’écran d’un message de déploiement](media/deploy-image-8.jpg)

Pour voir si le conteneur est en cours d’exécution, tapez : **kubectl get pods**.

Vous devriez voir es-acctdemo comme pod en cours d’exécution, comme dans l’image suivante.

![Capture d’écran d’es-acctdemo comme pod en cours d’exécution](media/deploy-image-9.png)

Félicitations ! Vous exécutez maintenant Enterprise Server dans Azure Kubernetes Service. Dans l’article suivant, je vais vous montrer comment accéder à la console d’administration d’Enterprise Server et comment exploiter Kubernetes pour monter en charge votre déploiement.

---
title: Créer un environnement de cluster Service Fabric dans Azure DevTest Labs
description: Découvrez comment créer un environnement avec un cluster Service Fabric autonome et comment démarrer et arrêter ce cluster à l’aide de planifications.
author: EMaher
ms.topic: article
ms.date: 06/26/2020
ms.author: enewman
ms.openlocfilehash: 530cf3b20820e34913612419d0ffa731a70f6a58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85484007"
---
# <a name="create-an-environment-with-self-contained-service-fabric-cluster-in-azure-devtest-labs"></a>Créer un environnement avec un cluster Service Fabric autonome dans Azure DevTest Labs
Cet article fournit des informations sur la création d’un environnement avec un cluster Service Fabric autonome dans Azure DevTest Labs. 

## <a name="overview"></a>Vue d’ensemble
DevTest Labs peut créer des environnements de test autonomes tels que définis par les modèles Azure Resource Manager. Ces environnements contiennent les deux ressources IaaS, comme les machines virtuelles et les ressources PaaS, comme Service Fabric. DevTest Labs vous permet de gérer des machines virtuelles dans un environnement avec des commandes permettant de contrôler ces machines virtuelles. Ces commandes vous permettent de démarrer ou d’arrêter une machine virtuelle en fonction d’une planification. De même, DevTest Labs peut aussi vous aider à gérer les clusters Service Fabric dans un environnement. Vous pouvez démarrer ou arrêter un cluster Service Fabric dans un environnement manuellement ou via une planification.

## <a name="create-a-service-fabric-cluster"></a>Créer un cluster Service Fabric
Les clusters Service Fabric sont créés à l’aide des environnements dans DevTest Labs. Chaque environnement est défini par un modèle Azure Resource Manager dans un référentiel Git. Le [référentiel Git public](https://github.com/Azure/azure-devtestlab/tree/master/Environments/) de DevTest Labs contient le modèle Resource Manager permettant de créer un cluster Service Fabric dans le dossier [ServiceFabric-Cluster](https://github.com/Azure/azure-devtestlab/tree/master/Environments/ServiceFabric-LabCluster). 

1. Commencez par créer un labo dans Azure DevTest Labs à l’aide des instructions de l’article suivant : [Créez un labo](devtest-lab-create-lab.md). Notez que l’option **Environnements publics** est **activée** par défaut. 
2. Vérifiez que le fournisseur Service Fabric est inscrit pour votre abonnement en procédant comme suit :
    1. Sélectionnez **Abonnements** dans le menu de navigation de gauche, puis sélectionnez votre **abonnement**
    2. Dans la page **Abonnement**, sélectionnez **Fournisseurs de ressources** dans la section **Paramètres** du menu de gauche. 
    3. Si **Microsoft.ServiceFabric** n’est pas inscrit, sélectionnez **Inscrire**. 
3. Dans la page **DevTest Lab** pour votre labo, sélectionnez **+ Ajouter** dans la barre d’outils. 
    
    ![Ajouter un bouton à la barre d’outils](./media/create-environment-service-fabric-cluster/add-button.png)
3. Dans la page **Choisir une base**, sélectionnez **Cluster Service Fabric Lab** dans la liste. 

    ![Sélectionnez Cluster Service Fabric Lab dans la liste](./media/create-environment-service-fabric-cluster/select-service-fabric-cluster.png)
4. Dans la page **Configurer les paramètres**, effectuez les étapes suivantes : 
    1. Spécifiez le **nom** de votre **environnement** de cluster. Il s’agit du nom du groupe de ressources dans Azure dans lequel le cluster Service Fabric va être créé. 
    2. Sélectionnez le **système d’exploitation** des machines virtuelles du cluster. La valeur par défaut est : **Windows**.
    3. Spécifiez le nom de l’**administrateur** du cluster. 
    4. Spécifiez le **mot de passe** de l’administrateur. 
    5. Pour le **certificat**, entrez vos informations de certificat sous forme de chaîne codée en Base64. Pour créer un fichier de certificat, procédez comme suit :
        1. Téléchargez le fichier **Create-ClusterCertificate.ps1** dans le [référentiel Git](https://github.com/Azure/azure-devtestlab/tree/master/Environments/ServiceFabric-LabCluster). Vous pouvez également cloner le référentiel sur votre ordinateur. 
        2. Lancez **PowerShell**. 
        3. Exécutez le fichier **ps1** à l’aide de la commande `.\Create-ClusterCertificate.ps1`. Vous voyez alors un fichier texte ouvert dans le Bloc-notes, avec les informations dont vous avez besoin pour remplir les champs du certificat sur cette page. . 
    6. Entrez le **mot de passe** du certificat.
    7. Spécifiez l’**empreinte numérique** de votre certificat.
    8. Sélectionnez **Ajouter** dans la page **Configurer les paramètres**. 

        ![Configurer les paramètres du cluster](./media/create-environment-service-fabric-cluster/configure-settings.png)
5. Une fois le cluster créé, vous voyez un groupe de ressources portant le nom de l’environnement que vous avez fourni à l’étape précédente. Lorsque vous le développez, vous voyez le cluster Service Fabric qu’il contient. Si l’état du groupe de ressources est bloqué au niveau **Création**, sélectionnez **Actualiser** dans la barre d’outils. L’environnement de **cluster Service Fabric** crée un cluster à 5 nœuds 1-nodetype sur Linux ou Windows.

    Dans l’exemple suivant, **mysfabricclusterrg** est le nom du groupe de ressources qui est créé spécifiquement pour le cluster Service Fabric. Il est important de noter que les environnements lab sont autonomes dans le groupe de ressources dans lequel ils sont créés. Cela signifie que le modèle qui définit l’environnement peut accéder uniquement aux ressources au sein du groupe de ressources nouvellement créé ou [des réseaux virtuels configurés pour être utilisés par le labo](devtest-lab-configure-vnet.md). Cet exemple crée toutes les ressources requises dans le même groupe de ressources.

    ![Cluster créé](./media/create-environment-service-fabric-cluster/cluster-created.png)

## <a name="start-or-stop-the-cluster"></a>Démarrer ou arrêter le cluster
Vous pouvez démarrer ou arrêter le cluster à partir de la page DevTest Lab ou de la page du cluster Service Fabric fourni par DevTest Labs. 

### <a name="from-the-devtest-lab-page"></a>À partir de la page DevTest Lab
Vous pouvez démarrer ou arrêter le cluster sur la page DevTest Lab de votre labo. 

1. Sélectionnez les **points de suspension (...)** pour le cluster Service Fabric, comme illustré dans l’image suivante : 

    ![Commandes de démarrage et d’arrêt pour le cluster](./media/create-environment-service-fabric-cluster/start-stop-on-devtest-lab-page.png)

2. Le menu contextuel propose deux commandes pour **démarrer** et **arrêter** le cluster. La commande de démarrage démarre tous les nœuds dans un cluster. La commande d’arrêt arrête démarre tous les nœuds dans un cluster. Une fois un cluster arrêté, le cluster Service Fabric reste prêt, mais aucun nœud n’est disponible jusqu’à ce que la commande de démarrage soit réémise sur le cluster dans le labo.

    Il existe quelques points à tenir en tête lorsque vous utilisez un cluster Service Fabric dans un environnement de test. Il peut s’écouler un certain temps avant que le cluster Service Fabric ne soit totalement actualisé une fois que les nœuds ont été redémarrés. Pour conserver les données entre l’arrêt et le démarrage, les données doivent être sauvegardées sur un disque managé attaché à la machine virtuelle. L’utilisation d’un disque managé connecté a des conséquences sur les performances et il est donc recommandé pour les environnements de test uniquement. Si le disque utilisé pour le stockage de données n’est pas sauvegardé, les données sont perdues lorsque la commande d’arrêt est émise sur le cluster.

### <a name="from-the-service-fabric-cluster-page"></a>Depuis la page du cluster Service Fabric 
Il existe une autre méthode pour démarrer ou arrêter le cluster. 

1. Dans l’arborescence de la page DevTest Lab, sélectionnez votre cluster Service Fabric. 

    ![Sélectionner votre cluster](./media/create-environment-service-fabric-cluster/select-cluster.png)

2. Dans la page **Cluster Service Fabric** de votre cluster, vous voyez les commandes sur la barre d’outils pour démarrer ou arrêter le cluster. 

    ![Commandes de démarrage et d’arrêt sur la page de cluster Service Fabric](./media/create-environment-service-fabric-cluster/start-stop-on-cluster-page.png)

## <a name="configure-auto-startup-and-auto-shutdown-schedule"></a>Configurer le démarrage automatique et l’arrêt automatique
Les clusters Service Fabric peuvent également être démarrés ou arrêtés en fonction d’une planification. Cette expérience est similaire à l’expérience des machines virtuelles dans un labo. Pour réaliser des économies, par défaut, chaque cluster créé automatiquement dans un labo s’arrête automatiquement au moment défini par la [stratégie d’arrêt](devtest-lab-set-lab-policy.md?#set-auto-shutdown-policy) du labo. Vous pouvez remplacer cette configuration en spécifiant si le cluster doit être arrêté ou en spécifiant l’heure à laquelle le cluster est arrêté. 

![Planifications existantes pour le démarrage automatique et l’arrêt automatique](./media/create-environment-service-fabric-cluster/existing-schedules.png)

### <a name="opt-in-to-the-auto-start-schedule"></a>Accepter la planification du démarrage automatique
Pour accepter la planification de démarrage, procédez comme suit :

1. Sélectionnez **Démarrage automatique** dans le menu de gauche.
2. Sélectionnez **Activé** pour **Autoriser la planification du démarrage automatique pour ce cluster Service Fabric**. Cette page est activée uniquement si le propriétaire du labo a autorisé les utilisateurs à démarrer automatiquement leurs machines virtuelles ou leurs clusters Service Fabric.
3. Sélectionnez **Enregistrer** dans la barre d’outils. 

    ![Page de démarrage automatique](./media/create-environment-service-fabric-cluster/set-auto-start-settings.png)

### <a name="configure-auto-shutdown-settings"></a>Configurer les paramètres d’arrêt automatique 
Pour modifier les paramètres d’arrêt, procédez comme suit :

1. Sélectionnez **Arrêt automatique** dans le menu de gauche. 
2. Sur cette page, vous pouvez refuser l’arrêt automatique en sélectionnant **Désactivé** pour **Activé**. 
3. Si vous avez sélectionné **Activé** pour **Activé**, procédez comme suit :
    1. Spécifiez l’**heure** de l’arrêt.
    2. Spécifiez le **fuseau horaire** de l’heure. 
    3. Spécifiez si vous souhaitez que DevTest Labs vous envoie des **notifications** avant l’arrêt automatique. 
    4. Si vous avez sélectionné **Oui** pour l’option de notification, spécifiez l’**URL du Webhook** et/ou l’**adresse e-mail** à laquelle envoyer les notifications. 
    5. Sélectionnez **Enregistrer** dans la barre d’outils.

        ![Page de l’arrêt automatique](./media/create-environment-service-fabric-cluster/auto-shutdown-settings.png)

## <a name="to-view-nodes-in-the-service-fabric-cluster"></a>Pour voir les nœuds dans le cluster Service Fabric
La page du cluster Service Fabric que vous avez vue précédemment dans la procédure est spécifique à la page DevTest Labs. Elle ne vous montre pas les nœuds du cluster. Pour voir plus d’informations sur le cluster, procédez comme suit :

1. Sur la page **DevTest Lab** de votre labo, sélectionnez le **groupe de ressources** dans l’arborescence dans la section **Mes machines virtuelles**.

    ![Sélection du groupe de ressources](./media/create-environment-service-fabric-cluster/select-resource-group.png)
2. Pour voir toutes les ressources d’un groupe de ressources sous forme de liste, utilisez la page **Groupe de ressources**. Sélectionnez votre **cluster Service Fabric** dans la liste. 

    ![Sélectionnez votre cluster dans la liste](./media/create-environment-service-fabric-cluster/select-cluster-resource-group-page.png)
3. Vous voyez la page **Cluster Service Fabric** de votre cluster. Il s’agit de la page fournie par Service Fabric. Elle contient toutes les informations sur les clusters, comme les nœuds, les types de nœuds, etc.

    ![Page d’accueil du cluster Service Fabric](./media/create-environment-service-fabric-cluster/service-fabric-cluster-page.png)

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur les environnements, consultez les articles suivants : 

- [Créer des environnements de plusieurs machines virtuelles et des ressources PaaS avec les modèles Azure Resource Manager](devtest-lab-create-environment-from-arm.md)
- [Configurer et utiliser des environnements publics dans Azure DevTest Labs](devtest-lab-configure-use-public-environments.md)

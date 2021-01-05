---
title: Créer une application conteneur Linux sur Service Fabric dans Azure
description: Dans ce démarrage rapide, vous allez concevoir une image Docker avec votre application, envoyez l’image vers un registre de conteneurs, puis déployez votre conteneur sur un cluster Service Fabric.
ms.topic: quickstart
ms.date: 07/22/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 27d9c96643feb805a785e1e535cd8cac2602082b
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92741918"
---
# <a name="quickstart-deploy-linux-containers-to-service-fabric"></a>Démarrage rapide : Déployer des conteneurs Linux sur Service Fabric

Azure Service Fabric est une plateforme de systèmes distribués pour le déploiement et la gestion de microservices et conteneurs extensibles et fiables.

Ce guide de démarrage rapide montre comment déployer des conteneurs Linux sur un cluster Service Fabric sur Azure. Une fois le déploiement terminé, vous disposez d’une application de vote composée d’un serveur web frontal Python et d’un serveur principal Redis en cours d’exécution dans un cluster Service Fabric. Vous apprendrez également comment basculer une application et mettre à l’échelle une application dans votre cluster.

![Page web de l’application de vote][quickstartpic]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide de démarrage rapide :

1. Créer un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer si vous n’avez pas d’abonnement

2. Installer [Azure CLI](/cli/azure/install-azure-cli-apt?view=azure-cli-latest)

3. Installer le [kit SDK et l’interface CLI Service Fabric](service-fabric-get-started-linux.md#installation-methods)

4. Installer [Git](https://git-scm.com/)


## <a name="get-the-application-package"></a>Obtenir le package d’application

Pour déployer des conteneurs sur Service Fabric, vous avez besoin d’un ensemble de fichiers manifestes (la définition d’application), qui décrivent les conteneurs individuels et l’application.

Dans une console, utilisez git pour cloner une copie de la définition d’application, puis remplacez les répertoires par le répertoire `Voting` dans votre clone.

```bash
git clone https://github.com/Azure-Samples/service-fabric-containers.git

cd service-fabric-containers/Linux/container-tutorial/Voting
```

## <a name="create-a-service-fabric-cluster"></a>Créer un cluster Service Fabric

Pour déployer l’application sur Azure, vous avez besoin d’un cluster Service Fabric pour exécuter l’application. Les commandes suivantes créent un cluster à cinq nœuds dans Azure.  Elles créent également un certificat auto-signé, l’ajoutent à un coffre de clés et téléchargent le certificat localement. Le nouveau certificat sert à sécuriser le cluster au moment du déploiement et à authentifier les clients.

```azurecli
#!/bin/bash

# Variables
ResourceGroupName='containertestcluster' 
ClusterName='containertestcluster' 
Location='eastus' 
Password='q6D7nN%6ck@6' 
Subject='containertestcluster.eastus.cloudapp.azure.com' 
VaultName='containertestvault' 
VmPassword='Mypa$$word!321'
VmUserName='sfadminuser'

# Login to Azure and set the subscription
az login

az account set --subscription <mySubscriptionID>

# Create resource group
az group create --name $ResourceGroupName --location $Location 

# Create secure five node Linux cluster. Creates a key vault in a resource group
# and creates a certficate in the key vault. The certificate's subject name must match 
# the domain that you use to access the Service Fabric cluster.  The certificate is downloaded locally.
az sf cluster create --resource-group $ResourceGroupName --location $Location --certificate-output-folder . --certificate-password $Password --certificate-subject-name $Subject --cluster-name $ClusterName --cluster-size 5 --os UbuntuServer1604 --vault-name $VaultName --vault-resource-group $ResourceGroupName --vm-password $VmPassword --vm-user-name $VmUserName
```

> [!Note]
> Le service frontal web est configuré pour écouter le trafic entrant sur le port 80. Par défaut, le port 80 est ouvert sur les machines virtuelles du cluster et sur l’équilibreur de charge Azure.
>

## <a name="configure-your-environment"></a>Configurer votre environnement

Service Fabric propose plusieurs outils que vous pouvez utiliser pour gérer un cluster et ses applications :

- Service Fabric Explorer, un outil reposant sur un navigateur.
- Interface CLI Service Fabric, qui s’exécute sur Azure CLI. 
- Commandes PowerShell.

Dans ce guide de démarrage rapide, vous utilisez l’interface CLI Service Fabric et Service Fabric Explorer (outil basé sur le web). Pour utiliser Service Fabric Explorer, vous devez importer le fichier PFX du certificat dans le navigateur. Par défaut, aucun mot de passe n’est associé au fichier PFX.

Mozilla Firefox est le navigateur par défaut dans Ubuntu 16.04. Pour importer le certificat dans Firefox, cliquez sur le bouton de menu dans le coin supérieur droit de votre navigateur, puis cliquez sur **Options** . Sur la page **Préférences** , recherchez « certificats » via le champ de recherche. Cliquez sur **Afficher les certificats** , sélectionnez l’onglet **Your Certificates** (Vos certificats), cliquez sur **Importer** et suivez les invites pour importer le certificat.

   ![Installer le certificat sur Firefox](./media/service-fabric-quickstart-containers-linux/install-cert-firefox.png)

## <a name="deploy-the-service-fabric-application"></a>Déployer l’application Service Fabric

1. Connectez-vous au cluster Service Fabric dans Azure à l’aide de l’interface CLI. Le point de terminaison est le point de terminaison de gestion de votre cluster. Dans la section précédente, vous avez créé le fichier PEM. 

    ```bash
    sfctl cluster select --endpoint https://containertestcluster.eastus.cloudapp.azure.com:19080 --pem containertestcluster22019013100.pem --no-verify
    ```

2. Utilisez le script d’installation pour copier la définition d’application de vote dans le cluster, inscrivez le type d’application et créez une instance de l’application.  Le fichier de certificat PEM doit se trouver dans le même répertoire que le fichier *install.sh* .

    ```bash
    ./install.sh
    ```

3. Ouvrez un navigateur web et accédez au point de terminaison Service Fabric Explorer pour votre cluster. Le point de terminaison a le format suivant : **https://\<my-azure-service-fabric-cluster-url>:19080/Explorer** , par exemple, `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`. </br>

4. Développez le nœud **Applications** pour vous assurer qu’une entrée s’affiche pour le type d’application de vote et l’instance que vous avez créée.

    ![Service Fabric Explorer][sfx]

5. Pour vous connecter au conteneur en cours d’exécution, ouvrez un navigateur web et accédez à l’URL de votre cluster. Par exemple, `http://containertestcluster.eastus.cloudapp.azure.com:80`. L’application de vote doit s’afficher dans le navigateur.

    ![Page web de l’application de vote][quickstartpic]

> [!NOTE]
> Vous pouvez également déployer des applications Service Fabric avec Docker Compose. Par exemple, la commande suivante peut servir à déployer et installer l’application sur le cluster Service Fabric à l’aide de Docker Compose.
>  ```bash
> sfctl compose create --deployment-name TestApp --file-path ../docker-compose.yml
> ```

## <a name="fail-over-a-container-in-a-cluster"></a>Basculer un conteneur dans un cluster

Service Fabric s’assure que vos instances de conteneur se déplacent automatiquement vers d’autres nœuds du cluster en cas d’échec. Il vous est également possible de drainer manuellement un nœud pour les conteneurs, puis de le déplacer normalement vers d’autres nœuds du cluster. Service Fabric propose plusieurs options pour mettre à l’échelle vos services. Dans les étapes suivantes, vous utilisez Service Fabric Explorer.

Pour basculer le conteneur frontal, effectuez les étapes suivantes :

1. Ouvrez Service Fabric Explorer dans votre cluster. Par exemple, `https://containertestcluster.eastus.cloudapp.azure.com:19080/Explorer`.
2. Cliquez sur le nœud **fabric:/Voting/azurevotefront** dans l’arborescence, puis développez le nœud de la partition (représenté par un GUID). Notez le nom du nœud dans l’arborescence, qui affiche les nœuds en cours d’exécution dans le conteneur. Par exemple, `_nodetype_1`.
3. Développez le nœud **Nœuds** dans l’arborescence. Cliquez sur les points de suspension (...) à côté du nœud qui exécute le conteneur.
4. Choisissez **Redémarrer** pour redémarrer ce nœud et confirmer l’action de redémarrage. Le redémarrage entraîne le basculement du conteneur vers un autre nœud du cluster.

    ![Affichage du nœud dans Service Fabric Explorer][sfxquickstartshownodetype]

## <a name="scale-applications-and-services-in-a-cluster"></a>Mettre à l’échelle les applications et services dans un cluster

Les services Service Fabric peuvent être facilement mis à l’échelle dans un cluster pour s’adapter à la charge sur les services. Pour mettre à l’échelle un service, vous modifiez le nombre d’instances s’exécutant dans le cluster.

Pour mettre à l’échelle le service frontal web, procédez comme suit :

1. Ouvrez Service Fabric Explorer dans votre cluster. Par exemple, `https://containertestcluster.eastus.cloudapp.azure.com:19080`.
2. Cliquez sur les points de suspension (trois points) à côté du nœud **fabric:/Voting/azurevotefront** dans l’arborescence, puis choisissez **Mettre le service à l’échelle** .

    ![Lancement du service de mise à l’échelle Service Fabric Explorer][containersquickstartscale]

    Vous pouvez maintenant choisir de mettre à l’échelle le nombre d’instances du service frontal web.

3. Définissez le nombre sur **2** , puis cliquez sur **Scale Service** (Mettre à l’échelle le service).
4. Cliquez sur le nœud **fabric:/Voting/azurevotefront** dans l’arborescence, puis développez le nœud de la partition (représenté par un GUID).

    ![Fin du service de mise à l’échelle Service Fabric Explorer][containersquickstartscaledone]

    Vous constatez désormais que le service possède deux instances. Les nœuds exécutés par les instances sont visibles dans l’arborescence.

Via cette tâche de gestion simple, vous avez doublé les ressources disponibles pour permettre au service frontal de traiter la charge utilisateur. Il est important de comprendre que vous n’avez pas besoin de plusieurs instances d’un service pour que celui-ci s’exécute de manière fiable. En cas de défaillance d’un service, Service Fabric veille à ce qu’une nouvelle instance du service s’exécute dans le cluster.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Utilisez le script de désinstallation (uninstall.sh) fourni dans le modèle pour supprimer l’instance de l’application du cluster et annuler l’inscription du type d’application. Ce script prend un certain temps pour nettoyer l’instance. Par conséquent, veillez à ne pas exécuter le script d’installation immédiatement après ce script. Vous pouvez utiliser Service Fabric Explorer afin de déterminer quand la suppression de l’instance et l’annulation de l’inscription du type d’application ont eu lieu.

```bash
./uninstall.sh
```

Le plus simple pour supprimer le cluster et toutes les ressources qu’il consomme consiste à supprimer le groupe de ressources.

Connectez-vous à Azure et sélectionnez l’ID d’abonnement avec lequel vous souhaitez supprimer le cluster. Vous pouvez trouver votre ID d’abonnement en vous connectant au portail Azure. Pour supprimer le groupe de ressources et toutes les ressources de cluster, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest).

```azurecli
az login
az account set --subscription <guid>
ResourceGroupName="containertestcluster"
az group delete --name $ResourceGroupName
```

Si vous avez fini d’utiliser votre cluster, vous pouvez supprimer le certificat du magasin de certificats. Par exemple :
- Sur Windows : utilisez le [composant logiciel enfichable MMC Certificats](/dotnet/framework/wcf/feature-details/how-to-view-certificates-with-the-mmc-snap-in). Veillez à sélectionner **My user account** (Mon compte d’utilisateur) lors de l’ajout du composant logiciel enfichable. Accédez à `Certificates - Current User\Personal\Certificates` et supprimez le certificat.
- Sur Mac : utilisez l’application Keychain.
- Sur Ubuntu : suivez les étapes que vous avez utilisées pour afficher les certificats, puis supprimez le certificat.

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez déployé une application de conteneur Linux dans un cluster Service Fabric dans Azure, vous avez basculé l’application et vous l’avez mise à l’échelle dans le cluster. Pour en savoir plus sur l’utilisation de conteneurs de Linux dans Service Fabric, passez au didacticiel sur les applications de conteneur Linux.

> [!div class="nextstepaction"]
> [Créer une application de conteneur Linux](./service-fabric-tutorial-create-container-images.md)

[sfx]: ./media/service-fabric-quickstart-containers-linux/containersquickstartappinstance.png
[quickstartpic]: ./media/service-fabric-quickstart-containers-linux/votingapp.png
[sfxquickstartshownodetype]:  ./media/service-fabric-quickstart-containers-linux/containersquickstartrestart.png
[containersquickstartscale]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscale.png
[containersquickstartscaledone]: ./media/service-fabric-quickstart-containers-linux/containersquickstartscaledone.png

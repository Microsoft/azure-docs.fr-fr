---
title: Configurer un cluster Linux Azure Service Fabric sur Windows
description: Cet article explique comment configurer des clusters Linux Service Fabric exécutés sur des machines de développement Windows. Cette approche est particulièrement utile pour le développement multiplateforme.
ms.topic: conceptual
ms.date: 10/16/2020
ms.openlocfilehash: 7b25a84e76773baea9f17430df1b7ba13aa661aa
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93087075"
---
# <a name="set-up-a-linux-service-fabric-cluster-on-your-windows-developer-machine"></a>Configurer un cluster Linux Service Fabric sur votre machine de développement Windows

Ce document explique comment configurer un cluster Linux Service Fabric local sur une machine de développement Windows. La configuration d'un cluster Linux local est utile pour tester rapidement les applications ciblées pour les clusters Linux mais développées sur une machine Windows.

## <a name="prerequisites"></a>Prérequis
Les clusters Service Fabric basés sur Linux ne s’exécutent pas sur Windows, mais pour activer le prototypage multiplateforme, nous avons fourni un conteneur Docker de cluster à boîtier unique Linux Service Fabric, qui peut être déployé via Docker pour Windows.

Avant de commencer, vous avez besoin des éléments suivants :

* Au moins 4 Go de RAM
* Dernière version de [Docker pour Windows](https://store.docker.com/editions/community/docker-ce-desktop-windows)
* Docker doit s’exécuter en mode Conteneurs Linux

>[!TIP]
> Pour installer Docker sur votre machine Windows, suivez les étapes indiquées dans la [documentation de Docker](https://store.docker.com/editions/community/docker-ce-desktop-windows/plans/docker-ce-desktop-windows-tier?tab=instructions). Après l’installation, [vérifiez votre installation](https://docs.docker.com/docker-for-windows/#check-versions-of-docker-engine-compose-and-machine).
>

## <a name="create-a-local-container-and-setup-service-fabric"></a>Créer un conteneur local et configurer Service Fabric
Pour configurer un conteneur Docker local et y exécuter un cluster Service Fabric, procédez comme suit :


1. Mettez à jour la configuration du démon Docker sur votre ordinateur hôte avec les informations suivantes, puis redémarrez le démon Docker : 

    ```json
    {
      "ipv6": true,
      "fixed-cidr-v6": "2001:db8:1::/64"
    }
    ```
    La méthode de mise à jour recommandée est la suivante : 

    * Accédez à l’icône Docker > Paramètres > Moteur Docker.
    * Ajoutez les nouveaux champs listés ci-dessus.
    * Appliquer et redémarrer : redémarrez ensuite le démon Docker pour que les modifications prennent effet.

2. Démarrez le cluster via PowerShell.<br/>
    <b>Ubuntu 18.04 LTS :</b>
    ```powershell
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u18
    ```

    <b>Ubuntu 16.04 LTS :</b>
    ```powershell
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mcr.microsoft.com/service-fabric/onebox:u16
    ```

    >[!TIP]
    > Par défaut, cela extraira l’image avec la dernière version de Service Fabric. Pour obtenir des révisions particulières, consultez la page [Onebox Service Fabric](https://hub.docker.com/_/microsoft-service-fabric-onebox) sur Docker Hub.



3. Facultatif : Générez votre image Service Fabric étendue.

    Dans le nouveau répertoire, créez un fichier nommé `Dockerfile` pour créer votre image personnalisée :

    >[!NOTE]
    >Vous pouvez adapter l’image ci-dessus avec un Dockerfile pour ajouter des programmes ou des dépendances supplémentaires dans votre conteneur.
    >Par exemple, l’ajout de `RUN apt-get install nodejs -y` permet de prendre en charge les applications `nodejs` comme exécutables invités.
    ```Dockerfile
    FROM mcr.microsoft.com/service-fabric/onebox:u18
    RUN apt-get install nodejs -y
    EXPOSE 19080 19000 80 443
    WORKDIR /home/ClusterDeployer
    CMD ["./ClusterDeployer.sh"]
    ```
    
    >[!TIP]
    > Par défaut, cela extraira l’image avec la dernière version de Service Fabric. Pour des révisions particulières, visitez la page [Docker Hub](https://hub.docker.com/r/microsoft/service-fabric-onebox/).

    Pour créer votre image réutilisable à partir du `Dockerfile`, ouvrez un terminal et `cd` vers le répertoire contenant votre `Dockerfile`, puis exécutez :

    ```powershell 
    docker build -t mysfcluster .
    ```
    
    >[!NOTE]
    >Cette opération prend un certain temps, mais ne doit être effectuée qu’une seule fois.

    Vous pouvez maintenant démarrer rapidement une copie locale de Service Fabric chaque fois que nécessaire en exécutant :

    ```powershell 
    docker run --name sftestcluster -d -v /var/run/docker.sock:/var/run/docker.sock -p 19080:19080 -p 19000:19000 -p 25100-25200:25100-25200 mysfcluster
    ```

    >[!TIP]
    >Donnez un nom à votre instance de conteneur de sorte qu’elle puisse être gérée plus facilement. 
    >
    >Si votre application écoute sur certains ports, ceux-ci doivent être spécifiés à l’aide de balises `-p` supplémentaires. Par exemple, si votre application écoute sur le port 8080, ajoutez la balise `-p` suivante :
    >
    >`docker run -itd -p 19000:19000 -p 19080:19080 -p 8080:8080 --name sfonebox mcr.microsoft.com/service-fabric/onebox:u18`
    >


4. Le démarrage du cluster prend un certain temps. Vous pouvez afficher les journaux d’activité à l’aide de la commande suivante ou accéder au tableau de bord pour afficher l’intégrité des clusters `http://localhost:19080` :

    ```powershell 
    docker logs sftestcluster
    ```

5. Une fois que le cluster a été déployé avec succès comme observé à l’étape 4, vous pouvez accéder à ``http://localhost:19080`` à partir de votre ordinateur Windows pour rechercher le tableau de bord Service Fabric Explorer. À ce stade, vous pouvez vous connecter à ce cluster à l’aide des outils de votre machine de développement Windows et déployer une application ciblée pour les clusters Linux Service Fabric. 

    > [!NOTE]
    > Le plug-in Eclipse n’est actuellement pas pris en charge sous Windows. 

6. Lorsque vous avez terminé, arrêtez puis nettoyez le conteneur avec cette commande :

    ```powershell 
    docker rm -f sftestcluster
    ```

### <a name="known-limitations"></a>Limites connues 
 
 Voici les limitations connues d’un cluster local qui s’exécute dans un conteneur sous Mac : 
 
 * Le service DNS ne fonctionne pas et n’est actuellement pas pris en charge dans le conteneur. [Problème no 132](https://github.com/Microsoft/service-fabric/issues/132)
 * L’exécution d’applications basées sur des conteneurs nécessite l’exécution de Service Fabric sur un hôte Linux. Les applications de conteneur imbriquées ne sont actuellement pas prises en charge.

## <a name="next-steps"></a>Étapes suivantes
* [Create and deploy your first Service Fabric Java application on Linux using Yeoman (Créer et déployer votre première application Java Service Fabric sur Linux à l’aide de Yeoman)](service-fabric-create-your-first-linux-application-with-java.md)
* Prise en main d'[Eclipse](./service-fabric-get-started-eclipse.md)
* Consulter d’autres [exemples Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)
* En savoir plus sur les [options de prise en charge de Service Fabric](service-fabric-support.md)


<!-- Image references -->

[publishdialog]: ./media/service-fabric-manage-multiple-environment-app-configuration/publish-dialog-choose-app-config.png
[app-parameters-solution-explorer]:./media/service-fabric-manage-multiple-environment-app-configuration/app-parameters-in-solution-explorer.png

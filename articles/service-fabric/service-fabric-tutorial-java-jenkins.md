---
title: Configurer Jenkins pour une application Java sur Service Fabric dans Azure
description: Dans ce tutoriel, découvrez comment configurer l’intégration continue à l’aide de Jenkins pour déployer une application Java Service Fabric.
ms.topic: tutorial
ms.date: 08/27/2018
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 4e29104f0586a8e18afb7a640903c57ce988b692
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91531970"
---
# <a name="tutorial-configure-a-jenkins-environment-to-enable-cicd-for-a-java-application-on-service-fabric"></a>Tutoriel : Configurer un environnement Jenkins pour activer l’intégration/le déploiement continus (CI/CD) pour une application Java sur Service Fabric

Ce tutoriel est la cinquième partie de la série. Il vous explique comment utiliser Jenkins pour déployer des mises à niveau pour votre application. Dans ce didacticiel, le plug-in Jenkins Service Fabric est utilisé en association avec un référentiel GitHub hébergeant l’application Voting pour déployer l’application vers un cluster.

Dans ce cinquième volet, vous apprenez à :
> [!div class="checklist"]
> * Déployer un conteneur Jenkins Service Fabric sur votre ordinateur
> * Configurer un environnement Jenkins pour le déploiement sur Service Fabric
> * Mettre à niveau votre application

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Générer une application Reliable Services Service Fabric de Java](service-fabric-tutorial-create-java-app.md)
> * [Déployer et déboguer l’application sur un cluster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Déployer l’application sur un cluster Azure](service-fabric-tutorial-java-deploy-azure.md)
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-java-elk.md)
> * Configurer l’intégration continue/le déploiement continu

## <a name="prerequisites"></a>Prérequis

* Installer Git sur votre ordinateur local à partir de [la page de téléchargements de Git](https://git-scm.com/downloads). Pour plus d’informations sur Git, consultez la [documentation Git](https://git-scm.com/docs).
* Avoir une connaissance pratique de [Jenkins](https://jenkins.io/).
* Créer un compte [GitHub](https://github.com/) et savoir comment utiliser GitHub.
* Installer [Docker](https://www.docker.com/community-edition) sur votre ordinateur.

## <a name="pull-and-deploy-service-fabric-jenkins-container-image"></a>Extraire et déployer l’image du conteneur Jenkins Service Fabric

Vous pouvez configurer Jenkins à l’intérieur ou en dehors d’un cluster Service Fabric. Les instructions suivantes indiquent comment le configurer en dehors d’un cluster à l’aide d’une image Docker fournie. Toutefois, un environnement de build Jenkins préconfiguré peut également être utilisé. L’image conteneur suivante est installée avec le plug-in Service Fabric, et est prête pour une utilisation immédiate avec Service Fabric.

1. Extrayez l’image du conteneur Jenkins de Service Fabric : ``docker pull rapatchi/jenkins:v10``. Cette image est fournie avec le plug-in Jenkins de Service Fabric préinstallé.

1. Exécutez l’image conteneur avec l’emplacement où se trouvent vos certificats sur votre ordinateur local monté.

    ```bash
    docker run -itd -p 8080:8080 -v /service-fabric-java-quickstart/AzureCluster rapatchi/jenkins:v10
    ```

1. Récupérez l’ID de l’instance d’image du conteneur. Vous pouvez répertorier tous les conteneurs Docker avec la commande ``docker ps –a``

1. Récupérer le mot de passe de votre instance Jenkins en exécutant la commande suivante :

    ```sh
    docker exec [first-four-digits-of-container-ID] cat /var/jenkins_home/secrets/initialAdminPassword
    ```

    Si l’ID du conteneur est 2d24a73b5964, utilisez 2d24.
    * Ce mot de passe est nécessaire pour vous connecter au tableau de bord Jenkins à partir du portail: ``http://<HOST-IP>:8080``
    * Après vous être connecté pour la première fois, vous pouvez créer votre propre compte utilisateur ou utiliser le compte administrateur.

1. Configurez GitHub pour utiliser Jenkins, en suivant les étapes présentées dans [Generating a new SSH key and adding it to the SSH agent](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/) (Génération d’une clé SSH et ajout de celle-ci à l’agent SSH). Étant donné que les commandes sont exécutées à partir du conteneur Docker, suivez les instructions pour l’environnement Linux.
   * Utilisez les instructions fournies par GitHub pour générer la clé SSH. Ensuite, ajoutez la clé SSH pour le compte GitHub qui héberge le référentiel.
   * Exécutez les commandes mentionnées dans le lien ci-dessus dans le shell Jenkins Docker (et non sur l’hôte).
   * Pour vous connecter au shell Jenkins depuis votre hôte, utilisez les commandes suivantes :

     ```sh
     docker exec -t -i [first-four-digits-of-container-ID] /bin/bash
     ```

     Vérifiez que le cluster ou la machine hébergeant l’image de conteneur Jenkins possède une adresse IP publique. Le fait d’avoir une adresse IP publique permet à l’instance Jenkins de recevoir des notifications de GitHub.

## <a name="create-and-configure-a-jenkins-job"></a>Créer et configurer un travail Jenkins

1. Tout d’abord, si vous n’avez pas de référentiel pour héberger le projet Voting sur GitHub, créez-en un. Le référentiel est appelé **dev_test** pour le reste de ce tutoriel.

1. Créez un **élément** sur votre tableau de bord Jenkins à l’emplacement ``http://<HOST-IP>:8080``.

1. Entrez un nom d’élément (par exemple, **MyJob**). Sélectionnez **free-style project** (projet libre), puis cliquez sur **OK**.

1. Accédez à la page du travail, puis cliquez sur **Configurer**.

   a. Dans la section générale, cochez la case **GitHub project** (Projet GitHub), puis spécifiez l’URL de votre projet GitHub. Cette URL héberge l’application Java Service Fabric que vous souhaitez intégrer au flux d’intégration continue et de déploiement continu Jenkins (CI/CD) (par exemple, ``https://github.com/testaccount/dev_test``).

   b. Sous la section **Gestion du code source**, sélectionnez **Git**. Spécifiez l’URL du référentiel qui héberge l’application Java Service Fabric que vous souhaitez intégrer au flux d’intégration continue/de déploiement continu Jenkins (par exemple, *`https://github.com/testaccount/dev_test.git`* ). Vous pouvez également spécifier ici la branche à générer (par exemple, **/master**).

1. Configurez votre *GitHub* (qui héberge le référentiel) afin qu’il soit en mesure de communiquer avec Jenkins. Utiliser les étapes suivantes :

   a. Accédez à la page de votre référentiel GitHub. Accédez à **Settings** > **Integrations and Services** (Paramètres -> Intégrations et services).

   b. Sélectionnez **Add Service** (Ajouter un service), tapez **Jenkins** et sélectionnez le **plug-in Jenkins-Github**.

   c. Entrez votre URL du webhook Jenkins (par défaut, il doit être ``http://<PublicIPorFQDN>:8081/github-webhook/``). Cliquez sur **Add/Update service** (Ajouter/Mettre à jour le service).

   d. Un événement de test est envoyé à votre instance Jenkins. Vous devez voir une coche verte à côté du webhook dans GitHub et votre projet est généré.

   ![Configuration de Jenkins Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsconfiguration.png)

1. Dans la section **Build Triggers** (Générer des déclencheurs), sélectionnez l’option de création souhaitée. Pour cet exemple, vous souhaitez déclencher une génération chaque fois qu’un envoi a lieu vers le référentiel. Vous sélectionnez donc **GitHub hook trigger for GITScm polling** (Déclencher un hook GitHub pour l’interrogation GITScm).

1. Sous la **section Build** (Générer), dans la liste déroulante **Add build step** (Ajouter une étape de génération), sélectionnez l’option **Invoke Gradle Script** (Appeler le script Gradle). Dans le widget qui apparaît, ouvrez le menu Avancé, spécifiez le chemin d’accès à **Root build script** (Script de génération racine) pour votre application. Il récupère le Gradle de la génération à partir du chemin spécifié et fonctionne en conséquence.

    ![Action de génération Jenkins de Service Fabric](./media/service-fabric-tutorial-java-jenkins/jenkinsbuildscreenshot.png)

1. Dans la liste déroulante **Post-Build Actions** (Actions post-génération), sélectionnez **Deploy Service Fabric Project** (Déployer le projet Service Fabric). Ici, vous devez fournir des détails sur le cluster où l’application Service Fabric compilée par Jenkins est déployée. Le chemin d’accès au certificat est l’emplacement où le volume a été monté (/tmp/myCerts).

    Vous pouvez également donner des informations supplémentaires pour déployer l’application. Consultez la capture d’écran ci-dessous pour avoir une idée des informations de l’application :

    ![Boîte de dialogue Actions post-build avec des options de déploiement de projet Service Fabric supplémentaires](./media/service-fabric-tutorial-java-jenkins/sfjenkins.png)

    > [!NOTE]
    > Le cluster ici peut être identique à celui qui héberge l’application de conteneur Jenkins dans le cas où vous utilisez Service Fabric pour déployer l’image de conteneur Jenkins.
    >

1. Cliquez sur **Enregistrer**.

## <a name="update-your-existing-application"></a>Mettre à jour votre application existante

1. Mettez à jour le titre du HTML dans le fichier *VotingApplication/VotingWebPkg/Code/wwwroot/index.html* en le remplaçant par **Service Fabric Voting Sample V2**.

    ```html
    <div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
        <div class="container-fluid">
            <div class="row">
                <div class="col-xs-8 col-xs-offset-2 text-center">
                    <h2>Service Fabric Voting Sample V2</h2>
                </div>
            </div>
        </div>
    </div>
    ```

1. Mettez à jour la version **ApplicationTypeVersion** et **ServiceManifestVersion** en les remplaçant par **2.0.0** dans le fichier *Voting/VotingApplication/ApplicationManifest.xml*.

    ```xml
    <?xml version="1.0" encoding="utf-8" standalone="no"?>
    <ApplicationManifest xmlns="http://schemas.microsoft.com/2011/01/fabric" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" ApplicationTypeName="VotingApplicationType" ApplicationTypeVersion="2.0.0">
      <Description>Voting Application</Description>
      <ServiceManifestImport>
        <ServiceManifestRef ServiceManifestName="VotingWebPkg" ServiceManifestVersion="2.0.0"/>
      </ServiceManifestImport>
      <ServiceManifestImport>
            <ServiceManifestRef ServiceManifestName="VotingDataServicePkg" ServiceManifestVersion="1.0.0"/>
        </ServiceManifestImport>
        <DefaultServices>
          <Service Name="VotingWeb">
             <StatelessService InstanceCount="1" ServiceTypeName="VotingWebType">
                <SingletonPartition/>
             </StatelessService>
          </Service>
       <Service Name="VotingDataService">
                <StatefulService MinReplicaSetSize="3" ServiceTypeName="VotingDataServiceType" TargetReplicaSetSize="3">
                    <UniformInt64Partition HighKey="9223372036854775807" LowKey="-9223372036854775808" PartitionCount="1"/>
                </StatefulService>
            </Service>
        </DefaultServices>
    </ApplicationManifest>
    ```

1. Mettez à jour le champ **Version** dans **ServiceManifest** et le champ **Version** dans la balise **CodePackage** du fichier  *Voting/VotingApplication/VotingWebPkg/ServiceManifest.xml* en les remplaçant par **2.0.0**.

    ```xml
    <CodePackage Name="Code" Version="2.0.0">
    <EntryPoint>
        <ExeHost>
        <Program>entryPoint.sh</Program>
        </ExeHost>
    </EntryPoint>
    </CodePackage>
    ```

1. Pour initialiser un travail Jenkins qui effectue une mise à niveau de l’application, envoyez (par push) vos nouvelles modifications à votre référentiel GitHub.

1. Dans Service Fabric Explorer, cliquez sur la liste déroulante **Applications**. Pour afficher l’état de votre mise à niveau, cliquez sur l’onglet **Upgrades in Progress** (Mises à niveau en cours).

    ![Mise à niveau en cours](./media/service-fabric-tutorial-create-java-app/upgradejava.png)

1. Si vous accédez à **http://\<Host-IP>:8080**, l’application Voting avec toutes ses fonctionnalités est maintenant opérationnelle.

    ![Application Voting en local](./media/service-fabric-tutorial-java-jenkins/votingv2.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Déployer un conteneur Jenkins Service Fabric sur votre ordinateur
> * Configurer un environnement Jenkins pour le déploiement sur Service Fabric
> * Mettre à niveau votre application

* Consulter d’autres [exemples Java](https://github.com/Azure-Samples/service-fabric-java-getting-started)

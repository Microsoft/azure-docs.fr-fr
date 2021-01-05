---
title: Déployer une application Java avec Open Liberty/WebSphere Liberty sur un cluster Azure Red Hat OpenShift 4
description: Déployez une application Java avec Open Liberty/WebSphere Liberty sur un cluster Azure Red Hat OpenShift 4.
author: jiangma
ms.author: jiangma
ms.service: container-service
ms.topic: conceptual
ms.date: 10/30/2020
keywords: java, jakartaee, javaee, microprofile, open-liberty, websphere-liberty, aro, openshift, red hat
ms.openlocfilehash: 0c17c911d1eefe646785314a26b6a9b1e964ca67
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493937"
---
# <a name="deploy-a-java-application-with-open-libertywebsphere-liberty-on-an-azure-red-hat-openshift-4-cluster"></a>Déployer une application Java avec Open Liberty/WebSphere Liberty sur un cluster Azure Red Hat OpenShift 4

Ce guide montre comment exécuter votre application Java, Java EE, [Jakarta EE](https://jakarta.ee/) ou [MicroProfile](https://microprofile.io/) sur le runtime Open Liberty/WebSphere Liberty, puis déployer l’application conteneurisée dans un cluster Azure Red Hat OpenShift (ARO) 4 à l’aide de l’opérateur Open Liberty. Cet article vous guide tout au long de la préparation d’une application Liberty, de la création de l’image Docker de l’application et de l’exécution de l’application conteneurisée sur un cluster ARO 4.  Pour plus d’informations sur Open Liberty, consultez [la page de projet Open Liberty](https://openliberty.io/). Pour plus d’informations sur IBM WebSphere Liberty, consultez [la page du produit WebSphere Liberty](https://www.ibm.com/cloud/websphere-liberty).

[!INCLUDE [aro-support](includes/aro-support.md)]

## <a name="prerequisites"></a>Prérequis

Remplissez les prérequis suivants pour suivre ce guide.

> [!NOTE]
> Azure Red Hat OpenShift requiert a minimum de 40 cœurs pour créer et exécuter un cluster OpenShift. Le quota de ressources Azure par défaut pour un nouvel abonnement Azure ne répond pas à cette exigence. Pour demander une augmentation de votre limite de ressources, consultez [Quota standard : augmenter les limites par série de machines virtuelles](../azure-portal/supportability/per-vm-quota-requests.md). Notez que l’abonnement d’essai gratuit n’est pas éligible à une augmentation de quota. [Passez à un abonnement Paiement à l’utilisation](../cost-management-billing/manage/upgrade-azure-subscription.md) avant de demander une augmentation de quota.

1. Préparez un ordinateur local avec un système d’exploitation de type UNIX installé (par exemple, Ubuntu, macOS).
1. Installez une implémentation de Java SE (par exemple, [AdoptOpenJDK OpenJDK 8 LTS/OpenJ9](https://adoptopenjdk.net/?variant=openjdk8&jvmVariant=openj9)).
1. Installez [Maven](https://maven.apache.org/download.cgi) 3.5.0 ou une version ultérieure.
1. Installez [Docker](https://docs.docker.com/get-docker/) pour votre système d’exploitation.
1. Installez [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) 2.0.75 ou une version ultérieure.
1. Vérifiez et installez [`envsubst`](https://command-not-found.com/envsubst) s’il n’est pas préinstallé sur votre système d’exploitation.
1. Clonez le code de cet exemple sur votre système local. L’exemple se trouve sur [GitHub](https://github.com/Azure-Samples/open-liberty-on-aro).
1. Suivez les instructions de la section [Créer un cluster Azure Red Hat OpenShift 4](./tutorial-create-cluster.md).

   Bien que l’étape « Récupération d’un secret d’extraction Red Hat » soit étiquetée comme étant facultative, **elle est requise pour cet article**.  Le secret d’extraction permet à votre cluster Azure Red Hat OpenShift de trouver l’opérateur Open Liberty.

   Si vous envisagez d’exécuter des applications gourmandes en mémoire sur le cluster, spécifiez la taille de machine virtuelle appropriée pour les nœuds Worker à l’aide du paramètre `--worker-vm-size`. Par exemple, `Standard_E4s_v3` est la taille de machine virtuelle minimale pour installer l’opérateur Elasticsearch sur un cluster. Pour plus d'informations, consultez les pages suivantes :

   * [Azure CLI pour créer un cluster](/cli/azure/aro?preserve-view=true&view=azure-cli-latest#az-aro-create)
   * [Tailles de machines virtuelles prises en charge pour la mémoire optimisée](./support-policies-v4.md#memory-optimized)
   * [Conditions préalables à l’installation de l’opérateur Elasticsearch](https://docs.openshift.com/container-platform/4.3/logging/cluster-logging-deploying.html#cluster-logging-deploy-eo-cli_cluster-logging-deploying)

1. Connectez-vous au cluster en suivant les étapes décrites dans [Se connecter à un cluster Azure Red Hat OpenShift 4](./tutorial-connect-cluster.md).
   * Veillez à suivre les étapes décrites dans la section « Installer l’interface CLI OpenShift », car nous allons utiliser la commande `oc` plus loin dans cet article.
   * Notez l’URL de la console de cluster qui ressemble à `https://console-openshift-console.apps.<random>.<region>.aroapp.io/`.
   * Notez les informations d’identification de `kubeadmin`.

1. Vérifiez que vous pouvez vous connecter à l’interface CLI OpenShift avec le jeton pour l’utilisateur `kubeadmin`.

### <a name="enable-the-built-in-container-registry-for-openshift"></a>Activer le registre de conteneurs intégré pour OpenShift

Les étapes de ce tutoriel créent une image Docker qui doit être envoyée (push) à un registre de conteneurs accessible à OpenShift. L’option la plus simple consiste à utiliser le registre intégré fourni par OpenShift. Pour activer le registre de conteneurs intégré, suivez les étapes décrites dans [Configurer le registre de conteneurs intégré pour Azure Red Hat OpenShift 4](built-in-container-registry.md). Trois éléments de ces étapes sont utilisés dans cet article :

* Le nom d’utilisateur et le mot de passe de l’utilisateur Azure AD pour la connexion à la console Web OpenShift.
* La sortie de `oc whoami` après avoir effectué les étapes de connexion à l’interface CLI OpenShift.  Cette valeur est appelée **aad-user** pour discussion.
* L’URL du registre de conteneurs.

Notez ces éléments à mesure que vous effectuez les étapes pour activer le registre de conteneurs intégré.

### <a name="create-an-openshift-namespace-for-the-java-app"></a>Créer un espace de noms OpenShift pour l’application Java

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification de `kubeadmin`.
2. Accédez à **Administration** > **Espaces de noms** > **Créer un espace de noms**.
3. Renseignez `open-liberty-demo` pour le **nom** et sélectionnez **Créer**, comme indiqué ci-dessous.

   ![créer un espace de noms](./media/howto-deploy-java-liberty-app/create-namespace.png)

### <a name="create-an-administrator-for-the-demo-project"></a>Créer un administrateur pour le projet de démonstration

Outre la gestion des images, **aad-user** disposera également des autorisations d’administration pour la gestion des ressources dans le projet de démonstration du cluster ARO 4.  Connectez-vous à l’interface CLI OpenShift et accordez à **aad-user** les privilèges nécessaires en procédant comme suit.

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification de `kubeadmin`.
1. Dans l’angle supérieur droit de la console web, développez le menu contextuel de l’utilisateur connecté, puis sélectionnez **Copier la commande de connexion**.
1. Connectez-vous à une nouvelle fenêtre d’onglets avec le même utilisateur si nécessaire.
1. Sélectionnez **Afficher le jeton**.
1. Copiez la valeur indiquée sous **Connectez-vous avec ce jeton** dans le presse-papiers, puis exécutez le jeton dans un interpréteur de commandes, comme illustré ici.
1. Exécutez les commandes suivantes pour accorder le rôle `admin` à **aad-user** dans l’espace de noms `open-liberty-demo`.

   ```bash
   # Switch to project "open-liberty-demo"
   oc project open-liberty-demo
   Now using project "open-liberty-demo" on server "https://api.x8xl3f4y.eastus.aroapp.io:6443".
   # Note: replace "<aad-user>" with the one noted by executing the steps in
   # Configure built-in container registry for Azure Red Hat OpenShift 4
   oc adm policy add-role-to-user admin <aad-user>
   clusterrole.rbac.authorization.k8s.io/admin added: "kaaIjx75vFWovvKF7c02M0ya5qzwcSJ074RZBfXUc34"
   ```

### <a name="install-the-open-liberty-openshift-operator"></a>Installer l’opérateur Open Liberty OpenShift

Après avoir créé le cluster et vous y être connecté, installez l’opérateur Open Liberty.  La page de démarrage principale de l’opérateur Open Liberty se trouve sur [GitHub](https://github.com/OpenLiberty/open-liberty-operator).

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification de `kubeadmin`.
2. Accédez à **Opérateurs** > **OperatorHub** et recherchez **Opérateur Open Liberty**.
3. Sélectionnez **Ouvrir l’opérateur Liberty** dans les résultats de la recherche.
4. Sélectionnez **Installer**.
5. Dans le menu contextuel **Créer un abonnement opérateur**, cochez **Tous les espaces de noms sur le cluster (par défaut)** pour **Mode d’installation**, **bêta** pour **Canal de mise à jour** et **Automatique** pour **Stratégie d’approbation** :

   ![créer un abonnement opérateur pour l’opérateur Open Liberty](./media/howto-deploy-java-liberty-app/install-operator.png)
6. Sélectionnez **S’abonner** et attendez une minute ou deux jusqu’à ce que l’opérateur Open Liberty s’affiche.
7. Observez l’opérateur Open Liberty avec l’état « Opération réussie ».  Si ce n’est pas le cas, diagnostiquez et résolvez le problème avant de continuer.
   :::image type="content" source="media/howto-deploy-java-liberty-app/open-liberty-operator-installed.png" alt-text="Opérateurs installés montrant qu’Open Liberty est installé.":::

## <a name="prepare-the-liberty-application"></a>Préparer l’application Liberty

Nous allons utiliser une application Java EE 8 comme exemple dans ce guide. Open Liberty est un serveur compatible avec le [profil complet Java EE 8](https://javaee.github.io/javaee-spec/javadocs/), de sorte qu’il peut facilement exécuter l’application.  Open Liberty est également [compatible avec le profil complet Jakarta EE 8](https://jakarta.ee/specifications/platform/8/apidocs/).

### <a name="run-the-application-on-open-liberty"></a>Exécuter l’application sur Open Liberty

Pour exécuter l’application sur Open Liberty, vous devez créer un fichier config de serveur Open Liberty pour que le [plug-in Liberty Maven](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin) puisse empaqueter l’application pour le déploiement. Le plug-in Liberty Maven n’est pas requis pour déployer l’application sur OpenShift.  Toutefois, nous l’utiliserons dans cet exemple avec le mode développeur (dev) d’Open Liberty.  Le mode développeur vous permet d’exécuter facilement l’application localement. Effectuez les étapes suivantes sur votre ordinateur local.

1. Copiez `2-simple/src/main/liberty/config/server.xml` dans `1-start/src/main/liberty/config`, en remplaçant le fichier de longueur zéro existant. Ce fichier `server.xml` configure le serveur Open Liberty avec les fonctionnalités Java EE.
1. Copiez `2-simple/pom.xml` dans `1-start/pom.xml`.  Cette étape ajoute le `liberty-maven-plugin` au fichier POM.
1. Changez le répertoire en `1-start` de votre clone local.
1. Exécutez `mvn clean package` dans une console pour générer un package WAR `javaee-cafe.war` dans le répertoire `./target`.
1. Exécutez `mvn liberty:dev` pour démarrer Open Liberty en mode dev.
1. Attendez que le serveur démarre. La sortie de la console doit se terminer par le message suivant :

   ```Text
   [INFO] CWWKM2015I: Match number: 1 is [6/10/20 10:26:09:517 CST] 00000022 com.ibm.ws.kernel.feature.internal.FeatureManager            A CWWKF0011I: The defaultServer server is ready to run a smarter planet. The defaultServer server started in 6.447 seconds..
   [INFO] Press the Enter key to run tests on demand. To stop the server and quit dev mode, use Ctrl-C or type 'q' and press the Enter key.
   [INFO] Source compilation was successful.
   ```

1. Ouvrez `http://localhost:9080/` dans votre navigateur pour accéder à la page d’accueil de l’application. L’application sera semblable à l’image suivante :

   ![Interface utilisateur web de JavaEE Cafe](./media/howto-deploy-java-liberty-app/javaee-cafe-web-ui.png)
1. Appuyez sur **Contrôle+C** pour arrêter l’application et le serveur Open Liberty.

Le répertoire `2-simple` de votre clone local montre le projet Maven avec les modifications ci-dessus déjà appliquées.

## <a name="prepare-the-application-image"></a>Préparer l’image de l’application

Pour déployer et exécuter votre application Liberty sur un cluster ARO 4, conteneurisez votre application en tant qu’image Docker à l’aide d’[images conteneur Open Liberty](https://github.com/OpenLiberty/ci.docker) ou d’[images conteneur WebSphere Liberty](https://github.com/WASdev/ci.docker).

### <a name="build-application-image"></a>Créer une image d’application

Procédez comme suit pour générer l’image de l’application :

1. Changez le répertoire en `2-simple` de votre clone local.
2. Exécutez `mvn clean package` pour empaqueter l’application.
3. Exécutez les commandes suivantes pour générer l’image de l’application.
   * Générer avec une image de base Open Liberty :

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary Open Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull .
     ```

   * Générer avec une image de base WebSphere Liberty :

     ```bash
     # Build and tag application image. This will cause Docker to pull the necessary WebSphere Liberty base images.
     docker build -t javaee-cafe-simple:1.0.0 --pull --file=Dockerfile-wlp .
     ```

### <a name="run-the-application-locally-with-docker"></a>Exécuter l’application localement avec Docker

Avant de déployer l’application conteneurisée sur un cluster distant, exécutez-la avec votre Docker local pour vérifier si elle fonctionne :

1. Exécutez `docker run -it --rm -p 9080:9080 javaee-cafe-simple:1.0.0` dans votre console.
2. Attendez que le serveur Liberty démarre et que l’application soit déployée correctement.
3. Ouvrez `http://localhost:9080/` dans votre navigateur pour accéder à la page d’accueil de l’application.
4. Appuyez sur **Contrôle+C** pour arrêter l’application et le serveur Liberty.

### <a name="push-the-image-to-the-container-image-registry"></a>Envoyer (push) l’image dans le registre d’images conteneur

Lorsque vous êtes satisfait de l’état de l’application, envoyez-la dans le registre d’images conteneur intégré en suivant les instructions ci-dessous.

#### <a name="log-in-to-the-openshift-cli-as-the-azure-ad-user"></a>Se connecter à l’interface CLI OpenShift en tant qu’utilisateur Azure AD

1. Connectez-vous à la console web OpenShift à partir de votre navigateur en utilisant les informations d’identification d’un utilisateur Azure AD.

   1. Utilisez une fonctionnalité de fenêtre de navigateur InPrivate, Incognito ou autre fonctionnalité équivalente pour vous connecter à la console.
   1. Sélectionnez **openid**

   > [!NOTE]
   > Notez le nom d’utilisateur et le mot de passe que vous utilisez pour vous connecter ici. Ces nom d’utilisateur et mot de passe fonctionneront pour l’administrateur en lien avec d’autres actions décrites dans cet article et dans d’autres.
1. Connectez-vous avec l’interface CLI OpenShift en procédant comme suit.  Pour la discussion, ce processus est appelé `oc login`.
   1. Dans l’angle supérieur droit de la console web, développez le menu contextuel de l’utilisateur connecté, puis sélectionnez **Copier la commande de connexion**.
   1. Connectez-vous à une nouvelle fenêtre d’onglets avec le même utilisateur si nécessaire.
   1. Sélectionnez **Afficher le jeton**.
   1. Copiez la valeur indiquée sous **Connectez-vous avec ce jeton** dans le presse-papiers, puis exécutez le jeton dans un interpréteur de commandes, comme illustré ici.

       ```bash
       oc login --token=XOdASlzeT7BHT0JZW6Fd4dl5EwHpeBlN27TAdWHseob --server=https://api.aqlm62xm.rnfghf.aroapp.io:6443
       Logged into "https://api.aqlm62xm.rnfghf.aroapp.io:6443" as "kube:admin" using the token provided.

       You have access to 57 projects, the list has been suppressed. You can list all projects with 'oc projects'

       Using project "default".
       ```

#### <a name="push-the-container-image-to-the-container-registry-for-openshift"></a>Envoyer (push) l’image conteneur dans le registre de conteneurs pour OpenShift

Exécutez ces commandes pour envoyer (push) l’image dans le registre de conteneurs pour OpenShift.

```bash
# Note: replace "<Container_Registry_URL>" with the fully qualified name of the registry
Container_Registry_URL=<Container_Registry_URL>

# Create a new tag with registry info that refers to source image
docker tag javaee-cafe-simple:1.0.0 ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0

# Sign in to the built-in container image registry
docker login -u $(oc whoami) -p $(oc whoami -t) ${Container_Registry_URL}
```

Une sortie réussie ressemble à ce qui suit.

```bash
WARNING! Using --password via the CLI is insecure. Use --password-stdin.
Login Succeeded
```

Envoyez (push) une image dans le registre d’images conteneur intégré à l’aide de la commande suivante.

```bash

docker push ${Container_Registry_URL}/open-liberty-demo/javaee-cafe-simple:1.0.0
```

## <a name="deploy-application-on-the-aro-4-cluster"></a>Déployer une application sur le cluster ARO 4

Vous pouvez maintenant déployer l’exemple d’application Liberty sur le cluster Azure Red Hat OpenShift 4 que vous avez créé précédemment dans la section Prérequis.

### <a name="deploy-the-application-from-the-web-console"></a>Déployer l’application à partir de la console Web

Comme nous utilisons l’opérateur Open Liberty pour gérer les applications Liberty, nous devons créer une instance de sa *définition de ressource personnalisée*, de type « OpenLibertyApplication ». L’opérateur prendra alors en charge tous les aspects de la gestion des ressources OpenShift requises pour le déploiement.

1. Connectez-vous à la console Web OpenShift à partir de votre navigateur en utilisant les informations d’identification de l’utilisateur Azure AD.
1. Développez **Accueil**, sélectionnez **Projets** > **open-liberty-demo**.
1. Accédez à **Opérateurs** > **Opérateurs installés**.
1. Au milieu de la page, sélectionnez **Opérateur Open Liberty**.
1. Au milieu de la page, sélectionnez **Application Open Liberty**.  La navigation dans les éléments de l’interface utilisateur reflète la hiérarchie réelle de confinement des technologies utilisées.
   <!-- Diagram source https://github.com/Azure-Samples/open-liberty-on-aro/blob/master/diagrams/aro-java-containment.vsdx -->
   ![Confinement Java ARO](./media/howto-deploy-java-liberty-app/aro-java-containment.png)
1. Sélectionnez **Créer OpenLibertyApplication**.
1. Remplacez le YAML généré par le vôtre, accessible sous `<path-to-repo>/2-simple/openlibertyapplication.yaml`.
1. Sélectionnez **Create** (Créer). Vous serez redirigé vers la liste des OpenLibertyApplications.
1. Sélectionnez **javaee-cafe-simple**.
1. Au milieu de la page, sélectionnez **Ressources**.
1. Dans le tableau, sélectionnez le lien pour **javaee-cafe-simple** avec le **type** **Itinéraire**.
1. Sur la page qui s’ouvre, sélectionnez le lien sous **Emplacement**.

Vous verrez la page d’accueil de l’application s’ouvrir dans le navigateur.

### <a name="delete-the-application-from-the-web-console"></a>Supprimer l’application à partir de la console Web

Lorsque vous en avez fini avec l’application, procédez comme suit pour supprimer l’application d’Open Shift.

1. Dans le volet de navigation gauche, développez l’entrée pour **Opérateurs**.
1. Sélectionnez **Opérateurs installés**.
1. Sélectionnez **Opérateur Open Liberty**.
1. Au milieu de la page, sélectionnez **Application Open Liberty**.
1. Sélectionnez le bouton de sélection (trois points verticaux), puis sélectionnez **Supprimer l’application OpenLiberty**.

### <a name="deploy-the-application-from-cli"></a>Déployer l’application à partir de l’interface CLI

Au lieu d’utiliser l’interface graphique utilisateur de la console Web, vous pouvez déployer l’application à partir de l’interface CLI. Si vous ne l’avez pas déjà fait, téléchargez et installez l’outil de ligne de commande `oc` en suivant la documentation de Red Hat [Prise en main avec l’interface CLI](https://docs.openshift.com/container-platform/4.2/cli_reference/openshift_cli/getting-started-cli.html).

1. Connectez-vous à la console Web OpenShift à partir de votre navigateur en utilisant les informations d’identification de l’utilisateur Azure AD.
2. Connectez-vous à l’interface CLI OpenShift avec le jeton de l’utilisateur Azure AD.
3. Changez le répertoire en `2-simple` de votre clone local, puis exécutez les commandes suivantes pour déployer votre application Liberty sur le cluster ARO 4.  La sortie de commande est également affichée inlined.

   ```bash
   # Switch to namespace "open-liberty-demo" where resources of demo app will belong to
   oc project open-liberty-demo

   Now using (or already on) project "open-liberty-demo" on server "https://api.aqlm62xm.rnfghf.aroapp.io:6443".

   # Create OpenLibertyApplication "javaee-cafe-simple"
   oc create -f openlibertyapplication.yaml

   openlibertyapplication.openliberty.io/javaee-cafe-simple created

   # Check if OpenLibertyApplication instance is created
   oc get openlibertyapplication javaee-cafe-simple

   NAME                 IMAGE                      EXPOSED   RECONCILED   AGE
   javaee-cafe-simple   javaee-cafe-simple:1.0.0   true      True         36s

   # Check if deployment created by Operator is ready
   oc get deployment javaee-cafe-simple

   NAME                 READY   UP-TO-DATE   AVAILABLE   AGE
   javaee-cafe-simple   1/1     1            0           102s
   ```

4. Avant de continuer, vérifiez que `1/1` se trouve sous la colonne `READY`.  Si ce n’est pas le cas, diagnostiquez et résolvez le problème avant de continuer.
5. Découvrez l’hôte de l’itinéraire vers l’application à l’aide de la commande `oc get route`, comme illustré ici.

   ```bash
   # Get host of the route
   HOST=$(oc get route javaee-cafe-simple --template='{{ .spec.host }}')
   echo "Route Host: $HOST"

   Route Host: javaee-cafe-simple-open-liberty-demo.apps.aqlm62xm.rnfghf.aroapp.io
   ```

   Une fois que l’application Liberty est opérationnelle, ouvrez la sortie de **Route Host** dans votre navigateur pour accéder à la page d’accueil de l’application.

### <a name="delete-the-application-from-cli"></a>Supprimer l’application à partir de l’interface CLI

Supprimez l’application de l’interface CLI en exécutant cette commande.

```bash
oc delete -f openlibertyapplication.yaml
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer le cluster ARO, suivez les étapes décrites dans [Tutoriel : Supprimer un cluster Azure Red Hat OpenShift 4](./tutorial-delete-cluster.md).

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide, vous avez appris à :
> [!div class="checklist"]
>
> * Préparer l’application Liberty
> * Générer l’image de l’application
> * Exécuter l’application conteneurisée sur un cluster ARO 4 à l’aide de l’interface graphique utilisateur et de l’interface CLI

Vous pouvez en savoir plus sur les références utilisées dans ce guide :

* [Open Liberty](https://openliberty.io/)
* [Azure Red Hat OpenShift](https://azure.microsoft.com/services/openshift/)
* [Opérateur Open Liberty](https://github.com/OpenLiberty/open-liberty-operator)
* [Configuration du serveur Open Liberty](https://openliberty.io/docs/ref/config/)
* [Plug-in Liberty Maven](https://github.com/OpenLiberty/ci.maven#liberty-maven-plugin)
* [Images conteneur Open Liberty](https://github.com/OpenLiberty/ci.docker)
* [Images conteneur WebSphere Liberty](https://github.com/WASdev/ci.docker)
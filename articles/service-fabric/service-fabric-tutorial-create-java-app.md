---
title: 'Tutoriel : Créer une application Java sur Azure Service Fabric'
description: Dans ce tutoriel, vous allez découvrir comment créer une application Reliable Services Java avec un service frontal ainsi qu’un service principal Reliable Services avec état, et déployer l’application sur un cluster.
ms.topic: tutorial
ms.date: 09/01/2018
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019, devx-track-java, devx-track-js
ms.openlocfilehash: d1f66177b1939d612a5ee80c99109c397b56cb37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91532191"
---
# <a name="tutorial-create-an-application-with-a-java-api-front-end-service-and-a-stateful-back-end-service-on-azure-service-fabric"></a>Tutoriel : Créer une application avec un service front-end d’API Java et un service back-end avec état sur Azure Service Fabric

Ce tutoriel est la première partie d’une série d’étapes. Quand vous avez terminé, vous disposez d’une application Voting avec un front-end Java qui enregistre les résultats de vote dans un service back-end avec état sur Azure Service Fabric. Cette série de didacticiels nécessite une machine de développeur Mac OSX ou Linux. Si vous ne souhaitez pas créer l’application de vote manuellement, vous pouvez [télécharger le code source pour obtenir l’application terminée](https://github.com/Azure-Samples/service-fabric-java-quickstart) et passer directement au [Guide de l’exemple d’application de vote](service-fabric-tutorial-create-java-app.md#walk-through-the-voting-sample-application). Envisagez également de suivre [Démarrage rapide : Reliable Services en Java](service-fabric-quickstart-java-reliable-services.md).

![Exemple de vote Service Fabric](./media/service-fabric-tutorial-create-java-app/service-fabric-java-voting-app-sample.png)

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * Générer une application Reliable Services Service Fabric de Java
> * [Déployer et déboguer l’application sur un cluster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Déployer l’application sur un cluster Azure](service-fabric-tutorial-java-deploy-azure.md)
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-java-elk.md)
> * [Configurer CI/CD](service-fabric-tutorial-java-jenkins.md)


Dans ce premier volet, vous apprenez à :

> [!div class="checklist"]
> * Créer un service fiable avec état Java
> * Créer un service d’application web sans état Java
> * Utiliser l’accès distant au service pour communiquer avec le service avec état
> * Déployer une application sur un cluster Service Fabric local

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* Configurez votre environnement de développement pour [Mac](service-fabric-get-started-mac.md) ou [Linux](service-fabric-get-started-linux.md). Suivez les instructions pour installer le plug-in Eclipse, Gradle, le Kit de développement logiciel (SDK) Service Fabric et l’interface CLI Service Fabric (sfctl).

## <a name="create-the-front-end-java-stateless-service"></a>Créer le service frontal sans état Java

Commencez par créer le service web frontal de l’application Voting. Une interface utilisateur web basée sur AngularJS envoie des demandes au service sans état Java, qui exécute un serveur HTTP léger. Ce service traite chaque demande et envoie un appel de procédure distante au service avec état pour stocker les votes. 

1. Ouvrez Eclipse.

2. Créez un projet via **Fichier** > **Nouveau** > **Autre** > **Service Fabric** > **Projet Service Fabric**.

    ![Nouveau projet Service Fabric dans Eclipse](./media/service-fabric-tutorial-create-java-app/service-fabric-project-wizard.png)

3. Dans la boîte de dialogue de l’**Assistant de projet Service Fabric**, nommez le projet **Voting** et sélectionnez **Suivant**.

    ![Choix du service sans état Java dans la boîte de dialogue Nouveau service](./media/service-fabric-tutorial-create-java-app/name-service-fabric-project-wizard.png) 

4. Dans la page **Ajouter un service**, sélectionnez **Service sans état** et nommez votre service **VotingWeb**. Sélectionnez **Terminer** pour créer le projet.

    ![Créer un service sans état pour votre projet Service Fabric]( ./media/service-fabric-tutorial-create-java-app/add-service-fabric-votingweb-service.png)

    Eclipse crée une application et un projet de service, et les affiche dans l’Explorateur de package.

    ![Explorateur de package Eclipse après la création d’une application]( ./media/service-fabric-tutorial-create-java-app/eclipse-package-explorer.png)

Ce tableau contient une brève description de chaque élément de l’Explorateur de package de la capture d’écran précédente. 

| **Élément de l’Explorateur de package** | **Description** |
| --- | --- |
| PublishProfiles | Contient les fichiers JSON décrivant les détails du profil des clusters locaux et Azure Service Fabric. Le contenu de ces fichiers est utilisé par le plug-in lors du déploiement de l’application. |
| Scripts | Contient des scripts d’assistance qui peuvent être utilisés dans la ligne de commande pour gérer rapidement votre application avec un cluster. |
| VotingApplication | Contient l’application Service Fabric transmise au cluster Service Fabric. |
| VotingWeb | Contient les fichiers sources du service sans état frontal ainsi que le fichier de build Gradle associé. |
| build.gradle | Fichier Gradle utilisé pour gérer le projet. |
| settings.gradle | Contient le nom des projets Gradle dans ce dossier. |

### <a name="add-html-and-javascript-to-the-votingweb-service"></a>Ajouter du code HTML et Javascript au service VotingWeb

Pour ajouter une interface utilisateur qui peut être rendue par le service sans état, ajoutez un fichier HTML. Ce fichier HTML est ensuite rendu par le serveur HTTP léger incorporé dans le service Java sans état.

1. Développez le répertoire *VotingApplication* pour atteindre le répertoire *VotingApplication/VotingWebPkg/Code*.

2. Cliquez avec le bouton droit sur le répertoire *Code* et sélectionnez **Nouveau** > **Dossier**.

3. Nommez le dossier *wwwroot* et sélectionnez **Terminer**.

    ![Créer le dossier wwwroot dans Eclipse](./media/service-fabric-tutorial-create-java-app/create-wwwroot-folder.png)

4. Ajoutez un fichier dans le dossier **wwwroot** appelé **index.html** et collez-y le contenu suivant.

```html
<!DOCTYPE html>
<html>
<script src="https://ajax.googleapis.com/ajax/libs/angularjs/1.6.4/angular.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/angular-ui-bootstrap/0.13.4/ui-bootstrap-tpls.min.js"></script>
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/3.3.7/css/bootstrap.min.css">
<body>

<script>
var app = angular.module('VotingApp', ['ui.bootstrap']);
app.controller("VotingAppController", ['$rootScope', '$scope', '$http', '$timeout', function ($rootScope, $scope, $http, $timeout) {
    $scope.votes = [];
    
    $scope.refresh = function () {
        $http.get('getStatelessList')
            .then(function successCallback(response) {
        $scope.votes = Object.assign(
            {},
            ...Object.keys(response.data) .
            map(key => ({[decodeURI(key)]: response.data[key]}))
        )
        },
        function errorCallback(response) {
            alert(response);
        });
    };

    $scope.remove = function (item) {
       $http.get("removeItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };

    $scope.add = function (item) {
        if (!item) {return;}
        $http.get("addItem", {params: { item: encodeURI(item) }})
            .then(function successCallback(response) {
                $scope.refresh();
            },
            function errorCallback(response) {
                alert(response);
            });
    };
}]);
</script>

<div ng-app="VotingApp" ng-controller="VotingAppController" ng-init="refresh()">
    <div class="container-fluid">
        <div class="row">
            <div class="col-xs-8 col-xs-offset-2 text-center">
                <h2>Service Fabric Voting Sample</h2>
            </div>
        </div>

        <div class="row">
            <div class="col-xs-offset-2">
                <form style="width:50% ! important;" class="center-block">
                    <div class="col-xs-6 form-group">
                        <input id="txtAdd" type="text" class="form-control" placeholder="Add voting option" ng-model="item" />
                    </div>
                    <button id="btnAdd" class="btn btn-default" ng-click="add(item)">
                        <span class="glyphicon glyphicon-plus" aria-hidden="true"></span>
                        Add
                    </button>
                </form>
            </div>
        </div>

        <hr />

        <div class="row">
            <div class="col-xs-8 col-xs-offset-2">
                <div class="row">
                    <div class="col-xs-4">
                        Click to vote
                    </div>
                </div>
                <div class="row top-buffer" ng-repeat="(key, value)  in votes">
                    <div class="col-xs-8">
                        <button class="btn btn-success text-left btn-block" ng-click="add(key)">
                            <span class="pull-left">
                                {{key}}
                            </span>
                            <span class="badge pull-right">
                                {{value}} Votes
                            </span>
                        </button>
                    </div>
                    <div class="col-xs-4">
                        <button class="btn btn-danger pull-right btn-block" ng-click="remove(key)">
                            <span class="glyphicon glyphicon-remove" aria-hidden="true"></span>
                            Remove
                        </button>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>

</body>
</html>
```

### <a name="update-the-votingwebjava-file"></a>Mettre à jour le fichier VotingWeb.java

Dans le sous-projet **VotingWeb**, ouvrez le fichier *VotingWeb/src/statelessservice/VotingWeb.java*. Le service **VotingWeb** est la passerelle vers le service sans état et est responsable de la configuration de l’écouteur de communication pour l’API frontale.

Remplacez la méthode **createServiceInstanceListeners** dans le fichier par le code suivant et enregistrez vos modifications.

```java
@Override
protected List<ServiceInstanceListener> createServiceInstanceListeners() {

    EndpointResourceDescription endpoint = this.getServiceContext().getCodePackageActivationContext().getEndpoint(webEndpointName);
    int port = endpoint.getPort();

    List<ServiceInstanceListener> listeners = new ArrayList<ServiceInstanceListener>();
    listeners.add(new ServiceInstanceListener((context) -> new HttpCommunicationListener(context, port)));
    return listeners;
}
```

### <a name="add-the-httpcommunicationlistenerjava-file"></a>Ajouter le fichier HTTPCommunicationListener.java

L’écouteur de communication HTTP agit comme un contrôleur qui configure le serveur HTTP et expose les API de définition des actions de vote. Cliquez avec le bouton droit sur le package *statelessservice* dans le dossier *VotingWeb/src/statelessservice*, puis sélectionnez **Nouveau** > **Fichier**.  Nommez le fichier *HttpCommunicationListener.java* et sélectionnez **Terminer**.

Remplacez le contenu du fichier par la commande suivante, puis enregistrez vos modifications.  Plus tard, dans Mettre à jour le fichier HttpCommunicationListener.java, ce fichier est modifié pour rendre, lire et écrire des données de vote à partir du service back-end.  Pour l’instant, l’écouteur retourne simplement le code HTML statique de l’application Voting.

```java
// ------------------------------------------------------------
//  Copyright (c) Microsoft Corporation.  All rights reserved.
//  Licensed under the MIT License (MIT). See License.txt in the repo root for license information.
// ------------------------------------------------------------

package statelessservice;

import com.google.gson.Gson;
import com.sun.net.httpserver.HttpExchange;
import com.sun.net.httpserver.HttpHandler;
import com.sun.net.httpserver.Headers;

import java.io.File;
import java.io.OutputStream;
import java.io.FileInputStream;

import java.net.InetSocketAddress;
import java.net.URI;

import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.CompletableFuture;
import java.util.logging.Level;
import java.util.logging.Logger;

import microsoft.servicefabric.services.communication.runtime.CommunicationListener;
import microsoft.servicefabric.services.runtime.StatelessServiceContext;
import microsoft.servicefabric.services.client.ServicePartitionKey;
import microsoft.servicefabric.services.remoting.client.ServiceProxyBase;
import microsoft.servicefabric.services.communication.client.TargetReplicaSelector;
import system.fabric.CancellationToken;

public class HttpCommunicationListener implements CommunicationListener {

    private static final Logger logger = Logger.getLogger(HttpCommunicationListener.class.getName());

    private static final String HEADER_CONTENT_TYPE = "Content-Type";
    private static final int STATUS_OK = 200;
    private static final int STATUS_NOT_FOUND = 404;
    private static final int STATUS_ERROR = 500;
    private static final String RESPONSE_NOT_FOUND = "404 (Not Found) \n";
    private static final String MIME = "text/html";
    private static final String ENCODING = "UTF-8";

    private static final String ROOT = "wwwroot/";
    private static final String FILE_NAME = "index.html";
    private StatelessServiceContext context;
    private com.sun.net.httpserver.HttpServer server;
    private ServicePartitionKey partitionKey;
    private final int port;

    public HttpCommunicationListener(StatelessServiceContext context, int port) {
        this.partitionKey = new ServicePartitionKey(0);
        this.context = context;
        this.port = port;
    }

    // Called by openAsync when the class is instantiated
    public void start() {
        try {
            logger.log(Level.INFO, "Starting Server");
            server = com.sun.net.httpserver.HttpServer.create(new InetSocketAddress(this.port), 0);
        } catch (Exception ex) {
            logger.log(Level.SEVERE, null, ex);
            throw new RuntimeException(ex);
        }

        // Responsible for rendering the HTML layout described in the previous step
        server.createContext("/", new HttpHandler() {
            @Override
            public void handle(HttpExchange t) {
                try {
                    File file = new File(ROOT + FILE_NAME).getCanonicalFile();

                    if (!file.isFile()) {
                      // Object does not exist or is not a file: reject with 404 error.
                      t.sendResponseHeaders(STATUS_NOT_FOUND, RESPONSE_NOT_FOUND.length());
                      OutputStream os = t.getResponseBody();
                      os.write(RESPONSE_NOT_FOUND.getBytes());
                      os.close();
                    } else {
                      Headers h = t.getResponseHeaders();
                      h.set(HEADER_CONTENT_TYPE, MIME);
                      t.sendResponseHeaders(STATUS_OK, 0);
    
                      OutputStream os = t.getResponseBody();
                      FileInputStream fs = new FileInputStream(file);
                      final byte[] buffer = new byte[0x10000];
                      int count = 0;
                      while ((count = fs.read(buffer)) >= 0) {
                        os.write(buffer,0,count);
                      }

                      fs.close();
                      os.close();
                    }
                } catch (Exception e) {
                    logger.log(Level.WARNING, null, e);
                }
            }
        });

        /*
        [Replace this entire comment block in the 'Connect the services' section]
        */

        server.setExecutor(null);
        server.start();
    }

    //Helper method to parse raw HTTP requests
    private Map<String, String> queryToMap(String query){
        Map<String, String> result = new HashMap<String, String>();
        for (String param : query.split("&")) {
            String pair[] = param.split("=");
            if (pair.length>1) {
                result.put(pair[0], pair[1]);
            }else{
                result.put(pair[0], "");
            }
        }
        return result;
    }

    //Called closeAsync when the service is shut down
    private void stop() {
        if (null != server)
            server.stop(0);
    }

    //Called by the Service Fabric runtime when this service is created on a node
    @Override
    public CompletableFuture<String> openAsync(CancellationToken cancellationToken) {
        this.start();
                    logger.log(Level.INFO, "Opened Server");
        String publishUri = String.format("http://%s:%d/", this.context.getNodeContext().getIpAddressOrFQDN(), port);
        return CompletableFuture.completedFuture(publishUri);
    }

    //Called by the Service Fabric runtime when the service is shut down
    @Override
    public CompletableFuture<?> closeAsync(CancellationToken cancellationToken) {
        this.stop();
        return CompletableFuture.completedFuture(true);
    }

    //Called by the Service Fabric runtime to forcibly shut this listener down
    @Override
    public void abort() {
        this.stop();
    }
}
```

### <a name="configure-the-listening-port"></a>Configurer le port d’écoute

Lorsque le service frontal VotingWeb est créé, Service Fabric sélectionne le port d’écoute du service.  Le service VotingWeb joue le rôle de serveur frontal de cette application et accepte le trafic externe. Associez ce service à un port fixe et connu. Dans l’Explorateur de package, ouvrez le fichier *VotingApplication/VotingWebPkg/ServiceManifest.xml*.  Recherchez la ressource **Endpoint** dans la section **Resources**, puis remplacez la valeur du **port** par 8080 (nous continuerons à utiliser ce port tout au long du tutoriel). Pour déployer et exécuter l’application localement, le port d’écoute de l’application doit être ouvert et disponible sur votre ordinateur. Collez l’extrait de code suivant dans l’élément **ServiceManifest** (par exemple, juste en dessous de l’élément ```<DataPackage>```).

```xml
<Resources>
    <Endpoints>
        <!-- This endpoint is used by the communication listener to obtain the port on which to
            listen. Please note that if your service is partitioned, this port is shared with
            replicas of different partitions that are placed in your code. -->
        <Endpoint Name="WebEndpoint" Protocol="http" Port="8080" />
    </Endpoints>
  </Resources>
```

## <a name="add-a-stateful-back-end-service-to-your-application"></a>Ajouter un service principal avec état à votre application

Maintenant que la structure du service d’API web Java est terminée, passons à la création du service principal avec état.

Service Fabric permet de stocker de manière cohérente et fiable vos données directement dans votre service à l’aide de collections fiables. Les collections fiables sont un ensemble de classes de collection hautement fiables et disponibles. L’utilisation de ces classes doit être connue de toute personne ayant déjà utilisé des collections Java.

1. Dans l’Explorateur de package, cliquez avec le bouton droit sur **Voting** dans le projet d’application et sélectionnez **Service Fabric** > **Ajouter un service Service Fabric**.

2. Dans la boîte de dialogue **Ajouter un service**, sélectionnez **Service avec état**, nommez le service **VotingDataService**, puis sélectionnez **Ajouter un service**.

    Une fois votre projet de service créé, votre application contient deux services. À mesure que vous continuez à développer votre application, vous pouvez ajouter d’autres services de la même façon. Chacun peut être mis à niveau et faire l'objet d'un contrôle de version indépendamment.

3. Eclipse crée un projet de service et l’affiche dans l’Explorateur de package.

    ![Explorateur de projets Eclipse](./media/service-fabric-tutorial-create-java-app/service-fabric-package-explorer-java.png)

### <a name="add-the-votingdataservicejava-file"></a>Ajouter le fichier VotingDataService.java

Le fichier *VotingDataService.java* contient les méthodes qui contiennent une logique pour la récupération, l’ajout et la suppression de votes dans les collections fiables. Ajoutez les méthodes de classe **VotingDataService** suivantes dans le fichier *VotingDataService/src/statefulservice/VotingDataService.java*.

```java
package statefulservice;

import java.util.HashMap;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CompletableFuture;
import java.util.concurrent.atomic.AtomicInteger;

import microsoft.servicefabric.services.communication.runtime.ServiceReplicaListener;

import microsoft.servicefabric.services.runtime.StatefulService;

import microsoft.servicefabric.services.remoting.fabrictransport.runtime.FabricTransportServiceRemotingListener;

import microsoft.servicefabric.data.ReliableStateManager;
import microsoft.servicefabric.data.Transaction;
import microsoft.servicefabric.data.collections.ReliableHashMap;
import microsoft.servicefabric.data.utilities.AsyncEnumeration;
import microsoft.servicefabric.data.utilities.KeyValuePair;

import system.fabric.StatefulServiceContext;

import rpcmethods.VotingRPC;

class VotingDataService extends StatefulService implements VotingRPC {
    private static final String MAP_NAME = "votesMap";
    private ReliableStateManager stateManager;

    protected VotingDataService (StatefulServiceContext statefulServiceContext) {
        super (statefulServiceContext);
    }

    @Override
    protected List<ServiceReplicaListener> createServiceReplicaListeners() {
        this.stateManager = this.getReliableStateManager();
        ArrayList<ServiceReplicaListener> listeners = new ArrayList<>();

        listeners.add(new ServiceReplicaListener((context) -> {
            return new FabricTransportServiceRemotingListener(context,this);
        }));

        return listeners;
    }

    // Method that will be invoked via RPC from the front end to retrieve the complete set of votes in the map
    public CompletableFuture<HashMap<String,String>> getList() {
        HashMap<String, String> tempMap = new HashMap<String, String>();

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            AsyncEnumeration<KeyValuePair<String, String>> kv = votesMap.keyValuesAsync(tx).get();
            while (kv.hasMoreElementsAsync().get()) {
                KeyValuePair<String, String> k = kv.nextElementAsync().get();
                tempMap.put(k.getKey(), k.getValue());
            }

            tx.close();


        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(tempMap);
    }

    // Method that will be invoked via RPC from the front end to add an item to the votes list or to increase the
    // vote count for a particular item
    public CompletableFuture<Integer> addItem(String itemToAdd) {
        AtomicInteger status = new AtomicInteger(-1);

        try {

            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.computeAsync(tx, itemToAdd, (k, v) -> {
                if (v == null) {
                    return "1";
                }
                else {
                    int numVotes = Integer.parseInt(v);
                    numVotes = numVotes + 1;
                    return Integer.toString(numVotes);
                }
            }).get();

            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

    // Method that will be invoked via RPC from the front end to remove an item
    public CompletableFuture<Integer> removeItem(String itemToRemove) {
        AtomicInteger status = new AtomicInteger(-1);
        try {
            ReliableHashMap<String, String> votesMap = stateManager
                    .<String, String> getOrAddReliableHashMapAsync(MAP_NAME).get();

            Transaction tx = stateManager.createTransaction();
            votesMap.removeAsync(tx, itemToRemove).get();
            tx.commitAsync().get();
            tx.close();

            status.set(1);
        } catch (Exception e) {
            e.printStackTrace();
        }

        return CompletableFuture.completedFuture(new Integer(status.get()));
    }

}
```

La structure du service frontal sans état et du service principal est maintenant créée.

## <a name="create-the-communication-interface-to-your-application"></a>Créer l’interface de communication de votre application

 L’étape suivante consiste à connecter le service sans état front-end et le service back-end. Les deux services utilisent une interface appelée VotingRPC qui définit les opérations de l’application Voting. Cette interface est implémentée par le service frontal et le service principal pour permettre les appels de procédure distante (RPC) entre les deux services. Eclipse ne prenant malheureusement pas en charge l’ajout de sous-projets Gradle, le package qui contient cette interface doit être ajouté manuellement.

1. Cliquez avec le bouton droit sur le projet **Voting** dans l’Explorateur de package, puis sélectionnez **Nouveau** > **Dossier**. Nommez le dossier **VotingRPC/src/rpcmethods**.

    ![Créer un package VotingRPC dans l’Explorateur de packages Eclipse](./media/service-fabric-tutorial-create-java-app/create-voting-rpc-package-java.png)

3. Sous *Voting/VotingRPC/src/rpcmethods*, créez un fichier nommé *VotingRPC.java* et collez la commande suivante dans ce fichier **VotingRPC.java**. 

    ```java
    package rpcmethods;
    
    import java.util.ArrayList;
    import java.util.concurrent.CompletableFuture;
    import java.util.List;
    import java.util.HashMap;
    
    import microsoft.servicefabric.services.remoting.Service;
    
    public interface VotingRPC extends Service {
        CompletableFuture<HashMap<String, String>> getList();
    
        CompletableFuture<Integer> addItem(String itemToAdd);
    
        CompletableFuture<Integer> removeItem(String itemToRemove);
    }
    ```

4. Créez un fichier vide nommé *build.gradle* dans le répertoire *Voting/VotingRPC* et collez-y le texte suivant. Ce fichier Gradle sert à générer le fichier jar importé par les autres services. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/out"
        delete "${projectDir}/VotingRPC.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0')
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}

        manifest {
                attributes(
                'Main-Class': 'rpcmethods.VotingRPC')
            baseName "VotingRPC"
            destinationDir = file('./')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }

    defaultTasks 'clean', 'jar'
    ```

5. Dans le fichier *Voting/settings.gradle*, ajoutez une ligne correspondant au projet nouvellement créé dans le build. 

    ```gradle
    include ':VotingRPC'
    ```

6. Dans le fichier *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java*, remplacez le bloc de commentaire par le texte suivant.  

    ```java
    server.createContext("/getStatelessList", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                t.sendResponseHeaders(STATUS_OK,0);
                OutputStream os = t.getResponseBody();
    
                HashMap<String,String> list = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").getList().get();
                String json = new Gson().toJson(list);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    
    server.createContext("/removeItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                OutputStream os = t.getResponseBody();
                URI r = t.getRequestURI();
    
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToRemove = params.get("item");

                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").removeItem(itemToRemove).get();
    
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
    
        }
    });
    
    server.createContext("/addItem", new HttpHandler() {
        @Override
        public void handle(HttpExchange t) {
            try {
                URI r = t.getRequestURI();
                Map<String, String> params = queryToMap(r.getQuery());
                String itemToAdd = params.get("item");

                OutputStream os = t.getResponseBody();
                Integer num = ServiceProxyBase.create(VotingRPC.class, new URI("fabric:/VotingApplication/VotingDataService"), partitionKey, TargetReplicaSelector.DEFAULT, "").addItem(itemToAdd).get();
                if (num != 1)
                {
                    t.sendResponseHeaders(STATUS_ERROR, 0);
                } else {
                    t.sendResponseHeaders(STATUS_OK,0);
                }
    
                String json = new Gson().toJson(num);
                os.write(json.getBytes(ENCODING));
                os.close();
            } catch (Exception e) {
                logger.log(Level.WARNING, null, e);
            }
        }
    });
    ```
7. Ajoutez l’instruction d’importation appropriée tout en haut du fichier *Voting/VotingWeb/src/statelessservice/HttpCommunicationListener.java*. 

    ```java
    import rpcmethods.VotingRPC; 
    ```

À ce stade, les interfaces frontale, principale et RPC sont fonctionnelles. L’étape suivante consiste à configurer correctement les scripts Gradle avant le déploiement vers un cluster Service Fabric. 

## <a name="walk-through-the-voting-sample-application"></a>Guide de l’exemple d’application de vote
L’application de vote se compose de deux services :
- Service frontal web (VotingWeb) : service frontal web Java qui fait office de page web et expose les API pour communiquer avec le service principal.
- Service principal (VotingDataService) : service web Java qui définit les méthodes invoquées via des appels de procédure distante (RPC) pour conserver les votes.

![Diagramme d’un exemple Voting](./media/service-fabric-tutorial-create-java-app/walkthrough-java-voting.png)

Lorsque vous effectuez une action dans l’application (ajouter un élément, voter, supprimer un élément), les événements suivants se produisent :
1. Un JavaScript envoie la requête appropriée à l’API web dans le service frontal web en tant que requête HTTP.

2. Le service web frontal utilise les fonctionnalités intégrées d’accès distant au service de Service Fabric pour rechercher et transmettre la requête au service principal. 

3. Le service principal définit des méthodes qui mettent à jour le résultat dans un dictionnaire fiable. Le contenu de ce dictionnaire fiable est répliqué sur plusieurs nœuds au sein du cluster et conservé sur disque. Toutes les données de l’application sont stockées dans le cluster. 

## <a name="configure-gradle-scripts"></a>Configurer des scripts Gradle 

Dans cette section, les scripts Gradle du projet sont configurés. 

1. Remplacez le contenu du fichier *Voting/build.gradle* par le code suivant.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    subprojects {
        apply plugin: 'java'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

2. Remplacez le contenu du fichier *Voting/VotingWeb/build.gradle* par le code suivant.

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingWebPkg/Code/VotingWeb.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingWebPkg/Code/lib")
            include('lib*.so')
        }
    }
    
    compileJava.dependsOn(explodeDeps)
    
    jar {
        from configurations.compile.collect {(it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes(
                'Main-Class': 'statelessservice.VotingWebServiceHost')
            baseName "VotingWeb"
            destinationDir = file('../VotingApplication/VotingWebPkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

3. Remplacez le contenu du fichier *Voting/VotingDataService/build.gradle*. 

    ```gradle
    apply plugin: 'java'
    apply plugin: 'eclipse'
    
    sourceSets {
      main {
         java.srcDirs = ['src']
         output.classesDir = 'out/classes'
          resources {
           srcDirs = ['src']
         }
       }
    }
    
    clean.doFirst {
        delete "${projectDir}/../lib"
        delete "${projectDir}/out"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/lib"
        delete "${projectDir}/../VotingApplication/VotingDataServicePkg/Code/VotingDataService.jar"
    }
    
    repositories {
        mavenCentral()
    }
    
    dependencies {
        compile ('com.microsoft.servicefabric:sf-actors:1.0.0-preview1')
        compile project(':VotingRPC')
    }
    
    task explodeDeps(type: Copy, dependsOn:configurations.compile) { task ->
        configurations.compile.filter {!it.toString().contains("native")}.each{
            from it
        }
    
        configurations.compile.filter {it.toString().contains("native")}.each{
            from zipTree(it)
        }
        into "../lib/"
        include "lib*.so", "*.jar"
    }
    
    compileJava.dependsOn(explodeDeps)
    
    task copyDeps<< {
        copy {
            from("../lib/")
            into("../VotingApplication/VotingDataServicePkg/Code/lib")
            include('lib*.so')
        }
    }
    
    jar {
        from configurations.compile.collect {
            (it.isDirectory() && !it.getName().contains("native")) ? it : zipTree(it)}
    
        manifest {
            attributes('Main-Class': 'statefulservice.VotingDataServiceHost')
    
            baseName "VotingDataService"
            destinationDir = file('../VotingApplication/VotingDataServicePkg/Code/')
        }
    
        exclude 'META-INF/*.RSA', 'META-INF/*.SF','META-INF/*.DSA'
    }
    
    defaultTasks 'clean', 'jar', 'copyDeps'
    ```

## <a name="deploy-application-to-local-cluster"></a>Déployer l’application sur le cluster local

À ce stade, l’application est prête à être déployée vers un cluster Service Fabric local.

1. Cliquez avec le bouton droit sur le projet **Voting** dans l’Explorateur de package, puis sélectionnez **Service Fabric** > **Créer l’application** pour créer l’application.

2. Exécutez votre cluster Service Fabric local. Cette étape varie en fonction de votre environnement de développement (Mac ou Linux).

    Sur un Mac, exécutez le cluster local à l’aide de la commande suivante : Remplacez la commande passée au paramètre **-v** par le chemin de votre espace de travail.

    ```bash
    docker run -itd -p 19080:19080 -p 8080:8080 -p --name sfonebox mcr.microsoft.com/service-fabric/onebox:latest
    ```
    Pour obtenir des instructions plus détaillées, consultez le [guide de configuration OS X](service-fabric-get-started-mac.md).

    Si vous exécutez le service sur un ordinateur Linux, démarrez le cluster local avec la commande suivante : 

    ```bash 
    sudo /opt/microsoft/sdk/servicefabric/common/clustersetup/devclustersetup.sh
    ```
    Pour obtenir des instructions plus détaillées, consultez le [guide de configuration Linux](service-fabric-get-started-linux.md).

4. Dans l’Explorateur de package pour Eclipse, cliquez avec le bouton droit sur le projet **Voting** et sélectionnez **Service Fabric** > **Publish Application** (Publier une application) 
5. Dans la fenêtre **Publish Application** (Publier une application), sélectionnez **Local.json** dans la liste déroulante, puis **Publier**.
6. À partir de votre navigateur web, accédez à http:\//localhost:8080 pour voir l’application en cours d’exécution sur le cluster Service Fabric local. 

## <a name="next-steps"></a>Étapes suivantes

Dans cette partie du tutoriel, vous avez appris à :

> [!div class="checklist"]
> * Créer un service Java en tant que service fiable avec état
> * Créer un service Java en tant que service web sans état
> * Ajouter une interface Java pour gérer les appels de procédure distante (RPC) entre vos services
> * Configurer vos scripts Gradle
> * Créer et déployer votre application sur un cluster Service Fabric local

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Déboguer une application Java déployée sur un cluster local](service-fabric-tutorial-debug-log-local-cluster.md)

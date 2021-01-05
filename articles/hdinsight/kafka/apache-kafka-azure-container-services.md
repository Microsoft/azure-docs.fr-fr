---
title: Utiliser Azure Kubernetes Service avec Kafka sur HDInsight
description: Découvrez comment utiliser Kafka sur HDInsight à partir d’images de conteneur hébergées dans Azure Kubernetes Service (AKS).
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: hdinsightactive
ms.date: 12/04/2019
ms.openlocfilehash: ab87f181f78158d2ea0dd6575a30e6087600f60c
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92485679"
---
# <a name="use-azure-kubernetes-service-with-apache-kafka-on-hdinsight"></a>Utiliser Azure Kubernetes Service avec Apache Kafka sur HDInsight

Découvrez comment utiliser Azure Kubernetes Service (AKS) avec [Apache Kafka](https://kafka.apache.org/) sur un cluster HDInsight. Les étapes de ce document utilisent une application Node.js hébergée dans AKS pour vérifier la connectivité avec Kafka. Cette application utilise le package [kafka-node](https://www.npmjs.com/package/kafka-node) pour communiquer avec Kafka. Elle utilise [Socket.io](https://socket.io/) pour les échanges entre le navigateur client et le serveur principal hébergé dans AKS liés aux événements.

[Apache Kafka](https://kafka.apache.org) est une plateforme de diffusion en continu distribuée open source qui permet de générer des pipelines de données et des applications de diffusion en continu en temps réel. Azure Kubernetes Service gère votre environnement Kubernetes hébergé et facilite et accélère le déploiement des applications en conteneur. À l’aide d’un réseau virtuel Azure, vous pouvez connecter les deux services.

> [!NOTE]  
> Ce document se concentre sur les étapes à suivre pour permettre à Azure Kubernetes Service de communiquer avec Kafka sur HDInsight. Nous prenons pour exemple un simple client Kafka pour montrer que la configuration fonctionne.

## <a name="prerequisites"></a>Prérequis

* [Azure CLI](/cli/azure/install-azure-cli)
* Abonnement Azure

Ce document suppose que vous êtes familiarisé avec la création et l’utilisation des services Azure suivants :

* Kafka sur HDInsight
* Azure Kubernetes Service
* Réseaux virtuels Azure

Ce document suppose également que vous avez parcouru le [tutoriel Azure Kubernetes Service](../../aks/tutorial-kubernetes-prepare-app.md). Cet article crée un service conteneur, un cluster Kubernetes, un registre de conteneurs et configure l’utilitaire `kubectl`.

## <a name="architecture"></a>Architecture

### <a name="network-topology"></a>Topologie du réseau

HDInsight et AKS utilisent un réseau virtuel Azure comme conteneur pour les ressources de calcul. Pour activer la communication entre HDInsight et AKS, vous devez activer la communication entre leurs réseaux. Les étapes décrites dans ce document utilisent le peering de réseau virtuel pour les réseaux. D’autres connexions, comme les VPN, doivent également fonctionner. Pour plus d’informations sur le peering, consultez le document [Peering de réseaux virtuels](../../virtual-network/virtual-network-peering-overview.md).

Le diagramme suivant illustre la topologie de réseau utilisée dans ce document :

![HDInsight dans un réseau virtuel, AKS dans un autre avec un appairage](./media/apache-kafka-azure-container-services/kafka-aks-architecture.png)

> [!IMPORTANT]  
> La résolution de noms n’est pas activée entre les réseaux homologués, l’adressage IP est donc utilisé. Par défaut, Kafka sur HDInsight est configuré pour retourner les noms d’hôte à la place des adresses IP lorsque les clients se connectent. Les étapes décrites dans ce document modifient Kafka pour utiliser la publication IP à la place.

## <a name="create-an-azure-kubernetes-service-aks"></a>Créer un cluster Azure Kubernetes Service (AKS)

Si vous n’avez pas déjà un cluster AKS, utilisez un des documents suivants pour apprendre à créer un :

* [Déployer un cluster Azure Kubernetes Service (AKS) : portail](../../aks/kubernetes-walkthrough-portal.md)
* [Déployer un cluster Azure Kubernetes Service (AKS) : CLI](../../aks/kubernetes-walkthrough.md)

> [!IMPORTANT]  
> AKS crée un réseau virtuel pendant l’installation dans un groupe de ressources **supplémentaire** . Le groupe de ressources supplémentaire suit la convention de nommage de **MC_resourceGroup_AKSclusterName_location** .  
> Ce réseau est homologué avec celui créé pour HDInsight dans la section suivante.

## <a name="configure-virtual-network-peering"></a>Configurer le peering de réseaux virtuels

### <a name="identify-preliminary-information"></a>Identifier des informations préliminaires

1. À partir du [portail Azure](https://portal.azure.com), localisez le **groupe de ressources** supplémentaire qui contient le réseau virtuel de votre cluster AKS.

2. À partir du groupe de ressources, sélectionnez la ressource __Réseau virtuel__ . Notez le nom. Il sera utile plus tard.

3. Sous **Paramètres** , sélectionnez __Espace d’adressage__ . Notez l’espace d’adressage indiqué.

### <a name="create-virtual-network"></a>Création d’un réseau virtuel

1. Pour créer un réseau virtuel pour HDInsight, accédez à __+ Créer une ressource__ > __Mise en réseau__ > __Réseau virtuel__ .

1. Créez le réseau à l’aide des instructions suivantes pour certaines propriétés :

    |Propriété | Valeur |
    |---|---|
    |Espace d’adressage|Vous devez utiliser un espace d’adressage qui ne chevauche pas celui utilisé par le réseau en cluster AKS.|
    |Emplacement|Utilisez le même __emplacement__ pour le réseau virtuel que celui que vous avez utilisé pour le cluster AKS.|

1. Attendez que le réseau virtuel ait été créé avant de passer à l’étape suivante.

### <a name="configure-peering"></a>Configurer le peering

1. Pour configurer le peering entre le réseau HDInsight et le réseau de cluster AKS, sélectionnez le réseau virtuel, puis sélectionnez __Peerings__ .

1. Sélectionnez __+ Ajouter__ et utilisez les valeurs suivantes pour remplir le formulaire :

    |Propriété |Valeur |
    |---|---|
    |Nom du peering de \<this VN> au réseau virtuel distant|entrez un nom unique pour cette configuration de peering.|
    |Réseau virtuel|Sélectionnez le réseau virtuel pour le **cluster AKS** .|
    |Nom du peering de \<this VN> à \<AKS VN>|Entrez un nom unique.|

    Laissez tous les autres champs sur la valeur par défaut, puis sélectionnez __OK__ pour configurer le peering.

## <a name="create-apache-kafka-cluster-on-hdinsight"></a>Créer un cluster Apache Kafka sur HDInsight

Lorsque vous créez le cluster Kafka sur le cluster HDInsight, vous devez joindre le réseau virtuel créé précédemment pour HDInsight. Pour plus d’informations sur la création d’un cluster Kafka, consultez le document [Créer un cluster Apache Kafka](apache-kafka-get-started.md).

## <a name="configure-apache-kafka-ip-advertising"></a>Configurer la publication d’adresses IP Apache Kafka

Suivez les étapes ci-dessous pour configurer Kafka afin qu’il publie des adresses IP plutôt que des noms de domaine :

1. À l’aide d’un navigateur web, accédez à `https://CLUSTERNAME.azurehdinsight.net`. Remplacez CLUSTERNAME par le nom du cluster Kafka sur HDInsight.

    Lorsque vous y êtes invité, utilisez le nom et le mot de passe utilisateur HTTPS correspondant au cluster. L’interface utilisateur web d’Ambari pour le cluster s’affiche.

2. Pour afficher plus d’informations sur Kafka, sélectionnez __Kafka__ dans la liste sur la gauche.

    ![Liste des services avec l’option Kafka en surbrillance](./media/apache-kafka-azure-container-services/select-kafka-service.png)

3. Pour afficher la configuration Kafka, sélectionnez __Configurations__ en haut et au centre de la page.

    ![Apache Ambari - Configuration des services](./media/apache-kafka-azure-container-services/select-kafka-config1.png)

4. Pour trouver la configuration __kafka-env__ , entrez `kafka-env` dans le champ __Filtre__ situé en haut à droite.

    ![Configuration Kafka, pour kafka-env](./media/apache-kafka-azure-container-services/search-for-kafka-env.png)

5. Pour configurer Kafka afin qu’il publie des adresses IP, ajoutez le texte suivant au bas du champ __kafka-env-template__  :

    ```bash
    # Configure Kafka to advertise IP addresses instead of FQDN
    IP_ADDRESS=$(hostname -i)
    echo advertised.listeners=$IP_ADDRESS
    sed -i.bak -e '/advertised/{/advertised@/!d;}' /usr/hdp/current/kafka-broker/conf/server.properties
    echo "advertised.listeners=PLAINTEXT://$IP_ADDRESS:9092" >> /usr/hdp/current/kafka-broker/conf/server.properties
    ```

6. Pour configurer l’interface écoutée par Kafka, entrez `listeners` dans le champ __Filtre__ situé en haut à droite.

7. Pour configurer Kafka afin qu’il écoute toutes les interfaces réseau, remplacez la valeur du champ __écouteurs__ par `PLAINTEXT://0.0.0.0:9092`.

8. Utilisez le bouton __Enregistrer__ pour enregistrer les modifications apportées à la configuration. Entrez un message texte décrivant les modifications. Sélectionnez __OK__ une fois les modifications apportées.

    ![Apache Ambari - Enregistrement de la configuration](./media/apache-kafka-azure-container-services/save-configuration-button.png)

9. Pour éviter des erreurs lors du redémarrage de Kafka, utilisez le bouton __Service Actions__ (Actions du service) et sélectionnez __Activer le mode de maintenance__ . Sélectionnez OK pour terminer cette opération.

    ![Actions de service, avec l’option Activer le mode de maintenance en surbrillance](./media/apache-kafka-azure-container-services/turn-on-maintenance-mode.png)

10. Pour redémarrer Kafka, utilisez le bouton __Redémarrer__ et sélectionnez __Restart All Affected__ (Redémarrer tous les éléments affectés). Confirmez le redémarrage, puis utilisez le bouton __OK__ une fois l’opération terminée.

    ![Bouton Redémarrer avec l’option Restart All Affected (Redémarrer tous les éléments affectés) en surbrillance](./media/apache-kafka-azure-container-services/restart-required-button.png)

11. Pour désactiver le mode de maintenance, utilisez le bouton __Service Actions__ (Actions du service) et sélectionnez __Désactiver le mode de maintenance__ . Sélectionnez **OK** pour terminer cette opération.

## <a name="test-the-configuration"></a>Tester la configuration

À ce stade, Kafka et Azure Kubernetes Service sont en communication via les réseaux virtuels appairés. Pour tester cette connexion, procédez comme suit :

1. Créez une rubrique Kafka qui est utilisée par l’application de test. Pour plus d’informations sur la création de rubriques Kafka, consultez le document [Créer un cluster Apache Kafka](apache-kafka-get-started.md).

2. Téléchargez l’exemple d’application à partir de [https://github.com/Blackmist/Kafka-AKS-Test](https://github.com/Blackmist/Kafka-AKS-Test).

3. Modifiez le fichier `index.js` et modifiez les lignes suivantes :

    * `var topic = 'mytopic'`: remplacez `mytopic` par le nom de la rubrique Kafka utilisé par cette application.
    * `var brokerHost = '176.16.0.13:9092`: remplacez `176.16.0.13` par l’adresse IP interne de l’un des hôtes du répartiteur pour votre cluster.

        Pour trouver l’adresse IP interne des hôtes du répartiteur (workernodes) dans le cluster, consultez le document [API REST Apache Ambari](../hdinsight-hadoop-manage-ambari-rest-api.md#get-the-internal-ip-address-of-cluster-nodes). Choisissez l’adresse IP de l’une des entrées dont le nom de domaine commence par `wn`.

4. À partir d’une ligne de commande dans le répertoire `src`, installez des dépendances et utilisez Docker pour générer une image pour le déploiement :

    ```bash
    docker build -t kafka-aks-test .
    ```

    > [!NOTE]  
    > Les packages requis par cette application sont archivés dans le référentiel, vous n’avez donc pas besoin d’utiliser l’utilitaire `npm` pour les installer.

5. Connectez-vous à votre service Azure Container Registry (ACR) et recherchez le nom de loginServer :

    ```azurecli
    az acr login --name <acrName>
    az acr list --resource-group myResourceGroup --query "[].{acrLoginServer:loginServer}" --output table
    ```

    > [!NOTE]  
    > Si vous ne connaissez pas le nom de votre service Azure Container Registry ou si vous ne savez pas comment utiliser Azure Kubernetes Service à l’aide de l’interface de ligne de commande Azure, consultez les [tutoriels AKS](../../aks/tutorial-kubernetes-prepare-app.md).

6. Marquez l’image `kafka-aks-test` locale avec le loginServer de votre ACR. Ajoutez également `:v1` à la fin pour indiquer la version de l’image :

    ```bash
    docker tag kafka-aks-test <acrLoginServer>/kafka-aks-test:v1
    ```

7. Envoyez l’image vers le registre :

    ```bash
    docker push <acrLoginServer>/kafka-aks-test:v1
    ```

    Cette opération prend plusieurs minutes.

8. Modifiez le fichier manifeste Kubernetes (`kafka-aks-test.yaml`) et remplacez `microsoft` par le nom loginServer d’ACR récupéré à l’étape 4.

9. Utilisez la commande suivante pour déployer les paramètres d’application à partir du manifeste :

    ```bash
    kubectl create -f kafka-aks-test.yaml
    ```

10. Utilisez la commande suivante pour surveiller le `EXTERNAL-IP` de l’application :

    ```bash
    kubectl get service kafka-aks-test --watch
    ```

    Une fois qu’une adresse IP externe est affectée, utilisez __CTRL + C__ pour quitter la surveillance

11. Ouvrez un navigateur web et entrez l’adresse IP externe pour le service. Vous accédez à une page similaire à l’image ci-dessous :

    ![Image de page web de test Apache Kafka](./media/apache-kafka-azure-container-services/test-web-page-image1.png)

12. Entrez le texte dans le champ, puis sélectionnez le bouton __Envoyer__ . Les données sont envoyées à Kafka. Ensuite, le consommateur Kafka dans l’application lit le message et l’ajoute à la section __Messages à partir de Kafka__ .

    > [!WARNING]  
    > Vous pouvez recevoir plusieurs copies d’un message. Ce problème se produit généralement lorsque vous actualisez votre navigateur une fois connecté, ou lorsque vous ouvrez plusieurs connexions de navigateur à l’application.

## <a name="next-steps"></a>Étapes suivantes

Cliquez sur les liens suivants pour apprendre à utiliser Apache Kafka sur HDInsight :

* [Prise en main d’Apache Kafka sur HDInsight](apache-kafka-get-started.md)

* [Utiliser MirrorMaker pour créer un réplica d’Apache Kafka sur HDInsight](apache-kafka-mirroring.md)

* [Utiliser Apache Storm avec Apache Kafka sur HDInsight](../hdinsight-apache-storm-with-kafka.md)

* [Utiliser Apache Spark avec Apache Kafka sur HDInsight](../hdinsight-apache-spark-with-kafka.md)

* [Se connecter à Apache Kafka via un réseau virtuel Azure](apache-kafka-connect-vpn-gateway.md)

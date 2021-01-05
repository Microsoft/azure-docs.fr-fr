---
title: Superviser vos applications dans Azure Service Fabric avec ELK
description: Dans ce tutoriel, découvrez comment configurer ELK et surveiller vos applications Service Fabric.
ms.topic: tutorial
ms.date: 02/26/2018
ms.custom: mvc, devx-track-java
ms.openlocfilehash: 5faf26230618161a5b908c9a544a43ec5c33b807
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91532020"
---
# <a name="tutorial-monitor-your-service-fabric-applications-using-elk"></a>Tutoriel : Surveiller vos applications dans Azure Service Fabric avec ELK

Ce tutoriel est la quatrième partie de la série. Il montre comment utiliser ELK (Elasticsearch, Logstash et Kibana) pour surveiller les applications Service Fabric s’exécutant dans Azure.

Dans ce quatrième volet, vous apprenez à :
> [!div class="checklist"]
> * Configurer le serveur ELK dans Azure
> * Configurer Logstash pour recevoir des journaux d’activité d’Event Hubs
> * Visualiser la plateforme et les journaux des applications dans Kibana

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :
> [!div class="checklist"]
> * [Générer une application Reliable Services Service Fabric de Java](service-fabric-tutorial-create-java-app.md)
> * [Déployer et déboguer l’application sur un cluster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * [Déployer l’application sur un cluster Azure](service-fabric-tutorial-java-deploy-azure.md)
> * Configurer la surveillance et les diagnostics pour l’application
> * [Configurer CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* Configurez votre application pour publier des journaux d’activité à l’emplacement spécifié dans la [deuxième partie](service-fabric-tutorial-debug-log-local-cluster.md).
* Terminez la [troisième partie](service-fabric-tutorial-java-deploy-azure.md) et utilisez un cluster Service Fabric en cours d’exécution configuré pour envoyer des journaux d’activité à Event Hubs.
* La stratégie dans Event Hubs qui a l’autorisation « Écouter » et la clé primaire associée à partir de la troisième série.

## <a name="download-the-voting-sample-application"></a>Télécharger l’exemple d’application de vote

Si vous n’avez pas généré l’exemple d’application de vote lors de la [première partie de cette série de didacticiels](service-fabric-tutorial-create-java-app.md), vous pouvez le télécharger. Dans une fenêtre Commande, exécutez la commande suivante pour cloner le référentiel de l’exemple d’application sur votre ordinateur local.

```bash
git clone https://github.com/Azure-Samples/service-fabric-java-quickstart
```

## <a name="create-an-elk-server-in-azure"></a>Créer un serveur ELK dans Azure

Vous pouvez utiliser un environnement ELK préconfiguré pour ce didacticiel. Si vous en avez déjà un, passez à la section **Configurer ELK**. Toutefois, si vous n’en avez pas, la procédure suivante en crée un dans Azure.

1. Créez un ELK certifié par [Bitnami](https://ms.portal.azure.com/#create/bitnami.elk4-6) dans Azure. Pour les besoins de ce tutoriel, il n’y a pas de spécifications particulières à suivre pour la création de ce serveur.

2. Accédez aux ressources dans le portail Azure, puis à l’onglet **Diagnostics de démarrage** dans la section **Support + Troubleshooting** (Support + Dépannage). Ensuite, cliquez sur l’onglet **Journal série**.

    ![Diagnostics-de-démarrage](./media/service-fabric-tutorial-java-elk/bootdiagnostics.png)
3. Effectuer une recherche sur les journaux d’activité pour le mot de passe est requis pour accéder à l’instance Kibana. L’extrait de code se présente comme suit :

    ```bash
    [   25.932766] bitnami[1496]: #########################################################################
    [   25.948656] bitnami[1496]: #                                                                       #
    [   25.962448] bitnami[1496]: #        Setting Bitnami application password to '[PASSWORD]'           #
    [   25.978137] bitnami[1496]: #        (the default application username is 'user')                   #
    [   26.004770] bitnami[1496]: #                                                                       #
    [   26.029413] bitnami[1496]: #########################################################################
    ```

4. Appuyez sur le bouton de connexion sur la page Vue d’ensemble du serveur dans le portail Azure pour obtenir les informations de connexion.

    ![Connexion de machine virtuelle](./media/service-fabric-tutorial-java-elk/vmconnection.png)

5. Établissez une connexion SSH sur le serveur qui héberge l’image ELK à l’aide de la commande suivante

    ```bash
    ssh [USERNAME]@[CONNECTION-IP-OF-SERVER]

    Example: ssh testaccount@104.40.63.157
    ```

## <a name="set-up-elk"></a>Configurer ELK

1. La première étape consiste à charger l’environnement ELK

    ```bash
    sudo /opt/bitnami/use_elk
    ```

2. Si vous utilisez un environnement existant, vous devez exécuter la commande suivante pour arrêter le service Logstash

    ```bash
    sudo /opt/bitnami/ctlscript.sh stop logstash
    ```

3. Exécutez la commande suivante pour installer le plug-in Logstash pour Event Hubs.

    ```bash
    logstash-plugin install logstash-input-azureeventhub
    ```

4. Créez ou modifiez votre fichier config Logstash existant avec le contenu suivant : si vous créez le fichier, il doit être créé à l’emplacement ```/opt/bitnami/logstash/conf/access-log.conf``` si vous utilisez l’image ELK Bitnami dans Azure.

    ```json
    input
    {
        azureeventhub
        {
            key => "[RECEIVER-POLICY-KEY-FOR-EVENT-HUB]"
            username => "[RECEIVER-POLICY-NAME]"
            namespace => "[EVENTHUB-NAMESPACENAME]"
            eventhub => "[EVENTHUB-NAME]"
            partitions => 4
        }
    }

    output {
         elasticsearch {
             hosts => [ "127.0.0.1:9200" ]
         }
     }
    ```

5. Pour vérifier la configuration, exécutez la commande suivante :

    ```bash
    /opt/bitnami/logstash/bin/logstash -f /opt/bitnami/logstash/conf/ --config.test_and_exit
    ```

6. Démarrer le service Logstash

    ```bash
    sudo /opt/bitnami/ctlscript.sh start logstash
    ```

7. Vérifiez votre Elasticsearch pour vous assurer que vous recevez des données

    ```bash
    curl 'localhost:9200/_cat/indices?v'
    ```

8. Accédez à votre tableau de bord Kibana à l’adresse **http:\//SERVER-IP** et entrez le nom d’utilisateur et un mot de passe pour Kibana. Si vous avez utilisé l’image ELK dans Azure, le nom d’utilisateur par défaut est « user » et le mot de passe est celui obtenu dans **Diagnostics de démarrage**.

    ![La capture d’écran montre un tableau de bord Kibana pour visualiser les journaux de la plateforme et des applications.](./media/service-fabric-tutorial-java-elk/kibana.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Obtenir un serveur ELK s’exécutant dans Azure
> * Configurer le serveur pour recevoir des informations de diagnostic de votre cluster Service Fabric

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Configurer l’intégration continue/le déploiement continu avec Jenkins](service-fabric-tutorial-java-jenkins.md)

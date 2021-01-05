---
title: Déployer une application Java sur un cluster Service Fabric dans Azure
description: Dans ce didacticiel, vous allez apprendre à déployer une application Java Service Fabric sur un cluster Azure Service Fabric.
ms.topic: tutorial
ms.date: 02/26/2018
ms.custom: mvc, devx-track-java, devx-track-azurecli
ms.openlocfilehash: c2e2b2883bfa01d3a36de5d58425449f6f973010
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97702155"
---
# <a name="tutorial-deploy-a-java-application-to-a-service-fabric-cluster-in-azure"></a>Tutoriel : Déployer une application Java sur un cluster Service Fabric dans Azure

Ce troisième didacticiel de la série vous montre comment déployer une application Service Fabric sur un cluster dans Azure.

Dans ce troisième volet, vous apprenez à :

> [!div class="checklist"]
> * Créer un cluster Linux sécurisé dans Azure
> * Déployer une application sur le cluster

Cette série de tutoriels vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * [Générer une application Reliable Services Service Fabric de Java](service-fabric-tutorial-create-java-app.md)
> * [Déployer et déboguer l’application sur un cluster local](service-fabric-tutorial-debug-log-local-cluster.md)
> * Déployer l’application sur un cluster Azure
> * [Configurer la surveillance et les diagnostics pour l’application](service-fabric-tutorial-java-elk.md)
> * [Configurer CI/CD](service-fabric-tutorial-java-jenkins.md)

## <a name="prerequisites"></a>Prérequis

Avant de commencer ce tutoriel :

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
* [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli?view=azure-cli-latest)
* Installez le Kit de développement logiciel (SDK) Service Fabric pour [Mac](service-fabric-get-started-mac.md) ou [Linux](service-fabric-get-started-linux.md)
* [Installez Python 3](https://wiki.python.org/moin/BeginnersGuide/Download)

## <a name="create-a-service-fabric-cluster-in-azure"></a>Créer un cluster Service Fabric dans Azure

Les étapes suivantes créent les ressources nécessaires pour déployer votre application sur un cluster Service Fabric. En outre, les ressources nécessaires pour surveiller l’intégrité de votre solution à l’aide de la pile ELK (Elasticsearch, Logstash, Kibana) sont configurées. Plus précisément, [Event Hubs](https://azure.microsoft.com/services/event-hubs/) est utilisé en tant que récepteur pour les journaux d’activité de Service Fabric. Il est configuré pour envoyer des journaux d’activité du cluster Service Fabric à votre instance Logstash.

1. Ouvrez un terminal et téléchargez le package suivant qui contient les scripts d’assistance et les modèles nécessaires pour créer les ressources dans Azure

    ```bash
    git clone https://github.com/Azure-Samples/service-fabric-java-quickstart.git
    ```

2. Connexion à votre compte Azure

    ```azurecli
    az login
    ```

3. Configurer l’abonnement Azure que vous souhaitez utiliser pour créer les ressources

    ```azurecli
    az account set --subscription [SUBSCRIPTION-ID]
    ```

4. À partir du dossier *service-fabric-java-quickstart/AzureCluster*, exécutez la commande suivante pour créer un certificat de cluster dans Key Vault. Ce certificat est utilisé pour sécuriser votre cluster Service Fabric. Indiquez la région (doit être identique à votre cluster Service Fabric), le nom du groupe de ressource de coffre de clés, le nom du coffre de clés, le mot de passe du certificat et le nom DNS du cluster.

    ```bash
    ./new-service-fabric-cluster-certificate.sh [REGION] [KEY-VAULT-RESOURCE-GROUP] [KEY-VAULT-NAME] [CERTIFICATE-PASSWORD] [CLUSTER-DNS-NAME-FOR-CERTIFICATE]

    Example: ./new-service-fabric-cluster-certificate.sh 'westus' 'testkeyvaultrg' 'testkeyvault' '<password>' 'testservicefabric.westus.cloudapp.azure.com'
    ```

    La commande précédente renvoie les informations suivantes qui doivent être notées pour une utilisation ultérieure.

    ```output
    Source Vault Resource Id: /subscriptions/<subscription_id>/resourceGroups/testkeyvaultrg/providers/Microsoft.KeyVault/vaults/<name>
    Certificate URL: https://<name>.vault.azure.net/secrets/<cluster-dns-name-for-certificate>/<guid>
    Certificate Thumbprint: <THUMBPRINT>
    ```

5. Créez un groupe de ressources pour le compte de stockage qui stocke vos journaux d’activité

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name teststorageaccountrg
    ```

6. Créez un compte de stockage qui sera utilisé pour stocker les journaux d’activité produits

    ```azurecli
    az storage account create -g [RESOURCE-GROUP-NAME] -l [REGION] --name [STORAGE-ACCOUNT-NAME] --kind Storage

    Example: az storage account create -g teststorageaccountrg -l westus --name teststorageaccount --kind Storage
    ```

7. Accédez au [portail Azure](https://portal.azure.com), puis à l’onglet **Signature d’accès partagé** pour votre compte de stockage. Générez le jeton SAP comme suit.

    ![Générer la SAP pour le stockage](./media/service-fabric-tutorial-java-deploy-azure/storagesas.png)

8. Copiez l’URL de SAP de compte et mettez-la de côté pour l’utiliser lors de la création de votre cluster Service Fabric. L’URL se présente comme suit :

    ```output
    ?sv=2017-04-17&ss=bfqt&srt=sco&sp=rwdlacup&se=2018-01-31T03:24:04Z&st=2018-01-30T19:24:04Z&spr=https,http&sig=IrkO1bVQCHcaKaTiJ5gilLSC5Wxtghu%2FJAeeY5HR%2BPU%3D
    ```

9. Créez un groupe de ressources qui contient les ressources Event Hub. Event Hubs est utilisé pour envoyer des messages de Service Fabric au serveur exécutant les ressources ELK.

    ```azurecli
    az group create --location [REGION] --name [RESOURCE-GROUP-NAME]

    Example: az group create --location westus --name testeventhubsrg
    ```

10. Créez une ressource Event Hubs à l’aide de la commande suivante. Suivez les invites pour entrer les informations de namespaceName, eventHubName, consumerGroupName, sendAuthorizationRule et receiveAuthorizationRule.

    ```azurecli
    az deployment group create -g [RESOURCE-GROUP-NAME] --template-file eventhubsdeploy.json

    Example:
    az deployment group create -g testeventhubsrg --template-file eventhubsdeploy.json
    Please provide string value for 'namespaceName' (? for help): testeventhubnamespace
    Please provide string value for 'eventHubName' (? for help): testeventhub
    Please provide string value for 'consumerGroupName' (? for help): testeventhubconsumergroup
    Please provide string value for 'sendAuthorizationRuleName' (? for help): sender
    Please provide string value for 'receiveAuthorizationRuleName' (? for help): receiver
    ```

    Copiez le contenu du champ de **sortie** dans la sortie JSON de la commande précédente. Les informations d’expéditeur sont utilisées lors de la création du cluster Service Fabric. La clé et le nom du récepteur doivent être enregistrés pour une utilisation dans le didacticiel suivant lorsque le service Logstash est configuré pour recevoir des messages à partir d’Event Hub. L’objet blob suivant est un exemple de sortie JSON :

    ```json
    "outputs": {
        "receiver Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "receiver Name": {
            "type": "String",
            "value": "receiver"
        },
        "sender Key": {
            "type": "String",
            "value": "[KEY]"
        },
        "sender Name": {
            "type": "String",
            "value": "sender"
        }
    }
    ```

11. Exécutez le script *eventhubssastoken.py* pour générer l’URL de la SAP pour la ressource EventHubs que vous avez créée. Cette URL de SAP est utilisée par le cluster Service Fabric pour envoyer des journaux d’activité à Event Hubs. Par conséquent, la stratégie **expéditeur** est utilisée pour générer l’URL. Le script renvoie l’URL de SAP pour la ressource Event Hubs qui est utilisée dans l’étape suivante :

    ```python
    python3 eventhubssastoken.py 'testeventhubs' 'testeventhubs' 'sender' '[PRIMARY-KEY]'
    ```

    Copiez la valeur du champ **sr** dans le JSON renvoyé. La valeur du champ **sr** est le jeton SAP pour EventHubs. L’URL suivante texte un exemple de champ **sr** :

    ```output
    https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender
    ```

    Votre URL SAP pour Event Hubs suit la structure : `https://<namespacename>.servicebus.windows.net/<eventhubsname>?sr=<sastoken>`. Par exemple : `https://testeventhubnamespace.servicebus.windows.net/testeventhub?sr=https%3A%2F%testeventhub.servicebus.windows.net%testeventhub&sig=7AlFYnbvEm%2Bat8ALi54JqHU4i6imoFxkjKHS0zI8z8I%3D&se=1517354876&skn=sender`

12. Ouvrez le fichier *sfdeploy.parameters.json* et remplacez le contenu suivant des étapes précédentes. [SAS-URL-STORAGE-ACCOUNT] a été indiqué à l’étape 8. [SAS-URL-EVENT-HUBS] a été indiqué à l’étape 11.

    ```json
    "applicationDiagnosticsStorageAccountName": {
        "value": "teststorageaccount"
    },
    "applicationDiagnosticsStorageAccountSasToken": {
        "value": "[SAS-URL-STORAGE-ACCOUNT]"
    },
    "loggingEventHubSAS": {
        "value": "[SAS-URL-EVENT-HUBS]"
    }
    ```

13. Ouvrez **sfdeploy.parameters.json**. Modifiez les paramètres suivants puis enregistrez le fichier.
    - **clusterName**. Utilisez uniquement des lettres minuscules et des chiffres.
    - **adminUserName** (pour une valeur autre que vide)
    - **adminPassword** (pour une valeur autre que vide)

14. Exécutez la commande ci-dessous pour créer votre cluster Service Fabric

    ```azurecli
    az sf cluster create --location 'westus' --resource-group 'testlinux' --template-file sfdeploy.json --parameter-file sfdeploy.parameters.json --secret-identifier <certificate_url_from_step4>
    ```

## <a name="deploy-your-application-to-the-cluster"></a>Déployer votre application sur le cluster

1. Avant de déployer votre application, vous devez ajouter l’extrait de code suivant au fichier *Voting/VotingApplication/ApplicationManifest.xml*. Le champ **X509FindValue** est l’empreinte renvoyée de l’étape 4 de la section **Créer un cluster Service Fabric dans Azure**. Cet extrait de code est imbriqué sous le champ **ApplicationManifest** (le champ racine).

    ```xml
    <Certificates>
          <SecretsCertificate X509FindType="FindByThumbprint" X509FindValue="[CERTIFICATE-THUMBPRINT]" />
    </Certificates>
    ```

2. Pour déployer votre application sur ce cluster, vous devez utiliser SFCTL pour établir une connexion au cluster. SFCTL nécessite un fichier PEM avec la clé publique et la clé privée pour se connecter au cluster. Exécutez la commande suivante pour générer un fichier PEM contenant à la fois la clé publique et la clé privée. 

    ```bash
    openssl pkcs12 -in <clustername>.<region>.cloudapp.azure.com.pfx -out sfctlconnection.pem -nodes -passin pass:<password>
    ```

3. Exécutez la commande suivante pour vous connecter au cluster.

    ```bash
    sfctl cluster select --endpoint https://<clustername>.<region>.cloudapp.azure.com:19080 --pem sfctlconnection.pem --no-verify
    ```

4. Pour déployer votre application, accédez au dossier *Voting/Scripts* et exécutez le script **install.sh**.

    ```bash
    ./install.sh
    ```

5. Pour accéder à Service Fabric Explorer, ouvrez votre navigateur favori et tapez `https://testlinuxcluster.westus.cloudapp.azure.com:19080`. Choisissez le certificat dans le magasin de certificats que vous souhaitez utiliser pour vous connecter à ce point de terminaison. Si vous utilisez un ordinateur Linux, les certificats qui ont été générés par le script *new-service-fabric-cluster-certificate.sh* doivent être importés dans Chrome pour afficher Service Fabric Explorer. Si vous utilisez un Mac, vous devez installer le fichier PFX dans votre chaîne de clé. Vous remarquerez que votre application a été installée sur le cluster.

    ![PFX Java Azure](./media/service-fabric-tutorial-java-deploy-azure/sfxjavaonazure.png)

6. Pour accéder à votre application, tapez `https://testlinuxcluster.westus.cloudapp.azure.com:8080`

    ![Application Voting Java Azure](./media/service-fabric-tutorial-java-deploy-azure/votingappjavaazure.png)

7. Pour désinstaller votre application du cluster, exécutez le script *uninstall.sh* dans le dossier **Scripts**

    ```bash
    ./uninstall.sh
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à :

> [!div class="checklist"]
> * Créer un cluster Linux sécurisé dans Azure
> * Créer les ressources nécessaires pour la surveillance avec ELK

Passez au tutoriel suivant :
> [!div class="nextstepaction"]
> [Monitor your Service Fabric applications using ELK](service-fabric-tutorial-java-elk.md) (Surveiller vos applications Service Fabric à l’aide d’ELK)

---
title: Déployer Consortium Hyperledger Fabric sur Azure Kubernetes Service
description: Guide pratique pour déployer un réseau de consortium Hyperledger Fabric sur Azure Kubernetes Service
ms.date: 08/06/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.openlocfilehash: 081c7a10ee091f573e8f999c94588ef85c784f74
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89651564"
---
# <a name="deploy-hyperledger-fabric-consortium-on-azure-kubernetes-service"></a>Déployer Consortium Hyperledger Fabric sur Azure Kubernetes Service

Vous pouvez utiliser le modèle Hyperledger Fabric sur Azure Kubernetes Service (AKS) pour déployer et configurer un réseau de consortium Hyperledger Fabric sur Azure.

À la fin de cet article, vous serez capable :

- d’utiliser Hyperledger Fabric et les composants d’un réseau blockchain Hyperledger Fabric ;
- de déployer et configurer un réseau de consortium Hyperledger Fabric sur Azure Kubernetes Service pour vos scénarios de production.

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="choose-an-azure-blockchain-solution"></a>Choisir une solution Azure Blockchain

Avant d’opter pour un modèle de solution, comparez votre scénario avec les cas d’utilisation courants des options Azure Blockchain disponibles :

Option | Modèle de service | Cas d’utilisation courant
-------|---------------|-----------------
Modèles de solution | IaaS | Les modèles de solution correspondent à des modèles Azure Resource Manager que vous pouvez utiliser pour provisionner une topologie de réseau blockchain entièrement configurée. Les modèles déploient et configurent les services de calcul, de mise en réseau et de stockage Microsoft Azure pour un type de réseau blockchain. Les modèles de solution sont fournis sans contrat SLA. Rendez-vous sur la [page Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html) pour obtenir de l’aide.
[Azure Blockchain Service](../service/overview.md) | PaaS | Azure Blockchain Service (préversion) simplifie la formation, la gestion et la gouvernance des réseaux blockchain de consortium. Utilisez Azure Blockchain Service pour les solutions nécessitant PaaS, la gestion de consortium ou la confidentialité des contrats et transactions.
[Azure Blockchain Workbench](../workbench/overview.md) | IaaS et PaaS | La préversion d’Azure Blockchain Workbench est une collection de services et de fonctionnalités Azure qui vous aide à créer et à déployer des applications blockchain afin de partager des processus métier et des données avec d’autres organisations. Utilisez Azure Blockchain Workbench pour le prototypage d’une solution blockchain ou d’une preuve de concept pour une application blockchain. Azure Blockchain Workbench est fourni sans contrat de niveau de service (SLA). Rendez-vous sur la [page Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html) pour obtenir de l’aide.

## <a name="hyperledger-fabric-consortium-architecture"></a>Architecture de consortium Hyperledger Fabric

Pour créer un réseau Hyperledger Fabric sur Azure, vous devez déployer un service de tri et une organisation dotée de nœuds homologues. À l’aide du modèle de solution Hyperledger Fabric sur Azure Kubernetes Service, vous pouvez créer des nœuds de tri ou des nœuds homologues. Vous devez déployer le modèle pour chaque nœud à créer.

Les composants fondamentaux créés dans le cadre du déploiement du modèle sont les suivants :

- **Nœuds des auteurs des commandes** : Nœud responsable du tri des transactions dans le registre. Avec les autres nœuds, les nœuds ordonnés constituent le service de classement du réseau Hyperledger Fabric.

- **Nœuds homologues** : Nœud qui héberge principalement des registres et des contrats intelligents, à savoir des éléments fondamentaux du réseau.

- **AC Fabric** : Autorité de certification pour Hyperledger Fabric. L’autorité de certification Fabric vous permet d’initialiser et de démarrer le processus serveur qui héberge l’autorité de certification. Elle vous permet de gérer les identités et les certificats. Chaque cluster AKS déployé dans le cadre du modèle aura un pod d’AC Fabric par défaut.

- **CouchDB ou LevelDB** : Bases de données d’état du monde pour les nœuds homologues. LevelDB est la base de données d’état par défaut incorporée dans le nœud homologue. Elle stocke les données du code chaîné sous forme de paires clé/valeur simples et prend en charge uniquement les requêtes de clé, de plage de clés et de clé composite. CouchDB est une autre base de données d’état facultative qui prend en charge les requêtes enrichies lorsque les valeurs de données du code chaîné sont modélisées au format JSON.

Le modèle de déploiement lance plusieurs ressources Azure dans votre abonnement. Les ressources Azure déployées sont les suivantes :

- **Cluster AKS** : Cluster Azure Kubernetes Service configuré selon les paramètres d’entrée fournis par le client. Le cluster AKS a plusieurs pods configurés pour l’exécution des composants du réseau Hyperledger Fabric. Les pods créés sont les suivants :

  - **Outils Fabric** : Les outils Fabric sont responsables de la configuration des composants Hyperledger Fabric.
  - **Pods homologues/des auteurs des commandes** : Nœuds du réseau Hyperledger Fabric.
  - **Proxy** : Pod proxy NGNIX par le biais duquel les applications clientes peuvent communiquer avec le cluster AKS.
  - **AC Fabric** : Pod qui exécute l’AC Fabric.
- **PostgreSQL** : Instance de base de données qui gère les identités de l’autorité de certification Fabric.

- **Coffre de clés** : Instance du service Azure Key Vault déployée pour enregistrer les informations d’identification de l’autorité de certification Fabric et les certificats racine fournis par le client. Le coffre est utilisé en cas de nouvelle tentative de déploiement du modèle, pour en gérer le mécanisme.
- **Disque managé** : Instance du service Disques managés Azure qui offre un magasin persistant pour le registre et pour la base de données d’état du monde du nœud homologue.
- **Adresse IP publique** : Point de terminaison du cluster AKS déployé pour communiquer avec le cluster.

## <a name="deploy-the-orderer-and-peer-organization"></a>Déployer l’organisation homologue et de l’ordonnanceur

Pour commencer, vous avez besoin d’un abonnement Azure qui peut prendre en charge le déploiement de plusieurs machines virtuelles et de comptes de stockage standard. Si vous n’avez pas d’abonnement Azure, vous pouvez créer un [compte Azure gratuit](https://azure.microsoft.com/free/).

Pour bien démarrer avec le déploiement des composants du réseau Hyperledger Fabric, accédez au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Créer une ressource** > **Blockchain**, puis recherchez **Hyperledger Fabric sur Azure Kubernetes Service (préversion)** .

2. Entrez les détails du projet sous l’onglet **De base**.

    ![Capture d’écran montrant l’onglet De base.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-basics.png)

3. Entrez les informations suivantes :
    - **Abonnement**: Choisissez le nom de l’abonnement dans lequel vous voulez déployer les composants du réseau Hyperledger Fabric.
    - **Groupe de ressources** : Créez un groupe de ressources ou choisissez-en un vide existant. Le groupe de ressources va contenir toutes les ressources déployées dans le cadre du modèle.
    - **Région** : Choisissez la région Azure dans laquelle vous voulez déployer le cluster Azure Kubernetes Service pour les composants Hyperledger Fabric. Le modèle est disponible dans toutes les régions où AKS est disponible. Choisissez une région dans laquelle votre abonnement n’atteint pas la limite du quota de machines virtuelles.
    - **Préfixe de ressource** : Entrez un préfixe servant à nommer les ressources qui sont déployées. Ce préfixe doit comprendre moins de six caractères et la combinaison de ces caractères doit inclure des chiffres et des lettres.
4. Sélectionnez l’onglet **Paramètres Fabric** pour définir les composants du réseau Hyperledger Fabric qui seront déployés.

    ![Capture d’écran montrant l’onglet Paramètres Fabric.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-settings.png)

5. Entrez les informations suivantes :
    - **Nom de l’organisation** : Entrez le nom de l’organisation Hyperledger Fabric, qui est requis pour les différentes opérations de plan de données. Le nom de l’organisation doit être unique par déploiement.
    - **Composant du réseau Fabric** : Choisissez **Service de tri** ou **Nœuds homologues** en fonction du composant de réseau blockchain à configurer.
    - **Nombre de nœuds** : Les deux types de nœuds sont les suivants :
        - **Service de tri** : Sélectionnez le nombre de nœuds pour la tolérance de panne du réseau. Le nombre de nœuds de tri pris en charge s’élève à 3, 5 et 7.
        - **Nœuds homologues** : Vous pouvez choisir 1 à 10 nœuds en fonction de vos besoins.
    - **Base de données d’état universelle de nœuds homologues** : Choisissez entre LevelDB et CouchDB. Ce champ s’affiche quand vous choisissez **Nœuds homologues** dans la liste déroulante **Composant du réseau Fabric**.
    - **Nom d’utilisateur de l’autorité de certification Fabric** : Entrez le nom d’utilisateur qui est utilisé pour l’authentification de l’autorité de certification Fabric.
    - **Mot de passe de l’AC Fabric** : Entrez le mot de passe pour l’authentification de l’AC Fabric.
    - **Confirmer le mot de passe** : Confirmez le mot de passe de l’AC Fabric.
    - **Certificats** : Si vous voulez utiliser vos propres certificats racines pour initialiser l’autorité de certification Fabric, choisissez l’option **Télécharger le certificat racine pour l’autorité de certification Fabric**. Dans le cas contraire, l’autorité de certification Fabric crée des certificats auto-signés par défaut.
    - **Certificat racine** : Chargez le certificat racine (clé publique) avec lequel l’autorité de certification Fabric doit être initialisée. Les certificats au format .pem sont pris en charge. Les certificats doivent être valides et se trouver dans un fuseau horaire UTC.
    - **Clé privée du certificat racine** : Chargez la clé privée du certificat racine. Si vous avez un certificat .pem, qui a une clé publique et privée combinée, chargez-le ici également.


6. Sélectionnez l’onglet **Paramètres du cluster AKS** pour définir la configuration du cluster Azure Kubernetes Service qui est l’infrastructure sous-jacente sur laquelle les composants du réseau Hyperledger Fabric seront installés.

    ![Capture d’écran montrant l’onglet Paramètres du cluster AKS.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/create-for-hyperledger-fabric-aks-cluster-settings-1.png)

7. Entrez les informations suivantes :
    - **Nom du cluster Kubernetes** : Modifiez le nom du cluster AKS, si nécessaire. Ce champ est prérempli en fonction du préfixe de la ressource fournie.
    - **Version de Kubernetes** : Choisissez la version de Kubernetes qui sera déployée sur le cluster. Selon la région que vous avez sélectionnée sous l’onglet **De base**, les versions prises en charge disponibles peuvent changer.
    - **Préfixe DNS** : Entrez un préfixe de nom DNS (Domain Name System) pour le cluster AKS. Vous utiliserez le DNS pour vous connecter à l’API Kubernetes lors de la gestion des conteneurs après la création du cluster.
    - **Taille du nœud** : Pour la taille du nœud Kubernetes, vous pouvez choisir dans la liste des références SKU de machine virtuelle disponibles sur Azure. Pour des performances optimales, nous vous recommandons le Standard DS3 v2.
    - **Nombre de nœuds** : Entrez le nombre de nœuds Kubernetes à déployer dans le cluster. Nous vous recommandons de maintenir ce nombre de nœuds supérieur ou égal au nombre de nœuds Hyperledger Fabric spécifié sous l’onglet **Paramètres Fabric**.
    - **ID de client du principal du service** : Entrez l’ID client d’un principal du service existant ou créez-en un. Un principal du service est obligatoire pour l’authentification AKS. Consultez les [étapes pour créer un principal du service](/powershell/azure/create-azure-service-principal-azureps?view=azps-3.2.0#create-a-service-principal).
    - **Clé secrète client du principal de service** : Entrez la clé secrète client du principal du service fourni dans l’ID client du principal du service.
    - **Confirmer la clé secrète client** : Confirmez la clé secrète client du principal du service.
    - **Activer la supervision de conteneurs** : Choisissez d’activer la supervision AKS, qui permet aux journaux AKS d’envoyer (push) à l’espace de travail Log Analytics spécifié.
    - **Espace de travail Log Analytics** : L’espace de travail Log Analytics est rempli avec l’espace de travail par défaut qui est créé en cas d’activation de la supervision.

8. Sélectionnez l’onglet **Vérifier et créer**. Cette étape déclenche la validation des valeurs que vous avez fournies.
9. Une fois la validation réussie, sélectionnez **Créer**.

    Le déploiement prend généralement entre 10 et 12 minutes. La durée peut varier en fonction de la taille et du nombre des nœuds AKS spécifiés.
10. Une fois le déploiement réussi, vous êtes averti par le biais des notifications Azure dans le coin supérieur droit.
11. Sélectionnez **Accéder au groupe de ressources** pour vérifier toutes les ressources créées dans le cadre du déploiement du modèle. Tous les noms de ressources commencent par le préfixe fourni sous l’onglet **Informations de base**.

## <a name="build-the-consortium"></a>Créer le consortium

Pour créer le consortium blockchain après le déploiement du service de tri et des nœuds homologues, effectuez les étapes suivantes dans l’ordre. Le script Azure Hyperledger Fabric (azhlf) vous aide à configurer le consortium, en créant le canal et en effectuant les opérations de code chaîné.

> [!NOTE]
> Le script Azure Hyperledger Fabric (azhlf) a été mis à jour pour fournir davantage de fonctionnalités. Pour faire référence à l’ancien script, consultez le fichier [Lisez-moi sur GitHub](https://github.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/blob/master/consortiumScripts/README.md). Ce script est compatible avec le modèle Hyperledger Fabric sur Azure Kubernetes Service version 2.0.0 et ultérieure. Pour vérifier la version du déploiement, suivez les étapes décrites dans [Résolution des problèmes](#troubleshoot).

> [!NOTE]
> Le script est fourni pour faciliter les scénarios de démonstration, de développement et de test uniquement. Le canal et le consortium créés par ce script possèdent des stratégies Hyperledger Fabric de base pour simplifier les scénarios de démonstration, développement et test. Pour la configuration en production, nous vous recommandons de mettre à jour les stratégies Hyperledger Fabric de canal/consortium selon les besoins de conformité de votre organisation à l’aide des API Hyperledger Fabric natives.


Toutes les commandes permettant d’exécuter le script Azure Hyperledger Fabric peuvent être exécutées par le biais de l’interface de ligne de commande (CLI) d’Azure Bash. Vous pouvez vous connecter à Azure Cloud Shell par le biais de l’option ![Modèle Hyperledger Fabric sur Azure Kubernetes Service](./media/hyperledger-fabric-consortium-azure-kubernetes-service/arrow.png) disponible en haut à droite du portail Azure. À l’invite de commandes, tapez `bash` et sélectionnez la touche Entrée pour basculer vers l’interface de ligne de commande Bash, ou sélectionnez **Bash** dans la barre d’outils Cloud Shell.

Pour plus d’informations, consultez [Azure Cloud Shell](../../cloud-shell/overview.md).

![Capture d’écran montrant des commandes dans Azure Cloud Shell.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/hyperledger-powershell.png)


L’illustration suivante montre la procédure pas à pas pour créer un consortium entre une organisation de l’ordonnanceur et une organisation homologue. Les sections suivantes présentent des commandes détaillées pour effectuer ces étapes.

![Diagramme du processus de création d’un consortium.](./media/hyperledger-fabric-consortium-azure-kubernetes-service/process-to-build-consortium-flow-chart.png)

Une fois l’installation initiale terminée, utilisez l’application cliente pour accomplir les opérations suivantes :  

- Gestion de canal
- Gestion de consortium
- Gestion de code chaîné

### <a name="download-client-application-files"></a>Télécharger les fichiers d’application cliente

Le premier programme d’installation consiste à télécharger les fichiers de l’application cliente. Exécutez les commandes suivantes pour télécharger tous les fichiers et packages nécessaires :

```bash-interactive
curl https://raw.githubusercontent.com/Azure/Hyperledger-Fabric-on-Azure-Kubernetes-Service/master/azhlfToolSetup.sh | bash
cd azhlfTool
npm install
npm run setup
```

Ces commandes clonent le code d’application cliente Azure Hyperledger Fabric à partir du dépôt GitHub public, puis chargent tous les packages npm dépendants. Une fois l’exécution de la commande terminée, vous pouvez voir un dossier node_modules dans le répertoire actif. Tous les packages requis sont chargés dans le dossier node_modules.

### <a name="set-up-environment-variables"></a>Définir les variables d’environnement

Toutes les variables d’environnement respectent la convention de nommage des ressources Azure.

#### <a name="set-environment-variables-for-the-orderer-organizations-client"></a>Définir des variables d’environnement pour le client de l’organisation de l’ordonnanceur

```bash
ORDERER_ORG_SUBSCRIPTION=<ordererOrgSubscription>
ORDERER_ORG_RESOURCE_GROUP=<ordererOrgResourceGroup>
ORDERER_ORG_NAME=<ordererOrgName>
ORDERER_ADMIN_IDENTITY="admin.$ORDERER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

#### <a name="set-environment-variables-for-the-peer-organizations-client"></a>Définir des variables d’environnement pour le client de l’organisation homologue

```bash
PEER_ORG_SUBSCRIPTION=<peerOrgSubscritpion>
PEER_ORG_RESOURCE_GROUP=<peerOrgResourceGroup>
PEER_ORG_NAME=<peerOrgName>
PEER_ADMIN_IDENTITY="admin.$PEER_ORG_NAME"
CHANNEL_NAME=<channelName>
```

En fonction du nombre d’organisations homologues dans votre consortium, vous devrez peut-être répéter les commandes homologues et définir la variable d’environnement en conséquence.

#### <a name="set-environment-variables-for-an-azure-storage-account"></a>Définir des variables d’environnement pour un compte de stockage Azure

```bash
STORAGE_SUBSCRIPTION=<subscriptionId>
STORAGE_RESOURCE_GROUP=<azureFileShareResourceGroup>
STORAGE_ACCOUNT=<azureStorageAccountName>
STORAGE_LOCATION=<azureStorageAccountLocation>
STORAGE_FILE_SHARE=<azureFileShareName>
```

Utilisez les commandes suivantes pour créer un compte de stockage Azure. Si vous avez déjà un compte de stockage Azure, ignorez cette étape.

```bash
az account set --subscription $STORAGE_SUBSCRIPTION
az group create -l $STORAGE_LOCATION -n $STORAGE_RESOURCE_GROUP
az storage account create -n $STORAGE_ACCOUNT -g  $STORAGE_RESOURCE_GROUP -l $STORAGE_LOCATION --sku Standard_LRS
```

Utilisez les commandes suivantes pour créer un partage de fichiers dans le compte de stockage Azure. Si vous avez déjà un partage de fichiers, ignorez cette étape.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
az storage share create  --account-name $STORAGE_ACCOUNT  --account-key $STORAGE_KEY  --name $STORAGE_FILE_SHARE
```

Utilisez les commandes suivantes pour générer une chaîne de connexion pour un partage de fichiers Azure.

```bash
STORAGE_KEY=$(az storage account keys list --resource-group $STORAGE_RESOURCE_GROUP  --account-name $STORAGE_ACCOUNT --query "[0].value" | tr -d '"')
SAS_TOKEN=$(az storage account generate-sas --account-key $STORAGE_KEY --account-name $STORAGE_ACCOUNT --expiry `date -u -d "1 day" '+%Y-%m-%dT%H:%MZ'` --https-only --permissions lruwd --resource-types sco --services f | tr -d '"')
AZURE_FILE_CONNECTION_STRING=https://$STORAGE_ACCOUNT.file.core.windows.net/$STORAGE_FILE_SHARE?$SAS_TOKEN

```

### <a name="import-an-organization-connection-profile-admin-user-identity-and-msp"></a>Importer un profil de connexion d’organisation, une identité d’utilisateur administrateur et un fournisseur de services managés

Utilisez les commandes suivantes pour extraire le profil de connexion de l’organisation, l’identité de l’utilisateur administrateur et le fournisseur de services managés du cluster Azure Kubernetes Service, puis stockez ces identités dans le magasin local de l’application cliente. Le répertoire *azhlfTool/stores* est un exemple de magasin local.

Pour l’organisation de l’ordonnanceur :

```bash
./azhlf adminProfile import fromAzure -o $ORDERER_ORG_NAME -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION
./azhlf connectionProfile import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME   
./azhlf msp import fromAzure -g $ORDERER_ORG_RESOURCE_GROUP -s $ORDERER_ORG_SUBSCRIPTION -o $ORDERER_ORG_NAME
```

Pour l’organisation homologue :

```bash
./azhlf adminProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf connectionProfile import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
./azhlf msp import fromAzure -g $PEER_ORG_RESOURCE_GROUP -s $PEER_ORG_SUBSCRIPTION -o $PEER_ORG_NAME
```

### <a name="create-a-channel"></a>Créer un canal

À partir du client de l’organisation de l’ordonnanceur, utilisez la commande suivante pour créer un canal qui contient uniquement l’organisation de l’ordonnanceur.  

```bash
./azhlf channel create -c $CHANNEL_NAME -u $ORDERER_ADMIN_IDENTITY -o $ORDERER_ORG_NAME
```

### <a name="add-a-peer-organization-for-consortium-management"></a>Ajouter une organisation homologue pour la gestion de consortium

>[!NOTE]
> Avant de commencer toute opération de consortium, vérifiez que vous avez terminé la configuration initiale de l’application cliente.  

Exécutez les commandes suivantes dans l’ordre indiqué pour ajouter une organisation homologue dans un canal et un consortium : 

1.  À partir du client de l’organisation homologue, chargez le fournisseur de services managés de l’organisation homologue sur le stockage Azure.

      ```bash
      ./azhlf msp export toAzureStorage -f  $AZURE_FILE_CONNECTION_STRING -o $PEER_ORG_NAME
      ```
2.  À partir du client de l’organisation de l’ordonnanceur, téléchargez le fournisseur de services managés de l’organisation homologue depuis le stockage Azure. Ensuite, émettez la commande permettant d’ajouter l’organisation homologue dans le canal et le consortium.

      ```bash
      ./azhlf msp import fromAzureStorage -o $PEER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel join -c  $CHANNEL_NAME -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ./azhlf consortium join -o $ORDERER_ORG_NAME  -u $ORDERER_ADMIN_IDENTITY -p $PEER_ORG_NAME
      ```

3.  À partir du client de l’organisation de l’ordonnanceur, chargez le profil de connexion de l’ordonnanceur sur le stockage Azure afin que l’organisation homologue puisse se connecter aux nœuds de l’ordonnanceur à l’aide de ce profil de connexion.

      ```bash
      ./azhlf connectionProfile  export toAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ```

4.  À partir du client de l’organisation homologue, téléchargez le profil de connexion de l’organisation de l’ordonnanceur depuis le stockage Azure. Exécutez ensuite la commande pour ajouter des nœuds homologues dans le canal.

      ```bash
      ./azhlf connectionProfile  import fromAzureStorage -o $ORDERER_ORG_NAME -f $AZURE_FILE_CONNECTION_STRING
      ./azhlf channel joinPeerNodes -o $PEER_ORG_NAME  -u $PEER_ADMIN_IDENTITY -c $CHANNEL_NAME --ordererOrg $ORDERER_ORG_NAME
      ```

De même, pour ajouter d’autres organisations homologues dans le canal, mettez à jour les variables d’environnement homologues en fonction de l’organisation homologue nécessaire et répétez les étapes 1 à 4.

### <a name="set-anchor-peers"></a>Définir des homologues d’ancrage

À partir du client de l’organisation homologue, exécutez la commande pour définir des homologues d’ancrage pour l’organisation homologue dans le canal spécifié.

>[!NOTE]
> Avant d’exécuter cette commande, vérifiez que l’organisation homologue est ajoutée au canal à l’aide des commandes de gestion de consortium.

```bash
./azhlf channel setAnchorPeers -c $CHANNEL_NAME -p <anchorPeersList> -o $PEER_ORG_NAME -u $PEER_ADMIN_IDENTITY --ordererOrg $ORDERER_ORG_NAME
```

`<anchorPeersList>` est la liste séparée par des espaces des nœuds homologues à définir en tant qu’homologue d’ancrage. Par exemple :

  - Définissez `<anchorPeersList>` sur `"peer1"` si vous souhaitez que seul le nœud peer1 soit homologue d’ancrage.
  - Définissez `<anchorPeersList>` sur `"peer1" "peer3"` si vous souhaitez définir les nœuds peer1 et peer3 en tant qu’homologues d’ancrage.

## <a name="chaincode-management-commands"></a>Commandes de gestion des codes chaînés

>[!NOTE]
> Avant de commencer toute opération de code chaîné, vérifiez que vous avez terminé la configuration initiale de l’application cliente.  

### <a name="set-the-chaincode-specific-environment-variables"></a>Définir les variables d’environnement propres au code chaîné

```bash
# Peer organization name where the chaincode operation will be performed
ORGNAME=<PeerOrgName>
USER_IDENTITY="admin.$ORGNAME"  
# If you are using chaincode_example02 then set CC_NAME=“chaincode_example02”
CC_NAME=<chaincodeName>  
# If you are using chaincode_example02 then set CC_VERSION=“1” for validation
CC_VERSION=<chaincodeVersion>
# Language in which chaincode is written. Supported languages are 'node', 'golang', and 'java'  
# Default value is 'golang'  
CC_LANG=<chaincodeLanguage>  
# CC_PATH contains the path where your chaincode is placed.
# If you are using chaincode_example02 to validate then CC_PATH=“/home/<username>/azhlfTool/samples/chaincode/src/chaincode_example02/go”
CC_PATH=<chaincodePath>  
# Channel on which chaincode will be instantiated/invoked/queried  
CHANNEL_NAME=<channelName>  
```

### <a name="install-chaincode"></a>Installer un code chaîné  

Exécutez la commande suivante pour installer un code chaîné sur l’organisation homologue.  

```bash
./azhlf chaincode install -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -p $CC_PATH -l $CC_LANG -v $CC_VERSION  

```
La commande installe un code chaîné sur tous les nœuds homologues de l’organisation homologue définie dans la variable d’environnement `ORGNAME`. Si votre canal contient au moins deux organisations homologues et que vous voulez installer un code chaîné sur toutes, exécutez cette commande séparément pour chaque organisation homologue.  

Procédez comme suit :  

1.  Définissez `ORGNAME` et `USER_IDENTITY` en fonction de `peerOrg1` et exécutez la commande `./azhlf chaincode install`.  
2.  Définissez `ORGNAME` et `USER_IDENTITY` en fonction de `peerOrg2` et exécutez la commande `./azhlf chaincode install`.  

### <a name="instantiate-chaincode"></a>Instancier un code chaîné  

À partir de l’application cliente homologue, exécutez la commande suivante pour instancier un code chaîné sur le canal.  

```bash
./azhlf chaincode instantiate -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -v $CC_VERSION -c $CHANNEL_NAME -f <instantiateFunc> --args <instantiateFuncArgs>
```

Transmettez le nom de la fonction d’instanciation et la liste d’arguments séparés par des espaces dans `<instantiateFunc>` et `<instantiateFuncArgs>` respectivement. Par exemple, pour instancier le code chaîné chaincode_example02.go, définissez `<instantiateFunc>` sur `init` et `<instantiateFuncArgs>` sur `"a" "2000" "b" "1000"`.

Vous pouvez également transmettre le fichier JSON de configuration de la collection à l’aide de l’indicateur `--collections-config`. Ou bien, définissez les arguments temporaires à l’aide de l’indicateur `-t` lors de l’instanciation d’un code chaîné utilisé pour des transactions privées.

Par exemple :

```bash
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath>
./azhlf chaincode instantiate -c $CHANNEL_NAME -n $CC_NAME -v $CC_VERSION -o $ORGNAME -u $USER_IDENTITY --collections-config <collectionsConfigJSONFilePath> -t <transientArgs>
```

La partie `<collectionConfigJSONFilePath>` correspond au chemin du fichier JSON qui contient les collections définies pour l’instanciation du code chaîné de données privées. Vous trouverez un exemple de fichier JSON de configuration de la collection relatif au répertoire *azhlfTool* dans le chemin suivant : `./samples/chaincode/src/private_marbles/collections_config.json`.
Transmettez `<transientArgs>` en tant que JSON valide dans un format de chaîne. Échappez les caractères spéciaux. Par exemple : `'{\\\"asset\":{\\\"name\\\":\\\"asset1\\\",\\\"price\\\":99}}'`

> [!NOTE]
> Exécutez la commande une seule fois à partir d’une organisation homologue dans le canal. Une fois la transaction envoyée à l’ordonnanceur, ce dernier distribue cette transaction à toutes les organisations homologues dans le canal. Le code chaîné est alors instancié sur tous les nœuds homologues et sur toutes les organisations homologues dans le canal.  

### <a name="invoke-chaincode"></a>Appeler un code chaîné  

À partir du client de l’organisation homologue, exécutez la commande suivante pour appeler la fonction du code chaîné :  

```bash
./azhlf chaincode invoke -o $ORGNAME -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <invokeFunc> -a <invokeFuncArgs>  
```

Transmettez le nom de la fonction d’appel et la liste des arguments séparés par des espaces dans `<invokeFunction>` et `<invokeFuncArgs>` , respectivement. En poursuivant avec l’exemple de code chaîné chaincode_example02.go, pour effectuer l’opération d’appel, définissez  `<invokeFunction>` sur  `invoke` et  `<invokeFuncArgs>` sur `"a" "b" "10"`.  

>[!NOTE]
> Exécutez la commande une seule fois à partir d’une organisation homologue dans le canal. Une fois la transaction envoyée à l’ordonnanceur, ce dernier distribue cette transaction à toutes les organisations homologues dans le canal. L’état du monde est alors mis à jour sur tous les nœuds homologues de toutes les organisations homologues dans le canal.  


### <a name="query-chaincode"></a>Interroger un code chaîné  

Exécutez la commande suivante pour interroger le code chaîné :  

```bash
./azhlf chaincode query -o $ORGNAME -p <endorsingPeers> -u $USER_IDENTITY -n $CC_NAME -c $CHANNEL_NAME -f <queryFunction> -a <queryFuncArgs> 
```
Les homologues d’endossement sont des homologues dans lesquels un code chaîné est installé, qui est appelé pour l’exécution de transactions. Vous devez définir `<endorsingPeers>` de sorte à y inclure les noms de nœuds homologues de l’organisation homologue actuelle. Listez les homologues d’endossement d’une combinaison donnée de code chaîné et de canal en les séparant par des espaces. Par exemple : `-p "peer1" "peer3"`.

Si vous utilisez *azhlfTool* pour installer du code chaîné, transmettez tous les noms de nœuds homologues en tant que valeur à l’argument d’homologue d’endossement. Le code chaîné est installé sur chaque nœud homologue de cette organisation. 

Transmettez le nom de la fonction de requête et la liste des arguments séparés par des espaces dans  `<queryFunction>` et  `<queryFuncArgs>`, respectivement. Là encore, en prenant comme référence le code chaîné chaincode_example02.go, pour interroger la valeur de « a» dans l’état du monde, définissez  `<queryFunction>` sur  `query` et  `<queryArgs>` sur `"a"`.  

## <a name="troubleshoot"></a>Dépanner

Exécutez les commandes suivantes pour rechercher la version de votre déploiement de modèle.

Définissez des variables d’environnement selon le groupe de ressources où le modèle a été déployé.

```bash

SWITCH_TO_AKS_CLUSTER() { az aks get-credentials --resource-group $1 --name $2 --subscription $3; }
AKS_CLUSTER_SUBSCRIPTION=<AKSClusterSubscriptionID>
AKS_CLUSTER_RESOURCE_GROUP=<AKSClusterResourceGroup>
AKS_CLUSTER_NAME=<AKSClusterName>
```
Exécutez la commande suivante pour imprimer la version du modèle.

```bash
SWITCH_TO_AKS_CLUSTER $AKS_CLUSTER_RESOURCE_GROUP $AKS_CLUSTER_NAME $AKS_CLUSTER_SUBSCRIPTION
kubectl describe pod fabric-tools -n tools | grep "Image:" | cut -d ":" -f 3

```

## <a name="support-and-feedback"></a>Support et commentaires

Pour vous tenir informé sur les offres du service Blockchain et obtenir des informations de l’équipe technique d’Azure Blockchain, visitez le [blog Azure Blockchain](https://azure.microsoft.com/blog/topics/blockchain/).

Pour faire des commentaires sur le produit ou suggérer de nouvelles fonctionnalités, postez votre idée ou votez pour une autre idée sur le [forum de commentaires Azure pour Blockchain](https://aka.ms/blockchainuservoice).

### <a name="community-support"></a>Support de la communauté pour les objets blob

Échangez avec les ingénieurs Microsoft et les experts de la communauté Azure Blockchain :

- [Page Microsoft Q&A](/answers/topics/azure-blockchain-workbench.html) 
   
  Le support d’ingénierie pour les modèles blockchain est limité aux problèmes de déploiement.
- [Microsoft Tech Community](https://techcommunity.microsoft.com/t5/Blockchain/bd-p/AzureBlockchain)
- [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-blockchain-workbench)

---
title: 'Tutoriel : Déployer WordPress sur un cluster AKS avec un serveur flexible MySQL à l’aide d’Azure CLI'
description: Découvrez comment créer et déployer rapidement WordPress sur AKS avec Azure Database pour MySQL - Serveur flexible.
ms.service: mysql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 11/25/2020
ms.custom: mvc
ms.openlocfilehash: a02cb30b0f00f732fa0c4ac9319a652ef5cb6fc1
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97657054"
---
# <a name="tutorial-deploy-wordpress-app-on-aks-with-azure-database-for-mysql---flexible-server"></a>Tutoriel : Déployer l’application WordPress sur AKS avec Azure Database pour MySQL - Serveur flexible

Dans ce guide de démarrage rapide, vous déployez une application WordPress sur un cluster AKS (Azure Kubernetes Service) avec Azure Database pour MySQL - Serveur flexible (préversion) à l’aide d’Azure CLI. 
**[AKS](../../aks/intro-kubernetes.md)** est un service Kubernetes managé qui vous permet de déployer et gérer rapidement des clusters. **[Azure Database pour MySQL - Serveur flexible (préversion)](overview.md)** est un service de base de données entièrement managé, conçu pour offrir un contrôle plus précis et une plus grande flexibilité des fonctions de gestion de base de données et des paramètres de configuration. Le service Serveur flexible est actuellement disponible en préversion.

> [!NOTE]
> - La fonctionnalité Serveur flexible Azure Database pour MySQL est actuellement disponible en préversion publique
> - Ce guide de démarrage rapide suppose une compréhension élémentaire des concepts liés à Kubernetes, WordPress et MySQL.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la dernière version d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

> [!NOTE]
> Si vous exécutez les commandes mentionnées dans ce guide de démarrage rapide localement (plutôt que dans Azure Cloud Shell), veillez à le faire en tant qu’administrateur.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Nous allons créer un groupe de ressources, *wordpress-project*, à l’aide de la commande [az group create][az-group-create], à l’emplacement *eastus*.

```azurecli-interactive
az group create --name wordpress-project --location eastus
```

> [!NOTE]
> L’emplacement du groupe de ressources correspond à l’endroit où sont stockées les métadonnées du groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création des ressources.

L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/wordpress-project",
  "location": "eastus",
  "managedBy": null,
  "name": "wordpress-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Créer un cluster AKS

Utilisez la commande [az aks create](/cli/azure/aks?view=azure-cli-latest&preserve-view=true#az-aks-create) pour créer un cluster AKS. L’exemple suivant crée un cluster à un nœud nommé *myAKSCluster*. L’exécution de cette commande prend plusieurs minutes.

```azurecli-interactive
az aks create --resource-group wordpress-project --name wordpresscluster--node-count 1 --generate-ssh-keys
```

Au bout de quelques minutes, la commande se termine et retourne des informations au format JSON sur le cluster.

> [!NOTE]
> Lors de la création d’un cluster AKS, un deuxième groupe de ressources est automatiquement créé pour stocker les ressources AKS. Consultez [Pourquoi deux groupes de ressources sont-ils créés avec AKS ?](../../aks/faq.md#why-are-two-resource-groups-created-with-aks)

## <a name="connect-to-the-cluster"></a>Se connecter au cluster

Pour gérer un cluster Kubernetes, vous utilisez [kubectl](https://kubernetes.io/docs/reference/kubectl/overview/), le client de ligne de commande Kubernetes. Si vous utilisez Azure Cloud Shell, `kubectl` est déjà installé. Pour installer `kubectl` en local, utilisez la commande [az aks install-cli](/cli/azure/aks?view=azure-cli-latest&preserve-view=true#az-aks-install-cli) :

```azurecli-interactive
az aks install-cli
```

Pour configurer `kubectl` afin de vous connecter à votre cluster Kubernetes, exécutez la commande [az aks get-credentials](/cli/azure/aks?view=azure-cli-latest&preserve-view=true#az-aks-get-credentials). Cette commande télécharge les informations d’identification et configure l’interface CLI Kubernetes pour les utiliser.

```azurecli-interactive
az aks get-credentials --resource-group wordpress-project --name wordpresscluster
```

> [!NOTE]
> La commande ci-dessus utilise l’emplacement par défaut pour le [fichier de configuration Kubernetes](https://kubernetes.io/docs/concepts/configuration/organize-cluster-access-kubeconfig/), à savoir `~/.kube/config`. Vous pouvez spécifier un autre emplacement pour votre fichier de configuration Kubernetes à l’aide de *--file*.

Pour vérifier la connexion à votre cluster, utilisez la commande [kubectl get]( https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) pour retourner une liste des nœuds du cluster.

```azurecli-interactive
kubectl get nodes
```

L’exemple de sortie suivant montre le nœud unique créé au cours des étapes précédentes. Vérifiez que l’état du nœud est *Ready* :

```output
NAME                       STATUS   ROLES   AGE     VERSION
aks-nodepool1-31718369-0   Ready    agent   6m44s   v1.12.8
```

## <a name="create-an-azure-database-for-mysql---flexible-server"></a>Créer un serveur flexible Azure Database pour MySQL
Créez un serveur flexible avec la commande [az mysql flexible-server create](/cli/azure/mysql/flexible-server?view=azure-cli-latest&preserve-view=true). La commande suivante crée un serveur en utilisant les valeurs par défaut du service et les valeurs issues du contexte local de votre interface Azure CLI :

```azurecli-interactive
az mysql flexible-server create --public-access <YOUR-IP-ADDRESS>
```

Le serveur créé possède les attributs suivants :
- Une nouvelle base de données vide, ```flexibleserverdb```, est créée au moment du provisionnement initial du serveur. Dans ce guide de démarrage rapide, nous allons utiliser cette base de données.
- Nom du serveur généré automatiquement, nom d’utilisateur administrateur, mot de passe administrateur, nom du groupe de ressources (s’il n’est pas déjà spécifié dans le contexte local) et même emplacement que votre groupe de ressources
- Valeurs par défaut du service pour les configurations restantes : Niveau de calcul (Modulable), Taille de calcul/SKU (B1MS), Période de rétention des sauvegardes (7 jours) et Version de MySQL (5.7)
- L’utilisation de l’argument public-access vous permet de créer un serveur avec un accès public protégé par des règles de pare-feu. En fournissant votre adresse IP pour ajouter la règle de pare-feu afin d’autoriser l’accès à partir de votre machine cliente.
- Étant donné que la commande utilise le contexte local, elle crée le serveur dans le groupe de ressources ```wordpress-project``` et dans la région ```eastus```.


### <a name="build-your-wordpress-docker-image"></a>Créer votre image Docker WordPress

Téléchargez la [dernière version de WordPress](https://wordpress.org/download/). Créer un répertoire ```my-wordpress-app``` pour votre projet et utiliser cette structure de dossiers simple

```
└───my-wordpress-app
    └───public
        ├───wp-admin
        │   ├───css
        . . . . . . .
        ├───wp-content
        │   ├───plugins
        . . . . . . .
        └───wp-includes
        . . . . . . .
        ├───wp-config-sample.php
        ├───index.php
        . . . . . . .
    └─── Dockerfile

```


Renommez ```wp-config-sample.php``` en ```wp-config.php``` et remplacez les lignes 21 à 32 par cet extrait de code. L’extrait de code ci-dessous lit l’hôte de base de données, le nom d’utilisateur et le mot de passe dans le fichier manifeste Kubernetes.

```php
//Using environment variables for DB connection information

// ** MySQL settings - You can get this info from your web host ** //
/** The name of the database for WordPress */

$connectstr_dbhost = getenv('DATABASE_HOST');
$connectstr_dbusername = getenv('DATABASE_USERNAME');
$connectstr_dbpassword = getenv('DATABASE_PASSWORD');

/** MySQL database name */
define('DB_NAME', 'flexibleserverdb');

/** MySQL database username */
define('DB_USER', $connectstr_dbusername);

/** MySQL database password */
define('DB_PASSWORD',$connectstr_dbpassword);

/** MySQL hostname */
define('DB_HOST', $connectstr_dbhost);

/** Database Charset to use in creating database tables. */
define('DB_CHARSET', 'utf8');

/** The Database Collate type. Don't change this if in doubt. */
define('DB_COLLATE', '');


/** SSL*/
define('MYSQL_CLIENT_FLAGS', MYSQLI_CLIENT_SSL);
```

### <a name="create-a-dockerfile"></a>Créer un Dockerfile
Créez un autre Dockerfile et copiez-y cet extrait de code. Ce Dockerfile configure le serveur web Apache avec PHP et active l’extension mysqli.

```docker
FROM php:7.2-apache
COPY public/ /var/www/html/
RUN docker-php-ext-install mysqli
RUN docker-php-ext-enable mysqli
```

### <a name="build-your-docker-image"></a>Générer votre image Docker
Vérifiez que vous êtes dans le répertoire ```my-wordpress-app``` sur un terminal à l’aide de la commande ```cd```. Exécutez la commande suivante pour générer l’image :

``` bash

docker build --tag myblog:latest .

```

Déployez votre image sur [Docker Hub](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) ou [Azure Container Registry](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Si vous utilisez Azure Container Registry (ACR), exécutez la commande ```az aks update``` pour attacher le compte ACR au cluster AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g wordpress-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Créer un fichier manifeste Kubernetes

Un fichier manifeste Kubernetes définit un état souhaité pour le cluster, notamment les images conteneur à exécuter. Nous allons créer un fichier manifeste nommé `mywordpress.yaml`, et y copier la définition YAML suivante.

>[!IMPORTANT]
> - Remplacez ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` par le nom et l’étiquette de votre image Docker WordPress réelle, par exemple ```docker-hub-user/myblog:latest```.
> - Mettez à jour la section ```env``` ci-dessous avec les valeurs ```SERVERNAME```, ```YOUR-DATABASE-USERNAME```, ```YOUR-DATABASE-PASSWORD``` de votre serveur flexible MySQL.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: wordpress-blog
spec:
  replicas: 1
  selector:
    matchLabels:
      app: wordpress-blog
  template:
    metadata:
      labels:
        app: wordpress-blog
    spec:
      containers:
      - name: wordpress-blog
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.mysql.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "flexibleserverdb"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - wordpress-blog
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: php-svc
spec:
  type: LoadBalancer
  ports:
    - port: 80
  selector:
    app: wordpress-blog
```

## <a name="deploy-wordpress-to-aks-cluster"></a>Déployer WordPress sur un cluster AKS
Déployez l’application à l’aide de la commande [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f mywordpress.yaml
```

L’exemple de sortie suivant montre que les déploiements et les ressources ont été créés correctement :

```output
deployment "wordpress-blog" created
service "php-svc" created
```

## <a name="test-the-application"></a>Test de l’application

Quand l’application s’exécute, un service Kubernetes expose le front-end de l’application sur Internet. L’exécution de ce processus peut prendre plusieurs minutes.

Pour surveiller la progression, utilisez la commande [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) avec l’argument `--watch`.

```azurecli-interactive
kubectl get service wordpress-blog --watch
```

Dans un premier temps, la valeur *EXTERNAL-IP* pour le service *wordpress-blog* apparaît comme étant *en attente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
wordpress-blog   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Quand l’adresse *EXTERNAL-IP* passe de l’état *pending* à une adresse IP publique réelle, utilisez `CTRL-C` pour arrêter le processus de surveillance `kubectl`. L’exemple de sortie suivant montre une adresse IP publique valide affectée au service :

```output
wordpress-blog  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

### <a name="browse-wordpress"></a>Parcourir WordPress

Ouvrez un navigateur web à l’adresse IP externe de votre service pour voir la page d’installation de WordPress.

   :::image type="content" source="./media/tutorial-deploy-wordpress-on-aks/wordpress-aks-installed-success.png" alt-text="Installation de WordPress réussie sur AKS avec un serveur flexible MySQL":::

>[!NOTE]
> - Le site WordPress n’utilise pas le protocole HTTPS. Il est recommandé d’[ACTIVER TLS avec vos propres certificats](../../aks/ingress-own-tls.md).
> - Vous pouvez activer le [routage HTTP](../../aks/http-application-routing.md) pour votre cluster.

## <a name="clean-up-the-resources"></a>Nettoyer les ressources

Pour éviter des frais Azure, vous devez nettoyer les ressources inutiles.  Lorsque vous n’avez plus besoin du cluster, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az_group_delete) pour supprimer le groupe de ressources, le service conteneur et toutes les ressources associées.

```azurecli-interactive
az group delete --name wordpress-project --yes --no-wait
```

> [!NOTE]
> Lorsque vous supprimez le cluster, le principal de service Azure Active Directory utilisé par le cluster AKS n’est pas supprimé. Pour obtenir des instructions sur la façon de supprimer le principal de service, consultez [Considérations et suppression du principal de service AKS](../../aks/kubernetes-service-principal.md#additional-considerations). Si vous avez utilisé une identité managée, l’identité est managée par la plateforme et n’a pas besoin d’être supprimée.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [accéder au tableau de bord web Kubernetes](../../aks/kubernetes-dashboard.md) pour votre cluster AKS
- Découvrez comment [mettre votre cluster à l’échelle](../../aks/tutorial-kubernetes-scale.md)
- Découvrez comment gérer votre [serveur flexible MySQL](./quickstart-create-server-cli.md)
- Découvrez comment [configurer les paramètres serveur](./how-to-configure-server-parameters-cli.md) pour votre serveur de base de données.


---
title: 'Tutoriel : Déployer Django sur un cluster AKS avec un serveur flexible PostgreSQL à l’aide d’Azure CLI'
description: Découvrez comment créer et déployer rapidement Django sur AKS avec Azure Database pour PostgreSQL – Serveur flexible.
ms.service: postgresql
author: mksuni
ms.author: sumuth
ms.topic: tutorial
ms.date: 12/10/2020
ms.custom: mvc
ms.openlocfilehash: 15e94b647b0f12baceffc60ad1c6ab81cc84cf22
ms.sourcegitcommit: 8c3a656f82aa6f9c2792a27b02bbaa634786f42d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97631939"
---
# <a name="tutorial-deploy-django-app-on-aks-with-azure-database-for-postgresql---flexible-server"></a>Tutoriel : Déployer l’application Django sur AKS avec Azure Database pour PostgreSQL – Serveur flexible

Dans ce guide de démarrage rapide, vous déployez une application Django sur un cluster AKS (Azure Kubernetes Service) avec Azure Database pour PostgreSQL – Serveur flexible (préversion) à l’aide d’Azure CLI.

**[AKS](../../aks/intro-kubernetes.md)** est un service Kubernetes managé qui vous permet de déployer et gérer rapidement des clusters. **[Azure Database pour PostgreSQL - Serveur flexible (préversion)](overview.md)** est un service de base de données entièrement géré conçu pour offrir un contrôle et une flexibilité plus granulaires des fonctions de gestion de base de données et des paramètres de configuration.

> [!NOTE]
> - Le serveur flexible Azure Database pour PostgreSQL est actuellement en préversion publique.
> - Ce guide de démarrage rapide suppose une compréhension élémentaire des concepts liés à Kubernetes, Django et PostgreSQL.

## <a name="pre-requisites"></a>Conditions préalables
[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

- Utilisez [Azure Cloud Shell](../../cloud-shell/quickstart.md) à l’aide de l’environnement Bash.

   [![Lancement de l’incorporation](https://shell.azure.com/images/launchcloudshell.png "Lancement d’Azure Cloud Shell")](https://shell.azure.com)  
- Si vous préférez, [installez](/cli/azure/install-azure-cli) Azure CLI pour exécuter les commandes de référence de l’interface de ligne de commande.
  - Si vous utilisez une installation locale, connectez-vous à Azure CLI à l’aide de la commande [az login](/cli/azure/reference-index#az-login).  Pour finir le processus d’authentification, suivez les étapes affichées dans votre terminal.  Consultez [Se connecter avec Azure CLI](/cli/azure/authenticate-azure-cli) pour connaître les autres options de connexion.
  - Lorsque vous y êtes invité, installez les extensions Azure CLI lors de la première utilisation.  Pour plus d’informations sur les extensions, consultez [Utiliser des extensions avec Azure CLI](/cli/azure/azure-cli-extensions-overview).
  - Exécutez [az version](/cli/azure/reference-index?#az_version) pour rechercher la version et les bibliothèques dépendantes installées. Pour effectuer une mise à niveau vers la dernière version, exécutez [az upgrade](/cli/azure/reference-index?#az_upgrade). Cet article nécessite la dernière version d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

> [!NOTE]
> Si vous exécutez les commandes mentionnées dans ce guide de démarrage rapide localement (plutôt que dans Azure Cloud Shell), veillez à le faire en tant qu’administrateur.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un groupe de ressources Azure est un groupe logique dans lequel des ressources Azure sont déployées et gérées. Nous allons créer un groupe de ressources, *django-project* à l’aide de la commande [az group create][az-group-create], à l’emplacement *eastus*.

```azurecli-interactive
az group create --name django-project --location eastus
```

> [!NOTE]
> L’emplacement du groupe de ressources correspond à l’endroit où sont stockées les métadonnées du groupe de ressources. C’est également là que vos ressources s’exécutent dans Azure si vous ne spécifiez pas une autre région lors de la création des ressources.

L’exemple de sortie suivant montre que le groupe de ressources a été créé correctement :

```json
{
  "id": "/subscriptions/<guid>/resourceGroups/django-project",
  "location": "eastus",
  "managedBy": null,
  
  "name": "django-project",
  "properties": {
    "provisioningState": "Succeeded"
  },
  "tags": null
}
```

## <a name="create-aks-cluster"></a>Créer un cluster AKS

Utilisez la commande [az aks create](/cli/azure/aks?view=azure-cli-latest&preserve-view=true#az-aks-create) pour créer un cluster AKS. L’exemple suivant crée un cluster à un nœud nommé *myAKSCluster*. L’exécution de cette commande prend plusieurs minutes.

```azurecli-interactive
az aks create --resource-group django-project --name djangoappcluster --node-count 1 --generate-ssh-keys
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
az aks get-credentials --resource-group django-project --name djangoappcluster
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

## <a name="create-an-azure-database-for-postgresql---flexible-server"></a>Créer un serveur flexible Azure Database pour PostgreSQL
Créez un serveur flexible avec la commande [az postgreSQL flexible-server create](/azure/postgresql/flexible-server?view=azure-cli-latest&preserve-view=true). La commande suivante crée un serveur en utilisant les valeurs par défaut du service et les valeurs issues du contexte local de votre interface Azure CLI :

```azurecli-interactive
az postgres flexible-server create --public-access <YOUR-IP-ADDRESS>
```

Le serveur créé possède les attributs suivants :
- Une nouvelle base de données vide, ```postgres```, est créée au moment du provisionnement initial du serveur. Dans ce guide de démarrage rapide, nous allons utiliser cette base de données.
- Nom du serveur généré automatiquement, nom d’utilisateur administrateur, mot de passe administrateur, nom du groupe de ressources (s’il n’est pas déjà spécifié dans le contexte local) et même emplacement que votre groupe de ressources
- Valeurs par défaut du service pour les configurations restantes : Niveau de calcul (Usage général), Taille de calcul/SKU (Standard_D2s_v3 qui utilise 2vCores), Période de rétention des sauvegardes (7 jours) et Version de PostgreSQL (12)
- L’utilisation de l’argument public-access vous permet de créer un serveur avec un accès public protégé par des règles de pare-feu. En fournissant votre adresse IP pour ajouter la règle de pare-feu afin d’autoriser l’accès à partir de votre machine cliente.
- Étant donné que la commande utilise le contexte local, elle crée le serveur dans le groupe de ressources ```django-project``` et dans la région ```eastus```.


## <a name="build-your-django-docker-image"></a>Générer votre image Docker Django

Créez une nouvelle [application Django](https://docs.djangoproject.com/en/3.1/intro/) ou utilisez votre projet Django existant. Assurez-vous que votre code se trouve dans cette structure de dossiers.

```
└───my-djangoapp
    └───views.py
    └───models.py
    └───forms.py
    ├───templates
        . . . . . . .
    ├───static
        . . . . . . .
└───my-django-project
    └───settings.py
    └───urls.py
    └───wsgi.py
        . . . . . . .
    └─── Dockerfile
    └─── requirements.txt
    └─── manage.py
    
```
Mettez à jour ```ALLOWED_HOSTS``` dans ```settings.py``` pour vous assurer que l’application Django utilise l’adresse IP externe qui est assignée à l’application Kubernetes.

```python
ALLOWED_HOSTS = ['*']
```

Mettez à jour la section ```DATABASES={ }``` dans le fichier ```settings.py```. L’extrait de code ci-dessous lit l’hôte de base de données, le nom d’utilisateur et le mot de passe dans le fichier manifeste Kubernetes.

```python
DATABASES={
   'default':{
      'ENGINE':'django.db.backends.postgresql_psycopg2',
      'NAME':os.getenv('DATABASE_NAME'),
      'USER':os.getenv('DATABASE_USER'),
      'PASSWORD':os.getenv('DATABASE_PASSWORD'),
      'HOST':os.getenv('DATABASE_HOST'),
      'PORT':'5432',
      'OPTIONS': {'sslmode': 'require'}
   }
}
```

### <a name="generate-a-requirementstxt-file"></a>Générer un fichier requirements.txt
Créez un fichier ```requirements.txt``` pour répertorier les dépendances pour l’application Django. Voici un exemple de fichier ```requirements.txt```. Vous pouvez utiliser [``` pip freeze > requirements.txt```](https://pip.pypa.io/en/stable/reference/pip_freeze/) pour générer un fichier requirements.txt pour votre application existante.

``` text
Django==2.2.17
postgres==3.0.0
psycopg2-binary==2.8.6
psycopg2-pool==1.1
pytz==2020.4
```

### <a name="create-a-dockerfile"></a>Créer un Dockerfile
Créez un nouveau fichier nommé ```Dockerfile``` et copiez l’extrait de code ci-dessous. Ce fichier Dockerfile est utilisé dans la configuration de Python 3.8 et l’installation de tous les éléments requis listés dans le fichier requirements.txt.

```docker
# Use the official Python image from the Docker Hub
FROM python:3.8.2

# Make a new directory to put our code in.
RUN mkdir /code

# Change the working directory.
WORKDIR /code

# Copy to code folder
COPY . /code/

# Install the requirements.
RUN pip install -r requirements.txt

# Run the application:
CMD python manage.py runserver 0.0.0.0:8000
```

### <a name="build-your-image"></a>Créer votre image
Vérifiez que vous êtes dans le répertoire ```my-django-app``` sur un terminal à l’aide de la commande ```cd```. Exécutez la commande suivante pour générer votre image de tableau d’affichage :

``` bash

docker build --tag myblog:latest .

```

Déployez votre image sur [Docker Hub](https://docs.docker.com/get-started/part3/#create-a-docker-hub-repository-and-push-your-image) ou [Azure Container Registry](../../container-registry/container-registry-get-started-azure-cli.md).

> [!IMPORTANT]
>Si vous utilisez Azure Container Registry (ACR), exécutez la commande ```az aks update``` pour attacher le compte ACR au cluster AKS.
>
>```azurecli-interactive
>az aks update -n myAKSCluster -g django-project --attach-acr <your-acr-name>
> ```
>

## <a name="create-kubernetes-manifest-file"></a>Créer un fichier manifeste Kubernetes

Un fichier manifeste Kubernetes définit un état souhaité pour le cluster, notamment les images conteneur à exécuter. Nous allons créer un fichier manifeste nommé ```djangoapp.yaml```, et y copier la définition YAML suivante.

>[!IMPORTANT]
> - Remplacez ```[DOCKER-HUB-USER/ACR ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]``` par le nom et l’étiquette de votre image Docker Django réelle, par exemple ```docker-hub-user/myblog:latest```.
> - Mettez à jour la section ```env``` ci-dessous avec les valeurs ```SERVERNAME```, ```YOUR-DATABASE-USERNAME``` et ```YOUR-DATABASE-PASSWORD``` de votre serveur flexible Postgres.


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: django-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: django-app
  template:
    metadata:
      labels:
        app: django-app
    spec:
      containers:
      - name: django-app
        image: [DOCKER-HUB-USER-OR-ACR-ACCOUNT]/[YOUR-IMAGE-NAME]:[TAG]
        ports:
        - containerPort: 80
        env:
        - name: DATABASE_HOST
          value: "SERVERNAME.postgres.database.azure.com"
        - name: DATABASE_USERNAME
          value: "YOUR-DATABASE-USERNAME"
        - name: DATABASE_PASSWORD
          value: "YOUR-DATABASE-PASSWORD"
        - name: DATABASE_NAME
          value: "postgres"
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: "app"
                    operator: In
                    values:
                    - django-app
              topologyKey: "kubernetes.io/hostname"
---
apiVersion: v1
kind: Service
metadata:
  name: python-svc
spec:
  type: LoadBalancer
  ports:
    - port: 8000
  selector:
    app: django-app
```

## <a name="deploy-django-to-aks-cluster"></a>Déployer Django sur le cluster AKS
Déployez l’application à l’aide de la commande [kubectl apply](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#apply) et spécifiez le nom de votre manifeste YAML :

```console
kubectl apply -f djangoapp.yaml
```

L’exemple de sortie suivant montre que les déploiements et les ressources ont été créés correctement :

```output
deployment "django-app" created
service "python-svc" created
```

Un déploiement ```django-app``` vous permet de décrire les détails de votre déploiement, tels que les images à utiliser pour l’application, le nombre de pods et la configuration des pods. Un service ```python-svc``` est créé pour exposer l’application via une adresse IP externe.

## <a name="test-the-application"></a>Test de l’application

Quand l’application s’exécute, un service Kubernetes expose le front-end de l’application sur Internet. L’exécution de ce processus peut prendre plusieurs minutes.

Pour surveiller la progression, utilisez la commande [kubectl get service](https://kubernetes.io/docs/reference/generated/kubectl/kubectl-commands#get) avec l’argument `--watch`.

```azurecli-interactive
kubectl get service django-app --watch
```

Dans un premier temps, la valeur *EXTERNAL-IP* pour le service *django-app* apparaît comme étant *en attente*.

```output
NAME               TYPE           CLUSTER-IP   EXTERNAL-IP   PORT(S)        AGE
django-app   LoadBalancer   10.0.37.27   <pending>     80:30572/TCP   6s
```

Quand l’adresse *EXTERNAL-IP* passe de l’état *pending* à une adresse IP publique réelle, utilisez `CTRL-C` pour arrêter le processus de surveillance `kubectl`. L’exemple de sortie suivant montre une adresse IP publique valide affectée au service :

```output
django-app  LoadBalancer   10.0.37.27   52.179.23.131   80:30572/TCP   2m
```

À présent, ouvrez un navigateur web à l’adresse IP externe de votre service pour visualiser l’application Django.  

>[!NOTE]
> - Actuellement, le site Django n’utilise pas le protocole HTTPS. Il est recommandé d’[ACTIVER TLS avec vos propres certificats](../../aks/ingress-own-tls.md).
> - Vous pouvez activer le [routage HTTP](../../aks/http-application-routing.md) pour votre cluster. Quand le routage http est activé, il configure un contrôleur d’entrée dans votre cluster AKS. À mesure que les applications sont déployées, la solution crée également des noms DNS accessibles publiquement pour les points de terminaison d’application.

## <a name="run-database-migrations"></a>Exécuter des migrations de base de données

Pour toute application Django, vous devez exécuter la migration de base de données ou collecter des fichiers statiques. Vous pouvez exécuter ces commandes de l’interpréteur de commandes Django avec ```$ kubectl exec <pod-name> -- [COMMAND]```.  Avant d’exécuter cette commande, vous devez rechercher le nom du pod avec ```kubectl get pods```. 

```bash
$ kubectl get pods
```

Vous verrez un résultat similaire à ceci :
```output
NAME                             READY   STATUS          RESTARTS   AGE
django-app-5d9cd6cd8-l6x4b     1/1     Running              0       2m
```

Une fois que le nom du pod a été trouvé, vous pouvez exécuter des migrations de base de données Django à l’aide de la commande ```$ kubectl exec <pod-name> -- [COMMAND]```. Notez que ```/code/``` est le répertoire de travail du projet défini dans ```Dockerfile``` ci-dessus.

```bash
$ kubectl exec django-app-5d9cd6cd8-l6x4b -- python /code/manage.py migrate
```

La sortie doit ressembler à ceci : 
```output 
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying admin.0003_logentry_add_action_flag_choices... OK
  . . . . . . 
```

Si vous rencontrez des problèmes, exécutez ```kubectl logs <pod-name>``` pour voir l’exception que lève votre application. Si l’application fonctionne correctement, une sortie semblable à celle-ci doit s’afficher lorsque vous exécutez ```kubectl logs```.

```output
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).

You have 17 unapplied migration(s). Your project may not work properly until you apply the migrations for app(s): admin, auth, contenttypes, sessions.
Run 'python manage.py migrate' to apply them.
December 08, 2020 - 23:24:14
Django version 2.2.17, using settings 'django_postgres_app.settings'
Starting development server at http://0.0.0.0:8000/
Quit the server with CONTROL-C.
```

## <a name="clean-up-the-resources"></a>Nettoyer les ressources

Pour éviter des frais Azure, vous devez nettoyer les ressources inutiles.  Lorsque vous n’avez plus besoin du cluster, utilisez la commande [az group delete](/cli/azure/group?view=azure-cli-latest&preserve-view=true#az_group_delete) pour supprimer le groupe de ressources, le service conteneur et toutes les ressources associées.

```azurecli-interactive
az group delete --name django-project --yes --no-wait
```

> [!NOTE]
> Lorsque vous supprimez le cluster, le principal de service Azure Active Directory utilisé par le cluster AKS n’est pas supprimé. Pour obtenir des instructions sur la façon de supprimer le principal de service, consultez [Considérations et suppression du principal de service AKS](../../aks/kubernetes-service-principal.md#additional-considerations). Si vous avez utilisé une identité managée, l’identité est managée par la plateforme et n’a pas besoin d’être supprimée.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [accéder au tableau de bord web Kubernetes](../../aks/kubernetes-dashboard.md) pour votre cluster AKS
- Découvrez comment [activer le déploiement continu](../../aks/deployment-center-launcher.md)
- Découvrez comment [mettre votre cluster à l’échelle](../../aks/tutorial-kubernetes-scale.md)
- Découvrez comment gérer votre [serveur flexible Postgres](./quickstart-create-server-cli.md)
- Découvrez comment [configurer les paramètres serveur](./howto-configure-server-parameters-using-cli.md) pour votre serveur de base de données.

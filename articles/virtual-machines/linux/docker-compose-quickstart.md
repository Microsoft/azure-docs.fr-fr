---
title: Utiliser Docker Compose
description: Procédure d’installation et d’utilisation de Docker et Compose sur des machines virtuelles Linux avec l’interface CLI Azure.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 02/14/2019
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: e9f824a3b7e1e5e482bbc04c448ca4930bf88678
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87500944"
---
# <a name="get-started-with-docker-and-compose-to-define-and-run-a-multi-container-application-in-azure"></a>Prise en main de Docker et Compose pour définir et exécuter une application à conteneurs multiples dans Azure
Avec [Compose](https://github.com/docker/compose), un fichier texte simple vous permet de définir une application composée de plusieurs conteneurs Docker. Vous faites ensuite tourner votre application dans une seule commande qui fait tout pour déployer votre environnement défini. À titre d’exemple, cet article vous explique comment configurer rapidement un blog WordPress avec une base de données SQL MariaDB de type backend sur une machine virtuelle Ubuntu. Vous pouvez également utiliser Compose pour configurer des applications plus complexes.

Cet article a été testé pour la dernière fois le 14/02/2019 à l’aide d’[Azure Cloud Shell](https://shell.azure.com/bash) et de l’[interface de ligne de commande Azure](/cli/azure/install-azure-cli) version 2.0.58.

## <a name="create-docker-host-with-azure-cli"></a>Créer un hôte Docker avec Azure CLI
Installez la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et connectez-vous à un compte Azure avec [az login](/cli/azure/reference-index).

Tout d’abord, créez un groupe de ressources pour votre environnement Docker avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name myDockerGroup --location eastus
```

Créez un fichier nommé *cloud-init.txt* et collez la configuration suivante. Entrez `sensible-editor cloud-init.txt` pour créer le fichier et afficher la liste des éditeurs disponibles. 

```yaml
#include https://get.docker.com
```

Créez maintenant une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). Utilisez le paramètre `--custom-data` à transmettre dans votre fichier de configuration cloud-init. Indiquez le chemin complet vers la configuration *cloud-init.txt* si vous avez enregistré le fichier en dehors de votre répertoire de travail actuel. L’exemple suivant montre la création d’une machine virtuelle nommée *myDockerVM* et ouvre le port 80 au trafic web.

```azurecli-interactive
az vm create \
    --resource-group myDockerGroup \
    --name myDockerVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys \
    --custom-data cloud-init.txt
az vm open-port --port 80 \
    --resource-group myDockerGroup \
    --name myDockerVM
```

Vous devez patienter quelques minutes le temps que la machine virtuelle soit créée, que les packages soient installés et que l’application démarre. Certaines tâches en arrière-plan continuent à s’exécuter une fois que l’interface CLI Azure vous renvoie à l’invite de commandes. Une fois la machine virtuelle créée, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. 

                 

## <a name="install-compose"></a>Installer Compose


SSH pour votre nouvelle machine virtuelle hôte Docker. Fournissez votre propre adresse IP.

```bash
ssh azureuser@10.10.111.11
```

Installez Compose sur la machine virtuelle.

```bash
sudo apt install docker-compose
```


## <a name="create-a-docker-composeyml-configuration-file"></a>Créer un fichier de configuration docker-compose.yml
Créez un fichier de configuration `docker-compose.yml` pour définir les conteneurs Docker s’exécutant sur la machine virtuelle. Le fichier indique l’image à exécuter sur chaque conteneur, les variables d’environnement et les dépendances nécessaires, les ports et les liens entre les conteneurs. Pour plus d’informations sur la syntaxe du fichier yml, consultez [Référence du fichier Compose](https://docs.docker.com/compose/compose-file/).

Créez un fichier *docker-compose.yml*. Utilisez votre éditeur de texte favori pour ajouter d’autres données au fichier. L’exemple suivant crée le fichier avec une invite de commandes pour `sensible-editor` afin de sélectionner un éditeur que vous souhaitez utiliser.

```bash
sensible-editor docker-compose.yml
```

Collez l’exemple suivant dans votre fichier Docker Compose. Cette configuration utilise des images du [registre DockerHub](https://registry.hub.docker.com/_/wordpress/) pour installer WordPress (le système open source de gestion de blogs et de contenu) et une base de données principale associée SQL MariaDB. Entrez votre propre *MYSQL_ROOT_PASSWORD*.

```yml
wordpress:
  image: wordpress
  links:
    - db:mysql
  ports:
    - 80:80

db:
  image: mariadb
  environment:
    MYSQL_ROOT_PASSWORD: <your password>
```

## <a name="start-the-containers-with-compose"></a>Démarrer les conteneurs avec Compose
Dans le même répertoire que votre fichier *docker-compose.yml*, exécutez la commande suivante (selon votre environnement, vous devrez peut-être exécuter `docker-compose` à l’aide de `sudo`) :

```bash
sudo docker-compose up -d
```

Cette commande démarre les conteneurs Docker spécifiés dans *docker-compose.yml*. L’exécution de cette étape peut prendre une minute ou deux. Vous voyez une sortie similaire à ce qui suit :

```
Creating wordpress_db_1...
Creating wordpress_wordpress_1...
...
```


Pour vérifier que les contrôleurs sont en cours d’exécution, tapez `sudo docker-compose ps`. Le résultat suivant doit s’afficher :

```
        Name                       Command               State         Ports
-----------------------------------------------------------------------------------
azureuser_db_1          docker-entrypoint.sh mysqld      Up      3306/tcp
azureuser_wordpress_1   docker-entrypoint.sh apach ...   Up      0.0.0.0:80->80/tcp
```

Vous pouvez maintenant vous connecter directement à WordPress sur la machine virtuelle sur le port 80. Ouvrez un navigateur Web et entrez l’adresse IP ou le nom de votre machine virtuelle. L’écran de démarrage de WordPress, vous permettant de terminer l’installation et de prendre en main l’application, doit maintenant s’afficher.

![Écran d’accueil WordPress](./media/docker-compose-quickstart/wordpressstart.png)

## <a name="next-steps"></a>Étapes suivantes
* Pour plus d’exemples de développement et de déploiement d’applications à plusieurs conteneurs, consultez la [Référence de la ligne de commande Compose](https://docs.docker.com/compose/reference/) et le [Guide d’utilisation](https://docs.docker.com/compose/).
* Utilisez un modèle Microsoft Azure Manager, le vôtre ou un élément de la [communauté](https://azure.microsoft.com/documentation/templates/), pour déployer une machine virtuelle Microsoft Azure avec Docker et une application configurée avec Compose. Par exemple, le modèle [Deploy a WordPress blog with Docker](https://github.com/Azure/azure-quickstart-templates/tree/master/docker-wordpress-mysql) (en anglais) utilise Docker et Compose pour déployer rapidement WordPress avec un serveur principal MySQL sur une machine virtuelle Ubuntu.
* Essayez d’intégrer Docker Compose à un cluster Docker Swarm. Pour examiner des scénarios, consultez [Utilisation de Compose avec Swarm](https://docs.docker.com/compose/swarm/) .

---
title: Déployer Elasticsearch sur une machine virtuelle de développement dans Azure
description: 'Didacticiel : installer Elastic Stack sur une machine virtuelle Linux de développement dans Azure'
services: virtual-machines-linux
documentationcenter: virtual-machines
author: rloutlaw
manager: justhe
tags: azure-resource-manager
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 10/11/2017
ms.author: routlaw
ms.openlocfilehash: 54ef051b7d8778e2eecd85bef2e57b62239ba114
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435312"
---
# <a name="install-the-elastic-stack-on-an-azure-vm"></a>Installer Elastic Stack sur une machine virtuelle Azure

Cet article vous guide dans le déploiement d’[Elasticsearch](https://www.elastic.co/products/elasticsearch), de [Logstash](https://www.elastic.co/products/logstash) et de [Kibana](https://www.elastic.co/products/kibana) sur une machine virtuelle Ubuntu dans Azure. Pour voir Elastic Stack en action, vous pouvez éventuellement vous connecter à Kibana et travailler avec des exemples de données de journalisation. 

Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Créer une machine virtuelle Ubuntu dans un groupe de ressources Azure
> * Installer Elasticsearch, Logstash et Kibana sur la machine virtuelle
> * Envoyer des exemples de données à Elasticsearch avec Logstash 
> * Ouvrir des ports et manipuler des données dans la console Kibana


 Ce déploiement convient pour le développement de base avec Elastic Stack. Pour plus d’informations sur Elastic Stack, notamment des recommandations pour un environnement de production, voir la [documentation d’Elastic](https://www.elastic.co/guide/index.html) et [Azure Architecture Center](/azure/architecture/elasticsearch/).

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.4 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. 

L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

## <a name="create-a-virtual-machine"></a>Création d'une machine virtuelle

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm). 

L’exemple suivant crée une machine virtuelle nommée *myVM* et des clés SSH si elles n’existent pas déjà dans un emplacement de clé par défaut. Pour utiliser un ensemble spécifique de clés, utilisez l’option `--ssh-key-value`.  

```azurecli-interactive
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image UbuntuLTS \
    --admin-username azureuser \
    --generate-ssh-keys
```

Lorsque la machine virtuelle a été créée, l’interface de ligne de commande Azure affiche des informations similaires à l’exemple suivant. Notez la valeur de `publicIpAddress`. Cette adresse est utilisée pour accéder à la machine virtuelle.

```output
{
  "fqdns": "",
  "id": "/subscriptions/<subscription ID>/resourceGroups/myResourceGroup/providers/Microsoft.Compute/virtualMachines/myVM",
  "location": "eastus",
  "macAddress": "00-0D-3A-23-9A-49",
  "powerState": "VM running",
  "privateIpAddress": "10.0.0.4",
  "publicIpAddress": "40.68.254.142",
  "resourceGroup": "myResourceGroup"
}
```

## <a name="ssh-into-your-vm"></a>Se connecter avec SSH à votre machine virtuelle

Pour trouver l’adresse IP publique de votre machine virtuelle, exécutez la commande [az network public-ip list](/cli/azure/network/public-ip) :

```azurecli-interactive
az network public-ip list --resource-group myResourceGroup --query [].ipAddress
```

Utilisez la commande suivante pour créer une session SSH avec la machine virtuelle. Remplacez l’adresse IP publique correcte de votre machine virtuelle. Dans cet exemple, l’adresse IP est *40.68.254.142*.

```bash
ssh azureuser@40.68.254.142
```

## <a name="install-the-elastic-stack"></a>Installer Elastic Stack

Importez la clé de signature d’Elasticsearch et mettez à jour votre liste de sources APT pour inclure le référentiel de packages élastique :

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -
echo "deb https://artifacts.elastic.co/packages/5.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-5.x.list
```

Installez Java sur la machine virtuelle, puis configurez la variable JAVA_HOME. Cela est nécessaire pour l’exécution des composants Elastic Stack.

```bash
sudo apt update && sudo apt install openjdk-8-jre-headless
export JAVA_HOME=/usr/lib/jvm/java-8-openjdk-amd64
```

Exécutez les commandes suivantes pour mettre à jour les sources de package Ubuntu et installer Elasticsearch, Kibana et Logstash.

```bash
sudo apt update && sudo apt install elasticsearch kibana logstash   
```

> [!NOTE]
> Des instructions d’installation détaillées, notamment concernant les dispositions de répertoire et la configuration initiale, figurent dans la [documentation d’Elastic](https://www.elastic.co/guide/en/elastic-stack/current/installing-elastic-stack.html).

## <a name="start-elasticsearch"></a>Démarrer Elasticsearch 

Démarrez Elasticsearch sur votre machine virtuelle avec la commande suivante :

```bash
sudo systemctl start elasticsearch.service
```

Cette commande ne générant aucune sortie, vérifiez qu’Elasticsearch est bien en cours d’exécution sur la machine virtuelle en utilisant la commande `curl` suivante :

```bash
sudo curl -XGET 'localhost:9200/'
```

Si Elasticsearch est en cours d’exécution, vous voyez une sortie telle que celle-ci :

```json
{
  "name" : "w6Z4NwR",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "SDzCajBoSK2EkXmHvJVaDQ",
  "version" : {
    "number" : "5.6.3",
    "build_hash" : "1a2f265",
    "build_date" : "2017-10-06T20:33:39.012Z",
    "build_snapshot" : false,
    "lucene_version" : "6.6.1"
  },
  "tagline" : "You Know, for Search"
}
```

## <a name="start-logstash-and-add-data-to-elasticsearch"></a>Démarrer Logstash et ajouter des données à Elasticsearch

Démarrez Logstash avec la commande suivante :

```bash 
sudo systemctl start logstash.service
```

Testez Logstash en mode interactif pour vérifier qu’il fonctionne correctement :

```bash
sudo /usr/share/logstash/bin/logstash -e 'input { stdin { } } output { stdout {} }'
```

Voici un [pipeline](https://www.elastic.co/guide/en/logstash/5.6/pipeline.html) Logstash de base, qui renvoie une entrée standard en écho à la sortie standard. 

```output
The stdin plugin is now waiting for input:
hello azure
2017-10-11T20:01:08.904Z myVM hello azure
```

Configurez Logstash pour transférer les messages du noyau de cet machine virtuelle vers Elasticsearch. Créez un fichier dans un répertoire vide nommé `vm-syslog-logstash.conf`, puis collez-y la configuration de Logstash suivante :

```Logstash
input {
    stdin {
        type => "stdin-type"
    }

    file {
        type => "syslog"
        path => [ "/var/log/*.log", "/var/log/*/*.log", "/var/log/messages", "/var/log/syslog" ]
        start_position => "beginning"
    }
}

output {

    stdout {
        codec => rubydebug
    }
    elasticsearch {
        hosts  => "localhost:9200"
    }
}
```

Testez cette configuration et envoyez les données syslog à Elasticsearch :

```bash
sudo /usr/share/logstash/bin/logstash -f vm-syslog-logstash.conf
```

Votre terminal affiche un écho des entrées syslog à mesure que celles-ci sont envoyées à Elasticsearch. Utilisez `CTRL+C` pour quitter Logstash après avoir envoyé des données.

## <a name="start-kibana-and-visualize-the-data-in-elasticsearch"></a>Démarrer Kibana et visualiser les données dans Elasticsearch

Modifiez `/etc/kibana/kibana.yml`, puis modifiez l’adresse IP que Kibana écoute de façon à pouvoir y accéder à partir de votre navigateur web.

```bash
server.host: "0.0.0.0"
```

Démarrez Kibana avec la commande suivante :

```bash
sudo systemctl start kibana.service
```

Ouvrez le port 5601 à partir d’Azure CLI pour autoriser l’accès à distance à la console Kibana :

```azurecli-interactive
az vm open-port --port 5601 --resource-group myResourceGroup --name myVM
```

Ouvrez la console Kibana, puis sélectionnez **Créer** pour générer un index par défaut basé sur les données syslog que vous avez envoyées à Elasticsearch. 

![Capture d’écran qui montre la console Kibana et met en évidence le bouton Créer.](media/elasticsearch-install/kibana-index.png)

Sur la console Kibana, sélectionnez **Découvrir** pour rechercher, parcourir et filtrer les événements syslog.

![Parcourir les événements Syslog dans Kibana](media/elasticsearch-install/kibana-search-filter.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez déployé Elastic Stack sur une machine virtuelle de développement dans Azure. Vous avez appris à :

> [!div class="checklist"]
> * Créer une machine virtuelle Ubuntu dans un groupe de ressources Azure
> * Installer Elasticsearch, Logstash et Kibana sur la machine virtuelle
> * Envoyer des exemples de données à Elasticsearch avec Logstash 
> * Ouvrir des ports et manipuler des données dans la console Kibana

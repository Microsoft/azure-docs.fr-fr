---
title: Gérer des journaux de flux à l’aide de Grafana
titleSuffix: Azure Network Watcher
description: Gérez et analysez les journaux de flux des groupes de sécurité réseau à l’aide de Network Watcher et de Grafana.
services: network-watcher
documentationcenter: na
author: damendo
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/15/2017
ms.author: damendo
ms.openlocfilehash: d522d305c70214009b8aa2886d07d2d5403dd2b1
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656306"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-using-network-watcher-and-grafana"></a>Gérer les journaux de flux des groupes de sécurité réseau avec Network Watcher et Grafana

Les [journaux de flux des groupes de sécurité réseau (NSG)](network-watcher-nsg-flow-logging-overview.md) fournissent des informations permettant de comprendre le trafic IP entrant et sortant des interfaces réseau. Ces journaux de flux affichent les flux entrants et sortants en fonction de la règle NSG, de la carte réseau à laquelle le flux s’applique, des informations relatives aux 5 tuples du flux (adresse IP source/de destination, port source/de destination, protocole), et de l’autorisation ou du refus du trafic.

Un réseau peut comprendre plusieurs groupes de sécurité réseau pour lesquels la journalisation du flux est activée. Une telle quantité de données de journalisation est difficile à analyser et empêche la bonne compréhension des journaux d’activité. Cet article fournit une solution pour gérer de manière centralisée les journaux de flux des groupes de sécurité réseau à l’aide de Grafana, un outil de création de graphes open source, à l’aide d’ElasticSearch, qui est un moteur de recherche et d’analytique distribué, et à l’aide de Logstash, qui est un pipeline open source de traitement des données côté serveur.  

## <a name="scenario"></a>Scénario

Les journaux de flux des groupes de sécurité réseau peuvent être activés avec Network Watcher et sont stockés dans le stockage Blob Azure. Un plug-in Logstash est utilisé pour connecter et traiter les journaux de flux à partir du stockage Blob et les envoyer à ElasticSearch.  Une fois que les journaux de flux sont stockés dans ElasticSearch, il peuvent être analysés et visualisés dans les tableaux de bord personnalisés de Grafana.

![Groupes de sécurité réseau, Network Watcher, Grafana](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig1.png)

## <a name="installation-steps"></a>Procédure d’installation :

### <a name="enable-network-security-group-flow-logging"></a>Activer les journaux des flux de groupe de sécurité réseau

Pour ce scénario, la journalisation des flux de groupe de sécurité réseau doit être activée sur au moins un groupe de sécurité réseau dans votre compte. Pour obtenir des instructions sur l’activation des journaux d’activité des flux de groupe de sécurité réseau, consultez l’article suivant [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Introduction à la journalisation des flux pour les groupes de sécurité réseau).

### <a name="setup-considerations"></a>Considérations relatives à la configuration

Dans cet exemple, Grafana, ElasticSearch et Logstash sont configurés sur un serveur Ubuntu 16.04 LTS déployé dans Azure. Cette configuration minimale est utilisée pour exécuter les trois composants, qui sont tous exécutés sur la même machine virtuelle. Cette configuration doit uniquement être utilisée à des fins de test et pour ses charges de travail non critiques. Logstash, Elasticsearch et Grafana peuvent tous être configurés de manière à pouvoir être mis à l’échelle de manière indépendante sur plusieurs instances. Pour plus d’informations, consultez la documentation relative à chacun de ces composants.

### <a name="install-logstash"></a>Installer Logstash

Logstash vous permet d’aplatir les journaux de flux au format JSON à un niveau de tuple de flux.

1. Pour installer Logstash, exécutez les commandes suivantes :

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```

2. Configurez Logstash pour analyser les journaux de flux et les envoyer à ElasticSearch. Créez un fichier logstash.conf à l’aide de ce qui suit :

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. Ajoutez le contenu suivant au fichier. Ajoutez le nom du compte de stockage et la clé d’accès correspondant à votre compte de stockage :

   ```bash
    input {
      azureblob
      {
        storage_account_name => "mystorageaccount"
        storage_access_key => "VGhpcyBpcyBhIGZha2Uga2V5Lg=="
        container => "insights-logs-networksecuritygroupflowevent"
        codec => "json"
        # Refer https://docs.microsoft.com/azure/network-watcher/network-watcher-read-nsg-flow-logs
        # Typical numbers could be 21/9 or 12/2 depends on the nsg log file types
        file_head_bytes => 12
        file_tail_bytes => 2
        # Enable / tweak these settings when event is too big for codec to handle.
        # break_json_down_policy => "with_head_tail"
        # break_json_batch_count => 2
      }
    }
    filter {
      split { field => "[records]" }
      split { field => "[records][properties][flows]"}
      split { field => "[records][properties][flows][flows]"}
      split { field => "[records][properties][flows][flows][flowTuples]"}

      mutate {
        split => { "[records][resourceId]" => "/"}
        add_field => { "Subscription" => "%{[records][resourceId][2]}"
          "ResourceGroup" => "%{[records][resourceId][4]}"
          "NetworkSecurityGroup" => "%{[records][resourceId][8]}" 
        }
        convert => {"Subscription" => "string"}
        convert => {"ResourceGroup" => "string"}
        convert => {"NetworkSecurityGroup" => "string"}
        split => { "[records][properties][flows][flows][flowTuples]" => "," }
        add_field => {
          "unixtimestamp" => "%{[records][properties][flows][flows][flowTuples][0]}"
          "srcIp" => "%{[records][properties][flows][flows][flowTuples][1]}"
          "destIp" => "%{[records][properties][flows][flows][flowTuples][2]}"
          "srcPort" => "%{[records][properties][flows][flows][flowTuples][3]}"
          "destPort" => "%{[records][properties][flows][flows][flowTuples][4]}"
          "protocol" => "%{[records][properties][flows][flows][flowTuples][5]}"
          "trafficflow" => "%{[records][properties][flows][flows][flowTuples][6]}"
          "traffic" => "%{[records][properties][flows][flows][flowTuples][7]}"
    "flowstate" => "%{[records][properties][flows][flows][flowTuples][8]}"
    "packetsSourceToDest" => "%{[records][properties][flows][flows][flowTuples][9]}"
    "bytesSentSourceToDest" => "%{[records][properties][flows][flows][flowTuples][10]}"
    "packetsDestToSource" => "%{[records][properties][flows][flows][flowTuples][11]}"
    "bytesSentDestToSource" => "%{[records][properties][flows][flows][flowTuples][12]}"
        }
        add_field => {
          "time" => "%{[records][time]}"
          "systemId" => "%{[records][systemId]}"
          "category" => "%{[records][category]}"
          "resourceId" => "%{[records][resourceId]}"
          "operationName" => "%{[records][operationName}}"
          "Version" => "%{[records][properties][Version}}"
          "rule" => "%{[records][properties][flows][rule]}"
          "mac" => "%{[records][properties][flows][flows][mac]}"
        }
        convert => {"unixtimestamp" => "integer"}
        convert => {"srcPort" => "integer"}
        convert => {"destPort" => "integer"}
        add_field => { "message" => "%{Message}" }        
      }
 
      date {
        match => ["unixtimestamp" , "UNIX"]
      }
    }
    output {
      stdout { codec => rubydebug }
      elasticsearch {
        hosts => "localhost"
        index => "nsg-flow-logs"
      }
    }
   ```

Le fichier de configuration Logstash fourni comporte trois parties : l’entrée, le filtre et la sortie.
La section d’entrée désigne la source d’entrée des journaux d’activité que Logstash va traiter. Dans ce cas, nous allons utiliser un plug-in d’entrée « azureblob » (installé dans les étapes suivantes) qui nous permettra d’accéder aux fichiers JSON de journalisation du flux des groupes de sécurité réseau qui sont stockés dans le stockage Blob. 

La section de filtre aplatit ensuite chaque fichier journal du flux pour que chaque tuple de flux et les propriétés qui lui sont associées deviennent un événement Logstash à part entière.

Enfin, la section de sortie transfère chaque événement Logstash vers le serveur ElasticSearch. N’hésitez pas à modifier le fichier de configuration Logstash pour l’adapter à vos besoins.

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>Installer le plug-in d’entrée Logstash pour le stockage Blob Azure

Ce plug-in Logstash vous permet d’accéder directement aux journaux de flux à partir de leur compte de stockage Blob désigné. Pour installer ce plug-in, exécutez la commande suivante à partir du répertoire d’installation par défaut de Logstash (dans ce cas, /usr/share/logstash/bin) :

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

Pour plus d’informations sur ce plug-in, consultez [Plug-in d’entrée Logstash pour Azure Storage Blob](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="install-elasticsearch"></a>Installer Elasticsearch

Vous pouvez utiliser le script suivant pour installer ElasticSearch. Pour plus d’informations sur l’installation d’ElasticSearch, consultez [Elastic Stack](https://www.elastic.co/guide/en/elastic-stack/current/index.html).

```bash
apt-get install apt-transport-https openjdk-8-jre-headless uuid-runtime pwgen -y
wget -qO - https://packages.elastic.co/GPG-KEY-elasticsearch | apt-key add -
echo "deb https://packages.elastic.co/elasticsearch/5.x/debian stable main" | tee -a /etc/apt/sources.list.d/elasticsearch-5.x.list
apt-get update && apt-get install elasticsearch
sed -i s/#cluster.name:.*/cluster.name:\ grafana/ /etc/elasticsearch/elasticsearch.yml
systemctl daemon-reload
systemctl enable elasticsearch.service
systemctl start elasticsearch.service
```

### <a name="install-grafana"></a>Installer Grafana

Pour installer et exécuter Grafana, exécutez les commandes suivantes :

```bash
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.5.1_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_4.5.1_amd64.deb
sudo service grafana-server start
```

Pour plus d’informations sur l’installation, consultez [Installing on Debian/Ubuntu](https://docs.grafana.org/installation/debian/) (Installation sur Debian/Ubuntu).

#### <a name="add-the-elasticsearch-server-as-a-data-source"></a>Ajouter le serveur ElasticSearch en tant que source de données

Ensuite, vous devez ajouter l’index ElasticSearch qui contient les journaux de flux en tant que source de données. Vous pouvez ajouter une source de données en sélectionnant **Ajouter une source de données** et en remplissant le formulaire. Vous trouverez un exemple de cette configuration dans la capture d’écran suivante :

![Ajouter une source de données](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig2.png)

#### <a name="create-a-dashboard"></a>Création d’un tableau de bord

Maintenant que vous avez correctement configuré Grafana pour lire l’index ElasticSearch qui contient les journaux de flux des groupes de sécurité réseau, vous pouvez créer et personnaliser des tableaux de bord. Pour créer un tableau de bord, sélectionnez **Créer votre premier tableau de bord**. L’exemple suivant de configuration de graphe montre les flux segmentés par une règle de groupe de sécurité réseau :

![Graphe du tableau de bord](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig3.png)

La capture d’écran suivante montre un graphe et un graphique contenant les flux principaux et leur fréquence. Les flux peuvent également être affichés par règle de groupe de sécurité réseau et par décision. Grafana est hautement personnalisable. Il est donc recommandé de créer des tableaux de bord adaptés à vos besoins de surveillance. L’exemple ci-dessous montre un tableau de bord classique :

![Capture d'écran présentant l'exemple de configuration de graphe, avec les flux segmentés par une règle de groupe de sécurité réseau.](./media/network-watcher-nsg-grafana/network-watcher-nsg-grafana-fig4.png)

## <a name="conclusion"></a>Conclusion

L’intégration de Network Watcher à ElasticSearch et à Grafana vous permet de visualiser et de gérer de manière centralisée les journaux de flux des groupes de sécurité réseau, ainsi que d’autres données. Grafana comprend d’autres fonctionnalités intéressantes de création de graphes qui peuvent également être utilisées pour gérer les journaux de flux et mieux comprendre votre trafic réseau. Maintenant qu’une instance Grafana est configurée et connectée à Azure, vous pouvez continuer à explorer les autres fonctionnalités.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur l’utilisation de [Network Watcher](network-watcher-monitoring-overview.md).


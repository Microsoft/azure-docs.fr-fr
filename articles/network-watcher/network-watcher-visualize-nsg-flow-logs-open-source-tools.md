---
title: Visualiser les journaux de flux de groupe de sécurité réseau - Pile élastique
titleSuffix: Azure Network Watcher
description: Gérez et analysez les journaux de flux des groupes de sécurité réseau dans Azure avec Network Watcher et la pile élastique.
services: network-watcher
documentationcenter: na
author: damendo
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: aca8c75f262e472cbc770c052b86d6e760ee449a
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026469"
---
# <a name="visualize-azure-network-watcher-nsg-flow-logs-using-open-source-tools"></a>Visualiser les journaux de flux NSG d’Azure Network Watcher à l’aide d’outils open source

Les journaux d’activité des flux de groupe de sécurité réseau fournissent des informations permettant de comprendre le trafic IP entrant et sortant sur les groupes de sécurité réseau. Ces journaux de flux affichent les flux entrants et sortants en fonction de la règle, de la carte réseau à laquelle le flux s’applique, des informations à 5 tuples sur le flux (adresse IP source/de destination, port source/de destination, protocole), et de l’autorisation ou du refus du trafic.

Ces journaux de flux peuvent être difficiles à analyser et à exploiter manuellement. Toutefois, il existe de nombreux outils open source qui peuvent aider à visualiser ces données. Cet article propose une solution pour visualiser ces journaux d’activité à l’aide de la Suite Elastic, qui vous permet d’indexer et de visualiser rapidement vos journaux de flux dans un tableau de bord Kibana.

## <a name="scenario"></a>Scénario

Dans cet article, nous allons configurer une solution qui vous permet de visualiser les journaux d’activité des flux de groupe de sécurité réseau à l’aide de la Suite Elastic.  Un plug-in d’entrée Logstash collecte les journaux de flux directement à partir de l’objet blob de stockage configuré pour contenir les journaux de flux. Ensuite, à l’aide de la Suite Elastic, les journaux de flux sont indexés et utilisés pour créer un tableau de bord Kibana permettant de visualiser les informations.

![Le diagramme illustre un scénario qui vous permet de visualiser les journaux de distribution de groupes de sécurité réseau à l’aide de la Suite Elastic.][scenario]

## <a name="steps"></a>Étapes

### <a name="enable-network-security-group-flow-logging"></a>Activer les journaux des flux de groupe de sécurité réseau
Pour ce scénario, la journalisation des flux de groupe de sécurité réseau doit être activée sur au moins un groupe de sécurité réseau dans votre compte. Pour obtenir des instructions sur l’activation des journaux d’activité des flux de groupe de sécurité réseau, consultez l’article suivant [Introduction to flow logging for Network Security Groups](network-watcher-nsg-flow-logging-overview.md) (Introduction à la journalisation des flux pour les groupes de sécurité réseau).

### <a name="set-up-the-elastic-stack"></a>Configurer la Suite Elastic
En connectant les journaux de flux NSG à la Suite Elastic, nous pouvons générer un tableau de bord Kibana qui nous permet d’effectuer des recherches, de créer des graphiques, d’analyser et d’obtenir des informations à partir de nos journaux d’activité.

#### <a name="install-elasticsearch"></a>Installer Elasticsearch

1. La Suite Elastic à partir de la version 5.0 et pour les versions ultérieures requiert Java 8. Exécutez la commande `java -version` pour vérifier la version que vous utilisez. Si Java n’est pas installé sur votre ordinateur, reportez-vous à la documentation relative aux [JDK pris en charge dans Azure](/azure/developer/java/fundamentals/java-jdk-long-term-support).
2. Téléchargez le package binaire approprié pour votre système :

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
   sudo dpkg -i elasticsearch-5.2.0.deb
   sudo /etc/init.d/elasticsearch start
   ```

   D’autres méthodes d’installation se trouvent sur la page [Elasticsearch Installation](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html) (Installation d’Elasticsearch).

3. Vérifiez qu’Elasticsearch est en cours d’exécution avec la commande :

    ```bash
    curl http://127.0.0.1:9200
    ```

    Une réponse similaire à celle ci-dessous devrait s’afficher :

    ```json
    {
    "name" : "Angela Del Toro",
    "cluster_name" : "elasticsearch",
    "version" : {
        "number" : "5.2.0",
        "build_hash" : "8ff36d139e16f8720f2947ef62c8167a888992fe",
        "build_timestamp" : "2016-01-27T13:32:39Z",
        "build_snapshot" : false,
        "lucene_version" : "6.1.0"
    },
    "tagline" : "You Know, for Search"
    }
    ```

Pour plus d’informations sur l’installation d’Elasticsearch, reportez-vous aux [instructions d’installation](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html).

### <a name="install-logstash"></a>Installer Logstash

1. Exécutez les commandes suivantes pour installer Logstash :

    ```bash
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
2. Ensuite, nous devons configurer Logstash pour accéder aux journaux de flux et les analyser. Créez un fichier logstash.conf à l’aide de :

    ```bash
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

3. Ajoutez le contenu suivant au fichier :

   ```
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

     mutate{
      split => { "[records][resourceId]" => "/"}
      add_field => {"Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"}
      convert => {"Subscription" => "string"}
      convert => {"ResourceGroup" => "string"}
      convert => {"NetworkSecurityGroup" => "string"}
      split => { "[records][properties][flows][flows][flowTuples]" => ","}
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
      convert => {"unixtimestamp" => "integer"}
      convert => {"srcPort" => "integer"}
      convert => {"destPort" => "integer"}        
     }

     date{
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

Pour plus d’informations sur l’installation de Logstash, reportez-vous à la [documentation officielle](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html).

### <a name="install-the-logstash-input-plugin-for-azure-blob-storage"></a>Installer le plug-in d’entrée Logstash pour le stockage Blob Azure

Ce plug-in Logstash vous permet d’accéder directement aux journaux de flux à partir de leur compte de stockage désigné. Pour installer ce plug-in, exécutez la commande suivante à partir du répertoire d’installation par défaut de Logstash (dans ce cas, /usr/share/logstash/bin) :

```bash
logstash-plugin install logstash-input-azureblob
```

Pour démarrer Logstash, exécutez la commande :

```bash
sudo /etc/init.d/logstash start
```

Pour plus d’informations sur ce plug-in, reportez-vous à la [documentation](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="install-kibana"></a>Installer Kibana

1. Exécutez les commandes suivantes pour installer Kibana :

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
   tar xzvf kibana-5.2.0-linux-x86_64.tar.gz
   ```

2. Pour exécuter Kibana, exécutez les commandes ci-dessous :

   ```bash
   cd kibana-5.2.0-linux-x86_64/
   ./bin/kibana
   ```

3. Pour afficher votre interface web Kibana, accédez à `http://localhost:5601`
4. Dans ce scénario, le modèle d’index utilisé pour les journaux de flux est « nsg-flow-logs ». Vous pouvez modifier le modèle d’index dans la section « sortie » de votre fichier logstash.conf.
5. Si vous souhaitez afficher le tableau de bord Kibana à distance, créez une règle de groupe de sécurité réseau entrante autorisant l’accès au **port 5601**.

### <a name="create-a-kibana-dashboard"></a>Créer un tableau de bord Kibana

L’image suivante montre un exemple de tableau de bord où sont affichés les tendances et les détails de vos alertes :

![figure 1][1]

Téléchargez le [fichier du tableau de bord](https://aka.ms/networkwatchernsgflowlogdashboard), le [fichier de visualisation](https://aka.ms/networkwatchernsgflowlogvisualizations) et le [fichier des recherches sauvegardées](https://aka.ms/networkwatchernsgflowlogsearch).

Dans l’onglet **Gestion** de Kibana, accédez à **Saved Objects** (Objets enregistrés) et importez les trois fichiers. Puis, à partir de l’onglet **Tableau de bord**, vous pouvez ouvrir et charger l’exemple de tableau de bord.

Vous avez également la possibilité de créer vos propres visualisations et tableaux de bord en fonction des mesures qui vous intéressent. Reportez-vous à la [documentation officielle](https://www.tutorialspoint.com/kibana/kibana_create_visualization.htm) de Kibana pour en savoir plus sur la création de visualisation Kibana.

### <a name="visualize-nsg-flow-logs"></a>Visualiser les journaux de flux NSG

L’exemple de tableau de bord fournit plusieurs visualisations des journaux de flux :

1. Flux par décision/direction dans le temps : graphiques chronologiques indiquant le nombre de flux sur une période de temps. Vous pouvez modifier l’unité de temps et l’étendue de ces deux visualisations. Le graphique de flux par décision affiche la proportion de décisions d’autorisation ou de refus, tandis que celui du flux par direction affiche la proportion du trafic entrant et sortant. Grâce à ces éléments visuels, vous pouvez examiner les tendances du trafic au fil du temps et identifier les pics ou les modèles inhabituels.

   ![Capture d’écran montrant un exemple de tableau de bord avec des flux par décision et sens dans le temps.][2]

2. Flux par port source/de destination : graphiques à secteurs montrant la répartition des flux en fonction de leurs ports respectifs. Grâce à cette vue, vous pouvez visualiser les ports les plus fréquemment utilisés. Si vous cliquez sur un port spécifique dans le graphique à secteurs, le reste du tableau de bord filtre les flux selon ce port.

   ![Capture d’écran montrant un exemple de tableau de bord avec flux par destination et port source.][3]

3. Nombre de flux et date du premier journal : mesures indiquant le nombre de flux enregistrés et la date du plus ancien journal capturé.

   ![Capture d’écran montrant un exemple de tableau de bord avec le nombre de flux et la première heure de journalisation.][4]

4. Flux par groupe de sécurité réseau et par règle : graphique à barres montrant la répartition des flux au sein de chaque groupe de sécurité réseau, ainsi que la répartition des règles au sein de ces derniers. À partir de ces données, vous pouvez identifier le groupe de sécurité réseau et les règles qui ont généré le plus de trafic.

   ![Capture d’écran montrant un exemple de tableau de bord avec des flux par groupe de sécurité réseau et règle.][5]

5. 10 principales adresses IP source/de destination : graphiques à barres indiquant les 10 principales adresses IP source et de destination. Vous pouvez ajuster ces graphiques pour afficher un nombre plus ou moins important d’adresses IP. À partir de ces données, vous pouvez identifier les adresses IP les plus fréquemment utilisées, ainsi que la décision de trafic (autorisation ou refus) prise par rapport à chacune d’entre elles.

   ![Capture d’écran montrant un exemple de tableau de bord avec des les dix principales adresses IP de source et de destination.][6]

6. Tuples de flux : ce tableau affiche les informations contenues dans chaque tuple du flux, ainsi que ses groupes de sécurité réseau et ses règles correspondants.

   ![Capture d’écran montrant les tuples de flux dans une table.][7]

À l’aide de la barre de requête située en haut du tableau de bord, vous pouvez filtrer le tableau de bord en fonction de n’importe quel paramètre des flux, tel que l’ID d’abonnement, les groupes de ressources, la règle ou toute autre variable d’intérêt. Pour plus d’informations sur les requêtes et les filtres de Kibana, reportez-vous à la [documentation officielle](https://www.elastic.co/guide/en/beats/packetbeat/current/kibana-queries-filters.html).

## <a name="conclusion"></a>Conclusion

En combinant les journaux d’activité des flux de groupe de sécurité réseau avec la Suite Elastic, nous bénéficions d’une solution puissante et personnalisable permettant de visualiser le trafic de notre réseau. Ces tableaux de bord vous permettent d’obtenir et de partager rapidement des analyses sur le trafic de votre réseau, mais aussi de filtrer et d’examiner toute anomalie potentielle. Grâce à Kibana, vous pouvez adapter ces tableaux de bord et créer des visualisations spécifiques pour répondre à tous vos besoins en matière de sécurité, d’audit et de conformité.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment visualiser vos journaux de flux de groupe de sécurité réseau avec Power BI en consultant la page [Visualizing Network Security Group flow logs with Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md) (Visualisation des journaux de flux de groupe de sécurité réseau avec Power BI).

<!--Image references-->

[scenario]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/scenario.png
[1]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure1.png
[2]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure2.png
[3]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure3.png
[4]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure4.png
[5]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure5.png
[6]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure6.png
[7]: ./media/network-watcher-visualize-nsg-flow-logs-open-source-tools/figure7.png
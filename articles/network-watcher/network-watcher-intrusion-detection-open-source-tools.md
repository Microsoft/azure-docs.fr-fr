---
title: Détecter les intrusions dans un réseau avec des outils open source
titleSuffix: Azure Network Watcher
description: Cet article explique comment utiliser Azure Network Watcher et des outils open source pour détecter les intrusions dans un réseau
services: network-watcher
documentationcenter: na
author: damendo
ms.assetid: 0f043f08-19e1-4125-98b0-3e335ba69681
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 02/22/2017
ms.author: damendo
ms.openlocfilehash: 6834f9cc8202f49f53c20f3941945bba4a4b2efd
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026744"
---
# <a name="perform-network-intrusion-detection-with-network-watcher-and-open-source-tools"></a>Détecter les intrusions dans un réseau avec Azure Network Watcher et des outils open source

Les captures de paquets sont essentielles pour implémenter des systèmes de détection d’intrusions dans un réseau et surveiller la sécurité du réseau. Il existe plusieurs outils open source de systèmes de détection d’intrusions qui traitent des captures de paquets et recherchent les signatures des activités malveillantes et des intrusions dans un réseau éventuelles. Les captures de paquets proposées par Network Watcher vous permettent d’analyser votre réseau et de détecter toute vulnérabilité ou intrusion dangereuse.

Suricata fait partie de ces outils open source. Il s’agit d’un moteur de systèmes de détection d’intrusions qui utilise des ensembles de règles pour surveiller le trafic réseau et qui déclenche des alertes en cas d’évènements suspects. Suricata propose un moteur multithread, ce qui signifie qu’il peut effectuer une analyse du trafic réseau de manière plus rapide et plus efficace. Pour plus d’informations sur Suricata et ses fonctionnalités, rendez-vous sur le site web correspondant : https://suricata-ids.org/.

## <a name="scenario"></a>Scénario

Cet article explique comment configurer votre environnement pour détecter des intrusions dans un réseau à l’aide de Network Watcher, Suricata et de la pile élastique. Network Watcher vous fournit les captures de paquets permettant de détecter les intrusions dans un réseau. Suricata traite les captures de paquets et déclenche des alertes basées sur les paquets qui correspondent à son ensemble de règles de menaces donné. Ces alertes sont stockées dans un fichier journal sur votre ordinateur local. La pile élastique vous permet d’indexer les journaux d’activité générés par Suricata et de les utiliser pour créer un tableau de bord Kibana. Vous bénéficiez ainsi d’une représentation visuelle des journaux d’activité et d’un moyen pour obtenir rapidement des informations sur les vulnérabilités éventuelles du réseau.  

![scénario d’application web simple][1]

Les deux outils open source peuvent être configurés sur une machine virtuelle Azure, ce qui vous permet d’effectuer cette analyse dans votre propre environnement réseau Azure.

## <a name="steps"></a>Étapes

### <a name="install-suricata"></a>Installer Suricata

Pour toutes les autres méthodes d’installation, consultez https://suricata.readthedocs.io/en/suricata-5.0.2/quickstart.html#installation

1. Dans le terminal de ligne de commande de votre machine virtuelle, exécutez les commandes suivantes :

    ```
    sudo add-apt-repository ppa:oisf/suricata-stable
    sudo apt-get update
    sudo sudo apt-get install suricata
    ```

1. Pour vérifier votre installation, exécutez la commande `suricata -h` pour afficher la liste complète des commandes.

### <a name="download-the-emerging-threats-ruleset"></a>Télécharger l’ensemble de règles de menaces émergentes

À ce stade, Suricata n’a aucune règle à exécuter. Vous pouvez créer vos propres règles s’il existe des menaces spécifiques à votre réseau que vous souhaitez détecter, mais vous pouvez aussi utiliser des ensembles de règles développées issus de plusieurs fournisseurs, tels que les menaces émergentes ou les règles VRT de Snort. Nous utilisons le groupe de règles de menaces émergentes en accès libre ici :

Téléchargez l’ensemble de règles et copiez-le dans le répertoire :

```
wget https://rules.emergingthreats.net/open/suricata/emerging.rules.tar.gz
tar zxf emerging.rules.tar.gz
sudo cp -r rules /etc/suricata/
```

### <a name="process-packet-captures-with-suricata"></a>Traiter les captures de paquets avec Suricata

Pour traiter les captures de paquets à l’aide de Suricata, exécutez la commande suivante :

```
sudo suricata -c /etc/suricata/suricata.yaml -r <location_of_pcapfile>
```
Pour consulter les alertes générées, parcourez le fichier fast.log :
```
tail -f /var/log/suricata/fast.log
```

### <a name="set-up-the-elastic-stack"></a>Configurer la Suite Elastic

Même si les journaux créés par Suricata contiennent des informations importantes sur l’activité de notre réseau, ceux-ci ne sont pas les plus faciles à lire et à comprendre. En connectant Suricata à la pile élastique, nous pouvons générer un tableau de bord Kibana qui nous permet de rechercher, de créer des graphiques, d’analyser et d’obtenir des informations à partir de nos journaux d’activité.

#### <a name="install-elasticsearch"></a>Installer Elasticsearch

1. La Suite Elastic à partir de la version 5.0 et pour les versions ultérieures requiert Java 8. Exécutez la commande `java -version` pour vérifier la version que vous utilisez. Si Java n’est pas installé sur votre ordinateur, reportez-vous à la documentation relative aux [JDK pris en charge dans Azure](/azure/developer/java/fundamentals/java-jdk-long-term-support).

1. Téléchargez le package binaire approprié pour votre système :

    ```
    curl -L -O https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-5.2.0.deb
    sudo dpkg -i elasticsearch-5.2.0.deb
    sudo /etc/init.d/elasticsearch start
    ```

    D’autres méthodes d’installation se trouvent sur la page [Elasticsearch Installation](https://www.elastic.co/guide/en/beats/libbeat/5.2/elasticsearch-installation.html) (Installation d’Elasticsearch).

1. Vérifiez qu’Elasticsearch est en cours d’exécution avec la commande :

    ```
    curl http://127.0.0.1:9200
    ```

    Une réponse similaire à celle ci-dessous devrait s’afficher :

    ```
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

Pour plus d’informations sur l’installation d’Elasticsearch, reportez-vous à la page [Installation](https://www.elastic.co/guide/en/elasticsearch/reference/5.2/_installation.html).

### <a name="install-logstash"></a>Installer Logstash

1. Exécutez les commandes suivantes pour installer Logstash :

    ```
    curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
    sudo dpkg -i logstash-5.2.0.deb
    ```
1. Ensuite, nous devons configurer Logstash pour qu’il puisse lire à partir de la sortie du fichier de eve.json. Créez un fichier logstash.conf à l’aide de :

    ```
    sudo touch /etc/logstash/conf.d/logstash.conf
    ```

1. Ajoutez le contenu suivant au fichier (veillez à ce que le chemin d’accès au fichier eve.json soit correct) :

    ```ruby
    input {
    file {
        path => ["/var/log/suricata/eve.json"]
        codec =>  "json"
        type => "SuricataIDPS"
    }

    }

    filter {
    if [type] == "SuricataIDPS" {
        date {
        match => [ "timestamp", "ISO8601" ]
        }
        ruby {
        code => "
            if event.get('[event_type]') == 'fileinfo'
            event.set('[fileinfo][type]', event.get('[fileinfo][magic]').to_s.split(',')[0])
            end
        "
        }

        ruby{
        code => "
            if event.get('[event_type]') == 'alert'
            sp = event.get('[alert][signature]').to_s.split(' group ')
            if (sp.length == 2) and /\A\d+\z/.match(sp[1])
                event.set('[alert][signature]', sp[0])
            end
            end
            "
        }
    }

    if [src_ip]  {
        geoip {
        source => "src_ip"
        target => "geoip"
        #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
        add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
        add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
        }
        mutate {
        convert => [ "[geoip][coordinates]", "float" ]
        }
        if ![geoip.ip] {
        if [dest_ip]  {
            geoip {
            source => "dest_ip"
            target => "geoip"
            #database => "/opt/logstash/vendor/geoip/GeoLiteCity.dat"
            add_field => [ "[geoip][coordinates]", "%{[geoip][longitude]}" ]
            add_field => [ "[geoip][coordinates]", "%{[geoip][latitude]}"  ]
            }
            mutate {
            convert => [ "[geoip][coordinates]", "float" ]
            }
        }
        }
    }
    }

    output {
    elasticsearch {
        hosts => "localhost"
    }
    }
    ```

1. Assurez-vous d’accorder les autorisations appropriées pour le fichier eve.json afin que Logstash puisse ingérer le fichier.
    
    ```
    sudo chmod 775 /var/log/suricata/eve.json
    ```

1. Pour démarrer Logstash, exécutez la commande :

    ```
    sudo /etc/init.d/logstash start
    ```

Pour plus d’informations sur l’installation de Logstash, reportez-vous à la [documentation officielle](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html).

### <a name="install-kibana"></a>Installer Kibana

1. Exécutez les commandes suivantes pour installer Kibana :

    ```
    curl -L -O https://artifacts.elastic.co/downloads/kibana/kibana-5.2.0-linux-x86_64.tar.gz
    tar xzvf kibana-5.2.0-linux-x86_64.tar.gz

    ```
1. Pour exécuter Kibana, exécutez les commandes ci-dessous :

    ```
    cd kibana-5.2.0-linux-x86_64/
    ./bin/kibana
    ```

1. Pour afficher votre interface web Kibana, accédez à `http://localhost:5601`
1. Dans ce scénario, le modèle d’index utilisé pour les journaux d’activité Suricata est « logstash-* ».

1. Si vous souhaitez afficher le tableau de bord Kibana à distance, créez une règle de groupe de sécurité réseau entrante autorisant l’accès au **port 5601**.

### <a name="create-a-kibana-dashboard"></a>Créer un tableau de bord Kibana

Dans le cadre de cet article, nous proposons un exemple de tableau de bord pour que vous puissiez visualiser les tendances et les informations associées à vos alertes.

1. Téléchargez le fichier du tableau de bord [ici](https://aka.ms/networkwatchersuricatadashboard), le fichier de visualisation [ici](https://aka.ms/networkwatchersuricatavisualization)et le fichier de recherche sauvegardée [ici](https://aka.ms/networkwatchersuricatasavedsearch).

1. Dans l’onglet **Gestion** de Kibana, accédez à **Saved Objects** (Objets enregistrés) et importez les trois fichiers. Puis, à partir de l’onglet **Tableau de bord**, vous pouvez ouvrir et charger l’exemple de tableau de bord.

Vous avez également la possibilité de créer vos propres visualisations et tableaux de bord en fonction des mesures qui vous intéressent. Reportez-vous à la [documentation officielle](https://www.tutorialspoint.com/kibana/kibana_create_visualization.htm) de Kibana pour en savoir plus sur la création de visualisation Kibana.

![tableau de bord Kibana][2]

### <a name="visualize-ids-alert-logs"></a>Visualiser les journaux d’activité d’alertes de système de détection d’intrusions

L’exemple de tableau de bord fournit plusieurs visualisations des journaux d’activité d’alertes Suricata :

1. Alerts by GeoIP (Alertes par GeoIP) : une carte affichant la répartition des alertes par pays/région d’origine en fonction de l’emplacement géographique (déterminé par l’adresse IP).

    ![geo ip][3]

1. Top 10 Alerts (Top 10 des alertes) : un résumé des 10 alertes déclenchées les plus fréquentes et leur description. Cliquer sur une alerte permet d’appliquer un filtre sur le tableau de bord pour qu’il affiche uniquement les informations associées à cette alerte spécifique.

    ![image 4][4]

1. Nombre d’alertes : le nombre total d’alertes déclenchées par le groupe de règles.

    ![image 5][5]

1. Top 20 Source/Destination IPs/Ports (Top 20 des ports/adresses IP source et de destination) : graphiques à secteurs montrant le top 20 des adresses IP et ports sur lesquels les alertes ont été déclenchées. Vous pouvez appliquer un filtre sur des adresses IP/ports spécifiques pour connaître le nombre et le type d’alertes déclenchées.

    ![image 6][6]

1. Résumé des alertes : un tableau récapitulant les spécificités de chaque alerte. Vous pouvez personnaliser ce tableau pour afficher d’autres paramètres qui vous intéressent pour chaque alerte.

    ![image 7][7]

Pour plus d’informations sur la création de tableaux de bord et de visualisations personnalisés, consultez la [documentation officielle de Kibana](https://www.elastic.co/guide/en/kibana/current/introduction.html).

## <a name="conclusion"></a>Conclusion

L’association des captures de paquets fournies par Network Watcher et des outils open source de système de détection d’intrusions tels que Suricata vous permet de détecter les intrusions dans un réseau pour un large éventail de menaces. Ces tableaux de bord vous donnent la possibilité d’identifier rapidement les tendances et les anomalies au sein de votre réseau, mais également d’analyser les données pour détecter les causes premières des alertes, telles que des agents utilisateur malveillants ou des ports vulnérables. Avec ces données extraites, vous pouvez prendre des décisions avisées sur l’attitude et les méthodes à adopter pour protéger votre réseau de toute tentative d’intrusion dangereuse, et créer des règles pour empêcher les intrusions futures.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment déclencher des captures de paquets en fonction des alertes en vous rendant sur [Use packet capture to do proactive network monitoring with Azure Functions](network-watcher-alert-triggered-packet-capture.md) (Utiliser des captures de paquets pour la surveillance proactive du réseau avec Azure Functions).

Découvrez comment visualiser vos journaux de flux de groupe de sécurité réseau avec Power BI en consultant la page [Visualizing Network Security Group flow logs with Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md) (Visualisation des journaux de flux de groupe de sécurité réseau avec Power BI).



<!-- images -->
[1]: ./media/network-watcher-intrusion-detection-open-source-tools/figure1.png
[2]: ./media/network-watcher-intrusion-detection-open-source-tools/figure2.png
[3]: ./media/network-watcher-intrusion-detection-open-source-tools/figure3.png
[4]: ./media/network-watcher-intrusion-detection-open-source-tools/figure4.png
[5]: ./media/network-watcher-intrusion-detection-open-source-tools/figure5.png
[6]: ./media/network-watcher-intrusion-detection-open-source-tools/figure6.png
[7]: ./media/network-watcher-intrusion-detection-open-source-tools/figure7.png
---
title: Analyser les journaux de flux des groupes de sécurité réseau Azure - Graylog | Microsoft Docs
description: Découvrez comment gérer et analyser les journaux de flux des groupes de sécurité réseau dans Azure à l’aide de Network Watcher et de Graylog.
services: network-watcher
documentationcenter: na
author: damendo
editor: ''
tags: azure-resource-manager
ms.assetid: ''
ms.service: network-watcher
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/19/2017
ms.author: damendo
ms.openlocfilehash: 8b363d90d05e95b017c3a655b57dbabc3712a155
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94965542"
---
# <a name="manage-and-analyze-network-security-group-flow-logs-in-azure-using-network-watcher-and-graylog"></a>Gérer et analyser les journaux de flux des groupes de sécurité réseau à l’aide de Network Watcher et de Graylog

Les [journaux de flux des groupes de sécurité réseau ](network-watcher-nsg-flow-logging-overview.md) fournissent des informations permettant de comprendre le trafic IP entrant et sortant des interfaces réseau Azure. Les journaux de flux affichent les flux entrants et sortants en fonction de la règle du groupe de sécurité réseau, l’interface réseau à laquelle le flux s’applique, des informations à 5 tuples (adresse IP source/de destination, port source/de destination, protocole) relatives au flux, et l’autorisation ou le refus du trafic.

Un réseau peut comprendre de nombreux groupes de sécurité réseau pour lesquels la journalisation de flux est activée. La journalisation de flux activée sur plusieurs groupes de sécurité réseau peut rendre difficile l’analyse et l’exploitation d’informations à partir de vos journaux d’activité. Cet article fournit une solution de gestion centralisée de ces journaux de flux des groupes de sécurité réseau avec l’utilisation de Graylog, un outil open source d’analyse et de gestion des journaux d’activité, et de Logstash, un pipeline open source de traitement des données côté serveur.

> [!Warning]
> Les étapes suivantes fonctionnent avec les journaux de flux version 1. Pour plus d’informations, consultez [Présentation de la journalisation des flux pour les groupes de sécurité réseau](network-watcher-nsg-flow-logging-overview.md). Les instructions suivantes ne fonctionnent pas avec la version 2 des fichiers journaux, sans modification.

## <a name="scenario"></a>Scénario

Les journaux de flux des groupes de sécurité réseau sont activés à l’aide de Network Watcher. Les journaux de flux affluent dans le stockage Blob Azure. Un plug-in Logstash assure la connexion et le traitement des journaux de flux à partir du stockage Blob, et les envoie à Graylog. Une fois les journaux de flux stockés dans Graylog, il peuvent être analysés et visualisés dans des tableaux de bord personnalisés.

![Workflow Graylog](./media/network-watcher-analyze-nsg-flow-logs-graylog/workflow.png)

## <a name="installation-steps"></a>Étapes d’installation

### <a name="enable-network-security-group-flow-logging"></a>Activer les journaux de flux des groupes de sécurité réseau

Pour ce scénario, la journalisation des flux de groupe de sécurité réseau doit être activée sur au moins un groupe de sécurité réseau dans votre compte. Pour obtenir des instructions sur l’activation des journaux de flux des groupes de sécurité réseau, consultez l’article suivant [Présentation de la journalisation des flux pour les groupes de sécurité réseau](network-watcher-nsg-flow-logging-overview.md).

### <a name="setting-up-graylog"></a>Configurer Graylog

Dans cet exemple, Graylog et Logstash sont configurés sur un serveur Ubuntu 14.04 déployé dans Azure.

- Reportez-vous à la [documentation](https://docs.graylog.org/en/2.2/pages/installation/os/ubuntu.html) de Graylog pour les instructions détaillées de l’installation sur Ubuntu.
- Veillez également à configurer l’interface web de Graylog en suivant les instructions de la [documentation](https://docs.graylog.org/en/2.2/pages/configuration/web_interface.html#configuring-webif).

Cet exemple utilise la configuration minimale de Graylog (c’est-à-dire une seule instance d’une installation Graylog), mais l’architecture de Graylog est adaptable aux diverses ressources, en fonction de votre système et de vos besoins en production. Pour obtenir plus d’informations sur les considérations architecturales, ou un guide détaillé sur l’architecture, consultez la [documentation](https://docs.graylog.org/en/2.2/pages/architecture.html) de Graylog et le [guide de l’architecture](https://www.slideshare.net/Graylog/graylog-engineering-design-your-architecture).

Vous pouvez installer Graylog de bien des façons, selon votre plateforme et vos préférences. Pour obtenir une liste complète des méthodes d’installation possibles, reportez-vous à la [documentation](https://docs.graylog.org/en/2.2/pages/installation.html) officielle de Graylog. L’application serveur Graylog s’exécute sur les distributions Linux et implique les prérequis suivants :

-  Java SE 8 ou version ultérieure – [Documentation d’Azul Azure JDK](/azure/developer/java/fundamentals/java-jdk-long-term-support)
-  Elastic Search 2.x (2.1.0 ou version ultérieure) - [Documentation d’installation Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/2.4/_installation.html)
-  MongoDB 2.4 ou version ultérieure – [Documentation d’installation MongoDB](https://docs.mongodb.com/manual/administration/install-on-linux/)

### <a name="install-logstash"></a>Installer Logstash

Logstash vous permet d’aplatir à un niveau de tuple de flux les journaux de flux au format JSON. L’aplanissement des journaux de flux facilite l’organisation et la recherche des journaux d’activité dans Graylog.

1. Pour installer Logstash, exécutez les commandes suivantes :

   ```bash
   curl -L -O https://artifacts.elastic.co/downloads/logstash/logstash-5.2.0.deb
   sudo dpkg -i logstash-5.2.0.deb
   ```

2. Configurez Logstash pour analyser les journaux de flux et les envoyer à Graylog. Créez un fichier logstash.conf :

   ```bash
   sudo touch /etc/logstash/conf.d/logstash.conf
   ```

3. Ajoutez le contenu suivant au fichier. Changez les valeurs mises en évidence pour refléter les détails de votre compte de stockage :

   ```
    input {
        azureblob
        {
            storage_account_name => "mystorageaccount"
            storage_access_key => "NrUZmx7pJSKaRJzvQbeiZWi5nBRWOTr7Wwr9DrvK7YtDBrADYxT1y0oEExtSlkDnGRt7qcRiZzEBCCyRYND8SxSt"
            container => "insights-logs-networksecuritygroupflowevent"
            registry_create_policy => "start_over"
            codec => "json"
            file_head_bytes => 21
            file_tail_bytes => 9
            # Possible options: `do_not_break`, `with_head_tail`, `without_head_tail`
            break_json_down_policy  => 'with_head_tail'
            break_json_batch_count => 2
            interval => 5
        }
    }
    
    filter {
        split { field => "[records]" }
        split { field => "[records][properties][flows]"}
        split { field => "[records][properties][flows][flows]"}
        split { field => "[records][properties][flows][flows][flowTuples]"
    }
    
     mutate {
        split => { "[records][resourceId]" => "/"}
        add_field =>{
                    "Subscription" => "%{[records][resourceId][2]}"
                    "ResourceGroup" => "%{[records][resourceId][4]}"
                    "NetworkSecurityGroup" => "%{[records][resourceId][8]}"
        }
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
        udp {
            host => "127.0.0.1"
            port => 12201
        }
    }
    ```
   Le fichier de configuration Logstash fourni comporte trois parties : l’entrée, le filtre et la sortie. La section d’entrée désigne la source d’entrée des journaux d’activité que Logstash va traiter. Dans ce cas, vous allez utiliser un plug-in d’entrée d’objets blob Azure (installé aux étapes suivantes) qui nous permet d’accéder aux fichiers JSON de journalisation des flux des groupes de sécurité réseau qui sont stockés dans le stockage Blob.

La section de filtre aplatit ensuite chaque fichier journal du flux pour que chaque tuple de flux et les propriétés qui lui sont associées deviennent un événement Logstash à part entière.

Enfin, la section de sortie transfère chaque événement Logstash vers le serveur Graylog. N’hésitez pas à modifier le fichier de configuration Logstash pour l’adapter à vos besoins.

   > [!NOTE]
   > Le fichier de configuration précédent suppose que le serveur Graylog a été configuré sur l’adresse IP de bouclage de l’hôte local 127.0.0.1. Si ce n’est pas le cas, assurez-vous de changer le paramètre d’hôte dans la section de sortie pour l’adresse IP correcte.

Pour plus d’informations sur l’installation de Logstash, reportez-vous à la [documentation](https://www.elastic.co/guide/en/beats/libbeat/5.2/logstash-installation.html) Logstash.

### <a name="install-the-logstash-input-plug-in-for-azure-blob-storage"></a>Installer le plug-in d’entrée Logstash pour le stockage Blob Azure

Ce plug-in Logstash vous permet d’accéder directement aux journaux de flux, à partir de leur compte de stockage Blob désigné. Pour installer le plug-in, exécutez la commande suivante à partir du répertoire d’installation de Logstash par défaut (dans ce cas, /usr/share/logstash/bin) :

```bash
cd /usr/share/logstash/bin
sudo ./logstash-plugin install logstash-input-azureblob
```

Pour obtenir plus d’informations sur ce plug-in, reportez-vous à la [documentation](https://github.com/Azure/azure-diagnostics-tools/tree/master/Logstash/logstash-input-azureblob).

### <a name="set-up-connection-from-logstash-to-graylog"></a>Configurer la connexion de Logstash à Graylog

Une connexion aux journaux de flux étant établie à l’aide de Logstash, et la configuration du serveur Graylog étant effectuée, il ne vous reste plus qu’à configurer Graylog pour qu’il accepte les fichiers journaux entrants.

1. Accédez à l’interface web de votre serveur Graylog au moyen de l’URL que vous avez configurée à cet effet. Vous pouvez accéder à l’interface en indiquant à votre navigateur l’emplacement `http://<graylog-server-ip>:9000/`

2. Pour accéder à la page de configuration, à droite dans la barre de navigation supérieure, sélectionnez le menu déroulant **System** (Système), puis cliquez sur **Inputs** (Entrées).
   Vous pouvez également accéder à l’emplacement suivant `http://<graylog-server-ip>:9000/system/inputs`

   ![Prise en main](./media/network-watcher-analyze-nsg-flow-logs-graylog/getting-started.png)

3. Pour lancer la nouvelle entrée, dans la zone de liste déroulante **Select input** (Sélectionner une entrée), sélectionnez *GELF UDP*, puis remplissez le formulaire. GELF est l’acronyme de « Graylog Extended Log Format » et signifie « format de journal étendu Graylog ». Le format GELF est développé par Graylog. Pour plus d’informations sur ses avantages, consultez la [documentation](https://docs.graylog.org/en/2.2/pages/gelf.html) Graylog.

   Veillez à lier l’entrée à l’adresse IP sur laquelle vous avez configurée votre serveur Graylog. L’adresse IP doit correspondre au champ **host** (hôte) de la sortie UDP du fichier de configuration Logstash. Le port par défaut doit être *12201*. Vérifiez que le port correspond au champ **port** dans la sortie UDP désignée dans le fichier de configuration Logstash.

   ![Capture d’écran montrant les entrées Graylog, avec des options pour lancer et rechercher des entrées.](./media/network-watcher-analyze-nsg-flow-logs-graylog/inputs.png)

   Dès que vous avez lancé l’entrée, celle-ci doit apparaître sous la section **Local inputs** (Entrées locales), comme illustré dans l’image suivante :

   ![Capture d’écran montrant la section Entrées locales qui contient l’entrée que vous avez lancée.](./media/network-watcher-analyze-nsg-flow-logs-graylog/local-inputs.png)

   Pour approfondir votre connaissance des entrées de message Graylog, reportez-vous à la [documentation](https://docs.graylog.org/en/2.2/pages/sending_data.html#what-are-graylog-message-inputs).

4. Une fois ces configurations effectuées, vous pouvez démarrer Logstash et commencer la lecture dans les journaux de flux au moyen de la commande suivante : `sudo systemctl start logstash.service`.

### <a name="search-through-graylog-messages"></a>Effectuer des recherches dans les messages Graylog

Après avoir laissé le temps à votre serveur Graylog de collecter les messages, vous pouvez effectuer des recherches dans les messages. Pour vérifier les messages envoyés à votre serveur Graylog, dans la page de configuration **Inputs** (Entrées), cliquez sur le bouton **Show received messages** (Afficher les messages reçus) de l’entrée GELF UDP que vous avez créée. Vous êtes dirigé vers un écran semblable à l’image suivante : 

![Capture d’écran montrant le serveur Graylog qui affiche le résultat de la recherche, l’histogramme et les messages.](./media/network-watcher-analyze-nsg-flow-logs-graylog/histogram.png)

En cliquant sur le lien bleu « %{Message} », vous développez chaque message pour afficher les paramètres de chaque tuple de flux, comme illustré dans l’image suivante :

![Capture d’écran montrant les détails d’un messages du serveur Graylog.](./media/network-watcher-analyze-nsg-flow-logs-graylog/messages.png)

Par défaut, tous les champs de message sont inclus dans la recherche si vous ne sélectionnez pas un champ particulier à rechercher. Si vous souhaitez rechercher des messages spécifiques (comme des tuples de flux à partir d’une adresse IP source particulière), vous pouvez utiliser le langage de requête de recherche Graylog tel qu’il est [documenté](https://docs.graylog.org/en/2.2/pages/queries.html).

## <a name="analyze-network-security-group-flow-logs-using-graylog"></a>Analyser les journaux de flux des groupes de sécurité réseau avec Graylog

À présent que Graylog est configuré et s’exécute, vous pouvez utiliser certaines de ses fonctionnalités pour mieux comprendre les données de vos journaux de flux. L’une d’elles consiste à se servir des tableaux de bord pour créer des vues spécifiques de vos données.

### <a name="create-a-dashboard"></a>Création d’un tableau de bord

1. Dans la barre de navigation supérieure, sélectionnez **Dashboards** (Tableaux de bord) ou accédez à `http://<graylog-server-ip>:9000/dashboards/`.

2. À partir de là, cliquez sur le bouton vert **Create dashboard** (Créer un tableau de bord) et renseignez la forme abrégée avec le titre et la description de votre tableau de bord. Cliquez sur le bouton **Save** (Enregistrer) pour créer le tableau de bord. Vous voyez un tableau de bord similaire à l’image suivante :

    ![Capture d’écran montrant les tableaux de bord du serveur Graylog, avec les options permettant de créer et de modifier ces tableaux de bord.](./media/network-watcher-analyze-nsg-flow-logs-graylog/dashboards.png)

### <a name="add-widgets"></a>Ajout de widgets

Vous pouvez cliquer sur le titre du tableau de bord pour afficher celui-ci, bien qu’il soit vide pour l’instant, car nous n’avons pas ajouté de widget. Un type de widget, utile et facile à ajouter au tableau de bord, sont les graphiques **Quick Values** (Valeurs rapides) qui affichent une liste des valeurs du champ sélectionné, et leur répartition.

1. Revenez aux résultats de la recherche de l’entrée UDP qui reçoit les journaux de flux en sélectionnant **Search** (Rechercher) dans la barre de navigation supérieure.

2. À gauche, en dessous du volet **Résultat de la recherche**, affichez l’onglet **Champs** qui répertorie les différents champs de chaque message de tuple de flux entrant.

3. Sélectionnez le paramètre souhaité dans lequel visualiser (dans cet exemple l’adresse IP source est sélectionnée). Pour afficher la liste de widgets possibles, cliquez sur la flèche déroulante bleue à gauche du champ, puis sélectionnez **Quick values** (Valeurs rapides) pour générer le widget. Vous devez obtenir un graphique similaire à l’illustration suivante :

   ![IP Source](./media/network-watcher-analyze-nsg-flow-logs-graylog/srcip.png)

4. À partir de là, vous pouvez sélectionner le bouton **Add to dashboard** (Ajouter au tableau de bord), situé dans le coin supérieur droit du widget, et sélectionner le tableau de bord correspondant à ajouter.

5. Revenez au tableau de bord pour voir le widget que vous venez d’ajouter.

   Vous pouvez ajouter bien d’autres widgets à votre tableau de bord, tels que des histogrammes et des compteurs, pour suivre les métriques importants, comme l’exemple du tableau de bord illustré dans l’image suivante :

   ![Tableau de bord des journaux de flux](./media/network-watcher-analyze-nsg-flow-logs-graylog/flowlogs-dashboard.png)

    Pour obtenir davantage d’explications sur les tableaux de bord et les autres types de widgets, reportez-vous à la [documentation](https://docs.graylog.org/en/2.2/pages/dashboards.html) de Graylog.

L’intégration de Network Watcher à Graylog vous permet de visualiser et de gérer de manière centralisée les journaux de flux des groupes de sécurité réseau. Graylog comprend d’autres puissantes fonctionnalités, telles que les flux de données en continu et les alertes, qui peuvent aussi servir à gérer plus étroitement les journaux de flux et vous aider à mieux comprendre votre trafic réseau. Graylog étant désormais configuré et connecté à Azure, vous pouvez continuer à explorer les autres fonctionnalités qu’il propose.

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment visualiser vos journaux de flux des groupes de sécurité réseau avec Power BI en consultant la page [Visualiser les journaux de flux des groupes de sécurité réseau avec Power BI](network-watcher-visualize-nsg-flow-logs-power-bi.md).
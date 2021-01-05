---
title: Analyser des données Twitter avec Apache Hive - Azure HDInsight
description: Découvrez comment utiliser Apache Hive et Apache Hadoop sur HDInsight pour transformer des données Twitter brutes en une table Hive utilisable dans une requête.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.custom: H1Hack27Feb2017,hdinsightactive
ms.date: 12/16/2019
ms.openlocfilehash: fe511ed2d6b724c1215f9986c9d6c50aae076935
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95971907"
---
# <a name="analyze-twitter-data-using-apache-hive-and-apache-hadoop-on-hdinsight"></a>Analyser des données Twitter avec Apache Hive et Apache Hadoop sur HDInsight

Découvrez comment utiliser [Apache Hive](https://hive.apache.org/) pour traiter des données Twitter. Une liste des utilisateurs de Twitter ayant envoyé le plus de tweets contenant un mot donné vous est ensuite retournée.

> [!IMPORTANT]  
> Les étapes décrites dans ce document ont été testées sur HDInsight 3.6.

## <a name="get-the-data"></a>Obtenir les données

Twitter vous permet de récupérer les données de chaque tweet sous la forme d’un document JavaScript Object Notation (JSON) via une API REST. [OAuth](https://oauth.net) est requis pour l'authentification de l'API.

### <a name="create-a-twitter-application"></a>Création d'une application Twitter

1. Dans un navigateur web, connectez-vous à [https://developer.twitter.com/apps/](https://developer.twitter.com/apps/). Sélectionnez le lien **Sign up now** si vous ne possédez pas de compte Twitter.

2. Sélectionnez **Create New App** (Créer une application).

3. Renseignez les champs **Name**, **Description** et **Website**. Vous pouvez créer une URL pour le champ **Website** . Le tableau suivant affiche quelques exemples de valeurs à utiliser :

   | Champ | Valeur |
   |--- |--- |
   | Nom |MyHDInsightApp |
   | Description |MyHDInsightApp |
   | Website |`https://www.myhdinsightapp.com` |

4. Sélectionnez **Yes, I agree**, puis **Create your Twitter application**.

5. Sélectionnez l’onglet **Permissions**. L'autorisation par défaut est **Read only**.

6. Sélectionnez l’onglet **Keys and Access Tokens** .

7. Sélectionnez **Create my access token**.

8. Sélectionnez **Test OAuth** dans le coin supérieur droit de la page.

9. Renseignez les valeurs **consumer key**, **Consumer secret**, **Access token** et **Access token secret**.

### <a name="download-tweets"></a>Téléchargement de tweets

Le code Python suivant télécharge 10 000 tweets à partir de Twitter et les enregistrera dans un fichier nommé **tweets.txt**.

> [!NOTE]  
> Les étapes suivantes sont effectuées sur le cluster HDInsight, puisque Python a déjà été installé.

1. Utilisez la [commande ssh](./hdinsight-hadoop-linux-use-ssh-unix.md) pour vous connecter à votre cluster. Modifiez la commande ci-dessous en remplaçant CLUSTERNAME par le nom de votre cluster, puis entrez la commande :

    ```cmd
    ssh sshuser@CLUSTERNAME-ssh.azurehdinsight.net
    ```

1. Utilisez les commandes suivantes pour installer [Tweepy](https://www.tweepy.org/), la [barre de progression](https://pypi.python.org/pypi/progressbar/2.2) et d’autres packages requis :

   ```bash
   sudo apt install python-dev libffi-dev libssl-dev
   sudo apt remove python-openssl
   python -m pip install virtualenv
   mkdir gettweets
   cd gettweets
   virtualenv gettweets
   source gettweets/bin/activate
   pip install tweepy progressbar pyOpenSSL requests[security]
   ```

1. Utilisez la commande suivante pour créer un fichier nommé **gettweets.py** :

   ```bash
   nano gettweets.py
   ```

1. Modifiez le code ci-dessous en remplaçant `Your consumer secret`, `Your consumer key`, `Your access token` et `Your access token secret` par les informations pertinentes de votre application Twitter. Collez ensuite le code modifié comme contenu du fichier **gettweets.py**.

   ```python
   #!/usr/bin/python

   from tweepy import Stream, OAuthHandler
   from tweepy.streaming import StreamListener
   from progressbar import ProgressBar, Percentage, Bar
   import json
   import sys

   #Twitter app information
   consumer_secret='Your consumer secret'
   consumer_key='Your consumer key'
   access_token='Your access token'
   access_token_secret='Your access token secret'

   #The number of tweets we want to get
   max_tweets=100

   #Create the listener class that receives and saves tweets
   class listener(StreamListener):
       #On init, set the counter to zero and create a progress bar
       def __init__(self, api=None):
           self.num_tweets = 0
           self.pbar = ProgressBar(widgets=[Percentage(), Bar()], maxval=max_tweets).start()

       #When data is received, do this
       def on_data(self, data):
           #Append the tweet to the 'tweets.txt' file
           with open('tweets.txt', 'a') as tweet_file:
               tweet_file.write(data)
               #Increment the number of tweets
               self.num_tweets += 1
               #Check to see if we have hit max_tweets and exit if so
               if self.num_tweets >= max_tweets:
                   self.pbar.finish()
                   sys.exit(0)
               else:
                   #increment the progress bar
                   self.pbar.update(self.num_tweets)
           return True

       #Handle any errors that may occur
       def on_error(self, status):
           print status

   #Get the OAuth token
   auth = OAuthHandler(consumer_key, consumer_secret)
   auth.set_access_token(access_token, access_token_secret)
   #Use the listener class for stream processing
   twitterStream = Stream(auth, listener())
   #Filter for these topics
   twitterStream.filter(track=["azure","cloud","hdinsight"])
   ```

    > [!TIP]  
    > Ajustez le filtre de rubriques sur la dernière ligne afin de suivre les mots clés courants. L’utilisation des mots clés courants au moment de l’exécution du script permet une capture plus rapide des données.

1. Appuyez sur **Ctrl + X**, puis sur **Y** pour enregistrer le fichier.

1. Utilisez la commande suivante pour exécuter le fichier et télécharger des tweets :

    ```bash
    python gettweets.py
    ```

    Un indicateur de progression s’affiche. Il progresse jusqu’à 100 % au fur et à mesure que les tweets sont téléchargés.

   > [!NOTE]  
   > Si la barre de progression n’avance pas, vous devez modifier le filtre pour effectuer le suivi des tendances. Lorsqu’il existe un grand nombre de tweets concernant la rubrique que vous filtrez, vous pouvez obtenir rapidement les 100 tweets nécessaires.

### <a name="upload-the-data"></a>Téléchargement des données

Pour télécharger les données vers le stockage HDInsight, utilisez les commandes suivantes :

```bash
hdfs dfs -mkdir -p /tutorials/twitter/data
hdfs dfs -put tweets.txt /tutorials/twitter/data/tweets.txt
```

Ces commandes stockent les données dans un emplacement accessible à tous les nœuds du cluster.

## <a name="run-the-hiveql-job"></a>Exécutez la tâche HiveQL

1. Utilisez la commande suivante pour créer un fichier contenant des instructions [HiveQL](https://cwiki.apache.org/confluence/display/Hive/LanguageManual) :

   ```bash
   nano twitter.hql
   ```

    Utilisez les données suivantes comme contenu du fichier :

   ```hiveql
   set hive.exec.dynamic.partition = true;
   set hive.exec.dynamic.partition.mode = nonstrict;
   -- Drop table, if it exists
   DROP TABLE tweets_raw;
   -- Create it, pointing toward the tweets logged from Twitter
   CREATE EXTERNAL TABLE tweets_raw (
       json_response STRING
   )
   STORED AS TEXTFILE LOCATION '/tutorials/twitter/data';
   -- Drop and recreate the destination table
   DROP TABLE tweets;
   CREATE TABLE tweets
   (
       id BIGINT,
       created_at STRING,
       created_at_date STRING,
       created_at_year STRING,
       created_at_month STRING,
       created_at_day STRING,
       created_at_time STRING,
       in_reply_to_user_id_str STRING,
       text STRING,
       contributors STRING,
       retweeted STRING,
       truncated STRING,
       coordinates STRING,
       source STRING,
       retweet_count INT,
       url STRING,
       hashtags array<STRING>,
       user_mentions array<STRING>,
       first_hashtag STRING,
       first_user_mention STRING,
       screen_name STRING,
       name STRING,
       followers_count INT,
       listed_count INT,
       friends_count INT,
       lang STRING,
       user_location STRING,
       time_zone STRING,
       profile_image_url STRING,
       json_response STRING
   );
   -- Select tweets from the imported data, parse the JSON,
   -- and insert into the tweets table
   FROM tweets_raw
   INSERT OVERWRITE TABLE tweets
   SELECT
       cast(get_json_object(json_response, '$.id_str') as BIGINT),
       get_json_object(json_response, '$.created_at'),
       concat(substr (get_json_object(json_response, '$.created_at'),1,10),' ',
       substr (get_json_object(json_response, '$.created_at'),27,4)),
       substr (get_json_object(json_response, '$.created_at'),27,4),
       case substr (get_json_object(json_response,    '$.created_at'),5,3)
           when "Jan" then "01"
           when "Feb" then "02"
           when "Mar" then "03"
           when "Apr" then "04"
           when "May" then "05"
           when "Jun" then "06"
           when "Jul" then "07"
           when "Aug" then "08"
           when "Sep" then "09"
           when "Oct" then "10"
           when "Nov" then "11"
           when "Dec" then "12" end,
       substr (get_json_object(json_response, '$.created_at'),9,2),
       substr (get_json_object(json_response, '$.created_at'),12,8),
       get_json_object(json_response, '$.in_reply_to_user_id_str'),
       get_json_object(json_response, '$.text'),
       get_json_object(json_response, '$.contributors'),
       get_json_object(json_response, '$.retweeted'),
       get_json_object(json_response, '$.truncated'),
       get_json_object(json_response, '$.coordinates'),
       get_json_object(json_response, '$.source'),
       cast (get_json_object(json_response, '$.retweet_count') as INT),
       get_json_object(json_response, '$.entities.display_url'),
       array(
           trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[1].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[2].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[3].text'))),
           trim(lower(get_json_object(json_response, '$.entities.hashtags[4].text')))),
       array(
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[1].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[2].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[3].screen_name'))),
           trim(lower(get_json_object(json_response, '$.entities.user_mentions[4].screen_name')))),
       trim(lower(get_json_object(json_response, '$.entities.hashtags[0].text'))),
       trim(lower(get_json_object(json_response, '$.entities.user_mentions[0].screen_name'))),
       get_json_object(json_response, '$.user.screen_name'),
       get_json_object(json_response, '$.user.name'),
       cast (get_json_object(json_response, '$.user.followers_count') as INT),
       cast (get_json_object(json_response, '$.user.listed_count') as INT),
       cast (get_json_object(json_response, '$.user.friends_count') as INT),
       get_json_object(json_response, '$.user.lang'),
       get_json_object(json_response, '$.user.location'),
       get_json_object(json_response, '$.user.time_zone'),
       get_json_object(json_response, '$.user.profile_image_url'),
       json_response
   WHERE (length(json_response) > 500);
   ```

1. Appuyez sur **Ctrl + X**, puis sur **Y** pour enregistrer le fichier.

1. Utilisez la commande suivante pour exécuter le HiveQL contenu dans le fichier :

   ```bash
   beeline -u 'jdbc:hive2://headnodehost:10001/;transportMode=http' -i twitter.hql
   ```

    Cette commande exécute le fichier **twitter.hql**. Une fois la requête terminée, vous voyez une invite `jdbc:hive2//localhost:10001/>`.

1. À l’invite Beeline, utilisez la requête suivante pour vérifier que les données ont été importées :

   ```hiveql
   SELECT name, screen_name, count(1) as cc
   FROM tweets
   WHERE text like "%Azure%"
   GROUP BY name,screen_name
   ORDER BY cc DESC LIMIT 10;
   ```

    Cette requête renvoie un maximum de 10 tweets contenant le mot **Azure** dans le corps du message.

    > [!NOTE]  
    > Si vous avez modifié le filtre dans le script `gettweets.py`, remplacez **Azure** avec l’un des filtres utilisés.

## <a name="next-steps"></a>Étapes suivantes

Vous avez appris à transformer un jeu de données JSON non structuré en une table [Apache Hive](https://hive.apache.org/) structurée. Pour en savoir plus sur Hive avec HDInsight, consultez les documents suivants :

* [Prise en main de HDInsight](hadoop/apache-hadoop-linux-tutorial-get-started.md)
* [Analyse des données sur les retards de vol avec HDInsight](./interactive-query/interactive-query-tutorial-analyze-flight-data.md)
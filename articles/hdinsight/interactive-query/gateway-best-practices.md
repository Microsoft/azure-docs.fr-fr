---
title: Présentation approfondie de la passerelle et meilleures pratiques pour Apache Hive dans Azure HDInsight
description: Découvrez comment parcourir les meilleures pratiques pour exécuter des requêtes Hive sur la passerelle Azure HDInsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: conceptual
ms.date: 04/01/2020
ms.openlocfilehash: 3db411df69a754857220867865522f8e4fa24030
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96011487"
---
# <a name="gateway-deep-dive-and-best-practices-for-apache-hive-in-azure-hdinsight"></a>Présentation approfondie de la passerelle et meilleures pratiques pour Apache Hive dans Azure HDInsight

La passerelle Azure HDInsight (la « Passerelle ») est le serveur frontal HTTPS pour les clusters HDInsight. La Passerelle est responsable des opérations suivantes : authentification, résolution de l’hôte, détection du service et autres fonctionnalités utiles nécessaires à un système distribué moderne. Les fonctionnalités fournies par la Passerelle entraînent une surcharge pour laquelle ce document décrira les meilleures pratiques à utiliser. Les techniques de résolution des problèmes de la Passerelle sont également abordées.

## <a name="the-hdinsight-gateway"></a>Passerelle HDInsight

La passerelle HDInsight est la seule partie d’un cluster HDInsight qui est publiquement accessible sur Internet. Le service de Passerelle est accessible sans passer par l’Internet public en utilisant le point de terminaison de passerelle interne `clustername-int.azurehdinsight.net`. Le point de terminaison de passerelle interne permet d’établir des connexions au service de passerelle sans quitter le réseau virtuel du cluster. La Passerelle traite toutes les requêtes envoyées au cluster et les transfère vers les composants et les hôtes de cluster appropriés.

Le diagramme suivant fournit une illustration approximative de la façon dont la Passerelle fournit une abstraction devant toutes les différentes possibilités de résolution de l’hôte dans HDInsight.

![Host Resolution Diagram](./media/gateway-best-practices/host-resolution-diagram.png "Diagramme de résolution de l’hôte")

## <a name="motivation"></a>Motivation

La motivation pour mettre une passerelle devant des clusters HDInsight est de fournir une interface pour la détection de service et l’authentification des utilisateurs. Les mécanismes d’authentification fournis par la passerelle sont particulièrement pertinents pour les clusters compatibles ESP.

Pour la détection de service, l’avantage de la passerelle est que chaque composant au sein du cluster est accessible en tant que point de terminaison différent sous le site web de la Passerelle (`clustername.azurehdinsight.net/hive2`), par opposition à une multitude de jumelages `host:port`.

Pour l’authentification, la Passerelle permet aux utilisateurs de s’authentifier à l’aide d’une paire d’informations d’identification `username:password`. Pour les clusters compatibles ESP, ces informations d’identification sont le nom d’utilisateur et le mot de passe du domaine de l’utilisateur. L’authentification auprès de clusters HDInsight via la Passerelle ne nécessite pas que le client obtienne un ticket Kerberos. Étant donné que la Passerelle accepte les informations d’identification `username:password` et acquiert le ticket Kerberos de l’utilisateur pour le compte de ce dernier, des connexions sécurisées peuvent être établies vers la Passerelle à partir de n’importe quel hôte client, y compris les clients reliés à des domaines AA-DDS différents de celui du cluster (ESP).

## <a name="best-practices"></a>Meilleures pratiques

La Passerelle est un service unique (avec équilibrage de charge entre deux hôtes) responsable du transfert des requêtes et de l’authentification. La Passerelle peut devenir un goulot d’étranglement en matière de débit pour les requêtes Hive qui dépassent une certaine taille. Une dégradation des performances des requêtes peut être observée lorsque des requêtes **SELECT** très volumineuses sont exécutées sur la Passerelle via ODBC ou JDBC. « Très volumineuses » désigne des requêtes qui représentent plus de 5 Go de données sur les lignes ou les colonnes. Cette requête peut inclure une longue liste de lignes ou un grand nombre de colonnes.

La dégradation des performances de la Passerelle autour des requêtes d’une taille importante est due au fait que les données doivent être transférées du magasin de données sous-jacent (ADLS Gen2) vers : le serveur Hive HDInsight, la Passerelle et enfin via les pilotes JDBC ou ODBC vers l’hôte client.

Le schéma suivant illustre les étapes impliquées dans une requête SELECT.

![Result Diagram](./media/gateway-best-practices/result-retrieval-diagram.png "Diagramme des résultats")

Apache Hive est une abstraction relationnelle sur un système de fichiers compatible HDFS. Cette abstraction signifie que les instructions **SELECT** dans Hive correspondent aux opérations **READ** sur le système de fichiers. Les opérations **READ** sont traduites dans le schéma approprié avant d’être signalées à l’utilisateur. La latence de ce processus augmente avec la taille des données et le nombre total de tronçons requis pour atteindre l’utilisateur final.

Un comportement similaire peut se produire lors de l’exécution d’instructions **CREATE** ou  **INSERT** de données volumineuses, car ces commandes correspondent aux opérations **WRITE** dans le système de fichiers sous-jacent. Envisagez d’écrire des données, telles que des ORC brutes, sur le système de fichiers ou le lac de données au lieu de les charger à l’aide des instructions **INSERT** ou **LOAD**.

Dans les clusters compatibles avec Pack Sécurité Entreprise, les stratégies Apache Ranger suffisamment complexes peuvent entraîner un ralentissement de la compilation des requêtes, ce qui peut conduire à une expiration du délai de la passerelle. Si une expiration du délai de la passerelle est constaté dans un cluster ESP, envisagez de réduire ou de combiner le nombre de stratégies Ranger.

## <a name="troubleshooting-techniques"></a>Techniques de résolution des problèmes

Il existe plusieurs moyens d’atténuer et de comprendre les problèmes de performance rencontrés dans le cadre du comportement ci-dessus. Utilisez la liste de contrôle suivante lorsque vous rencontrez une dégradation des performances des requêtes sur la passerelle HDInsight :

* Utilisez la clause **LIMIT** lors de l’exécution de requêtes **SELECT** volumineuses. La clause **LIMIT** permet de réduire le nombre total de lignes signalées à l’hôte client. La clause **LIMIT** concerne uniquement la génération de résultats et ne modifie pas le plan de requête. Pour appliquer la clause **LIMIT** au plan de requête, utilisez la configuration `hive.limit.optimize.enable`. **LIMIT** peut être combinée à un décalage à l’aide de la forme d’argument **LIMIT x,y**.

* Nommez les colonnes qui vous intéressent lorsque vous exécutez des requêtes **SELECT** au lieu d’utiliser **SELECT \** _. Si vous sélectionnez moins de colonnes, la quantité de données lues diminue.

_ Essayez d’exécuter la requête qui vous intéresse par le biais d’Apache Beeline. Si la récupération des résultats via Apache Beeline prend beaucoup de temps, attendez-vous à des retards lorsque vous récupérez les mêmes résultats via des outils externes.

* Testez une requête Hive de base pour vous assurer qu’une connexion à la Passerelle HDInsight peut être établie. Essayez d’exécuter une requête de base à partir de deux outils externes ou plus pour vous assurer qu’aucun outil ne rencontre de problème.

* Examinez toutes les alertes Apache Ambari pour vous assurer que les services HDInsight sont intègres. Les alertes Ambari peuvent être intégrées à Azure Monitor à des fins de création de rapports et d’analyse.

* Si vous utilisez un metastore Hive externe, vérifiez que la DTU de la base de données SQL Azure pour le metastore Hive n’a pas atteint sa limite. Si la DTU approche de sa limite, vous devez augmenter la taille de la base de données.

* Assurez-vous que tous les outils tiers tels que PBI ou Tableau utilisent la pagination pour afficher les tables ou les bases de données. Pour obtenir de l’aide au sujet de la pagination, contactez vos partenaires de support technique. Le principal outil utilisé pour la pagination est le paramètre `fetchSize` JDBC. Une petite taille d’extraction peut entraîner une dégradation des performances de la passerelle, mais une taille d’extraction trop importante peut entraîner une expiration du délai de la passerelle. Le paramétrage de la taille de l’extraction doit être effectué en fonction de la charge de travail.

* Si votre pipeline de données implique la lecture d’une grande quantité de données à partir du stockage sous-jacent du cluster HDInsight, envisagez d’utiliser un outil qui interagit directement avec Stockage Azure, par exemple Azure Data Factory.

* Envisagez d’utiliser Apache Hive LLAP lors de l’exécution de charges de travail interactives, car LLAP peut fournir une expérience plus fluide pour retourner rapidement des résultats de requête.

* Envisagez d’augmenter le nombre de threads disponibles pour le service Metastore Hive à l’aide de `hive.server2.thrift.max.worker.threads`. Ce paramètre est particulièrement utile lorsqu’un grand nombre d’utilisateurs simultanés envoient des requêtes au cluster.

* Réduisez le nombre de nouvelles tentatives utilisées pour atteindre la Passerelle à partir de n’importe quel outil externe. Si plusieurs nouvelles tentatives sont utilisées, envisagez de suivre une stratégie exponentielle d’interruption et de nouvelles tentatives.

* Envisagez d’activer la compression Hive à l’aide des configurations `hive.exec.compress.output` et `hive.exec.compress.intermediate`.

## <a name="next-steps"></a>Étapes suivantes

* [Apache Beeline sur HDInsight](../hadoop/apache-hadoop-use-hive-beeline.md)
* [Étapes de résolution du problème d’expiration du délai de la passerelle HDInsight](./troubleshoot-gateway-timeout.md)
* [Réseaux virtuels pour HDInsight](../hdinsight-plan-virtual-network-deployment.md)
* [HDInsight avec Express Route](../connect-on-premises-network.md)
---
title: Erreurs de connectivité temporaires - Azure Database pour MySQL
description: Découvrez comment gérer les erreurs de connectivité temporaires et vous connecter efficacement à Azure Database pour MySQL.
keywords: connexion mysql,chaîne de connexion,problèmes de connectivité,erreur temporaire,erreur de connexion, se connecter efficacement
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 89673c14c38947dc5aeb91cacde1eb2755e84138
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542607"
---
# <a name="handle-transient-errors-and-connect-efficiently-to-azure-database-for-mysql"></a>Gérer les erreurs temporaires et se connecter efficacement à Azure Database pour MySQL

Cet article explique comment gérer les erreurs temporaires et se connecter efficacement à Azure Database pour MySQL.

## <a name="transient-errors"></a>Erreurs temporaires

Une erreur temporaire, aussi connue sous le nom de défaut temporaire, est une erreur qui se résout d’elle-même. Le plus souvent, ces erreurs se manifestent sous forme de connexion au serveur de base de données en cours de suppression. De même, les nouvelles connexions à un serveur ne peuvent pas être ouvertes. Des erreurs temporaires peuvent se produire par exemple en cas de défaillance matérielle ou réseau. Elles peuvent également être dues au déploiement d'une nouvelle version de service PaaS. La plupart de ces événements sont automatiquement corrigés par le système en moins de 60 secondes. En matière de conception et de développement d'applications dans le cloud, il convient de s'attendre à des erreurs temporaires. Partez du principe qu’elles peuvent se produire dans n’importe quel composant à n’importe quel moment et qu'il existe une logique appropriée pour faire face à ces situations.

## <a name="handling-transient-errors"></a>Gestion des erreurs temporaires

Les erreurs temporaires doivent être gérées à l’aide de la logique de nouvelle tentative. Situations qui doivent être prises en compte :

* Une erreur se produit lorsque vous essayez d’ouvrir une connexion
* Une connexion inactive est supprimée côté serveur. Lorsque vous essayez d’émettre une commande, elle ne peut pas s’exécuter
* Une connexion active qui exécute actuellement une commande est supprimée.

Le premier et le deuxième cas sont assez simples à gérer. Essayez d’ouvrir à nouveau la connexion. Lorsque vous réussissez, l’erreur temporaire a été atténuée par le système. Vous pouvez utiliser de nouveau votre base de données Azure Database pour MySQL. Nous vous recommandons d’attendre avant de réessayer la connexion. Abandonnez si les tentatives initiales échouent. Ainsi, le système peut utiliser toutes les ressources disponibles pour résoudre la situation d’erreur. Nous vous recommandons de procéder comme suit :

* Patientez 5 secondes avant votre première nouvelle tentative.
* Pour chaque nouvelle tentative suivante, augmentez l’attente de façon exponentielle, jusqu’à 60 secondes.
* Définissez un nombre maximal de tentatives au-delà duquel votre application considère que l’opération a échoué.

Lorsqu’une connexion échoue avec une transaction active, il est plus difficile de gérer correctement la récupération. Il existe deux cas : Si la transaction était en lecture seule par nature, vous pouvez rouvrir la connexion et relancer la transaction. Toutefois, si la transaction a également été écrite dans la base de données, vous devez déterminer si elle a été restaurée ou si elle a réussi avant que l’erreur temporaire se produise. Dans ce cas, il se peut que vous n’ayez tout simplement pas reçu l’accusé de réception de validation du serveur de base de données.

Une façon d’y remédier est de générer un identifiant unique sur le client qui est utilisé pour toutes les tentatives. Vous transmettez cet ID unique dans le cadre de la transaction au serveur et vous le stockez dans une colonne avec une contrainte unique. Ainsi, vous pouvez retenter la transaction en toute sécurité. Elle aboutit si la transaction précédente a été restaurée et que l’ID unique généré par le client n’existe pas encore dans le système. Elle échoue et indique une violation de clé en double si l’ID unique a été précédemment stocké, car la transaction précédente s’est correctement terminée.

Si votre programme communique avec Azure Database pour MySQL par le biais d’un intergiciel (middleware) tiers, demandez au fournisseur s’il contient une logique de nouvelle tentative pour les erreurs temporaires.

Veillez à tester votre logique de nouvelle tentative. Par exemple, essayez d’exécuter votre code tout en effectuant un scale-up ou un scale-down des ressources de calcul de votre serveur Azure Database pour MySQL. Votre application doit être en mesure de gérer la courte interruption qui survient au cours de cette opération sans le moindre problème.

## <a name="connect-efficiently-to-azure-database-for-mysql"></a>Se connecter efficacement à Azure Database pour MySQL

Les connexions de base de données constituent une ressource limitée. En utilisant efficacement le regroupement de connexions pour accéder à Azure Database pour MySQL, vous pourrez donc optimiser les performances. La section ci-dessous explique comment utiliser le regroupement de connexions ou des connexions persistantes pour accéder plus efficacement à Azure Database pour MySQL.

## <a name="access-databases-by-using-connection-pooling-recommended"></a>Accéder aux bases de données à l’aide d’un regroupement de connexions (recommandé)

La gestion des connexions de base de données peut avoir un impact significatif sur les performances de l’application dans son ensemble. Pour optimiser les performances de votre application, vous devez avoir pour objectif de réduire le nombre de fois où les connexions sont établies et le temps nécessaire à l’établissement des connexions dans les chemins de code clés. Nous vous recommandons vivement d’utiliser un regroupement de connexions de base de données ou des connexions persistantes pour vous connecter à Azure Database pour MySQL. Le regroupement de connexions de base de données gère la création, la gestion et l’allocation des connexions de base de données. Quand un programme demande une connexion de base de données, il ne crée pas une nouvelle connexion, mais hiérarchise l’allocation des connexions de base de données inactives existantes. Une fois que le programme a fini d’utiliser la connexion de base de données, cette connexion est récupérée en vue d’une utilisation ultérieure au lieu d’être simplement fermée.

À des fins d’illustration, cet article inclut [un échantillon de code](./sample-scripts-java-connection-pooling.md) en JAVA. Pour plus d’informations, consultez la page sur [Apache Commons DBCP](https://commons.apache.org/proper/commons-dbcp/).

> [!NOTE]
> Le serveur configure un mécanisme de délai d’expiration pour fermer une connexion à l’état inactif depuis un certain temps afin de libérer des ressources. Veillez à configurer le système de vérification pour vérifier l’efficacité des connexions persistantes quand vous les utilisez. Pour plus d’informations, consultez la section [Configurer les mécanismes de vérification côté clients pour vérifier l’efficacité des connexions persistantes](concepts-connectivity.md#configure-verification-mechanisms-in-clients-to-confirm-the-effectiveness-of-persistent-connections).

## <a name="access-databases-by-using-persistent-connections-recommended"></a>Accéder aux bases de données à l’aide de connexions persistantes (recommandé)

Le concept de connexions persistantes est similaire à celui du regroupement de connexions. Le remplacement de connexions courtes par des connexions persistantes nécessite uniquement des modifications de code mineures. Il améliore cependant les performances de façon considérable dans de nombreux scénarios d’application courants.

## <a name="access-databases-by-using-wait-and-retry-mechanism-with-short-connections"></a>Accéder aux bases de données à l’aide du mécanisme d’attente et de nouvelle tentative avec des connexions courtes

Si vous êtes limité en ressources, nous vous recommandons vivement d’utiliser le regroupement de bases de données ou les connexions persistantes pour accéder aux bases de données. Si votre application utilise des connexions courtes et rencontre des échecs de connexion quand vous approchez le nombre maximal de connexions simultanées, vous pouvez essayer le mécanisme d’attente et de nouvelle tentative. Vous pouvez définir un délai d’attente approprié, avec un délai plus court après la première tentative. Ensuite, vous pouvez essayer d’attendre les événements plusieurs fois.

## <a name="configure-verification-mechanisms-in-clients-to-confirm-the-effectiveness-of-persistent-connections"></a>Configurer les mécanismes de vérification côté clients pour vérifier l’efficacité des connexions persistantes

Le serveur configure un mécanisme de délai d’expiration pour fermer une connexion à l’état inactif depuis un certain temps afin de libérer des ressources. Un nouvel accès à la base de données par le client revient à créer une demande de connexion entre le client et le serveur. Pour vérifier l’efficacité des connexions tout au long de leur utilisation, configurez un mécanisme de vérification sur le client. Comme indiqué dans l’exemple suivant, vous pouvez utiliser le regroupement de connexions JDBC Tomcat pour configurer ce mécanisme de vérification.

Si vous définissez le paramètre TestOnBorrow, en cas de nouvelle demande, le pool de connexions vérifie automatiquement l’efficacité de toutes les connexions inactives disponibles. Si l’une de ces connexions est efficace, elle est retournée directement. Celles qui ne le sont pas sont retirées par le pool de connexions, qui crée alors une connexion efficace et la retourne. Ce processus garantit un accès efficace à la base de données. 

Pour plus d’informations sur les différents paramètres, consultez le [document officiel de présentation du pool de connexions JDBC](https://tomcat.apache.org/tomcat-7.0-doc/jdbc-pool.html#Common_Attributes). Vous devez principalement définir les trois paramètres suivants : TestOnBorrow (avec la valeur true), ValidationQuery (avec la valeur SELECT 1) et ValidationQueryTimeout (avec la valeur 1). L’exemple de code spécifique est fourni ci-dessous :

```java
public class SimpleTestOnBorrowExample {
      public static void main(String[] args) throws Exception {
          PoolProperties p = new PoolProperties();
          p.setUrl("jdbc:mysql://localhost:3306/mysql");
          p.setDriverClassName("com.mysql.jdbc.Driver");
          p.setUsername("root");
          p.setPassword("password");
            // The indication of whether objects will be validated by the idle object evictor (if any). 
            // If an object fails to validate, it will be dropped from the pool. 
            // NOTE - for a true value to have any effect, the validationQuery or validatorClassName parameter must be set to a non-null string. 
          p.setTestOnBorrow(true); 

            // The SQL query that will be used to validate connections from this pool before returning them to the caller.
            // If specified, this query does not have to return any data, it just can't throw a SQLException.
          p.setValidationQuery("SELECT 1");

            // The timeout in seconds before a connection validation queries fail. 
            // This works by calling java.sql.Statement.setQueryTimeout(seconds) on the statement that executes the validationQuery. 
            // The pool itself doesn't timeout the query, it is still up to the JDBC driver to enforce query timeouts. 
            // A value less than or equal to zero will disable this feature.
          p.setValidationQueryTimeout(1);
            // set other useful pool properties.
          DataSource datasource = new DataSource();
          datasource.setPoolProperties(p);

          Connection con = null;
          try {
            con = datasource.getConnection();
            // execute your query here
          } finally {
            if (con!=null) try {con.close();}catch (Exception ignore) {}
          }
      }
  }
```

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes de connexion à la base de données Azure Database pour MySQL](howto-troubleshoot-common-connection-issues.md)

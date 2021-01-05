---
title: Questions fréquentes (FAQ) sur les technologies open source
description: Obtenez des réponses aux questions fréquentes sur les technologies open source d’Azure App Service.
author: genlin
manager: dcscontentpm
tags: top-support-issue
ms.assetid: 2fa5ee6b-51a6-4237-805f-518e6c57d11b
ms.topic: article
ms.date: 10/31/2018
ms.author: genli
ms.custom: seodec18, devx-track-python
ms.openlocfilehash: 36dfbf0fda060a8f273fee64098d6234b575088c
ms.sourcegitcommit: beacda0b2b4b3a415b16ac2f58ddfb03dd1a04cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/31/2020
ms.locfileid: "97831837"
---
# <a name="open-source-technologies-faqs-for-web-apps-in-azure"></a>FAQ sur les technologies open source pour Azure Web Apps

Cet article contient des réponses aux questions fréquemment posées sur les problèmes liés aux technologies open source de la [fonctionnalité Web Apps d’Azure App Service](https://azure.microsoft.com/services/app-service/web/).

[!INCLUDE [support-disclaimer](../../includes/support-disclaimer.md)]

## <a name="how-do-i-turn-on-php-logging-to-troubleshoot-php-issues"></a>Comment activer la journalisation PHP pour résoudre les problèmes PHP ?

Pour activer la journalisation PHP :

1. Connectez-vous à votre **site web Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
2. Dans le menu principal, sélectionnez **Console de débogage** > **CMD**.
3. Sélectionnez le dossier **Site**.
4. Sélectionnez le dossier **wwwroot**.
5. Sélectionnez l’icône **+** , puis sélectionnez **Nouveau fichier**.
6. Définissez le nom de fichier sur **.user.ini**.
7. Sélectionnez l’icône du crayon en regard de **.user.ini**.
8. Dans le fichier, ajoutez ce code : `log_errors=on`
9. Sélectionnez **Enregistrer**.
10. Sélectionnez l’icône du crayon en regard de **wp-config.php**.
11. Remplacez le texte par le code suivant :
    ```php
    //Enable WP_DEBUG modedefine('WP_DEBUG', true);//Enable debug logging to /wp-content/debug.logdefine('WP_DEBUG_LOG', true);
    //Suppress errors and warnings to screendefine('WP_DEBUG_DISPLAY', false);//Suppress PHP errors to screenini_set('display_errors', 0);
    ```
12. Dans le portail Azure, dans le menu de l’application web, redémarrez votre application web.

Pour plus d’informations, consultez [Activer les journaux d’activité d’erreurs WordPress](/archive/blogs/azureossds/logging-php-errors-in-wordpress-2).

## <a name="how-do-i-log-python-application-errors-in-apps-that-are-hosted-in-app-service"></a>Comment journaliser les erreurs d’application Python dans des applications hébergées dans App Service ?
[!INCLUDE [web-sites-python-troubleshooting-wsgi-error-log](../../includes/web-sites-python-troubleshooting-wsgi-error-log.md)]

## <a name="how-do-i-change-the-version-of-the-nodejs-application-that-is-hosted-in-app-service"></a>Comment modifier la version de l’application Node.js hébergée dans App Service ?

Pour modifier la version de l’application Node.js, vous pouvez utiliser l’une des options suivantes :

* Dans le portail Azure, utilisez **Paramètres de l’application**.
  1. Dans le portail Azure, accédez à votre application web.
  2. Dans le panneau **Paramètres**, sélectionnez **Paramètres de l’application**.
  3. Dans **Paramètres de l’application**, vous pouvez inclure WEBSITE_NODE_DEFAULT_VERSION en tant que clé et la version de Node.js souhaitée en tant que valeur.
  4. Accédez à votre **console Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net`).
  5. Pour vérifier la version de Node.js, entrez la commande suivante :  
     ```
     node -v
     ```
* Modifiez le fichier iisnode.yml. La modification de la version de Node.js dans le fichier iisnode.yml configure uniquement l’environnement d’exécution utilisé par iisnode. Votre cmd Kudu et le reste utilisent toujours la version de Node.js définie sous **Paramètres de l’application** dans le portail Azure.

  Pour définir le fichier iisnode.yml manuellement, créez un fichier iisnode.yml dans le dossier racine de votre application. Incluez la ligne suivante dans le fichier :
  ```yml
  nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
  ```
   
* Définissez le fichier iisnode.yml à l’aide de package.json durant le déploiement de contrôle de code source.
  Le processus de déploiement de contrôle de code source Azure implique les étapes suivantes :
  1. Déplace le contenu vers l’application web Azure.
  2. Crée un script de déploiement par défaut, s’il n’en existe pas (fichiers deploy.cmd, .deployment) dans le dossier racine de l’application web.
  3. Exécute un script de déploiement dans lequel il crée un fichier iisnode.yml si vous indiquez la version de Node.js dans le fichier package.json > moteur `"engines": {"node": "5.9.1","npm": "3.7.3"}`
  4. Le fichier iisnode.yml contient la ligne de code suivante :
      ```yml
      nodeProcessCommandLine: "D:\Program Files (x86)\nodejs\5.9.1\node.exe"
      ```

## <a name="i-see-the-message-error-establishing-a-database-connection-in-my-wordpress-app-thats-hosted-in-app-service-how-do-i-troubleshoot-this"></a>Le message « Erreur lors de l’établissement d’une connexion à la base de données » s’affiche dans mon application WordPress qui est hébergée dans App Service. Comment puis-je résoudre ce problème ?

Si cette erreur s’affiche dans votre application Azure WordPress, exécutez les étapes détaillées dans [Activer les journaux d’activité d’erreurs WordPress](/archive/blogs/azureossds/logging-php-errors-in-wordpress-2) pour activer php_errors.log et debug.log.

Lorsque les journaux d’activité sont activés, reproduisez l’erreur, puis vérifiez les journaux d’activité pour voir si le nombre de connexions disponible est épuisé :
```
[09-Oct-2015 00:03:13 UTC] PHP Warning: mysqli_real_connect(): (HY000/1226): User ‘abcdefghijk79' has exceeded the ‘max_user_connections’ resource (current value: 4) in D:\home\site\wwwroot\wp-includes\wp-db.php on line 1454
```

Si cette erreur s’affiche dans vos fichiers debug.log ou php_errors.log, votre application dépasse le nombre de connexions. Si l’hébergement s’effectue sur ClearDB, vérifiez le nombre de connexions disponibles dans votre [plan de service](https://www.cleardb.com/pricing.view).

## <a name="how-do-i-debug-a-nodejs-app-thats-hosted-in-app-service"></a>Comment déboguer une application Node.js hébergée dans App Service ?

1.  Accédez à votre **console Kudu** (`https://*yourwebsitename*.scm.azurewebsites.net/DebugConsole`).
2.  Accédez à votre dossier de journaux d’activité d’application (D:\home\LogFiles\Application).
3.  Vérifiez le contenu du fichier logging_errors.txt.

## <a name="how-do-i-install-native-python-modules-in-an-app-service-web-app-or-api-app"></a>Comment installer des modules Python natifs dans une application web App Service ou une application API ?

Certains packages ne peuvent pas être installés à l’aide de PIP dans Azure. Le package n’est peut-être pas disponible dans l’index de package Python ou un compilateur peut être nécessaire (aucun compilateur n’est disponible sur l’ordinateur qui exécute l’application web dans App Service). Pour plus d’informations sur l’installation des modules natifs dans App Service Web Apps et API Apps, consultez [Installer des modules Python dans App Service](/archive/blogs/azureossds/install-native-python-modules-on-azure-web-apps-api-apps).

## <a name="how-do-i-deploy-a-django-app-to-app-service-by-using-git-and-the-new-version-of-python"></a>Comment déployer une application Django dans App Service à l’aide de Git et de la nouvelle version de Python ?

Pour plus d’informations sur l’installation de Django, consultez [Déploiement d’une application Django dans App Service](/archive/blogs/azureossds/deploying-django-app-to-azure-app-services-using-git-and-new-version-of-python).

## <a name="where-are-the-tomcat-log-files-located"></a>Où se trouvent les fichiers journaux Tomcat ?

Pour les déploiements personnalisés et la Place de marché Azure :

* Emplacement du dossier : D:\home\site\wwwroot\bin\apache-tomcat-8.0.33\logs
* Fichiers d’intérêt :
    * catalina.*aaaa-mm-jj*.log
    * host-manager.*aaaa-mm-jj*.log
    * localhost.*aaaa-mm-jj*.log
    * manager.*aaaa-mm-jj*.log
    * site_access_log.*aaaa-mm-jj*.log


Pour les déploiements **Paramètres de l’application** dans le portail :

* Emplacement du dossier : D:\home\LogFiles
* Fichiers d’intérêt :
    * catalina.*aaaa-mm-jj*.log
    * host-manager.*aaaa-mm-jj*.log
    * localhost.*aaaa-mm-jj*.log
    * manager.*aaaa-mm-jj*.log
    * site_access_log.*aaaa-mm-jj*.log

## <a name="how-do-i-troubleshoot-jdbc-driver-connection-errors"></a>Comment résoudre les erreurs de connexion du pilote JDBC ?

Le message suivant peut s’afficher dans vos journaux d’activité Tomcat :

```
The web application[ROOT] registered the JDBC driver [com.mysql.jdbc.Driver] but failed to unregister it when the web application was stopped. To prevent a memory leak,the JDBC Driver has been forcibly unregistered
```

Pour résoudre l’erreur :

1. Supprimez le fichier sqljdbc*.jar du dossier app/lib.
2. Si vous utilisez le serveur web Tomcat ou Place de marché Azure Tomcat personnalisé, copiez ce fichier .jar dans le dossier lib de Tomcat.
3. Si vous activez Java à partir du portail Azure (sélectionnez **Java 1.8** > **Serveur Tomcat**), copiez le fichier sqljdbc.*jar dans le dossier parallèle à votre application. Ensuite, ajoutez le paramètre classpath suivant au fichier web.config :

    ```xml
    <httpPlatform>
    <environmentVariables>
    <environmentVariablename ="JAVA_OPTS" value=" -Djava.net.preferIPv4Stack=true
    -Xms128M -classpath %CLASSPATH%;[Path to the sqljdbc*.jarfile]" />
    </environmentVariables>
    </httpPlatform>
    ```

## <a name="why-do-i-see-errors-when-i-attempt-to-copy-live-log-files"></a>Pourquoi des erreurs s’affichent lorsque j’essaye de copier des fichiers journaux en direct ?

Si vous essayez de copier des fichiers journaux en direct pour une application Java (par exemple, Tomcat), cette erreur FTP peut s’afficher :

```
Error transferring file [filename] Copying files from remote side failed.
    
The process cannot access the file because it is being used by another process.
```

Le message d’erreur peut varier, selon le client FTP.

Ce problème de verrouillage concerne toutes les applications Java. Seul Kudu prend en charge le téléchargement de ce fichier pendant l’exécution de l’application.

Le fait d’arrêter l’application permet à FTP d’accéder à ces fichiers.

Une autre solution consiste à écrire une tâche web qui s’exécute selon une planification et copie ces fichiers dans un répertoire différent. Pour obtenir un exemple, consultez le projet [CopyLogsJob](https://github.com/kamilsykora/CopyLogsJob).

## <a name="where-do-i-find-the-log-files-for-jetty"></a>Où trouver les fichiers journaux pour Jetty ?

Pour les déploiements personnalisés et Place de marché Azure, le fichier journal se trouve dans le dossier D:\home\site\wwwroot\bin\jetty-distribution-9.1.2.v20140210\logs. Notez que l’emplacement du dossier dépend de la version de Jetty que vous utilisez. Par exemple, le chemin d’accès fourni ici s’applique à Jetty 9.1.2. Recherchez jetty_ *AAAA_MM_JJ*.stderrout.log.

Pour les déploiements Paramètre de l’application dans le portail, le fichier journal se trouve sous D:\home\LogFiles. Recherchez jetty_ *AAAA_MM_JJ*.stderrout.log

## <a name="can-i-send-email-from-my-azure-web-app"></a>Puis-je envoyer un e-mail à partir de mon application web Azure ?

App Service ne dispose pas d’une fonctionnalité de messagerie intégrée. Consultez cette [discussion sur le dépassement de capacité de la file](https://stackoverflow.com/questions/17666161/sending-email-from-azure) afin d’obtenir des solutions alternatives pour envoyer un e-mail à partir de votre application.

## <a name="why-does-my-wordpress-site-redirect-to-another-url"></a>Pourquoi mon site WordPress me redirige-t-il vers une autre URL ?

Si vous avez récemment migré vers Azure, il est possible que WordPress vous redirige vers l’ancienne URL du domaine. Cela est dû à un paramètre dans la base de données MySQL.

WordPress Buddy+ est une extension de site Azure que vous pouvez utiliser pour mettre à jour l’URL de redirection directement dans la base de données. Pour plus d’informations sur l’utilisation de WordPress Buddy+, consultez [Outils WordPress et migration MySQL avec WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows).

Ou si vous préférez mettre à jour manuellement l’URL de redirection à l’aide de requêtes SQL ou PHPMyAdmin, consultez [WordPress : redirection vers une URL incorrecte](/archive/blogs/azureossds/wordpress-redirecting-to-wrong-url).

## <a name="how-do-i-change-my-wordpress-sign-in-password"></a>Comment modifier mon mot de passe de connexion à WordPress ?

Si vous avez oublié votre mot de passe de connexion à WordPress, vous pouvez utiliser WordPress Buddy+ pour le mettre à jour. Pour réinitialiser votre mot de passe, installez l’extension de site Azure WordPress Buddy+, puis effectuez les étapes décrites dans [Outils WordPress et migration MySQL avec WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows).

## <a name="i-cant-sign-in-to-wordpress-how-do-i-resolve-this"></a>Je ne peux pas me connecter à WordPress. Comment résoudre ce problème ?

Si vous ne pouvez pas accéder à WordPress après l’installation d’un plug-in, celui-ci est peut-être défectueux. WordPress Buddy+ est une extension de site Azure qui peut vous aider à désactiver des plug-ins dans WordPress. Pour plus d’informations, consultez [Outils WordPress et migration MySQL avec WordPress Buddy+](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows).

## <a name="how-do-i-migrate-my-wordpress-database"></a>Comment migrer ma base de données WordPress ?

Vous disposez de plusieurs options pour migrer la base de données MySQL connectée à votre site web WordPress :

* Développeurs : utilisez [l’invite de commandes ou PHPMyAdmin](/archive/blogs/azureossds/migrating-data-between-mysql-databases-using-kudu-console-azure-app-service)
* Non-développeurs : Utilisez [WordPress Buddy +](https://www.electrongeek.com/blog/2016/12/21/wordpress-buddy-site-extension-for-app-service-on-windows)

## <a name="how-do-i-help-make-wordpress-more-secure"></a>Comment sécuriser davantage WordPress ?

Pour en savoir plus sur les meilleures pratiques de sécurité pour WordPress, consultez [Meilleures pratiques relatives à la sécurité de WordPress dans Azure](/archive/blogs/azureossds/best-practices-for-wordpress-security-on-azure).

## <a name="i-am-trying-to-use-phpmyadmin-and-i-see-the-message-access-denied-how-do-i-resolve-this"></a>J’essaye d’utiliser PHPMyAdmin et le message « Accès refusé » s’affiche. Comment résoudre ce problème ?

Vous pouvez rencontrer ce problème si la fonctionnalité MySQL dans l’application n’est pas encore exécutée dans cette instance d’App Service. Pour résoudre ce problème, essayez d’accéder à votre site web. Ceci démarre les processus requis, notamment le processus MySQL dans l’application. Pour vérifier que MySQL dans l’application est en cours d’exécution, vérifiez que mysqld.exe est répertorié dans les processus dans l’Explorateur de processus.

Après vous être assuré que MySQL dans l’application est en cours d’exécution, essayez d’utiliser PHPMyAdmin.

## <a name="i-get-an-http-403-error-when-i-try-to-import-or-export-my-mysql-in-app-database-by-using-phpmyadmin-how-do-i-resolve-this"></a>J’obtiens une erreur HTTP 403 lorsque j’essaie d’importer ou d’exporter ma base de données MySQL dans l’application à l’aide de PHPMyadmin. Comment résoudre ce problème ?

Si vous utilisez une version antérieure de Chrome, vous rencontrez sans doute un bogue connu. Pour résoudre ce problème, mettez à niveau vers une version plus récente de Chrome. Essayez également d’utiliser un autre navigateur, comme Internet Explorer ou Microsoft Edge, dans lequel le problème ne se produit pas.
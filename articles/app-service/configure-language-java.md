---
title: Configurer des applications Java
description: Découvrez comment configurer des applications Java à exécuter sur Azure App Service. Cet article présente les tâches de configuration les plus courantes.
keywords: azure app service, application web, windows, oss, java, tomcat, jboss
author: jasonfreeberg
ms.devlang: java
ms.topic: article
ms.date: 04/12/2019
ms.author: jafreebe
ms.reviewer: cephalin
ms.custom: seodec18, devx-track-java, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
ms.openlocfilehash: 0334e259f75440cae25f1e165c0621c85f7c7705
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97804001"
---
# <a name="configure-a-java-app-for-azure-app-service"></a>Configurer une application Java pour Azure App Service

Azure App Service permet aux développeurs Java de rapidement générer, déployer et mettre à l’échelle leurs applications web Java SE, Tomcat et JBoss EAP sur un service complètement managé. Déployez des applications avec les plug-ins Maven à partir de la ligne de commande ou dans des éditeurs comme IntelliJ, Eclipse ou Visual Studio Code.

Ce guide fournit les concepts et instructions clés aux développeurs Java qui utilisent App Service. Si vous n’avez jamais utilisé Azure App Service, commencez par lire [Démarrage rapide avec Java](quickstart-java.md). Des questions générales sur l’utilisation d’App Service qui ne sont pas spécifiques au développement Java sont traitées dans [Questions fréquentes (FAQ) sur App Service](faq-configuration-and-management.md).

## <a name="deploying-your-app"></a>Déploiement de votre application

Vous pouvez utiliser le [plug-in Azure Web App pour Maven](https://github.com/microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md) pour déployer vos fichiers .war ou .jar. Le déploiement avec des IDE populaires est également pris en charge avec [Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/) ou [Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse).

Sinon, votre méthode de déploiement dépend du type de votre archive :

### <a name="java-se"></a>Java SE

Pour déployer des fichiers .jar dans Java SE, utilisez le point de terminaison `/api/zipdeploy/` du site Kudu. Pour plus d’informations sur cette API, voir [cette documentation](./deploy-zip.md#rest).

### <a name="tomcat"></a>Tomcat

Pour déployer des fichiers .war sur Tomcat, utilisez le point de terminaison `/api/wardeploy/` pour effectuer un POST de votre fichier d’archive. Pour plus d’informations sur cette API, voir [cette documentation](./deploy-zip.md#deploy-war-file).

::: zone pivot="platform-linux"

### <a name="jboss-eap"></a>JBoss EAP

Pour déployer des fichiers .war sur JBoss, utilisez le point de terminaison `/api/wardeploy/` pour effectuer un POST de votre fichier d’archive. Pour plus d’informations sur cette API, voir [cette documentation](./deploy-zip.md#deploy-war-file).

Pour déployer des fichiers .ear, [utilisez FTP](deploy-ftp.md).

::: zone-end

Ne déployez pas votre fichier .war ou .jar en utilisant un FTP. L’outil FTP est conçu pour charger des scripts de démarrage, des dépendances ou d’autres fichiers de runtime. Il ne s’agit pas du meilleur choix pour le déploiement d’applications web.

## <a name="logging-and-debugging-apps"></a>Journalisation et débogage des applications

Vous trouverez des rapports de performances, des visualisations de trafic et des contrôles d’intégrité pour chaque application dans le portail Azure. Pour plus d’informations, consultez [Présentation des diagnostics Azure App Service](overview-diagnostics.md).

### <a name="stream-diagnostic-logs"></a>Diffuser les journaux de diagnostic

::: zone pivot="platform-windows"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

::: zone-end
::: zone pivot="platform-linux"

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-linux-no-h.md)]

::: zone-end

Pour plus d’informations, consultez [Diffuser des journaux dans Cloud Shell](troubleshoot-diagnostic-logs.md#in-cloud-shell).

::: zone pivot="platform-linux"

### <a name="ssh-console-access"></a>Accès à la console SSH

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

### <a name="troubleshooting-tools"></a>Outils de résolution des problèmes

Les images Java intégrées sont basées sur le système d’exploitation [Alpine Linux](https://alpine-linux.readthedocs.io/en/latest/getting_started.html). Utilisez le gestionnaire de package `apk` pour installer des outils ou commandes de résolution des problèmes.

::: zone-end

### <a name="flight-recorder"></a>Boîte noire SQL

Tous les runtimes Java sur App Service qui utilisent des JVM Azul sont fournis avec la boîte noire Zulu. Vous pouvez l’utiliser pour enregistrer les événements JVM, système et d’application, ainsi que pour résoudre les problèmes dans vos applications Java.

::: zone pivot="platform-windows"

#### <a name="timed-recording"></a>Enregistrement chronométré

Pour effectuer un enregistrement programmé, vous aurez besoin du PID (ID de processus) de l’application Java. Pour rechercher le PID, ouvrez un navigateur sur le site GCL de votre application web à l’adresse https://<votre-nom-de-site>.scm.azurewebsites.net/ProcessExplorer/. Cette page affiche les processus en cours d’exécution dans votre application web. Recherchez le processus nommé « java » dans le tableau et copiez le PID (ID de processus) correspondant.

Ouvrez ensuite la **Console de débogage** dans la barre d’outils supérieure du site GCL et exécutez la commande suivante. Remplacez `<pid>` par l’ID de processus que vous avez copié précédemment. Cette commande démarre un enregistrement de 30 secondes du profileur de votre application Java et génère un fichier nommé `timed_recording_example.jfr` dans le répertoire `D:\home`.

```
jcmd <pid> JFR.start name=TimedRecording settings=profile duration=30s filename="D:\home\timed_recording_example.JFR"
```

::: zone-end
::: zone pivot="platform-linux"

Connectez-vous avec SSH à votre instance App Service et exécutez la commande `jcmd` pour afficher la liste de tous les processus Java en cours d’exécution. En plus de jcmd lui-même, vous devez voir votre application Java en cours d’exécution avec un numéro d’identification de processus (pid).

```shell
078990bbcd11:/home# jcmd
Picked up JAVA_TOOL_OPTIONS: -Djava.net.preferIPv4Stack=true
147 sun.tools.jcmd.JCmd
116 /home/site/wwwroot/app.jar
```

Exécutez la commande suivante pour démarrer un enregistrement de 30 secondes de la machine virtuelle Java. La machine virtuelle Java est profilée et un fichier JFR nommé *jfr_example.jfr* est créé dans le répertoire d’accueil. (Remplacez 116 par le pid de votre application Java.)

```shell
jcmd 116 JFR.start name=MyRecording settings=profile duration=30s filename="/home/jfr_example.jfr"
```

Pendant l’intervalle de 30 secondes, vous pouvez vérifier le déroulement de l’enregistrement en exécutant `jcmd 116 JFR.check`. Cette commande affiche tous les enregistrements du processus Java donné.

#### <a name="continuous-recording"></a>Enregistrement continu

Vous pouvez utiliser la boîte noire SQL Zulu pour profiler en continu votre application Java avec un impact minimal sur les performances du runtime ([source](https://assets.azul.com/files/Zulu-Mission-Control-data-sheet-31-Mar-19.pdf)). Pour ce faire, exécutez la commande d’Azure CLI suivante pour créer un paramètre d’application nommé JAVA_OPTS avec la configuration nécessaire. Le contenu du paramètre d’application JAVA_OPTS est transmis à la commande `java` au démarrage de votre application.

```azurecli
az webapp config appsettings set -g <your_resource_group> -n <your_app_name> --settings JAVA_OPTS=-XX:StartFlightRecording=disk=true,name=continuous_recording,dumponexit=true,maxsize=1024m,maxage=1d
```

Une fois l’enregistrement démarré, vous pouvez vider les données d’enregistrement à tout moment à l’aide de la commande `JFR.dump`.

```shell
jcmd <pid> JFR.dump name=continuous_recording filename="/home/recording1.jfr"
```

::: zone-end

#### <a name="analyze-jfr-files"></a>Analyser les fichiers `.jfr`

Utilisez [FTPS](deploy-ftp.md) pour télécharger votre fichier JFR sur votre ordinateur local. Pour analyser le fichier JFR, téléchargez et installez [Zulu Mission Control](https://www.azul.com/products/zulu-mission-control/). Pour obtenir des instructions sur Zulu Mission Control, consultez la [documentation d’Azul](https://docs.azul.com/zmc/) et les [instructions d’installation](/java/azure/jdk/java-jdk-flight-recorder-and-mission-control).

### <a name="app-logging"></a>Journalisation des applications

::: zone pivot="platform-windows"

Activez [Journal des applications](troubleshoot-diagnostic-logs.md#enable-application-logging-windows) via le portail Azure ou [Azure CLI](/cli/azure/webapp/log#az-webapp-log-config) pour configurer App Service de sorte à écrire la sortie de console standard de votre application et les flux d’erreur de console standard dans le système de fichiers local ou le service Stockage Blob Azure. La journalisation sur l’instance du système de fichiers App Service locale est désactivée 12 heures après avoir été configurée. Si vous en avez besoin plus longtemps, configurez l’application pour écrire la sortie sur un conteneur de stockage d’objets blob. Vous trouverez vos journaux d’application Java et Tomcat dans le répertoire */home/LogFiles/Application/* .

::: zone-end
::: zone pivot="platform-linux"

Activez [Journal des applications](troubleshoot-diagnostic-logs.md#enable-application-logging-linuxcontainer) via le portail Azure ou [Azure CLI](/cli/azure/webapp/log#az-webapp-log-config) pour configurer App Service de sorte à écrire la sortie de console standard de votre application et les flux d’erreur de console standard dans le système de fichiers local ou le service Stockage Blob Azure. Si vous en avez besoin plus longtemps, configurez l’application pour écrire la sortie sur un conteneur de stockage d’objets blob. Vous trouverez vos journaux d’application Java et Tomcat dans le répertoire */home/LogFiles/Application/* .

La journalisation du Stockage Blob Azure pour les services App Services Linux ne peut être configurée qu’avec [Azure Monitor (préversion)](./troubleshoot-diagnostic-logs.md#send-logs-to-azure-monitor-preview). 

::: zone-end

Si votre application utilise [Logback](https://logback.qos.ch/) ou [Log4j](https://logging.apache.org/log4j) pour le traçage, vous pouvez transférer ces traces pour révision vers Azure Application Insights en suivant les instructions de configuration des frameworks de journalisation dans [Exploration des journaux d’activité de traces Java dans Application Insights](../azure-monitor/app/java-trace-logs.md).

## <a name="customization-and-tuning"></a>Personnalisation et réglage

Azure App Service pour Linux prend en charge le réglage et la personnalisation prêts à l’emploi par le biais du portail Azure et de l’interface CLI. Consultez les articles suivants pour configurer des applications web spécifiques non-Java :

- [Configurer les paramètres d’application](configure-common.md#configure-app-settings)
- [Configurer un nom de domaine personnalisé](app-service-web-tutorial-custom-domain.md)
- [Configurer des liaisons SSL](configure-ssl-bindings.md)
- [Ajouter un CDN](../cdn/cdn-add-to-web-app.md)
- [Configurer le site Kudu](https://github.com/projectkudu/kudu/wiki/Configurable-settings#linux-on-app-service-settings)


### <a name="set-java-runtime-options"></a>Définir les options de runtime Java

Pour définir la mémoire allouée ou d’autres options de runtime JVM, créez un [paramètre d’application](configure-common.md#configure-app-settings) nommé `JAVA_OPTS` avec les options. App Service transmet ce paramètre comme variable d’environnement au runtime Java quand il démarre.

Dans le portail Azure, sous **Paramètres d’application** de l’application web, créez un paramètre d’application appelé `JAVA_OPTS` qui contient les paramètres supplémentaires tels que `-Xms512m -Xmx1204m`.

Pour configurer le paramètre d’application à partir du plug-in Maven, ajoutez des étiquettes paramètre/valeur dans la section du plug-in Azure. L’exemple suivant définit une taille de segment de mémoire Java minimale et maximale spécifique :

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Xms512m -Xmx1204m</value>
    </property>
</appSettings>
```

Les développeurs exécutant une seule application avec un seul emplacement de déploiement dans leur plan App Service peuvent utiliser les options suivantes :

- Instances B1 et S1 : `-Xms1024m -Xmx1024m`
- Instances B2 et S2 : `-Xms3072m -Xmx3072m`
- Instances B3 et S3 : `-Xms6144m -Xmx6144m`

Lors du réglage des paramètres de segment de mémoire de l’application, consultez les détails de votre plan App Service et prenez en compte qu’avec plusieurs applications et emplacements de déploiement, vous devez trouver l’allocation de mémoire optimale.

### <a name="turn-on-web-sockets"></a>Activer les sockets web

Activez la prise en charge des sockets web dans le portail Azure dans les **Paramètres d’application** de l’application. Vous devrez redémarrer l’application pour que le paramètre prenne effet.

Activez la prise en charge des sockets web via l’interface Azure CLI avec la commande suivante :

```azurecli-interactive
az webapp config set --name <app-name> --resource-group <resource-group-name> --web-sockets-enabled true
```

Redémarrez ensuite votre application :

```azurecli-interactive
az webapp stop --name <app-name> --resource-group <resource-group-name>
az webapp start --name <app-name> --resource-group <resource-group-name>
```

### <a name="set-default-character-encoding"></a>Définir l’encodage de caractères par défaut

Dans le portail Azure, sous **Paramètres d’application** de l’application web, créez un paramètre d’application appelé `JAVA_OPTS` avec la valeur `-Dfile.encoding=UTF-8`.

Vous pouvez aussi configurer le paramètre d’application à l’aide du plug-in Maven App Service. Ajoutez le nom du paramètre et les étiquettes des valeurs dans la configuration du plug-in :

```xml
<appSettings>
    <property>
        <name>JAVA_OPTS</name>
        <value>-Dfile.encoding=UTF-8</value>
    </property>
</appSettings>
```

### <a name="pre-compile-jsp-files"></a>Précompiler les fichiers JSP

Pour améliorer les performances des applications Tomcat, vous pouvez compiler vos fichiers JSP avant le déploiement sur App Service. Vous pouvez utiliser le [plug-in Maven](https://sling.apache.org/components/jspc-maven-plugin/plugin-info.html) fourni par Apache Sling, ou ce [fichier de build Ant](https://tomcat.apache.org/tomcat-9.0-doc/jasper-howto.html#Web_Application_Compilation).

## <a name="secure-applications"></a>Sécuriser les applications

Les applications Java exécutées dans App Service exigent les mêmes [bonnes pratiques de sécurité](../security/fundamentals/paas-applications-using-app-services.md) que les autres applications.

### <a name="authenticate-users-easy-auth"></a>Authentifier les utilisateurs (authentification facile)

Configurez l’authentification de l’application dans le portail Azure avec l’option **Authentification et autorisation**. À partir de là, vous pouvez activer l’authentification en utilisant Azure Active Directory ou des identifiants de réseaux sociaux tels que Facebook, Google ou GitHub. La configuration du portail Azure fonctionne seulement si vous configurez un seul fournisseur d’authentification. Pour plus d’informations, consultez [Configurer votre application App Service pour utiliser une connexion Azure Active Directory](configure-authentication-provider-aad.md) et les articles connexes sur d’autres fournisseurs d’identités. Si vous devez activer plusieurs fournisseurs de connexion, suivez les instructions de l’article [Personnaliser l’authentification App Service](app-service-authentication-how-to.md).

#### <a name="java-se"></a>Java SE

Les développeurs Spring Boot peuvent utiliser le [démarreur Spring Boot Azure Active Directory](/java/azure/spring-framework/configure-spring-boot-starter-java-app-with-azure-active-directory) pour sécuriser les applications à l’aide d’API et d’annotations Spring Security familières. Veillez à augmenter la taille d’en-tête maximale dans votre fichier *application.properties*. Nous vous suggérons une valeur de `16384`.

#### <a name="tomcat"></a>Tomcat

Votre application Tomcat peut accéder directement aux revendications de l’utilisateur à partir du servlet en forçant le type de l’objet Principal en objet Map. L’objet Map mappe chaque type de revendication avec une collection des revendications de ce type. Dans le code ci-dessous, `request` est une instance de `HttpServletRequest`.

```java
Map<String, Collection<String>> map = (Map<String, Collection<String>>) request.getUserPrincipal();
```

Vous pouvez maintenant inspecter l’objet `Map` pour une revendication spécifique. Par exemple, l’extrait de code suivant effectue une itération dans tous les types de revendications et imprime le contenu de chaque collection.

```java
for (Object key : map.keySet()) {
        Object value = map.get(key);
        if (value != null && value instanceof Collection {
            Collection claims = (Collection) value;
            for (Object claim : claims) {
                System.out.println(claims);
            }
        }
    }
```

Pour déconnecter les utilisateurs, utilisez le chemin `/.auth/ext/logout`. Pour effectuer d’autres actions, consultez la documentation sur l’[authentification et les autorisations App Service](./app-service-authentication-how-to.md). Il existe également une documentation officielle sur l’[interface HttpServletRequest](https://tomcat.apache.org/tomcat-5.5-doc/servletapi/javax/servlet/http/HttpServletRequest.html) de Tomcat et ses méthodes. Les méthodes servlet suivantes sont également alimentées en fonction de la configuration de votre App Service :

```java
public boolean isSecure()
public String getRemoteAddr()
public String getRemoteHost()
public String getScheme()
public int getServerPort()
```

Pour désactiver cette fonctionnalité, créez un paramètre d’application nommé `WEBSITE_AUTH_SKIP_PRINCIPAL` avec la valeur `1`. Pour désactiver tous les filtres de servlet ajoutés par App Service, créez un paramètre nommé `WEBSITE_SKIP_FILTERS` avec la valeur `1`.

### <a name="configure-tlsssl"></a>Configurer TLS/SSL

Suivez les instructions dans [Sécuriser un nom DNS personnalisé avec une liaison SSL dans Azure App Service](configure-ssl-bindings.md) pour télécharger un certificat SSL existant et le lier au nom de domaine de votre application. Par défaut, votre application autorisera toujours les connexions HTTP. Suivez les étapes spécifiques du tutoriel pour appliquer SSL et TLS.

### <a name="use-keyvault-references"></a>Utiliser des références KeyVault

[Azure Key Vault](../key-vault/general/overview.md) fournit une gestion des secrets centralisée avec des stratégies d’accès et un historique d’audit. Vous pouvez stocker des secrets (tels que des mots de passe ou chaînes de connexion) dans KeyVault et accéder à ces secrets dans votre application via des variables d’environnement.

Tout d’abord, suivez les instructions de [Autoriser votre application à accéder à Key Vault](app-service-key-vault-references.md#granting-your-app-access-to-key-vault) et de [création d’une référence KeyVault à votre secret dans un paramètre d’application](app-service-key-vault-references.md#reference-syntax). Vous pouvez vérifier la résolution de la référence en secret en imprimant la variable d’environnement tout en accédant à distance au terminal App Service.

Pour injecter ces secrets dans votre fichier de configuration Spring ou Tomcat, utilisez la syntaxe d’injection de variable d’environnement (`${MY_ENV_VAR}`). Pour les fichiers de configuration Spring, consultez cette documentation sur les [configurations externalisées](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html).

::: zone pivot="platform-linux"

### <a name="use-the-java-key-store"></a>Utiliser le magasin de clés Java

Par défaut, tous les certificats publics ou privés [chargés sur App Service Linux](configure-ssl-certificate.md) seront chargés dans les magasins de clés Java respectifs au démarrage du conteneur. Après avoir chargé votre certificat, vous devrez redémarrer votre App Service afin de le charger dans le Java Key Store. Les certificats publics sont chargés dans le magasin de clés sur `$JAVA_HOME/jre/lib/security/cacerts`, et les certificats privés sont stockés dans `$JAVA_HOME/lib/security/client.jks`.

Une configuration supplémentaire peut être nécessaire pour chiffrer votre connexion JDBC avec des certificats dans le magasin de clés Java. Reportez-vous à la documentation du pilote JDBC que vous avez choisi.

- [PostgreSQL](https://jdbc.postgresql.org/documentation/head/ssl-client.html)
- [SQL Server](/sql/connect/jdbc/connecting-with-ssl-encryption)
- [MySQL](https://dev.mysql.com/doc/connector-j/5.1/en/connector-j-reference-using-ssl.html)
- [MongoDB](https://mongodb.github.io/mongo-java-driver/3.4/driver/tutorials/ssl/)
- [Cassandra](https://docs.datastax.com/en/developer/java-driver/4.3/)

#### <a name="initialize-the-java-key-store"></a>Initialiser le magasin de clés Java

Pour initialiser l’objet `import java.security.KeyStore`, chargez le fichier de magasin de clés avec le mot de passe. Le mot de passe par défaut pour les deux magasins de clés est « changeit ».

```java
KeyStore keyStore = KeyStore.getInstance("jks");
keyStore.load(
    new FileInputStream(System.getenv("JAVA_HOME")+"/lib/security/cacets"),
    "changeit".toCharArray());

KeyStore keyStore = KeyStore.getInstance("pkcs12");
keyStore.load(
    new FileInputStream(System.getenv("JAVA_HOME")+"/lib/security/client.jks"),
    "changeit".toCharArray());
```

#### <a name="manually-load-the-key-store"></a>Charger manuellement le magasin de clés

Vous pouvez charger les certificats manuellement dans le magasin de clés. Créez un paramètre d'application, `SKIP_JAVA_KEYSTORE_LOAD`, avec une valeur de `1` pour désactiver le chargement automatique des certificats dans le magasin de clés. Tous les certificats publics chargés sur App Service via le portail Azure sont stockés sous `/var/ssl/certs/`. Les certificats privés sont stockés sous `/var/ssl/private/`.

Vous pouvez interagir ou déboguer l'outil Java Key Tool en [ouvrant une connexion SSH](configure-linux-open-ssh-session.md) sur votre App Service et en exécutant la commande `keytool`. Consultez la [documentation de Key Tool](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/keytool.html) pour obtenir une liste des commandes. Pour plus d'informations sur l'API KeyStore, reportez-vous à la [documentation officielle](https://docs.oracle.com/javase/8/docs/api/java/security/KeyStore.html).

::: zone-end

## <a name="configure-apm-platforms"></a>Configurer des plateformes d’APM

Cette section explique comment connecter des applications Java déployées sur Azure App Service sur Linux avec les plateformes de supervision des performances d’application (APM) NewRelic et AppDynamics.

### <a name="configure-new-relic"></a>Configurer New Relic

::: zone pivot="platform-windows"

1. Créez un compte NewRelic sur [NewRelic.com](https://newrelic.com/signup)
2. Téléchargez l’agent Java à partir de NewRelic, son nom doit ressembler à *newrelic-java-x.x.x.zip*.
3. Copiez votre clé de licence, vous en aurez besoin pour configurer l’agent par la suite.
4. [Connectez-vous avec SSH à votre instance App Service](configure-linux-open-ssh-session.md) et créez un répertoire */home/site/wwwroot/apm*.
5. Chargez les fichiers de l’agent Java de NewRelic décompressés dans un répertoire sous */home/site/wwwroot/apm*. Les fichiers de votre agent doivent se trouver dans */home/site/wwwroot/apm/newrelic*.
6. Modifiez le fichier YAML dans */home/site/wwwroot/apm/newrelic/newrelic.yml* et remplacez la valeur de la licence d’espace réservé par votre propre clé de licence.
7. Dans le portail Azure, accédez à votre application dans App Service et créez un paramètre d’application.

    - Pour les applications **Java SE**, créez une variable d’environnement nommée `JAVA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.
    - Pour **Tomcat**, créez une variable d’environnement nommée `CATALINA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.

::: zone-end
::: zone pivot="platform-linux"

1. Créez un compte NewRelic sur [NewRelic.com](https://newrelic.com/signup)
2. Téléchargez l’agent Java à partir de NewRelic, son nom doit ressembler à *newrelic-java-x.x.x.zip*.
3. Copiez votre clé de licence, vous en aurez besoin pour configurer l’agent par la suite.
4. [Connectez-vous avec SSH à votre instance App Service](configure-linux-open-ssh-session.md) et créez un répertoire */home/site/wwwroot/apm*.
5. Chargez les fichiers de l’agent Java de NewRelic décompressés dans un répertoire sous */home/site/wwwroot/apm*. Les fichiers de votre agent doivent se trouver dans */home/site/wwwroot/apm/newrelic*.
6. Modifiez le fichier YAML dans */home/site/wwwroot/apm/newrelic/newrelic.yml* et remplacez la valeur de la licence d’espace réservé par votre propre clé de licence.
7. Dans le portail Azure, accédez à votre application dans App Service et créez un paramètre d’application.
   
    - Pour les applications **Java SE**, créez une variable d’environnement nommée `JAVA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.
    - Pour **Tomcat**, créez une variable d’environnement nommée `CATALINA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/newrelic/newrelic.jar`.

::: zone-end

>  Si vous avez déjà une variable d’environnement pour `JAVA_OPTS` ou `CATALINA_OPTS`, ajoutez l’option `-javaagent:/...` à la fin de la valeur actuelle.

### <a name="configure-appdynamics"></a>Configurer AppDynamics

::: zone pivot="platform-windows"

1. Créez un compte AppDynamics sur [AppDynamics.com](https://www.appdynamics.com/community/register/)
2. Téléchargez l’agent Java à partir du site web AppDynamics, le nom de fichier ressemble à *AppServerAgent-x.x.x.xxxxx.zip*
3. Utilisez la [console Kudu](https://github.com/projectkudu/kudu/wiki/Kudu-console) pour créer un répertoire */home/site/wwwroot/apm*.
4. Chargez les fichiers de l’agent Java dans un répertoire sous */home/site/wwwroot/apm*. Les fichiers de votre agent doivent se trouver dans */home/site/wwwroot/apm/appdynamics*.
5. Dans le portail Azure, accédez à votre application dans App Service et créez un paramètre d’application.

   - Pour les applications **Java SE**, créez une variable d’environnement nommée `JAVA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>`, où `<app-name>` est le nom de votre instance App Service.
   - Pour les applications **Tomcat**, créez une variable d’environnement nommée `CATALINA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>`, où `<app-name>` est le nom de votre instance App Service.

::: zone-end
::: zone pivot="platform-linux"

1. Créez un compte AppDynamics sur [AppDynamics.com](https://www.appdynamics.com/community/register/)
2. Téléchargez l’agent Java à partir du site web AppDynamics, le nom de fichier ressemble à *AppServerAgent-x.x.x.xxxxx.zip*
3. [Connectez-vous avec SSH à votre instance App Service](configure-linux-open-ssh-session.md) et créez un répertoire */home/site/wwwroot/apm*.
4. Chargez les fichiers de l’agent Java dans un répertoire sous */home/site/wwwroot/apm*. Les fichiers de votre agent doivent se trouver dans */home/site/wwwroot/apm/appdynamics*.
5. Dans le portail Azure, accédez à votre application dans App Service et créez un paramètre d’application.

   - Pour les applications **Java SE**, créez une variable d’environnement nommée `JAVA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>`, où `<app-name>` est le nom de votre instance App Service.
   - Pour les applications **Tomcat**, créez une variable d’environnement nommée `CATALINA_OPTS` avec la valeur `-javaagent:/home/site/wwwroot/apm/appdynamics/javaagent.jar -Dappdynamics.agent.applicationName=<app-name>`, où `<app-name>` est le nom de votre instance App Service.

::: zone-end

> [!NOTE]
>  Si vous avez déjà une variable d’environnement pour `JAVA_OPTS` ou `CATALINA_OPTS`, ajoutez l’option `-javaagent:/...` à la fin de la valeur actuelle.

## <a name="configure-data-sources"></a>Configurer des sources de données

### <a name="java-se"></a>Java SE

Pour vous connecter à des sources de données dans des applications Spring Boot, nous vous suggérons de créer des chaînes de connexion et de les injecter dans votre fichier *application.properties*.

1. Dans la section « Configuration » de la page App Service, définissez un nom pour la chaîne, collez votre chaîne de connexion JDBC dans le champ de valeur, puis définissez le type sur « Personnalisé ». Vous pouvez éventuellement définir cette chaîne de connexion en tant que paramètre d’emplacement.

    Cette chaîne de connexion est accessible à notre application en tant que variable d’environnement nommée `CUSTOMCONNSTR_<your-string-name>`. Par exemple, la chaîne de connexion que nous avons créée ci-dessus sera nommée `CUSTOMCONNSTR_exampledb`.

2. Dans votre fichier *application.properties*, référencez cette chaîne de connexion avec le nom de variable d’environnement. Dans notre exemple, nous utiliserions ce qui suit.

    ```yml
    app.datasource.url=${CUSTOMCONNSTR_exampledb}
    ```

Consultez la [documentation Spring Boot sur l’accès aux données](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-data-access.html) et les [configurations externalisées](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html) pour plus d’informations à ce sujet.

::: zone pivot="platform-windows"

### <a name="tomcat"></a>Tomcat

Ces instructions s’appliquent à toutes les connexions de base de données. Vous devrez indiquer dans les espaces réservés le nom de la classe de pilote de la base de données que vous avez choisie ainsi que le fichier JAR. Vous disposez d’une table contenant des noms de classes et de téléchargements de pilotes pour les bases de données courantes.

| Base de données   | Nom de la classe du pilote                             | Pilote JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Télécharger](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Télécharger](https://dev.mysql.com/downloads/connector/j/) (sélectionnez « Indépendant de la plateforme ») |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Télécharger](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#download)                                                           |

Pour configurer Tomcat afin d’utiliser Java Database Connectivity (JDBC) ou l’API Java Persistence (JPA), commencez par personnaliser la variable d’environnement `CATALINA_OPTS` lue par Tomcat au démarrage. Définissez ces valeurs via un paramètre d’application dans le [plug-in Maven App Service](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md) :

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

Ou définissez les variables d’environnement dans la page **Configuration** > **Paramètres d’application** du portail Azure.

Ensuite, déterminez si la source de données doit être mise à la disposition d’une seule application ou de toutes les applications exécutées sur le servlet Tomcat.

#### <a name="application-level-data-sources"></a>Sources de données au niveau de l’application

1. Créez un fichier *context.xml* dans le répertoire *META-INF /* de votre projet. Créez le répertoire *META-INF/* s’il n’existe pas.

2. Dans *context.xml*, ajoutez un élément `Context` pour lier la source de données à une adresse JNDI. Remplacez l’espace réservé `driverClassName` par le nom de la classe de votre pilote à l’aide du tableau ci-dessus.

    ```xml
    <Context>
        <Resource
            name="jdbc/dbconnection"
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}"
            password="${connURL}"
        />
    </Context>
    ```

3. Mettez à jour le fichier *web.xml* de votre application pour utiliser la source de données dans votre application.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="finalize-configuration"></a>Finaliser la configuration

Pour finir, nous allons placer les fichiers du pilote JAR dans le classpath Tomcat et redémarrer votre App Service. Veillez à ce que les fichiers du pilote JDBC soient disponibles pour le chargeur de classes Tomcat en les mettant dans le répertoire */home/tomcat/lib*. (S’il n’existe pas déjà, créez ce répertoire.) Pour charger ces fichiers sur votre instance App Service, procédez comme suit :

1. Dans le [Cloud Shell](https://shell.azure.com), installez l’extension d’application web :

    ```azurecli-interactive
    az extension add -–name webapp
    ```

2. Exécutez la commande CLI suivante pour créer un tunnel SSH de votre système local vers App Service :

    ```azurecli-interactive
    az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
    ```

3. Connectez-vous au port de tunneling local avec votre client SFTP et chargez les fichiers dans le dossier */home/tomcat/lib*.

Vous pouvez également utiliser un client FTP pour charger le pilote JDBC. Suivez ces [instructions pour obtenir vos informations d’identification FTP](deploy-configure-credentials.md).

---

::: zone-end
::: zone pivot="platform-linux"

### <a name="tomcat"></a>Tomcat

Ces instructions s’appliquent à toutes les connexions de base de données. Vous devrez indiquer dans les espaces réservés le nom de la classe de pilote de la base de données que vous avez choisie ainsi que le fichier JAR. Vous disposez d’une table contenant des noms de classes et de téléchargements de pilotes pour les bases de données courantes.

| Base de données   | Nom de la classe du pilote                             | Pilote JDBC                                                                      |
|------------|-----------------------------------------------|------------------------------------------------------------------------------------------|
| PostgreSQL | `org.postgresql.Driver`                        | [Télécharger](https://jdbc.postgresql.org/download.html)                                    |
| MySQL      | `com.mysql.jdbc.Driver`                        | [Télécharger](https://dev.mysql.com/downloads/connector/j/) (sélectionnez « Indépendant de la plateforme ») |
| SQL Server | `com.microsoft.sqlserver.jdbc.SQLServerDriver` | [Télécharger](/sql/connect/jdbc/download-microsoft-jdbc-driver-for-sql-server?view=sql-server-2017#download)                                                           |

Pour configurer Tomcat afin d’utiliser Java Database Connectivity (JDBC) ou l’API Java Persistence (JPA), commencez par personnaliser la variable d’environnement `CATALINA_OPTS` lue par Tomcat au démarrage. Définissez ces valeurs via un paramètre d’application dans le [plug-in Maven App Service](https://github.com/Microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md) :

```xml
<appSettings>
    <property>
        <name>CATALINA_OPTS</name>
        <value>"$CATALINA_OPTS -Ddbuser=${DBUSER} -Ddbpassword=${DBPASSWORD} -DconnURL=${CONNURL}"</value>
    </property>
</appSettings>
```

Ou définissez les variables d’environnement dans la page **Configuration** > **Paramètres d’application** du portail Azure.

Ensuite, déterminez si la source de données doit être mise à la disposition d’une seule application ou de toutes les applications exécutées sur le servlet Tomcat.

#### <a name="application-level-data-sources"></a>Sources de données au niveau de l’application

1. Créez un fichier *context.xml* dans le répertoire *META-INF /* de votre projet. Créez le répertoire *META-INF/* s’il n’existe pas.

2. Dans *context.xml*, ajoutez un élément `Context` pour lier la source de données à une adresse JNDI. Remplacez l’espace réservé `driverClassName` par le nom de la classe de votre pilote à l’aide du tableau ci-dessus.

    ```xml
    <Context>
        <Resource
            name="jdbc/dbconnection"
            type="javax.sql.DataSource"
            url="${dbuser}"
            driverClassName="<insert your driver class name>"
            username="${dbpassword}"
            password="${connURL}"
        />
    </Context>
    ```

3. Mettez à jour le fichier *web.xml* de votre application pour utiliser la source de données dans votre application.

    ```xml
    <resource-env-ref>
        <resource-env-ref-name>jdbc/dbconnection</resource-env-ref-name>
        <resource-env-ref-type>javax.sql.DataSource</resource-env-ref-type>
    </resource-env-ref>
    ```

#### <a name="shared-server-level-resources"></a>Ressources au niveau du serveur partagées

Pour ajouter une source de données partagée au niveau du serveur, vous devez modifier le fichier server.xml de Tomcat. Tout d’abord, téléchargez un [script de démarrage](faq-app-service-linux.md#built-in-images) et définissez le chemin d’accès au script dans **Configuration** > **Commande de démarrage**. Vous pouvez télécharger le script de démarrage à l’aide de [FTP](deploy-ftp.md).

Votre script de démarrage crée une [transformation xsl](https://www.w3schools.com/xml/xsl_intro.asp) sur le fichier server.xml et génère le fichier xml résultant sur `/usr/local/tomcat/conf/server.xml`. Le script de démarrage doit installer libxslt via apk. Votre fichier xsl et votre script de démarrage peuvent être téléchargés via FTP. Vous trouverez ci-dessous un exemple de script de démarrage.

```sh
# Install libxslt. Also copy the transform file to /home/tomcat/conf/
apk add --update libxslt

# Usage: xsltproc --output output.xml style.xsl input.xml
xsltproc --output /home/tomcat/conf/server.xml /home/tomcat/conf/transform.xsl /usr/local/tomcat/conf/server.xml
```

Un exemple de fichier xsl est fourni ci-dessous. L’exemple de fichier xsl ajoute un nouveau nœud de connecteur au fichier server.xml Tomcat.

```xml
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
  <xsl:output method="xml" indent="yes"/>

  <xsl:template match="@* | node()" name="Copy">
    <xsl:copy>
      <xsl:apply-templates select="@* | node()"/>
    </xsl:copy>
  </xsl:template>

  <xsl:template match="@* | node()" mode="insertConnector">
    <xsl:call-template name="Copy" />
  </xsl:template>

  <xsl:template match="comment()[not(../Connector[@scheme = 'https']) and
                                 contains(., '&lt;Connector') and
                                 (contains(., 'scheme=&quot;https&quot;') or
                                  contains(., &quot;scheme='https'&quot;))]">
    <xsl:value-of select="." disable-output-escaping="yes" />
  </xsl:template>

  <xsl:template match="Service[not(Connector[@scheme = 'https'] or
                                   comment()[contains(., '&lt;Connector') and
                                             (contains(., 'scheme=&quot;https&quot;') or
                                              contains(., &quot;scheme='https'&quot;))]
                                  )]
                      ">
    <xsl:copy>
      <xsl:apply-templates select="@* | node()" mode="insertConnector" />
    </xsl:copy>
  </xsl:template>

  <!-- Add the new connector after the last existing Connnector if there is one -->
  <xsl:template match="Connector[last()]" mode="insertConnector">
    <xsl:call-template name="Copy" />

    <xsl:call-template name="AddConnector" />
  </xsl:template>

  <!-- ... or before the first Engine if there is no existing Connector -->
  <xsl:template match="Engine[1][not(preceding-sibling::Connector)]"
                mode="insertConnector">
    <xsl:call-template name="AddConnector" />

    <xsl:call-template name="Copy" />
  </xsl:template>

  <xsl:template name="AddConnector">
    <!-- Add new line -->
    <xsl:text>&#xa;</xsl:text>
    <!-- This is the new connector -->
    <Connector port="8443" protocol="HTTP/1.1" SSLEnabled="true" 
               maxThreads="150" scheme="https" secure="true" 
               keystroreFile="${{user.home}}/.keystore" keystorePass="changeit"
               clientAuth="false" sslProtocol="TLS" />
  </xsl:template>
  
</xsl:stylesheet>
```

#### <a name="finalize-configuration"></a>Finaliser la configuration

Enfin, placez les fichiers du pilote JAR dans le classpath Tomcat puis redémarrez votre App Service.

1. Veillez à ce que les fichiers du pilote JDBC soient disponibles pour le chargeur de classes Tomcat en les mettant dans le répertoire */home/tomcat/lib*. (S’il n’existe pas déjà, créez ce répertoire.) Pour charger ces fichiers sur votre instance App Service, procédez comme suit :

    1. Dans le [Cloud Shell](https://shell.azure.com), installez l’extension d’application web :

      ```azurecli-interactive
      az extension add -–name webapp
      ```

    2. Exécutez la commande CLI suivante pour créer un tunnel SSH de votre système local vers App Service :

      ```azurecli-interactive
      az webapp remote-connection create --resource-group <resource-group-name> --name <app-name> --port <port-on-local-machine>
      ```

    3. Connectez-vous au port de tunneling local avec votre client SFTP et chargez les fichiers dans le dossier */home/tomcat/lib*.

    Vous pouvez également utiliser un client FTP pour charger le pilote JDBC. Suivez ces [instructions pour obtenir vos informations d’identification FTP](deploy-configure-credentials.md).

2. Si vous avez créé une source de données de niveau serveur, redémarrez l’application App Service Linux. Tomcat rétablit `CATALINA_BASE` sur `/home/tomcat` et utilise la configuration mise à jour.

### <a name="jboss-eap"></a>JBoss EAP

Il existe trois étapes de base lors de l’[inscription d’une source de données avec JBoss EAP](https://access.redhat.com/documentation/en-us/red_hat_jboss_enterprise_application_platform/7.0/html/configuration_guide/datasource_management) : chargement du pilote JDBC, ajout du pilote JDBC en tant que module et inscription du module. App Service est un service d’hébergement sans état. Par conséquent, les commandes de configuration pour l’ajout et l’inscription du module de source de données doivent être scriptées et appliquées au démarrage du conteneur.

1. Obtenez le pilote JDBC de votre base de données. 
2. Créez un fichier de définition de module XML pour le pilote JDBC. L’exemple ci-dessous est une définition de module pour PostgreSQL.

    ```xml
    <?xml version="1.0" ?>
    <module xmlns="urn:jboss:module:1.1" name="org.postgres">
        <resources>
        <!-- ***** IMPORTANT : REPLACE THIS PLACEHOLDER *******-->
        <resource-root path="/home/site/deployments/tools/postgresql-42.2.12.jar" />
        </resources>
        <dependencies>
            <module name="javax.api"/>
            <module name="javax.transaction.api"/>
        </dependencies>
    </module>
    ```

1. Placez vos commandes d’interface de ligne de commande JBoss dans un fichier nommé `jboss-cli-commands.cli`. Les commandes JBoss doivent ajouter le module et l’inscrire en tant que source de données. L’exemple ci-dessous montre les commandes d’interface de ligne de commande JBoss pour PostgreSQL.

    ```bash
    #!/usr/bin/env bash
    module add --name=org.postgres --resources=/home/site/deployments/tools/postgresql-42.2.12.jar --module-xml=/home/site/deployments/tools/postgres-module.xml

    /subsystem=datasources/jdbc-driver=postgres:add(driver-name="postgres",driver-module-name="org.postgres",driver-class-name=org.postgresql.Driver,driver-xa-datasource-class-name=org.postgresql.xa.PGXADataSource)

    data-source add --name=postgresDS --driver-name=postgres --jndi-name=java:jboss/datasources/postgresDS --connection-url=${POSTGRES_CONNECTION_URL,env.POSTGRES_CONNECTION_URL:jdbc:postgresql://db:5432/postgres} --user-name=${POSTGRES_SERVER_ADMIN_FULL_NAME,env.POSTGRES_SERVER_ADMIN_FULL_NAME:postgres} --password=${POSTGRES_SERVER_ADMIN_PASSWORD,env.POSTGRES_SERVER_ADMIN_PASSWORD:example} --use-ccm=true --max-pool-size=5 --blocking-timeout-wait-millis=5000 --enabled=true --driver-class=org.postgresql.Driver --exception-sorter-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLExceptionSorter --jta=true --use-java-context=true --valid-connection-checker-class-name=org.jboss.jca.adapters.jdbc.extensions.postgres.PostgreSQLValidConnectionChecker
    ```

1. Créez un script de démarrage, `startup_script.sh`, qui appelle les commandes d’interface de ligne de commande JBoss. L’exemple ci-dessous montre comment appeler `jboss-cli-commands.cli`. Plus tard, vous allez configurer App Service pour exécuter ce script au démarrage du conteneur. 

    ```bash
    $JBOSS_HOME/bin/jboss-cli.sh --connect --file=/home/site/deployments/tools/jboss-cli-commands.cli
    ```

1. À l’aide d’un client FTP de votre choix, chargez votre pilote JDBC, `jboss-cli-commands.cli`, `startup_script.sh` et la définition du module dans `/site/deployments/tools/`.
2. Configurez votre site pour qu’il exécute `startup_script.sh` au démarrage du conteneur. Dans le portail Azure, accédez à **Configuration** > **Paramètres généraux** > **Commande de démarrage**. Définissez le champ de commande de démarrage sur `/home/site/deployments/tools/startup_script.sh`. **Enregistrez** les changements apportés.

Pour confirmer que la source de source a été ajoutée au serveur JBoss, connectez-vous avec SSH dans votre application web et exécutez `$JBOSS_HOME/bin/jboss-cli.sh --connect`. Une fois que vous êtes connecté à JBoss, exécutez `/subsystem=datasources:read-resource` pour imprimer la liste des sources de données.

::: zone-end

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="choosing-a-java-runtime-version"></a>Choix d’une version du runtime Java

App Service permet aux utilisateurs de choisir la version majeure de la JVM, telle que Java 8 ou Java 11, ainsi que sa version mineure, telle que 1.8.0_232 ou 11.0.5. Vous pouvez également décider de mettre à jour automatiquement la version mineure quand de nouvelles versions mineures sont disponibles. Dans la plupart des cas, les sites de production doivent utiliser des versions mineures épinglées de la JVM. Cela empêchera des interruptions non anticipées lors d’une mise à jour automatique de la version mineure.

Si vous choisissez d’épingler la version mineure, vous devrez mettre à jour régulièrement la version mineure de la JVM sur le site. Pour vous assurer que votre application exécute la version mineure la plus récente, créez un emplacement de préproduction et incrémentez la version mineure sur le site intermédiaire. Une fois que vous avez confirmé que l’application s’exécute correctement sur la nouvelle version mineure, vous pouvez permuter les emplacements de préproduction et de production.

## <a name="jboss-eap-hardware-options"></a>Options matérielles de JBoss EAP

JBoss EAP est disponible uniquement sur les options matérielles Premium et Isolé. Les clients qui ont créé un site JBoss EAP sur un niveau Gratuit, Partagé, De base ou Standard pendant la préversion publique doivent évoluer vers le niveau matériel Premium ou Isolé pour éviter un comportement inattendu.

## <a name="java-runtime-statement-of-support"></a>Informations de prise en charge du runtime Java

### <a name="jdk-versions-and-maintenance"></a>Versions JDK et maintenance

Le kit de développement Java (JDK) pris en charge d’Azure est [Zulu](https://www.azul.com/downloads/azure-only/zulu/) fourni par [Azul Systems](https://www.azul.com/). Les builds Azul Zulu Enterprise d’OpenJDK sont une distribution gratuite, multiplateforme et prête pour la production d’OpenJDK pour Azure et Azure Stack pris en charge par Microsoft et Azul Systems. Elles contiennent tous les composants nécessaires pour générer et exécuter des applications Java SE. Vous pouvez installer le JDK à partir de l’[installation du JDK Java](/azure/developer/java/fundamentals/java-jdk-long-term-support).

Les mises à jour de la version majeure sont fournies via de nouvelles options de runtime dans Azure App Service. Les clients effectuent une mise à jour avec ces versions plus récentes de Java en configurant leur déploiement App Service et doivent s’occuper des tests et de s’assurer que la mise à jour majeure répond à leurs besoins.

Les kits JDK pris en charge sont automatiquement mis à jour tous les trimestres, en janvier, avril, juillet et octobre de chaque année. Pour plus d’informations sur Java sur Azure, consultez [ce document de support](/azure/developer/java/fundamentals/java-jdk-long-term-support).

### <a name="security-updates"></a>Mises à jour de sécurité

Les correctifs des principales vulnérabilités de sécurité seront publiés dès qu’Azul Systems les mettra à disposition. Une vulnérabilité « majeure » est définie par un score de base de 9.0 ou supérieur dans le système [NIST Common Vulnerability Scoring System, version 2](https://nvd.nist.gov/vuln-metrics/cvss).

Tomcat 8.0 a atteint sa [fin de vie (EOL) le 30 septembre 2018](https://tomcat.apache.org/tomcat-80-eol.html). Bien que le runtime soit toujours disponible sur Azure App Service, Azure n’appliquera pas de correctifs de sécurité à Tomcat 8.0. Si possible, migrez vos applications vers Tomcat 8.5 ou 9.0. Tomcat 8.5 et 9.0 sont tous deux disponibles sur Azure App Service. Pour plus d’informations, rendez-vous sur le [site officiel de Tomcat](https://tomcat.apache.org/whichversion.html). 

### <a name="deprecation-and-retirement"></a>Dépréciation et retrait

Si un runtime Java pris en charge est retiré, les développeurs Azure utilisant le runtime affecté reçoivent un avis de dépréciation au moins six mois avant son retrait.


### <a name="local-development"></a>Développement local

Les développeurs peuvent télécharger l’édition Production du kit Azul Zulu Enterprise JDK à partir du [site de téléchargement d’Azul](https://www.azul.com/downloads/azure-only/zulu/) pour développer en local.

### <a name="development-support"></a>Prise en charge du développement

La prise en charge produit du [kit JDK Zulu d’Azul pris en charge par Azure](https://www.azul.com/downloads/azure-only/zulu/) est disponible via Microsoft si vous développez pour Azure ou [Azure Stack](https://azure.microsoft.com/overview/azure-stack/) avec un [plan de support Azure éligible](https://azure.microsoft.com/support/plans/).

## <a name="next-steps"></a>Étapes suivantes

Visitez le centre [Azure pour les développeurs Java](/java/azure/) pour trouver des guides de démarrage rapide Azure, des tutoriels et la documentation de référence Java.

Des questions générales sur l’utilisation d’App Service pour Linux qui ne sont pas spécifiques au développement Java sont traitées dans [Questions fréquentes (FAQ) sur App Service Linux](faq-app-service-linux.md).

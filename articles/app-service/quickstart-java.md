---
title: 'Démarrage rapide : Créer une application Java dans Azure App Service'
description: Déployez votre premier programme Hello World Java dans Azure App Service en quelques minutes. Le plug-in Azure Web App pour Maven facilite le déploiement d’applications Java.
keywords: azure, app service, application web, windows, linux, java, maven, démarrage rapide
author: jasonfreeberg
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.devlang: Java
ms.topic: quickstart
ms.date: 08/01/2020
ms.author: jafreebe
ms.custom: mvc, seo-java-july2019, seo-java-august2019, seo-java-september2019
zone_pivot_groups: app-service-platform-windows-linux
adobe-target: true
ms.openlocfilehash: b54df350e3f59720a92d7c1e74e28d9fcafddf94
ms.sourcegitcommit: 7e97ae405c1c6c8ac63850e1b88cf9c9c82372da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97802471"
---
# <a name="quickstart-create-a-java-app-on-azure-app-service"></a>Démarrage rapide : Créer une application Java dans Azure App Service

[Azure App Service](overview.md) offre un service d’hébergement web hautement évolutif appliquant des mises à jour correctives automatiques.  Ce guide de démarrage rapide montre comment utiliser [Azure CLI](/cli/azure/get-started-with-azure-cli) avec le [plug-in Azure Web App pour Maven](https://github.com/Microsoft/azure-maven-plugins/tree/develop/azure-webapp-maven-plugin) afin de déployer un fichier .jar ou .war. Utilisez les onglets pour basculer entre les instructions Java SE et Tomcat.


> [!NOTE]
> Vous pouvez également le faire à l'aide d'IDE couramment utilisés comme IntelliJ et Eclipse. Consultez nos documents similaires dans le [Guide de démarrage d’Azure Toolkit for IntelliJ](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app) ou le [Guide de démarrage rapide d’Azure Toolkit for Eclipse](/azure/developer/java/toolkit-for-eclipse/create-hello-world-web-app).


![Exemple d’application en cours d’exécution dans Azure App Service](./media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-java-app"></a>Créer une application Java

# <a name="java-se"></a>[Java SE](#tab/javase)

Clonez l’exemple de projet [Spring Boot Getting Started](https://github.com/spring-guides/gs-spring-boot).

```bash
git clone https://github.com/spring-guides/gs-spring-boot
```

Accédez au répertoire du projet terminé.

```bash
cd gs-spring-boot/complete
```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

Exécutez la commande Maven suivante dans l’invite Cloud Shell pour créer une application nommée `helloworld` :

```bash
mvn archetype:generate "-DgroupId=example.demo" "-DartifactId=helloworld" "-DarchetypeArtifactId=maven-archetype-webapp" "-Dversion=1.0-SNAPSHOT"
```

Ensuite, changez votre répertoire de travail vers le dossier du projet :

```bash
cd helloworld
```

---

## <a name="configure-the-maven-plugin"></a>Configurer le plug-in Maven

Le processus de déploiement sur Azure App Service utilise vos informations d’identification Azure dans Azure CLI automatiquement. Si Azure CLI n’est pas installé localement, le plug-in Maven effectue l’authentification avec OAuth ou la connexion de l’appareil. Pour plus d’informations, consultez [Authentification avec les plug-ins Maven](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

Exécutez la commande Maven ci-dessous pour configurer le déploiement. Cette commande vous permet de configurer le système d’exploitation App Service, la version de Java et la version de Tomcat.

```bash
mvn com.microsoft.azure:azure-webapp-maven-plugin:1.12.0:config
```

::: zone pivot="platform-windows"

# <a name="java-se"></a>[Java SE](#tab/javase)

1. Quand vous êtes invité à choisir l’option **Subscription**, sélectionnez l’abonnement (`Subscription`) approprié en entrant le numéro indiqué au début de la ligne.
1. Quand vous êtes invité à choisir l’option **Application web**, acceptez l’option par défaut `<create>` en appuyant sur Entrée ou sélectionnez une application existante.
1. Quand vous êtes invité à choisir l’option **Système d’exploitation**, sélectionnez **Windows** en entrant `3`.
1. Quand vous êtes invité à choisir l’option **Niveau tarifaire**, sélectionnez **B2** en entrant `2`.
1. Utilisez la version de Java par défaut, **Java 8**, en appuyant sur Entrée.
1. Enfin, appuyez sur Entrée dans la dernière invite pour confirmer vos sélections.

    Le résumé généré ressemble à l’extrait de code ci-dessous.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007390755
    ResourceGroup : spring-boot-1599007390755-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : java 8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 41.118 s
    [INFO] Finished at: 2020-09-01T17:43:45-07:00
    [INFO] ------------------------------------------------------------------------
    ```

# <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. Quand vous êtes invité à choisir l’option **Subscription**, sélectionnez l’abonnement (`Subscription`) approprié en entrant le numéro indiqué au début de la ligne.
1. Quand vous êtes invité à choisir l’option **Application web**, acceptez l’option par défaut `<create>` en appuyant sur Entrée ou sélectionnez une application existante.
1. Quand vous êtes invité à choisir l’option **Système d’exploitation**, sélectionnez **Windows** en entrant `3`.
1. Quand vous êtes invité à choisir l’option **Niveau tarifaire**, sélectionnez **B2** en entrant `2`.
1. Utilisez la version de Java par défaut, **Java 8**, en appuyant sur Entrée.
1. Utilisez le conteneur web par défaut, **Tomcat 8.5**, en appuyant sur Entrée.
1. Enfin, appuyez sur Entrée dans la dernière invite pour confirmer vos sélections.

    Le résumé généré ressemble à l’extrait de code ci-dessous.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003152123
    ResourceGroup : helloworld-1599003152123-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Windows
    Java : 1.8
    WebContainer : tomcat 8.5
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 03:03 min
    [INFO] Finished at: 2020-09-01T16:35:30-07:00
    [INFO] ------------------------------------------------------------------------
    ```

---

::: zone-end
::: zone pivot="platform-linux"

### <a name="java-se"></a>[Java SE](#tab/javase)

1. Quand vous êtes invité à choisir l’option **Subscription**, sélectionnez l’abonnement (`Subscription`) approprié en entrant le numéro indiqué au début de la ligne.
1. Quand vous êtes invité à choisir l’option **Application web**, acceptez l’option par défaut `<create>` en appuyant sur Entrée ou sélectionnez une application existante.
1. Quand vous êtes invité à choisir l’option **Système d’exploitation**, sélectionnez **Linux** en appuyant sur Entrée.
1. Quand vous êtes invité à choisir l’option **Niveau tarifaire**, sélectionnez **B2** en entrant `2`.
1. Utilisez la version de Java par défaut, **Java 8**, en appuyant sur Entrée.
1. Enfin, appuyez sur Entrée dans la dernière invite pour confirmer vos sélections.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : spring-boot-1599007116351
    ResourceGroup : spring-boot-1599007116351-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : JAVA 8-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 20.925 s
    [INFO] Finished at: 2020-09-01T17:38:51-07:00
    [INFO] ------------------------------------------------------------------------
    ```

### <a name="tomcat"></a>[Tomcat](#tab/tomcat)

1. Quand vous êtes invité à choisir l’option **Subscription**, sélectionnez l’abonnement (`Subscription`) approprié en entrant le numéro indiqué au début de la ligne.
1. Quand vous êtes invité à choisir l’option **Application web**, acceptez l’option par défaut `<create>` en appuyant sur Entrée ou sélectionnez une application existante.
1. Quand vous êtes invité à choisir l’option **Système d’exploitation**, sélectionnez **Linux** en appuyant sur Entrée.
1. Quand vous êtes invité à choisir l’option **Niveau tarifaire**, sélectionnez **B2** en entrant `2`.
1. Utilisez la version de Java par défaut, **Java 8**, en appuyant sur Entrée.
1. Utilisez le conteneur web par défaut, **Tomcat 8.5**, en appuyant sur Entrée.
1. Enfin, appuyez sur Entrée dans la dernière invite pour confirmer vos sélections.

    ```
    Please confirm webapp properties
    Subscription Id : ********-****-****-****-************
    AppName : helloworld-1599003744223
    ResourceGroup : helloworld-1599003744223-rg
    Region : westeurope
    PricingTier : Basic_B2
    OS : Linux
    RuntimeStack : TOMCAT 8.5-jre8
    Deploy to slot : false
    Confirm (Y/N)? : Y
    [INFO] Saving configuration to pom.
    [INFO] ------------------------------------------------------------------------
    [INFO] BUILD SUCCESS
    [INFO] ------------------------------------------------------------------------
    [INFO] Total time: 50.785 s
    [INFO] Finished at: 2020-09-01T16:43:09-07:00
    [INFO] ------------------------------------------------------------------------
    ```

---

::: zone-end

Vous pouvez modifier les configurations d’App Service directement dans votre `pom.xml` si nécessaire. Voici les plus courantes :

Propriété | Obligatoire | Description | Version
---|---|---|---
`<schemaVersion>` | false | Spécifiez la version du schéma de configuration. Les valeurs prises en charge sont les suivantes : `v1`, `v2`. | 1.5.2
`<subscriptionId>` | false | Spécifiez l’ID d’abonnement. | 0.1.0+
`<resourceGroup>` | true | Groupe de ressources Azure pour votre application web. | 0.1.0+
`<appName>` | true | Le nom de votre application web. | 0.1.0+
`<region>` | true | Spécifie la région où votre application web sera hébergée ; la valeur par défaut est **westeurope**. Toutes les régions valides dans la section [Régions prises en charge](https://github.com/microsoft/azure-maven-plugins/blob/develop/azure-webapp-maven-plugin/README.md). | 0.1.0+
`<pricingTier>` | false | Le niveau tarifaire pour votre application web. La valeur par défaut est **P1V2** pour une charge de travail de production, tandis que **B2** est la valeur minimale recommandée pour le développement/test Java. [En savoir plus](https://azure.microsoft.com/pricing/details/app-service/linux/)| 0.1.0+
`<runtime>` | true | La configuration de l’environnement de runtime, dont vous pouvez afficher les détails [ici](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | 0.1.0+
`<deployment>` | true | La configuration de déploiement, dont vous pouvez afficher les détails [ici](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Web-App:-Configuration-Details). | 0.1.0+

Soyez attentif aux valeurs `<appName>` et `<resourceGroup>` (`helloworld-1590394316693` et `helloworld-1590394316693-rg` dans la démonstration). Elles seront utilisées ultérieurement.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=quickstart-java&step=config)

## <a name="deploy-the-app"></a>Déployer l’application

Le plug-in Maven utilise les informations d’identification du compte provenant d’Azure CLI pour effectuer le déploiement sur App Services. [Connectez-vous à Azure CLI](/cli/azure/authenticate-azure-cli) avant de continuer.

```azurecli
az login
```

Ensuite, vous pouvez déployer votre application Java sur Azure à l’aide de la commande suivante.

```bash
mvn package azure-webapp:deploy
```

Une fois le déploiement terminé, votre application est prête à l’adresse `http://<appName>.azurewebsites.net/` (`http://helloworld-1590394316693.azurewebsites.net` dans la démonstration). Ouvrez l’URL avec votre navigateur web local. Vous devez voir l’écran suivant :

![Exemple d’application en cours d’exécution dans Azure App Service](./media/quickstart-java/java-hello-world-in-browser-azure-app-service.png)

**Félicitations !** Vous avez déployé votre première application Java sur App Service.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=app-service-linux-quickstart&step=deploy)

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure au sein d’un groupe de ressources. Si vous ne pensez pas avoir besoin de ces ressources plus tard, supprimez le groupe de ressources du portail en exécutant la commande suivante dans Cloud Shell :

```azurecli-interactive
az group delete --name <your resource group name; for example: helloworld-1558400876966-rg> --yes
```

L’exécution de cette commande peut prendre une minute.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Se connecter à Azure DB pour PostgreSQL à l’aide de Java](../postgresql/connect-java.md)

> [!div class="nextstepaction"]
> [Configurer CI/CD](deploy-continuous-deployment.md)

> [!div class="nextstepaction"]
> [Informations sur la tarification](https://azure.microsoft.com/pricing/details/app-service/linux/)

> [!div class="nextstepaction"]
> [Agréger les journaux et les métriques](troubleshoot-diagnostic-logs.md)

> [!div class="nextstepaction"]
> [Monter en puissance](manage-scale-up.md)

> [!div class="nextstepaction"]
> [Ressources Azure pour les développeurs Java](/java/azure/)

> [!div class="nextstepaction"]
> [Configurer votre application Java](configure-language-java.md)

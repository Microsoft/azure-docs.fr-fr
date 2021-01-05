---
title: Configurer des applications sur le portail
description: Apprenez à configurer les paramètres communs d’une application App Service dans le portail Azure. Paramètres d’application, configuration d’application, chaînes de connexion, plateforme, pile de langage, conteneur, etc.
keywords: azure app service, web app, paramètres d’app, variables d’environnement
ms.assetid: 9af8a367-7d39-4399-9941-b80cbc5f39a0
ms.topic: article
ms.date: 12/07/2020
ms.custom: devx-track-csharp, seodec18
ms.openlocfilehash: ec9e3b6ca4b07003852681523a21b87ab7b8671b
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825957"
---
# <a name="configure-an-app-service-app-in-the-azure-portal"></a>Configurer une app App Service dans le portail Azure

Cet article explique comment configurer les paramètres courants d’une application web, d’un back end mobile ou d’une application API à l’aide du [portail Azure].

## <a name="configure-app-settings"></a>Configuration des paramètres d’application

Dans App Service, les paramètres d’application sont des variables transmises comme des variables d’environnement au code de l’application. Pour les applications Linux et les conteneurs personnalisés, App Service envoie les paramètres d’application au conteneur via l’indicateur `--env` pour définir la variable d'environnement dans le conteneur.

Dans le [portail Azure], recherchez et sélectionnez **App Services**, puis sélectionnez votre application. 

![Rechercher App Services](./media/configure-common/search-for-app-services.png)

Dans le menu de gauche de l'application, sélectionnez **Configuration** > **Paramètres de l'application**.

![Paramètres de l’application](./media/configure-common/open-ui.png)

Les développeurs ASP.NET et ASP.NET Core définissent les paramètres de l'application dans App Service comme ils le font avec `<appSettings>` dans *Web.config* ou *appsettings.json*, mais les valeurs dans App Service remplacent celles du fichier *Web.config* ou *appsettings.json*. Vous pouvez conserver de façon sécurisée les paramètres de développement (par exemple, le mot de passe MySQL local) dans *Web.config* ou *appsettings.json*, ainsi que les secrets de production (par exemple le mot de passe de la base de données MySQL Azure) dans App Service. Le même code utilise vos paramètres de développement lorsque vous déboguez localement, et utilise vos secrets de production lorsque vous les déployez sur Azure.

De la même façon, d'autres piles de langage récupèrent les paramètres de l'application en tant que variables d'environnement lors de l'exécution. Pour les étapes spécifiques à la pile de langage, voir :

- [ASP.NET Core](configure-language-dotnetcore.md#access-environment-variables)
- [Node.JS](configure-language-nodejs.md#access-environment-variables)
- [PHP](configure-language-php.md#access-environment-variables)
- [Python](configure-language-python.md#access-environment-variables)
- [Java](configure-language-java.md#configure-data-sources)
- [Ruby](configure-language-ruby.md#access-environment-variables)
- [Conteneurs personnalisés](configure-custom-container.md#configure-environment-variables)

Une fois stockés, les paramètres d’application sont toujours chiffrés (chiffrement au repos).

> [!NOTE]
> Les paramètres d’application peuvent également être résolus à partir de [Key Vault](../key-vault/index.yml) à l’aide des [références Key Vault](app-service-key-vault-references.md).

### <a name="show-hidden-values"></a>Afficher les valeurs masquées

Par défaut, les valeurs des paramètres de l'application sont masquées dans le portail pour des raisons de sécurité. Pour afficher une valeur masquée d'un paramètre d'application, cliquez sur le champ **Valeur** de ce paramètre. Pour afficher les valeurs de tous les paramètres de l'application, cliquez sur le bouton **Afficher la valeur**.

### <a name="add-or-edit"></a>Ajouter ou modifier

Pour ajouter un nouveau paramètre d'application, cliquez sur **Nouveau paramètre d'application**. Dans la boîte de dialogue, vous pouvez [conserver le paramètre dans l'emplacement actuel](deploy-staging-slots.md#which-settings-are-swapped).

Pour modifier un paramètre, cliquez sur le bouton **Modifier** sur le côté droit.

Lorsque vous avez terminé, cliquez sur **Mettre à jour**. N'oubliez pas de cliquer sur **Enregistrer** dans la page **Configuration**.

> [!NOTE]
> Dans un conteneur Linux par défaut ou un conteneur Linux personnalisé, une structure de clé JSON imbriquée dans le nom du paramètre de l'application, par exemple `ApplicationInsights:InstrumentationKey`, doit être configurée dans App Service en tant que `ApplicationInsights__InstrumentationKey` pour le nom de la clé. En d’autres termes, vous devez remplacer `:` par `__` (trait de soulignement double).
>

### <a name="edit-in-bulk"></a>Modification par lots

Pour ajouter ou modifier par lots des paramètres d'application, cliquez sur le bouton **Modification avancée**. Lorsque vous avez terminé, cliquez sur **Mettre à jour**. N'oubliez pas de cliquer sur **Enregistrer** dans la page **Configuration**.

Les paramètres de l'application affichent le format JSON suivant :

```json
[
  {
    "name": "<key-1>",
    "value": "<value-1>",
    "slotSetting": false
  },
  {
    "name": "<key-2>",
    "value": "<value-2>",
    "slotSetting": false
  },
  ...
]
```

### <a name="automate-app-settings-with-the-azure-cli"></a>Automatiser les paramètres d’application avec Azure CLI

Vous pouvez utiliser Azure CLI pour créer et gérer des paramètres à partir de la ligne de commande.

- Affectez une valeur à un paramètre avec [az webapp config app Settings set](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_set) :

    ```azurecli-interactive
    az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings <setting-name>="<value>"
    ```
        
    Remplacez `<setting-name>` par le nom du paramètre, et `<value>` par la valeur à assigner à celui-ci. Cette commande crée le paramètre s’il n’existe pas déjà.
    
- Affichez tous les paramètres et leurs valeurs avec [az webapp config appsettings list](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_list) :
    
    ```azurecli-interactive
    az webapp config appsettings list --name <app-name> --resource-group <resource-group-name>
    ```
    
- Supprimez un ou plusieurs paramètres avec [az webapp config app settings delete](/cli/azure/webapp/config/appsettings#az_webapp_config_appsettings_delete) :

    ```azurecli-interactive
    az webapp config appsettings delete --name <app-name> --resource-group <resource-group-name> --setting-names {<names>}
    ```
    
    Remplacez `<names>` par une liste de noms de paramètres séparés par des espaces.

## <a name="configure-connection-strings"></a>Configuration des chaînes de connexion

Dans le [portail Azure], recherchez et sélectionnez **App Services**, puis sélectionnez votre application. Dans le menu de gauche de l'application, sélectionnez **Configuration** > **Paramètres de l'application**.

![Paramètres de l’application](./media/configure-common/open-ui.png)

Les développeurs ASP.NET et ASP.NET Core définissent les chaînes de connexion dans App Service comme ils le font avec `<connectionStrings>` dans *Web.config*, mais les valeurs que vous définissez dans App Service remplacent celles du fichier *Web.config*. Vous pouvez conserver en lieu sûr les paramètres de développement (par exemple un fichier de base de données) dans *Web.config* et les secrets de production (par exemple des informations d’identification SQL Database) dans App Service. Le même code utilise vos paramètres de développement lorsque vous déboguez localement, et utilise vos secrets de production lorsque vous les déployez sur Azure.

Pour d’autres piles de langage, il est préférable d'utiliser des [paramètres d’application](#configure-app-settings) car les chaînes de connexion nécessitent un formatage spécial dans les clés variables pour pouvoir accéder aux valeurs. 

> [!NOTE]
> Il existe un cas où vous pouvez utiliser des chaînes de connexion au lieu des paramètres d’application pour les langages autre que .NET : certains types de base de données Azure sont sauvegardés avec l’application _uniquement_ si vous configurez une chaîne de connexion pour la base de données dans votre application App Service. Pour plus d’informations, consultez [Éléments sauvegardés](manage-backup.md#what-gets-backed-up). Si vous n'avez pas besoin de cette sauvegarde automatisée, utilisez les paramètres de l'application.

Lors de l'exécution, les chaînes de connexion sont disponibles en tant que variables d'environnement, préfixées par les types de connexion suivants :

* SQLServer : `SQLCONNSTR_`  
* MySQL : `MYSQLCONNSTR_` 
* SQLAzure : `SQLAZURECONNSTR_` 
* Personnalisé : `CUSTOMCONNSTR_`
* PostgreSQL : `POSTGRESQLCONNSTR_`  

Par exemple, une chaîne de connexion MySql nommée *connectionstring1* est accessible comme variable d’environnement `MYSQLCONNSTR_connectionString1`. Pour les étapes spécifiques à la pile de langage, voir :

- [ASP.NET Core](configure-language-dotnetcore.md#access-environment-variables)
- [Node.JS](configure-language-nodejs.md#access-environment-variables)
- [PHP](configure-language-php.md#access-environment-variables)
- [Python](configure-language-python.md#access-environment-variables)
- [Java](configure-language-java.md#configure-data-sources)
- [Ruby](configure-language-ruby.md#access-environment-variables)
- [Conteneurs personnalisés](configure-custom-container.md#configure-environment-variables)

Une fois stockées, les chaînes de connexion sont toujours chiffrées (chiffrement au repos).

> [!NOTE]
> Les chaînes de connexion peuvent également être résolues à partir de [Key Vault](../key-vault/index.yml) à l’aide des [références Key Vault](app-service-key-vault-references.md).

### <a name="show-hidden-values"></a>Afficher les valeurs masquées

Par défaut, les valeurs des chaînes de connexion sont masquées dans le portail pour des raisons de sécurité. Pour afficher une valeur masquée d'une chaîne de connexion, cliquez simplement sur le champ **Valeur** de cette chaîne. Pour afficher les valeurs de toutes les chaînes de connexion, cliquez sur le bouton **Afficher la valeur**.

### <a name="add-or-edit"></a>Ajouter ou modifier

Pour ajouter une nouvelle chaîne de connexion, cliquez sur **Nouvelle chaîne de connexion**. Dans la boîte de dialogue, vous pouvez [conserver la chaîne de connexion dans l'emplacement actuel](deploy-staging-slots.md#which-settings-are-swapped).

Pour modifier un paramètre, cliquez sur le bouton **Modifier** sur le côté droit.

Lorsque vous avez terminé, cliquez sur **Mettre à jour**. N'oubliez pas de cliquer sur **Enregistrer** dans la page **Configuration**.

### <a name="edit-in-bulk"></a>Modification par lots

Pour ajouter ou modifier par lots des chaînes de connexion, cliquez sur le bouton **Modification avancée**. Lorsque vous avez terminé, cliquez sur **Mettre à jour**. N'oubliez pas de cliquer sur **Enregistrer** dans la page **Configuration**.

Les chaînes de connexion affichent le format JSON suivant :

```json
[
  {
    "name": "name-1",
    "value": "conn-string-1",
    "type": "SQLServer",
    "slotSetting": false
  },
  {
    "name": "name-2",
    "value": "conn-string-2",
    "type": "PostgreSQL",
    "slotSetting": false
  },
  ...
]
```

<a name="platform"></a>
<a name="alwayson"></a>

## <a name="configure-general-settings"></a>Configurer les paramètres généraux

Dans le [portail Azure], recherchez et sélectionnez **App Services**, puis sélectionnez votre application. Dans le menu de gauche de l'application, sélectionnez **Configuration** > **Paramètres généraux**.

![Paramètres généraux :](./media/configure-common/open-general.png)

Ici, vous pouvez configurer quelques paramètres communs pour l'application. Certains paramètres vous obligent à [monter en puissance vers des niveaux de tarification supérieurs](manage-scale-up.md).

- **Paramètres de la pile** : La pile logicielle pour exécuter l'application, y compris le langage et les versions SDK.

    Pour les applications Linux et les applications de conteneur personnalisées, vous pouvez sélectionner la version du runtime de langage et définir une **commande de démarrage** facultative ou un fichier de commandes de démarrage.

    ![Paramètres généraux pour les conteneurs Linux](./media/configure-common/open-general-linux.png)

- **Paramètres de la plateforme** : Vous permet de configurer les paramètres de la plateforme d'hébergement, notamment :
    - **Nombre de bits** : 32 bits ou 64 bits.
    - **Protocole WebSocket** : Pour [ASP.NET SignalR] ou [socket.io](https://socket.io/), par exemple.
    - **Toujours actif** : Garde l'application chargée même s’il n'y a aucun trafic. Vous devez l'activer pour les WebJobs continus ou pour les WebJobs déclenchés à l’aide d’une expression CRON.
      > [!NOTE]
      > Avec la fonctionnalité Always On, l’équilibreur de charge frontal envoie une requête à la racine de l’application. Impossible de configurer ce point de terminaison d’application du App Service.
    - **Version de pipeline gérée** : Le [mode pipeline] IIS. Définissez ce paramètre sur **Classique** si vous utilisez une application héritée nécessitant une version plus ancienne d'IIS.
    - **Version HTTP** : Utilisez la version **2.0** pour activer la prise en charge du protocole [HTTPS/2](https://wikipedia.org/wiki/HTTP/2).
    > [!NOTE]
    > Les navigateurs les plus récents prennent en charge le protocole HTTP/2 sur TLS uniquement, alors que le trafic non chiffré continue d’utiliser HTTP/1.1. Pour vous assurer que les navigateurs clients se connectent à votre application avec le protocole HTTP/2, sécurisez votre nom DNS personnalisé. Pour plus d’informations, consultez [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md).
    - **Affinité ARR** : Dans un déploiement multi-instance, assurez-vous que le client est routé vers la même instance pendant toute la session. Vous pouvez définir cette option sur **Off** (Désactivée) pour les applications sans état.
- **Débogage** : Activez le débogage à distance pour les applications [ASP.NET](troubleshoot-dotnet-visual-studio.md#remotedebug), [ASP.NET Core](/visualstudio/debugger/remote-debugging-azure) ou [Node.js](configure-language-nodejs.md#debug-remotely). Cette option se désactive automatiquement au bout de 48 heures.
- **Certificats clients entrants** : demander des certificats clients dans une [authentification mutuelle](app-service-web-configure-tls-mutual-auth.md).

## <a name="configure-default-documents"></a>Configurer les documents par défaut

Ce paramètre ne concerne pour les applications Windows.

Dans le [portail Azure], recherchez et sélectionnez **App Services**, puis sélectionnez votre application. Dans le menu de gauche de l'application, sélectionnez **Configuration** > **Documents par défaut**.

![Documents par défaut](./media/configure-common/open-documents.png)

Le document par défaut est la page web qui s’affiche à l’URL racine pour un site web. Le premier fichier correspondant dans la liste est utilisé. Pour ajouter un nouveau document par défaut, cliquez sur **Nouveau document**. N’oubliez pas de cliquer sur **Enregistrer**.

Si l’application utilise des modules qui effectuent un routage en fonction de l’URL, au lieu de proposer du contenu statique, il n’existe aucun document par défaut.

## <a name="configure-path-mappings"></a>Configurez les mappages de chemin d'accès

Dans le [portail Azure], recherchez et sélectionnez **App Services**, puis sélectionnez votre application. Dans le menu de gauche, sélectionnez **Configuration** > **Mappages de chemin d’accès**.

![Mappages de chemin d’accès](./media/configure-common/open-path.png)

> [!NOTE] 
> L’onglet **Mappages de chemin d’accès** peut afficher des paramètres spécifiques au système d’exploitation qui diffèrent de l’exemple présenté ici.

### <a name="windows-apps-uncontainerized"></a>Applications Windows (non conteneurisées)

Pour les applications Windows, vous pouvez personnaliser les mappages de gestionnaire IIS ainsi que les applications et répertoires virtuels.

Les mappages de gestionnaire vous permettent d’ajouter des processeurs de script personnalisés pour gérer les requêtes portant sur des extensions de fichiers spécifiques. Pour ajouter un gestionnaire personnalisé, cliquez sur **Nouveau mappage de gestionnaire**. Configurez le gestionnaire comme suit :

- **Extension**. Extension de fichier à gérer, par exemple *\*.php* ou *handler.fcgi*.
- **Processeur de script**. Chemin d'accès absolu du processeur de script vers vous. Les requêtes de fichiers qui correspondent à l'extension de fichier sont traitées par le processeur de script. Utilisez le chemin `D:\home\site\wwwroot` pour faire référence au répertoire racine de votre application Web.
- **Arguments**. Arguments de ligne de commande facultatifs pour le processeur de script.

Chaque application comporte le chemin racine par défaut (`/`) mappé à `D:\home\site\wwwroot`, où votre code est déployé par défaut. Si votre racine d'application figure dans un autre dossier, ou si votre référentiel contient plusieurs applications, vous pouvez modifier ou ajouter ici des applications et des répertoires virtuels. 

À partir de l’onglet **Mappages de chemin d’accès**, cliquez sur **Nouvelle application virtuelle ou nouveau répertoire virtuel**. 

- Pour mapper un répertoire virtuel à un chemin d’accès physique, laissez la case à cocher **Répertoire** activée. Indiquez le répertoire virtuel et le chemin d’accès relatif (physique) correspondant à la racine du site web (`D:\home`).
- Pour marquer un répertoire virtuel en tant qu’application web, désactivez la case à cocher **Répertoire**.
  
  ![Case à cocher Répertoire](./media/configure-common/directory-check-box.png)

### <a name="containerized-apps"></a>Applications conteneurisées

Vous pouvez [ajouter du stockage personnalisé pour votre application conteneurisée](configure-connect-to-azure-storage.md). Les applications conteneurisées comprennent toutes les applications Linux ainsi que les conteneurs personnalisés Windows et Linux fonctionnant sur App Service. Cliquez sur **Nouveau montage de stockage Azure** et configurez votre stockage personnalisé comme suit :

- **Name** : Le nom d’affichage.
- **Options de configuration** : **De base** ou **Avancé**.
- **Comptes de stockage** : Le compte de stockage avec le conteneur de votre choix.
- **Type de stockage** : **Objets blob Azure** ou **Fichiers Azure**.
  > [!NOTE]
  > Les applications de conteneur Windows ne prennent en charge que les fichiers Azure.
- **Conteneur de stockage** : Pour la configuration de base, le conteneur de votre choix.
- **Nom de partage** : Pour une configuration avancée, le nom du partage de fichiers.
- **Clé d’accès** : Pour une configuration avancée, la clé d'accès.
- **Chemin de montage**  : Le chemin absolu dans votre conteneur pour monter le stockage personnalisé.

Pour plus d’informations, consultez [Accéder à Stockage Azure en tant que partage réseau à partir d’un conteneur dans App Service](configure-connect-to-azure-storage.md).

## <a name="configure-language-stack-settings"></a>Configurer les paramètres de la pile de langage

Pour les applications Linux, voir :

- [ASP.NET Core](configure-language-dotnetcore.md)
- [Node.JS](configure-language-nodejs.md)
- [PHP](configure-language-php.md)
- [Python](configure-language-python.md)
- [Java](configure-language-java.md)
- [Ruby](configure-language-ruby.md)

## <a name="configure-custom-containers"></a>Configurer des conteneurs personnalisés

Voir [Exécuter un conteneur Linux personnalisé dans Azure App Service](configure-custom-container.md)

## <a name="next-steps"></a>Étapes suivantes

- [Configuration d’un nom de domaine personnalisé dans Azure App Service]
- [Configurer des environnements intermédiaires dans Azure App Service]
- [Sécuriser un nom DNS personnalisé avec une liaison TLS/SSL dans Azure App Service](configure-ssl-bindings.md)
- [Activer les journaux de diagnostic](troubleshoot-diagnostic-logs.md)
- [Mettre à l’échelle une application dans Azure App Service]
- [Concepts de base de la supervision dans Azure App Service]
- [Modifier les paramètres applicationHost.config avec applicationHost.xdt](https://github.com/projectkudu/kudu/wiki/Xdt-transform-samples)

<!-- URL List -->

[ASP.NET SignalR]: https://www.asp.net/signalr
[Portail Azure]: https://portal.azure.com/
[Configuration d’un nom de domaine personnalisé dans Azure App Service]: ./app-service-web-tutorial-custom-domain.md
[Configurer des environnements intermédiaires dans Azure App Service]: ./deploy-staging-slots.md
[How to: Monitor web endpoint status]: ./web-sites-monitor.md
[Concepts de base de la supervision dans Azure App Service]: ./web-sites-monitor.md
[mode pipeline]: https://www.iis.net/learn/get-started/introduction-to-iis/introduction-to-iis-architecture#Application
[Mettre à l’échelle une application dans Azure App Service]: ./manage-scale-up.md

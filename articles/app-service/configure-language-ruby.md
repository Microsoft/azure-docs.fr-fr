---
title: Configurer des applications Ruby - Azure App Service
description: Découvrez comment configurer un conteneur Ruby prédéfini pour votre application. Cet article présente les tâches de configuration les plus courantes.
ms.topic: quickstart
ms.date: 06/18/2020
ms.reviewer: astay; kraigb
ms.custom: mvc, seodec18, devx-track-azurecli
ms.openlocfilehash: 038d62573b491325adc60647debf17fa87e06cfe
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92743691"
---
# <a name="configure-a-linux-ruby-app-for-azure-app-service"></a>Configurer une application Ruby Linux pour Azure App Service

Cet article explique la façon dont [Azure App Service](overview.md) exécute des applications Ruby dans un conteneur Linux, et comment personnaliser le comportement d’App Service si nécessaire. Les applications Ruby doivent être déployées avec les [gemmes](https://rubygems.org/gems) nécessaires.

Ce guide fournit des concepts et des instructions clés aux développeurs Ruby qui utilisent un conteneur Linux intégré dans App Service. Si vous n’avez jamais utilisé Azure App Service, vous devez d’abord suivre le [guide de démarrage rapide Ruby](quickstart-ruby.md) et le [tutoriel sur l’utilisation de Ruby avec PostgreSQL](tutorial-ruby-postgres-app.md).

## <a name="show-ruby-version"></a>Afficher la version de Ruby

Pour afficher la version actuelle de Ruby, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp config show --resource-group <resource-group-name> --name <app-name> --query linuxFxVersion
```

Pour afficher toutes les versions prises en charge de Ruby, exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) :

```azurecli-interactive
az webapp list-runtimes --linux | grep RUBY
```

Vous pouvez exécuter une version non prise en charge de Ruby en créant votre propre image conteneur. Pour plus d’informations, consultez [Utiliser une image Docker personnalisée](tutorial-custom-container.md?pivots=container-linux).

## <a name="set-ruby-version"></a>Définir la version de Ruby

Exécutez la commande suivante dans [Cloud Shell](https://shell.azure.com) pour définir la version 2.3 de Ruby :

```azurecli-interactive
az webapp config set --resource-group <resource-group-name> --name <app-name> --linux-fx-version "RUBY|2.3"
```

> [!NOTE]
> Si vous voyez des erreurs semblables à celles-ci au moment du déploiement :
> ```
> Your Ruby version is 2.3.3, but your Gemfile specified 2.3.1
> ```
> or
> ```
> rbenv: version `2.3.1' is not installed
> ```
> Cela signifie que la version de Ruby configurée dans votre projet est différente de celle installée dans le conteneur que vous exécutez (`2.3.3`, dans l’exemple ci-dessus). Dans l’exemple ci-dessus, regardez *Gemfile* et *.ruby-version* pour voir si la version de Ruby n’est pas définie ou si elle est définie sur la version qui est installée dans le conteneur que vous exécutez (`2.3.3`, dans l’exemple ci-dessus).

## <a name="access-environment-variables"></a>Accéder aux variables d’environnement

Dans App Service, vous pouvez [définir les paramètres de l’application](configure-common.md#configure-app-settings) en dehors de votre code d’application. Vous pouvez ensuite y accéder à l’aide du modèle standard [ENV['\<path-name>']](https://ruby-doc.org/core-2.3.3/ENV.html). Par exemple, pour accéder à un paramètre d’application nommé `WEBSITE_SITE_NAME`, utilisez le code suivant :

```ruby
ENV['WEBSITE_SITE_NAME']
```

## <a name="customize-deployment"></a>Personnaliser le déploiement

Si vous déployez un [dépôt Git](deploy-local-git.md) ou un [package Zip](deploy-zip.md) lorsque les processus de génération sont activés, le moteur de déploiement (Kudu) exécute automatiquement les étapes de post-déploiement suivantes par défaut :

1. Vérifiez qu’un *Gemfile* existe.
1. Exécutez `bundle clean`. 
1. Exécutez `bundle install --path "vendor/bundle"`.
1. Exécutez `bundle package` pour empaqueter les gemmes dans le dossier du fournisseur ou du cache.

### <a name="use---without-flag"></a>Utiliser l’indicateur --without

Pour exécuter `bundle install` avec l’indicateur [--without](https://bundler.io/man/bundle-install.1.html), définissez le [paramètre d’application](configure-common.md#configure-app-settings) `BUNDLE_WITHOUT` sur une liste de groupes séparés par des virgules. Par exemple, la commande suivante le définit sur `development,test`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings BUNDLE_WITHOUT="development,test"
```

Si ce paramètre est défini, le moteur de déploiement exécute `bundle install` avec `--without $BUNDLE_WITHOUT`.

### <a name="precompile-assets"></a>Précompiler les ressources

Par défaut, les étapes de post-déploiement ne précompilent pas les ressources. Pour activer la précompilation des ressources, définissez le [paramètre d’application](configure-common.md#configure-app-settings) `ASSETS_PRECOMPILE` sur `true`. Ensuite, la commande `bundle exec rake --trace assets:precompile` est exécutée à la fin des étapes de post-déploiement. Par exemple :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings ASSETS_PRECOMPILE=true
```

Pour plus d’informations, consultez [Traiter les ressources statiques](#serve-static-assets).

## <a name="customize-start-up"></a>Personnaliser le démarrage

Par défaut, le conteneur Ruby démarre le serveur Rails en suivant la procédure ci-dessous (pour plus d’informations, consultez le [script de démarrage](https://github.com/Azure-App-Service/ruby/blob/master/2.3.8/startup.sh)) :

1. Il génère une valeur [secret_key_base](https://edgeguides.rubyonrails.org/security.html#environmental-security), si elle n’existe pas déjà. Cette valeur est nécessaire pour que l’application puisse s’exécuter en mode production.
1. Définissez la variable d’environnement `RAILS_ENV` sur `production`.
1. Supprimez le fichier *.pid* qui se trouve dans le répertoire *tmp/pids* depuis une précédente exécution d’un serveur Rails.
1. Vérifiez que toutes les dépendances sont installées. Si ce n’est pas le cas, essayez d’installer les gemmes à partir du *répertoire du fournisseur ou du cache* local.
1. Exécutez `rails server -e $RAILS_ENV`.

Vous pouvez personnaliser le processus de démarrage de différentes manières :

- [En traitant les ressources statiques](#serve-static-assets)
- [En exécutant en mode hors production](#run-in-non-production-mode)
- [En définissant manuellement secret_key_base](#set-secret_key_base-manually)

### <a name="serve-static-assets"></a>Traiter les ressources statiques

Le serveur Rails du conteneur Ruby s’exécute en mode production par défaut, et [part du principe que les ressources sont précompilées et traitées par votre serveur web](https://guides.rubyonrails.org/asset_pipeline.html#in-production). Pour traiter les ressources statiques à partir du serveur Rails, vous devez faire deux choses :

- **Précompiler les ressources** - [Précompilez les ressources statiques localement](https://guides.rubyonrails.org/asset_pipeline.html#local-precompilation) et déployez-les manuellement. Vous pouvez aussi laisser le moteur de déploiement se charger de la précompilation (consultez [Précompiler les ressources](#precompile-assets)).
- **Activer le traitement des fichiers statiques**  : pour servir des ressources statiques du conteneur Ruby, [définissez le paramètre d’application `RAILS_SERVE_STATIC_FILES`](configure-common.md#configure-app-settings) sur `true`. Par exemple :

    ```azurecli-interactive
    az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_SERVE_STATIC_FILES=true
    ```

### <a name="run-in-non-production-mode"></a>Exécuter en mode hors production

Par défaut, le serveur Rails s’exécute en mode production. Pour une exécution en mode développement, par exemple, définissez le [paramètre d’application](configure-common.md#configure-app-settings) `RAILS_ENV` sur `development`.

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings RAILS_ENV="development"
```

Toutefois, ce paramètre seul entraîne le démarrage du serveur Rails en mode développement, qui accepte uniquement les requêtes localhost et qui n’est pas accessible en dehors du conteneur. Pour accepter les requêtes des clients distants, définissez le [paramètre d’application](configure-common.md#configure-app-settings) `APP_COMMAND_LINE` sur `rails server -b 0.0.0.0`. Ce paramètre d’application vous permet d’exécuter une commande personnalisée dans le conteneur Ruby. Par exemple :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings APP_COMMAND_LINE="rails server -b 0.0.0.0"
```

### <a name="set-secret_key_base-manually"></a><a name="set-secret_key_base-manually"></a> Définir manuellement secret_key_base

Pour utiliser votre propre valeur `secret_key_base` au lieu de laisser App Service en générer une automatiquement, définissez le [paramètre d’application](configure-common.md#configure-app-settings) `SECRET_KEY_BASE` sur la valeur souhaitée. Par exemple :

```azurecli-interactive
az webapp config appsettings set --name <app-name> --resource-group <resource-group-name> --settings SECRET_KEY_BASE="<key-base-value>"
```

## <a name="access-diagnostic-logs"></a>Accéder aux journaux de diagnostic

[!INCLUDE [Access diagnostic logs](../../includes/app-service-web-logs-access-no-h.md)]

## <a name="open-ssh-session-in-browser"></a>Ouvrir une session SSH dans un navigateur

[!INCLUDE [Open SSH session in browser](../../includes/app-service-web-ssh-connect-builtin-no-h.md)]

[!INCLUDE [robots933456](../../includes/app-service-web-configure-robots933456.md)]

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Tutoriel : Application Rails avec PostgreSQL](tutorial-ruby-postgres-app.md)

> [!div class="nextstepaction"]
> [Questions fréquentes (FAQ) sur App Service sur Linux](faq-app-service-linux.md)

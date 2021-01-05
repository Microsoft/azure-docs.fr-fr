---
title: 'Démarrage rapide : Votre première requête Ruby'
description: Dans ce guide de démarrage rapide, vous suivez les étapes pour activer la gemme Resource Graph pour Ruby et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: 4ffd7f33fb83b7a1f247f687b77dd2ef9c8e5bc9
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057261"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-ruby"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de Ruby

La première étape pour utiliser Azure Resource Graph consiste à vérifier que les gemmes nécessaires pour Ruby sont installées. Ce guide de démarrage rapide décrit le processus d’ajout des gemmes à votre installation Ruby.

Au terme de ce processus, vous aurez ajouté les gemmes à votre installation Ruby et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.
- Un principal de service Azure, y compris les _clientId_ et _clientSecret_.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="create-the-resource-graph-project"></a>Créer le projet Resource Graph

Pour que Ruby puisse interroger Azure Resource Graph, la gemme doit être ajoutée au `Gemfile`. Cette gemme fonctionne partout où Ruby peut être utilisé, notamment avec [Azure Cloud Shell](https://shell.azure.com), [bash sur Windows 10](/windows/wsl/install-win10), ou par le biais d’une installation locale.

1. Vérifiez que la version la plus récente de Ruby est installée (**2.7.1** au minimum). Si ce n’est pas le cas, téléchargez-la à l’adresse [Ruby-Lang.org](https://www.ruby-lang.org/en/downloads/).

1. Dans l’environnement Ruby de votre choix, initialisez un bundle dans un nouveau dossier de projet :

   ```bash
   # Initialize a bundle to create a new Gemfile
   bundle init
   ```

1. Mettez à jour votre `Gemfile` avec les gemmes nécessaires à Azure Resource Graph. Le fichier mis à jour doit ressembler à l’exemple suivant :

   ```file
   # frozen_string_literal: true

   source "https://rubygems.org"

   git_source(:github) {|repo_name| "https://github.com/#{repo_name}" }

   # gem "rails"
   gem 'azure_mgmt_resourcegraph', '~> 0.17.2'
   ```

1. Dans le dossier du projet, exécutez `bundle install`. Vérifiez que les gemmes ont été installées avec `bundle list`.

1. Dans le même dossier de projet, créez `argQuery.rb` avec le code suivant et enregistrez le fichier mis à jour :

   ```ruby
   #!/usr/bin/env ruby

   require 'azure_mgmt_resourcegraph'
   ARG = Azure::ResourceGraph::Profiles::Latest::Mgmt

   # Get arguments and set options
   options = {
       tenant_id: ARGV[0],
       client_id: ARGV[1],
       client_secret: ARGV[2],
       subscription_id: ARGV[3]
   }

   # Create Resource Graph client from options
   argClient = ARG::Client.new(options)

   # Create Resource Graph QueryRequest for subscription with query
   query_request = ARGModels::QueryRequest.new.tap do |qr|
       qr.subscriptions = [ARGV[3]]
       qr.query = ARGV[4]
   end

   # Get the resources from Resource Graph
   response = argClient.resources(query_request)

   # Convert data to JSON and output
   puts response.data.to_json
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Le script Ruby étant enregistré et prêt à l’emploi, testons une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

Dans chaque appel de `argQuery`, vous devez remplacer les variables utilisées par vos propres valeurs :

- Remplacer `{tenantId}` par l’ID de votre locataire
- `{clientId}` - Remplacer l’ID client de votre principal de service
- `{clientSecret}` - Remplacer le secret client de votre principal de service
- Remplacer `{subscriptionId}` par votre ID d’abonnement

1. Définissez les répertoires sur le dossier de projet dans lequel vous avez créé les fichiers `Gemfile` et `argClient.rb`.

1. Exécutez votre première requête Azure Resource Graph à l’aide des gemmes et de la méthode `resources` :

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5"
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Remplacez le dernier paramètre par `argQuery.rb` et la requête par `order by` suivi de la propriété **Name** :

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | limit 5 | order by name asc"
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Remplacez le dernier paramètre par `argQuery.rb` et changez la requête en `order by` pour classer d’abord les résultats selon la propriété **Name**, puis limitez (`limit`) les résultats aux cinq premiers :

   ```bash
   ruby argQuery.rb "{tenantId}" "{clientId}" "{clientSecret}" "{subscriptionId}" "Resources | project name, type | order by name asc | limit 5"
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer les gemmes installées de votre environnement Ruby, vous pouvez utiliser la commande suivante :

```bash
# Remove the installed gems from the Ruby environment
gem uninstall azure_mgmt_resourcegraph
```

> [!NOTE]
> La gemme `azure_mgmt_resourcegraph` a des dépendances telles que `ms_rest` et `ms_rest_azure` qui ont peut-être également été installées en fonction de votre environnement. Vous pouvez également désinstaller ces gemmes si vous n’en avez plus besoin.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez ajouté les gemmes Resource Graph à votre environnement Ruby et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)

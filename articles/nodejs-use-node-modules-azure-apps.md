---
title: Utilisation de modules Node.js
description: Découvrez comment utiliser des modules Node.js avec Azure App Service ou Cloud Services.
services: ''
documentationcenter: nodejs
author: rloutlaw
manager: rloutlaw
editor: ''
ms.assetid: c0e6cd3d-932d-433e-b72d-e513e23b4eb6
ms.service: multiple
ms.workload: na
ms.tgt_pltfrm: na
ms.devlang: nodejs
ms.topic: article
ms.date: 08/17/2016
ms.author: routlaw
ms.openlocfilehash: f61f08f32ebb2b721846d3c3017405af99421104
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95542216"
---
# <a name="using-nodejs-modules-with-azure-applications"></a>Utilisation de modules Node.js avec des applications Azure
Ce document fournit des instructions sur l'utilisation de modules Node.js avec des applications hébergées sur Azure. Il vous explique comment vous assurer que votre application utilise une version spécifique d’un module et vous présente l’utilisation de modules natifs avec Azure.

Si vous savez utiliser les modules Node.js, les fichiers **package.json** et **npm-shrinkwrap.json**, vous trouverez ci-dessous un résumé rapide des sujets abordés dans cet article :

* Azure App Service comprend les fichiers **package.json** et **npm-shrinkwrap.json** et peut installer des modules basés sur les entrées de ces fichiers.

* Azure Cloud Services s’attend à ce que tous les modules soient installés dans l’environnement de développement et que le répertoire **node\_modules** soit inclus dans le déploiement du package. Il est possible d'activer la prise en charge pour l'installation des modules en utilisant les fichiers **package.json** ou **npm-shrinkwrap.json** sur Cloud Services. Néanmoins, cette configuration requiert la personnalisation des scripts par défaut utilisés pour les projets Cloud Services. Pour obtenir un exemple de configuration de cet environnement, consultez la page [Tâche de démarrage d'Azure permettant d'exécuter l’installation npm pour éviter le déploiement des modules de nœuds](https://github.com/woloski/nodeonazure-blog/blob/master/articles/startup-task-to-run-npm-in-azure.markdown)

> [!NOTE]
> Les machines virtuelles Azure ne sont pas abordées dans cet article, car l’expérience de déploiement sur une machine virtuelle dépend du système d'exploitation hébergé par cette dernière.
>
>

## <a name="nodejs-modules"></a>Modules Node.js
Les modules sont des packages JavaScript chargeables qui fournissent une fonctionnalité spécifique à votre application. Les modules sont habituellement installés à l'aide de l'outil de ligne de commandes **napm**. Néanmoins, certains modules (le module http par exemple) sont intégrés au package Node.js principal.

Lorsque les modules sont installés, ils sont stockés dans le répertoire **node\_modules** à la racine de la structure de répertoires de votre application. Chaque module du répertoire **node\_modules** conserve son propre répertoire qui comporte tous les modules dont il dépend. Cette structure se répète pour chaque module, tout au long de la chaîne de dépendance. Cet environnement permet que chaque module installé soit doté de ses propres exigences de version pour les modules dont il dépend, ce qui peut néanmoins résulter en une structure de répertoires assez volumineuse.

Lors du déploiement du répertoire **modules\_de nœuds** dans votre application, la taille du déploiement est accrue comparée à l'utilisation d’un fichier **package.json** ou **npm-shrinkwrap.json**. Néanmoins, ce type de déploiement garantit que la version des modules utilisés en production est la même que ceux utilisés en développement.

### <a name="native-modules"></a>Modules natifs
Alors que la plupart des modules sont simplement des fichiers JavaScript en texte brut, certains modules sont des images binaires propres à des plateformes. Ces modules sont compilés au moment de l'installation, en général à l'aide de Python et node-gyp. Azure Cloud Services se reposant sur le déploiement du dossier **node\_modules** dans l'application, tout module natif inclus dans les modules installés doit fonctionner dans un service cloud, du moment où il a été installé et compilé dans un système de développement Windows.

Azure App Service ne prend pas en charge tous les modules natifs et risque de ne pas pouvoir les compiler si des prérequis spécifiques sont requis. Alors que certains modules populaires tels que MongoDB ont des dépendances natives facultatives et fonctionnent correctement sans celles-ci, deux solutions se sont avérées opérationnelles avec presque tous les modules natifs disponibles à ce jour :

* Exécutez **npm install** sur un ordinateur Windows sur lequel sont installés tous les composants requis du module native. Ensuite, déployez le dossier **node\_modules** créé dans le cadre de l'application sur Azure App Service.

  * Avant de compiler, vérifiez que votre installation locale de Node.js dispose de la bonne architecture et que la version est aussi proche que possible de celle utilisée dans Azure (les valeurs peuvent être vérifiées lors de l’exécution dans les propriétés **process.arch** et **process.version**).

* Azure App Service peut être configuré pour exécuter des scripts Shell ou Bash personnalisés pendant le déploiement, ce qui vous permet d'exécuter des commandes personnalisées et de configurer précisément l'exécution de **npm install** . Pour afficher une vidéo montrant comment procéder, consultez [Scripts de déploiement de site Web personnalisés avec Kudu](https://azure.microsoft.com/resources/videos/custom-web-site-deployment-scripts-with-kudu/).

### <a name="using-a-packagejson-file"></a>Utilisation du fichier package.json

Le fichier **package.json** permet de spécifier les dépendances de premier niveau requises par votre application afin que la plateforme d’hébergement puisse installer ces dépendances, plutôt que de vous obliger à inclure le dossier **node\_modules** dans le cadre du déploiement. Une fois que l'application a été déployée, la commande **npm install** permet d'analyser le fichier **package.json** et d'installer toutes les dépendances répertoriées.

Au cours du développement, vous pouvez utiliser les paramètres **--save**, **--save-dev** ou **--save-optional** lors de l'installation de modules afin d'ajouter automatiquement une entrée pour le module à votre fichier **package.json**. Pour plus d'informations, consultez la page [npm-install](https://docs.npmjs.com/cli/install).

Un problème potentiel avec le fichier **package.json** est qu'il n'indique la version que pour les dépendances de niveau supérieur. Chaque module installé peut spécifier ou non la version des modules dont il dépend. Ainsi, il est possible que vous obteniez une chaîne de dépendance différente de celle utilisée en développement.

> [!NOTE]
> Pendant le déploiement sur Azure App Service, si votre fichier <b>package.json</b> fait référence à un module natif, une erreur similaire à l’exemple ci-dessous s'affiche au moment de la publication de l'application à l'aide de Git :
>
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
>
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
>
>

### <a name="using-a-npm-shrinkwrapjson-file"></a>Utilisation d'un fichier npm-shrinkwrap.json
Le fichier **npm-shrinkwrap.json** tente de résoudre les limitations de contrôle de version des modules du fichier **package.json**. Alors que le fichier **package.json** n'inclut que des versions pour les modules de niveau supérieur, le fichier **npm-shrinkwrap.json** contient les exigences de version pour l'ensemble de la chaîne de dépendance des modules.

Lorsque votre application est prête pour la production, vous pouvez verrouiller les exigences de version et créer un fichier **npm-shrinkwrap.json** à l'aide de la commande **npm shrinkwrap**. Cette commande utilise les versions actuellement installées dans le dossier **modules\_de nœuds** et les enregistre dans le fichier **npm-shrinkwrap.json**. Une fois que l'application a été déployée vers l'environnement d'hébergement, la commande **npm install** permet d'analyser le fichier **npm-shrinkwrap.json** et d'installer toutes les dépendances répertoriées. Pour plus d’informations, consultez [npm-shrinkwrap](https://docs.npmjs.com/cli/shrinkwrap).

> [!NOTE]
> Pendant le déploiement sur Azure App Service, si votre fichier <b>npm-shrinkwrap.json</b> fait référence à un module natif, une erreur similaire à l’exemple ci-dessous s'affiche au moment de la publication de l'application à l'aide de Git :
>
> npm ERR! module-name@0.6.0 install: 'node-gyp configure build'
>
> npm ERR! 'cmd "/c" "node-gyp configure build"' failed with 1
>
>

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous savez comment utiliser les modules Node.js avec Azure, découvrez comment [spécifier la version de Node.js](https://github.com/squillace/nodejs-microservice), [créer et déployer une application web Node.js](app-service/quickstart-nodejs.md) et [utiliser l'interface de ligne de commande Azure pour Mac et Linux](https://azure.microsoft.com/blog/using-windows-azure-with-the-command-line-tools-for-mac-and-linux/).

Pour plus d’informations, consultez le [Centre pour développeurs Node.js](/azure/developer/javascript/).

[specify the Node.js version]: ./app-service/overview.md
[How to use the Azure Command-Line Interface for Mac and Linux]:cli-install-nodejs.md
[Custom Website Deployment Scripts with Kudu]: https://channel9.msdn.com/Shows/Azure-Friday/Custom-Web-Site-Deployment-Scripts-with-Kudu-with-David-Ebbo
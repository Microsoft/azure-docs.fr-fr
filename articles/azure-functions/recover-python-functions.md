---
title: Résoudre des problèmes d’applications de fonction Python dans Azure Functions
description: Découvrez comment résoudre des problèmes de fonctions Python.
author: Hazhzeng
ms.topic: article
ms.date: 07/29/2020
ms.author: hazeng
ms.custom: devx-track-python
ms.openlocfilehash: 9b9f5d389eda5d74e7e78cfcfa9a46fba7276cbd
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87846035"
---
# <a name="troubleshoot-python-errors-in-azure-functions"></a>Résoudre des erreurs Python dans Azure Functions

Voici la liste des guides de résolution des problèmes courants dans des fonctions Python :

* [ModuleNotFoundError et ImportError](#troubleshoot-modulenotfounderror)
* [Impossible d’importer « cygrpc »](#troubleshoot-cannot-import-cygrpc)

## <a name="troubleshoot-modulenotfounderror"></a>Résolution du problème ModuleNotFoundError

Cette section vous aide à résoudre des erreurs liées à un module dans votre application de fonction Python. Ces erreurs entraînent généralement le message d’erreur Azure Functions suivant :

> `Exception: ModuleNotFoundError: No module named 'module_name'.`

Ce problème se produit quand une application de fonction Python ne parvient pas à charger un module Python. La cause racine de cette erreur est l’un des problèmes suivants :

- [Le package est introuvable](#the-package-cant-be-found)
- [Le package n’est pas résolu avec un fichier wheel Linux approprié](#the-package-isnt-resolved-with-proper-linux-wheel)
- [Le package n’est pas compatible avec la version de l’interpréteur Python](#the-package-is-incompatible-with-the-python-interpreter-version)
- [Le package est en conflit avec d’autres packages](#the-package-conflicts-with-other-packages)
- [Le package ne prend en charge que les plateformes Windows ou macOS](#the-package-only-supports-windows-or-macos-platforms)

### <a name="view-project-files"></a>Afficher les fichiers projet

Pour identifier la cause réelle de votre problème, vous devez récupérer les fichiers projet Python qui s’exécutent sur votre application de fonction. Si ces fichiers projet ne sont pas disponibles sur votre ordinateur local, vous pouvez les obtenir de l’une des manières suivantes :

- Si l’application de fonction a un paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` et que sa valeur est une URL, téléchargez le fichier en copiant l’URL et en la collant dans votre navigateur.
- Si l’application de fonction a un paramètre d’application `WEBSITE_RUN_FROM_PACKAGE` et qu’il a la valeur `1`, accédez à `https://<app-name>.scm.azurewebsites.net/api/vfs/data/SitePackages` et téléchargez le fichier à partir de la dernière URL `href`.
- Si l’application de fonction n’a pas le paramètre d’application indiqué ci-dessus, accédez à `https://<app-name>.scm.azurewebsites.net/api/settings` et recherchez l’URL sous `SCM_RUN_FROM_PACKAGE`. Téléchargez le fichier en copiant l’URL et en la collant dans votre navigateur.
- Si aucune de ces propositions ne vous donne les résultats escomptés, accédez à `https://<app-name>.scm.azurewebsites.net/DebugConsole` et révélez le contenu sous `/home/site/wwwroot`.

Le reste de cet article détaille la procédure à suivre pour résoudre les causes potentielles de cette erreur : inspection du contenu de votre application de fonction, identification de la cause racine et résolution du problème spécifique.

### <a name="diagnose-modulenotfounderror"></a>Diagnostiquer ModuleNotFoundError

Cette section détaille les causes racines potentielles des erreurs liées à un module. Une fois que vous aurez déterminé la cause racine probable, vous pourrez accéder aux solutions d’atténuation associées.

#### <a name="the-package-cant-be-found"></a>Le package est introuvable

Accédez à `.python_packages/lib/python3.6/site-packages/<package-name>` ou à `.python_packages/lib/site-packages/<package-name>`. Si le chemin au fichier n’existe pas, ce chemin manquant est probablement la cause racine.

L’utilisation d’outils tiers ou obsolètes durant le déploiement peut occasionner ce problème.

Consultez [Activer la build distante](#enable-remote-build) ou [Créer des dépendances natives](#build-native-dependencies) à des fins d’atténuation.

#### <a name="the-package-isnt-resolved-with-proper-linux-wheel"></a>Le package n’est pas résolu avec un fichier wheel Linux approprié

Accédez à `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` ou à `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. Utilisez votre éditeur de texte préféré pour ouvrir le fichier **wheel** et consultez la section **Tag:** . Si la valeur de la balise ne contient pas **linux**, le problème peut se situer là.

Les fonctions Python s’exécutent uniquement sur Linux dans Azure : Le runtime Functions v2.x s’exécute sur Debian Stretch et le runtime v3.x sur Debian Buster. L’artefact doit contenir les binaires Linux appropriés. L’utilisation de l’indicateur `--build local` dans Core Tools, d’outils tiers ou d’outils obsolètes peut entraîner l’utilisation de binaires plus anciens.

Consultez [Activer la build distante](#enable-remote-build) ou [Créer des dépendances natives](#build-native-dependencies) à des fins d’atténuation.

#### <a name="the-package-is-incompatible-with-the-python-interpreter-version"></a>Le package n’est pas compatible avec la version de l’interpréteur Python

Accédez à `.python_packages/lib/python3.6/site-packages/<package-name>-<version>-dist-info` ou à `.python_packages/lib/site-packages/<package-name>-<version>-dist-info`. À l’aide d’un éditeur de texte, ouvrez le fichier METADATA et examinez la section **Classifiers:** . Si la section ne contient pas `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` ou `Python :: 3.8`, cela signifie que la version du package est trop ancienne ou, plus vraisemblablement, que le package est déjà hors de maintenance.

Vous pouvez vérifier la version de Python de votre application de fonction à partir du [portail Azure](https://portal.azure.com). Accédez à votre application de fonction, choisissez **Explorateur de ressources**, puis sélectionnez **Accéder**.

:::image type="content" source="media/recover-module-not-found/resource-explorer.png" alt-text="Ouvrir l’Explorateur de ressources pour l’application de fonction dans le portail":::

Une fois l’explorateur chargé, recherchez **LinuxFxVersion**, qui indique la version de Python.

Consultez [Mettre à jour votre package vers la dernière version](#update-your-package-to-the-latest-version) ou [Remplacer le package par des équivalents](#replace-the-package-with-equivalents) à des fins d’atténuation.

#### <a name="the-package-conflicts-with-other-packages"></a>Le package est en conflit avec d’autres packages

Si vous avez vérifié que le package est correctement résolu avec les fichiers wheel Linux appropriés, il peut y avoir un conflit avec d’autres packages. Dans certains packages, les documentations PyPi peuvent clarifier les modules incompatibles. Par exemple, dans [`azure 4.0.0`](https://pypi.org/project/azure/4.0.0/), vous trouverez la déclaration suivante :

<pre>This package isn't compatible with azure-storage.
If you installed azure-storage, or if you installed azure 1.x/2.x and didn’t uninstall azure-storage,
you must uninstall azure-storage first.</pre>

Vous trouverez la documentation de votre version de package dans `https://pypi.org/project/<package-name>/<package-version>`.

Consultez [Mettre à jour votre package vers la dernière version](#update-your-package-to-the-latest-version) ou [Remplacer le package par des équivalents](#replace-the-package-with-equivalents) à des fins d’atténuation.

#### <a name="the-package-only-supports-windows-or-macos-platforms"></a>Le package ne prend en charge que les plateformes Windows ou macOS

Ouvrez `requirements.txt` dans un éditeur de texte et vérifiez le package dans `https://pypi.org/project/<package-name>`. Certains packages s’exécutent uniquement sur des plateformes Windows ou macOS. Par exemple, pywin32 s’exécute uniquement sur Windows.

L’erreur `Module Not Found` peut ne pas se produire quand vous utilisez Windows ou macOS pour le développement local. Toutefois, le package n’est pas importé dans Azure Functions qui utilise Linux au moment de l’exécution. Cela est probablement dû à l’utilisation de `pip freeze` pour exporter un environnement virtuel dans requirements.txt à partir de votre ordinateur Windows ou macOS durant l’initialisation du projet.

Consultez [Remplacer le package par des équivalents](#replace-the-package-with-equivalents) ou [Fabriquer vous-même requirements.txt](#handcraft-requirementstxt) à des fins d’atténuation.

### <a name="mitigate-modulenotfounderror"></a>Atténuer ModuleNotFoundError

Voici des mesures d’atténuation potentielles pour les problèmes liés à un module. Utilisez les [diagnostics ci-dessus](#diagnose-modulenotfounderror) pour déterminer les mesures d’atténuation à essayer.

#### <a name="enable-remote-build"></a>Activer la build distante

Vérifiez que la build distante est activée. La façon de procéder dépend de votre méthode de déploiement.

## <a name="visual-studio-code"></a>[Visual Studio Code](#tab/vscode)
Vérifiez que la dernière version de l’[extension Azure Functions pour Visual Studio Code](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) est installée. Vérifiez que `.vscode/settings.json` existe et qu’il contient le paramètre `"azureFunctions.scmDoBuildDuringDeployment": true`. Si ce n’est pas le cas, créez ce fichier avec le paramètre `azureFunctions.scmDoBuildDuringDeployment` activé et redéployez le projet.

## <a name="azure-functions-core-tools"></a>[Azure Functions Core Tools](#tab/coretools)

Vérifiez que la dernière version d’[Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases) est installée. Accédez au dossier de votre projet de fonction local et utilisez `func azure functionapp publish <app-name>` pour le déploiement.

## <a name="manual-publishing"></a>[Publication manuelle](#tab/manual)

Si vous publiez manuellement votre package dans le point de terminaison `https://<app-name>.scm.azurewebsites.net/api/zipdeploy`, vérifiez que **SCM_DO_BUILD_DURING_DEPLOYMENT** et **ENABLE_ORYX_BUILD** ont la valeur **true**. Pour en savoir plus, voyez [comment utiliser les paramètres d’application](functions-how-to-use-azure-function-app-settings.md#settings).

---

#### <a name="build-native-dependencies"></a>Générer des dépendances natives

Vérifiez que les dernières versions de **docker** et d’[Azure Functions Core Tools](https://github.com/Azure/azure-functions-core-tools/releases) sont installées. Accédez au dossier de votre projet de fonction local et utilisez `func azure functionapp publish <app-name> --build-native-deps` pour le déploiement.

#### <a name="update-your-package-to-the-latest-version"></a>Mettre à jour votre package avec la dernière version

Accédez à la dernière version du package dans `https://pypi.org/project/<package-name>` et examinez la section **Classifieurs**. Le package doit être `OS Independent` ou compatible avec `POSIX` ou `POSIX :: Linux` dans **Système d’exploitation**. En outre, le langage de programmation doit contenir `Python :: 3`, `Python :: 3.6`, `Python :: 3.7` ou `Python :: 3.8`.

Si tout ceci est correct, vous pouvez mettre à jour le package avec la dernière version en modifiant la ligne `<package-name>~=<latest-version>` dans requirements.txt.

#### <a name="handcraft-requirementstxt"></a>Fabriquer vous-même requirements.txt

Certains développeurs utilisent `pip freeze > requirements.txt` pour générer la liste des packages Python pour leurs environnements de développement. Bien que cette pratique fonctionne le plus souvent, des problèmes peuvent se présenter dans les scénarios de déploiement multiplateforme. C’est notamment le cas si vous développez des fonctions localement sur Windows ou macOS, mais que vous les publiez sur une application de fonction qui s’exécute sur Linux. Dans un tel scénario, `pip freeze` peut introduire des dépendances inattendues spécifiques au système d’exploitation ou des dépendances pour votre environnement de développement local. Ces dépendances peuvent provoquer l’arrêt de l’application de fonction Python en cas d’exécution sur Linux.

La bonne pratique consiste à vérifier l’instruction import dans chaque fichier .py du code source de votre projet et à archiver uniquement ces modules dans le fichier requirements.txt. Ainsi, la résolution des packages est correctement gérée sur différents systèmes d’exploitation.

#### <a name="replace-the-package-with-equivalents"></a>Remplacer le package par des équivalents

Nous devons d’abord examiner si nous disposons de la version la plus récente du package dans `https://pypi.org/project/<package-name>`. En général, ce package a sa propre page GitHub. Accédez à la section **Issues** sur GitHub et lancez une recherche pour savoir si votre problème a été résolu. Si tel est le cas, mettez à jour le package avec la dernière version.

Parfois, le package peut être intégré à la bibliothèque [Python Standard Library](https://docs.python.org/3/library/) (par exemple, pathlib). Dans ce cas, étant donné que nous fournissons une certaine distribution Python dans Azure Functions (Python 3.6, Python 3.7 et Python 3.8), le package dans votre fichier requirements.txt doit être supprimé.

Toutefois, si vous êtes confronté à un problème qui n’a pas été résolu et que vous devez respecter des échéances, je vous encourage à faire des recherches et à trouver un package similaire pour votre projet. En général, la communauté Python met à votre disposition une grande variété de bibliothèques similaires.

---

## <a name="troubleshoot-cannot-import-cygrpc"></a>La résolution de problème ne peut pas importer « cygrpc »

Cet article vous aide à résoudre les erreurs liées à « cygrpc » dans votre application de fonction Python. Ces erreurs entraînent généralement le message d’erreur Azure Functions suivant :

> `Cannot import name 'cygrpc' from 'grpc._cython'`

Cette erreur se produit quand une application de fonction Python ne parvient pas à démarrer avec un interpréteur Python approprié. La cause racine de cette erreur est l’un des problèmes suivants :

- [L’interpréteur python ne correspond pas à l’architecture du système d’exploitation](#the-python-interpreter-mismatches-os-architecture)
- [L’interpréteur Python n’est pas pris en charge par le Worker Python Azure Functions](#the-python-interpreter-is-not-supported-by-azure-functions-python-worker)

### <a name="diagnose-cygrpc-reference-error"></a>Diagnostiquer l’erreur de référence « cygrpc »

#### <a name="the-python-interpreter-mismatches-os-architecture"></a>L’interpréteur Python est mal incompatible l’architecture du système d’exploitation

Cela est probablement dû au fait qu’un interpréteur Python 32 bits est installé sur votre système d’exploitation 64 bits.

Si vous exécutez sur un système d’exploitation x64, vérifiez que votre interpréteur Python 3.6, 3.7 ou 3.8 est également sur une version 64 bits.

Vous pouvez vérifier le nombre de bits de votre interpréteur Python à l’aide des commandes suivantes :

Sur Windows dans PowerShell : `py -c 'import platform; print(platform.architecture()[0])'`

Sur un interpréteur de commandes de type Unix : `python3 -c 'import platform; print(platform.architecture()[0])'`

En cas d’incompatibilité entre le nombre de bits de l’interpréteur Python et l’architecture du système d’exploitation, téléchargez un interpréteur python approprié à partir de [Python Software Foundation](https://python.org/downloads/release).

#### <a name="the-python-interpreter-is-not-supported-by-azure-functions-python-worker"></a>L’interpréteur Python n’est pas pris en charge par le Worker Python Azure Functions

Le Worker Python Azure Functions prend uniquement en charge Python 3.6, 3.7 et 3.8.
Vérifiez si votre interpréteur Python correspond à notre version attendue en utilisant `py --version` sous Windows ou `python3 --version` dans des systèmes de type Unix. Assurez-vous que le résultat retourné est Python 3.6.x, Python 3.7.x ou Python 3.8.x.

Si votre version de l’interpréteur Python ne répond pas à notre attente, téléchargez l’interpréteur Python 3.6, 3.7 ou 3.8 à partir de [Python Software Foundation](https://python.org/downloads/release).

## <a name="next-steps"></a>Étapes suivantes

Si vous ne parvenez pas à résoudre votre problème, signalez-le à l’équipe Functions :

> [!div class="nextstepaction"]
> [Signaler un problème non résolu](https://github.com/Azure/azure-functions-python-worker/issues)

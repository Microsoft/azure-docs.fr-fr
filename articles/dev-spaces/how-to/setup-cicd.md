---
title: Utilisation de l’intégration continue/la livraison continue (CI/CD) avec Azure Dev Spaces
services: azure-dev-spaces
author: DrEsteban
ms.author: stevenry
ms.date: 12/17/2018
ms.topic: conceptual
manager: gwallace
description: Découvrez comment configurer l’intégration continue/le déploiement continu à l’aide d’Azure DevOps avec Azure Dev Spaces
keywords: Docker, Kubernetes, Azure, AKS, Azure Container Service, conteneurs
ms.openlocfilehash: de409aa060034c9ba0faaaa56ce21f904b02cdac
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017778"
---
# <a name="use-cicd-with-azure-dev-spaces"></a>Utiliser CI/CD avec Azure Dev Spaces

[!INCLUDE [Azure Dev Spaces deprecation](../../../includes/dev-spaces-deprecation.md)]

Cet article vous guide tout au long de la configuration de l’intégration continue/du déploiement continu (CI/CD) sur Azure Kubernetes Service (AKS) où Dev Spaces est activé. CI/CD vers AKS permet de déployer automatiquement des mises à jour d’application chaque fois que du code validé est envoyé (push) à votre dépôt source. L’utilisation de CI/CD conjointement avec un cluster compatible Dev Spaces est utile, car elle permet de tenir à jour une base de référence de l’application que l’équipe peut utiliser.

![Exemple de diagramme CI/CD](../media/common/ci-cd-simple.png)

Bien que cet article vous guide avec Azure DevOps, les mêmes concepts sont applicables aux systèmes CI/CD tels que Jenkins, TeamCity, etc.

## <a name="prerequisites"></a>Prérequis
* Cluster AKS (Azure Kubernetes Service) avec Azure Dev Spaces activé
* [Interface CLI Azure Dev Spaces installée](upgrade-tools.md)
* [Organisation Azure DevOps avec un projet](/azure/devops/user-guide/sign-up-invite-teammates?view=vsts)
* [ACR (Azure Container Registry)](../../container-registry/container-registry-get-started-azure-cli.md)
    * Détails du [compte d’administrateur](../../container-registry/container-registry-authentication.md#admin-account) Azure Container Registry disponibles
* [Autoriser votre cluster AKS à tirer (pull) de votre registre Azure Container Registry](../../aks/cluster-container-registry-integration.md)

## <a name="download-sample-code"></a>Télécharger l’exemple de code
Pour des questions de temps, créons une duplication (fork) de notre exemple de dépôt GitHub de code. Accédez à https://github.com/Azure/dev-spaces et sélectionnez **Duplication (fork)** . Une fois que le processus de duplication est terminé, **clonez** votre version dupliquée du dépôt localement. Par défaut, la branche _maîtresse_ est extraite, mais nous avons inclus quelques modifications pour gagner du temps dans la branche _azds_updates_, qui doit également avoir été transférée au cours de la duplication. La branche _azds_updates_ contient des mises à jour que nous vous demandons d’effectuer manuellement dans les sections du tutoriel Dev Spaces, ainsi que certains fichiers YAML et JSON prédéfinis pour simplifier le déploiement du système CI/CD. Vous pouvez utiliser une commande similaire à `git checkout -b azds_updates origin/azds_updates` pour extraire la branche _azds_updates_ dans votre dépôt local.

## <a name="dev-spaces-setup"></a>Configuration de Dev Spaces
Créez un espace appelé _dev_ à l’aide de la commande `azds space select`. L’espace _dev_ sera utilisé par votre pipeline CI/CD pour envoyer vos modifications de code. Il va également servir à créer des _espaces enfants_ selon _dev_.

```cmd
azds space select -n dev
```

Quand vous êtes invité à sélectionner un espace de développement parent, sélectionnez _\<none\>_ .

Une fois que votre espace de développement a été créé, vous devez déterminer le suffixe de l’hôte. Utilisez la commande `azds show-context` pour afficher le suffixe d’hôte du contrôleur d’entrée Azure Dev Spaces.

```cmd
$ azds show-context
Name   ResourceGroup    DevSpace  HostSuffix
-----  ---------------  --------  ----------------------
MyAKS  MyResourceGroup  dev       fedcba098.eus.azds.io
```

Dans l’exemple ci-dessus, le suffixe d’hôte est _fedcba098.eus.azds.io_. Cette valeur est utilisée ultérieurement lors de la création de votre définition de mise en production.

L’espace _dev_ contient toujours le dernier état du dépôt, une base de référence, afin que les développeurs puissent créer des _espaces enfants_ de _dev_ pour tester leurs modifications isolées dans le contexte de l’application plus volumineuse. Ce concept est décrit plus en détail dans les tutoriels Dev Spaces.

## <a name="creating-the-build-definition"></a>Création de la définition de build
Ouvrez votre projet d’équipe Azure DevOps dans un navigateur web et accédez à la section _Pipelines_. Tout d’abord, cliquez sur votre photo de profil dans le coin supérieur droit du site Azure DevOps, ouvrez le volet des fonctionnalités en préversion et désactivez l’option _Nouvelle expérience utilisateur de création de pipeline YAML_ :

![Ouverture du volet des fonctionnalités en préversion](../media/common/preview-feature-open.png)

L’option à désactiver :

![Option Nouvelle expérience utilisateur de création de pipeline YAML](../media/common/yaml-pipeline-preview-feature.png)

> [!Note]
> La fonctionnalité en préversion _Nouvelle expérience utilisateur de création de pipeline YAML_ Azure DevOps est en conflit avec la création de pipelines de build prédéfinis pour l’instant. Vous devez la désactiver pour le moment afin de déployer notre pipeline de build prédéfini.

Dans la branche _azds_updates_, nous avons inclus un simple [pipeline YAML Azure](/azure/devops/pipelines/yaml-schema?view=vsts&tabs=schema) qui définit les étapes de génération nécessaires pour *mywebapi* et *webfrontend*.

Selon le langage que vous avez choisi, le pipeline YAML a été archivé dans un chemin semblable à : `samples/dotnetcore/getting-started/azure-pipelines.dotnetcore.yml`

Pour créer un pipeline à partir de ce fichier :
1. Dans la page principale de votre projet DevOps, accédez à Pipelines > Builds.
1. Sélectionnez l’option permettant de créer un **Nouveau** pipeline de build.
1. Sélectionnez **GitHub** comme source, donnez l’autorisation avec votre compte GitHub si nécessaire, puis sélectionnez la branche _azds_updates_ à partir de votre version dupliquée du dépôt de l’exemple d’application _dev-spaces_.
1. Sélectionnez **Configuration sous forme de code** ou **YAML** comme modèle.
1. Une page de configuration s’affiche à présent pour votre pipeline de build. Comme mentionné ci-dessus, naviguez jusqu’à l’emplacement propre au langage pour le **chemin du fichier YAML** à l’aide du bouton **...** . Par exemple : `samples/dotnetcore/getting-started/azure-pipelines.dotnet.yml`.
1. Accédez à l’onglet **Variables**.
1. Ajoutez manuellement _dockerId_ comme variable, qui correspond au nom d’utilisateur de votre [compte d’administrateur Azure Container Registry](../../container-registry/container-registry-authentication.md#admin-account). (Élément mentionné dans les prérequis de l’article)
1. Ajoutez manuellement _dockerPassword_ comme variable, qui correspond au mot de passe de votre [compte d’administrateur Azure Container Registry](../../container-registry/container-registry-authentication.md#admin-account). Veillez à spécifier _dockerPassword_ en tant que Secret (en sélectionnant l’icône de verrou) pour des raisons de sécurité.
1. Sélectionnez **Enregistrer et mettre en file d’attente**.

Vous disposez maintenant d’une solution CI qui va générer automatiquement *mywebapi* et *webfrontend* pour toute mise à jour envoyée vers la branche _azds_updates_ de votre duplication GitHub. Vous pouvez vérifier que les images Docker ont été envoyées en accédant au portail Azure, en sélectionnant votre registre Azure Container Registry et en accédant à l’onglet **Dépôts**. La génération des images et leur affichage dans votre registre de conteneurs peuvent prendre plusieurs minutes.

![Dépôts Azure Container Registry](../media/common/ci-cd-images-verify.png)

## <a name="creating-the-release-definition"></a>Création de la définition de mise en production

1. Dans la page principale de votre projet DevOps, accédez à Pipelines > Mises en production
1. Si vous travaillez sur un tout nouveau projet DevOps qui ne contient pas encore une définition de mise en production, vous devez d’abord en créer une vide avant de continuer. L’option d’importation ne s’affiche pas dans l’interface utilisateur tant que vous n’avez pas de définition de mise en production existante.
1. Sur la gauche, cliquez sur le bouton **+ Nouveau**, puis sur **Importer un pipeline**.
1. Cliquez sur **Parcourir** et sélectionnez `samples/release.json` à partir de votre projet.
1. Cliquez sur **OK**. Notez que le volet Pipeline est chargé avec la page de modification de définition de mise en production. Notez aussi la présence de certaines icônes d’avertissement rouges indiquant des détails spécifiques au cluster qui doivent toujours être configurés.
1. Sur la gauche du volet Pipeline, cliquez sur la bulle **Ajouter un artefact**.
1. Dans la liste déroulante **Source**, sélectionnez le pipeline de build créé plus tôt.
1. Pour **Version par défaut**, choisissez **La dernière de la branche par défaut du pipeline de build avec les étiquettes**.
1. Laissez **Balises** vide.
1. Affectez à **Alias source** la valeur `drop`. Le valeur **Alias source** est utilisée par les tâches de mise en production prédéfinies ; vous devez donc la définir.
1. Cliquez sur **Add**.
1. Cliquez maintenant sur l’icône représentant un éclair sur la source d’artefact `drop` nouvellement créée, comme indiqué ci-dessous :

    ![Configuration du déploiement continu de l’artefact de mise en production](../media/common/release-artifact-cd-setup.png)
1. Activez le **déclencheur de déploiement continu**.
1. Placez le curseur sur l’onglet **Tâches** en regard de **Pipeline** et cliquez sur _dev_ pour modifier les tâches de la phase _dev_.
1. Vérifiez qu’**Azure Resource Manager** est sélectionné sous **Type de connexion**. Les trois contrôles de liste déroulante sont mis en surbrillance en rouge : ![Configuration de la définition de mise en production](../media/common/release-setup-tasks.png)
1. Sélectionnez l’abonnement Azure que vous utilisez avec Azure Dev Spaces. Vous devrez peut-être aussi cliquer sur **Autoriser**.
1. Sélectionnez le groupe de ressources et le cluster que vous utilisez avec Azure Dev Spaces.
1. Cliquez sur **Travail de l’Agent**.
1. Sélectionnez **Ubuntu 1604 hébergé** sous **Pool d’agents**.
1. Placez le curseur sur le sélecteur **Tâches** en haut, et cliquez sur _prod_ pour modifier les tâches de la phase _prod_.
1. Vérifiez qu’**Azure Resource Manager** est sélectionné sous **Type de connexion**. Spécifiez l’abonnement Azure, le groupe de ressources et le cluster que vous utilisez avec Azure Dev Spaces.
1. Cliquez sur **Travail de l’Agent**.
1. Sélectionnez **Ubuntu 1604 hébergé** sous **Pool d’agents**.
1. Cliquez sur l’onglet **Variables** afin de mettre à jour les variables pour votre mise en production.
1. Mettez à jour la valeur de **DevSpacesHostSuffix** en remplaçant **UPDATE_ME** par votre suffixe d’hôte. Le suffixe d’hôte s’est affiché quand vous avez exécuté la commande `azds show-context` précédemment.
1. Cliquez sur **Enregistrer** dans le coin supérieur droit, puis sur **OK**.
1. Cliquez sur **+ Mise en production** (en regard du bouton Enregistrer) et **Créer une mise en production**.
1. Sous **Artefacts**, vérifiez que la build la plus récente de votre pipeline de build est sélectionnée.
1. Cliquez sur **Créer**.

Un processus de mise en production automatisé commence, déployant les graphiques *mywebapi* et *webfrontend* sur votre cluster Kubernetes dans l’espace de niveau supérieur _dev_. Vous pouvez superviser la progression de votre mise en production dans le portail web Azure DevOps :

1. Accédez à la section **Mises en production** sous **Pipelines**.
1. Cliquez sur le pipeline de mise en production correspondant à l’exemple d’application.
1. Cliquez sur le nom de la dernière mise en production.
1. Placez le curseur sur **dev** sous **Phases** et cliquez sur **Journaux**.

La mise en production est terminée quand toutes les tâches sont terminées.

> [!TIP]
> Si votre mise en production échoue avec un message d’erreur de type *ÉCHEC DE LA MISE À NIVEAU : expiration du délai d’attente pour la condition*, essayez d’inspecter les pods dans votre cluster [à l’aide du tableau de bord Kubernetes](../../aks/kubernetes-dashboard.md). Si vous voyez que les pods ne parviennent pas à démarrer avec des messages d’erreur comme *Échec de l’extraction de l’image « azdsexample.azurecr.io/mywebapi:122 » : erreur rpc : code = inconnu desc = Réponse d’erreur du démon : Obtention de https:\//azdsexample.azurecr.io/v2/mywebapi/manifests/122 : non autorisée : authentification requise*, cela peut signifier que votre cluster n’a pas été autorisé à tirer de votre registre Azure Container Registry. Vérifiez que vous avez rempli le prérequis [Autoriser votre cluster AKS à tirer (pull) de votre registre Azure Container Registry](../../aks/cluster-container-registry-integration.md).

Vous disposez maintenant d’un pipeline CI/CD entièrement automatisé pour votre duplication GitHub des exemples d’applications Dev Spaces. Chaque fois que vous validez et envoyez le code, le pipeline de build va générer et transmettre les images *mywebapi* et *webfrontend* à votre instance ACR personnalisée. Ensuite, le pipeline de mise en production va déployer le graphique Helm pour chaque application dans l’espace _dev_ sur votre cluster compatible Dev Spaces.

## <a name="accessing-your-_dev_-services"></a>Accès à vos services _dev_
Après le déploiement, la version _dev_ de *webfrontend* est accessible avec une URL publique comme : `http://dev.webfrontend.fedcba098.eus.azds.io`. Vous trouverez cette URL en exécutant la commande `azds list-uri` : 

```cmd
$ azds list-uris

Uri                                           Status
--------------------------------------------  ---------
http://dev.webfrontend.fedcba098.eus.azds.io  Available
```

## <a name="deploying-to-production"></a>Déploiement en production

Pour promouvoir manuellement une mise en production particulière en _prod_ en utilisant le système CI/CD créé dans ce tutoriel :
1. Accédez à la section **Mises en production** sous **Pipelines**.
1. Cliquez sur le pipeline de mise en production correspondant à l’exemple d’application.
1. Cliquez sur le nom de la dernière mise en production.
1. Placez le curseur sur la zone **prod** sous **Phases** et cliquez sur **Déployer**.
    ![Promouvoir en production](../media/common/prod-promote.png)
1. Placez à nouveau le curseur sur la zone **prod** sous **Phases** et cliquez sur **Journaux**.

La mise en production est terminée quand toutes les tâches sont terminées.

La phase _prod_ du pipeline CI/CD utilise un équilibreur de charge au lieu du contrôleur d’entrée Dev Spaces pour fournir l’accès aux services _prod_. Les services déployés dans la phase _prod_ sont accessibles en tant qu’adresses IP plutôt que noms DNS. Dans un environnement de production, vous pouvez choisir de créer votre propre contrôleur d’entrée pour héberger vos services en fonction de votre propre configuration DNS.

Pour déterminer l’adresse IP du service webfrontend, cliquez sur l’étape **Imprimer l’adresse IP publique webfrontend** pour développer la sortie du journal. Utilisez l’adresse IP affichée dans la sortie du journal pour accéder à l’application **webfrontend**.

```cmd
...
2019-02-25T22:53:02.3237187Z webfrontend can be accessed at http://52.170.231.44
2019-02-25T22:53:02.3320366Z ##[section]Finishing: Print webfrontend public IP
...
```

## <a name="dev-spaces-instrumentation-in-production"></a>Instrumentation Dev Spaces en production
Même si l’instrumentation Dev Spaces a été conçue pour ne _pas_ interférer avec le fonctionnement normal de votre application, nous vous recommandons d’exécuter vos charges de travail de production dans un espace de noms Kubernetes qui n’est pas activé avec Dev Spaces. L’utilisation de ce type d’espace de noms Kubernetes signifie que vous devez créer votre espace de noms de production à l’aide de l’interface CLI `kubectl` ou permettre à votre système CI/CD de le créer lors du premier déploiement Helm. La _sélection_ ou toute création d’un espace à l’aide d’outils Dev Spaces ajoute l’instrumentation Dev Spaces à cet espace de noms.

Voici un exemple de structure d’espace de noms qui prend en charge le développement de fonctionnalités, l’environnement « dev » _et_ de production, le tout dans un seul cluster Kubernetes :

![Exemple de structure d’espace de noms](../media/common/cicd-namespaces.png)

> [!Tip]
> Si vous avez déjà créé un espace `prod` et voulez simplement l’exclure de l’instrumentation Dev Spaces (sans le supprimer !), vous pouvez le faire avec la commande de l’interface CLI Dev Spaces suivante :
>
> `azds space remove -n prod --no-delete`
>
> Vous devrez peut-être supprimer tous les pods de l’espace de noms `prod` une fois cette opération effectuée afin qu’ils puissent être recréés sans l’instrumentation Dev Spaces.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage plus sur le fonctionnement d’Azure Dev Spaces.

> [!div class="nextstepaction"]
> [Fonctionnement d’Azure Dev Spaces](../how-dev-spaces-works.md)

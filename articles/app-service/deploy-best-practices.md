---
title: Bonnes pratiques de déploiement
description: Découvrez les principaux mécanismes de déploiement dans Azure App Service. Trouvez des recommandations propres au langage et autres mises en garde.
keywords: Azure App Service, application web, déployer, déploiement, pipelines, build
author: jasonfreeberg
ms.assetid: bb51e565-e462-4c60-929a-2ff90121f41d
ms.topic: article
ms.date: 07/31/2019
ms.author: jafreebe
ms.openlocfilehash: 37c1854aeb1a1fa3d9283c00b07c665b213b306c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91708150"
---
# <a name="deployment-best-practices"></a>Meilleures pratiques de déploiement

Chaque équipe de développement a des exigences uniques qui peuvent compliquer l’implémentation d’un pipeline de déploiement efficace sur un service cloud. Cet article présente les trois principaux composants du déploiement sur App Service : les sources de déploiement, les pipelines de génération et les mécanismes de déploiement. Cet article présente également quelques-unes des meilleures pratiques et des conseils pour des piles de langage spécifiques.

## <a name="deployment-components"></a>Composants de déploiement

### <a name="deployment-source"></a>Source de déploiement

Une source de déploiement correspond à l’emplacement de votre code d’application. Pour les applications de production, la source de déploiement est généralement un référentiel hébergé par un logiciel de gestion de version, comme [GitHub, BitBucket ou Azure Repos](deploy-continuous-deployment.md). Pour les scénarios de développement et de test, la source de déploiement peut être [un projet sur votre ordinateur local](deploy-local-git.md). App Service prend également en charge les [dossiers OneDrive et Dropbox](deploy-content-sync.md) comme sources de déploiement. Bien que les dossiers cloud puissent faciliter la prise en main d’App Service, il n’est généralement pas recommandé d’utiliser cette source pour les applications de production au niveau de l’entreprise. 

### <a name="build-pipeline"></a>Pipeline de build

Une fois que vous avez choisi une source de déploiement, l’étape suivante consiste à choisir un pipeline de build. Un pipeline de build lit votre code source à partir de la source de déploiement et exécute une série d’étapes (telles que la compilation de code, la minimisation du HTML et du JavaScript, l’exécution de tests et l’empaquetage de composants) pour rendre l’application exécutable. Les commandes spécifiques exécutées par le pipeline de build dépendent de votre pile de langage. Ces opérations peuvent être exécutées sur un serveur de builds, comme Azure Pipelines, ou exécutées localement.

### <a name="deployment-mechanism"></a>Mécanisme de déploiement

Le mécanisme de déploiement est l’action utilisée pour placer votre application intégrée dans le répertoire */home/site/wwwroot* de votre application web. Le répertoire */wwwroot* est un emplacement de stockage monté partagé par toutes les instances de votre application web. Lorsque le mécanisme de déploiement place votre application dans ce répertoire, vos instances reçoivent une notification pour synchroniser les nouveaux fichiers. App Service prend en charge les mécanismes de déploiement suivants :

- Points de terminaison Kudu : [Kudu](https://github.com/projectkudu/kudu/wiki) est l’outil de productivité de développement open source qui s’exécute en tant que processus distinct dans App Service Windows, et en tant que deuxième conteneur dans App Service Linux. Kudu gère les déploiements continus et fournit des points de terminaison HTTP pour le déploiement, comme zipdeploy.
- FTP et WebDeploy : À l' aide des [informations d’identification de votre site ou de votre utilisateur](deploy-configure-credentials.md), vous pouvez télécharger des fichiers [via FTP](deploy-ftp.md) ou WebDeploy. Ces mécanismes ne passent pas par Kudu.  

Les outils de déploiement, tels qu’Azure Pipelines, Jenkins et les plug-ins d’éditeur utilisent un de ces mécanismes de déploiement.

## <a name="use-deployment-slots"></a>Utiliser des emplacements de déploiement

Dans la mesure du possible, utilisez des [emplacements de déploiement](deploy-staging-slots.md) lors du déploiement d’un nouveau build de production. Lorsque vous utilisez un plan App Service Standard ou mieux, vous pouvez déployer votre application dans un environnement intermédiaire, valider vos modifications et effectuer des tests de vérification de build. Lorsque vous êtes prêt, vous pouvez échanger vos emplacements intermédiaires et de production. L’opération d’échange préchauffe les instances de worker nécessaires pour correspondre à votre échelle de production, ce qui élimine les temps d’arrêt.

### <a name="continuously-deploy-code"></a>Déployer du code en continu

Si votre projet a désigné des branches pour le test, l’assurance qualité et la mise en lots, chacune de ces branches doit être déployée en continu vers un emplacement de préproduction. (C’est ce que l’on appelle la [conception Gitflow](https://www.atlassian.com/git/tutorials/comparing-workflows/gitflow-workflow).) Cela permet à vos parties prenantes d’évaluer et de tester facilement la branche déployée. 

Le déploiement continu ne doit jamais être activé pour votre emplacement de production. Au lieu de cela, votre branche de production (souvent maître) doit être déployée sur un emplacement de non-production. Lorsque vous êtes prêt à mettre en production la branche de base, échangez-la dans l’emplacement de production. Un échange en production, au lieu d’un déploiement en production, vous permet d’éviter les temps d’arrêt et de restaurer les modifications en les échangeant à nouveau. 

![Diagramme montrant le flux entre les branches Dev, Staging et Master, ainsi que les emplacements dans lesquels elles sont déployées.](media/app-service-deploy-best-practices/slot_flow_code_diagam.png)

### <a name="continuously-deploy-containers"></a>Déployer des conteneurs en continu

Pour les conteneurs personnalisés de Docker ou d’autres registres de conteneurs, déployez l’image dans un emplacement de préproduction et échangez en production pour éviter les temps d’arrêt. L’automatisation est plus complexe que le déploiement du code, car vous devez envoyer (push) l’image vers un registre de conteneurs et mettre à jour la balise d’image sur l’application web.

Pour chaque branche que vous souhaitez déployer sur un emplacement, configurez l’automatisation pour effectuer les opérations suivantes à chaque validation de la branche.

1. **Générez et balisez l’image**. Dans le cadre du pipeline de build, balisez l’image avec l’ID de validation git, l’horodateur ou d’autres informations identifiables. Il est préférable de ne pas utiliser la balise « latest » par défaut. Sinon, il est difficile de retracer le code actuellement déployé, ce qui rend le débogage beaucoup plus compliqué.
1. **Envoyez (push) l’image balisée**. Une fois l’image générée et balisée, le pipeline envoie (push) l’image à notre registre de conteneurs. À l’étape suivante, l’emplacement de déploiement extraira l’image balisée du registre de conteneurs.
1. **Mettez à jour l’emplacement de déploiement avec la nouvelle balise d’image**. Lorsque cette propriété est mise à jour, le site redémarre automatiquement et extrait la nouvelle image conteneur.

![Contrôle de l’utilisation de l’emplacement](media/app-service-deploy-best-practices/slot_flow_container_diagram.png)

Vous trouverez ci-dessous des exemples d’infrastructures d’automatisation courantes.

### <a name="use-azure-devops"></a>Utiliser Azure DevOps

App Service est doté de la [livraison continue intégrée](deploy-continuous-deployment.md) pour les conteneurs via le centre de déploiement. Accédez à votre application dans le [Portail Azure](https://portal.azure.com/) et sélectionnez **Centre de déploiement** sous **Déploiement**. Suivez les instructions pour sélectionner votre référentiel et votre branche. Cela permet de configurer un pipeline de build et de mise en production DevOps pour générer, baliser et déployer automatiquement votre conteneur lorsque de nouvelles validations sont envoyées (push) à la branche que vous avez sélectionnée.

### <a name="use-github-actions"></a>Utiliser GitHub Actions

Vous pouvez également automatiser le déploiement de vos conteneurs à l’aide de [GitHub Actions](deploy-container-github-action.md).  Le fichier de workflow ci-dessous génère et balise le conteneur avec l’ID de validation, l’envoie (push) à un registre de conteneurs et met à jour l’emplacement de site spécifié avec la nouvelle balise d’image.

```yaml
name: Build and deploy a container image to Azure Web Apps

on:
  push:
    branches:
    - <your-branch-name>

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
    - uses: actions/checkout@master

    -name: Authenticate using a Service Principal
      uses: azure/actions/login@v1
      with:
        creds: ${{ secrets.AZURE_SP }}

    - uses: azure/container-actions/docker-login@v1
      with:
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}

    - name: Build and push the image tagged with the git commit hash
      run: |
        docker build . -t contoso/demo:${{ github.sha }}
        docker push contoso/demo:${{ github.sha }}

    - name: Update image tag on the Azure Web App
      uses: azure/webapps-container-deploy@v1
      with:
        app-name: '<your-webapp-name>'
        slot-name: '<your-slot-name>'
        images: 'contoso/demo:${{ github.sha }}'
```

### <a name="use-other-automation-providers"></a>Utiliser d’autres fournisseurs d’automatisation

Les étapes répertoriées ci-dessus s’appliquent à d’autres utilitaires d’automatisation tels que CircleCI ou Travis CI. Toutefois, vous devez utiliser Azure CLI pour mettre à jour les emplacements de déploiement avec de nouvelles balises d’image lors de la dernière étape. Pour utiliser Azure CLI dans votre script d’automatisation, générez un principal de service à l’aide de la commande suivante.

```shell
az ad sp create-for-rbac --name "myServicePrincipal" --role contributor \
   --scopes /subscriptions/{subscription}/resourceGroups/{resource-group} \
   --sdk-auth
```

Dans votre script, connectez-vous en utilisant `az login --service-principal` et en fournissant les informations du principal. Vous pouvez ensuite utiliser `az webapp config container set` pour définir le nom du conteneur, la balise, l’URL du registre et le mot de passe du registre. Vous trouverez ci-dessous quelques liens utiles pour créer votre processus CI de conteneur.

- [Connexion à Azure CLI sur Circle CI](https://circleci.com/orbs/registry/orb/circleci/azure-cli) 

## <a name="language-specific-considerations"></a>Considérations spécifiques au langage

### <a name="java"></a>Java

Utilisez l’API [zipdeploy/](deploy-zip.md) de Kudu pour déployer des applications JAR, et [wardeploy/](deploy-zip.md#deploy-war-file) pour des applications WAR. Si vous utilisez Jenkins, vous pouvez utiliser ces API directement lors de votre phase de déploiement. Pour plus d’informations, consultez [cet article](/azure/developer/jenkins/deploy-to-azure-app-service-using-azure-cli).

### <a name="node"></a>Nœud

Par défaut, Kudu exécute les étapes de génération de votre application Node (`npm install`). Si vous utilisez un service de build tel qu’Azure DevOps, la build Kudu n’est pas nécessaire. Pour désactiver la build Kudu, créez un paramètre d’application `SCM_DO_BUILD_DURING_DEPLOYMENT`, avec une valeur de `false`.

### <a name="net"></a>.NET 

Par défaut, Kudu exécute les étapes de génération de votre application .NET (`dotnet build`). Si vous utilisez un service de build tel qu’Azure DevOps, la build Kudu n’est pas nécessaire. Pour désactiver la build Kudu, créez un paramètre d’application `SCM_DO_BUILD_DURING_DEPLOYMENT`, avec une valeur de `false`.

## <a name="other-deployment-considerations"></a>Autres points à prendre en considération pour le déploiement

### <a name="local-cache"></a>cache local

Le contenu Azure App Service est stocké sur Stockage Azure est exposé de manière durable en tant que partage de contenu. Toutefois, certaines applications ont uniquement besoin d’un magasin de contenu en lecture seule très performant à partir duquel elles peuvent s’exécuter avec une haute disponibilité. Ces applications peuvent tirer parti de l’utilisation du [cache local](overview-local-cache.md). Le cache local n’est pas recommandé pour les sites de gestion de contenu tels que WordPress.

Utilisez toujours le cache local conjointement avec les [emplacements de déploiement](deploy-staging-slots.md) pour éviter les temps d’arrêt. Consultez [cette section](overview-local-cache.md#best-practices-for-using-app-service-local-cache) pour plus d’informations sur l’utilisation conjointe de ces fonctionnalités.

### <a name="high-cpu-or-memory"></a>Utilisation de l’UC ou de mémoire élevée

Si votre plan App Service utilise plus de 90 % de l’UC ou de la mémoire disponible, la machine virtuelle sous-jacente risque de rencontrer des problèmes lors du traitement de votre déploiement. Dans ce cas, mettez temporairement à l’échelle le nombre d’instances pour effectuer le déploiement. Une fois le déploiement terminé, vous pouvez remettre le nombre d’instances à sa valeur précédente.

Pour plus d’informations sur les bonnes pratiques, consultez [Diagnostics App Service](./overview-diagnostics.md) pour connaître les bonnes pratiques applicables spécifiques à votre ressource.

- Accédez à votre application web dans le [portail Azure](https://portal.azure.com).
- Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes** pour ouvrir Diagnostics App Service.
- Choisissez la vignette de page d’accueil **Bonnes pratiques**.
- Cliquez sur **Bonnes pratiques pour la disponibilité et les performances** ou **Bonnes pratiques pour une configuration optimale** afin d’afficher l’état actuel de votre application en ce qui concerne ces bonnes pratiques.

Vous pouvez également utiliser ce lien pour ouvrir directement Diagnostics App Service pour votre ressource : `https://ms.portal.azure.com/?websitesextension_ext=asd.featurePath%3Ddetectors%2FParentAvailabilityAndPerformance#@microsoft.onmicrosoft.com/resource/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.Web/sites/{siteName}/troubleshoot`.
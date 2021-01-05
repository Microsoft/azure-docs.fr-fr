---
title: Gérer les mises à niveau de nœuds AKS à l’aide de GitHub Actions
titleSuffix: Azure Kubernetes Service
description: Découvrez comment mettre à jour des nœuds AKS à l’aide de GitHub Actions
services: container-service
ms.topic: article
ms.date: 11/27/2020
ms.openlocfilehash: 504c99a7305a5010f8dbe56a30b53d6fc3a13c5b
ms.sourcegitcommit: 4c89d9ea4b834d1963c4818a965eaaaa288194eb
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96607838"
---
# <a name="apply-security-updates-to-azure-kubernetes-service-aks-nodes-automatically-using-github-actions"></a>Appliquer automatiquement des correctifs de sécurité aux nœuds Azure Kubernetes service (AKS) à l’aide de GitHub Actions

Les correctifs de sécurité sont un élément essentiel de la gestion de la sécurité et de la conformité de votre cluster AKS à l’aide des derniers correctifs pour le système d’exploitation sous-jacent. Ces mises à jour incluent des correctifs de sécurité ou des mises à jour du noyau. Certaines mises à jour nécessitent un redémarrage du nœud pour terminer le processus.

L’exécution de `az aks upgrade` vous permet d’appliquer des mises à jour sans interruption de service. La commande gère l’application des dernières mises à jour à tous les nœuds de votre cluster, le bouclage et le drainage du trafic vers les nœuds, et le redémarrage des nœuds, puis l’autorisation du trafic vers les nœuds mis à jour. Si vous mettez à jour vos nœuds à l’aide d’une autre méthode, AKS ne redémarre pas automatiquement vos nœuds.

> [!NOTE]
> La principale différence entre l’utilisation ou non de `az aks upgrade` avec l’indicateur `--node-image-only` est que, lorsqu’il est utilisé, seules les images de nœud sont mises à niveau. En cas d’omission, les images de nœud et la version du plan de contrôle Kubernetes seront mises à niveau. Pour obtenir des informations plus approfondies, vous pouvez consulter la [documentation relative aux mises à niveau gérées sur les nœuds][managed-node-upgrades-article] et la [documentation relative aux mises à niveau de cluster][cluster-upgrades-article].

Tous les nœuds Kubernetes sont exécutés sur une machine virtuelle Azure standard. Ces machines virtuelles peuvent être basées sur Windows ou Linux. Les machines virtuelles Linux utilisent une image Ubuntu, avec le système d’exploitation configuré pour rechercher automatiquement les mises à jour toutes les nuits.

Lorsque vous utilisez la commande `az aks upgrade`, Azure CLI crée une augmentation des nouveaux nœuds avec les correctifs de sécurité et de noyau les plus récents. Ces nœuds sont initialement bouclés pour empêcher toute application d’y être planifiée jusqu’à ce que la mise à jour soit terminée. Une fois l’opération terminée, Azure boucle (rend le nœud indisponible pour la planification de nouvelles charges de travail) et draine (déplace les charges de travail existantes vers un autre nœud) les anciens nœuds et débloque les nouveaux, transférant de fait toutes les applications planifiées vers les nouveaux nœuds.

Ce processus est préférable à la mise à jour manuelle des noyaux basés sur Linux, car Linux nécessite un redémarrage lorsqu’une nouvelle mise à jour du noyau est installée. Si vous mettez à jour le système d’exploitation manuellement, vous devez également redémarrer la machine virtuelle et bloquer et drainer manuellement toutes les applications.

Cet article vous montre comment automatiser le processus de mise à jour des nœuds AKS. Vous allez utiliser GitHub Actions et Azure CLI pour créer une tâche de mise à jour basée sur `cron` qui s’exécute automatiquement.

## <a name="before-you-begin"></a>Avant de commencer

Cet article suppose que vous avez un cluster AKS existant. Si vous avez besoin d’un cluster AKS, consultez le guide de démarrage rapide d’AKS [avec Azure CLI][aks-quickstart-cli]ou avec le [Portail Azure][aks-quickstart-portal].

Azure CLI 2.0.59 (ou une version ultérieure) doit également être installé et configuré. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI][install-azure-cli].

Cet article suppose également que vous disposez d’un compte [GitHub][github] pour y créer vos actions.

## <a name="create-a-timed-github-action"></a>Créer une action GitHub chronométrée

`cron` est un utilitaire qui vous permet d’exécuter un ensemble de commandes ou un travail sur une planification automatisée. Pour créer un travail dans le but de mettre à jour vos nœuds AKS selon une planification automatisée, vous avez besoin d’un référentiel pour héberger vos actions. En règle générale, les actions GitHub sont configurées dans le même référentiel que votre application, mais vous pouvez utiliser n’importe quel référentiel. Pour cet article, nous allons utiliser votre [référentiel de profils][profile-repository]. Si vous n’en avez pas, créez un nouveau référentiel portant le même nom que votre nom d’utilisateur GitHub.

1. Accédez à votre référentiel sur GitHub.
1. Cliquez sur l’onglet **Actions** en haut de la page.
1. Si vous avez déjà configuré un workflow dans ce référentiel, vous êtes redirigé vers la liste des exécutions terminées. Dans le cas présent, cliquez sur le bouton **Nouveau workflow**. S’il s’agit de votre premier workflow dans le référentiel, GitHub vous présente quelques modèles de projet. Cliquez sur le lien **Configurer un workflow soi-même** sous le texte de description.
1. Modifiez les balises `name` et `on` du workflow comme indiqué ci-dessous. GitHub Actions utilise la même [syntaxe cron POSIX][cron-syntax] que n’importe quel système Linux. Dans cette planification, nous indiquons que le workflow doit s’exécuter tous les 15 jours à 3 h.

    ```yml
    name: Upgrade cluster node images
    on:
      schedule:
        - cron: '0 3 */15 * *'
    ```

1. Créez un nouveau travail à l’aide de ce qui suit. Ce travail est nommé `upgrade-node`, s’exécute sur un agent Ubuntu et se connecte à votre compte Azure CLI pour exécuter les étapes nécessaires à la mise à niveau des nœuds.

    ```yml
    name: Upgrade cluster node images

    on:
      schedule:
        - cron: '0 3 */15 * *'

    jobs:
      upgrade-node:
        runs-on: ubuntu-latest
    ```

## <a name="set-up-the-azure-cli-in-the-workflow"></a>Configurer Azure CLI dans le workflow

Dans la clé `steps`, vous allez définir tout le travail que le workflow exécutera pour mettre à niveau les nœuds.

Téléchargez Azure CLI, puis connectez-vous.

1. Sur le côté droit de l’écran GitHub Actions, recherchez la *barre de recherche du marketplace* et saisissez **« Connexion Azure »** .
1. Vous obtenez comme résultat une action appelée **Connexion Azure** publiée **par Azure** :

      :::image type="content" source="media/node-upgrade-github-actions/azure-login-search.png" alt-text="Résultats de la recherche montrant deux lignes, la première action est appelée « Connexion Azure » et la deuxième « Connexion Azure Container Registry ».":::

1. Cliquez sur **Connexion Azure**. Dans l’écran suivant, cliquez sur l’**icône de copie** en haut à droite de l’exemple de code.

    :::image type="content" source="media/node-upgrade-github-actions/azure-login.png" alt-text="Volet des résultats d’action de Connexion Azure avec l’exemple de code ci-dessous, le carré rouge autour d’une icône de copie met en évidence le point de clic":::

1. Collez le code suivant sous la clé `steps` :

      ```yml
      name: Upgrade cluster node images

      on:
        schedule:
          - cron: '0 3 */15 * *'

      jobs:
        upgrade-node:
          runs-on: ubuntu-latest

          steps:
            - name: Azure Login
              uses: Azure/login@v1.1
              with:
                creds: ${{ secrets.AZURE_CREDENTIALS }}
      ```

1. À partir d’Azure CLI, exécutez la commande suivante pour générer un nouveau nom d’utilisateur et un nouveau mot de passe.

    ```azurecli-interactive
    az ad sp create-for-rbac -o json
    ```

    La sortie doit ressembler au code JSON suivant :

    ```output
    {
      "appId": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "displayName": "azure-cli-xxxx-xx-xx-xx-xx-xx",
      "name": "http://azure-cli-xxxx-xx-xx-xx-xx-xx",
      "password": "xXxXxXxXx",
      "tenant": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
    }
    ```

1. **Dans une nouvelle fenêtre de navigateur**, accédez à votre référentiel GitHub et ouvrez l’onglet **Paramètres** du référentiel. Cliquez sur **Secrets** puis sur **Nouveau secret de référentiel**.
1. Pour *Nom*, utilisez `AZURE_CREDENTIALS`.
1.  Pour *Valeur*, ajoutez l’intégralité du contenu à partir de la sortie de l’étape précédente où vous avez créé un nom d’utilisateur et un mot de passe.

    :::image type="content" source="media/node-upgrade-github-actions/azure-credential-secret.png" alt-text="Formulaire montrant AZURE_CREDENTIALS comme titre secret et la sortie de la commande exécutée collée au format JSON":::

1. Cliquez sur **Ajouter un secret**.

L’interface CLI utilisée par votre action sera enregistrée dans votre compte Azure et prête à exécuter des commandes.

Pour créer les étapes nécessaires à l’exécution des commandes Azure CLI :

1. Accédez à la **page de recherche** sur le *marketplace GitHub* sur le côté droit de l’écran et recherchez *Action Azure CLI*. Choisissez *Action Azure CLI par Azure*.

    :::image type="content" source="media/node-upgrade-github-actions/azure-cli-action.png" alt-text="Résultat de la recherche pour « Action Azure CLI » avec le premier résultat montrant qu’elle a été créée par Azure":::

1. Cliquez sur le bouton de copie dans la *résultat du marketplace GitHub* et collez le contenu de l’action dans l’éditeur principal, sous l’étape *Connexion Azure*, similaire à ce qui suit :

    ```yml
    name: Upgrade cluster node images

    on:
      schedule:
        - cron: '0 3 */15 * *'

    jobs:
      upgrade-node:
        runs-on: ubuntu-latest

        steps:
          - name: Azure Login
            uses: Azure/login@v1.1
            with:
              creds: ${{ secrets.AZURE_CREDENTIALS }}
          - name: Upgrade node images
            uses: Azure/cli@v1.0.0
            with:
              inlineScript: az aks upgrade -g {resourceGroupName} -n {aksClusterName} --node-image-only
    ```

    > [!TIP]
    > Vous pouvez découpler les paramètres `-g` et `-n` de la commande en les ajoutant à des secrets similaires aux étapes précédentes. Remplacez les espaces réservés `{resourceGroupName}` et `{aksClusterName}` par leurs équivalents secrets, par exemple `${{secrets.RESOURCE_GROUP_NAME}}` et `${{secrets.AKS_CLUSTER_NAME}}`.

1. Renommez le fichier `upgrade-node-images`.
1. Cliquez sur **Démarrer la validation**, ajoutez un titre de message et enregistrez le workflow.

Une fois que vous avez créé la validation, le workflow est enregistré et prêt à être exécuté.

> [!NOTE]
> Pour mettre à niveau un seul pool de nœuds au lieu de tous les pools de nœuds sur le cluster, ajoutez le paramètre `--name` à la commande `az aks nodepool upgrade` pour spécifier le nom du pool de nœuds. Par exemple :
> ```azurecli-interactive
> az aks nodepool upgrade -g {resourceGroupName} --cluster-name {aksClusterName} --name {{nodePoolName}} --node-image-only
> ```

## <a name="run-the-github-action-manually"></a>Exécuter l’action GitHub manuellement

Vous pouvez exécuter le workflow manuellement, en plus de l’exécution planifiée, en ajoutant un nouveau déclencheur `on` appelé `workflow_dispatch`. Le fichier terminé doit ressembler au code YAML ci-dessous :

```yml
name: Upgrade cluster node images

on:
  schedule:
    - cron: '0 3 */15 * *'
  workflow_dispatch:

jobs:
  upgrade-node:
    runs-on: ubuntu-latest

    steps:
      - name: Azure Login
        uses: Azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      # Code for upgrading one or more node pools
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les images de nœud les plus récentes, consultez les [notes de publication AKS](https://github.com/Azure/AKS/releases).
- Apprenez à mettre à niveau la version de Kubernetes avec la section [Mettre à niveau un cluster AKS][cluster-upgrades-article].
- En savoir plus sur les pools de nœuds multiples et sur la mise à niveau des pools de nœuds avec la section [Créer et gérer les pools de nœuds multiples][use-multiple-node-pools].
- Apprenez-en davantage sur les [pools de nœuds système][system-pools].
- Pour savoir comment réduire les coûts à l’aide d’instances Spot, consultez [Ajouter un pool de nœuds spot à AKS][spot-pools].

<!-- LINKS - external -->
[github]: https://github.com
[profile-repository]: https://docs.github.com/en/free-pro-team@latest/github/setting-up-and-managing-your-github-profile/about-your-profile
[cron-syntax]: https://pubs.opengroup.org/onlinepubs/9699919799/utilities/crontab.html#tag_20_25_07

<!-- LINKS - internal -->
[aks-quickstart-cli]: kubernetes-walkthrough.md
[aks-quickstart-portal]: kubernetes-walkthrough-portal.md
[install-azure-cli]: /cli/azure/install-azure-cli
[managed-node-upgrades-article]: node-image-upgrade.md
[cluster-upgrades-article]: upgrade-cluster.md
[system-pools]: use-system-pools.md
[spot-pools]: spot-node-pool.md
[use-multiple-node-pools]: use-multiple-node-pools.md

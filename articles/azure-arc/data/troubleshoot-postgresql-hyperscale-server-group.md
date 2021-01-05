---
title: Résoudre des problèmes de groupes de serveurs PostgreSQL Hyperscale
description: Résoudre des problèmes de groupes de serveurs PostgreSQL Hyperscale avec un Jupyter Notebook
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: caaab07200a8631935a2b5d5368a0c16ea9a60c5
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92320213"
---
# <a name="troubleshooting-postgresql-hyperscale-server-groups"></a>Résolution des problèmes des groupes de serveurs PostgreSQL Hyperscale
Cet article décrit quelques techniques permettant de résoudre les problèmes d’un groupe de serveurs. En plus de cet article, vous pouvez apprendre à utiliser [Kibana](monitor-grafana-kibana.md) pour effectuer des recherches dans les journaux ou [Grafana](monitor-grafana-kibana.md) pour visualiser les métriques relatives à votre groupe de serveurs. 

## <a name="getting-more-details-about-the-execution-of-an-azdata-command"></a>Récupération d’informations supplémentaires sur l’exécution d’une commande azdata
Le paramètre **--debug** peut être ajouté à n’importe quelle commande azdata. Il permet d’afficher sur la console des informations supplémentaires sur l’exécution de cette commande, pour mieux comprendre son comportement.
Par exemple, vous pouvez exécuter :
```console
azdata arc postgres server create -n postgres01 -w 2 --debug
```

or
```console
azdata arc postgres server edit -n postgres01 --extension SomeExtensionName --debug
```

Par ailleurs, vous pouvez utiliser le paramètre --help sur n’importe quelle commande azdata pour afficher de l’aide ou la liste des paramètres d’une commande spécifique. Par exemple :
```console
azdata arc postgres server create --help
```


## <a name="collecting-logs-of-the-data-controller-and-your-server-groups"></a>Collecte des journaux du contrôleur de données et des groupes de serveurs
Lisez l’article [Récupération des journaux pour les services de données compatibles avec Azure Arc](troubleshooting-get-logs.md).



## <a name="interactive-troubleshooting-with-jupyter-notebooks-in-azure-data-studio"></a>Résolution interactive des problèmes liés aux notebooks Jupyter dans Azure Data Studio
Les blocs-notes peuvent documenter des procédures en incluant un contenu Markdown pour décrire la procédure à suivre. Ils peuvent également fournir du code exécutable pour automatiser une procédure.  Ce modèle est utile pour tout, des procédures d’exploitation standard aux guides de dépannage.

Par exemple, résolvons les problèmes liés à un groupe de serveurs PostgreSQL Hyperscale, qui peuvent survenir lors de l’utilisation d’Azure Data Studio.

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

### <a name="install-tools"></a>Installer des outils

Installez Azure Data Studio, `kubectl` et [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)], sur l’ordinateur client que vous utilisez pour exécuter le bloc-notes dans Azure Data Studio. Pour ce faire, suivez les instructions fournies dans [Installer les outils clients](install-client-tools.md)

### <a name="update-the-path-environment-variable"></a>Mettre à jour la variable d’environnement PATH

Assurez-vous que ces outils peuvent être appelés à partir de n’importe quel emplacement sur cet ordinateur client. Par exemple, sur un ordinateur client Windows, mettez à jour la variable d’environnement système PATH et ajoutez le dossier dans lequel vous avez installé kubectl.

### <a name="sign-in-with-azure-data-cli-azdata"></a>Connectez-vous avec [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]

Connectez-vous à votre contrôleur de données Arc à partir de cet ordinateur client et avant de lancer Azure Data Studio. Pour ce faire, exécutez une commande telle que :

```console
azdata login --endpoint https://<IP address>:<port>
```

Remplacez `<IP address>` par l’adresse IP de votre cluster Kubernetes, et `<port>` par le port que Kubernetes écoute. Vous êtes invité à entrer un nom d’utilisateur et un mot de passe. Pour plus de détails, exécutez :_

```console
azdata login --help
```

### <a name="log-into-your-kubernetes-cluster-with-kubectl"></a>Connectez-vous à votre cluster Kubernetes avec kubectl

Pour ce faire, vous pouvez utiliser les exemples de commandes fournis dans [ce billet de blog](https://blog.christianposta.com/kubernetes/logging-into-a-kubernetes-cluster-with-kubectl/).
Vous devez exécuter des commandes telles que :

```console
kubectl config view
kubectl config set-credentials kubeuser/my_kubeuser --username=<your Arc Data Controller Admin user name> --password=<password>
kubectl config set-cluster my_kubeuser --server=https://<IP address>:<port>
kubectl config set-context default/my_kubeuser/ArcDataControllerAdmin --user=ArcDataControllerAdmin/my_kubeuser --namespace=arc --cluster=my_kubeuser
kubectl config use-context default/my_kubeuser/ArcDataControllerAdmin
```

#### <a name="the-troubleshooting-notebook"></a>Bloc-notes de résolution des problèmes

Lancez Azure Data Studio et ouvrez le bloc-notes de résolution des problèmes. 

Implémentez les étapes décrites dans [033-manage-Postgres-with-AzureDataStudio.md](manage-postgresql-hyperscale-server-group-with-azure-data-studio.md) pour :

1. Vous connecter à votre contrôleur de données Arc
2. Cliquez avec le bouton droit sur votre instance Postgres, puis choisissez **[Gérer]**
3. Sélectionnez le **tableau de bord [Diagnostiquer et résoudre les problèmes]**
4. Sélectionnez le **lien [Résoudre des problèmes]**

:::image type="content" source="media/postgres-hyperscale/ads-controller-postgres-troubleshooting-notebook.jpg" alt-text="Azure Data Studio – Ouvrir le bloc-notes de résolution des problèmes PostgreSQL":::

La fenêtre **TSG100 – Bloc-notes de l’outil de résolution des problèmes de PostgreSQL Hyperscale activé avec par Azure Arc** s’ouvre : :::image type="content" source="media/postgres-hyperscale/ads-controller-postgres-troubleshooting-notebook2.jpg" alt-text="Azure Data Studio – Ouvrir le bloc-notes de résolution des problèmes PostgreSQL":::

#### <a name="run-the-scripts"></a>Exécuter les scripts
Sélectionnez le bouton « Exécuter tout » en haut pour exécuter tout le bloc-notes en une seule fois, ou parcourez et exécutez chaque cellule de code une par une.

Affichez la sortie de l’exécution des cellules de code pour tout problème potentiel.

Nous ajouterons des détails au bloc-notes au fil du temps afin d’identifier les problèmes courants et la manière de les résoudre.

## <a name="next-step"></a>Étape suivante
- En savoir plus sur l’[obtention de journaux pour les services de données activés par Azure Arc](troubleshooting-get-logs.md)
- En savoir plus sur la [recherche dans les journaux avec Kibana](monitor-grafana-kibana.md)
- En savoir plus sur la [surveillance avec Grafana](monitor-grafana-kibana.md)
- Créer vos propres blocs-notes

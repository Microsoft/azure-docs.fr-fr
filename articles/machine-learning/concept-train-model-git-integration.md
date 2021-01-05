---
title: Intégration de Git pour Azure Machine Learning
titleSuffix: Azure Machine Learning
description: Découvrez la façon dont Azure Machine Learning s’intègre à un référentiel Git local pour suivre les informations relatives au référentiel, aux branches et aux validations actuelles dans le cadre d’une exécution de formation.
services: machine-learning
ms.service: machine-learning
ms.subservice: core
ms.topic: conceptual
ms.author: jordane
author: jpe316
ms.date: 11/16/2020
ms.openlocfilehash: 989fc7cb66cf5381d174a3aad12f84f5b055aab8
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94701634"
---
# <a name="git-integration-for-azure-machine-learning"></a>Intégration de Git pour Azure Machine Learning

[Git](https://git-scm.com/) est un système de gestion de versions connu qui vous permet de partager vos projets et de collaborer. 

Azure Machine Learning prend entièrement en charge les dépôts Git pour le suivi du travail. Vous pouvez cloner des dépôts directement dans le système de fichiers de votre espace de travail partagé, ou utiliser Git sur votre station de travail locale ou à partir d’un pipeline CI/CD.

Lors de la soumission d’un travail à Azure Machine Learning, si les fichiers sources sont stockés dans un dépôt Git local, les informations sur le dépôt font l’objet d’un suivi dans le cadre du processus d’entraînement.

Étant donné qu’Azure Machine Learning effectue le suivi des informations à partir d’un dépôt Git local, il n’est lié à aucun dépôt central spécifique. Votre dépôt peut être cloné à partir de GitHub, GitLab, Bitbucket, Azure DevOps ou de tout autre service compatible Git.

## <a name="clone-git-repositories-into-your-workspace-file-system"></a>Cloner des dépôts Git dans le système de fichiers de votre espace de travail
Azure Machine Learning fournit un système de fichiers partagé pour tous les utilisateurs de l’espace de travail.
Pour cloner un dépôt Git dans ce partage de fichiers, nous vous recommandons de créer une instance de calcul et d’ouvrir un terminal.
Une fois le terminal ouvert, vous avez accès à un client Git complet et pouvez cloner et utiliser Git par le biais de l’interface CLI Git.

Nous vous recommandons de cloner le dépôt dans le répertoire de vos utilisateurs afin que d’autres ne puissent pas créer de collisions directement sur votre branche de travail.

Vous pouvez cloner n’importe quel dépôt Git auprès duquel vous pouvez vous authentifier (GitHub, Azure Repos, BitBucket, etc.)

Pour plus d’informations sur le clonage, consultez le guide d’[utilisation de l’interface CLI Git](https://guides.github.com/introduction/git-handbook/).

## <a name="authenticate-your-git-account-with-ssh"></a>Authentifier votre compte Git avec SSH
### <a name="generate-a-new-ssh-key"></a>Générer une nouvelle clé SSH
1) [Ouvrez la fenêtre de terminal](./how-to-run-jupyter-notebooks.md#terminal) sous l’onglet Notebook Azure Machine Learning.

2) Collez le texte ci-dessous, en substituant votre adresse e-mail.

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

Cette opération crée une clé SSH, qui utilise l’e-mail fourni en tant qu’étiquette.

```
> Generating public/private rsa key pair.
```

3) Quand vous êtes invité à entrer un fichier dans lequel enregistrer la clé, appuyez sur Entrée. Cette opération accepte l’emplacement de fichier par défaut.

4) Vérifiez que l’emplacement par défaut est « /home/azureuser/.ssh » et appuyez sur Entrée. Sinon, spécifiez l’emplacement « /home/azureuser/.ssh ».

> [!TIP]
> Assurez-vous que la clé SSH est enregistrée dans « /home/azureuser/.ssh ». Ce fichier est enregistré sur l’instance de calcul et n’est accessible que par le propriétaire de cette instance.

```
> Enter a file in which to save the key (/home/azureuser/.ssh/id_rsa): [Press enter]
```

5) À l’invite, tapez une phrase secrète sécurisée. Nous vous recommandons d’ajouter une phrase secrète à votre clé SSH pour renforcer la sécurité.

```
> Enter passphrase (empty for no passphrase): [Type a passphrase]
> Enter same passphrase again: [Type passphrase again]
```

### <a name="add-the-public-key-to-git-account"></a>Ajouter la clé publique au compte Git
1) Dans votre fenêtre de terminal, copiez le contenu du fichier de la clé publique. Si vous avez renommé la clé, remplacez id_rsa.pub par le nom de fichier de la clé publique.

```bash
cat ~/.ssh/id_rsa.pub
```
> [!TIP]
> **Opérations Copier et Coller dans le terminal**
> * Windows : `Ctrl-Insert` pour copier et `Ctrl-Shift-v` ou `Shift-Insert` pour coller.
> * Mac OS : `Cmd-c` pour copier et `Cmd-v` pour coller.
> * FireFox/IE peuvent ne pas prendre en charge correctement les autorisations de Presse-papiers.

2) Sélectionnez la sortie de la clé et copiez-la dans le Presse-papiers.

+ [GitHub](https://docs.github.com/github/authenticating-to-github/adding-a-new-ssh-key-to-your-github-account)

+ [GitLab](https://docs.gitlab.com/ee/ssh/#adding-an-ssh-key-to-your-gitlab-account)

+ [Azure DevOps](/azure/devops/repos/git/use-ssh-keys-to-authenticate?view=azure-devops#step-2--add-the-public-key-to-azure-devops-servicestfs) : Démarrez à l’**étape 2**.

+ [BitBucket](https://support.atlassian.com/bitbucket-cloud/docs/set-up-an-ssh-key/#SetupanSSHkey-ssh2). Démarrez à **étape 4**.

### <a name="clone-the-git-repository-with-ssh"></a>Cloner le dépôt Git avec SSH

1) Copiez l’URL du clonage Git SSH à partir du dépôt Git.

2) Collez l’URL dans la commande `git clone` ci-dessous, pour utiliser votre URL du dépôt Git SSH. Cela ressemble à ce qui suit :

```bash
git clone git@example.com:GitUser/azureml-example.git
Cloning into 'azureml-example'...
```

Une réponse semblable à la suivante s’affiche :

```bash
The authenticity of host 'example.com (192.30.255.112)' can't be established.
RSA key fingerprint is SHA256:nThbg6kXUpJWGl7E1IGOCspRomTxdCARLviKw6E5SY8.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'github.com,192.30.255.112' (RSA) to the list of known hosts.
```

SSH peut afficher l’empreinte digitale SSH du serveur et vous demander de la vérifier. Vous devez vérifier que l’empreinte digitale affichée correspond à l’une des empreintes digitales de la page de clés publiques SSH.

SSH affiche cette empreinte digitale quand il se connecte à un hôte inconnu pour vous protéger contre les [attaques de l’intercepteur](/previous-versions/windows/it-pro/windows-2000-server/cc959354(v=technet.10)). Une fois que vous avez accepté l’empreinte digitale de l’hôte, SSH ne vous sollicite que si l’empreinte change.

3) Quand vous êtes invité à indiquer si vous souhaitez poursuivre la connexion, tapez `yes`. Git clone le dépôt et configure le dépôt distant d’origine afin qu’il se connecte avec SSH pour les prochaines commandes Git.

## <a name="track-code-that-comes-from-git-repositories"></a>Suivre le code provenant de dépôts Git

Lorsque vous soumettez une exécution d’entraînement à partir du kit SDK Python ou de l’interface CLI de Machine Learning, les fichiers nécessaires à l’entraînement du modèle sont chargés dans votre espace de travail. Si la commande `git` est disponible dans votre environnement de développement, le processus de chargement l’utilise pour vérifier si les fichiers sont stockés dans un dépôt Git. Si tel est le cas, les informations de votre dépôt Git sont également chargées lors de l’exécution d’entraînement. Ces informations sont stockées dans les propriétés suivantes pour l’exécution d’entraînement :

| Propriété | Commande Git utilisée pour récupérer la valeur | Description |
| ----- | ----- | ----- |
| `azureml.git.repository_uri` | `git ls-remote --get-url` | URI à partir duquel votre dépôt a été cloné. |
| `mlflow.source.git.repoURL` | `git ls-remote --get-url` | URI à partir duquel votre dépôt a été cloné. |
| `azureml.git.branch` | `git symbolic-ref --short HEAD` | Branche active lorsque l’exécution a été envoyée. |
| `mlflow.source.git.branch` | `git symbolic-ref --short HEAD` | Branche active lorsque l’exécution a été envoyée. |
| `azureml.git.commit` | `git rev-parse HEAD` | Hachage de validation du code qui a été envoyé pour l’exécution. |
| `mlflow.source.git.commit` | `git rev-parse HEAD` | Hachage de validation du code qui a été envoyé pour l’exécution. |
| `azureml.git.dirty` | `git status --porcelain .` | `True`, si l’intégrité de la validation/branche est compromise ; sinon, `false`. |

Ces informations sont envoyées pour les exécutions qui utilisent un estimateur, un pipeline Machine Learning ou une exécution de script.

Si vos fichiers d’entraînement ne se trouvent pas dans un dépôt Git dans votre environnement de développement, ou si la commande `git` n’est pas disponible, aucune information liée à Git ne fait l’objet d’un suivi.

> [!TIP]
> Pour vérifier si la commande git est disponible dans votre environnement de développement, ouvrez une session de l’interpréteur de commandes, une invite de commandes, PowerShell ou une autre interface de ligne de commande, puis saisissez la commande suivante :
>
> ```
> git --version
> ```
>
> Si elle est installée et dans le chemin d’accès, vous recevez une réponse similaire à `git version 2.4.1`. Pour plus d’informations sur l’installation de git dans votre environnement de développement, consultez le [site de Git](https://git-scm.com/).

## <a name="view-the-logged-information"></a>Voir les informations journalisées

Les informations Git sont stockées dans les propriétés d’une exécution d’entraînement. Vous pouvez voir ces informations en utilisant le portail Azure, le kit SDK Python et l’interface CLI. 

### <a name="azure-portal"></a>Portail Azure

1. Dans le [portail Studio](https://ml.azure.com), sélectionnez votre espace de travail.
1. Sélectionnez __Expériences__, puis sélectionnez l’une de vos expériences.
1. Sélectionnez l’une des exécutions dans la colonne __NUMÉRO D’EXÉCUTION__.
1. Sélectionnez __Sorties + Journaux__, puis développez les entrées __logs__ (journaux) et __azureml__. Sélectionnez le lien qui commence par __###\_azure__.

Les informations journalisées contiennent du texte similaire au code JSON suivant :

```json
"properties": {
    "_azureml.ComputeTargetType": "batchai",
    "ContentSnapshotId": "5ca66406-cbac-4d7d-bc95-f5a51dd3e57e",
    "azureml.git.repository_uri": "git@github.com:azure/machinelearningnotebooks",
    "mlflow.source.git.repoURL": "git@github.com:azure/machinelearningnotebooks",
    "azureml.git.branch": "master",
    "mlflow.source.git.branch": "master",
    "azureml.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "mlflow.source.git.commit": "4d2b93784676893f8e346d5f0b9fb894a9cf0742",
    "azureml.git.dirty": "True",
    "AzureML.DerivedImageName": "azureml/azureml_9d3568242c6bfef9631879915768deaf",
    "ProcessInfoFile": "azureml-logs/process_info.json",
    "ProcessStatusFile": "azureml-logs/process_status.json"
}
```

### <a name="python-sdk"></a>Kit de développement logiciel (SDK) Python

Après l’envoi d’une exécution d’entraînement, un objet [Run](/python/api/azureml-core/azureml.core.run%28class%29?preserve-view=true&view=azure-ml-py) est retourné. L’attribut `properties` de cet objet contient les informations Git journalisées. Par exemple, le code suivant récupère le hachage de validation :

```python
run.properties['azureml.git.commit']
```

### <a name="cli"></a>Interface de ligne de commande

Vous pouvez utiliser la commande CLI `az ml run` pour récupérer les propriétés d’une exécution. Par exemple, la commande suivante retourne les propriétés de la dernière exécution dans l’expérience nommée `train-on-amlcompute` :

```azurecli-interactive
az ml run list -e train-on-amlcompute --last 1 -w myworkspace -g myresourcegroup --query '[].properties'
```

Pour plus d’informations, consultez la documentation de référence [az ml run](/cli/azure/ext/azure-cli-ml/ml/run?preserve-view=true&view=azure-cli-latest).

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser des cibles de calcul pour la formation des modèles](how-to-set-up-training-targets.md)
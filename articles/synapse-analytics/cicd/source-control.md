---
title: Contrôle de code source dans Synapse Studio
description: Découvrez comment configurer le contrôle de code source dans Azure Synapse Studio
services: synapse-analytics
author: liud
ms.service: synapse-analytics
ms.topic: conceptual
ms.date: 11/20/2020
ms.author: liud
ms.reviewer: pimorano
ms.openlocfilehash: b6cadbf5c3a33c1a954a47f37b33ad8703f40b69
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350736"
---
# <a name="source-control-in-azure-synapse-studio"></a>Contrôle de code source dans Azure Synapse Studio

Par défaut, Azure Synapse Studio crée directement par rapport au service Synapse. Mais cette expérience présente les limites suivantes :

- Synapse Studio n’inclut pas de stockage temporaire pour le stockage de vos modifications. La seule façon d’enregistrer et de partager les modifications consiste à utiliser l’option **Publier** pour publier toutes les modifications directement vers le service Synapse.

- Synapse Studio n’est pas optimisé pour la collaboration ou le contrôle de version.

Pour le contrôle de code source, Synapse Studio vous permet d'associer votre espace de travail à un référentiel Git, Azure DevOps ou GitHub. Cet article explique comment configurer et utiliser un espace de travail Synapse avec le référentiel Git activé. Nous mettons également en évidence quelques bonnes pratiques et un guide de dépannage.

> [!NOTE]
> L’intégration Git d’Azure Synapse Studio n’est pas disponible dans le cloud Azure Government.

## <a name="configure-git-repository-in-synapse-studio"></a>Configurer le référentiel Git dans Synapse Studio 

Après avoir lancé Synapse Studio, vous pouvez configurer un référentiel Git dans votre espace de travail. Un espace de travail Synapse Studio ne peut être associé qu’à un seul référentiel Git à la fois. 

### <a name="configuration-method-1-global-bar"></a>Méthode de configuration 1 : barre globale

Dans la barre globale de Synapse Studio, sélectionnez le menu déroulant **Synapse Live** , puis **Configurer un référentiel de code**.

![Configurer les paramètres du référentiel de code à partir de la création](media/configure-repo-1.png)

### <a name="configuration-method-2-manage-hub"></a>Méthode de configuration 2 : Gérer le hub

Accédez au hub de gestion de Synapse Studio. Sélectionnez **Configuration Git** dans la section **Contrôle de code source**. Si vous n’avez pas de référentiel connecté, cliquez sur **Configure** (Configurer).

![Configurer les paramètres du référentiel de code à partir du hub de gestion](media/configure-repo-2.png)

> [!NOTE]
> Les utilisateurs autorisés en tant que contributeur d’espace de travail, propriétaire ou rôle de niveau supérieur peuvent configurer, modifier le paramètre et déconnecter le référentiel Git dans Azure Synapse Studio 

Vous pouvez connecter le référentiel Git Azure DevOps ou GitHub dans votre espace de travail.

## <a name="connect-with-azure-devops-git"></a>Se connecter au référentiel Git Azure DevOps 

Vous pouvez associer un espace de travail Synapse à un référentiel Azure DevOps pour le contrôle de code source, la collaboration, la gestion des versions, etc. Si vous n’avez pas de référentiel Azure DevOps, suivez [ces instructions](/azure/devops/organizations/accounts/create-organization-msa-or-work-student) pour créer vos ressources de référentiel.

### <a name="azure-devops-git-repository-settings"></a>Paramètres du référentiel Git Azure DevOps

Quand vous vous connectez à votre référentiel Git, commencez par sélectionner Azure DevOps git comme type de référentiel, choisissez un locataire Azure AD dans la liste déroulante, puis cliquez sur **Continuer**.

![Configurer les paramètres du référentiel de code](media/connect-with-azuredevops-repo-selected.png)

Le volet de configuration affiche les paramètres Azure DevOps git suivants :

| Paramètre | Description | Valeur |
|:--- |:--- |:--- |
| **Type de référentiel** | Type du dépôt de code Azure Repos.<br/> | Azure DevOps Git ou GitHub |
| **Azure Active Directory** | Le nom de votre abonné Azure AD. | `<your tenant name>` |
| **Compte Azure DevOps** | Nom de votre organisation Azure Repos. Vous trouverez le nom de votre organisation Azure Repos sur `https://{organization name}.visualstudio.com`. Vous pouvez vous [connecter à votre organisation Azure Repos](https://www.visualstudio.com/team-services/git/) pour accéder à votre profil Visual Studio et visualiser vos dépôts et projets. | `<your organization name>` |
| **Nom du projet** | Nom de votre projet Azure Repos. Vous trouverez le nom de votre projet Azure Repos sur `https://{organization name}.visualstudio.com/{project name}`. | `<your Azure Repos project name>` |
| **RepositoryName** | Nom de votre dépôt de code Azure Repos. Les projets Azure Repos contiennent des dépôts Git pour gérer votre code source à mesure que votre projet se développe. Vous pouvez créer un nouveau référentiel ou utiliser un référentiel existant déjà présent dans le projet. | `<your Azure Repos code repository name>` |
| **Branche de collaboration** | Votre branche de collaboration Azure Repos utilisée pour la publication. Par défaut, il s’agit de `master`. Modifiez ce paramètre au cas où vous souhaitez publier des ressources à partir d’une autre branche. Vous pouvez sélectionner des branches existantes ou en créer de nouvelles. | `<your collaboration branch name>` |
| **Dossier racine** | Votre dossier racine de votre branche de collaboration Azure Repos. | `<your root folder name>` |
| **Import existing resources to repository** (Importer des ressources existantes dans le référentiel) | Indique s’il convient d’importer des ressources existantes à partir de Synapse Studio dans un référentiel Azure Repos Git. Cochez la case pour importer vos ressources d’espace de travail (à l’exception des pools) dans le référentiel Git associé au format JSON. Cette action exporte chaque ressource individuellement. Lorsque cette case n’est pas cochée, les ressources existantes ne sont pas importées. | Cochée (par défaut) |
| **Importer la ressource dans cette branche** | Sélectionnez la branche dans laquelle les ressources (script SQL, notebook, définition de travail Spark, jeu de données, flux de données, etc.) sont importées. 

Vous pouvez également utiliser le lien du référentiel pour pointer rapidement vers le référentiel git auquel vous souhaitez vous connecter. 

### <a name="use-a-different-azure-active-directory-tenant"></a>Utiliser un autre locataire Azure Active Directory

Le dépôt Git Azure Repos peut se trouver dans un autre locataire Azure Active Directory. Pour définir un autre locataire Azure AD, vous devez disposer des droits d’administrateur pour l’abonnement Azure que vous utilisez. Pour plus d’informations, consultez la rubrique [Modifier l’administrateur d’abonnement](https://docs.microsoft.com/azure/cost-management-billing/manage/add-change-subscription-administrator#assign-a-subscription-administrator).

> [!IMPORTANT]
> Pour se connecter à un autre annuaire Azure Active Directory, l’utilisateur connecté doit être membre d’Active Directory. 

### <a name="use-your-personal-microsoft-account"></a>Ajouter votre compte Microsoft personnel

Pour utiliser un compte Microsoft personnel à des fins d'intégration de Git, vous pouvez lier votre référentiel Azure personnel au répertoire Active Directory de votre organisation.

1. Ajoutez votre compte Microsoft personnel au répertoire Active Directory de votre organisation en tant qu’invité. Pour plus d'informations, voir [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](https://docs.microsoft.com/azure/active-directory/external-identities/add-users-administrator).

2. Connectez-vous au portail Azure avec votre compte Microsoft personnel. Basculez ensuite vers le répertoire Active Directory de votre organisation.

3. Accédez à la section Azure DevOps qui affiche désormais votre référentiel personnel. Sélectionnez le référentiel et connectez-vous à Active Directory.

Au terme de cette procédure de configuration, votre référentiel personnel est disponible lorsque vous configurez l’intégration de Git dans Synapse Studio.

Pour plus d’informations sur la connexion de référentiels Azure au répertoire Active Directory de votre organisation, voir [Connecter votre organisation à Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad).

## <a name="connect-with-github"></a>Se connecter avec GitHub 

 Vous pouvez associer un espace de travail à un référentiel GitHub pour le contrôle du code source, la collaboration et la gestion des versions. Si vous n’avez pas un compte ou un référentiel GitHub, suivez [ces instructions](https://github.com/join) pour créer vos ressources.

L’intégration de GitHub à Synapse Studio prend à la fois en charge le service GitHub public ([https://github.com](https://github.com)) que GitHub Enterprise. Vous pouvez utiliser des référentiels GitHub publics et privés à condition de disposer des autorisations de lecture et d'écriture pour ceux-ci dans GitHub.

### <a name="github-settings"></a>Paramètres GitHub

Quand vous vous connectez à votre référentiel git, commencez par sélectionner GitHub comme type de référentiel, puis indiquez votre compte GitHub ou votre URL GitHub Enterprise Server si vous utilisez GitHub Enterprise Server, puis cliquez sur **Continuer**.

![Paramètres du référentiel GitHub](media/connect-with-github-repo-1.png)

Le volet de configuration affiche les paramètres du dépôt GitHub suivants :

| **Paramètre** | **Description**  | **Valeur**  |
|:--- |:--- |:--- |
| **Type de référentiel** | Type du dépôt de code Azure Repos. | GitHub |
| **Utiliser GitHub Enterprise** | Cochez la case pour sélectionner GitHub Enterprise | non sélectionné (par défaut) |
| **URL GitHub Enterprise** | URL racine de GitHub Enterprise (doit être HTTPS pour un serveur local GitHub Enterprise). Par exemple : `https://github.mydomain.com`. Obligatoire uniquement si l’option **Utiliser GitHub Enterprise** est sélectionnée | `<your GitHub enterprise url>` |                                                           
| **Compte GitHub** | Le nom de votre compte GitHub. Vous trouverez ce nom dans https:\//github.com/{nom du compte}/{nom du référentiel}. En naviguant sur cette page, vous êtes invité à entrer les informations d’identification GitHub OAuth sur votre compte GitHub. | `<your GitHub account name>` |
| **Nom du dépôt**  | Le nom de votre référentiel de code GitHub. Les comptes GitHub contiennent des référentiels Git pour gérer votre code source. Vous pouvez créer un nouveau référentiel ou utiliser un référentiel existant déjà présent dans le compte. | `<your repository name>` |
| **Branche de collaboration** | Votre branche de collaboration GitHub utilisée pour la publication. Par défaut, il s’agit de la branche principale. Modifiez ce paramètre au cas où vous souhaitez publier des ressources à partir d’une autre branche. | `<your collaboration branch>` |
| **Dossier racine** | Votre dossier racine de votre branche de collaboration GitHub. |`<your root folder name>` |
| **Import existing resources to repository** (Importer des ressources existantes dans le référentiel) | Indique s’il convient d’importer des ressources existantes à partir de Synapse Studio dans un référentiel Git. Cochez la case pour importer vos ressources d’espace de travail (à l’exception des pools) dans le référentiel Git associé au format JSON. Cette action exporte chaque ressource individuellement. Lorsque cette case n’est pas cochée, les ressources existantes ne sont pas importées. | Activée (par défaut) |
| **Importer la ressource dans cette branche** | Sélectionnez la branche dans laquelle les ressources (script SQL, notebook, définition de travail Spark, jeu de données, flux de données, etc.) sont importées. 

### <a name="github-organizations"></a>Organisations GitHub

La connexion à une organisation GitHub nécessite que l’organisation accorde l’autorisation à Synapse Studio. Un utilisateur disposant d’autorisations d’administrateur sur l’organisation doit effectuer les étapes ci-dessous.

#### <a name="connecting-to-github-for-the-first-time"></a>Première connexion à GitHub

Si vous vous connectez à GitHub à partir de Synapse Studio pour la première fois, procédez comme suit pour vous connecter à une organisation GitHub.

1. Dans le volet Configuration git, entrez le nom de l’organisation dans le champ *Compte GitHub*. Une invite de connexion à GitHub s’affiche. 

1. Connectez-vous à l’aide des informations d’identification de l’utilisateur.

1. Vous êtes invité à autoriser Synapse en tant qu’application appelée *Azure Synapse*. Dans cet écran, une option vous permet d’accorder à Synapse l’autorisation d’accéder à l’organisation. Si vous ne voyez pas l’option permettant d’accorder une autorisation, demandez à un administrateur d’accorder manuellement l’autorisation par le biais de GitHub.

Une fois ces étapes effectuées, votre espace de travail est en mesure de se connecter aux référentiels publics et privés au sein de votre organisation. Si vous ne parvenez pas à vous connecter, essayez de vider le cache du navigateur et de réessayer.

## <a name="version-control"></a>Gestion de versions

Les systèmes de contrôle de version (également appelé _contrôle du code source_) permettent aux développeurs de collaborer sur le code et de suivre les modifications. Le contrôle du code source est un outil essentiel pour les projets impliquant plusieurs développeurs.

### <a name="creating-feature-branches"></a>Création de branches de fonctionnalités

Chaque référentiel Git associé à Synapse Studio comporte une branche de collaboration. (`main` ou `master` est la branche de collaboration par défaut). Les utilisateurs peuvent également créer des branches de fonctionnalités en cliquant sur **+ Nouvelle branche** dans la liste déroulante des branches. Une fois le volet de la nouvelle branche affiché, entrez le nom de votre branche de fonctionnalités.

![Créer une branche](media/create-new-branch.png)

Lorsque vous êtes prêt à fusionner les modifications de votre branche de fonctionnalités dans votre branche de collaboration, cliquez sur la liste déroulante des branches et sélectionnez **Créer la demande de tirage (pull request)** . Cette action vous dirigera vers un fournisseur Git où vous pouvez augmenter les demandes de tirage, procéder à des révisions du code et fusionner les modifications dans votre branche de collaboration. Vous êtes uniquement autorisé à publier sur le service Synapse à partir de votre branche de collaboration. 

![Créer une nouvelle demande de tirage (pull request)](media/create-pull-request.png)

### <a name="configure-publishing-settings"></a>Configurer les paramètres de publication

Par défaut, Synapse Studio génère les modèles d’espace de travail et les enregistre dans une branche appelée `workspace_publish`. Pour configurer une branche de publication personnalisée, ajoutez un fichier `publish_config.json` au dossier racine dans la branche de collaboration. Lors de la publication, Synapse Studio lit ce fichier, recherche le champ `publishBranch`, puis enregistre tous les modèles d’espace de travail dans l’emplacement spécifié. Si la branche n’existe pas, Synapse Studio le crée automatiquement. Vous trouverez ci-dessous un exemple de ce à quoi ressemble ce fichier :

```json
{
    "publishBranch": "workspace_publish"
}
```

Azure Synapse Studio ne peut avoir qu’une seule branche de publication à la fois. Quand vous spécifiez une nouvelle branche de publication, la branche de publication précédente n’est pas supprimée. Si vous souhaitez la supprimer, faites-le manuellement.

### <a name="publish-code-changes"></a>Publier les modifications de code

Après avoir fusionné des modifications dans la branche de collaboration, cliquez sur **Publier** pour publier manuellement les modifications de votre code dans la branche de collaboration pour le service Synapse.

![Publier les modifications](media/gitmode-publish.png)

Un volet latéral s’ouvre, dans lequel vous confirmez que la branche de publication et les modifications en attente sont correctes. Une fois que vous avez vérifié vos modifications, cliquez sur **OK** pour confirmer la publication.

![Confirmer la branche de publication correcte](media/publish-change.png)

> [!IMPORTANT]
> La branche de collaboration n’est pas représentative de ce qui est déployé dans le service. Les modifications apportées à la branche de collaboration *doivent* être publiées manuellement.

## <a name="switch-to-a-different-git-repository"></a>Passer à un autre dépôt Git

Pour basculer vers un autre référentiel Git, accédez à la page de configuration de Git dans le hub de gestion, sous **Contrôle de code source**. Sélectionnez **Déconnecter**. 

![Icône Git](media/remove-repository.png)

Entrez le nom de votre espace de travail, puis cliquez sur **Déconnecter** pour supprimer le référentiel Git associé à votre espace de travail.

Après avoir supprimé l’association avec le dépôt actuel, vous pouvez configurer vos paramètres Git pour utiliser un autre dépôt, puis importer des ressources existantes dans le nouveau dépôt.

> [!IMPORTANT]
> La suppression de la configuration Git d’un espace de travail ne supprime rien dans le référentiel. L’espace de travail Synapse contient toujours toutes les ressources publiées. Vous pouvez continuer à le modifier directement par rapport au service.

## <a name="best-practices-for-git-integration"></a>Meilleures pratiques d'intégration Git

-   **Autorisations**. Une fois que vous disposez d’un référentiel git connecté à votre espace de travail, toute personne ayant accès à votre référentiel git avec un rôle dans votre espace de travail est en mesure de mettre à jour les artefacts, notamment un script SQL, un notebook, une définition de travail Spark, un jeu de données, un flux de données et le pipeline en mode git. En règle générale, vous ne souhaitez pas autoriser tous les membres de l’équipe à mettre à jour l’espace de travail. Accordez uniquement l’autorisation d’accès au référentiel git aux auteurs d’artefacts de l’espace de travail Synapse. 
-   **Collaboration**. Il est recommandé de ne pas autoriser les archivages directs dans la branche de collaboration. Cette restriction peut aider à éviter les bogues, car chaque archivage passe par un processus de demande de tirage décrit dans [Création de branches de fonctionnalités](source-control.md#creating-feature-branches).
-   **Mode réel Synapse**. Après la publication en mode git, toutes les modifications sont reflétées en mode réel Synapse. En mode réel Synapse, la publication est désactivée. Vous pouvez afficher et exécuter des artefacts en mode réel si vous avez reçu l’autorisation appropriée. 
-   **Modifier les artefacts dans Studio**. Synapse Studio est le seul endroit où vous pouvez activer le contrôle de code source de l’espace de travail et synchroniser automatiquement les modifications dans Git. Toute modification via le Kit de développement logiciel (SDK), PowerShell, n’est pas synchronisée avec git. Nous vous recommandons de toujours modifier l’artefact dans Studio lorsque git est activé.

## <a name="troubleshooting-git-integration"></a>Résolution des problèmes d’intégration Git

### <a name="access-to-git-mode"></a>Accès au mode git 

Si vous avez reçu l’autorisation d’accès au référentiel git GitHub lié à votre espace de travail, mais que vous ne pouvez pas accéder au mode git : 

1. Effacez le cache de votre navigateur et actualisez la page. 

1. Connectez-vous à votre compte GitHub.

### <a name="stale-publish-branch"></a>Branche de publication obsolète

Si la branche de publication n’est pas synchronisée avec la branche de collaboration et contient des ressources obsolètes malgré une publication récente, essayez d’effectuer les étapes suivantes :

1. Supprimez votre dépôt Git actuel

1. Reconfigurez Git avec les mêmes paramètres, mais vérifiez que l’option **Import existing resources to repository** (Importer des ressources existantes dans le référentiel) est sélectionnée, puis choisissez la même branche.  

1. Créez une demande de tirage pour fusionner les modifications apportées à la branche de collaboration 

## <a name="unsupported-features"></a>Fonctionnalités non prises en charge

- Synapse Studio n’autorise pas le cherry-picking des validations ni la publication sélective des ressources. 
- Synapse Studio ne prend pas en charge le message de validation de personnalisation.
- Par défaut, l’action de suppression dans Studio sera validée directement dans git

## <a name="next-steps"></a>Étapes suivantes

* Pour implémenter l’intégration et le déploiement continus, consultez [Intégration et livraison continues (CI/CD)](continuous-integration-deployment.md).

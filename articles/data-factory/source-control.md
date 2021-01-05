---
title: Contrôle de code source
description: Découvrez comment configurer le contrôle de code source dans Azure Data Factory
services: data-factory
ms.service: data-factory
ms.workload: data-services
author: dcstwh
ms.author: weetok
manager: anandsub
ms.reviewer: ''
ms.topic: conceptual
ms.custom: seo-lt-2019
ms.date: 11/02/2020
ms.openlocfilehash: d219ff8469c471a37deb47e0f217292d70e8f0f9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96497110"
---
# <a name="source-control-in-azure-data-factory"></a>Contrôle de code source dans Azure Data Factory
[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Par défaut, l’interface utilisateur d’Azure Data Factory publie directement pour le service de fabrique de données. Cette expérience présente les limites suivantes :

- Le service Data Factory n’inclut pas de référentiel pour stocker les entités JSON de vos modifications. La seule façon d’enregistrer les modifications consiste à utiliser le bouton **Publier tout** pour publier toutes les modifications directement vers le service de fabrique de données.
- Le service de fabrique de données n’est pas optimisé pour la collaboration ou le contrôle de version.

Pour améliorer l’expérience de création, Azure Data Factory permet de configurer un dépôt Git avec Azure Repos ou GitHub. Git est un système de contrôle de version qui facilite le suivi des modifications et la collaboration. Cet article explique comment configurer et utiliser un référentiel Git, met en évidence les meilleures pratiques et constitue un guide de résolution des problèmes.

> [!NOTE]
> L’intégration Git d’Azure Data Factory n’est pas disponible dans le cloud Azure Government.

Pour en savoir plus sur la façon dont Azure Data Factory s’intègre à git, consultez le didacticiel vidéo de 15 minutes ci-dessous :

> [!VIDEO https://www.microsoft.com/videoplayer/embed/RE4GNKv]

## <a name="advantages-of-git-integration"></a>Avantages de l’intégration Git

Voici quelques-uns des avantages que l’intégration Git apporte à l’expérience de création :

-   **Contrôle de la source :** Lorsque vos charges de travail de fabrique de données sont cruciales, vous pouvez intégrer votre fabrique avec Git bénéficier de plusieurs avantages de contrôle source tels que les suivants :
    -   Possibilité de suivre/auditer les modifications.
    -   Possibilité d’annuler les modifications qui ont introduit de bogues.
-   **Sauvegardes partielles :** Lorsque vous créez pour le service de fabrique de données, vous ne pouvez pas enregistrer les modifications en tant que brouillon, et toutes les publications doivent passer par la validation de la fabrique de données. Si vos pipelines ne sont pas finis ou si vous ne souhaitez tout simplement pas perdre de modifications en cas de panne de votre ordinateur, l’intégration Git permet d’apporter des modifications incrémentielles aux ressources de la fabrique de données, quel qu’en soit l’état. La configuration d’un dépôt Git vous permet d’enregistrer les modifications, et donc de publier uniquement après que vous avez testé vos modifications et en êtes satisfait.
-   **Collaboration et contrôle :** Si plusieurs membres de votre équipe contribuent à la même fabrique, vous pouvez leur permettre de collaborer via un processus de révision du code. Vous pouvez également configurer votre fabrique de manière à ce que tous les contributeurs n’aient pas des autorisations égales. Certains membres de l’équipe peuvent n’être autorisés qu’à apporter des modifications via Git, tandis que d’autres sont autorisés à publier les modifications dans la fabrique.
-   **Intégration et livraison continus améliorés :**  Si vous déployez dans plusieurs environnements au travers d’un [processus de livraison continue](continuous-integration-deployment.md), l’intégration Git facilite certaines actions. Certaines de ces actions sont les suivantes :
    -   Configurez votre pipeline de mise en production afin qu’il se déclenche automatiquement dès que des modifications sont apportées à votre fabrique « dev ».
    -   Personnalisez les propriétés de votre fabrique disponibles en tant que paramètres dans le modèle Resource Manager. Cela permet de ne conserver que les propriétés requises en tant que paramètres, et de coder en dur tout le reste.
-   **Meilleures performances :** Une fabrique moyenne avec l’intégration Git se charge 10 fois plus rapidement qu’une création pour le service de fabrique de données. Cette amélioration des performances est due au fait que les ressources sont téléchargées via Git.

> [!NOTE]
> La création directe avec le service Data Factory est désactivée dans l’expérience en matière d’interface utilisateur Azure Data Factory quand un dépôt Git est configuré. Les modifications apportées via PowerShell ou un kit de développement logiciel (SDK) sont publiées directement dans le service Data Factory et ne sont pas entrées dans Git.

## <a name="connect-to-a-git-repository"></a>Se connecter à un référentiel Git

Il existe quatre façons différentes de connecter un référentiel Git à votre fabrique de données pour Azure Repos et GitHub. Après vous être connecté à un référentiel Git, vous pouvez visualiser et gérer votre configuration dans le [hub de gestion](author-management-hub.md) sous **Configuration Git** dans la section **Contrôle de code source**.

### <a name="configuration-method-1-home-page"></a>Méthode de configuration 1 : page d'accueil

Dans la page d’accueil Azure Data Factory, sélectionnez **Set up Code Repository** (Configurer le dépôt de code).

![Configurer un référentiel de code à partir de la page d’accueil](media/author-visually/configure-repo.png)

### <a name="configuration-method-2-authoring-canvas"></a>Méthode de configuration 2 : Zone de travail de création

Dans la zone de travail de création de l’expérience en matière d’interface utilisateur Azure Data Factory, sélectionnez le menu déroulant **Data Factory**, puis **Set up Code Repository** (Configurer le dépôt de code).

![Configurer les paramètres du référentiel de code à partir de la création](media/author-visually/configure-repo-2.png)

### <a name="configuration-method-3-management-hub"></a>Méthode de configuration 3 : Hub de gestion

Accédez au hub de gestion dans l’expérience utilisateur ADF. Sélectionnez **Configuration Git** dans la section **Contrôle de code source**. Si vous n’avez pas de référentiel connecté, cliquez sur **Set up code repository** (Configurer le référentiel de code).

![Configurer les paramètres du référentiel de code à partir du hub de gestion](media/author-visually/configure-repo-3.png)

### <a name="configuration-method-4-during-factory-creation"></a>Méthode de configuration 4 : Lors de la création de la fabrique

Lorsque vous créez une fabrique de données dans le Portail Azure, vous pouvez configurer les informations de référentiel Git dans l’onglet **Configuration Git**.

> [!NOTE]
> Lors de la configuration de Git dans le Portail Azure, il est nécessaire d’entrer manuellement des paramètres tels que le nom du projet et le nom du référentiel, dans une liste déroulante.

![Configurer les paramètres du référentiel de code à partir du Portail Azure](media/author-visually/configure-repo-4.png)

## <a name="author-with-azure-repos-git-integration"></a>Créer avec l’intégration Azure Repos Git

La création visuelle avec l’intégration Azure Repos Git prend en charge le contrôle du code source et la collaboration pour les utiliser sur vos pipelines Data Factory. Vous pouvez associer une fabrique de données à un dépôt d’organisation Azure Repos Git pour le contrôle du code source, la collaboration, la gestion des versions, etc. Une seule organisation Azure Repos Git peut avoir plusieurs dépôts, mais un dépôt Azure Repos Git ne peut être associé qu’à une seule fabrique de données. Si vous n’avez pas de dépôt ni d’organisation Azure Repos, suivez [ces instructions](/azure/devops/organizations/accounts/create-organization-msa-or-work-student) pour créer vos ressources.

> [!NOTE]
> Vous pouvez stocker les fichiers de scripts et de données dans un dépôt Azure Repos Git. Toutefois, vous devez charger les fichiers manuellement dans le stockage Azure. Un pipeline de fabrique de données ne charge pas automatiquement les fichiers de scripts ou de données stockés dans un référentiel Azure Repos Git sur le stockage Azure.

### <a name="azure-repos-settings"></a>Paramètres Azure Repos

![Configurer les paramètres du référentiel de code](media/author-visually/repo-settings.png)

Le volet de configuration affiche les paramètres du dépôt de code Azure Repos suivants :

| Paramètre | Description | Valeur |
|:--- |:--- |:--- |
| **Type de référentiel** | Type du dépôt de code Azure Repos.<br/> | Azure DevOps Git ou GitHub |
| **Azure Active Directory** | Le nom de votre abonné Azure AD. | `<your tenant name>` |
| **Organisation Azure Repos** | Nom de votre organisation Azure Repos. Vous trouverez le nom de votre organisation Azure Repos sur `https://{organization name}.visualstudio.com`. Vous pouvez vous [connecter à votre organisation Azure Repos](https://www.visualstudio.com/team-services/git/) pour accéder à votre profil Visual Studio et visualiser vos dépôts et projets. | `<your organization name>` |
| **Nom du projet** | Nom de votre projet Azure Repos. Vous trouverez le nom de votre projet Azure Repos sur `https://{organization name}.visualstudio.com/{project name}`. | `<your Azure Repos project name>` |
| **RepositoryName** | Nom de votre dépôt de code Azure Repos. Les projets Azure Repos contiennent des dépôts Git pour gérer votre code source à mesure que votre projet se développe. Vous pouvez créer un nouveau référentiel ou utiliser un référentiel existant déjà présent dans le projet. | `<your Azure Repos code repository name>` |
| **Branche de collaboration** | Votre branche de collaboration Azure Repos utilisée pour la publication. Par défaut, il s’agit de `master`. Modifiez ce paramètre au cas où vous souhaitez publier des ressources à partir d’une autre branche. | `<your collaboration branch name>` |
| **Dossier racine** | Votre dossier racine de votre branche de collaboration Azure Repos. | `<your root folder name>` |
| **Import existing Data Factory resources to repository** (Importer des ressources Data Factory existantes dans le référentiel) | Indique s’il convient d’importer des ressources de fabrique de données existantes à partir du **canevas de création** de l’expérience utilisateur dans un dépôt Azure Repos Git. Activez la case pour importer vos ressources de fabrique de données dans le référentiel Git associé au format JSON. Cette action exporte chaque ressource individuellement (autrement dit, les services et jeux de données liés sont exportés dans des fichiers JSON distincts). Lorsque cette case n’est pas activée, les ressources existantes ne sont pas importées. | Activée (par défaut) |
| **Branche sur laquelle importer la ressource** | Indique dans quelle branche les ressources de la fabrique de données (pipelines, ensembles de données, services liés, etc.) sont importées. Vous pouvez importer des ressources dans l’une des branches suivantes : a. Collaboration b. Créer c. Utiliser l’existant |  |

> [!NOTE]
> Si vous utilisez Microsoft Edge et que vous ne voyez aucune valeur dans la liste déroulante de votre compte Azure DevOps, ajoutez https://*.visualstudio.com à la liste des sites de confiance.

### <a name="use-a-different-azure-active-directory-tenant"></a>Utiliser un autre locataire Azure Active Directory

Le dépôt Git Azure Repos peut se trouver dans un autre locataire Azure Active Directory. Pour définir un autre locataire Azure AD, vous devez disposer des droits d’administrateur pour l’abonnement Azure que vous utilisez. Pour plus d’informations, consultez la rubrique [Modifier l’administrateur d’abonnement](../cost-management-billing/manage/add-change-subscription-administrator.md#to-assign-a-user-as-an-administrator).

> [!IMPORTANT]
> Pour se connecter à un autre annuaire Azure Active Directory, l’utilisateur connecté doit être membre d’Active Directory. 

### <a name="use-your-personal-microsoft-account"></a>Ajouter votre compte Microsoft personnel

Pour utiliser un compte Microsoft personnel à des fins d'intégration de Git, vous pouvez lier votre référentiel Azure personnel au répertoire Active Directory de votre organisation.

1. Ajoutez votre compte Microsoft personnel au répertoire Active Directory de votre organisation en tant qu’invité. Pour plus d'informations, voir [Ajouter des utilisateurs Azure Active Directory B2B Collaboration dans le Portail Azure](../active-directory/external-identities/add-users-administrator.md).

2. Connectez-vous au portail Azure avec votre compte Microsoft personnel. Basculez ensuite vers le répertoire Active Directory de votre organisation.

3. Accédez à la section Azure DevOps qui affiche désormais votre référentiel personnel. Sélectionnez le référentiel et connectez-vous à Active Directory.

Au terme de cette procédure de configuration, votre référentiel personnel est disponible lorsque vous configurez l’intégration de Git dans l’interface utilisateur de Data Factory.

Pour plus d’informations sur la connexion de référentiels Azure au répertoire Active Directory de votre organisation, voir [Connecter votre organisation Azure DevOps à Azure Active Directory](/azure/devops/organizations/accounts/connect-organization-to-azure-ad).

## <a name="author-with-github-integration"></a>Créer avec l’intégration de GitHub

La création avec l’intégration de GitHub prend en charge le contrôle du code source et la collaboration pendant l’utilisation de vos pipelines de fabrique de données. Vous pouvez associer une fabrique de données à un référentiel de compte GitHub pour le contrôle du code source, la collaboration et la gestion des versions. Un compte GitHub peut avoir plusieurs référentiels, mais un référentiel GitHub ne peut être associé qu’à une seule fabrique de données. Si vous n’avez pas un compte ou un référentiel GitHub, suivez [ces instructions](https://github.com/join) pour créer vos ressources.

L’intégration de GitHub à Data Factory prend à la fois en charge le service GitHub public ([https://github.com](https://github.com)) que GitHub Enterprise. Vous pouvez utiliser des référentiels GitHub publics et privés avec Data Factory à condition de disposer des autorisations de lecture et d'écriture pour ceux-ci dans GitHub.

Pour configurer un dépôt GitHub, vous devez disposer des droits d’administrateur pour l’abonnement Azure que vous utilisez.

### <a name="github-settings"></a>Paramètres GitHub

![Paramètres du référentiel GitHub](media/author-visually/github-integration-image2.png)

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
| **Import existing Data Factory resources to repository** (Importer des ressources Data Factory existantes dans le référentiel) | Indique s’il faut importer des ressources de fabrique de données existantes à partir de la zone de travail de création de l’expérience en matière d’interface utilisateur dans un dépôt GitHub. Activez la case pour importer vos ressources de fabrique de données dans le référentiel Git associé au format JSON. Cette action exporte chaque ressource individuellement (autrement dit, les services et jeux de données liés sont exportés dans des fichiers JSON distincts). Lorsque cette case n’est pas activée, les ressources existantes ne sont pas importées. | Activée (par défaut) |
| **Branche sur laquelle importer la ressource** | Indique dans quelle branche les ressources de la fabrique de données (pipelines, ensembles de données, services liés, etc.) sont importées. Vous pouvez importer des ressources dans l’une des branches suivantes : a. Collaboration b. Créer c. Utiliser l’existant |  |

### <a name="github-organizations"></a>Organisations GitHub

La connexion à une organisation GitHub nécessite que l’organisation accorde l’autorisation à Azure Data Factory. Un utilisateur disposant d’autorisations d’administrateur sur l’organisation doit effectuer les étapes ci-dessous pour permettre à la fabrique de données de se connecter.

#### <a name="connecting-to-github-for-the-first-time-in-azure-data-factory"></a>Première connexion à GitHub dans Azure Data Factory

Si vous vous connectez à GitHub à partir d’Azure Data Factory pour la première fois, procédez comme suit pour vous connecter à une organisation GitHub.

1. Dans le volet Configuration git, entrez le nom de l’organisation dans le champ *Compte GitHub*. Une invite de connexion à GitHub s’affiche. 
1. Connectez-vous à l’aide des informations d’identification de l’utilisateur.
1. Vous êtes invité à autoriser Azure Data Factory en tant qu’application appelée *AzureDataFactory*. Dans cet écran, une option vous permet d’accorder à ADF l’autorisation d’accéder à l’organisation. Si vous ne voyez pas l’option permettant d’accorder une autorisation, demandez à un administrateur d’accorder manuellement l’autorisation par le biais de GitHub.

Une fois ces étapes effectuées, votre fabrique est en mesure de se connecter aux référentiels publics et privés au sein de votre organisation. Si vous ne parvenez pas à vous connecter, essayez de vider le cache du navigateur et de réessayer.

#### <a name="already-connected-to-github-using-a-personal-account"></a>Déjà connecté à GitHub à l’aide d’un compte personnel

Si vous êtes déjà connecté à GitHub et que vous avez accordé uniquement l’autorisation d’accéder à un compte personnel, suivez les étapes ci-dessous pour accorder des autorisations à une organisation. 

1. Accédez à GitHub et ouvrez **Paramètres**.

    ![Ouvrir les paramètres GitHub](media/author-visually/github-settings.png)

1. Sélectionnez **Applications**. Dans l’onglet **Authorized OAuth apps** (Applications OAuth autorisées), vous devez trouver *AzureDataFactory*.

    ![Sélectionner les applications OAuth](media/author-visually/github-organization-select-application.png)

1. Sélectionnez l’application et accordez-lui l’accès à votre organisation.

    ![Accorder l'accès](media/author-visually/github-organization-grant.png)

Une fois ces étapes effectuées, votre fabrique est en mesure de se connecter aux référentiels publics et privés au sein de votre organisation. 

### <a name="known-github-limitations"></a>Limitations connues de GitHub

- Vous pouvez stocker les fichiers de scripts et de données dans un référentiel GitHub. Toutefois, vous devez charger les fichiers manuellement dans le stockage Azure. Un pipeline Data Factory ne charge pas automatiquement les fichiers de scripts ou de données stockés dans un référentiel GitHub vers le stockage Azure.

- Les versions de GitHub Enterprise antérieures à 2.14.0 ne fonctionnent pas dans le navigateur Microsoft Edge.

- L’intégration de GitHub aux outils de création visuelle Data Factory ne fonctionne que dans la version généralement disponible de Data Factory.


- Un maximum de 1 000 entités par type de ressource (par exemple, des pipelines et des jeux de données) peut être extrait à partir d’une seule branche GitHub. Si cette limite est atteinte, il est suggéré de fractionner vos ressources en fabriques distinctes. Azure DevOps Git n’a pas cette limitation.

## <a name="version-control"></a>Gestion de versions

Les systèmes de contrôle de version (également appelé _contrôle du code source_) permettent aux développeurs de collaborer sur le code et de suivre les modifications apportées à la base de code. Le contrôle du code source est un outil essentiel pour les projets impliquant plusieurs développeurs.

### <a name="creating-feature-branches"></a>Création de branches de fonctionnalités

Chaque dépôt Azure Repos Git associé à une fabrique de données comporte une branche de collaboration. (`main` est la branche de collaboration par défaut). Les utilisateurs peuvent également créer des branches de fonctionnalités en cliquant sur **+ Nouvelle branche** dans la liste déroulante des branches. Une fois le volet de la nouvelle branche affiché, entrez le nom de votre branche de fonctionnalités.

![Créer une branche](media/author-visually/new-branch.png)

Lorsque vous êtes prêt à fusionner les modifications de votre branche de fonctionnalités dans votre branche de collaboration, cliquez sur la liste déroulante des branches et sélectionnez **Créer la demande de tirage (pull request)** . Cette action vous dirige vers Azure Repos Git où vous pouvez augmenter les demandes de tirage, procéder à des revues du code et fusionner les modifications dans votre branche de collaboration. (`master` est la valeur par défaut). Vous êtes uniquement autorisé à publier sur le service Data Factory à partir de votre branche de collaboration. 

![Créer une nouvelle demande de tirage (pull request)](media/author-visually/create-pull-request.png)

### <a name="configure-publishing-settings"></a>Configurer les paramètres de publication

Par défaut, la fabrique de données génère les modèles Resource Manager de la fabrique publiée et les enregistre dans une branche nommée `adf_publish`. Pour configurer une branche de publication personnalisée, ajoutez un fichier `publish_config.json` au dossier racine dans la branche de collaboration. Lors de la publication, ADF lit ce fichier, recherche le champ `publishBranch`, puis enregistre tous les modèles Resource Manager dans l’emplacement spécifié. Si la branche n’existe pas, la fabrique de données le crée automatiquement. Vous trouverez ci-dessous un exemple de ce à quoi ressemble ce fichier :

```json
{
    "publishBranch": "factory/adf_publish"
}
```

Azure Data Factory ne peut avoir qu’une seule branche de publication à la fois. Quand vous spécifiez une nouvelle branche de publication, Data Factory ne supprime pas la branche de publication précédente. Si vous souhaitez la supprimer, faites-le manuellement.

> [!NOTE]
> Data Factory lit uniquement le fichier `publish_config.json` lors du chargement de la fabrique. Si la fabrique est déjà chargée sur le portail, actualisez le navigateur pour que vos modifications prennent effet.

### <a name="publish-code-changes"></a>Publier les modifications de code

Après avoir fusionné des modifications dans la branche de collaboration (`master` est la valeur par défaut), cliquez sur **Publier** pour publier manuellement les modifications de votre code dans la branche principale pour le service Data Factory.

![Publier les modifications apportées au service Data Factory](media/author-visually/publish-changes.png)

Un volet latéral s’ouvre, dans lequel vous confirmez que la branche de publication et les modifications en attente sont correctes. Une fois que vous avez vérifié vos modifications, cliquez sur **OK** pour confirmer la publication.

![Confirmer la branche de publication correcte](media/author-visually/configure-publish-branch.png)

> [!IMPORTANT]
> La branche principale n’est pas représentative de ce qui est déployé dans le service Data Factory. La branche principale *doit* être publiée manuellement sur le service Data Factory.

## <a name="best-practices-for-git-integration"></a>Meilleures pratiques d'intégration Git

### <a name="permissions"></a>Autorisations

En règle générale, vous ne souhaitez pas autoriser tous les membres de l’équipe à mettre à jour la fabrique de données. Les paramètres des autorisations suivants sont recommandés :

*   Tous les membres de l’équipe doivent disposer d’autorisations de lecture dans la fabrique de données.
*   Seul un ensemble sélectionné de personnes doit être autorisé à publier sur la fabrique de données. Pour ce faire, ces personnes doivent avoir le rôle **Contributeur de Data Factory** sur le **groupe de ressources** contenant la fabrique de données. Pour plus d’informations sur les autorisations, consultez [Rôles et autorisations pour Azure Data Factory](concepts-roles-permissions.md).

Il est recommandé de ne pas autoriser les archivages directs dans la branche de collaboration. Cette restriction peut aider à éviter les bogues, car chaque archivage passe par un processus de demande de tirage décrit dans [Création de branches de fonctionnalités](source-control.md#creating-feature-branches).

### <a name="using-passwords-from-azure-key-vault"></a>Utilisation de mots de passe à partir d’Azure Key Vault

Il est recommandé d’utiliser Azure Key Vault pour stocker les chaînes de connexion, les mots de passe ou l’authentification d’identité managée pour des services liés Data Factory. Pour des raisons de sécurité, la fabrique de données ne stocke pas les secrets dans Git. Toutes les modifications apportées aux services liés contenant des secrets tels que des mots de passe sont immédiatement publiées dans le service Azure Data Factory.

L’utilisation de Key Vault de l’authentification MSI facilite également l’intégration et le déploiement continus, car vous n’avez pas à fournir ces secrets lors du déploiement du modèle Resource Manager.

## <a name="troubleshooting-git-integration"></a>Résolution des problèmes d’intégration Git

### <a name="stale-publish-branch"></a>Branche de publication obsolète

Si la branche de publication n’est pas synchronisée avec la branche principale et contient des ressources obsolètes malgré une publication récente, essayez d’effectuer les étapes suivantes :

1. Supprimez votre dépôt Git actuel
1. Reconfigurez Git avec les mêmes paramètres, mais vérifiez que l’option **Import existing Data Factory resources to repository** (Importer des ressources Data Factory existantes dans le dépôt) est sélectionnée et choisissez **Nouvelle branche**
1. Créez une demande de tirage pour fusionner les modifications apportées à la branche de collaboration 

Voici quelques exemples de situations qui peuvent provoquer une branche de publication obsolète :
- Un utilisateur a plusieurs branches. Dans une branche de fonctionnalité, l’utilisateur a supprimé un service lié qui n’est pas associé à AKV (les services liés non-AKV sont publiés immédiatement, qu’ils soient dans Git ou non) et n’a jamais fusionné la branche de fonctionnalité dans la branche de collaboration.
- Un utilisateur a modifié la fabrique de données à l’aide du SDK ou de PowerShell.
- Un utilisateur a déplacé toutes les ressources vers une nouvelle branche et a essayé de publier pour la première fois. Les services liés doivent être créés manuellement au moment de l’importation des ressources.
- Un utilisateur charge manuellement un service lié ou un fichier JSON de runtime d’intégration non-AKV. Il fait référence à cette ressource à partir d’une autre ressource telle qu’un jeu de données, un service lié ou un pipeline. Un service lié non-AKV créé par le biais de l’expérience utilisateur est publié immédiatement parce que les informations d’identification doivent être chiffrées. Si vous chargez un jeu de données qui fait référence à ce service lié et essayez de le publier, l’expérience utilisateur autorise cette opération puisqu’il existe dans l’environnement git. Il est rejeté au moment de la publication dans la mesure où il n’existe pas dans le service de fabrique de données.

## <a name="switch-to-a-different-git-repository"></a>Passer à un autre dépôt Git

Pour basculer vers un autre référentiel Git, accédez à la page de configuration de Git dans le hub de gestion, sous **Contrôle de code source**. Sélectionnez **Déconnecter**. 

![Icône Git](media/author-visually/remove-repository.png)

Entrez le nom de votre fabrique de données, puis cliquez sur **Confirmer** pour supprimer le dépôt Git associé à votre fabrique de données.

![Supprimer l’association au référentiel Git actuel](media/author-visually/remove-repository-2.png)

Après avoir supprimé l’association avec le dépôt actuel, vous pouvez configurer vos paramètres Git pour utiliser un autre dépôt, puis importer des ressources Data Factory dans le nouveau dépôt.

> [!IMPORTANT]
> La suppression de la configuration Git d’une fabrique de données ne supprime rien dans le référentiel. La fabrique contient toujours toutes les ressources publiées. Vous pouvez continuer à la modifier directement par rapport au service.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la surveillance et la gestion des pipelines, consultez [Surveiller et gérer les pipelines par programmation](monitor-programmatically.md).
* Pour implémenter l’intégration et le déploiement continus, consultez [Intégration et livraison continues (CI/CD) dans Azure Data Factory](continuous-integration-deployment.md).

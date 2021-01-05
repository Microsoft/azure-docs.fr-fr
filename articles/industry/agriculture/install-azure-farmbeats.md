---
title: Installer Azure FarmBeats
description: Cet article explique comment installer Azure FarmBeats dans votre abonnement Azure
author: usha-rathnavel
ms.topic: article
ms.date: 1/17/2020
ms.author: atinb
ms.openlocfilehash: 5863dcc20fb13f1bb203c68ad168655371130601
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92674563"
---
# <a name="install-azure-farmbeats"></a>Installer Azure FarmBeats

Cet article explique comment installer Azure FarmBeats dans votre abonnement Azure.

Azure FarmBeats est une offre B2B (entreprise à entreprise) disponible sur la Place de marché Azure. Cette solution permet d’agréger des jeux de données agricoles provenant de différents fournisseurs et de générer des insights exploitables. Avec Azure FarmBeats, il est possible de créer des modèles d’intelligence artificielle (IA) ou de Machine Learning (ML) basés sur les jeux de données fusionnés. Les deux principaux composants d’Azure FarmBeats sont :

- **Datahub**  : couche d’API qui permet l’agrégation, la normalisation et la contextualisation de divers jeux de données agricoles issus de différents fournisseurs.

- **Accélérateur**  : application web basée sur Datahub. Il vous permet d’accélérer le développement et la visualisation de votre modèle. L’accélérateur se sert des API Azure FarmBeats pour assurer la visualisation des données de capteurs ingérées sous forme de graphiques et la visualisation de la sortie du modèle sous forme de cartes.

## <a name="general-information"></a>Informations générales

### <a name="components-installed"></a>Composants installés

Quand vous installez Azure FarmBeats, voici les ressources qui sont provisionnées dans votre abonnement Azure :

| Ressources Azure installées  | Composant Azure FarmBeats  |
|---------|---------|
| Application Insights   |      Datahub et Accélérateur      |
| App Service     |     Datahub et Accélérateur     |
| Plan App Service   | Datahub et Accélérateur  |
| ApiConnection    |  Datahub       |
| Cache Azure pour Redis       | Datahub      |
| Azure Cosmos DB   |  Datahub       |
| Azure Data Factory V2       |     Datahub et Accélérateur      |
| Compte Azure Batch    | Datahub   |
| Azure Key Vault |  Datahub et Accélérateur        |
| Compte Azure Maps       |     Accélérateur    |
| Espace de noms Event Hub    |     Datahub      |
| Application logique      |  Datahub       |
| Compte de stockage      |     Datahub et Accélérateur      |
| Time Series Insights     |    Datahub    |

### <a name="costs-incurred"></a>Coûts induits

Les coûts d’Azure FarmBeats représentent la somme des coûts des services Azure sous-jacents. Le prix des services Azure peut être calculé à l’aide de la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator). Le coût réel de l’installation complète varie en fonction de l’utilisation. Pour les deux composants, le coût à l’état stable est le suivant :

- Datahub – moins de 10 USD par jour
- Accélérateur – moins de 2 USD par jour

### <a name="regions-supported"></a>Régions prises en charge

Actuellement, Azure FarmBeats est pris en charge dans les environnement de cloud public dans les régions suivantes :

- Australie Est
- USA Centre
- USA Est
- USA Est 2
- USA Ouest
- USA Ouest 2
- Europe Nord
- Europe Ouest
- Asie Est
- Asie Sud-Est

### <a name="time-taken"></a>Temps nécessaire

L’installation complète d’Azure FarmBeats, préparation et installation comprises, prend moins d’une heure.

## <a name="prerequisites"></a>Prérequis

Avant d'entamer l'installation proprement dite d'Azure FarmBeats, vous devez suivre les étapes ci-dessous :

### <a name="verify-permissions"></a>Vérifier les autorisations

Pour installer Azure FarmBeats, vous devez disposer des autorisations suivantes sur le locataire Azure :

- Locataire - Créateur d'application AAD
- Abonnement - Propriétaire
- Groupe de ressources dans lequel FarmBeats est installé - Propriétaire

Les deux premières autorisations sont requises pour l'étape de [création de l'application AAD](#create-an-aad-application). Si nécessaire, vous pouvez demander à une personne disposant des autorisations appropriées de créer l'application AAD.

La personne qui exécute l’installation de FarmBeats à partir de la Place de Marché doit être propriétaire du groupe de ressources dans lequel FarmBeats est installé. Les propriétaires d’abonnement le sont automatiquement quand le groupe de ressources est créé. Pour d’autres, vous devez créer préalablement le groupe de ressources et demander au propriétaire de l’abonnement de vous ajouter comme propriétaire du groupe de ressources.

Vous pouvez vérifier vos autorisations d’accès sur le Portail Azure en suivant les instructions relatives au [contrôle d'accès en fonction du rôle Azure](../../role-based-access-control/check-access.md).

### <a name="decide-subscription-and-region"></a>Choisir un abonnement et une région

Vous aurez besoin de l’ID d’abonnement Azure et de la région dans laquelle vous souhaitez installer Azure FarmBeats. Choisissez l’une des régions listées dans la section [Régions prises en charge](#regions-supported).

Notez l’ **ID d’abonnement Azure** et la **région Azure** .

### <a name="create-an-aad-application"></a>Créer une application AAD

Azure FarmBeats exige la création et l’inscription d’applications Azure Active Directory. Pour exécuter le script de création AAD, les autorisations suivantes sont nécessaires :

- Locataire - Créateur d'application AAD
- Abonnement - Propriétaire

Suivez les étapes ci-dessous sur une instance de Cloud Shell à l'aide de l'environnement PowerShell. Lors de leur première utilisation, les utilisateurs sont invités à sélectionner un abonnement et à créer un compte de stockage. Effectuez l’installation en suivant les instructions.

1. Téléchargez le [script du générateur d'application AAD](https://aka.ms/FarmBeatsAADScript).

    ```azurepowershell-interactive
        wget -q https://aka.ms/FarmBeatsAADScript -O ./create_aad_script.ps1
    ```

2. Par défaut, le fichier est téléchargé dans votre répertoire de base. Accédez au répertoire.

    ```azurepowershell-interactive
        cd
    ```

3. Exécutez le script AAD.

    ```azurepowershell-interactive
        ./create_aad_script.ps1
    ```

4. Le script demande les trois entrées suivantes :

    - **Nom du site web FarmBeats**  : il s’agit du préfixe d’URL unique pour votre application web FarmBeats. Si le préfixe est déjà utilisé, le script génère une erreur. Une fois installé, votre déploiement FarmBeats est accessible à l’adresse https://\<FarmBeats-website-name>.azurewebsites.net et les API Swagger à l’adresse https://\<FarmBeats-website-name>-api.azurewebsites.net

    - **ID de connexion Azure**  : indiquez l’ID de connexion Azure de l’utilisateur que vous souhaitez ajouter comme administrateur de FarmBeats. Cet utilisateur peut ensuite accorder l’accès à l’application web FarmBeats à d’autres utilisateurs. L’ID de connexion se présente généralement sous la forme john.doe@domain.com. Le nom d’utilisateur principal Azure est également pris en charge.

    - **ID de l'abonnement**  : il s’agit de l’ID de l’abonnement dans lequel vous souhaitez installer Azure FarmBeats.

5. Le script AAD, dont l'exécution prend environ 2 minutes, génère des valeurs à l'écran ainsi que dans un fichier JSON situé dans le même répertoire. Si le script a été exécuté par une autre personne, demandez-lui de partager cette sortie avec vous.

### <a name="create-sentinel-account"></a>Créer un compte Sentinel

Votre installation Azure FarmBeats vous permet de bénéficier des images satellite de la mission [Sentinel-2](https://scihub.copernicus.eu/) de l'Agence spatiale européenne pour votre exploitation agricole. Pour configurer cette installation, vous devez disposer d'un compte Sentinel.

Suivez les étapes permettant de créer un compte gratuit Sentinel :

1. Accédez à la page d’[inscription](https://aka.ms/SentinelRegistration) officielle.
2. Fournissez les informations nécessaires (prénom, nom, nom d’utilisateur, mot de passe et ID d’e-mail) et complétez le formulaire.
3. Un lien de vérification est envoyé à l'adresse e-mail enregistrée. Sélectionnez le lien fourni dans l’e-mail et effectuez la vérification.

Votre processus d’inscription est terminé. Au terme de la vérification, notez votre **Nom d'utilisateur Sentinel** et votre **Mot de passe Sentinel** .

## <a name="install"></a>Installer

Vous êtes maintenant prêt à installer FarmBeats. Procédez comme suit pour lancer l'installation :

1. Connectez-vous au portail Azure. Sélectionnez votre compte en haut à droite, puis basculez vers le locataire Azure AD où vous voulez installer Azure FarmBeats.

2. Sur le portail, accédez à la Place de marché Azure et recherchez-y **Azure FarmBeats** .

3. Une nouvelle fenêtre s’affiche avec une vue d’ensemble d’Azure FarmBeats. Sélectionnez **Create** (Créer).

4. Une nouvelle fenêtre s’affiche. Terminez le processus d’inscription en choisissant l’abonnement, le groupe de ressources et l’emplacement où vous souhaitez installer Azure FarmBeats.

5. Dans la section **Alertes de service FarmBeats** , entrez l'adresse e-mail à laquelle les alertes de service liées à Azure FarmBeats doivent être envoyées. Sélectionnez **Suivant** en bas de la page pour accéder à l'onglet **Dépendances** .

    ![Onglet Informations de base](./media/install-azure-farmbeats/create-azure-farmbeats-basics.png)

6. Copiez les différentes entrées de la sortie du [script AAD](#create-an-aad-application) dans les entrées de la section Application AAD.

7. Entrez le nom d'utilisateur et le mot de passe du [compte Sentinel](#create-sentinel-account) dans la section Compte Sentinel. Sélectionnez **Suivant** pour accéder à l'onglet **Vérifier et créer** .

    ![Onglet Dépendances](./media/install-azure-farmbeats/create-azure-farmbeats-dependencies.png)

8. Après avoir validé les informations entrées, sélectionnez **OK** . La page Conditions d’utilisation apparaît. Lisez les conditions d'utilisation et sélectionnez **Créer** pour entamer l'installation. Vous êtes redirigé vers une page qui vous permet de suivre la progression de l'installation.

Au terme du processus, vous pouvez vérifier l’installation et commencer à utiliser le portail FarmBeats en accédant au site web dont vous avez fourni le nom lors de l’installation : https://\<FarmBeats-website-name>.azurewebsites.net. L'interface utilisateur FarmBeats apparaît, avec une option permettant de créer des exploitations agricoles.

L’URL de **DataHub** est https://\<FarmBeats-website-name>-api.azurewebsites.net/swagger. Vous avez ici accès aux différents objets d'API FarmBeats et pouvez effectuer des opérations REST sur les API.

## <a name="upgrade"></a>Mettre à niveau

Pour procéder à la mise à niveau de FarmBeats vers la dernière version disponible, suivez les étapes ci-dessous sur une instance de Cloud Shell à l'aide de l'environnement PowerShell. L'utilisateur doit être le propriétaire de l'abonnement avec lequel FarmBeats est installé.

Lors de leur première utilisation, les utilisateurs sont invités à sélectionner un abonnement et à créer un compte de stockage. Effectuez l’installation en suivant les instructions.

1. Téléchargez le [script de mise à niveau](https://aka.ms/FarmBeatsUpgradeScript).

    ```azurepowershell-interactive
        wget –q https://aka.ms/FarmBeatsUpgradeScript -O ./upgrade-farmbeats.ps1
    ```

2. Par défaut, le fichier est téléchargé dans votre répertoire de base. Accédez au répertoire.

    ```azurepowershell-interactive
        cd
    ```

3. Exécutez le script de mise à niveau.

    ```azurepowershell-interactive
        ./upgrade-farmbeats.ps1 -InputFilePath [Path to input.json file]
    ```

Le chemin d'accès au fichier input.json est facultatif. S'il n'est pas spécifié, le script demandera toutes les entrées requises. La mise à niveau dure environ 30 minutes.

## <a name="uninstall"></a>Désinstaller l’interface

Pour désinstaller Azure FarmBeats Datahub ou l’accélérateur, effectuez les étapes suivantes :

1. Connectez-vous au portail Azure et **supprimez les groupes de ressources** où sont installés ces composants.

2. Accédez à Azure Active Directory et **supprimez l’application Azure AD** liée à l’installation d’Azure FarmBeats.

## <a name="next-steps"></a>Étapes suivantes

Vous avez découvert comment installer Azure FarmBeats dans votre abonnement Azure. À présent, découvrez comment [ajouter des utilisateurs](manage-users-in-azure-farmbeats.md#manage-users) à votre instance Azure FarmBeats.
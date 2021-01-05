---
title: Déployer Azure Blockchain Workbench Preview
description: Guide pratique pour déployer Azure Blockchain Workbench Preview
ms.date: 07/16/2020
ms.topic: how-to
ms.reviewer: ravastra
ms.custom: references_regions
ms.openlocfilehash: b46a35b45a51d0cc76942c4ca142c4c7792a28b4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87077028"
---
# <a name="deploy-azure-blockchain-workbench-preview"></a>Déployer Azure Blockchain Workbench Preview

Azure Blockchain Workbench Preview est déployé avec un modèle de solution dans la Place de marché Azure. Ce modèle simplifie le déploiement des composants nécessaires à la création d’applications blockchain. Une fois déployé, Blockchain Workbench permet d’accéder aux applications clientes afin de créer et gérer les utilisateurs et les applications blockchain.

Pour plus d’informations sur les composants Blockchain Workbench, consultez [Architecture Azure Blockchain Workbench](architecture.md).

[!INCLUDE [Preview note](./includes/preview.md)]

## <a name="prepare-for-deployment"></a>Préparer le déploiement

Blockchain Workbench vous permet de déployer un registre blockchain, ainsi qu’un ensemble de services Azure pertinents plus souvent utilisés pour générer une application basée sur les blockchains. Le déploiement de Blockchain Workbench entraîne l’approvisionnement des services Azure suivants au sein d’un groupe de ressources dans votre abonnement Azure.

* Plan App Service (standard)
* Application Insights
* Event Grid
* Azure Key Vault
* Service Bus
* SQL Database (Standard S0)
* Compte de stockage Azure (LRS standard)
* Groupe de machines virtuelles identiques avec capacité de 1
* Groupe de ressources de réseau virtuel (avec Load Balancer, groupe de sécurité réseau, adresse IP publique, réseau virtuel)
* Azure Blockchain Service. Si vous utilisez un déploiement Blockchain Workbench précédent, envisagez de redéployer Azure Blockchain Workbench pour utiliser Azure Blockchain Service.

Voici un exemple de déploiement créé dans le groupe de ressources **myblockchain**.

![Exemple de déploiement](media/deploy/example-deployment.png)

Le coût associé à Blockchain Workbench est un agrégat du coût des services Azure sous-jacents. Les informations de tarification pour les services Azure peuvent être calculées à l’aide de la [calculatrice de prix](https://azure.microsoft.com/pricing/calculator/).

## <a name="prerequisites"></a>Prérequis

Azure Blockchain Workbench requiert des inscriptions d’applications et la configuration Azure AD. Vous pouvez choisir d’effectuer les [configurations manuellement](#azure-ad-configuration) pour Azure AD avant le déploiement ou l’exécution d’un script post-déploiement. Si vous redéployez Blockchain Workbench, consultez la [configuration Azure AD](#azure-ad-configuration) pour vérifier votre configuration Azure AD.

> [!IMPORTANT]
> La solution Workbench n’a pas besoin d’être déployée dans le même locataire que celui que vous utilisez pour inscrire une application Azure AD. Elle doit être déployée dans un locataire pour lequel vous disposez d’autorisations suffisantes pour déployer des ressources. Pour plus d’informations sur les locataires Azure AD, consultez [Obtention d’un client Azure Active Directory](../../active-directory/develop/quickstart-create-new-tenant.md) et [Intégration d’applications dans Azure Active Directory](../../active-directory/develop/quickstart-register-app.md).

## <a name="deploy-blockchain-workbench"></a>Déployer Blockchain Workbench

Une fois que les étapes préalables requises ont été exécutées, vous êtes prêt à déployer Blockchain Workbench. Les sections suivantes expliquent comment déployer l’infrastructure.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez votre compte en haut à droite, puis basculez vers le locataire Azure AD où vous voulez déployer Azure Blockchain Workbench.
1. Sélectionnez **Créer une ressource** dans le coin supérieur gauche du portail Azure.
1. Sélectionnez **Blockchain** > **Azure Blockchain Workbench (préversion)** .

    ![Créer Azure Blockchain Workbench](media/deploy/blockchain-workbench-settings-basic.png)

    | Paramètre | Description  |
    |---------|--------------|
    | Préfixe de ressource | Identificateur unique court pour votre déploiement. Cette valeur est utilisée comme base pour les noms des ressources. |
    | Nom d’utilisateur de la machine virtuelle | Le nom d’utilisateur sert en tant qu’administrateur pour toutes les machines virtuelles. |
    | Type d'authentification | Indiquez si vous souhaitez utiliser un mot de passe ou une clé pour la connexion aux machines virtuelles. |
    | Mot de passe | Le mot de passe est utilisé pour la connexion aux machines virtuelles. |
    | SSH | Utilisez une clé publique RSA au format ligne unique et commençant par **ssh-rsa** ou utilisez le format PEM multiligne. Vous pouvez générer des clés SSH à l’aide de `ssh-keygen` sur Linux et OS X, ou à l’aide de PuTTYGen sur Windows. Pour plus d’informations sur les clés SSH, consultez [Comment utiliser des clés SSH avec Windows sur Azure](../../virtual-machines/linux/ssh-from-windows.md). |
    | Mot de passe de base de données et de Blockchain | Spécifiez le mot de passe à utiliser pour accéder à la base de données créée dans le cadre du déploiement. Le mot de passe doit remplir trois des quatre conditions suivantes : il doit comprendre entre 12 et 72 caractères, 1 caractère minuscule, 1 caractère majuscule, 1 chiffre et 1 caractère spécial qui ne soit pas le signe dièze (#), le symbole de pourcentage (%), une virgule (,), une étoile (*), un guillemet inverse (\`), un guillemet double ("), un guillemet unique ('), un tiret (-) ou un point-virgule (;) |
    | Région du déploiement | Spécifiez où déployer les ressources Blockchain Workbench. Pour une disponibilité optimale, cette valeur doit correspondre au paramètre d’emplacement **Région**. Toutes les régions ne sont pas disponibles pendant la préversion. Les fonctionnalités peuvent ne pas être disponibles dans certaines régions. Azure Blockchain Data Manager est disponible dans les régions Azure suivantes : USA Est et Europe Ouest.|
    | Abonnement | Spécifiez l’abonnement Azure que vous souhaitez utiliser pour votre déploiement. |
    | Groupes de ressources | Créez un groupe de ressources en sélectionnant **Créer** et donnez un nom unique au groupe de ressources. |
    | Emplacement | Spécifiez la région où vous souhaitez déployer l’infrastructure. |

1. Sélectionnez **OK** à la fin de la section de configuration du paramètre de base.

1. Dans **Paramètres avancés**, choisissez si vous souhaitez créer un réseau blockchain ou utiliser un réseau blockchain de preuve d’autorité existant.

    Pour **Créer** :

    L’option *Créer* déploie un registre Azure Blockchain Service Quorum avec la référence SKU de base par défaut.

    ![Paramètres avancés pour le nouveau réseau blockchain](media/deploy/advanced-blockchain-settings-new.png)

    | Paramètre | Description  |
    |---------|--------------|
    | Niveau tarifaire d’Azure Blockchain Service | Choisissez le niveau **De base** ou **Standard** d’Azure Blockchain Service qui est utilisé pour Blockchain Workbench |
    | Paramètres Azure Active Directory | Choisissez **Ajouter ultérieurement**.</br>Remarque : Si vous avez choisi de [préconfigurer Azure AD](#azure-ad-configuration) ou si vous redéployez, optez pour *Ajouter maintenant*. |
    | Sélection de machine virtuelle | Sélectionnez le niveau de performance de stockage préféré et la taille de machine virtuelle pour votre réseau blockchain. Choisissez une plus petite taille de machine virtuelle, comme *Standard DS1 v2*, si vous êtes sur un abonnement doté de limites de service faibles, tel que l’abonnement gratuit d’Azure. |

    Pour **Utiliser l’existant** :

    L’option *Utiliser l’existant* vous permet de spécifier un réseau blockchain Ethereum Proof-of-Authority (PoA). Les points de terminaison doivent répondre aux exigences suivantes.

   * Le point de terminaison doit être un réseau blockchain Ethereum Proof-of-Authority (PoA).
   * Le point de terminaison doit être accessible publiquement sur le réseau.
   * Le réseau blockchain PoA doit être configuré de sorte que le prix du gaz soit défini sur la valeur zéro.

     > [!NOTE]
     > Les comptes Blockchain Workbench ne sont pas financés. Si des fonds sont requis, les transactions échouent.

     ![Paramètres avancés pour le réseau blockchain existant](media/deploy/advanced-blockchain-settings-existing.png)

     | Paramètre | Description  |
     |---------|--------------|
     | Point de terminaison Ethereum RPC | Indiquez le point de terminaison RPC d’un réseau blockchain PoA existant. Le point de terminaison commence par https:// ou http:// et se termine par un numéro de port. Par exemple : `http<s>://<network-url>:<port>` |
     | Paramètres Azure Active Directory | Choisissez **Ajouter ultérieurement**.</br>Remarque : Si vous avez choisi de [préconfigurer Azure AD](#azure-ad-configuration) ou si vous redéployez, optez pour *Ajouter maintenant*. |
     | Sélection de machine virtuelle | Sélectionnez le niveau de performance de stockage préféré et la taille de machine virtuelle pour votre réseau blockchain. Choisissez une plus petite taille de machine virtuelle, comme *Standard DS1 v2*, si vous êtes sur un abonnement doté de limites de service faibles, tel que l’abonnement gratuit d’Azure. |

1. Sélectionnez **Vérifier + créer** pour terminer la configuration de Paramètres avancés.

1. Passez en revue le récapitulatif pour vérifier que vos paramètres sont corrects.

    ![Résumé](media/deploy/blockchain-workbench-summary.png)

1. Sélectionnez **Créer** pour accepter les conditions et déployer votre Azure Blockchain Workbench.

Le déploiement peut prendre jusqu’à 90 minutes. Vous pouvez utiliser le portail Azure pour surveiller la progression. Dans le groupe de ressources nouvellement créé, sélectionnez **Déploiements > Vue d’ensemble** pour connaître l’état des artefacts déployés.

> [!IMPORTANT]
> Après le déploiement, vous devez implémenter les paramètres Active Directory. Si vous avez choisi **Ajouter ultérieurement**, vous devez exécuter le [script de configuration Azure AD](#azure-ad-configuration-script).  Si vous avez choisi **Ajouter maintenant**, vous devez [configurer l’URL de réponse](#configuring-the-reply-url).

## <a name="blockchain-workbench-web-url"></a>URL web Blockchain Workbench

Une fois le déploiement de Blockchain Workbench terminé, un nouveau groupe de ressources contient vos ressources Blockchain Workbench. Les services Blockchain Workbench sont accessibles par le biais d’une URL web. Les étapes suivantes vous montrent comment récupérer l’URL web de l’infrastructure déployée.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Dans le volet de navigation de gauche, sélectionnez **Groupes de ressources**.
1. Choisissez le nom du groupe de ressources spécifié lors du déploiement de Blockchain Workbench.
1. Sélectionnez l’en-tête de colonne **TYPE** pour trier la liste par ordre alphabétique et par type.
1. Deux ressources présentent le type **App Service**. Sélectionnez la ressource de type **App Service** *sans* le suffixe « -api ».

    ![Liste App Service](media/deploy/resource-group-list.png)

1. Dans la section **Vue d’ensemble** App Service, copiez la valeur **URL** qui représente l’URL web sur votre Blockchain Workbench déployé.

    ![Base App Service](media/deploy/app-service.png)

Pour associer un nom de domaine personnalisé avec Blockchain Workbench, voir [Configuration d’un nom de domaine personnalisé pour une application web dans Azure App Service utilisant Traffic Manager](../../app-service/configure-domain-traffic-manager.md).

## <a name="azure-ad-configuration-script"></a>Script de configuration Azure AD

Azure AD doit être configuré pour achever votre déploiement Azure Blockchain Workbench. Vous utiliserez un script PowerShell pour effectuer la configuration.

1. Dans un navigateur, accédez à [l’URL web de Blockchain Workbench](#blockchain-workbench-web-url).
1. Des instructions pour configurer Azure AD à l’aide de Cloud Shell s’affichent. Copiez la commande et lancez Cloud Shell.

    ![Lancer le script AAD](media/deploy/launch-aad-script.png)

1. Choisissez le locataire Azure AD où vous avez déployé Azure Blockchain Workbench.
1. Dans l’environnement PowerShell de Cloud Shell, collez et exécutez la commande.
1. Lorsque vous y êtes invité, entrez le locataire Azure AD que vous souhaitez utiliser pour Azure Blockchain Workbench. Il s’agit du locataire contenant les utilisateurs pour Blockchain Workbench.

    > [!IMPORTANT]
    > L’utilisateur authentifié nécessite des autorisations pour créer des inscriptions d’applications Azure AD et accorder des autorisations d’applications déléguées dans le locataire. Vous devrez peut-être demander à un administrateur du locataire d’exécuter le script de configuration Azure AD ou créer un nouveau locataire.

    ![Entrer le locataire Azure AD](media/deploy/choose-tenant.png)

1. Vous serez invité à vous authentifier auprès du locataire Azure AD à l’aide d’un navigateur. Ouvrez l’URL web dans un navigateur, entrez le code et authentifiez-vous.

    ![Authentification avec un code](media/deploy/authenticate.png)

1. Le script génère plusieurs messages d’état. Vous obtenez un message d’état **SUCCÈS** si le client a bien été déployé.
1. Accédez à l’URL de Blockchain Workbench. Vous êtes invité à donner votre consentement pour accorder des autorisations de lecture du répertoire. Ceci permet à l’application web Blockchain Workbench d’accéder aux utilisateurs dans le locataire. Si vous êtes l’administrateur du locataire, vous pouvez choisir de donner votre consentement pour toute l’organisation. Cette option accepte le consentement pour tous les utilisateurs dans le locataire. Dans le cas contraire, chaque utilisateur est invité à fournir son consentement à la première utilisation de l’application web Blockchain Workbench.
1. Sélectionnez **Accepter** pour donner votre consentement.

     ![Donner son consentement pour lire les profils utilisateurs](media/deploy/graph-permission-consent.png)

1. Une fois le consentement donné, vous pouvez utiliser l’application web Blockchain Workbench.

Vous avez terminé votre déploiement Azure Blockchain Workbench. Consultez la section [Étapes suivantes](#next-steps) pour suivre les suggestions et commencer à utiliser votre déploiement.

## <a name="azure-ad-configuration"></a>Configuration Azure AD

Si vous choisissez de configurer manuellement ou de vérifier les paramètres Azure AD avant le déploiement, effectuez toutes les étapes de cette section. Si vous préférez configurer automatiquement les paramètres Azure AD, utilisez le [script de configuration Azure AD](#azure-ad-configuration-script) après avoir déployé Blockchain Workbench.

### <a name="blockchain-workbench-api-app-registration"></a>Inscription d’application API Blockchain Workbench

Le déploiement de Blockchain Workbench nécessite l’inscription d’une application Azure AD. Vous avez besoin d’un locataire Azure Active Directory (Azure AD) pour inscrire l’application. Vous pouvez utiliser un locataire existant ou en créer un. Si vous utilisez un locataire Azure AD existant, vous avez besoin d’autorisations suffisantes pour inscrire les applications, accorder les autorisations API Graph et autoriser l’accès des invités au sein d’un locataire Azure AD. Si vous n’avez pas d’autorisations suffisantes dans un locataire Azure AD existant, créez un locataire.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez votre compte en haut à droite, puis basculez vers le locataire Azure AD souhaité. Le locataire doit être locataire de l’administrateur de l’abonnement où Azure Blockchain Workbench est déployé et vous devez disposer d’autorisations suffisantes pour inscrire les applications.
1. Sélectionnez le service **Azure Active Directory** dans le volet de navigation gauche. Sélectionnez **Inscription d’applications** > **Nouvelle inscription**.

    ![Inscription d'application](media/deploy/app-registration.png)

1. Indiquez un **nom** d’affichage et choisissez **Comptes dans cet annuaire organisationnel uniquement**.

    ![Créer une inscription d’application](media/deploy/app-registration-create.png)

1. Sélectionnez **Inscrire** pour inscrire l’application Azure AD.

### <a name="modify-manifest"></a>Modifier le manifeste

Ensuite, vous devez modifier le manifeste pour utiliser des rôles d’application dans Azure AD afin de spécifier les administrateurs Blockchain Workbench.  Pour plus d’informations sur les manifestes de l’application, consultez [Manifeste de l’application Azure Active Directory](../../active-directory/develop/reference-app-manifest.md).

1. Un GUID est requis pour le manifeste. Vous pouvez générer un GUID à l’aide de la commande PowerShell `[guid]::NewGuid()` ou `New-GUID` cmdlet. Une autre option consiste à utiliser un site web générateur de GUID.
1. Pour l’application que vous avez inscrite, sélectionnez **Manifeste** dans la section **Gérer**.
1. Vous vous apprêtez à mettre à jour la section **appRoles** du manifeste. Remplacez `"appRoles": []` par le fichier JSON fourni. Remplacez la valeur du champ `id` par le GUID que vous avez généré.
    ![Modifier le manifeste](media/deploy/edit-manifest.png)

    ``` json
    "appRoles": [
         {
           "allowedMemberTypes": [
             "User",
             "Application"
           ],
           "displayName": "Administrator",
           "id": "<A unique GUID>",
           "isEnabled": true,
           "description": "Blockchain Workbench administrator role allows creation of applications, user to role assignments, etc.",
           "value": "Administrator"
         }
       ],
    ```

    > [!IMPORTANT]
    > La valeur **Administrateur** est nécessaire pour identifier les administrateurs Blockchain Workbench.

1. Dans le manifeste, définissez aussi la valeur **Oauth2AllowImplicitFlow** sur **true**.

    ``` json
    "oauth2AllowImplicitFlow": true,
    ```

1. Sélectionnez **Enregistrer** pour enregistrer les modifications apportées au manifeste.

### <a name="add-graph-api-required-permissions"></a>Ajouter les autorisations requises pour l’API Graph

L’application API doit demander l’autorisation de l’utilisateur pour accéder au répertoire. Définissez l’autorisation requise suivante pour l’application API :

1. Dans l’inscription de l’application *API Blockchain*, sélectionnez **Autorisations de l’API**. Par défaut, l’autorisation **User.Read** de l’API Graph est ajoutée.
1. L’application Workbench requiert un accès en lecture aux informations du profil de base des utilisateurs. Dans *Autorisations configurées*, sélectionnez **Ajouter une autorisation**. Dans **API Microsoft**, sélectionnez **Microsoft Graph**.
1. Étant donné que l’application Workbench utilise les informations d’identification de l’utilisateur authentifié, sélectionnez **Autorisations déléguées**.
1. Dans la catégorie *Utilisateur*, choisissez l’autorisation **User.ReadBasic.All**.

    ![Configuration de l’inscription de l’application Azure AD qui montre l’ajout de l’autorisation déléguée User.ReadBasic.All de Microsoft Graph](media/deploy/add-graph-user-permission.png)

    Sélectionnez **Ajouter des autorisations**.

1. Dans *Autorisations configurées*, sélectionnez **Accorder un consentement administrateur** pour le domaine, puis sélectionnez **Oui** pour l’invite de vérification.

   ![Accorder des autorisations](media/deploy/client-app-grant-permissions.png)

   L’octroi d’autorisation permet à Blockchain Workbench d’accéder aux utilisateurs dans le répertoire. L’autorisation de lecture est requise pour la recherche et l’ajout de membres dans Blockchain Workbench.

### <a name="get-application-id"></a>Obtenir l’ID de l’application

L’ID de l’application et les informations du locataire sont requis pour le déploiement. Rassemblez et stockez les informations pour les utiliser pendant le déploiement.

1. Pour l’application que vous avez inscrite, sélectionnez **Vue d’ensemble**.
1. Copiez et stockez la valeur **ID d’application** pour une utilisation ultérieure pendant le déploiement.

    ![Propriétés de l’application API](media/deploy/app-properties.png)

    | Paramètre à stocker  | Utiliser pendant le déploiement |
    |------------------|-------------------|
    | ID d’application (client) | Configuration Azure Active Directory > ID de l’application |

### <a name="get-tenant-domain-name"></a>Obtenir un nom de domaine de locataire

Rassemblez et stockez le nom de domaine du locataire Active Directory où les applications sont inscrites.

Sélectionnez le service **Azure Active Directory** dans le volet de navigation gauche. Sélectionnez **Noms de domaine personnalisés**. Copiez et stockez le nom de domaine.

![Nom de domaine](media/deploy/domain-name.png)

### <a name="guest-user-settings"></a>Paramètres d’utilisateur invité

Si vous avez des utilisateurs invités dans votre locataire Azure AD, suivez les étapes supplémentaires pour vous assurer que l’affectation et la gestion d’utilisateurs de Blockchain Workbench fonctionne correctement.

1. Basculez votre locataire Azure AD et sélectionnez **Azure Active Directory > Paramètres utilisateur > Gérer les paramètres de collaboration externe**.
1. Définissez **Les autorisations de l’utilisateur invité sont limitées** sur **Non**.
    ![Paramètres de collaboration externe](media/deploy/user-collaboration-settings.png)

## <a name="configuring-the-reply-url"></a>Configuration de l’URL de réponse

Une fois Azure Blockchain Workbench déployé, vous devez configurer **l’URL de réponse** de l’application cliente Azure Active Directory (Azure AD) de l’URL web du Blockchain Workbench déployé.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Vérifiez que vous êtes dans le locataire sur lequel vous avez inscrit l’application cliente Azure AD.
1. Sélectionnez le service **Azure Active Directory** dans le volet de navigation gauche. Sélectionnez **Inscriptions d’applications**.
1. Sélectionnez l’application cliente Azure AD que vous avez enregistrée dans la section Configuration requise.
1. Sélectionnez **Authentification**.
1. Spécifiez l’URL web principale du déploiement Azure Blockchain Workbench que vous avez récupérée dans la section [URL web Blockchain Workbench](#blockchain-workbench-web-url). L’URL de réponse est préfixée par `https://`. Par exemple : `https://myblockchain2-7v75.azurewebsites.net`

    ![URL de réponse d’authentification](media/deploy/configure-reply-url.png)

1. Dans la section **Paramètres avancés**, cochez **Jetons d’accès** et **Jetons d’ID**.

    ![Paramètres avancés d’authentification](media/deploy/authentication-advanced-settings.png)

1. Sélectionnez **Enregistrer** pour mettre à jour l’inscription du client.

## <a name="remove-a-deployment"></a>Supprimer un déploiement

Lorsqu’un déploiement n’est plus nécessaire, vous pouvez le supprimer en supprimant le groupe de ressources Blockchain Workbench.

1. Dans le portail Azure, accédez à **Groupe de ressources** dans le volet de navigation de gauche et sélectionnez le groupe de ressources que vous souhaitez supprimer.
1. Sélectionnez **Supprimer le groupe de ressources**. Validez la suppression en entrant le nom du groupe de ressources, puis sélectionnez **Supprimer**.

    ![Supprimer un groupe de ressources](media/deploy/delete-resource-group.png)

## <a name="next-steps"></a>Étapes suivantes

Dans cet article sur les procédures, vous avez déployé Azure Blockchain Workbench. Pour apprendre à créer une application blockchain, passez à l’article suivant.

> [!div class="nextstepaction"]
> [Créer une application blockchain dans Azure Blockchain Workbench](create-app.md)

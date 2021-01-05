---
title: Créer, configurer des clusters du Pack Sécurité Entreprise - Azure
description: Découvrez comment créer et configurer des clusters de Pack Sécurité Entreprise dans Azure HDInsight
services: hdinsight
ms.service: hdinsight
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 12/10/2019
ms.openlocfilehash: 07bd92e17b827dfeede5958587cecbdc97694329
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003920"
---
# <a name="create-and-configure-enterprise-security-package-clusters-in-azure-hdinsight"></a>Créer et configurer des clusters de Pack Sécurité Entreprise dans Azure HDInsight

Le Pack Sécurité Entreprise (ESP) pour Azure HDInsight vous donne accès à l’authentification basée sur Active Directory, à la prise en charge multi-utilisateur et au contrôle d’accès en fonction du rôle pour vos clusters Apache Hadoop dans Azure. Les clusters ESP HDInsight permettent aux organisations, qui adhèrent aux stratégies de sécurité strictes de l’entreprise, de traiter les données sensibles de façon sécurisée.

Ce guide montre comment créer un cluster ESP Azure HDInsight. Il montre également comment créer une machine virtuelle IaaS Windows sur laquelle Active Directory et le système DNS (Domain Name System) sont activés. Utilisez ce guide pour configurer les ressources en vue d’autoriser les utilisateurs locaux à se connecter à un cluster ESP HDInsight.

Le serveur que vous créez vient remplacer votre environnement local *actuel*. Vous l’utiliserez pour les étapes d’installation et de configuration. Plus tard, vous recommencerez ces étapes dans votre propre environnement.

Ce guide vous aidera également à créer un environnement d’identité hybride via la synchronisation du hachage de mot de passe avec Azure Active Directory (Azure AD). Ce guide est un complément au guide [Utilisation de l’ESP dans HDInsight](apache-domain-joined-architecture.md).

Avant d’utiliser ce processus dans votre propre environnement :

* Configurez Active Directory et le service DNS.
* Activez Azure AD.
* Synchronisez les comptes d’utilisateurs locaux avec Azure AD.

![Diagramme montrant l’architecture d’Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0002.png)

## <a name="create-an-on-premises-environment"></a>Créer un environnement local

Dans cette section, vous allez utiliser un modèle de déploiement rapide Azure pour créer des machines virtuelles, configurer les services DNS et créer une forêt Active Directory.

1. Accédez au modèle de déploiement rapide pour [créer une machine virtuelle Azure avec une nouvelle forêt Active Directory](https://azure.microsoft.com/resources/templates/active-directory-new-domain/).

1. Sélectionnez **Déployer sur Azure**.
1. Connectez-vous à votre abonnement Azure.
1. Dans la page **Créer une machine virtuelle Azure avec une nouvelle forêt Active Directory**, entrez les informations suivantes :

    |Propriété | Valeur |
    |---|---|
    |Abonnement|Sélectionnez l’abonnement dans lequel déployer les ressources.|
    |Resource group|Sélectionnez **Créer**, puis entrez le nom `OnPremADVRG`|
    |Emplacement|Sélectionnez un emplacement.|
    |Nom d’utilisateur d’administrateur|`HDIFabrikamAdmin`|
    |Mot de passe d’administrateur|Entrez un mot de passe.|
    |Nom de domaine|`HDIFabrikam.com`|
    |Préfixe DNS|`hdifabrikam`|

    Conservez les valeurs par défaut restantes.

    ![Modèle pour la création d’une machine virtuelle Azure avec une nouvelle forêt Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-azure-vm-ad-forest.png)

1. Lisez les **termes et conditions**, puis cochez la case **J’accepte les termes et conditions mentionnés ci-dessus**.
1. Sélectionnez **Acheter**, puis supervisez le déploiement et attendez qu’il se termine. Le déploiement prend environ 30 minutes.

## <a name="configure-users-and-groups-for-cluster-access"></a>Configurer des utilisateurs et groupes pour l’accès au cluster

Dans cette section, vous allez créer les utilisateurs qui auront accès au cluster HDInsight à la fin de ce guide.

1. Connectez-vous au contrôleur de domaine via le Bureau à distance.
    1. Dans le portail Azure, accédez à **Groupes de ressources** > **OnPremADVRG** > **adVM** > **Connect**.
    1. Dans la liste déroulante **Adresse IP**, sélectionnez l’adresse IP publique.
    1. Sélectionnez **Télécharger le fichier RDP**, puis ouvrez le fichier.
    1. Utilisez le nom d’utilisateur `HDIFabrikam\HDIFabrikamAdmin`.
    1. Entrez le mot de passe que vous avez choisi pour le compte Administrateur.
    1. Sélectionnez **OK**.

1. Dans le tableau de bord **Gestionnaire de serveur** du contrôleur de domaine, accédez à **Outils** > **Utilisateurs et ordinateurs Active Directory**.

    ![Dans le tableau de bord Gestionnaire de serveur, ouvrez Gestion Active Directory.](./media/apache-domain-joined-create-configure-enterprise-security-cluster/server-manager-active-directory-screen.png)

1. Créez deux utilisateurs : **HDIAdmin** et **HDIUser**. Ces deux utilisateurs vous permettront de vous connecter à des clusters HDInsight.

    1. Dans la page **Utilisateurs et ordinateurs Active Directory**, cliquez avec le bouton droit sur `HDIFabrikam.com`, puis accédez à **Nouveau** > **Utilisateur**.

        ![Créer un utilisateur Active Directory](./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-user.png)

    1. Dans la page **Nouvel objet - utilisateur**, entrez `HDIUser` pour **Prénom** et le **Nom d’ouverture de session de l’utilisateur**. Les autres champs sont renseignés automatiquement. Sélectionnez ensuite **Suivant**.

        ![Créer le premier objet d’utilisateur administrateur](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0020.png)

    1. Dans la fenêtre contextuelle qui s’affiche, entrez un mot de passe pour le nouveau compte. Sélectionnez **Le mot de passe n’expire jamais**, puis **OK** dans le message contextuel.
    1. Sélectionnez **Suivant**, puis **Terminer** pour créer le nouveau compte.
    1. Répétez les étapes ci-dessus pour créer l’utilisateur `HDIAdmin`.

        ![Créer le deuxième objet d’utilisateur administrateur](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0024.png)

1. Créez un groupe de sécurité global.

    1. Dans la page **Utilisateurs et ordinateurs Active Directory**, cliquez avec le bouton droit sur `HDIFabrikam.com`, puis accédez à **Nouveau** > **Groupe**.

    1. Entrez `HDIUserGroup` dans la zone de texte **Nom du groupe**.

    1. Sélectionnez **OK**.

    ![Créer un groupe Active Directory](./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-active-directory-group.png)

    ![Créer un objet](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0028.png)

1. Ajoutez des membres à **HDIUserGroup**.

    1. Cliquez avec le bouton droit sur **HDIUser**, puis sélectionnez **Ajouter à un groupe...** .
    1. Dans la zone de texte **Entrez les noms des objets à sélectionner**, entrez `HDIUserGroup`. Sélectionnez ensuite **OK**, puis une nouvelle fois **OK** dans la fenêtre contextuelle.
    1. Répétez les étapes précédentes pour le compte **HDIAdmin**.

        ![Ajouter le membre HDIUser au groupe HDIUserGroup](./media/apache-domain-joined-create-configure-enterprise-security-cluster/active-directory-add-users-to-group.png)

Vous venez de créer votre environnement Active Directory. Vous venez d’ajouter deux utilisateurs, ainsi qu’un groupe d’utilisateurs qui peuvent accéder au cluster HDInsight.

Les utilisateurs seront synchronisés avec Azure AD.

### <a name="create-an-azure-ad-directory"></a>Création d’un répertoire Azure AD

1. Connectez-vous au portail Azure.
1. Sélectionnez **Créer une ressource**, puis tapez `directory`. Sélectionnez **Azure Active Directory** > **Créer**.
1. Sous **Nom de l’organisation**, entrez `HDIFabrikam`.
1. Sous **Nom de domaine initial**, entrez `HDIFabrikamoutlook`.
1. Sélectionnez **Create** (Créer).

    ![Création d’un répertoire Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-new-directory.png)

### <a name="create-a-custom-domain"></a>Créer un domaine personnalisé

1. Dans votre nouvel **Azure Active Directory**, sous **Gérer**, sélectionnez **Noms de domaine personnalisés**.
1. Sélectionnez **+ Ajouter un domaine personnalisé**.
1. Sous **Nom de domaine personnalisé**, entrez `HDIFabrikam.com`, puis sélectionnez **Ajouter un domaine**.
1. Renseignez le champ [Ajouter vos informations DNS au bureau d’enregistrement de domaines](../../active-directory/fundamentals/add-custom-domain.md#add-your-dns-information-to-the-domain-registrar).

![Créer un domaine personnalisé](./media/apache-domain-joined-create-configure-enterprise-security-cluster/create-custom-domain.png)

### <a name="create-a-group"></a>Créer un groupe

1. Dans votre nouvel **Azure Active Directory**, sous **Gérer**, sélectionnez **Groupes**.
1. Sélectionnez **+ Nouveau groupe**.
1. Dans la zone de texte **Nom du groupe**, entrez `AAD DC Administrators`.
1. Sélectionnez **Create** (Créer).

## <a name="configure-your-azure-ad-tenant"></a>Configurer votre locataire Azure AD

Maintenant, vous allez configurer votre locataire Azure AD afin de pouvoir synchroniser les utilisateurs et les groupes de votre instance locale d’Active Directory avec le cloud.

Créez un administrateur de locataire Azure Active Directory.

1. Connectez-vous au portail Azure et sélectionnez le locataire Azure AD **HDIFabrikam**.

1. Accédez à **Gérer** > **Utilisateurs** > **Nouvel utilisateur**.

1. Entrez les informations suivantes pour le nouvel utilisateur :

    **Identité**

    |Propriété |Description |
    |---|---|
    |Nom d'utilisateur|Entrez `fabrikamazureadmin` dans la zone de texte. Dans la liste déroulante du nom de domaine, sélectionnez `hdifabrikam.com`|
    |Nom| Entrez `fabrikamazureadmin`.|

    **Mot de passe**
    1. Sélectionnez **Me permettre de créer le mot de passe**.
    1. Entrez le mot de passe sécurisé de votre choix.

    **Groupes et rôles**
    1. Sélectionnez **0 groupes sélectionnés**.
    1. Choisissez **Administrateurs AAD DC**, puis **Sélectionner**.

    ![Boîte de dialogue Groupes Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-group-member.png)

    1. Sélectionnez **Utilisateur**.
    1. Choisissez **Administrateur général**, puis **Sélectionner**.

    ![Boîte de dialogue Rôle Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/azure-ad-add-role-member.png)

1. Sélectionnez **Create** (Créer).

1. Demandez ensuite au nouvel utilisateur de se connecter au portail Azure où il sera invité à modifier le mot de passe. Vous devez effectuer cette opération avant de configurer Microsoft Azure Active Directory Connect.

## <a name="sync-on-premises-users-to-azure-ad"></a>Synchroniser les utilisateurs locaux avec Azure AD

### <a name="configure-microsoft-azure-active-directory-connect"></a>Configurer Microsoft Azure Active Directory Connect

1. À partir du contrôleur de domaine, téléchargez [Microsoft Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594).

1. Ouvrez le fichier exécutable que vous avez téléchargé, puis acceptez les termes du contrat de licence. Sélectionnez **Continuer**.

1. Cliquez sur **Utiliser la configuration rapide**.

1. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe de l’administrateur général pour l’instance Azure AD. Utilisez le nom d’utilisateur `fabrikamazureadmin@hdifabrikam.com` que vous avez créé lorsque vous avez configuré votre locataire Active Directory. Sélectionnez ensuite **Suivant**.

    ![Page « Connexion à Azure AD ».](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0058.png)

1. Dans la page **Connexion à Active Directory Domain Services**, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur d’entreprise. Utilisez le nom d’utilisateur `HDIFabrikam\HDIFabrikamAdmin` et le mot de passe que vous avez créés précédemment. Sélectionnez ensuite **Suivant**.

   ![Page « Connexion à ADDS ».](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0060.png)
1. Dans la page **Configuration de la connexion à Azure AD**, sélectionnez **Suivant**.
   ![Page « Configuration de la connexion à Azure AD »](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0062.png)

1. Dans la page **Prêt à configurer**, sélectionnez **Installer**.

   ![Page « Prêt à configurer »](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0064.png)

1. Dans la page **Configuration terminée**, sélectionnez **Quitter**.
   ![Page « Configuration terminée »](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0078.png)

1. Une fois la synchronisation terminée, vérifiez que les utilisateurs que vous avez créés dans le répertoire IaaS sont synchronisés avec Azure AD.
   1. Connectez-vous au portail Azure.
   1. Sélectionnez **Azure Active Directory** > **HDIFabrikam** > **Utilisateurs**.

### <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur

Créez une identité managée affectée par l’utilisateur que vous pourrez utiliser pour configurer Azure AD Domain Services (Azure AD DS). Pour plus d’informations, consultez [Créer, répertorier, supprimer ou affecter un rôle à une identité managée affectée par l’utilisateur à l’aide du portail Azure](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md).

1. Connectez-vous au portail Azure.
1. Sélectionnez **Créer une ressource**, puis tapez `managed identity`. Sélectionnez **Identité managée affectée par l’utilisateur** > **Créer**.
1. Pour **Nom de ressource**, entrez `HDIFabrikamManagedIdentity`.
1. Sélectionnez votre abonnement.
1. Sous **Groupe de ressources**, sélectionnez **Créer**, puis entrez `HDIFabrikam-CentralUS`.
1. Sous **Emplacement**, sélectionnez **USA Centre**.
1. Sélectionnez **Create** (Créer).

![Créer une identité managée affectée par l’utilisateur](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0082.png)

### <a name="enable-azure-ad-ds"></a>Activer Azure AD DS

Effectuez les étapes suivantes pour activer Azure AD DS. Pour plus d’informations, consultez [Activer Azure AD DS à l’aide du portail Azure](../../active-directory-domain-services/tutorial-create-instance.md).

1. Créez un réseau virtuel pour héberger Azure AD DS. Exécutez le code PowerShell suivant.

    ```powershell
    # Sign in to your Azure subscription
    $sub = Get-AzSubscription -ErrorAction SilentlyContinue
    if(-not($sub))
    {
        Connect-AzAccount
    }

    # If you have multiple subscriptions, set the one to use
    # Select-AzSubscription -SubscriptionId "<SUBSCRIPTIONID>"
    
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS' -Location 'Central US' -Name 'HDIFabrikam-AADDSVNET' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'AADDS-subnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Connectez-vous au portail Azure.
1. Sélectionnez **Créer une ressource**, entrez `Domain services`, puis sélectionnez **Azure AD Domain Services** > **Créer**.
1. Dans la page **Informations de base** :
    1. Sous **Nom de l’annuaire**, sélectionnez l’annuaire Azure AD que vous avez créé : **HDIFabrikam**.
    1. Dans **Nom du domaine DNS**, entrez *HDIFabrikam.com*.
    1. Sélectionnez votre abonnement.
    1. Spécifiez le groupe de ressources **HDIFabrikam-CentralUS**. Pour **Emplacement**, sélectionnez **USA Centre**.

        ![Informations de base Azure AD DS](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0084.png)

1. Dans la page **Réseau**, sélectionnez le réseau (**HDIFabrikam-VNET**) et le sous-réseau (**AADDS-subnet**) que vous avez créés à l’aide du script PowerShell. Vous pouvez aussi choisir **Créer** pour créer un réseau virtuel.

    ![Étape « Créer un réseau virtuel »](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0086.png)

1. Dans la page **Groupe d’administrateurs**, vous devez voir une notification indiquant qu’un groupe nommé **Administrateurs AAD DC** a déjà été créé pour administrer ce groupe. Vous pouvez modifier l’appartenance de ce groupe si vous le souhaitez. Toutefois, dans ce cas, vous n’avez pas besoin de le modifier. Sélectionnez **OK**.

    ![Voir le groupe d’administrateurs Azure AD](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0088.png)

1. Dans la page **Synchronisation**, permettez à la synchronisation de s’effectuer complètement en sélectionnant **Tout** > **OK**.

    ![Activer la synchronisation Azure AD DS](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0090.png)

1. Dans la page **Résumé**, vérifiez les détails concernant Azure AD DS, puis sélectionnez **OK**.

    ![Récapitulatif de l’activation d’Azure AD Domain Services](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0092.png)

Une fois que vous avez activé Azure AD DS, un serveur DNS local s’exécute sur les machines virtuelles Azure AD.

### <a name="configure-your-azure-ad-ds-virtual-network"></a>Configurer votre réseau virtuel Azure AD DS

Utilisez les étapes suivantes pour configurer votre réseau virtuel Azure AD DS (**HDIFabrikam-AADDSVNET**) afin qu’il utilise vos serveurs DNS personnalisés.

1. Recherchez les adresses IP de vos serveurs DNS personnalisés.
    1. Sélectionnez la ressource Azure AD DS `HDIFabrikam.com`.
    1. Sous **Gérer**, sélectionnez **Propriétés**.
    1. Les adresses IP se trouvent sous **Adresse IP sur le réseau virtuel**.

    ![Rechercher les adresses IP DNS personnalisées pour Azure AD DS](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0096.png)

1. Configurez **HDIFabrikam-AADDSVNET** pour utiliser les adresses IP personnalisées 10.0.0.4 et 10.0.0.5.

    1. Sous **Paramètres**, sélectionnez **Serveurs DNS**.
    1. Sélectionnez **Personnalisé**.
    1. Dans la zone de texte, entrez la première adresse IP (*10.0.0.4*).
    1. Sélectionnez **Enregistrer**.
    1. Répétez ces étapes pour ajouter l’autre adresse IP (*10.0.0.5*).

Dans notre scénario, nous avons configuré Azure AD DS pour utiliser les adresses IP 10.0.0.4 et 10.0.0.5, en définissant la même adresse IP sur le réseau virtuel Azure AD DS :

![Page Serveurs DNS personnalisés](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0098.png)

## <a name="securing-ldap-traffic"></a>Sécurisation du trafic LDAP

Le protocole LDAP (Lightweight Directory Access Protocol) est utilisé pour la lecture et l’écriture sur Azure Active Directory. Vous pouvez rendre le trafic LDAP confidentiel et sécurisé à l’aide de la technologie SSL (Secure Sockets Layer) ou TLS (Transport Layer Security). Vous pouvez activer le protocole LDAP sur SSL (LDAPS) en installant un certificat au format correct.

Pour plus d’informations sur le trafic LDAP sécurisé, consultez [Configurer le protocole LDAP sécurisé pour un domaine managé Azure AD DS](../../active-directory-domain-services/tutorial-configure-ldaps.md).

Dans cette section, vous allez créer un certificat auto-signé, télécharger le certificat et configurer le protocole LDAP sécurisé (LDAPS) pour le domaine managé Azure AD DS **HDIFabrikam**.

Le script suivant crée un certificat pour **HDIFabrikam**. Le certificat est enregistré sous le chemin *LocalMachine*.

```powershell
$lifetime = Get-Date
New-SelfSignedCertificate -Subject hdifabrikam.com `
-NotAfter $lifetime.AddDays(365) -KeyUsage DigitalSignature, KeyEncipherment `
-Type SSLServerAuthentication -DnsName *.hdifabrikam.com, hdifabrikam.com
```

> [!NOTE]  
> Tous les utilitaires ou applications qui créent une requête PKCS \#10 valide peuvent être utilisés pour former la demande de certificat TLS/SSL.

Vérifiez que le certificat se trouve dans le magasin **personnel** de l’ordinateur :

1. Démarrez Microsoft Management Console (MMC).
1. Ajoutez le composant logiciel enfichable **Certificats** qui gère les certificats sur l’ordinateur local.
1. Développez **Certificats (Ordinateur local)**  > **Personnel** > **Certificats**. Un nouveau certificat doit se trouver dans le magasin **personnel**. Ce certificat est émis avec le nom d’hôte complet.

    ![Vérifier la création du certificat local](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0102.png)

1. Dans le volet de droite, cliquez avec le bouton droit sur le certificat que vous avez créé. Pointez sur **Toutes les tâches**, puis sélectionnez **Exporter**.

1. Dans la page **Exporter la clé privée**, sélectionnez **Oui, exporter la clé privée**. L’ordinateur sur lequel la clé privée sera importée a besoin d’elle pour lire les messages chiffrés.

    ![Page Exporter la clé privée de l’Assistant Exportation de certificat](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0103.png)

1. Dans la page **Format de fichier d’exportation**, laissez les valeurs par défaut, puis sélectionnez **Suivant**.
1. Dans la page **Mot de passe**, tapez un mot de passe pour la clé privée. Pour **Chiffrement**, sélectionnez **TripleDES-SHA1**. Sélectionnez ensuite **Suivant**.
1. Dans la page **Fichier à exporter**, tapez le chemin et le nom du fichier de certificat exporté, puis sélectionnez **Suivant**. Le nom de fichier doit porter l’extension .pfx. Ce fichier est configuré dans le portail Azure pour établir une connexion sécurisée.
1. Activez le protocole LDAP sécurisé pour un domaine managé Azure AD DS.
    1. Dans le portail Azure, sélectionnez le domaine `HDIFabrikam.com`.
    1. Sous **Gérer**, sélectionnez **LDAP sécurisé**.
    1. Dans la page **LDAP sécurisé**, sous **LDAP sécurisé**, sélectionnez **Activer**.
    1. Recherchez le fichier de certificat .pfx que vous avez exporté sur votre ordinateur.
    1. Saisissez le mot de passe du certificat.

    ![Activer LDAP sécurisé](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0113.png)

1. Maintenant que vous avez activé le protocole LDAP sécurisé (LDAPS), rendez-le accessible en activant le port 636.
    1. Dans le groupe de ressources **HDIFabrikam-CentralUS**, sélectionnez le groupe de sécurité réseau **AADDS-HDIFabrikam.com-NSG**.
    1. Sous **Paramètres**, sélectionnez **Règles de sécurité de trafic entrant** > **Ajouter**.
    1. Dans la page **Ajouter une règle de sécurité de trafic entrant**, entrez les propriétés suivantes, puis sélectionnez **Ajouter** :

        | Propriété | Valeur |
        |---|---|
        | Source | Quelconque |
        | Source port ranges | * |
        | Destination | Quelconque |
        | Destination port range | 636 |
        | Protocol | Quelconque |
        | Action | Allow |
        | Priority | \<Desired number> |
        | Nom | Port_LDAP_636 |

    ![Boîte de dialogue « Ajouter une règle de sécurité de trafic entrant »](./media/apache-domain-joined-create-configure-enterprise-security-cluster/add-inbound-security-rule.png)

**HDIFabrikamManagedIdentity** est l’identité managée affectée par l’utilisateur. Le rôle Contributeur aux services de domaine HDInsight est associé à cette identité managée qui pourra donc lire, créer, modifier et supprimer des opérations de services de domaine.

![Créer une identité managée attribuée par l’utilisateur](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0117.png)

## <a name="create-an-esp-enabled-hdinsight-cluster"></a>Créer un cluster ESP HDInsight

Cette étape nécessite les prérequis suivants :

1. Créez un groupe de ressources *HDIFabrikam-WestUS* dans la région **USA Ouest**.
1. Créez un réseau virtuel qui hébergera le cluster ESP HDInsight.

    ```powershell
    $virtualNetwork = New-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS' -Location 'West US' -Name 'HDIFabrikam-HDIVNet' -AddressPrefix 10.1.0.0/16
    $subnetConfig = Add-AzVirtualNetworkSubnetConfig -Name 'SparkSubnet' -AddressPrefix 10.1.0.0/24 -VirtualNetwork $virtualNetwork
    $virtualNetwork | Set-AzVirtualNetwork
    ```

1. Créez une relation d’appairage entre le réseau virtuel qui héberge Azure AD DS (`HDIFabrikam-AADDSVNET`) et celui qui héberge le cluster ESP HDInsight (`HDIFabrikam-HDIVNet`). Utilisez le code PowerShell suivant pour appairer ces deux réseaux virtuels.

    ```powershell
    Add-AzVirtualNetworkPeering -Name 'HDIVNet-AADDSVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS')

    Add-AzVirtualNetworkPeering -Name 'AADDSVNet-HDIVNet' -RemoteVirtualNetworkId (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-WestUS').Id -VirtualNetwork (Get-AzVirtualNetwork -ResourceGroupName 'HDIFabrikam-CentralUS')
    ```

1. Créez un compte Azure Data Lake Storage Gen2 nommé **Hdigen2store**. Configurez le compte avec l’identité gérée par l’utilisateur **HDIFabrikamManagedIdentity**. Pour plus d’informations, consultez [Utiliser Azure Data Lake Storage Gen2 avec des clusters Azure HDInsight](../hdinsight-hadoop-use-data-lake-storage-gen2.md).

1. Configurez un serveur DNS personnalisé sur le réseau virtuel **HDIFabrikam-AADDSVNET**.
    1. Dans le Portail Azure, accédez à > **Groupes de ressources** > **OnPremADVRG** > **HDIFabrikam-AADDSVNET** > **Serveurs DNS**.
    1. Sélectionnez **Personnalisé**, puis entrez *10.0.0.4* et *10.0.0.5*.
    1. Sélectionnez **Enregistrer**.

        ![Enregistrer les paramètres DNS personnalisés pour un réseau virtuel](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0123.png)

1. Créez un cluster Spark HDInsight ESP.
    1. Sélectionnez **Personnalisé (taille, paramètres, applications)** .
    1. Entrez les informations nécessaires dans **Informations de base** (section 1). Vérifiez que le **Type de cluster** est bien **Spark 2.3 (HDI 3.6)** . Vérifiez que le **Groupe de ressources** est bien **HDIFabrikam-CentralUS**.

    1. Dans **Sécurité + réseau** (section 2), renseignez les informations suivantes :
        * Sous **Pack Sécurité Entreprise**, sélectionnez **Activé**.
        * Sélectionnez **Utilisateur administrateur de cluster**, puis sélectionnez le compte **HDIAdmin** que vous avez créé pour l’utilisateur administrateur local. Cliquez sur **Sélectionner**.
        * Sélectionnez **Groupe d’accès au cluster** > **HDIUserGroup**. Tout utilisateur que vous ajouterez à ce groupe sera en mesure d’accéder aux clusters HDInsight.

            ![Sélection du groupe d’accès au cluster HDIUserGroup](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0129.jpg)

    1. Effectuez les autres étapes de la configuration du cluster et vérifiez les détails dans le **Résumé du cluster**. Sélectionnez **Create** (Créer).

1. Connectez-vous à l’interface utilisateur Ambari pour le cluster nouvellement créé à l’adresse `https://CLUSTERNAME.azurehdinsight.net`. Utilisez le nom d’utilisateur d’administrateur `hdiadmin@hdifabrikam.com` et son mot de passe.

    ![Fenêtre de connexion à l’interface utilisateur Apache Ambari](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0135.jpg)

1. Dans le tableau de bord du cluster, sélectionnez **Rôles**.
1. Dans la page **Rôles**, sous **Affecter des rôles à**, à côté du rôle **Administrateur de cluster**, entrez le groupe *hdiusergroup*. 

    ![Affecter le rôle d’administrateur de cluster à hdiusergroup](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0137.jpg)

1. Ouvrez votre client Secure Shell (SSH) et connectez-vous au cluster. Utilisez l’utilisateur **hdiuser** que vous avez créé dans l’instance locale d’Active Directory.

    ![Se connecter au cluster à l’aide du client SSH](./media/apache-domain-joined-create-configure-enterprise-security-cluster/hdinsight-image-0139.jpg)

Si vous parvenez à vous connecter avec ce compte, c’est que vous avez configuré votre cluster ESP correctement pour la synchronisation avec votre instance locale d’Active Directory.

## <a name="next-steps"></a>Étapes suivantes

Lisez [Introduction à la sécurité Apache Hadoop avec l’ESP](hdinsight-security-overview.md).
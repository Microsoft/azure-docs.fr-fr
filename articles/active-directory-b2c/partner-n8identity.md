---
title: Tutoriel de configuration de N8 Identity avec Azure Active Directory B2C
titleSuffix: Azure AD B2C
description: Tutoriel pour la configuration de l’outil d’administration TheAccessHub avec Azure Active Directory B2C afin de gérer la migration des comptes client et l’administration des demandes de service client.
services: active-directory-b2c
author: gargi-sinha
manager: martinco
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 10/26/2020
ms.author: gasinh
ms.subservice: B2C
ms.openlocfilehash: 337275cef0f2159cb5fac40ac0435408baf3bbef
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96170920"
---
# <a name="tutorial-for-configuring-theaccesshub-admin-tool-with-azure-active-directory-b2c"></a>Tutoriel pour la configuration de l’outil d’administration TheAccessHub avec Azure Active Directory B2C

Dans cet exemple de tutoriel, nous fournissons des conseils sur la façon d’intégrer Azure Active Directory (AD) B2C à l’[outil d’administration TheAccessHub](https://n8id.com/products/theaccesshub-admintool/) à partir de N8 Identity. Cette solution traite de la migration des comptes client et de l’administration des demandes de service client.  

Cette solution vous convient si vous avez au moins l’un des besoins suivants :

- Vous avez un site existant et vous souhaitez migrer vers Azure AD B2C. Cependant, la migration de vos comptes client et notamment des mots de passe vous pose problème.

- Vous avez besoin d’un outil pour permettre à votre représentant du service client de gérer les comptes Azure AD B2C.

- Vous avez l’obligation d’utiliser une administration déléguée pour vos demandes de service client.

- Vous souhaitez synchroniser et fusionner vos données de plusieurs référentiels dans Azure AD B2C.

## <a name="pre-requisites"></a>Conditions préalables

Avant de commencer, vérifiez que vous disposez des éléments suivants :

- Un abonnement Azure AD Si vous ne disposez d’aucun abonnement, vous pouvez obtenir [un compte gratuit](https://azure.microsoft.com/free/).

- Un [locataire Azure AD B2C](./tutorial-create-tenant.md). Le locataire doit être lié à votre abonnement Azure.

- Un environnement d’outil d’administration TheAccessHub : contactez [N8 Identity](https://n8id.com/contact/) pour configurer un nouvel environnement.

- [Facultatif] Informations de connexion et d’identification pour toutes les bases de données ou protocoles LDAP à partir desquels vous voulez migrer les données client.

- [Facultatif] Un environnement Azure AD B2C configuré pour utiliser des [stratégies personnalisées](./custom-policy-get-started.md), si vous souhaitez intégrer l’outil d’administration TheAccessHub à votre flux de stratégie d’inscription.

## <a name="scenario-description"></a>Description du scénario

L’outil d’administration TheAccessHub s’exécute comme n’importe quelle autre application dans Azure. Il peut être exécuté soit dans l’abonnement Azure de N8 Identity, soit dans l’abonnement du client. Le diagramme d’architecture suivant illustre l’implémentation.

![Image montrant le diagramme de l’architecture n8identity](./media/partner-n8identity/n8identity-architecture-diagram.png)

|Étape | Description |
|:-----| :-----------|
| 1. | L’utilisateur accède à une page de connexion. Les utilisateurs sélectionnent l’option d’inscription pour créer un compte et entrer des informations sur la page. Azure AD B2C collecte les attributs de l’utilisateur.
| 2. | Azure AD B2C appelle l’outil d’administration TheAccessHub et transmet les attributs de l’utilisateur
| 3. | L’outil d’administration TheAccessHub recherche les informations de l’utilisateur actuel dans votre base de données.  
| 4. | L’enregistrement du client est synchronisé de la base de données vers l’outil d’administration TheAccessHub.
| 5. | L’outil d’administration TheAccessHub partage les données avec l’administrateur du service client/support technique délégué.
| 6. | L’outil d’administration TheAccessHub synchronise les enregistrements de l’utilisateur avec Azure AD B2C.
| 7. |En fonction de la réponse de l’outil d’administration TheAccessHub, qui indique la réussite ou l’échec de l’opération, Azure AD B2C envoie un e-mail personnalisé à l’utilisateur.

## <a name="create-a-global-admin-in-your-azure-ad-b2c-tenant"></a>Créer un administrateur général dans votre locataire Azure AD B2C

L’outil d’administration TheAccessHub requiert des autorisations pour agir au nom d’un administrateur général afin de lire les informations utilisateur et effectuer des modifications dans votre locataire Azure AD B2C. Les modifications que vous pourrez apporter à vos administrateurs normaux n’affecteront par la capacité de l’outil d’administration TheAccessHub à interagir avec le locataire.

Pour créer un administrateur général, procédez comme suit :

1. Dans le portail Azure, connectez-vous à votre locataire Azure AD B2C en tant qu’administrateur. Accédez à **Azure Active Directory** > **Utilisateurs**
2. Sélectionnez **Nouvel utilisateur**
3. Sélectionnez **Créer un utilisateur** pour créer un utilisateur de répertoire normal et non un client
4. Remplissez le formulaire dédié à l’identité

   a. Entrez le nom d’utilisateur, par exemple « theaccesshub »

   b. Entrez le nom, par exemple « TheAccessHub Service Account »

5. Sélectionnez **Afficher le mot de passe** et copiez le mot de passe initial pour une utilisation ultérieure
6. Affectez le rôle d’administrateur général

   a. Sélectionnez le rôle actuel **Utilisateur** de l’utilisateur pour le modifier

   b. Vérifiez l’enregistrement pour Administrateur général

   c. **Sélectionnez** > **Créer**

## <a name="connect-theaccesshub-admin-tool-with-your-azure-ad-b2c-tenant"></a>Connecter l’outil d’administration TheAccessHub à votre locataire Azure AD B2C

L’outil d’administration TheAccessHub utilise l’API Graph de Microsoft pour lire et modifier votre annuaire. Il agit comme un administrateur général dans votre locataire. Des autorisations supplémentaires sont requises par l’outil d’administration TheAccessHub. Vous pouvez les octroyer au sein même de l’outil.

Pour autoriser l’outil d’administration TheAccessHub à accéder à votre annuaire, procédez comme suit :

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Configuration Azure AD B2C**

3. Sélectionnez **Autoriser la connexion**

4. Dans la nouvelle fenêtre, connectez-vous avec votre compte d’administrateur général. Vous serez peut-être invité à réinitialiser votre mot de passe si c’est la première fois que vous vous connectez avec le nouveau compte de service.

5. Suivez les invites et sélectionnez **Accepter** pour octroyer les autorisations requises à l’outil d’administration TheAccessHub.

## <a name="configure-a-new-csr-user-using-your-enterprise-identity"></a>Configurer un nouvel utilisateur de service client à l’aide de votre identité d’entreprise

Créez un utilisateur de service client/support technique qui peut accéder à l’outil d’administration TheAccessHub à l’aide de ses informations d’identification d’entreprise Azure Active Directory existantes.

Pour configurer l’utilisateur de service client/support technique avec l’authentification unique, les étapes suivantes sont recommandées :

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification fournies par N8 Identity.

2. Accédez à **Outils du gestionnaire** > **Gérer les collègues**

3. Sélectionnez **Ajouter un collègue**

4. Sélectionnez **Type de collègue Administrateur Azure**

5. Entrez les informations de profil du collègue

   a. Vous pouvez sélectionner une Organisation d’origine, qui contrôlera les personnes autorisées à gérer cet utilisateur.

   b. Pour ID de connexion/Nom d’utilisateur Azure AD, indiquez le nom d’utilisateur principal à partir du compte Azure Active Directory de l’utilisateur.

   c. Dans l’onglet Rôles TheAccessHub, cochez le rôle géré Support technique. L’utilisateur pourra ainsi accéder à la vue de gestion des collègues. L’utilisateur devra cependant être placé dans un groupe ou désigné comme propriétaire d’organisation pour agir sur les clients.

6. Sélectionnez **Envoyer**.

## <a name="configure-a-new-csr-user-using-a-new-identity"></a>Configurer un nouvel utilisateur de service client à l’aide d’une nouvelle identité

Créez un utilisateur de service client/support technique qui peut accéder à l’outil d’administration TheAccessHub avec un nouvel identifiant local propre à cet outil. Cette méthode sert principalement aux organisations qui n’utilisent pas d’annuaire Azure AD pour leur entreprise.

Pour [configurer un utilisateur de service client/support technique](https://youtu.be/iOpOI2OpnLI) sans authentification unique, procédez comme suit :

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification fournies par N8 Identity

2. Accédez à **Outils du gestionnaire** > **Gérer les collègues**

3. Sélectionnez **Ajouter un collègue**

4. Sélectionnez **Type de collègue Administrateur local**

5. Entrez les informations de profil du collègue

   a. Vous pouvez sélectionner une Organisation d’origine, qui contrôlera les personnes autorisées à gérer cet utilisateur.

   b. Dans l’onglet **Rôles TheAccessHub**, cochez le rôle géré **Support technique**. L’utilisateur pourra ainsi accéder à la vue de gestion des collègues. L’utilisateur devra cependant être placé dans un groupe ou désigné comme propriétaire d’organisation pour agir sur les clients.

6. Copiez les attributs **ID/adresse e-mail de connexion** et **Mot de passe à usage unique**. Transmettez-les au nouvel utilisateur. Il utilisera ces informations d’identification pour se connecter à l’outil d’administration TheAccessHub. L’utilisateur sera invité à entrer un nouveau mot de passe lors de sa première connexion.

7. Sélectionnez **Envoyer**

## <a name="configure-partitioned-csr-administration"></a>Configurer une administration de service client partitionnée

Les autorisations pour gérer les clients et les utilisateurs de service client/support technique dans l’outil d’administration TheAccessHub sont gérées à l’aide d’une hiérarchie d’organisation. Tous les collègues et clients résident dans une organisation d’origine. Certains collègues ou groupes de collègues peuvent être désignés comme propriétaires d’organisation.  Les propriétaires d’organisation peuvent gérer (modifier) les collègues et les clients dans les organisations ou sous-organisations dont ils sont propriétaires. Pour permettre à plusieurs collègues de gérer un ensemble d’utilisateurs, un groupe peut être créé. Le groupe peut ensuite être désigné comme propriétaire d’organisation. Ainsi, tous les membres du groupe peuvent gérer les collègues et les clients de l’organisation.

### <a name="create-a-new-group"></a>Créer un groupe

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des **informations d’identification** qui vous sont fournies par N8 Identity

2. Accédez à **Organisation > Gérer les groupes**

3. Sélectionnez > **Ajouter un groupe**

4. Entrez un **nom**, une **description** et un **propriétaire** pour le nouveau groupe

5. Recherchez les collègues que vous voulez ajouter comme membres du groupe et cochez les cases correspondantes, puis sélectionnez >**Ajouter**

6. Tous les membres du groupe sont affichés en bas de la page.

7. Vous pouvez supprimer des membres du groupe en sélectionnant le symbole **x** situé au bout de la ligne

8. Sélectionnez **Envoyer**

### <a name="create-a-new-organization"></a>Créer une organisation

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à Organisation > **Gérer les organisations**

3. Sélectionnez > **Ajouter une organisation**

4. Indiquez un **nom d’organisation**, un **propriétaire d’organisation** et une **organisation parente**.

    a. Le nom de l’organisation est idéalement une valeur qui correspond à vos données client. Lorsque vous chargez les données d’un collègue ou d’un client, si vous indiquez le nom de l’organisation dans la charge, le collègue peut être automatiquement placé dans l’organisation.

    b. Le propriétaire représente la personne ou le groupe qui devra gérer les clients et les collègues dans cette organisation et ses éventuelles sous-organisations.

    c. L’organisation parente indique quelle autre organisation est, par nature, responsable de cette organisation.

5. Sélectionnez **Envoyer**.

### <a name="modify-the-hierarchy-via-the-tree-view"></a>Modifier la hiérarchie à l’aide de l’arborescence

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Outils du gestionnaire** > **Arborescence**

3. Cette représentation vous permet de visualiser les collègues et les groupes qui peuvent gérer des organisations.

4. Les hiérarchies peuvent être modifiées en faisant glisser les organisations au-dessus de celles dont vous voulez qu’elles dépendent.

5. Sélectionnez **Enregistrer** lorsque vous avez fini de modifier la hiérarchie.

## <a name="customize-welcome-notification"></a>Personnaliser la notification d’accueil

Si vous utilisez TheAccessHub pour migrer des utilisateurs d’une précédente solution d’authentification vers Azure AD B2C, vous pouvez personnaliser la notification d’accueil, qui est envoyée à l’utilisateur par TheAccessHub lors de la migration. Ce message comprend généralement le lien permettant au client de définir un nouveau mot de passe dans le répertoire Azure AD B2C.

Pour personnaliser la notification :

1. Connectez-vous à TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Notifications**

3. Sélectionnez le **modèle Créer un collègue**

4. Sélectionnez **Modifier**

5. Modifiez les champs Message et Modèle en fonction de vos besoins. Le champ Modèle prend en charge le langage HTML et peut envoyer des notifications au format HTML aux adresses e-mail des clients.

6. Lorsque vous avez terminé, sélectionnez **Enregistrer**.

## <a name="migrate-data-from-external-data-sources-to-azure-ad-b2c"></a>Migrer des données de sources de données externes vers Azure AD B2C

À l’aide de l’outil d’administration TheAccessHub, vous pouvez importer des données de différentes bases de données, protocoles LDAP et fichiers CSV, puis envoyer (push) ces données vers votre locataire Azure AD B2C. Pour cela, les données sont chargées dans le type de collègue Utilisateur Azure AD B2C au sein de l’outil d’administration TheAccessHub.  Si la source des données n’est pas Azure, celles-ci sont placées à la fois dans TheAccessHub et dans Azure AD B2C. Si la source de vos données externes n’est pas un simple fichier .csv sur votre ordinateur, configurez une source de données avant de procéder au chargement des données. Les étapes ci-dessous décrivent la création d’une source de données et le chargement des données.

### <a name="configure-a-new-data-source"></a>Configurer une nouvelle source de données

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Sources de données**

3. Sélectionnez **Ajouter une source de données**

4. Indiquez un **nom** et un **type** pour la source de données

5. Renseignez les données du formulaire, en fonction de votre type de source de données :

   **Pour les bases de données**

   a. **Type** : base de données

   b. **Type de base de données** : sélectionnez une base de données parmi les types de base de données pris en charge.

   c. **URL de connexion** : entrez une chaîne de connexion JDBC bien formée. Par exemple : ``jdbc:postgresql://myhost.com:5432/databasename``

   d. **Nom d’utilisateur** : entrez le nom d’utilisateur pour accéder à la base de données

   e. **Mot de passe** : entrez le mot de passe pour accéder à la base de données

   f. **Requête** : entrez la requête SQL pour extraire les informations du client. Par exemple : ``SELECT * FROM mytable;``

   g. Sélectionnez **Tester la connexion**. Vous verrez un échantillon de vos données pour vérifier que la connexion fonctionne.

   **Pour les protocoles LDAP**

   a. **Type** : LDAP

   b. **Hôte** : entrez le nom d’hôte ou l’adresse IP de l’ordinateur sur lequel s’exécute le serveur LDAP. Par exemple : ``mysite.com``

   c. **Port** : entrez le numéro de port dans lequel le serveur LDAP écoute.

   d. **SSL** : cochez la case si l’outil d’administration TheAccessHub doit communiquer avec le serveur LDAP de manière sécurisée à l’aide de SSL. L’utilisation de SSL est fortement recommandée.

   e. **Nom unique de connexion** : entrez le nom unique du compte d’utilisateur pour la connexion et la recherche LDAP

   f. **Mot de passe** : entrez le mot de passe de l’utilisateur

   g. **ND de base** : entrez le nom unique en haut de la hiérarchie dans laquelle vous voulez effectuer la recherche

   h. **Filtre** : entrez la chaîne de filtre LDAP, qui obtiendra vos enregistrements client

   i. **Attributs** : entrez une liste d’attributs séparés par des virgules à partir de vos enregistrements client à transmettre à l’outil d’administration TheAccessHub

   j. Sélectionnez **Tester la connexion**. Un échantillon de vos données vous permet de vérifier que la connexion fonctionne.

   **Pour OneDrive**

   a. **Type** : OneDrive Entreprise

   b. Sélectionnez **Autoriser la connexion**

   c. Une nouvelle fenêtre vous invite à vous connecter à **OneDrive**. Connectez-vous avec un utilisateur doté d’un accès en lecture à votre compte OneDrive. L’outil d’administration TheAccessHub agit pour que cet utilisateur lise les fichiers de charge CSV.

   d. Suivez les invites et sélectionnez **Accepter** pour octroyer les autorisations requises à l’outil d’administration TheAccessHub.

6. Lorsque vous avez terminé, sélectionnez **Enregistrer**.  

### <a name="synchronize-data-from-your-data-source-into-azure-ad-b2c"></a>Synchroniser des données de votre source de données vers Azure AD B2C

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Synchronisation de données**

3. Sélectionnez **Nouvelle charge**

4. Sélectionnez le **Type de collègue** Utilisateur Azure AD B2C

5. Sélectionnez **Source**, puis dans la boîte de dialogue contextuelle, sélectionnez votre source de données. Si vous avez créé une source de données OneDrive, sélectionnez également le fichier.

6. Si vous ne souhaitez pas créer de nouveaux comptes client avec cette charge, définissez la première stratégie : **SI le collègue est introuvable dans TheAccessHub ALORS** sur **Ne rien faire**

7. Si vous ne souhaitez pas mettre à jour des comptes client existants avec cette charge, définissez la deuxième stratégie **SI les données source et TheAccessHub ne correspondent pas ALORS** sur **Ne rien faire**

8. Sélectionnez **Suivant**.

9. Dans la **configuration Search-Mapping**, nous cherchons à mettre en corrélation les enregistrements de charge avec les clients déjà chargés dans l’outil d’administration TheAccessHub. Choisissez un ou plusieurs attributs d’identification dans la source. Faites correspondre les attributs avec un attribut de l’outil d’administration TheAccessHub qui contient les mêmes valeurs. Si une correspondance est trouvée, l’enregistrement existant est remplacé. Dans le cas contraire, un client est créé. Vous pouvez séquencer un certain nombre de ces vérifications. Par exemple, vous pouvez commencer par vérifier l’adresse e-mail, puis le prénom et le nom.

10. Dans le menu de gauche, sélectionnez **Mappage de données** .

11. Dans la configuration Data-Mapping, définissez les attributs de l’outil d’administration TheAccessHub qui doivent être renseignés à partir de vos attributs source. Il n’est pas nécessaire de mapper tous les attributs. Les attributs non mappés restent inchangés pour les clients existants.

12. Si vous mappez sur l’attribut org_name avec une valeur qui est le nom d’une organisation existante, les nouveaux clients créés sont placés dans cette organisation.

13. Sélectionnez **Suivant**.

14. Une planification Quotidienne/hebdomadaire ou Mensuelle peut être spécifiée si cette charge doit être récurrente. Sinon, conservez la valeur par défaut **Maintenant**.

15. Sélectionnez **Envoyer**

16. Si l’option **Maintenant** a été sélectionnée, un nouvel enregistrement est immédiatement ajouté à l’écran Synchronisations de données. Une fois que la phase de validation a atteint 100 %, sélectionnez le **nouvel enregistrement** pour voir le résultat attendu pour la charge. Pour les charges planifiées, ces enregistrements apparaissent uniquement après l’heure planifiée.

17. S’il n’y a pas d’erreur, sélectionnez **Exécuter** pour valider les modifications. Autrement, sélectionnez **Supprimer** dans le menu **Plus** pour supprimer la charge. Vous pouvez ensuite corriger les données source ou les mappages de charge, puis réessayer. S’il n’y a que peu d’erreurs, vous pouvez modifier manuellement les enregistrements et sélectionner **Mettre à jour** sur chaque enregistrement pour apporter des corrections. Enfin, vous pouvez continuer à traiter les erreurs et les résoudre plus tard en tant que **Interventions de support** dans l’outil d’administration TheAccessHub.

18. Lorsque l’enregistrement **Synchronisation des données** atteint 100 % dans la phase de chargement, toutes les modifications résultant de la charge ont été lancées. Les clients doivent commencer à apparaître ou à recevoir des modifications dans Azure AD B2C.

## <a name="synchronize-azure-ad-b2c-customer-data"></a>Synchroniser des données client d’Azure AD B2C 

Dans le cadre d’une opération unique ou en cours, l’outil d’administration TheAccessHub peut synchroniser toutes les informations client d’Azure AD B2C vers l’outil de synchronisation TheAccessHub. Cela permet de s’assurer que les administrateurs du service client/support technique voient des informations client à jour.

Pour synchroniser des données d’Azure AD B2C vers l’outil d’administration TheAccessHub :

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Synchronisation de données**

3. Sélectionnez **Nouvelle charge**

4. Sélectionnez le **Type de collègue** Utilisateur Azure AD B2C

5. Pour l’étape **Options**, laissez les valeurs par défaut.

6. Sélectionnez **Suivant**.

7. Pour l’étape **Recherche et mappage des données**, laissez les valeurs par défaut. Si vous mappez sur l’attribut **org_name** avec une valeur qui est le nom d’une organisation existante, les nouveaux clients créés sont placés dans cette organisation.

8. Sélectionnez **Suivant**.

9. Une planification Quotidienne/hebdomadaire ou Mensuelle peut être spécifiée si cette charge doit être récurrente. Sinon, conservez la valeur par défaut : **Maintenant**. Nous vous recommandons de synchroniser régulièrement à partir d’Azure AD B2C.

10. Sélectionnez **Envoyer**

11. Si l’option **Maintenant** a été sélectionnée, un nouvel enregistrement est immédiatement ajouté à l’écran Synchronisations de données. Une fois que la phase de validation a atteint 100 %, sélectionnez le nouvel enregistrement pour voir le résultat attendu pour la charge. Pour les charges planifiées, ces enregistrements apparaissent uniquement après l’heure planifiée.

12. S’il n’y a pas d’erreur, sélectionnez **Exécuter** pour valider les modifications. Autrement, sélectionnez **Supprimer** dans le menu Plus pour supprimer la charge. Vous pouvez ensuite corriger les données source ou les mappages de charge, puis réessayer. S’il n’y a que peu d’erreurs, vous pouvez modifier manuellement les enregistrements et sélectionner **Mettre à jour** sur chaque enregistrement pour apporter des corrections. Enfin, vous pouvez continuer à traiter les erreurs et les résoudre plus tard en tant qu’Interventions de support dans l’outil d’administration TheAccessHub.

13. Lorsque l’enregistrement **Synchronisation des données** atteint 100 % dans la phase de chargement, toutes les modifications résultant de la charge ont été lancées.

## <a name="configure-azure-ad-b2c-policies"></a>Configurer des stratégies Azure AD B2C

La synchronisation occasionnelle de l’outil d’administration TheAccessHub est limitée dans sa capacité à conserver son état à jour avec Azure AD B2C. Nous pouvons tirer parti de l’API de l’outil d’administration TheAccessHub et des stratégies Azure AD B2C pour informer l’outil d’administration TheAccessHub des modifications en temps réel. Cette solution nécessite une connaissance technique des [stratégies personnalisées d’Azure AD B2C](./custom-policy-get-started.md). Dans la section suivante, nous vous donnons un exemple de création de stratégie et un certificat sécurisé pour informer l’outil d’administration TheAccessHub de la création de nouveaux comptes dans vos stratégies personnalisées d’inscription.

### <a name="create-a-secure-credential-to-invoke-theaccesshub-admin-tools-api"></a>Créer des informations d’identification sécurisées pour appeler l’API de l’outil d’administration TheAccessHub

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Outils de l’administrateur** > **Sécurité de l’API**

3. Sélectionnez **Générer**

4. Copiez le **mot de passe du certificat**

5. Sélectionnez **Télécharger** pour obtenir le certificat client.

6. Suivez ce [tutoriel](./secure-rest-api.md#https-client-certificate-authentication ) pour ajouter le certificat client à Azure AD B2C.

### <a name="retrieve-your-custom-policy-examples"></a>Récupérer vos exemples de stratégies personnalisées

1. Connectez-vous à l’outil d’administration TheAccessHub à l’aide des informations d’identification qui vous sont fournies par N8 Identity

2. Accédez à **Administrateur système** > **Outils de l’administrateur** > **Stratégies Azure B2C**

3. Indiquez votre domaine de locataire Azure AD B2C et les deux ID Identity Experience Framework de votre configuration Identity Experience Framework

4. Sélectionnez **Enregistrer**.

5. Sélectionnez **Télécharger** pour obtenir un fichier zip avec les stratégies de base qui ajoutent les clients à l’outil d’administration TheAccessHub lorsque ceux-ci s’inscrivent.

6. Suivez ce [tutoriel](./custom-policy-get-started.md) pour commencer à créer des stratégies personnalisées dans Azure AD B2C.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations, consultez les articles suivants :

- [Stratégies personnalisées dans Azure AD B2C](./custom-policy-overview.md)

- [Bien démarrer avec les stratégies personnalisées dans Azure AD B2C](./custom-policy-get-started.md?tabs=applications)
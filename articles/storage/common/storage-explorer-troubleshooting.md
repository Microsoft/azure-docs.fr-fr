---
title: Guide de résolution des problèmes de l’Explorateur de stockage Azure | Microsoft Docs
description: Vue d’ensemble des techniques de débogage pour l’Explorateur Stockage Azure
services: storage
author: Deland-Han
manager: dcscontentpm
ms.service: storage
ms.topic: troubleshooting
ms.date: 07/28/2020
ms.author: delhan
ms.openlocfilehash: 8bffe0c3871eae12f3b875a96301136d11dfc516
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92783791"
---
# <a name="azure-storage-explorer-troubleshooting-guide"></a>Guide de résolution des problèmes de l’Explorateur de stockage Azure

L’Explorateur Stockage Microsoft Azure est une application autonome qui vous permet d’utiliser facilement les données Stockage Azure sur Windows, macOS et Linux. L’application peut se connecter aux comptes de stockage hébergés sur Azure, les clouds nationaux et Azure Stack.

Ce guide résume les solutions aux problèmes couramment rencontrés dans l’Explorateur Stockage.

## <a name="azure-rbac-permissions-issues"></a>Problèmes d’autorisations Azure RBAC

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) permet une gestion d’accès très précise pour les ressources Azure en combinant des jeux d’autorisations dans des _rôles_ . Voici quelques stratégies permettant d’optimiser le fonctionnement d’Azure RBAC dans Explorateur Stockage.

### <a name="how-do-i-access-my-resources-in-storage-explorer"></a>Comment accéder à mes ressources dans l’Explorateur Stockage ?

Si vous avez des difficultés à accéder aux ressources de stockage via Azure RBAC, c’est que les rôles qui vous ont été attribués ne sont peut-être pas les bons. Les sections suivantes décrivent les autorisations que l’Explorateur Stockage vous demande actuellement pour pouvoir accéder à vos ressources de stockage. Contactez l’administrateur de votre compte Azure si vous n’êtes pas sûr de disposer des autorisations ou des rôles appropriés.

#### <a name="read-listget-storage-accounts-permissions-issue"></a>Problème d’autorisations « Lecture : Répertorier/obtenir le ou les comptes de stockage »

Vous devez être autorisé à lister les comptes de stockage. Pour obtenir cette autorisation, le rôle _Lecteur_ doit vous être attribué.

#### <a name="list-storage-account-keys"></a>Répertorier les clés de compte de stockage

L’Explorateur Stockage peut également utiliser des clés de compte pour authentifier les requêtes. Vous pouvez accéder aux clés de compte par le biais de rôles plus puissants, comme le rôle _Contributeur_ .

> [!NOTE]
> Les clés d’accès accordent des autorisations illimitées à toute personne qui les détient. Par conséquent, nous vous déconseillons de transmettre ces clés aux utilisateurs du compte. Si vous devez révoquer des clés d’accès, vous pouvez les regénérer à partir du [portail Azure](https://portal.azure.com/).

#### <a name="data-roles"></a>Rôles de données

Vous devez disposer d’au moins un rôle qui accorde l’accès en lecture aux données à partir des ressources. Par exemple, si vous voulez lister ou télécharger des objets blob, vous devez au moins détenir le rôle _Lecteur des données Blob du stockage_ .

### <a name="why-do-i-need-a-management-layer-role-to-see-my-resources-in-storage-explorer"></a>Pourquoi ai-je besoin d’un rôle d’accès à la couche de gestion pour voir mes ressources dans l’Explorateur Stockage ?

Le stockage Azure présente deux couches d’accès : _gestion_ et _données_ . Les abonnements et les comptes de stockage sont accessibles via la couche de gestion. Les conteneurs, objets blob et autres ressources de données sont accessibles via la couche de données. Par exemple, si vous souhaitez obtenir la liste de vos comptes de stockage depuis Azure, vous devez envoyer une requête au point de terminaison de gestion. Si vous voulez consulter la liste des conteneurs d’objets blob dans un compte, vous envoyez une requête au point de terminaison du service approprié.

Les rôles Azure peuvent vous octroyer des autorisations d’accès à ces couches de données ou de gestion. Le rôle Lecteur vous accorde, par exemple, un accès en lecture seule aux ressources de la couche de gestion.

À proprement parler, le rôle Lecteur ne fournit aucune autorisation d’accès à la couche de données, et il n’est pas nécessaire pour accéder à la couche de données.

L’Explorateur Stockage vous permet d’accéder facilement à vos ressources en rassemblant les informations nécessaires pour vous connecter à vos ressources Azure. Par exemple, pour afficher vos conteneurs d’objets blob, l’Explorateur Stockage envoie une demande « lister les conteneurs » au point de terminaison du service blob. Pour obtenir ce point de terminaison, l’Explorateur Stockage recherche la liste des abonnements et comptes de stockage auxquels vous avez accès. Pour rechercher vos abonnements et comptes de stockage, l’Explorateur Stockage a aussi besoin d’un accès à la couche de gestion.

Si vous ne disposez d’un rôle qui accorde une autorisation d’accès à la couche de gestion, l’Explorateur Stockage ne peut pas obtenir les informations dont il a besoin pour se connecter à la couche de données.

### <a name="what-if-i-cant-get-the-management-layer-permissions-i-need-from-my-administrator"></a>Que se passe-t-il si je n’obtiens pas les autorisations d’accès à la couche de gestion que mon administrateur doit me fournir ?

Si vous souhaitez accéder à des files d’attente ou des conteneurs d’objets BLOB, vous pouvez les attacher à ces ressources à l’aide de vos informations d’identification Azure.

1. Ouvrez la boîte de dialogue de connexion.
2. Sélectionnez Ajouter une ressource via Azure Active Directory (Azure AD). Cliquez sur Suivant.
3. Sélectionnez le compte d’utilisateur et le locataire associés à la ressource à laquelle vous effectuez l’attachement. Cliquez sur Suivant.
4. Sélectionnez le type de ressource, entrez l’URL de la ressource, puis entrez un nom d’affichage unique pour la connexion. Cliquez sur Suivant. Cliquez sur Se connecter.

Pour les autres types de ressource, nous n’avons actuellement pas de solution liée à Azure RBAC. En guise de moyen de contournement, vous pouvez demander un URI SAP à [attacher à votre ressource](../../vs-azure-tools-storage-manage-with-storage-explorer.md?tabs=linux#use-a-shared-access-signature-uri).

### <a name="recommended-azure-built-in-roles"></a>Rôles intégrés Azure recommandés

Il existe plusieurs rôles Azure intégrés qui peuvent fournir les autorisations nécessaires pour utiliser l’Explorateur Stockage. Certains de ces rôles sont les suivants :
- [Propriétaire](../../role-based-access-control/built-in-roles.md#owner) : Gérez tout, notamment l’accès aux ressources. **Remarque**  : Ce rôle vous donne accès aux clés.
- [Collaborateur](../../role-based-access-control/built-in-roles.md#contributor) : Gérez tout, à l’exception de l’accès aux ressources. **Remarque**  : Ce rôle vous donne accès aux clés.
- [Lecteur](../../role-based-access-control/built-in-roles.md#reader) : Lisez et listez les ressources.
- [Contributeur de compte de stockage](../../role-based-access-control/built-in-roles.md#storage-account-contributor) : Gérez en totalité les comptes de stockage. **Remarque**  : Ce rôle vous donne accès aux clés.
- [Propriétaire des données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-owner) : Ayez un accès total aux données et conteneurs d’objets blob du Stockage Azure.
- [Contributeur aux données Blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-contributor) : Lire, écrire et supprimer des conteneurs et objets blob du stockage Azure.
- [Lecteur des données blob du stockage](../../role-based-access-control/built-in-roles.md#storage-blob-data-reader) : Lire et répertorier des conteneurs et objets blob du stockage Azure.

## <a name="error-self-signed-certificate-in-certificate-chain-and-similar-errors"></a>Erreur : Certificat auto-signé dans la chaîne de certificats (et erreurs similaires)

Les erreurs de certificat se produisent généralement dans l’une des situations suivantes :

- L’application est connectée via un _proxy transparent_ , ce qui signifie qu’un serveur (par exemple, le serveur de votre société) intercepte le trafic HTTPS, le déchiffre, puis le chiffre à l’aide d’un certificat auto-signé.
- Vous exécutez une application qui injecte un certificat TLS/SSL auto-signé dans les messages HTTPS que vous recevez. Les antivirus et les logiciels d’inspection du trafic réseau sont des exemples de ce type d’application.

Quand l’Explorateur Stockage voit un certificat auto-signé ou non approuvé, il ne sait plus si le message HTTPS reçu a été modifié. Si vous avez une copie du certificat auto-signé, vous pouvez donner instruction à l’Explorateur Stockage de lui faire confiance en effectuant les étapes suivantes :

1. Obtenez une copie X.509 encodée en base 64 du certificat (.cer).
2. Accédez à **Modifier** > **Certificats SSL** > **Importer les certificats** , puis utilisez le sélecteur de fichiers pour rechercher, sélectionner et ouvrir le fichier .cer.

Ce problème peut aussi se produire s’il existe plusieurs certificats (racine et intermédiaire). Pour corriger cette erreur, les deux certificats doivent être ajoutés.

Si vous ne savez pas d’où provient le certificat, déterminez-le en suivant ces étapes :

1. Installez OpenSSL.
    * [Windows](https://slproweb.com/products/Win32OpenSSL.html) : n’importe quelle version légère devrait suffire.
    * Mac et Linux : normalement fourni avec votre système d’exploitation.
2. Exécutez OpenSSL.
    * Windows : ouvrez le répertoire d’installation, sélectionnez **/bin/** , puis double-cliquez sur **openssl.exe** .
    * Mac et Linux : exécutez `openssl` à partir d’un terminal.
3. Exécutez `s_client -showcerts -connect microsoft.com:443`.
4. Recherchez les certificats auto-signés. Si vous ne parvenez pas à identifier les certificats auto-signés avec certitude, notez les endroits où le sujet `("s:")` et l’émetteur `("i:")` sont identiques.
5. Chaque fois que vous trouvez un certificat auto-signé, copiez et collez tout ce qui compris entre `-----BEGIN CERTIFICATE-----` et `-----END CERTIFICATE-----` dans un nouveau fichier .cer.
6. Ouvrez l’Explorateur Stockage, puis accédez à **Modifier** > **Certificats SSL** > **Importer des certificats** . Servez-vous ensuite du sélecteur de fichiers pour rechercher, sélectionner et ouvrir les fichiers .cer que vous avez créés.

Si vous ne trouvez aucun certificat auto-signé en suivant ces étapes, contactez-nous via l’outil de commentaires. Vous pouvez aussi ouvrir l’Explorateur Stockage à partir de la ligne de commande en utilisant l’indicateur `--ignore-certificate-errors`. Quand il est ouvert avec cet indicateur, l’Explorateur Stockage ignore les erreurs de certificat.

## <a name="sign-in-issues"></a>Problèmes de connexion

### <a name="blank-sign-in-dialog-box"></a>Boîte de dialogue de connexion vide

Les boîtes de dialogue de connexion vides apparaissent le plus souvent quand les services ADFS (Active Directory Federation Services) demandent à l’Explorateur Stockage d’effectuer une redirection, ce qui n’est pas pris en charge par Electron. Pour contourner ce problème, vous pouvez essayer d’utiliser le Flux de code d’appareil pour la connexion. Pour ce faire, procédez comme suit :

1. Dans la barre d’outils verticale gauche, ouvrez **Paramètres** . Dans le panneau Paramètres, accédez à **Application** > **Se connecter** . Activez **Utiliser la connexion via le flux de code de l’appareil** .
2. Ouvrez la boîte de dialogue **Connecter** (via l’icône de fiche dans la barre verticale à gauche ou en sélectionnant **Ajouter un compte** dans le panneau de compte).
3. Choisissez l’environnement auquel vous voulez vous connecter.
4. Sélectionnez **Connexion** .
5. Suivez les instructions figurant sur le panneau.

Si vous ne parvenez pas à vous connecter au compte que vous voulez utiliser parce que votre navigateur par défaut est déjà connecté à un autre compte, effectuez l’une des tâches suivantes :

- copier manuellement le lien et le code dans une session privée de votre navigateur, ou
- copier manuellement le lien et le code dans un autre navigateur.

### <a name="reauthentication-loop-or-upn-change"></a>Boucle de réauthentification ou modification du nom UPN

Si vous êtes dans une boucle de réauthentification ou que vous avez changé le nom UPN de l’un de vos comptes, effectuez ces étapes :

1. Supprimez tous les comptes, puis fermez l’Explorateur Stockage.
2. Supprimez le dossier .IdentityService de votre ordinateur. Sur Windows, le dossier se situe dans `C:\users\<username>\AppData\Local`. Pour Mac et Linux, vous pouvez trouver le dossier à la racine de votre répertoire utilisateur.
3. Si vous exécutez Mac ou Linux, vous devez aussi supprimer l’entrée Microsoft.Developer.IdentityService du magasin de clés de votre système d’exploitation. Sur Mac, le magasin de clés est l’application *Gnome Keychain* . Dans Linux, l’application est généralement appelée _Keyring_ , mais le nom peut être différent selon votre distribution.

### <a name="conditional-access"></a>Accès conditionnel

En raison d’une limitation de la bibliothèque Azure AD utilisée par l’Explorateur Stockage, l’accès conditionnel n’est pas pris en charge quand l’Explorateur Stockage est utilisé sur Windows 10, Linux ou macOS.

## <a name="mac-keychain-errors"></a>Erreurs de trousseau Mac

Il peut arriver que le trousseau macOS entre dans un état qui s’avère problématique pour la bibliothèque d’authentification de l’Explorateur Stockage. Pour sortir le trousseau de cet état, suivez ces étapes :

1. Fermez l’Explorateur Stockage.
2. Ouvrez le trousseau (appuyez sur Commande + Barre d’espace, tapez **trousseau** , puis appuyez sur Entrée).
3. Sélectionnez le trousseau « login ».
4. Sélectionnez l’icône de cadenas pour verrouiller le trousseau. (À la fin du processus, le cadenas est fermé. Cette opération peut prendre quelques secondes, selon les applications qui sont ouvertes).

    ![Icône de cadenas](./media/storage-explorer-troubleshooting/unlockingkeychain.png)

5. Ouvrez l’Explorateur de stockage.
6. Vous obtenez un message tel que « Le hub de service souhaite accéder au trousseau ». Entrez le mot de passe de votre compte Administrateur Mac et sélectionnez **Toujours autoriser** (ou **Autoriser** si **Toujours autoriser** n’est pas proposé).
7. Essayez de vous connecter.

### <a name="general-sign-in-troubleshooting-steps"></a>Étapes générales de résolution des problèmes de connexion

* Si vous êtes sur macOS et que la fenêtre de connexion n’apparaît jamais sur la boîte de dialogue **En attente d’authentification** , essayez [ces étapes](#mac-keychain-errors).
* Redémarrez l’Explorateur Stockage.
* Si la fenêtre d’authentification est vide, patientez au moins une minute avant de fermer la boîte de dialogue d’authentification.
* Vérifiez que vos paramètres de proxy et de certificat sont correctement configurés pour votre ordinateur et pour l’Explorateur Stockage.
* Si vous exécutez Windows et que vous avez accès à Visual Studio 2019 sur la même machine et aux informations de connexion, essayez de vous connecter à Visual Studio 2019. Après une connexion réussie à Visual Studio 2019, vous pouvez ouvrir l’Explorateur Stockage et voir votre compte dans le panneau des comptes.

Si aucune de ces méthodes ne fonctionne, [signalez un problème dans GitHub](https://github.com/Microsoft/AzureStorageExplorer/issues).

### <a name="missing-subscriptions-and-broken-tenants"></a>Abonnements manquants et locataires en échec

Si vous ne parvenez pas à récupérer vos abonnements après vous être connecté, essayez l’une des méthodes de dépannage suivantes :

* Vérifiez que votre compte a accès aux abonnements attendus. Vous pouvez vérifier votre accès en vous connectant au portail de l’environnement Azure que vous essayez d’utiliser.
* Vérifiez que vous êtes connecté via l’environnement Azure approprié (Azure, Azure Chine 21Vianet, Azure Allemagne, Azure US Government ou Environnement personnalisé).
* Si vous vous trouvez derrière un serveur proxy, vérifiez que vous avez correctement configuré le proxy de l’Explorateur Stockage.
* Essayez de supprimer et de rajouter le compte.
* S’il existe un lien « Plus d’informations », examinez les messages d’erreur signalés pour les locataires en échec. Si vous ne savez pas comment répondre aux messages d’erreur, n’hésitez pas à [signaler un problème dans GitHub](https://github.com/Microsoft/AzureStorageExplorer/issues).

## <a name="cant-remove-an-attached-account-or-storage-resource"></a>Impossible de supprimer un compte ou une ressource de stockage attachés

Si vous ne pouvez pas supprimer un compte ou une ressource de stockage attachés via l’interface utilisateur, vous pouvez supprimer manuellement toutes les ressources attachées en supprimant les dossiers suivants :

* Windows : `%AppData%/StorageExplorer`
* MacOS : `/Users/<your_name>/Library/Application Support/StorageExplorer`
* Linux : `~/.config/StorageExplorer`

> [!NOTE]
> Fermez l’Explorateur Stockage avant de supprimer ces dossiers.

> [!NOTE]
> Si vous avez déjà importé des certificats SSL, sauvegardez le contenu du répertoire `certs`. Vous pourrez utiliser la sauvegarde plus tard pour réimporter vos certificats SSL.

## <a name="proxy-issues"></a>Problèmes de proxy

Dans un premier temps, vérifiez que les informations suivantes que vous avez entrées sont correctes :

* URL de proxy et numéro de port
* Nom d’utilisateur et mot de passe si le proxy les demande

> [!NOTE]
> L’Explorateur Stockage ne prend pas en charge les fichiers de configuration automatique du proxy pour la configuration des paramètres de proxy.

### <a name="common-solutions"></a>Solutions courantes

Si vous rencontrez encore des problèmes, essayez les méthodes de résolution des problèmes suivantes :

* Si vous pouvez vous connecter à Internet sans utiliser votre proxy, vérifiez que l’Explorateur Stockage fonctionne sans les paramètres de proxy activés. Si c’est le cas, le dysfonctionnement est peut-être lié à vos paramètres de proxy. Contactez votre administrateur pour identifier les problèmes.
* Vérifiez que les autres applications qui utilisent le serveur proxy fonctionnent normalement.
* Vérifiez que vous pouvez vous connecter au portail correspondant à l’environnement Azure que vous essayez d’utiliser.
* Vérifiez que vous pouvez recevoir des réponses de vos points de terminaison de service. Entrez une des URL de point de terminaison dans votre navigateur. Si vous pouvez vous connecter, vous devez recevoir InvalidQueryParameterValue ou une réponse XML équivalente.
* Si quelqu’un d’autre utilise également l’Explorateur de stockage avec votre serveur proxy, vérifiez que cette personne peut se connecter. Si c’est le cas, vous devrez peut-être contacter l’administrateur de votre serveur proxy.

### <a name="tools-for-diagnosing-issues"></a>Outils pour diagnostiquer les problèmes

Si vous disposez d’outils de mise en réseau, tels que Fiddler pour Windows, vous pouvez diagnostiquer les problèmes comme suit :

* Si vous devez passer par votre serveur proxy, vous devez peut-être configurer votre outil de mise en réseau pour vous connecter via le proxy.
* Vérifiez le numéro de port utilisé par votre outil de mise en réseau.
* Entrez l’URL de l’hôte local et le numéro de port de l’outil de mise en réseau en tant que paramètres de proxy dans l’Explorateur de stockage. Si vous faites cela correctement, votre outil réseau commence à journaliser les demandes réseau que l’Explorateur Stockage a adressées aux points de terminaison de service et de gestion. Par exemple, si vous entrez `https://cawablobgrs.blob.core.windows.net/` pour votre point de terminaison d’objet blob dans un navigateur, vous recevez une réponse qui ressemble à la suivante :

  ![Exemple de code](./media/storage-explorer-troubleshooting/4022502_en_2.png)

  Cette réponse sous-entend que la ressource existe, même si vous ne pouvez pas y accéder.

### <a name="contact-proxy-server-admin"></a>Contacter l’administrateur du serveur proxy

Si les paramètres de votre proxy sont corrects, vous devrez peut-être contacter l’administrateur de votre serveur proxy en vue de :

* Vérifier que votre proxy ne bloque pas le trafic vers les points de terminaison de gestion ou de ressources Azure.
* Vérifier le protocole d’authentification utilisé par votre serveur proxy. L’Explorateur Stockage ne prend pas actuellement en charge les proxies NTLM.

## <a name="unable-to-retrieve-children-error-message"></a>Message d’erreur indiquant qu’il est impossible de récupérer les enfants

Si vous êtes connecté à Azure par le biais d’un proxy, vérifiez que vos paramètres de proxy sont corrects. Si le propriétaire de l’abonnement ou du compte vous a accordé l’accès à une ressource, vérifiez que vous êtes habilité à lire ou à lister cette ressource.

## <a name="connection-string-doesnt-have-complete-configuration-settings"></a>La chaîne de connexion n’a pas de paramètres de configuration complets

Si vous obtenez ce message d’erreur, il est possible que vous n’avez pas les autorisations nécessaires pour obtenir les clés de votre compte de stockage. Pour vérifier que c’est le cas, accédez au portail et recherchez votre compte de stockage. Pour cela, vous pouvez cliquer avec le bouton droit sur le nœud de votre compte de stockage et sélectionner **Ouvrir dans le portail** . Accédez ensuite au panneau **Clé d’accès** . Si vous n’avez pas les autorisations permettant d’afficher les clés, vous obtenez le message : « Vous n’avez pas accès ». Pour contourner ce problème, vous pouvez soit obtenir la clé du compte auprès de quelqu’un d’autre et attacher le nom et la clé, soit demander à quelqu’un une signature d’accès partagé pour le compte de stockage et l’utiliser pour attacher le compte de stockage.

Si vous ne voyez pas les clés du compte, signalez un problème dans GitHub pour nous permettre de vous aider à le résoudre.

## <a name="error-occurred-while-adding-new-connection-typeerror-cannot-read-property-version-of-undefined"></a>Une erreur s’est produite pendant l’ajout d’une nouvelle connexion : Type d’erreur : Impossible de lire la propriété « version » non définie

Si vous obtenez ce message d’erreur pendant que vous essayez d’ajouter une connexion personnalisée, il se peut que les données de connexion stockées dans le gestionnaire d’informations d’identification local sont endommagées. Pour contourner ce problème, essayez de supprimer vos connexions locales endommagées, puis rajoutez-les :

1. Démarrez l’Explorateur Stockage. Dans le menu, accédez à **Aide** > **Activer/désactiver les outils de développement** .
2. Dans la fenêtre ouverte, sous l’onglet **Application** , accédez à **Stockage local** (côté gauche) > **file://** .
3. Selon le type de la connexion avec laquelle vous rencontrez un problème, recherchez sa clé, puis copiez sa valeur dans un éditeur de texte. La valeur est un tableau constitué de vos noms de connexion personnalisés qui se présente comme suit :
    * Comptes de stockage
        * `StorageExplorer_CustomConnections_Accounts_v1`
    * Conteneurs d’objets blob
        * `StorageExplorer_CustomConnections_Blobs_v1`
        * `StorageExplorer_CustomConnections_Blobs_v2`
    * Partages de fichiers
        * `StorageExplorer_CustomConnections_Files_v1`
    * Files d’attente
        * `StorageExplorer_CustomConnections_Queues_v1`
    * Tables
        * `StorageExplorer_CustomConnections_Tables_v1`
4. Une fois que vous avez enregistré vos noms de connexion actuels, définissez la valeur dans Outils de développement sur `[]`.

Si vous souhaitez conserver les connexions non endommagées, vous pouvez suivre les étapes ci-dessous pour localiser les connexions endommagées. Si vous ne voyez pas d’inconvénient à perdre toutes les connexions existantes, vous pouvez ignorer ces étapes et suivre les instructions propres à la plateforme pour effacer vos données de connexion.

1. À partir d’un éditeur de texte, ajoutez à nouveau chaque nom de connexion à Outils de développement, puis vérifiez si la connexion fonctionne toujours.
2. Si une connexion fonctionne correctement, c’est qu’elle n’est pas endommagée et vous pouvez l’y laisser sans risque. Si une connexion ne fonctionne pas, supprimez sa valeur de Outils de développement, puis enregistrez-la de façon à la rajouter ultérieurement.
3. Répétez l’opération jusqu’à ce que vous ayez examiné toutes vos connexions.

Une fois que vous les avez toutes passées en revue, pour tous les noms de connexion qui ne sont pas rajoutés, vous devez effacer leurs données endommagées (le cas échéant) et les rajouter en suivant les étapes normales dans l’Explorateur Stockage :

# <a name="windows"></a>[Windows](#tab/Windows)

1. Dans le menu **Démarrer** , recherchez le **Gestionnaire d’informations d’identification** et ouvrez-le.
2. Accédez à **Informations d’identification Windows** .
3. Sous **Informations d’identification génériques** , recherchez les entrées qui comportent la clé `<connection_type_key>/<corrupted_connection_name>` (par exemple, `StorageExplorer_CustomConnections_Accounts_v1/account1`).
4. Supprimez ces entrées et rajoutez les connexions.

# <a name="macos"></a>[macOS](#tab/macOS)

1. Ouvrez Spotlight (Commande + Barre d’espace) et recherchez **Trousseau d’accès** .
2. Recherchez les entrées qui comportent la clé `<connection_type_key>/<corrupted_connection_name>` (par exemple, `StorageExplorer_CustomConnections_Accounts_v1/account1`).
3. Supprimez ces entrées et rajoutez les connexions.

# <a name="linux"></a>[Linux](#tab/Linux)

La gestion des informations d’identification locales varie en fonction de la distribution Linux. Si votre distribution Linux n’intègre pas d’outil de gestion des informations d’identification locales à interface graphique utilisateur (GUI), vous pouvez installer un outil tiers pour gérer vos informations d’identification locales. Par exemple, vous pouvez utiliser [Seahorse](https://wiki.gnome.org/Apps/Seahorse/), outil GUI open source de gestion des informations d’identification locales Linux.

1. Ouvrez votre outil de gestion des informations d’identification locales et recherchez vos informations d’identification enregistrées.
2. Recherchez les entrées qui comportent la clé `<connection_type_key>/<corrupted_connection_name>` (par exemple, `StorageExplorer_CustomConnections_Accounts_v1/account1`).
3. Supprimez ces entrées et rajoutez les connexions.
---

Si vous continuez de rencontrer cette erreur après avoir exécuté ces étapes ou si vous souhaitez faire part de vous soupçons quant à l’origine de l’endommagement des connexions, [signalez un problème](https://github.com/microsoft/AzureStorageExplorer/issues) sur notre page GitHub.

## <a name="issues-with-sas-url"></a>Problèmes liés à l’URL SAP

Si vous vous connectez à un service via une URL de signature d’accès partagé (SAP) et que vous rencontrez cette erreur :

* Vérifiez que l’URL fournit les autorisations nécessaires pour lire ou répertorier les ressources.
* Vérifiez que l’URL n’a pas expiré.
* Si l’URL SAP est basée sur une stratégie d’accès, vérifiez que cette dernière n’a pas été révoquée.

Si vous avez accidentellement procédé à un attachement au moyen d’une URL SAP non valide et que vous ne pouvez pas maintenant effectuer de détachement, suivez ces étapes :

1. Pendant que vous exécutez l’Explorateur Stockage, appuyez sur F12 pour ouvrir la fenêtre Outils de développement.
2. Sous l’onglet **Application** , sélectionnez **Stockage local** > **file://** dans l’arborescence située à gauche.
3. Recherchez la clé associée au type de service de l’URI SAS problématique. Par exemple, si l’URI SAS incorrect concerne un conteneur d’objets blob, recherchez la clé nommée `StorageExplorer_AddStorageServiceSAS_v1_blob`.
4. La valeur de la clé doit être un tableau JSON. Recherchez l’objet associé à l’URI incorrect, puis supprimez-le.
5. Appuyez sur Ctrl+R pour recharger l’Explorateur de stockage.

## <a name="linux-dependencies"></a>Dépendances Linux

### <a name="snap"></a>Snap

L’Explorateur Stockage 1.10.0 et les versions ultérieures sont disponibles en tant que snap sur le Snap Store. Le snap Explorateur Stockage installe automatiquement toutes ses dépendances, et il est mis à jour dès qu’une nouvelle version du snap est disponible. L’installation du snap Explorateur Stockage est la méthode d’installation recommandée.

Pour bien fonctionner, l’Explorateur Stockage a besoin d’un gestionnaire de mots de passe que vous pouvez être amené à connecter manuellement. Vous pouvez connecter l’Explorateur Stockage au gestionnaire de mots de passe de votre système en exécutant la commande suivante :

```bash
snap connect storage-explorer:password-manager-service :password-manager-service
```

### <a name="targz-file"></a>Fichier.tar.gz

Vous pouvez aussi télécharger l’application sous forme de fichier. tar. gz, mais vous devrez installer les dépendances manuellement.

L'Explorateur Stockage fourni dans le téléchargement .tar.gz est uniquement pris en charge pour les versions suivantes d’Ubuntu. Il se peut que l’Explorateur Stockage fonctionne sur d’autres distributions Linux, mais elles ne sont pas officiellement prises en charge.

- Ubuntu 20.04 x64
- Ubuntu 18.04 x64
- Ubuntu 16.04 x64

L’Explorateur Stockage exige l’installation de .NET Core sur votre système. Nous vous recommandons .NET Core 2.1, mais l’Explorateur Stockage fonctionnera également avec la version 2.2.

> [!NOTE]
> L’Explorateur Stockage 1.7.0 et versions antérieures nécessitent .NET Core 2.0. Si vous disposez d’une version plus récente de .NET Core, vous devrez appliquer une [mise à jour corrective de l’Explorateur Stockage](#patching-storage-explorer-for-newer-versions-of-net-core). Si vous exécutez l’Explorateur Stockage 1.8.0 ou une version ultérieure, vous devez utiliser .NET Core 2.1.

# <a name="ubuntu-2004"></a>[Ubuntu 20.04](#tab/2004)

1. Téléchargez le fichier .tar.gz de l’Explorateur Stockage.
2. Installez le [runtime .NET Core](/dotnet/core/install/linux) :
   ```bash
   wget https://packages.microsoft.com/config/ubuntu/20.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb; \
     sudo dpkg -i packages-microsoft-prod.deb; \
     sudo apt-get update; \
     sudo apt-get install -y apt-transport-https && \
     sudo apt-get update && \
     sudo apt-get install -y dotnet-runtime-2.1
   ```

# <a name="ubuntu-1804"></a>[Ubuntu 18.04](#tab/1804)

1. Téléchargez le fichier .tar.gz de l’Explorateur Stockage.
2. Installez le [runtime .NET Core](/dotnet/core/install/linux) :
   ```bash
   wget https://packages.microsoft.com/config/ubuntu/18.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb; \
     sudo dpkg -i packages-microsoft-prod.deb; \
     sudo apt-get update; \
     sudo apt-get install -y apt-transport-https && \
     sudo apt-get update && \
     sudo apt-get install -y dotnet-runtime-2.1
   ```

# <a name="ubuntu-1604"></a>[Ubuntu 16.04](#tab/1604)

1. Téléchargez le fichier .tar.gz de l’Explorateur Stockage.
2. Installez le [runtime .NET Core](/dotnet/core/install/linux) :
   ```bash
   wget https://packages.microsoft.com/config/ubuntu/16.04/packages-microsoft-prod.deb -O packages-microsoft-prod.deb; \
     sudo dpkg -i packages-microsoft-prod.deb; \
     sudo apt-get update; \
     sudo apt-get install -y apt-transport-https && \
     sudo apt-get update && \
     sudo apt-get install -y dotnet-runtime-2.1
   ```
---

De nombreuses bibliothèques requise par l’Explorateur Stockage sont préinstallées avec les installations standard Canonical d’Ubuntu. Il se peut que certaines de ces bibliothèques soient manquantes dans les environnements personnalisés. Si vous rencontrez des problèmes lors du lancement de l’Explorateur Stockage, nous vous recommandons de vérifier si les packages suivants sont installés sur votre système :

- iproute2
- libasound2
- libatm1
- libgconf2-4
- libnspr4
- libnss3
- libpulse0
- libsecret-1-0
- libx11-xcb1
- libxss1
- libxtables11
- libxtst6
- xdg-utils

### <a name="patching-storage-explorer-for-newer-versions-of-net-core"></a>Mise à jour corrective de l’Explorateur Stockage pour les versions plus récentes de .NET Core

Pour l’Explorateur Stockage 1.7.0 ou version antérieure, vous devrez peut-être appliquer une mise à jour corrective à la version de .NET Core utilisée par l’Explorateur Stockage :

1. Téléchargez la version 1.5.43 de StreamJsonRpc [à partir de NuGet](https://www.nuget.org/packages/StreamJsonRpc/1.5.43). Recherchez le lien « Download package » sur le côté droit de la page.
2. Après avoir téléchargé le package, remplacez son extension de fichier `.nupkg` par `.zip`.
3. Décompressez le package.
4. Ouvrez le dossier `streamjsonrpc.1.5.43/lib/netstandard1.1/`.
5. Copiez `StreamJsonRpc.dll` aux emplacements suivants dans le dossier de l’Explorateur Stockage :
   * `StorageExplorer/resources/app/ServiceHub/Services/Microsoft.Developer.IdentityService/`
   * `StorageExplorer/resources/app/ServiceHub/Hosts/ServiceHub.Host.Core.CLR.x64/`

## <a name="open-in-explorer-from-the-azure-portal-doesnt-work"></a>« Ouvrir dans l’Explorateur » sur le portail Azure ne fonctionne pas

Si le bouton **Ouvrir dans l’Explorateur** du portail Azure ne fonctionne pas, vérifiez que vous utilisez un navigateur compatible. Voici les navigateurs dont la compatibilité a été testée :
* Microsoft Edge
* Mozilla Firefox
* Google Chrome
* Microsoft Internet Explorer

## <a name="next-steps"></a>Étapes suivantes

Si aucune de ces solutions ne fonctionne pour vous, [signalez un problème dans GitHub](https://github.com/Microsoft/AzureStorageExplorer/issues). Vous pouvez aussi cliquer sur le bouton **Report issue to GitHub** en bas à gauche.

![Commentaires](./media/storage-explorer-troubleshooting/feedback-button.PNG)
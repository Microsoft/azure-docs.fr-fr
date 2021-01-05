---
title: Authentification LDAP et serveur Azure MFA - Azure Active Directory
description: Déploiement de l’authentification LDAP et du serveur Azure Multi-Factor Authentication.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 07/11/2018
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 8961cccbd57294e477f0d33202fe91cd292b7814
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742219"
---
# <a name="ldap-authentication-and-azure-multi-factor-authentication-server"></a>Authentification LDAP et serveur Azure Multi-Factor Authentication

Par défaut, le serveur Azure Multi-Factor Authentication est configuré pour importer ou synchroniser des utilisateurs à partir d'Active Directory. Toutefois, il peut être configuré pour être lié à d’autres répertoires LDAP, comme un répertoire ADAM ou un contrôleur de domaine Active Directory spécifique. Lorsqu’il est connecté à un annuaire par LDAP, le serveur Azure Multi-Factor Authentication peut servir de proxy LDAP pour effectuer les authentifications. Il permet également d’utiliser la liaison LDAP comme cible RADIUS, pour pré-authentifier les utilisateurs avec l’authentification IIS ou pour effectuer l’authentification principale dans le portail de l’utilisateur Azure MFA.

Pour utiliser Azure Multi-Factor Authentication comme proxy LDAP, insérez le serveur Azure Multi-Factor Authentication entre le client LDAP (par exemple, un équipement VPN ou une application) et le serveur d’annuaire LDAP. Le serveur Azure Multi-Factor Authentication doit être configuré pour communiquer avec les serveurs des clients et l’annuaire LDAP. Dans cette configuration, le serveur Azure Multi-Factor Authentication accepte les requêtes LDAP provenant des serveurs clients et des applications, puis les transmet au serveur de répertoire LDAP cible pour valider les informations d'identification principales. Si l’annuaire LDAP valide les informations d’identification principales, Azure Multi-Factor Authentication effectue une seconde vérification d’identité et renvoie une réponse au client LDAP. L’authentification n’est complète que si l’authentification du serveur LDAP et la vérification en deux étapes aboutissent.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="configure-ldap-authentication"></a>Configurer l’authentification LDAP

Pour configurer l'authentification LDAP, installez le serveur Azure Multi-Factor Authentication sur un serveur Windows. Procédez comme suit :

### <a name="add-an-ldap-client"></a>Ajouter un client LDAP

1. Dans le serveur Azure Multi-Factor Authentication, cliquez sur l’icône d’authentification LDAP dans le menu de gauche.
2. Cochez la case **Activer l’authentification LDAP**.

   ![Authentification LDAP dans le serveur MFA](./media/howto-mfaserver-dir-ldap/ldap2.png)

3. Dans l’onglet Clients, modifiez le port TCP et le port SSL (TLS) si le service LDAP Azure Multi-Factor Authentication doit se connecter à des ports non standard pour écouter les requêtes LDAP.
4. Si vous envisagez d’utiliser LDAPS, du client vers le serveur Azure Multi-Factor Authentication (MFA), un certificat TLS/SSL doit être installé sur le même serveur que le serveur MFA. Cliquez sur **Parcourir** en regard de la zone Certificat SSL (TLS) et sélectionnez un certificat à utiliser pour la connexion sécurisée.
5. Cliquez sur **Add**.
6. Dans la boîte de dialogue Ajouter un client LDAP, entrez l’adresse IP de l’appliance, du serveur ou de l’application qui s’authentifie auprès du serveur, et un nom d’application (facultatif). Le nom de l'application apparaît dans les rapports du serveur Azure Multi-Factor Authentication et peut être affiché dans les messages d'authentification SMS ou d’application mobile.
7. Cochez la case de **correspondance d’utilisateur Authentification multifacteur Azure requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à la vérification en deux étapes. Si un grand nombre d’utilisateurs n’ont pas encore été importés sur le serveur et/ou ne sont pas soumis à la vérification en deux étapes, laissez la case désactivée. Voir le fichier d’aide du serveur MFA pour plus d’informations sur cette fonctionnalité.

Répétez ces étapes pour ajouter d’autres clients LDAP.

### <a name="configure-the-ldap-directory-connection"></a>Configurer la connexion à l’annuaire LDAP

Lorsque l'authentification multifacteur Azure est configurée pour recevoir des authentifications LDAP, il doit transférer via un proxy ces authentifications au répertoire LDAP. Par conséquent, l'onglet Cible affiche une seule option grisée pour utiliser une cible LDAP.

> [!NOTE]
> Il n’est pas garanti que l’intégration d’annuaire fonctionne correctement avec des répertoires autres que Microsoft Active Directory Domain Services.

1. Pour configurer la connexion à l’annuaire LDAP, cliquez sur l’icône **Intégration d’annuaire**.
2. Sous l’onglet Paramètres, sélectionnez le bouton radio **Utiliser une configuration LDAP spécifique**.
3. Sélectionnez **Modifier…** .
4. Dans la boîte de dialogue Modifier la configuration LDAP, remplissez les champs avec les informations requises pour se connecter au répertoire LDAP. Le fichier d’aide du serveur Azure Multi-Factor Authentication fournit une description des champs.

    ![Configuration LDAP pour l’intégration d’annuaire](./media/howto-mfaserver-dir-ldap/ldap.png)

5. Testez la connexion LDAP en cliquant sur le bouton **Test**.
6. Si le test de connexion LDAP a réussi, cliquez sur le bouton **OK**.
7. Cliquez sur l’onglet **Filtres**. Le serveur est préconfiguré pour charger des conteneurs, des groupes de sécurité et des utilisateurs à partir d’Active Directory. En cas de liaison à un autre annuaire LDAP, vous devrez probablement modifier les filtres affichés. Pour plus d’informations sur les filtres, cliquez sur le lien **Aide**.
8. Cliquez sur onglet **Attributes** . Le serveur est préconfiguré pour mapper les attributs à partir d’Active Directory.
9. En cas de liaison à un autre annuaire LDAP ou de modification des mappages d’attributs préconfigurés, cliquez sur **Modifier…** .
10. Dans la boîte de dialogue Modifier les attributs, modifiez les mappages d'attributs LDAP de votre répertoire. Les noms d’attribut peuvent être saisis ou sélectionnés à l’aide du bouton **…** en regard de chaque champ. Pour plus d’informations sur les attributs, cliquez sur le lien **Aide**.
11. Cliquez sur le bouton **OK** .
12. Cliquez sur l’icône **Paramètres de la société** et sélectionnez l’onglet **Résolution du nom d’utilisateur**.
13. Si vous vous connectez à Active Directory à partir d’un serveur joint à un domaine, laissez le bouton radio **Utiliser des identificateurs de sécurité Windows (SID) pour les noms d’utilisateur correspondants** sélectionné. Sinon, sélectionnez le bouton radio **Utiliser un attribut d’identificateur unique LDAP pour les noms d’utilisateur correspondants**.

Lorsque le bouton radio **Utiliser un attribut d’identificateur unique LDAP pour les noms d’utilisateur correspondants** est sélectionné, le serveur Azure Multi-Factor Authentication tente de résoudre chaque nom d’utilisateur en un identificateur unique dans l’annuaire LDAP. Une recherche LDAP est effectuée sur les attributs Nom d’utilisateur définis dans l’onglet Intégration d’annuaire -> Attributs. Lorsqu’un utilisateur s’authentifie, le nom d’utilisateur est résolu avec l’identificateur unique de l’annuaire LDAP. L’identificateur unique est utilisé pour faire correspondre l’utilisateur dans le fichier de données Azure Multi-Factor Authentication. Cela permet des comparaisons sans prise en compte de la casse et autorise les noms d’utilisateur longs et courts.

Une fois ces opérations terminées, le serveur MFA écoute sur les ports configurés, les demandes d’accès LDAP provenant des clients configurés, et agit comme un proxy pour ces demandes à l’annuaire LDAP à des fins d’authentification.

## <a name="configure-ldap-client"></a>Configurer un client LDAP

Pour configurer le client LDAP, suivez ces indications :

* Configurez votre appliance, votre serveur ou votre application pour vous authentifier par LDAP auprès du serveur Azure Multi-Factor Authentication, comme s’il s’agissait de votre annuaire LDAP. Utilisez les mêmes paramètres que pour vous connecter directement à votre annuaire LDAP, sauf pour le nom du serveur ou l’adresse IP qui sera celui ou celle du serveur Azure Multi-Factor Authentication.
* Réglez le délai d’expiration LDAP sur 30 à 60 secondes pour pouvoir valider les informations d’identification de l’utilisateur dans l’annuaire LDAP, effectuer la vérification en deux étapes, recevoir leur réponse et répondre à la demande d’accès LDAP.
* Si vous utilisez LDAPS, l’appliance ou le serveur qui effectue les requêtes LDAP doit approuver le certificat TLS/SSL installé sur le serveur Azure Multi-Factor Authentication.

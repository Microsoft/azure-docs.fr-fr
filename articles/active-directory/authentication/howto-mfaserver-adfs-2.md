---
title: Utiliser Azure MFA Server avec AD FS 2.0 - Azure Active Directory
description: Voici la page d'authentification multifacteur Azure qui explique la prise en main de l'authentification multifacteur Azure et d’AD FS 2.0.
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
ms.openlocfilehash: d144716dbc5636451405f1124d4c3949f585d6b2
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742644"
---
# <a name="configure-azure-multi-factor-authentication-server-to-work-with-ad-fs-20"></a>Configuration du serveur Azure Multi-Factor Authentication pour travailler avec AD FS 2.0

Cet article s’adresse aux organisations qui sont fédérées avec Azure Active Directory et qui souhaitent sécuriser leurs ressources locales ou dans le cloud. Protégez vos ressources en utilisant le serveur Azure Multi-Factor Authentication et en le configurant de manière à ce qu’il fonctionne avec AD FS afin que la vérification en deux étapes se déclenche pour les points de terminaison de valeur élevée.

Cette documentation traite de l'utilisation du serveur Azure Multi-Factor Authentication avec AD FS 2.0. Pour obtenir des informations sur AD FS, consultez [Sécurisez vos ressources cloud et locales à l’aide du serveur Azure Multi-Factor Authentication avec AD FS dans Windows Server 2012 R2](howto-mfaserver-adfs-2012.md).

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Si vous utilisez une authentification multifacteur basée sur le cloud, consultez [Sécuriser les ressources cloud avec Azure AD Multi-Factor Authentication et AD FS](howto-mfa-adfs.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

## <a name="secure-ad-fs-20-with-a-proxy"></a>Sécurisation d’AD FS 2.0 avec un proxy

Pour sécuriser AD FS 2.0 avec un proxy, installez le serveur Azure Multi-Factor Authentication sur le serveur de proxy AD FS.

### <a name="configure-iis-authentication"></a>Configuration de l’authentification IIS

1. Dans le serveur Azure Multi-Factor Authentication, cliquez sur l’icône de **Authentification IIS** dans le menu de gauche.
2. Cliquez sur l’onglet **Par formulaire**.
3. Cliquez sur **Add**.

   ![Fenêtre de l’authentification IIS dans MFA Server](./media/howto-mfaserver-adfs-2/setup1.png)

4. Pour détecter automatiquement les variables de nom d’utilisateur, de mot de passe et de domaine, entrez l’URL de connexion (par exemple, `https://sso.contoso.com/adfs/ls`) dans la boîte de dialogue Configuration automatique du site web basé sur des formulaires et cliquez sur **OK**.
5. Cochez la case de **correspondance d’utilisateur Authentification multifacteur Azure requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à la vérification en deux étapes. Si un grand nombre d’utilisateurs n’ont pas encore été importés sur le serveur et/ou ne seront pas soumis à la vérification en deux étapes, laissez la case désactivée.
6. Si les variables de page ne peuvent pas être détectées automatiquement, cliquez sur le bouton **Spécifier manuellement...** dans la boîte de dialogue Auto-configurer un site Web basé sur formulaire.
7. Dans la boîte de dialogue Ajouter un site Web basé sur formulaire, entrez l’URL de la page de connexion AD FS dans le champ URL de soumission (par exemple, `https://sso.contoso.com/adfs/ls`) et entrez un nom d’application. Le nom de l'application apparaît dans les rapports du serveur Azure Multi-Factor Authentication et peut être affiché dans les messages d'authentification SMS ou d’application mobile.
8. Définissez le format de demande sur **POST ou GET**.
9. Entrez les variables de nom d'utilisateur (ctl00$ ContentPlaceHolder1$ UsernameTextBox) et de mot de passe (ctl00$ ContentPlaceHolder1$ PasswordTextBox). Si votre page de connexion basée sur formulaire affiche une zone de texte de domaine, entrez la variable de domaine également. Pour rechercher les noms des zones de saisie dans la page de connexion, accédez à la page de connexion dans un navigateur web, cliquez avec le bouton droit sur la page et sélectionnez **Afficher la source**.
10. Cochez la case de **correspondance d’utilisateur Authentification multifacteur Azure requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à la vérification en deux étapes. Si un grand nombre d’utilisateurs n’ont pas encore été importés sur le serveur et/ou ne seront pas soumis à la vérification en deux étapes, laissez la case désactivée.

    ![Ajouter à MFA Server un site web basé sur des formulaires](./media/howto-mfaserver-adfs-2/manual.png)

11. Cliquez sur **Avancé…** pour passer en revue les paramètres avancés. Vous pouvez notamment :

    - Sélectionner un fichier de page de refus personnalisé
    - Mettre en cache les authentifications réussies sur le site web à l’aide de cookies
    - Sélectionner le mode d’authentification des informations d’identification principales

12. Le serveur proxy AD FS n’étant pas susceptible d’être joint au domaine, vous pouvez utiliser LDAP pour vous connecter à votre contrôleur de domaine pour l’importation et la pré-authentification des utilisateurs. Dans la boîte de dialogue Site Web basé sur des formulaires avancés, cliquez sur l’onglet **Authentification principale** et sélectionnez **Liaison LDAP** pour le type de pré-authentification.
13. Lorsque vous avez terminé, cliquez sur **OK** pour revenir à la boîte de dialogue Ajouter un site Web basé sur des formulaires.
14. Cliquez sur **OK** pour fermer la boîte de dialogue.
15. Une fois les variables de page et d’URL détectées ou entrées, les données de site Web s’affichent dans le panneau Par formulaire.
16. Cliquez sur l’onglet **Module natif** et sélectionnez le serveur, le site web sous lequel s’exécute le proxy AD FS (par exemple, « Site web par défaut ») ou l’application de proxy AD FS (par exemple, « ls » sous « adfs ») pour activer le plug-in IIS au niveau souhaité.
17. Cochez la case **Activer l’authentification IIS** en haut de l’écran.

L'authentification IIS est maintenant activée.

### <a name="configure-directory-integration"></a>Configuration de l’intégration d’annuaire

Vous avez activé l’authentification IIS, mais pour effectuer la pré-authentification de votre Active Directory (AD) via LDAP, vous devez configurer la connexion LDAP au niveau du contrôleur de domaine.

1. Cliquez sur l’icône **Intégration d’annuaire**.
2. Sous l’onglet Paramètres, sélectionnez le bouton radio **Utiliser une configuration LDAP spécifique**.

   ![Configurer les paramètres LDAP pour des paramètres LDAP spécifiques](./media/howto-mfaserver-adfs-2/ldap1.png)

3. Cliquez sur **Modifier**.
4. Dans la boîte de dialogue Modifier la configuration LDAP, remplissez les champs avec les informations requises pour se connecter au contrôleur de domaine Active Directory. Le fichier d’aide du serveur Azure Multi-Factor Authentication fournit une description des champs.
5. Testez la connexion LDAP en cliquant sur le bouton **Test**.

   ![Tester la configuration LDAP dans MFA Server](./media/howto-mfaserver-adfs-2/ldap2.png)

6. Si le test de connexion LDAP a réussi, cliquez sur **OK**.

### <a name="configure-company-settings"></a>Configuration des paramètres de la société

1. Ensuite, cliquez sur l’icône **Paramètres de la société** et sélectionnez l’onglet **Résolution du nom d’utilisateur**.
2. Sélectionnez le bouton radio **Utiliser un attribut d’identificateur unique LDAP pour les noms d’utilisateur correspondants**.
3. Si les utilisateurs entrent leur nom d’utilisateur au format « domaine\nom_utilisateur », le serveur doit être en mesure de retirer le domaine du nom d’utilisateur lorsqu’il crée la requête LDAP. Ceci peut être effectué via un paramètre du registre.
4. Ouvrez l’éditeur du registre et accédez à HKEY_LOCAL_MACHINE/SOFTWARE/Wow6432Node/Positive Networks/PhoneFactor sur un serveur 64 bits. Si vous utilisez un serveur 32 bits, retirez la partie « Wow6432Node » du chemin d’accès. Créez une clé de registre DWORD appelée « UsernameCxz_stripPrefixDomain » et définissez la valeur sur 1. Le proxy AD FS est désormais protégé par le serveur Azure Multi-Factor Authentication.

Assurez-vous que les utilisateurs ont été importés sur le serveur à partir d'Active Directory. Consultez la [section Adresses IP approuvées](#trusted-ips) si vous souhaitez autoriser des adresses IP internes, de telle manière que la vérification en deux étapes ne soit pas demandée lors de la connexion au site web à partir de ces emplacements.

![Éditeur du Registre pour configurer les paramètres de la société](./media/howto-mfaserver-adfs-2/reg.png)

## <a name="ad-fs-20-direct-without-a-proxy"></a>AD FS 2.0 Direct sans proxy

Vous pouvez sécuriser AD FS lorsque le proxy AD FS n’est pas utilisé. Installez le serveur Azure Multi-Factor Authentication sur le serveur AD FS et configurez le serveur selon la procédure suivante :

1. Depuis le serveur Azure Multi-Factor Authentication, cliquez sur l’icône **Authentification IIS** dans le menu de gauche.
2. Cliquez sur l’onglet **HTTP**.
3. Cliquez sur **Add**.
4. Dans la boîte de dialogue Ajouter une URL de base, entrez l’URL du site web AD FS où l’authentification HTTP est effectuée (par exemple, `https://sso.domain.com/adfs/ls/auth/integrated`) dans le champ URL de base. Ensuite, entrez un nom d’application (facultatif). Le nom de l'application apparaît dans les rapports du serveur Azure Multi-Factor Authentication et peut être affiché dans les messages d'authentification SMS ou d’application mobile.
5. Si vous le souhaitez, vous pouvez régler le délai de déconnexion en cas d'inactivité et la durée maximale des sessions.
6. Cochez la case de **correspondance d’utilisateur Authentification multifacteur Azure requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à la vérification en deux étapes. Si un grand nombre d’utilisateurs n’ont pas encore été importés sur le serveur et/ou ne seront pas soumis à la vérification en deux étapes, laissez la case désactivée.
7. Cochez la case de cache de cookie si vous le souhaitez.

   ![AD FS 2.0 Direct sans proxy](./media/howto-mfaserver-adfs-2/noproxy.png)

8. Cliquez sur **OK**.
9. Cliquez sur l’onglet **Module natif** et sélectionnez le serveur, le site web (par exemple, « Site web par défaut ») ou l’application AD FS (par exemple, « ls » sous « adfs ») pour activer le plug-in IIS au niveau souhaité.
10. Cochez la case **Activer l’authentification IIS** en haut de l’écran.

Le proxy AD FS est désormais protégé par le serveur Azure Multi-Factor Authentication.

Assurez-vous que les utilisateurs ont été importés sur le serveur à partir d'Active Directory. Consultez la section Adresses IP approuvées si vous souhaitez autoriser des adresses IP internes, de sorte que la vérification en deux étapes ne soit pas demandée lors de la connexion au site web à partir de ces emplacements.

## <a name="trusted-ips"></a>Adresses IP approuvées

Les adresses IP approuvées permettent aux utilisateurs de contourner l’authentification multifacteur Azure pour les requêtes de sites Web provenant d’adresses IP ou de sous-réseaux spécifiques. Par exemple, vous pouvez ne pas requérir la vérification en deux étapes lorsque les utilisateurs se connectent à partir du bureau. Pour ce faire, vous devez définir le sous-réseau du bureau comme adresse IP approuvée.

### <a name="to-configure-trusted-ips"></a>Pour configurer des adresses IP de confiance

1. Dans la section Authentification IIS, cliquez sur l’onglet **Adresses IP approuvées**.
2. Cliquez sur **Ajouter…** .
3. Lorsque la boîte de dialogue Ajouter des adresses IP approuvées s’affiche, sélectionnez les boutons **Adresse IP unique**, **Plage d’adresses IP** ou **Sous-réseau**.
4. Entrez l’adresse IP, la plage d’adresses IP ou le sous-réseau à autoriser. Si vous entrez un sous-réseau, sélectionnez le masque réseau approprié et cliquez sur **OK**.

![Configurer les adresses IP approuvées vers MFA Server](./media/howto-mfaserver-adfs-2/trusted.png)

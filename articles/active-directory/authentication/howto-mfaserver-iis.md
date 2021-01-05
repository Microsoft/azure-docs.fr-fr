---
title: Authentification IIS et serveur Azure MFA - Azure Active Directory
description: Déploiement de l’authentification IIS et du serveur Azure Multi-Factor Authentication.
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
ms.openlocfilehash: c522fb950e2489afce44553d233ac3eddf138730
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742168"
---
# <a name="configure-azure-multi-factor-authentication-server-for-iis-web-apps"></a>Configuration d’un serveur Azure Multi-Factor Authentication pour les applications web IIS

Utilisez la section Authentification IIS du serveur Azure Multi-Factor Authentication (MFA) pour activer et configurer l'authentification IIS pour l'intégration avec les applications Web Microsoft IIS. Le serveur Azure MFA installe un plug-in capable de filtrer les demandes envoyées au serveur Web IIS pour ajouter Azure Multi-Factor Authentication. Le plug-in IIS prend en charge l'authentification par formulaire et l'authentification intégrée Windows HTTP. Les adresses IP de confiance peuvent également être configurées pour exempter les adresses IP internes de l'authentification à deux facteurs.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisés avec l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.
>
> Lorsque vous utilisez la fonction Azure Multi-Factor Authentication basée sur le cloud, il n’existe aucune alternative au plug-in IIS fourni par le serveur Azure Multi-Factor Authentication (MFA). Vous utilisez à la place un proxy d’application web (WAP) de type Services de fédération Active Directory (AD FS) ou Azure Active Directory.

![Authentification IIS dans le serveur MFA](./media/howto-mfaserver-iis/iis.png)

## <a name="using-form-based-iis-authentication-with-azure-multi-factor-authentication-server"></a>Utilisation de l'authentification IIS basée sur le formulaire avec le serveur Azure Multi-Factor Authentication

Pour sécuriser une application Web IIS qui utilise l'authentification par formulaire, installez le serveur Azure Multi-Factor Authentication sur le serveur Web IIS et configurez le serveur conformément à la procédure suivante :

1. Dans le serveur Azure Multi-Factor Authentication, cliquez sur l’icône de Authentification IIS dans le menu de gauche.
2. Cliquez sur l’onglet **Par formulaire**.
3. Cliquez sur **Add**.
4. Pour détecter automatiquement les variables de nom d’utilisateur, de mot de passe et de domaine, entrez l’URL de connexion (par exemple, `https://localhost/contoso/auth/login.aspx`) dans la boîte de dialogue Configuration automatique du site web basé sur des formulaires et cliquez sur **OK**.
5. Cochez la case de **correspondance d'utilisateur Authentification multifacteur requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à l'authentification multifacteur. Si un grand nombre d'utilisateurs n'ont pas encore été importés sur le serveur et/ou ne seront pas soumis à l'authentification multifacteur, laissez la case désactivée.
6. Si les variables de page ne peuvent pas être détectées automatiquement, cliquez sur **Spécifier manuellement** dans la boîte de dialogue Auto-configurer un site Web basé sur formulaire.
7. Dans la boîte de dialogue Ajouter un site Web basé sur formulaire, entrez l'URL de la page de connexion dans le champ URL de soumission et entrez un nom d'application (facultatif). Le nom de l'application apparaît dans les rapports du serveur Azure Multi-Factor Authentication et peut être affiché dans les messages d'authentification SMS ou d’application mobile.
8. Sélectionnez le format de demande approprié. Pour la plupart des applications Web, le format est défini sur « **POST or GET** ».
9. Entrez les variables Nom d'utilisateur, Mot de passe et Domaine (si elles apparaissent sur la page de connexion). Pour rechercher les noms des zones de saisis, accédez à la page de connexion dans un navigateur web, cliquez avec le bouton droit sur la page et sélectionnez **Afficher la source**.
10. Cochez la case **Correspondance d'utilisateur Authentification multifacteur Azure requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à l'authentification multifacteur. Si un grand nombre d'utilisateurs n'ont pas encore été importés sur le serveur et/ou ne seront pas soumis à l'authentification multifacteur, laissez la case désactivée.
11. Cliquez sur **Avancé** pour passer en revue les paramètres avancés, notamment :

    - Sélectionner un fichier de page de refus personnalisé
    - Mettre en cache les authentifications réussies sur le site web pendant une période de temps à l’aide de cookies
    - Indiquez si vous souhaitez authentifier les informations d’identification principales par rapport à un domaine Windows dans un service d’annuaire LDAP ou sur un serveur RADIUS.

12. Cliquez sur **OK** pour revenir à la boîte de dialogue Ajouter un site Web basé sur des formulaires.
13. Cliquez sur **OK**.
14. Une fois les variables de page et d’URL détectées ou entrées, les données de site Web s’affichent dans le panneau Par formulaire.

## <a name="using-integrated-windows-authentication-with-azure-multi-factor-authentication-server"></a>Utilisation de l'authentification Windows intégrée avec le serveur Azure Multi-Factor Authentication

Pour sécuriser une application Web IIS qui utilise l'authentification Windows HTTP intégrée, installez le serveur Azure MFA sur le serveur Web IIS et configurez le serveur conformément à la procédure suivante :

1. Dans le serveur Azure Multi-Factor Authentication, cliquez sur l’icône de Authentification IIS dans le menu de gauche.
2. Cliquez sur l’onglet **HTTP**.
3. Cliquez sur **Add**.
4. Dans la boîte de dialogue Ajouter une URL de base, entrez l’URL du site web où l’authentification HTTP est effectuée (par exemple, `http://localhost/owa`) et entrez un nom d’application (facultatif). Le nom de l'application apparaît dans les rapports du serveur Azure Multi-Factor Authentication et peut être affiché dans les messages d'authentification SMS ou d’application mobile.
5. Réglez le délai de déconnexion en cas d'inactivité et la durée maximale des sessions si les valeurs par défaut sont insuffisantes
6. Cochez la case de **correspondance d'utilisateur Authentification multifacteur requise** si tous les utilisateurs ont été ou seront importés sur le serveur et soumis à l'authentification multifacteur. Si un grand nombre d'utilisateurs n'ont pas encore été importés sur le serveur et/ou ne seront pas soumis à l'authentification multifacteur, laissez la case désactivée.
7. Cochez la case **Cache de cookie** si vous le souhaitez.
8. Cliquez sur **OK**.

## <a name="enable-iis-plug-ins-for-azure-multi-factor-authentication-server"></a>Activation des plug-ins IIS pour le serveur Azure Multi-Factor Authentication

Après avoir configuré les URL et les réglages d’authentification basés sur formulaire ou HTTP, sélectionnez les emplacements où les plug-ins IIS d'Azure multi-Factor Authentification IIS doivent être chargés et activés dans IIS. Procédez comme suit :

1. Si vous exécutez IIS 6, cliquez sur l’onglet **ISAPI**. Sélectionnez le site Web où l'application Web est exécutée (par exemple, Site Web par défaut) pour activer le plug-in de filtre ISAPI Azure Multi-Factor Authentication pour ce site.
2. Si vous exécutez IIS 7 ou version ultérieure, cliquez sur l’onglet **Module natif**. Sélectionnez le serveur, les sites web ou les applications pour activer le plug-in IIS au niveau souhaité.
3. Cochez la case **Activer l’authentification IIS** en haut de l’écran. L’application IIS sélectionnée est désormais protégée par le serveur Azure Multi-Factor Authentication. Assurez-vous que les utilisateurs ont été importés sur le serveur.

## <a name="trusted-ips"></a>Adresses IP approuvées

Les adresses IP approuvées permettent aux utilisateurs de contourner l'authentification multifacteur Azure pour les requêtes de sites Web provenant d'adresses IP ou de sous-réseaux spécifiques. Il se peut par exemple que vous souhaitiez exempter certains utilisateurs de l'authentification multifacteur Azure lorsqu’ils se connectent du bureau. Pour ce faire, vous devez définir le sous-réseau du bureau comme adresse IP approuvée. Pour configurer les adresses IP de confiance, utilisez la procédure suivante :

1. Dans la section Authentification IIS, cliquez sur l’onglet **Adresses IP approuvées**.
2. Cliquez sur **Add**.
3. Lorsque la boîte de dialogue Ajouter des adresses IP de confiance s'affiche, sélectionnez le bouton **Adresse IP unique**, **Plage d'adresses IP** ou **Sous-réseau**.
4. Entrez l’adresse IP, la plage d’adresses IP ou le sous-réseau à autoriser. Si vous entrez un sous-réseau, sélectionnez le masque réseau approprié et cliquez sur **OK**.

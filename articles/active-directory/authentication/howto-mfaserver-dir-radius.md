---
title: RADIUS et Azure MFA Server - Azure Active Directory
description: Déploiement de l’authentification RADIUS et du serveur Azure Multi-Factor Authentication.
services: multi-factor-authentication
ms.service: active-directory
ms.subservice: authentication
ms.topic: how-to
ms.date: 11/21/2019
ms.author: justinha
author: justinha
manager: daveba
ms.reviewer: michmcla
ms.collection: M365-identity-device-management
ms.openlocfilehash: 23e2eafae185551c79d651ec6dcf39ad11eb8534
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96742117"
---
# <a name="integrate-radius-authentication-with-azure-multi-factor-authentication-server"></a>Intégration de l'authentification RADIUS avec le serveur Azure Multi-Factor Authentication

RADIUS est un protocole standard pour accepter les demandes d'authentification et traiter ces demandes. Le serveur Azure Multi-Factor Authentication peut fonctionner comme un serveur RADIUS. Pour ajouter la vérification en deux étapes, insérez-le entre votre client RADIUS (appliance VPN) et la cible d’authentification. Il peut s’agir d’Active Directory, d’un annuaire LDAP, ou bien d’un autre serveur RADIUS. Pour garantir le fonctionnement Azure Multi-Factor Authentication (MFA), vous devez configurer le serveur Azure MFA pour communiquer avec les serveurs clients et la cible de l’authentification. Le serveur Azure MFA accepte les requêtes provenant d'un client RADIUS, valide les informations d'identification par rapport à la cible de l'authentification, ajoute l’authentification multifacteur Azure et renvoie une réponse au client RADIUS. La demande d’authentification est complète uniquement si l'authentification principale et l'authentification multifacteur Azure réussissent.

> [!IMPORTANT]
> Depuis le 1er juillet 2019, Microsoft ne propose plus MFA Server pour les nouveaux déploiements. Les nouveaux clients qui souhaitent imposer une authentification multifacteur lors des événements de connexion doivent utiliser la fonctionnalité d’authentification multifacteur Azure AD basée sur le cloud.
>
> Pour commencer à utiliser l’authentification multifacteur basée sur le cloud, consultez [Tutoriel : Événements de connexion utilisateur sécurisée avec Azure AD Multi-Factor Authentication](tutorial-enable-azure-mfa.md).
>
> Si vous utilisez une authentification multifacteur basée sur le cloud, consultez [Intégrer votre infrastructure NPS existante à Azure Multi-Factor Authentication](howto-mfa-nps-extension.md).
>
> Les clients existants qui ont activé le serveur MFA avant le 1er juillet 2019 peuvent télécharger la dernière version, les futures mises à jour et générer des informations d’identification d’activation comme d’habitude.

> [!NOTE]
> Le serveur MFA prend uniquement en charge les protocoles RADIUS PAP (Password Authentication Protocol) et MSCHAPv2 (Microsoft Challenge Handshake Authentication Protocol) lorsqu'il agit comme un serveur RADIUS.  Les autres protocoles comme EAP (Extensible Authentication Protocol) peuvent être utilisés lorsque le serveur MFA agit comme un proxy RADIUS vers un autre serveur RADIUS prenant en charge ce protocole.
>
> Dans cette configuration, les jetons SMS et OATH unidirectionnels ne fonctionnent pas car le serveur MFA n'est pas capable d'initier correctement une réponse RADIUS Challenge à l'aide d’autres protocoles.

![Authentification Radius dans MFA Server](./media/howto-mfaserver-dir-radius/radius.png)

## <a name="add-a-radius-client"></a>Ajouter un client RADIUS

Pour configurer l'authentification RADIUS, installez le serveur Azure Multi-Factor Authentication sur un serveur Windows. Si vous utilisez un environnement Active Directory, le serveur doit être lié au domaine à l'intérieur du réseau. Utilisez la procédure suivante pour configurer le serveur Azure Multi-Factor Authentication :

1. Sur le serveur Azure Multi-Factor Authentication, cliquez sur l'icône de l'authentification RADIUS dans le menu de gauche.
2. Cochez la case **Activer l'authentification RADIUS**.
3. Dans l’onglet Clients, modifiez le port d'authentification et le port de gestion si le service Azure MFA RADIUS doit écouter les demandes RADIUS sur des ports non standard.
4. Cliquez sur **Add**.
5. Entrez l'adresse IP de l'appliance ou du serveur qui s'authentifieront auprès du serveur Azure Multi-Factor Authentication, un nom d’application (facultatif) et un secret partagé.

   Le nom de l’application apparaît dans les rapports et peut être affiché dans les messages d’authentification SMS ou d’application mobile.

   Le secret partagé doit être identique sur le serveur Azure Multi-Factor Authentication et sur l’appliance/serveur.

6. Cochez la case de **correspondance d’utilisateur Authentification multifacteur requise** si tous les utilisateurs ont été importés sur le serveur et soumis à l’authentification multifacteur. Si un grand nombre d’utilisateurs n’ont pas encore été importés sur le serveur ou ne sont pas soumis à la vérification en deux étapes, laissez la case désactivée.
7. Cochez la case **Activer un jeton OATH de secours** si vous souhaitez utiliser les codes d’accès OATH à partir d’applications de vérification mobile comme méthode de sauvegarde.
8. Cliquez sur **OK**.

Répétez les étapes 4 à 8 pour ajouter autant de clients RADIUS que nécessaire.

## <a name="configure-your-radius-client"></a>Configurer votre client RADIUS

1. Cliquez sur l'onglet **Cible**.
   * Si le serveur Azure MFA est installé sur un serveur associé à un domaine dans un environnement Active Directory, sélectionnez **Domaine Windows**.
   * Si les utilisateurs doivent être authentifiés par rapport à un répertoire LDAP, sélectionnez la **liaison LDAP**.
      Sélectionnez l’icône Intégration de répertoires et modifiez la configuration LDAP sous l’onglet Paramètres afin que le serveur puisse se lier à votre répertoire. Vous trouverez des instructions sur la configuration LDAP dans le [guide de configuration du proxy LDAP](howto-mfaserver-dir-ldap.md).
   * Si les utilisateurs doivent être authentifiés sur un autre serveur RADIUS, sélectionnez **Serveur(s) RADIUS**.
1. Cliquez sur **Ajouter** pour configurer le serveur vers lequel le serveur Azure MFA enverra des demandes RADIUS par proxy.
1. Dans la boîte de dialogue Ajouter un serveur RADIUS, entrez l'adresse IP du serveur RADIUS et un secret partagé.

   Le secret partagé doit être identique sur le serveur Azure Multi-Factor Authentication et sur le serveur RADIUS. Modifiez le port d'authentification et le port de gestion si des ports différents sont utilisés par le serveur RADIUS.

1. Cliquez sur **OK**.
1. Ajoutez le serveur Azure MFA comme client RADIUS dans l'autre serveur RADIUS afin qu'il puisse traiter les demandes d'accès qu’il reçoit du serveur Azure MFA. Utilisez le même secret partagé configuré sur le serveur Azure Multi-Factor Authentication.

Répétez ces étapes pour ajouter des serveurs RADIUS supplémentaires. Configurez l’ordre dans lequel le serveur Azure MFA doit les appeler à l’aide des boutons **Monter** et **Descendre**.

Vous avez configuré avec succès le serveur Azure Multi-Factor Authentication. Le serveur écoute désormais sur les ports configurés pour les demandes d'accès RADIUS à partir de clients configurés.

## <a name="radius-client-configuration"></a>Configuration du client RADIUS

Pour configurer le client RADIUS, suivez ces indications :

* Configurez votre appliance/serveur pour l’authentification via RADIUS auprès de l’adresse IP du serveur Azure Multi-Factor Authentication, qui agit en tant que serveur RADIUS.
* Utilisez le même secret partagé configuré précédemment.
* Configurez le délai d’expiration RADIUS sur 30 à 60 secondes pour pouvoir valider les informations d’identification de l’utilisateur, effectuez la vérification en deux étapes, recevez la réponse et répondez à la demande d’accès RADIUS.

## <a name="next-steps"></a>Étapes suivantes

Apprenez à [utiliser l'authentification RADIUS pour l'intégration](howto-mfa-nps-extension.md) si vous disposez d'Azure AD Multi-Factor Authentication dans le cloud. 

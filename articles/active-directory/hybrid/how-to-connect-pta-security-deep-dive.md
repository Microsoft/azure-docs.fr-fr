---
title: Immersion dans la sécurité de l’authentification directe Azure Active Directory | Microsoft Docs
description: Cet article décrit comment l’authentification directe Azure Active Directory (Azure AD) protège vos comptes locaux
services: active-directory
keywords: Authentification directe Azure AD Connect, installation d’Active Directory, composants requis pour Azure AD, SSO, Authentification unique
documentationcenter: ''
author: billmath
manager: daveba
ms.service: active-directory
ms.workload: identity
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: how-to
ms.date: 05/27/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 08a73c2b1be4b17136ba19e7efb71c2b21359fdf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89280143"
---
# <a name="azure-active-directory-pass-through-authentication-security-deep-dive"></a>Immersion dans la sécurité de l’authentification directe Azure Active Directory

Cet article décrit plus en détail le fonctionnement de l’authentification directe Azure Active Directory (Azure AD). Il se concentre sur les aspects de la fonctionnalité relevant de la sécurité. Cet article est destiné aux administrateurs de la sécurité et informatiques, aux personnes en charge de la conformité et de la sécurité et aux autres professionnels de l’informatique qui sont responsables de la sécurité et de la conformité informatiques dans les PME et dans les grandes entreprises.

Les sujets abordés sont les suivants :
- informations techniques détaillées sur l’installation et l’inscription des agents d’authentification ;
- informations techniques détaillées sur le chiffrement des mots de passe lors de la connexion de l’utilisateur ;
- sécurité des canaux entre les agents d’authentification locale et Azure AD ;
- informations techniques détaillées sur la façon de maintenir la sécurité des agents d’authentification ;
- autres rubriques relatives à la sécurité.

## <a name="key-security-capabilities"></a>Principales fonctionnalités de sécurité

Voici les aspects clés de cette fonctionnalité relevant de la sécurité :
- Elle repose sur une architecture mutualisée sécurisée qui assure l’isolation des demandes de connexion entre les locataires.
- Les mots de passe locaux ne sont jamais stockés dans le cloud sous quelque forme que ce soit.
- Les agents d’authentification locale, qui écoutent et répondent aux demandes de validation de mot de passe, établissent uniquement des connexions sortantes à partir de votre réseau. Il n’est pas nécessaire d’installer ces agents d’authentification dans un réseau de périmètre (DMZ). En tant que bonne pratique, traitez tous les serveurs exécutant des agents d’authentification comme des systèmes de niveau 0 (voir [référence](/windows-server/identity/securing-privileged-access/securing-privileged-access-reference-material)).
- Seuls les ports standard (80 et 443) permettent d’établir une communication sortante des agents d’authentification à Azure AD. Vous n’avez pas besoin d’ouvrir des ports d’entrée sur votre pare-feu. 
  - Le port 443 est utilisé pour toutes les communications sortantes authentifiées.
  - Le port 80 est utilisé uniquement pour télécharger les listes de révocation de certificats (CRL) pour s’assurer que les certificats utilisés par cette fonctionnalité n’ont pas été révoqués.
  - Pour connaître la configuration réseau complète requise, consultez [Authentification directe Azure Active Directory : Démarrage rapide](how-to-connect-pta-quick-start.md#step-1-check-the-prerequisites).
- Les mots de passe fournis par les utilisateurs pendant la connexion sont chiffrés dans le cloud avant que les agents d’authentification locale ne l’est acceptent pour être validés dans l’annuaire Active Directory.
- Le canal HTTPS entre Azure AD et l’agent d’authentification locale est sécurisé à l’aide de l’authentification mutuelle.
- Il protège vos comptes utilisateur en toute transparence avec les [stratégies d’accès conditionnel d’Azure AD](../conditional-access/overview.md), y compris l’authentification multifacteur (MFA), [en bloquant l’authentification héritée](../conditional-access/concept-conditional-access-conditions.md) et [en filtrant des attaques de mot de passe par recherche exhaustive](../authentication/howto-password-smart-lockout.md).

## <a name="components-involved"></a>Composants impliqués

Pour plus d’informations générales sur la sécurité opérationnelle et la sécurité des données et des services Azure AD, consultez le [Centre de gestion de la confidentialité](https://azure.microsoft.com/support/trust-center/). Les composants suivants sont impliqués lorsque vous utilisez l’authentification directe pour la connexion de l’utilisateur :
- **Azure AD STS** : service d’émission de jeton de sécurité (STS) sans état qui traite les demandes de connexion et émet des jetons de sécurité pour les navigateurs, les clients ou les services des utilisateurs en fonction des besoins.
- **Azure Service Bus** : offre une communication cloud avec une messagerie d’entreprise et une communication relayée qui vous aide à connecter des solutions locales au cloud.
- **Agent d’authentification Azure AD Connect** : Un composant local qui écoute et répond aux requêtes de validation de mot de passe.
- **Azure SQL Database** : contient des informations sur les agents d’authentification de votre locataire, y compris ses métadonnées et clés de chiffrement.
- **Active Directory** : annuaire Active Directory local, où sont stockés vos comptes d’utilisateurs et leurs mots de passe.

## <a name="installation-and-registration-of-the-authentication-agents"></a>Installation et inscription des agents d’authentification

Les agents d’authentification sont installés et inscrits auprès d’Azure AD lorsque vous effectuez les tâches suivantes :
   - [Activer l’authentification directe via Azure AD Connect](./how-to-connect-pta-quick-start.md#step-2-enable-the-feature)
   - [Ajouter d’autres agents d’authentification pour garantir la haute disponibilité des demandes de connexion](./how-to-connect-pta-quick-start.md#step-4-ensure-high-availability) 
   
Le fonctionnement d’un agent d’authentification implique trois phases principales :

1. Installation de l’agent d’authentification
2. Inscription de l’agent d’authentification
3. Initialisation de l’agent d’authentification

Les sections suivantes décrivent ces étapes de manière détaillée.

### <a name="authentication-agent-installation"></a>Installation de l’agent d’authentification

Seuls les administrateurs généraux peuvent installer un agent d’authentification (à l’aide d’Azure AD Connect ou de façon autonome) sur un serveur local. Cette installation ajoute deux nouvelles entrées dans la liste **Panneau de configuration** > **Programmes** > **Programmes et fonctionnalités** :
- L’application de l’agent d’authentification proprement dite. Cette application s’exécute avec des privilèges [NetworkService](/windows/win32/services/networkservice-account).
- L’application de mise à jour qui est utilisée pour mettre à jour automatiquement l’agent d’authentification. Cette application s’exécute avec des privilèges [LocalSystem](/windows/win32/services/localsystem-account).

>[!IMPORTANT]
>Du point de vue de la sécurité, les administrateurs doivent traiter le serveur exécutant l’agent PTA comme s’il s’agissait d’un contrôleur de domaine.  Les serveurs d’agent PTA doivent être renforcés de la même manière que décrit dans [Sécurisation des contrôleurs de domaine contre les attaques](/windows-server/identity/ad-ds/plan/security-best-practices/securing-domain-controllers-against-attack)

### <a name="authentication-agent-registration"></a>Inscription de l’agent d’authentification

Une fois l’agent d’authentification installé, il doit être inscrit auprès d’Azure AD. Azure AD affecte à chaque agent d’authentification un certificat d’identité numérique unique qu’il peut utiliser pour sécuriser la communication avec Azure AD.

La procédure d’inscription lie également l’agent d’authentification à votre locataire. Azure AD sait ainsi que cet agent d’authentification spécifique est le seul autorisé à gérer les demandes de validation de mot de passe de votre locataire. Cette procédure est répétée pour chaque nouvel agent d’authentification que vous inscrivez.

Les agents d’authentification suivent la procédure ci-dessous pour s’inscrire auprès d’Azure AD :

![Inscription de l’agent](./media/how-to-connect-pta-security-deep-dive/pta1.png)

1. Azure AD demande tout d’abord à un administrateur général de se connecter à Azure AD avec ses informations d’identification. Pendant la connexion, l’agent d’authentification acquiert un jeton d’accès qu’il peut utiliser pour le compte de l’administrateur général.
2. L’agent d’authentification génère ensuite une paire de clés : une clé publique et une clé privée.
    - Cette paire de clés est générée à l’aide du chiffrement standard RSA 2 048 bits.
    - La clé privée reste sur le serveur local sur lequel l’agent d’authentification se trouve.
3. L’agent d’authentification effectue une demande d’inscription auprès d’Azure AD via le protocole HTTPS, les composants suivants étant inclus dans la demande :
    - Le jeton d’accès obtenu à l’étape 1.
    - La clé publique générée à l’étape 2.
    - Une demande de signature de certificat (CSR ou demande de certificat). Cette demande s’applique à un certificat d’identité numérique, avec Azure AD comme autorité de certification (AC).
4. Azure AD valide le jeton d’accès dans la demande d’inscription et vérifie que la demande provient d’un administrateur général.
5. Azure AD signe ensuite et renvoie un certificat d’identité numérique à l’agent d’authentification.
    - L’autorité de certification racine dans Azure AD est utilisée pour signer le certificat. 

      > [!NOTE]
      > Cette autorité de certification ne figure _pas_ dans le magasin des autorités de certification racine reconnues approuvées Windows.
    - L’autorité de certification est utilisée uniquement par la fonctionnalité d’authentification directe. L’autorité de certification est utilisée uniquement pour signer les CSR lors de l’inscription de l’agent d’authentification.
    -  Aucun des autres services Azure AD n’utilise cette autorité de certification.
    - L’objet du certificat (Nom unique ou DN) est défini sur votre ID de locataire. Ce DN est un GUID qui identifie de manière unique votre locataire. Avec ce DN, le certificat ne peut donc être utilisé qu’avec votre locataire.
6. Azure AD stocke la clé publique de l’agent d’authentification dans une base de données dans Azure SQL Database, qui est accessible uniquement à Azure AD.
7. Le certificat (émis à l’étape 5) est stocké sur le serveur local dans le magasin de certificats Windows (plus précisément à l’emplacement [CERT_SYSTEM_STORE_LOCAL_MACHINE](/windows/win32/seccrypto/system-store-locations#CERT_SYSTEM_STORE_LOCAL_MACHINE)). Il est utilisé par l’agent d’authentification et les applications du programme de mise à jour.

### <a name="authentication-agent-initialization"></a>Initialisation de l’agent d’authentification

Lorsque l’agent d’authentification démarre, pour la première fois après son inscription ou après le redémarrage d’un serveur, il recherche un moyen de communiquer en toute sécurité avec le service Azure AD et accepte les demandes de validation de mot de passe.

![Initialisation de l’agent](./media/how-to-connect-pta-security-deep-dive/pta2.png)

Voici comment les agents d’authentification sont initialisés :

1. L’agent d’authentification adresse une demande de démarrage sortante à Azure AD. 
    - Cette demande est effectuée sur le port 443 et sur un canal HTTPS mutuellement authentifié. La demande utilise le même certificat que celui émis lors de l’inscription de l’agent d’authentification.
2. Azure AD répond à la demande en fournissant une clé d’accès à une file d’attente Azure Service Bus qui est propre à votre locataire et qui est identifiée par votre ID de locataire.
3. L’agent d’authentification établit une connexion HTTPS sortante persistante (sur le port 443) avec la file d’attente. 
    - L’agent d’authentification est maintenant prêt à récupérer et à gérer les demandes de validation de mot de passe.

Si vous avez inscrit plusieurs agents d’authentification sur votre locataire, la procédure d’initialisation permet de s’assurer que chacun d’eux se connecte à la même file d’attente Service Bus. 

## <a name="process-sign-in-requests"></a>Traiter les demandes de connexion 

Le diagramme suivant montre comment l’authentification directe traite les demandes de connexion de l’utilisateur.

![Traiter la connexion](./media/how-to-connect-pta-security-deep-dive/pta3.png)

L’authentification directe traite une demande de connexion de l’utilisateur comme suit : 

1. Un utilisateur tente d’accéder à une application, par exemple, [Outlook Web App](https://outlook.office365.com/owa).
2. Si l’utilisateur n’est pas déjà connecté, l’application redirige le navigateur vers la page de connexion d’Azure AD.
3. Le service Azure AD STS répond avec la page **Connexion utilisateur**.
4. L’utilisateur entre son nom d’utilisateur dans la page **Connexion utilisateur**, puis sélectionne le bouton **Suivant**.
5. L’utilisateur entre son mot de passe dans la page **Connexion utilisateur**, puis sélectionne le bouton **Se connecter**.
6. Le nom d’utilisateur et le mot de passe sont envoyés à Azure AD STS dans une requête POST HTTPS.
7. Azure AD STS récupère les clés publiques de tous les agents d’authentification inscrits sur votre locataire à partir d’Azure SQL Database et les utilise pour chiffrer le mot de passe.
    - Azure AD STS produit « N » valeurs de mot de passe chiffré pour « N » agents d’authentification inscrits sur votre locataire.
8. Azure AD STS place la demande de validation de mot de passe (qui inclut le nom d’utilisateur et les valeurs de mot de passe chiffré) dans la file d’attente Service Bus propre à votre locataire.
9. Étant donné que les agents d’authentification initialisés sont connectés en permanence à la file d’attente Service Bus, l’un des agents d’authentification disponibles récupère la demande de validation de mot de passe.
10. L’agent d’authentification localise la valeur de mot de passe chiffré propre à sa clé publique, à l’aide d’un identificateur, et la déchiffre à l’aide de sa clé privée.
11. L’agent d’authentification tente de valider le nom d’utilisateur et le mot de passe dans l’annuaire Active Directory local à l’aide de l’[API LogonUser Win32](/windows/win32/api/winbase/nf-winbase-logonusera) avec le paramètre **dwLogonType** défini sur **LOGON32_LOGON_NETWORK**. 
    - Cette API est la même que celle utilisée par les services de fédération Active Directory (AD FS) pour connecter les utilisateurs dans un scénario de connexion fédérée.
    - Cette API sur le processus de résolution standard de Windows Server pour localiser le contrôleur de domaine.
12. L’agent d’authentification reçoit le résultat depuis Active Directory, tel que réussite, nom d’utilisateur ou mot de passe incorrect, ou mot de passe expiré.

   > [!NOTE]
   > Si l’agent d’authentification échoue lors du processus de connexion, la requête de connexion toute entière est supprimée. Les requêtes de connexion d'un agent d'authentification ne sont pas transférées vers un autre agent d'authentification local. Ces agents communiquent uniquement avec le cloud, et non entre eux.
   
13. L’agent d’authentification retransmet le résultat à Azure AD STS via un canal HTTPS mutuellement authentifié sortant sur le port 443. L’authentification mutuelle utilise le certificat précédemment émis pour l’agent d’authentification lors de l’inscription.
14. Azure AD STS vérifie que ce résultat est mis en corrélation avec la demande de connexion spécifique sur votre locataire.
15. Azure AD STS poursuit avec la procédure de connexion configurée. Par exemple, si la validation du mot de passe aboutit, l’utilisateur peut devoir s’authentifier via Multi-Factor Authentication ou être redirigé vers l’application.

## <a name="operational-security-of-the-authentication-agents"></a>Sécurité opérationnelle des agents d’authentification

Pour vous assurer que l’authentification directe reste sécurisée sur le plan opérationnel, Azure AD renouvelle régulièrement les certificats des agents d’authentification. Azure AD déclenche les renouvellements. Les renouvellements ne sont pas régis par les agents d’authentification proprement dits.

![Sécurité opérationnelle](./media/how-to-connect-pta-security-deep-dive/pta4.png)

Pour renouveler la relation d’approbation d’un agent d’authentification avec Azure AD :

1. L’agent d’authentification effectue régulièrement un test ping dans Azure AD (toutes les heures) pour vérifier s’il est temps de renouveler son certificat. Le certificat est renouvelé 30 jours avant son expiration.
    - Cette vérification est effectuée via un canal HTTPS mutuellement authentifié et utilise le même certificat que celui émis lors de l’inscription.
2. Si le service indique qu’il est temps de procéder au renouvellement, l’agent d’authentification génère une nouvelle paire de clés : une clé publique et une clé privée.
    - Ces clés sont générées à l’aide du chiffrement standard RSA 2 048 bits.
    - La clé privée ne quitte jamais le serveur local.
3. L’agent d’authentification effectue ensuite une demande de renouvellement de certificat auprès d’Azure AD via le protocole HTTPS, les composants suivants étant inclus dans la demande :
    - Le certificat existant qui est récupéré à partir de l’emplacement CERT_SYSTEM_STORE_LOCAL_MACHINE dans le magasin de certificats Windows. Comme aucun administrateur général n’est impliqué dans cette procédure, aucun jeton d’accès n’est nécessaire pour le compte de l’administrateur général.
    - La clé publique générée à l’étape 2.
    - Une demande de signature de certificat (CSR ou demande de certificat). Cette demande s’applique à un nouveau certificat d’identité numérique, avec Azure AD comme autorité de certification.
4. Azure AD valide le certificat existant dans la demande de renouvellement de certificat. Il vérifie ensuite que la demande provient d’un agent d’authentification inscrit sur votre locataire.
5. Si le certificat existant est toujours valide, Azure AD signe un nouveau certificat d’identité numérique et le délivre à l’agent d’authentification. 
6. Si le certificat existant est arrivé à expiration, Azure AD supprime l’agent d’authentification dans la liste des agents d’authentification inscrits de votre locataire. Un administrateur général doit alors installer et inscrire un nouvel agent d’authentification manuellement.
    - Utilisez l’autorité de certification racine Azure AD pour signer le certificat.
    - Définissez l’objet du certificat (Nom unique ou DN) sur votre ID de locataire, qui est un GUID qui identifie de manière unique votre locataire. Avec le DN, le certificat ne peut être utilisé qu’avec votre locataire.
6. Azure AD stocke la nouvelle clé publique de l’agent d’authentification dans une base de données dans Azure SQL Database, qui accessible uniquement à Azure AD. Il invalide également l’ancienne clé publique associée à l’agent d’authentification.
7. Le nouveau certificat (émis à l’étape 5) est ensuite stocké sur le serveur dans le magasin de certificats Windows (plus précisément à l’emplacement [CERT_SYSTEM_STORE_CURRENT_USER](/windows/win32/seccrypto/system-store-locations#CERT_SYSTEM_STORE_CURRENT_USER)).
    - Étant donné que la procédure de renouvellement d’approbation se produit de manière non interactive (sans la présence de l’administrateur général), l’agent d’authentification n’a plus accès pour mettre à jour le certificat existant à l’emplacement CERT_SYSTEM_STORE_LOCAL_MACHINE. 
    
   > [!NOTE]
   > Ce procédure ne supprime pas le certificat proprement dit de l’emplacement CERT_SYSTEM_STORE_LOCAL_MACHINE.
8. Le nouveau certificat est utilisé dorénavant pour l’authentification. Tous les renouvellements ultérieurs du certificat remplacent le certificat dans CERT_SYSTEM_STORE_LOCAL_MACHINE.

## <a name="auto-update-of-the-authentication-agents"></a>Mise à jour automatique des agents d’authentification

L’application de mise à jour met automatiquement à jour l’agent d’authentification lors de la sortie d’une nouvelle version (avec résolution des bogues ou amélioration des performances). L’application de mise à jour ne traite pas les requêtes de validation de mot de passe de votre locataire.

Azure AD héberge la nouvelle version du logiciel en tant que **package Windows Installer (MSI)** signé. Le package MSI est signé à l’aide de [Microsoft Authenticode](/previous-versions/windows/internet-explorer/ie-developer/platform-apis/ms537359(v=vs.85)) avec l’algorithme de chiffrement SHA256. 

![Mise à jour automatique](./media/how-to-connect-pta-security-deep-dive/pta5.png)

Pour mettre à jour automatiquement un agent d’authentification :

1. L’application du programme de mise à jour effectue un test ping dans Azure AD toutes les heures pour vérifier si une nouvelle version de l’agent d’authentification est disponible. 
    - Cette vérification est effectuée via un canal HTTPS mutuellement authentifié à l’aide du même certificat que celui émis lors de l’inscription. L’agent d’authentification et le programme de mise à jour partagent le certificat stocké sur le serveur.
2. Si une nouvelle version est disponible, Azure AD retourne le MSI signé au programme de mise à jour.
3. Le programme de mise à jour vérifie que le MSI est signé par Microsoft.
4. Le programme de mise à jour exécute le MSI. Cette action implique les étapes suivantes :

   > [!NOTE]
   > Le programme de mise à jour s’exécute avec les privilèges [Système Local](/windows/win32/services/localsystem-account).

    - Arrête le service Agent d’authentification
    - Installe la nouvelle version de l’agent d’authentification sur le serveur
    - Redémarre le service Agent d’authentification

>[!NOTE]
>Si vous avez inscrit plusieurs agents d’authentification sur votre locataire, Azure AD ne renouvelle pas leurs certificats ou ne les met pas à jour en même temps. Azure AD effectue ces procédures une par une pour s’assurer de la haute disponibilité des requêtes de connexion.
>


## <a name="next-steps"></a>Étapes suivantes
- [Limitations actuelles](how-to-connect-pta-current-limitations.md) : Découvrez les scénarios pris en charge et ceux qui ne le sont pas.
- [Démarrage rapide](how-to-connect-pta-quick-start.md) : soyez opérationnel sur l’authentification directe Azure AD.
- [Migrer à partir d’AD FS vers l’authentification directe](https://aka.ms/adfstoptadpdownload) : guide détaillé de la migration d’AD FS (ou d’autres technologies de fédération) vers l’authentification directe.
- [Verrouillage intelligent](../authentication/howto-password-smart-lockout.md) : configurez la fonctionnalité Verrouillage intelligent sur votre locataire pour protéger les comptes d’utilisateur.
- [Fonctionnement](how-to-connect-pta-how-it-works.md) : découvrez les principes de fonctionnement de l’authentification directe Azure AD.
- [Forum Aux Questions](how-to-connect-pta-faq.md) : obtenez des réponses aux questions fréquemment posées.
- [Résoudre les problèmes](tshoot-connect-pass-through-authentication.md) : apprenez à résoudre les problèmes courants liés à la fonctionnalité d’authentification directe.
- [Authentification unique fluide Azure AD](how-to-connect-sso.md) : explorez en détail cette fonctionnalité complémentaire.
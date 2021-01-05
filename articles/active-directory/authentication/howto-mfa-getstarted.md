---
title: Considérations relatives au déploiement d'Azure AD Multi-Factor Authentication
description: Familiarisez-vous avec les considérations et la stratégie de déploiement pour une implémentation réussie d'Azure AD Multi-Factor Authentication
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
ms.openlocfilehash: c9ee81abd7cd0268a7cbd6b16aa6065ec7b54bef
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861304"
---
# <a name="plan-an-azure-ad-multi-factor-authentication-deployment"></a>Planifier un déploiement d'Azure AD Multi-Factor Authentication

Les personnes se connectent aux ressources d’organisation dans des scénarios plus en plus complexes. Les personnes se connectent à partir d’appareils personnels, publics et appartenant à l’organisation, sur le réseau d’entreprise ou non, à l’aide de smartphones, tablettes, PC et ordinateurs portables, souvent sur plusieurs plateformes. Dans ce monde toujours connecté, aux multiples appareils et plateformes, la sécurité des comptes d’utilisateur est plus importante que jamais. Les mots de passe, quelle que soit leur complexité, utilisés sur les appareils, les réseaux et les plateformes ne suffisent plus à garantir la sécurité des comptes d’utilisateur, en particulier lorsque les utilisateurs ont tendance à réutiliser les mots de passe sur plusieurs comptes. Des attaques sophistiquées d’hameçonnage et d’ingénierie sociale peuvent conduire à la publication et à la vente des noms d’utilisateur et des mots de passe sur le web invisible.

[Azure AD Multi-Factor Authentication (MFA)](concept-mfa-howitworks.md) permet de protéger l'accès aux données et aux applications. Il fournit une couche supplémentaire de sécurité à l’aide d’une deuxième forme d’authentification. Les organisations peuvent utiliser l’[accès conditionnel](../conditional-access/overview.md) pour que la solution réponde à leurs besoins spécifiques.

Ce guide de déploiement explique comment planifier puis tester un déploiement d'Azure AD Multi-Factor Authentication.

Pour avoir un aperçu rapide d'Azure AD Multi-Factor Authentication, puis revenir en arrière afin d'examiner les considérations relatives au déploiement :

> [!div class="nextstepaction"]
> [Activer l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md)

## <a name="prerequisites"></a>Prérequis

Avant de lancer un déploiement d'Azure AD Multi-Factor Authentication, certains éléments doivent être pris en compte.

| Scénario | Configuration requise |
| --- | --- |
| Environnement d’identité **cloud uniquement** avec authentification moderne | **Aucune tâche de configuration requise supplémentaire** |
| Scénarios d’identité **hybride** | [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) est déployé, et les identités des utilisateurs sont synchronisées ou fédérées avec Azure Active Directory Connect en local avec Azure Active Directory. |
| Applications héritées locales publiées pour l’accès au cloud | Le [proxy d’application](../manage-apps/application-proxy.md) Azure AD est déployé. |
| Utilisation d'Azure AD MFA avec l'authentification RADIUS | Un [serveur NPS (Network Policy Server)](howto-mfa-nps-extension.md) est déployé. |
| Les utilisateurs disposent de Microsoft Office 2010 ou une version antérieure, ou d’Apple Mail pour iOS 11 ou une version antérieure. | Effectuez une mise à niveau vers [Microsoft Office 2013](https://support.microsoft.com/help/4041439/modern-authentication-configuration-requirements-for-transition-from-o) ou une version ultérieure, et Apple Mail pour iOS 12 ou une version ultérieure. L’accès conditionnel n’est pas pris en charge par les protocoles d’authentification hérités. |

## <a name="plan-user-rollout"></a>Planifier le déploiement des utilisateurs

Votre plan de déploiement MFA doit inclure un déploiement pilote suivi par des vagues de déploiement compatibles avec votre capacité de prise en charge. Commencez le déploiement en appliquant vos stratégies d’accès conditionnel à un petit groupe d’utilisateurs pilotes. Après avoir évalué l’impact sur les utilisateurs pilotes, le processus utilisé et les comportements d’inscription, vous pouvez ajouter d’autres groupes à la stratégie ou ajouter d’autres utilisateurs aux groupes existants.

### <a name="user-communications"></a>Communication avec les utilisateurs

Il est essentiel d'informer les utilisateurs des modifications à venir, des conditions d'inscription à Azure AD MFA et de toutes les actions utilisateur nécessaires, à travers des communications planifiées. Nous vous recommandons de développer ces communications conjointement avec des représentants d’au sein de votre organisation, comme les services Communications, Gestion des changements ou Ressources humaines.

Microsoft fournit des [modèles de communication](https://aka.ms/mfatemplates) et une [documentation destinée aux utilisateurs finaux](../user-help/security-info-setup-signin.md) pour vous aider à rédiger vos communications. Vous pouvez diriger les utilisateurs vers [https://myprofile.microsoft.com](https://myprofile.microsoft.com) pour qu’ils s’inscrivent directement en sélectionnant les liens **Informations de sécurité** sur cette page.

## <a name="deployment-considerations"></a>Points à prendre en considération pour le déploiement

Azure AD Multi-Factor Authentication est déployé en appliquant des stratégies avec l'accès conditionnel. Une stratégie d’accès conditionnel peuvent exiger que les utilisateurs procèdent à une authentification multifacteur lorsque certains critères sont remplis :

* Tous les utilisateurs, un utilisateur, un membre de groupe ou un rôle affecté
* Accès à une application cloud spécifique
* Plateforme d’appareil.
* État de l’appareil
* Emplacement réseau ou adresse IP géolocalisée
* Applications clientes
* Risque de connexion (requiert Identity Protection)
* Conformité de l’appareil
* Appareil joint Azure AD hybride
* Application cliente approuvée

Utilisez les posters personnalisables et les modèles d’e-mails disponibles dans les [documents déploiement de l’authentification multifacteur](https://www.microsoft.com/download/details.aspx?id=57600&WT.mc_id=rss_alldownloads_all) pour déployer l’authentification multifacteur dans votre organisation.

## <a name="enable-multi-factor-authentication-with-conditional-access"></a>Activer l’authentification multifacteur avec l’accès conditionnel

Les stratégies d’accès conditionnel forcent l’inscription, obligeant les utilisateurs non inscrits à effectuer l’inscription à la première connexion, un facteur de sécurité important.

[Azure AD Identity Protection](../identity-protection/howto-identity-protection-configure-risk-policies.md) contribue à la fois à une stratégie d'inscription et à des stratégies de détection et de correction automatisées des risques pour l'historique d'Azure AD Multi-Factor Authentication. Vous pouvez créer des stratégies pour imposer des modifications de mot de passe lorsqu’une identité risque d’être compromise ou exiger l’authentification multifacteur lors d’une connexion est considérée à risque par les [événements](../identity-protection/overview-identity-protection.md) suivants :

* Informations d’identification divulguées
* Connexions depuis des adresses IP anonymes
* Voyage impossible vers des emplacements inhabituels
* Connexions depuis des emplacements non connus
* Connexions depuis des appareils infectés
* Connexions depuis des adresses IP avec des activités suspectes

Certaines des détections des risques détectées par Azure Active Directory Identity Protection se produisent en temps réel, et d’autres nécessitent un traitement en mode hors connexion. Les administrateurs peuvent choisir de bloquer les utilisateurs qui présentent des comportements à risque et y remédier manuellement, exiger une modification de mot de passe ou demander une authentification multifacteur dans le cadre de leurs stratégies d’accès conditionnel.

## <a name="define-network-locations"></a>Définir les emplacements réseau

Nous recommandons aux organisations d’utiliser l’accès conditionnel pour définir leur réseau à l’aide d’[emplacements nommés](../conditional-access/location-condition.md#named-locations). Si votre organisation utilise Identity Protection, envisagez d’appliquer des stratégies en fonction des risques au lieu des emplacements nommés.

### <a name="configuring-a-named-location"></a>Configuration d’un emplacement nommé

1. Ouvrez **Azure Active Directory** dans le portail Azure.
2. Sélectionnez **Sécurité**
3. Dans **Gérer**, choisissez **Emplacements nommés**
4. Sélectionnez **Nouvel emplacement**
5. Dans le champ **Nom**, indiquez un nom explicite.
6. Indiquez si vous définissez l’emplacement à l’aide de *plages d’adresses IP* ou de *pays/régions*
   1. Si vous utilisez des *plages d’adresses IP*
      1. Décidez s’il faut *marquer l’emplacement comme approuvé*. La connexion à partir d’un emplacement approuvé réduit le risque de connexion d’un utilisateur. Marquez uniquement cet emplacement comme approuvé si vous savez que les plages d’adresses IP saisies sont établies et crédibles dans votre organisation.
      2. Spécifiez les plages d’adresses IP.
   2. Si vous utilisez des *pays/régions*
      1. Développez le menu déroulant et sélectionnez les pays ou régions que vous souhaitez définir pour cet emplacement nommé.
      2. Décidez s’il faut inclure des *zones inconnues*. Les zones inconnues sont les adresses IP qui ne peuvent être mappées à aucun pays ou aucune région.
7. Sélectionnez **Créer**

## <a name="plan-authentication-methods"></a>Planifier les méthodes d’authentification

Les administrateurs peuvent choisir les [méthodes d’authentification](../authentication/concept-authentication-methods.md) qu’ils souhaitent mettre à disposition des utilisateurs. Il est important autoriser plusieurs méthodes d’authentification afin que les utilisateurs aient une méthode de secours au cas où la principale méthode n’est pas disponible. Les administrateurs peuvent activer les méthodes suivantes :

> [!TIP]
> Microsoft recommande d'utiliser Microsoft Authenticator (application mobile) comme méthode principale pour Azure AD Multi-Factor Authentication afin de bénéficier d'une expérience utilisateur plus sécurisée et améliorée. L’application Microsoft Authenticator [respecte](https://azure.microsoft.com/resources/microsoft-nist/) également les niveaux d’assurance relatif à l’authentification du National Institute of Standards and Technology. 

### <a name="notification-through-mobile-app"></a>Notification via une application mobile

Une notification Push est envoyée à l’application Microsoft Authenticator sur votre appareil mobile. L’utilisateur consulte la notification et sélectionne **Approuver** pour valider la vérification. Les notifications Push sur l’application mobile constituent l’option la moins intrusive pour les utilisateurs. Elles sont également l’option la plus fiable et sécurisée, car elles utilisent une connexion de données plutôt que le réseau de téléphonie.

> [!NOTE]
> Si le personnel de votre organisation travaille ou voyage en Chine, la méthode de **notification via application mobile** sur les **appareils Android** ne fonctionne pas dans ce pays ou cette région. D’autres méthodes doivent être proposées aux utilisateurs.

### <a name="verification-code-from-mobile-app"></a>Code de vérification de l’application mobile

Une application mobile telle que Microsoft Authenticator génère un nouveau code de vérification OATH toutes les 30 secondes. L’utilisateur entre ce code de vérification dans l’interface de connexion. L’option d’application mobile peut être utilisée que le téléphone dispose ou non d’un signal cellulaire ou données.

### <a name="call-to-phone"></a>Appel vers le téléphone

Un appel vocal automatisé est passé à l’utilisateur. L’utilisateur répond à l’appel et appuie sur **#** sur le clavier du téléphone pour valider son authentification. L’appel téléphonique est une méthode de sauvegarde très efficace comme alternative aux notifications et au code de vérification à partir d’une application mobile.

### <a name="text-message-to-phone"></a>Message texte vers le téléphone

L’utilisateur reçoit un SMS avec le code de vérification et il est invité à entrer ce code dans l’interface de connexion.

### <a name="choose-verification-options"></a>Choisir les options de vérification

1. Accédez à **Azure Active Directory**, **Utilisateurs**, **Multi-Factor Authentication**.

   ![Accès au portail Multi-Factor Authentication à partir du panneau Utilisateurs Azure AD dans le portail Azure](media/howto-mfa-getstarted/users-mfa.png)

1. Sous le nouvel onglet qui vient de s’ouvrir dans votre navigateur, accédez aux **paramètres de service**.
1. Sous **options de vérification**, cochez toutes les cases pour les méthodes disponibles pour les utilisateurs.

   ![Configurer des méthodes de vérification dans l’onglet de paramètres de service Authentification multifacteur](media/howto-mfa-getstarted/mfa-servicesettings-verificationoptions.png)

1. Cliquez sur **Save**(Enregistrer).
1. Fermez l’onglet **paramètres de service**.

## <a name="plan-registration-policy"></a>Planifier la stratégie d'inscription

Les administrateurs doivent déterminer comment les utilisateurs inscrivent leurs méthodes. Les organisations doivent [activer la nouvelle expérience d'inscription combinée](howto-registration-mfa-sspr-combined.md) pour Azure AD MFA et la réinitialisation de mot de passe en libre-service (SSPR). La fonctionnalité SSPR permet aux utilisateurs de réinitialiser leur mot de passe de manière sécurisée avec les mêmes méthodes que l’authentification multifacteur. Nous vous recommandons cette inscription combinée car il s'agit d'une excellente expérience pour les utilisateurs puisqu'elle permet une inscription unique pour les deux services. L'activation des mêmes méthodes pour SSPR et Azure AD MFA permet aux utilisateurs de s'inscrire pour utiliser ces deux fonctionnalités.

### <a name="registration-with-identity-protection"></a>Inscription avec Identity Protection

Si votre organisation utilise Azure Active Directory Identity Protection, [configurez la stratégie d’inscription MFA](../identity-protection/howto-identity-protection-configure-mfa-policy.md) pour inviter les utilisateurs à s’inscrire la prochaine fois qu’ils se connectent de manière interactive.

### <a name="registration-without-identity-protection"></a>Inscription sans Identity Protection

Si votre organisation n’a pas de licences qui activent Identity Protection, les utilisateurs sont invités à s’inscrire la prochaine fois que l’authentification multifacteur est requise lors de la connexion. Les utilisateurs ne peuvent pas s’inscrire à MFA s’ils n’utilisent des applications protégées avec MFA. Il est important que tous les utilisateurs s’inscrivent, de sorte que les malfaiteurs ne puissent pas deviner leur mot de passe et s’inscrire à MFA en leur nom, prenant alors le contrôle du compte.

#### <a name="enforcing-registration"></a>Inscription forcée

En procédant comme suit, un accès conditionnel stratégie peut forcer les utilisateurs à s’inscrire à MFA.

1. Créez un groupe et ajoutez tous les utilisateurs qui ne sont pas encore inscrits.
2. À l’aide de l’accès conditionnel, forcez l’authentification multifacteur pour ce groupe lors de l’accès à toutes les ressources.
3. Périodiquement, réévaluez l’appartenance au groupe et retirez les utilisateurs qui se sont inscrits.

Vous pouvez identifier les utilisateurs inscrits et non inscrits à Azure AD MFA à l'aide des commandes PowerShell qui reposent sur le [module MSOnline PowerShell](/powershell/azure/active-directory/install-msonlinev1).

#### <a name="identify-registered-users"></a>Identifier les utilisateurs inscrits

```PowerShell
Get-MsolUser -All | where {$_.StrongAuthenticationMethods -ne $null} | Select-Object -Property UserPrincipalName | Sort-Object userprincipalname 
```

#### <a name="identify-non-registered-users"></a>Identifier les utilisateurs non inscrits

```PowerShell
Get-MsolUser -All | where {$_.StrongAuthenticationMethods.Count -eq 0} | Select-Object -Property UserPrincipalName | Sort-Object userprincipalname 
```

### <a name="convert-users-from-per-user-mfa-to-conditional-access-based-mfa"></a>Convertir des utilisateurs de l’authentification multifacteur par utilisateur à l’authentification multifacteur basée sur l’accès conditionnel

Si vos utilisateurs ont été activés à l'aide d'une authentification Azure AD MFA appliquée par Azure ou reposant sur l'utilisateur, la commande PowerShell suivante peut vous aider à passer à une authentification Azure AD MFA basée sur l'accès conditionnel.

Exécutez ce script PowerShell dans une fenêtre ISE ou enregistrez-le en tant que fichier `.PS1` à exécuter localement.

```PowerShell
# Sets the MFA requirement state
function Set-MfaState {

    [CmdletBinding()]
    param(
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $ObjectId,
        [Parameter(ValueFromPipelineByPropertyName=$True)]
        $UserPrincipalName,
        [ValidateSet("Disabled","Enabled","Enforced")]
        $State
    )

    Process {
        Write-Verbose ("Setting MFA state for user '{0}' to '{1}'." -f $ObjectId, $State)
        $Requirements = @()
        if ($State -ne "Disabled") {
            $Requirement =
                [Microsoft.Online.Administration.StrongAuthenticationRequirement]::new()
            $Requirement.RelyingParty = "*"
            $Requirement.State = $State
            $Requirements += $Requirement
        }

        Set-MsolUser -ObjectId $ObjectId -UserPrincipalName $UserPrincipalName `
                     -StrongAuthenticationRequirements $Requirements
    }
}

# Disable MFA for all users
Get-MsolUser -All | Set-MfaState -State Disabled
```

> [!NOTE]
> Nous avons récemment modifié le comportement et le script PowerShell ci-dessus en conséquence. Auparavant, le script enregistrait les méthodes MFA, désactivait l’authentification MFA et restaurait les méthodes. Ce n’est plus nécessaire maintenant que le comportement par défaut de désactivation n’efface plus les méthodes.

## <a name="plan-conditional-access-policies"></a>Planifier les stratégies d’accès conditionnel

Pour planifier votre stratégie d’accès conditionnel, qui déterminera quand l’authentification multifacteur et d’autres contrôles sont requis, reportez-vous à [Stratégies d’accès conditionnel courantes](../conditional-access/concept-conditional-access-policy-common.md).

Il est important que vous évitiez de perdre l’accès à votre locataire Azure AD. Vous pouvez pallier l’impact d’un défaut d’accès administratif [en créant plusieurs comptes d’accès d’urgence dans votre locataire](../roles/security-emergency-access.md) et en les excluant de la stratégie d’accès conditionnel.

### <a name="create-conditional-access-policy"></a>Créer une stratégie d’accès conditionnel

1. Connectez-vous au [portail Azure](https://portal.azure.com) à l’aide d’un compte d’administrateur général.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
   ![Créer une stratégie d’accès conditionnel pour activer l’authentification multifacteur pour les utilisateurs du portail Azure dans le groupe pilote](media/howto-mfa-getstarted/conditionalaccess-newpolicy.png)
1. Entrez un nom explicite pour votre stratégie.
1. Sous **utilisateurs et groupes** :
   * Dans l’onglet **Inclure**, sélectionnez la case d’option **Tous les utilisateurs**
   * Dans l’onglet **Exclure**, cochez la case **Utilisateurs et groupes** et choisissez vos comptes d’accès d’urgence.
   * Cliquez sur **Done**.
1. Sous **Applications Cloud**, sélectionnez la case d’option **Toutes les applications cloud**.
   * FACULTATIF : sous l’onglet **Exclure**, choisissez les applications cloud pour lesquelles votre organisation ne nécessite pas d’authentification multifacteur.
   * Cliquez sur **Done**.
1. Sous la section **Conditions** :
   * FACULTATIF : si vous avez activé Azure Identity Protection, vous pouvez choisir d’évaluer le risque de connexion dans le cadre de la stratégie.
   * FACULTATIF : si vous avez configuré des emplacements approuvés ou des emplacements nommés, vous pouvez spécifier l’inclusion ou l’exclusion de ces emplacements à partir de la stratégie.
1. Sous **Accord**, assurez-vous que la case d’option **Accorder l’accès** est sélectionnée.
    * Cochez la case **Exiger une authentification multifacteur**.
    * Cliquez sur **Sélectionner**.
1. Ignorez la section **Session**.
1. Basculez **Activer la stratégie** sur **Activé**.
1. Cliquez sur **Créer**.

## <a name="plan-integration-with-on-premises-systems"></a>Planifier l’intégration avec les systèmes locaux

Certaines applications héritées et locales qui ne s’authentifient pas directement auprès d’Azure AD nécessitent des étapes supplémentaires pour utiliser l’authentification multifacteur, notamment :

* Applications locales héritées, qui devront utiliser le proxy d’application.
* Applications RADIUS locales, qui devront utiliser l’adaptateur MFA avec un serveur NPS.
* Applications AD FS locales, qui devront utiliser l’adaptateur MFA avec AD FS 2016 ou version ultérieure.

Les applications qui s’authentifient directement auprès d’Azure AD et disposent d’une authentification moderne (WS-Fed, SAML, OAuth, OpenID Connect) peuvent tirer parti des stratégies d’accès conditionnel sans intermédiaire.

### <a name="use-azure-ad-mfa-with-azure-ad-application-proxy"></a>Utiliser Azure AD MFA avec le proxy d'application Azure AD

Les applications locales peuvent être publiées sur votre locataire Azure AD via le [proxy d'application Azure AD](../manage-apps/application-proxy.md) et tirer parti d'Azure AD Multi-Factor Authentication si elles sont configurées pour utiliser la pré-authentification Azure AD.

Ces applications sont soumises aux stratégies d'accès conditionnel qui imposent l'authentification Azure AD MFA, tout comme n'importe quelle autre application intégrée à Azure AD.

De même, si l'authentification Azure AD MFA est appliquée pour toutes les connexions utilisateur, les applications locales publiées avec le proxy d'application Azure AD seront protégées.

### <a name="integrating-azure-ad-multi-factor-authentication-with-network-policy-server"></a>Intégration d'Azure AD Multi-Factor Authentication au serveur NPS (Network Policy Server)

L'extension NPS (Network Policy Server) d'Azure AD MFA permet d'ajouter des fonctionnalités MFA basées sur le cloud à votre infrastructure d'authentification à l'aide de vos serveurs existants. Avec cette extension NPS, vous pouvez ajouter des vérifications par appel téléphonique, SMS ou application téléphonique à votre flux d’authentification. Cette intégration présente les limites suivantes :

* Avec le protocole CHAPv2, seuls les appels vocaux et les notifications Push de l’application d’authentification sont pris en charge.
* Les stratégies d’accès conditionnel ne peuvent pas être appliquées.

L'extension NPS joue le rôle d'adaptateur entre RADIUS et l'authentification Azure AD MFA basée sur le cloud pour fournir un second facteur d'authentification et protéger le [VPN](howto-mfa-nps-extension-vpn.md), les [connexions de passerelle des services Bureau à distance](howto-mfa-nps-extension-rdg.md) ou les autres applications compatibles avec RADIUS. Les utilisateurs qui s'inscrivent à Azure AD MFA dans cet environnement devront justifier toutes les tentatives d'authentification. L'absence de stratégies d'accès conditionnel signifie que l'authentification multifacteur est toujours requise.

#### <a name="implementing-your-nps-server"></a>Implémentation de votre serveur NPS (Network Policy Server)

Si une instance de NPS est déployée et en cours d'utilisation, consultez [Intégrer votre infrastructure NPS existante à Azure AD Multi-Factor Authentication](howto-mfa-nps-extension.md). Si vous configurez le serveur NPS pour la première fois, reportez-vous à [Serveur NPS (Network Policy Server)](/windows-server/networking/technologies/nps/nps-top) pour obtenir des instructions. Vous trouverez des conseils dans l'article [Résoudre les messages d'erreur liés à l'extension NPS d'Azure AD Multi-Factor Authentication](howto-mfa-nps-extension-errors.md).

#### <a name="prepare-nps-for-users-that-arent-enrolled-for-mfa"></a>Anticiper la présence d’utilisateurs qui ne sont pas inscrits pour l’authentification MFA sur le serveur NPS (Network Policy Server)

Choisissez ce qui doit se passer lorsque des utilisateurs qui ne sont pas inscrits auprès de MFA tentent de s'authentifier. Utilisez le paramètre de registre `REQUIRE_USER_MATCH` dans le chemin d’accès au registre `HKLM\Software\Microsoft\AzureMFA` pour contrôler le comportement de la fonctionnalité. Ce paramètre ne dispose que d’une seule option de configuration.

| Clé | Valeur | Default |
| --- | --- | --- |
| `REQUIRE_USER_MATCH` | TRUE/FALSE | Non défini (équivaut à TRUE) |

L’objectif de ce paramètre est de déterminer l’action à exécuter lorsqu’un utilisateur n’est pas inscrit pour l’authentification MFA. Les effets de la modification de ce paramètre sont répertoriés dans le tableau ci-dessous.

| Paramètres | État de l’utilisateur dans MFA | Effets |
| --- | --- | --- |
| Clé inexistante | Non inscrit | Authentification MFA réussie |
| Valeur définie sur True ou non définie | Non inscrit | Authentification MFA réussie |
| Clé définie sur False | Non inscrit | Authentification sans MFA |
| Clé définie sur False ou True | Inscrit | Authentification avec MFA obligatoire |

### <a name="integrate-with-active-directory-federation-services"></a>Intégration aux services de fédération Active Directory (AD FS)

Si votre organisation est fédérée avec Azure AD, vous pouvez utiliser [Azure AD Multi-Factor Authentication pour sécuriser les ressources AD FS](multi-factor-authentication-get-started-adfs.md), à la fois en local et dans le cloud. Azure AD MFA vous permet de réduire le nombre de mots de passe et constitue un moyen plus sûr de s'authentifier. Depuis Windows Server 2016, vous pouvez configurer Azure AD MFA pour l'authentification principale.

Contrairement à AD FS dans Windows Server 2012 R2, l'adaptateur AD FS 2016 Azure AD MFA s'intègre directement à Azure AD et ne requiert pas de serveur Azure MFA local. L'adaptateur Azure AD MFA est intégré à Windows Server 2016, et aucune installation supplémentaire n'est requise.

Lorsqu'Azure AD MFA est utilisé avec AD FS 2016 et que l'application cible est soumise à la stratégie d'accès conditionnel, des considérations supplémentaires s'appliquent :

* L’accès conditionnel est disponible lorsque l’application est une partie de confiance pour Azure AD, fédérée avec AD FS 2016 ou version ultérieure.
* L’accès conditionnel n’est pas disponible lorsque l’application est une partie de confiance pour AD FS 2016 ou AD FS 2019, et gérée ou fédérée avec AD FS 2016 ou AD FS 2019.
* L'accès conditionnel n'est pas non plus disponible lorsqu'AD FS 2016 ou AD FS 2019 est configuré pour utiliser Azure AD MFA comme méthode d'authentification principale.

#### <a name="ad-fs-logging"></a>Journalisation AD FS

La journalisation AD FS 2016 ou AD FS 2019 standard à la fois dans le journal de sécurité Windows et dans le journal de l’administrateur AD FS permet de collecter des informations sur les requêtes d’authentification et leur réussite ou échec. Les données de journal des événements au sein de ces événements indiquent si Azure AD MFA a été utilisé. Par exemple, un ID d’événement d’audit AD FS 1200 peut contenir :

```
<MfaPerformed>true</MfaPerformed>
<MfaMethod>MFA</MfaMethod>
```

#### <a name="renew-and-manage-certificates"></a>Renouveler et gérer des certificats

Sur chaque serveur AD FS, sous l'option Mon magasin de l'ordinateur local, il y aura un certificat Azure AD MFA auto-signé intitulé Microsoft AD FS Azure MFA, et contenant la date d'expiration de celui-ci. Vérifiez la période de validité de ce certificat sur chaque serveur AD FS pour déterminer la date d’expiration.

Si la période de validité de vos certificats arrive à expiration, [générez et confirmez un nouveau certificat MFA sur chaque serveur AD FS](/windows-server/identity/ad-fs/operations/configure-ad-fs-and-azure-mfa#configure-the-ad-fs-servers).

Les instructions suivantes vous expliquent comment gérer les certificats Azure AD MFA sur vos serveurs AD FS. Lorsque vous configurez AD FS avec Azure AD MFA, les certificats générés par la cmdlet PowerShell `New-AdfsAzureMfaTenantCertificate` sont valides pendant deux ans. Renouvelez les certificats et installez-les avant l’expiration pour éviter les interruptions de service d’authentification multifacteur.

## <a name="implement-your-plan"></a>Implémenter votre planification

Maintenant que vous avez planifié votre solution, vous pouvez l’implémenter en suivant les étapes ci-dessous :

1. Respectez les prérequis.
   1. Déployez [Azure AD Connect](../hybrid/whatis-hybrid-identity.md) pour les scénarios hybrides.
   1. Déployez le [proxy d’application Azure AD](../manage-apps/application-proxy.md) pour les applications locales publiées pour l’accès au cloud.
   1. Déployez [NPS](/windows-server/networking/technologies/nps/nps-top) pour l’authentification RADIUS.
   1. Vérifiez que les utilisateurs ont effectué la mise à niveau vers des versions prises en charge de Microsoft Office avec l’authentification moderne activée.
1. Configurez les [méthodes d’authentification](#choose-verification-options) choisies.
1. Définissez vos [emplacements réseau nommés](../conditional-access/location-condition.md#named-locations).
1. Sélectionnez des groupes pour commencer le déploiement de MFA.
1. Configurez vos [stratégies d’accès conditionnel](#create-conditional-access-policy).
1. Configurez votre stratégie d’inscription MFA.
   1. [MFA et SSPR combinés](howto-registration-mfa-sspr-combined.md)
   1. Avec [Identity Protection](../identity-protection/howto-identity-protection-configure-mfa-policy.md)
1. Envoyez des communications aux utilisateurs et faites en sorte qu’ils s’inscrivent sur [https://aka.ms/mfasetup](https://aka.ms/mfasetup).
1. [Suivez les inscriptions.](#identify-non-registered-users)

> [!TIP]
> Les utilisateurs du cloud gouvernemental peuvent s’inscrire sur [https://aka.ms/GovtMFASetup](https://aka.ms/GovtMFASetup)

## <a name="manage-your-solution"></a>Gérer votre solution

Rapports Azure AD MFA

Azure AD Multi-Factor Authentication fournit des rapports via le portail Azure :

| Rapport | Emplacement | Description |
| --- | --- | --- |
| Alertes relatives à l’utilisation et aux fraudes | Azure AD > Connexions | Fournit des informations sur l’utilisation globale, un récapitulatif par utilisateur, des informations détaillées sur les utilisateurs, ainsi que l’historique des alertes de fraude émises au cours de la plage de dates spécifiée. |

## <a name="troubleshoot-mfa-issues"></a>Résoudre les problèmes relatifs à MFA

Des solutions aux problèmes fréquemment rencontrés avec Azure AD MFA sont proposées dans l'article [Résolution des problèmes liés à Azure AD Multi-Factor Authentication](https://support.microsoft.com/help/2937344/troubleshooting-azure-multi-factor-authentication-issues) du centre de support Microsoft.

## <a name="next-steps"></a>Étapes suivantes

Pour avoir un aperçu d'Azure AD Multi-Factor Authentication, suivez ce tutoriel :

> [!div class="nextstepaction"]
> [Activer l’authentification multifacteur Azure AD](tutorial-enable-azure-mfa.md)

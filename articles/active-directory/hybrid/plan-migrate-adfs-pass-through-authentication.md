---
title: 'Azure AD Connect : Migrer depuis la fédération vers PTA pour Azure AD'
description: Cet article contient des informations sur le déplacement d’un environnement d’identité hybride de la fédération à l’authentification directe.
services: active-directory
author: billmath
manager: daveba
ms.reviewer: martincoetzer
ms.service: active-directory
ms.workload: identity
ms.topic: conceptual
ms.date: 05/29/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: a10c4c0e6e40636e4803e054155d6fdaa12a9366
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96858550"
---
# <a name="migrate-from-federation-to-pass-through-authentication-for-azure-active-directory"></a>Migrer de la fédération à l’authentification directe pour Azure Active Directory

Cet article décrit comment faire passer les domaines de votre organisation d’AD FS (Active Directory Federation Services) à l’authentification directe.

> [!NOTE]
> Modifier votre méthode d’authentification implique une planification, des tests et de possibles temps d’arrêt. Le [lancement intermédiaire](how-to-connect-staged-rollout.md) fournit un autre moyen de tester et d’effectuer une migration progressive de la fédération vers l’authentification cloud à l’aide de l’authentification directe.
> 
> Si vous prévoyez d’utiliser le lancement intermédiaire, n’oubliez pas de désactiver les fonctionnalités de lancement intermédiaire une fois que vous avez terminé le basculement.  Pour davantage d’informations, consultez [Migrer vers l’authentification cloud avec un lancement intermédiaire](how-to-connect-staged-rollout.md)


## <a name="prerequisites-for-migrating-to-pass-through-authentication"></a>Prérequis pour la migration vers l’authentification directe

Les prérequis suivants sont nécessaires pour migrer de l’utilisation d’AD FS vers l’utilisation de l’authentification directe.

### <a name="update-azure-ad-connect"></a>Mettre à jour Azure AD Connect

Pour pouvoir effectuer les étapes nécessaires pour migrer vers l’utilisation de l’authentification directe, vous devez disposer d’[Azure Active Directory Connect](https://www.microsoft.com/download/details.aspx?id=47594) (Azure AD Connect) 1.1.819.0 ou ultérieur. Dans Azure AD Connect 1.1.819.0, la façon dont conversion de la connexion est effectuée change considérablement. Le temps total nécessaire pour depuis AD FS vers l’authentification cloud dans cette version est réduit d’une durée potentielle de plusieurs heures à quelques minutes.

> [!IMPORTANT]
> Vous pouvez lire dans des documentations, des outils et des blogs obsolètes que la conversion de l’utilisateur est nécessaire quand vous convertissez des domaines de l’identité fédérée à l’identité managée. La *conversion des utilisateurs* n’est plus nécessaire. Microsoft s’emploie à mettre à jour la documentation et les outils afin de refléter ce changement.

Pour mettre à jour Azure AD Connect, suivez les étapes de [Azure AD Connect : Mettre à niveau vers la version la plus récente](./how-to-upgrade-previous-version.md).

### <a name="plan-authentication-agent-number-and-placement"></a>Planifier le numéro et le positionnement de l’agent d’authentification

Pour bénéficier de l’authentification directe, vous devez déployer des agents légers sur le serveur Azure AD Connect et sur vos serveurs locaux qui exécutent Windows Server. Pour réduire la latence, installez les agents aussi près que possible de vos contrôleurs de domaine Active Directory.

Pour la plupart des clients, deux ou trois agents d’authentification suffisent à offrir la haute disponibilité et la capacité nécessaire. Un locataire peut avoir un maximum de 12 agents inscrits. Le premier agent est toujours installé sur le serveur Azure AD Connect lui-même. Pour en savoir plus sur les limitations de l’agent et sur les options de déploiement des agents, consultez [Authentification directe Azure AD : Limitations actuelles](./how-to-connect-pta-current-limitations.md).

### <a name="plan-the-migration-method"></a>Planifier la méthode de migration

Vous pouvez choisir entre deux méthodes pour migrer de la gestion fédérée des identités vers l’authentification directe et l’authentification unique fluide. La méthode utilisée dépend de la configuration initiale de votre instance AD FS.

* **Azure AD Connect**. Si vous avez configuré à l’origine AD FS avec Azure AD Connect, vous *devez* passer à l’authentification directe en utilisant l’Assistant Azure AD Connect.

   Azure AD Connect exécute automatiquement l’applet de commande **Set-MsolDomainAuthentication** quand vous changez la méthode de connexion des utilisateurs. Azure AD Connect annule automatiquement la fédération de tous les domaines fédérés vérifiés dans votre locataire Azure AD.

   > [!NOTE]
   > Actuellement, si vous avez utilisé à l’origine Azure AD Connect pour configurer AD FS, vous ne pouvez pas éviter l’annulation de la fédération de tous les domaines de votre locataire quand vous passez à l’authentification directe pour la connexion des utilisateurs.
‎
* **Azure AD Connect avec PowerShell**. Vous pouvez utiliser cette méthode seulement si vous n’avez pas configuré à l’origine AD FS avec Azure AD Connect. Pour cette option, vous devez toujours changer la méthode d’authentification des utilisateurs via l’Assistant Azure AD Connect. La principale différence avec cette option est que l’Assistant n’exécute pas automatiquement l’applet de commande **Set-MsolDomainAuthentication**. Avec cette option, vous avez un contrôle total sur les domaines qui sont convertis et dans quel ordre.

Pour savoir quelle méthode utiliser, effectuez les étapes des sections suivantes.

#### <a name="verify-current-user-sign-in-settings"></a>Vérifier les paramètres de connexion utilisateur définis actuellement

1. Connectez-vous au [portail Azure](https://aad.portal.azure.com/) en utilisant un compte d’administrateur général.
2. Dans la section **Connexion utilisateur**, vérifiez les paramètres suivants :
   * **Fédération** est défini sur **Activé**.
   * **Authentification unique fluide** est défini sur **Désactivé**.
   * **Authentification directe** est défini sur **Désactivé**.

   ![Capture d’écran des paramètres de la section Connexion utilisateur d’Azure AD Connect](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image1.png)

#### <a name="verify-how-federation-was-configured"></a>Vérifier la configuration de la fédération

1. Sur votre serveur Azure AD Connect, ouvrez Azure AD Connect. Sélectionnez **Configurer**.
2. Dans la page **Tâches supplémentaires**, sélectionnez **Afficher la configuration actuelle**, puis sélectionnez **Suivant**.<br />
 
   ![Capture d’écran de l’option Afficher la configuration actuelle dans la page Tâches supplémentaires](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image2.png)<br />
3. Sous **Tâches supplémentaires > Gérer la fédération**, accédez à **Services Active Directory Federation Services (ADFS)** .<br />

   * Si la configuration d’AD FS apparaît dans cette section, vous pouvez considérer qu’AD FS a été initialement configuré à l’aide d’Azure AD Connect. Vous pouvez convertir vos domaines de l’identité fédérée en identité managée en utilisant l’option **Modifier la connexion utilisateur** d’Azure AD Connect. Pour plus d’informations sur le processus, consultez la section **Option A : Configurer l’authentification directe à l’aide d’Azure AD Connect**.
   * Si AD FS n’est pas listé dans les paramètres actuels, vous devez convertir manuellement vos domaines de l’identité fédérée à l’identité managée avec PowerShell. Pour plus d’informations sur ce processus, consultez la section **Option B : Passer de la fédération à l’authentification directe avec Azure AD Connect et PowerShell**.

### <a name="document-current-federation-settings"></a>Documenter les paramètres de fédération actuels

Pour trouver le paramètre de fédération actuel, exécutez l’applet de commande **Get-MsolDomainFederationSettings** :

``` PowerShell
Get-MsolDomainFederationSettings -DomainName YourDomain.extention | fl *
```

Exemple :

``` PowerShell
Get-MsolDomainFederationSettings -DomainName Contoso.com | fl *
```

Vérifiez les paramètres qui peuvent avoir été personnalisés pour la conception de votre fédération et la documentation du déploiement. En particulier, recherchez les personnalisations dans **PreferredAuthenticationProtocol**, **SupportsMfa** et **PromptLoginBehavior**.

Pour plus d’informations, voir les articles suivants :

* [AD FS prompt=login parameter support](/windows-server/identity/ad-fs/operations/ad-fs-prompt-login)
* [Set-MsolDomainAuthentication](/powershell/module/msonline/set-msoldomainauthentication)

> [!NOTE]
> Si **SupportsMfa** est défini sur **True**, cela signifie que vous utilisez une solution d’authentification multifacteur locale pour injecter un deuxième facteur dans le flux d’authentification des utilisateurs. Cette configuration ne fonctionne plus pour les scénarios d’authentification d’Azure AD. 
>
> Utilisez plutôt le service cloud Azure AD Multi-Factor Authentication pour effectuer la même fonction. Évaluez soigneusement vos besoins d’authentification multifacteur avant de continuer. Avant de convertir vos domaines, veillez à bien comprendre comment utiliser Azure AD Multi-Factor Authentication, les implications en matière de gestion des licences et le processus d’inscription des utilisateurs.

#### <a name="back-up-federation-settings"></a>Sauvegarder les paramètres de fédération

Même si aucune modification n’est apportée aux autres parties de confiance de votre batterie AD FS pendant le processus, nous vous recommandons de disposer d’une sauvegarde de votre batterie de serveurs AD FS valide et actuelle, à partir de laquelle vous pouvez faire une restauration. Vous pouvez créer une sauvegarde valide actuelle avec l’[outil de restauration rapide d’AD FS](/windows-server/identity/ad-fs/operations/ad-fs-rapid-restore-tool) gratuit de Microsoft. Vous pouvez utiliser l’outil pour sauvegarder AD FS, et pour restaurer une batterie de serveurs existante ou pour en créer une.

Si vous choisissez de ne pas utiliser l’outil de restauration rapide d’AD FS, exportez au moins les approbations de partie de confiance de la plateforme d’identités Microsoft 365 et toutes les règles de revendication personnalisées associées que vous avez ajoutées. Vous pouvez exporter l’approbation de partie de confiance et les règles de revendication associées en utilisant l’exemple PowerShell suivant :

``` PowerShell
(Get-AdfsRelyingPartyTrust -Name "Microsoft Office 365 Identity Platform") | Export-CliXML "C:\temp\O365-RelyingPartyTrust.xml"
```

## <a name="deployment-considerations-and-using-ad-fs"></a>Considérations relatives au déploiement et utilisation d’AD FS

Cette section décrit les considérations relatives au déploiement et des détails sur l’utilisation d’AD FS.

### <a name="current-ad-fs-use"></a>Utilisation actuelle d’AD FS

Avant de passer de l’identité fédérée à l’identité managée, examinez attentivement votre utilisation actuelle d’AD FS pour Azure AD, Microsoft 365 et autres applications (approbations de partie de confiance). En particulier, considérez les scénarios décrits dans le tableau suivant :

| Si | Alors |
|-|-|
| Vous prévoyez d’utiliser AD FS avec d’autres applications (autres qu’Azure AD et Microsoft 365). | Après avoir converti vos domaines, vous utiliserez AD FS et Azure AD. Considérez l’expérience utilisateur. Dans certains scénarios, les utilisateurs doivent peut-être s’authentifier deux fois : une fois auprès d’Azure AD (où un utilisateur bénéficie d’un accès avec authentification unique à d’autres applications, comme Microsoft 365), et une deuxième fois dans les applications encore liées à AD FS comme approbation de partie de confiance. |
| Votre instance d’AD FS est très personnalisée et s’appuie sur des paramètres de personnalisation spécifiques du fichier onload.js (par exemple si vous avez modifié l’expérience de connexion afin que les utilisateurs utilisent seulement un format **SamAccountName** pour leur nom d’utilisateur au lieu d’un nom d’utilisateur principal (UPN), ou si votre organisation a fortement personnalisé l’expérience de connexion). Le fichier onload.js ne peut pas être dupliqué dans Azure AD. | Avant de continuer, vous devez vérifier qu’Azure AD peut répondre à vos spécifications de personnalisation actuelles. Pour plus d’informations et des conseils, consultez les sections relatives à la personnalisation d’AD FS.|
| Vous utilisez AD FS pour bloquer des versions antérieures des clients d’authentification.| Vous pouvez remplacer les contrôles d’AD FS qui bloquent les versions antérieures des clients d’authentification en utilisant une combinaison de [contrôles d’accès conditionnel](../conditional-access/concept-conditional-access-conditions.md) et de [règles d’accès de client Exchange Online](/exchange/clients-and-mobile-in-exchange-online/client-access-rules/client-access-rules). |
| Vous obligez les utilisateurs à effectuer une authentification multifacteur via une solution de serveur d’authentification multifacteur locale quand les utilisateurs s’authentifient auprès d’AD FS.| Dans un domaine d’identité managée, vous ne pouvez pas injecter une demande d’authentification multifacteur via la solution d’authentification multifacteur locale dans le flux d’authentification. Vous pouvez cependant utiliser le service Azure AD Multi-Factor Authentication pour l’authentification multifacteur une fois que le domaine est converti.<br /><br /> Si vos utilisateurs n’utilisent pas actuellement Azure AD Multi-Factor Authentication, une étape ponctuelle d’inscription des utilisateurs est nécessaire. Vous devez préparer et communiquer l’inscription planifiée à vos utilisateurs. |
| Vous utilisez actuellement des stratégies de contrôle d’accès (règles AuthZ) dans AD FS pour contrôler l’accès à Microsoft 365.| Vous pouvez envisager de remplacer les stratégies par des [stratégies d’accès conditionnel](../conditional-access/overview.md) et des [règles d’accès du client Exchange Online](/exchange/clients-and-mobile-in-exchange-online/client-access-rules/client-access-rules) d’Azure AD équivalentes.|

### <a name="common-ad-fs-customizations"></a>Personnalisations courantes d’AD FS

Cette section décrit les personnalisations courantes d’AD FS.

#### <a name="insidecorporatenetwork-claim"></a>Revendication InsideCorporateNetwork

AD FS émet la revendication **InsideCorporateNetwork** si l’utilisateur qui s’authentifie se trouve dans le réseau d’entreprise. Cette revendication est ensuite passée à Azure AD. La revendication est utilisée pour contourner l’authentification multifacteur en fonction de l’emplacement réseau de l’utilisateur. Pour savoir comment déterminer si cette fonctionnalité est actuellement disponible dans AD FS, consultez [Adresses IP de confiance pour les utilisateurs fédérés](../authentication/howto-mfa-adfs.md).

La revendication **InsideCorporateNetwork** n’est plus disponible une fois que vos domaines sont convertis à l’authentification directe. Vous pouvez utiliser des [emplacements nommés dans Azure AD](../reports-monitoring/quickstart-configure-named-locations.md) pour remplacer cette fonctionnalité.

Une fois que vous avez configuré des emplacements nommés, vous devez mettre à jour toutes les stratégies d’accès conditionnel configurées de façon à inclure ou à exclure les valeurs **Tous les emplacements approuvés** ou **Adresses IP approuvées MFA** pour refléter les emplacements nommés nouvellement créés.

Pour plus d’informations sur la condition [Emplacement](../conditional-access/location-condition.md) dans l’accès conditionnel, consultez **Emplacements d’accès conditionnel Azure Active Directory**.

#### <a name="hybrid-azure-ad-joined-devices"></a>Appareils joints à Azure AD Hybride

Quand vous joignez un appareil à Azure AD, vous pouvez créer des règles d’accès conditionnel qui imposent que les appareils répondent à vos standards d’accès pour la sécurité et la conformité. De même, les utilisateurs peuvent se connecter à un appareil en utilisant un compte professionnel ou scolaire d’une organisation au lieu d’un compte personnel. Quand vous utilisez des appareils joints à Azure AD Hybride, vous pouvez joindre à Azure AD vos appareils joints à un domaine Active Directory. Votre environnement fédéré peut avoir été configuré pour utiliser cette fonctionnalité.

Pour garantir le bon fonctionnement de la jonction hybride pour les appareils joints au domaine une fois vos domaines convertis à l’authentification directe, pour les clients Windows 10, vous devez utiliser Azure AD Connect pour synchroniser les comptes d’ordinateur Active Directory avec Azure AD.

Pour les comptes d’ordinateur Windows 8 et Windows 7, la jointure hybride utilise l’authentification unique fluide pour inscrire l’ordinateur dans Azure AD. Vous n’avez pas à synchroniser les comptes d’ordinateur Windows 8 et Windows 7 comme vous le faites pour les appareils Windows 10. Cependant, vous devez déployer un fichier workplacejoin.exe mis à jour (via un package .msi) sur les clients Windows 8 et Windows 7 afin qu’ils puissent s’inscrire avec l’authentification unique fluide. [Téléchargez le package .msi](https://www.microsoft.com/download/details.aspx?id=53554).

Pour plus d’informations, consultez [Configurer les appareils joints à Azure AD Hybride](../devices/hybrid-azuread-join-plan.md).

#### <a name="branding"></a>Personnalisation

Si votre organisation [a personnalisé vos pages de connexion AD FS](/windows-server/identity/ad-fs/operations/ad-fs-user-sign-in-customization) pour afficher des informations plus pertinentes pour l’organisation, envisagez de faire des [personnalisations similaires pour la page de connexion Azure AD](../fundamentals/customize-branding.md).

S’il est possible d’apporter des personnalisations similaires, il faut s’attendre à certaines modifications visuelles après la conversion. Vous pouvez fournir des informations sur les modifications attendues dans vos communications aux utilisateurs.

> [!NOTE]
> La personnalisation d’organisation est disponible seulement si vous avez acheté une licence Premium ou De base pour Azure Active Directory, ou si vous disposez d’une licence Microsoft 365.

## <a name="plan-for-smart-lockout"></a>Planifier le verrouillage intelligent

Le verrouillage intelligent d’Azure AD protège contre les attaques des mots de passe par force brute. Le verrouillage intelligent empêche le verrouillage d’un compte Active Directory local quand l’authentification directe est utilisée et qu’une stratégie de groupe de verrouillage de compte est définie dans Active Directory.

Pour plus d’informations, consultez [Verrouillage intelligent Azure Active Directory](../authentication/howto-password-smart-lockout.md).

## <a name="plan-deployment-and-support"></a>Planification du déploiement et du support

Effectuez les tâches décrites dans cette section pour planifier le déploiement et le support.

### <a name="plan-the-maintenance-window"></a>Planifier la fenêtre de maintenance

Même si le processus de conversion de domaine est relativement rapide, Azure AD peut continuer à envoyer des demandes d’authentification à vos serveurs AD FS pendant jusqu’à 4 heures après la fin de la conversion. Pendant cet intervalle de quatre heures, et en fonction des différents caches du côté service, Azure AD peut ne pas accepter ces authentifications. Les utilisateurs peuvent alors recevoir une erreur. L’utilisateur peut néanmoins toujours s’authentifier auprès d’AD FS, mais Azure AD n’accepte plus le jeton émis de l’utilisateur, car cette approbation de la fédération est maintenant supprimée.

Seuls les utilisateurs qui accèdent aux services via un navigateur web pendant cette période postérieure à la conversion avant que le cache côté service soit effacé sont affectés. Les clients hérités (Exchange ActiveSync, Outlook 2010/2013) ne sont normalement pas affectés, car Exchange Online conserve un cache de leurs informations d’identification pour un laps de temps défini. Le cache est utilisé pour réauthentifier l’utilisateur en mode silencieux. L’utilisateur ne doit pas retourner à AD FS. Les informations d’identification stockées sur l’appareil pour ces clients sont utilisées pour se réauthentifier eux-mêmes en mode silencieux une fois cette mise en cache effacée. Les utilisateurs ne doivent normalement pas recevoir d’invites demandant un mot de passe à la suite du processus de conversion de domaine.

Les clients d’authentification moderne (Office 2016 et Office 2013, applications iOS et Android) utilisent un jeton d’actualisation valide pour obtenir de nouveaux jetons d’accès permettant de continuer à accéder aux ressources au lieu de retourner à AD FS. Ces clients sont protégés des demandes de mot de passe résultant du processus de conversion de domaine. Les clients continuent de fonctionner sans configuration supplémentaire.

> [!IMPORTANT]
> N’arrêtez pas votre environnement AD FS ni ne supprimez l’approbation de partie de confiance de Microsoft 365 avant d’avoir vérifié que tous les utilisateurs réussissent à s’authentifier avec l’authentification cloud.

### <a name="plan-for-rollback"></a>Plan de restauration

Si vous rencontrez un problème majeur que vous ne pouvez pas résoudre rapidement, vous pouvez décider de revenir à la solution de fédération. Il est important de planifier ce qu’il faut si votre déploiement ne se passe pas comme prévu. Si la conversion du domaine ou des utilisateurs échoue pendant le déploiement, ou si vous devez revenir à la fédération, vous devez comprendre comment limiter les dégâts d’une panne et réduire l’effet sur vos utilisateurs.

#### <a name="to-roll-back"></a>Pour annuler

Pour planifier l’annulation, consultez la documentation de conception et de déploiement de la fédération pour connaître les détails spécifiques à votre déploiement. Le processus doit inclure ces tâches :

* Conversion des domaines managés en domaines fédérés avec l’applet de commande **Convert-MSOLDomainToFederated**.
* Si nécessaire, configuration de règles de revendication supplémentaires.

### <a name="plan-communications"></a>Planifier les communications

Une partie importante de la planification du déploiement et du support est de veiller à ce que vos utilisateurs finaux soient informés de façon proactive des modifications à venir. Les utilisateurs doivent savoir à l’avance ce qu’ils peuvent subir et ce qui est attendu d’eux.

Une fois que l’authentification directe et l’authentification unique fluide sont déployées, l’expérience de connexion des utilisateurs pour l’accès à Microsoft 365 et aux autres ressources authentifiées via Azure AD change. Les utilisateurs qui sont en dehors du réseau voient seulement la page de connexion Azure AD. Ces utilisateurs ne sont pas redirigés vers la page basée sur des formulaires qui est présentée par les serveurs proxy d’applications web externes.

Incluez les éléments suivants dans votre stratégie de communication :

* Avertissez les utilisateurs des fonctionnalités à venir et publiées en utilisant :
   * L’e-mail et d’autres canaux de communication internes.
   * Des visuels, comme des affiches.
   * Des communications de la part de la direction, directes ou autres.
* Déterminez qui va personnaliser les communications, et qui va les envoyer et quand.

## <a name="implement-your-solution"></a>Implémenter votre solution

Vous avez planifié votre solution. Vous pouvez maintenant l’implémenter. L’implémentation implique les composants suivants :

* Préparation pour l’authentification unique fluide.
* Changement de la méthode de connexion pour l’authentification directe et activation de l’authentification unique fluide.

### <a name="step-1-prepare-for-seamless-sso"></a>Étape 1 : Préparer pour l’authentification unique fluide

Pour que vos appareils utilisent l’authentification unique fluide, vous devez ajouter une URL Azure AD aux paramètres de zone intranet des utilisateurs via une stratégie de groupe dans Active Directory.

Par défaut, les navigateurs web déterminent automatiquement la zone appropriée, Internet ou intranet, à partir d’une URL. Par exemple, **http:\/\/contoso/** est mappée à la zone intranet, tandis que **http:\/\/intranet.contoso.com** est mappée à la zone Internet (car l’URL contient un point). Les navigateurs envoient des tickets Kerberos à un point de terminaison cloud, comme l’URL Azure AD, seulement si vous ajoutez explicitement l’URL à la zone intranet du navigateur.

Effectuez les étapes pour [déployer](./how-to-connect-sso-quick-start.md) les modifications nécessaires sur vos appareils.

> [!IMPORTANT]
> Ce changement ne modifie pas la façon dont vos utilisateurs se connectent à Azure AD. Cependant, il est important d’appliquer cette configuration à tous vos appareils avant de continuer. Les utilisateurs qui se connectent sur des appareils qui n’ont pas reçu cette configuration doivent simplement entrer un nom d’utilisateur et un mot de passe pour se connecter à Azure AD.

### <a name="step-2-change-the-sign-in-method-to-pass-through-authentication-and-enable-seamless-sso"></a>Étape 2 : Changer la méthode de connexion pour l’authentification directe et activer l’authentification unique fluide

Vous avez deux options pour changer la méthode de connexion pour l’authentification directe et activer l’authentification unique fluide.

#### <a name="option-a-configure-pass-through-authentication-by-using-azure-ad-connect"></a>Option A : Configurer l’authentification directe à l’aide d’Azure AD Connect

Utilisez cette méthode si vous avez initialement configuré votre environnement AD FS avec Azure AD Connect. Vous ne pouvez pas utiliser cette méthode si vous *n’avez pas* initialement configuré votre environnement AD FS avec Azure AD Connect.

> [!IMPORTANT]
> Une fois les étapes suivantes terminées, tous vos domaines sont convertis de l’identité fédérée à l’identité managée. Pour plus d’informations, consultez [Planifier la méthode de migration](#plan-the-migration-method).

Changez d’abord la méthode de connexion :

1. Sur le serveur Azure AD Connect, ouvrez l’Assistant Azure AD Connect.
2. Sélectionnez **Modifier la connexion utilisateur**, puis sélectionnez **Suivant**. 
3. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur général.
4. Dans la page **Connexion utilisateur**, sélectionnez le bouton **Authentification directe**, sélectionnez **Activer l’authentification unique**, puis sélectionnez **Suivant**.
5. Dans la page **Activer l’authentification unique**, entrez les informations d’identification d’un compte d’administrateur de domaine, puis sélectionnez **Suivant**.

   > [!NOTE]
   > Les informations d’identification du compte d’administrateur de domaine sont nécessaires pour activer l’authentification unique fluide. Le processus effectue les actions suivantes, qui nécessitent ces autorisations élevées. Les informations d’identification du compte d’administrateur de domaine ne sont pas stockées dans Azure AD Connect ni dans Azure AD. Les informations d’identification du compte d’administrateur de domaine sont utilisées seulement pour activer la fonctionnalité. Les informations d’identification sont supprimées quand le processus se termine avec succès.
   >
   > 1. Un compte d’ordinateur nommé AZUREADSSOACC (qui représente Azure AD) est créé dans votre instance Active Directory locale.
   > 2. La clé de déchiffrement Kerberos du compte d’ordinateur est partagée de façon sécurisée avec Azure AD.
   > 3. Deux noms de principal du service Kerberos sont créés pour représenter les deux URL utilisées lors de la connexion à Azure AD.

6. Dans la page **Prêt à configurer**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée. Ensuite, sélectionnez **Configurer**.<br />

   ![Capture d’écran de la page Prêt à configurer](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image8.png)<br />
7. Dans le Portail Azure AD, sélectionnez **Azure Active Directory**, puis **Azure AD Connect**.
8. Vérifiez ces paramètres :
   * **Fédération** est défini sur **Désactivé**.
   * **Authentification unique fluide** est défini sur **Activé**.
   * **Authentification directe** est défini sur **Activé**.<br />

   ![Capture d’écran montrant les paramètres de la section Connexion utilisateur](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image9.png)<br />

Ensuite, déployez des méthodes d’authentification supplémentaires :

1. Dans le Portail Azure, accédez à **Azure Active Directory** > **Azure AD Connect**, puis sélectionnez **Authentification directe**.
2. Dans la page **Authentification directe**, sélectionnez le bouton **Télécharger**.
3. Dans la page **Télécharger l’agent**, sélectionnez **Accepter les conditions et télécharger**.

   Le téléchargement des agents d’authentification supplémentaires démarre. Installez l’agent d’authentification secondaire sur un serveur joint à un domaine. 

   > [!NOTE]
   > Le premier agent est toujours installé sur le serveur Azure AD Connect lui-même, car il fait partie des modifications de la configuration apportées dans la section **Connexion utilisateur** de l’outil Azure AD Connect. Installez les agents d’authentification supplémentaires sur un serveur distinct. Nous vous recommandons d’avoir deux ou trois agents d’authentification supplémentaires disponibles. 

4. Exécutez l’installation de l’agent d’authentification. Lors de l’installation, vous devez fournir les informations d’identification d’un compte d’administrateur général.

   ![Capture d’écran montrant le bouton Installer que vous utilisez pour exécuter le package de l’agent d’authentification Microsoft Azure AD Connect.](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image11.png)

   ![Capture d’écran montrant la page de connexion Microsoft.](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image12.png)

5. Quand l’agent d’authentification est installé, vous pouvez revenir à la page d’intégrité de l’agent d’authentification directe pour vérifier l’état des agents supplémentaires.

Accédez à [Tests et étapes suivantes](#testing-and-next-steps).

> [!IMPORTANT]
> Ignorez la section **Option B : Passer de la fédération à l’authentification directe avec Azure AD Connect et PowerShell**. Les étapes décrites dans cette section ne s’appliquent pas si vous avez choisi l’option A pour changer la méthode de connexion en authentification directe et pour activer l’authentification unique fluide. 

#### <a name="option-b-switch-from-federation-to-pass-through-authentication-by-using-azure-ad-connect-and-powershell"></a>Option B : Passer de la fédération à l’authentification directe avec Azure AD Connect et PowerShell

Utilisez cette option si vous n’avez pas initialement configuré vos domaines fédérés avec Azure AD Connect.

Activez d’abord l’authentification directe :

1. Sur le serveur Azure AD Connect, ouvrez l’Assistant Azure AD Connect.
2. Sélectionnez **Modifier la connexion utilisateur**, puis sélectionnez **Suivant**.
3. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur général.
4. Dans la page **Connexion utilisateur**, sélectionnez le bouton **Authentification directe**. Sélectionnez **Activer l’authentification unique**, puis sélectionnez **Suivant**.
5. Dans la page **Activer l’authentification unique**, entrez les informations d’identification d’un compte d’administrateur de domaine, puis sélectionnez **Suivant**.

   > [!NOTE]
   > Les informations d’identification du compte d’administrateur de domaine sont nécessaires pour activer l’authentification unique fluide. Le processus effectue les actions suivantes, qui nécessitent ces autorisations élevées. Les informations d’identification du compte d’administrateur de domaine ne sont pas stockées dans Azure AD Connect ni dans Azure AD. Les informations d’identification du compte d’administrateur de domaine sont utilisées seulement pour activer la fonctionnalité. Les informations d’identification sont supprimées quand le processus se termine avec succès.
   > 
   > 1. Un compte d’ordinateur nommé AZUREADSSOACC (qui représente Azure AD) est créé dans votre instance Active Directory locale.
   > 2. La clé de déchiffrement Kerberos du compte d’ordinateur est partagée de façon sécurisée avec Azure AD.
   > 3. Deux noms de principal du service Kerberos sont créés pour représenter les deux URL utilisées lors de la connexion à Azure AD.

6. Dans la page **Prêt à configurer**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée. Ensuite, sélectionnez **Configurer**.<br />

   ![Capture d’écran montrant la page Prêt à configurer et le bouton Configurer](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image18.png)<br />
   Les étapes suivantes se produisent quand vous sélectionnez **Configurer** :

   1. Le premier agent d’authentification directe est installé.
   2. La fonctionnalité d’authentification directe est activée.
   3. L’authentification unique fluide est activée.

7. Vérifiez ces paramètres :
   * **Fédération** est défini sur **Activé**.
   * **Authentification unique fluide** est défini sur **Activé**.
   * **Authentification directe** est défini sur **Activé**.
   
   ![Capture d’écran montrant les paramètres à vérifier dans la section Connexion utilisateur.](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image19.png)
8. Sélectionnez **Authentification directe** et vérifiez que l’état est **Actif**.<br />
   
   Si l’agent d’authentification n’est pas actif, suivez des [étapes de dépannage](./tshoot-connect-pass-through-authentication.md) avant de passer au processus de conversion du domaine de l’étape suivante. Si vous convertissez vos domaines avant de vérifier que vos agents d’authentification directe ont bien été installés et qu’ils sont à l’état **Actif** dans le portail Azure, vous risquez de provoquer une interruption de l’authentification.

Ensuite, déployez les agents d’authentification supplémentaires :

1. Dans le Portail Azure, accédez à **Azure Active Directory** > **Azure AD Connect**, puis sélectionnez **Authentification directe**.
2. Dans la page **Authentification directe**, sélectionnez le bouton **Télécharger**. 
3. Dans la page **Télécharger l’agent**, sélectionnez **Accepter les conditions et télécharger**.
 
   Le téléchargement de l’agent d’authentification démarre. Installez l’agent d’authentification secondaire sur un serveur joint à un domaine.

   > [!NOTE]
   > Le premier agent est toujours installé sur le serveur Azure AD Connect lui-même, car il fait partie des modifications de la configuration apportées dans la section **Connexion utilisateur** de l’outil Azure AD Connect. Installez les agents d’authentification supplémentaires sur un serveur distinct. Nous vous recommandons d’avoir deux ou trois agents d’authentification supplémentaires disponibles.
 
4. Exécutez l’installation de l’agent d’authentification. Lors de l’installation, vous devez entrer les informations d’identification d’un compte d’administrateur général.<br />

   ![Capture d’écran montrant le bouton Installer dans la page Package de l’agent d’authentification Microsoft Azure AD Connect](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image23.png)<br />
   ![Capture d’écran montrant la page de connexion](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image24.png)<br />
5. Quand l’agent d’authentification est installé, vous pouvez revenir à la page d’intégrité de l’agent d’authentification directe pour vérifier l’état des agents supplémentaires.

À ce stade, l’authentification fédérée est toujours active et opérationnelle pour vos domaines. Pour poursuivre le déploiement, vous devez convertir chaque domaine pour le faire passer de l’identité fédérée à l’identité managée, afin que l’authentification directe commence à traiter les demandes d’authentification du domaine.

Vous n’êtes pas obligé de convertir tous les domaines en même temps. Vous pouvez choisir de démarrer avec un domaine de test sur votre locataire de production ou de démarrer avec votre domaine qui a le plus petit nombre d’utilisateurs.

Effectuez la conversion en utilisant le module PowerShell Azure AD :

1. Dans PowerShell, connectez-vous au portail Azure en utilisant un compte d’administrateur général.
2. Pour convertir le premier domaine, exécutez la commande suivante :
 
   ``` PowerShell
   Set-MsolDomainAuthentication -Authentication Managed -DomainName <domain name>
   ```
 
3. Dans le portail Azure AD, sélectionnez **Azure Active Directory** > **Azure AD Connect**.
4. Une fois que vous avez converti tous vos domaines fédérés, vérifiez ces paramètres :
   * **Fédération** est défini sur **Désactivé**.
   * **Authentification unique fluide** est défini sur **Activé**.
   * **Authentification directe** est défini sur **Activé**.<br />

   ![Capture d’écran montrant les paramètres de la section Connexion utilisateur du portail Azure AD.](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image26.png)<br />

## <a name="testing-and-next-steps"></a>Tests et étapes suivantes

Effectuez les tâches suivantes pour vérifier l’authentification directe et terminer le processus de conversion.

### <a name="test-pass-through-authentication"></a>Tester l’authentification directe 

Quand votre locataire utilisait l’identité fédérée, les utilisateurs étaient redirigés de la page de connexion d’Azure AD vers votre environnement AD FS. Maintenant que le locataire est configuré pour utiliser l’authentification directe au lieu de l’authentification fédérée, les utilisateurs ne sont pas redirigés vers AD FS. Au lieu de cela, ils se connectent directement sur la page de connexion d’Azure AD.

Pour tester l’authentification directe :

1. Ouvrez Internet Explorer en mode InPrivate, afin que l’authentification unique fluide ne vous connecte pas automatiquement.
2. Accédez à la page de connexion d’Office 365 ([https://portal.office.com](https://portal.office.com/)).
3. Entrez un UPN d’utilisateur, puis sélectionnez **Suivant**. Veillez à entrer l’UPN d’un utilisateur hybride qui a été synchronisé à partir de votre instance d’Active Directory locale et qui a déjà utilisé l’authentification fédérée. Une page sur laquelle vous entrez le nom d’utilisateur et le mot de passe apparaît :

   ![Capture d’écran montrant la page de connexion où vous entrez un nom d’utilisateur](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image27.png)

   ![Capture d’écran montrant la page de connexion où vous entrez un mot de passe](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image28.png)

4. Après avoir entré le mot de passe et sélectionné **Se connecter**, vous êtes redirigé vers le portail Office 365.

   ![Capture d’écran montrant le portail Office 365](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image29.png)

### <a name="test-seamless-sso"></a>Tester l’authentification unique fluide

Pour tester l’authentification unique fluide :

1. Connectez-vous à un ordinateur joint au domaine, connecté au réseau d’entreprise.
2. Dans Internet Explorer ou dans Chrome, accédez à une des URL suivantes (remplacez « contoso » par votre domaine) :

   * https:\/\/myapps.microsoft.com/contoso.com
   * https:\/\/myapps.microsoft.com/contoso.onmicrosoft.com

   L’utilisateur est redirigé brièvement à la page de connexion d’Azure AD, qui affiche le message « Tentative de connexion en cours ». L’utilisateur n’est pas invité à entrer un nom d’utilisateur ou un mot de passe.<br />

   ![Capture d’écran montrant la page de connexion et le message d’Azure AD](media/plan-migrate-adfs-pass-through-authentication/migrating-adfs-to-pta_image30.png)<br />
3. L’utilisateur est redirigé et connecté avec succès au panneau d’accès :

   > [!NOTE]
   > L’authentification unique fluide fonctionne sur les services Microsoft 365 qui prennent en charge l’indication du domaine (par exemple, myapps.microsoft.com/contoso.com). Actuellement, le portail Microsoft 365 (portal.office.com) ne prend en charge les indications de domaine. Les utilisateurs doivent entrer un UPN. Une fois qu’un UPN est entré, l’authentification unique fluide récupère le ticket Kerberos pour le compte de l’utilisateur. L’utilisateur est connecté sans devoir entrer un mot de passe.

   > [!TIP]
   > Envisagez de déployer la [jonction Azure AD Hybride sur Windows 10](../devices/overview.md) pour une expérience d’authentification unique améliorée.

### <a name="remove-the-relying-party-trust"></a>Supprimer l’approbation de partie de confiance

Après avoir vérifié le bon fonctionnement de l’authentification de tous les utilisateurs et clients via Azure AD, vous pouvez supprimer sans danger l’approbation de partie de confiance de Microsoft 365.

Si vous n’utilisez pas AD FS à d’autres fins (c’est-à-dire pour d’autres approbations de partie de confiance), vous pouvez à ce stade mettre AD FS hors service sans problème.

### <a name="rollback"></a>Restauration

Si vous découvrez un problème majeur et que vous ne pouvez pas le résoudre rapidement, vous pouvez choisir de revenir à la solution de fédération.

Consultez la documentation de conception et de déploiement de la fédération pour connaître les détails spécifiques à votre déploiement. Le processus doit impliquer ces tâches :

* Convertir les domaines managés à l’authentification fédérée avec l’applet de commande **Convert-MSOLDomainToFederated**.
* Si nécessaire, configurez des règles de revendication supplémentaires.

### <a name="sync-userprincipalname-updates"></a>Synchroniser les mises à jour de userPrincipalName

Historiquement, les mises à jour de l’attribut **UserPrincipalName**, qui utilise le service de synchronisation à partir de l’environnement local, sont bloquées sauf si les deux conditions suivantes sont remplies :

* L’utilisateur est dans un domaine d’identité (non fédérée) managée.
* Aucune licence n’a été affectée à l’utilisateur.

Pour savoir comment vérifier ou activer cette fonctionnalité, consultez [Synchroniser les mises à jour de userPrincipalName](./how-to-connect-syncservice-features.md).

## <a name="roll-over-the-seamless-sso-kerberos-decryption-key"></a>Substituer la clé de déchiffrement Kerberos pour l’authentification unique fluide

Il est important de changer fréquemment la clé de déchiffrement Kerberos du compte d’ordinateur AZUREADSSOACC (qui représente Azure AD). Le compte d’ordinateur AZUREADSSOACC est créé dans votre forêt Active Directory locale. Nous vous recommandons fortement de changer la clé de déchiffrement Kerberos au moins tous les 30 jours, pour vous aligner sur la façon dont les membres du domaine Active Directory soumettent des changements de mot de passe. Comme aucun appareil n’est associé à l’objet de compte d’ordinateur AZUREADSSOACC, vous devez effectuer ce changement de clé manuellement.

Lancez le remplacement de la clé de déchiffrement Kerberos de l’authentification unique fluide sur le serveur local qui exécute Azure AD Connect.

Pour plus d’informations, consultez [Comment puis-je substituer la clé de déchiffrement Kerberos du compte d’ordinateur AZUREADSSOACC ?](./how-to-connect-sso-faq.md).

## <a name="monitoring-and-logging"></a>Surveillance et journalisation

Surveillez les serveurs qui exécutent les agents d’authentification pour conserver la disponibilité de la solution. En plus des compteurs de performance généraux du serveur, les agents d’authentification exposent des objets de performance qui peuvent être utilisés pour comprendre les erreurs et les statistiques associées à l’authentification.

Les agents d’authentification consignent les opérations dans les journaux des événements Windows sous Application and Service Logs\Microsoft\AzureAdConnect\AuthenticationAgent\Admin.

Vous pouvez également activer la journalisation pour le dépannage.

Pour plus d’informations, consultez [Résoudre les problèmes de l’authentification directe Azure Active Directory](./tshoot-connect-pass-through-authentication.md).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus d’informations sur les [principes de conception d’Azure AD Connect](plan-connect-design-concepts.md).
* Choisir l’[authentification appropriée](./choose-ad-authn.md).
* Découvrez plus d’informations sur les [topologies prises en charge](plan-connect-design-concepts.md).

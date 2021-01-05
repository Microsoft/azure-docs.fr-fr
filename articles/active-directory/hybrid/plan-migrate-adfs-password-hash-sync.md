---
title: 'Azure AD Connect : Passer de la fédération à la synchronisation de hachage de mot de passe (PHS) pour Azure AD | Microsoft Docs'
description: Cet article contient des informations sur le déplacement d’un environnement d’identité hybride de la fédération à la synchronisation de hachage de mot de passe.
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
ms.openlocfilehash: dca888bf9e3dc75e80764949a11d95efe3514635
ms.sourcegitcommit: 21c3363797fb4d008fbd54f25ea0d6b24f88af9c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/08/2020
ms.locfileid: "96861814"
---
# <a name="migrate-from-federation-to-password-hash-synchronization-for-azure-active-directory"></a>Migrer de la fédération à la synchronisation de hachage de mot de passe pour Azure Active Directory

Cet article décrit comment faire passer les domaines de votre organisation d’AD FS (Active Directory Federation Services) à la synchronisation de hachage de mot de passe.

> [!NOTE]
> Modifier votre méthode d’authentification implique une planification, des tests et de possibles temps d’arrêt. Le [lancement intermédiaire](how-to-connect-staged-rollout.md) fournit un autre moyen de tester et d’effectuer une migration progressive de la fédération vers l’authentification cloud à l’aide de la synchronisation de hachage du mot de passe.
>
> Si vous prévoyez d’utiliser le lancement intermédiaire, n’oubliez pas de désactiver les fonctionnalités de lancement intermédiaire une fois que vous avez terminé le basculement.  Pour davantage d’informations, consultez [Migrer vers l’authentification cloud à l’aide du lancement intermédiaire](how-to-connect-staged-rollout.md).


## <a name="prerequisites-for-migrating-to-password-hash-synchronization"></a>Prérequis pour la migration vers la synchronisation de hachage de mot de passe

Les prérequis suivants sont nécessaires pour migrer de l’utilisation d’AD FS vers l’utilisation de la synchronisation de hachage de mot de passe.


### <a name="update-azure-ad-connect"></a>Mettre à jour Azure AD Connect

Pour réaliser correctement les étapes de migration vers la synchronisation de hachage du mot de passe, vous devez disposer au minimum d’[Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594) version 1.1.819.0. Cette version contient des modifications significatives au niveau de la conversion de la connexion et réduit le temps global du passage de l’authentification fédérée à l’authentification cloud de quelques heures à quelques minutes.


> [!IMPORTANT]
> Vous pouvez lire dans des documentations, des outils et des blogs obsolètes que la conversion de l’utilisateur est nécessaire quand vous convertissez des domaines de l’identité fédérée à l’identité managée. La *conversion des utilisateurs* n’est plus nécessaire. Microsoft s’emploie à mettre à jour la documentation et les outils afin de refléter ce changement.

Pour mettre à jour Azure AD Connect, suivez les étapes de [Azure AD Connect : Mettre à niveau vers la version la plus récente](./how-to-upgrade-previous-version.md).

### <a name="password-hash-synchronization-required-permissions"></a>Autorisations requises pour la synchronisation du hachage de mot de passe

Vous pouvez configurer Azure AD Connect en utilisant des paramètres express ou une installation personnalisée. Si vous avez utilisé l’option d’installation personnalisée, les [autorisations nécessaires](./reference-connect-accounts-permissions.md) pour la synchronisation de hachage de mot de passe risquent de ne pas être en place.

Le compte de service AD DS d’Azure AD Connect a besoin des autorisations suivantes pour pouvoir synchroniser les hachages de mot de passe :

* Répliquer les changements d’annuaires
* Répliquer les changements d’annuaire Tout

C’est le bon moment pour vérifier que ces autorisations sont en place pour tous les domaines de la forêt.

### <a name="plan-the-migration-method"></a>Planifier la méthode de migration

Vous pouvez choisir entre deux méthodes pour migrer de la gestion fédérée des identités vers la synchronisation de hachage de mot de passe et l’authentification unique fluide. La méthode utilisée dépend de la configuration initiale de votre instance AD FS.

* **Azure AD Connect**. Si vous avez configuré à l’origine AD FS avec Azure AD Connect, vous *devez* passer à la synchronisation de hachage de mot de passe en utilisant l’Assistant Azure AD Connect.

   Azure AD Connect exécute automatiquement l’applet de commande **Set-MsolDomainAuthentication** quand vous changez la méthode de connexion des utilisateurs. Azure AD Connect annule automatiquement la fédération de tous les domaines fédérés vérifiés dans votre locataire Azure AD.

   > [!NOTE]
   > Actuellement, si vous avez utilisé à l’origine Azure AD Connect pour configurer AD FS, vous ne pouvez pas éviter l’annulation de la fédération de tous les domaines de votre locataire quand vous passez à la synchronisation de hachage de mot de passe pour la connexion des utilisateurs. ‎
* **Azure AD Connect avec PowerShell**. Vous pouvez utiliser cette méthode seulement si vous n’avez pas configuré à l’origine AD FS avec Azure AD Connect. Pour cette option, vous devez toujours changer la méthode d’authentification des utilisateurs via l’Assistant Azure AD Connect. La principale différence avec cette option est que l’Assistant n’exécute pas automatiquement l’applet de commande **Set-MsolDomainAuthentication**. Avec cette option, vous avez un contrôle total sur les domaines qui sont convertis et dans quel ordre.

Pour savoir quelle méthode utiliser, effectuez les étapes des sections suivantes.

#### <a name="verify-current-user-sign-in-settings"></a>Vérifier les paramètres de connexion utilisateur définis actuellement

Pour vérifier les paramètres actuels de connexion des utilisateurs :

1. Connectez-vous au [portail Azure](https://aad.portal.azure.com/) en utilisant un compte d’administrateur général.
2. Dans la section **Connexion utilisateur**, vérifiez les paramètres suivants :
   * **Fédération** est défini sur **Activé**.
   * **Authentification unique fluide** est défini sur **Désactivé**.
   * **Authentification directe** est défini sur **Désactivé**.

   ![Capture d’écran des paramètres de la section Connexion utilisateur d’Azure AD Connect](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image1.png)

#### <a name="verify-the-azure-ad-connect-configuration"></a>Vérifier la configuration d’Azure AD Connect

1. Sur votre serveur Azure AD Connect, ouvrez Azure AD Connect. Sélectionnez **Configurer**.
2. Dans la page **Tâches supplémentaires**, sélectionnez **Afficher la configuration actuelle**, puis sélectionnez **Suivant**.<br />

   ![Capture d’écran de l’option Afficher la configuration actuelle sélectionnée dans la page Tâches supplémentaires](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image2.png)<br />
3. Dans la page **Vérification de votre solution**, notez l’état de **Synchronisation de hachage de mot de passe**.<br /> 

   * Si **Synchronisation de hachage de mot de passe** est défini sur **Désactivé**, effectuez les étapes de cet article pour l’activer.
   * Si **Synchronisation de hachage de mot de passe** est défini sur **Activé**, vous pouvez ignorer la section **Étape 1 : Activer la synchronisation de hachage de mot de passe** de cet article.
4. Dans la page **Vérification de votre solution** n faites défiler jusqu’à **Services Active Directory Federation Services (ADFS)** .<br />

   * ‎Si la configuration d’AD FS apparaît dans cette section, vous pouvez considérer qu’AD FS a été initialement configuré à l’aide d’Azure AD Connect. Vous pouvez convertir vos domaines de l’identité fédérée en identité managée en utilisant l’option **Modifier la connexion utilisateur** d’Azure AD Connect. Le processus est détaillé dans la section **Option A : Passer de la fédération à la synchronisation de hachage de mot de passe avec Azure AD Connect**.
   * Si AD FS n’est pas listé dans les paramètres actuels, vous devez convertir manuellement vos domaines de l’identité fédérée à l’identité managée avec PowerShell. Pour plus d’informations sur ce processus, consultez la section **Option B : Passer de la fédération à la synchronisation de hachage de mot de passe avec Azure AD Connect et PowerShell**.

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
> Si **SupportsMfa** est défini sur **True**, cela signifie que vous utilisez une solution d’authentification multifacteur locale pour injecter un deuxième facteur dans le flux d’authentification des utilisateurs. Ce programme d’installation ne fonctionne plus pour les scénarios d’authentification Azure AD après conversion de ce domaine de l’authentification fédérée à l’authentification gérée. Après la désactivation de la fédération, vous interrompez la relation avec votre fédération en local, y compris les adaptateurs MFA locaux. 
>
> Au lieu de cela, utilisez le service cloud Azure AD Multi-Factor Authentication pour la même fonction. Évaluez soigneusement vos besoins d’authentification multifacteur avant de continuer. Avant de convertir vos domaines, veillez à bien comprendre comment utiliser Azure AD Multi-Factor Authentication, les implications en matière de gestion des licences et le processus d’inscription des utilisateurs.

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

AD FS émet la revendication **InsideCorporateNetwork** si l’utilisateur qui s’authentifie se trouve dans le réseau d’entreprise. Cette revendication est ensuite passée à Azure AD. La revendication est utilisée pour contourner l’authentification multifacteur en fonction de l’emplacement réseau de l’utilisateur. Pour savoir comment déterminer si cette fonctionnalité est actuellement activée dans AD FS, consultez [Adresses IP de confiance pour les utilisateurs fédérés](../authentication/howto-mfa-adfs.md).

La revendication **InsideCorporateNetwork** n’est plus disponible une fois que vos domaines sont convertis à la synchronisation de hachage de mot de passe. Vous pouvez utiliser des [emplacements nommés dans Azure AD](../reports-monitoring/quickstart-configure-named-locations.md) pour remplacer cette fonctionnalité.

Une fois que vous avez configuré des emplacements nommés, vous devez mettre à jour toutes les stratégies d’accès conditionnel configurées de façon à inclure ou à exclure les valeurs **Tous les emplacements approuvés** ou **Adresses IP approuvées MFA** pour refléter les emplacements nommés nouvellement créés.

Pour plus d’informations sur la condition [Emplacement](../conditional-access/location-condition.md) dans l’accès conditionnel, consultez **Emplacements d’accès conditionnel Azure Active Directory**.

#### <a name="hybrid-azure-ad-joined-devices"></a>Appareils joints à Azure AD Hybride

Quand vous joignez un appareil à Azure AD, vous pouvez créer des règles d’accès conditionnel qui imposent que les appareils répondent à vos standards d’accès pour la sécurité et la conformité. De même, les utilisateurs peuvent se connecter à un appareil en utilisant un compte professionnel ou scolaire d’une organisation au lieu d’un compte personnel. Quand vous utilisez des appareils joints à Azure AD Hybride, vous pouvez joindre à Azure AD vos appareils joints à un domaine Active Directory. Votre environnement fédéré peut avoir été configuré pour utiliser cette fonctionnalité.

Pour garantir le bon fonctionnement de la jonction hybride sur tous les appareils joints au domaine une fois les domaines convertis à la synchronisation du hachage de mot de passe, vous devez utiliser les options d’appareil Azure AD Connect afin de synchroniser les comptes d’ordinateur Active Directory sur Azure AD pour les clients Windows 10. 

Pour les comptes d’ordinateur Windows 8 et Windows 7, la jointure hybride utilise l’authentification unique fluide pour inscrire l’ordinateur dans Azure AD. Vous n’avez pas à synchroniser les comptes d’ordinateur Windows 8 et Windows 7 comme vous le faites pour les appareils Windows 10. Cependant, vous devez déployer un fichier workplacejoin.exe mis à jour (via un package .msi) sur les clients Windows 8 et Windows 7 afin qu’ils puissent s’inscrire avec l’authentification unique fluide. [Téléchargez le package .msi](https://www.microsoft.com/download/details.aspx?id=53554).

Pour plus d’informations, consultez [Configurer les appareils joints à Azure AD Hybride](../devices/hybrid-azuread-join-plan.md).

#### <a name="branding"></a>Personnalisation

Si votre organisation [a personnalisé vos pages de connexion AD FS](/windows-server/identity/ad-fs/operations/ad-fs-user-sign-in-customization) pour afficher des informations plus pertinentes pour l’organisation, envisagez de faire des [personnalisations similaires pour la page de connexion Azure AD](../fundamentals/customize-branding.md).

S’il est possible d’apporter des personnalisations similaires, il faut s’attendre à certaines modifications visuelles après la conversion. Vous pouvez fournir des informations sur les modifications attendues dans vos communications aux utilisateurs.

> [!NOTE]
> La personnalisation d’organisation est disponible seulement si vous avez acheté une licence Premium ou De base pour Azure Active Directory, ou si vous disposez d’une licence Microsoft 365.

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

Une fois que la synchronisation de hachage de mot de passe et l’authentification unique fluide sont déployées, l’expérience de connexion des utilisateurs pour l’accès à Microsoft 365 et aux autres ressources authentifiées via Azure AD change. Les utilisateurs qui sont en dehors du réseau voient seulement la page de connexion Azure AD. Ces utilisateurs ne sont pas redirigés vers la page basée sur des formulaires qui est présentée par les serveurs proxy d’applications web externes.

Incluez les éléments suivants dans votre stratégie de communication :

* Avertissez les utilisateurs des fonctionnalités à venir et publiées en utilisant :
   * L’e-mail et d’autres canaux de communication internes.
   * Des visuels, comme des affiches.
   * Des communications de la part de la direction, directes ou autres.
* Déterminez qui va personnaliser les communications, et qui va les envoyer et quand.

## <a name="implement-your-solution"></a>Implémenter votre solution

Vous avez planifié votre solution. Vous pouvez maintenant l’implémenter. L’implémentation implique les composants suivants :

* Activation de la synchronisation de hachage de mot de passe.
* Préparation pour l’authentification unique fluide.
* Changement de la méthode de connexion pour la synchronisation de hachage de mot de passe et activation de l’authentification unique fluide.

### <a name="step-1-enable-password-hash-synchronization"></a>Étape 1 : Activer la synchronisation de hachage de mot de passe

La première étape de l’implémentation de cette solution consiste à activer la synchronisation de hachage de mot de passe avec l’Assistant Azure AD Connect. La synchronisation de hachage de mot de passe est une fonctionnalité facultative que vous pouvez activer dans les environnements qui utilisent la fédération. Cela n’a aucun effet sur le flux d’authentification. Dans ce cas, Azure AD Connect démarre la synchronisation des hachages de mot de passe sans affecter les utilisateurs qui se connectent en utilisant la fédération.

Pour cette raison, nous vous recommandons d’effectuer cette étape en amont, en préparation de la modification de la méthode de connexion de votre domaine. Ensuite, vous avez suffisamment de temps pour vérifier que la synchronisation de hachage de mot de passe fonctionne correctement.

Pour activer la synchronisation de hachage de mot de passe :

1. Sur le serveur Azure AD Connect, ouvrez l’Assistant Azure AD Connect, puis sélectionnez **Configurer**.
2. Sélectionnez **Personnaliser les options de synchronisation**, puis cliquez sur **Suivant**.
3. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur général.
4. Dans la page **Connexion de vos annuaires**, cliquez sur **Suivant**.
5. Dans la page **Filtrage par domaine ou unité d’organisation**, cliquez sur **Suivant**.
6. Dans la page **Fonctionnalités facultatives**, sélectionnez **Synchronisation de mot de passe**, puis cliquez sur **Suivant**.
 
   ![Capture d’écran de l’option Synchronisation de mot de passe sélectionnée dans la page Fonctionnalités facultatives](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image6.png)<br />
7. Sélectionnez **Suivant** dans les pages restantes. Dans la dernière page, sélectionnez **Configurer**.
8. Azure AD Connect commence la synchronisation des hachages de mot de passe lors de la synchronisation suivante.

Une fois la synchronisation de hachage de mot de passe activée, les hachages de mot de passe de tous les utilisateurs dans le périmètre de synchronisation d’Azure AD Connect sont réeffectués et sont écrits dans Azure AD. En fonction du nombre d’utilisateurs, cette opération peut prendre de quelques minutes à plusieurs heures.

Pour la planification, vous pouvez estimer qu’environ 20 000 utilisateurs sont traités en 1 heure.

Pour vérifier que la synchronisation de hachage de mot de passe fonctionne correctement, utilisez la tâche **Résolution des problèmes** de l’Assistant Azure AD Connect :

1. Ouvrez une nouvelle session Windows PowerShell sur votre serveur Azure AD Connect avec l’option Exécuter en tant qu’administrateur.
2. Exécutez `Set-ExecutionPolicy RemoteSigned` ou `Set-ExecutionPolicy Unrestricted`.
3. Lancez l’Assistant Azure AD Connect.
4. Accédez à la page **Tâches supplémentaires**, sélectionnez **Résoudre les problèmes**, puis sélectionnez **Suivant**.
5. Dans la page **Résolution des problèmes**, cliquez sur **Lancer** pour ouvrir le menu de dépannage dans PowerShell.
6. Dans le menu principal, sélectionnez **Résoudre les problèmes de synchronisation de hachage de mot de passe**.
7. Dans le sous-menu, sélectionnez **La synchronisation du hachage de mot de passe ne fonctionne pas du tout**.

Pour résoudre les problèmes, consultez [Résoudre les problèmes de synchronisation du hachage de mot de passe avec Azure AD Connect Sync](./tshoot-connect-password-hash-synchronization.md).

### <a name="step-2-prepare-for-seamless-sso"></a>Étape 2 : Préparer pour l’authentification unique fluide

Pour que vos appareils utilisent l’authentification unique fluide, vous devez ajouter une URL Azure AD aux paramètres de zone intranet des utilisateurs via une stratégie de groupe dans Active Directory.

Par défaut, les navigateurs web déterminent automatiquement la zone appropriée, Internet ou intranet, à partir d’une URL. Par exemple, **http:\/\/contoso/** est mappée à la zone intranet, tandis que **http:\/\/intranet.contoso.com** est mappée à la zone Internet (car l’URL contient un point). Les navigateurs envoient des tickets Kerberos à un point de terminaison cloud, comme l’URL Azure AD, seulement si vous ajoutez explicitement l’URL à la zone intranet du navigateur.

Effectuez les étapes pour [déployer](./how-to-connect-sso-quick-start.md) les modifications nécessaires sur vos appareils.

> [!IMPORTANT]
> Ce changement ne modifie pas la façon dont vos utilisateurs se connectent à Azure AD. Cependant, il est important d’appliquer cette configuration à tous vos appareils avant de continuer. Les utilisateurs qui se connectent sur des appareils qui n’ont pas reçu cette configuration doivent simplement entrer un nom d’utilisateur et un mot de passe pour se connecter à Azure AD.

### <a name="step-3-change-the-sign-in-method-to-password-hash-synchronization-and-enable-seamless-sso"></a>Étape 3 : Changer la méthode de connexion pour la synchronisation de hachage de mot de passe et activer l’authentification unique fluide

Vous avez deux options pour changer la méthode de connexion pour la synchronisation de hachage de mot de passe et l’activation de l’authentification unique fluide.

#### <a name="option-a-switch-from-federation-to-password-hash-synchronization-by-using-azure-ad-connect"></a>Option A : Passer de la fédération à la synchronisation de hachage de mot de passe avec Azure AD Connect

Utilisez cette méthode si vous avez initialement configuré votre environnement AD FS avec Azure AD Connect. Vous ne pouvez pas utiliser cette méthode si vous *n’avez pas* initialement configuré votre environnement AD FS avec Azure AD Connect.

Changez d’abord la méthode de connexion :

1. Sur le serveur Azure AD Connect, ouvrez l’Assistant Azure AD Connect.
2. Sélectionnez **Modifier la connexion utilisateur**, puis sélectionnez **Suivant**. 

   ![Capture d’écran de l’option Modifier la connexion utilisateur dans la page Tâches supplémentaires](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image7.png)<br />
3. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur général.
4. Dans la page **Connexion utilisateur**, sélectionnez le bouton **Synchronisation de hachage de mot de passe**. Veillez à cocher la case **Ne pas convertir les comptes utilisateur**. L’option est dépréciée. Sélectionnez **Activer l’authentification unique**, puis sélectionnez **Suivant**.

   ![Capture d’écran de la page Activer l’authentification unique](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image8.png)<br />

   > [!NOTE]
   > À compter d’Azure AD Connect version 1.1.880.0, la case **Authentification unique fluide** est cochée par défaut.

   > [!IMPORTANT]
   > Vous pouvez ignorer les avertissements indiquant que la conversion des utilisateurs et la synchronisation de hachage de mot de passe complète sont des étapes obligatoires pour le passage de la fédération à l’authentification cloud. Notez que ces étapes ne sont plus nécessaires. Si vous voyez toujours ces avertissements, vérifiez que vous exécutez la version la plus récente d’Azure AD Connect ainsi que la dernière version de ce guide. Pour plus d’informations, consultez la section [Mettre à jour Azure AD Connect](#update-azure-ad-connect).

5. Dans la page **Activer l’authentification unique**, entrez les informations d’identification du compte d’administrateur de domaine, puis sélectionnez **Suivant**.

   ![Capture d’écran de la page Activer l’authentification unique dans laquelle vous pouvez entrer les informations d’identification du compte Administrateur de domaine.](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image9.png)<br />

   > [!NOTE]
   > Les informations d’identification du compte d’administrateur de domaine sont nécessaires pour activer l’authentification unique fluide. Le processus effectue les actions suivantes, qui nécessitent ces autorisations élevées. Les informations d’identification du compte d’administrateur de domaine ne sont pas stockées dans Azure AD Connect ni dans Azure AD. Les informations d’identification du compte d’administrateur de domaine sont utilisées seulement pour activer la fonctionnalité. Les informations d’identification sont supprimées quand le processus se termine avec succès.
   >
   > 1. Un compte d’ordinateur nommé AZUREADSSOACC (qui représente Azure AD) est créé dans votre instance Active Directory locale.
   > 2. La clé de déchiffrement Kerberos du compte d’ordinateur est partagée de façon sécurisée avec Azure AD.
   > 3. Deux noms de principal du service Kerberos sont créés pour représenter les deux URL utilisées lors de la connexion à Azure AD.

6. Dans la page **Prêt à configurer**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée. Ensuite, sélectionnez **Configurer**.

      ![Capture d’écran de la page Prêt à configurer](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image10.png)<br />

   > [!IMPORTANT]
   > À ce stade, tous vos domaines fédérés passent à l’authentification managée. La synchronisation du hachage de mot de passe est la nouvelle méthode d’authentification.

7. Dans le portail Azure AD, sélectionnez **Azure Active Directory** > **Azure AD Connect**.
8. Vérifiez ces paramètres :
   * **Fédération** est défini sur **Désactivé**.
   * **Authentification unique fluide** est défini sur **Activé**.
   * **Synchronisation de mot de passe** est défini sur **Activé**.<br /> 

   ![Capture d’écran montrant les paramètres de la section Connexion utilisateur du portail Azure AD](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image11.png)<br />

Accédez à [Tests et étapes suivantes](#testing-and-next-steps).

   > [!IMPORTANT]
   > Ignorez la section **Option B : Passer de la fédération à la synchronisation de hachage de mot de passe avec Azure AD Connect et PowerShell**. Les étapes décrites dans cette section ne s’appliquent pas si vous avez choisi l’option A pour changer la méthode de connexion en synchronisation de hachage de mot de passe et pour activer l’authentification unique fluide.

#### <a name="option-b-switch-from-federation-to-password-hash-synchronization-using-azure-ad-connect-and-powershell"></a>Option B : Passer de la fédération à la synchronisation de hachage de mot de passe avec Azure AD Connect et PowerShell

Utilisez cette option si vous n’avez pas initialement configuré vos domaines fédérés avec Azure AD Connect. Au cours de ce processus, vous activez l’authentification unique fluide et vous faites passer vos domaines de fédérés en managés.

1. Sur le serveur Azure AD Connect, ouvrez l’Assistant Azure AD Connect.
2. Sélectionnez **Modifier la connexion utilisateur**, puis sélectionnez **Suivant**.
3. Dans la page **Connexion à Azure AD**, entrez le nom d’utilisateur et le mot de passe pour un compte d’administrateur général.
4. Dans la page **Connexion utilisateur**, sélectionnez le bouton **Synchronisation de hachage de mot de passe**. Sélectionnez **Activer l’authentification unique**, puis sélectionnez **Suivant**.

   Avant l’activation de la synchronisation de hachage de mot de passe : ![Capture d’écran montrant l’option Ne pas configurer dans la page Connexion utilisateur](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image12.png)<br />

   Après l’activation de la synchronisation de hachage de mot de passe : ![Capture d’écran montrant de nouvelles options sur la page Connexion utilisateur](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image13.png)<br />
   
   > [!NOTE]
   > À compter d’Azure AD Connect version 1.1.880.0, la case **Authentification unique fluide** est cochée par défaut.

5. Dans la page **Activer l’authentification unique**, entrez les informations d’identification d’un compte d’administrateur de domaine, puis sélectionnez **Suivant**.

   > [!NOTE]
   > Les informations d’identification du compte d’administrateur de domaine sont nécessaires pour activer l’authentification unique fluide. Le processus effectue les actions suivantes, qui nécessitent ces autorisations élevées. Les informations d’identification du compte d’administrateur de domaine ne sont pas stockées dans Azure AD Connect ni dans Azure AD. Les informations d’identification du compte d’administrateur de domaine sont utilisées seulement pour activer la fonctionnalité. Les informations d’identification sont supprimées quand le processus se termine avec succès.
   >
   > 1. Un compte d’ordinateur nommé AZUREADSSOACC (qui représente Azure AD) est créé dans votre instance Active Directory locale.
   > 2. La clé de déchiffrement Kerberos du compte d’ordinateur est partagée de façon sécurisée avec Azure AD.
   > 3. Deux noms de principal du service Kerberos sont créés pour représenter les deux URL utilisées lors de la connexion à Azure AD.

6. Dans la page **Prêt à configurer**, vérifiez que la case **Démarrez le processus de synchronisation une fois la configuration terminée** est cochée. Ensuite, sélectionnez **Configurer**.

   ![Capture d’écran montrant le bouton Configurer sur la page Prêt à configurer](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image15.png)<br />
   Quand vous sélectionnez le bouton **Configurer**, l’authentification unique fluide est configurée comme indiqué dans l’étape précédente. La configuration de synchronisation de hachage de mot de passe n’est pas modifiée, car elle a été activée précédemment.

   > [!IMPORTANT]
   > Aucune modification n’est apportée à la façon dont les utilisateurs se connectent pour l’instant.

7. Dans le portail Azure AD, vérifiez ces paramètres :
   * **Fédération** est défini sur **Activé**.
   * **Authentification unique fluide** est défini sur **Activé**.
   * **Synchronisation de mot de passe** est défini sur **Activé**.

   ![Capture d’écran montrant les paramètres de la section Connexion utilisateur](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image16.png)

#### <a name="convert-domains-from-federated-to-managed"></a>Convertir les domaines fédérés en domaines managés

À ce stade, la fédération est toujours activée et opérationnelle pour vos domaines. Pour poursuivre le déploiement, chaque domaine fédéré doit être converti en domaine managé pour forcer l’authentification utilisateur via la synchronisation de hachage de mot de passe.

> [!IMPORTANT]
> Vous n’êtes pas obligé de convertir tous les domaines en même temps. Vous pouvez choisir de démarrer avec un domaine de test sur votre locataire de production ou de démarrer avec votre domaine qui a le plus petit nombre d’utilisateurs.

Effectuez la conversion en utilisant le module PowerShell Azure AD :

1. Dans PowerShell, connectez-vous au portail Azure en utilisant un compte d’administrateur général.
2. Pour convertir le premier domaine, exécutez la commande suivante :

   ``` PowerShell
   Set-MsolDomainAuthentication -Authentication Managed -DomainName <domain name>
   ```

3. Dans le portail Azure AD, sélectionnez **Azure Active Directory** > **Azure AD Connect**.
4. Vérifiez que le domaine a été converti en domaine managé en exécutant la commande suivante :

   ``` PowerShell
   Get-MsolDomain -DomainName <domain name>
   ```

## <a name="testing-and-next-steps"></a>Tests et étapes suivantes

Effectuez les tâches suivantes pour vérifier la synchronisation de hachage de mot de passe et terminer le processus de conversion.

### <a name="test-authentication-by-using-password-hash-synchronization"></a>Tester l’authentification avec la synchronisation de hachage de mot de passe 

Quand votre locataire utilisait l’identité fédérée, les utilisateurs étaient redirigés de la page de connexion d’Azure AD vers votre environnement AD FS. Maintenant que le locataire est configuré pour utiliser la synchronisation de hachage de mot de passe au lieu de l’authentification fédérée, les utilisateurs ne sont pas redirigés vers AD FS. Au lieu de cela, ils se connectent directement sur la page de connexion d’Azure AD.

Pour tester la synchronisation de hachage de mot de passe :

1. Ouvrez Internet Explorer en mode InPrivate, afin que l’authentification unique fluide ne vous connecte pas automatiquement.
2. Accédez à la page de connexion d’Office 365 ([https://portal.office.com](https://portal.office.com/)).
3. Entrez un UPN d’utilisateur, puis sélectionnez **Suivant**. Veillez à entrer l’UPN d’un utilisateur hybride qui a été synchronisé à partir de votre instance d’Active Directory locale et qui a déjà utilisé l’authentification fédérée. Une page sur laquelle vous entrez le nom d’utilisateur et le mot de passe apparaît :

   ![Capture d’écran montrant la page de connexion où vous entrez un nom d’utilisateur](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image18.png)

   ![Capture d’écran montrant la page de connexion où vous entrez un mot de passe](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image19.png)

4. Après avoir entré le mot de passe et sélectionné **Se connecter**, vous êtes redirigé vers le portail Office 365.

   ![Capture d’écran montrant le portail Office 365](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image20.png)


### <a name="test-seamless-sso"></a>Tester l’authentification unique fluide

1. Connectez-vous à un ordinateur joint au domaine, connecté au réseau d’entreprise.
2. Dans Internet Explorer ou dans Chrome, accédez à une des URL suivantes (remplacez « contoso » par votre domaine) :

   * https:\/\/myapps.microsoft.com/contoso.com
   * https:\/\/myapps.microsoft.com/contoso.onmicrosoft.com

   L’utilisateur est redirigé brièvement à la page de connexion d’Azure AD, qui affiche le message « Tentative de connexion en cours ». L’utilisateur n’est pas invité à entrer un nom d’utilisateur ou un mot de passe.<br />

   ![Capture d’écran montrant la page de connexion et le message d’Azure AD](media/plan-migrate-adfs-password-hash-sync/migrating-adfs-to-phs_image21.png)<br />
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

### <a name="troubleshooting"></a>Dépannage

Votre équipe de support doit comprendre comment résoudre les problèmes d’authentification qui surviennent pendant ou après le passage de l’authentification fédérée à l’authentification managée. Utilisez la documentation de dépannage suivante pour aider votre équipe de support à se familiariser avec les procédures de dépannage courantes, et les mesures à prendre pour isoler et résoudre le problème.

[Résoudre les problèmes de synchronisation de hachage de mot de passe d’Azure Active Directory](./tshoot-connect-password-hash-synchronization.md)

[Résoudre les problèmes d’authentification unique fluide Azure Active Directory](./tshoot-connect-sso.md)

## <a name="roll-over-the-seamless-sso-kerberos-decryption-key"></a>Substituer la clé de déchiffrement Kerberos pour l’authentification unique fluide

Il est important de changer fréquemment la clé de déchiffrement Kerberos du compte d’ordinateur AZUREADSSOACC (qui représente Azure AD). Le compte d’ordinateur AZUREADSSOACC est créé dans votre forêt Active Directory locale. Nous vous recommandons fortement de changer la clé de déchiffrement Kerberos au moins tous les 30 jours, pour vous aligner sur la façon dont les membres du domaine Active Directory soumettent des changements de mot de passe. Comme aucun appareil n’est associé à l’objet de compte d’ordinateur AZUREADSSOACC, vous devez effectuer ce changement de clé manuellement.

Lancez le remplacement de la clé de déchiffrement Kerberos de l’authentification unique fluide sur le serveur local qui exécute Azure AD Connect.

Pour plus d’informations, consultez [Comment puis-je substituer la clé de déchiffrement Kerberos du compte d’ordinateur AZUREADSSOACC ?](./how-to-connect-sso-faq.md).

## <a name="next-steps"></a>Étapes suivantes

* Découvrez plus d’informations sur les [principes de conception d’Azure AD Connect](plan-connect-design-concepts.md).
* Choisir l’[authentification appropriée](./choose-ad-authn.md).
* Découvrez plus d’informations sur les [topologies prises en charge](plan-connect-design-concepts.md).

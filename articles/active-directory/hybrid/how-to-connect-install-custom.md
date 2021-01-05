---
title: Personnaliser une installation de Azure Active Directory Connect
description: Cet article explique les options de l’installation personnalisée d’Azure AD Connect. Utilisez ces instructions pour installer Active Directory via Azure AD Connect.
services: active-directory
keywords: Qu’est-ce que Azure AD Connect, Installation d’Active Directory, Composants requis pour Azure AD
documentationcenter: ''
author: billmath
manager: daveba
ms.assetid: 6d42fb79-d9cf-48da-8445-f482c4c536af
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 09/10/2020
ms.subservice: hybrid
ms.author: billmath
ms.collection: M365-identity-device-management
ms.openlocfilehash: 3afeadff71bd373354b891bd6690d94d28fc0805
ms.sourcegitcommit: 93329b2fcdb9b4091dbd632ee031801f74beb05b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92096349"
---
# <a name="custom-installation-of-azure-active-directory-connect"></a>Installation personnalisée d’Azure Active Directory Connect
Utilisez les *paramètres personnalisés* dans Azure Active Directory (Azure AD) lorsque vous souhaitez davantage d’options d’installation. Utilisez ces paramètres, par exemple, si vous avez plusieurs forêts ou si vous souhaitez configurer des fonctionnalités facultatives. Utilisez les paramètres personnalisés dans tous les cas où l’option d’[installation rapide](how-to-connect-install-express.md) ne convient pas à vos besoins de déploiement ou de topologie.

Configuration requise :
- [Téléchargez Azure AD Connect](https://go.microsoft.com/fwlink/?LinkId=615771).
- Effectuez les étapes préalables dans [Azure AD Connect : matériel et prérequis](how-to-connect-install-prerequisites.md). 
- Assurez-vous de disposer des comptes comme décrit dans [Autorisations et comptes Azure AD Connect](reference-connect-accounts-permissions.md).

## <a name="custom-installation-settings"></a>Paramètres de l’installation personnalisée 

Pour configurer une installation personnalisée pour Azure AD Connect, consultez les pages de l’Assistant décrites dans les sections suivantes.

### <a name="express-settings"></a>Paramètres Express
Dans la page **Paramètres Express** , sélectionnez **Personnaliser** pour démarrer une installation des paramètres personnalisés.  Le reste de cet article vous guide tout au long du processus d’installation personnalisée. Utilisez les liens suivants pour accéder rapidement aux informations d’une page particulière :

- [Composants requis](#install-required-components)
- [Connexion utilisateur](#user-sign-in)
- [Connexion à Azure AD](#connect-to-azure-ad)
- [Synchronisation](#sync-pages)

### <a name="install-required-components"></a>Installation des composants requis
Lorsque vous installez les services de synchronisation, vous pouvez laisser la section de configuration facultative de côté. Azure AD Connect configure toutes les options automatiquement. Il configure une instance Base de données locale SQL Server 2012 Express, crée les groupes appropriés et attribue des autorisations. Si vous souhaitez modifier les valeurs par défaut, désactivez les cases appropriées.  Le tableau ci-dessous résume ces options et fournit des liens vers des informations supplémentaires. 

![Capture d’écran montrant les sélections facultatives pour les composants d’installation requis dans Azure AD Connect.](./media/how-to-connect-install-custom/requiredcomponents2.png)

| Configuration facultative | Description |
| --- | --- |
|Spécifier un emplacement d’installation personnalisé| Vous permet de modifier le chemin d’installation par défaut d’Azure AD Connect.|
| Utiliser un serveur SQL Server existant |Permet de spécifier le nom du serveur SQL et le nom de l’instance. Choisissez cette option si vous souhaitez utiliser un serveur de base de données existant. Saisissez le nom de l’instance dans la zone **Nom de l’instance** , suivi d’une virgule et du numéro de port, si la navigation n’est pas activée sur votre serveur SQL Server.  Ensuite, spécifiez le nom de la base de données Azure AD Connect.  Vos privilèges SQL déterminent si une base de données peut être créée ou si votre administrateur SQL doit créer la base de données à l’avance.  Si vous disposez d’autorisations d’administrateur de SQL Server, consultez [Installer Azure AD Connect à l’aide d’une base de données existante](how-to-connect-install-existing-database.md).  Si vous avez reçu des autorisations déléguées (DBO), consultez [Installer Azure AD Connect à l’aide d’autorisations administrateur déléguées SQL](how-to-connect-install-sql-delegation.md). |
| Utiliser un compte de service existant |Par défaut, Azure AD Connect fournit un compte de service virtuel pour les services de synchronisation. Si vous utilisez une instance SQL Server distante ou un proxy qui exige une authentification, vous pouvez utiliser un *compte de service administré* ou un compte de service protégé par mot de passe au sein du domaine. Dans ce cas, entrez le compte que vous souhaitez utiliser. Pour exécuter l’installation, vous devez être un administrateur système dans SQL afin de pouvoir créer des informations d’identification de connexion pour le compte de service. Pour en savoir plus, consultez la page [Autorisations et comptes Azure AD Connect](reference-connect-accounts-permissions.md#adsync-service-account). </br></br>À l’aide de la version la plus récente, l’administrateur SQL peut désormais approvisionner la base de données hors bande. L’administrateur Azure AD Connect peut alors l’installer avec les droits de propriétaire de la base de données.  Pour plus d’informations, consultez la page [Installer Azure AD Connect à l’aide d’autorisations administrateur déléguées SQL](how-to-connect-install-sql-delegation.md).|
| Spécifier des groupes de synchronisation personnalisés |Par défaut, lorsque les services de synchronisation sont installés, Azure AD Connect crée quatre groupes locaux sur le serveur. Ces groupes sont les Administrateurs, les Opérateurs, Parcourir et Réinitialisation de mot de passe. Vous pouvez spécifier vos propres groupes ici. Les groupes doivent être locaux sur le serveur. Ils ne peuvent pas se situer dans le domaine. |
|Importer les paramètres de synchronisation (préversion)|Vous permet d’importer des paramètres à partir d’une autre version d’Azure AD Connect.  Pour plus d’informations, consultez [Importer et exporter des paramètres de configuration Azure AD Connect](how-to-connect-import-export-config.md).|

### <a name="user-sign-in"></a>Connexion de l’utilisateur
Après avoir installé les composants requis, sélectionnez la méthode d’authentification unique de vos utilisateurs. Le tableau suivant décrit brièvement les options disponibles. Pour une description complète des méthodes de connexion, consultez [Connexion de l’utilisateur](plan-connect-user-signin.md).

![Capture d’écran montrant la page de connexion de l’utilisateur. L’option de synchronisation de hachage de mot de passe est sélectionnée.](./media/how-to-connect-install-custom/usersignin4.png)

| Option d’authentification unique | Description |
| --- | --- |
| Synchronisation de hachage de mot de passe |Les utilisateurs peuvent se connecter aux services cloud Microsoft, comme Microsoft 365, à l’aide du mot de passe qu’ils utilisent dans leur réseau local. Les mots de passe utilisateur sont synchronisés avec Azure AD sous la forme d’un hachage de mot de passe. L’authentification a lieu dans le Cloud. Pour plus d’informations, consultez [Synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md). |
|Authentification directe|Les utilisateurs peuvent se connecter aux services cloud Microsoft, comme Microsoft 365, à l’aide du mot de passe qu’ils utilisent dans leur réseau local.  Les mots de passe de l’utilisateur sont validés en les transmettant vers le contrôleur de domaine Active Directory local.
| Fédération avec AD FS |Les utilisateurs peuvent se connecter aux services cloud Microsoft, comme Microsoft 365, à l’aide du mot de passe qu’ils utilisent dans leur réseau local.  Les utilisateurs sont redirigés vers leur instance ADFS locale pour l’authentification. L’authentification a lieu localement. |
| Fédération avec PingFederate|Les utilisateurs peuvent se connecter aux services cloud Microsoft, comme Microsoft 365, à l’aide du mot de passe qu’ils utilisent dans leur réseau local.  Ils sont redirigés vers leur instance PingFederate locale pour la connexion. L’authentification a lieu localement. |
| Ne pas configurer |Aucune fonctionnalité de connexion utilisateur n’est installée ou configurée. Choisissez cette option si vous disposez déjà d’un serveur de fédération tiers ou d’une autre solution. |
|Activer l’authentification unique|Cette option est disponible avec la synchronisation de hachage de mot de passe et l’authentification directe. Elle offre une expérience d’authentification unique pour les utilisateurs de bureau sur des réseaux d’entreprise. Pour plus d’informations, consultez [Authentification unique](how-to-connect-sso.md). </br></br>**Remarque :** Cette option n’est pas disponible pour les clients AD FS. AD FS offre déjà le même niveau d’authentification unique.</br>

### <a name="connect-to-azure-ad"></a>Se connecter à Azure AD
Sur la page **Connexion à Azure AD** , entrez un compte et un mot de passe d’administrateur général. Si vous avez sélectionné l’option **Fédération avec AD FS** sur la page précédente, ne vous connectez pas avec un compte dans un domaine que vous envisagez d’activer pour la fédération. 

Vous souhaiterez peut-être utiliser un compte du domaine *onmicrosoft.com* par défaut, qui est fourni avec votre locataire Azure AD. Ce compte est utilisé uniquement pour créer un compte de service dans Azure AD. Il n’est pas utilisé une fois l’installation terminée.
  
![Capture d’écran montrant la page « Se connecter à Azure AD ».](./media/how-to-connect-install-custom/connectaad.png)

Si l’authentification multifacteur est activée sur votre compte d’administrateur général, vous fournissez à nouveau le mot de passe dans la fenêtre de connexion, et vous devez effectuer le test d’authentification multifacteur. Ce test peut consister en un code de vérification ou à passer un appel téléphonique.  

![Capture d’écran montrant la page « Se connecter à Azure AD ». Un champ d’authentification multifacteur invite l’utilisateur à entrer un code.](./media/how-to-connect-install-custom/connectaadmfa.png)

[Privileged Identity Management](../privileged-identity-management/pim-getting-started.md) peut aussi être activé sur le compte d’administrateur global.

Si vous recevez une erreur ou que vous avez des problèmes de connectivité, consultez [Résoudre les problèmes de connectivité liés à Azure AD Connect](tshoot-connect-connectivity.md).

## <a name="sync-pages"></a>Synchroniser les pages

Les sections suivantes décrivent les pages de la section **Sync** .

### <a name="connect-your-directories"></a>Connexion de vos annuaires
Pour vous connecter aux services de domaine Active Directory (Azure AD DS), Azure AD Connect a besoin du nom de la forêt et des informations d’identification d’un compte doté d’autorisations suffisantes.

![Screenshot that shows the "Connect your directories" page.](./media/how-to-connect-install-custom/connectdir01.png)

Une fois que vous avez entré le nom de la forêt et sélectionné **Ajouter un répertoire** , une fenêtre s’affiche. Le tableau ci-dessous décrit vos options disponibles.

| Option | Description |
| --- | --- |
| Créer un compte | Créez le compte Azure AD DS dont Azure AD Connect a besoin pour se connecter à la forêt Active Directory lors de la synchronisation d’annuaires. Une fois cette option sélectionnée, entrez le nom d’utilisateur et le mot de passe d’un compte d’administrateur d’entreprise.  Le compte d’administrateur d’entreprise fourni est utilisé par Azure AD Connect pour créer le compte Azure AD DS requis. Vous pouvez entrer la partie domaine au format NetBIOS ou au format FQDN. Autrement dit, entrez *FABRIKAM\administrator* ou *fabrikam.com\administrator* . |
| Utiliser un compte existant | Fournissez le compte Azure AD DS existant qu’Azure AD Connect peut utiliser pour se connecter à la forêt Active Directory lors de la synchronisation d’annuaires. Vous pouvez entrer la partie domaine au format NetBIOS ou au format FQDN. Autrement dit, entrez *FABRIKAM\syncuser* ou *fabrikam.com\syncuser* . Ce compte peut être un compte d’utilisateur normal, car seules des autorisations de lecture par défaut sont nécessaires. Toutefois, selon votre scénario, vous pouvez avoir besoin d’autorisations supplémentaires. Pour en savoir plus, consultez la page [Autorisations et comptes Azure AD Connect](reference-connect-accounts-permissions.md#create-the-ad-ds-connector-account). |

![Capture d’écran montrant la page « Connecter un annuaire » et la fenêtre de compte de forêt AD, dans laquelle vous pouvez choisir de créer un nouveau compte ou d’utiliser un compte existant.](./media/how-to-connect-install-custom/connectdir02.png)

>[!NOTE]
> À partir de la build 1.4.18.0, vous ne pouvez plus utiliser un compte administrateur d’entreprise ou un compte administrateur de domaine comme compte de connecteur Azure AD DS. Lorsque vous sélectionnez **Utiliser un compte existant** , si vous essayez d’entrer un compte d’administrateur d’entreprise ou un compte d’administrateur de domaine, l’erreur suivante s’affiche : « Using an Enterprise or Domain administrator account for your AD forest account is not allowed. Let Azure AD Connect create the account for you or specify a synchronization account with the correct permissions. »
>

### <a name="azure-ad-sign-in-configuration"></a>Configuration de connexion AD Azure
Sur la page **Configuration de connexion Azure AD** , examinez les domaines UPN (User Principal Name) dans l’Azure AD DS local. Ces domaines UPN ont été vérifiés dans Azure AD. Cette page vous permet également de configurer l’attribut à utiliser pour userPrincipalName.

![Capture d’écran montrant les domaines non vérifiés sur la page « Configuration de la connexion Azure AD ».](./media/how-to-connect-install-custom/aadsigninconfig2.png)  

Passez en revue chaque domaine marqué comme **Non ajouté** ou **Non vérifié** . Assurez-vous que les domaines que vous utilisez ont été vérifiés dans Azure AD. Une fois vos domaines vérifiés, sélectionnez l’icône d’actualisation circulaire. Pour plus d’informations, consultez [Ajouter et vérifier le domaine](../fundamentals/add-custom-domain.md).

L'attribut *userPrincipalName* est utilisé par les utilisateurs lorsqu'ils se connectent à Azure AD et Microsoft 365. Azure AD doit vérifier les domaines (également nommés « Suffixe UPN ») avant la synchronisation des utilisateurs. Microsoft vous recommande de conserver la valeur d’attribut userPrincipalName par défaut. 

Si l’attribut userPrincipalName n’est pas routable et ne peut pas être vérifié, vous pouvez sélectionner un autre attribut. Par exemple, vous pouvez choisir une adresse de messagerie électronique comme attribut contenant l’ID de connexion. Lorsque vous utilisez un attribut autre que userPrincipalName, il s’agit d’un *ID secondaire* . 

La valeur de l’attribut ID secondaire doit suivre la norme RFC 822. Vous pouvez utiliser un ID secondaire avec la synchronisation de hachage de mot de passe, l’authentification directe et la fédération. Dans Active Directory, l’attribut ne peut pas être défini en tant que valeurs multiples, même s’il ne possède qu’une seule valeur. Pour plus d’informations sur l’ID secondaire, consultez [Authentification directe : questions fréquentes](./how-to-connect-pta-faq.md#does-pass-through-authentication-support-alternate-id-as-the-username-instead-of-userprincipalname).

>[!NOTE]
> Lorsque vous activez l’authentification directe, vous devez disposer d’au moins un domaine vérifié pour pouvoir continuer l’assistant d’installation personnalisée.

> [!WARNING]
> Les ID secondaires ne sont pas compatibles avec certaines charges de travail Microsoft 365. Pour plus d’informations, consultez [Configuration d’ID secondaires de connexion](/windows-server/identity/ad-fs/operations/configuring-alternate-login-id).
>

### <a name="domain-and-ou-filtering"></a>Filtrage domaine et unité organisationnelle
Par défaut, tous les domaines et unités d’organisation (UO) sont synchronisés. Si vous ne souhaitez pas synchroniser certains domaines ou UO pour Azure AD, vous pouvez effacer les sélections appropriées.  

![Capture d’écran montrant la page de filtrage de domaine et d’UO.](./media/how-to-connect-install-custom/domainoufiltering.png)  

Cette page configure le filtrage basé sur le domaine et sur les unités d'organisation. Si vous envisagez d’apporter des modifications, consultez les pages [Filtrage par domaine](how-to-connect-sync-configure-filtering.md#domain-based-filtering) et [Filtrage par unité d’organisation](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering). Certaines unités d’organisation étant essentielles aux fonctionnalités, vous devez les conserver sélectionnées.

Si vous utilisez le filtrage par unité d’organisation avec une version d’Azure AD Connect antérieure à la version 1.1.524.0, les nouvelles unités d’organisation sont synchronisées par défaut. Si vous ne souhaitez pas que les nouvelles unités d’organisation soient synchronisées, vous pouvez ajuster le comportement par défaut après l’étape [Filtrage par unité d’organisation](how-to-connect-sync-configure-filtering.md#organizational-unitbased-filtering). Pour Azure AD Connect 1.1.524.0 ou supérieure, vous pouvez indiquer si les nouvelles unités d’organisation doivent être synchronisées.

Si vous prévoyez d’utiliser le [filtrage basé sur le groupe](#sync-filtering-based-on-groups), assurez-vous que l’unité d’organisation avec le groupe est incluse, et non filtrée à l’aide du filtrage par unité d’organisation. Le filtrage par unité d’organisation est évalué avant le filtrage basé sur le groupe.

Il est également possible que certains domaines ne soient pas accessibles en raison de restrictions de pare-feu. Ces domaines sont désélectionnés par défaut et présentent un avertissement.  

![Capture d’écran montrant les domaines inaccessibles.](./media/how-to-connect-install-custom/unreachable.png)  

S’il s’affiche, vérifiez que ces domaines sont effectivement inaccessibles et que ce message est normal.

### <a name="uniquely-identifying-your-users"></a>Identification unique de vos utilisateurs

Sur la page **Identification des utilisateurs** , choisissez comment identifier les utilisateurs dans vos annuaires locaux et comment les identifier à l’aide de l’attribut sourceAnchor.

#### <a name="select-how-users-should-be-identified-in-your-on-premises-directories"></a>Sélectionnez la façon dont les utilisateurs doivent être identifiés dans vos répertoires locaux
La fonctionnalité *Correspondance entre les forêts* vous permet de définir la méthode de représentation des utilisateurs de vos forêts Azure AD DS dans Azure AD. Il est possible de représenter seulement une fois chaque utilisateur entre toutes les forêts, ou de présenter une combinaison des comptes activés et désactivés. L’utilisateur peut également être représenté en tant que contact dans certaines forêts.

![Capture d’écran montrant la page au sein de laquelle vous pouvez identifier vos utilisateurs de manière unique.](./media/how-to-connect-install-custom/unique2.png)

| Paramètre | Description |
| --- | --- |
| [Les utilisateurs ne sont représentés qu’une seule fois à travers toutes les forêts](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |Tous les utilisateurs sont créés en tant qu’objets individuels dans Azure AD. Les objets ne sont pas associés dans le métaverse. |
| [Attribut de messagerie](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |Cette option associe des utilisateurs et des contacts si l’attribut de messagerie a la même valeur dans des forêts différentes. Utilisez cette option si vos contacts ont été créés avec GALSync. Si cette option est choisie, les objets utilisateur dont l’attribut de messagerie n’est pas rempli ne sont pas synchronisés avec Azure AD. |
| Attributs [ObjectSID et msExchangeMasterAccountSID/ msRTCSIP-OriginatorSID](plan-connect-topologies.md#multiple-forests-single-azure-ad-tenant) |Cette option associe un utilisateur activé dans une forêt de comptes à un utilisateur désactivé dans une forêt de ressources. Dans Exchange, cette configuration est connue en tant que « boîte aux lettres liée ». Vous pouvez utiliser cette option si vous utilisez uniquement Lync et si Exchange n’est pas présent dans la forêt de ressources. |
| Attributs SAMAccountName et MailNickName |Cette option associe des attributs où l’ID de connexion est requis pour rechercher l’utilisateur. |
| Choisir un attribut spécifique |Cette option vous permet de sélectionner votre propre attribut. Si cette option est choisie, les objets utilisateur dont l’attribut (sélectionné) n’est pas rempli ne sont pas synchronisés avec Azure AD. **Limitation**  : Seuls les attributs qui se trouvent déjà dans le métaverse sont disponibles pour cette option. |

#### <a name="select-how-users-should-be-identified-by-using-a-source-anchor"></a>Sélectionnez la façon dont les utilisateurs doivent être identifiés à l’aide d’une ancre source
L’attribut *sourceAnchor* ne varie pas pendant la durée de vie d’un objet utilisateur. Il s’agit de la clé primaire liant l’utilisateur local avec l’utilisateur dans Azure AD.

| Paramètre | Description |
| --- | --- |
| Laisser Azure gérer l'ancre source | Sélectionnez cette option si vous souhaitez qu’Azure AD sélectionne l’attribut pour vous. Si vous sélectionnez cette option, Azure AD Connect applique la logique de sélection d’attribut sourceAnchor décrite dans l’article [Utilisation de msDS-ConsistencyGuid en tant que sourceAnchor](plan-connect-design-concepts.md#using-ms-ds-consistencyguid-as-sourceanchor). Une fois l’installation personnalisée terminée, vous voyez l’attribut qui a été sélectionné en tant qu’attribut sourceAnchor. |
| Choisir un attribut spécifique | Sélectionnez cette option si vous souhaitez spécifier un attribut AD existant comme attribut sourceAnchor. |

Étant donné que l’attribut sourceAnchor ne peut pas être modifié, vous devez choisir un attribut approprié. Pour cela, nous vous recommandons objectGUID. Cet attribut ne change pas, sauf si le compte d’utilisateur est déplacé entre les forêts ou les domaines. Ne choisissez pas les attributs susceptibles de changer si une personne se marie ou si son affectation est modifiée. 

Vous ne pouvez pas utiliser d’attributs incluant un arobase (@), vous ne pouvez donc pas utiliser l’email et userPrincipalName. Par ailleurs, l’attribut respecte la casse ; si vous déplacez un objet entre des forêts, veillez à conserver ses minuscules et majuscules. Les attributs binaires sont codés en base 64, mais les autres types d’attributs restent à l’état non codé. 

Dans les scénarios de fédération et dans certaines interfaces Azure AD, l’attribut sourceAnchor est également appelé *immutableID* . 

Vous trouverez plus d’informations sur l’ancre source dans l’article [Principes de conception](plan-connect-design-concepts.md#sourceanchor).

### <a name="sync-filtering-based-on-groups"></a>Filtrage de synchronisation basé sur les groupes
Le filtrage sur la fonctionnalité de groupes vous permet de synchroniser uniquement un petit sous-ensemble d’objets pour un pilote. Pour pouvoir utiliser cette fonctionnalité, créez un groupe à cette fin dans votre instance locale d’Active Directory. Ensuite, ajoutez les utilisateurs et groupes qui doivent être synchronisés sur Azure AD en tant que membres directs. Vous pouvez ajouter et supprimer ultérieurement des utilisateurs à ce groupe pour tenir à jour la liste des objets présents dans Azure AD. 

Les objets que vous voulez synchroniser doivent tous être des membres directs du groupe. Les utilisateurs, les groupes, les contacts ou les ordinateurs/appareils doivent tous être des membres directs. L’appartenance à un groupe imbriqué n’est pas résolue. Lorsque vous ajoutez un groupe en tant que membre, seul le groupe est ajouté. Ses membres ne le sont pas.

![Capture d’écran montrant la page au sein de laquelle vous pouvez paramétrer le filtrage des utilisateurs et des appareils.](./media/how-to-connect-install-custom/filter2.png)

> [!WARNING]
> Cette fonctionnalité est uniquement destinée à prendre en charge un déploiement pilote. Ne l’utilisez pas dans un environnement de production complet.
>

Dans un déploiement de production complet, il serait difficile de maintenir un seul groupe et tous les objets à synchroniser. À la place de la fonctionnalité de filtrage par groupe, utilisez l’une des méthodes décrites dans [Configurer le filtrage](how-to-connect-sync-configure-filtering.md).

### <a name="optional-features"></a>Fonctionnalités facultatives
Sur la page suivante, vous pouvez sélectionner des fonctionnalités facultatives pour votre scénario.

>[!WARNING]
>Les versions d’Azure AD Connect 1.0.8641.0 et antérieures s’appuient sur Azure Access Control Service pour la réécriture du mot de passe.  Ce service a été supprimé le 7 novembre 2018.  Si vous utilisez l’une de ces versions d’Azure AD Connect et que vous avez activé la réécriture du mot de passe, il est possible que les utilisateurs ne puissent plus modifier ou réinitialiser leurs mots de passe une fois le service supprimé. Ces versions d’Azure AD Connect ne prennent pas en charge la réécriture du mot de passe.
>
>Pour plus d’informations, consultez [Migrer à partir d’Azure Access Control Service](../azuread-dev/active-directory-acs-migration.md).
>
>Si vous souhaitez utiliser la réécriture du mot de passe, téléchargez la [dernière version d’Azure AD Connect](https://www.microsoft.com/download/details.aspx?id=47594).

![Capture d’écran montrant la page « Fonctionnalités facultatives ».](./media/how-to-connect-install-custom/optional2a.png)

> [!WARNING]
> Si Azure AD Sync ou la synchronisation directe (DirSync) est activé, n’activez pas les fonctionnalités de réécriture dans Azure AD Connect.



| Fonctionnalités facultatives | Description |
| --- | --- |
| Déploiement Exchange hybride |La fonctionnalité de déploiement Exchange hybride permet la coexistence de boîtes aux lettres Exchange en local et dans Microsoft 365. Azure AD Connect synchronise un ensemble spécifique [d’attributs](reference-connect-sync-attributes-synchronized.md#exchange-hybrid-writeback) d’Azure AD dans votre annuaire local. |
| Dossiers publics de messagerie Exchange | La fonctionnalité Dossiers publics de messagerie Exchange vous permet de synchroniser les objets Dossier public à extension messagerie de votre instance Active Directory locale avec Azure AD. |
| Application Azure AD et filtrage des attributs |En activant le filtrage des attributs et l’application Azure AD, vous pouvez adapter l’ensemble des attributs synchronisés. Cette option ajoute deux autres pages de configuration dans l’Assistant. Pour en savoir plus, voir [Application Azure AD et filtrage des attributs](#azure-ad-app-and-attribute-filtering). |
| Synchronisation de hachage de mot de passe |Si vous avez sélectionné la fédération comme solution de connexion, vous pouvez activer la synchronisation de hachage de mot de passe. Vous pouvez ensuite l’utiliser comme option de sauvegarde.  </br></br>Si vous avez sélectionné l’authentification directe, vous pouvez activer cette option pour assurer la prise en charge pour les clients hérités et permettre une sauvegarde.</br></br> Pour plus d’informations, consultez [Synchronisation de hachage de mot de passe](how-to-connect-password-hash-synchronization.md).|
| Réécriture du mot de passe |Utilisez cette option pour vous assurer que les modifications de mot de passe provenant d’Azure AD sont réécrites dans votre répertoire local. Pour en savoir plus, voir [Prise en main de la gestion de mot de passe](../authentication/tutorial-enable-sspr.md). |
| Écriture différée de groupe |Si vous utilisez des groupes de Microsoft 365, vous pouvez représenter des groupes dans votre instance locale de Active Directory. Cette option est disponible uniquement si Exchange est présent dans votre instance Active Directory locale. Pour plus d’informations, consultez [Réécriture de groupe Azure AD Connect](how-to-connect-group-writeback.md).|
| Écriture différée des appareils |Permet d’écrire de façon différée des objets d’appareil dans Azure AD, au sein de l’instance Active Directory locale, dans le cadre de scénarios à accès conditionnel. Pour en savoir plus, voir [Azure AD Connect : Activation de l’écriture différée des appareils](how-to-connect-device-writeback.md). |
| Synchronisation des attributs des extensions d’annuaire |Sélectionnez cette option pour synchroniser les attributs spécifiés avec Azure AD. Pour en savoir plus, voir [Extensions d’annuaire](how-to-connect-sync-feature-directory-extensions.md). |

### <a name="azure-ad-app-and-attribute-filtering"></a>Application Azure AD et filtrage des attributs
Si vous souhaitez limiter les attributs à synchroniser dans Azure AD, commencez par sélectionner les services que vous utilisez. Si vous modifiez les sélections dans cette page, vous devez sélectionner explicitement un nouveau service en réexécutant l’assistant d’installation.

![Capture d’écran montrant les fonctionnalités facultatives des applications Azure AD.](./media/how-to-connect-install-custom/azureadapps2.png)

Selon les services sélectionnés à l’étape précédente, cette page affiche tous les attributs synchronisés. Cette liste est une combinaison de tous les types d’objet en cours de synchronisation. Si vous avez besoin que certains attributs ne soient pas synchronisés, vous pouvez effacer leur sélection.

![Capture d’écran montrant les fonctionnalités facultatives des attributs Azure AD.](./media/how-to-connect-install-custom/azureadattributes2.png)

> [!WARNING]
> La suppression d’attributs peut affecter la fonctionnalité. Pour consulter des recommandations et meilleures pratiques, voir [Attributs à synchroniser](reference-connect-sync-attributes-synchronized.md#attributes-to-synchronize).
>

### <a name="directory-extension-attribute-sync"></a>Synchronisation des attributs des extensions d’annuaire
Vous pouvez étendre le schéma dans Azure AD en utilisant des attributs personnalisés ajoutés par votre organisation ou d’autres attributs dans Active Directory. Pour utiliser cette fonctionnalité, sélectionnez **Synchronisation des attributs des extensions d’annuaire** dans la page **Fonctionnalités facultatives** . Sur la page **Extensions d’annuaire** , vous pouvez sélectionner d’autres attributs à synchroniser.

>[!NOTE]
>La zone **Attributs disponibles** respecte la casse.

![Capture d’écran affichant la page « Extensions d'annuaire ».](./media/how-to-connect-install-custom/extension2.png)

Pour en savoir plus, voir [Extensions d’annuaire](how-to-connect-sync-feature-directory-extensions.md).

### <a name="enabling-single-sign-on"></a>Activation de l'authentification unique
Sur la page **Authentification unique** , vous configurez l’authentification unique pour une utilisation avec la synchronisation de mot de passe ou l’authentification directe. Vous effectuez cette étape une fois pour chaque forêt synchronisée avec Azure AD. La configuration implique deux étapes :

1.  création du compte d’ordinateur nécessaire dans votre instance Active Directory locale.
2.  configuration de la zone intranet des ordinateurs clients pour prendre en charge l’authentification unique.

#### <a name="create-the-computer-account-in-active-directory"></a>Créer le compte d’ordinateur dans Active Directory
Pour chaque forêt ajoutée à l’aide d’Azure AD Connect, vous devez fournir les informations d’identification de l’administrateur de domaine afin que le compte d’ordinateur puisse être créé dans chaque forêt. Les informations d'identification sont uniquement utilisées pour créer le compte. Elles ne sont pas stockées ou utilisées pour une autre opération. Ajoutez les informations d’identification sur la page **Activer l’authentification unique** , comme le montre l’image suivante.

![Capture d’écran montant la page « Activer l’authentification unique ». Les informations d'identification de la forêt sont ajoutées.](./media/how-to-connect-install-custom/enablesso.png)

>[!NOTE]
>Vous pouvez ignorer les forêts pour lesquelles vous ne souhaitez pas utiliser l’authentification unique.

#### <a name="configure-the-intranet-zone-for-client-machines"></a>Configurer la zone intranet pour les ordinateurs clients
Pour que le client se connecte automatiquement dans la zone intranet, vérifiez que l’URL fait partie de celle-ci. Cela garantit que l’ordinateur joint au domaine envoie automatiquement un ticket Kerberos à Azure AD lorsqu’il est connecté au réseau d’entreprise.

Sur un ordinateur qui possède les outils de gestion de stratégie de groupe :

1.  Ouvrez les outils de gestion de stratégie de groupe.
2.  Modifiez la stratégie de groupe qui sera appliquée à tous les utilisateurs. Par exemple, la stratégie de domaine par défaut.
3.  Accédez à **Configuration utilisateur** > **Modèles d'administration** > **Composants Windows** > **Internet Explorer** > **Panneau de configuration Internet** > **Page Sécurité** . Puis sélectionnez **Liste des attributions de sites aux zones** .
4.  Activez la stratégie. Puis dans la boîte de dialogue, entrez le nom de valeur de `https://autologon.microsoftazuread-sso.com` et la valeur de `1`. Votre configuration doit ressembler à l’image suivante.
  
    ![Capture d’écran montrant les zones Intranet.](./media/how-to-connect-install-custom/sitezone.png)

6.  Sélectionnez **OK** deux fois.

## <a name="configuring-federation-with-ad-fs"></a>Configuration de la fédération avec AD FS
Vous pouvez configurer AD FS avec Azure AD Connect en quelques clics. Avant de commencer, vous devez :

* Windows Server 2012 R2 ou une version ultérieure pour le serveur de fédération. La gestion à distance doit être activée.
* Windows Server 2012 R2 ou une version ultérieure pour le serveur proxy d’application web. La gestion à distance doit être activée.
* Un certificat TLS/SSL pour le nom de service de fédération que vous prévoyez d’utiliser (par exemple, sts.contoso.com).

>[!NOTE]
>Vous pouvez mettre à jour le certificat TLS/SSL de la batterie de serveurs AD FS à l’aide du logiciel Azure AD Connect et ce, même si vous ne l’utilisez pas pour gérer l’approbation de votre fédération.

### <a name="ad-fs-configuration-prerequisites"></a>Conditions préalables à la configuration AD FS
Pour configurer votre batterie AD FS avec Azure AD Connect, vérifiez que WinRM est activé sur les serveurs distants. Vérifiez que vous avez effectué les autres tâches dans les [conditions préalables de la fédération](how-to-connect-install-prerequisites.md#prerequisites-for-federation-installation-and-configuration). Veillez également à suivre les exigences des ports répertoriées dans le tableau [Azure AD Connect et serveurs de Fédération/WAP](reference-connect-ports.md#table-3---azure-ad-connect-and-ad-fs-federation-serverswap).

### <a name="create-a-new-ad-fs-farm-or-use-an-existing-ad-fs-farm"></a>Création d’une batterie de serveurs AD FS ou utilisation d’une batterie de serveurs AD FS existante
Vous pouvez utiliser une batterie de serveurs AD FS ou en créer une nouvelle. Si vous choisissez de créer une batterie de serveurs, vous devez fournir le certificat TLS/SSL. Si ce dernier est protégé par un mot de passe, vous êtes invité à entrer ce mot de passe.

![Capture d’écran montrant la page « Batterie de serveurs AD FS »](./media/how-to-connect-install-custom/adfs1.png)

Si vous choisissez d’utiliser une batterie de serveurs AD FS existante, vous voyez la page sur laquelle vous pouvez configurer la relation d’approbation entre AD FS et Azure AD.

>[!NOTE]
>Vous pouvez utiliser Azure AD Connect pour gérer une seule batterie de serveurs AD FS. Si vous disposez d’une approbation de fédération où Azure AD est configuré sur la batterie de serveurs AD FS sélectionnée, Azure AD Connect recrée l’approbation à partir de zéro.

### <a name="specify-the-ad-fs-servers"></a>Spécification des serveurs AD FS
Spécifiez les serveurs sur lesquels vous souhaitez installer AD FS. Vous pouvez ajouter un ou plusieurs serveurs selon vos besoins de capacité. Avant la configuration, joignez tous les serveurs AD FS à Active Directory. Cette étape n’est pas requise pour les serveurs proxy d’application Web. 

Microsoft recommande d’installer un seul serveur AD FS pour les déploiements de test et pilote. Ensuite, ajoutez et déployez d’autres serveurs pour répondre à vos besoins de mise à l’échelle en réexécutant Azure AD Connect après la configuration initiale.

> [!NOTE]
> Avant d’effectuer cette configuration, assurez-vous que tous les serveurs sont joints à un domaine Active Directory.
>


![Capture d’écran montrant la page « Serveurs de fédération ».](./media/how-to-connect-install-custom/adfs2.png)

### <a name="specify-the-web-application-proxy-servers"></a>Spécification des serveurs proxy d’application web
Spécifiez vos serveurs proxy d’application web. Le serveur proxy d’application Web est déployé dans votre réseau de périmètre, face à l’extranet. Il prend en charge les demandes d’authentification provenant de l’extranet. Vous pouvez ajouter un ou plusieurs serveurs selon vos besoins de capacité. 

Microsoft recommande d’installer un serveur proxy d’application web unique pour un déploiement de test ou pilote. Ensuite, ajoutez et déployez d’autres serveurs pour répondre à vos besoins de mise à l’échelle en réexécutant Azure AD Connect après la configuration initiale. Nous vous recommandons de prévoir un nombre suffisant de serveurs proxy pour répondre aux demandes d’authentification à partir de l’intranet.

> [!NOTE]
> - Si le compte que vous utilisez n’est pas un compte d’administrateur local sur les serveurs proxy d'application web, vous êtes invité à saisir les informations d’identification de l’administrateur.
> - Vérifiez la connectivité HTTP/HTTPS entre le serveur Azure AD Connect et le serveur proxy d’application web avant de spécifier les serveurs proxy d'application web.
> - Assurez-vous qu’il existe une connectivité HTTP/HTTPS entre le serveur d’applications web et le serveur AD FS pour autoriser les transmissions de demandes d’authentification.
>


![Capture d’écran montrant la page Serveurs proxy d’application Web.](./media/how-to-connect-install-custom/adfs3.png)

Vous êtes invité à saisir des informations d’identification afin que le serveur d’application web puisse établir une connexion sécurisée avec le serveur AD FS. Ces informations d’identification doivent être destinées à un compte d’administrateur local sur le serveur AD FS.

![Capture d’écran montrant la page « Informations d’identification ». Les informations d’identification de l’administrateur sont entrées dans les champs Nom d’utilisateur et Mot de passe.](./media/how-to-connect-install-custom/adfs4.png)

### <a name="specify-the-service-account-for-the-ad-fs-service"></a>Spécification du compte de service pour le service AD FS
Le service AD FS requiert un compte de service de domaine pour authentifier les utilisateurs et rechercher les informations utilisateur dans Active Directory. Il prend en charge 2 types de compte de service :

* **Compte de service géré de groupe**  : Ce type de compte a été introduit dans AD DS par Windows Server 2012. Ce type de compte fournit des services tels que AD FS. Il s’agit d’un compte unique dans lequel vous n’avez pas besoin de mettre à jour le mot de passe régulièrement. Utilisez cette option s’il existe déjà des contrôleurs de domaine Windows Server 2012 dans le domaine auquel appartiennent vos serveurs AD FS.
* **Compte d’utilisateur de domaine**  : Ce type de compte vous oblige à fournir un mot de passe et à le mettre à jour régulièrement lorsqu’il expire. Utilisez cette option uniquement s’il n’y a aucun contrôleur de domaine Windows Server 2012 dans le domaine auquel appartiennent vos serveurs AD FS.

Si vous avez sélectionné **Créer un compte de service géré de groupe** et que cette fonctionnalité n’a jamais été utilisée dans Active Directory, saisissez des informations d’identification d’administrateur d’entreprise. Ces informations serviront à initialiser le magasin de clés et à activer la fonctionnalité dans Active Directory.

> [!NOTE]
> Azure AD Connect vérifie si le service AD FS est déjà inscrit en tant que nom du principal de service (SPN) dans le domaine.  Azure AD DS n’autorise pas l’inscription de SPN en double en même temps.  Si un SPN en double est trouvé, vous ne pouvez pas poursuivre l’opération avant de l’avoir supprimé.

![Capture d’écran montrant la page « Compte de service AD FS ».](./media/how-to-connect-install-custom/adfs5.png)

### <a name="select-the-azure-ad-domain-that-you-want-to-federate"></a>Sélection du domaine Azure AD à fédérer
Utilisez la page **Domaine Azure AD** pour configurer la relation de fédération entre AD FS et Azure AD. Ici, vous configurez AD FS pour fournir des jetons de sécurité à Azure AD. Vous configurez également Azure AD pour approuver les jetons de cette instance de AD FS. 

Cette page vous permet de configurer un seul domaine durant l’installation initiale. Vous pouvez configurer ultérieurement des domaines supplémentaires en réexécutant Azure AD Connect.

![Capture d’écran montrant la page « Domaine Azure AD ».](./media/how-to-connect-install-custom/adfs6.png)

### <a name="verify-the-azure-ad-domain-selected-for-federation"></a>Vérification du domaine Azure AD sélectionné pour la fédération
Lorsque vous sélectionnez le domaine que vous souhaitez fédérer, Azure AD Connect vous fournit les informations que vous pouvez utiliser pour vérifier un domaine non vérifié. Pour plus d’informations, consultez [Ajouter et vérifier le domaine](../fundamentals/add-custom-domain.md).

![Capture d’écran montrant la page « Domaine Azure AD », y compris les informations que vous pouvez utiliser pour vérifier le domaine.](./media/how-to-connect-install-custom/verifyfeddomain.png)

> [!NOTE]
> Azure AD Connect tente de vérifier le domaine pendant l’étape de configuration. Si vous n’ajoutez pas les enregistrements DNS (Domain Name System) nécessaires, la configuration ne peut pas être effectuée.
>


## <a name="configuring-federation-with-pingfederate"></a>Configuration de la fédération avec PingFederate
Vous pouvez configurer PingFederate avec Azure AD Connect en quelques clics. Les prérequis suivants sont obligatoires :
- PingFederate 8.4 ou version ultérieure.  Pour plus d’informations, consultez [Intégration de PingFederate à Azure Active Directory et Microsoft 365](https://docs.pingidentity.com/bundle/O365IG20_sm_integrationGuide/page/O365IG_c_integrationGuide.html).
- Un certificat TLS/SSL pour le nom de service de fédération que vous prévoyez d’utiliser (par exemple, sts.contoso.com).

### <a name="verify-the-domain"></a>Vérifier le domaine
Après avoir choisi de configurer la fédération avec PingFederate, vous êtes invité à vérifier le domaine que vous voulez fédérer.  Sélectionnez le domaine dans le menu déroulant.

![Capture d’écran montrant la page « Domaine Azure AD ». L’exemple de domaine « contoso.com » est sélectionné.](./media/how-to-connect-install-custom/ping1.png)

### <a name="export-the-pingfederate-settings"></a>Exporter les paramètres PingFederate


Configurez PingFederate en tant que serveur de fédération pour chaque domaine Azure fédéré.  Sélectionnez **Paramètres d’exportation** pour partager ces informations avec votre administrateur PingFederate.  L’administrateur du serveur de fédération met à jour la configuration, puis fournit l’URL et le numéro de port du serveur PingFederate pour qu’Azure AD Connect puisse vérifier les paramètres de métadonnées.  

![Capture d’écran montrant la page « Paramètres PingFederate ». Le bouton « Exporter les paramètres » s’affiche dans la partie supérieure de la page.](./media/how-to-connect-install-custom/ping2.png)

Contactez votre administrateur PingFederate pour résoudre les éventuels problèmes de validation.  Voici une image montrant des informations concernant un serveur PingFederate qui n’a pas de relation d’approbation valide avec Azure.

![Capture d’écran montrant les informations du serveur : Le serveur PingFederate a été trouvé, mais la connexion du fournisseur de services pour Azure est manquante ou désactivée.](./media/how-to-connect-install-custom/ping5.png)




### <a name="verify-federation-connectivity"></a>Vérifier la connectivité de fédération
Azure AD Connect tente de valider les points de terminaison d’authentification récupérés à partir des métadonnées PingFederate de l’étape précédente.  Azure AD Connect tente d’abord de résoudre les points de terminaison à l’aide de vos serveurs DNS locaux.  Ensuite, une tentative de résolution des points de terminaison à l’aide d’un fournisseur DNS externe est effectuée.  Contactez votre administrateur PingFederate pour résoudre les éventuels problèmes de validation.  

![Capture d’écran montrant la page « Vérifier la connectivité ».](./media/how-to-connect-install-custom/ping3.png)

### <a name="verify-federation-sign-in"></a>Vérifiez la connexion de fédération
Enfin, vous pouvez vérifier le flux de connexion fédérée tout juste configuré en vous connectant au domaine fédéré. Si votre connexion réussit, la fédération avec PingFederate est correctement configurée.

![Capture d’écran montrant la page « Vérifier la connexion fédérée ». Un message en bas indique une connexion réussie.](./media/how-to-connect-install-custom/ping4.png)

## <a name="configure-and-verify-pages"></a>Pages de configuration et de vérification
La configuration se produit sur la page **Configurer** .

> [!NOTE]
> Avant de poursuivre l’installation, si vous avez configuré la fédération, vérifiez que vous avez configuré la [Résolution de noms pour les serveurs de fédération](how-to-connect-install-prerequisites.md#name-resolution-for-federation-servers).
>



![Capture d’écran montrant la page « Prêt à configurer ».](./media/how-to-connect-install-custom/readytoconfigure2.png)

### <a name="use-staging-mode"></a>Utiliser le mode intermédiaire
Vous pouvez configurer un nouveau serveur de synchronisation en parallèle avec le mode intermédiaire. Si vous souhaitez utiliser ce programme d’installation, un seul serveur de synchronisation peut s’exporter vers un répertoire dans le cloud. Mais si vous voulez procéder à un déplacement à partir d’un autre serveur (par exemple, un serveur exécutant DirSync), vous pouvez activer Azure AD Connect en mode intermédiaire. 

Lorsque vous activez le programme d’installation intermédiaire, le moteur de synchronisation importe et synchronise les données normalement. Mais il n’exporte aucune donnée vers Azure AD ou Active Directory. En mode intermédiaire, les fonctionnalités de synchronisation/de réécriture du mot de passe sont désactivées.

![Capture d’écran montrant l’option « Activer le mode intermédiaire ».](./media/how-to-connect-install-custom/stagingmode.png)

En mode intermédiaire, vous pouvez modifier le moteur de synchronisation selon vos besoins et examiner ce qui doit être exporté. Lorsque la configuration vous convient, réexécutez l’Assistant Installation et désactivez le mode intermédiaire. 

Les données sont désormais exportées vers Azure AD à partir de ce serveur. Veillez à désactiver l’autre serveur en même temps, pour qu’un seul serveur puisse exporter de manière active.

Pour en savoir plus, voir [Mode intermédiaire](how-to-connect-sync-staging-server.md).

### <a name="verify-your-federation-configuration"></a>Vérification de votre configuration de fédération
Lorsque vous cliquez sur le bouton **Vérifier** , Azure AD Connect vérifie la configuration DNS. Les paramètres suivants sont vérifiés :

* **Connectivité intranet**
    * Résoudre le nom de domaine complet de fédération : Azure AD Connect vérifie si le nom de domaine complet de fédération peut être résolu par DNS pour garantir la connectivité. Si Azure AD Connect ne peut pas résoudre le nom de domaine complet, la vérification échoue. Vérifiez qu’un enregistrement DNS est présent pour le nom de domaine complet du service de fédération, afin que la vérification puisse être menée à bien.
    * Enregistrement A DNS : Azure AD Connect vérifie s’il existe un enregistrement A pour votre service de fédération. En l’absence d’enregistrement A, la vérification échoue. Créez un enregistrement A plutôt qu’un enregistrement CNAME pour votre nom de domaine complet de fédération afin de mener à bien la vérification.
* **Connectivité extranet**
    * Résoudre le nom de domaine complet de fédération : Azure AD Connect vérifie si le nom de domaine complet de fédération peut être résolu par DNS pour garantir la connectivité.

      ![Capture d’écran montrant la page « Installation terminée ».](./media/how-to-connect-install-custom/completed.png)

      ![Capture d’écran montrant la page « Installation terminée ». Un message indique que la configuration de l’intranet a été vérifiée.](./media/how-to-connect-install-custom/adfs7.png)

Pour valider l’authentification de bout en bout, vous devez effectuer manuellement un ou plusieurs des tests suivants :

* Une fois la synchronisation terminée, dans Azure AD Connect, utilisez la tâche supplémentaire **Vérifier la connexion fédérée** pour vérifier l’authentification d’un compte d’utilisateur local de votre choix.
* Vérifiez que vous pouvez vous connecter avec un navigateur à partir d’une machine jointe au domaine sur l’intranet. Se connecter à https://myapps.microsoft.com. Utilisez ensuite votre compte connecté pour vérifier la connexion. Le compte d’administrateur Azure AD DS intégré n’est pas synchronisé et ne peut pas être utilisé pour la vérification.
* Vérifiez que vous pouvez vous connecter à partir d’un appareil, depuis l’extranet. Sur un ordinateur personnel ou un appareil mobile, connectez-vous à https://myapps.microsoft.com. Fournissez ensuite vos informations d’identification.
* Valider la connexion à un client complet. Se connecter à https://testconnectivity.microsoft.com. Ensuite, sélectionnez **Office 365** > **Test d’authentification unique Office 365** .

## <a name="troubleshoot"></a>Dépanner
Cette section contient des informations de dépannage que vous pouvez utiliser si vous rencontrez un problème lors de l’installation d’Azure AD Connect.

Lorsque vous personnalisez une installation d’Azure AD Connect, sur la page **Installer les composants requis** , vous pouvez sélectionner **Utiliser un serveur SQL existant** . L’erreur suivante peut s’afficher : « The ADSync database already contains data and cannot be overwritten » (La base de données ADSync contient déjà des données et ne peut pas être écrasée). Please remove the existing database and try again. (La base de données ADSync contient déjà des données et ne peut pas être écrasée. Veuillez supprimer la base de données existante et réessayer.) »

![Screenshot that shows the "Install required components" page. Une erreur apparaît dans le bas de la page.](./media/how-to-connect-install-custom/error1.png)

Vous voyez cette erreur, car une base de données nommée *ADSync* existe déjà sur l’instance SQL du serveur SQL que vous avez spécifiée.

Cette erreur survient généralement après avoir désinstallé Azure AD Connect.  La base de données n’est pas supprimée de l’ordinateur qui exécute le serveur SQL lorsque vous désinstallez Azure AD Connect.

Pour corriger ce problème :

1. Vérifiez la base de données ADSync utilisée par Azure AD Connect avant sa désinstallation. Assurez-vous que la base de données n’est plus utilisée.

2. Sauvegardez la base de données.

3. Supprimer la base de données :
    1. Utilisez **Microsoft SQL Server Management Studio** pour vous connecter à une instance SQL. 
    1. Recherchez la base de données **ADSync** et cliquez dessus avec le bouton droit.
    1. Dans le menu contextuel, sélectionnez **Supprimer** .
    1. Sélectionnez **OK** pour supprimer la base de données.

![Capture d’écran montrant Microsoft SQL Server Management Studio. Une synchronisation AD est sélectionnée.](./media/how-to-connect-install-custom/error2.png)

Après avoir supprimé la base de données ADSync, sélectionnez **Installer** pour tenter à nouveau l’installation.

## <a name="next-steps"></a>Étapes suivantes
Une fois l’installation terminée, déconnectez-vous de Windows. Reconnectez-vous à Windows avant d’utiliser le gestionnaire Synchronization Service Manager ou l’éditeur de règles de synchronisation.

Azure AD Connect étant installé, vous pouvez passer à [Vérification de l’installation et affectation des licences](how-to-connect-post-installation.md).

Pour plus d’informations sur les fonctionnalités que vous avez activées pendant l’installation, consultez [Prévenir les suppressions accidentelles](how-to-connect-sync-feature-prevent-accidental-deletes.md) et [Azure AD Connect Health](how-to-connect-health-sync.md).

Pour plus d’informations sur les autres rubriques courantes, consultez [Azure AD Connect Sync : Scheduler](how-to-connect-sync-feature-scheduler.md) et [Intégrer vos identités locales à Azure AD](whatis-hybrid-identity.md).

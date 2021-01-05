---
title: Configurer des revendications de groupe pour des applications avec Azure Active Directory | Microsoft Docs
description: Informations sur la façon de configurer des revendications de groupe utilisables avec Azure AD.
services: active-directory
documentationcenter: ''
ms.reviewer: paulgarn
manager: daveba
ms.subservice: hybrid
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 02/27/2019
ms.author: billmath
author: billmath
ms.openlocfilehash: bef5942707c1ded22ba82bdb0d945b9fdb23fffa
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96349348"
---
# <a name="configure-group-claims-for-applications-with-azure-active-directory"></a>Configurer des revendications de groupe pour des applications avec Azure Active Directory

Azure Active Directory peut fournir une information d’appartenance de groupe d’utilisateurs dans des jetons utilisables au sein d’applications.  Deux modèles principaux sont pris en charge :

- Groupes identifiés par leur attribut d’identificateur d’objet Azure Active Directory
- Groupes identifiés par les attributs sAMAccountName ou GroupSID pour les groupes et utilisateurs synchronisés Active Directory (AD)

> [!IMPORTANT]
> Il existe plusieurs mises en garde concernant cette fonctionnalité :
>
>- La prise en charge de l’utilisation d’attributs sAMAccountName et d’identificateur de sécurité synchronisés localement est conçue pour permettre le déplacement d’applications existantes à partir d’AD FS et d’autres fournisseurs d’identité. Les groupes gérés dans Azure AD ne contiennent pas les attributs nécessaires pour émettre ces revendications.
>- Dans les grandes entreprises, le nombre de groupes dont un utilisateur est un membre peut dépasser la limite qu’Azure Active Directory ajoute à un jeton. 150 groupes pour un jeton SAML, et 200 pour un jeton JWT. Cela peut entraîner des résultats imprévisibles. Si vos utilisateurs ont un grand nombre d’appartenances à des groupes, nous vous recommandons d’utiliser l’option permettant de limiter les groupes émis dans les revendications aux groupes appropriés pour l’application.  
>- Pour un nouveau développement d’application, dans les cas où l’application peut être configurée pour celui-ci, et où une prise en charge de groupe imbriqué n’est pas obligatoire, il est recommandé que l’autorisation dans l’application soit basée sur les rôles d’application plutôt que sur des groupes.  Cela a pour effet de limiter la quantité d’informations qui doivent figurer dans le jeton, de sécuriser davantage et de séparer l’affectation d’utilisateurs de la configuration de l’application.

## <a name="group-claims-for-applications-migrating-from-ad-fs-and-other-identity-providers"></a>Revendications de groupe pour les applications migrant à partir d’AD FS et d’autres fournisseurs d’identité

De nombreuses applications configurées pour s’authentifier auprès d’AD FS s’appuient sur des informations d’appartenance de groupe sous la forme d’attributs de groupe Windows Active Directory.   Ces attributs sont le groupe sAMAccountName qui peut être qualifié par un nom de domaine ou l’identificateur de sécurité du groupe Windows (GroupSID).  Lorsque l’application est fédérée avec AD FS, AD FS utilise la fonction de TokenGroups pour récupérer les appartenances de groupe pour l’utilisateur.

Une application qui a été déplacée d’AD FS a besoin de revendications au même format. Des revendications de groupe et de rôle peuvent être émises à partir d’Azure Active Directory, qui contiennent le domaine qualifié sAMAccountName ou GroupSID synchronisé à partir d’Active Directory au lieu de l’attribut objectID Azure Active Directory du groupe.

Les formats pris en charge pour les revendications de groupe sont les suivants :

- **ObjectId de groupe Azure Active Directory** (disponible pour tous les groupes)
- **sAMAccountName** (disponible pour les groupes synchronisés à partir d’Active Directory)
- **NetbiosDomain\sAMAccountName** (disponible pour les groupes synchronisés à partir d’Active Directory)
- **DNSDomainName\sAMAccountName** (disponible pour les groupes synchronisés à partir d’Active Directory)
- **Identificateur de sécurité de groupe local** (disponible pour les groupes synchronisés à partir d’Active Directory)

> [!NOTE]
> Les attributs sAMAccountName et de SID de groupe local sont disponibles uniquement sur des objets de groupe synchronisés à partir d’Active Directory.   Ils ne sont pas disponibles sur des groupes créés dans Azure Active Directory ou Office 365.   Les applications configurées dans Azure Active Directory pour obtenir des attributs de groupe local synchronisés obtiennent ceux-ci uniquement pour des groupes synchronisés.

## <a name="options-for-applications-to-consume-group-information"></a>Options pour l’utilisation d’informations de groupe par les applications

Les applications peuvent appeler le point de terminaison de groupes MS Graph pour obtenir des informations de groupe pour l’utilisateur authentifié. Cet appel garantit que tous les groupes dont un utilisateur est membre sont disponibles même s’il existe un grand nombre de groupes impliqués.  L’énumération de groupes est alors indépendante des limitations de taille de jeton.

Toutefois, si une application existante attend de commencer les informations de groupe via des revendications, Azure Active Directory peut être configuré avec un certain nombre de formats de revendications différents.  Considérez les options suivantes :

- Lors de l’utilisation de l’appartenance de groupe à des fins d’autorisation dans l’application, il est préférable d’utiliser l’attribut ObjectID du groupe. L’attribut ObjectID du groupe est immuable et unique dans Azure Active Directory et disponible pour tous les groupes.
- Si vous utilisez l’attribut sAMAccountName du groupe local pour l’autorisation, utilisez les noms de domaines complets afin de réduire la probabilité de situations de conflits de noms. L’attribut sAMAccountName peut être unique au sein d’un domaine Active Directory mais, si plusieurs domaines Active Directory sont synchronisés avec un locataire Azure Active Directory, il est possible que plusieurs groupes aient le même nom.
- Envisagez d’utiliser des [rôles d’application](../../active-directory/develop/howto-add-app-roles-in-azure-ad-apps.md) pour fournir une couche d’indirection entre l’appartenance de groupe et l’application.   L’application prend alors des décisions d’autorisation interne basées sur les revendications de rôle dans le jeton.
- Si l’application est configurée pour obtenir des attributs de groupe qui ont été synchronisés à partir d’Active Directory et qu’un groupe ne contient pas ces attributs, celui-ci n’est pas inclus dans les revendications.
- Les revendications de groupe dans les jetons incluent des groupes imbriqués, sauf lors de l’utilisation de l’option permettant de restreindre les revendications de groupe aux groupes affectés à l’application.  Si un utilisateur est membre du GroupeB et que celui-ci est membre de GroupeA, les revendications de groupe pour l’utilisateur contiennent le GroupeA et le GroupeB. Quand les utilisateurs d’une organisation disposent d’un nombre important d’appartenances de groupe, le nombre de groupes listés dans le jeton peut faire augmenter la taille du jeton.  Azure Active Directory limite le nombre de groupes qu’il émet dans un jeton à 150 pour les assertions SAML, et à 200 pour JWT.  Si un utilisateur est membre d’un plus grand nombre de groupes, les groupes sont omis et un lien vers le point de terminaison Graph est inclus à la place, pour obtenir les informations des groupes.

## <a name="prerequisites-for-using-group-attributes-synchronized-from-active-directory"></a>Conditions préalables à l’utilisation d’attributs de groupe synchronisés à partir d’Active Directory

Les revendications d’appartenance de groupe peuvent être émises dans les jetons pour n’importe quel groupe si vous utilisez le format ObjectId. Pour utiliser des revendications de groupe dans des formats autres que l’attribut ObjectId de groupe, les groupes doivent être synchronisés à partir d’Active Directory à l’aide d’Azure AD Connect.

Il existe deux étapes pour configurer Azure Active Directory afin qu’il émette des noms de groupe pour les groupes Active Directory.

1. **Synchronisation des noms de groupe à partir d’Active Directory** Avant qu’Azure Active Directory puisse émettre les noms de groupes ou un identificateur de sécurité (SID) de groupe local dans des revendications de groupe ou de rôle, les attributs requis doivent être synchronisés à partir d’Active Directory.  Vous devez exécuter Azure AD Connect version 1.2.70 ou ultérieure.   Les version d’Azure AD Connect antérieures à la version 1.2.70 synchronisent les objets de groupe à partir d’Active Directory, mais n’incluent pas les attributs de nom de groupe requis.  Effectuez une mise à niveau vers la version actuelle.

2. **Configuration de l’inscription d’application dans Azure Active Directory afin d’inclure les revendications de groupe dans les jetons** Les revendications de groupe peuvent être configurées dans la section Applications d’entreprise du portail ou à l’aide du manifeste de l’application dans la section Inscriptions d’applications.  Pour configurer les revendications de groupe dans le manifeste de l’application, voir « Configuration de l’inscription d’application Azure Active Directory pour les attributs de groupe » ci-dessous.

## <a name="add-group-claims-to-tokens-for-saml-applications-using-sso-configuration"></a>Ajouter des revendications de groupe aux jetons pour les applications SAML en configurant l’authentification unique

Pour configurer les revendications de groupe pour une application SAML de galerie ou hors galerie, ouvrez **Applications d’entreprise**, cliquez sur l’application dans la liste, sélectionnez **Configuration de l’authentification unique**, puis sélectionnez **Attributs et revendications de l’utilisateur**.

Cliquez sur **Ajouter une revendication de groupe**.  

![Capture d’écran montrant la page « Attributs et revendications de l’utilisateur » avec l’option « Ajouter une revendication de groupe » sélectionnée.](media/how-to-connect-fed-group-claims/group-claims-ui-1.png)

Utilisez les cases d’option pour sélectionner les groupes à inclure dans le jeton

![Capture d’écran montrant la fenêtre « Revendications de groupe » avec l’option « Groupes de sécurité » sélectionnée.](media/how-to-connect-fed-group-claims/group-claims-ui-2.png)

| Sélection | Description |
|----------|-------------|
| **Tous les groupes** | Émet des groupes de sécurité, ainsi que des rôles et des listes de distribution.  |
| **Groupes de sécurité** | Émet les groupes de sécurité dont l’utilisateur est membre dans la revendication de groupe |
| **Rôles d’annuaire** | Si l’utilisateur se voit affecter des rôles d’annuaire, ceux-ci sont émis en tant que revendication « wids » (aucune revendication de groupe n’est émise) |
| **Groupes affectés à l’application** | Émet uniquement les groupes qui sont explicitement affectés à l’application et dont l’utilisateur est membre |

Par exemple, pour émettre tous les groupes de sécurité dont l’utilisateur est membre, sélectionnez Groupes de sécurité

![Capture d’écran montrant la fenêtre « Revendications de groupe » avec l’option « Groupes de sécurité » sélectionnée et le menu déroulant « Attribut source » ouvert.](media/how-to-connect-fed-group-claims/group-claims-ui-3.png)

Pour émettre des groupes à l’aide d’attributs Active Directory synchronisés à partir d’Active Directory au lieu d’ObjectID Azure AD, sélectionnez le format requis dans la liste déroulante. Seuls les groupes synchronisés à partir d’Active Directory sont inclus dans les revendications.

![Capture d’écran montrant le menu déroulant « Attribut source » ouvert.](media/how-to-connect-fed-group-claims/group-claims-ui-4.png)

Pour émettre uniquement les groupes attribués à l’application, sélectionnez **Groupes attribués à l’application**

![Capture d’écran montrant la fenêtre « Revendications de groupe » avec l’option « Groupes affectés à l’application » sélectionnée.](media/how-to-connect-fed-group-claims/group-claims-ui-4-1.png)

Les groupes attribués à l’application sont inclus dans le jeton.  Les autres groupes dont l’utilisateur est membre sont omis.  Avec cette option, les groupes imbriqués ne sont pas inclus et l’utilisateur doit être un membre direct du groupe attribué à l’application.

Pour changer les groupes attribués à l’application, sélectionnez l’application dans la liste **Applications d’entreprise** puis cliquez sur **Utilisateurs et groupes** dans le menu de navigation de gauche de l’application.

Pour plus d’informations sur la gestion de l’attribution de groupe aux applications, consultez le document [Attribuer un utilisateur ou un groupe à des applications](../../active-directory/manage-apps/assign-user-or-group-access-portal.md).

### <a name="advanced-options"></a>Options avancées

La manière sont les revendications de groupe sont émises peut être modifiée par les paramètres disponibles sous Options avancées

Personnaliser le nom de la revendication de groupe :  Si cette option est sélectionnée, un type de revendication différent peut être spécifié pour les revendications de groupe.   Entrez le type de revendication dans le champ Nom, et l’espace de noms facultatif pour la revendication dans le champ Espace de noms.

![Capture d’écran montrant la section « Options avancées » avec l’option « Personnaliser le nom de la revendication de groupe » sélectionnée et les valeurs « Nom » et « Espace de noms » entrées.](media/how-to-connect-fed-group-claims/group-claims-ui-5.png)

Certaines applications requièrent que les informations d’appartenance de groupe apparaissent dans la revendication « rôle ». Vous pouvez éventuellement émettre les groupes de l’utilisateur en tant que rôles en cochant la case « Émettre les groupes en tant que revendications de rôle ».

![Capture d’écran montrant la section « Options avancées » avec les options « Personnaliser le nom de la revendication de groupe » et « Émettre les groupes en tant que revendications de rôle » sélectionnées.](media/how-to-connect-fed-group-claims/group-claims-ui-6.png)

> [!NOTE]
> Si l’option pour émettre des données de groupe en tant que rôles est utilisée, seuls les groupes s’affichent dans la revendication de rôle.  Les rôles d’application auxquels l’utilisateur est affecté n’apparaissent pas dans la revendication de rôle.

### <a name="edit-the-group-claims-configuration"></a>Modifier la configuration des revendications de groupe

Une fois qu’une configuration de revendication de groupe a été ajoutée à la configuration Attributs et revendications de l’utilisateur, l’option permettant d’ajouter une revendication de groupe est grisée.  Pour changer la configuration de revendication de groupe, cliquez sur la revendication de groupe dans la liste **Revendications supplémentaires**.

![Interface utilisateur des revendications](media/how-to-connect-fed-group-claims/group-claims-ui-7.png)

## <a name="configure-the-azure-ad-application-registration-for-group-attributes"></a>Configurer l’inscription d’application Azure AD pour les attributs de groupe

Les revendications de groupe peuvent également être configurées dans la section [Revendications facultatives](../../active-directory/develop/active-directory-optional-claims.md) du [manifeste de l’application](../../active-directory/develop/reference-app-manifest.md).

1. Dans le portail -> Azure Active Directory -> Inscriptions d’applications, sélectionnez Application -> Manifeste

2. Activer les revendications d’appartenance de groupe en modifiant la revendication groupMembershipClaim

Les valeurs autorisées sont :

| Sélection | Description |
|----------|-------------|
| **"All"** | Émet des groupes de sécurité, ainsi que des rôles et des listes de distribution |
| **"SecurityGroup"** | Émet les groupes de sécurité dont l’utilisateur est membre dans la revendication de groupe |
| **"DirectoryRole"** | Si l’utilisateur se voit affecter des rôles d’annuaire, ceux-ci sont émis en tant que revendication « wids » (aucune revendication de groupe n’est émise) |
| **"ApplicationGroup"** | Émet uniquement les groupes qui sont explicitement affectés à l’application et dont l’utilisateur est membre |
| **"None"** | Aucun groupe n’est retourné. (Non sensible à la case, le paramètre « None » fonctionne également et il peut être défini directement dans le manifeste d’application.) |

   Par exemple :

   ```json
   "groupMembershipClaims": "SecurityGroup"
   ```

   Par défaut, les ObjectId de groupe sont émis dans la valeur de revendication de groupe.  Pour modifier la valeur de revendication afin qu’elle contienne des attributs de groupe local, ou pour modifier le type de revendication en rôle, utilisez une configuration OptionalClaims comme suit :

3. Définissez des revendications facultatives de configuration de nom de groupe.

   Si vous souhaitez que les groupes dans le jeton contiennent les attributs de groupe AD local, spécifiez la revendication facultative de type de jeton à laquelle appliquer la configuration dans la section des revendications facultatives.  Plusieurs types de jetons peuvent être répertoriés :

   - idToken pour le jeton d’ID d’OIDC ;
   - accessToken pour le jeton d’accès OAuth/OIDC ;
   - Saml2Token pour les jetons SAML.

   > [!NOTE]
   > Le type de jeton Saml2Token s’applique aux jetons de format SAML1.1 et SAML2.0

   Pour chaque type de jeton pertinent, modifiez la revendication de groupe pour utiliser la section OptionalClaims dans le manifeste. Le schéma d’OptionalClaims est le suivant :

   ```json
   {
   "name": "groups",
   "source": null,
   "essential": false,
   "additionalProperties": []
   }
   ```

   | Schéma de revendications facultatives | Valeur |
   |----------|-------------|
   | **name:** | Doit être « groupes » |
   | **source:** | Non utilisé. Omettez ou spécifiez la valeur null |
   | **essential:** | Non utilisé. Omettez ou spécifiez la valeur false |
   | **additionalProperties:** | Liste de propriétés supplémentaires.  Les options valides sont « sam_account_name », « dns_domain_and_sam_account_name », « netbios_domain_and_sam_account_name » et « emit_as_roles » |

   Dans additionalProperties une seule des options « sam_account_name », « dns_domain_and_sam_account_name » ou « netbios_domain_and_sam_account_name » est requise.  Si plusieurs options sont présentes, la première est utilisée et les autres ignorées.

   Certaines applications requièrent des informations de groupe sur l’utilisateur dans la revendication de rôle.  Pour modifier le type de revendication de revendication de groupe en revendication de rôle, ajoutez « emit_as_roles » aux propriétés supplémentaires.  Les valeurs de groupe sont émises dans la revendication de rôle.

   > [!NOTE]
   > Si l’option « emit_as_roles » est utilisée, les rôles d’application configurés auxquels l’utilisateur est affecté n’apparaissent pas dans la revendication de rôle

### <a name="examples"></a>Exemples

Émettre des groupes en tant que noms de groupes dans des jetons d’accès OAuth au format dnsDomainName\SAMAccountName

```json
"optionalClaims": {
    "accessToken": [{
        "name": "groups",
        "additionalProperties": ["dns_domain_and_sam_account_name"]
    }]
}
 ```

Pour émettre des noms de groupes à retourner au format netbiosDomain\samAccountName en tant que revendication de rôles dans des jetons d’ID SAML et OIDC :

```json
"optionalClaims": {
    "saml2Token": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }],

    "idToken": [{
        "name": "groups",
        "additionalProperties": ["netbios_name_and_sam_account_name", "emit_as_roles"]
    }]
 }
 ```

## <a name="next-steps"></a>Étapes suivantes

- [Ajouter une autorisation à une application web ASP.NET Core à l’aide de groupes et revendications de groupe (exemple de code)](https://github.com/Azure-Samples/active-directory-aspnetcore-webapp-openidconnect-v2/blob/master/5-WebApp-AuthZ/5-2-Groups/README.md)
- [Affecter un utilisateur ou un groupe à une application d’entreprise](../../active-directory/manage-apps/assign-user-or-group-access-portal.md)
- [Configurer des revendications de rôle](../../active-directory/develop/active-directory-enterprise-app-role-management.md)

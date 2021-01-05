---
title: 'Accès conditionnel : Exiger l’acceptation un appareil géré - Azure Active Directory'
description: Découvrez comment configurer des stratégies d’accès conditionnel Azure Active Directory (Azure AD) basées sur les appareils qui exigent des appareils gérés pour accéder aux applications cloud.
services: active-directory
ms.service: active-directory
ms.subservice: conditional-access
ms.topic: how-to
ms.date: 10/16/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: jairoc
ms.collection: M365-identity-device-management
ms.openlocfilehash: c11b58954eefda67f981d618b04ab2bd69fa6b43
ms.sourcegitcommit: 3bdeb546890a740384a8ef383cf915e84bd7e91e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93077759"
---
# <a name="how-to-require-managed-devices-for-cloud-app-access-with-conditional-access"></a>Procédure : Exiger des appareils gérés pour accéder aux applications cloud avec l’accès conditionnel

Dans un monde où la mobilité et le cloud occupent le premier plan, Azure Active Directory (Azure AD) vous permet de vous authentifier à l’aide de l’authentification unique sur les applications et les services, où que vous soyez. Les utilisateurs autorisés peuvent accéder à vos applications cloud à partir d’un large éventail d’appareils, y compris les appareils mobiles et personnels. Toutefois, de nombreux environnements comprennent des applications qui ne doivent être accessibles que par des appareils répondant à vos standards de conformité et de sécurité. Ces appareils sont également appelés « appareils gérés ». 

Cet article explique comment vous pouvez configurer des stratégies d’accès conditionnel qui exigent des appareils gérés pour accéder à certaines applications cloud de votre environnement. 

## <a name="prerequisites"></a>Prérequis

Le fait d’exiger des appareils gérés pour accéder aux applications cloud associe **l’accès conditionnel Azure AD** et la **gestion des appareils Azure AD**. Si l’un de ces aspects vous est inconnu, consultez d’abord les rubriques suivantes :

- **[Accès conditionnel dans Azure Active Directory](./overview.md)** : cet article fournit une vue d’ensemble conceptuelle de l’accès conditionnel et explique la terminologie associée.
- **[Présentation de la gestion des appareils dans Azure Active Directory](../devices/overview.md)** : cet article vous donne une vue d’ensemble des différentes options dont vous disposez pour faire passer les appareils sous le contrôle de l’organisation. 
- Dans **Windows 10 Creators Update (version 1703)** ou version ultérieure, la prise en charge de Chrome nécessite l’installation de l’ [extension Comptes Windows 10](https://chrome.google.com/webstore/detail/windows-10-accounts/ppnbnpeolgkicgegkbkbjmhlideopiji). Cette extension est nécessaire lorsqu’une stratégie d’accès conditionnel exige des informations concernant l’appareil.

>[!NOTE] 
> Nous vous recommandons d’utiliser une stratégie d’accès conditionnel en fonction de l’appareil d’Azure AD pour obtenir la meilleure mise en œuvre après l’authentification initiale de l’appareil. Cela comprend la fermeture de sessions si l’appareil devient non conforme et sort du flux de code d’appareil.


## <a name="scenario-description"></a>Description du scénario

Il n’est pas facile de trouver le bon équilibre entre sécurité et productivité. L’augmentation du nombre d’appareils pris en charge pour accéder aux ressources cloud permet d’améliorer la productivité de vos utilisateurs. Toutefois, il est probable que vous ne souhaitiez pas que certaines ressources de votre environnement soient accessibles par des appareils dont le niveau de protection est inconnu. Pour ce type de ressources, vous devez exiger que les utilisateurs y accèdent uniquement à l’aide d’un appareil géré. 

L’accès conditionnel Azure AD permet de répondre à cette exigence avec une seule stratégie qui accorde l’accès :

- à certaines applications cloud ;
- pour certains utilisateurs et groupes ;
- Exiger un appareil géré

## <a name="managed-devices"></a>Appareils gérés  

En termes simples, les appareils gérés sont ceux que l’organisation peut contrôler *d’une manière ou d’une autre*. Dans Azure AD, les prérequis pour un appareil géré implique son inscription auprès d’Azure AD. L’inscription d’un appareil crée une identité pour lui sous la forme d’un objet appareil. Cet objet est utilisé par Azure pour effectuer le suivi des informations d’état relatives à un appareil. En tant qu’administrateur Azure AD, vous pouvez déjà utiliser cet objet pour permuter (activer/désactiver) l’état d’un appareil.
  
:::image type="content" source="./media/require-managed-devices/32.png" alt-text="Capture d’écran du volet Appareil dans Azure AD. Les éléments Activer et Désactiver sont mis en évidence." border="false":::

Pour inscrire un appareil auprès d’Azure AD, vous avez trois possibilités : 

- **Appareils inscrits sur Azure AD** : pour inscrire un appareil personnel auprès d’Azure AD
- **Appareils joints à Azure AD** : pour inscrire auprès d’Azure AD un appareil Windows 10 professionnel qui n’est pas joint à une version locale d’AD. 
- **Appareils joints à une version Azure AD Hybride** : pour inscrire auprès d’Azure AD un appareil Windows 10 ou de bas niveau pris en charge joint à un AD local.

Ces trois options sont décrites dans l’article [Qu’est-ce qu’une identité d’appareil ?](../devices/overview.md)

Pour avoir le statut d’appareil géré, un appareil inscrit doit être soit un **appareil hybride joint à Azure AD** , soit un **appareil marqué comme conforme**.  

:::image type="content" source="./media/require-managed-devices/47.png" alt-text="Capture d’écran du volet Octroi d’Azure AD. Accorder l’accès est sélectionné, tout comme les cases à cocher pour que les appareils soient conformes et aient une jointure hybride Azure AD." border="false":::
 
## <a name="require-hybrid-azure-ad-joined-devices"></a>Exiger un appareil joint à une version hybride d’Azure AD

Dans votre stratégie d’accès conditionnel, vous pouvez sélectionner **Exiger un appareil joint à une version Azure AD Hybride** pour indiquer que les applications cloud sélectionnées sont uniquement accessibles à l’aide d’un appareil géré. 

:::image type="content" source="./media/require-managed-devices/10.png" alt-text="Capture d’écran du volet Octroi d’Azure AD. Accorder l’accès est sélectionné. Une case à cocher exigeant que les appareils aient une jointure hybride Azure AD est également sélectionnée." border="false":::

Ce paramètre s’applique uniquement à Windows 10 ou aux appareils de bas niveau tels que Windows 7 ou Windows 8 qui sont joints à un AD local. Vous pouvez uniquement inscrire ces appareils auprès d’Azure AD en utilisant une jointure à une version hybride d’Azure AD, ce qui correspond à un [processus automatisé](../devices/hybrid-azuread-join-plan.md) pour inscrire un appareil Windows 10. 

:::image type="content" source="./media/require-managed-devices/45.png" alt-text="Table répertoriant le nom, l’état activé, le système d’exploitation, la version, le type de jointure, le propriétaire, le système MDM et l’état de conformité d’un appareil. L’état de conformité est Non." border="false":::

Qu’est-ce qui transforme un appareil joint à une version hybride d’Azure AD en appareil géré ?  Pour les appareils joints à une version locale d’Active Directory, le contrôle est supposé être appliqué à l’aide de solutions de gestion comme **Configuration Manager** ou une **stratégie de groupe**. Étant donné qu’il n’existe aucune méthode permettant à Azure AD de déterminer si une de ces méthodes a été appliquée à un appareil, exiger un appareil joint à une version hybride d’Azure AD constitue un mécanisme relativement faible pour exiger un appareil géré. Il vous appartient en tant qu’administrateur de déterminer si les méthodes appliquées à vos appareils joints au domaine local sont suffisamment fortes pour constituer un appareil géré si ces appareils sont également joints à une version hybride d’Azure AD.

## <a name="require-device-to-be-marked-as-compliant"></a>Exiger que l’appareil soit marqué comme conforme

L’option *Exiger que l’appareil soit marqué comme conforme* est la façon la plus forte de demander un appareil géré.

:::image type="content" source="./media/require-managed-devices/11.png" alt-text="Capture d’écran du volet Octroi d’Azure AD. Accorder l’accès est sélectionné. Une case à cocher exigeant qu’un appareil soit marqué comme conforme est également sélectionnée." border="false":::

Cette option exige qu’un appareil soit inscrit auprès d’Azure AD et qu’il soit marqué comme conforme par :
         
- Intune
- Système MDM tiers qui gère les appareils Windows 10 via l’intégration d’Azure AD. Les systèmes MDM tiers pour les systèmes d’exploitation autres que Windows 10 ne sont pas pris en charge.
 
:::image type="content" source="./media/require-managed-devices/46.png" alt-text="Table répertoriant le nom, l’état activé, le système d’exploitation, la version, le type de jointure, le propriétaire, le système MDM et l’état de conformité d’un appareil. L’état de conformité est mis en évidence." border="false":::

Pour un appareil marqué comme conforme, vous pouvez partir du principe que : 

- Les appareils mobiles que votre personnel utilise pour accéder aux données d’entreprise sont gérés.
- Les appareils mobiles que votre personnel utilise sont gérés.
- Les informations de votre entreprise sont protégées grâce au contrôle de la manière dont votre personnel y accède et les partage.
- L’appareil et ses applications sont conformes aux exigences de sécurité de l’entreprise.

### <a name="scenario-require-device-enrollment-for-ios-and-android-devices"></a>Scénario : Exiger l’inscription des appareils iOS et Android

Dans ce scénario, Contoso a décidé que tout accès mobile aux ressources Microsoft 365 doit passer par un appareil inscrit. Tous les utilisateurs de Contoso se connectent déjà à l’aide d’informations d’identification Azure AD et disposent des licences qui leur sont attribuées, notamment Azure AD Premium P1 ou P2 et Microsoft Intune.

Les organisations doivent suivre les étapes ci-dessous pour exiger l’utilisation d’un appareil mobile inscrit.

1. Connectez-vous au **portail Microsoft Azure** en tant qu’administrateur général, administrateur de sécurité ou administrateur de l’accès conditionnel.
1. Accédez à **Azure Active Directory** > **Sécurité** > **Accès conditionnel.**
1. Sélectionnez **Nouvelle stratégie**.
1. Donnez un nom à votre stratégie. Nous recommandons aux organisations de créer une norme explicite pour les noms de leurs stratégies.
1. Sous **Affectations** , sélectionnez **Utilisateurs et groupes**
   1. Sous **Inclure** , sélectionnez **Tous les utilisateurs** ou les **Utilisateurs et groupes** particuliers auxquels vous souhaitez appliquer cette stratégie. 
   1. Sélectionnez **Terminé**.
1. Sous **Applications cloud ou actions** > **Inclure** , sélectionnez **Office 365**.
1. Sous **Conditions** , sélectionnez **Plateformes d’appareils**.
   1. Définissez **Configurer** sur **Oui**.
   1. Incluez **Android** et **iOS**.
1. Sous **Contrôles d’accès** > **Octroyer** , sélectionnez les options suivantes :
   - **Exiger que l’appareil soit marqué comme conforme**
1. Confirmez vos paramètres et réglez **Activer la stratégie** sur **Activé**.
1. Sélectionnez **Créer** pour créer et activer votre stratégie.

### <a name="known-behavior"></a>Comportement connu

Lorsque le [flux OAuth de code d’appareil](../develop/v2-oauth2-device-code.md) est utilisé, ni le contrôle Exiger une autorisation d’appareil géré ni la condition d’état d’appareil ne sont pris en charge. En effet, l’appareil qui effectue l’authentification ne peut pas fournir son état à celui qui fournit un code, et l’état de l’appareil dans le jeton est verrouillé sur celui qui effectue l’authentification. Utilisez plutôt le contrôle Exiger l’autorisation d’authentification multifacteur.

Sur des systèmes Windows 7, iOS, Android, macOS, et dans certains navigateurs web tiers, Azure AD identifie l’appareil à l’aide d’un certificat client, qui est provisionné lorsque l’appareil est inscrit auprès d’Azure AD. Lorsqu’un utilisateur se connecte pour la première fois via le navigateur, l’utilisateur est invité à sélectionner le certificat. L’utilisateur final doit sélectionner ce certificat pour pouvoir continuer à utiliser le navigateur.

## <a name="next-steps"></a>Étapes suivantes

[Évaluez l’impact des stratégies d’accès conditionnel avant de les activer en mode rapport seul](concept-conditional-access-report-only.md).

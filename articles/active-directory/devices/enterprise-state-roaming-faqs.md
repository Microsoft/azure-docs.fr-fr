---
title: FAQ Enterprise State Roaming – Azure Active Directory
description: Forum aux questions sur Enterprise State Roaming
services: active-directory
ms.service: active-directory
ms.subservice: devices
ms.topic: troubleshooting
ms.date: 02/12/2020
ms.author: joflore
author: MicrosoftGuyJFlo
manager: daveba
ms.reviewer: na
ms.collection: M365-identity-device-management
ms.openlocfilehash: cb889298a09c30a629c69442ebf31bc735af31d1
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96173122"
---
# <a name="settings-and-data-roaming-faq"></a>FAQ sur l’itinérance des paramètres et des données

Cet article répond à certaines questions que les administrateurs informatiques peuvent se poser sur les paramètres et la synchronisation des données d’application.

## <a name="what-data-roams"></a>Quelles données sont itinérantes ?

Les **Paramètres Windows** : les paramètres du PC intégrés au système d’exploitation Windows. En règle générale, ce sont des paramètres qui personnalisent votre PC. Ils incluent les catégories principales suivantes :

* Le *Thème* comprenant des fonctionnalités telles que les paramètres de thème du bureau et de la barre des tâches.
* Les *paramètres Internet Explorer* comprenant les onglets récemment ouverts et les favoris.
* Les *Paramètres du navigateur Microsoft Edge*, tel que les favoris et la liste de lecture.
* Les *Mots de passe* comprenant les mots de passe Internet, les profils Wi-Fi, etc.
* Les *Préférences linguistiques* comprenant la disposition du clavier, la langue du système, la date et l’heure, etc.
* Les *Options d’ergonomie* comprenant les thèmes à contraste élevé, le Narrateur et la Loupe.
* *Autres paramètres Windows*, tels ceux de la souris.

> [!NOTE]
> Cet article s’applique au navigateur HTML hérité Microsoft Edge lancé avec Windows 10 en juillet 2015. L’article ne s’applique pas au nouveau navigateur Microsoft Edge basé sur Chromium publié le 15 janvier 2020. Pour plus d’informations sur le comportement de synchronisation pour le nouveau navigateur Microsoft Edge, consultez l’article [Synchronisation de Microsoft Edge](/deployedge/microsoft-edge-enterprise-sync).

**Données d’application** : Les applications Windows universelles peuvent écrire des données de paramètres dans un dossier « roaming » (itinérance), et toutes les données écrites dans ce dossier sont automatiquement synchronisées. Il incombe aux développeurs d’applications de concevoir eux-mêmes une application pour tirer parti de cette fonctionnalité. Pour plus d’informations sur la façon de développer une application Windows universelle qui utilise l’itinérance, consultez [l’API de stockage des données d’application](/windows/uwp/design/app-settings/store-and-retrieve-app-data) et le [blog des développeurs d’applications itinérantes pour Windows 8](https://blogs.windows.com/windowsdeveloper/2016/05/04/roaming-app-data-and-the-user-experience/).

## <a name="what-account-is-used-for-settings-sync"></a>Quel compte est utilisé pour la synchronisation des paramètres ?

Dans Windows 8.1, la synchronisation des paramètres a toujours utilisé les comptes Microsoft consommateur. Les utilisateurs de l’entreprise avaient la possibilité de connecter un compte Microsoft à leur compte de domaine Active Directory pour avoir accès à la synchronisation des paramètres. Dans Windows 10, cette fonctionnalité « compte Microsoft connecté » est remplacée par une structure de compte principal et secondaire.

Le compte principal est défini en tant que compte utilisé pour se connecter à Windows. Il peut s’agir d’un compte Microsoft, d’un compte Azure Active Directory (Azure AD), d’un compte Active Directory local ou d’un compte local. En plus du compte principal, les utilisateurs de Windows 10 peuvent ajouter à leur appareil un ou plusieurs comptes cloud. Un compte secondaire est généralement un compte Microsoft, Active Directory, ou un autre compte tel que Gmail ou Facebook. Ces comptes secondaires permettent d’accéder à des services supplémentaires tels que l’authentification unique et le Windows Store, mais ne prennent pas en charge la synchronisation des paramètres.

Dans Windows 10, seul le compte principal de l’appareil peut être utilisé pour la synchronisation des paramètres (consultez la rubrique [Comment passer de la synchronisation des paramètres de compte Microsoft dans Windows 8 à la synchronisation des paramètres Azure AD dans Windows 10 ?](enterprise-state-roaming-faqs.md#how-do-i-upgrade-from-microsoft-account-settings-sync-in-windows-8-to-azure-ad-settings-sync-in-windows-10)).

Les données des différents comptes d’utilisateur du périphérique ne sont jamais mélangées. Il existe deux règles de synchronisation des paramètres :

* les paramètres Windows sont toujours itinérants avec le compte principal ;
* les données d’application sont marquées avec le compte utilisé pour l’acquisition de l’application. Seules les applications marquées avec le compte principal sont synchronisées. Le marquage de propriété des applications est déterminé lors du chargement d’une application en version test via le Windows Store ou via la gestion des appareils mobiles (MDM).

S’il est impossible d’identifier le propriétaire de l’application, celle-ci est itinérante avec le compte principal. Si un appareil est mis à niveau à partir de Windows 8 ou Windows 8.1 vers Windows 10, toutes les applications seront marquées comme étant acquises par le compte Microsoft. Cela est dû au fait que la plupart des utilisateurs acquièrent des applications via le Windows Store et qu’il n’existait aucune prise en charge de Windows Store pour les comptes Azure AD avant Windows 10. Si une application est installée via une licence en mode hors connexion, l’application est marquée en utilisant le compte principal de l’appareil.

> [!NOTE]
> Les appareils Windows 10 détenus par une entreprise et connectés à Azure AD ne peuvent plus connecter leurs comptes Microsoft à un compte de domaine. Sur les appareils Windows 10 joints à un environnement Active Directory ou Azure AD, il n’est plus possible de connecter un compte Microsoft à un compte de domaine et de synchroniser toutes les données de l’utilisateur sur le compte Microsoft (par exemple, le compte Microsoft en itinérance via la fonctionnalité « connected Microsoft Account and Active Directory », soit « compte Microsoft connecté et Active Directory »).

## <a name="how-do-i-upgrade-from-microsoft-account-settings-sync-in-windows-8-to-azure-ad-settings-sync-in-windows-10"></a>Comment passer de la synchronisation des paramètres de compte Microsoft dans Windows 8 à la synchronisation des paramètres Azure AD dans Windows 10 ?

Si vous êtes joint au domaine Active Directory fonctionnant sous Windows 8.1 avec un compte Microsoft connecté, vous synchronisez les paramètres via votre compte Microsoft. Après la mise à niveau vers Windows 10, vous continuez à synchroniser les paramètres utilisateur via le compte Microsoft tant que vous êtes un utilisateur joint au domaine  et que le domaine Active Directory ne se connecte pas avec Azure AD.

Si le domaine Active Directory local ne se connecte pas avec Azure AD, votre appareil essaie de synchroniser les paramètres à l’aide du compte Azure AD connecté. Si l’administrateur Azure AD n’active pas Enterprise State Roaming, votre compte Azure AD connecté cessera la synchronisation des paramètres. Si vous êtes un utilisateur Windows 10 et que vous vous connectez avec une identité Azure AD, vous pouvez commencer la synchronisation des paramètres Windows dès que votre administrateur active la synchronisation des paramètres via Azure AD.

Si vous avez stocké des données personnelles sur votre appareil d’entreprise, vous devez avoir conscience que le système d’exploitation Windows et les données d’application commencent à se synchroniser avec Azure AD. Cela entraîne les conséquences suivantes :

* Les paramètres de votre compte personnel Microsoft se séparent des paramètres de vos comptes Azure AD professionnels ou scolaires puisque la synchronisation des paramètres de compte Microsoft et d’Azure AD utilisent désormais des comptes distincts.
* Les données personnelles (mots de passe Wi-Fi, informations d’identification web et favoris d’Internet Explorer) qui étaient auparavant synchronisées via un compte Microsoft seront maintenant synchronisées via Azure AD.

## <a name="how-do-microsoft-account-and-azure-ad-enterprise-state-roaming-interoperability-work"></a>Comment fonctionne l’interopérabilité d’Azure AD Enterprise State Roaming et des comptes Microsoft ?

Dans les versions Windows 10 publiées à partir de novembre 2015 inclus, Enterprise State Roaming n’est pris en charge que pour un compte à la fois. Si vous vous connectez à Windows à l’aide d’un compte Azure AD professionnel ou scolaire, toutes les données seront synchronisées via Azure AD. Si vous vous connectez à Windows à l’aide d’un compte Microsoft personnel, toutes les données sont synchronisées via le compte Microsoft. En itinérance, les données d’application universelles utilisent uniquement le compte de connexion principal de l’appareil seulement si la licence de l’application est détenue par le compte principal. Les données d’application universelles appartenant à un compte secondaire ne seront pas synchronisées.

## <a name="do-settings-sync-for-azure-ad-accounts-from-multiple-tenants"></a>Les paramètres se synchronisent-ils pour les comptes Azure AD regroupant plusieurs clients ?

Lorsque plusieurs comptes Azure AD regroupant différents locataires Azure AD se trouvent sur le même appareil, vous devez mettre à jour le registre de l’appareil afin de communiquer avec le service Azure Rights Management pour chaque locataire Azure AD.  

1. Recherchez le GUID de chaque client Azure AD. Ouvrez le portail Azure et sélectionnez un client Azure AD. Le GUID du locataire se trouve dans la page Propriétés du locataire sélectionné (https://portal.azure.com/#blade/Microsoft_AAD_IAM/ActiveDirectoryMenuBlade/Properties), étiqueté **ID de répertoire**. 
2. Dès que vous disposez du GUID, vous devez ajouter la clé de Registre **HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\SettingSync\WinMSIPC\<tenant ID GUID>** .
   À partir de la clé **tenant ID GUID**, créez une nouvelle valeur de chaînes multiples (REG-MULTI-SZ) nommée **AllowedRMSServerUrls**. Pour ses données, spécifiez les URL de point de distribution de licence des autres clients Azure auxquels l’appareil accède.
3. Vous trouverez les URL de point de distribution de licence en exécutant l’applet de commande **Get-AadrmConfiguration** dans le module AADRM. Si les valeurs des paramètres **LicensingIntranetDistributionPointUrl** et **LicensingExtranetDistributionPointUrl** sont différentes, spécifiez les deux valeurs. Si les valeurs sont les mêmes, ne spécifiez la valeur qu’une seule fois.

## <a name="what-are-the-roaming-settings-options-for-existing-windows-desktop-applications"></a>Quelles sont les options disponibles pour les paramètres d’itinérance des applications de bureau Windows existantes ?

L’itinérance ne fonctionne qu’avec les applications Windows universelles. Deux options permettent d’activer l’itinérance sur une application de bureau Windows existante :

* [Desktop Bridge](/windows/msix/desktop/source-code-overview) vous permet de transférer vos applications de bureau Windows existantes vers la plateforme Windows universelle. À ce stade, seules quelques modifications de code mineures sont nécessaires pour pouvoir tirer parti de l’itinérance des données d’application d’Azure AD. Desktop Bridge fournit à vos applications l’identité d’application nécessaire pour activer l’itinérance des données d’application pour les applications de bureau existantes.
* [User Experience Virtualization (UE-V)](/previous-versions//dn458947(v=vs.85)) vous permet de créer un modèle de paramètres personnalisés pour les applications de bureau Windows existantes et d’activer le mode itinérant pour les applications Win32. Cette option n’oblige pas le développeur d’application à modifier le code de l’application. UE-V est limité à l’itinérance Active Directory en local pour les clients qui ont acheté Microsoft Desktop Optimization Pack.

Les administrateurs peuvent configurer UE-V pour déplacer simplement les données d’application de bureau Windows en modifiant l’itinérance des paramètres du système d’exploitation Windows et des données d’application universelle à l’aide des [stratégies de groupe UE-V](/microsoft-desktop-optimization-pack/uev-v2/configuring-ue-v-2x-with-group-policy-objects-both-uevv2), notamment :

* La stratégie de groupe « Roam Windows settings » (Activer l’itinérance des paramètres Windows)
* La stratégie de groupe « Do not synchronize Windows Apps » (Ne pas synchroniser les applications Windows)
* Internet Explorer roaming in the applications section (Itinérance IE dans la section des applications)

À l’avenir, il est possible que Microsoft cherche des moyens d’intégrer complètement UE-V à Windows et de l’étendre pour permettre l’itinérance des paramètres via le cloud Azure AD.

## <a name="can-i-store-synced-settings-and-data-on-premises"></a>Puis-je stocker localement les paramètres et les données synchronisés ?

Enterprise State Roaming stocke toutes les données synchronisées dans le Microsoft Cloud. UE-V offre une solution d’itinérance locale.

## <a name="who-owns-the-data-thats-being-roamed"></a>À qui appartiennent les données en itinérance ?

Les données en itinérance via Enterprise State Roaming sont détenues par les entreprises. Les données sont stockées dans un centre de données Azure. Toutes les données utilisateur sont chiffrées à l’aide du service Azure Rights Management d’Azure Information Protection lorsqu’elles sont en transit et lorsqu’elles sont au repos dans le cloud. Il s’agit d’une amélioration par rapport à la synchronisation des paramètres de compte Microsoft, qui chiffre seulement certaines données sensibles telles que les informations d’identification utilisateur avant de quitter l’appareil.

Microsoft s’engage à protéger les données client. Les données de paramètres d’un utilisateur de l’entreprise sont automatiquement chiffrées par le service Azure Rights Management avant de quitter un appareil Windows 10, pour ne pas qu’elles puissent être lues par les autres utilisateurs. Si votre organisation dispose d’un abonnement payant au service Azure Rights Management, vous pouvez utiliser d’autres fonctionnalités de protection, telles que le suivi et la révocation de documents, la protection automatique des e-mails contenant des informations sensibles, et la gestion de vos propres clés (solution BYOK). Pour plus d’informations sur ces fonctionnalités et sur le fonctionnement de ce service de protection, consultez [En quoi consiste Azure Rights Management ?](/azure/information-protection/what-is-information-protection).

## <a name="can-i-manage-sync-for-a-specific-app-or-setting"></a>Puis-je gérer la synchronisation d’un paramètre ou d’une application spécifique ?

Dans Windows 10, il n’existe aucun paramètre de MDM ou de stratégie de groupe permettant de désactiver l’itinérance pour une application en particulier. Les administrateurs client peuvent désactiver la synchronisation des données d’application pour toutes les applications sur un appareil géré, mais il n’existe aucun contrôle plus précis au niveau ou au sein de l’application.

## <a name="how-can-i-enable-or-disable-roaming"></a>Comment puis-je activer ou désactiver l’itinérance ?

Dans l’application **Paramètres**, accédez à **Comptes** > **Synchroniser vos paramètres**. Depuis cette page, vous pouvez voir le compte utilisé pour l’itinérance des paramètres et activer ou désactiver l’itinérance pour chacun des groupes de paramètres.

## <a name="what-is-microsofts-recommendation-for-enabling-roaming-in-windows-10"></a>Quelle est la recommandation de Microsoft pour l’activation de l’itinérance dans Windows 10 ?

Microsoft propose quelques solutions d’itinérances de paramètres, notamment les profils utilisateur itinérant, UE-V et Enterprise State Roaming.  Microsoft s’engage à investir davantage dans la solution Enterprise State Roaming pour les futures versions de Windows. Si votre organisation n’est pas prête à déplacer les données dans le cloud ou qu’elle ne se sent pas à l’aise avec cette idée, nous vous recommandons d’utiliser UE-V comme principale technologie d’itinérance. Si votre organisation a besoin d’une prise en charge de l’itinérance pour les applications de bureau Windows existantes, mais qu’elle est prête à migrer vers le cloud, nous vous recommandons d’utiliser à la fois Enterprise State Roaming et UE-V. Bien que UE-V et Enterprise State Roaming soient des technologies très similaires, elles ne sont pas mutuellement exclusives. Elles se complètent pour vous assurer que votre organisation offre des services d’itinérance dont vos utilisateurs ont besoin.  

Lorsque vous utilisez à la fois Enterprise State Roaming et UE-V, les règles suivantes s’appliquent :

* Enterprise State Roaming est l’agent itinérant principal sur l’appareil. UE-V est utilisé pour compléter l’« intervalle Win32 ».
* Les paramètres d’itinérance UE-V pour Windows et les données d’application UWP modernes doivent être désactivés lorsque vous utilisez des stratégies de groupe UE-V. Ceux-ci sont déjà couverts par Enterprise State Roaming.

## <a name="how-does-enterprise-state-roaming-support-virtual-desktop-infrastructure-vdi"></a>Comment la solution Enterprise State Roaming prend-elle en charge Virtual Desktop Infrastructure (VDI) ?

Enterprise State Roaming est pris en charge sur les SKU client Windows 10, mais pas sur les éditions serveur. Si une machine virtuelle cliente est hébergée sur un hyperviseur alors que vous vous connectez à distance à la machine virtuelle, vos données seront alors en mode itinérant. Si plusieurs utilisateurs partagent le même système d’exploitation et que les utilisateurs se connectent à distance à un serveur pour bénéficier d’une expérience de bureau complète, l’itinérance peut ne pas fonctionner. Ce dernier scénario n’est pas officiellement pris en charge.

## <a name="what-happens-when-my-organization-purchases-a-subscription-that-includes-azure-rights-management-after-using-roaming"></a>Que se passe-t-il quand mon organisation achète un abonnement comprenant Azure Rights Management après avoir utilisé l’itinérance ?

Si votre organisation utilise déjà l’itinérance dans Windows 10 avec l’abonnement gratuit à Azure Rights Management en utilisation limitée, l’achat d’un [abonnement payant](https://azure.microsoft.com/pricing/details/information-protection/) au service de protection Azure Rights Management n’a aucun impact sur les fonctionnalités de l’itinérance et votre administrateur informatique n’a pas non plus besoin de changer la configuration.

## <a name="known-issues"></a>Problèmes connus

Pour obtenir la liste des problèmes connus, consultez la documentation indiquée dans la section [Résolution des problèmes](enterprise-state-roaming-troubleshooting.md). 

## <a name="next-steps"></a>Étapes suivantes 

Pour une vue d’ensemble, consultez [Vue d’ensemble d’Enterprise State Roaming](enterprise-state-roaming-overview.md).
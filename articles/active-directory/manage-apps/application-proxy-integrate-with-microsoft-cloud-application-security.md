---
title: Intégrer des applications locales avec Cloud App Security – Azure AD
description: Configurez une application locale dans Azure Active Directory pour utiliser Microsoft Cloud App Security (MCAS). Utilisez le contrôle d’application par accès conditionnel MCAS pour surveiller et contrôler les sessions en temps réel en fonction des stratégies d’accès conditionnel. Vous pouvez appliquer ces stratégies aux applications locales qui utilisent le proxy d’application dans Azure Active Directory (Azure AD).
author: kenwith
manager: celestedg
ms.service: active-directory
ms.subservice: app-mgmt
ms.workload: identity
ms.topic: how-to
ms.date: 12/19/2018
ms.author: kenwith
ms.reviewer: japere
ms.collection: M365-identity-device-management
ms.openlocfilehash: 889168782acabaafa4200a8ebd097e431a68ede3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84764568"
---
# <a name="configure-real-time-application-access-monitoring-with-microsoft-cloud-app-security-and-azure-active-directory"></a>Configurer la supervision de l’accès aux applications en temps réel avec Microsoft Cloud App Security et Azure Active Directory
Configurez une application locale dans Azure Active Directory (Azure AD) pour utiliser Microsoft Cloud App Security (MCAS) pour la supervision en temps réel. MCAS utilise le contrôle d’application par accès conditionnel pour surveiller et contrôler les sessions en temps réel en fonction des stratégies d’accès conditionnel. Vous pouvez appliquer ces stratégies aux applications locales qui utilisent le proxy d’application dans Azure Active Directory (Azure AD).

Voici quelques exemples de types de stratégies que vous pouvez créer avec MCAS :

- Bloquez ou protégez le téléchargement de documents sensibles sur les appareils non gérés.
- Surveillez à quel moment les utilisateurs à haut risque se connectent aux applications, puis consignez leurs actions dans le cadre de la session. Avec ces informations, vous pouvez analyser le comportement de l’utilisateur pour déterminer comment appliquer les stratégies de session.
- Utilisez les certificats clients ou la conformité des périphériques pour bloquer l’accès à des applications spécifiques à partir des appareils non gérés.
- Restreignez les sessions utilisateur des réseaux hors entreprise. Vous pouvez donner un accès restreint aux utilisateurs accédant à une application en dehors de votre réseau d’entreprise. Par exemple, cet accès restreint peut empêcher l’utilisateur de télécharger des documents sensibles.

Pour plus d’informations, consultez [Protéger les applications avec le Contrôle d’accès conditionnel aux applications Microsoft Cloud App Security](/cloud-app-security/proxy-intro-aad).

## <a name="requirements"></a>Spécifications

Licence :

- Licence EMS E5 ou 
- Azure Active Directory Premium P1 et MCAS Standalone.

Application locale :

- L’application locale doit utiliser Kerberos Constrained Delegation (KCD)

Configurez le proxy d’application :

- Configurez Azure AD pour utiliser le proxy d’application, y compris la préparation de votre environnement et l’installation du connecteur Proxy d’application. Pour accéder à un tutoriel, consultez [Ajouter des applications locales pour un accès à distance via le service Proxy d’application d’Azure AD](application-proxy-add-on-premises-application.md). 

## <a name="add-on-premises-application-to-azure-ad"></a>Intégrer une application locale dans Azure AD

Intégrez une application locale dans Azure AD. Pour un démarrage rapide, consultez [Intégrer une application locale dans Azure AD](application-proxy-add-on-premises-application.md#add-an-on-premises-app-to-azure-ad). Lors de l’ajout de l’application, assurez-vous de régler les deux paramètres suivants dans le panneau **Ajouter votre application locale** :

- **Pré-authentification** : Entrez **Azure Active Directory**.
- **Traduisez les URL dans le corps de l’application** : Choisissez **Oui**.

Ces deux paramètres sont nécessaires pour que l’application fonctionne avec MCAS.

## <a name="test-the-on-premises-application"></a>Tester l’application locale

Après avoir intégré votre application dans Azure AD, effectuez la procédure [Tester l’application](application-proxy-add-on-premises-application.md#test-the-application) pour ajouter un utilisateur de test et tester la connexion. 

## <a name="deploy-conditional-access-app-control"></a>Déployer le contrôle d’application par accès conditionnel

Pour configurer votre application avec le contrôle d’application à accès conditionnel, suivez les instructions de la section [Déployer le Contrôle d’application par accès conditionnel Azure AD](/cloud-app-security/proxy-deployment-aad).


## <a name="test-conditional-access-app-control"></a>Tester le contrôle d’application par accès conditionnel

Pour tester le déploiement des applications Azure AD avec le contrôle d’application à accès conditionnel, suivez les instructions de la section [Tester le déploiement d’applications Azure AD](/cloud-app-security/proxy-deployment-aad).






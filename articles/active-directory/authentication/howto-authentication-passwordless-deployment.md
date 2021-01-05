---
title: Planifier un déploiement d’authentification sans mot de passe avec Azure AD
description: Apprendre à planifier et à déployer une implémentation d'authentification sans mot de passe pour Azure Active Directory
services: active-directory
ms.service: active-directory
ms.subservice: authentication
ms.topic: conceptual
ms.date: 01/30/2020
ms.author: baselden
author: justinha
manager: daveba
ms.reviewer: baselden, librown
ms.collection: M365-identity-device-management
ms.openlocfilehash: a786907c5c954aa45de266b6d92dd47867a8445d
ms.sourcegitcommit: ad83be10e9e910fd4853965661c5edc7bb7b1f7c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/06/2020
ms.locfileid: "96743613"
---
# <a name="plan-a-passwordless-authentication-deployment-in-azure-active-directory"></a>Planifier un déploiement d’authentification sans mot de passe dans Azure Active Directory

> [!NOTE]
> Pour créer une version hors connexion de ce plan de déploiement, utilisez la fonctionnalité d’impression Enregistrer au format PDF de votre navigateur.

La plupart des cyberattaques commencent par un nom d’utilisateur et un mot de passe compromis. Les organisations essaient de contrer la menace en demandant aux utilisateurs d’adopter l’une des approches suivantes :

- Mots de passe longs
- Mots de passe complexes
- Mots de passe changés fréquemment
- Authentification multifacteur (MFA)

Les [études menées par Microsoft montrent](https://aka.ms/passwordguidance) que ces efforts demandés gênent les utilisateurs et entraînent une augmentation des coûts de support. Pour en savoir plus, consultez l’article [Your Pa$$word doesn’t matter](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Your-Pa-word-doesn-t-matter/ba-p/731984).

### <a name="benefits-of-passwordless-authentication"></a>Avantages de l’authentification sans mot de passe

- **Sécurité renforcée**. Réduisez les risques d’attaques par hameçonnage et pulvérisation de mots de passe en retirant les mots de passe de la surface d’attaque.
-  **Meilleure expérience utilisateur**. Offrez aux utilisateurs un moyen pratique d’accéder aux données où qu’ils soient. Fournissez un accès facile aux applications et services comme Outlook, OneDrive ou Office tout en étant mobile.
-  **Insights robustes**. Obtenez des insights sur l’activité sans mot de passe des utilisateurs au moyen d’une journalisation et d’un audit fiables.

Avec une approche sans mot de passe, le mot de passe est remplacé par une chose que vous avez et une chose que vous êtes ou que vous connaissez. Par exemple, Windows Hello Entreprise peut utiliser un mouvement biométrique tel qu’un visage ou une empreinte digitale, ou un code confidentiel propre à un appareil et qui n’est pas transmis sur un réseau.

## <a name="passwordless-authentication-methods"></a>Méthodes d’authentification sans mot de passe
Microsoft propose trois options d’authentification sans mot de passe qui conviennent à de nombreux scénarios. Ces méthodes peuvent être utilisées conjointement :

- [Windows Hello Entreprise](./concept-authentication-passwordless.md) est idéal pour les personnes utilisant leur ordinateur Windows dédié.
- La connexion par clé de sécurité avec les [clés de sécurité FIDO2](./concept-authentication-passwordless.md) s’avère particulièrement pratique pour les utilisateurs qui se connectent à des machines partagées comme des bornes, mais aussi dans des situations où l’utilisation de téléphones est limitée, et pour des identités à privilèges élevés.
- La connexion par téléphone avec [l’application Microsoft Authenticator](./concept-authentication-passwordless.md) est utile pour fournir une option sans mot de passe aux personnes disposant d’appareils mobiles. L’application Authenticator convertit n’importe quel téléphone iOS ou Android en information d’identification forte dépourvue de mot de passe, permettant aux utilisateurs de se connecter sur tout navigateur ou plateforme. Les utilisateurs se connectent avec l’obtention d’une notification sur leur téléphone, la mise en correspondance d’un nombre affiché à l’écran avec celui de leur téléphone et l’utilisation de leurs données biométriques ou de leur code confidentiel pour la confirmation.

### <a name="passwordless-authentication-scenarios"></a>Scénarios d’authentification sans mot de passe

Les méthodes d’authentification sans mot de passe de Microsoft permettent différents scénarios. Tenez compte des besoins de votre organisation, des prérequis et des fonctionnalités de chaque méthode d’authentification pour sélectionner votre stratégie d’authentification sans mot de passe. Nous recommandons à toutes les organisations équipées d’appareils Windows 10 d’utiliser Windows Hello Entreprise. Ensuite, ajoutez une connexion par téléphone (avec l’application Microsoft Authenticator) ou des clés de sécurité pour des scénarios supplémentaires.

| Scénario | Authentification par téléphone | Clés de sécurité | Windows Hello Entreprise |
| --- | --- | --- | --- |
| **Connexion de l’ordinateur** : <br> Depuis un appareil Windows 10 attribué | **Non** | **Oui** <br> Avec biométrie, code confidentiel | **Oui**<br>avec reconnaissance biométrique et/ou code confidentiel |
| **Connexion de l’ordinateur** : <br> Depuis un appareil Windows 10 partagé | **Non** | **Oui** <br> Avec biométrie, code confidentiel  | **Non** |
| **Connexion avec une application web** : <br>Depuis un ordinateur dédié à l’utilisateur | **Oui** | **Oui** <br> L’authentification unique fournie aux applications est activée par la connexion de l’ordinateur | **Oui**<br> L’authentification unique fournie aux applications est activée par la connexion de l’ordinateur |
| **Connexion avec une application web** : <br> Depuis un appareil mobile ou non Windows | **Oui** | **Non** | **Non** |
| **Connexion de l’ordinateur** : <br> Ordinateur non Windows | **Non** | **Non** | **Non** |

Pour plus d’informations sur la sélection de la meilleure méthode pour votre organisation, consultez [Choix d’une méthode sans mot de passe](./concept-authentication-passwordless.md#choose-a-passwordless-method).

## <a name="prerequisites"></a>Prérequis

Les organisations doivent respecter les prérequis suivants avant d’initier un déploiement sans mot de passe :

| Configuration requise | Application Authenticator | Clés de sécurité FIDO2 |
| --- | --- | --- |
| [L'inscription combinée d'Azure AD Multi-Factor Authentication et de la réinitialisation de mot de passe en libre-service (SSPR)](howto-registration-mfa-sspr-combined.md) est activée | √ | √ |
| [Les utilisateurs peuvent exécuter Azure AD Multi-Factor Authentication](howto-mfa-getstarted.md) | √ | √ |
| [Les utilisateurs se sont inscrits à Azure AD Multi-Factor Authentication et SSPR](howto-registration-mfa-sspr-combined.md) | √ | √ |
| [Les utilisateurs ont inscrit leurs appareils mobiles sur Azure Active Directory](../devices/overview.md) | √ |   |
| Windows 10 version 1809 ou supérieure, utilisant un navigateur pris en charge, comme Microsoft Edge ou Mozilla Firefox <br> (version 67 ou supérieure). <br> *Microsoft recommande la version 1903 ou supérieure pour la prise en charge native*. |   | √ |
| Des clés de sécurité FIDO2 compatibles. Vérifiez que vous utilisez un appareil de sécurité FIDO2 [vérifié et testé Microsoft](./concept-authentication-passwordless.md), ou un autre appareil de sécurité FIDO2 compatible. |   | √ |

### <a name="prerequisites-for-windows-hello-for-business"></a>Prérequis pour Windows Hello Entreprise

Les prérequis pour Windows Hello dépendent étroitement de la configuration, locale, hybride ou cloud uniquement, dans laquelle vous déployez. Pour plus d’informations, consultez la [liste complète des prérequis pour Windows Hello Entreprise](/windows/security/identity-protection/hello-for-business/hello-identity-verification).

### <a name="azure-ad-multi-factor-authentication"></a>Authentification multifacteur Azure AD

Les utilisateurs inscrivent leur méthode sans mot de passe dans le cadre du processus d'inscription Azure AD Multi-Factor Authentication. L’authentification multifacteur avec un nom d’utilisateur et un mot de passe, associée à une autre méthode inscrite, peut être utilisée comme solution de secours, au cas où ils ne peuvent pas utiliser leur téléphone ou leur clé de sécurité dans certains scénarios.

### <a name="licensing"></a>Licence 
Il n’y a aucun coût supplémentaire pour l’authentification sans mot de passe ; en revanche, certains prérequis peuvent nécessiter un abonnement Premium. Pour obtenir des informations détaillées sur les fonctionnalités et les licences, consultez la page [Tarification Azure Active Directory](https://azure.microsoft.com/pricing/details/active-directory/). 

## <a name="develop-a-plan"></a>Développer un plan

Prenez en compte les besoins de votre entreprise et les cas d’utilisation pour chaque méthode d’authentification. Sélectionnez ensuite l’option qui répond le mieux à vos besoins.

### <a name="use-cases"></a>Cas d'utilisation

Le tableau suivant présente les cas d’usage à implémenter au cours de ce projet.

| Domaine | Description |
| --- | --- |
| **y accéder** | La connexion sans mot de passe est disponible à partir d’un appareil personnel ou d’entreprise, au sein ou en dehors du réseau d’entreprise. |
| **Audit** | Les données d’utilisation sont accessibles aux administrateurs pour effectuer un audit pratiquement en temps réel. <br> Les données d’utilisation sont téléchargées dans les systèmes de l’entreprise au moins tous les 29 jours, sinon l’outil SIEM est utilisé. |
| **Gouvernance** | Le cycle de vie des affectations d’utilisateurs à la méthode d’authentification appropriée et aux groupes associés est défini et supervisé. |
| **Sécurité** | L’accès à la méthode d’authentification appropriée est contrôlé par le biais des affectations d’utilisateurs et de groupes. <br> Seuls les utilisateurs autorisés peuvent utiliser la connexion sans mot de passe. |
| **Performances** | Les chronologies de propagation des affectations d’accès sont documentées et surveillées. <br> Les durées de connexion sont mesurées pour améliorer l’utilisation. |
| **Expérience utilisateur** | Les utilisateurs connaissent la compatibilité des appareils mobiles. <br> Les utilisateurs peuvent configurer la connexion sans mot de passe de l’application Authenticator. |
| **Support** | Les utilisateurs savent comment trouver une assistance pour les problèmes de connexion sans mot de passe. |

### <a name="engage-the-right-stakeholders"></a>Impliquer les parties prenantes appropriées

Les échecs de projets informatiques, lorsqu’ils se produisent, proviennent généralement d’une disparité entre les attentes et l’impact, les responsabilités et les résultats. Pour éviter un tel cas de figure, [veillez à faire appel aux bonnes personnes](../fundamentals/active-directory-deployment-plans.md#include-the-right-stakeholders) et à ce que les rôles des parties prenantes soient bien compris.

### <a name="plan-communications"></a>Planifier les communications

La communication est essentielle à la réussite de tout nouveau service. Communiquez préalablement sur la façon dont l’expérience des utilisateurs va changer, à quel moment cela va se faire et comment il est possible d’obtenir de l’aide en cas de problème.

Vos communications aux utilisateurs finaux doivent inclure les informations suivantes :

- [Activation de l’expérience d’inscription à la sécurité combinée](howto-authentication-passwordless-phone.md)
- [Téléchargement de l’application Microsoft Authenticator](../user-help/user-help-auth-app-download-install.md)
- [Inscription dans l’application Microsoft Authenticator](howto-authentication-passwordless-phone.md)
- [Connexion avec votre téléphone](../user-help/user-help-auth-app-sign-in.md)

Microsoft fournit des [modèles de communication](https://aka.ms/mfatemplates) Multi-Factor Authentication, des [modèles de communication](https://www.microsoft.com/download/details.aspx?id=56768) pour la réinitialisation de mot de passe en libre-service (SSPR) et une [documentation destinée aux utilisateurs finaux](../user-help/security-info-setup-signin.md), afin de vous aider à rédiger vos messages. Vous pouvez diriger les utilisateurs vers [https://myprofile.microsoft.com](https://myprofile.microsoft.com/) pour qu’ils s’inscrivent directement en sélectionnant les liens **Informations de sécurité** sur cette page.

### <a name="plan-to-pilot"></a>Prévoir le pilote

Lorsque vous déployez l’authentification sans mot de passe, vous devez d’abord activer un ou plusieurs groupes pilotes. Vous pouvez [créer des groupes](../fundamentals/active-directory-groups-create-azure-portal.md) spécialement à cet effet. Ajoutez les utilisateurs pilotes aux groupes. Activez ensuite de nouvelles méthodes d’authentification sans mot de passe pour les groupes sélectionnés.

Les groupes peuvent être synchronisés à partir d’un annuaire local, ou à partir d’Azure AD. À partir du moment où vous êtes satisfait des résultats de votre pilote, vous pouvez activer l’authentification sans mot de passe pour tous les utilisateurs.

Consultez les [Bonnes pratiques pour un pilote](../fundamentals/active-directory-deployment-plans.md) dans la page des plans de déploiement.

## <a name="plan-passwordless-authentication-with-the-microsoft-authenticator-app"></a>Planifier l’authentification sans mot de passe avec l’application Microsoft Authenticator

L’application Microsoft Authenticator est téléchargeable gratuitement à partir de Google Play ou d’Apple App Store. [En savoir plus sur le téléchargement de l’application Microsoft Authenticator](https://www.microsoft.com/p/microsoft-authenticator/9nblgggzmcj6). Demandez aux utilisateurs de télécharger l’application Microsoft Authenticator et de suivre les instructions pour activer la connexion par téléphone. 

L’application transforme n’importe quel téléphone iOS ou Android en informations d’identification fortes et sans mot de passe. Les utilisateurs se connectent à n’importe quelle plateforme ou n’importe quel navigateur en obtenant une notification sur leur téléphone, en faisant correspondre un nombre affiché à l’écran à celui affiché sur leur téléphone, puis en utilisant les données biométriques ou un code confidentiel pour confirmer. [Consultez les détails sur le fonctionnement de l’application Microsoft Authenticator](./concept-authentication-passwordless.md#microsoft-authenticator-app).

![se connecter avec l’application Authenticator](./media/howto-authentication-passwordless-deployment/passwordless-dp-sign-in.png)

### <a name="technical-considerations-for-the-microsoft-authenticator-app"></a>Considérations techniques relatives à l’application Microsoft Authenticator

**Intégration AD FS** - Lorsqu’un utilisateur active l’information d’identification sans mot de passe Microsoft Authenticator, l’authentification de cet utilisateur implique par défaut l’envoi d’une notification pour approbation. Les utilisateurs d’un locataire hybride ne peuvent pas être dirigés vers ADFS pour se connecter, sauf s’ils sélectionnent « Utiliser votre mot de passe à la place ». Ce processus contourne également les stratégies d’accès conditionnel locales, ainsi que les flux d’authentification directe. Toutefois, si un paramètre *login_hint* est spécifié, l’utilisateur est automatiquement transféré vers AD FS et contourne la possibilité d’utiliser l’information d’identification sans mot de passe.

**Serveur Azure AD Multi-Factor Authentication** - Les utilisateurs finaux activés pour une authentification multifacteur via le serveur Azure MFA local d'une organisation peuvent créer et utiliser une information d'identification unique de connexion par téléphone sans mot de passe. Si l’utilisateur tente de mettre à niveau plusieurs installations (supérieures à 5) de Microsoft Authenticator avec cette information d’identification, cette modification peut générer une erreur.

**Inscription de l’appareil** - Pour utiliser l’application Authenticator dans l’authentification sans mot de passe, l’appareil doit être inscrit dans le locataire Azure AD et ne pas être un appareil partagé. Un appareil ne peut être inscrit que dans un seul locataire. Cette limite signifie que seulement un compte professionnel ou scolaire est pris en charge pour la connexion par téléphone avec l’application Authenticator.

## <a name="plan-passwordless-authentication-with-fido2-security-keys"></a>Planifier une authentification sans mot de passe avec des clés de sécurité FIDO2
Il existe trois types de déploiements de connexion sans mot de passe disponibles avec des clés de sécurité :

-    Applications web Azure Active Directory sur un navigateur pris en charge
-    Appareils Windows 10 joints à Azure Active Directory
-    Appareils Windows 10 joints à Azure Active Directory hybride (préversion)
     -    Donne accès aux ressources informatiques et locales. Pour plus d’informations sur l’accès aux ressources locales, consultez [Authentification unique auprès de ressources locales à l’aide de clés FIDO2](./howto-authentication-passwordless-security-key-on-premises.md).

Vous devez activer le paramètre **Clés de sécurité FIDO2 compatibles**. Microsoft a annoncé des [partenariats clés avec des fournisseurs de clés FIDO2](https://techcommunity.microsoft.com/t5/Azure-Active-Directory-Identity/Microsoft-passwordless-partnership-leads-to-innovation-and-great/ba-p/566493).

**Pour les applications web Azure AD et les appareils Windows joints à Azure AD** :

-    Windows 10 version 1809 ou ultérieure utilisant un navigateur pris en charge, comme Microsoft Edge ou Mozilla Firefox (version 67 ou ultérieure). 
-    Windows 10 version 1809 prend en charge la connexion FIDO2 et peut nécessiter le déploiement d’un logiciel du fabricant de la clé FIDO2. Nous vous recommandons d’utiliser la version 1903 ou une version ultérieure. 

**Pour les appareils joints à un domaine Azure Active Directory hybride** : 
-    Windows 10 version 2004 ou ultérieure
-    Serveurs de domaine entièrement corrigés exécutant Windows Server 2016 ou 2019.
-    Version la plus récente d’Azure AD Connect

Pour obtenir la liste complète des conditions requises, consultez [Activer la connexion par clé de sécurité sans mot de passe à des appareils Windows 10 à l’aide d’Azure Active Directory](./howto-authentication-passwordless-security-key-windows.md#requirements).


### <a name="security-key-life-cycle"></a>Cycle de vie des clés de sécurité

Les clés de sécurité permettent l’accès à vos ressources, et vous devez prévoir la gestion de ces appareils physiques.

1. **Distribution de clés** : prévoyez la façon de provisionner les clés dans votre organisation. Vous pouvez opter pour un processus de provisionnement centralisé, ou autoriser les utilisateurs finals à acheter des clés compatibles FIDO 2.0.
1. **Activation de clés** : les utilisateurs finals doivent activer eux-mêmes la clé de sécurité. Ils inscrivent leurs clés de sécurité sur [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo) et activent le deuxième facteur (code confidentiel ou biométrie) à la première utilisation.
1. **Désactivation d’une clé** : la clé de sécurité étant une fonctionnalité en phase de préversion, un administrateur ne dispose d’aucun moyen pour supprimer une clé à partir d’un compte d’utilisateur. L’utilisateur doit la supprimer. Si une clé est perdue ou volée :
   1. Supprimez l’utilisateur de tout groupe activé pour l’authentification sans mot de passe.
   1. Vérifiez que la clé est supprimée en tant que méthode d’authentification.
   1. Émettez une nouvelle clé. **Remplacement de clés** : les utilisateurs peuvent activer deux clés de sécurité en même temps. Lors du remplacement d’une clé de sécurité, assurez-vous que l’utilisateur a également supprimé la clé qui est remplacée.

### <a name="enable-windows-10-support"></a>Activer la prise en charge de Windows 10

L’activation de la connexion Windows 10 au moyen de clés de sécurité FIDO2 nécessite l’activation de la fonctionnalité de fournisseur d’informations d’identification dans Windows 10. Choisissez l’un des éléments suivants :

- [Activer le fournisseur d’informations d’identification avec Intune](howto-authentication-passwordless-security-key-windows.md#enable-with-intune)
   - Le déploiement Intune est l’option recommandée.
- [Activer le fournisseur d’informations d’identification avec un package d’approvisionnement](howto-authentication-passwordless-security-key-windows.md#enable-with-a-provisioning-package)
   - Si le déploiement Intune n’est pas possible, les administrateurs doivent déployer un package sur chaque machine pour activer la fonctionnalité de fournisseur d’informations d’identification. L’installation du package peut être effectuée à l’aide de l’une des options suivantes :
      - Stratégie de groupe ou Configuration Manager
      - Installation locale sur une machine Windows 10
- [Activer le fournisseur d’informations d’identification avec une stratégie de groupe](howto-authentication-passwordless-security-key-windows.md#enable-with-group-policy)
   - Uniquement prise en charge pour les appareils joints à Azure AD Hybride.

#### <a name="enable-on-premises-integration"></a>Activer l’intégration locale

Pour permettre l'accès aux ressources locales, suivez les étapes de la section [Activer la connexion avec clé de sécurité sans mot de passe à des ressources locales (préversion)](howto-authentication-passwordless-security-key-on-premises.md).

> [!IMPORTANT]
> Ces étapes doivent également être effectuées pour que tout appareil Azure AD Hybride joint utilise les clés de sécurité FIDO2 pour l'ouverture de session Windows 10.

### <a name="register-security-keys"></a>Inscrire des clés de sécurité

Les utilisateurs doivent inscrire leur clé de sécurité sur chacune de leurs machines Windows 10 jointes à Azure Active Directory.

Pour plus d’informations, consultez [Inscription des utilisateurs et gestion des clés de sécurité FIDO2](howto-authentication-passwordless-security-key.md#user-registration-and-management-of-fido2-security-keys).


## <a name="plan-auditing-security-and-testing"></a>Planifier l’audit, la sécurité et les tests
La planification de l’audit qui répond à vos infrastructures organisationnelles et de conformité est un élément essentiel de votre déploiement.

### <a name="auditing-passwordless"></a>Audit sans mot de passe

Azure AD propose des rapports fournissant des insights techniques et commerciaux. Demandez à vos propriétaires d’applications métier et techniques de s’attribuer la propriété de ces rapports et de les consommer en fonction des exigences de votre organisation.

La section Méthodes d’**authentification** dans le portail Azure Active Directory permet aux administrateurs d’activer et de gérer les paramètres des informations d’identification sans mot de passe.

Azure AD ajoute des entrées aux journaux d’audit dans les cas suivants :

- Un administrateur apporte des modifications dans la section Méthodes d’authentification.
- Un utilisateur apporte tout type de modification à ses informations d’identification dans Azure Active Directory.

Le tableau ci-dessous fournit quelques exemples de scénarios de création de rapports typiques :

|   | Gérer le risque. | Augmenter la productivité | Gouvernance et conformité |
| --- | --- | --- | --- |
| **Types de rapports** | Méthodes d’authentification - utilisateurs enregistrés pour l’inscription à la sécurité combinée | Méthodes d’authentification - utilisateurs inscrits pour la notification d’application | Connexions : vérifier qui accède au locataire et comment |
| **Actions potentielles** | Utilisateurs cibles pas encore inscrits | Favoriser l’adoption de l’application Microsoft Authenticator ou des clés de sécurité | Révoquer l’accès ou appliquer des stratégies de sécurité supplémentaires pour les administrateurs |

**Azure AD conserve la plupart des données d’audit pendant 30 jours** et rend les données disponibles via l’API ou le portail d’administration Azure afin que vous puissiez les télécharger dans vos systèmes d’analyse. Si vous avez besoin d’une rétention plus longue, exportez et consommez les journaux dans un outil SIEM comme [Azure Sentinel](../../sentinel/connect-azure-active-directory.md), Splunk ou Sumo Logic. [Découvrez-en plus sur l’affichage de vos rapports d’accès et d’utilisation](../reports-monitoring/overview-reports.md).

Les utilisateurs peuvent inscrire et gérer leurs informations d’identification en accédant à [https://aka.ms/mysecurityinfo](https://aka.ms/mysecurityinfo). Ce lien dirige les utilisateurs vers l’expérience de gestion des informations d’identification de l’utilisateur final, qui a été activée par le biais de l’expérience d’inscription combinée SSPR/MFA. Azure AD enregistre l’inscription des appareils de sécurité FIDO2 et les modifications apportées aux méthodes d’authentification par les utilisateurs.

### <a name="plan-security"></a>Planifier la sécurité 
Dans le cadre de ce plan de lancement, Microsoft recommande l’activation de l’authentification sans mot de passe pour tous les comptes administrateurs privilégiés.

Lorsque les utilisateurs activent ou désactivent le compte sur une clé de sécurité, ou qu’ils réinitialisent le second facteur pour la clé de sécurité sur leurs machines Windows 10, une entrée est ajoutée au journal de sécurité, sous les ID d’événement suivants : *4670* et *5382*.

### <a name="plan-testing"></a>Planifier les tests

À chaque étape de votre déploiement, lorsque vous testez les scénarios et l'adoption, assurez-vous que les résultats sont conformes à vos attentes.

#### <a name="testing-the-microsoft-authenticator-app"></a>Test de l’application Microsoft Authenticator

Voici des exemples de cas de test pour l’authentification sans mot de passe, avec l’application Microsoft Authenticator :

| Scénario | Résultats attendus |
| --- | --- |
| L’utilisateur peut inscrire l’application Microsoft Authenticator | L’utilisateur peut inscrire l’application à partir de aka.ms/mysecurityinfo |
| L’utilisateur peut activer la connexion par téléphone | Connexion par téléphone configurée pour le compte professionnel |
| L’utilisateur peut accéder à une application avec une connexion par téléphone | L’utilisateur procède au processus de connexion par téléphone et accède à l’application. |
| Test de restauration de l’inscription de la connexion par téléphone en désactivant la connexion sans mot de passe Microsoft Authenticator dans l’écran Méthodes d’authentification du portail Azure Active Directory | Les utilisateurs autorisés précédemment ne peuvent pas utiliser la connexion sans mot de passe à partir de Microsoft Authenticator. |
| Suppression de la connexion par téléphone depuis l’application Microsoft Authenticator | Le compte professionnel n’est plus disponible sur Microsoft Authenticator |

#### <a name="testing-security-keys"></a>Test des clés de sécurité

Voici des exemples de cas de test pour l’authentification sans mot de passe, avec des clés de sécurité.

**Connexion FIDO sans mot de passe pour des appareils Windows 10 joints à Azure Active Directory**

| Scénario | Résultats attendus |
| --- | --- |
| L’utilisateur peut inscrire l’appareil FIDO2 (1809) | L’utilisateur peut inscrire l’appareil FIDO2 en passant par Paramètres > Comptes > Options de connexion > Clé de sécurité |
| L’utilisateur peut réinitialiser l’appareil FIDO2 (1809) | L’utilisateur peut réinitialiser l’appareil FIDO2 à l’aide du logiciel du fabricant |
| L’utilisateur peut se connecter avec l’appareil FIDO2 (1809) | L’utilisateur peut sélectionner Clé de sécurité dans la fenêtre de connexion et se connecter avec succès. |
| L’utilisateur peut inscrire l’appareil FIDO2 (1903) | L’utilisateur peut inscrire l’appareil FIDO2 en passant par Paramètres > Comptes > Options de connexion > Clé de sécurité |
| L’utilisateur peut réinitialiser l’appareil FIDO2 (1903) | L’utilisateur peut réinitialiser l’appareil FIDO2 en passant par Paramètres > Comptes > Options de connexion > Clé de sécurité |
| L’utilisateur peut se connecter avec l’appareil FIDO2 (1903) | L’utilisateur peut sélectionner Clé de sécurité dans la fenêtre de connexion et se connecter avec succès. |

**Connexion FIDO sans mot de passe pour des applications web Azure AD**

| Scénario | Résultats attendus |
| --- | --- |
| L’utilisateur peut inscrire l’appareil FIDO2 sur aka.ms/mysecurityinfo par l’intermédiaire de Microsoft Edge | L’inscription doit s’être déroulée correctement |
| L’utilisateur peut inscrire l’appareil FIDO2 sur aka.ms/mysecurityinfo par l’intermédiaire de Firefox | L’inscription doit s’être déroulée correctement |
| L’utilisateur peut se connecter à OneDrive en ligne avec l’appareil FIDO2 par l’intermédiaire de Microsoft Edge | La connexion doit s’être déroulée correctement |
| L’utilisateur peut se connecter à OneDrive en ligne avec l’appareil FIDO2 par l’intermédiaire de Firefox | La connexion doit s’être déroulée correctement |
| Test de restauration de l’inscription de l’appareil FIDO2 en désactivant les clés de sécurité FIDO2 dans la fenêtre Méthodes d’authentification du portail Azure Active Directory | Les utilisateurs seront invités à se connecter à l’aide de leur clé de sécurité. Les utilisateurs parviennent à se connecter et une erreur s’affiche : « Votre stratégie d’entreprise vous demande d’utiliser une autre méthode pour vous connecter ». Les utilisateurs doivent alors pouvoir sélectionner une autre méthode pour se connecter correctement. Fermez la fenêtre et connectez-vous de nouveau pour vérifier qu’ils n’affichent pas le même message d’erreur. |

### <a name="plan-for-rollback"></a>Plan de restauration

Bien que l’authentification sans mot de passe soit une fonctionnalité légère, avec un impact minimal sur les utilisateurs finals, il peut s’avérer nécessaire d’effectuer une restauration.

La restauration implique que l’administrateur se connecte au portail Azure Active Directory, qu’il sélectionne les méthodes d’authentification fortes souhaitées et remplace la valeur de l’option d’activation par **Non**. Ce processus désactive la fonctionnalité sans mot de passe pour tous les utilisateurs.

Les utilisateurs qui ont déjà enregistré des appareils de sécurité FIDO2 sont alors invités à utiliser l’appareil de sécurité à leur prochaine connexion tandis que l’erreur suivante s’affiche :

![choisir un autre mode de connexion](./media/howto-authentication-passwordless-deployment/passwordless-choose-sign-in.png)

## <a name="deploy-and-troubleshoot-passwordless-authentication"></a>Déployer l’authentification sans mot de passe et détecter un problème

Suivez ci-dessous les étapes correspondant à la méthode de votre choix.

### <a name="required-administrative-roles"></a>Rôles d’administration nécessaires

| Rôle Azure AD | Description |
| --- | --- |
| Administrateur général|Rôle doté des privilèges minimums pouvant implémenter une expérience d’inscription combinée. |
| Administrateur d’authentification | Rôle avec le moins de privilèges possible pour implémenter et gérer les méthodes d’authentification. |
| Utilisateur | Rôle avec le moins de privilèges possible pour configurer l’application Authenticator sur l’appareil, ou pour inscrire un appareil clé de sécurité destiné à la connexion au web ou à Windows 10. |

### <a name="deploy-phone-sign-in-with-the-microsoft-authenticator-app"></a>Déployer la connexion par téléphone avec l’application Microsoft Authenticator

Suivez les étapes décrites dans l’article [Activer la connexion sans mot de passe avec l’application Microsoft Authenticator](howto-authentication-passwordless-phone.md) pour activer l’application Microsoft Authenticator en tant que méthode d’authentification sans mot de passe dans votre organisation.

### <a name="deploy-fido2-security-key-sign-in"></a>Déployer la connexion par clé de sécurité FIDO2

Suivez les étapes de l’article [Activer la connexion par clé de sécurité sans mot de passe pour Azure AD](howto-authentication-passwordless-security-key.md) afin d’activer les clés de sécurité FIDO2 comme méthodes d’authentification sans mot de passe.

### <a name="troubleshoot-phone-sign-in"></a>Résoudre les problèmes de la connexion par téléphone

| Scénario | Solution |
| --- | --- |
| L’utilisateur ne peut pas effectuer d’inscription combinée. | Vérifiez que [l’inscription combinée](concept-registration-mfa-sspr-combined.md) est activée. |
| L’utilisateur ne peut pas activer la connexion par téléphone avec l’application Authenticator. | Assurez-vous que l’utilisateur se trouve dans l’étendue du déploiement. |
| L’utilisateur n’est PAS dans l’étendue de l’authentification sans mot de passe, mais l’option de connexion sans mot de passe lui est proposée, sans qu’il puisse parvenir à l’utiliser. | Ce scénario se produit lorsque l’utilisateur a activé la connexion par téléphone dans l’application avant que la stratégie soit créée. <br> *Pour activer la connexion* : ajoutez l’utilisateur à l’étendue des utilisateurs autorisés à se connecter sans mot de passe. <br> *Pour bloquer la connexion* : demandez à l’utilisateur de supprimer ses informations d’identification de l’application. |

### <a name="troubleshoot-security-key-sign-in"></a>Résoudre les problèmes de connexion par clé de sécurité

| Scénario | Solution |
| --- | --- |
| L’utilisateur ne peut pas effectuer d’inscription combinée. | Vérifiez que [l’inscription combinée](concept-registration-mfa-sspr-combined.md) est activée. |
| L’utilisateur ne peut pas ajouter de clé de sécurité dans ses [paramètres de sécurité](https://aka.ms/mysecurityinfo). | Vérifiez que les [clés de sécurité](howto-authentication-passwordless-security-key.md) sont activées. |
| L’utilisateur ne peut pas ajouter de clé de sécurité dans les options de connexion Windows 10. | [Vérifiez que les clés de sécurité sont prévues pour la connexion Windows](./concept-authentication-passwordless.md) |
| **Message d’erreur** : Nous avons détecté que ce navigateur ou ce système d’exploitation ne prend pas en charge les clés de sécurité FIDO2. | Les appareils de sécurité FIDO2 sans mot de passe ne peuvent être inscrits que dans les navigateurs pris en charge (Microsoft Edge, Firefox version 67) sur Windows 10 version 1809 ou ultérieure. |
| **Message d’erreur** : La stratégie de votre entreprise vous demande d’utiliser une autre méthode pour vous connecter. | Vérifiez que les clés de sécurité sont activées dans le locataire. |
| L’utilisateur ne peut pas gérer ma clé de sécurité sur Windows 10 version 1809 | La version 1809 vous demande d’utiliser le logiciel de gestion de clés de sécurité fourni par le fournisseur de clés FIDO2. Contactez le fournisseur pour obtenir de l’aide. |
| Je pense que ma clé de sécurité FIDO2 peut être défectueuse, comment la tester. | Accédez à [https://webauthntest.azurewebsites.net/](https://webauthntest.azurewebsites.net/), entrez les informations d’identification d’un compte de test, connectez la clé de sécurité suspecte, sélectionnez le bouton **+** en haut à droite de l’écran, puis cliquez sur créer et procédez à la création. Si ce scénario échoue, votre appareil est peut-être défectueux. |

## <a name="next-steps"></a>Étapes suivantes

- [Activer la connexion par clé de sécurité sans mot de passe pour Azure AD](howto-authentication-passwordless-security-key.md)
- [Activer la connexion sans mot de passe avec l’application Microsoft Authenticator](howto-authentication-passwordless-phone.md)
- [En savoir plus sur les insights et l’utilisation de méthodes d’authentification](howto-authentication-methods-usage-insights.md)
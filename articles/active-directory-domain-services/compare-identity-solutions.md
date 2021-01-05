---
title: Comparer les services Active Directory dans Azure | Microsoft Docs
description: Dans cette vue d’ensemble, vous comparez les différentes offres d’identité pour Active Directory Domain Services, Azure Active Directory et Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: overview
ms.date: 06/08/2020
ms.author: justinha
ms.openlocfilehash: 479cc036ed3231d970d46eef9d89daa39a0b0876
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620186"
---
# <a name="compare-self-managed-active-directory-domain-services-azure-active-directory-and-managed-azure-active-directory-domain-services"></a>Comparer les services Active Directory Domain Services automanagés, Azure Active Directory et les services Azure Active Directory Domain Services managés

Pour fournir aux applications, aux services ou aux appareils un accès à une identité centrale, il existe trois façons courantes d’utiliser des services Active Directory dans Azure. Ce choix de solutions d’identité vous offre la possibilité d’utiliser le répertoire le plus approprié pour les besoins de votre organisation. Par exemple, si vous gérez principalement des utilisateurs uniquement dans le cloud qui exécutent des appareils mobiles, il n’est pas judicieux de créer et d’exécuter votre propre solution d’identité Active Directory Domain Services (AD DS). Au lieu de cela, vous pouvez simplement utiliser Azure Active Directory.

Même si les trois solutions d’identité basées sur Active Directory ont un nom et une technologie en commun, elles sont conçues pour fournir des services qui répondent à différentes demandes des clients. Au niveau supérieur, ces solutions d’identité et ensembles de fonctionnalités sont les suivants :

* **Active Directory Domain Services (AD DS)** : serveur LDAP (Lightweight Directory Access Protocol) prêt pour l’entreprise qui fournit des fonctionnalités clés telles que l’identité et l’authentification, la gestion des objets ordinateur, la stratégie de groupe et les approbations.
    * AD DS est un composant central dans de nombreuses organisations disposant d’un environnement informatique local et fournit des fonctionnalités d’authentification de compte d’utilisateur et de gestion d’ordinateurs de base.
    * Pour plus d’informations, consultez [Vue d’ensemble d’Active Directory Domain Services dans la documentation de Windows Server][overview-adds].
* **Azure Active Directory (Azure AD)**  : gestion des identités et des appareils mobiles basée sur le cloud qui fournit des services d’authentification et de compte d’utilisateur pour les ressources telles que Microsoft 365, le portail Azure ou les applications SaaS.
    * Azure AD peut être synchronisé avec un environnement AD DS local pour fournir une identité unique aux utilisateurs qui travaillent en mode natif dans le cloud.
    * Pour plus d’informations sur Azure AD, consultez [Qu’est-ce qu’Azure Active Directory ?][whatis-azuread]
* **Azure Active Directory Domain Services (Azure AD DS)**  : fournit des services de domaine managés avec un sous-ensemble de fonctionnalités AD DS traditionnelles entièrement compatibles, comme la jonction de domaine, la stratégie de groupe, le protocole LDAP et l’authentification Kerberos/NTLM.
    * Azure AD DS s’intègre à Azure AD, qui peut lui-même se synchroniser avec un environnement AD DS local. Cette capacité étend les cas d’usage de l’identité centrale aux applications web traditionnelles qui s’exécutent dans Azure dans le cadre d’une stratégie lift-and-shift.
    * Pour en savoir plus sur la synchronisation avec Azure AD et en local, consultez [Synchronisation des objets et des informations d’identification dans un domaine managé][synchronization].

Cet article de vue d’ensemble compare la façon dont ces solutions d’identité peuvent fonctionner ensemble ou être utilisées indépendamment, en fonction des besoins de votre organisation.

> [!div class="nextstepaction"]
> [Pour commencer, créez un domaine managé Azure AD DS à l’aide du portail Azure][tutorial-create].

## <a name="azure-ad-ds-and-self-managed-ad-ds"></a>Azure AD DS et instance AD DS automanagée

Si vous avez des applications et des services qui doivent accéder à des mécanismes d’authentification traditionnels tels que Kerberos ou NTLM, il existe deux façons de fournir Active Directory Domain Services dans le cloud :

* Un domaine *managé* que vous créez avec Azure Active Directory Domain Services (Azure AD DS). Microsoft crée et gère les ressources requises.
* Un domaine *automanagé* que vous créez et configurez à l’aide de ressources traditionnelles telles que les machines virtuelles, le système d’exploitation invité Windows Server et Active Directory Domain Services (AD DS). Vous continuez ensuite à administrer ces ressources.

Avec Azure AD DS, les composants de service de base sont déployés et gérés pour vous par Microsoft en tant qu’expérience de domaine *managé*. Vous ne pouvez pas déployer, gérer, corriger et sécuriser l’infrastructure AD DS pour des composants tels que les machines virtuelles, le système d’exploitation Windows Server ou les contrôleurs de domaine.

Azure AD DS fournit un sous-ensemble plus petit de fonctionnalités à un environnement AD DS automanagé traditionnel, ce qui réduit la complexité de la conception et de la gestion. Par exemple, il n’existe pas de forêts, de domaines, de sites et de liens de réplication Active Directory à concevoir ni à entretenir. Vous pouvez toujours [créer des approbations de forêt entre des environnements Azure AD DS et locaux][create-forest-trust].

Pour les applications et les services qui s’exécutent dans le cloud et qui ont besoin d’accéder à des mécanismes d’authentification traditionnels, tels que Kerberos ou NTLM, Azure AD DS offre une expérience de domaine managé avec charge administrative minimale. Pour plus d’informations, consultez [Concepts de gestion pour les comptes d’utilisateur, les mots de passe et l’administration dans Azure AD DS][administration-concepts].

Lorsque vous déployez et exécutez un environnement AD DS automanagé, vous devez gérer tous les composants d’infrastructure et de répertoire associés. Il existe une surcharge de maintenance supplémentaire avec un environnement AD DS automanagé, mais vous pouvez ensuite effectuer des tâches supplémentaires, telles que l’extension du schéma ou la création d’approbations de forêt.

Les modèles de déploiement courants pour un environnement AD DS automanagé qui fournit l’identité aux applications et services dans le cloud sont les suivants :

* **Instance AD DS autonome et cloud uniquement** : les machines virtuelles Azure sont configurées en tant que contrôleurs de domaine et un environnement distinct AD DS cloud uniquement est créé. Cet environnement AD DS ne s’intègre pas à un environnement AD DS local. Un ensemble différent d’informations d’identification est utilisé pour la connexion et l’administration des machines virtuelles dans le cloud.
* **Déploiement de forêts de ressources** : les machines virtuelles Azure sont configurées en tant que contrôleurs de domaine et un domaine AD DS qui fait partie d’une forêt existante est créé. Une relation d’approbation est ensuite configurée dans un environnement AD DS local. Vous pouvez joindre au domaine d’autres machines virtuelles Azure dans cette forêt de ressources dans le cloud. L’authentification de l’utilisateur a lieu sur une connexion VPN/ExpressRoute à votre environnement AD DS local.
* **Étendre un domaine local à Azure** : un réseau virtuel Azure se connecte à un réseau local à l’aide d’une connexion VPN/ExpressRoute. Les machines virtuelles Azure se connectent à ce réseau virtuel Azure, ce qui leur permet de joindre un domaine à l’environnement AD DS local.
    * Une alternative consiste à créer des machines virtuelles Azure et à les promouvoir en tant que contrôleurs de domaine de réplication à partir du domaine AD DS local. Ces contrôleurs de domaine répliquent via une connexion VPN/ExpressRoute vers l’environnement local AD DS. Le domaine AD DS local est efficacement étendu dans Azure.

Le tableau suivant présente certaines des fonctionnalités dont vous pouvez avoir besoin pour votre organisation, ainsi que les différences entre un domaine managé Azure AD DS et un domaine automanagé AD DS :

| **Fonctionnalité** | **Azure AD DS** | **AD DS automanagé** |
| ----------- |:---------------:|:----------------------:|
| **Service managé**                               | **&#x2713;** | **&#x2715;** |
| **Déploiements sécurisés**                            | **&#x2713;** | L’administrateur sécurisee le déploiement. |
| **Serveur DNS**                                    | **&#x2713;** (service géré) |**&#x2713;** |
| **Privilèges d’administrateur d’entreprise ou de domaine** | **&#x2715;** | **&#x2713;** |
| **jonction de domaine**                                   | **&#x2713;** | **&#x2713;** |
| **Authentification de domaine à l’aide de NTLM et Kerberos** | **&#x2713;** | **&#x2713;** |
| **Délégation Kerberos contrainte**               | Basé sur des ressources | Basée sur des ressources et basé sur des comptes|
| **Structure d’unité d’organisation personnalisée**                           | **&#x2713;** | **&#x2713;** |
| **Stratégie de groupe**                                  | **&#x2713;** | **&#x2713;** |
| **Extensions de schéma**                             | **&#x2715;** | **&#x2713;** |
| **Approbations de domaine/forêt AD**                     | **&#x2713;** (approbations de forêt sortantes unidirectionnelles uniquement) | **&#x2713;** |
| **LDAP sécurisé (LDAPS)**                           | **&#x2713;** | **&#x2713;** |
| **Lecture LDAP**                                     | **&#x2713;** | **&#x2713;** |
| **Écriture LDAP**                                    | **&#x2713;** (dans le domaine managé) | **&#x2713;** |
| **Déploiements géolocalisés**                   | **&#x2715;** | **&#x2713;** |

## <a name="azure-ad-ds-and-azure-ad"></a>Azure AD DS et Azure AD

Azure AD vous permet de gérer l’identité des appareils utilisés par l’organisation et de contrôler l’accès aux ressources de l’entreprise depuis ces appareils. Les utilisateurs peuvent aussi inscrire leur appareil personnel (modèle BYO, bring-your-own) sur Azure AD, ce qui fournit une identité à l’appareil. Par la suite, Azure AD authentifie l’appareil lorsqu’un utilisateur se connecte à Azure AD et utilise cet appareil pour accéder aux ressources sécurisées. Vous pouvez gérer l’appareil à l’aide de logiciels de gestion des appareils mobiles (MDM), tels que Microsoft Intune. Cette fonctionnalité de gestion vous permet de restreindre l’accès aux ressources sensibles à partir d’appareils gérés et conformes aux stratégies.

Les ordinateurs traditionnels et les ordinateurs portables peuvent également être joints à Azure AD. Ce mécanisme offre les mêmes avantages que l’inscription d’un appareil personnel à Azure AD, par exemple pour permettre aux utilisateurs de se connecter à l’appareil à l’aide de leurs informations d’identification d’entreprise.

Les appareils joints à Azure AD vous offrent les avantages suivants :

* Authentification unique (SSO) pour les applications sécurisées par Azure AD
* Itinérance compatible avec la stratégie de l’entreprise pour les paramètres utilisateur sur les appareils.
* Accès au Windows Store pour Entreprises avec vos informations d’identification professionnelles.
* Windows Hello Entreprise.
* Accès restreint aux applications et aux ressources sur les appareils conformes à la stratégie d’entreprise.

Les appareils peuvent être joints à Azure AD avec ou sans déploiement hybride qui inclut un environnement AD DS local. Le tableau suivant présente les modèles de propriété d’appareil courants et la façon dont ils sont généralement joints à un domaine :

| **Type d’appareil**                                        | **Plateformes d’appareils**             | **Mécanisme**          |
|:----------------------------------------------------------| -------------------------------- | ---------------------- |
| Appareils personnels                                          | Windows 10, iOS, Android, macOS | Appareils inscrits sur Azure AD    |
| Appareil appartenant à l’organisation et non joint à AD DS local | Windows 10                       | Appareil joints Azure AD        |
| Appareil appartenant à l’organisation et joint à AD DS local  | Windows 10                       | Appareils joints Azure AD hybrides |

Sur un appareil inscrit ou joint à Azure AD, l’authentification de l’utilisateur s’effectue par le biais des protocoles modernes OAuth/OpenID Connect. Ces protocoles sont conçus pour fonctionner sur Internet et sont très utiles dans les scénarios de mobilité, où les utilisateurs accèdent aux ressources d’entreprise depuis n’importe quel endroit.

Avec les appareils joints à Azure AD DS, les applications peuvent utiliser les protocoles Kerberos et NTLM pour l’authentification. Par conséquent, ils peuvent prendre en charge les applications héritées migrées pour s’exécuter sur des machines virtuelles Azure dans le cadre d’une stratégie lift-and-shift. Le tableau suivant présente les différences entre la façon dont les appareils sont représentés et peuvent s’authentifier auprès du répertoire :

| **Aspect**                      | **Joint à Azure AD**                                 | **Joint à Azure AD DS**                                                    |
|:--------------------------------| --------------------------------------------------- | ------------------------------------------------------------------------- |
| Appareil contrôlé par            | Azure AD                                            | Domaine managé Azure AD DS                                                |
| Représentation dans l’annuaire | Objets appareil dans le répertoire Azure AD            | Objets ordinateur dans le domaine managé Azure AD DS                        |
| Authentification                  | Protocoles OAuth et OpenID Connect              | Protocoles Kerberos et NTLM                                               |
| Gestion                      | Logiciels de gestion des appareils mobiles (GAM) tels qu’Intune | Stratégie de groupe                                                              |
| Mise en réseau                      | Fonctionne sur Internet                             | Doit être connecté ou appairé au réseau virtuel sur lequel le domaine managé est déployé |
| Idéal pour...                    | Appareils mobiles ou de bureau des utilisateurs finaux                  | Machines virtuelles de serveur déployées dans Azure                                              |


Si l’environnement local AD DS et Azure AD sont configurés pour l’authentification fédérée avec ADFS, aucun hachage de mot de passe (actuel/valide) n’est disponible dans Azure DS. Il est possible que les comptes d’utilisateurs Azure AD créés avant l’implémentation de l’authentification fédérée disposent d’un ancien hachage de mot de passe, mais il est peu probable que celui-ci corresponde à un hachage de leur mot de passe local. De ce fait, Azure AD DS ne pourra pas valider les informations d’identification des utilisateurs

## <a name="next-steps"></a>Étapes suivantes

Pour bien démarrer avec l’utilisation d’Azure AD DS, [créez un domaine managé Azure AD DS à l’aide du portail Azure][tutorial-create].

Consultez également les pages [Concepts de gestion pour les comptes d’utilisateur, les mots de passe et l’administration dans Azure AD DS][administration-concepts] et [Synchronisation des objets et des informations d’identification dans un domaine managé][synchronization].

<!-- INTERNAL LINKS -->
[manage-dns]: manage-dns.md
[deploy-kcd]: deploy-kcd.md
[custom-ou]: create-ou.md
[manage-gpos]: manage-group-policy.md
[tutorial-ldaps]: tutorial-configure-ldaps.md
[tutorial-create]: tutorial-create-instance.md
[whatis-azuread]: ../active-directory/fundamentals/active-directory-whatis.md
[overview-adds]: /windows-server/identity/ad-ds/get-started/virtual-dc/active-directory-domain-services-overview
[create-forest-trust]: tutorial-create-forest-trust.md
[administration-concepts]: administration-concepts.md
[synchronization]: synchronization.md

---
title: Résoudre les problèmes de verrouillage de compte dans Azure AD Domain Services | Microsoft Docs
description: Découvrez comment résoudre les problèmes courants de verrouillage de compte dans Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: 7967347fa63c657ba6211328bdd1d55512358521
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96618771"
---
# <a name="troubleshoot-account-lockout-problems-with-an-azure-active-directory-domain-services-managed-domain"></a>Résoudre les problèmes de verrouillage de compte avec un domaine managé Azure Active Directory Domain Services

Pour empêcher toute tentative de connexion malveillante, un domaine géré Azure Active Directory Domain Services (Azure AD DS) verrouille les comptes après un seuil défini. Ce verrouillage de compte peut également se produire par accident, sans tentative de connexion malveillante. Par exemple, si un utilisateur entre plusieurs fois un mot de passe incorrect ou qu’un service tente d’utiliser un ancien mot de passe, le compte est verrouillé.

Cet article explique pourquoi les verrouillages de compte se produisent, comment configurer ce comportement et examiner les audits de sécurité pour remédier aux événements de verrouillage.

## <a name="what-is-an-account-lockout"></a>Qu’est-ce qu’un verrouillage de compte ?

Un compte utilisateur dans un domaine managé Azure AD DS est verrouillé lorsqu’un seuil défini pour les tentatives de connexion infructueuses a été atteint. Ce comportement de verrouillage de compte est conçu pour vous protéger contre les tentatives répétées de connexion par force brute qui pourraient indiquer une attaque numérique automatisée.

**Par défaut, si cinq tentatives de mot de passe incorrectes ont lieu en l’espace de deux minutes, le compte est verrouillé pendant 30 minutes.**

Les seuils de verrouillage de compte par défaut sont configurés à l’aide d’une stratégie de mot de passe affinée. En présence de conditions requises spécifiques, vous pouvez remplacer ces seuils de verrouillage de compte par défaut. Cela étant, il est déconseillé d’augmenter les limites de seuil pour tenter de réduire le nombre de verrouillages de compte. Penchez-vous dans un premier temps sur la source du comportement de verrouillage de compte.

### <a name="fine-grained-password-policy"></a>Stratégie de mot de passe affinée

Les stratégies de mot de passe affinées (SMPA) vous permettent d’appliquer des restrictions spécifiques pour les stratégies de verrouillage de compte et de mot de passe à différents utilisateurs d’un domaine. La SMPA affecte uniquement les utilisateurs au sein d’un domaine managé. Les utilisateurs de cloud et de domaine synchronisés dans le domaine managé à partir d’Azure AD ne sont affectés que par les stratégies de mot de passe définies à l’intérieur du domaine managé. Leurs comptes dans Azure AD ou dans un répertoire local ne sont pas affectés.

Les stratégies sont distribuées par le biais de l’association de groupes dans le domaine managé, et les modifications que vous apportez sont appliquées à la connexion utilisateur suivante. La modification de la stratégie ne déverrouille pas un compte d’utilisateur qui est déjà verrouillé.

Pour plus d’informations sur les stratégies de mot de passe affinées, ainsi que sur les différences entre les utilisateurs créés directement dans Azure AD Directory et synchronisés à partir d’Azure AD, consultez [Configurer des stratégies de mot de passe et de verrouillage de compte][configure-fgpp].

## <a name="common-account-lockout-reasons"></a>Raisons courantes d'un verrouillage de compte

Les raisons les plus courantes expliquant un verrouillage de compte, sans intention ou facteur malveillant, incluent les scénarios suivants :

* **L’utilisateur s’est lui-même verrouillé.**
    * Après une récente modification du mot de passe, l’utilisateur a-t-il continué d'utiliser un précédent mot de passe ? La stratégie de verrouillage de compte par défaut de cinq échecs de connexion en 2 minutes peut être due au fait que l’utilisateur a utilisé par inadvertance un ancien mot de passe.
* **Une application ou un service est associé à un ancien mot de passe.**
    * Si un compte est utilisé par des applications ou services, ces ressources peuvent essayer à plusieurs reprises de se connecter à l’aide d’un ancien mot de passe. Un tel comportement entraîne le verrouillage du compte.
    * Essayez de limiter l’utilisation du compte par plusieurs applications ou services différents, et enregistrez les informations d’identification utilisées. En cas de modification d'un mot de passe de compte, mettez à jour les applications ou services associés en conséquence.
* **Le mot de passe a été modifié dans un environnement différent et le nouveau mot de passe n’a pas encore été synchronisé.**
    * Si un mot de passe de compte est modifié en dehors du domaine managé, par exemple dans un environnement AD DS local, la synchronisation du mot de passe modifié peut prendre plusieurs minutes via Azure AD et dans le domaine managé.
    * Si l'utilisateur tente de se connecter à une ressource dans le domaine managé avant la fin du processus de synchronisation du mot de passe, son compte est verrouillé.

## <a name="troubleshoot-account-lockouts-with-security-audits"></a>Résoudre les problèmes de verrouillage de compte avec les audits de sécurité

Pour résoudre les problèmes liés aux événements de verrouillage de compte et à leur source, [activez les audits de sécurité pour Azure AD DS][security-audit-events]. Les événements d’audit sont uniquement capturés à partir du moment où vous activez cette fonctionnalité. Idéalement, vous devez activer les audits de sécurité *avant* d'être amené à remédier à un problème de verrouillage de compte. Si un compte d’utilisateur se heurte à des problèmes répétés de verrouillage, vous pouvez activer les audits de sécurité pour éviter qu'ils se reproduisent.

Une fois les audits de sécurité activés, les exemples de requêtes suivants vous expliquent comment examiner les *Événements de verrouillage de compte*, code *4740*.

Affichez tous les événements de verrouillage de compte au cours des sept derniers jours :

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= ago(7d)
| where OperationName has "4740"
```

Affichez tous les événements de verrouillage de compte au cours des sept derniers jours pour le compte appelé *driley*.

```Kusto
AADDomainServicesAccountLogon
| where TimeGenerated >= ago(7d)
| where OperationName has "4740"
| where "driley" == tolower(extract("Logon Account:\t(.+[0-9A-Za-z])",1,tostring(ResultDescription)))
```

Affichez tous les événements de verrouillage de compte entre le 26 juin 2020 à 9h00. et le 1er juillet 2020 minuit, triés par ordre croissant de date et heure :

```Kusto
AADDomainServicesAccountManagement
| where TimeGenerated >= datetime(2020-06-26 09:00) and TimeGenerated <= datetime(2020-07-01)
| where OperationName has "4740"
| sort by TimeGenerated asc
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les stratégies de mot de passe affinées afin d'ajuster les seuils de verrouillage de compte, consultez [Configurer des stratégies de mot de passe et de verrouillage de compte][configure-fgpp].

Si vous rencontrez toujours des problèmes pour joindre votre machine virtuelle au domaine managé, [trouvez de l’aide et ouvrez un ticket de support pour Azure Active Directory][azure-ad-support].

<!-- INTERNAL LINKS -->
[configure-fgpp]: password-policy.md
[security-audit-events]: security-audit-events.md
[azure-ad-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md

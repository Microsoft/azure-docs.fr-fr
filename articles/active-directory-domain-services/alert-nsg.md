---
title: Résoudre les alertes de groupe de sécurité réseau dans Azure AD DS | Microsoft Docs
description: Découvrez comment résoudre les alertes liées à la configuration du groupe de sécurité réseau pour Azure Active Directory Domain Services.
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 95f970a7-5867-4108-a87e-471fa0910b8c
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: troubleshooting
ms.date: 07/06/2020
ms.author: justinha
ms.openlocfilehash: d8f2e77b7225306844cec85363a2971eaac4eebd
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96620254"
---
# <a name="known-issues-network-configuration-alerts-in-azure-active-directory-domain-services"></a>Problèmes connus : Alertes de configuration réseau dans Azure Active Directory Domain Services

Pour une bonne communication entre les applications et services et Azure Active Directory Domain Services (Azure AD DS), des ports réseau spécifiques doivent être ouverts afin d'autoriser le trafic. Dans Azure, vous contrôlez le flux du trafic à l’aide de groupes de sécurité réseau. L’état d’intégrité d’un domaine managé Azure AD DS affiche une alerte si les règles de groupe de sécurité réseau requises ne sont pas respectées.

Cet article vous aide à comprendre et à résoudre les alertes courantes découlant de problèmes de configuration des groupes de sécurité réseau.

## <a name="alert-aadds104-network-error"></a>Alerte AADDS104 : Erreur réseau

### <a name="alert-message"></a>Message d’alerte

*Microsoft ne peut pas atteindre les contrôleurs de domaine pour ce domaine géré. Cela peut se produire si un groupe de sécurité réseau (NSG) configuré sur votre réseau virtuel bloque l’accès à un domaine géré. Une autre raison possible est s’il existe un itinéraire défini par l’utilisateur qui bloque le trafic entrant à partir d’Internet.*

Les règles de groupe de sécurité réseau non valides sont la cause la plus courante d’erreurs réseau pour Azure AD DS. Le groupe de sécurité réseau pour le réseau virtuel doit autoriser l’accès à des ports et protocoles spécifiques. Si ces ports sont bloqués, la plateforme Azure ne peut pas surveiller ou mettre à jour le domaine managé. La synchronisation entre le répertoire Azure AD et Azure AD DS est également impactée. Veillez à garder ces ports par défaut ouverts pour éviter les interruptions de service.

## <a name="default-security-rules"></a>Règles de sécurité par défaut

Les règles de sécurité de trafic entrant et sortant par défaut suivantes sont appliquées au groupe de sécurité réseau d'un domaine managé. Ces règles sécurisent Azure AD DS et permettent à la plateforme Azure de surveiller, de gérer et de mettre à jour le domaine managé.

### <a name="inbound-security-rules"></a>Règles de sécurité de trafic entrant

| Priority | Nom | Port | Protocol | Source | Destination | Action |
|----------|------|------|----------|--------|-------------|--------|
| 101      | AllowSyncWithAzureAD | 443 | TCP | AzureActiveDirectoryDomainServices | Quelconque | Allow |
| 201      | AllowRD | 3389 | TCP | CorpNetSaw | Quelconque | Allow |
| 301      | AllowPSRemoting | 5986| TCP | AzureActiveDirectoryDomainServices | Quelconque | Allow |
| 65 000    | AllVnetInBound | Quelconque | Quelconque | VirtualNetwork | VirtualNetwork | Allow |
| 65 001    | AllowAzureLoadBalancerInBound | Quelconque | Quelconque | AzureLoadBalancer | Quelconque | Allow |
| 65 500    | DenyAllInBound | Quelconque | Quelconque | Quelconque | Quelconque | Deny |

> [!NOTE]
> Vous pouvez [configurer LDAP sécurisé][configure-ldaps] pour disposer d'une règle supplémentaire autorisant le trafic entrant. Cette règle supplémentaire est requise pour une communication LDAPS correcte.

### <a name="outbound-security-rules"></a>Règles de sécurité de trafic entrant

| Priority | Nom | Port | Protocol | Source | Destination | Action |
|----------|------|------|----------|--------|-------------|--------|
| 65 000    | AllVnetOutBound | Quelconque | Quelconque | VirtualNetwork | VirtualNetwork | Allow |
| 65 001    | AllowAzureLoadBalancerOutBound | Quelconque | Quelconque |  Quelconque | Internet | Allow |
| 65 500    | DenyAllOutBound | Quelconque | Quelconque | Quelconque | Quelconque | Deny |

>[!NOTE]
> Azure AD DS requiert un accès sortant non restreint depuis le réseau virtuel. Nous vous déconseillons de créer d'autres règles de groupe de sécurité réseau pouvant restreindre l’accès sortant pour le réseau virtuel.

## <a name="verify-and-edit-existing-security-rules"></a>Vérifier et modifier les règles de sécurité existantes

Pour vérifier les règles de sécurité existantes et vous assurer que les ports par défaut sont ouverts, procédez comme suit :

1. Dans le Portail Azure, recherchez et sélectionnez **Groupes de sécurité réseau**.
1. Sélectionnez le groupe de sécurité réseau associé à votre domaine managé, par exemple *AADDS-contoso.com-NSG*.
1. Sur la page **Vue d'ensemble**, les règles de sécurité de trafic entrant et sortant existantes s’affichent.

    Examinez les règles de trafic entrant et sortant, puis comparez-les à la liste des règles requises dans la section précédente. Si nécessaire, sélectionnez et supprimez toutes les règles personnalisées qui bloquent le trafic requis. Si l’une des règles requises est manquante, ajoutez une règle dans la section suivante.

    Après avoir ajouté ou supprimé des règles pour autoriser le trafic requis, l’intégrité du domaine managé se met automatiquement à jour dans les deux heures, et l’alerte est supprimée.

### <a name="add-a-security-rule"></a>Ajouter une règle de sécurité

Pour ajouter une règle de sécurité manquante, procédez comme suit :

1. Dans le Portail Azure, recherchez et sélectionnez **Groupes de sécurité réseau**.
1. Sélectionnez le groupe de sécurité réseau associé à votre domaine managé, par exemple *AADDS-contoso.com-NSG*.
1. Sous **Paramètres** dans le panneau de gauche, cliquez sur *Règles de sécurité de trafic entrant* ou *Règles de sécurité de trafic sortant* selon la règle à ajouter.
1. Sélectionnez **Ajouter**, puis créez la règle requise en fonction du port, du protocole, de la direction, etc. Lorsque vous êtes prêt, sélectionnez **OK**.

L’ajout et l’affichage de la règle de sécurité dans la liste prend quelques instants.

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez toujours des problèmes, [formulez une demande de support Azure][azure-support] pour bénéficier d’une aide supplémentaire.

<!-- INTERNAL LINKS -->
[azure-support]: ../active-directory/fundamentals/active-directory-troubleshooting-support-howto.md
[configure-ldaps]: tutorial-configure-ldaps.md

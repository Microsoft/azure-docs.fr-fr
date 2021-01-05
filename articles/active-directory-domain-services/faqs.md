---
title: Forum aux questions sur les services de domaine Azure AD | Microsoft Docs
description: Lisez et comprenez certaines des questions fréquemment posées sur la configuration, l’administration et la disponibilité d’Azure Active Directory Domain Services
services: active-directory-ds
author: justinha
manager: daveba
ms.assetid: 48731820-9e8c-4ec2-95e8-83dba1e58775
ms.service: active-directory
ms.subservice: domain-services
ms.workload: identity
ms.topic: how-to
ms.date: 09/30/2020
ms.author: justinha
ms.openlocfilehash: e4e7a64b9f9d7283de728216934854f4ef8f8fd1
ms.sourcegitcommit: 8192034867ee1fd3925c4a48d890f140ca3918ce
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/05/2020
ms.locfileid: "96619731"
---
# <a name="frequently-asked-questions-faqs-about-azure-active-directory-ad-domain-services"></a>Foire aux questions (FAQ) sur Azure Active Directory (AD) Domain Services

Cette page répond aux questions fréquemment posées sur les services de domaine Azure Active Directory.

## <a name="configuration"></a>Configuration

* [Puis-je créer plusieurs domaines managés pour un seul annuaire Azure AD ?](#can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory)
* [Puis-je activer Azure AD Domain Services dans un réseau virtuel Classic ?](#can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network)
* [Puis-je activer les services de domaine Azure AD dans un réseau virtuel Azure Resource Manager ?](#can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network)
* [Puis-je migrer un domaine managé existant depuis un réseau virtuel Classic vers un réseau Resource Manager virtuel ?](#can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network)
* [Puis-je activer Azure Active Directory Domain Services dans un abonnement Azure CSP (fournisseur de solutions de Cloud) ?](#can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription)
* [Puis-je activer les services de domaine Azure AD dans un annuaire Azure AD fédéré ? Je ne synchronise pas les hachages de mot de passe avec Azure AD. Puis-je activer les services de domaine Azure AD pour cet annuaire ?](#can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory)
* [Puis-je rendre les services de domaine Azure AD disponibles dans plusieurs réseaux virtuels au sein de mon abonnement ?](#can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription)
* [Puis-je activer les services de domaine Azure AD à l’aide de PowerShell ?](#can-i-enable-azure-ad-domain-services-using-powershell)
* [Puis-je activer Azure AD Domain Services à l’aide d’un modèle Resource Manager ?](#can-i-enable-azure-ad-domain-services-using-a-resource-manager-template)
* [Puis-je ajouter des contrôleurs de domaine à un domaine géré par les services de domaine Azure AD ?](#can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain)
* [Les utilisateurs invités de mon annuaire peuvent-ils utiliser Azure Active Directory Domain Services ?](#can-guest-users-be-invited-to-my-directory-use-azure-ad-domain-services)
* [Puis-je déplacer un domaine managé Azure AD Domain Services existant vers un abonnement, un groupe de ressources, une région ou un réseau virtuel différent ?](#can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network)
* [Azure AD Domain Services inclut-il des options de haute disponibilité ?](#does-azure-ad-domain-services-include-high-availability-options)

### <a name="can-i-create-multiple-managed-domains-for-a-single-azure-ad-directory"></a>Puis-je créer plusieurs domaines managés pour un seul annuaire Azure AD ?
Non. Vous ne pouvez créer qu’un seul domaine managé pris en charge par Azure AD Domain Services par annuaire Azure AD.

### <a name="can-i-enable-azure-ad-domain-services-in-a-classic-virtual-network"></a>Puis-je activer Azure AD Domain Services dans un réseau virtuel Classic ?
Les réseaux virtuels classiques ne sont pas pris en charge pour les nouveaux déploiements. Les domaines managés existants qui sont déployés dans des réseaux virtuels classiques continuent à être pris en charge jusqu’à leur mise hors service le 1er mars 2023. Pour les déploiements existants, vous pouvez [migrer Azure AD Domain Services depuis le modèle de réseau virtuel classique vers Resource Manager](migrate-from-classic-vnet.md).

Pour plus d’informations, consultez l’[annonce de dépréciation officielle](https://azure.microsoft.com/updates/we-are-retiring-azure-ad-domain-services-classic-vnet-support-on-march-1-2023/).

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-resource-manager-virtual-network"></a>Puis-je activer les services de domaine Azure AD dans un réseau virtuel Azure Resource Manager ?
Oui. Azure AD Domain Services peut être activé dans un réseau virtuel Azure Resource Manager. Les réseaux virtuels Azure classiques ne sont plus disponibles quand vous créez un domaine managé.

### <a name="can-i-migrate-my-existing-managed-domain-from-a-classic-virtual-network-to-a-resource-manager-virtual-network"></a>Puis-je migrer un domaine managé existant depuis un réseau virtuel Classic vers un réseau Resource Manager virtuel ?
Oui. Pour plus d’informations, consultez [Migrer Azure Active Directory Domain Services depuis le modèle de réseau virtuel classique vers Resource Manager](migrate-from-classic-vnet.md).

### <a name="can-i-enable-azure-ad-domain-services-in-an-azure-csp-cloud-solution-provider-subscription"></a>Puis-je activer Azure Active Directory Domain Services dans un abonnement Azure CSP (fournisseur de solutions de Cloud) ?
Oui. Pour plus d'informations, consultez [Activer Azure Active Directory Domain Services dans les abonnements Azure CSP](csp.md).

### <a name="can-i-enable-azure-ad-domain-services-in-a-federated-azure-ad-directory-i-do-not-synchronize-password-hashes-to-azure-ad-can-i-enable-azure-ad-domain-services-for-this-directory"></a>Puis-je activer les services de domaine Azure AD dans un annuaire Azure AD fédéré ? Je ne synchronise pas les hachages de mot de passe avec Azure AD. Puis-je activer les services de domaine Azure AD pour cet annuaire ?
Non. Les services de domaine Azure AD ont besoin d’un accès aux hachages de mot de passe des comptes d’utilisateur afin d’authentifier les utilisateurs via NTLM ou Kerberos. Dans un annuaire fédéré, les hachages de mot de passe ne sont pas stockés dans l’annuaire Azure AD. Par conséquent, les services de domaine Azure AD ne fonctionnent pas avec ces annuaires Azure AD.

Toutefois, si vous utilisez Azure AD Connect pour la synchronisation de hachage du mot de passe, vous pouvez utiliser Azure AD Domain Services parce que les valeurs de hachage du mot de passe sont stockées dans Azure AD.

### <a name="can-i-make-azure-ad-domain-services-available-in-multiple-virtual-networks-within-my-subscription"></a>Puis-je rendre les services de domaine Azure AD disponibles dans plusieurs réseaux virtuels au sein de mon abonnement ?
Le service lui-même ne prend pas directement en charge ce scénario. Votre domaine managé n’est disponible que dans un seul réseau virtuel à la fois. Toutefois, vous pouvez configurer la connectivité entre plusieurs réseaux virtuels afin d’exposer les services de domaine Azure AD à d’autres réseaux virtuels. Pour plus d’informations, consultez [connexion aux réseaux virtuels dans Azure à l’aide de passerelles VPN](../vpn-gateway/vpn-gateway-howto-vnet-vnet-portal-classic.md)ou [homologation de réseaux virtuels](../virtual-network/virtual-network-peering-overview.md).

### <a name="can-i-enable-azure-ad-domain-services-using-powershell"></a>Puis-je activer les services de domaine Azure AD à l’aide de PowerShell ?
Oui. Pour plus d'informations, consultez [Activer Azure Active Directory Domain Services à l’aide de PowerShell](powershell-create-instance.md).

### <a name="can-i-enable-azure-ad-domain-services-using-a-resource-manager-template"></a>Puis-je activer Azure AD Domain Services à l’aide d’un modèle Resource Manager ?
Oui, vous pouvez créer un domaine managé Azure AD Domain Services à l’aide d’un modèle Resource Manager. Un principal de service et un groupe Azure AD pour l'administration doivent être créés à l'aide du portail Azure ou d'Azure PowerShell avant le déploiement du modèle. Pour plus d’informations, consultez [Créer un domaine managé Azure Active Directory Domain Services à l’aide d’un modèle Resource Manager](template-create-instance.md). Lorsque vous créez un domaine managé Azure AD Domain Services dans le portail Azure, vous avez aussi la possibilité d'exporter le modèle pour l'utiliser avec d’autres déploiements.

### <a name="can-i-add-domain-controllers-to-an-azure-ad-domain-services-managed-domain"></a>Puis-je ajouter des contrôleurs de domaine à un domaine géré par les services de domaine Azure AD ?
Non. Le domaine fourni par les services de domaine Azure AD est un domaine géré. Vous n’avez pas besoin d’approvisionner, de configurer ou de gérer des contrôleurs de domaine pour ce domaine. Ces activités de gestion sont fournies en tant que service par Microsoft. Par conséquent, vous ne pouvez pas ajouter de contrôleurs de domaine supplémentaires (en lecture/écriture ou en lecture seule) pour le domaine géré.

### <a name="can-guest-users-be-invited-to-my-directory-use-azure-ad-domain-services"></a>Les utilisateurs invités de mon annuaire peuvent-ils utiliser Azure Active Directory Domain Services ?
Non. Les utilisateurs invités de votre annuaire Azure AD qui utilisent le processus d’invitation [Azure AD B2B](../active-directory/external-identities/what-is-b2b.md) sont synchronisés avec votre domaine managé Azure AD Domain Services. Toutefois, les mots de passe de ces utilisateurs ne sont pas stockés dans votre annuaire Azure AD. C’est pourquoi Azure AD Domain Services n’a aucun moyen de synchroniser les hachages NTLM et Kerberos pour ces utilisateurs dans votre domaine managé. De tels utilisateurs ne peuvent pas se connecter ou joindre des ordinateurs au domaine géré.

### <a name="can-i-move-an-existing-azure-ad-domain-services-managed-domain-to-a-different-subscription-resource-group-region-or-virtual-network"></a>Puis-je déplacer un domaine managé Azure AD Domain Services existant vers un abonnement, un groupe de ressources, une région ou un réseau virtuel différent ?
Non. Après avoir créé un domaine managé Azure AD Domain Services, vous ne pouvez pas déplacer le domaine managé vers un autre groupe de ressources, réseau virtuel, abonnement, etc. Veillez à sélectionner l’abonnement, le groupe de ressources, la région et le réseau virtuel les plus appropriés quand vous déployez le domaine managé.

### <a name="does-azure-ad-domain-services-include-high-availability-options"></a>Azure AD Domain Services inclut-il des options de haute disponibilité ?

Oui. Chaque domaine managé Azure Active Directory Domain Services comprend deux contrôleurs de domaine. Vous ne gérez pas ces contrôleurs de domaine et ne vous y connectez pas, car ils font partie du service géré. Si vous déployez Azure AD Domain Services dans une région qui prend en charge les Zones de disponibilité, les contrôleurs de domaine sont répartis entre les zones. Dans les régions qui ne prennent pas en charge les Zones de disponibilité, les contrôleurs de domaine sont distribués entre les Groupes à haute disponibilité. Vous ne disposez d’aucune option de configuration ou d’aucun contrôle de gestion sur cette distribution. Pour plus d’informations, voir [Options de disponibilité pour les machines virtuelles dans Azure](../virtual-machines/availability.md).

## <a name="administration-and-operations"></a>Administration et opérations

* [Puis-je me connecter au contrôleur de domaine de mon domaine géré à l’aide du Bureau à distance ?](#can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop)
* [J’ai activé Azure AD Domain Services. Quel compte d’utilisateur dois-je utiliser pour joindre des ordinateurs à ce domaine ?](#ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain)
* [Est-ce que je dispose des privilèges d’administrateur de domaine pour le domaine géré fourni par les services de domaine Azure AD ?](#do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services)
* [Puis-je modifier les appartenances aux groupes à l’aide de LDAP ou d’autres outils d’administration AD sur des domaines gérés ?](#can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains)
* [Combien de temps faut-il pour que les modifications que j’apporte à mon annuaire Azure AD soient visibles dans mon domaine géré ?](#how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain)
* [Puis-je étendre le schéma du domaine géré fourni par les services de domaine Azure AD ?](#can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services)
* [Puis-je modifier ou ajouter des enregistrements DNS dans mon domaine géré ?](#can-i-modify-or-add-dns-records-in-my-managed-domain)
* [Quelle est la stratégie de durée de vie des mots de passe dans un domaine managé ?](#what-is-the-password-lifetime-policy-on-a-managed-domain)
* [Le service Azure AD Domain Services assure-t-il une protection par verrouillage du compte AD ?](#does-azure-ad-domain-services-provide-ad-account-lockout-protection)
* [Puis-je configurer le système de fichiers DFS (DFS) et la réplication dans Azure AD Domain Services ?](#can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services)
* [Comment les mises à jour Windows sont-elles appliquées dans Azure AD Domain Services ?](#how-are-windows-updates-applied-in-azure-ad-domain-services)

### <a name="can-i-connect-to-the-domain-controller-for-my-managed-domain-using-remote-desktop"></a>Puis-je me connecter au contrôleur de domaine de mon domaine géré à l’aide du Bureau à distance ?
Non. Vous n’êtes pas autorisé à vous connecter aux contrôleurs de domaine pour le domaine géré, via le Bureau à distance. Les membres du groupe *AAD DC Administrators* peuvent administrer le domaine géré à l’aide des outils d’administration AD, tels que le centre d’administration d’Active Directory (ADAC) ou AD PowerShell. Ces outils sont installés à l’aide de la fonctionnalité *Outils d’administration de serveur distant* sur un serveur Windows joint au domaine géré. Pour plus d’informations, consultez [Créer une machine virtuelle de gestion pour configurer et administrer un domaine managé Azure AD Domain Services](tutorial-create-management-vm.md).

### <a name="ive-enabled-azure-ad-domain-services-what-user-account-do-i-use-to-domain-join-machines-to-this-domain"></a>J’ai activé Azure AD Domain Services. Quel compte d’utilisateur dois-je utiliser pour joindre des ordinateurs à ce domaine ?
Tout compte d’utilisateur faisant partie du domaine managé peut joindre une machine virtuelle. Les membres du groupe *Administrateurs AAD DC* disposent d’un accès Bureau à distance aux machines qui ont été jointes au domaine managé.

### <a name="do-i-have-domain-administrator-privileges-for-the-managed-domain-provided-by-azure-ad-domain-services"></a>Est-ce que je dispose des privilèges d’administrateur de domaine pour le domaine géré fourni par les services de domaine Azure AD ?
Non. Vous ne disposez pas des privilèges d’administrateur sur le domaine géré. Les privilèges *Administrateur de domaine* et *Administrateur d’entreprise* ne sont pas disponibles pour vous dans le domaine. Les membres des groupes d’administrateurs de domaine ou d’administrateurs d’entreprise de votre instance Active Directory locale ne se voient accorder aucun privilège d’administrateur de domaine ou d’entreprise sur le domaine managé.

### <a name="can-i-modify-group-memberships-using-ldap-or-other-ad-administrative-tools-on-managed-domains"></a>Puis-je modifier les appartenances aux groupes à l’aide de LDAP ou d’autres outils d’administration AD sur des domaines gérés ?
Il n’est pas possible de modifier les utilisateurs et les groupes synchronisés à partir d’Azure Active Directory sur Azure AD Domain Services, car leur source d’origine est Azure Active Directory. En revanche, tout utilisateur ou groupe provenant du domaine managé peut être modifié.

### <a name="how-long-does-it-take-for-changes-i-make-to-my-azure-ad-directory-to-be-visible-in-my-managed-domain"></a>Combien de temps faut-il pour que les modifications que j’apporte à mon annuaire Azure AD soient visibles dans mon domaine géré ?
Les modifications apportées à votre annuaire Azure AD à l’aide de l’interface utilisateur d’Azure Active Directory ou de PowerShell sont synchronisées automatiquement avec votre domaine géré. Ce processus de synchronisation se produit en arrière-plan. Il n’y a pas de période définie pour cette synchronisation pour effectuer toutes les modifications d’objets.

### <a name="can-i-extend-the-schema-of-the-managed-domain-provided-by-azure-ad-domain-services"></a>Puis-je étendre le schéma du domaine géré fourni par les services de domaine Azure AD ?
Non. Le schéma est administré par Microsoft pour le domaine géré. Les extensions de schéma ne sont pas prises en charge par les services de domaine Azure AD.

### <a name="can-i-modify-or-add-dns-records-in-my-managed-domain"></a>Puis-je modifier ou ajouter des enregistrements DNS dans mon domaine géré ?
Oui. Les membres du groupe *AAD DC Administrators* bénéficient de privilèges *Administrateur DNS*, afin de modifier les enregistrements DNS sur le domaine géré. Ces utilisateurs peuvent utiliser la console du Gestionnaire DNS sur un ordinateur exécutant Windows Server joint au domaine managé afin de gérer le système DNS. Pour utiliser la console du Gestionnaire DNS, installez les *outils de serveur DNS*, qui font partie de la fonctionnalité facultative *Outils d’administration de serveur distant* sur le serveur. Pour plus d’informations, consultez [Administrer DNS dans un domaine managé Azure AD Domain Services](manage-dns.md).

### <a name="what-is-the-password-lifetime-policy-on-a-managed-domain"></a>Quelle est la stratégie de durée de vie des mots de passe dans un domaine managé ?
Par défaut, la durée de vie des mots de passe dans un domaine managé Azure AD Domain Services est de 90 jours. Cette durée de vie des mots de passe n’est pas synchronisée avec celle configurée dans Azure AD. Par conséquent, il est possible qu’un mot de passe utilisateur expire dans votre domaine managé, mais soit toujours valide dans Azure AD. Dans les scénarios comme celui-ci, les utilisateurs doivent modifier leur mot de passe dans Azure AD. Le nouveau mot de passe est ensuite synchronisé avec votre domaine managé. Si vous souhaitez modifier la durée de vie par défaut d’un mot de passe dans un domaine géré, vous pouvez [créer et configurer des stratégies de mot de passe personnalisées.](password-policy.md).

En outre, la stratégie de mot de passe Azure AD pour *DisablePasswordExpiration* est synchronisée avec un domaine managé. Lorsque *DisablePasswordExpiration* est appliqué à un utilisateur dans Azure AD, la valeur *UserAccountControl* pour l’utilisateur synchronisé dans le domaine managé a *DONT_EXPIRE_PASSWORD* appliqué.

Lorsque les utilisateurs réinitialisent leur mot de passe dans Azure AD, l’attribut *forceChangePasswordNextSignIn=true* est appliqué. Un domaine managé synchronise cet attribut à partir de Azure AD. Lorsque le domaine managé détecte que *forceChangePasswordNextSignIn* est défini pour un utilisateur synchronisé à partir de Azure AD, l’attribut *pwdLastSet* dans le domaine managé est défini sur *0*, ce qui invalide le mot de passe actuellement défini.

### <a name="does-azure-ad-domain-services-provide-ad-account-lockout-protection"></a>Le service Azure AD Domain Services assure-t-il une protection par verrouillage du compte AD ?
Oui. Cinq tentatives de saisie de mot de passe non valide en 2 minutes dans le domaine managé entraînent le verrouillage d’un compte d’utilisateur pendant 30 minutes. Après ces 30 minutes, le compte d’utilisateur est automatiquement déverrouillé. Les tentatives de saisie de mot de passe non valide dans le domaine managé ne verrouillent pas le compte d’utilisateur dans Azure AD. Le compte d’utilisateur est verrouillé uniquement dans votre domaine managé Azure AD Domain Services. Pour plus d'informations, consultez [Stratégies de mot de passe et de verrouillage de compte sur les domaines managés](password-policy.md).

### <a name="can-i-configure-distributed-file-system-and-replication-within-azure-ad-domain-services"></a>Puis-je configurer le système de fichiers DFS et la réplication dans Azure AD Domain Services ?
Non. Le système de fichiers DFS (Distributed File System) et la réplication ne sont pas disponibles lors de l’utilisation d’Azure AD Domain Services.

### <a name="how-are-windows-updates-applied-in-azure-ad-domain-services"></a>Comment les mises à jour Windows sont-elles appliquées dans Azure AD Domain Services ?
Les contrôleurs de domaine dans un domaine managé appliquent automatiquement les mises à jour Windows requises. Il n’y a rien à configurer ni à administrer ici. Veillez à ne pas créer de règles de groupe de sécurité réseau qui bloquent le trafic sortant vers les mises à jour Windows. Pour vos propres machines virtuelles jointes au domaine géré, vous êtes responsable de la configuration et de l’application des mises à jour requises du système d’exploitation et des applications.

## <a name="billing-and-availability"></a>Facturation et disponibilité

* [Les services de domaine Azure AD sont-ils payants ?](#is-azure-ad-domain-services-a-paid-service)
* [Le service peut-il être essayé gratuitement ?](#is-there-a-free-trial-for-the-service)
* [Puis-je suspendre un domaine géré par Azure AD Domain Services ?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [Puis-je basculer Azure AD Domain Services vers une autre région pour un événement de récupération d’urgence ?](#can-i-pause-an-azure-ad-domain-services-managed-domain)
* [Puis-je obtenir les services de domaine Azure AD dans le cadre d’Enterprise Mobility Suite (EMS) ? Ai-je besoin d’Azure AD Premium pour utiliser les services de domaine Azure AD ?](#can-i-fail-over-azure-ad-domain-services-to-another-region-for-a-dr-event)
* [Dans quelles régions Azure le service est-il disponible ?](#can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services)

### <a name="is-azure-ad-domain-services-a-paid-service"></a>Les services de domaine Azure AD sont-ils payants ?
Oui. Pour plus d’informations, consultez la [page relative aux prix appliqués](https://azure.microsoft.com/pricing/details/active-directory-ds/).

### <a name="is-there-a-free-trial-for-the-service"></a>Le service peut-il être essayé gratuitement ?
Azure AD Domain Services est inclus dans l’essai gratuit d’Azure. Vous pouvez vous inscrire pour bénéficier d’un [essai gratuit d’un mois d’Azure](https://azure.microsoft.com/pricing/free-trial/).

### <a name="can-i-pause-an-azure-ad-domain-services-managed-domain"></a>Puis-je suspendre un domaine géré par Azure AD Domain Services ?
Non. Une fois que vous avez activé un domaine géré par Azure AD Domain Services, le service est disponible dans votre réseau virtuel sélectionné jusqu’à ce que vous supprimiez le domaine géré. Il n’existe aucun moyen de suspendre le service. La facturation horaire se poursuivra jusqu’à ce que vous supprimiez le domaine managé.

### <a name="can-i-fail-over-azure-ad-domain-services-to-another-region-for-a-dr-event"></a>Puis-je basculer Azure AD Domain Services vers une autre région pour un événement de récupération d’urgence ?
Non. Azure AD Domain Services ne fournit pas actuellement de modèle de déploiement géoredondant. Il est limité à un seul réseau virtuel dans une région Azure. Si vous souhaitez utiliser plusieurs régions Azure, vous devez exécuter vos contrôleurs de domaine Active Directory sur des machines virtuelles IaaS Azure. Pour des conseils en architecture, consultez [Étendre votre domaine Active Directory local à Azure](/azure/architecture/reference-architectures/identity/adds-extend-domain).

### <a name="can-i-get-azure-ad-domain-services-as-part-of-enterprise-mobility-suite-ems-do-i-need-azure-ad-premium-to-use-azure-ad-domain-services"></a>Puis-je obtenir les services de domaine Azure AD dans le cadre d’Enterprise Mobility Suite (EMS) ? Ai-je besoin d’Azure AD Premium pour utiliser les services de domaine Azure AD ?
Non. Les services de domaine Azure AD constituent un service Azure avec paiement à l’utilisation et ne font pas partie d’EMS. Azure Active Directory Domain Services peut être utilisé avec toutes les éditions d’Azure AD (Gratuit et Premium). La facturation se fait à l’utilisation, sur une base horaire.

### <a name="what-azure-regions-is-the-service-available-in"></a>Dans quelles régions Azure le service est-il disponible ?
Pour consulter une liste des régions Azure dans lesquelles les services de domaine Azure AD sont disponibles, consultez la page [Régions Azure](https://azure.microsoft.com/regions/#services/).

## <a name="troubleshooting"></a>Dépannage

Reportez-vous au [Guide de résolution des problèmes](troubleshoot.md) pour trouver les solutions aux problèmes courants liés à la configuration ou à l’administration d’Azure AD Domain Services.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur Azure AD Domain Services, consultez [Présentation d’Azure Active Directory Domain Services](overview.md).

Pour bien démarrer, consultez [Créer et configurer un domaine managé Azure Active Directory Domain Services](tutorial-create-instance.md).

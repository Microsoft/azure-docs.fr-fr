---
title: 'Concepts : Accès et identité dans Azure Kubernetes Service (AKS)'
description: Découvrez l’accès et l’identité dans Azure Kubernetes Service (AKS), notamment l’intégration d’Azure Active Directory, le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes) ainsi que les rôles et les liaisons.
services: container-service
ms.topic: conceptual
ms.date: 07/07/2020
author: palma21
ms.author: jpalma
ms.openlocfilehash: 983b1a5e024a44733fab418a67375f232e66cfe4
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96457166"
---
# <a name="access-and-identity-options-for-azure-kubernetes-service-aks"></a>Options d’accès et d’identité pour Azure Kubernetes Service (AKS)

Il existe différentes façons d'authentifier, de contrôler/autoriser l'accès et de sécuriser les clusters Kubernetes. Avec le contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes), vous pouvez accorder à des utilisateurs, groupes et comptes de service l’accès aux seules ressources dont ils ont besoin. À l’aide d’Azure Kubernetes Service (AKS), vous pouvez améliorer la structure de sécurité et des autorisations en utilisant Azure Active Directory et Azure RBAC. Ces approches vous aident à sécuriser l’accès à votre cluster et fournissent uniquement les autorisations minimales requises aux développeurs et aux opérateurs.

Cet article présente les principaux concepts vous permettant de vous authentifier et d’affecter des autorisations dans AKS :

- [Contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes)](#kubernetes-role-based-access-control-kubernetes-rbac)
  - [Rôles et ClusterRoles](#roles-and-clusterroles)
  - [RoleBindings et ClusterRoleBindings](#rolebindings-and-clusterrolebindings) 
  - [Comptes de service Kubernetes](#kubernetes-service-accounts)
- [Intégration d’Azure Active Directory](#azure-active-directory-integration)
- [Azure RBAC](#azure-role-based-access-control-azure-rbac)
  - [Azure RBAC pour autoriser l’accès à la ressource AKS](#azure-rbac-to-authorize-access-to-the-aks-resource)
  - [Utiliser Azure RBAC pour l’autorisation Kubernetes (préversion)](#azure-rbac-for-kubernetes-authorization-preview)


## <a name="kubernetes-role-based-access-control-kubernetes-rbac"></a>Contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes)

Afin de fournir un filtrage granulaire des actions que les utilisateurs peuvent effectuer, Kubernetes recourt au contrôle d’accès en fonction du rôle Kubernetes (RBAC Kubernetes). Ce mécanisme de contrôle vous permet d’affecter à des utilisateurs ou à des groupes d’utilisateurs, l’autorisation d’accomplir des opérations, telles que la création ou la modification de ressources, ou encore l’affichage de journaux d’activité à partir de charges de travail d’applications en cours d’exécution. La portée de ces autorisations peut être limitée à un seul espace de noms, ou accordée à l’ensemble du cluster AKS. Avec le contrôle RBAC de Kubernetes, vous créez des *rôles* pour définir des autorisations, puis affectez ces rôles aux utilisateurs avec les *liaisons de rôle*.

Pour plus d’informations, consultez [Utilisation de l’autorisation RBAC Kubernetes][kubernetes-rbac].


### <a name="roles-and-clusterroles"></a>Rôles et ClusterRoles

Avant d’attribuer des autorisations aux utilisateurs avec RBAC Kubernetes, vous devez tout d’abord définir ces autorisations en tant que *rôle*. Les rôles Kubernetes *accordent* des autorisations. Le concept d’autorisation *refusée* n’existe pas.

Les rôles sont utilisés pour accorder des autorisations dans un espace de noms. Si vous devez accorder des autorisations sur l’ensemble du cluster ou sur des ressources de cluster en dehors d’un espace de noms donné, vous pouvez utiliser *ClusterRoles* à la place.

Un ClusterRole fonctionne de la même façon pour accorder des autorisations aux ressources, mais il peut être appliqué aux ressources dans l’ensemble du cluster, et non dans un espace de noms spécifique.

### <a name="rolebindings-and-clusterrolebindings"></a>RoleBindings et ClusterRoleBindings

Une fois les rôles définis pour accorder des autorisations aux ressources, vous affectez ces autorisations RBAC Kubernetes au moyen d’un *RoleBinding*. Si votre cluster AKS[ s’intègre avec Azure Active Directory (Azure AD)](#azure-active-directory-integration), les liaisons sont la manière dont ces utilisateurs Azure AD se voient accorder des autorisations pour effectuer des actions au sein du cluster. Pour plus d’informations, consultez la section [Contrôler l’accès aux ressources de cluster à l’aide du contrôle d’accès en fonction du rôle Kubernetes et des identités Azure Active Directory](azure-ad-rbac.md).

Les liaisons de rôle servent à assigner des rôles pour un espace de noms donné. Cette approche vous permet de séparer logiquement un cluster AKS unique, avec des utilisateurs uniquement capable d’accéder aux ressources d’application dans leur espace de noms attribué. Si vous avez besoin de lier des rôles sur l’ensemble du cluster, ou sur des ressources de cluster en dehors d’un espace de noms donné, vous pouvez utiliser *ClusterRoleBindings* à la place.

Un ClusterRoleBinding fonctionne de la même façon pour lier des rôles aux utilisateurs, mais il peut être appliqué aux ressources sur l’ensemble du cluster, et non à un espace de noms spécifique. Ce moyen vous permet d’accorder aux administrateurs ou aux techniciens du support technique l’accès à toutes les ressources dans le cluster AKS.


> [!NOTE]
> Toute action de cluster prise par Microsoft/AKS est effectuée avec le consentement de l’utilisateur sous un rôle Kubernetes intégré `aks-service` et une liaison de rôle intégrée `aks-service-rolebinding`. Ce rôle permet à AKS de dépanner et de diagnostiquer les problèmes de cluster, mais ne peut pas modifier les autorisations, ni créer des rôles ou des liaisons de rôle, ni d’autres actions à privilèges élevés. L’accès aux rôles est activé uniquement sous les tickets de support actifs avec l’accès juste-à-temps (JIT). En savoir plus sur les stratégies de prise en charge [AKS](support-policies.md).


### <a name="kubernetes-service-accounts"></a>Comptes de service Kubernetes

Un des principaux types d’utilisateur dans Kubernetes est le *compte de service*. Un compte de service existe dans l’API Kubernetes et est géré par cette API. Les informations d’identification des comptes de service sont stockées en tant que secrets Kubernetes, ce qui leur permet d’être utilisées par les pods autorisés à communiquer avec le serveur d’API. La plupart des requêtes d’API fournissent un jeton d’authentification pour un compte de service ou un compte d’utilisateur normal.

Les comptes d'utilisateurs normaux permettent un accès plus traditionnel pour les administrateurs ou les développeurs humains, et pas seulement pour les services et les processus. Kubernetes ne fournit pas de solution de gestion des identités dans laquelle les comptes et les mots de passe d'utilisateurs standard sont stockés. Par contre, il est possible d’intégrer des solutions d’identité externes à Kubernetes. Pour les clusters AKS, cette solution d’identité intégrée est Azure Active Directory.

Pour plus d’informations sur les options d’identité dans Kubernetes, consultez [Authentification Kubernetes][kubernetes-authentication].

## <a name="azure-active-directory-integration"></a>Intégration d’Azure Active Directory

La sécurité des clusters AKS peut être améliorée avec l’intégration d’Azure Active Directory (AD). S’appuyant sur des décennies de gestion des identités d’entreprise, Azure AD est un service multilocataire, basé sur le cloud pour la gestion des identités et des annuaires, qui combine les principaux services d’annuaire, la gestion des accès aux applications et la protection des identités. Avec Azure AD, vous pouvez intégrer des identités locales dans les clusters AKS pour fournir une source unique de sécurité et de gestion des comptes.

![Intégration d’Azure Active Directory aux clusters AKS](media/concepts-identity/aad-integration.png)

Avec les clusters AKS intégrés Azure AD, vous pouvez accorder aux utilisateurs ou aux groupes l’accès aux ressources Kubernetes dans un espace de noms ou au sein du cluster. Pour obtenir un contexte de configuration `kubectl`, un utilisateur peut exécuter la commande [az aks get-credentials][az-aks-get-credentials]. Lorsqu’un utilisateur interagit ensuite avec le cluster AKS par le biais de `kubectl`, il est invité à se connecter avec ses informations d’identification Azure AD. Cette solution fournit une source unique pour les informations d’identification de mots de passe et de gestion des comptes utilisateur. L’utilisateur peut uniquement accéder aux ressources, tel que défini par l’administrateur du cluster.

L’authentification Azure AD est fournie aux clusters AKS à l’aide d’OpenID Connect. OpenID Connect est une couche d’identité basée sur le protocole OAuth 2.0. Pour plus d’informations sur OpenID Connect, voir la [documentation sur Open ID Connect][openid-connect]. Depuis le cluster Kubernetes, [l’authentification par jeton de Webhook][webhook-token-docs] est utilisée pour vérifier les jetons d’authentification. L’authentification par jeton de Webhook est configurée et gérée en tant que partie du cluster AKS.

### <a name="webhook-and-api-server"></a>Webhook et serveur API

![Webhook et le flux d’authentification du serveur API](media/concepts-identity/auth-flow.png)

Comme indiqué dans le graphique ci-dessus, le serveur API appelle le serveur webhook AKS et effectue les étapes suivantes :

1. L’application cliente Azure AD est utilisée par kubectl pour connecter des utilisateurs avec le [flux d’octroi d’autorisation d’appareil OAuth 2,0](../active-directory/develop/v2-oauth2-device-code.md).
2. Azure AD fournit un access_token, id_token et un refresh_token.
3. L’utilisateur envoie une requête à kubectl avec un access_token à partir de kubeconfig.
4. Kubectl envoie l’access_token au serveur API.
5. Le serveur API est configuré avec le serveur webhook d’authentification pour effectuer la validation.
6. Le serveur webhook d’authentification confirme que la signature JSON Web Token est valide en vérifiant la clé de signature publique Azure AD.
7. L’application serveur utilise les informations d’identification fournies par l’utilisateur pour interroger les appartenances aux groupes de l’utilisateur connecté à partir de l’API Graph MS.
8. Une réponse est envoyée au serveur API avec des informations utilisateur telles que la revendication UPN (nom d’utilisateur principal) du jeton d’accès et l’appartenance au groupe de l’utilisateur en fonction de l’ID d’objet.
9. L’API effectue une décision d’autorisation basée sur le rôle Kubernetes/RoleBinding.
10. Une fois l’autorisation accordée, le serveur API renvoie une réponse à kubectl.
11. Kubectl envoie les commentaires à l’utilisateur.
 
**Découvrez comment intégrer AKS à AAD [ici](managed-aad.md).**

## <a name="azure-role-based-access-control-azure-rbac"></a>Contrôle d’accès en fonction du rôle Azure (Azure RBAC)

Le contrôle RBAC Azure est un système d’autorisation basé sur [Azure Resource Manager](../azure-resource-manager/management/overview.md) qui propose une gestion affinée des accès des ressources Azure.

 Le contrôle RBAC Azure est conçu pour fonctionner sur les ressources de votre abonnement AKS, tandis que la conception de RBAC Kubernetes le destine aux ressources Kubernetes de votre cluster AKS. 

Avec le contrôle RBAC Azure, vous créez une *définition de rôle* qui décrit les autorisations à appliquer. Cette définition de rôle est ensuite affectée, via une *attribution de rôle*, à un utilisateur ou à un groupe sur une *étendue particulière*, qui peut être une ressource individuelle, un groupe de ressources ou l’abonnement.

Pour plus d’informations, consultez [Qu’est-ce que le contrôle d’accès en fonction du rôle Azure (Azure RBAC) ?][azure-rbac]

Il existe deux niveaux d’accès nécessaires pour pleinement utiliser un cluster AKS : 
1. [Accédez à la ressource AKS dans votre abonnement Azure](#azure-rbac-to-authorize-access-to-the-aks-resource). Ce niveau d’accès vous permet de contrôler les opérations de mise à l’échelle ou de mise à niveau de votre cluster à l’aide des API AKS, ainsi que d’extraire votre kubeconfig.
2. Accédez à l’API Kubernetes. Ce niveau d’accès est contrôlé par le [RBAC Kubernetes](#kubernetes-role-based-access-control-kubernetes-rbac) (traditionnellement) ou par l’[intégration du RBAC Azure à AKS pour l’autorisation Kubernetes](#azure-rbac-for-kubernetes-authorization-preview).

### <a name="azure-rbac-to-authorize-access-to-the-aks-resource"></a>Azure RBAC pour autoriser l’accès à la ressource AKS

Avec RBAC Azure, vous pouvez fournir à vos utilisateurs (ou identités) un accès granulaire aux ressources AKS dans un ou plusieurs abonnements. Par exemple, vous pouvez avoir le [rôle de contributeur d’Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-contributor-role) qui vous permet d’effectuer des actions telles que mettre à l’échelle et mettre à niveau votre cluster. Alors qu’un autre utilisateur peut avoir le [rôle d’administrateur de cluster d’Azure Kubernetes Service](../role-based-access-control/built-in-roles.md#azure-kubernetes-service-cluster-admin-role) qui autorise uniquement l’extraction du kubeconfig d’administration.

Vous pouvez également donner à votre utilisateur le rôle de [Contributeur](../role-based-access-control/built-in-roles.md#contributor) général, qui englobe les autorisations ci-dessus et toutes les actions possibles sur la ressource AKS, à l’exception de la gestion des autorisations.

Découvrez plus d’informations sur l’utilisation d’Azure RBAC pour sécuriser l’accès au fichier kubeconfig qui donne accès à l’API Kubernetes [ici](control-kubeconfig-access.md).

### <a name="azure-rbac-for-kubernetes-authorization-preview"></a>Utiliser Azure RBAC pour l’autorisation Kubernetes (préversion)

Avec l’intégration RBAC Azure, AKS utilise un serveur webhook d’autorisation Kubernetes pour vous permettre de gérer les autorisations et les attributions de ressources de cluster K8s intégrées à Azure AD à l’aide de la définition de rôle Azure et des attributions de rôles.

![Azure RBAC pour le flux d’autorisation Kubernetes](media/concepts-identity/azure-rbac-k8s-authz-flow.png)

Comme indiqué dans le diagramme ci-dessus, lorsque vous utilisez l’intégration RBAC Azure, toutes les requêtes envoyées à l’API Kubernetes suivent le même processus d’authentification, comme expliqué dans la [section Intégration d’Azure Active](#azure-active-directory-integration). 

Mais après cela, au lieu de s’appuyer uniquement sur le RBAC Kubernetes pour l’autorisation, la requête est en fait autorisée par Azure, à condition que l’identité qui a fait la demande existe dans AAD. Si l’identité n’existe pas dans AAD, par exemple un compte de service Kubernetes, le RBAC Azure ne démarrera pas et il s’agira du RBAC Kubernetes normal.

Dans ce scénario, vous pouvez donner aux utilisateurs l’un des quatre rôles intégrés, ou créer des rôles personnalisés comme vous le feriez avec les rôles Kubernetes, mais dans ce cas, à l’aide des mécanismes et des API Azure RBAC. 

Cette fonctionnalité vous permet, par exemple, de fournir non seulement aux utilisateurs des autorisations d’accès à la ressource AKS sur tous les abonnements, mais aussi de les configurer et de leur attribuer le rôle et les autorisations qu’ils auront dans chacun de ces clusters qui contrôlent l’accès à l’API Kubernetes. Par exemple, vous pouvez accorder le rôle `Azure Kubernetes Service RBAC Viewer` sur l’étendue de l’abonnement et son destinataire peut répertorier et obtenir tous les objets Kubernetes de tous les clusters, mais pas les modifier.


#### <a name="built-in-roles"></a>Rôles intégrés

AKS propose quatre rôles intégrés. Ils sont similaires aux rôles intégrés [Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/#user-facing-roles) mais avec quelques différences comme la prise en charge des CRD. Pour obtenir la liste complète des actions autorisées par chaque rôle intégré, consultez [ici](../role-based-access-control/built-in-roles.md).

| Role                                | Description  |
|-------------------------------------|--------------|
| Azure Kubernetes Service RBAC Viewer  | Autorise l’accès en lecture seule pour voir la plupart des objets dans un espace de noms. Il n’autorise pas l’affichage des rôles ou des liaisons de rôles. Ce rôle n’autorise pas l’affichage des `Secrets`, car la lecture du contenu de secrets permet d’accéder aux informations d’identification `ServiceAccount` dans l’espace de noms, ce qui permet l’accès aux API comme n’importe quel `ServiceAccount` dans l’espace de noms (une forme d’élévation de privilèges)  |
| Azure  Kubernetes Service RBAC Writer | Autorise l’accès en lecture/écriture pour la plupart des objets dans un espace de noms. Ce rôle n’autorise pas l’affichage ni la modification des rôles et des liaisons de rôles. Toutefois, il permet d’accéder à `Secrets` et d’exécuter des pods comme tout ServiceAccount dans l’espace de noms. Il peut donc être utilisé pour obtenir les niveaux d’accès API de n’importe quel ServiceAccount dans l’espace de noms. |
| Azure Kubernetes Service RBAC Admin  | Autorise l’accès administrateur, normalement accordé au sein d’un espace de noms. Autorise l’accès en lecture/écriture à la plupart des ressources dans un espace de noms (ou dans l’étendue du cluster), y compris la possibilité de créer des rôles et des liaisons de rôles dans l’espace de noms. Ce rôle n’autorise pas l’accès en écriture au quota de ressources ou à l’espace de noms lui-même. |
| Azure Kubernetes Service RBAC Cluster Admin  | Autorise l’accès de super utilisateur qui permet d’effectuer n’importe quelle action sur toutes les ressources. Ce rôle donne un contrôle total sur l’ensemble des ressources dans le cluster et dans tous les espaces de noms. |

**Pour savoir comment activer Azure RBAC pour l’autorisation Kubernetes [lisez cet article](manage-azure-rbac.md).**

## <a name="summary"></a>Résumé

Ce tableau récapitule les méthodes permettant aux utilisateurs de s’authentifier auprès de Kubernetes lorsque l’intégration Azure AD est activée.  Dans tous les cas, la séquence de commandes de l’utilisateur est la suivante :
1. Exécuter `az login` pour s’authentifier sur Azure.
1. Exécuter `az aks get-credentials` pour télécharger les informations d’identification du cluster dans `.kube/config`.
1. Exécuter les commandes `kubectl` (la première peut déclencher l’authentification basée sur le navigateur pour s’authentifier auprès du cluster, comme décrit dans le tableau suivant).

L’attribution de rôle dont il est question dans la deuxième colonne correspond à l’octroi de rôle Azure RBAC indiqué sous l’onglet **Contrôle d’accès** du portail Azure. Le groupe Azure AD administrateur de cluster est affiché sous l’onglet **Configuration** dans le portail (ou avec le nom de paramètre `--aad-admin-group-object-ids` dans Azure CLI).

| Description        | Octroi de rôle requis| Groupe(s) Azure AD administrateur de cluster | Quand l’utiliser |
| -------------------|------------|----------------------------|-------------|
| Connexion administrateur héritée à l’aide d’un certificat client| **Rôle administrateur Azure Kubernetes**. Ce rôle permet l’utilisation de `az aks get-credentials` avec l’indicateur `--admin`, ce qui télécharge un [certificat administrateur de cluster hérité (non-Azure AD)](control-kubeconfig-access.md) dans le `.kube/config` de l’utilisateur. Il s’agit du seul objectif du rôle administrateur Azure Kubernetes.|n/a|Si vous êtes bloqué et n’avez pas accès à un groupe Azure AD valide avec accès à votre cluster.| 
| Azure AD avec liaisons (cluster)RoleBindings manuelles| **Rôle d’utilisateur Azure Kubernetes**. Le rôle d’utilisateur permet l’utilisation de `az aks get-credentials` sans indicateur `--admin`. (Il s’agit de l’unique objectif du rôle d’utilisateur Azure Kubernetes.) Ainsi, un cluster Azure AD peut télécharger [une entrée vide](control-kubeconfig-access.md) dans `.kube/config`, ce qui déclenche l’authentification basée sur le navigateur lorsqu’il est utilisé pour la première fois par `kubectl`.| L’utilisateur ne se trouve dans aucun de ces groupes. L’utilisateur ne se trouvant dans aucun groupe d’administration de cluster, ses droits sont entièrement contrôlés par n’importe quelle liaison RoleBindings ou ClusterRoleBindings configurée par les administrateurs de cluster. Les liaisons (cluster)RoleBindings [désignent les utilisateurs ou groupes Azure AD](azure-ad-rbac.md) en tant que `subjects`. Si aucune de ces liaisons n’a été configurée, l’utilisateur ne sera pas en mesure d’exécuter de commandes `kubectl`.|Si vous souhaitez un contrôle d’accès affiné et que vous n’utilisez pas Azure RBAC pour l’autorisation Kubernetes. Notez que l’utilisateur qui configure les liaisons doit se connecter à l’aide d’une des autres méthodes répertoriées dans ce tableau.|
| Azure AD par membre du groupe d’administration| Identique à ce qui précède|L’utilisateur est membre d’un des groupes répertoriés ici. AKS génère automatiquement une liaison ClusterRoleBinding qui lie tous les groupes répertoriés au rôle de Kubernetes `cluster-admin`. Ainsi, les utilisateurs de ces groupes peuvent exécuter toutes les commandes `kubectl` comme `cluster-admin`.|Si vous souhaitez accorder aux utilisateurs des droits d’administrateur complets et que vous n’utilisez _pas_ Azure RBAC pour l’autorisation Kubernetes.|
| Azure AD avec Azure RBAC pour l’autorisation Kubernetes|Deux rôles : Premièrement, **rôle d’utilisateur Azure Kubernetes** (comme ci-dessus). Deuxièmement, l’un des rôles «Azure Kubernetes Service **RBAC**... » répertoriés ci-dessus, ou votre propre alternative personnalisée.|Le champ des rôles d’administrateur de l’onglet Configuration n’est pas pertinent lorsqu’Azure RBAC pour l’autorisation Kubernetes est activé.|Vous utilisez Azure RBAC pour l’autorisation Kubernetes. Cette approche vous offre un contrôle affiné, sans avoir à configurer de liaisons RoleBindings ou ClusterRoleBindings.|

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec RBAC Azure AD et Kubernetes, consultez [Intégrer Azure Active Directory à AKS][aks-aad].
- Pour plus d'informations sur les bonnes pratiques, consultez [Bonnes pratiques relatives à l'authentification et à l'autorisation dans AKS][operator-best-practices-identity].
- Pour vous familiariser avec l’autorisation Azure RBAC pour Kubernetes, consultez [utilisez Azure RBAC pour autoriser l’accès au sein du cluster Azure Kubernetes Service (AKS)](manage-azure-rbac.md).
- Pour commencer à sécuriser votre fichier kubeconfig, consultez [Limiter l’accès au fichier de configuration de cluster](control-kubeconfig-access.md)

Pour plus d’informations sur les concepts fondamentaux de Kubernetes et d’AKS, consultez les articles suivants :

- [Clusters et charges de travail Kubernetes/AKS][aks-concepts-clusters-workloads]
- [Sécurité Kubernetes/AKS][aks-concepts-security]
- [Réseaux virtuels Kubernetes/AKS][aks-concepts-network]
- [Stockage Kubernetes/AKS][aks-concepts-storage]
- [Mise à l’échelle Kubernetes/AKS][aks-concepts-scale]

<!-- LINKS - External -->
[kubernetes-authentication]: https://kubernetes.io/docs/reference/access-authn-authz/authentication
[webhook-token-docs]: https://kubernetes.io/docs/reference/access-authn-authz/authentication/#webhook-token-authentication
[kubernetes-rbac]: https://kubernetes.io/docs/reference/access-authn-authz/rbac/

<!-- LINKS - Internal -->
[openid-connect]: ../active-directory/develop/v2-protocols-oidc.md
[az-aks-get-credentials]: /cli/azure/aks#az-aks-get-credentials
[azure-rbac]: ../role-based-access-control/overview.md
[aks-aad]: managed-aad.md
[aks-concepts-clusters-workloads]: concepts-clusters-workloads.md
[aks-concepts-security]: concepts-security.md
[aks-concepts-scale]: concepts-scale.md
[aks-concepts-storage]: concepts-storage.md
[aks-concepts-network]: concepts-network.md
[operator-best-practices-identity]: operator-best-practices-identity.md

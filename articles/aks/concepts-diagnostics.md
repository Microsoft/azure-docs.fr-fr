---
title: Vue d’ensemble des diagnostics Azure Kubernetes Service (AKS)
description: Découvrez les clusters d'auto-diagnostic dans Azure Kubernetes Service.
services: container-service
author: yunjchoi
ms.topic: conceptual
ms.date: 11/04/2019
ms.author: yunjchoi
ms.openlocfilehash: e8921152177d3e4534ca9fb48cf209aed6e1b27c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183360"
---
# <a name="azure-kubernetes-service-diagnostics-preview-overview"></a>Vue d’ensemble des diagnostics d’Azure Kubernetes Service

La résolution des problèmes de cluster Azure Kubernetes Service (AKS) joue un rôle important en termes de gestion de votre cluster, notamment si celui-ci exécute des charges de travail stratégiques. AKS Diagnostics offre une expérience d'auto-diagnostic intelligente qui vous permet d’identifier et de résoudre les problèmes de votre cluster. Natif du cloud, AKS Diagnostics peut être utilisé sans configuration ni coût de facturation supplémentaires.

Cette fonctionnalité est désormais disponible en préversion publique.

## <a name="open-aks-diagnostics"></a>Ouvrir AKS Diagnostics

Pour accéder à AKS Diagnostics :

- Accédez à votre cluster Kubernetes dans le [portail Azure](https://portal.azure.com).
- Dans le volet de navigation de gauche, cliquez sur **Diagnostiquer et résoudre les problèmes** pour ouvrir AKS Diagnostics.
- Choisissez la catégorie la plus adaptée au problème de votre cluster en utilisant les mots clés dans la vignette de la page d’accueil ou entrez un mot clé décrivant votre problème dans la barre de recherche, par exemple _Problèmes de nœud de cluster_.

![Page d’accueil](./media/concepts-diagnostics/aks-diagnostics-homepage.png)

## <a name="view-a-diagnostic-report"></a>Afficher un rapport de diagnostic

Après avoir cliqué sur une catégorie, vous pouvez afficher un rapport de diagnostic spécifique à votre cluster. Le rapport de diagnostic détaille intelligemment tout problème éventuel dans votre cluster à l'aide d'icônes d'état. Vous pouvez accéder à chaque rubrique en cliquant sur **Informations supplémentaires** afin d'obtenir une description détaillée du problème, des actions recommandées, des liens vers des documents utiles, des métriques liées, ainsi que des données de journalisation. Les rapports de diagnostic sont générés de façon intelligente en fonction de l’état actuel de votre cluster après exécution de nombreuses vérifications. En outre, les rapports de diagnostic permettent d'identifier le problème de votre cluster et de trouver la procédure à suivre pour y remédier.

![Rapport de diagnostic](./media/concepts-diagnostics/diagnostic-report.png)

![Rapport de diagnostic développé](./media/concepts-diagnostics/node-issues.png)

## <a name="cluster-insights"></a>Cluster Insights (Insights de cluster)

Les vérifications de diagnostic suivantes sont disponibles dans **Insights de cluster**.

### <a name="cluster-node-issues"></a>Cluster Node Issues

Cluster Node Issues (Problèmes de nœud de cluster) identifie les éventuels problèmes de nœud susceptibles d'entraîner un comportement inattendu de votre cluster.

- Problèmes de préparation de nœud
- Échecs de nœud
- Ressources insuffisantes
- Configuration IP de nœud manquante
- Échecs CNI de nœud
- Nœud introuvable
- Nœud hors tension
- Échec d'authentification de nœud
- Proxy Kube de nœud obsolète

### <a name="create-read-update--delete-operations"></a>Create, Read, Update & Delete Operations (Opérations create, read, update et delete)

CRUD Operations vérifie les opérations CRUD susceptibles d'entraîner des problèmes dans votre cluster.

- Erreur de l'opération de suppression de sous-réseau en cours d’utilisation
- État de l'opération de suppression du groupe de sécurité réseau
- Erreur de l’opération de suppression de la table de route en cours d’utilisation
- Erreur d'approvisionnement des ressources référencées
- Erreur de l’opération de suppression d'adresse IP publique
- Échec de déploiement en raison du quota de déploiement
- Erreur d’opération en raison de la stratégie de l’organisation
- Inscription de l'abonnement manquante
- Échec d'approvisionnement d’une extension de machine virtuelle
- Capacité du sous-réseau
- Erreur de dépassement de quota

### <a name="identity-and-security-management"></a>Gestion de la sécurité et identité

La gestion de la sécurité et des identités détecte les erreurs d’authentification et d’autorisation susceptibles d’empêcher la communication avec votre cluster.

- Échecs d'autorisation de nœud
- Erreurs 401
- Erreurs 403

## <a name="next-steps"></a>Étapes suivantes

Collectez les journaux pour faciliter la résolution des problèmes de votre cluster à l’aide d'[AKS Periscope](https://aka.ms/aksperiscope).

Consultez la [section relative aux pratiques de triage](/azure/architecture/operator-guides/aks/aks-triage-practices) du Guide des opérations AKS (2e journée).

Publiez vos questions ou vos commentaires sur [UserVoice](https://feedback.azure.com/forums/914020-azure-kubernetes-service-aks) en ajoutant « [Diag] » dans le titre.
---
title: Introduction à Azure Red Hat OpenShift
description: Découvrez les fonctionnalités et les avantages de Microsoft Azure Red Hat OpenShift pour déployer et gérer des applications basées sur des conteneurs.
author: jimzim
ms.author: jzim
ms.service: container-service
ms.topic: overview
ms.date: 11/13/2020
ms.custom: mvc
ms.openlocfilehash: 1bf3141876ee56ee1361f19a67689ca3b2f4f89a
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94685288"
---
# <a name="azure-red-hat-openshift"></a>Azure Red Hat OpenShift

Le service Microsoft *Azure Red Hat OpenShift* vous permet de déployer des clusters [OpenShift](https://www.openshift.com/) entièrement gérés.

Azure Red Hat OpenShift est une extension de [Kubernetes](https://kubernetes.io/). L’exécution de conteneurs en production avec Kubernetes nécessite des outils et des ressources supplémentaires. Cela implique souvent de jongler avec les registres d’images, la gestion du stockage, les solutions de mise en réseau et les outils de journalisation et de supervision, qui doivent tous être versionnés et testés ensemble. La création d’applications basées sur des conteneurs requiert encore plus de travail d’intégration avec des intergiciels, des infrastructures, des bases de données et des outils CI/CD. Azure Red Hat OpenShift combine tout cela dans une seule plateforme, facilitant les opérations des équipes informatiques tout en donnant aux équipes d’application ce dont elles ont besoin pour exécuter.

Azure Red Hat OpenShift est conçue, exploitée et prise en charge conjointement par Red Hat et par Microsoft pour fournir une expérience de support intégré. Il n’y a aucune machine virtuelle à faire fonctionner et aucune mise à jour corrective n’est nécessaire. Les nœuds principaux, d’infrastructure et d’application sont corrigés, mis à jour et supervisés en votre nom par Red Hat et Microsoft. Vos clusters Azure Red Hat OpenShift sont déployés dans votre abonnement Azure et inclus sur votre facture Azure.

Vous pouvez choisir vos propres solutions de Registre, de mise en réseau, de stockage et CI/CD, ou utiliser les solutions intégrées pour la gestion automatisée du code source, les builds de conteneurs et d’applications, les déploiements, la mise à l’échelle, la gestion de l’intégrité et bien plus encore. Azure Red Hat OpenShift fournit une expérience d’authentification intégrée via Azure Active Directory.

Pour commencer, suivez le tutoriel [Créer un cluster Azure Red Hat OpenShift](tutorial-create-cluster.md).

## <a name="access-security-and-monitoring"></a>Accès, sécurité et surveillance

Pour améliorer la sécurité et la gestion, Azure Red Hat OpenShift vous permet d’intégrer Azure Active Directory (Azure AD) et d’utiliser le contrôle d’accès en fonction du rôle Kubernetes (contrôle RBAC Kubernetes). Vous pouvez également superviser l’intégrité de votre cluster et de vos ressources.

## <a name="cluster-and-node"></a>Cluster et nœud

Les nœuds Azure Red Hat OpenShift s’exécutent sur des machines virtuelles Azure. Vous pouvez connecter le stockage à des nœuds et des pods et mettre à niveau les composants du cluster.

## <a name="service-level-agreement"></a>Contrat de niveau de service

Azure Red Hat OpenShift offre un contrat de niveau de service pour garantir que le service sera disponible au moins 99,95 % du temps. Pour plus d’informations sur le contrat de niveau de service, consultez [SLA pour Azure Red Hat OpenShift](https://azure.microsoft.com/en-au/support/legal/sla/openshift/v1_0/).

## <a name="next-steps"></a>Étapes suivantes

Découvrez les conditions préalables pour Azure Red Hat OpenShift :

> [!div class="nextstepaction"]
> [Configurer votre environnement de développement](tutorial-create-cluster.md)

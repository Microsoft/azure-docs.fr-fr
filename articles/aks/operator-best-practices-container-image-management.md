---
title: Meilleures pratiques opérateur - Gestion des images de conteneur dans Azure Kubernetes Services (AKS)
description: Découvrez les meilleures pratiques qui s’appliquent pour l’opérateur de cluster concernant la gestion et la sécurisation des images de conteneur dans Azure Kubernetes Service (ACS)
services: container-service
ms.topic: conceptual
ms.date: 12/06/2018
ms.openlocfilehash: 4d90ac5b8ab44925b73be2f22cd7a79458da9f5a
ms.sourcegitcommit: a92fbc09b859941ed64128db6ff72b7a7bcec6ab
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92073549"
---
# <a name="best-practices-for-container-image-management-and-security-in-azure-kubernetes-service-aks"></a>Meilleures pratiques relatives à la gestion et à la sécurisation des images de conteneur dans Azure Kubernetes Services (AKS)

Lorsque vous développez et exécutez des applications dans Azure Kubernetes Service (ACS), la sécurité de vos conteneurs et de vos images de conteneur est primordiale. Les conteneurs qui incluent des images de base obsolètes ou des runtimes d’application non corrigés constituent un risque de sécurité et un possible vecteur d’attaque. Pour réduire ces risques, vous devez intégrer des outils capables de rechercher et de résoudre les problèmes dans vos conteneurs au moment de la génération aussi bien que de l’exécution. Plus la vulnérabilité ou l’image de base obsolète est identifiée tôt dans le processus, plus le cluster est sécurisé. Dans cet article, le terme *conteneurs* se réfère à la fois aux images de conteneur stockées dans un Registre de conteneurs, et aux conteneurs en cours d’exécution.

Cet article est dédié à la sécurisation de votre conteneur dans AKS. Vous allez apprendre à effectuer les actions suivantes :

> [!div class="checklist"]
> * Rechercher et corriger les vulnérabilités de l’image
> * Déclencher et redéployer automatiquement les images de conteneur lors de la mise à jour d’une image de base

Vous pouvez également consulter les bonnes pratiques relatives à la [sécurité des clusters][best-practices-cluster-security] et à la [sécurité du pod][best-practices-pod-security].

Vous pouvez également utiliser [la sécurité du conteneur dans Security Center][security-center-containers] pour vous aider à analyser les vulnérabilités de vos conteneurs.  Il existe également [l’intégration du registre de conteneurs Azure][security-center-acr] avec Security Center pour vous aider à protéger vos images et votre registre contre les vulnérabilités.

## <a name="secure-the-images-and-run-time"></a>Sécuriser les images et le runtime

**Meilleures pratiques** : analysez vos images de conteneur à la recherche de vulnérabilités et déployez uniquement les images ayant réussi la validation. Mettez régulièrement à jour les images de base et le runtime de l’application, puis redéployez les charges de travail dans le cluster AKS.

Un problème avec l’adoption de charges de travail basées sur les conteneurs concerne la vérification de la sécurité des images et du runtime utilisés pour générer vos propres applications. Comment vous assurer que vous n’introduisez pas de failles de sécurité dans vos déploiements ? Votre flux de travail de déploiement doit inclure un processus d’analyse des images conteneur s’appuyant sur des outils tels que [Twistlock][twistlock] ou [Aqua][aqua], puis autoriser uniquement les images vérifiées à être déployées.

![Analyser et corriger les images de conteneur, les valider et les déployer](media/operator-best-practices-container-security/scan-container-images-simplified.png)

Dans un exemple concret, vous pouvez utiliser un pipeline d’intégration continue et de déploiement continu (CI/CD) pour automatiser les analyses, la vérification et les déploiements d’image. Azure Container Registry inclut ces fonctions d’analyse de vulnérabilités.

## <a name="automatically-build-new-images-on-base-image-update"></a>Générer automatiquement les nouvelles images sur la mise à jour de l’image de base

**Meilleures pratiques** : comme vous utilisez des images de base pour les images de l’application, utilisez l’automatisation pour générer de nouvelles images lorsque l’image de base est mise à jour. Comme ces images de base incluent généralement des correctifs de sécurité, mettez à jour les images de conteneur d’application en aval.

Chaque fois qu’une image de base est mis à jour, les images de conteneur en aval doivent également être mises à jour. Ce processus de génération doit être intégré aux pipelines de validation et de déploiement comme [Azure Pipelines][azure-pipelines] ou Jenkins. Ces pipelines permettent de s’assurer que vos applications continuent à s’exécuter sur les images de base mises à jour. Une fois que vos images de conteneur d’application sont validées, les déploiements AKS peuvent ensuite être mis à jour pour exécuter les images les plus récentes et les plus sécurisées.

Azure Container Registry Tasks peut également mettre à jour automatiquement les images de conteneur lorsque l’image de base est mise à jour. Cette fonctionnalité vous permet de générer un petit nombre d’images de base et de les mettre régulièrement à jour avec les correctifs de bogues et de sécurité.

Pour plus d’informations sur la mise à jour des images de base, consultez [Automatiser la génération des images en fonction de la mise à jour d’une image de base avec Azure Container Registry Tasks][acr-base-image-update].

## <a name="next-steps"></a>Étapes suivantes

Cet article était dédié à la sécurisation de vos conteneurs. Pour implémenter quelques-unes de ces pratiques, consultez les articles suivants :

* [Automatiser la génération des images en fonction de la mise à jour d’une image de base avec Azure Container Registry Tasks][acr-base-image-update]

<!-- EXTERNAL LINKS -->
[azure-pipelines]: /azure/devops/pipelines/?view=vsts
[twistlock]: https://www.twistlock.com/
[aqua]: https://www.aquasec.com/

<!-- INTERNAL LINKS -->
[best-practices-cluster-security]: operator-best-practices-cluster-security.md
[best-practices-pod-security]: developer-best-practices-pod-security.md
[acr-base-image-update]: ../container-registry/container-registry-tutorial-base-image-update.md
[security-center-containers]: ../security-center/container-security.md
[security-center-acr]: ../security-center/defender-for-container-registries-introduction.md
---
title: Informations sur les serveurs configuration/processus/cible maître Azure Site Recovery
description: Cet article offre une vue d’ensemble sur l’utilisation des serveurs de configuration, des serveurs de processus et des serveurs cible maîtres lors de la configuration de la récupération d’urgence de machines virtuelles VMware locales sur Azure avec Azure Site Recovery
ms.topic: conceptual
ms.date: 03/17/2020
ms.openlocfilehash: cd5ded18d1a8f1f5fd96212d37725bb5db13002f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80062092"
---
# <a name="about-site-recovery-components-configuration-process-master-target"></a>Informations sur les composants Site Recovery (configuration, processus, cible maître)

Cet article décrit les serveurs de configuration, les serveurs de processus et les serveurs cibles maîtres utilisés par le service [Site Recovery](site-recovery-overview.md) pour répliquer les machines virtuelles VMware et les serveurs physiques sur Azure.

## <a name="configuration-server"></a>Serveur de configuration

Pour la reprise d’activité des machines virtuelles VMware locales et des serveurs physiques, déployez un serveur local de configuration Site Recovery.

**Paramètre** | **Détails** | **Liens**
--- | --- | ---
**Composants**  | La machine serveur de configuration exécute tous les composants locaux de Site Recovery, y compris le serveur de configuration, le serveur de processus et le serveur cible maître.<br/><br/> Lorsque vous configurez le serveur de configuration, tous les composants sont installés automatiquement. | [Lisez](vmware-azure-common-questions.md#configuration-server) le Forum aux questions sur le serveur de configuration.
**Rôle** | Le serveur de configuration coordonne la communication entre les ordinateurs locaux et Azure.et gère la réplication des données. | Découvrez l’architecture de la récupération d’urgence des [machines virtuelles VMware](vmware-azure-architecture.md) et des [serveurs physiques](physical-azure-architecture.md) dans Azure.
**Conditions requises VMware** | Pour la récupération d’urgence des machines virtuelles VMware locales, vous devez installer et exécuter le serveur de configuration en tant que machine virtuelle VMware locale hautement disponible. | [En savoir plus sur](vmware-azure-deploy-configuration-server.md#prerequisites) les conditions préalables.
**Déploiement de VMware** | Nous vous recommandons de déployer le serveur de configuration en utilisant un modèle OVA téléchargé. Cette méthode permet de configurer simplement un serveur de configuration qui répond à toutes les exigences et conditions préalables.<br/><br/> Si, pour une raison quelconque, vous ne parvenez pas à déployer une machine virtuelle VMware à l’aide d’un modèle OVA, vous pouvez configurer manuellement les machines du serveur de configuration, comme décrit ci-dessous pour la récupération d’urgence des machines physiques. | [Déployez](vmware-azure-deploy-configuration-server.md#deploy-a-configuration-server-through-an-ova-template) un modèle OVA.
**Exigences des serveurs physiques** | Pour la récupération d’urgence sur des serveurs physiques locaux, vous déployez le serveur de configuration manuellement. | [En savoir plus sur](physical-azure-set-up-source.md#prerequisites) les conditions préalables.
**Déploiement de serveur physique** | S’il ne peut pas être installé en tant que machine virtuelle VMware, vous pouvez l’installer sur un serveur physique. | [Déployez](physical-azure-set-up-source.md#set-up-the-source-environment) le serveur de configuration manuellement.

## <a name="process-server"></a>Serveur de traitement

Un serveur de processus gère les données de réplication pendant le basculement et la restauration automatique, et installe le service Mobilité pour les machines virtuelles VMware et les serveurs physiques locaux.

**Paramètre** | **Détails** | **Liens**
--- | --- | ---
**Déploiement**  | Par défaut, le serveur de processus est installé lorsque le serveur de configuration est déployé. <br/><br/> Un serveur de processus local est requis pour la reprise d’activité et la réplication des machines virtuelles VMware et des serveurs physiques locaux. | [Plus d’informations](vmware-azure-architecture.md#architectural-components)
**Rôle (local**) | Reçoit les données de réplication des machines activées pour la réplication. <br/><br/> Optimise les données de réplication avec la mise en cache, la compression et le chiffrement, et les envoie vers le stockage Azure. <br/><br/> Effectue une installation Push du service Mobilité Site Recovery sur les machines virtuelles VMware et les serveurs physiques locaux que vous voulez répliquer. <br/><br/> Effectue la découverte automatique des machines locales. | [Plus d’informations](vmware-azure-enable-replication.md)
**Rôle (restauration automatique à partir de Microsoft Azure)** | Après le basculement à partir de votre site local, vous configurez un serveur de processus dans Azure, en tant que machine virtuelle Azure, pour gérer le basculement vers votre emplacement local.<br/><br/> Dans Azure, le serveur de traitement est temporaire. La machine virtuelle Azure peut être supprimée à l’issue de la restauration automatique. | [Plus d’informations](vmware-azure-set-up-process-server-azure.md)
**Mise à l'échelle** | Pour des déploiements plus importants, vous pouvez configurer sur site des serveurs de processus supplémentaires et évolutifs. Des serveurs supplémentaires effectuent un scale-out de la capacité en gérant un plus grand nombre de machines de réplication et un plus grand volume de trafic de réplication.<br/><br/> Vous pouvez déplacer les machines entre deux serveurs de processus afin d’équilibrer le trafic de réplication. | [Plus d’informations](vmware-azure-set-up-process-server-scale.md)

## <a name="master-target-server"></a>Serveur cible maître

Le serveur cible maître gère les données de réplication pendant la restauration automatique à partir d’Azure.

- Par défaut, le serveur cible maître est installé sur le serveur de configuration.
- Pour les déploiements à grande échelle, vous pouvez ajouter un serveur cible maître distinct à des fins de restauration automatique.

## <a name="next-steps"></a>Étapes suivantes

- Examinez [l’architecture](vmware-azure-architecture.md) de la récupération d’urgence des machines virtuelles VMware et des serveurs physiques.
- Passez en revue les [exigences et les prérequis](vmware-physical-azure-support-matrix.md) de la récupération d’urgence des machines virtuelles VMware et des serveurs physiques vers Azure.

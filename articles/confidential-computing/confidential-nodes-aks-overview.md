---
title: Nœuds d’informatique confidentielle sur Azure Kubernetes Service (AKS) - Préversion publique
description: Nœuds d’informatique confidentielle sur AKS
services: virtual-machines
author: agowdamsft
ms.service: container-service
ms.topic: overview
ms.date: 9/22/2020
ms.author: amgowda
ms.openlocfilehash: 1b945ac9f656a227bcc3335cb0ec995626f98f77
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94564172"
---
# <a name="confidential-computing-nodes-on-azure-kubernetes-service-public-preview"></a>Nœuds d’informatique confidentielle sur Azure Kubernetes Service (préversion publique)

L’[informatique confidentielle Azure](overview.md) vous permet de protéger vos données sensibles pendant leur utilisation. Les infrastructures sous-jacentes protègent ces données des autres applications, administrateurs et fournisseurs de cloud avec un environnement de conteneur d’exécution de confiance basé sur le matériel.

## <a name="overview"></a>Vue d’ensemble

Azure Kubernetes Service (AKS) prend en charge l’ajout de [nœuds d’informatique confidentielle DCsv2](confidential-computing-enclaves.md) reposant sur la technologie Intel SGX. Ces nœuds peuvent exécuter des charges de travail sensibles au sein d’un environnement d’exécution de confiance (l’environnement TEE) basé sur le matériel en permettant au code utilisateur d’allouer des régions privées de mémoire. Ces régions de mémoire privées sont appelées enclaves. Les enclaves sont conçues pour protéger le code et les données des processus qui s’exécutent avec des privilèges plus élevés. Le modèle d’exécution SGX supprime les couches intermédiaires du système d’exploitation invité, du système d’exploitation hôte et de l’hyperviseur. Le modèle d’*exécution isolée par conteneur basée sur le matériel* permet aux applications de s’exécuter directement avec le processeur, tout en conservant le bloc spécial de mémoire chiffré. Les nœuds d’informatique confidentielle favorisent la posture de sécurité globale des applications conteneur sur AKS et renforcent sensiblement la stratégie de conteneur de défense en profondeur. 

![vue d’ensemble d’un nœud SGX](./media/confidential-nodes-aks-overview/sgxaksnode.jpg)

## <a name="aks-confidential-nodes-features"></a>Fonctionnalités des nœuds confidentiels AKS

- Isolation des conteneurs au niveau des processus, basée sur le matériel, par le biais de l’environnement TEE SGX 
- Clusters de pools de nœuds hétérogènes (combinaison de pools de nœuds confidentiels et non confidentiels)
- Planification de pods basée sur la mémoire EPC (Encrypted Page Cache)
- Pilote SGX DCAP préinstallé
- Patch Intel FSGS préinstallé
- Prise en charge de la mise à l’échelle automatique des clusters et de la mise à l’échelle automatique des pods horizontaux en fonction de la consommation processeur
- Helper d’attestation hors processus via le DaemonSet AKS
- Prise en charge des conteneurs Linux par le biais de nœuds Worker sur les machines virtuelles Ubuntu 18.04 de deuxième génération

## <a name="aks-provided-daemon-sets-addon"></a>Ensembles de démons fournis par AKS (module complémentaire)

#### <a name="sgx-device-plugin"></a>SGX Device Plugin <a id="sgx-plugin"></a>

SGX Device Plugin implémente l’interface du plug-in d’appareil Kubernetes pour la mémoire EPC. Concrètement, ce plug-in fait de la mémoire EPC un type de ressource supplémentaire dans Kubernetes. Les utilisateurs peuvent spécifier des limites sur cette ressource de la même façon que sur les autres ressources. Outre la fonction de planification, le plug-in d’appareil facilite l’attribution d’autorisations de pilote d’appareil SGX aux conteneurs de charges de travail confidentiels. Un exemple d’implémentation d’un déploiement basé sur la mémoire EPC (`kubernetes.azure.com/sgx_epc_mem_in_MiB`) est disponible [ici](https://github.com/Azure-Samples/confidential-computing/blob/main/containersamples/helloworld/helm/templates/helloworld.yaml)

#### <a name="sgx-quote-helper-service"></a>Service SGX Quote Helper <a id="sgx-quote"></a>

Les applications de l’enclave qui effectuent une attestation à distance doivent générer une déclaration, appelée QUOTE. La QUOTE fournit une preuve cryptographique de l’identité et de l’état de l’application, et de l’environnement où s’exécute l’enclave. La génération d’une QUOTE repose sur certains composants logiciels de confiance d’Intel, qui font partie des composants logiciels de la plateforme SGX (PSW/DCAP). Ce PSW est inclus sous la forme d’un ensemble de démons (un DaemonSet) qui s’exécute sur chaque nœud. Il peut être utilisé au moment de la demande d’une QUOTE d’attestation par des applications enclavées. L’utilisation du service fourni par AKS aide à mieux assurer la compatibilité entre le PSW et les autres composants logiciels sur l’hôte. [Découvrez-en plus](confidential-nodes-out-of-proc-attestation.md) sur son utilisation et ses fonctionnalités.

## <a name="programming--application-models"></a>Programmation et modèles d’application

### <a name="confidential-containers"></a>Conteneurs confidentiels

Les [conteneurs confidentiels](confidential-containers.md) exécutent les programmes existants et la plupart des Runtime des **langages de programmation courants** (Python, Node, Java, etc.), avec leurs dépendances de bibliothèque existantes, sans modification ou recompilation du code source. Ce modèle est le modèle le plus rapide pour implémenter la confidentialité par les projets open source et les partenaires Azure. Les images conteneur qui sont prêtes à être exécutées dans les enclaves sécurisées sont appelées conteneurs confidentiels.

### <a name="enclave-aware-containers"></a>Conteneurs reconnaissant les enclaves

AKS prend en charge les applications qui sont développées pour s’exécuter sur des nœuds confidentiels et utiliser un **ensemble d’instructions spéciales** exposées par les SDK et les frameworks. Ce modèle d’application offre un contrôle maximal de vos applications avec une base TCB (Trusted Computing Base) réduite. [Découvrez-en plus](enclave-aware-containers.md) sur les conteneurs reconnaissant les enclaves.

## <a name="next-steps"></a>Étapes suivantes

[Déployer un cluster AKS avec des nœuds d’informatique confidentielle](./confidential-nodes-aks-get-started.md)

[Démarrer rapidement avec des exemples de conteneurs confidentiels](https://github.com/Azure-Samples/confidential-container-samples)

[Liste des références SKU DCsv2](../virtual-machines/dcv2-series.md)

<!-- LINKS - external -->
[Azure Attestation]: ../attestation/index.yml


<!-- LINKS - internal -->
[DC Virtual Machine]: /confidential-computing/virtual-machine-solutions
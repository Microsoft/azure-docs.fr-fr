---
title: Forum aux questions concernant la prise en charge des nœuds confidentiels sur Azure Kubernetes service (AKS)
description: Trouvez des réponses à certaines des questions les plus fréquentes concernant Azure Container Service (AKS) et la prise en charge des nœuds d’informatique confidentielle Azure (ACC).
author: agowdamsft
ms.service: container-service
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: amgowda
ms.openlocfilehash: b4b6c04a1e35031387a006b6350ebf6cbe502fee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90993023"
---
# <a name="frequently-asked-questions-about-confidential-computing-nodes-on-azure-kubernetes-service-aks"></a>Forum aux questions concernant les nœuds d’informatique confidentielle sur Azure Kubernetes service (AKS)

Cet article répond à des questions fréquentes sur les nœuds d’informatique confidentielle basés sur Intel SGX sur Azure Kubernetes service (AKS). Si vous avez d’autres questions, envoyez un e-mail à l’adresse acconaks@microsoft.com.

## <a name="what-service-level-agreement-sla-and-azure-support-is-provided-during-the-preview"></a>Quels sont le Contrat de niveau de service (SLA) et le Support Azure fournis pendant la période de préversion ? 

Aucun SLA n’est fourni durant la période de préversion du produit, telle que définie [ici](https://azure.microsoft.com/support/legal/preview-supplemental-terms/). En revanche, un support technique est assuré via le Support Azure.

## <a name="what-is-attestation-and-how-can-we-do-attestation-of-apps-running-in-enclaves"></a>Qu’est-ce qu’une attestation et comment pouvons-nous délivrer une attestation d’applications s’exécutant dans des enclaves ? 

Une attestation est un processus consistant à démontrer et à valider qu’un élément de logiciel a été correctement instancié sur la plateforme matérielle spécifique. Elle garantit également l’existence de preuves vérifiables garantissant qu’elle s’exécute dans une plateforme sécurisée et qu’elle n’a pas été falsifiée. [Apprenez-en davantage](attestation.md) sur la façon dont une attestation est délivrée pour des applications d’enclave.

## <a name="can-i-bring-my-existing-containerized-applications-and-run-it-on-aks-with-azure-confidential-computing"></a>Puis-je venir avec mes applications en conteneur existantes et les exécuter sur AKS avec l’informatique confidentielle Azure ? 

Oui. Pour plus d’informations sur les activateurs de plateforme, consultez la [page relative aux conteneurs confidentiels](confidential-containers.md).

## <a name="what-intel-sgx-driver-version-is-installed-in-the-aks-image"></a>Quelle version du pilote Intel SGX est installée dans l’image AKS ? 

Actuellement, les machines virtuelles DCSv2 d’informatique confidentielle Azure sont installées avec un pilote Intel SGX DCAP 1.33. 

## <a name="can-i-open-an-azure-support-ticket-if-i-run-into-issues"></a>Puis-je ouvrir un ticket de Support Azure en cas de problème ? 

Oui. Le Support Azure est fourni pour la préversion. Aucun contrat SLA n’est associé, car le produit est en phase de préversion.

## <a name="can-i-inject-post-install-scriptscustomize-drivers-to-the-nodes-provisioned-by-aks"></a>Puis-je injecter des scripts post-installation/personnaliser des pilotes sur les nœuds approvisionnés par AKS ? 

Non. Les [nœuds d’informatique confidentielle basés sur le moteur AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md) prennent en charge des nœuds d’informatique confidentielle qui autorisent des installations personnalisées.

## <a name="should-i-be-using-a-docker-base-image-to-get-started-on-enclave-applications"></a>Dois-je utiliser une image de base Docker pour commencer à utiliser des applications d’enclave ? 

Divers activateurs (éditeurs de logiciels indépendants et projets OSS) permettent d’activer des conteneurs confidentiels. Pour plus d’informations et pour obtenir des références individuelles à des implémentations, consultez la [page relative aux conteneurs confidentiels](confidential-containers.md).

## <a name="can-i-run-acc-nodes-with-other-standard-aks-skus-build-a-heterogenous-node-pool-cluster"></a>Puis-je exécuter des nœuds ACC avec d’autres références (SKU) AKS standard (créer un cluster de pools de nœuds hétérogènes) ? 

Oui. Vous pouvez exécuter différents pools de nœuds dans le même cluster AKS, notamment des nœuds ACC. Pour cibler vos applications d’enclave sur un pool de nœuds spécifique, pensez à ajouter des sélecteurs de nœud ou à appliquer des limites EPC. Pour plus d’informations concernant le démarrage rapide sur des nœuds confidentiels, consultez [cette rubrique](confidential-nodes-aks-get-started.md).

## <a name="can-i-run-windows-nodes-and-windows-containers-with-acc"></a>Puis-je exécuter des nœuds et des conteneurs Windows avec ACC ? 

Pas pour l'instant. Si vous avez besoin de nœuds ou de conteneurs Windows, contactez-nous. 

## <a name="what-if-my-container-size-is-more-than-available-epc-memory"></a>Que se passe-t-il si la taille de mon conteneur est supérieure à la mémoire EPC disponible ? 

La mémoire EPC s’applique à la partie de votre application qui est programmée pour s’exécuter dans l’enclave. La taille totale de votre conteneur n’est pas le bon indicateur pour comparer celui-ci avec la mémoire EPC maximale disponible. En fait, les machines DCSv2 avec SGX autorisent une mémoire de machine virtuelle maximale de 32 Go que votre partie non approuvée de l’application utiliserait. Toutefois, si votre conteneur consomme plus que la mémoire EPC disponible, les performances de la partie du programme s’exécutant dans l’enclave peuvent être affectées.

Pour mieux gérer la mémoire EPC dans les nœuds Worker, envisagez de gérer des limites de mémoire EPC via Kubernetes. Référez-vous à l’exemple ci-dessous :

```yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: sgx-test
  labels:
    app: sgx-test
spec:
  template:
    metadata:
      labels:
        app: sgx-test
    spec:
      containers:
      - name: sgxtest
        image: oeciteam/sgx-test:1.0
        resources:
          limits:
            kubernetes.azure.com/sgx_epc_mem_in_MiB: 10 # This limit will automatically place the job into confidential computing node. Alternatively you can target deployment to nodepools
      restartPolicy: Never
  backoffLimit: 0
```

## <a name="what-happens-if-my-enclave-consumes-more-than-maximum-available-epc-memory"></a>Que se passe-t-il si mon enclave consomme plus que la mémoire EPC disponible maximale ? 

La mémoire EPC disponible totale est partagée entre les applications d’enclave dans les mêmes machines virtuelles ou nœuds Worker. Si votre application utilise plus de mémoire EPC que celle disponible, ses performances peuvent en être affectées. Pour cette raison, nous vous recommandons de définir la tolérance par application dans votre fichier yaml de déploiement afin de mieux gérer la mémoire EPC disponible par nœud Worker, comme dans les exemples ci-dessus. Vous pouvez également augmenter les tailles de machine virtuelle du pool de nœuds Worker ou ajouter des nœuds. 

## <a name="why-cant-i-do-forks--and-exec-to-run-multiple-processes-in-my-enclave-application"></a>Pourquoi ne puis-je pas effectuer des fourches () et exécutions pour exécuter plusieurs processus dans mon application d’enclave ? 

À l’heure actuelle, les machines virtuelles de référence (SKU) DCsv2 d’informatique confidentielle Azure prennent en charge un seul espace d’adressage pour le programme s’exécutant dans une enclave. Un processus unique est une limitation actuelle conçue autour de la haute sécurité. Toutefois, des activateurs de conteneurs confidentiels peuvent avoir d’autres implémentations pour surmonter cette limitation.

## <a name="do-you-automatically-install-any-additional-daemonsets-to-expose-the-sgx-drivers"></a>Installez-vous automatiquement des contrôleurs DaemonSet supplémentaires pour exposer les pilotes SGX ? 

Oui. Le nom du contrôleur DaemonSet est sgx-device-plugin et sgx-quote-helper. Apprenez-en davantage sur leurs objectifs respectifs [ici](confidential-nodes-aks-overview.md).  

## <a name="what-is-the-vm-sku-i-should-be-choosing-for-confidential-computing-nodes"></a>Quelle référence (SKU) de machine virtuelle dois-je choisir pour les nœuds d’informatique confidentielle ? 

Références (SKU) DCSv2. Les [références (SKU) DCSv2](../virtual-machines/dcv2-series.md) sont disponibles dans les [régions prises en charge](https://azure.microsoft.com/global-infrastructure/services/?products=virtual-machines&regions=all).

## <a name="can-i-still-schedule-and-run-non-enclave-containers-on-confidential-computing-nodes"></a>Puis-je continuer à planifier et exécuter des conteneurs autres que d’enclave sur des nœuds d’informatique confidentielle ? 

Oui. Les machines virtuelles sont également dotées d’une mémoire ordinaire capable d’exécuter des charges de travail de conteneur standard. Prenez en considération le modèle de sécurité et de menace de vos applications avant de choisir les modèles de déploiement.

## <a name="can-i-provision-aks-with-dcsv2-node-pools-through-azure-portal"></a>Puis-je approvisionner AKS avec des pools de nœuds DCSv2 via le portail Azure ? 

Oui. Azure CLI peut également être utilisé en guise d’alternative comme expliqué [ici](confidential-nodes-aks-get-started.md).

## <a name="what-ubuntu-version-and-vm-generation-is-supported"></a>Quelles sont la version d’Ubuntu et la génération de machine virtuelle prises en charge ? 

18.04 sur la génération 2. 

## <a name="can-we-change-the-current-intel-sgx-dcap-diver-version-on-aks"></a>Pouvez-vous modifier la version actuelle du pilote Intel SGX DCAP sur AKS ? 

Non. Pour effectuer des installations personnalisées, nous vous recommandons de choisir des déploiements de [nœuds Worker d’informatique confidentielle de moteur AKS](https://github.com/Azure/aks-engine/blob/master/docs/topics/sgx.md). 

## <a name="what-version-of-kubernetes-do-you-support-and-recommend"></a>Quelle version de Kubernetes prenez-vous en charge et recommandez-vous ? 

Nous prenons en charge et recommandons Kubernetes, versions 1.16 et ultérieures. 

## <a name="what-are-the-known-current-limitation-or-technical-limitations-of-the-product-in-preview"></a>Quelles sont la limitation actuelle connue ou les limitations techniques du produit en préversion ? 

- Prend en charge uniquement les nœuds de machines virtuelles de deuxième génération Ubuntu 18.04. 
- Aucune prise en charge de nœuds ou conteneurs Windows.
- La mise à l’échelle automatique de pod horizontal basée sur la mémoire EPC n’est pas prise en charge. La mise à l’échelle basée sur la mémoire ordinaire et l’UC est prise en charge.
- Le service Dev Spaces sur AKS pour les applications confidentielles n’est pas pris en charge actuellement

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les conteneurs confidentiels, consultez la [page relative aux conteneurs confidentiels](confidential-containers.md).
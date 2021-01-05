---
title: Attestation d'enclaves dans Azure
description: Apprenez à utiliser l'attestation pour vérifier que votre environnement approuvé d'informatique confidentielle est sécurisé
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.workload: infrastructure
ms.topic: conceptual
ms.date: 9/22/2020
ms.author: JenCook
ms.openlocfilehash: 86856d1f66b5b7d723c907c17d7179ffcd2b82ab
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94565668"
---
# <a name="attesting-sgx-enclaves"></a>Attestation d'enclaves SGX

L'informatique confidentielle sur Azure propose des machines virtuelles basées sur Intel SGX qui peuvent isoler une partie de votre code ou de vos données. Lorsque vous utilisez ces [enclaves](confidential-computing-enclaves.md), vous devez vérifier que votre environnement approuvé est sécurisé. Cette vérification est le processus d’attestation. 

## <a name="overview"></a>Vue d’ensemble 

L’attestation permet à une partie de confiance d’avoir davantage confiance dans le fait (1) que son logiciel fonctionne dans une enclave et (2) que l’enclave est à jour et sécurisée. Par exemple, une enclave demande au matériel sous-jacent de générer des informations d’identification qui incluent la preuve que l’enclave existe sur la plateforme. Le rapport peut ensuite être remis à une deuxième enclave qui vérifie que le rapport a été généré sur la même plateforme.

![attestation du code dans l'enclave](media/attestation/attestation.png)



L’attestation doit être implémentée à l’aide d’un service d’attestation sécurisé qui est compatible avec le logiciel système et le silicium. Exemples de services que vous pouvez utiliser :

- [Microsoft Azure Attestation (préversion)](../attestation/overview.md) ou
- [Services d'attestation et d'approvisionnement d'Intel](https://software.intel.com/sgx/attestation-services)


Ces services sont tous compatibles avec l'infrastructure Intel SGX d'informatique confidentielle Azure. 

## <a name="next-steps"></a>Étapes suivantes
Essayez les [exemples Microsoft Azure Attestation pour les applications prenant en charge les enclaves](/samples/azure-samples/microsoft-azure-attestation/sample-code-for-intel-sgx-attestation-using-microsoft-azure-attestation/).
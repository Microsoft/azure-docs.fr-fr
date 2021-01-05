---
title: Outils de développement d’informatique confidentielle Azure
description: Utilisez des outils et bibliothèques pour développer des applications pour l’informatique confidentielle
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: JenCook
ms.openlocfilehash: 6bb3b8dbc7887419f7901a52b56c25f60c869abb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90993014"
---
# <a name="application-development-on-intel-sgx"></a>Développement d’applications sur Intel SGX 


L’infrastructure d’informatique confidentielle requiert des outils et logiciels spécifiques. Cette page décrit spécifiquement les concepts liés au développement d’applications pour des machines virtuelles d’informatique confidentielle Azure s’exécutant sur Intel SGX. Avant de lire cette page, [lisez la présentation des machines virtuelles Intel SGX et des enclaves](confidential-computing-enclaves.md). 

Pour tirer parti de la puissance des enclaves et des environnements isolés, vous devez utiliser des outils qui prennent en charge l’informatique confidentielle. Différents outils prennent en charge le développement d’applications enclaves. Par exemple, vous pouvez utiliser ces frameworks open source : 

- [Kit de développement logiciel (SDK) Open Enclave (SDK OE)](#oe-sdk)
- [Confidential Consortium Framework (CCF)](#ccf)

## <a name="overview"></a>Vue d’ensemble

Une application créée avec des enclaves est partitionnée de deux façons :

1. Un composant « non approuvé » (l’hôte)
1. Un composant « approuvé » (l’enclave)


![Développement d’applications](media/application-development/oe-sdk.png)


**L’hôte** correspond à l’emplacement sur lequel s’exécute votre application enclave. Il s’agit d’un environnement non approuvé. Le code de l’enclave déployé sur l’hôte n’est pas accessible à l’hôte. 

**L’enclave** est l’endroit où s’exécutent le code de l’application et ses données en cache/sa mémoire. Les calculs sécurisés doivent avoir lieu dans les enclaves pour garantir la protection des secrets et des données sensibles. 


Lors de la conception de l’application, il est important d’identifier et de déterminer quelle partie de l’application doit s’exécuter dans les enclaves. Le code que vous choisissez de placer dans le composant approuvé est isolé du reste de votre application. Une fois l’enclave initialisée et le code chargé en mémoire, ce code ne peut être ni lu ni modifié à partir des composants non approuvés. 

## <a name="open-enclave-software-development-kit-oe-sdk"></a>SDK Open Enclave (SDK OE) <a id="oe-sdk"></a>

Utilisez une bibliothèque ou un framework pris en charge par votre fournisseur si vous souhaitez écrire du code qui s’exécute dans une enclave. Le [SDK Open Enclave](https://github.com/openenclave/openenclave) (SDK OE) est un SDK open source qui permet de créer une couche d’abstraction sur différents matériels prenant en charge l’informatique confidentielle. 

Le SDK OE est conçu pour servir de couche d’abstraction unique sur n’importe quel matériel sur n’importe quel fournisseur de solutions cloud. Le SDK OE peut être utilisé sur des machines virtuelles d’informatique confidentielle Azure pour créer et exécuter des applications sur des enclaves.

## <a name="confidential-consortium-framework-ccf"></a>CCF (Confidential Consortium Framework) <a id="ccf"></a>

Le [CCF](https://github.com/Microsoft/CCF) est un réseau distribué de nœuds, chacun exécutant ses propres enclaves. Le réseau de nœuds de confiance vous permet d’exécuter un registre distribué. Le registre fournit des composants sécurisés et fiables à l’usage du protocole. 

![Nœuds CCF](media/application-development/ccf.png)

Cette infrastructure open source offre une haute confidentialité granulaire et une gouvernance de consortium pour blockchain. Chaque nœud utilisant des TEE, vous pouvez garantir un consensus et un traitement des transactions sécurisés.


## <a name="next-steps"></a>Étapes suivantes 
- [Déployer une machine virtuelle série DCsv2 d’informatique confidentielle](quick-create-portal.md)
- [Télécharger et installer le kit de développement logiciel (SDK) OE et commencer à développer des applications](https://github.com/openenclave/openenclave)
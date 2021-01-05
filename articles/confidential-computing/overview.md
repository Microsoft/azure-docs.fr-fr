---
title: Vue d’ensemble de l’informatique confidentielle Azure
description: Vue d’ensemble de l’informatique confidentielle Azure (ACC, Azure Confidential Computing)
services: virtual-machines
author: JBCook
ms.service: virtual-machines
ms.subservice: workloads
ms.topic: overview
ms.date: 09/22/2020
ms.author: JenCook
ms.openlocfilehash: 44479db6b29075e87b0d2dcef6f0d9bc1881738c
ms.sourcegitcommit: 04fb3a2b272d4bbc43de5b4dbceda9d4c9701310
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94560823"
---
# <a name="confidential-computing-on-azure"></a>Informatique confidentielle dans Azure

L’informatique confidentielle Azure vous permet d’isoler vos données sensibles pendant qu’elles sont traitées dans le cloud. De nombreux secteurs utilisent l’informatique confidentielle afin de protéger leurs données pour :

- Sécuriser des données financières
- Protéger les informations des patients
- Exécuter des processus de machine learning sur des informations sensibles
- Exécuter des algorithmes sur des jeux de données chiffrés provenant de plusieurs sources


## <a name="overview"></a>Vue d’ensemble
<p><p>


> [!VIDEO https://www.youtube.com/embed/rT6zMOoLEqI]

Nous savons que la sécurisation de vos données cloud est importante. Nous sommes conscients de vos préoccupations. Voici quelques questions que nos clients peuvent se poser lors de la migration de charges de travail sensibles vers le cloud : 

- Comment m’assurer que Microsoft ne peut pas accéder aux données qui ne sont pas chiffrées ?
- Comment éviter les menaces de sécurité provenant d’administrateurs privilégiés dans mon entreprise ?
- Par quels autres moyens est-ce que je peux empêcher des tiers d’accéder aux données sensibles des clients ?

Microsoft Azure vous aide à réduire la surface d’attaque pour renforcer la protection de vos données. Azure propose déjà de nombreux outils pour protéger les [**données au repos**](../security/fundamentals/encryption-atrest.md), par le biais de modèles tels que le chiffrement côté client et le chiffrement côté serveur. Azure permet également de chiffrer les [**données en transit**](../security/fundamentals/data-encryption-best-practices.md#protect-data-in-transit) au moyen de mécanismes reposant sur des protocoles sécurisés tels que TLS et HTTPS. Cette page introduit une troisième forme de chiffrement, à savoir le chiffrement des **données en cours d’utilisation**.

## <a name="introduction-to-confidential-computing"></a>Présentation de l’informatique confidentielle  

L’informatique confidentielle est un terme spécialisé du [Confidential Computing Consortium](https://confidentialcomputing.io/) (CCC), une fondation dédiée à la définition de l’informatique confidentielle et à l’accélération de son adoption. Le CCC définit l’informatique confidentielle comme suit : La protection des données utilisées en effectuant des traitements dans un environnement d’exécution approuvé (TEE, Trusted Execution Environment) matériel.

Un TEE est un environnement dans lequel seul le code autorisé est exécuté. Aucune donnée dans le TEE ne peut être lue ou falsifiée par du code en dehors de cet environnement. 

### <a name="lessen-the-need-for-trust"></a>Limiter le besoin d’approbation
L’exécution de charges de travail sur le cloud demande une approbation. Vous donnez cette approbation à différents fournisseurs en activant différents composants de votre application.


**Fournisseurs de logiciels d’application** : Approuvez un logiciel en déployant localement, en utilisant de l’open source ou en créant un logiciel d’application en interne.

**Fournisseurs de matériel** : Approuvez du matériel en utilisant du matériel local ou du matériel d’origine interne. 

**Fournisseurs d’infrastructure** : Approuvez des fournisseurs de cloud ou gérez vos propres centres de données locaux.


L’informatique confidentielle Azure facilite l’approbation du fournisseur de cloud, en réduisant le besoin d’approbation de différents aspects de l’infrastructure de cloud computing. L’informatique confidentielle Azure minimise l’approbation pour le noyau du système d’exploitation hôte, l’hyperviseur, l’administrateur de la machine virtuelle et l’administrateur de l’hôte.

### <a name="reducing-the-attack-surface"></a>Réduction de la surface d’attaque
Le TCB (Trusted Computing Base) fait référence à l’ensemble des composants matériels, des microprogrammes et des logiciels d’un système qui fournissent un environnement sécurisé. Les composants au sein du TCB sont considérés comme « critiques ». Si un composant au sein du TCB est compromis, toute la sécurité du système peut être en péril. 

Plus le TCB est petit, plus la sécurité est élevée. Il y a moins de risques d’exposition à différentes vulnérabilités, programmes malveillants, attaques et personnes malveillantes. L’informatique confidentielle Azure vise à réduire le TCB pour vos charges de travail cloud en offrant des environnements TEE. Les environnements TEE réduisent votre TCB à des fichiers binaires, du code et des bibliothèques d’exécution approuvés. Quand vous utilisez l’infrastructure et les services Azure pour de l’informatique confidentielle, vous pouvez supprimer de votre TCB tout ce qui provient de Microsoft.


## <a name="using-azure-for-cloud-based-confidential-computing"></a>Utilisation d’Azure pour l’informatique confidentielle basée sur le cloud <a id="cc-on-azure"></a>

L’informatique confidentielle Azure vous permet de tirer parti des avantages de l’informatique confidentielle dans un environnement virtualisé. Vous pouvez désormais utiliser des outils, des logiciels et une infrastructure cloud pour créer des applications sur du matériel sécurisé.  

**Empêcher les accès non autorisés** : Traitez des données sensibles dans le cloud. N’ayez aucun doute sur le fait qu’Azure fournit la meilleure protection possible des données, avec peu ou pas de modifications par rapport à ce qui se fait aujourd’hui.

**Conformité réglementaire** : Migrez vers le cloud et gardez le contrôle total des données de façon à satisfaire aux réglementations légales relatives à la protection des informations personnelles et à la sécurité des adresses IP de l’organisation.

**Collaboration sécurisée et non fiable** : Attaquez-vous aux problèmes des travaux à l’échelle du secteur en examinant les données entre différentes organisations, même concurrentes, pour permettre des analyses des données de grande envergure et obtenir des insights plus approfondis.

**Traitement isolé** : Offrez une nouvelle vague de produits qui suppriment la responsabilité légale sur les données privées avec un traitement à l’aveugle. Les données utilisateur ne peuvent même pas être récupérées par le fournisseur de services. 

## <a name="get-started"></a>Bien démarrer
### <a name="azure-compute"></a>Azure Compute
Créez des applications sur des offres IaaS d’informatique confidentielle dans Azure.
- Machines virtuelles : [DCsv2-Series](confidential-computing-enclaves.md)
- Azure Kubernetes (AKS) : [Orchestrer des conteneurs confidentiels](confidential-nodes-aks-overview.md)

### <a name="azure-security"></a>Sécurité Azure 
Garantissez la sécurité de vos charges de travail grâce aux méthodes de vérification et à la gestion de clés matérielles. 
- Attestation : [Microsoft Azure Attestation (préversion)](../attestation/overview.md)
- Gestion des clés : Managed-HSM (préversion)

### <a name="develop"></a>Développer
Commencez à utiliser le développement d’applications activées pour les enclaves et déployez des algorithmes confidentiels en utilisant l’infrastructure d’inférence confidentielle.
- Écrivez des applications à exécuter sur des machines virtuelles DCsv2 : [SDK Open-enclave](https://github.com/openenclave/openenclave)
- Modèles ML confidentiels dans le runtime ONNX : [Inférence confidentielle (bêta)](https://aka.ms/confidentialinference)

## <a name="next-steps"></a>Étapes suivantes

Déployez une machine virtuelle de la série DCsv2 et installez le SDK OE sur celle-ci.

> [!div class="nextstepaction"]
> [Déployer une machine virtuelle d’informatique confidentielle dans la Place de marché Azure](quick-create-marketplace.md)
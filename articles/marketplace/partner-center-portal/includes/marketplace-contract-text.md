---
title: Fichier include
description: fichier include de texte pour le contrat Standard du marketplace commercial de Microsoft
documentationcenter: partner-center-commercial-marketplace
ms.service: marketplace
ms.subservice: partnercenter-marketplace-publisher
ms.topic: include
ms.date: 01/22/2020
ms.custom: include file
author: mingshen-ms
ms.author: mingshen
ms.openlocfilehash: e1119cf5c40cdcb1fd598b85410ea9149a7cc146
ms.sourcegitcommit: 4313e0d13714559d67d51770b2b9b92e4b0cc629
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/27/2020
ms.locfileid: "91400982"
---
Microsoft fournit un modèle de contrat Standard pour le marketplace commercial.

- **Utiliser le contrat Standard pour la place de marché commercial Microsoft ?**

Pour simplifier le processus d’approvisionnement pour les clients et réduire la complexité juridique pour les éditeurs de logiciels, Microsoft propose un contrat Standard pour la place de marché commerciale de Microsoft afin de faciliter les transactions sur la place de marché. Plutôt que d’élaborer des conditions générales personnalisées, les éditeurs de la place de marché commerciale peuvent choisir de proposer leur logiciel dans le cadre du contrat Standard, que les clients ne doivent accepter qu’une seule fois. Le contrat Standard est disponible ici : https://go.microsoft.com/fwlink/?linkid=2041178.

Vous pouvez choisir d’utiliser le contrat Standard au lieu de fournir vos propres conditions générales en activant la case à cocher « Utiliser le contrat Standard pour la place de marché commerciale ».

![Utiliser la case à cocher Contrat Standard](./media/use-standard-contract.png)

> [!NOTE]
> Après avoir publié une offre à l’aide du contrat Standard pour la place de marché commerciale de Microsoft, vous ne pouvez pas utiliser vos propres conditions générales. Vous devez faire un choix entre les deux. Vous proposez votre solution avec le contrat Standard **ou** avec vos propres conditions générales. Si vous souhaitez modifier les conditions du contrat Standard, vous pouvez le faire par le biais des Modifications du contrat Standard.

**Modifications du contrat Standard**

Les modifications du contrat Standard permettent aux éditeurs de sélectionner le contrat Standard par souci de simplicité et de personnaliser les conditions pour leur produit ou entreprise. Les clients sont invités à examiner les modifications apportées au contrat, puis à accepter le contrat Standard de Microsoft.

Deux types de modifications sont proposées aux éditeurs de la place de marché commerciale :

- Modifications universelles : Ces modifications sont appliquées de manière universelle au contrat Standard de tous les clients. Les modifications universelles s’affichent pour chaque client de l’offre dans le flux d’achat. Les clients doivent accepter les conditions du contrat Standard et la modification avant de pouvoir utiliser votre offre.
- Modifications personnalisées : Il s’agit de modifications spéciales apportées au contrat Standard qui ciblent certains clients uniquement via les ID de locataires Azure. Les éditeurs peuvent choisir le locataire qu'ils souhaitent cibler. Seuls les clients du locataire recevront les conditions personnalisées dans le flux d’achat de l’offre.  Les clients doivent accepter les conditions du contrat Standard et la ou les modifications avant de pouvoir utiliser votre offre.

>[!NOTE]
> Ces deux types de modifications s’ajoutent l’un à l’autre. Les clients ciblés par les modifications personnalisées reçoivent aussi la modification universelle apportée au contrat Standard lors de l’achat.

**Termes d’amendement universel au contrat Standard pour la place de marché commerciale Microsoft** : Entrez les conditions de l’amendement universel dans cette zone. Vous pouvez fournir un seul amendement universel par offre. Vous pouvez entrer un nombre illimité de caractères dans cette zone. Ces termes sont affichés pour les clients dans AppSource, la Place de marché Azure et/ou le portail Azure pendant le processus de découverte et d’achat.

**Conditions d'amendements personnalisées du contrat Standard pour la place de marché commerciale de Microsoft**: Commencez par sélectionner **Ajouter des conditions d’amendement personnalisées**. Vous pouvez fournir jusqu’à 10 termes d’amendement personnalisés par offre.

- **Termes d’amendement personnalisés**: Saisissez vos conditions d’amendement personnalisées dans la zone Conditions d’amendement personnalisées. Vous pouvez entrer un nombre illimité de caractères dans cette zone. Seuls les clients des ID de locataire que vous spécifiez pour ces conditions personnalisées seront présentés avec les termes de l’amendement personnalisé dans le flow d’achat de l’offre dans le Portail Azure.  
- **ID de locataire** (obligatoire) : Chaque amendement personnalisé peut être ciblé vers jusqu’à 20 ID de locataire. Si vous ajoutez un amendement personnalisé, vous devez fournir au moins un ID de locataire. L’ID de locataire identifie votre client dans Azure. Vous pouvez demander à votre client cet ID et il peut le trouver en accédant à portal.azure.com > Azure Active Directory > Propriétés. La valeur de l’ID de répertoire est l’ID de locataire (par exemple, 50c464d3-4930-494c-963C-1e951d15360e). Vous pouvez également rechercher l’ID de locataire de l’organisation de votre client à l’aide de son URL de nom de domaine sur le site [Quel est mon ID de locataire Microsoft Azure et Microsoft 365 ?](https://www.whatismytenantid.com)
- **Description** (facultatif) : Vous pouvez éventuellement fournir une description claire de l’ID de locataire, qui vous aide à identifier le client que vous ciblez avec cet amendement.

**Conditions générales**

Si vous souhaitez fournir vos propres conditions générales, vous pouvez les saisir dans le champ Conditions générales. Vous pouvez saisir jusqu’à 10 000 caractères dans ce champ. Si vos conditions générales nécessitent une description plus longue, saisissez un lien URL unique dans ce champ où trouver les conditions générales. Ceci affichera un lien actif aux clients.

Les clients doivent accepter ces conditions avant de pouvoir essayer de votre application.

Sélectionnez **Enregistrer le brouillon** avant de continuer.

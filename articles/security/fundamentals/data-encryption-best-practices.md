---
title: Bonnes pratiques en matière de chiffrement et de sécurité des données - Microsoft Azure
description: Cet article détaille les meilleures pratiques en matière de chiffrement et de sécurité des données, à l’aide de capacités Azure intégrées.
services: security
documentationcenter: na
author: TerryLanfear
manager: rkarlin
editor: TomSh
ms.assetid: 17ba67ad-e5cd-4a8f-b435-5218df753ca4
ms.service: security
ms.subservice: security-fundamentals
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 03/09/2020
ms.author: terrylan
ms.openlocfilehash: c88a2cf33cf1149a17af4a82dcf2858822f0c6d4
ms.sourcegitcommit: e2dc549424fb2c10fcbb92b499b960677d67a8dd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94696147"
---
# <a name="azure-data-security-and-encryption-best-practices"></a>Bonnes pratiques relatives au chiffrement et à la sécurité des données dans Azure
Cet article décrit les bonnes pratiques relatives au chiffrement et à la sécurité des données.

Ces meilleures pratiques font l’objet d’un consensus et sont compatibles avec les capacités et fonctionnalités actuelles de la plateforme Azure. Les opinions et technologies évoluent au fil du temps ; cet article est régulièrement mis à jour de manière à tenir compte de ces changements.

## <a name="protect-data"></a>Protection des données
Pour assurer la protection des données dans le cloud, vous devez tenir compte des états que les données peuvent présenter, mais aussi des contrôles disponibles pour ces états. Les bonnes pratiques pour la sécurité et le chiffrement des données Azure s’apparentent aux états des données suivants :

- Au repos : Ceci inclut tous les objets de stockage, conteneurs et types d’informations présents de manière statique sur un support physique, qu’il s’agisse d’un disque magnétique ou d’un disque optique.
- En transit : Quand des données sont transférées entre des composants, des emplacements ou des programmes, elles sont considérées comme étant « en transit ». Exemples : transfert sur le réseau, sur un bus des services (du réseau local au cloud et inversement, y compris les connexions hybrides telles que ExpressRoute), ou pendant un traitement d’entrée/sortie.

## <a name="choose-a-key-management-solution"></a>Choisir une solution de gestion des clés

La protection de vos clés est essentielle pour protéger vos données dans le cloud.

[Azure Key Vault](../../key-vault/general/overview.md) permet de protéger les clés de chiffrement et les secrets utilisés par les services et les applications cloud. Key Vault rationalise le processus de gestion de clés et vous permet de garder le contrôle des clés qui accèdent à vos données et les chiffrent. Les développeurs peuvent créer des clés pour le développement et le test en quelques minutes, puis les migrer en clés de production. Les administrateurs de sécurité peuvent accorder (et annuler) les autorisations sur les clés, si nécessaire.

Vous pouvez utiliser Key Vault pour créer plusieurs conteneurs sécurisés, désignés sous le terme de coffres. Ces coffres s’appuient sur des modules de sécurité matérielle. Les coffres contribuent à réduire les risques de perte accidentelle des informations de sécurité en centralisant le stockage des secrets d’application. En outre, les coffres de clés contrôlent et journalisent l’accès à l’ensemble de leur contenu. Azure Key Vault peut gérer la demande et le renouvellement des certificats TLS (Transport Layer Security). Il offre des fonctionnalités pour une solution robuste de gestion de cycle de vie de certificat.

Azure Key Vault a été conçu pour prendre en charge les clés et les secrets d’application. Il n’est pas destiné à faire office de magasin des mots de passe utilisateur.

Voici les bonnes pratiques de sécurité relatives à l’utilisation de Key Vault.

**Bonne pratique** : Accordez l’accès aux utilisateurs, aux groupes et aux applications au niveau d’une étendue spécifique.   
**Détail** : Utilisez des rôles prédéfinis Azure RBAC. Par exemple, pour autoriser un utilisateur à gérer des coffres de clés, vous devez attribuer un rôle prédéfini [Collaborateur de coffre de clés](../../role-based-access-control/built-in-roles.md) à cet utilisateur dans une étendue spécifique. L’étendue dans ce cas correspond à un abonnement, à un groupe de ressources ou simplement à un coffre de clés spécifique. Si les rôles prédéfinis ne répondent pas à vos besoins, vous pouvez [définir vos propres rôles](../../role-based-access-control/custom-roles.md).

**Bonne pratique** : Contrôlez ce à quoi les utilisateurs ont accès.   
**Détail** : L’accès à un coffre de clés est contrôlé via deux interfaces distinctes : le plan de gestion et le plan de données. Les contrôles d’accès au plan de gestion et au plan de données fonctionnent indépendamment.

Utilisez Azure RBAC pour contrôler le contenu auquel ont accès les utilisateurs. Par exemple, si vous souhaitez autoriser une application à utiliser les clés d’un coffre de clés, vous avez seulement besoin de lui accorder des autorisations d’accès au plan de données à l’aide de stratégies d’accès au coffre de clés. Aucun accès au plan de gestion n’est requis pour cette application. À l’inverse, si vous souhaitez qu’un utilisateur puisse lire les propriétés et les balises d’un coffre, mais pas accéder aux clés, secrets ou certificats, vous pouvez lui accorder un accès en lecture à l’aide d’Azure RBAC. Aucun accès au plan de données n’est requis.

**Bonne pratique** : Stockez les certificats dans votre coffre de clés. Vos certificats ont une grande valeur. Entre de mauvaises mains, la sécurité de votre application ou de vos données peut être compromise.   
**Détail** : Azure Resource Manager peut déployer de façon sécurisée des certificats stockés dans Azure Key Vault sur des machines virtuelles Azure quand ces dernières sont déployées. En définissant les stratégies d’accès appropriées pour le coffre de clés, vous contrôlez également qui peut accéder à votre certificat. Le fait que vous puissiez gérer tous vos certificats en un seul endroit dans Azure Key Vault constitue un autre avantage. Pour plus d’informations, consultez [Déployer des certificats sur les machines virtuelles à partir de coffres de clés gérés par les clients](/archive/blogs/kv/updated-deploy-certificates-to-vms-from-customer-managed-key-vault).

**Bonne pratique** : Veillez à pouvoir récupérer d’une suppression de coffres de clés ou d’objets de coffres de clés.   
**Détail** : La suppression de coffres de clés ou d’objets de coffres de clés peut être faite par inadvertance ou être due à une activité malveillante. Activez les fonctionnalités de suppression réversible et de protection contre le vidage de Key Vault, en particulier pour les clés utilisées pour chiffre des données au repos. La suppression de ces clés est similaire à une perte de données. Vous pouvez donc récupérer des coffres et objets de coffres supprimés si besoin. Effectuez des opérations de récupération Key Vault quotidiennement.

> [!NOTE]
> Si un utilisateur dispose d’autorisations de contributeur (Azure RBAC) pour un plan de gestion d’un coffre de clés, il peut s’accorder à lui-même l’accès au plan de données en définissant une stratégie d’accès au coffre de clés. Il est recommandé de contrôler étroitement quels utilisateurs disposent d’un accès « Collaborateur » à vos coffres de clés afin de vous assurer que seules les personnes autorisées peuvent gérer vos coffres de clés, clés, secrets et certificats.
>
>

## <a name="manage-with-secure-workstations"></a>Gestion de stations de travail sécurisées

> [!NOTE]
> L’administrateur ou le propriétaire de l’abonnement doit utiliser une station de travail avec un accès sécurisé ou privilégié.
>
>

Dans la mesure où la grande majorité des atteintes ciblent l’utilisateur final, le point de terminaison devient l’un des principaux points d’attaque. Si un pirate informatique compromet le point de terminaison, il peut se servir des informations d’identification pour accéder aux données de l’organisation. La plupart des attaques visant le point de terminaison exploitent le fait que les utilisateurs occupent le rôle d’administrateur dans leurs stations de travail locales.

**Bonne pratique** : Utilisez une station de travail de gestion sécurisée pour protéger les comptes, les tâches et les données sensibles.   
**Détail** : Utilisez une [station de travail avec accès privilégié](/windows-server/identity/securing-privileged-access/privileged-access-workstations) pour réduire la surface d’attaque au sein des stations de travail. Ces stations de travail de gestion sécurisées peuvent vous aider à limiter certaines attaques, afin d’optimiser la sécurité de vos données.

**Bonne pratique** : Appliquez une protection aux points de terminaison.   
**Détail** : Appliquez des stratégies de sécurité sur l’ensemble des appareils utilisés pour consommer des données, quel que soit l’emplacement de ces dernières (dans le cloud ou en local).

## <a name="protect-data-at-rest"></a>Protection des données au repos

Le [chiffrement des données au repos](https://cloudblogs.microsoft.com/microsoftsecure/2015/09/10/cloud-security-controls-series-encrypting-data-at-rest/) est une étape obligatoire vers la confidentialité, la conformité et la souveraineté des données.

**Bonne pratique** : Appliquez le chiffrement des disques pour protéger vos données.   
**Détail** : Utilisez [Azure Disk Encryption](./azure-disk-encryption-vms-vmss.md). Il permet aux administrateurs informatiques de chiffrer les disques des machines virtuelles IaaS Windows et Linux. Il utilise la fonctionnalité standard BitLocker de Windows et la fonctionnalité dm-crypt de Linux pour chiffrer les volumes des disques du système d’exploitation et des données.

Stockage Azure et Azure SQL Database chiffrent les données au repos par défaut, et de nombreux services offrent le chiffrement en option. Vous pouvez utiliser Azure Key Vault pour garder le contrôle des clés qui accèdent à vos données et les chiffrent. Consultez [Prise en charge du modèle de chiffrement des fournisseurs de ressources Azure pour en savoir](encryption-atrest.md#azure-resource-providers-encryption-model-support).

**Bonnes pratiques** : Utilisez le chiffrement pour réduire les risques d’accès non autorisé aux données.   
**Détail** : Chiffrez vos lecteurs avant d’y écrire des données sensibles.

Les organisations qui ne chiffrent pas leurs données sont plus exposées aux problèmes de confidentialité des données. Par exemple, les utilisateurs non autorisés ou non fiables peuvent voler des données dans des comptes compromis ou accéder indûment à des données codées en ClearFormat. Les entreprises doivent prouver leur diligence et utiliser les contrôles de sécurité appropriés pour améliorer la sécurité des données afin de se conformer aux règlementations du secteur.

## <a name="protect-data-in-transit"></a>Protection des données en transit

La protection des données en transit doit être un aspect essentiel de votre stratégie de protection des données. Comme les données transitent entre différents emplacements, dans les deux sens, nous vous recommandons d’utiliser systématiquement les protocoles SSL/TLS pour ces déplacements de données. Dans certains cas, vous souhaiterez isoler l’intégralité du canal de communication entre vos infrastructures locales et sur le cloud, via un réseau privé virtuel (VPN).

Pour les données qui transitent entre votre infrastructure locale et Azure, vous devez envisager le recours aux dispositifs de protection appropriés, comme HTTPS ou VPN. Lors de l’envoi du trafic chiffré entre un réseau virtuel Azure et un emplacement local sur l’Internet public, utilisez la [passerelle VPN Azure](../../vpn-gateway/index.yml).

Voici les bonnes pratiques relatives à la passerelle VPN Azure, et les protocoles SSL/TLS et HTTPS.

**Bonne pratique** : Sécurisez l’accès depuis plusieurs stations de travail locales à un réseau virtuel Azure.   
**Détail** : Utilisez un [VPN de site à site](../../vpn-gateway/vpn-gateway-howto-site-to-site-resource-manager-portal.md).

**Bonne pratique** : Sécurisez l’accès depuis une station de travail locale à un réseau virtuel Azure.   
**Détail** : Utilisez un [VPN de point à site](../../vpn-gateway/vpn-gateway-howto-point-to-site-classic-azure-portal.md).

**Bonne pratique** : Déplacez les jeux de données plus grands via une liaison WAN dédiée à haute vitesse.   
**Détail** : Utilisez [ExpressRoute](../../expressroute/expressroute-introduction.md). Si vous choisissez d’utiliser ExpressRoute, vous pouvez également chiffrer les données au niveau des applications par le biais de SSL/TLS ou d’autres protocoles pour une protection renforcée.

**Bonne pratique** : Interagissez avec Stockage Azure via le portail Azure.   
**Détail** : Toutes les transactions se font via HTTPS. [L’API de stockage REST](/rest/api/storageservices/) par le biais de HTTPS peut également être utilisée pour interagir avec le [stockage Azure](https://azure.microsoft.com/services/storage/).

Les organisations qui ne parviennent pas à protéger les données en transit sont plus sensibles aux [attaques d’intercepteur](/previous-versions/office/skype-server-2010/gg195821(v=ocs.14)), aux [écoutes électroniques](/previous-versions/office/skype-server-2010/gg195641(v=ocs.14)) et au piratage de session. Ces attaques peuvent être la première étape d’un processus visant à accéder à des données confidentielles.

## <a name="secure-email-documents-and-sensitive-data"></a>Sécuriser des courriers, des documents et des données sensibles

Vous tenez à contrôler et à sécuriser les courriers, les documents et les données sensibles que vous partagez en dehors de votre entreprise. [Azure Information Protection](/azure/information-protection/) est une solution basée sur le cloud aidant une organisation à classer, étiqueter et protéger ses documents et ses courriers. Cela peut être fait automatiquement par les administrateurs qui définissent les règles et les conditions, manuellement par les utilisateurs, ou d’une manière mixte où les utilisateurs reçoivent des recommandations.

La classification est identifiable à tout moment, quel que soit l’endroit où les données sont stockées ou avec qui elles sont partagées. Les étiquettes incluent des marquages visuels tels que les en-têtes, les pieds de page et les filigranes. Les métadonnées sont ajoutées aux en-têtes des courriers et des fichiers en texte en clair. Le texte en clair s’assure que les autres services, tels que les solutions pour éviter la perte de données, puissent identifier la classification et prendre les mesures appropriées.

La technologie de protection utilise Azure Rights Management (Azure RMS). Cette technologie est intégrée à d’autres services cloud et applications Microsoft, comme Microsoft 365 et Azure Active Directory. Cette technologie de protection utilise des stratégies de chiffrement, d’identité et d’autorisation. La protection appliquée à l’aide de Azure RMS reste associée aux documents et aux courriers, indépendamment de leur emplacement, au sein ou en dehors de votre organisation, des réseaux, des serveurs de fichiers et des applications.

Cette solution de protection des informations vous permet de contrôler vos données, même lorsqu’elles sont partagées avec d’autres personnes. Vous pouvez aussi utiliser Azure RMS avec vos propres applications métier et solutions de protection des informations d’éditeurs de logiciels, qu’elles soient en local ou dans le cloud.

Nous vous recommandons :

- De [déployer Azure Information Protection](/azure/information-protection/deployment-roadmap) pour votre organisation.
- D’appliquer des étiquettes qui reflètent les besoins de votre entreprise. Par exemple : Appliquez une étiquette nommée « Hautement confidentiel » à tous vos documents et courriers contenant des données ultrasecrètes, afin de classifier et protéger ces données. Ensuite, seuls les utilisateurs autorisés peuvent accéder à ces données, avec les restrictions que vous spécifiez.
- Configurez [la journalisation de l’utilisation d’Azure RMS](/azure/information-protection/log-analyze-usage) afin de surveiller l’utilisation du service de protection par votre organisation.

Les organisations bénéficiant d’une [classification](https://download.microsoft.com/download/0/A/3/0A3BE969-85C5-4DD2-83B6-366AA71D1FE3/Data-Classification-for-Cloud-Readiness.pdf) et d’une protection insuffisantes des données sont plus vulnérables aux fuites ou aux utilisations malveillantes des données. Avec une protection de fichier adaptée, vous pouvez analyser les flux de données pour obtenir des informations sur votre entreprise, détecter des comportements à risque et prendre les mesures correctives adéquates, suivre l’accès aux documents, etc.

## <a name="next-steps"></a>Étapes suivantes

Consultez l’article [Bonnes pratiques et tendances Azure relatives à la sécurité](best-practices-and-patterns.md) pour découvrir d’autres bonnes pratiques en matière de sécurité à appliquer dans le cadre de la conception, du déploiement et de la gestion de vos solutions cloud avec Azure.

Les ressources suivantes fournissent des informations générales sur la sécurité Azure et les services Microsoft associés :
* [Blog de l’équipe de sécurité Azure](/archive/blogs/azuresecurity/) : pour obtenir des informations à jour sur les dernières actualités sur la sécurité Azure
* [Centre de réponse aux problèmes de sécurité Microsoft](https://technet.microsoft.com/library/dn440717.aspx) : emplacement où les vulnérabilités de sécurité Microsoft, dont les problèmes rencontrés avec Azure, peuvent être rapportées ou signalées par e-mail à l’adresse secure@microsoft.com
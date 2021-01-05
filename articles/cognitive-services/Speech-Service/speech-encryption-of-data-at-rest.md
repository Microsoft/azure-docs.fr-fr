---
title: Chiffrement du service Speech pour les données au repos
titleSuffix: Azure Cognitive Services
description: Microsoft propose des clés de chiffrement gérées par Microsoft et vous permet également de gérer vos abonnements Cognitive Services à l’aide de vos propres clés, appelées clés gérées par le client (CMK). Cet article traite du chiffrement des données au repos pour le service vocal.
author: erindormier
manager: venkyv
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 08/28/2020
ms.author: egeaney
ms.openlocfilehash: aa0fe33dff0161767b74546aad49003d8fc70c16
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95015254"
---
# <a name="speech-service-encryption-of-data-at-rest"></a>Chiffrement du service Speech pour les données au repos

Le service Speech chiffre automatiquement vos données lors de leur conservation dans le cloud. Le chiffrement du service Speech protège vos données et vous aide à répondre aux engagements de votre entreprise en matière de sécurité et de conformité.

## <a name="about-cognitive-services-encryption"></a>À propos du chiffrement de Cognitive Services

Les données sont chiffrées et déchiffrées à l'aide du chiffrement [AES 256 bits](https://en.wikipedia.org/wiki/FIPS_140-2) certifié [FIPS 140-2](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard). Le chiffrement et le déchiffrement sont transparents, ce qui signifie que le chiffrement et l’accès sont gérés automatiquement. Vos données étant sécurisées par défaut, vous n’avez pas besoin de modifier votre code ou vos applications pour tirer parti du chiffrement.

## <a name="about-encryption-key-management"></a>À propos de la gestion des clés de chiffrement

Lorsque vous utilisez Custom Speech et Custom Voice, le service Speech peut stocker les données suivantes dans le cloud :  

* Données de trace Speech : uniquement si vous activez la trace pour votre point de terminaison personnalisé
* Données de formation et de test chargées

Par défaut, vos données sont stockées dans un compte de stockage Microsoft et votre abonnement utilise des clés de chiffrement gérées par Microsoft. Vous pouvez également préparer votre propre compte de stockage. L’accès au magasin est géré par la fonctionnalité Identité managée, et le service Speech ne peut pas accéder directement à vos propres données, notamment les données de trace Speech, les données de formation de personnalisation et les modèles personnalisés.

Pour plus d’informations sur Identité managée, consultez [Que sont les identités managées ?](../../active-directory/managed-identities-azure-resources/overview.md).

## <a name="bring-your-own-storage-byos-for-customization-and-logging"></a>Mode Bring your own storage (BYOS) pour la personnalisation et la journalisation

Pour demander l’accès à la fonctionnalité Mode Bring your own storage (BYOS), remplissez puis envoyez le  [formulaire de demande Service Speech - bring your own storage (BYOS)](https://aka.ms/cogsvc-cmk). Une fois votre demande approuvée, vous devrez créer votre propre compte de stockage pour y stocker les données requises pour la personnalisation et la journalisation. Lorsque vous ajoutez un compte de stockage, la ressource du service Speech active une identité managée attribuée par le système. Une fois que l’identité managée affectée par le système est activée, cette ressource est inscrite auprès d’Azure Active Directory (AAD). Une fois inscrite, l’identité managée aura accès au compte de stockage. Vous trouverez ici plus d’informations sur les identités managées. Pour plus d’informations sur Identité managée, consultez [Que sont les identités managées ?](../../active-directory/managed-identities-azure-resources/overview.md).

> [!IMPORTANT]
> Si vous désactivez les identités managées attribuées par le système, l’accès au compte de stockage sera supprimé. Les composants du service Speech qui requièrent l’accès au compte de stockage cesseront de fonctionner.  

Actuellement, le service Speech ne prend pas en charge Customer Lockbox. Cependant, les données client peuvent être stockées à l'aide de BYOS, ce qui vous permet d'effectuer des contrôles de données semblables à ceux de [Customer Lockbox](../../security/fundamentals/customer-lockbox-overview.md). N'oubliez pas que les données du service Speech restent et sont traitées dans la région où la ressource Speech a été créée. Cela s'applique à toutes les données au repos ainsi qu'aux données en transit. Lorsque vous utilisez des fonctionnalités de personnalisation, telles que Custom Speech et Custom Voice, toutes les données client sont transférées, stockées et traitées dans la région où résident votre BYOS (si utilisé) et la ressource du service Speech.

> [!IMPORTANT]
> Microsoft **n'utilise pas** les données client pour améliorer ses modèles vocaux. En outre, si la journalisation des points de terminaison est désactivée et qu'aucune personnalisation n'est utilisée, aucune donnée client n'est stockée. 

## <a name="next-steps"></a>Étapes suivantes

* [Formulaire de demande Service Speech - bring your own storage (BYOS)](https://aka.ms/cogsvc-cmk)
* [Que sont les identités managées ?](../../active-directory/managed-identities-azure-resources/overview.md)
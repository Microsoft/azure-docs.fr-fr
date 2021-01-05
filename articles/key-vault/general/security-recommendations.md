---
title: Recommandations de sécurité pour Azure Key Vault
description: Recommandations de sécurité pour Azure Key Vault. La mise en œuvre de ces conseils vous aidera à répondre à vos obligations de sécurité comme décrit dans notre modèle de responsabilité partagée.
services: key-vault
author: msmbaldwin
manager: rkarlin
ms.service: key-vault
ms.subservice: general
ms.topic: article
ms.date: 09/30/2019
ms.author: mbaldwin
ms.custom: security-recommendations
ms.openlocfilehash: f96df14fb754578b58fb67ac116ac79cff3ffdcb
ms.sourcegitcommit: 7863fcea618b0342b7c91ae345aa099114205b03
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93289848"
---
# <a name="security-recommendations-for-azure-key-vault"></a>Recommandations de sécurité pour Azure Key Vault

Cet article contient des recommandations de sécurité pour Azure Key Vault. Mettez en œuvre ces recommandations pour répondre à vos obligations de sécurité comme décrit dans notre modèle de responsabilité partagée. Pour plus d’informations sur les mesures prises par Microsoft pour répondre à ses responsabilités de fournisseur de services, lisez [Responsabilités partagées pour le cloud computing](https://gallery.technet.microsoft.com/Shared-Responsibilities-81d0ff91).

Certaines recommandations contenues dans cet article peuvent être supervisées automatiquement par Azure Security Center. Azure Security Center est la première ligne de défense dans la protection de vos ressources Azure. Il analyse régulièrement l’état de sécurité de vos ressources Azure pour identifier les vulnérabilités potentielles. Il fournit ensuite des recommandations sur la façon de les corriger.

- Pour plus d’informations sur les recommandations Azure Security Center, consultez [Recommandations de sécurité dans Azure Security Center](../../security-center/security-center-recommendations.md).
- Pour plus d’informations sur Azure Security Center, consultez [Qu’est-ce qu’Azure Security Center ?](../../security-center/security-center-introduction.md)

## <a name="data-protection"></a>Protection de données

| Recommandation | Commentaires | Security Center |
|-|----|--|
|Activer la suppression réversible | La [suppression réversible](soft-delete-overview.md) permet de récupérer des coffres et des objets de coffre supprimés |  - |
| Limiter l’accès aux données du coffre  | Suivez le principe du moindre privilège et limitez les membres de votre organisation qui ont accès aux données du coffre. |  - |

## <a name="identity-and-access-management"></a>Gestion de l’identité et de l’accès

| Recommandation | Commentaires | Security Center |
|-|----|--|
| Limiter le nombre d’utilisateurs avec accès contributeur | Si un utilisateur dispose d’autorisations Contributeur pour le plan de gestion d’un coffre de clés, il peut s’accorder à lui-même l’accès au plan de données en définissant une stratégie d’accès Key Vault. Vous devez contrôler étroitement qui a un accès de rôle Contributeur à vos coffres de clés. Assurez-vous que seules les personnes autorisées ayant besoin d’un accès peuvent accéder à vos coffres et les gérer. Vous pouvez lire [Sécuriser l’accès à un coffre de clés](secure-your-key-vault.md). | - |

## <a name="monitoring"></a>Surveillance

| Recommandation | Commentaires | Security Center |
|-|----|--|
 Les journaux de diagnostic doivent être activés dans Key Vault | Activez les journaux d’activité et conservez-les un an maximum. Permet de recréer les pistes d’activité à des fins d’investigation en cas d’incident de sécurité ou de compromission du réseau. | [Oui](../../security-center/security-center-identity-access.md) |
| Limiter les utilisateurs autorisés à accéder à vos journaux Azure Key Vault. | Les [journaux Key Vault](logging.md) enregistrent les informations sur les activités effectuées dans votre coffre, telles que la création ou la suppression de coffres, de clés et de secrets, et peuvent être utilisés pendant une investigation. |  - |

## <a name="networking"></a>Mise en réseau

| Recommandation | Commentaires | Security Center |
|-|----|--|
|Limiter l’exposition réseau | L’accès réseau doit être limité aux réseaux virtuels utilisés par les solutions nécessitant un accès au coffre. Consultez les informations relatives aux [points de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md). | - |

## <a name="next-steps"></a>Étapes suivantes

Vérifiez auprès de votre fournisseur d’application pour voir s’il existe des exigences de sécurité supplémentaires. Pour plus d’informations sur le développement d’applications sécurisées, consultez [Documentation sur le développement sécurisé](https://azure.microsoft.com/resources/develop-secure-applications-on-azure/).
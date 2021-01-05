---
title: Refus d’accès réseau public – Portail Azure – Azure Database pour PostgreSQL – Serveur unique
description: Découvrez comment configurer et gérer le refus d’accès réseau public avec le Portail Azure pour votre serveur unique Azure Database pour PostgreSQL.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 03/10/2020
ms.openlocfilehash: bd777fe42dc8fa3ec4643fa9607f5ca8b7aac795
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240322"
---
# <a name="deny-public-network-access-in-azure-database-for-postgresql-single-server-using-azure-portal"></a>Refus d’accès réseau public dans Azure Database pour PostgreSQL – Serveur unique avec le Portail Azure

Cet article explique comment configurer un serveur unique Azure Database pour PostgreSQL de façon à refuser toutes les configurations publiques et à n’autoriser que les connexions établies via des points de terminaison privés afin d’améliorer la sécurité réseau.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

* Un [Serveur unique Azure Database pour PostgreSQL](quickstart-create-server-database-portal.md)

## <a name="set-deny-public-network-access"></a>Définition du refus d’accès réseau public

Suivez les étapes ci-dessous pour définir le refus d’accès réseau public pour un serveur unique PostgreSQL :

1. Sur le [Portail Azure](https://portal.azure.com/), sélectionnez votre serveur unique Azure Database pour PostgreSQL.

1. Sur la page du serveur unique PostgreSQL, cliquez sur **Sécurité des connexions** sous **Paramètres** afin d’ouvrir la page de configuration de la sécurité des connexions.

1. Dans **Refus d’accès réseau public** , sélectionnez **Oui** afin d’activer le refus d’accès réseau public pour votre serveur unique PostgreSQL.

    :::image type="content" source="./media/howto-deny-public-network-access/deny-public-network-access.PNG" alt-text="Azure Database pour PostgreSQL – Serveur unique – Refus d’accès réseau":::

1. Cliquez sur **Enregistrer** pour enregistrer les modifications.

1. Une notification confirme que le paramètre de sécurité des connexions a bien été activé.

    :::image type="content" source="./media/howto-deny-public-network-access/deny-public-network-access-success.png" alt-text="Azure Database pour PostgreSQL – Serveur unique – Refus d’accès réseau réussi":::

## <a name="next-steps"></a>Étapes suivantes

Découvrez [comment créer des alertes sur des métriques](howto-alert-on-metric.md).

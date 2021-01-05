---
title: Configuration TLS – Portail Azure – Azure Database for MariaDB
description: Découvrez comment paramétrer la configuration TLS à l’aide du portail Azure pour Azure Database for MariaDB.
author: mksuni
ms.author: sumuth
ms.service: mariadb
ms.topic: how-to
ms.date: 06/02/2020
ms.openlocfilehash: 55a664ceb3524a0c545e037dcba7a2af90034ad4
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93241982"
---
# <a name="configuring-tls-settings-in-azure-database-for-mariadb-using-azure-portal"></a>Configuration des paramètres TLS dans Azure Database for MariaDB à l’aide du portail Azure

Cet article explique comment configurer un serveur Azure Database for MariaDB pour imposer une version TLS minimale pour autoriser le trafic des connexions et refuser toutes les connexions dont la version TLS est inférieure à la version TLS minimale configurée, ce qui améliore la sécurité du réseau.

Vous pouvez imposer la version TLS pour la connexion à Azure Database for MariaDB en définissant la version TLS minimale pour leur serveur de base de données. Par exemple, la définition de la version minimale du paramètre TLS sur TLS 1.0 signifie que votre serveur autorisera les connexions en provenance de clients utilisant TLS 1.0, 1.1 et 1.2+. Si vous choisissez la valeur 1.2, en revanche, cela signifie que vous autorisez uniquement les connexions en provenance des clients utilisant TLS 1.2+ et que toutes les connexions à l’aide des protocoles TLS 1.0 et TLS 1.1 sont rejetées.

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce guide pratique, il vous faut :

* Une instance [Azure Database for MariaDB](quickstart-create-mariaDB-server-database-using-azure-portal.md)

## <a name="set-tls-configurations-for-azure-database-for-mariadb"></a>Définir des configurations TLS pour Azure Database for MariaDB

Procédez comme suit pour définir la version TLS minimale du serveur MariaDB :

1. Dans le [portail Azure](https://portal.azure.com/), sélectionnez votre serveur Azure Database for MariaDB existant.

1. Sur la page du serveur MariaDB, cliquez sur **Sécurité des connexions** sous **Paramètres** afin d’ouvrir la page de configuration de la sécurité des connexions.

1. Dans **Version TLS minimale** , sélectionnez **1.2** pour refuser les connexions dont la version TLS est antérieure à TLS 1.2 pour votre serveur MariaDB.

    ![Configuration TLS d’Azure Database for MariaDB](./media/howto-tls-configurations/tls-configurations.png)

1. Cliquez sur **Enregistrer** pour enregistrer les modifications.

1. Une notification confirme que le paramètre de sécurité de la connexion a bien été activé.

    ![Configuration TLS réussie pour Azure Database for MariaDB](./media/howto-tls-configurations/tls-configurations-success.png)

## <a name="next-steps"></a>Étapes suivantes

Découvrez [comment créer des alertes sur des métriques](howto-alert-metric.md)
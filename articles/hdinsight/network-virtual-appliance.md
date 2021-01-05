---
title: Configurer une appliance virtuelle réseau dans Azure HDInsight
description: Découvrez comment configurer un certain nombre de fonctionnalités supplémentaires pour votre appliance virtuelle réseau dans Azure HDInsight.
author: hrasheed-msft
ms.author: hrasheed
ms.reviewer: jasonh
ms.service: hdinsight
ms.topic: how-to
ms.date: 06/30/2020
ms.openlocfilehash: 557f0a6a37747d3a461ced8de16fd1fcf0d1abab
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95524111"
---
# <a name="configure-network-virtual-appliance-in-azure-hdinsight"></a>Configurer une appliance virtuelle réseau dans Azure HDInsight

> [!Important]
> Les informations suivantes sont requises **uniquement** si vous souhaitez configurer une appliance virtuelle réseau (NVA) autre que le [Pare-feu Azure](./hdinsight-restrict-outbound-traffic.md).

La balise FQDN du service Pare-feu Azure est automatiquement configurée pour autoriser le trafic dans la plupart des FQDN importants courants. L’utilisation d’une autre appliance virtuelle réseau vous oblige à configurer un certain nombre de fonctionnalités supplémentaires. Gardez les facteurs suivants à l’esprit pendant la configuration de votre appliance virtuelle réseau :

* Les services pouvant prendre en charge les points de terminaison de service peuvent être configurés avec des points de terminaison de service, ce qui entraîne le contournement du NVA, généralement pour des considérations relatives aux coûts ou aux performances.
* Si ResourceProviderConnection est défini sur *outbound* (sortant), vous pouvez utiliser des points de terminaison privés pour le stockage et des serveurs SQL pour les metastores, et il n’est pas nécessaire de les ajouter aux NVA (Appliances virtuelles réseau).
* Les dépendances d’adresses IP sont destinées au trafic non HTTP/S (à la fois le trafic TCP et UDP).
* Les points de terminaison HTTP/HTTPS avec des noms FQDN peuvent être approuvés dans votre dispositif NVA.
* Affectez la table de routage créée à votre sous-réseau HDInsight.

## <a name="service-endpoint-capable-dependencies"></a>Dépendances compatibles avec les points de terminaison de service

Vous pouvez éventuellement activer un ou plusieurs points de terminaison de service suivants, ce qui entraînera le contournement du NVA. Cette option peut être utile pour de grandes quantités de transferts de données afin de réduire les coûts et d’optimiser les performances. 

| **Point de terminaison** |
|---|
| Azure SQL |
| Stockage Azure |
| Azure Active Directory |

### <a name="ip-address-dependencies"></a>Dépendances des adresses IP

| **Point de terminaison** | **Détails** |
|---|---|
| Adresses IP publiées [ici](hdinsight-management-ip-addresses.md) | Ces adresses IP sont destinées au fournisseur de ressources HDInsight et doivent être incluses dans l’UDR pour éviter le routage asymétrique. Cette règle est requise uniquement si ResourceProviderConnection est défini sur *Inbound* (Entrant). Si ResourceProviderConnection est défini sur *Outbound* (Sortant), ces adresses IP ne sont pas nécessaires dans l’UDR.  |
| Adresses IP privées AAD-DS | Nécessaire uniquement pour les clusters ESP, si les réseaux virtuels ne sont pas appairés.|


### <a name="fqdn-httphttps-dependencies"></a>Dépendances HTTP/HTTPS FQDN

Vous pouvez obtenir la liste des dépendances de FQDN (principalement Stockage Azure et Azure Service Bus) pour configurer votre appliance virtuelle réseau (NVA) [dans ce référentiel](https://github.com/Azure-Samples/hdinsight-fqdn-lists/). La liste régionale est disponible [ici](https://github.com/Azure-Samples/hdinsight-fqdn-lists/tree/master/Regional). Ces dépendances sont utilisées par le fournisseur de ressources (RP) HDInsight pour créer et surveiller/gérer les clusters avec succès. Celles-ci incluent des journaux de télémétrie/diagnostic, des métadonnées d’approvisionnement, des configurations liées au cluster, des scripts, etc. Cette liste de dépendances de FQDN peut changer avec la publication des mises à jour futures de HDInsight.

La liste ci-dessous ne fournit que quelques noms de domaine complets (FQDN) qui peuvent être nécessaires pour les mises à jour correctives du système d’exploitation et de la sécurité, ou les validations de certificats *après* la création du cluster et pendant la durée de vie des opérations du cluster :

| **Dépendances d’exécution FQDN**                                                          |
|---|
| azure.archive.ubuntu.com:80                                           |
| security.ubuntu.com:80                                                |
| ocsp.msocsp.com:80                                                    |
| ocsp.digicert.com:80                                                  |
| microsoft.com/pki/mscorp/cps/default.htm:443                                      |
| microsoft.com:80                                                      |
|login.windows.net:443                                                  |
|login.microsoftonline.com:443                                          |

## <a name="next-steps"></a>Étapes suivantes

* [Utiliser un pare-feu pour limiter le trafic sortant](./hdinsight-restrict-outbound-traffic.md)
* [Architecture de réseau virtuel Azure HDInsight](hdinsight-virtual-network-architecture.md)

---
title: Versions d’Azure Key Vault
description: Les différentes versions d’Azure Key Vault
services: key-vault
author: msmbaldwin
ms.service: key-vault
ms.topic: conceptual
ms.date: 06/30/2019
ms.author: mbaldwin
ms.openlocfilehash: 2396aac67a17352987d9d6e3b49535c19ffe8917
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92792376"
---
# <a name="key-vault-versions"></a>Versions de Key Vault

Voici les nouveautés d’Azure Key Vault. Les nouvelles fonctionnalités et améliorations sont également annoncées sur le [canal des mises à jour d’Azure Key Vault](https://azure.microsoft.com/updates/?category=security&query=Key%20vault).

## <a name="june-2020"></a>Juin 2020

Azure Monitor pour Key Vault est désormais en préversion.  Azure Monitor permet une supervision complète de vos coffres de clés en fournissant une vue unifiée des informations relatives aux requêtes, au niveau de performance, aux échecs et à la latence de Key Vault. Pour plus d’informations, consultez [Azure Monitor pour Key Vault (préversion).](../../azure-monitor/insights/key-vault-insights-overview.md).

## <a name="may-2020"></a>Mai 2020

BYOK (« Bring Your Own Key ») pour Key Vault est désormais en disponibilité générale. Consultez la [spécification BYOK d’Azure Key Vault](../keys/byok-specification.md) et découvrez comment [importer des clés protégées par HSM dans Key Vault (BYOK)](../keys/hsm-protected-keys-byok.md).

## <a name="march-2020"></a>Mars 2020

Points de terminaison privés désormais disponibles en préversion. Le service Azure Private Link vous permet d’accéder à Azure Key Vault et aux services clients/partenaires hébergés sur Azure via un point de terminaison privé de votre réseau virtuel.  Découvrez comment [intégrer Key Vault avec Azure Private Link](private-link-service.md).

## <a name="2019"></a>2019

- Mise en production des kits SDK Azure Key Vault de nouvelle génération. Pour obtenir des exemples de leur utilisation, consultez les guides de démarrage rapide des secrets Azure Key Vault pour [Python](../secrets/quick-create-python.md), [.NET](../secrets/quick-create-net.md), [Java](../secrets/quick-create-java.md) et [Node.js](../secrets/quick-create-node.md)
- Nouvelles stratégies Azure pour la gestion des certificats de coffres de clés. Consultez les [définitions intégrées d’Azure Policy pour Key Vault](../policy-reference.md).
- L’extension de machine virtuelle Azure Key Vault est désormais en disponibilité générale.  Consultez [Extension de machine virtuelle Key Vault pour Linux](../../virtual-machines/extensions/key-vault-linux.md) et [Extension de machine virtuelle Key Vault pour Windows](../../virtual-machines/extensions/key-vault-windows.md).
- La gestion des secrets basée sur les événements pour Azure Key Vault est désormais disponible dans Azure Event Grid. Pour plus d’informations, consultez [le schéma Event Grid des événements dans Azure Key Vault] (../../event-grid/event-schema-key-vault.md], puis découvrez comment [recevoir et répondre aux notifications du coffre de clés avec Azure Event Grid](event-grid-tutorial.md).

## <a name="2018"></a>2018

Nouvelles fonctionnalités et intégrations mises en production cette année :

- Intégration avec Azure Functions. Pour obtenir un exemple de scénario montrant comment tirer profit d’[Azure Functions](../../azure-functions/index.yml) dans les opérations liées au coffre de clés, consultez [Automatiser la rotation d’un secret](../secrets/tutorial-rotation.md). 
- [Intégration avec Azure Databricks](/azure/databricks/scenarios/store-secrets-azure-key-vault). Ainsi, Azure Databricks prend désormais en charge deux types d’étendue de secrets : Azure Key Vault et Databricks. Pour plus d’informations, consultez [Créer une étendue de secrets Azure Key Vault](/azure/databricks/security/secrets/secret-scopes#--create-an-azure-key-vault-backed-secret-scope)
- [Points de terminaison de service de réseau virtuel pour Azure Key Vault](overview-vnet-service-endpoints.md).

## <a name="2016"></a>2016

Nouvelles fonctionnalités mises en production cette année :

- Clés de compte de stockage managé. Fonctionnalité des clés de compte de stockage ajoutée pour faciliter l’intégration avec le service Stockage Azure. Pour plus d’informations, consultez la rubrique relative à la [vue d’ensemble des clés de compte de stockage géré](../secrets/overview-storage-keys.md).
- Suppression réversible. Fonctionnalité de suppression réversible ajoutée pour améliorer la protection des données de vos coffres de clés et objets de coffre de clés. Pour plus d’informations, consultez la rubrique [Vue d’ensemble de la suppression réversible d’Azure Key Vault](./soft-delete-overview.md).

## <a name="2015"></a>2015

Nouvelles fonctionnalités mises en production cette année :
- Gestion de certificats. Ajoutée en tant que fonctionnalité à la version GA 2015-06-01 le 26 septembre 2016.

La disponibilité générale (version 2015-06-01) a été annoncée le 24 juin 2015. Les modifications suivantes ont été apportées à cette version : 
- Suppression d’une clé - champ « use » supprimé.
- Obtention des informations sur une clé - champ « use » supprimé.
- Importation d’une clé dans un coffre - champ « use » supprimé.
- Restauration d’une clé - champ « use » supprimé.     
- Remplacement de « RSA_OAEP » par « RSA-OAEP » dans les algorithmes RSA. Voir [Présentation des clés, des secrets et des certificats](about-keys-secrets-certificates.md).    
 
La deuxième préversion (version 2015-02-01-preview) a été annoncée le 20 avril 2015. Pour plus d’informations, consultez le billet de blog [REST API Update](/archive/blogs/kv/rest-api-update) (Mise à jour de l’API REST). Les tâches suivantes ont été mises à jour :
 
- Liste des clés d’un coffre (ajout de la prise en charge de la pagination à l’opération).
- Liste des versions d’une clé (ajout de l’opération pour dresser la liste des versions d’une clé).  
- Liste des secrets d’un coffre (ajout de la prise en charge de la pagination).
- Liste des versions d’un secret (ajout pour dresser la liste des versions d’un secret).  
- Toutes les opérations (ajout de timestamps de création/mise à jour aux attributs).  
- Création d’un secret (ajout d’un élément Content-Type aux secrets).
- Création d’une clé (ajout d’étiquettes en tant qu’informations facultatives).
- Création d’un secret (ajout d’étiquettes en tant qu’informations facultatives).
- Mise à jour d’une clé (ajout d’étiquettes en tant qu’informations facultatives).
- Mise à jour d’un secret (ajout d’étiquettes en tant qu’informations facultatives).
- Modification de la taille maximale des secrets de 10 Ko à 25 Ko. Voir [Présentation des clés, des secrets et des certificats](about-keys-secrets-certificates.md).    
 
## <a name="2014"></a>2014
 
La première préversion (version 2014-12-08-preview) a été annoncée le 8 janvier 2015.  
 
## <a name="next-steps"></a>Étapes suivantes

- [Présentation des clés, des secrets et des certificats](about-keys-secrets-certificates.md)
- [Clés](../keys/index.yml)
- [Secrets](../secrets/index.yml)
- [Certificates](../certificates/index.yml)
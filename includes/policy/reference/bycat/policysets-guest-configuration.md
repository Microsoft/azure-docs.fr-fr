---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 3f71eb1045f08a2eb6121c8c1c2ce86685606acf
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94986900"
---
|Nom |Description |Stratégies |Version |
|---|---|---|---|
|[Auditer les machines avec des paramètres de sécurité de mot de passe non sécurisés](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_WindowsPasswordSettingsAINE.json) |Cette initiative permet de déployer les exigences de stratégie et d’auditer les machines avec des paramètres de sécurité de mot de passe non sécurisés. Pour plus d’informations sur les stratégies Guest Configuration, consultez [https://aka.ms/gcpol](https://aka.ms/gcpol) |9 |1.0.0 |
|[Déployer les prérequis pour activer les stratégies Guest Configuration sur les machines virtuelles](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_Prerequisites.json) |Cette initiative ajoute une identité managée affectée par le système, et déploie l’extension Guest Configuration appropriée à la plateforme sur les machines virtuelles pouvant être supervisées par des stratégies Guest Configuration. Il s’agit d’un prérequis pour toutes les stratégies Guest Configuration, et il doit être affecté à l’étendue d’attribution des stratégies avant l’utilisation d’une stratégie Guest Configuration. Pour plus d’informations sur Guest Configuration, consultez [https://aka.ms/gcpol](https://aka.ms/gcpol). |4 |1.0.0-preview |
|[Les machines Windows doivent répondre aux exigences de la base de référence de sécurité Azure](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policySetDefinitions/Guest%20Configuration/GuestConfiguration_AzureBaseline.json) |Cette initiative audite les machines Windows avec des paramètres qui ne respectent pas la base de référence de sécurité Azure. Pour plus d’informations, consultez [https://aka.ms/gcpol](https://aka.ms/gcpol). |29 |2.0.0-preview |

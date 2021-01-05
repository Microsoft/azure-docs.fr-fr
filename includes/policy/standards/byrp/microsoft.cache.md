---
author: DCtheGeek
ms.service: azure-policy
ms.topic: include
ms.date: 11/20/2020
ms.author: dacoulte
ms.custom: generated
ms.openlocfilehash: 8aba76832117650c0833958362732e446755756f
ms.sourcegitcommit: 9889a3983b88222c30275fd0cfe60807976fd65b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94992287"
---
## <a name="azure-security-benchmark"></a>Benchmark de sécurité Azure

Le [benchmark de sécurité Azure](../../../../articles/security/benchmarks/overview.md) fournit des recommandations sur la façon dont vous pouvez sécuriser vos solutions cloud sur Azure. Pour voir comment ce service correspond totalement au benchmark de sécurité Azure, consultez les [fichiers de correspondance du benchmark de sécurité Azure](https://github.com/MicrosoftDocs/SecurityBenchmarks/tree/master/Azure%20Offer%20Security%20Baselines).

Pour passer en revue la façon dont les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – Benchmark de sécurité Azure](../../../../articles/governance/policy/samples/azure-security-benchmark.md).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protection des données |4.4 |Chiffrer toutes les informations sensibles en transit |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="hipaa-hitrust-92"></a>HIPAA HITRUST 9.2

Pour voir comment les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – HIPAA HITRUST 9.2](../../../../articles/governance/policy/samples/hipaa-hitrust-9-2.md).
Pour plus d’informations sur cette norme de conformité, consultez [HIPAA HITRUST 9.2](https://www.hhs.gov/hipaa/for-professionals/security/laws-regulations/index.html).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Contrôle de connexion réseau |0809.01n2Organizational.1234 - 01.n |Le trafic réseau est contrôlé conformément à la stratégie de contrôle d’accès de l’organisation par le biais du pare-feu et d’autres restrictions réseau pour chaque point d’accès réseau ou interface managée du service de télécommunication externe. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Contrôle de connexion réseau |0810.01n2Organizational.5 - 01.n |Les informations transmises sont sécurisées et, au minimum, chiffrées sur les réseaux publics ouverts. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Contrôle de connexion réseau |0811.01n2Organizational.6 - 01.n |Les exceptions à la stratégie de flux de trafic sont documentées avec un besoin métier/mission de prise en charge, la durée de l’exception, et sont examinées au moins une fois par an ; les exceptions à la stratégie de flux de trafic sont supprimées quand elles ne sont plus prises en charge par un besoin métier/mission explicite. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Contrôle de connexion réseau |0812.01n2Organizational.8 - 01.n |Les appareils distants qui établissent une connexion non distante ne sont pas autorisés à communiquer avec les ressources externes (distantes). |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Contrôle de connexion réseau |0814.01n1Organizational.12 - 01.n |La capacité des utilisateurs à se connecter au réseau interne est limitée à l’aide d’une stratégie de refus par défaut et d’autorisation par exception sur les interfaces gérées conformément à la stratégie de contrôle d’accès et aux exigences des applications cliniques et métier. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Identification des risques liés à des parties externes |1451.05iCSPOrganizational.2 - 05.i |Les fournisseurs de services cloud conçoivent et implémentent des contrôles pour atténuer et limiter les risques de sécurité des données grâce à une séparation correcte des tâches, à un accès en fonction du rôle et à un accès de moindre privilège pour l’ensemble du personnel au sein de leur chaîne d’approvisionnement. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Transactions en ligne |0946.09y2Organizational.14 - 09.y |L’organisation exige l’utilisation d’un chiffrement entre, et l’utilisation de signatures électroniques par, chacune des parties impliquées dans la transaction. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="nist-sp-800-171-r2"></a>NIST SP 800-171 R2

Pour voir comment les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – NIST SP 800-171 R2](../../../../articles/governance/policy/samples/nist-sp-800-171-r2.md).
Pour plus d’informations sur cette norme de conformité, consultez [NIST SP 800-171 R2](https://csrc.nist.gov/publications/detail/sp/800-171/rev-2/final).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protection du système et des communications |3.13.1 |Superviser, contrôler et protéger les communications (c’est-à-dire les informations transmises ou reçues par les systèmes de l’organisation) aux limites externes et aux limites internes clés des systèmes de l’organisation. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |
|Protection du système et des communications |3.13.8 |Implémenter des mécanismes de chiffrement pour empêcher toute divulgation non autorisée de CUI pendant la transmission, sauf en cas de protection par d’autres dispositifs de protection physique. |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |

## <a name="nist-sp-800-53-r4"></a>NIST SP 800-53 R4

Pour voir comment les composants intégrés Azure Policy disponibles pour tous les services Azure correspondent à ce standard de conformité, consultez [Conformité réglementaire Azure Policy – NIST SP 800-53 R4](../../../../articles/governance/policy/samples/nist-sp-800-53-r4.md).
Pour plus d’informations sur cette norme de conformité, consultez [NIST SP 800-53 R4](https://nvd.nist.gov/800-53).

|Domain |ID du contrôle |Titre du contrôle |Policy<br /><sub>(Portail Azure)</sub> |Version de la stratégie<br /><sub>(GitHub)</sub>  |
|---|---|---|---|---|
|Protection du système et des communications |SC-8 (1) |Confidentialité et intégrité des transmissions \| Protection par chiffrement ou protection physique différente |[Seules les connexions sécurisées à votre instance Azure Cache pour Redis doivent être activées](https://portal.azure.com/#blade/Microsoft_Azure_Policy/PolicyDetailBlade/definitionId/%2Fproviders%2FMicrosoft.Authorization%2FpolicyDefinitions%2F22bee202-a82f-4305-9a2a-6d7f44d4dedb) |[1.0.0](https://github.com/Azure/azure-policy/blob/master/built-in-policies/policyDefinitions/Cache/RedisCache_AuditSSLPort_Audit.json) |


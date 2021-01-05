---
title: 'Passerelle VPN Azure : Configurer la capture de paquets'
description: Découvrez la fonctionnalité de capture de paquets que vous pouvez utiliser sur les passerelles VPN pour cerner plus précisément la cause d’un problème.
services: vpn-gateway
author: radwiv
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/03/2020
ms.author: radwiv
ms.openlocfilehash: c6aa2b63344ef629d55f5a4efb66e4ee0e55975e
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94661033"
---
# <a name="configure-packet-capture-for-vpn-gateways"></a>Configurer la capture de paquets pour les passerelles VPN

Les problèmes de connectivité et de performance sont souvent complexes. Le simple fait de cerner la cause du problème peut demander beaucoup de temps et d’énergie. La capture de paquets peut vous aider à restreindre l’étendue d’un problème à certaines parties du réseau. Elle peut vous aider à déterminer si le problème se situe sur le réseau côté client, côté Azure ou entre les deux. Une fois que vous avez réussi à cerner le problème avec précision, vous pouvez déboguer et prendre des mesures correctives de manière plus efficace.

Il existe plusieurs outils de capture de paquets communément disponibles. L’obtention de captures de paquets pertinentes avec ces outils peut être fastidieuse, surtout dans des scénarios de trafic à volume élevé. Les capacités de filtrage fournies par la capture de paquets Azure VPN Gateway sont particulièrement utiles. Vous pouvez utiliser la capture de paquets VPN Gateway conjointement avec d’autres outils de capture de paquets courants.

## <a name="vpn-gateway-packet-capture-filtering-capabilities"></a>Fonctionnalités de filtrage de la capture de paquets VPN Gateway

Vous pouvez effectuer une capture de paquets VPN Gateway sur la passerelle ou sur une connexion spécifique, en fonction de vos besoins. Vous pouvez également effectuer une capture de paquets sur plusieurs tunnels à la fois. Vous pouvez capturer le trafic unidirectionnel ou bidirectionnel, le trafic IKE et ESP, et les paquets internes, ainsi que filtrer sur une passerelle VPN.

Utiliser un filtre à cinq tuples (sous-réseau source, sous-réseau de destination, port source, port de destination, protocole) et des indicateurs TCP (SYN, ACK, FIN, URG, PSH, RST) est pratique pour isoler les problèmes dans un trafic à volume élevé.

Les exemples suivants de code JSON et d’un schéma JSON permettent d’expliquer chaque propriété. Voici quelques limitations à prendre en compte quand vous effectuez des captures de paquets :
- Dans le schéma présenté ici, le filtre est un tableau, mais un seul filtre peut être utilisé à la fois actuellement.
- Vous ne pouvez pas effectuer plusieurs captures de paquets à l’échelle de la passerelle en même temps.
- Vous ne pouvez pas non plus effectuer plusieurs captures de paquets simultanément sur la même connexion. En revanche, vous pouvez effectuer plusieurs captures de paquets à la fois sur des connexions différentes.
- Cinq captures de paquets au maximum peuvent être exécutées en parallèle par passerelle. Ces captures de paquets peuvent être une combinaison de captures de paquets à l’échelle de la passerelle et de captures de paquets par connexion.

### <a name="example-json"></a>Exemple JSON
```JSON-interactive
{
  "TracingFlags": 11,
  "MaxPacketBufferSize": 120,
  "MaxFileSize": 200,
  "Filters": [
    {
      "SourceSubnets": [
        "20.1.1.0/24"
      ],
      "DestinationSubnets": [
        "10.1.1.0/24"
      ],
      "SourcePort": [
        500
      ],
      "DestinationPort": [
        4500
      ],
      "Protocol": [
        6
      ],
      "TcpFlags": 16,
      "CaptureSingleDirectionTrafficOnly": true
    }
  ]
}
```
### <a name="json-schema"></a>Schéma JSON
```JSON-interactive
{
    "type": "object",
    "title": "The Root Schema",
    "description": "The root schema input JSON filter for packet capture",
    "default": {},
    "additionalProperties": true,
    "required": [
        "TracingFlags",
        "MaxPacketBufferSize",
        "MaxFileSize",
        "Filters"
    ],
    "properties": {
        "TracingFlags": {
            "$id": "#/properties/TracingFlags",
            "type": "integer",
            "title": "The Tracingflags Schema",
            "description": "Tracing flags that customer can pass to define which packets are to be captured. Supported values are CaptureESP = 1, CaptureIKE = 2, CaptureOVPN = 8. The final value is OR of the bits.",
            "default": 11,
            "examples": [
                11
            ]
        },
        "MaxPacketBufferSize": {
            "$id": "#/properties/MaxPacketBufferSize",
            "type": "integer",
            "title": "The Maxpacketbuffersize Schema",
            "description": "Maximum buffer size of each packet. The capture will only contain contents of each packet truncated to this size.",
            "default": 120,
            "examples": [
                120
            ]
        },
        "MaxFileSize": {
            "$id": "#/properties/MaxFileSize",
            "type": "integer",
            "title": "The Maxfilesize Schema",
            "description": "Maximum file size of the packet capture file. It is a circular buffer.",
            "default": 100,
            "examples": [
                100
            ]
        },
        "Filters": {
            "$id": "#/properties/Filters",
            "type": "array",
            "title": "The Filters Schema",
            "description": "An array of filters that can be passed to filter inner ESP traffic.",
            "default": [],
            "examples": [
                [
                    {
                        "Protocol": [
                            6
                        ],
                        "CaptureSingleDirectionTrafficOnly": true,
                        "SourcePort": [
                            500
                        ],
                        "DestinationPort": [
                            4500
                        ],
                        "TcpFlags": 16,
                        "SourceSubnets": [
                            "20.1.1.0/24"
                        ],
                        "DestinationSubnets": [
                            "10.1.1.0/24"
                        ]
                    }
                ]
            ],
            "additionalItems": true,
            "items": {
                "$id": "#/properties/Filters/items",
                "type": "object",
                "title": "The Items Schema",
                "description": "An explanation about the purpose of this instance.",
                "default": {},
                "examples": [
                    {
                        "SourcePort": [
                            500
                        ],
                        "DestinationPort": [
                            4500
                        ],
                        "TcpFlags": 16,
                        "SourceSubnets": [
                            "20.1.1.0/24"
                        ],
                        "DestinationSubnets": [
                            "10.1.1.0/24"
                        ],
                        "Protocol": [
                            6
                        ],
                        "CaptureSingleDirectionTrafficOnly": true
                    }
                ],
                "additionalProperties": true,
                "required": [
                    "SourceSubnets",
                    "DestinationSubnets",
                    "SourcePort",
                    "DestinationPort",
                    "Protocol",
                    "TcpFlags",
                    "CaptureSingleDirectionTrafficOnly"
                ],
                "properties": {
                    "SourceSubnets": {
                        "$id": "#/properties/Filters/items/properties/SourceSubnets",
                        "type": "array",
                        "title": "The Sourcesubnets Schema",
                        "description": "An array of source subnets that need to match the Source IP address of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                "20.1.1.0/24"
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/SourceSubnets/items",
                            "type": "string",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": "",
                            "examples": [
                                "20.1.1.0/24"
                            ]
                        }
                    },
                    "DestinationSubnets": {
                        "$id": "#/properties/Filters/items/properties/DestinationSubnets",
                        "type": "array",
                        "title": "The Destinationsubnets Schema",
                        "description": "An array of destination subnets that need to match the Destination IP address of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                "10.1.1.0/24"
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/DestinationSubnets/items",
                            "type": "string",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": "",
                            "examples": [
                                "10.1.1.0/24"
                            ]
                        }
                    },
                    "SourcePort": {
                        "$id": "#/properties/Filters/items/properties/SourcePort",
                        "type": "array",
                        "title": "The Sourceport Schema",
                        "description": "An array of source ports that need to match the Source port of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                500
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/SourcePort/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                500
                            ]
                        }
                    },
                    "DestinationPort": {
                        "$id": "#/properties/Filters/items/properties/DestinationPort",
                        "type": "array",
                        "title": "The Destinationport Schema",
                        "description": "An array of destination ports that need to match the Destination port of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                4500
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/DestinationPort/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                4500
                            ]
                        }
                    },
                    "Protocol": {
                        "$id": "#/properties/Filters/items/properties/Protocol",
                        "type": "array",
                        "title": "The Protocol Schema",
                        "description": "An array of protocols that need to match the Protocol of a packet. Packet can match any one value in the array of inputs.",
                        "default": [],
                        "examples": [
                            [
                                6
                            ]
                        ],
                        "additionalItems": true,
                        "items": {
                            "$id": "#/properties/Filters/items/properties/Protocol/items",
                            "type": "integer",
                            "title": "The Items Schema",
                            "description": "An explanation about the purpose of this instance.",
                            "default": 0,
                            "examples": [
                                6
                            ]
                        }
                    },
                    "TcpFlags": {
                        "$id": "#/properties/Filters/items/properties/TcpFlags",
                        "type": "integer",
                        "title": "The Tcpflags Schema",
                        "description": "A list of TCP flags. The TCP flags set on the packet must match any flag in the list of flags provided. FIN = 0x01,SYN = 0x02,RST = 0x04,PSH = 0x08,ACK = 0x10,URG = 0x20,ECE = 0x40,CWR = 0x80. An OR of flags can be provided.",
                        "default": 0,
                        "examples": [
                            16
                        ]
                    },
                    "CaptureSingleDirectionTrafficOnly": {
                        "$id": "#/properties/Filters/items/properties/CaptureSingleDirectionTrafficOnly",
                        "type": "boolean",
                        "title": "The Capturesingledirectiontrafficonly Schema",
                        "description": "A flags which when set captures reverse traffic also.",
                        "default": false,
                        "examples": [
                            true
                        ]
                    }
                }
            }
        }
    }
}
```

## <a name="set-up-packet-capture-by-using-powershell"></a>Configurer la capture de paquets avec PowerShell

Les exemples ci-dessous montrent des commandes PowerShell qui démarrent et arrêtent les captures de paquets. Pour plus d’informations sur les options de paramètre, consultez [Start-AzVirtualnetworkGatewayPacketCapture](/powershell/module/az.network/start-azvirtualnetworkgatewaypacketcapture).

### <a name="start-packet-capture-for-a-vpn-gateway"></a>Démarrer la capture de paquets pour une passerelle VPN

```azurepowershell-interactive
Start-AzVirtualnetworkGatewayPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayName"
```

Vous pouvez utiliser le paramètre facultatif `-FilterData` pour appliquer un filtre.

### <a name="stop-packet-capture-for-a-vpn-gateway"></a>Arrêter la capture de paquets pour une passerelle VPN

```azurepowershell-interactive
Stop-AzVirtualNetworkGatewayPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayName" -SasUrl "YourSASURL"
```

### <a name="start-packet-capture-for-a-vpn-gateway-connection"></a>Démarrer la capture de paquets pour une connexion de passerelle VPN

```azurepowershell-interactive
Start-AzVirtualNetworkGatewayConnectionPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayConnectionName"
```

Vous pouvez utiliser le paramètre facultatif `-FilterData` pour appliquer un filtre.

### <a name="stop-packet-capture-on-a-vpn-gateway-connection"></a>Arrêter la capture de paquets sur une connexion de passerelle VPN

```azurepowershell-interactive
Stop-AzVirtualNetworkGatewayConnectionPacketCapture -ResourceGroupName "YourResourceGroupName" -Name "YourVPNGatewayConnectionName" -SasUrl "YourSASURL"
```

## <a name="key-considerations"></a>Considérations relatives aux clés

- Effectuer des captures de paquets peut impacter les performances. N’oubliez donc pas d’arrêter la capture de paquets quand vous n’en avez plus besoin.
- La durée de capture de paquets minimale suggérée est de 600 secondes. En raison de problèmes de synchronisation entre plusieurs composants sur le chemin, les captures de paquets d’une durée plus courte risquent de fournir des données incomplètes.
- Les fichiers de données de capture de paquets sont générés au format PCAP. Utilisez Wireshark ou d’autres applications couramment disponibles pour ouvrir les fichiers PCAP.
- Les captures de paquets ne sont pas prises en charge sur les passerelles basées sur des stratégies.
- Si le paramètre `SASurl` n’est pas configuré correctement, la trace peut échouer avec des erreurs de stockage. Pour obtenir des exemples sur la façon de générer correctement un paramètre `SASurl`, consultez [Stop-AzVirtualNetworkGatewayPacketCapture](/powershell/module/az.network/stop-azvirtualnetworkgatewaypacketcapture).



## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la passerelle VPN, consultez [Qu’est-ce qu’une passerelle VPN ?](vpn-gateway-about-vpngateways.md).
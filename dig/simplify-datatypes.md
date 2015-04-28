
code=====string
oid=uri=string



基本类型
boolean
integer
string
decimal
uri
base64Binary
instant
date
dateTime
time

扩展
| extension | ---------- | 否0..* | ---------- |
| extension.url | uri | 是1..1 | ---------- |
| extension.value[x] | ---------- | 否0..1 | x可以是Integer、Decimal、DateTime、Date、Instant、String、Uri、Boolean、Code、Base64Binary、Coding、CodeableConcept、Attachment、Identifier、Quantity、Range、Period、Ratio、HumanName、Address、ContactPoint、Timing、Signature、Reference |

| modifierExtension | ---------- | 否0..* | ---------- |
| modifierExtension.url | uri | 是1..1 | ---------- |
| modifierExtension.value[x] | ---------- | 否0..1 | x可以是Integer、Decimal、DateTime、Date、Instant、String、Uri、Boolean、Code、Base64Binary、Coding、CodeableConcept、Attachment、Identifier、Quantity、Range、Period、Ratio、HumanName、Address、ContactPoint、Timing、Signature、Reference |



coding 类型
| ResourceType.coding | |----------| 否0..* | ---------- |
| ResourceType.coding.system | string | 否0..1 | xs:anyURI |
| ResourceType.coding.version | string | 否0..1 | ---------- |
| ResourceType.coding.code | code | 否0..1 | ---------- |
| ResourceType.coding.display | string| 否0..1 | ---------- |

Narrative 类型
| ResourceType.Narrative | |----------| 否0..* | ---------- |
| ResourceType.Narrative.status | string | 是1..1 | code |
| ResourceType.Narrative.div | string | 是1..1 | xhtml |

Identifier 类型

| Identifier.use | code | 否0..1 | ---------- |
| Identifier.type | CodeableConcept | 否0..1 |  |
| Identifier.type.coding | ----------| 否0..* | ---------- |
| Identifier.type.coding.system | string | 否0..1 | xs:anyURI |
| Identifier.type.coding.version | string | 否0..1 | ---------- |
| Identifier.type.coding.code | code | 否0..1 | ---------- |
| Identifier.type.coding.display | string| 否0..1 | ---------- |
| Identifier.type.text | string | 否0..1 | ---------- |
| Identifier.system | string  | 否0..1 | uri |
| Identifier.value | string | 否0..1 |  |
| Identifier.period | ---------- | 否0..1 | ---------- |
| Identifier.period.start | dateTime | 否0..1 | ---------- |
| Identifier.period.end | dateTime | 否0..1 |  |
| Identifier.assigner | ---------- | 否0..1 | Organization引用 |
| Identifier.assigner.reference | string | 否0..1 | ---------- |
| Identifier.assigner.display | string | 否0..1 |  |

period 类型
|  Period.start | dateTime | 否0..1 | ---------- |
|  Period.end | dateTime | 否0..1 |  |

CodeableConcept 类型

| ResourceType.CodeableConcept.coding |----------| |----------| 否0..* | ---------- |
| ResourceType.CodeableConcept.coding.system | string | 否0..1 | xs:anyURI |
| ResourceType.CodeableConcept.coding.version | string | 否0..1 | ---------- |
| ResourceType.CodeableConcept.coding.code | code | 否0..1 | ---------- |
| ResourceType.CodeableConcept.coding.display | string| 否0..1 | ---------- |
| ResourceType.CodeableConcept.text | string | 否0..1 | ---------- |

Reference 类型
| Reference.reference | string | 否0..1 | ---------- |
| Reference.display | string | 否0..1 |  |


基本resource和domainResource的字段整合
| 字段名称 | 数据类型 | 必须存在 | 说明 |
| ResourceType | ---------- | ---------- | ---------- |
| ResourceType.id | string | 否0..1 | ---------- |
| ResourceType.meta | ---------- | 否0..1 | ---------- |
| ResourceType.meta.versionId | string | 否0..1 | ---------- |
| ResourceType.meta.lastUpdated | instant | 否0..1 | ---------- |
| ResourceType.meta.profile | uri | 否0..* | ---------- |
| ResourceType.meta.security |----------  | 否0..* | ---------- |
| ResourceType.meta.security.system | uri | 否0..1 | ---------- |
| ResourceType.meta.security.version | string | 否0..1 | ---------- |
| ResourceType.meta.security.code | code | 否0..1 | ---------- |
| ResourceType.meta.security.display | string| 否0..1 | ---------- |
| ResourceType.meta.security.primary | boolean | 否0..1 | ---------- |
| ResourceType.meta.tag | |----------| 否0..* | ---------- |
| ResourceType.meta.tag.system | uri | 否0..1 | ---------- |
| ResourceType.meta.tag.version | string | 否0..1 | ---------- |
| ResourceType.meta.tag.code | code | 否0..1 | ---------- |
| ResourceType.meta.tag.display | string| 否0..1 | ---------- |
| ResourceType.meta.tag.primary | boolean | 否0..1 | ---------- |
| ResourceType.implicitRules | uri | 否0..1 | ---------- |
| ResourceType.language | string | 否0..1 |  默认为zh |
| ResourceType.text | ---------- | 否0..1 | ---------- |
| ResourceType.text.status | string | 是1..1 | code |
| ResourceType.text.div | string | 是1..1 | xhtml |
| ResourceType.contained | ResourceType | 否0..* | ---------- |
| ResourceType.extension | ---------- | 否0..* | ---------- |
| ResourceType.extension.url 	 | ---------- | 是1..1 | ---------- |
| ResourceType.extension.value[x] 	 | ---------- | 否0..1 | x可以是Integer、Decimal、DateTime、Date、Instant、String、Uri、Boolean、Code、Base64Binary、Coding、CodeableConcept、Attachment、Identifier、Quantity、Range、Period、Ratio、HumanName、Address、ContactPoint、Timing、Signature、Reference |
| ResourceType.modifierExtension | ---------- | 否0..* | ---------- |
| ResourceType.modifierExtension.url 	 | ---------- | 是1..1 | ---------- |
| ResourceType.modifierExtension.value[x] 	 | ---------- | 否0..1 | x可以是Integer、Decimal、DateTime、Date、Instant、String、Uri、Boolean、Code、Base64Binary、Coding、CodeableConcept、Attachment、Identifier、Quantity、Range、Period、Ratio、HumanName、Address、ContactPoint、Timing、Signature、Reference |

---
title: "JSONSchema"
date: 2021-12-05T11:30:03+00:00
tags: ["json"]
series: []
categories: ["数据结构"]
showToc: true
TocOpen: false
draft: false
hidemeta: false
comments: false
description: "Desc Text."
canonicalURL: "https://canonical.url/to/page"
disableHLJS:  false
disableShare: false
disableHLJS: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
---

Document Ref [JSONSchema](http://json-schema.org/)，[Validation](https://json-schema.org/draft/2020-12/json-schema-validation.html)

# 1.什么是JSONSchema

**JSON Schema** 是一个允许您**注释** 和**验证** JSON 文档的词汇表。

# 2.构建JSONSchema

## 2.1 Example

```json
{
"productId": 1,
"productName": "A green door",
"price": 12.50,
"tags": [ "home", "green" ]
}
```

如上json有一些问题，比如 `productId`是什么,`productName`是什么,如何验证里面的字段,`price`可不可以为0，上述JSON只是一份不完备的JSON文档

## 2.2 开始构建

```
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product in the catalog",
  "type": "object"
}
```

- `$schema`:  根据哪个json草案/标准进行编写的，还有版本信息
- `$id`:  定义了Schema的 URI，以及Schema内其他 URI 引用解析所依据的基本 URI
- `title,description`: 描述这个JSON文档的一些信息
- `type`: 这个字段是对JSON文档的第一个`约束`，如上表明json文档为对象类型

type还有如下几种类型

| 类型 | 描述 |
| - | - |
| null | 空 |
| boolean | 布尔 |
| object | 对象 |
| array | 数组 |
| number | 数字 |
| string | 字符串 |
| integer | 数字 |

## 2.3 属性

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    }
  },
  "required": [ "productId" ]
}
```

- 使用`properties`定义属性，属性必须为一个对象，此对象的每个值都必须是有效的 JSON 模式。
- `productId`是对象中的一个属性，拥有类型和描述两个字段
- `required`的值必须是一个数组，如上表明`productId`是必须要验证的

## 2.4 更多限制

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

- price增加了`exclusiveMinimum`，`exclusiveMinimum`装饰的属性的值必须不是零。
- 如果你需要包含0，那么可以使用`minimum`最小值来控制

## 2.5 验证Numeric类型

- multipleOf 其中之一
- maximum 最大限度
- exclusiveMaximum 不包含最大限度
- minimum 最小限度
- exclusiveMinimum 不包含最小限度

## 2.6 验证 String类型

- maxLength 最大长度
- minLength 最小长度
- pattern 正则

## 2.7 验证 array 类型

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

- maxItems 最大数量
- minItems 最小数量
- uniqueItems 每个元素是不是都是唯一的
- maxContains
- minContains

## 2.8 验证 Objects 类型

```json
{
  "$id": "https://example.com/address.schema.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "description": "An address similar to http://microformats.org/wiki/h-card",
  "type": "object",
  "properties": {
    "post-office-box": {
      "type": "string"
    },
    "extended-address": {
      "type": "string"
    },
    "street-address": {
      "type": "string"
    },
    "locality": {
      "type": "string"
    },
    "region": {
      "type": "string"
    },
    "postal-code": {
      "type": "string"
    },
    "country-name": {
      "type": "string"
    }
  },
  "required": [ "locality", "region", "country-name" ],
  "dependentRequired": {
    "post-office-box": [ "street-address" ],
    "extended-address": [ "street-address" ]
  }
}
```

- maxProperties 最大属性数量
- minProperties 最小属性数量
- required 必须有的属性列表
- dependentRequired 标记属性需要哪些属性依赖

## 2.9嵌套数据结构

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    },
    "dimensions": {
      "type": "object",
      "properties": {
        "length": {
          "type": "number"
        },
        "width": {
          "type": "number"
        },
        "height": {
          "type": "number"
        }
      },
      "required": [ "length", "width", "height" ]
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

- dimensions 作为 `properties`其中还有一层`properties`和`required`

## 2.10引用外部定义Schema

```json
{
  "$id": "https://example.com/geographical-location.schema.json",
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "title": "Longitude and Latitude",
  "description": "A geographical coordinate on a planet (most commonly Earth).",
  "required": [ "latitude", "longitude" ],
  "type": "object",
  "properties": {
    "latitude": {
      "type": "number",
      "minimum": -90,
      "maximum": 90
    },
    "longitude": {
      "type": "number",
      "minimum": -180,
      "maximum": 180
    }
  }
}
```

如上定义了一个`object`

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "$id": "https://example.com/product.schema.json",
  "title": "Product",
  "description": "A product from Acme's catalog",
  "type": "object",
  "properties": {
    "productId": {
      "description": "The unique identifier for a product",
      "type": "integer"
    },
    "productName": {
      "description": "Name of the product",
      "type": "string"
    },
    "price": {
      "description": "The price of the product",
      "type": "number",
      "exclusiveMinimum": 0
    },
    "tags": {
      "description": "Tags for the product",
      "type": "array",
      "items": {
        "type": "string"
      },
      "minItems": 1,
      "uniqueItems": true
    },
    "dimensions": {
      "type": "object",
      "properties": {
        "length": {
          "type": "number"
        },
        "width": {
          "type": "number"
        },
        "height": {
          "type": "number"
        }
      },
      "required": [ "length", "width", "height" ]
    },
    "warehouseLocation": {
      "description": "Coordinates of the warehouse where the product is located.",
      "$ref": "https://example.com/geographical-location.schema.json"
    }
  },
  "required": [ "productId", "productName", "price" ]
}
```

- `warehouseLocation`使用此属性*名称*来包含外部JSONSchema


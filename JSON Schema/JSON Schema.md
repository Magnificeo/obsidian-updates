> [!note|badge]
> JSON Schema - это спецификация для описания структуры JSON-документов.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "array",
  "items": {
    "type": "object",
    "properties": {
      "id": {
        "type": "string"
      },
      "name": {
        "type": "string"
      },
      "age": {
        "type": "integer",
        "minimum": 18
      },
      "addresses": {
        "type": "array",
        "items": {
          "type": "object",
          "properties": {
            "street": {
              "type": "string"
            },
            "city": {
              "type": "string"
            },
            "postalCode": {
              "type": "string",
              "pattern": "^[0-9]{5}$"
            }
          },
          "required": ["street", "city", "postalCode"]
        }
      }
    },
    "required": ["id", "name", "age"]
  }
}
```

![[Pasted image 20250110092001.png]]
![[Pasted image 20250110094257.png]]
![[Pasted image 20250110092101.png]]

> [!note|badge]
> `$schema` - это версия спецификации.

# AnyOf, AllOf, OneOf

# Чтобы одно из нескольких условий было выполнено - `anyOf`

```json
{
  "type": "object",
  "properties": {
    "price": {
      "anyOf": [{ "type": "number" }, { "type": "string" }]
    }
  }
}
```

# `oneOf` - строго одно

> [!note|badge]
> Если взять предыдущий пример и заменить anyOf на oneOf, то не будет никакой разницы.

> [!tip|badge]
> `anyOf` позволяет объекту соответствовать нескольким схемам одновременно, тогда как `oneOf` требует, чтобы объект соответствовал только одной из указанных схем.

```json
{
  "oneOf": [
    {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "age": { "type": "integer" }
      },
      "required": ["name", "age"]
    },
    {
      "type": "object",
      "properties": {
        "brand": { "type": "string" },
        "model": { "type": "string" }
      },
      "required": ["brand", "model"]
    }
  ]
}
```

В этом случае объект будет валидным, если он является либо человеком, либо автомобилем, но не может содержать атрибуты обоих. Если бы было `anyOf`, то объект мог иметь все 4 свойства.

#  Чтобы выполнялись все условия - `allOf`

```json
{
  "type": "object",
  "properties": {
    "product": {
      "allOf": [
        { "type": "string" },
        { "minLength": 3 },
        { "pattern": "^[A-Z].*$" }
      ]
    }
  }
}
```
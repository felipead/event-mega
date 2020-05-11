<p align="center">
    <img alt="MEGA" src="https://github.com/mega-distributed/event-mega/raw/master/resources/logo/mega_logo_large.png">
</p>

---

This is the MEGA protocol for event-streaming. Implementations:

- [SQS MEGA](https://github.com/mega-distributed/sqs-mega)

## The MEGA protocol

All MEGA events must be encoded using JSON format. The following attributes are expected:


|  Attribute         | Required?    | Type     | Description |
| ------------------ |:------------:|:--------:| ----------- |
| `protocol`         | required     | string   | Should contain the `mega` string. Necessary in order to allow MEGA events to co-exist and distinguish themselves from other events or protocols. |
| `version`          | required     | integer  | Specifies the version of the protocol. Here we consciously avoid the unnecessary complexities of [semantic versioning](https://semver.org). The current version is 1, and this number will only increase if backward incompatible changes are introduced. |
| `event.name`       | required     | string   | The name of the event, which will be used for subscribers to match. Can be any lower-cased alphanumeric string and can include the following special characters: `.`, `-`, `_`. It is advisable to use a full-qualified name, with a namespace specifing the name of the domain. For example, `shopping_cart.item.added`. |
| `event.timestamp`  | required     | [ISO-9601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) datetime | Timestamp that indicates when the event happened. |
| `event.version`    | _optional_   | integer  | This is the version of the event, and defaults to 1. It is highly recommended that your events are versioned, to allow breaking changes to be introduced more easily in the future. However, we also want to avoid the complexities of semantic versioning and only increase the version when backward incompatible changes are needed. Design your event subscribers to be [tolerant readers](https://martinfowler.com/bliki/TolerantReader.html). This will allow your architecture to evolve more easily. |
| `event.publisher`  | _optional_   | string   | Any string that can identify the publisher of the event, like the name of a service,  system or domain. |
| `event.subject`    | _optional_   | string   | The subject to which the event refers to. It is usually an entity identifier, like the database primary key. For example, if the event is about an item being added to the user's shopping cart in an e-commerce system, the subject could be the shopping cart ID or even the user ID. |
| `event.attributes` | _optional_   | object   | Dictionary of application-specific event attributes. Keep it minimal and only send the data that is specific to the event. These attributes will be used for pattern matching with subscribers. |
| `event.entity`     | _optional_   | object   | Current representation of the object or entity that this event refers to. For example, if the event is about an item that was added to the shopping cart, we can use this attribute to transmit the full contents of the shopping cart. |
| `event.extra`      | _optional_   | object   | Any further application data or metadata that can be associated with this event. |

Example:

```json
{
    "protocol": "mega",
    "version": 1,
    "event": {
        "name": "shopping_cart.item.added",
        "timestamp": "2020-05-04T15:53:23.123",
        "version": 1,
        "publisher": "shopping_cart",
        "subject": "987650",
        "attributes": {
            "item_id": "61fcc874-624e-40f8-8fd7-0e663c7837e8",
            "quantity": 5
        },
        "entity": {
            "id": "18a3f92e-1fbf-45eb-8769-d836d0a1be55",
            "user_id": 987650,
            "items": [
                {
                    "id": "61fcc874-624e-40f8-8fd7-0e663c7837e8",
                    "price": "19.99",
                    "quantity": 10
                },
                {
                    "id": "3c7f8798-1d3d-47de-82dd-c6c5e0de74ee",
                    "price": "102.50",
                    "quantity": 1
                }
                {
                    "id": "bba76edc-8afc-4fde-b4c4-ea58a230c5d6",
                    "price": "24.99",
                    "quantity": 3
                }
            ],
            "currency": "USD",
            "value": "377.37",
            "discount": "20.19",
            "subtotal": "357.18",
            "estimated_shipping": "10.00",
            "estimated_tax": "33.05",
            "estimated_total": "400.23",
            "created_at": "2020-05-03T12:20:23.000",
            "updated_at": "2020-05-04T15:52:01.000"
        },
        "extra": {
            "channel": "website",
            "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1 Safari/605.1.15",
            "user_ip_address": "177.182.205.103"
        }
    }
}
```

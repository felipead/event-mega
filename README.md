<p align="center">
    <img alt="MEGA" src="https://github.com/mega-distributed/mega-event-protocol/raw/master/resources/logo/mega_logo_large.png">
</p>

---

This is the MEGA protocol for event-streaming. Implementations:

- [SQS MEGA](https://github.com/mega-distributed/sqs-mega)

## The MEGA event-streaming protocol

MEGA uses JSON as its data-interchange and object representation format. Most implementations also supports [BJSON](http://bsonspec.org) (Binary JSON), which is more compact and efficient.

To ease adoption, MEGA implementations should be tolerant to events or messages
that do not adhere to the protocol. The idea is that MEGA can happily co-exist
with your existing architecture or legacy systems. In order to accomplish this,
it uses the following JSON attribute to identify a MEGA payload:

```
"protocol": "mega"
```

A MEGA event can be very simple:
```json
{
    "protocol": "mega",
    "version": 1,
    "event": {
        "name": "user.created",
        "timestamp": "2020-05-04T15:53:27.823",
        "subject": "987650"
    }
}
```

It can also be more useful:
```json
{
    "protocol": "mega",
    "version": 1,
    "event": {
        "name": "user.updated",
        "timestamp": "2020-05-04T15:53:27.823",
        "version": 1,
        "subject": "987650",
        "publisher": "user",
        "attributes": {
            "username": "john.doe"
        }
    },
    "object": {
        "current": {
            "id": 987650,
            "name": "John Doe",
            "username": "john.doe",
            "ssn": "497279436",
            "birthdate": "1986-02-15"
        }
    }
}
```

MEGA can also be as much feature-rich as you would like:
```json
{
    "protocol": "mega",
    "version": 1,
    "event": {
        "name": "shopping_cart.item.added",
        "version": 1,
        "timestamp": "2020-05-04T15:53:23.123",
        "publisher": "shopping-cart-service",
        "subject": "987650",
        "attributes": {
            "item_id": "61fcc874-624e-40f8-8fd7-0e663c7837e8",
            "quantity": 5
        }
    },
    "object": {
        "name": "shopping_cart",
        "version": 2,
        "current": {
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
        "previous": {
            "id": "18a3f92e-1fbf-45eb-8769-d836d0a1be55",
            "user_id": 987650,
            "items": [
                {
                    "id": "61fcc874-624e-40f8-8fd7-0e663c7837e8",
                    "price": "19.99",
                    "quantity": 5
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
            "value": "277.42",
            "discount": "10.09",
            "subtotal": "267.33",
            "estimated_shipping": "10.00",
            "estimated_tax": "24.96",
            "estimated_total": "302.29",
            "created_at": "2020-05-03T12:20:23.000",
            "updated_at": "2020-05-04T13:47:08.000"
        }
    },
    "extra": {
        "channel": "website",
        "user_agent": "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_4) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/13.1 Safari/605.1.15",
        "user_ip_address": "177.182.205.103"
    }
}
```

Here is a list of all supported attributes:

|  Attribute         | Required?    | Type     | Description |
| ------------------ |:------------:|:--------:| ----------- |
| `protocol`         | required     | string   | Should contain the `mega` string. Necessary in order to allow MEGA events to co-exist and distinguish themselves from other events or protocols. |
| `version`          | required     | integer  | Specifies the version of the protocol. Do not confuse it with the version of the event or object payload. Here we consciously avoid the unnecessary complexities of [semantic versioning](https://semver.org). The current version is 1, and this number will only increase if backward incompatible changes are introduced. |
| `event.name`       | required     | string   | The name of the event, which will be used for subscribers to match. Can be any lower-cased alphanumeric string and can include the following special characters: `.`, `-`, `_`. It is advisable to use a full-qualified name, with a namespace specifying the name of the domain. For example, `shopping_cart.item.added`. |
| `event.timestamp`  | required     | [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) | Timestamp that indicates when the event happened. If no timezone is specified, UTC will be used. |
| `event.version`    | _optional_   | integer  | This is the version of the event, and defaults to 1. It is highly recommended that your events are versioned, to allow breaking changes to be introduced more easily in the future. However, we also want to avoid the complexities of semantic versioning and only increase the version when backward incompatible changes are needed. Design your event subscribers to be [tolerant readers](https://martinfowler.com/bliki/TolerantReader.html). This will allow your architecture to evolve more easily. |
| `event.publisher`  | _optional_   | string   | Any string that can identify the publisher of the event, like the name of a service,  system or domain. |
| `event.subject`    | _optional_   | string   | The subject to which the event refers to. It is usually an entity identifier, like the database primary key. For example, if the event is about an item being added to the user's shopping cart in an e-commerce system, the subject could be the shopping cart ID or even the user ID. |
| `event.attributes` | _optional_   | object   | Dictionary of application-specific event attributes. Keep it minimal and only send the data that is specific to the event. These attributes will be used for pattern matching with subscribers. |
| `object.current`   | _optional_   | object   | Current representation of the object or entity that this event refers to. For example, if the event is about an item that was added to the shopping cart, we can use this attribute to transmit the full contents of the shopping cart. |
| `object.previous`  | _optional_   | object   | Previous representation of the object or entity that this event refers to. This is useful to transmit the state of the entity right before the event happened, if such information is available. |
| `object.name`      | _optional_   | string   | If you want, you can give a name to identify your object or entity. |
| `object.version`   | _optional_   | object   | This is the version of the object or entity representation. Here we give you the opportunity to version your objects differently than your events. This is because entities change more frequently, while events do not change so often. If not specified, it defaults to 1. |
| `extra`            | _optional_   | object   | Any further application data or metadata that can be associated with this event. |

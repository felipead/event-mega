<p align="center">
    <img alt="MEGA" src="https://github.com/mega-distributed/event-mega/raw/master/resources/logo/mega_logo_large.png">
</p>

---

## The Event MEGA protocol

- `protocol` (_required_, string): should contain the `MEGA` string.
- `version` (_required_, integer): specifies the version of the protocol. Here we consciously avoid the unnecessary complexities of [semantic versioning](https://semver.org). The current version is 1, and this number will only increase if backward incompatible changes are introduced.
- `event.name` (_required_, string): the name of the event, which will be used for subscribers to match. Can be any lower-cased alphanumeric string and can include the following special characters: `.`, `-`, `_`.
- `event.timestamp` (_required_, datetime): a [ISO-9601](https://en.wikipedia.org/wiki/ISO_8601#Combined_date_and_time_representations) timestamp that indicates when the event happened.
- `event.version` (_optional_, integer): this is the version of the event. It is highly recommended that your events are versioned, to allow breaking changes to be introduced more easily in the future. However, we also want to avoid the complexities of semantic versioning and only increase the version when backward incompatible changes are needed. Design your event subscribers to be [tolerant readers](https://martinfowler.com/bliki/TolerantReader.html). This will allow your architecture to evolve more easily.
- `event.publisher` (_optional_, string): any string that can identify the publisher of the event, like the name of a service,  system or domain.
- `event.subject` (_optional_, string): the subject to which the event refers to. It is usually an entity identifier, like the database primary key. For example, if the event is about an item being added to the user's shopping cart in an e-commerce system, the subject could be the user ID.
- `data` (_optional_): a JSON object containing any application-specific data.

Example:

```json
{
    "protocol": "MEGA",
    "version": 1,
    "event": {
        "name": "shopping_cart.item_added",
        "timestamp": "2020-05-04T15:53:23.123",
        "version": 1,
        "publisher": "shopping-cart-service",
        "subject": "987650"
    },
    "data": {
        "item_id": "61fcc874-624e-40f8-8fd7-0e663c7837e8",
        "item_quantity": 5
    }
}
```

# [JSON API](http://jsonapi.org) spec implemented in PHP 7. Immutable

The goal of this library is to ensure strict validity of JSON API documents being produced.

JSON:
```json
{
    "data": {
        "type": "articles",
        "id": "1",
        "attributes": {
            "title": "Rails is Omakase"
        },
        "relationships": {
            "author": {
                "data": {
                    "type": "people",
                    "id": "9"
                },
                "links": {
                    "self": "/articles/1/relationships/author",
                    "related": "/articles/1/author"
                }
            }
        }
    }
}
```
PHP:
```php
<?php
use JsonApiPhp\JsonApi\Attribute;
use JsonApiPhp\JsonApi\DataDocument;
use JsonApiPhp\JsonApi\Link\RelatedLink;
use JsonApiPhp\JsonApi\Link\SelfLink;
use JsonApiPhp\JsonApi\ResourceIdentifier;
use JsonApiPhp\JsonApi\ResourceObject;
use JsonApiPhp\JsonApi\ToOne;

echo json_encode(
    new DataDocument(
        new ResourceObject(
            'articles',
            '1',
            new Attribute('title', 'Rails is Omakase'),
            new ToOne(
                'author',
                new ResourceIdentifier('author', '9'),
                new SelfLink('/articles/1/relationships/author'),
                new RelatedLink('/articles/1/author')
            )
        )
    ),
    JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES
);
```
## Installation
`composer require json-api-php/json-api`

## [Documentation](https://github.com/json-api-php/json-api/blob/master/README.md#documentation)

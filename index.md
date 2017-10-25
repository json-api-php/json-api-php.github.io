[JSON API spec](http://jsonapi.org/format/) | [GitHub](https://github.com/json-api-php/json-api) 
# JSON API in PHP 7

This library is an attempt to express business rules of [JSON API v1.0](http://jsonapi.org/format/) 
specification in a set of PHP 7 classes. We adhere to the following concepts:
- Test-first development
- Tests as documentation
- OOP (as much as PHP supports it), we respect SOLID principles

## Example

This JSON response...
<!-- name=my_json -->
```json
{
    "data": [
        {
            "type": "articles",
            "id": "1",
            "attributes": {
                "title": "JSON API paints my bikeshed!"
            },
            "relationships": {
                "author": {
                    "data": {
                        "type": "people",
                        "id": "9"
                    },
                    "links": {
                        "self": "http://example.com/articles/1/relationships/author",
                        "related": "http://example.com/articles/1/author"
                    }
                },
                "comments": {
                    "data": [
                        {
                            "type": "comments",
                            "id": "5"
                        },
                        {
                            "type": "comments",
                            "id": "12"
                        }
                    ],
                    "links": {
                        "self": "http://example.com/articles/1/relationships/comments",
                        "related": "http://example.com/articles/1/comments"
                    }
                }
            },
            "links": {
                "self": "http://example.com/articles/1"
            }
        }
    ],
    "links": {
        "self": "http://example.com/articles",
        "next": "http://example.com/articles?page[offset]=2",
        "last": "http://example.com/articles?page[offset]=10"
    },
    "included": [
        {
            "type": "people",
            "id": "9",
            "attributes": {
                "first-name": "Dan",
                "last-name": "Gebhardt",
                "twitter": "dgeb"
            },
            "links": {
                "self": "http://example.com/people/9"
            }
        },
        {
            "type": "comments",
            "id": "5",
            "attributes": {
                "body": "First!"
            },
            "relationships": {
                "author": {
                    "data": {
                        "type": "people",
                        "id": "2"
                    }
                }
            },
            "links": {
                "self": "http://example.com/comments/5"
            }
        },
        {
            "type": "comments",
            "id": "12",
            "attributes": {
                "body": "I like XML better"
            },
            "relationships": {
                "author": {
                    "data": {
                        "type": "people",
                        "id": "9"
                    }
                }
            },
            "links": {
                "self": "http://example.com/comments/12"
            }
        }
    ]
}
```
...can be build with this PHP code
<!-- assert=output expect=my_json -->
```php
<?php
use JsonApiPhp\JsonApi\Document;
use JsonApiPhp\JsonApi\Document\Resource\Linkage\MultiLinkage;
use JsonApiPhp\JsonApi\Document\Resource\Linkage\SingleLinkage;
use JsonApiPhp\JsonApi\Document\Resource\Relationship;
use JsonApiPhp\JsonApi\Document\Resource\ResourceIdentifier;
use JsonApiPhp\JsonApi\Document\Resource\ResourceObject;


$dan = new ResourceObject('people', '9');
$dan->setAttribute('first-name', 'Dan');
$dan->setAttribute('last-name', 'Gebhardt');
$dan->setAttribute('twitter', 'dgeb');
$dan->setLink('self', 'http://example.com/people/9');

$comment05 = new ResourceObject('comments', '5');
$comment05->setAttribute('body', 'First!');
$comment05->setLink('self', 'http://example.com/comments/5');
$comment05->setRelationship(
    'author',
    Relationship::fromLinkage(new SingleLinkage(new ResourceIdentifier('people', '2')))
);

$comment12 = new ResourceObject('comments', '12');
$comment12->setAttribute('body', 'I like XML better');
$comment12->setLink('self', 'http://example.com/comments/12');
$comment12->setRelationship(
    'author',
    Relationship::fromLinkage(new SingleLinkage($dan->toIdentifier()))
);

$author = Relationship::fromLinkage(new SingleLinkage($dan->toIdentifier()));
$author->setLink('self', 'http://example.com/articles/1/relationships/author');
$author->setLink('related', 'http://example.com/articles/1/author');

$comments = Relationship::fromLinkage(new MultiLinkage($comment05->toIdentifier(), $comment12->toIdentifier()));
$comments->setLink('self', 'http://example.com/articles/1/relationships/comments');
$comments->setLink('related', 'http://example.com/articles/1/comments');

$article = new ResourceObject('articles', '1');
$article->setAttribute('title', 'JSON API paints my bikeshed!');
$article->setLink('self', 'http://example.com/articles/1');
$article->setRelationship('author', $author);
$article->setRelationship('comments', $comments);

$doc = Document::fromResources($article);
$doc->setIncluded($dan, $comment05, $comment12);
$doc->setLink('self', 'http://example.com/articles');
$doc->setLink('next', 'http://example.com/articles?page[offset]=2');
$doc->setLink('last', 'http://example.com/articles?page[offset]=10');

echo json_encode($doc, JSON_PRETTY_PRINT | JSON_UNESCAPED_SLASHES);
```
## API
Please refer to [the tests](https://github.com/json-api-php/json-api/tree/master/test) for the full API documentation:
* [Documents](https://github.com/json-api-php/json-api/tree/master/test/Document/DocumentTest.php). Creating documents with primary data, errors, and meta. 
Adding links and API version to a document.
    * [Compound Documents](https://github.com/json-api-php/json-api/tree/master/test/Document/CompoundDocumentTest.php). Resource linkage.
* [Errors](https://github.com/json-api-php/json-api/tree/master/test/Document/ErrorTest.php)
* [Resources](https://github.com/json-api-php/json-api/tree/master/test/Document/Resource/ResourceTest.php)
* [Relationships](https://github.com/json-api-php/json-api/tree/master/test/Document/Resource/Relationship/RelationshipTest.php)
* [Linkage](https://github.com/json-api-php/json-api/tree/master/test/Document/Resource/Relationship/LinkageTest.php)

## Installation
`composer require json-api-php/json-api`

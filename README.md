# A filtering and sorting DSL for the [Django REST framework](http://www.django-rest-framework.org/)

This package provides a small [domain-specific language](https://en.wikipedia.org/wiki/Domain-specific_language)
(DSL) for filtering and sorting the views provided using Django
REST framework by GET parameters.

The filtering and sorting is performed by Django's querysets, so the
[SQL injection
protection](https://docs.djangoproject.com/en/1.11/topics/security/#sql-injection-protection) provided
by django is also used for the filtering. The fields of the model can be used in the queries in the DSL.

## Syntax

### Filtering

The basic filter syntax for a comparison of a field to a value is as follow

    [field] ([negation]) [filter operator] [value]

The `field` is the name of the model field to be queried. The `value` is the
value the field shall be compared to. Values for strings and timestamps need
to be quoted in single quotes. For boolean fields the values `true` and `false`
are valid.

The possible values for the `filter operator` are listed in the table futher
below. Some of these filters can be negated with the optional keyword `not`, to see
which operators support negation also see the table below.


| Filter operator | Alias | Meaning | Negatable with "not" | Field type requirements |
| --- | --- | --- | --- | --- |
| `=` | `eq` | "equal to" | `=` is not, but the `eq` alias is negatable | - |
| `!=` | `not eq` | "not equal to" | no             | - |
| `<` | `lt` |"less than"|no             | - |
| `>` | `gt` |"greater than"| no             | - |
| `<=` | `lte` |"less than or equal to"| no             | - |
| `>=` | `gte` |"greater than or equal to"| no | - |
| `contains` || substring search (case sensitive) | yes | requires text or char fields |
| `icontains` || substring search (case insensitive)| yes | requires text or char fields |
| `startswith` || substring search at the beginning of the field value (case sensitive) | yes | requires text or char fields |
| `istartswith` || substring search at the beginning of the field value (case insensitive)| yes | requires text or char fields |
| `endswith` || substring search at the end of the field value (case sensitive) | yes | requires text or char fields |
| `iendswith` || substring search at the end of the field value (case insensitive)| yes | requires text or char fields |
| `isnull` || value must be NULL | yes | - |

It is possible to combine multiple filters using the logical operators `and`
and `or`. `or` has precedence over `and`.

The default name of the GET parameter for filtering is `filter`.

### Sorting

The basic sorting syntax is

    ([direction])(field)

The name of the `field` is mandatory. The direction is optional an when not set
`+`, which means ascending order is implied. To sort in descending order use
`-`.


Multiple sorting operations can be chained by separating them with `,`. The
first terms will be used first for the ordering, then the following term(s)
will be used.

The default name of the GET parameter for sorting is `sort`.

### Example queries

The queries in this section use the API provided by the unittest
application in the `tests` directory of this repository.

The queries need to be correctly escaped before being send. The ones
listed further down this section are listed un-escaped for a better
readability. Escape should happen in the following form: `age < legs` should be
escaped to become:

    GET /animal?filter=age+%3C+legs

| Description | Filter query | Sort query |
| --- | --- | --- |
| An empty filter | | |
| Match an integer field | age = 132 ||
| Match the boolean field "is_bird" to be true | is_bird = true ||
| Match NULL values | favorite_food isnull ||
| Match Non-NULL values | favorite_food not isnull ||
| Compare a timestamp | birthday < '2007-10-13T11:13:09.250219+00:00' ||
| Chain multiple filters with "AND" | name = 'tortoise' and age >= 100 ||
| Chain multiple filters with "OR" | name = 'tortoise' or name = 'dog' ||
| Implicit sort by the field "name" in ascending order || name |
| Explicit sort by the field "name" in ascending order || +name |
| Sort by the field "name" in descending order || -name |
| Sort by the "legs" field in descending order and name in ascending order || -legs,+name |
| Compare the two fields "age" and "legs" | age < legs ||
| Match the field name to contain the substring "rtoi" | name contains 'rtoi' ||
| Match the field name to NOT contain the substring "rtoi" | name not contains 'rtoi' ||
| Combination of filtering and sorting | name startswith 'd' | sort=-id |


## Using this module in your application

This module just provides a custom [`DjangoFilterBackend`](http://www.django-rest-framework.org/api-guide/filtering/#djangofilterbackend) which
can be used like in the following example:


    from rest_framework import generics
    from rest_framework_filterdsl import FilterDSLBackend
    ...

    class MyListView(generics.ListAPIView):
        ...
        filter_backends = (FilterDSLBackend,)
        ...


Settings like the names of the GET parameters or the casts for the user-provided
values can be customized by subclassing the `FilterDSLBackend` implementation.


For more example usage please also see the unittests of this repository. Additional
the REST framework [filtering documentation](http://www.django-rest-framework.org/api-guide/filtering/) may
be helpful.

# Unittests

This projects includes a set of unittests implemented using
[pytest](https://docs.pytest.org/en/latest/index.html). To run these use the
`runtests.sh` script.
# License

See [LICENSE.txt](LICENSE.txt).

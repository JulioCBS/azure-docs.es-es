---
title: 'Referencia de operadores lógicos de OData: Azure Search'
description: Operadores lógicos de OData and, or y not, en las consultas de Azure Search.
ms.date: 06/13/2019
services: search
ms.service: search
ms.topic: conceptual
author: brjohnstmsft
ms.author: brjohnst
manager: nitinme
translation.priority.mt:
- de-de
- es-es
- fr-fr
- it-it
- ja-jp
- ko-kr
- pt-br
- ru-ru
- zh-cn
- zh-tw
ms.openlocfilehash: bf4939a40a2fdf1c8fc6cf97beca0184b1604c98
ms.sourcegitcommit: bb8e9f22db4b6f848c7db0ebdfc10e547779cccc
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 08/20/2019
ms.locfileid: "69647989"
---
# <a name="odata-logical-operators-in-azure-search---and-or-not"></a>Operadores lógicos de OData en Azure Search: `and`, `or`, `not`

[Las expresiones de filtro de OData](query-odata-filter-orderby-syntax.md) en Azure Search son expresiones booleanas que se evalúan como `true` o `false`. Puede escribir un filtro complejo si escribe una serie de [filtros más sencillos](search-query-odata-comparison-operators.md) y los combina mediante los operadores lógicos de [álgebra booleana](https://en.wikipedia.org/wiki/Boolean_algebra):

- `and`: un operador binario que se evalúa como `true` si las dos subexpresiones a la izquierda y la derecha se evalúan como `true`.
- `or`: un operador binario que se evalúa como `true` si una de las subexpresiones a la izquierda o la derecha se evalúa como `true`.
- `not`: un operador unario que se evalúa como `true` si su subexpresión se evalúa como `false` y viceversa.

Estos operadores, junto con los [operadores de colección `any` y `all`](search-query-odata-collection-operators.md), permiten construir filtros que pueden expresar criterios de búsqueda muy complejos.

## <a name="syntax"></a>Sintaxis

En la siguiente EBNF ([forma de Backus-Naur extendida](https://en.wikipedia.org/wiki/Extended_Backus–Naur_form)) se define la gramática de una expresión de OData en la que se usan los operadores lógicos.

<!-- Upload this EBNF using https://bottlecaps.de/rr/ui to create a downloadable railroad diagram. -->

```
logical_expression ::=
    boolean_expression ('and' | 'or') boolean_expression
    | 'not' boolean_expression
```

También está disponible un diagrama de sintaxis interactivo:

> [!div class="nextstepaction"]
> [Diagrama de sintaxis de OData para Azure Search](https://azuresearch.github.io/odata-syntax-diagram/#logical_expression)

> [!NOTE]
> Vea [Referencia de sintaxis de expresiones OData para Azure Search](search-query-odata-syntax-reference.md) para obtener la EBNF completa.

Hay dos formas de expresiones lógicas: binarias (`and`/`or`), donde hay dos subexpresiones, y unarias (`not`), donde solo hay una. Las subexpresiones pueden ser expresiones booleanas de cualquier tipo:

- Campos o variables de rango de tipo `Edm.Boolean`.
- Funciones que devuelven valores de tipo `Edm.Boolean`, como `geo.intersects` o `search.ismatch`.
- [Expresiones de comparación](search-query-odata-comparison-operators.md), como `rating gt 4`.
- [Expresiones de colección](search-query-odata-collection-operators.md), como `Rooms/any(room: room/Type eq 'Deluxe Room')`.
- Los literales booleanos `true` o `false`.
- Otras expresiones lógicas que se construyen con `and`, `or` y `not`.

> [!IMPORTANT]
> En algunas situaciones no todos los tipos de subexpresión se pueden usar con `and`/`or`, especialmente dentro de expresiones lambda. Vea [Operadores de colección de OData en Azure Search](search-query-odata-collection-operators.md#limitations) para más información.

### <a name="logical-operators-and-null"></a>Operadores lógicos y `null`

La mayoría de las expresiones booleanas, como las funciones y las comparaciones, no pueden generar valores `null`, y los operadores lógicos no se pueden aplicar directamente al literal `null` (por ejemplo, no se permite `x and null`). Pero los campos booleanos pueden ser `null`, por lo que debe tener en cuenta cómo se comportan los operadores `and`, `or` y `not` en presencia de NULL. Esto se resume en la tabla siguiente, donde `b` es un campo de tipo `Edm.Boolean`:

| Expression | Resultado cuando `b` es `null` |
| --- | --- |
| `b` | `false` |
| `not b` | `true` |
| `b eq true` | `false` |
| `b eq false` | `false` |
| `b eq null` | `true` |
| `b ne true` | `true` |
| `b ne false` | `true` |
| `b ne null` | `false` |
| `b and true` | `false` |
| `b and false` | `false` |
| `b or true` | `true` |
| `b or false` | `false` |

Cuando un campo booleano `b` aparece por sí mismo en una expresión de filtro, se comporta como si se hubiese escrito `b eq true`, por lo que si `b` es `null`, la expresión se evalúa como `false`. De forma similar, `not b` se comporta como `not (b eq true)`, por lo que se evalúa como `true`. De esta manera, los campos `null` se comportan igual que `false`. Esto es coherente con su comportamiento cuando se combinan con otras expresiones mediante `and` y `or`, como se muestra en la tabla anterior. A pesar de esto, una comparación directa con `false` (`b eq false`) se seguirá evaluando como `false`. En otras palabras, `null` no es igual a `false`, aunque se comporte como tal en las expresiones booleanas.

## <a name="examples"></a>Ejemplos

Comparar documentos donde el campo `rating` esté entre 3 y 5, inclusive:

    rating ge 3 and rating le 5

Comparar documentos donde todos los elementos del campo `ratings` sean menores que 3 o mayores que 5:

    ratings/all(r: r lt 3 or r gt 5)

Comparar documentos donde el campo `location` está dentro del polígono determinado, y el documento no contiene el término "public".

    geo.intersects(location, geography'POLYGON((-122.031577 47.578581, -122.031577 47.678581, -122.131577 47.678581, -122.031577 47.578581))') and not search.ismatch('public')

Comparar documentos para hoteles en Vancouver, Canadá donde hay una habitación deluxe con una tarifa base inferior a 160:

    Address/City eq 'Vancouver' and Address/Country eq 'Canada' and Rooms/any(room: room/Type eq 'Deluxe Room' and room/BaseRate lt 160)

## <a name="next-steps"></a>Pasos siguientes  

- [Filtros de Azure Search](search-filters.md)
- [Información general sobre el lenguaje de expresiones OData para Azure Search](query-odata-filter-orderby-syntax.md)
- [Referencia de sintaxis de expresiones OData para Azure Search](search-query-odata-syntax-reference.md)
- [Search Documents &#40;Azure Search Service REST API&#41;](https://docs.microsoft.com/rest/api/searchservice/Search-Documents) (Búsqueda en documentos [API REST de Azure Search Service])

---
title: Directivas de indexación de Azure Cosmos DB
description: Obtenga información sobre la configuración y cambio de la directiva de indexación predeterminada para una indexación automática y un mayor rendimiento en Azure Cosmos DB.
author: ThomasWeiss
ms.service: cosmos-db
ms.topic: conceptual
ms.date: 09/10/2019
ms.author: thweiss
ms.openlocfilehash: 944c05a28eb33c659bf4aaa600985530122f8d3e
ms.sourcegitcommit: e97a0b4ffcb529691942fc75e7de919bc02b06ff
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 09/15/2019
ms.locfileid: "71000322"
---
# <a name="indexing-policies-in-azure-cosmos-db"></a>Directivas de indexación en Azure Cosmos DB

En Azure Cosmos DB, cada contenedor tiene una directiva de indexación que determina cómo se deben indexar los elementos del contenedor. La directiva de indexación predeterminada para los contenedores recién creados indexa todas las propiedades de todos los contenedores, lo que exige que se usen índices de rango para todas las cadenas o números, e índices espaciales para todos los objetos GeoJSON del tipo Point. Esto permite obtener un rendimiento elevado de consultas sin tener que pensar en la indexación y administración de índices por adelantado.

En algunas situaciones, puede que quiera invalidar este comportamiento automático para ajustarse mejor a sus requerimientos. Puede personalizar la directiva de indexación de un contenedor estableciendo su *modo de indexación* e incluir o excluir las *rutas de acceso de propiedad*.

> [!NOTE]
> El método de actualización de las directivas de indexación que se describe en este artículo solo se aplica a la API de Azure Cosmos DB SQL (Core).

## <a name="indexing-mode"></a>Modo de indexación

Azure Cosmos DB admite dos modos de indexación:

- **Coherente**: El índice se actualiza de forma sincrónica al crear, actualizar o eliminar elementos. Esto significa que la coherencia de las consultas de lectura será la [coherencia configurada para la cuenta](consistency-levels.md).
- **Ninguna**: La indexación está deshabilitada en el contenedor. Esto se utiliza normalmente cuando se usa un contenedor como un almacén de pares clave-valor puro sin necesidad de índices secundarios. También se puede usar para mejorar el rendimiento de las operaciones masivas. Una vez completadas las operaciones masivas, el modo de índice se puede establecer en Coherente y supervisarse mediante [IndexTransformationProgress](how-to-manage-indexing-policy.md#use-the-net-sdk-v2) hasta que se complete.

> [!NOTE]
> Cosmos DB también admite un modo de indexación diferida. La indexación diferida realiza actualizaciones en el índice con un nivel de prioridad mucho menor cuando el motor no realiza ningún otro trabajo. Esto puede producir resultados de consulta **incoherentes o incompletos**. Además, el uso de la indexación diferida en lugar de "Ninguno" para operaciones masivas tampoco proporciona ninguna ventaja, ya que cualquier cambio en el modo de índice hará que se quite el índice y se vuelva a crear. Por estas razones, no recomendamos su uso a los clientes. Para mejorar el rendimiento de las operaciones masivas, establezca el modo de índice en Ninguno, vuelva al modo Coherente y supervise la propiedad `IndexTransformationProgress` en el contenedor hasta la finalización.

De forma predeterminada, la directiva de indexación se establece en `automatic`. Esto se consigue al establecer la propiedad `automatic` de la directiva de indexación en `true`. Al establecer esta propiedad en `true`, se permite que Azure Cosmos DB indexe automáticamente los documentos a medida que se escriben.

## <a name="including-and-excluding-property-paths"></a>Inclusión y exclusión de rutas de acceso de propiedad

Una directiva de indexación personalizada puede especificar rutas de acceso de propiedad que se incluyen o excluyen de forma explícita de la indexación. Al optimizar el número de rutas de acceso que se indexan, puede reducir la cantidad de almacenamiento que utiliza el contenedor y mejorar la latencia de las operaciones de escritura. Estas rutas de acceso se definen siguiendo [el método descrito en la sección de introducción a la indexación](index-overview.md#from-trees-to-property-paths), con las siguientes adiciones:

- Una ruta de acceso que lleva a un valor escalar (cadena o número) termina con `/?`.
- Los elementos de una matriz se dirigen juntos a través de la notación `/[]` (en lugar de `/0`, `/1` etcétera).
- El carácter comodín `/*` puede utilizarse para que coincida con cualquier elemento debajo del nodo.

Tomando el mismo ejemplo de nuevo:

```
    {
        "locations": [
            { "country": "Germany", "city": "Berlin" },
            { "country": "France", "city": "Paris" }
        ],
        "headquarters": { "country": "Belgium", "employees": 250 }
        "exports": [
            { "city": "Moscow" },
            { "city": "Athens" }
        ]
    }
```

- La ruta de acceso `employees` de `headquarters` es `/headquarters/employees/?`.

- La ruta de acceso `country` de `locations` es `/locations/[]/country/?`.

- La ruta de acceso de todo lo que incluye `headquarters` es `/headquarters/*`.

Por ejemplo, podríamos incluir la ruta de acceso `/headquarters/employees/?`. Esta ruta de acceso garantiza que indexamos la propiedad employees, pero no se indexará ningún archivo JSON anidado adicional en esta propiedad.

## <a name="includeexclude-strategy"></a>Inclusión o exclusión de estrategia

Cualquier directiva de indexación tiene que incluir la ruta de acceso raíz `/*` así como una ruta de acceso incluida o una excluida.

- Incluya la ruta de acceso raíz para excluir selectivamente las rutas de acceso que no se deban indexar. Este es el enfoque recomendado, ya que permite a Azure Cosmos DB indexar de forma proactiva las propiedades nuevas que se pueden agregar a su modelo.
- Excluya la ruta de acceso raíz para incluir selectivamente las rutas de acceso que se deban indexar.

- Para las rutas de acceso con caracteres normales que incluyen: caracteres alfanuméricos y _ (guion bajo), no tiene que aplicar el escape a la cadena de ruta de acceso en comillas dobles (por ejemplo, "/path/?"). Para las rutas de acceso con otros caracteres especiales, necesitará aplicar el escape a la cadena de ruta de acceso en comillas dobles (por ejemplo, "/\"path-abc\"/?"). Si se esperan caracteres especiales en la ruta de acceso, puede aplicar el escape a las rutas de acceso por motivos de seguridad. Funcionalmente, no hay ninguna diferencia si aplica el escape a todas las rutas de acceso frente a aplicarlo solo a las que tienen caracteres especiales.

- De forma predeterminada, la propiedad del sistema "ETag" se excluye de la indexación, a menos que la ETag se agregue a la ruta de acceso incluida para la indexación.

Al incluir y excluir las rutas de acceso, puede encontrar los atributos siguientes:

- `kind` puede ser `range` o `hash`. La funcionalidad de un índice de intervalo proporciona todas las funcionalidades de un índice de hash, por lo que recomendamos usar un índice de intervalo.

- `precision` es un número que se define en el nivel de índice para rutas de acceso incluidas. Un valor de `-1` indica la precisión máxima. Recomendamos que establezca siempre este valor en `-1`.

- `dataType` puede ser `String` o `Number`. Esto indica los tipos de propiedades JSON que se indexarán.

Cuando no se especifica, estas propiedades tendrán los valores predeterminados siguientes:

| **Nombre de la propiedad**     | **Valor predeterminado** |
| ----------------------- | -------------------------------- |
| `kind`   | `range` |
| `precision`   | `-1`  |
| `dataType`    | `String` y `Number` |

Consulte [esta sección](how-to-manage-indexing-policy.md#indexing-policy-examples) para obtener ejemplos de directivas de indexación a fin de incluir y excluir rutas de acceso.

## <a name="spatial-indexes"></a>Índices espaciales

Al definir una ruta de acceso espacial en la directiva de indexación, debe definir qué índice ```type``` se debe aplicar a esa ruta de acceso. Los tipos posibles para los índices espaciales son:

* Punto

* Polygon

* MultiPolygon

* LineString

De forma predeterminada, Azure Cosmos DB no creará ningún índice espacial. Si quiere usar funciones integradas de SQL espaciales, debe crear un índice espacial en las propiedades necesarias. Consulte [esta sección](geospatial.md) para obtener ejemplos de directivas de indexación sobre la adición de índices espaciales.

## <a name="composite-indexes"></a>Índices compuestos

Las consultas que tienen una cláusula `ORDER BY` con dos o más propiedades requieren un índice compuesto. También puede definir un índice compuesto para mejorar el rendimiento de muchas consultas de igualdad y de intervalo. De forma predeterminada, no hay índices compuestos definidos, por lo que debe [agregar índices compuestos](how-to-manage-indexing-policy.md#composite-indexing-policy-examples) según sea necesario.

Al definir un índice compuesto, especifique lo siguiente:

- Dos o más rutas de acceso de propiedad. La secuencia en la que se definen las rutas de acceso de propiedad importa.

- El orden (ascendente o descendente).

> [!NOTE]
> Al agregar un índice compuesto, al igual que con otros tipos de índice, las consultas pueden devolver resultados incoherentes cuando se actualiza el índice.

### <a name="order-by-queries-on-multiple-properties"></a>Consultas ORDER BY en varias propiedades:

Las consideraciones siguientes se usan cuando se utilizan índices compuestos para las consultas con una cláusula `ORDER BY` con dos o más propiedades:

- Si las rutas de acceso del índice compuesto no coinciden con la secuencia de las propiedades de la cláusula `ORDER BY`, el índice compuesto no puede admitir la consulta.

- El orden de las rutas de acceso del índice compuesto (ascendente o descendente) también debe coincidir con el valor de `order` de la cláusula `ORDER BY`.

- El índice compuesto también admite una cláusula `ORDER BY` con el orden inverso en todas las rutas de acceso.

Tenga en cuenta el ejemplo siguiente, en el que se define un índice compuesto en las propiedades name, age y _ts:

| **Índice compuesto**     | **Consulta `ORDER BY` de ejemplo**      | **¿Es compatible con el índice compuesto?** |
| ----------------------- | -------------------------------- | -------------- |
| ```(name ASC, age ASC)```   | ```SELECT * FROM c ORDER BY c.name ASC, c.age asc``` | ```Yes```            |
| ```(name ASC, age ASC)```   | ```SELECT * FROM c ORDER BY c.age ASC, c.name asc```   | ```No```             |
| ```(name ASC, age ASC)```    | ```SELECT * FROM c ORDER BY c.name DESC, c.age DESC``` | ```Yes```            |
| ```(name ASC, age ASC)```     | ```SELECT * FROM c ORDER BY c.name ASC, c.age DESC``` | ```No```             |
| ```(name ASC, age ASC, timestamp ASC)``` | ```SELECT * FROM c ORDER BY c.name ASC, c.age ASC, timestamp ASC``` | ```Yes```            |
| ```(name ASC, age ASC, timestamp ASC)``` | ```SELECT * FROM c ORDER BY c.name ASC, c.age ASC``` | ```No```            |

Debe personalizar la directiva de indexación para que pueda atender todas las consultas `ORDER BY` necesarias.

### <a name="queries-with-filters-on-multiple-properties"></a>Consultas con filtros en varias propiedades

Si una consulta tiene filtros en dos o más propiedades, puede resultar útil crear un índice compuesto para estas propiedades.

Por ejemplo, considere la siguiente consulta que tiene un filtro de igualdad en dos propiedades:

```sql
SELECT * FROM c WHERE c.name = "John" AND c.age = 18
```

Esta consulta será más eficaz, ya que tardará menos tiempo y consumirá menos RU si es capaz de aprovechar un índice compuesto en (name ASC, age ASC).

Las consultas con filtros de intervalo también se pueden optimizar con un índice compuesto. Sin embargo, la consulta solo puede tener un solo filtro de intervalo. Los filtros de intervalo incluyen `>`, `<`, `<=`, `>=` y `!=`. El filtro de intervalo debe definirse en último lugar en el índice compuesto.

Considere la consulta siguiente con los filtros de igualdad y de intervalo:

```sql
SELECT * FROM c WHERE c.name = "John" AND c.age > 18
```

Esta consulta será más eficaz con un índice compuesto de (name ASC, age ASC). Sin embargo, la consulta no utilizaría ningún índice compuesto en (age ASC, name ASC) porque los filtros de igualdad primero deben definirse en el índice compuesto.

Las consideraciones siguientes se usan cuando se crean índices compuestos para las consultas con filtros en varias propiedades:

- Las propiedades del filtro de la consulta deben coincidir con las del índice compuesto. Si una propiedad se encuentra en el índice compuesto, pero no se incluye en la consulta como filtro, la consulta no utilizará el índice compuesto.
- Si una consulta tiene propiedades adicionales en el filtro que no se definieron en un índice compuesto, se usará una combinación de índices compuestos y de intervalos para evaluar la consulta. De este modo, se requerirán menos RU que si se usan exclusivamente los índices de intervalo.
- Si una propiedad tiene un filtro de intervalo (`>`, `<`, `<=`, `>=` o `!=`), esta propiedad se debe definir en último lugar en el índice compuesto. Si una consulta tiene más de un filtro de intervalo, no utilizará el índice compuesto.
- Al crear un índice compuesto para optimizar las consultas con varios filtros, el valor de `ORDER` del índice compuesto no tendrá ningún impacto en los resultados. Esta propiedad es opcional.
- Si no define ningún índice compuesto para una consulta con filtros en varias propiedades, la consulta se realizará correctamente de todos modos. Sin embargo, el costo de RU de la consulta se puede reducir con un índice compuesto.

Tenga en cuenta los ejemplos siguientes, en los que se define un índice compuesto en las propiedades name, age y timestamp:

| **Índice compuesto**     | **Consulta de ejemplo**      | **¿Es compatible con el índice compuesto?** |
| ----------------------- | -------------------------------- | -------------- |
| ```(name ASC, age ASC)```   | ```SELECT * FROM c WHERE c.name = "John" AND c.age = 18``` | ```Yes```            |
| ```(name ASC, age ASC)```   | ```SELECT * FROM c WHERE c.name = "John" AND c.age > 18```   | ```Yes```             |
| ```(name DESC, age ASC)```    | ```SELECT * FROM c WHERE c.name = "John" AND c.age > 18``` | ```Yes```            |
| ```(name ASC, age ASC)```     | ```SELECT * FROM c WHERE c.name != "John" AND c.age > 18``` | ```No```             |
| ```(name ASC, age ASC, timestamp ASC)``` | ```SELECT * FROM c WHERE c.name = "John" AND c.age = 18 AND c.timestamp > 123049923``` | ```Yes```            |
| ```(name ASC, age ASC, timestamp ASC)``` | ```SELECT * FROM c WHERE c.name = "John" AND c.age < 18 AND c.timestamp = 123049923``` | ```No```            |

### <a name="queries-with-a-filter-as-well-as-an-order-by-clause"></a>Consultas con un filtro y una cláusula ORDER BY

Si una consulta aplica un filtro en una o más propiedades y tiene propiedades diferentes en la cláusula ORDER BY, puede resultar útil agregar las propiedades del filtro a la cláusula `ORDER BY`.

Por ejemplo, al agregar las propiedades del filtro a la cláusula ORDER BY, se podría volver a escribir la siguiente consulta para aprovechar un índice compuesto:

Consulta mediante un índice de intervalo:

```sql
SELECT * FROM c WHERE c.name = "John" ORDER BY c.timestamp
```

Consulta mediante un índice de compuesto:

```sql
SELECT * FROM c WHERE c.name = "John" ORDER BY c.name, c.timestamp
```

El mismo patrón y las optimizaciones de consulta se pueden generalizar para consultas con varios filtros de igualdad:

Consulta mediante un índice de intervalo:

```sql
SELECT * FROM c WHERE c.name = "John", c.age = 18 ORDER BY c.timestamp
```

Consulta mediante un índice de compuesto:

```sql
SELECT * FROM c WHERE c.name = "John", c.age = 18 ORDER BY c.name, c.age, c.timestamp
```

Las consideraciones siguientes se usan cuando se crean índices compuestos para optimizar una consulta con un filtro y una cláusula `ORDER BY`:

* Si la consulta aplica filtros en las propiedades, estos deben incluirse en primer lugar en la cláusula `ORDER BY`.
* Si no define ningún índice compuesto en una consulta con un filtro en una propiedad y una cláusula independiente `ORDER BY` mediante una propiedad diferente, la consulta se realizará correctamente. Sin embargo, el costo de RU de la consulta se puede reducir con un índice compuesto, especialmente si la propiedad de la cláusula `ORDER BY` tiene una cardinalidad alta.
* Todavía se aplican todas las consideraciones para crear índices compuestos para consultas `ORDER BY` con varias propiedades, así como consultas con filtros en varias propiedades.


| **Índice compuesto**                      | **Consulta `ORDER BY` de ejemplo**                                  | **¿Es compatible con el índice compuesto?** |
| ---------------------------------------- | ------------------------------------------------------------ | --------------------------------- |
| ```(name ASC, timestamp ASC)```          | ```SELECT * FROM c WHERE c.name = "John" ORDER BY c.name ASC, c.timestamp ASC``` | `Yes` |
| ```(name ASC, timestamp ASC)```          | ```SELECT * FROM c WHERE c.name = "John" ORDER BY c.timestamp ASC, c.name ASC``` | `No`  |
| ```(name ASC, timestamp ASC)```          | ```SELECT * FROM c WHERE c.name = "John" ORDER BY c.timestamp ASC``` | ```No```   |
| ```(age ASC, name ASC, timestamp ASC)``` | ```SELECT * FROM c WHERE c.age = 18 and c.name = "John" ORDER BY c.age ASC, c.name ASC,c.timestamp ASC``` | `Yes` |
| ```(age ASC, name ASC, timestamp ASC)``` | ```SELECT * FROM c WHERE c.age = 18 and c.name = "John" ORDER BY c.timestamp ASC``` | `No` |

## <a name="modifying-the-indexing-policy"></a>Modificación de la directiva de indexación

Se puede actualizar en cualquier momento una directiva de indexación de un contenedor [mediante Azure Portal o uno de los SDK admitidos](how-to-manage-indexing-policy.md). Una actualización de la directiva de indexación desencadena una transformación del índice antiguo al nuevo, que se realiza en línea y en local (por lo que no se consume ningún espacio de almacenamiento adicional durante la operación). El índice de la directiva antigua se transforma eficientemente en la nueva directiva sin que ello afecte a la disponibilidad de escritura ni al rendimiento aprovisionado en el contenedor. La transformación del índice es una operación asincrónica, y el tiempo que tarda en completarse depende del rendimiento aprovisionado, el número de elementos y su tamaño.

> [!NOTE]
> Mientras la reindexación está en curso, es posible que las consultas no devuelvan todos los resultados coincidentes y la hará sin devolver ningún error. Esto significa que los resultados de consultas no pueden ser coherentes hasta que se complete la transformación del índice. Es posible realizar un seguimiento del progreso de transformación del índice [mediante uno de los SDK](how-to-manage-indexing-policy.md).

Si el modo nuevo de la directiva de indexación se establece en coherente, no se puede aplicar ningún otro cambio de directiva de indexación mientras la transformación del índice esté en curso. Se puede cancelar una transformación del índice en ejecución al establecer el modo de la directiva de indexación en None (lo que inmediatamente anulará el índice).

## <a name="indexing-policies-and-ttl"></a>Directivas de indexación y TTL

La [característica periodo de vida (TTL)](time-to-live.md) requiere que la indexación esté activa en el contenedor que esté activado. Esto significa que:

- No es posible activar el TTL en un contenedor en el que se establece el modo de indexación en None.
- No es posible establecer el modo de indexación en None en un contenedor donde el TTL está activado.

Para escenarios donde no se necesita indexar ninguna ruta de acceso de propiedad, pero el TTL es necesario, puede usar una directiva de indexación con lo siguiente:

- Un modo de indexación establecido en Consistent.
- Ninguna ruta de acceso incluida.
- `/*` como única ruta de acceso excluida.

## <a name="next-steps"></a>Pasos siguientes

Obtenga más información acerca de la indexación en los siguientes artículos:

- [Introducción a la indexación](index-overview.md)
- [Cómo administrar la directiva de indexación](how-to-manage-indexing-policy.md)

Active Record
=============

[[Active Record] (http://en.wikipedia.org/wiki/Active_record_pattern) proporciona una interfaz orientada a objetos
para acceder y manipular los datos almacenados en bases de datos. Una clase Active Record está asociado con una tabla de base de datos,
una instancia de registro activo corresponde a una fila de esa tabla, y un atributo * * de un Registro Activo
ejemplo representa el valor de una columna en particular en la fila. En lugar de escribir sentencias SQL primas,
usted podrá acceder a los atributos de Active Record y llamar a los métodos de Active Record para acceder y manipular los datos almacenados
en tablas de bases de datos.

Por ejemplo, supongamos que `` del cliente es una clase Active Record que se asocia con la tabla `` del cliente
y `name` es una columna de la tabla` `del cliente. Se puede escribir el siguiente código para insertar una nueva
fila en la tabla `` del cliente:

`` `Php
$ Cliente = new Cliente ();
$ Cliente-> name = 'Qiang';
$ Cliente-> save ();
`` `

El código anterior es equivalente a usar la siguiente instrucción SQL prima para MySQL, que es menos
intuitiva, más propenso a errores, e incluso pueden tener problemas de compatibilidad si está utilizando un tipo diferente de base de datos:

`` `Php
$ Db-> createCommand ( "INSERT INTO` `del cliente (` name`) VALORES (: nombre) ', [
    ': Nombre' => 'Qiang,
]) -> Ejecutar ();
`` `

Yii proporciona el soporte Active Record para las siguientes bases de datos relacionales:

* MySQL 4.1 o posterior: a través de [[yü \ db \ ActiveRecord]]
* PostgreSQL 7.3 o posterior: a través de [[yü \ db \ ActiveRecord]]
* SQLite 2 y 3: a través de [[yü \ db \ ActiveRecord]]
* Microsoft SQL Server 2008 o posterior: a través de [[yü \ db \ ActiveRecord]]
* Oracle: a través de [[yü \ db \ ActiveRecord]]
* CUBRID 9.3 o posterior: a través de [[yü \ db \ ActiveRecord]] (Tenga en cuenta que debido a una [error] (http://jira.cubrid.org/browse/APIS-658) en
  la extensión PDO CUBRID, citando de los valores no funcionará, por lo que necesita CUBRID 9.3 como el cliente, así como el servidor)
* Sphinx: a través de [[yü \ esfinge \ ActiveRecord]], requiere la extensión `yii2-sphinx`
* Elasticsearch: a través de [[yü \ elasticsearch \ ActiveRecord]], requiere la extensión `yii2-elasticsearch`

Además, Yu también es compatible con el uso de Active Record con las siguientes bases de datos NoSQL:

* Redis 2.6.12 o posterior: a través de [[yü \ Redis \ ActiveRecord]], requiere la extensión `yii2-redis`
* MongoDB 1.3.0 o posterior: a través de [[yü \ mongodb \ ActiveRecord]], requiere la extensión `yii2-mongodb`

En este tutorial, vamos a describir principalmente el uso de Active Record para bases de datos relacionales.
Sin embargo, la mayoría del contenido aquí descrito son aplicables a Active Record para bases de datos NoSQL también.


## La declaración de clases Active Record <span id = "declaran-AR-clases"> </ span>

Para empezar, declare una clase Active Record mediante la ampliación [[yü \ db \ ActiveRecord]]. Debido a que cada Registro Activo
clase está asociado a una tabla de base de datos, en esta clase debe reemplazar el [[yü \ db \ ActiveRecord :: nombreTabla () | nombreTabla ()]]
Método para especificar la tabla de la clase está asociado.

En el siguiente ejemplo, declaramos una clase Active Record llamado `` del cliente para la tabla de base de datos `del cliente`.

`` `Php
App \ modelos de espacio de nombres;

utilizar yü \ db \ ActiveRecord;

la clase de cliente se extiende ActiveRecord
{
    const STATUS_INACTIVE = 0;
    const STATUS_ACTIVE = 1;
    
    / **
     * @return String el nombre de la tabla asociada con esta clase ActiveRecord.
     * /
    nombreTabla public static function ()
    {
        retorno "cliente";
    }
}
`` `

Active Record casos son considerados como [modelos] (structure-models.md). Por esta razón, se suele poner Active Record
categorías de dicha aplicación `espacio de nombres \ models` (u otros espacios de nombres para mantener las clases del modelo).

Debido a que [[yü \ db \ ActiveRecord]] se extiende desde [[yü \ Base \ Modelo]], hereda todos * * [modelo] (structure-models.md) características,
tales como atributos, reglas de validación, la serialización de datos, etc.


## Conexión con bases de datos <span id = "db-conexión"> </ span>

De forma predeterminada, Active Record utiliza el [componente de aplicación] `db` (structure-application-components.md)
como el [[yü \ db \ Conexión | conexión DB]] para acceder y manipular los datos de base de datos. Como se explica en
[Base de datos Access Objects] (db-dao.md), se puede configurar el componente `db` en la configuración de la aplicación, como se muestra
abajo,

`` `Php
regreso [
    'Componentes' => [
        'Db' ​​=> [
            'Clase' => 'yü \ db \ Conexión ",
            'DSN' => 'mysql: host = localhost; dbname = testdb',
            'Nombre de usuario' => 'demostración',
            'Password' => 'demostración',
        ],
    ],
];
`` `

Si desea utilizar una conexión de base de datos diferente que no sea el componente `db`, debe reemplazar
la [[yü \ db \ ActiveRecord :: GetDB () | GetDB ()]] método:

`` `Php
la clase de cliente se extiende ActiveRecord
{
    // ...

    public static function GetDB ()
    {
        // Utilizar el componente de aplicación "DB2"
        volver \ Yii :: app-$> DB2;
    }
}
`` `


## Consulta de Datos <span id = "consulta-data"> </ span>

Después de declarar una clase Active Record, que se puede utilizar para consultar los datos de la tabla de base de datos correspondiente.
El proceso suele tardar los tres pasos siguientes:

1. Crear un nuevo objeto de consulta llamando al [[yü \ db \ ActiveRecord :: find ()]] método;
2. Construir el objeto de consulta llamando a los métodos de construcción de consultas [] (# db-query-builder.md edificio-consultas);
3. Llame a un [método de consulta] (db-query-builder.md # query-métodos) para recuperar datos en términos de casos de Active Record.

Como se puede ver, esto es muy similar al procedimiento con [generador de consultas] (db-query-builder.md). La unica diferencia
es que en lugar de utilizar el operador `new` para crear un objeto de consulta, se llama [[yü \ db \ ActiveRecord :: find ()]]
para devolver un nuevo objeto de consulta, que es de la clase [[yü \ db \ ActiveQuery]].

A continuación se presentan algunos ejemplos que muestran cómo utilizar consulta activa para consultar datos:

`` `Php
// Devuelve un solo cliente cuyo ID es 123
// SELECT * FROM `` del cliente donde `id` = 123
$ Cliente = Cliente :: find ()
    -> Donde ([ 'id' => 123])
    -> Uno ();

// Devolver todos los clientes activos y ordenarlos por sus identificaciones
// SELECT * FROM `` del cliente DONDE'Estado` = 1 ORDER BY `id`
$ Clientes = Cliente :: find ()
    -> Donde ([ 'status' => Cliente :: STATUS_ACTIVE])
    -> OrderBy ( 'id')
    -> Todo ();

// Devuelve el número de clientes activos
// SELECT COUNT (*) FROM `` del cliente DONDE'Estado` = 1
$ Count = Cliente :: find ()
    -> Donde ([ 'status' => Cliente :: STATUS_ACTIVE])
    -> Count ();

// Devolver todos los clientes en una matriz indexada por ID de cliente
// SELECT * FROM `` del cliente
$ Clientes = Cliente :: find ()
    -> IndexBy ( 'id')
    -> Todo ();
`` `

En el, `` del cliente por encima de $ es un objeto `` del cliente mientras que `` de los clientes $ es un array de objetos `` del cliente. Son
toda poblada con los datos recuperados de la tabla `` del cliente.

> Información: Debido a que [[yü \ db \ ActiveQuery]] se extiende desde [[yü \ db \ query]], puede utilizar * todos * los métodos de construcción de consultas y
  métodos de consulta como se describe en la Sección [Generador de consultas] (db-query-builder.md).

Debido a que es una tarea común para consultar por los valores de clave principal o un conjunto de valores de columna, Yii ofrece atajo dos
métodos para este propósito:

- [[Yü \ db \ ActiveRecord :: findOne ()]]: devuelve una única instancia de Active Record poblada de la primera fila del resultado de la consulta.
- [[Yü \ db \ ActiveRecord :: findAll ()]]: devuelve un conjunto de instancias de Active Record pobladas con * todos * resultado de la consulta.

Ambos métodos pueden tomar uno de los siguientes formatos de parámetros:

- Un valor escalar: el valor se trata como el valor de la clave primaria deseada a ser buscado. Yii determinará
  automáticamente qué columna es la columna de clave principal mediante la lectura de la información de esquema de base de datos.
- Un conjunto de valores escalares: la matriz se trata como los valores de clave primaria deseados que se buscaba.
- Una matriz asociativa: las claves son los nombres de columna y los valores son los correspondientes desean valores de las columnas para
  que buscarlo. Por favor refiérase a [Formato Hash] (# db-query-builder.md de hash-formato) para más detalles.
  
El código siguiente muestra cómo se pueden utilizar estos métodos:

`` `Php
// Devuelve un solo cliente cuyo ID es 123
// SELECT * FROM `` del cliente donde `id` = 123
$ Cliente = Cliente :: findOne (123);

// Devuelve clientes cuyo ID es 100, 101, 123 o 124
// SELECT * FROM `` del cliente donde `id` EN (100, 101, 123, 124)
$ Clientes = Cliente :: findAll ([100, 101, 123, 124]);

// Devuelve un cliente activo cuyo ID es 123
// SELECT * FROM `` del cliente donde `id` = 123 Y = 1'Estado`
$ Cliente = Cliente :: findOne ([
    'Id' => 123,
    'Status' => Cliente :: STATUS_ACTIVE,
]);

// Devuelve todos los clientes inactivos
// SELECT * FROM `` del cliente DONDE'Estado` = 0
$ Clientes = Cliente :: findAll ([
    'Status' => Cliente :: STATUS_INACTIVE,
]);
`` `

> Nota: Ni [[yü \ db \ ActiveRecord :: findOne ()]] ni [[yü \ db \ ActiveQuery :: uno ()]] se sumará a LIMIT 1`
  la instrucción SQL generada. Si su consulta puede devolver el número de filas de datos, usted debe llamar `límite (1)` explícitamente
  para mejorar el rendimiento, por ejemplo, `Cliente :: find () -> límite (1) -> uno ()`.

Además de utilizar los métodos de construcción de consultas, también se puede escribir LSQ primas para consultar datos y poblar los resultados en
objetos de Active Record. Puede hacerlo llamando al método [[yü \ db \ ActiveRecord :: findBySql ()]]:

`` `Php
// Devuelve todos los clientes inactivos
$ Sql ​​= "SELECT * FROM cliente DONDE estado =: Estado ';
$ Clientes = Cliente :: findBySql ($ sql, [ ': status' => Cliente :: STATUS_INACTIVE]) -> todo ();
`` `

No llamar a los métodos de construcción de consultas adicionales después de llamar [[yü \ db \ ActiveRecord :: findBySql () | findBySql ()]], ya que
será ignorado.


## Acceso a los datos <span id = "-Acceso a los datos"> </ span>

Como se ha mencionado, los datos traídos de la base de datos se rellenan en casos de Active Record, y
cada fila del resultado de la consulta corresponde a una única instancia de Active Record. Se puede acceder a los valores de la columna
mediante el acceso a los atributos de los casos de registro activo, por ejemplo,

`` `Php
// "Id" y "e-mail" son los nombres de las columnas en la tabla "cliente"
$ Cliente = Cliente :: findOne (123);
$ Id = $ cliente-> Identificación;
$ Email = $ cliente-> correo electrónico;
`` `

> Nota: Los atributos de Active Record llevan el nombre de las columnas de las tablas asociadas de manera mayúsculas y minúsculas.
  Yii automáticamente define un atributo en Active Record para cada columna de la tabla asociada.
  NO debe redeclare cualquiera de los atributos.

Debido a que los atributos de Active Record llevan el nombre de las columnas de la tabla, es posible que usted está escribiendo código PHP como
`$ Cliente-> first_name`, que utiliza guiones para separar las palabras en los nombres de atributo si las columnas de tabla son
llamado de esta manera. Si usted está preocupado acerca de la consistencia estilo de código, debe cambiar el nombre de las columnas de tabla en consecuencia
(Para usar camelCase, por ejemplo).


### Datos Transformación <span id = "data-transformación"> </ span>

Sucede a menudo que los datos que se introducen y / o se muestran están en un formato que es diferente de la utilizada en
almacenar los datos en una base de datos. Por ejemplo, en la base de datos que está almacenando los cumpleaños de los clientes, las marcas de tiempo UNIX
(Que no es un buen diseño, aunque), mientras que en la mayoría de los casos usted querrá manipular los cumpleaños como cadenas en
el formato de ` 'AAAA / MM / DD'`. Para lograr este objetivo, se pueden definir los métodos de transformación de datos * * en el `` del cliente
clase Active Record como la siguiente:

`` `Php
la clase de cliente se extiende ActiveRecord
{
    // ...

    getBirthdayText función pública ()
    {
        fecha de retorno ( 'A / M / D', $ this-> cumpleaños);
    }
    
    setBirthdayText función pública ($ value)
    {
        $ This-> cumpleaños = strtotime (valor $);
    }
}
`` `

Ahora en su código PHP, en lugar de acceder `$ cliente-> birthday`, usted podrá acceder a` $ cliente-> birthdayText`, que
le permitirá a los clientes de entrada y visualización de cumpleaños en el formato de ` 'AAAA / MM / DD'`.

> Consejo: El ejemplo anterior muestra una forma genérica de transformar los datos en diferentes formatos. Si está trabajando con
> Valores de fecha, es posible utilizar [DateValidator] (fecha # tutorial-core-validators.md) y [[yü \ jui \ DatePicker | DatePicker]],
> Que es más fácil de usar y más potente.


### Recuperación de datos en matrices <span id = "data-en-arrays"> </ span>

Mientras que la recuperación de datos en términos de objetos de Active Record es conveniente y flexible, que no siempre es deseable
cuando se tiene que traer de vuelta una gran cantidad de datos debido a la gran capacidad de memoria. En este caso, se puede recuperar
datos utilizando matrices de PHP llamando al [[yü \ db \ ActiveQuery :: asArray () | asArray ()]] antes de ejecutar un método de consulta:

`` `Php
// Devolver todos los clientes
// Cada cliente se devuelve como una matriz asociativa
$ Clientes = Cliente :: find ()
    -> AsArray ()
    -> Todo ();
`` `

> Nota: Si bien este método ahorra memoria y mejora el rendimiento, que está más cerca de la capa de abstracción de base de datos más baja
  y se perderá la mayor parte de las características de Active Record. Una distinción muy importante radica en el tipo de datos de
  los valores de la columna. Cuando regresa de datos en casos de Active Record, los valores de las columnas serán encasillados automáticamente
  de acuerdo con los tipos de columna reales; por el contrario cuando regrese de datos en las matrices, los valores de las columnas serán
  cuerdas (ya que son el resultado de la DOP sin ningún procesamiento), independientemente de sus tipos de columnas reales.
   

### Recuperación de datos en lotes <span id = "data-en-lotes"> </ span>

En [Generador de consultas] (db-query-builder.md), hemos explicado que es posible utilizar * * consulta por lotes para reducir al mínimo su memoria
uso cuando se consulta una gran cantidad de datos de la base de datos. Es posible utilizar la misma técnica en Active Record. Por ejemplo,

`` `Php
// Vendería por 10 clientes a la vez
foreach (Customer :: find () -> (10) lotes como $ clientes) {
    // $ clientes es un conjunto de 10 o menos objetos de clientes
}

// Podido recuperar 10 clientes a la vez y repetir uno por uno
foreach (Customer :: find () -> cada uno (10) de $ cliente) {
    // $ Cliente es un objeto Cliente
}

// Consulta por lotes con la carga ansiosa
foreach (Customer :: find () -> with ( 'órdenes') -> cada uno () $ como cliente) {
    // $ Cliente es un objeto Cliente con relación a los "órdenes" poblado
}
`` `


## Almacenamiento de datos <span id = "insertar-actualización-data"> </ span>

El uso de Active Record, puede guardar fácilmente los datos a la base de datos mediante la adopción de las siguientes medidas:

1. Preparar una instancia Active Record
2. Asignar nuevos valores a los atributos de Active Record
3. Llamar a [[yü \ db \ ActiveRecord :: save ()]] para guardar los datos en la base de datos.

Por ejemplo,

`` `Php
// Insertar una nueva fila de datos
$ Cliente = new Cliente ();
$ Cliente-> name = 'James';
$ Cliente-> email = 'james@example.com';
$ Cliente-> save ();

// Actualizar una fila existente de los datos
$ Cliente = Cliente :: findOne (123);
$ Cliente-> email = 'james@newexample.com';
$ Cliente-> save ();
`` `

El [[yü \ db \ ActiveRecord :: save () | save ()]] método puede insertar o actualizar una fila de datos, dependiendo del estado
de la instancia de Active Record. Si la instancia se crea nuevamente a través del operador `new`, llamando
[[Yü \ db \ ActiveRecord :: save () | save ()]] hará que la inserción de una nueva fila; Si la instancia es el resultado de un método de consulta,
llamando [[yü \ db \ ActiveRecord :: save () | save ()]], se actualizará la fila asociada con la instancia.

Puede diferenciar los dos estados de una instancia Active Record comprobando su
[[Yü \ db \ ActiveRecord :: isNewRecord | isNewRecord]] valor de la propiedad. Esta propiedad también es utilizado por
[[Yü \ db \ ActiveRecord :: save () | save ()]] internamente como sigue:

`` `Php
Guardar función pública ($ runValidation = true, $ attributeNames = null)
{
    if ($ this-> getIsNewRecord ()) {
        return $ this-> insert ($ runValidation, $ attributeNames);
    } Else {
        return $ this-> actualización ($ runValidation, $ attributeNames) == false!;
    }
}
`` `

> Consejo: Usted puede llamar a [[yü \ db \ ActiveRecord :: insert () | insert ()]] o [[yü \ db \ ActiveRecord :: update () | update ()]]
  directamente al insertar o actualizar una fila.
  

### Validación de datos <span id = "datos de validación"> </ span>

Debido a que [[yü \ db \ ActiveRecord]] se extiende desde [[yü \ Base \ Modelo]], que comparte la misma [validación de datos] (input-validation.md) función.
Se puede declarar reglas de validación reemplazando la [[yü \ db \ ActiveRecord :: reglas () | (reglas)]] y llevar a cabo el método
validación de datos llamando al [[yü \ db \ ActiveRecord :: validate () | Validar] ()] método.

Cuando se llama [[yü \ db \ ActiveRecord :: save () | save ()]], por defecto se llamará [[yü \ db \ ActiveRecord :: validate () | validate ()]]
automáticamente. Sólo cuando pasa la validación, será en realidad salvar a los datos; de lo contrario se devuelva `false`,
y se puede comprobar el [[yü \ db \ ActiveRecord :: errores | errores]] propiedad para recuperar los mensajes de error de validación.

> Consejo: Si está seguro de que sus datos no necesitan la validación (por ejemplo, los datos proceden de fuentes confiables),
  puede llamar `save (falso)` para omitir la validación.


### Massive Asignación <span id = "masiva asignación"> </ span>

Al igual que los modelos [] normales (structure-models.md), las instancias de Active Record también disfrutan de la [función de asignación masiva] (# structure-models.md masiva-asignación).
Al utilizar esta función, puede asignar valores a los múltiples atributos de una instancia de registro activo en una sola instrucción PHP,
como se muestra a continuación. Recuerde que solamente [atributos de seguridad] (# structure-models.md-atributos de seguridad) se pueden asignar de forma masiva, sin embargo.

`` `Php
$ values ​​= [
    'Nombre' => 'de James,
    'Email' => 'james@example.com',
];

$ Cliente = new Cliente ();

$ Cliente-> atributos de valores = $;
$ Cliente-> save ();
`` `


### Actualización de Contadores <span id = "actualización de contadores"> </ span>

Es una tarea común para aumentar o disminuir una columna en una tabla de base de datos. Llamamos a estas columnas columnas "counter".
Puede utilizar [[yü \ db \ ActiveRecord :: updateCounters () | updateCounters ()]] para actualizar una o varias columnas de contador.
Por ejemplo,

`` `Php
$ Post = Post :: findOne (100);

// ACTUALIZACIÓN `` POST` SET view_count` = `view_count` + 1 donde` id` = 100
$ Post-> updateCounters ([ 'view_count' => 1]);
`` `

> Nota: Si usa [[yü \ db \ ActiveRecord :: save ()]] para actualizar una columna de contador, que puede terminar con resultados inexactos,
  porque es probable que el mismo contador está siendo salvado por múltiples solicitudes, que leen y escriben el mismo valor del contador.


### Atributos sucios <span id = "dirty-atributos"> </ span>

Cuando se llama [[yü \ db \ ActiveRecord :: save () | save ()]] para guardar una instancia de Active Record, únicos atributos * * sucios
se salvan. Un atributo es considerada sucia * * si su valor ha sido modificado desde que se cargó de DB o
guardado en DB más recientemente. Tenga en cuenta que se llevará a cabo la validación de datos sin tener en cuenta si el Registro Activo
instancia tiene atributos sucios o no.

Active Record mantiene automáticamente la lista de atributos sucios. Lo hace mediante el mantenimiento de una versión anterior de
los valores de los atributos y comparándolos con la más reciente. Puede llamar [[yü \ db \ ActiveRecord :: getDirtyAttributes ()]]
para obtener los atributos que son actualmente sucias. También puede llamar a [[yü \ db \ ActiveRecord :: markAttributeDirty ()]]
para marcar de forma explícita un atributo tan sucio.

Si usted está interesado en los valores de los atributos antes de su modificación más reciente, puede llamar
[[Yü \ db \ ActiveRecord :: getOldAttributes () | getOldAttributes ()]] o [[yü \ db \ ActiveRecord :: getOldAttribute () | getOldAttribute ()]].

> Nota: La comparación de los valores antiguos y nuevos se llevará a cabo utilizando el `` === operador, de modo que un valor será considerada sucia
> Incluso si tiene el mismo valor pero un tipo diferente. Esto es a menudo el caso cuando el modelo recibe la entrada del usuario desde
> Formularios HTML donde cada valor se representa como una cadena.
> Para asegurar el tipo correcto para, por ejemplo, valores enteros que pueden aplicar un [filtro de validación] (# input-validation.md de datos de filtrado):
> `[ 'AttributeName', 'filtro', 'filtro' => 'intval'] '. Esto funciona con todas las funciones de PHP como Encasillamiento
> [Intval ()] (http://php.net/manual/en/function.intval.php), [floatval ()] (http://php.net/manual/en/function.floatval.php) ,
> [Boolval] (http://php.net/manual/en/function.boolval.php), etc ...

### Valores de atributo predeterminados <id = "-atributos-valores por omisión" span> </ span>

Algunas de las columnas de tabla pueden tener valores por defecto definidos en la base de datos. A veces, es posible que desee rellenar previamente su
formulario web para una instancia de registro activo con estos valores por defecto. Para evitar la escritura de los mismos valores por defecto de nuevo,
puede llamar a [[yü \ db \ ActiveRecord :: loadDefaultValues ​​() | loadDefaultValues ​​()]] para rellenar los valores predeterminados definidos-DB
en los correspondientes atributos de Active Record:

`` `Php
$ Cliente = new Cliente ();
$ Cliente-> loadDefaultValues ​​();
// $ Cliente-> XYZ se le asignará el valor por defecto declarado en la definición de la columna "xyz"
`` `


### Atributos Encasillamiento <span id = "atributos-encasillamiento"> </ span>

Siendo poblado por resultados de la consulta [[yü \ db \ ActiveRecord]] realiza encasillado automática por sus valores de atributos, utilizando
Información [esquema de la tabla de base de datos] (# db-dao.md base de datos de esquema). Esto permite que los datos recuperados de la columna de la tabla
declarado como entero que se rellena en ActiveRecord ejemplo con PHP número entero, booleano booleano y así sucesivamente.
Sin embargo, el mecanismo de encasillamiento tiene varias limitaciones:

* Valores de coma flotante no se convertirán y se representan como cadenas, de lo contrario pueden perder precisión.
* La conversión de los valores enteros depende de la capacidad entera del sistema operativo que utilice. En particular:
  valores de la columna declarada como 'entero sin signo "o" entero grande' se convertirán al número entero de PHP sólo a 64 bits
  sistema de la operación, mientras que en los 32 bits - que estará representada como cadenas.

Tenga en cuenta que el atributo encasillado sólo se realice durante poblar ejemplo ActiveRecord del resultado de la consulta. No hay
la conversión automática de los valores cargados desde la solicitud HTTP o fijados directamente a través del acceso a la propiedad.
El esquema de la tabla también puede ser utilizado mientras se prepara sentencias SQL para el almacenamiento de los datos de ActiveRecord, asegurando
los valores están obligados a la consulta con el tipo correcto. Sin embargo, los valores de atributo de instancia ActiveRecord no serán
convertido durante el proceso de guardar.

> Consejo: se puede utilizar [[Yii \ comportamientos \ AttributeTypecastBehavior]] para facilitar valores de atributos encasillamiento
  sobre la validación o el ahorro de ActiveRecord.


### Actualización de varias filas <span id = "Actualización en múltiples hileras"> </ span>

Los métodos descritos anteriormente todo el trabajo sobre casos individuales de Active Record, inserción o actualización de individuo causando
filas de la tabla. Para actualizar varias filas al mismo tiempo, debe llamar al [[yü \ db \ ActiveRecord :: updateAll () | updateAll ()]], en cambio,
que es un método estático.

`` `Php
// ACTUALIZACIÓN `` del cliente SET'Estado` = 1 donde `` email` COMO% @ example.com% `
Cliente :: updateAll ([ 'status' => Cliente :: STATUS_ACTIVE], [ 'como', 'correo electrónico', '@ example.com']);
`` `

Del mismo modo, puede llamar al [[yü \ db \ ActiveRecord :: updateAllCounters () | (updateAllCounters)]] para actualizar las columnas de contador
varias filas al mismo tiempo.

`` `Php
// ACTUALIZACIÓN `` del cliente SET `` age` = 1 + age`
Clientes :: updateAllCounters ([ 'edad' => 1]);
`` `


## Borrado de datos <span id = "borrar-data"> </ span>

Para eliminar una sola fila de datos, primero recuperar la instancia de Active Record correspondiente a esa fila y luego llamar
la [[yü \ db \ ActiveRecord :: delete ()]] método.

`` `Php
$ Cliente = Cliente :: findOne (123);
$ Cliente-> delete ();
`` `

Puede llamar [[yü \ db \ ActiveRecord :: deleteAll ()]] para borrar varias o todas las filas de datos. Por ejemplo,

`` `Php
Cliente :: deleteAll ([ 'status' => Cliente :: STATUS_INACTIVE]);
`` `

> Nota: Tenga mucho cuidado cuando se llama [[yü \ db \ ActiveRecord :: deleteAll () | deleteAll ()]], ya que pueden quedar total
  borrar todos los datos de la tabla si se comete un error en la especificación de la condición.


## Activo registrar los ciclos de vida <span id = "ciclos de la vida ar"> </ span>

Es importante entender los ciclos de vida de Registro Activo cuando se utiliza para diferentes propósitos.
Durante cada ciclo de la vida, una cierta secuencia de los métodos será invocado, y se puede anular estos métodos
para tener la oportunidad de personalizar el ciclo de vida. También puede responder a ciertos acontecimientos desencadenados Active Record
durante un ciclo de vida para inyectar el código personalizado. Estos eventos son especialmente útiles cuando se está desarrollando
Active Record [comportamientos] (concept-behaviors.md) que necesitan para personalizar los ciclos de vida de Active Record.

En lo que sigue, vamos a resumir los diversos ciclos de vida activos de registro y los métodos / eventos que están involucrados
en los ciclos de vida.


### Nueva instancia del Ciclo de Vida <span id = "nueva instancia en el ciclo de la vida"> </ span>

Cuando se crea una nueva instancia de registro activo a través del operador `new`, el siguiente ciclo de vida va a pasar:

1. constructor de la clase.
2. [[yü \ db \ ActiveRecord :: init () | init ()]]: dispara un [[yü \ db \ ActiveRecord :: EVENT_INIT | EVENT_INIT]] evento.


### Consulta de Datos del Ciclo de Vida <span id = "consulta-el ciclo de vida de datos"> </ span>

Cuando la consulta de datos a través de uno de los [métodos que consultan] (#-consulta de datos), ambos recientemente llenado Active RECORD
someterse al siguiente ciclo de vida:

1. constructor de la clase.
2. [[yü \ db \ ActiveRecord :: init () | init ()]]: dispara un [[yü \ db \ ActiveRecord :: EVENT_INIT | EVENT_INIT]] evento.
3. [[yü \ db \ ActiveRecord :: afterFind () | afterFind ()]]: dispara un [[yü \ db \ ActiveRecord :: EVENT_AFTER_FIND | EVENT_AFTER_FIND]] evento.


### Almacenamiento de datos del Ciclo de Vida <span id = "salvar-ciclo-vida de los datos"> </ span>

Cuando se llama a [[yü \ db \ ActiveRecord :: save () | save ()]] para insertar o actualizar una instancia Active Record, el siguiente
ciclo de vida va a pasar:

1. [[yü \ db \ ActiveRecord :: beforeValidate () | () beforeValidate]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_BEFORE_VALIDATE | EVENT_BEFORE_VALIDATE]] evento. Si el método devuelve `false`
   o [[yü \ Base \ ModelEvent :: isValid]] es `false`, se omitirá el resto de los pasos.
2. Realiza la validación de datos. Si falla la validación de datos, los pasos posteriores al paso 3 se evitará.
3. [[yü \ db \ ActiveRecord :: afterValidate () | () afterValidate]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_AFTER_VALIDATE | EVENT_AFTER_VALIDATE]] evento.
4. [[yü \ db \ ActiveRecord :: beforeSave () | () BeforeSave]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_BEFORE_INSERT | EVENT_BEFORE_INSERT]]
   o [[yü \ db \ ActiveRecord :: EVENT_BEFORE_UPDATE | EVENT_BEFORE_UPDATE]] evento. Si el método devuelve `false`
   o [[yü \ Base \ ModelEvent :: isValid]] es `false`, se omitirá el resto de los pasos.
5. Realiza la inserción de datos real o actualización.
6. [[yü \ db \ ActiveRecord :: afterSave () | () afterSave]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_AFTER_INSERT | EVENT_AFTER_INSERT]]
   o [[yü \ db \ ActiveRecord :: EVENT_AFTER_UPDATE | EVENT_AFTER_UPDATE]] evento.
   

### Borrado de Datos del Ciclo de Vida <span id = "borrar-ciclo-vida de los datos"> </ span>

Cuando se llama a [[yü \ db \ ActiveRecord :: delete () | delete ()]] para eliminar una instancia Active Record, el siguiente
ciclo de vida va a pasar:

1. [[yü \ db \ ActiveRecord :: beforeDelete () | () beforeDelete]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_BEFORE_DELETE | EVENT_BEFORE_DELETE]] evento. Si el método devuelve `false`
   o [[yü \ Base \ ModelEvent :: isValid]] es `false`, se omitirá el resto de los pasos.
2. Realiza la eliminación de datos real.
3. [[yü \ db \ ActiveRecord :: afterDelete () | () afterDelete]]: disparadores
   un [[yü \ db \ ActiveRecord :: EVENT_AFTER_DELETE | EVENT_AFTER_DELETE]] evento.


> Nota: Llamar a cualquiera de los métodos siguientes no iniciará ninguna de las anteriores ciclos de vida, ya que trabajan en la
> Base de datos directamente y no sobre una base registro:
>
> - [[Yü \ db \ ActiveRecord :: updateAll ()]]
> - [[Yü \ db \ ActiveRecord :: deleteAll ()]]
> - [[yü \ db \ ActiveRecord :: updateCounters ()]]
> - [[yü \ db \ ActiveRecord :: updateAllCounters ()]]

Refrescante ### Datos del Ciclo de Vida <span id = "-ciclo-refrescante vida de los datos"> </ span>

Cuando se llama a [[yü \ db \ ActiveRecord :: Refresh () | refresh ()]] para actualizar una instancia Active Record, la
[[Yü \ db \ ActiveRecord :: EVENT_AFTER_REFRESH | EVENT_AFTER_REFRESH]] evento se dispara si de actualización es exitosa y el método devuelve `true`.


## Trabajando con las transacciones <span id = "transaccionales-operaciones"> </ span>

Hay dos formas de utilizar las transacciones [] (# Si realiza db-dao.md-Transacciones) mientras se trabaja con Active Record.

La primera forma es encerrar de manera explícita las llamadas a métodos Active Record en un bloque transaccional, como se muestra a continuación,

`` `Php
$ Cliente = Cliente :: findOne (123);

Cliente :: GetDB () -> transacción (function ($ db) utilizar ($ cliente) {
    $ Cliente-> id = 200;
    $ Cliente-> save ();
    // ... Otras operaciones de base de datos ...
});

// o alternativamente

$ Transacción = Cliente :: GetDB () -> beginTransaction ();
tratar {
    $ Cliente-> id = 200;
    $ Cliente-> save ();
    // ... Otras operaciones de base de datos ...
    $ Transac-> commit ();
} Catch (\ Excepción $ e) {
    $ Transac-> rollback ();
    tirar $ e;
}
`` `

La segunda forma es hacer una lista de las operaciones de base de datos que requieren soporte transaccional en el [[yü \ db \ ActiveRecord :: transacciones ()]]
Método. Por ejemplo,

`` `Php
la clase de cliente se extiende ActiveRecord
{
    transacciones de función pública ()
    {
        regreso [
            'Admin' => self :: OP_INSERT,
            'Api' => self :: OP_INSERT | self :: OP_UPDATE | self :: OP_DELETE,
            // Lo anterior es equivalente a la siguiente:
            // 'Api' => self :: OP_ALL,
        ];
    }
}
`` `

Los [[yü \ db \ ActiveRecord :: transacciones ()]] método debe devolver una matriz cuyas claves son [escenario] (# escenarios structure-models.md)
nombres y valores son las operaciones correspondientes que deben estar cerrados dentro de las transacciones. Se debe utilizar la siguiente
constantes para referirse a diferentes operaciones de base de datos:

* [[Yü \ db \ ActiveRecord :: OP_INSERT | OP_INSERT]]: operación de inserción realizado por [[yü \ db \ ActiveRecord :: insert () | insert ()]];
* [[Yü \ db \ ActiveRecord :: OP_UPDATE | OP_UPDATE]]: operación de actualización realizada por [[yü \ db \ ActiveRecord :: update () | update ()]];
* [[Yü \ db \ ActiveRecord :: OP_DELETE | OP_DELETE]]: operación de eliminación realizada por [[yü \ db \ ActiveRecord :: delete () | delete ()]].

Usa los `` | para concatenar las constantes anteriores para indicar múltiples operaciones. También puede utilizar el atajo
constante [[yü \ db \ ActiveRecord :: OP_ALL | OP_ALL]] para referirse a las tres operaciones anteriores.

Las transacciones que se crean mediante este método se pondrá en marcha antes de llamar [[yü \ db \ ActiveRecord :: beforeSave () | beforeSave ()]]
y serán realizados después del [[yü \ db \ ActiveRecord :: afterSave () | afterSave ()]] se ha ejecutado.

## Cerraduras optimistas <span id = "optimistas Cerraduras"> </ span>

El bloqueo optimista es una manera de prevenir los conflictos que pueden ocurrir cuando una sola fila de datos está siendo
actualizada por varios usuarios. Por ejemplo, tanto el usuario A y el usuario B están editando el mismo artículo de la wiki
al mismo tiempo. Después de un usuario guarda sus ediciones, los clics del usuario B en el botón "Guardar" en un intento de
guardar sus ediciones también. Debido a que el usuario B estaba trabajando en una versión caduca del artículo,
sería deseable tener una forma de evitar que salvar el artículo y le muestran algunos mensaje de sugerencia.

El bloqueo optimista resuelve el problema anterior mediante el uso de una columna para registrar el número de versión de cada fila.
Cuando una fila se guarda con un número de versión obsoleta, un [[yü \ db \ StaleObjectException]] excepción
será lanzado, lo que impide la fila que se guarden. El bloqueo optimista sólo se admite cuando se
actualizar o eliminar una fila existente de datos utilizando [[yü \ db \ ActiveRecord :: update ()]] o [[yü \ db \ ActiveRecord :: delete ()]],
respectivamente.

Para utilizar el bloqueo optimista,

1. Crear una columna en la tabla de base de datos asociada a la clase Active Record para almacenar el número de versión de cada fila.
   La columna debe ser de tipo entero grande (en MySQL sería `DEFAULT BIGINT 0`).
2. Reemplazar el método [] [yü \ db \ ActiveRecord :: optimisticLock ()] para devolver el nombre de esta columna.
3. En el formulario Web que toma las entradas del usuario, añadir un campo oculto para almacenar el número de versión actual de la fila está actualizando.
   Asegúrese de que su atributo de versión cuenta con reglas de validación de entrada y valida con éxito.
4. En la acción del controlador que actualiza la fila usando Active Record, tratar de atrapar la [[yü \ db \ StaleObjectException]]
   excepción. Implementar la lógica de negocio es necesario (por ejemplo, la fusión de los cambios, lo que provocó los datos staled) para resolver el conflicto.
   
Por ejemplo, suponga que la columna de versión se denomina como `version`. Puede implementar el bloqueo optimista con el código como
el seguimiento.


```php
class Customer extends \yii\db\ActiveRecord
{
    /**
     * Defines read-only virtual property for aggregation data.
     */
    public function getOrdersCount()
    {
        if ($this->isNewRecord) {
            return null; // this avoid calling a query searching for null primary keys
        }
        
        return $this->ordersAggregation[0]['counted'];
    }

    /**
     * Declares normal 'orders' relation.
     */
    public function getOrders()
    {
        return $this->hasMany(Order::className(), ['customer_id' => 'id']);
    }

    /**
     * Declares new relation based on 'orders', which provides aggregation.
     */
    public function getOrdersAggregation()
    {
        return $this->getOrders()
            ->select(['customer_id', 'counted' => 'count(*)'])
            ->groupBy('customer_id')
            ->asArray(true);
    }

    // ...
}

foreach (Customer::find()->with('ordersAggregation')->all() as $customer) {
    echo $customer->ordersCount; // outputs aggregation data from relation without extra query due to eager loading
}

$customer = Customer::findOne($pk);
$customer->ordersCount; // output aggregation data from lazy loaded relation
```

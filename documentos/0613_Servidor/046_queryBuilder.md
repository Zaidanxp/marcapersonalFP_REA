4.6. Constructor de consultas (Query Builder)

Laravel incluye una serie de clases que nos facilita la construcción de consultas y otro tipo de operaciones con la base de datos. Además, al utilizar estas clases, creamos una notación mucho más legible, compatible con todos los tipos de bases de datos soportados por Laravel y que nos previene de cometer errores o de ataques por inyección de código SQL.
Consultas

Para realizar una "Select" que devuelva todas las filas de una tabla utilizaremos el siguiente código:

$users = DB::table('users')->get();

foreach ($users as $user)
{
    echo $user->name;
}

En el ejemplo se utiliza el constructor DB::tabla indicando el nombre de la tabla sobre la que se va a realizar la consulta, y por último se llama al método get() para obtener todas las filas de la misma.

Si queremos obtener un solo elemento podemos utilizar first en lugar de get, de la forma:

$user = DB::table('users')->first();

echo $user->name;

Clausula where

Para filtrar los datos usamos la clausula where, indicando el nombre de la columna y el valor a filtrar:

$user = DB::table('users')->where('name', 'Pedro')->get();

echo $user->name;

En este ejemplo, la clausula where filtrará todas las filas cuya columna name sea igual a Pedro. Si queremos realizar otro tipo de filtrados, como columnas que tengan un valor mayor (>), mayor o igual (>=), menor (<), menor o igual (<=), distinto del indicado (<>) o usar el operador like, lo podemos indicar como segundo parámetro de la forma:

$users = DB::table('users')->where('votes', '>', 100)->get();

$users = DB::table('users')->where('status', '<>', 'active')->get();

$users = DB::table('users')->where('name', 'like', 'T%')->get();

Si añadimos más clausulas where a la consulta por defecto se unirán mediante el operador lógico AND. En caso de que queramos utilizar el operador lógico OR lo tendremos que realizar usando orWhere de la forma:

$users = DB::table('users')
                    ->where('votes', '>', 100)
                    ->orWhere('name', 'Pedro')
                    ->get();

orderBy / groupBy / having_

También podemos utilizar los métodos orderBy, groupBy y having en las consultas:

$users = DB::table('users')
                    ->orderBy('name', 'desc')
                    ->groupBy('count')
                    ->having('count', '>', 100)
                    ->get();

Offset / Limit

Si queremos indicar un offset o limit lo realizaremos mediante los métodos skip (para el offset) y take (para limit), por ejemplo:

$users = DB::table('users')->skip(10)->take(5)->get();

Transacciones

Laravel también permite crear transacciones sobre un conjunto de operaciones:

DB::transaction(function()
{
    DB::table('users')->update(array('votes' => 1));

    DB::table('posts')->delete();
});

En caso de que se produzca cualquier excepción en las operaciones que se realizan en la transacción se desharían todos los cambios aplicados hasta ese momento de forma automática.
Más informacion

Para más información sobre la construcción de Querys (join, insert, update, delete, agregados, etc.) podéis consultar la documentación de Laravel en su sitio web:

http://laravel.com/docs/queries

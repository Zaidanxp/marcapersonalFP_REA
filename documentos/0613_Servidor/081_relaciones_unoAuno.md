# Relaciones Uno a Uno

Una relación **Uno a Uno** es un tipo de relación muy básica. Por ejemplo, un modelo `User` podría estar asociado con un modelo `Curriculo`. Para definir esta relación, colocaremos un método `curriculo()` en el modelo `User`. El método `curriculo()` debe llamar al método `hasOne()` y devolver su resultado. El método `hasOne()` está disponible para su modelo a través de la clase base `Illuminate\Database\Eloquent\Model` del modelo:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;
 
class User extends Model
{
    /**
     * Get the curriculo associated with the user.
     */
    public function curriculo(): HasOne
    {
        return $this->hasOne(Curriculo::class);
    }
}
```

El primer argumento pasado al método `hasOne()` es el nombre de la clase del modelo relacionado. Una vez definida la relación, podemos recuperar el registro relacionado usando las propiedades dinámicas de Eloquent. Las propiedades dinámicas te permiten acceder a los métodos de relación como si fueran propiedades definidas en el modelo:

```php
$curriculo = User::find(1)->curriculo;
```

## Especificando la _foreign key_

_Eloquent_ determina la **foreign key** de la relación basándose en el nombre del modelo padre. En este caso, el modelo `Curriculo` se asume automáticamente que tiene una foreign key `user_id`. Si deseas anular esta convención, puedes pasar un segundo argumento al método `hasOne()`:

```php
return $this->hasOne(Curriculo::class, 'foreign_key');
```

Además, _Eloquent_ asume que la foreign key debe tener un valor que coincida con la columna de clave primaria del padre. En otras palabras, _Eloquent_ buscará el valor de la columna `id` del usuario en la columna `user_id` del registro `Curriculo`. Si desea que la relación utilice una clave primaria que no sea `id` o la propiedad `$primaryKey` de su modelo, puede pasar un tercer argumento al método `hasOne()`:

```php
return $this->hasOne(Curriculo::class, 'foreign_key', 'local_key');
```

## Definiendo la relación inversa

Al igual que podemos acceder a la relación `curriculo` desde el modelo `User`, también podemos acceder a la relación `user` desde el modelo `Curriculo`. Podemos definir la relación inversa de una relación `hasOne()` definiendo un método de relación que devuelva una llamada al método `belongsTo()`:

```php
<?php
 
namespace App\Models;
 
use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;
 
class Curriculo extends Model
{
    /**
     * Get the user that owns the curriculo.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```

Cuando invocamos el método `user`, Eloquent recuperará el modelo `User` que está relacionado con el `Curriculo` que estamos accediendo. En este caso, _Eloquent_ intentará encontrar un modelo `User` que tenga un `id` que coincida con la columna `user_id` en el modelo `Curriculo`.

_Eloquent_ determina el nombre de la **foreign key** examinando el nombre del método de relación y añadiendo el sufijo `_id` al nombre del método. Por lo tanto, en este caso, _Eloquent_ asume que el modelo `Curriculo` tiene una columna `user_id`. Sin embargo, si la **foreign key** en el modelo `Curriculo` no fuera `user_id`, podríamos pasar un nombre de clave personalizado como segundo argumento al método `belongsTo()`:

```php
/**
 * Get the user that owns the curriculo.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key');
}
```

No es nuestro caso, pero si el modelo padre no usara `id` como su clave primaria, o si deseara encontrar el modelo asociado utilizando una columna diferente, podría pasar un tercer argumento al método `belongsTo()` especificando la clave personalizada de la tabla padre:

```php
/**
 * Get the user that owns the curriculo.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

## Ejemplo

Vamos a concretar el ejemplo de **relación uno a uno**, entre el modelo `User` y el modelo `Curriculo`. El modelo `User` tendrá una relación `hasOne()` con el modelo `Curriculo` y, por su parte, el modelo `Curriculo` tendrá una relación `belongsTo()` con el modelo `User`:

### `hasOne()`

Vamos a incluir el siguiente código al final de la definición del modelo `User` (_dentro de las llaves que cierran la clase_):

```php
    /**
     * Get the curriculo associated with the user.
     */
    public function curriculo(): HasOne
    {
        return $this->hasOne(Curriculo::class);
    }

```

Sin olvidarnos de añadir la correspondiente clase al principio del fichero:

```php
use Illuminate\Database\Eloquent\Relations\HasOne;
```

### `belongsTo()`

Vamos a incluir el siguiente código al final de la definición del modelo `Curriculo`:

```php
    /**
     * Get the user that owns the curriculo.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
    
```

Sin olvidarnos de añadir la correspondiente clase al principio del fichero:

```php
use Illuminate\Database\Eloquent\Relations\BelongsTo;
```

## Probemos su funcionamiento

Para probar el funcionamiento de la relación, vamos a modificar los datos que se envían de cada usuario, haciendo que se añadan los datos de su curriculo. Para ello, modificaremos el método `toArray()` del recurso `UserResource`:

```php
     public function toArray(Request $request): array
     {
-        return parent::toArray($request);
+        return array_merge(parent::toArray($request), [
+            'curriculo' => new CurriculoResource($this->curriculo),
+        ]);
     }

```

Una vez hecho esto, podemos hacer una petición GET a la ruta `/api/v1/users` y comprobar que los datos de los currículos están asociados a cada usuario.

> **Nota** Debemos llevar mucho cuidado de no generar un bucle infinito a la hora de utilizar las relaciones entre modelos. Por ejemplo, si en el método `toArray()` del recurso `CurriculoResource` devolviéramos el usuario al que pertenece el currículo, se produciría un bucle infinito, ya que el usuario devolvería el currículo, que a su vez devolvería el usuario, y así sucesivamente.

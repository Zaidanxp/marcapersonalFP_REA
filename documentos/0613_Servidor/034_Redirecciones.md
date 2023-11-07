# 3.4. Redirecciones

Como respuesta a una petición también podemos devolver una redirección. Esta opción será interesante cuando, por ejemplo, el usuario no esté logueado y lo queramos redirigir al formulario de _login_, o cuando se produzca un error en la validación de una petición y queramos redirigir a otra ruta.

Para esto simplemente tenemos que utilizar el método `redirect()`` indicando como parámetro la ruta a redireccionar, por ejemplo:

`return redirect('user/login');`

O si queremos volver a la ruta anterior simplemente podemos usar el método back:

`return back();`

## Redirección a una acción de un controlador

También podemos redirigir a un método de un controlador mediante el método `action()` de la forma:

`return redirect()->action('HomeController@index');`

Si queremos añadir parámetros para la llamada al método del controlador tenemos que añadirlos pasando un _array_ como segundo parámetro:

`return redirect()->action('UserController@profile', [1]);`

## Redirección con los valores de la petición

Las redirecciones se suelen utilizar tras obtener algún error en la validación de un formulario o tras procesar algunos parámetros de entrada. En este caso, para que al mostrar el formulario con los errores producidos podamos añadir los datos que había escrito el usuario tendremos que volver a enviar los valores enviados con la petición usando el método `withInput()`:

```
return redirect('form')->withInput();

// O para reenviar los datos de entrada excepto algunos:
return redirect('form')->withInput($request->except('password'));
```

Este método también lo podemos usar con la función `back()` o con la función `action()`:

`return back()->withInput();`

`return redirect()->action('HomeController@index')->withInput();`

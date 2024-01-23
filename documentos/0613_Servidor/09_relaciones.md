# Relaciones

A menudo, una tabla de la base de datos está relacionadas con alguna otra tabla. Por ejemplo, un `curriculo` está relacionado con un `user`.

_Eloquent_ facilita la gestión de estas relaciones y soporta las relaciones más comunes:

- [Uno a Uno]()
- [Uno a Muchos]()
- [Muchos a Muchos]()
- [Uno a Uno a través de una tabla pivot]()
- [Uno a Muchos a través de una tabla pivot]()
- [Uno a Uno (Polymórficas)]()
- [Uno a Muchos (Polymórficas)]()
- [Muchos a Muchos (Polymórficas)]()

Aunque en la lista anterior incluye todos los tipos de relaciones, para el alcance de este curso, veremos las más usuales.

## Definiendo Relaciones

Las relaciones en _Eloquent_ se definen como **métodos** en las clases correspondientes a los modelos. En los capítulos siguientes, veremos cómo definir cada una de las relaciones de los modelos.

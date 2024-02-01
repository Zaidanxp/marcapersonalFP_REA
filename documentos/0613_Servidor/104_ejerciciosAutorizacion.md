# Ejercicios autorización

## Autorización de controladores

Se debe crear políticas para cada uno de los modelos que manejamos, de acuerdo a la siguiente tabla:

Modelo| index | show | store | update | destroy
-|-|-|-|-|-
Actividad | anónimo | anónimo | docente | propietario | propietario
Ciclo | anónimo | anónimo | admin | admin | admin
Competencia | anónimo | anónimo | admin | admin | admin
Competencias_Actividades | anónimo | anónimo | docente | propietario | propietario
Curriculo | anónimo | anónimo | estudiante | propietario | propietario
Empresa | anónimo | anónimo | docente | docente | propietario
FamiliaProfesional | anónimo | anónimo | admin | admin | admin
Idioma | anónimo | anónimo | admin | admin | admin
ParticipanteProyecto | anónimo | anónimo | docente | propietario (tutor) | propietario (tutor)
ProyectoCiclo | anónimo | anónimo | docente | propietario (tutor) | propietario (tutor)
Proyecto | anónimo | anónimo | docente | propietario (tutor) | propietario (tutor)
Reconocimiento | anónimo | anónimo | docente | propietario | propietario
User_ciclo | anónimo | anónimo | estudiante | propietario | propietario
UserCompetencia | anónimo | anónimo | estudiante | propietario | propietario
User | anónimo | anónimo | anónimo | propietario | propietario
UsersIdioma | anónimo | anónimo | estudiante | propietario | propietario

Ten en cuenta que el middleware de autenticación lo debemos incluir en el constructor del controlador, para que se aplique a todos los métodos, a excepción de los que permiten un acceso anónimo.

## Asignación automática del propietario

En el caso de los modelos que tienen alguna referencia al propietario, debemos asignar automáticamente, como propietario, al usuario autenticado, en el momento de crear el recurso. Para ello, debemos modificar el método `store` de cada controlador, de forma que se asigne el propietario al crear el recurso.

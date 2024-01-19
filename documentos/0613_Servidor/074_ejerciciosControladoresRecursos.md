# Ejercicios de controladores de recursos

Se nos ha solicitado incrementar la cantidad de tipos de recursos que debemos manejar. Por ello, debemos realizar todo lo necesario para que nuestra API responda a todas las peticiones posibles de los recursos asociados a las tablas del esquema relacional de MarcaPersonalFP, que se relacionan a continuación:

- competencias
- competencias-actividades
- empresas
- users-competencias
- idiomas
- users-idiomas

La gestión que debemos realizar incluye:

- la definición de la tabla en la base de datos
- el llenado de las tablas con datos de prueba
- la respuesta a las rutas asociadas a los nombres de las tablas
- la definición de las páginas correspondientes en React-admin

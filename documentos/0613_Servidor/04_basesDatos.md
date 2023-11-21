# 4. Base de datos

_Laravel_ facilita la configuración y el uso de diferentes tipos de base de datos: _MySQL_, _PostgreSQL_, _SQLite_ y _SQL Server_. En el fichero de configuración (`config/database.php`) tenemos que indicar todos los parámetros de acceso a nuestras bases de datos y además especificar cuál es la conexión que se utilizará por defecto. No obstante, en lugar de especificar directamente la configuración en el archivo `config/database.php`, por lo general, será suficiente con establecer los valores en el archivo `.env`.

En _Laravel_ podemos hacer uso de varias bases de datos a la vez, aunque sean de distinto tipo. Por defecto, se accederá a la que especifiquemos en la configuración y si queremos acceder a otra conexión lo tendremos que indicar expresamente al realizar la consulta.

En este capítulo veremos como configurar una base de datos, como crear tablas y especificar sus campos desde código, como inicializar la base de datos y como construir consultas tanto de forma directa como a través del _ORM_ llamado _Eloquent_.

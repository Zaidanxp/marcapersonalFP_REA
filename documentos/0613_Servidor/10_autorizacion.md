# Autorización

Hasta ahora, hemos visto que las rutas las protegíamos para impedir que usuarios no autenticados pudieran alcanzarlas.

No obstante, en la mayoría de las aplicaciones, las acciones serán permitidas en función del usuario que esté autenticado.

Laravel ofrece 2 formas de gestionar los permisos en función del usuario autenticado:

- Gates
- Policies

Aunque los **`gates`** ofrecen una forma sencilla de manejar las autorizaciones, la mayoría de las aplicaciones requerirán el sistema de **`policies`** para gestionarlas, por lo que será el sistema que utilizaremos en nuestro desarrollo.
# Roles y permisos.

Son numerosas las aplicaciones que combinan la autenticación de usuario con la gestión de roles y sus permisos. Esta gestión se puede simplificar con la utilización de librerías como [laravel/spatie](). Un explicación del uso básico de esa librería la podemos encontrar [aquí](https://spatie.be/index.php/docs/laravel-permission/v6/basic-usage/basic-usage).

Aunque la utilización de *permisos* tiene sus ventajas, para nuestra aplicación vamos a combinar las `policies` con los tipos de usuario o roles que pueden utilizar la aplicación:

- **usuarios anónimos**,
- **administrador**: aquel cuyo email coincide con `ADMIN_EMAIL`,
- **docentes**: todos aquellos que tienen un email del dominio `TEACHER_EMAIL_DOMAIN`,
- **estudiantes**:  todos aquellos que tienen un email del dominio `STUDENT_EMAIL_DOMAIN`,
- **propietario de un recurso**: si un recurso tiene una propiedad user_id, aquel usuario cuyo id coincide con el valor anterior,
- **empresas**: acceden mediante un token cuya gestión definiremos más adelante.

> Las variables de entorno `TEACHER_EMAIL_DOMAIN` y `STUDENT_EMAIL_DOMAIN` habrá que definirlas con los dominios asignados por la institución a docentes y estudiantes, tanto en el fichero `.env.example` como en el archivo `.env`.

Para gestionar estos **_roles_**, vamos a crear sendos métodos en el modelo `User`, como se muestra a continuación:

```php
    // Gestión de roles
    public function esAdmin(): bool
    {
        return $this->email === env('ADMIN_EMAIL');
    }

    public function esDocente(): bool
    {
        return $this->getEmailDomain() === env('TEACHER_EMAIL_DOMAIN');
    }

    public function esEstudiante(): bool
    {
        return $this->getEmailDomain() === env('STUDENT_EMAIL_DOMAIN');
    }

    public function esPropietario($recurso, $propiedad = 'user_id'): bool
    {
        return $recurso && $recurso->$propiedad === $this->id;
    }

    private function getEmailDomain(): string
    {
        $dominio = explode('@', $this->email)[1];
        return $dominio;
    }  
```

## Generar usuarios con roles

Además, queremos modificar la inicialización de los usuarios para tener 10 docentes y 30 estudiantes, por lo que debemos modificar los archivos correspondientes

```diff
// en database/seeders/UsersTableSeeder.php
    public function run(): void
             'password' => env('ADMIN_PASSWORD', 'password'),
         ]);
 
-        User::factory(10)->create();
+        // Crear 10 usuarios con el estado docente
+        User::factory(10)->docente()->create();
+        // Crear 30 usuarios con el estado estudiante
+        User::factory(30)->estudiante()->create();
 
 
    }
```

```diff
// en database/factories/UserFactory.php
     public function definition(): array
     {
         return [
-            'name' => fake()->name(),
+            'name' => fake()->unique()->userName(),
             'email' => fake()->unique()->safeEmail(),
             'email_verified_at' => now(),
             'password' => '$2y$10$92IXUNpkjO0rOQ5byMi.Ye4oKoEa3Ro9llC/.og/at2.uheWG/igi', // password
             'remember_token' => Str::random(10),
@@ -37,4 +36,24 @@ public function unverified(): static
             'email_verified_at' => null,
         ]);
     }
+
+    /**
+     * Indicate that the model's email  should be student.
+     */
+    public function estudiante(): static
+    {
+        return $this->state(fn (array $attributes) => [
+            'email' => $attributes['name'] . '@' . env('STUDENT_EMAIL_DOMAIN', 'student.com'),
+        ]);
+    }
+
+    /**
+     * Indicate that the model's email  should be teacher.
+     */
+    public function docente(): static
+    {
+        return $this->state(fn (array $attributes) => [
+            'email' => $attributes['name'] . '@' . env('TEACHER_EMAIL_DOMAIN', 'teacher.com'),
+        ]);
+    }
 }
```
## Adaptar las `policies` a los roles anteriores

Después de los cambios anteriores, podríamos modificar el fichero de _políticas_ asociadas a `currículo` de la siguiente forma:

```diff
// en /app/Policies/CurriculoPolicy.php
@@ -17,7 +17,7 @@ class CurriculoPolicy
      */
     public function before(User $user, $ability)
     {
-        if($user->email === env('ADMIN_EMAIL')) return true;
+        if($user->esAdmin()) return true;
     }
 
     /**
@@ -41,7 +41,7 @@ public function view(User $user, Curriculo $curriculo): bool
      */
     public function create(User $user): bool
     {
-        return $user->email === env('ADMIN_EMAIL');
+        return $user->esEstudiante();
     }
 
     /**
@@ -49,7 +49,7 @@ public function create(User $user): bool
      */
     public function update(User $user, Curriculo $curriculo): bool
     {
-        return $user->id === $curriculo->user_id;
+        return $user->esPropietario($curriculo);
     } 
```
## Ejercicios

Debemos crear políticas para cada uno de los modelos que manejamos, de acuerdo a la siguiente tabla:

Modelo| index | show | store | update | destroy
-|-|-|-|-|-
Actividad | anónimo | anónimo | docente | propietario | propietario
Ciclo | anónimo | anónimo | admin | admin | admin
Competencia | anónimo | anónimo | admin | admin | admin
Competencias_Actividades | anónimo | anónimo | docente | propietario | propietario
Curriculo | anónimo | anónimo | estudiante | propietario | propietario
Empresa | anónimo | anónimo | docente | docente | docente
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

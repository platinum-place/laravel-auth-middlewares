# Usos dentro de Laravel Passport

## Laravel Passport: Modelo `Client` y `client_credentials`

Laravel Passport implementa autenticación OAuth2 para APIs. Su modelo `Client` representa las aplicaciones que
interactúan con tu API, usando un `client_id` y `client_secret` para autenticarse.

### Autenticación `client_credentials`

El flujo `client_credentials` permite que un cliente obtenga un token de acceso sin un usuario. Este token se usa para
acceder a recursos protegidos de la API.

#### Flujo de Autenticación:

1. **Obtener un Token:**  
   El cliente envía su `client_id` y `client_secret` a la ruta `/oauth/token` para obtener un token de acceso.

2. **Usar el Token:**  
   El token se incluye en las solicitudes a los endpoints protegidos usando el encabezado
   `Authorization: Bearer <token>`.

Consulta la [documentación oficial de Passport](https://laravel.com/docs/10.x/passport) para más información.

# Uso: Utilizar la instancia Client en toda la aplicación

### Acceso a la Instancia del Cliente

La instancia `Client` que utiliza un endpoint no es accesible directamente dentro de la función que se esté ejecutando.
Para poder acceder a ella desde cualquier `$request` o `request()`, puedes agregar lo siguiente:

```php
$request->setUserResolver(fn () => $client);
```

Optimizando este procedimiento, podremos colocar dicha función en lugares estratégicos para utilizar `auth()->user()`
que acostumbramos en Laravel Breeze, por ejemplo.

## Autenticación 'auth.client'

Al sobrescribir el middleware `CheckClientCredentials`, podemos atrapar la instancia de `client` cuando se solicita el acceso mediante un bearer token. De esta forma, al validar el cliente con el token, podremos extraer la instancia `client` y inyectarla en el request.

```php
    protected function validateCredentials($token)
    {
        if (! $token) {
            throw new AuthenticationException;
        }

        request()->setUserResolver(fn () => $token->client);
    }
```

## Autenticación 'auth.basic'

Si estamos implementando la autenticación Basic Auth y queremos que los clientes de Passport sean los mismos que tendrán acceso, podemos lograrlo sobrescribiendo la clase `AuthenticateWithBasicAuth`.

De esta manera, podremos inyectar la instancia `client` luego de localizarla utilizando las credenciales proporcionadas por `getUser()` y `getPassword()`, que son parte del proceso de autenticación Basic Auth.

El método de búsqueda y la seguridad del proceso quedan a interpretación propia y pueden ser ajustados según los requisitos del proyecto.

```php
    public function handle($request, Closure $next, $guard = null, $field = null)
    {
        $model = \App\Models\Client::where('id', $request->getUser())
            ->where('secret', $request->getPassword())
            ->first();

        if (! $model) {
            return response(['message' => __('Unauthorized')], 401);
        }

        $request->setUserResolver(fn () => $model);

        return $next($request);
    }
```

# Laravel Auth Middlewares

### Auth Basic with a 'username' field

To use Laravel's native Basic Authentication with a "username" instead of "email," you need to modify the `findForPassport` method in the `User` model to query by the `username` field. Additionally, you may need to customize the authentication logic by overriding the default `BasicAuth` controller or middleware. This allows you to change the authentication behavior while still using Laravel’s built-in Basic Authentication system.

### Laravel Passport: Client Credentials Grant Tokens with global access to the client instance

To inject the client instance when obtaining an access token and use it in a call, you can access the client instance that invoked the token through request()->user(). This is similar to using the user() helper, and it allows you to retrieve the authenticated user along with the associated client instance.

# Socialite Provider for Vipps

## 1. Installation

```bash
// This assumes that you have composer installed globally
composer require digitive/vipps-socialite-provider
```

## 2. Service Provider

Add to app.php:

``` php
'providers' => [
    Laravel\Socialite\SocialiteServiceProvider::class,
    \SocialiteProviders\Manager\ServiceProvider::class,
];
```

## 3. Event Listener

* Add `SocialiteProviders\Manager\SocialiteWasCalled` event to your `listen[]` array  in `app/Providers/EventServiceProvider`.

Example:

```php
/**
 * The event handler mappings for the application.
 *
 * @var array
 */
protected $listen = [
    \SocialiteProviders\Manager\SocialiteWasCalled::class => [
        '\SocialiteProviders\Vipps\VippsExtendSocialite@handle',
    ],
];
```
## 4. Configuration setup

You will need to add vipps to the services configuration file so that after config files are cached for usage in production environment (Laravel command `artisan config:cache`) all config is still available.

#### Add to `config/services.php`.

```php
'vipps' => [
    'client_id' => env('VIPPS_CLIENT_ID'),
    'client_secret' => env('VIPPS_CLIENT_SECRET'),
    'redirect' => env('VIPPS_REDIRECT_URI'),
],
```

Remember to whitelist the redirect_uri in the [Vipps portal](https://portal.vipps.no/). Client ID and secret is also available in the [Vipps portal](https://portal.vipps.no/).

## 5. Usage

* [Laravel docs on configuration](http://laravel.com/docs/master/configuration)
* Guzzle version 7 or later is required
* You should now be able to use it like you would regularly use Socialite (assuming you have the facade installed):

To initiate the Vipps login, add this to your controller

```php
return Socialite::driver('vipps')->redirect();
```

You've now gotten a user token from Vipps in your callback function. Now we need to 
use the user token to get the phone number of the authenticated user.

```php
$user = Socialite::driver('vipps')->stateless()->user();
```

Example for a VippsAuthController:

```php
<?php
 
 namespace App\Http\Controllers\Api;
 
 use App\Http\Controllers\Controller;
 use Illuminate\Http\Request;
 use Laravel\Socialite\Facades\Socialite;
 
 class VippsAuthController extends Controller
 {
     // User clicked Login in with Vipps button
     public function index(Request $request)
     {
         return Socialite::driver('vipps')->redirect();
     }
 
     // Vipps callback function (VIPPS_REDIRECT_URL in .env)
     public function handleCallback()
     {
         $user = Socialite::driver('vipps')->stateless()->user();
 
         if (!$user) {
             //Handle errors for missing user
         }

         //Authenticate user
     }
}
```

If you need multiple redirect URLs you can define a redirect url in the controller.

Example for specified redirect URL:

```php
Socialite::driver('vipps')->redirectUrl($redirectUrl)->redirect();
```

The same goes for scopes

Example for specified scopes:

```php
Socialite::diver('vipps')->scopes(['openid', 'api_version_2', 'phoneNumber', 'name'])->redirect();
```

## Vipps guidelines

* When using Vipps login you need to use the login button svgs provided by Vipps.
Go to [Vipps design guidelines](https://github.com/vippsas/vipps-design-guidelines) for more info.

## License
MIT © [Digitive AS](https://digitive.no)

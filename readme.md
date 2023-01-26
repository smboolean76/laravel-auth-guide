# Laravel Auth - Backoffice & Frontoffice

## Creazione progetto

- Creo una cartella nel computer e la apro con VSCode
- Dal terminale lancio `composer create-project laravel/laravel .`
- Creo il DB su MAMP e lo collego al progetto settando le variabili nel file .env

## Setup progetto con autenticazione

- Installiamo il pacchetto laravel breeze `composer require laravel/breeze --dev`
- Creiamo lo scaffolding di default con blade (opzione 0, no, no) `php artisan breeze:install`
- Installiamo il pacchetto di preset `composer require pacificdev/laravel_9_preset`
- Lanciamo il comando di preset con auth `php artisan preset:ui bootstrap --auth`
- Installiamo e compiliamo gli asset `npm install && npm run dev`

## Eseguiamo le migration

Verifichaimo di aver creato e collegato correttamente il database, quindi lanciamo `php artisan migrate`

## Organizzazione del progetto

### Definizione rotte 🗺

definiamo ora le rotte per la parte di backoffice, raggruppandole con namespace, prefisso, middleware auth e name.

```php
// routes/web.php

// 🕊️ Rotte pubbliche del frontoffice
Route::get('/', function () {
    return view('welcome');
});

// 🚫 Tutte le rotte protette da autenticazione del backoffice
Route::middleware('auth')->prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', function () {
        return view('admin.dashboard');
    })->name('dashboard');
});

// 🛡️ Tutte le rotte di autenticazione (registrazione, login ecc...)
require __DIR__.'/auth.php';
```

### Modifica path della costante HOME ↩️

Cambiamo l'url di redirect dopo il login (al posto di `/dashboard`, scriviamo `/admin/dashboard`)

```php
// app/Providers/RouteServiceProvider.php

class RouteServiceProvider extends ServiceProvider
{
    /**
     * The path to the "home" route for your application.
     *
     * Typically, users are redirected here after authentication.
     *
     * @var string
     */
    public const HOME = '/admin/dashboard';

		// ...
}
```

### Organizzazione views 🗂

#### Layout base per dashboard

Creiamo il file `layouts/admin.blade.php`, che sarà il layout da utilizzare per la dashboard e tutte le pagine del backoffice.

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />

    <!-- CSRF Token -->
    <meta name="csrf-token" content="{{ csrf_token() }}" />

    <title>{{ config('app.name', 'Laravel') }}</title>

    <!-- Fontawesome 6 cdn -->
    <link
      rel="stylesheet"
      href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.2.0/css/all.min.css"
      integrity="sha512-xh6O/CkQoPOWDdYTDqeRdPCVd1SpvCA9XXcUnZS2FmJNp1coAFzvtCN9BmamE+4aHK8yyUHUSCcJHgXloTyT2A=="
      crossorigin="anonymous"
      referrerpolicy="no-referrer"
    />

    <!-- Fonts -->
    <link rel="dns-prefetch" href="//fonts.gstatic.com" />
    <link
      href="https://fonts.googleapis.com/css?family=Nunito"
      rel="stylesheet"
    />

    <!-- Usando Vite -->
    @vite(['resources/js/app.js'])
  </head>

  <body>
    <div id="app">
      <header
        class="navbar navbar-dark sticky-top bg-dark flex-md-nowrap p-2 shadow"
      >
        <a class="navbar-brand col-md-3 col-lg-2 me-0 px-3" href="/"
          >BoolPress</a
        >
        <button
          class="navbar-toggler position-absolute d-md-none collapsed"
          type="button"
          data-bs-toggle="collapse"
          data-bs-target="#sidebarMenu"
          aria-controls="sidebarMenu"
          aria-expanded="false"
          aria-label="Toggle navigation"
        >
          <span class="navbar-toggler-icon"></span>
        </button>
        <input
          class="form-control form-control-dark w-100"
          type="text"
          placeholder="Search"
          aria-label="Search"
        />
        <div class="navbar-nav">
          <div class="nav-item text-nowrap ms-2">
            <a
              class="nav-link"
              href="{{ route('logout') }}"
              onclick="event.preventDefault();
                    document.getElementById('logout-form').submit();"
            >
              {{ __('Logout') }}
            </a>
            <form
              id="logout-form"
              action="{{ route('logout') }}"
              method="POST"
              class="d-none"
            >
              @csrf
            </form>
          </div>
        </div>
      </header>

      <div class="container-fluid vh-100">
        <div class="row h-100">
          <nav
            id="sidebarMenu"
            class="col-md-3 col-lg-2 d-md-block bg-dark navbar-dark sidebar collapse"
          >
            <div class="position-sticky pt-3">
              <ul class="nav flex-column">
                <li class="nav-item">
                  <a
                    class="nav-link text-white {{ Route::currentRouteName() == 'admin.dashboard' ? 'bg-secondary' : '' }}"
                    href="{{route('admin.dashboard')}}"
                  >
                    <i class="fa-solid fa-tachometer-alt fa-lg fa-fw"></i>
                    Dashboard
                  </a>
                </li>
              </ul>
            </div>
          </nav>

          <main class="col-md-9 ms-sm-auto col-lg-10 px-md-4">
            @yield('content')
          </main>
        </div>
      </div>
    </div>
  </body>
</html>
```

#### Organizzazione views admin

Creaiamo una sottocartella `resources/views/admin/` e copiamoci il file **dashboard.blade.php**.
Qui inseriremo, divise per cartelle, tutte le views delle nostre CRUD di **backoffice**.

#### Modifica file dashboard.blade.php

Modifichiamo il file dashboard in modo che estenda il layout appena creato.

## Crud time 🥕

Arrivati a questo punto possiamo dedicarci alla creazione delle nostre **CRUD** per ogni singola risorsa.
Possiamo utilizzare il seguente comando che permette di creare model, migration, controller di tipo resource, seeder e classi Request (per gestire la validazione).
Es.
`php artisan make:model Post -rcmsR`

**N.B** il controller che viene generato tramite lo shortcut, viene messo nella cartella Controllers. Spostiamolo in una cartella Admin modificando il namespace e importanto la classe del Controller di base.

### Gestire le rotte

Aggiungiamo nel gruppo delle rotte protette da autenticazione la rotta per le CRUD di ogni singola risorsa.
**N.B** Verifica che il controller sia stato importato
Es.

```php
// routes/web.php

use App\Http\Controllers\Admin\PostController;

// ...

// 🚫 Tutte le rotte protette da autenticazione
Route::middleware('auth')->prefix('admin')->name('admin.')->group(function () {
    Route::get('/dashboard', function () {
        return view('admin.dashboard');
    })->name('dashboard');

    Route::resource('posts', PostController::class);
});

// ...
```

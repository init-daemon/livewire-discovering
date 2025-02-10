# LIVEWIRE DISCOVERING - Livewire Cheat Sheet

## Components

### Création d'un composant
- `php artisan make:livewire [CreatePost|create-post]` : **kebab-case** ou **PascalCase**.
- `php artisan make:livewire [Posts\\CreatePost|posts.create-post]` : Précise le **namespace** (création dans un sous-dossier, ici `posts`).

### Propriété d'un composant
```php
namespace App\Livewire;

use Livewire\Component;

class CreatePost extends Component
{
    public $title = 'Post title...'; // Accessible via Livewire, personnalisé avec :title="nouveau titre"
    
    public function render()
    {
        return view('livewire.create-post', [
            'author' => Auth::user()->name, // Propriété en lecture seule
        ]);
    }
}
```

### Accéder aux propriétés dans une vue
```blade
<!-- livewire.create-post -->
<div>
    <h1>Title: "{{ $title }}"</h1>
</div>
```

## `wire:key` : Gestion des éléments dans une boucle `foreach`
```blade
<div>
    @foreach ($posts as $post)
        <div wire:key="{{ $post->id }}"></div>
        @livewire(PostItem::class, ['post' => $post], key($post->id)) <!-- Si on utilise @livewire -->
    @endforeach
</div>
```

## `wire:model` : Synchronisation des données
```blade
<form wire:submit.prevent="submit">
    <label for="title">Title:</label>
    <input type="text" id="title" wire:model="title"> 
</form>
```
```php
class Card extends Component
{
    public $title = 'title';
    
    public function submit()
    {
        dd($this->title); // Affiche la nouvelle valeur
    }
}
```

### `wire:model.live` :
- Synchronisation en temps réel (utile pour des validations en direct).

## Passer des données à un composant
```blade
<livewire:create-post title="Initial Title" /> <!-- Valeur statique -->
<livewire:create-post :title="$title" /> <!-- Valeur dynamique -->
```
- Si `$title` est modifié **après** le chargement du composant, **la nouvelle valeur ne sera pas transmise**.

### `mount` : Hook pour initialiser les valeurs des propriétés
```php
public function mount($title, $description)
{
    $this->description = strtoupper($title);
    $this->title = $description;
}
```

## Full-page components
- Associe un composant Livewire à une route pour créer une page complète.
```php
Route::get('/posts/create', CreatePost::class); // Affiche CreatePost sur /posts/create
```

### Le layout
```bash
php artisan livewire:layout # Création du fichier resources/views/components/layouts/app.blade.php
```

```blade
<!-- resources/views/components/layouts/app.blade.php -->
<html>
    <head>
        <title>{{ $title ?? 'Page Title' }}</title>
    </head>
    <body>
        {{ $slot }} <!-- Contenu de la page -->
    </body>
</html>
```

### Configuration globale
```php
// config/layouts.php
'layout' => 'layouts.app', // Définit le layout global
```

### Définir un layout pour un composant full-page
1. Au-dessus de `render()`
```php
class CreatePost extends Component
{
    #[Layout('layouts.app')]
    public function render()
    {
        return view('livewire.create-post');
    }
}
```

2. Au-dessus de la déclaration de la classe
```php
#[Layout('layouts.app')] 
class CreatePost extends Component
```

3. Méthode `layout()`
```php
public function render()
{
    return view('livewire.create-post')->layout('layouts.app');
}
```

4. Utilisation traditionnelle de Blade avec `@extends`
```blade
<body>
    @yield('content')
</body>
```

```php
public function render()
{
    return view('livewire.show-posts')->extends('layouts.app');
}
```

#### `#[Title('Titre de la page')]` : Définir le titre de la page
1. Déclaration au-dessus de `render()` ou `class`
```php
#[Title('Create Post')]
public function render()
{
    return view('livewire.create-post');
}
```
2. Utilisation de la méthode `title()`
```php
return view('livewire.create-post')->title('Create Post');
```

#### Accéder aux paramètres de la route
```php
class ShowPost extends Component
{
    public Post $post; 
    
    public function mount($id)
    {
        $this->post = Post::findOrFail($id); // Route: /posts/{id}
    }

    public function render()
    {
        return view('livewire.show-post');
    }
}
```

# LIVEWIRE DISCOVERING - Livewire Cheat Sheet

# *Components*

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
- Lors d'une execution d'une action, ici c'est l'execution de submit
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



# **Properties**

## Initialisation des Propriétés

...-

## Affectation en Masse

```php
class UpdatePost extends Component
{
    public $post;
    public $title;
    public $description;

    public function mount(Post $post)
    {
        $this->post = $post;
        // `only` retourne un tableau associatif,
        // et `fill` permet d'assigner automatiquement les propriétés.
        $this->fill($post->only('title', 'description'));
    }
}
```

## `reset()`: Réinitialisation des Propriétés

- `` réinitialise une propriété à sa **valeur initiale** (avant `mount()`), et non à la valeur définie dans `mount()`.

### Exemple :

```php
//...
    public $todos = [];
    public $todo = '';  // Valeur par défaut

    public function mount()
    {
        $this->todo = 'Initial Todo';  // Modifiée dans mount()
    }

    public function addTodo()
    {
        $this->todos[] = $this->todo;
        $this->reset('todo');  // Réinitialise 'todo' à sa valeur initiale (vide)
    }
//...
```

### Comportement :

- remet la valeur à son état **avant `mount()`**.
- Pour réinitialiser à la valeur définie dans `mount()`, fais-le manuellement :

```php
$this->todo = 'Initial Todo';  // Réinitialisation manuelle
```

## `$this->pull('todo')`

- Fonctionne comme `reset()`, mais **retourne l'ancienne valeur avant réinitialisation**.

```php
$this->todos[] = $this->pull('todo'); // Retourne l'ancienne valeur et réinitialise `todo`
```

## Types Supportés par Livewire

### Types Primitifs

- `array`, `string`, `int`, `float`, `bool`, `null`

### Types Laravel

> Ces types sont **désérialisés en JSON** et **reconvertis en PHP** à chaque requête.

- `BackedEnum`
- `Illuminate\Support\Collection`
- `Illuminate\Database\Eloquent\Collection`
- `Illuminate\Database\Eloquent\Model`
- `DateTime`
- `Carbon\Carbon`
- `Illuminate\Support\Stringable`

### Types Personnalisés

- Utilisation de `Wireables` et `Synthesizers`

## Accès aux Propriétés via JavaScript

### Récupération

- `$wire.propertyName`
- `$wire.get('propertyName')`

### Manipulation

- `$wire.todo = nouvelleValeur` *(pas de synchronisation serveur)*
- `$wire.set('todo', nouvelleValeur, true/false)` *(synchronisation si **``**)*

## Sécurité

Il est essentiel de valider et d'autoriser les propriétés avant de les enregistrer dans la base de données, comme pour une requête de contrôleur.

### Sécuriser les Propriétés

- `` : Rend une propriété en lecture seule (utile pour verrouiller un `id`).
- `` avant modification :
  ```php
  public function update()
  {
      $post = Post::findOrFail($this->id);    
      $this->authorize('update', $post); // https://laravel.com/docs/11.x/authorization
      $post->update(...);
  }
  ```
- \*\*Utilisation d'un \*\*`` : Livewire verrouille automatiquement l'`id` pour éviter sa modification.
  ```php
  class UpdatePost extends Component
  {
      public Post $post;
      public $title;
      public $content;

      public function mount(Post $post)
      {
          $this->post = $post;
          $this->title = $post->title;
          $this->content = $post->content;
      }

      public function update()
      {
          $this->post->update([
              'title' => $this->title,
              'content' => $this->content,
          ]);
      }
  ```

### Exposition des Éléments Sensibles

#### Nom de classe exposé

- Solution : Utiliser `Relation::morphMap()` pour un alias dans une relation polymorphique.

```json
{
    "type": "model",
    "class": "App\Models\Post", // Classe exposée
    "key": 1,
    "relationships": []
}
```

```php
// Solution
namespace App\Providers;
use Illuminate\Support\ServiceProvider;
use Illuminate\Database\Eloquent\Relations\Relation;

class AppServiceProvider extends ServiceProvider
{
    public function boot()
    {
        Relation::morphMap([
            'post' => 'App\Models\Post', // "class": "post"
        ]);
    }
}
```

### Contraintes Eloquent non conservées entre requêtes dans Livewire [?]

...

## `#[Computed]`: Propriété Calculée

- Une propriété `` est une méthode **mise en cache durant la requête**, évitant un recalcul inutile. Elle est accessible comme une propriété publique.

### Déclaration

```php
use Livewire\Attributes\Computed;
//..
    public $userId;

    #[Computed]
    public function user()
    {
        return User::find($this->userId);
    }
//..
```

### Utilisation dans Blade

Accès à la propriété calculée via `$this` :

```blade
<div>
    <h1>{{ $this->user->name }}</h1>
</div>
```

### **Avantages de **``** en Livewire**

1. **Optimisation des performances** : mise en cache des valeurs pour éviter des appels inutiles.
2. **Accès aux données dérivées** : permet de générer des valeurs basées sur d'autres propriétés.
3. **Simplification du rendu Blade** : accessible directement dans le template via `$this->property`.
4. **Busting du cache** : possibilité d'invalider le cache avec `unset($this->property)`.
5. **Cache persistant entre requêtes** : `#[Computed(persist: true)]` conserve la valeur jusqu'à expiration.
6. **Cache global entre composants** : `#[Computed(cache: true)]` partage une valeur en cache entre plusieurs composants.
7. **Conditionnement du chargement des données** : exécute une requête seulement si elle est nécessaire.
8. **Utilisation dans les templates inline** : permet d’accéder aux données sans méthode `render()`.
9. **Réduction du boilerplate** : supprime le besoin de passer des données via `render()`.

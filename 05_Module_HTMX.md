# Module 5: Django-HTMX - Interactions sans JavaScript ⚡

## 🎯 Objectif du Module
Utiliser **HTMX** pour créer des interactions modernes **sans JavaScript**, juste avec **HTML + Django Tags**.

---

## 1️⃣ Qu'est-ce qu'HTMX ?

**HTMX** = "HTML eXtended" = HTML avec des **super pouvoirs**!

### Sans HTMX (Traditionnel)
```
1. Utilisateur clique sur un lien
2. La PAGENTIÈRE se recharge
3. Attendez... attendez... 🔄
4. La page s'affiche enfin
```

### Avec HTMX (Moderne)
```
1. Utilisateur clique sur un bouton
2. JUSTE le morceau de page qu'il faut se met à jour
3. Instantanément! ⚡
4. Pas de rechargement complet
```

---

## 2️⃣ Installation et Configuration

### Installation
```bash
pip install django-htmx
```

### settings.py
```python
INSTALLED_APPS = [
    'django_htmx',
    # ... autres apps
]
```

### Template (base.html)
```html
<!DOCTYPE html>
<html>
<head>
    <script src="https://unpkg.com/htmx.org@1.9.10"></script>
</head>
<body>
    <!-- Ton contenu -->
</body>
</html>
```

---

## 3️⃣ Les Attributs HTMX Essentiels

### Attribut 1: `hx-get` - Charger du HTML
```html
<!-- Quand on clique, charger le HTML de cette URL -->
<button hx-get="/books/popular/">
    Charger les livres populaires
</button>

<!-- Résultat: Django appelle la view /books/popular/
     La view retourne du HTML
     HTMX l'insère dans la page! -->
```

### Attribut 2: `hx-post` - Envoyer des données
```html
<!-- Quand on soumet, envoyer les données en POST -->
<form hx-post="/books/add/">
    <input name="title" placeholder="Titre">
    <input name="author" placeholder="Auteur">
    <button type="submit">Ajouter</button>
</form>

<!-- Résultat: Django reçoit POST
     Traite les données
     Retourne du HTML
     HTMX l'affiche! -->
```

### Attribut 3: `hx-target` - Où insérer le résultat
```html
<!-- Insérer dans cet élément -->
<div id="books-list">
    Chargement...
</div>

<button hx-get="/books/" hx-target="#books-list">
    Charger les livres
</button>

<!-- Résultat: le contenu de #books-list est remplacé -->
```

### Attribut 4: `hx-swap` - Comment insérer
```html
<!-- Replace: remplacer l'élément entier (défaut) -->
<button hx-get="/new-content/" hx-swap="innerHTML">
    Les enfants sont remplacés
</button>

<!-- outerHTML: remplacer l'élément + ses enfants -->
<button hx-get="/new-element/" hx-swap="outerHTML">
    L'élément entier est remplacé
</button>

<!-- beforeend: insérer à la fin (append) -->
<ul id="items">
    <li>Item 1</li>
</ul>
<button hx-get="/new-item/" hx-target="#items" hx-swap="beforeend">
    Ajouter un item à la fin
</button>

<!-- afterbegin: insérer au début (prepend) -->
<button hx-get="/new-item/" hx-target="#items" hx-swap="afterbegin">
    Ajouter un item au début
</button>
```

### Attribut 5: `hx-trigger` - Quand déclencher
```html
<!-- Au clic (défaut) -->
<button hx-get="/action/">Clique-moi</button>

<!-- Au changement -->
<select hx-get="/filter/" hx-trigger="change">
    <option>Option 1</option>
    <option>Option 2</option>
</select>

<!-- Avec délai -->
<input hx-get="/search/" hx-trigger="keyup changed delay:500ms" />

<!-- Après un délai -->
<div hx-get="/poll/" hx-trigger="every 5s"></div>
```

### Attribut 6: `hx-confirm` - Demander avant
```html
<button hx-post="/delete/" hx-confirm="Êtes-vous sûr?">
    Supprimer
</button>
```

---

## 4️⃣ Exemple Complet: Ajouter Un Livre avec HTMX

### views.py
```python
from django.shortcuts import render, redirect
from .models import Book
from .forms import BookForm

def books_list(request):
    books = Book.objects.all()
    return render(request, 'library/books_list.html', {'books': books})

# Cette view RETOURNE JUSTE DU HTML (pas de page entière)
def add_book_form(request):
    form = BookForm()
    return render(request, 'library/add_book_form.html', {'form': form})

# Cette view TRAITE l'ajout et RETOURNE JUSTE LE NOUVEAU LIVRE
def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        
        if form.is_valid():
            book = form.save()
            
            # HTMX: retourner JUSTE le livre créé
            return render(request, 'library/book_item.html', {'book': book})
        else:
            # HTMX: retourner le form avec les erreurs
            return render(request, 'library/add_book_form.html', {'form': form})
    
    return redirect('books_list')
```

### urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('books/', views.books_list, name='books_list'),
    path('books/add/form/', views.add_book_form, name='add_book_form'),
    path('books/add/', views.add_book, name='add_book'),
]
```

### books_list.html
```html
{% extends "base.html" %}

{% block content %}

<div class="library">
    <h1>Bibliothèque</h1>
    
    <!-- Bouton pour afficher le formulaire -->
    <button hx-get="{% url 'add_book_form' %}" 
            hx-target="#add-book-container"
            class="btn-primary">
        ➕ Ajouter un livre
    </button>
    
    <!-- Container pour le formulaire -->
    <div id="add-book-container"></div>
    
    <!-- Liste des livres -->
    <div id="books-container">
        {% for book in books %}
            {% include 'library/book_item.html' %}
        {% endfor %}
    </div>
</div>

{% endblock %}
```

### add_book_form.html (JUSTE le formulaire, pas toute la page!)
```html
<form hx-post="{% url 'add_book' %}"
      hx-target="#books-container"
      hx-swap="beforeend"
      class="form-add-book">
    
    {% csrf_token %}
    
    {{ form.title }}
    {{ form.author }}
    
    {% if form.non_field_errors %}
        <div class="errors">{{ form.non_field_errors }}</div>
    {% endif %}
    
    <button type="submit">Ajouter</button>
    
    <!-- Fermer le formulaire après succès -->
    <script>
        // Après succès, vider le container
        document.getElementById('add-book-container').innerHTML = '';
    </script>
</form>
```

### book_item.html (JUSTE UN LIVRE!)
```html
<div class="book-item" id="book-{{ book.id }}">
    <h3>{{ book.title }}</h3>
    <p>Auteur: {{ book.author }}</p>
    <a href="{% url 'book_detail' book.id %}">Voir détails</a>
    
    <button hx-post="{% url 'delete_book' book.id %}"
            hx-confirm="Êtes-vous sûr?"
            hx-target="#book-{{ book.id }}"
            hx-swap="swap:outerHTML swap:show:top"
            class="btn-danger">
        🗑️ Supprimer
    </button>
</div>
```

---

## 5️⃣ Exemple 2: Filtrer en Temps Réel (Search)

### views.py
```python
def search_books(request):
    query = request.GET.get('q', '')
    
    if query:
        books = Book.objects.filter(title__icontains=query)
    else:
        books = Book.objects.none()
    
    # Retourner JUSTE les résultats (pas toute la page)
    return render(request, 'library/search_results.html', {'books': books})
```

### search.html
```html
{% extends "base.html" %}

{% block content %}

<div class="search-container">
    <h1>Chercher des Livres</h1>
    
    <!-- L'input déclenche la recherche au keystroke -->
    <input type="text"
           name="q"
           placeholder="Chercher un livre..."
           hx-get="{% url 'search_books' %}"
           hx-target="#search-results"
           hx-trigger="keyup changed delay:300ms"
           class="search-input">
    
    <!-- Container pour les résultats -->
    <div id="search-results"></div>
</div>

{% endblock %}
```

### search_results.html
```html
{% if books %}
    <div class="results-count">{{ books|length }} résultat{{ books|pluralize }}</div>
    
    <ul class="results-list">
        {% for book in books %}
            <li>
                <a href="{% url 'book_detail' book.id %}">
                    {{ book.title }}
                </a>
                <span class="author">{{ book.author }}</span>
            </li>
        {% endfor %}
    </ul>
{% elif query %}
    <p class="no-results">Aucun livre trouvé pour "{{ query }}"</p>
{% else %}
    <p class="placeholder">Commence à taper pour chercher...</p>
{% endif %}
```

---

## 6️⃣ Exemple 3: Like/Unlike avec HTMX

### models.py
```python
class BookLike(models.Model):
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    book = models.ForeignKey(Book, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        unique_together = ('user', 'book')
```

### views.py
```python
from django.contrib.auth.decorators import login_required

@login_required
def toggle_like(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    
    like, created = BookLike.objects.get_or_create(
        user=request.user,
        book=book
    )
    
    if not created:
        # Déjà liké, donc unlike
        like.delete()
    
    # Retourner JUSTE le bouton mis à jour
    is_liked = BookLike.objects.filter(
        user=request.user,
        book=book
    ).exists()
    
    return render(request, 'library/like_button.html', {
        'book': book,
        'is_liked': is_liked,
        'likes_count': book.booklike_set.count(),
    })
```

### like_button.html
```html
<div id="like-btn-{{ book.id }}">
    {% if is_liked %}
        <button hx-post="{% url 'toggle_like' book.id %}"
                hx-target="#like-btn-{{ book.id }}"
                hx-swap="innerHTML"
                class="btn-liked">
            ❤️ {{ likes_count }}
        </button>
    {% else %}
        <button hx-post="{% url 'toggle_like' book.id %}"
                hx-target="#like-btn-{{ book.id }}"
                hx-swap="innerHTML"
                class="btn-not-liked">
            🤍 {{ likes_count }}
        </button>
    {% endif %}
</div>
```

---

## 7️⃣ Pièges Courants avec HTMX

### ❌ Piège 1: Oublier {% csrf_token %}
```html
<!-- MAUVAIS -->
<form hx-post="/books/add/">
    <!-- N'oublie pas le token! -->
</form>

<!-- BON -->
<form hx-post="/books/add/">
    {% csrf_token %}
</form>
```

### ❌ Piège 2: Retourner la page entière
```python
# MAUVAIS: retourner toute la page
def add_book(request):
    form = BookForm(request.POST)
    return render(request, 'library/books_list.html', {...})  # BAD!

# BON: retourner JUSTE le morceau
def add_book(request):
    form = BookForm(request.POST)
    if form.is_valid():
        book = form.save()
        return render(request, 'library/book_item.html', {'book': book})
```

### ❌ Piège 3: Mélanger GET et POST
```html
<!-- MAUVAIS: GET ne peut pas envoyer un formulaire -->
<form hx-get="/books/add/">
    <input name="title">
    <button>Ajouter</button>
</form>

<!-- BON -->
<form hx-post="/books/add/">
    <input name="title">
    <button>Ajouter</button>
</form>
```

---

## ✅ Points Clés À Retenir

- ✅ HTMX = mise à jour partielle sans recharger
- ✅ `hx-get` = charger du HTML
- ✅ `hx-post` = envoyer des données
- ✅ `hx-target` = où insérer le résultat
- ✅ `hx-swap` = comment insérer
- ✅ Les views doivent retourner JUSTE le morceau, pas la page entière!
- ✅ Toujours inclure `{% csrf_token %}` dans les formulaires POST

**Prêt pour Module 6? La manipulation des DONNÉES (JSON, CSV, Listes)! 🎯**

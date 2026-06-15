# Module 2: Les Views - Comment Elles Reçoivent les Données 🎯

## 🎯 Objectif du Module
Maîtriser les **views Function-Based (FBV)** et comprendre comment elles manipulent les données.

---

## 1️⃣ Qu'est-ce Qu'une View ?

Une **view** est une **fonction Python** qui:
1. **Reçoit** un `request` (REQUEST)
2. **Traite** les données
3. **Retourne** une réponse (RESPONSE)

```python
def ma_view(request):
    # 1. Recevoir
    print(request.method)  # 'GET' ou 'POST'
    
    # 2. Traiter
    data = "Voici mes données"
    
    # 3. Retourner
    return HttpResponse(data)
```

---

## 2️⃣ Les Types de RESPONSE

### Type 1: HttpResponse (Texte brut)
```python
from django.http import HttpResponse

def simple_view(request):
    return HttpResponse("Bonjour!")
```

### Type 2: render() (Template HTML) ⭐ LE PLUS UTILISÉ
```python
from django.shortcuts import render

def books_view(request):
    books = ['Django', 'Python', 'Web']
    return render(request, 'books.html', {'books': books})
```

### Type 3: JsonResponse (Données JSON)
```python
from django.http import JsonResponse

def api_view(request):
    data = {'status': 'ok', 'count': 42}
    return JsonResponse(data)
```

### Type 4: redirect() (Redirection)
```python
from django.shortcuts import redirect

def old_view(request):
    return redirect('new_view_name')
```

---

## 3️⃣ Accéder aux Données du REQUEST

### Données GET (dans l'URL)
```html
<!-- Utilisateur clique sur ce lien -->
<a href="/search/?query=django&sort=date">Chercher</a>
```

```python
def search_view(request):
    # Accéder aux données GET
    query = request.GET.get('query', 'défaut')  # 'django'
    sort = request.GET.get('sort', 'nom')       # 'date'
    
    # request.GET est un DICTIONNAIRE!
    print(request.GET)  # <QueryDict: {'query': ['django'], 'sort': ['date']}>
    
    return render(request, 'search.html', {
        'query': query,
        'sort': sort,
    })
```

### Données POST (du formulaire)
```html
<!-- Utilisateur remplit et envoie le formulaire -->
<form method="post" action="/add-book/">
    <input name="title" value="Django Pro">
    <input name="author" value="Alex">
    <button type="submit">Ajouter</button>
</form>
```

```python
def add_book_view(request):
    if request.method == 'POST':
        # Accéder aux données POST
        title = request.POST.get('title')    # 'Django Pro'
        author = request.POST.get('author')  # 'Alex'
        
        # request.POST est un DICTIONNAIRE!
        print(request.POST)  # <QueryDict: {'title': ['Django Pro'], 'author': ['Alex']}>
        
        # Créer un livre
        Book.objects.create(title=title, author=author)
        
        return redirect('books_list')
    
    # GET: afficher le formulaire
    return render(request, 'add_book.html')
```

### Données de l'utilisateur connecté
```python
def profile_view(request):
    if request.user.is_authenticated:
        username = request.user.username  # 'tresor'
        email = request.user.email         # 'tresor@email.com'
        
        return render(request, 'profile.html', {
            'username': username,
            'email': email,
        })
    else:
        return redirect('login')
```

---

## 4️⃣ Traiter les Données (LA PARTIE IMPORTANTE!)

### Récupérer de la Base de Données
```python
from .models import Book

def books_list(request):
    # Récupérer TOUS les livres
    books = Book.objects.all()  # C'est une LISTE (QuerySet)
    
    # Filtrer les livres
    django_books = Book.objects.filter(title__contains='Django')
    
    # Compter
    count = Book.objects.count()
    
    # Récupérer UNE instance
    try:
        first_book = Book.objects.get(id=1)
    except Book.DoesNotExist:
        first_book = None
    
    return render(request, 'books.html', {
        'books': books,
        'django_books': django_books,
        'count': count,
        'first_book': first_book,
    })
```

### Manipuler les Données (Listes, Dicts)
```python
def stats_view(request):
    books = Book.objects.all()
    
    # LISTE: extraire les titres
    titles = [book.title for book in books]
    # ['Django 101', 'Python Pro', 'Web Dev']
    
    # DICTIONNAIRE: compter par auteur
    authors_stats = {}
    for book in books:
        if book.author not in authors_stats:
            authors_stats[book.author] = 0
        authors_stats[book.author] += 1
    # {'Alex': 2, 'Bob': 1}
    
    # DICTIONNAIRE: créer un résumé
    summary = {
        'total_books': len(books),
        'authors': list(set(book.author for book in books)),
        'by_author': authors_stats,
    }
    
    return render(request, 'stats.html', {'summary': summary})
```

---

## 5️⃣ Exemple Complet: Bibliothèque

### models.py
```python
from django.db import models

class Author(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    
    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE)
    isbn = models.CharField(max_length=13)
    published_year = models.IntegerField()
    
    def __str__(self):
        return self.title
```

### views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from .models import Book, Author

# View 1: Lister tous les livres
def books_list(request):
    # Récupérer des données
    books = Book.objects.all().order_by('-published_year')
    
    # Ajouter du contexte
    context = {
        'books': books,
        'count': books.count(),
    }
    
    # Retourner le template avec les données
    return render(request, 'library/books_list.html', context)

# View 2: Voir les détails d'un livre
def book_detail(request, book_id):
    # Récupérer UN livre spécifique
    book = get_object_or_404(Book, id=book_id)
    
    # Récupérer les livres du même auteur
    related_books = Book.objects.filter(author=book.author).exclude(id=book_id)
    
    return render(request, 'library/book_detail.html', {
        'book': book,
        'related_books': related_books,
    })

# View 3: Créer un livre
def create_book(request):
    if request.method == 'POST':
        # Recevoir les données du formulaire
        title = request.POST.get('title')
        author_id = request.POST.get('author_id')
        isbn = request.POST.get('isbn')
        published_year = request.POST.get('published_year')
        
        # Créer le livre
        Book.objects.create(
            title=title,
            author_id=author_id,
            isbn=isbn,
            published_year=published_year,
        )
        
        # Rediriger vers la liste
        return redirect('books_list')
    
    # GET: afficher le formulaire
    authors = Author.objects.all()
    return render(request, 'library/create_book.html', {
        'authors': authors,
    })

# View 4: Chercher des livres
def search_books(request):
    query = request.GET.get('q', '')  # Récupérer la requête de recherche
    
    if query:
        # Filtrer les livres par titre ou auteur
        books = Book.objects.filter(
            title__icontains=query
        ) | Book.objects.filter(
            author__name__icontains=query
        )
    else:
        books = []
    
    return render(request, 'library/search.html', {
        'query': query,
        'books': books,
        'results_count': len(books),
    })
```

### urls.py
```python
from django.urls import path
from . import views

urlpatterns = [
    path('books/', views.books_list, name='books_list'),
    path('books/<int:book_id>/', views.book_detail, name='book_detail'),
    path('books/create/', views.create_book, name='create_book'),
    path('search/', views.search_books, name='search_books'),
]
```

---

## 6️⃣ Les Pièges Courants

### ❌ Piège 1: Oublier le context
```python
# MAUVAIS
def bad_view(request):
    books = Book.objects.all()
    return render(request, 'books.html')  # books n'est pas passé!

# BON
def good_view(request):
    books = Book.objects.all()
    return render(request, 'books.html', {'books': books})
```

### ❌ Piège 2: N-queries problem
```python
# MAUVAIS (lent!)
books = Book.objects.all()
for book in books:
    print(book.author.name)  # Une requête par livre!

# BON (rapide!)
books = Book.objects.select_related('author').all()
for book in books:
    print(book.author.name)  # Une seule requête!
```

### ❌ Piège 3: Oublier les exceptions
```python
# MAUVAIS
book = Book.objects.get(id=999)  # Crash si n'existe pas!

# BON
book = get_object_or_404(Book, id=999)  # Retourne 404 automatiquement
```

---

## 7️⃣ Exercice 2: Créer Tes Views

### Ton Défi:
1. Crée une view `author_list()` qui récupère tous les auteurs
2. Crée une view `author_detail()` qui affiche les livres d'un auteur
3. Crée une view `author_books_count()` qui affiche le nombre total de livres par auteur

### Structure à Créer:
```python
def author_list(request):
    # À faire
    pass

def author_detail(request, author_id):
    # À faire
    pass

def author_books_count(request):
    # À faire
    pass
```

**Solution en bas de ce fichier!**

---

## ✅ Points Clés

- ✅ View = fonction qui reçoit `request` et retourne `response`
- ✅ `render()` = envoyer un template avec des données
- ✅ `JsonResponse()` = envoyer du JSON (pour HTMX!)
- ✅ `request.GET` = données de l'URL
- ✅ `request.POST` = données du formulaire
- ✅ `context` = dictionnaire avec les données
- ✅ Toujours passer le `context` à `render()`!

---

## 🎁 SOLUTIONS AUX EXERCICES

### Solution Exercice 2:
```python
# Solution 1: Lister tous les auteurs
def author_list(request):
    authors = Author.objects.all()
    return render(request, 'library/author_list.html', {
        'authors': authors,
        'count': authors.count(),
    })

# Solution 2: Détail d'un auteur avec ses livres
def author_detail(request, author_id):
    author = get_object_or_404(Author, id=author_id)
    books = Book.objects.filter(author=author)
    
    return render(request, 'library/author_detail.html', {
        'author': author,
        'books': books,
        'books_count': books.count(),
    })

# Solution 3: Compter les livres par auteur
def author_books_count(request):
    # Créer un dictionnaire avec author: count
    authors = Author.objects.all()
    author_stats = {}
    
    for author in authors:
        author_stats[author.name] = Book.objects.filter(author=author).count()
    
    # OU avec une boucle en un ligne (Pythonique!)
    author_stats = {
        author.name: Book.objects.filter(author=author).count()
        for author in authors
    }
    
    return render(request, 'library/author_stats.html', {
        'author_stats': author_stats,
    })
```

**Prêt pour Module 3? On va apprendre les FORMS! 📝**

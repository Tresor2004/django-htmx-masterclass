# Module 3: Les Forms - Validation et Circulation des Données 📝

## 🎯 Objectif du Module
Comprendre comment les **Django Forms** valident et circulent les données de manière sécurisée.

---

## 1️⃣ Pourquoi les Django Forms ?

### Sans Django Forms (DANGEREUX!)
```python
# ❌ MAUVAIS - Pas de validation, pas de sécurité!
def add_book(request):
    if request.method == 'POST':
        title = request.POST.get('title')  # Quoi si c'est vide?
        author_id = request.POST.get('author_id')  # Quoi si ça n'existe pas?
        
        # On crée le livre directement... BOOM si données invalides!
        Book.objects.create(title=title, author_id=author_id)
```

### Avec Django Forms (SÛRE!)
```python
# ✅ BON - Validation automatique et sécurité!
from django import forms

class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author']

def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        
        if form.is_valid():  # Valide les données automatiquement!
            form.save()
            return redirect('books_list')
    else:
        form = BookForm()
    
    return render(request, 'add_book.html', {'form': form})
```

**Avantages:**
- ✅ Validation automatique
- ✅ Protection CSRF (sécurité)
- ✅ Affichage HTML automatique
- ✅ Messages d'erreur prêts
- ✅ Type checking (int, email, etc.)

---

## 2️⃣ Créer Une Form

### Type 1: ModelForm (directement du Model)
```python
from django import forms
from .models import Book, Author

# Form directement liée au model Book
class BookForm(forms.ModelForm):
    class Meta:
        model = Book  # Utilise le model Book
        fields = ['title', 'author', 'isbn', 'published_year']  # Champs à afficher


# Form avec personnalisation
class AuthorForm(forms.ModelForm):
    # Ajouter une aide text
    name = forms.CharField(
        max_length=100,
        help_text="Nom complet de l'auteur"
    )
    
    class Meta:
        model = Author
        fields = ['name', 'email']
        labels = {
            'name': 'Nom de l\'auteur',
            'email': 'Email',
        }
```

### Type 2: Regular Form (custom)
```python
class SearchForm(forms.Form):
    # Pas lié à un model, juste pour récupérer des données
    query = forms.CharField(
        max_length=100,
        label='Chercher un livre',
        required=False,
    )
    
    sort_by = forms.ChoiceField(
        choices=[
            ('title', 'Titre'),
            ('author', 'Auteur'),
            ('date', 'Date'),
        ],
        label='Trier par',
    )
    
    year_from = forms.IntegerField(
        label='Année de (au moins)',
        required=False,
    )
```

---

## 3️⃣ Utiliser une Form dans une View

### Le Cycle Complet
```python
from django.shortcuts import render, redirect
from .forms import BookForm

def add_book(request):
    # Step 1: User visits GET /books/add/
    if request.method == 'GET':
        form = BookForm()  # Créer une form vide
        return render(request, 'add_book.html', {'form': form})
    
    # Step 2: User submits POST /books/add/
    if request.method == 'POST':
        # Créer une form avec les données POST
        form = BookForm(request.POST)
        
        # Valider les données
        if form.is_valid():
            # Les données sont correctes!
            form.save()  # Sauvegarder automatiquement
            return redirect('books_list')
        else:
            # Les données sont invalides
            # Réafficher le form avec les erreurs
            return render(request, 'add_book.html', {'form': form})


# Code plus concis (même chose):
def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('books_list')
    else:
        form = BookForm()
    
    return render(request, 'add_book.html', {'form': form})
```

---

## 4️⃣ Accéder aux Données de la Form

### Après Validation
```python
def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        
        if form.is_valid():
            # Accéder aux données nettoyées et validées
            title = form.cleaned_data['title']
            author = form.cleaned_data['author']
            isbn = form.cleaned_data['isbn']
            
            print(f"Titre: {title}")
            print(f"Auteur: {author}")
            
            # Ou juste sauvegarder directement
            form.save()
            
            return redirect('books_list')
    
    else:
        form = BookForm()
    
    return render(request, 'add_book.html', {'form': form})
```

### Accéder aux Erreurs
```python
def add_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST)
        
        if not form.is_valid():
            # form.errors est un DICTIONNAIRE d'erreurs
            print(form.errors)
            # {'title': ['Ce champ est obligatoire.'], 'isbn': ['Format invalide.']}
            
            # Accéder à une erreur spécifique
            if 'title' in form.errors:
                print(form.errors['title'])
            
            # Afficher toutes les erreurs
            for field, errors in form.errors.items():
                for error in errors:
                    print(f"{field}: {error}")
        
        return render(request, 'add_book.html', {'form': form})
    
    form = BookForm()
    return render(request, 'add_book.html', {'form': form})
```

---

## 5️⃣ Exemple Complet: Formulaire de Réaction

### models.py
```python
from django.db import models
from django.contrib.auth.models import User

class Review(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    rating = models.IntegerField(choices=[(i, i) for i in range(1, 6)])  # 1 à 5
    comment = models.TextField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return f"Review de {self.user} pour {self.book}"
```

### forms.py
```python
from django import forms
from .models import Review

class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = ['rating', 'comment']
        widgets = {
            'rating': forms.RadioSelect(
                choices=[(i, f"{i} étoiles") for i in range(1, 6)]
            ),
            'comment': forms.Textarea(
                attrs={
                    'rows': 5,
                    'placeholder': 'Votre avis sur le livre...',
                    'class': 'form-control',
                }
            ),
        }
        labels = {
            'rating': 'Note',
            'comment': 'Commentaire',
        }
```

### views.py
```python
from django.shortcuts import render, redirect, get_object_or_404
from django.contrib.auth.decorators import login_required
from .models import Book, Review
from .forms import ReviewForm

@login_required  # L'utilisateur doit être connecté
def add_review(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    
    if request.method == 'POST':
        form = ReviewForm(request.POST)
        
        if form.is_valid():
            # Créer l'instance mais ne pas sauvegarder
            review = form.save(commit=False)
            
            # Ajouter les données supplémentaires
            review.book = book
            review.user = request.user
            
            # Maintenant sauvegarder
            review.save()
            
            return redirect('book_detail', book_id=book.id)
    else:
        form = ReviewForm()
    
    return render(request, 'add_review.html', {
        'form': form,
        'book': book,
    })

def book_detail(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    reviews = Review.objects.filter(book=book).order_by('-created_at')
    
    # Calculer la moyenne des notes
    if reviews.count() > 0:
        avg_rating = sum(r.rating for r in reviews) / reviews.count()
    else:
        avg_rating = 0
    
    return render(request, 'book_detail.html', {
        'book': book,
        'reviews': reviews,
        'avg_rating': avg_rating,
        'reviews_count': reviews.count(),
    })
```

---

## 6️⃣ Valider les Données Personnalisées

### Validation Simple
```python
class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = ['rating', 'comment']
    
    # Valider un champ spécifique
    def clean_comment(self):
        comment = self.cleaned_data.get('comment')
        
        if len(comment) < 10:
            raise forms.ValidationError('Le commentaire doit avoir au moins 10 caractères.')
        
        if 'spam' in comment.lower():
            raise forms.ValidationError('Votre commentaire contient du spam!')
        
        return comment
    
    # Valider plusieurs champs
    def clean(self):
        cleaned_data = super().clean()
        rating = cleaned_data.get('rating')
        comment = cleaned_data.get('comment')
        
        # Si rating est 5, le commentaire doit être positif
        if rating == 5 and not comment:
            raise forms.ValidationError('Veuillez expliquer votre note 5 étoiles!')
        
        return cleaned_data
```

---

## 7️⃣ Flux Complet: Données REQUEST → FORM → VIEW → RESPONSE

```python
# Exemple: Utilisateur ajoute une review

# 1. GET /books/1/review/add/
# Django crée un form vide et l'affiche

def add_review(request, book_id):
    book = get_object_or_404(Book, id=book_id)
    
    if request.method == 'GET':
        # 1. Créer un form vide
        form = ReviewForm()
        return render(request, 'add_review.html', {'form': form})


# 2. Utilisateur remplit et soumet le formulaire
# POST /books/1/review/add/
# REQUEST.POST = {'rating': '5', 'comment': 'Excellent livre!'}

    if request.method == 'POST':
        # 2. Créer un form avec les données POST
        form = ReviewForm(request.POST)
        # form.data = {'rating': '5', 'comment': 'Excellent livre!'}
        
        # 3. Valider (automatiquement!)
        if form.is_valid():
            # form.cleaned_data = {'rating': 5, 'comment': 'Excellent livre!'}
            # (Notez: rating est maintenant un int, pas une string!)
            
            # 4. Sauvegarder
            review = form.save(commit=False)
            review.book = book
            review.user = request.user
            review.save()
            
            # 5. Rediriger (RESPONSE)
            return redirect('book_detail', book_id=book.id)
        
        # Si invalide: réafficher le form avec les erreurs
        # RESPONSE = render avec form.errors affichées
        return render(request, 'add_review.html', {
            'form': form,
            'book': book,
        })
```

---

## 8️⃣ Pièges Courants

### ❌ Piège 1: Oublier `csrf_token` dans le template
```html
<!-- MAUVAIS: form ne fonctionne pas en POST! -->
<form method="post">
    {{ form }}
</form>

<!-- BON: inclure le token CSRF -->
<form method="post">
    {% csrf_token %}
    {{ form }}
</form>
```

### ❌ Piège 2: Utiliser `request.POST` directement au lieu de form
```python
# MAUVAIS: pas de validation!
title = request.POST.get('title')

# BON: utiliser la form validée
form = BookForm(request.POST)
if form.is_valid():
    title = form.cleaned_data['title']
```

### ❌ Piège 3: Sauvegarder sans valider
```python
# MAUVAIS
form = BookForm(request.POST)
form.save()  # Crash si invalid!

# BON
form = BookForm(request.POST)
if form.is_valid():
    form.save()
```

---

## 9️⃣ Exercice 3: Créer Un Formulaire d'Ajout d'Auteur

### Ton Défi:
1. Crée une `AuthorForm` dans `forms.py`
2. Crée une view `add_author()` qui affiche et traite le formulaire
3. Ajoute une validation personnalisée: l'email doit être valide
4. Affiche les erreurs dans le template

### Hints:
```python
# forms.py
class AuthorForm(forms.ModelForm):
    class Meta:
        # À faire
        pass

# views.py
def add_author(request):
    # À faire
    pass
```

**Solution plus bas!**

---

## ✅ Points Clés À Retenir

- ✅ Forms = validation automatique + sécurité
- ✅ `is_valid()` = valide et rend les données nettoyées
- ✅ `cleaned_data` = dictionnaire avec les données validées
- ✅ `commit=False` = créer sans sauvegarder
- ✅ `form.errors` = dictionnaire avec les erreurs
- ✅ Toujours inclure `{% csrf_token %}` dans le template!

---

## 🎁 SOLUTION EXERCICE 3

### forms.py
```python
class AuthorForm(forms.ModelForm):
    class Meta:
        model = Author
        fields = ['name', 'email']
        labels = {
            'name': 'Nom de l\'auteur',
            'email': 'Email',
        }
    
    # Validation personnalisée pour l'email
    def clean_email(self):
        email = self.cleaned_data.get('email')
        
        # Vérifier que l'email n'existe pas déjà
        if Author.objects.filter(email=email).exists():
            raise forms.ValidationError('Cet email existe déjà!')
        
        return email
```

### views.py
```python
def add_author(request):
    if request.method == 'POST':
        form = AuthorForm(request.POST)
        
        if form.is_valid():
            form.save()
            return redirect('author_list')
        # else: réafficher le form avec les erreurs
    else:
        form = AuthorForm()
    
    return render(request, 'add_author.html', {'form': form})
```

### Template (add_author.html)
```html
<form method="post">
    {% csrf_token %}
    
    {{ form }}
    
    {% if form.non_field_errors %}
        <div class="alert alert-danger">
            {{ form.non_field_errors }}
        </div>
    {% endif %}
    
    <button type="submit">Ajouter</button>
</form>
```

**Prêt pour Module 4? On va apprendre à AFFICHER les données dans les Templates! 🎨**

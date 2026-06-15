# 🚀 GUIDE DE DÉMARRAGE - Comment Utiliser Cette Masterclass

## 📋 Avant de Commencer

Assure-toi d'avoir:
- ✅ Python 3.8+ installé
- ✅ pip (gestionnaire de paquets Python)
- ✅ Un terminal/console de commande
- ✅ Un éditeur de code (VS Code, PyCharm, etc.)

---

## 🎯 Les 7 Modules Expliqués

### **Module 1: Les Fondamentaux** 📖
**Fichier:** `01_Module_Fondamentaux.md`

**Ce que tu vas apprendre:**
- Comment Django fonctionne (REQUEST → VIEW → RESPONSE)
- Le cycle complet d'une requête
- Les structures de données (listes, dicts, tuples)

**Durée:** 30 minutes
**Exercice:** Créer ta première view

---

### **Module 2: Les Views** 🎯
**Fichier:** `02_Module_Views.md`

**Ce que tu vas apprendre:**
- Comment une view reçoit les données
- `request.GET` et `request.POST`
- Comment récupérer depuis la base de données
- Manipuler les données en Python

**Durée:** 45 minutes
**Exercice:** Créer des views pour lister et filtrer

---

### **Module 3: Les Forms** 📝
**Fichier:** `03_Module_Forms.md`

**Ce que tu vas apprendre:**
- Créer des formulaires Django
- Valider les données automatiquement
- Gestion des erreurs
- Circulation des données: REQUEST → FORM → VIEW

**Durée:** 45 minutes
**Exercice:** Créer un formulaire complet avec validation

---

### **Module 4: Les Templates** 🎨
**Fichier:** `04_Module_Templates.md`

**Ce que tu vas apprendre:**
- Tags Django: `{{ }}`, `{% %}`
- Boucles et conditions dans les templates
- Afficher les données
- Héritage de templates

**Durée:** 45 minutes
**Exercice:** Créer un template complet

---

### **Module 5: Django-HTMX** ⚡
**Fichier:** `05_Module_HTMX.md`

**Ce que tu vas apprendre:**
- HTMX sans JavaScript
- Mise à jour partielle de pages
- `hx-get`, `hx-post`, `hx-target`
- Chercher en temps réel
- Likes/reactions dynamiques

**Durée:** 1 heure
**Exercice:** Créer une recherche en temps réel

---

### **Module 6: Structures de Données** 📊
**Fichier:** `06_Module_DataStructures.md`

**Ce que tu vas apprendre:**
- Listes, dictionnaires, tuples dans Django
- JSON (envoi/réception)
- CSV (import/export)
- Circulation des données complexes

**Durée:** 1 heure
**Exercice:** Exporter et importer un CSV

---

### **Module 7: Projet Complet** 🎓
**Fichier:** `07_Module_ProjetComplet.md`

**Ce que tu vas apprendre:**
- Projet complet: Bibliothèque en ligne
- Tous les modules combinés
- Code commenté ligne par ligne
- Models, Views, Forms, Templates, HTMX

**Durée:** 3-4 heures
**Projet:** Construire la bibliothèque complète

---

## 💻 Installation du Projet

### Étape 1: Créer l'environnement virtuel

```bash
# Naviguer dans ton dossier de projet
cd mon_projet_django

# Créer l'environnement virtuel
python -m venv venv

# Activer l'environnement
# Sur Linux/Mac:
source venv/bin/activate

# Sur Windows:
venv\Scripts\activate
```

### Étape 2: Installer les dépendances

```bash
# Installer Django et django-htmx
pip install django==4.2 django-htmx

# Vérifier l'installation
python -m django --version
```

### Étape 3: Créer le projet

```bash
# Créer le projet Django
django-admin startproject config .

# Créer l'app library
python manage.py startapp library
```

### Étape 4: Configuration de base

**config/settings.py:**
```python
# Ajouter 'library' et 'django_htmx' dans INSTALLED_APPS
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Ajouter ces deux:
    'django_htmx',
    'library',
]
```

### Étape 5: Les migrations

```bash
# Créer les migrations Django initiales
python manage.py migrate

# Créer un superuser (admin)
python manage.py createsuperuser
# Suis les instructions pour créer ton compte

# Lancer le serveur
python manage.py runserver
```

Visite: **http://localhost:8000/admin/**

---

## 📚 Comment Utiliser Cette Masterclass

### Option 1: Apprendre Module par Module

```
1. Lis le Module 1 entièrement
2. Comprends les concepts
3. Fais l'exercice
4. Passe au Module 2
5. Répète jusqu'au Module 7
```

**Durée totale:** 8-10 heures

---

### Option 2: Apprendre en Construisant

```
1. Lis le Module 7 (Projet Complet)
2. Copie le code commenté
3. Comprends chaque section
4. Modifie-le pour ton besoin
5. Reviens aux modules spécifiques si besoin
```

**Durée totale:** 5-6 heures

---

### Option 3: Référence Rapide

```
Besoin de: _______________ → Va au Module ___
- Comprendre le cycle           → 1
- Récupérer des données         → 2
- Valider un formulaire         → 3
- Afficher dans le template     → 4
- Mettre à jour sans recharger  → 5
- Gérer JSON/CSV                → 6
- Projet complet                → 7
```

---

## 🔄 Flux de Travail Typique pour Chaque Fonctionnalité

### Exemple: Ajouter une Review

#### Étape 1: Crée le Model (Module 7)
```python
class Review(models.Model):
    book = models.ForeignKey(Book, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    rating = models.IntegerField(choices=RATING_CHOICES)
    comment = models.TextField()
```

#### Étape 2: Crée la Form (Module 3)
```python
class ReviewForm(forms.ModelForm):
    class Meta:
        model = Review
        fields = ['rating', 'comment']
```

#### Étape 3: Crée la View (Module 2)
```python
def add_review(request, book_id):
    if request.method == 'POST':
        form = ReviewForm(request.POST)
        if form.is_valid():
            review = form.save(commit=False)
            review.book_id = book_id
            review.user = request.user
            review.save()
            return redirect('book_detail', book_id=book_id)
    else:
        form = ReviewForm()
    return render(request, 'add_review.html', {'form': form})
```

#### Étape 4: Crée le Template (Module 4)
```html
<form method="post">
    {% csrf_token %}
    {{ form }}
    <button type="submit">Envoyer Review</button>
</form>
```

#### Étape 5: Ajoute la Route (Module 7)
```python
path('book/<int:book_id>/review/', views.add_review, name='add_review'),
```

---

## 🎓 Les 5 Pièges à Éviter

### ❌ Piège 1: Oublier le context dans render()
```python
# MAUVAIS
return render(request, 'template.html')  # Les données ne vont pas!

# BON
return render(request, 'template.html', {'data': data})
```

### ❌ Piège 2: Oublier {% csrf_token %} dans les formulaires
```html
<!-- MAUVAIS -->
<form method="post">
    <input name="title">
</form>

<!-- BON -->
<form method="post">
    {% csrf_token %}
    <input name="title">
</form>
```

### ❌ Piège 3: Ne pas valider les données
```python
# MAUVAIS
form = BookForm(request.POST)
form.save()  # Crash si invalide!

# BON
form = BookForm(request.POST)
if form.is_valid():
    form.save()
```

### ❌ Piège 4: Retourner la page entière avec HTMX
```python
# MAUVAIS avec HTMX
return render(request, 'library/books_list.html', {...})  # Retourne la page entière!

# BON avec HTMX
return render(request, 'library/book_item.html', {'book': book})  # Juste le morceau!
```

### ❌ Piège 5: Oublier les migrations
```bash
# Après modifier models.py:
python manage.py makemigrations  # Créer les migrations
python manage.py migrate         # Appliquer les migrations
```

---

## 📖 Glossaire Rapide

| Terme | Signification |
|-------|---------------|
| **View** | Fonction qui traite une requête |
| **Model** | Structure d'une table en BD |
| **Form** | Formulaire qui valide les données |
| **Template** | Fichier HTML avec tags Django |
| **Context** | Dictionnaire de données pour le template |
| **QuerySet** | Liste de résultats depuis la BD |
| **ForeignKey** | Relation entre deux models |
| **HTMX** | Biblothèque pour mis à jour partielle |
| **CSRF Token** | Protection contre les attaques |
| **Middleware** | Code qui s'exécute avant/après les views |

---

## 🚀 Commandes Django Essentielles

```bash
# Créer un projet
django-admin startproject config .

# Créer une app
python manage.py startapp library

# Créer les migrations
python manage.py makemigrations

# Appliquer les migrations
python manage.py migrate

# Créer un superuser
python manage.py createsuperuser

# Lancer le serveur
python manage.py runserver

# Shell Django (tester du code)
python manage.py shell

# Afficher les queries SQL
python manage.py sqlmigrate library 0001
```

---

## 💡 Tips & Tricks

### Déboguer une Vue
```python
def my_view(request):
    books = Book.objects.all()
    
    # Afficher dans la console
    print(f"Books: {books}")
    print(f"Count: {books.count()}")
    
    return render(request, 'template.html', {'books': books})
```

### Déboguer un Template
```html
<!-- Afficher une variable -->
{{ book }}

<!-- Afficher le type -->
{{ book|default:"N/A" }}

<!-- Afficher tout le contexte -->
{{ debug }}
```

### Utiliser Django Shell
```bash
python manage.py shell

>>> from library.models import Book
>>> books = Book.objects.all()
>>> print(books)
>>> book = books.first()
>>> print(book.title)
>>> exit()
```

---

## 📞 Besoin d'Aide?

### Erreurs Courantes

**1. "No such table: library_book"**
```bash
# Solution: faire les migrations
python manage.py makemigrations
python manage.py migrate
```

**2. "csrf_token" not found**
```django
# Solution: ajouter dans le template
{% csrf_token %}
```

**3. "reverse() not found"**
```python
# Solution: vérifier que le name existe dans urls.py
path('books/', views.books_list, name='books_list'),
```

---

## 🎯 Ordre d'Apprentissage Recommandé

```
Jour 1:
├─ Module 1: Fondamentaux (30 min)
├─ Module 2: Views (45 min)
└─ Exercices (30 min)

Jour 2:
├─ Module 3: Forms (45 min)
├─ Module 4: Templates (45 min)
└─ Mini-projet (1 heure)

Jour 3:
├─ Module 5: HTMX (1 heure)
├─ Module 6: Data Structures (1 heure)
└─ Exercices (1 heure)

Jour 4:
└─ Module 7: Projet Complet (4 heures)

TOTAL: ~12 heures pour maîtriser!
```

---

## ✅ Checklist de Maîtrise Finale

- ⬜ Comprendre le cycle REQUEST/RESPONSE
- ⬜ Créer un Model et faire les migrations
- ⬜ Créer une View et la tester
- ⬜ Créer une Form avec validation
- ⬜ Afficher les données dans un Template
- ⬜ Utiliser HTMX pour l'interactivité
- ⬜ Gérer les listes et dictionnaires
- ⬜ Exporter/Importer CSV
- ⬜ Construire un projet complet
- ⬜ Déboguer une application

**Quand tu as coché TOUT: Tu es prêt pour production! 🚀**

---

## 🎓 Prochaines Étapes

Après cette masterclass, tu peux:
1. ✅ Construire tes propres projets Django
2. ✅ Embaucher dans des boîtes Django
3. ✅ Apprendre le REST (Django REST Framework)
4. ✅ Apprendre les tests unitaires
5. ✅ Déployer sur un serveur (Heroku, DigitalOcean, etc.)

---

## 📝 Notes Personnelles

Crée un fichier `NOTES.md` et ajoute tes observations:

```markdown
# Mes Notes Django-HTMX

## Ce que j'ai appris
- ...

## Pièges rencontrés
- ...

## Ideas pour des projets
- ...

## Questions à explorer
- ...
```

---

**Bon apprentissage! Tu vas devenir un expert Django! 🚀✨**

**Questions? Reviens relire les modules, la réponse y est! 💪**

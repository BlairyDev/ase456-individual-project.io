+++
menus = 'main'
title = 'Documentation'
+++

Github Link: https://github.com/BlairyDev/manga-reader-app

# 📚 Manga Reader App — Architecture & Feature Documentation

This document describes the high-level architecture, feature behavior, data flow, and API usage for the **Manga Reader App**, built with **Flutter** and powered by the **MangaDex API**.  
It serves as a developer guide and reference for future maintenance and extension.

---

# 🏛️ 1. System Overview

The Manga Reader App is a Flutter-based mobile application that allows users to:

- Browse recent manga/manhwa/manhua series  
- View detailed information about a selected series  
- Read chapters (vertical or horizontal mode)  
- Search for titles  
- Save series locally in a library (SQLite)  
- Manage app settings, theme, backup, and data export/import

The backend data is retrieved via the **MangaDex REST API**.  
Local persistence uses **SQLite** (via `sqflite` package).

---

# 🔧 2. Architecture

## 2.1 App Layers

Android/iOS ---> 
            <---     Mangadex API

MVVM pattern

## 🗂️ Project Structure (Architecture)

This project follows **MVVM** (Model–View–ViewModel) with clean folder organization.

```
lib/
 ├── data/
 │    ├── models/         # Manga.dart and etc.
 |    ├──repositories/
 │    └── services/       # API services
 ├── view/
 │    ├── home_screen.dart
 │    ├── detail_screen.dart
 │    ├── library_screen.dart
 │    └── search_screen.dart
 │    └── settings_screen.dart
 │    └── chapter_screen.dart
 ├── view_models/         # HomeViewModel, DetailViewModel, etc.
 ├── di/           # GetIt Dependency Injection
 └── main.dart
```


## Testing
- Unit Testing 35/35
- for the source code please visit this link: https://github.com/BlairyDev/manga-reader-app
```bash
flutter test


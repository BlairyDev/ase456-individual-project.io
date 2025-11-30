+++
date = '2025-11-29T19:28:15-05:00'
draft = true
title = 'Api Docs'
+++

# Manga Reader API & Database Documentation

## Table of Contents
1. [Database Schema](#database-schema)
2. [Database Service API](#database-service-api)
3. [MangaDex API Service](#mangadex-api-service)
4. [Data Models](#data-models)

---

## Database Schema

### manga Table

Stores user's manga library collection.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INTEGER | PRIMARY KEY AUTOINCREMENT | Auto-incrementing unique identifier |
| mangaId | TEXT | - | MangaDex manga identifier |
| title | TEXT | - | Manga title |
| description | TEXT | - | Manga description/synopsis |
| authors | TEXT | - | JSON-encoded list of author names |
| artists | TEXT | - | JSON-encoded list of artist names |
| tags | TEXT | - | JSON-encoded list of tags |
| coverArtUrl | TEXT | - | URL to cover art image |
| status | TEXT | - | Publication status (ongoing, completed, etc.) |

**Note**: `authors`, `artists`, and `tags` columns store JSON-encoded string arrays.

---

## Database Service API

### Singleton Instance
```dart
DatabaseService.instance
```
Access the singleton instance of the database service.

### Methods

#### `Future<Database> get db`
Returns the database instance, initializing it if necessary.

**Returns**: `Future<Database>`

---

#### `Future<Database> initDb()`
Initializes the SQLite database at the standard database path.

**Returns**: `Future<Database>` - Initialized database instance

**Database Location**: `{databasesPath}/mangareader.db`

---

#### `Future<int> insertManga(Manga manga)`
Inserts a new manga entry into the library.

**Parameters**:
- `manga` (Manga): Manga object to insert

**Returns**: `Future<int>` - Row ID of inserted manga

**Example**:
```dart
final manga = Manga(
  mangaId: 'abc123',
  title: 'Example Manga',
  description: 'A great story',
  authors: ['Author Name'],
  artists: ['Artist Name'],
  tags: ['Action', 'Adventure'],
  coverArtUrl: 'https://example.com/cover.jpg',
  status: 'ongoing'
);

int rowId = await DatabaseService.instance.insertManga(manga);
```

---

#### `Future<List<Map<String, dynamic>>> queryAllMangas()`
Retrieves all manga entries from the library.

**Returns**: `Future<List<Map<String, dynamic>>>` - List of manga records

**Example**:
```dart
List<Map<String, dynamic>> library = await DatabaseService.instance.queryAllMangas();
```

---

#### `Future<bool> checkInLibrary(String mangaId)`
Checks if a manga exists in the user's library.

**Parameters**:
- `mangaId` (String): MangaDex manga identifier

**Returns**: `Future<bool>` - `true` if manga exists, `false` otherwise

**Example**:
```dart
bool exists = await DatabaseService.instance.checkInLibrary('abc123');
```

---

#### `Future<int> removeManga(String mangaId)`
Removes a manga from the library by its manga ID.

**Parameters**:
- `mangaId` (String): MangaDex manga identifier

**Returns**: `Future<int>` - Number of rows deleted (0 or 1)

**Example**:
```dart
int deleted = await DatabaseService.instance.removeManga('abc123');
```

---

#### `Future<bool> exportDatabase()`
Exports the entire manga library to a JSON file in the downloads directory.

**Returns**: `Future<bool>` - `true` if export successful, `false` otherwise

**Output Location**: `{downloadsPath}/manga_export.json` (or `manga_export(n).json` if file exists)

**File Format**: JSON array of manga objects

**Example**:
```dart
bool success = await DatabaseService.instance.exportDatabase();
if (success) {
  print('Library exported successfully');
}
```

---

#### `Future<String> getDatabasePath(String databaseName)`
Returns the full path for a database file.

**Parameters**:
- `databaseName` (String): Name of the database file

**Returns**: `Future<String>` - Full path to database file

---

#### `Future<bool> importDatabase()`
Imports manga library from a JSON file selected by the user.

**Returns**: `Future<bool>` - `true` if import successful, `false` otherwise

**File Format**: JSON array with manga objects containing:
- `mangaId`, `title`, `description`, `coverArtUrl`, `status`
- `authors`, `artists`, `tags` (JSON-encoded string arrays)

**Behavior**: Only imports manga that don't already exist in the library (checks via `checkInLibrary`)

**Example**:
```dart
bool success = await DatabaseService.instance.importDatabase();
if (success) {
  print('Library imported successfully');
}
```

---

## MangaDex API Service

Base URL: `https://api.mangadex.org`

### Methods

#### `Future<Map<String, dynamic>> runAPI(String API)`
Generic API request handler.

**Parameters**:
- `API` (String): Full API endpoint URL

**Returns**: `Future<Map<String, dynamic>>` - Parsed JSON response

**Throws**: `Exception` if request fails (status code != 200)

---

#### `Future<Map<String, dynamic>> fetchMangaSeries(int offset)`
Fetches a paginated list of manga series ordered by latest uploaded chapter.

**Parameters**:
- `offset` (int): Pagination offset (increments of 6)

**Returns**: `Future<Map<String, dynamic>>` - MangaDex API response

**API Endpoint**: `GET /manga`

**Query Parameters**:
- `limit`: 6
- `offset`: {offset}
- `includedTagsMode`: AND
- `excludedTagsMode`: OR
- `contentRating[]`: safe
- `order[latestUploadedChapter]`: desc
- `includes[]`: artist, author, cover_art

**Example**:
```dart
var response = await MangadexApiService().fetchMangaSeries(0);
// Fetches first 6 manga series
```

---

#### `Future<Map<String, dynamic>> fetchMangaChapters(String mangaId)`
Fetches aggregated chapter data for a specific manga.

**Parameters**:
- `mangaId` (String): MangaDex manga identifier

**Returns**: `Future<Map<String, dynamic>>` - Chapter aggregation data

**API Endpoint**: `GET /manga/{id}/aggregate`

**Query Parameters**:
- `translatedLanguage[]`: en

**Example**:
```dart
var chapters = await MangadexApiService().fetchMangaChapters('abc123');
```

---

#### `Future<Map<String, dynamic>> fetchChapterImageList(String chapterId)`
Fetches the image URLs for a specific chapter.

**Parameters**:
- `chapterId` (String): MangaDex chapter identifier

**Returns**: `Future<Map<String, dynamic>>` - Chapter image server data

**API Endpoint**: `GET /at-home/server/{id}`

**Example**:
```dart
var imageData = await MangadexApiService().fetchChapterImageList('chapter123');
```

---

#### `Future<Map<String, dynamic>> fetchSearchManga(String title, int offset)`
Searches for manga by title with pagination.

**Parameters**:
- `title` (String): Search query (automatically URI-encoded)
- `offset` (int): Pagination offset (increments of 6)

**Returns**: `Future<Map<String, dynamic>>` - Search results

**API Endpoint**: `GET /manga`

**Query Parameters**:
- `limit`: 6
- `offset`: {offset}
- `title`: {encoded title}
- `contentRating[]`: safe, suggestive, erotica
- `order[latestUploadedChapter]`: desc
- `includes[]`: artist, author, cover_art

**Example**:
```dart
var results = await MangadexApiService().fetchSearchManga('One Piece', 0);
```

---

## Data Models

### Manga Class

```dart
class Manga {
  final String mangaId;
  final String title;
  final String description;
  final List<String> authors;
  final List<String> artists;
  final List<String> tags;
  final String coverArtUrl;
  final String status;
  
  Map<String, dynamic> toMap() {
    return {
      'mangaId': mangaId,
      'title': title,
      'description': description,
      'authors': jsonEncode(authors),  // Stored as JSON string
      'artists': jsonEncode(artists),  // Stored as JSON string
      'tags': jsonEncode(tags),        // Stored as JSON string
      'coverArtUrl': coverArtUrl,
      'status': status,
    };
  }
}
```

---

## Error Handling

### Database Operations
All database operations return `false` or throw exceptions on failure. Wrap calls in try-catch blocks:

```dart
try {
  bool success = await DatabaseService.instance.exportDatabase();
} catch (e) {
  print('Error: $e');
}
```

### API Operations
API calls throw `Exception` on HTTP failure:

```dart
try {
  var data = await MangadexApiService().fetchMangaSeries(0);
} catch (e) {
  print('API Error: $e');
}
```

---

## Usage Examples

### Complete Workflow Example

```dart
// Initialize API service
final apiService = MangadexApiService();
final dbService = DatabaseService.instance;

// Fetch manga series
var response = await apiService.fetchMangaSeries(0);

// Check if manga is in library
bool inLibrary = await dbService.checkInLibrary('manga123');

// Add to library if not present
if (!inLibrary) {
  final manga = Manga(/* ... */);
  await dbService.insertManga(manga);
}

// Get all library manga
var library = await dbService.queryAllMangas();

// Export library
await dbService.exportDatabase();

// Import library
await dbService.importDatabase();
```
# Almeida Offline — Downloadable Database Assets

Official repository hosting standalone, modular database packs and companion Bibles for the **Almeida Offline** Bible study application.

## Overview

To keep the initial installation size of **Almeida Offline** lightweight (~30–40 MB total download), modular per-language databases are distributed as compressed `.db.gz` packages hosted on GitHub Releases CDN.

Upon first launch (or when downloading additional study languages), the mobile app queries `manifest.json`, downloads the corresponding `.db.gz` file, and decompresses it locally directly into the app sandbox.

---

## Available Language Packs (Release v1.0.0)

| Language | Tools Package | Tools Size | Tools SHA-256 | Companion Bible | Bible Size | Bible SHA-256 | Total Download |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **Português (PT)** | `biblical_tools_pt.db.gz` | 27.49 MB | `c17e61f9bd4e124f...` | `bible_pt.db.gz` *(JFA)* | 4.85 MB | `df8b8dbb3e8111ce...` | **32.34 MB** |
| **English (EN)** | `biblical_tools_en.db.gz` | 26.89 MB | `68f794d0610b8832...` | `bible_en.db.gz` *(KJV)* | 5.19 MB | `7321d25b21cc4852...` | **32.08 MB** |
| **Español (ES)** | `biblical_tools_es.db.gz` | 27.46 MB | `f261702ef81941dd...` | `bible_es.db.gz` *(RV 1909)* | 3.22 MB | `b574ae0c318aa421...` | **30.68 MB** |
| **Deutsch (DE)** | `biblical_tools_de.db.gz` | 27.80 MB | `765398018bded153...` | `bible_de.db.gz` *(Luther 1912)* | 3.25 MB | `7050cc3c5360e586...` | **31.05 MB** |
| **Italiano (IT)** | `biblical_tools_it.db.gz` | 27.46 MB | `f79c2ade145678a0...` | `bible_it.db.gz` *(Diodati 1649)* | 3.48 MB | `4eebe74d83d79a9b...` | **30.95 MB** |
| **日本語 (JA)** | `biblical_tools_ja.db.gz` | 28.14 MB | `ed93250dd9a636e5...` | `bible_ja.db.gz` *(口語訳 1954/55)* | 9.28 MB | `167ad847a09ae3a7...` | **37.42 MB** |

Full SHA-256 checksums and automated parsing specifications are available in [`manifest.json`](manifest.json).

---

## Contents of `biblical_tools_{lang}.db`

Each standalone language pack is a standalone SQLite database containing:

1. **Strong's Hebrew Lexicon (8,674 entries)** — Original Hebrew lemmas, transliteration, pronunciation, and definitions localized in the selected language.
2. **Strong's Greek Lexicon (5,521 entries)** — Original Greek lemmas, transliteration, pronunciation, and definitions localized in the selected language.
3. **Easton's Bible Dictionary (3,961 entries)** — Complete biblical encyclopedic articles localized in the selected language.
4. **Smith's Bible Dictionary (4,310 entries)** — Classical biblical scholarship dictionary localized in the selected language.
5. **Cross-References (344,799 links)** — Complete Treasury of Scripture Knowledge (TSK) cross-reference network.
6. **Semantic Categories (107 categories)** — 18 major biblical domains localized in the selected language.
7. **Thematic Topics (6,713 topics)** — Multilingual topical index.
8. **Full-Text Search (FTS5)** — Pre-indexed virtual tables (`lexicon_fts`, `dictionary_fts`, `topics_fts`) optimized for instant prefix and sub-millisecond full-text queries.

---

## Integration in Flutter

```dart
import 'dart:io';
import 'package:archive/archive.dart';
import 'package:http/http.dart' as http;
import 'package:path/path.dart' as p;
import 'package:sqflite/sqflite.dart';

Future<void> downloadAndInstallLanguagePack({
  required String downloadUrl,
  required String targetFileName,
  required Function(double progress) onProgress,
}) async {
  final databasesPath = await getDatabasesPath();
  final targetPath = p.join(databasesPath, targetFileName);
  
  final client = http.Client();
  final request = http.Request('GET', Uri.parse(downloadUrl));
  final response = await client.send(request);
  
  final totalBytes = response.contentLength ?? 0;
  var receivedBytes = 0;
  final gzBytes = <int>[];

  await for (final chunk in response.stream) {
    gzBytes.addAll(chunk);
    receivedBytes += chunk.length;
    if (totalBytes > 0) {
      onProgress(receivedBytes / totalBytes);
    }
  }

  // Decompress gzip stream directly into the database directory
  final decompressedBytes = GZipDecoder().decodeBytes(gzBytes);
  await File(targetPath).writeAsBytes(decompressedBytes, flush: true);
}
```

---

## Attribution & Licenses

All underlying texts, lexicons, and source materials are strictly in the Public Domain or open licensed:
- **Strong's Exhaustive Concordance**: Public Domain (James Strong, 1890).
- **Easton's Bible Dictionary**: Public Domain (M.G. Easton, 1897).
- **Smith's Bible Dictionary**: Public Domain (William Smith, 1884).
- **Treasury of Scripture Knowledge (TSK)**: Public Domain (Jerome H. Smith, 1836).
- **Biblical Texts**:
  - Portuguese: João Ferreira de Almeida (Public Domain).
  - English: King James Version (Public Domain).
  - Spanish: Reina-Valera 1909 (Public Domain).
  - German: Luther Bibel 1912 (Public Domain).
  - Italian: Giovanni Diodati 1649 (Public Domain).
  - Japanese: 口語訳聖書 Kougo-yaku 1954/1955 (Public Domain in Japan under 50-year post-mortem term).

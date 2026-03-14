# English Practice METU

## Proje

METU ogrencileri icin Ingilizce kelime ve gramer pratik uygulamasi. Tek sayfalik (SPA), vanilla JS, framework yok.

- **GitHub:** https://github.com/T-Necat/english-practice-metu
- **Branch:** master
- **Deploy:** Statik hosting (GitHub Pages veya benzeri)

## Teknik Yapi

- **Tek dosya:** `index.html` — tum HTML, CSS ve JS icinde embedded
- **Veri:** `words.json` / `words_data.js` — 536 kelime, 9 konu
- **Harici bagimlilik:** Google Fonts (Inter), Puter.com cloud storage API
- **Framework:** Yok — vanilla JS, vanilla CSS

## CSS Mimarisi

- **Tema sistemi:** 5 tema (default, cool, dark, midnight, forest) — `[data-theme]` attribute + CSS variables
- **Responsive:** `clamp()` ile fluid sizing, 2 breakpoint:
  - `@media (max-width: 768px)` — tablet
  - `@media (max-width: 520px)` — mobil
- **Desktop max-width'ler:** `clamp(300px, 90%, CAP)` pattern'i — CAP degerleri elemana gore 900-1200px arasi
- **Mobil breakpoint'lere dokunma** — mobil UX zaten iyi durumda

## Sayfa Yapisi

| Sayfa | HTML ID | Aciklama |
|-------|---------|----------|
| Kelimeler | `#wordsPage` | Flashcard + filtreler (Hepsi, Favoriler, Quiz, Eslestir, Grammar) |
| Konular | `#contentPage` | Gramer/telaffuz konulari, unit bazli (Unit 1-3) |
| Notlar | `#notesPage` | Kullanici notlari, Puter cloud sync, filtreler (Tumu, Genel, Konulara Bagli) |

## Commit Kurallari

- **Co-Authored-By satiri EKLEME** — commit'ler tamamen kullanici adina
- Conventional commits (fix:, feat:, refactor:, chore:)
- Commit mesajlari Ingilizce

## Onemli Notlar

- Quiz, Eslestir ve Grammar butonlari basit filtre degil — overlay/mod acan butonlar, ozel davranislari var
- Sidebar desktop'ta 320px sabit, mobilde drawer (slide-in)
- Kart flip animasyonu 3D perspective ile calisiyor (backface-visibility, rotateY)
- Touch target minimum 44-48px (mobilde)

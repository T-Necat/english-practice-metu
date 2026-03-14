# Desktop Layout & Island Navigation Design

**Date:** 2026-03-14
**Status:** Approved

## Problem

Desktop'ta (MacBook Air 14" ~ 4K 27") icerik 680px'de kilitli ve ortada cok bos alan kaliyor. Top-nav + score-bar + filter-row ayri ayri satirlar olarak dikeyde gereksiz yer kapliyor.

## Design Decisions

### 1. Fluid Desktop Layout (Completed)

Desktop max-width degerleri fluid/oransal sisteme gecti:

| Element | Eski | Yeni |
|---------|------|------|
| card-wrapper, filter-row, answer-section, nav-row, top-nav, header, progress-bar | clamp(300px, 80vw, 680px) veya 680px | clamp(300px, 90%, 1100px) |
| content-page | 760px | clamp(300px, 90%, 1000px) |
| notes-page | 760px | clamp(300px, 90%, 1200px) |
| quiz-container | 600px | clamp(300px, 90%, 900px) |

Mobil breakpoint'ler (<=768px ve <=520px) dokunulmadi.

### 2. Pill Island Navigation

Mevcut top-nav (3 ayri buton) ve filter-row (5 ayri buton) tek bir "pill island" icinde birlesecek.

**Kapali hali:**
- Yuvarlak (pill) seklinde tek satir ada
- 3 tab gorunur: Kelimeler | Konular | Notlar
- Aktif tab koyu arka planli, diger tablar soluk
- Kompakt, minimum yer kaplar

**Acik hali (aktif tab'a tiklaninca):**
- Ada asagiya dogru buyur (slide-down animasyon)
- Tab satirinin altinda o sayfaya ait alt-filtreler gorunur
- Tekrar aktif tab'a tiklaninca kapanir (toggle)
- Farkli tab'a tiklaninca sayfa degisir, ada acik kalir ve yeni tab'in alt-filtreleri gosterilir

**Her tab'in alt-filtreleri:**
- **Kelimeler:** Hepsi, Favoriler, Quiz, Eslestir, Grammar (mevcut `.filter-row` butonlari). Not: Quiz, Eslestir ve Grammar butonlari basit filtre degil, overlay/mod acan butonlar. Mevcut inline style'lari (accent renk, dark bg) korunacak ve island icinde de ayirt edici kalacak. Bu butonlarin mevcut event listener'lari ve davranislari degismeyecek.
- **Konular:** Unit 1, Unit 2, Unit 3 (mevcut `.content-nav-btn` butonlari)
- **Notlar:** Tumu, Genel, Konulara Bagli (mevcut `.notes-filter-btn` butonlari)

**CSS/HTML degisiklikleri:**
- `.top-nav`, `.filter-row`, `.content-nav`, `.notes-filter-row` yerine tek `.nav-island` container
- `.nav-island` icinde `.nav-island-tabs` (ust satir) ve 3 ayri `.nav-island-filters` grubu (her sayfa icin birer tane)
- Aktif sayfanin filter grubu gorunur, digerleri gizli
- `.nav-island-filters` animasyonu CSS grid trick ile: `grid-template-rows: 0fr` (kapali) → `1fr` (acik), `transition: grid-template-rows 0.3s ease`. Icerik wrapper'i `overflow: hidden; min-height: 0` olacak. Bu yontem `max-height` hack'inden daha temiz ve icerik boyutuna bagli degil.
- Pill border-radius: 24px (kapali), 20px (acik), `transition: border-radius 0.3s` ile yumusak gecis

**Mobil davranisi:**
- Ayni island yapisi mobilde de calisir, mobil breakpoint'lerdeki boyut/padding ayarlari korunur
- Mevcut mobil touch target boyutlari (min 44-48px) korunur

### 3. Score Bar — Kart Icinde Ust Serit

Score bar (dogru/yanlis sayaci + ilerleme) kartin disinda ayri satir olmak yerine kartin icinde ust serit olarak entegre edilecek.

**Yapi:**
- Kart (`.card-wrapper > .card-container > .card-face.card-front`) icinde ust kisimda ince bir serit
- Sol: dogru (yesil) + yanlis (kirmizi) sayaci
- Sag: ilerleme (orn. 47 / 536)
- Hafif arka plan rengi ile karttan ayrilir (tema uyumlu)
- Ince alt border ile gorsel ayrim
- Score bar sadece Kelimeler sayfasinda (flashcard modunda) gorunur. Konular ve Notlar sayfalarinda score bar yok.

**CSS degisiklikleri:**
- card-front'un flex layout'u degisecek: `justify-content: center` yerine `flex-direction: column` + score strip uste, kelime icerigi kalan alanda ortalanacak (`flex: 1; display: flex; align-items: center; justify-content: center`)
- `.score-bar` mevcut CSS'i guncellenir: flex, space-between, padding icin
- Kart icinde konumlandirilir (card-front'un ilk cocugu)
- Arka plan: tema'nin `--tag-bg` veya `--fav-bg` degiskeni
- Border-bottom: 1px solid var(--card-border)

## Scope

**Yapilacak:**
- Pill island nav component (HTML + CSS + JS toggle logic)
- 3 sayfa icin alt-filtre gruplari (Kelimeler/Konular/Notlar)
- Mevcut `.filter-row`, `.content-nav`, `.notes-filter-row` kaldirilip island'a tasinacak
- Score bar'i kart icine tasima
- Tum 5 temada test

**Yapilmayacak:**
- Mobil breakpoint degisiklikleri
- Sidebar degisiklikleri
- Yeni sayfa/feature ekleme

## Technical Notes

- Tek dosya: `index.html` (embedded CSS + JS)
- Tema sistemi CSS variables ile calisiyor, yeni bilesenler ayni degiskenleri kullanmali
- JavaScript: vanilla JS, framework yok
- Mevcut event listener'lar (tab switch, filter click) yeni DOM yapisina adapte edilmeli

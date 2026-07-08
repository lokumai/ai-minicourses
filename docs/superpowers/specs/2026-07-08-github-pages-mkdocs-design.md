# GitHub Pages Yayını — MkDocs Material Tasarımı

**Tarih:** 2026-07-08
**Durum:** Onaylandı (brainstorming sonucu)

## Amaç

Repodaki mevcut markdown mini-kurslarını, kopya oluşturmadan, aynı repodan
GitHub Pages üzerinde otomatik yayınlamak. Markdown tek kaynak (single source
of truth) olarak kalır; her `main` push'unda site kendini günceller.

## Kapsam

- **Faz 1 (bu tasarım):** Sadece İngilizce içerik yayınlanır.
- **Faz 2 (ileride):** `_tr.md` dosyaları `mkdocs-static-i18n` eklentisiyle
  dil düğmesi olarak eklenir. Bu tasarımın kapsamı dışındadır.

## Mimari

```
push → GitHub Actions → mkdocs build --strict → GitHub Pages
```

- **Araç:** MkDocs + Material teması.
- **Config:** Repo kökünde `mkdocs.yml`, `docs_dir: mini-courses`.
- **Deploy:** `.github/workflows/deploy-docs.yml` — `main`'e push'ta çalışır,
  `mkdocs build --strict` sonrası `actions/upload-pages-artifact` +
  `actions/deploy-pages` ile yayınlar.
- **Adres:** `https://lokumai.github.io/ai-minicourses/`
- **Tek manuel adım:** Repo Settings → Pages → Source: "GitHub Actions".

## Bileşenler

### 1. `mkdocs.yml`

- `site_name: AI Mini-Courses`, `site_url` Pages adresi.
- `theme: material` — koyu/açık palet, arama (`search` eklentisi) varsayılan.
- `docs_dir: mini-courses`.
- `exclude_docs: "*_tr.md"` — TR dosyaları build dışı (repoda kalırlar).
  `scratchpad/` dizini de build dışı bırakılır.
- Açık `nav` tanımı: kategori sırası korunur; hazır olmayan kategoriler
  başlıkta `🚧 Draft` etiketi taşır (gizlenmez):
  - Fundamentals (tam)
  - 🚧 Intermediate (Draft)
  - 🚧 Expert (Draft)
  - 🚧 Ecosystem (Draft)
  - 🚧 Protocols & Specs (Draft)
  - 🚧 Optional (Draft)

### 2. Ana sayfa — `mini-courses/index.md`

- Yeni bir karşılama sayfası (kopya değil): kısa tanıtım + kategori tablosu
  ve "Start with Fundamentals" yönlendirmesi.
- Kök `README.md` GitHub vitrini olarak kalır; içine canlı site linki eklenir.

### 3. CI workflow — `.github/workflows/deploy-docs.yml`

- Tetikleyici: `push` → `main` (+ elle `workflow_dispatch`).
- Adımlar: checkout → Python kur → `pip install mkdocs-material` →
  `mkdocs build --strict` → Pages artifact upload → deploy.
- Pages izinleri: `pages: write`, `id-token: write`.

## Veri Akışı ve Linkler

- Görseller (`1_fundamentals/images/…`) göreli yol kullandığı için MkDocs'ta
  değişiklik gerektirmez.
- `--strict` mod: kırık iç link veya eksik görsel CI'ı kırar — canlı gelişen
  içerikte site sessizce bozulmaz.
- Kategori `README.md` dosyaları MkDocs tarafından dizin index'i olarak
  render edilir (`index.md` yoksa `README.md` kabul edilir).

## Hata Yönetimi

- Build hatası → Actions kırmızı, mevcut site bozulmadan yayında kalır.
- Kırık linkler `--strict` ile PR/push aşamasında yakalanır.

## Test / Doğrulama

- Lokal: `mkdocs serve` ile önizleme.
- CI: `mkdocs build --strict` başarılı olmalı.
- Yayın sonrası: ana sayfa, Fundamentals modüllerinden biri ve bir görselin
  yüklendiği elle doğrulanır.

## Kapsam Dışı (YAGNI)

- TR i18n (Faz 2), özel alan adı, analytics, versiyonlama, arama dışında
  ek eklentiler, özel tema/CSS.

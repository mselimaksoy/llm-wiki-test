# llm-wiki-test

Andrej Karpathy'nin [LLM Wiki](./llm-wiki.md) pattern'ini hands-on denemek için kurulmuş küçük bir oyun alanı. Konu: **Türk Kahvesi** (bilerek küçük ve tanıdık seçildi).

Pattern'in özeti: RAG gibi her soruda kaynakları yeniden taramak yerine, LLM **kalıcı, birikimli bir markdown wiki** inşa edip bakımını yapar. Sen kaynak ekler ve soru sorarsın; LLM özetler, çapraz referans verir, çelişkileri işaretler, sayfaları günceller.

## Repo yapısı

```
.
├── llm-wiki.md          # Karpathy'nin orijinal pattern açıklaması (referans)
├── CLAUDE.md            # Wiki'nin "schema"sı — LLM'e nasıl bakım yapacağını anlatır
├── raw/                 # Ham kaynaklar (immutable)
├── wiki/                # LLM'in ürettiği katman
│   ├── index.md             # İçerik kataloğu
│   ├── log.md               # Kronolojik olay kaydı
│   ├── sources/             # Her ham kaynağın özeti
│   ├── entities/            # Cezve, Fincan, Köpük, ...
│   ├── concepts/            # Demleme Yöntemi, Fal, Şeker Tercihleri, ...
│   └── synthesis/           # Query cevaplarından file edilmiş sayfalar
├── .claude/skills/      # Claude Code skill'leri (ör. /ingest)
├── LEARNINGS.md         # Pattern'i çalıştırırken öğrenilenler
└── HOW-IT-WORKS.md      # Retrieval nasıl çalışır, klasik RAG'den farkı
```

## Pattern'in dört operasyonu (hepsi denendi)

| Operasyon | Nerede görülür |
|---|---|
| **Ingest** | `wiki/log.md` içindeki `ingest` entry'leri; her sayfanın frontmatter'ındaki `sources:` |
| **Query** | `wiki/synthesis/Makbuliyet Kriterleri.md` — bir soruya verilen cevap, kalıcı sayfa olarak file edildi |
| **Lint** | `wiki/log.md` sonundaki lint entry'si: orphan sayfa, sığ sayfa, eksik kavram tespitleri |
| **Schema** | `CLAUDE.md` — tüm bunların kurallarını yazan tek dosya |

## Kendi domainine uyarlamak için

1. Bu repo'yu klonla, yeni isim ver.
2. `raw/` altındaki dosyaları sil, kendi kaynaklarını koy.
3. `CLAUDE.md`'yi domainine göre düzenle.
4. `wiki/` altındaki içeriği sil, `index.md` ve `log.md`'ı boşalt.
5. Claude Code'u aç, `/ingest` ile başla.

Obsidian'da bu klasörü vault olarak açarsan `[[wikilink]]`'ler ve graph view düzgün çalışır.

## Daha fazla okuma

- Pattern açıklaması: [`llm-wiki.md`](./llm-wiki.md)
- Pattern'i çalıştırırken somut öğrenilenler: [`LEARNINGS.md`](./LEARNINGS.md)
- Retrieval'ın nasıl çalıştığı, klasik RAG'den farkı: [`HOW-IT-WORKS.md`](./HOW-IT-WORKS.md)

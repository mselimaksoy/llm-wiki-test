# llm-wiki-test

Andrej Karpathy'nin [LLM Wiki](./llm-wiki.md) pattern'ini hands-on denemek için kurulmuş küçük bir oyun alanı.

Pattern'in özeti: RAG gibi her soruda kaynakları yeniden taramak yerine, LLM **kalıcı, birikimli bir markdown wiki** inşa edip bakımını yapar. Sen kaynak ekler ve soru sorarsın; LLM özetler, çapraz referans verir, çelişkileri işaretler, sayfaları günceller.

## Repo yapısı

```
.
├── llm-wiki.md          # Karpathy'nin orijinal pattern açıklaması (referans)
└── demo/                # Pattern'i çalıştıran somut mini örnek
    ├── CLAUDE.md        # Wiki'nin "schema"sı — LLM'e nasıl bakım yapacağını anlatır
    ├── raw/             # Ham kaynaklar (immutable). Bu örnekte 2 dosya.
    └── wiki/            # LLM'in ürettiği katman
        ├── index.md             # İçerik kataloğu
        ├── log.md               # Kronolojik olay kaydı (ingest, query, lint)
        ├── sources/             # Her ham kaynağın özeti
        ├── entities/            # Cezve, Fincan, Köpük, ...
        ├── concepts/            # Demleme Yöntemi, Fal, Şeker Tercihleri, ...
        └── synthesis/           # Query cevaplarından file edilmiş sayfalar
```

## Demo konusu: Türk Kahvesi

Bilerek küçük ve tanıdık bir konu seçildi — pattern'in mekaniğini, konunun karmaşıklığı arkasına saklanmadan görebilmek için.

İki sahte kaynak işlendi:
1. `raw/01-kahve-tarihi.md` — kısa tarih, ilk kahvehane, UNESCO tescili.
2. `raw/02-demleme.md` — demleme reçetesi, köpük, şeker tercihleri.

İkinci kaynak ingest edildiğinde [`Demleme Yöntemi`](demo/wiki/concepts/Demleme%20Y%C3%B6ntemi.md) ve [`Cezve`](demo/wiki/entities/Cezve.md) sayfaları **yeniden yazılmadı, üzerine eklendi** — pattern'in "compounding" özelliği. Her sayfanın frontmatter'ındaki `sources:` listesi hangi kaynaklardan beslendiğini gösterir.

## Pattern'in dört operasyonu (demo'da hepsi çalıştırıldı)

| Operasyon | Demoda nerede görülür |
|---|---|
| **Ingest** | `wiki/log.md` içindeki iki `ingest` entry'si; her sayfanın frontmatter'ındaki `sources:` |
| **Query** | `wiki/synthesis/Makbuliyet Kriterleri.md` — bir soruya verilen cevap, kalıcı sayfa olarak file edildi |
| **Lint** | `wiki/log.md` sonundaki lint entry'si: orphan sayfa, sığ sayfa, eksik kavram tespitleri |
| **Schema** | `demo/CLAUDE.md` — tüm bunların kurallarını yazan tek dosya |

## Kendi domainine uyarlamak için

1. `demo/` klasörünü kopyala, yeni isim ver.
2. `raw/` altındaki dosyaları sil, kendi kaynaklarını koy.
3. `CLAUDE.md`'yi domainine göre düzenle (entity/concept kategorilerini değiştir).
4. `wiki/` altındaki içeriği sil, `index.md` ve `log.md`'ı boşalt.
5. LLM agent'a (Claude Code, Codex, vs.) klasörü aç ve "yeni kaynak ekledim, ingest et" de.

Obsidian'da `demo/` klasörünü vault olarak açarsan `[[wikilink]]`'ler ve graph view düzgün çalışır.

## Demodan çıkarımlar

Pattern'i çalıştırırken somut olarak ne öğrenildiğine dair notlar: [`demo/LEARNINGS.md`](./demo/LEARNINGS.md).

## Referans

- Pattern açıklaması: [`llm-wiki.md`](./llm-wiki.md)
- Karpathy'nin tweet/yazı bağlamı: pattern, RAG'in "her sorguda sıfırdan başlama" sorununu adresliyor.

# Türk Kahvesi Wiki — Schema

Bu, "Türk Kahvesi" konulu küçük bir LLM-bakımlı wiki'dir. Pattern: kaynaklar ham olarak `raw/` altında durur; LLM bunları okuyup `wiki/` altında yapılandırılmış, birbirine bağlı markdown sayfaları üretir ve bakımını yapar.

## Klasör yapısı

- `raw/` — ham kaynaklar. **Asla değiştirme.** Sadece oku.
- `wiki/` — LLM'in sahip olduğu katman. Tüm sayfalar burada.
- `wiki/index.md` — içerik kataloğu (kategorilere göre).
- `wiki/log.md` — kronolojik olay kaydı (append-only).
- `wiki/entities/` — somut şeyler (kişi, yer, nesne): Cezve, Fincan, Telve, vb.
- `wiki/concepts/` — soyut kavramlar: Demleme Yöntemi, Köpük, Fal, vb.
- `wiki/sources/` — her ham kaynak için özet sayfası.

## Sayfa formatı

Her wiki sayfası şu YAML frontmatter ile başlar:

```
---
type: entity | concept | source | synthesis
sources: [01-kahve-tarihi, 02-demleme]   # bu sayfayı besleyen raw dosyalar
updated: YYYY-MM-DD
---
```

Cross-link'ler Obsidian tarzı `[[Sayfa Adı]]` kullanır.

## Workflow'lar

### Ingest (yeni kaynak geldiğinde)
1. `raw/`'daki dosyayı oku.
2. `wiki/sources/<slug>.md` altında özet sayfası oluştur.
3. Kaynakta geçen entity ve concept'leri çıkar. Her biri için ya yeni sayfa aç, ya mevcut sayfayı güncelle (yeni bilgi eklerken hangi kaynaktan geldiğini frontmatter `sources:` listesine ekle).
4. Çelişki varsa sayfada `> ⚠️ Çelişki:` bloğuyla işaretle.
5. `index.md`'i güncelle.
6. `log.md`'a `## [YYYY-MM-DD] ingest | <başlık>` ile entry ekle; hangi sayfaların değiştiğini listele.

### Query (soru sorulduğunda)
1. `index.md`'i tara, ilgili sayfaları aç, oku, sentezle, citation ver.
2. Cevap kalıcı değer taşıyorsa `wiki/synthesis/` altına yeni bir sayfa olarak file et.
3. `log.md`'a `## [YYYY-MM-DD] query | <soru>` entry'si ekle.

### Lint (sağlık kontrolü)
- Orphan sayfalar (gelen link yok)
- Çelişkiler
- Kavramı geçen ama sayfası olmayan terimler
- Eskimiş claim'ler
Bulguları `log.md`'a yaz, otomatik düzeltme yapmadan önce sor.

## Stil

- Türkçe yaz.
- Sayfalar kısa ve referans-tarzı olsun, makale değil.
- Her iddia için kaynak göster: `(kaynak: [[source: 01-kahve-tarihi]])`

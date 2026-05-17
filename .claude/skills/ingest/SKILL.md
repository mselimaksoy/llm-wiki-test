---
name: ingest
description: raw/ klasörüne yeni eklenen bir kaynağı wiki'ye işle. Kullanıcı "ingest et", "şu dosyayı ingest et", "yeni kaynak" gibi ifadeler kullandığında veya /ingest yazdığında tetikle.
---

# Ingest Skill

raw/ altındaki yeni bir kaynağı CLAUDE.md'deki schema'ya göre wiki'ye işler.

## Adımlar

1. Hangi raw dosya işlenecek? Kullanıcı belirttiyse onu kullan. Belirtmediyse `ls raw/` ile listele ve sor.
2. Dosyayı oku.
3. CLAUDE.md'yi oku (kuralları hatırlamak için).
4. Kullanıcıyla 1-2 cümlelik özet üzerinden hizala: "Bu kaynaktan şu entity/concept'leri çıkarmayı planlıyorum, devam edeyim mi?"
5. Hizalama tamam → şunları yap:
   - `wiki/sources/<slug>.md` oluştur (frontmatter + ana noktalar + etkilenen sayfalar listesi)
   - Yeni entity/concept sayfalarını aç
   - Mevcut sayfaları **üzerine yazma** — frontmatter'daki `sources:` listesine yeni kaynağı ekle, içeriği genişlet
   - Çelişki varsa `> ⚠️ Çelişki:` ile işaretle
   - `wiki/index.md`'i güncelle
   - `wiki/log.md`'a `## [YYYY-MM-DD] ingest | <başlık>` entry'si ekle; yeni/güncellenen sayfaları listele
6. Sonunda kullanıcıya kısa rapor: kaç yeni sayfa, kaç güncellenmiş sayfa, hangi link'leri eklediğin.

## Notlar

- Tarihi `date +%Y-%m-%d` ile al, uydurma.
- raw/ altındaki dosyaları **asla değiştirme**.
- Sayfa sayısı çoksa Write çağrılarını paralel yap.

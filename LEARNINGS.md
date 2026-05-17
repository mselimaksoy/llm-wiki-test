# Demo'dan Çıkarımlar

Türk kahvesi mini-MVP'sini baştan sona çalıştırdıktan sonra pattern hakkında somut olarak öğrenilenler.

## 1. Schema her şeyin merkezi

[`CLAUDE.md`](./CLAUDE.md) olmadan LLM "wiki bakımcısı" değil, sıradan bir asistan. Sayfa formatını, klasör yapısını ve workflow'ları (ingest/query/lint) orada sabitlemek lazım. Schema iyi yazıldığında LLM'in her oturumda aynı disipline dönmesi garanti olur; yazılmadığında her seferinde stil ve yapı kayar.

## 2. Compounding'i ikinci ingest'te gördük

[`wiki/concepts/Demleme Yöntemi.md`](./wiki/concepts/Demleme%20Y%C3%B6ntemi.md) ilk ingest'te 2 satırlık bir stub'tı; ikinci kaynak gelince 5 adımlı reçeteye dönüştü. Frontmatter'daki `sources:` listesi her iki kaynağı da gösteriyor — yani sayfa, üzerine yazılarak değil, **birikerek** büyüdü.

RAG'de bu birikim olmaz: her sorguda iki kaynak yeniden taranır ve LLM cevabı sıfırdan kurgular. Burada ise sentez bir kez yapılıp diske yazıldı.

## 3. Synthesis sayfaları query'lerin kazancını kalıcı yapıyor

[`wiki/synthesis/Makbuliyet Kriterleri.md`](./wiki/synthesis/Makbuliyet%20Kriterleri.md) bir query'nin cevabıydı; chat history'de kaybolmak yerine wiki'ye file edildi. Aynı soru gelirse hazır sayfa var; başka bir query bu sayfaya link verebilir. Karpathy'nin "explorations compound" dediği mekanizma tam olarak bu.

## 4. Log + index, LLM'in hafıza desteği

- [`wiki/log.md`](./wiki/log.md) → "geçen sefer ne yaptık" sorusunu cevaplar. Tutarlı prefix (`## [YYYY-MM-DD] ingest | ...`) sayesinde `grep` ile parse edilebilir.
- [`wiki/index.md`](./wiki/index.md) → embedding-RAG yerine "önce kataloğu oku, sonra ilgili sayfaya dal" mantığını mümkün kılıyor. Küçük/orta ölçekte vektör DB'ye gerek bırakmıyor.

## 5. Lint, bilgi boşluklarını geri yansıtıyor

Log'daki son lint entry'si "köpüğün neden makbuliyet kriteri olduğu yazılı değil" diye not düştü. Bu, bir sonraki kaynağı aramak için doğal bir tetikleyici — **wiki kendi büyüme yönünü öneriyor**. Orphan sayfalar ve sığ entity'ler de aynı işlevi görüyor.

## Sonraki adımlar

İki gerçek yol:
- **(a)** Bu klasörü Obsidian vault olarak aç, graph view'da bağlantıları görsel olarak gez.
- **(b)** Gerçek bir domain seç ([`CLAUDE.md`](./CLAUDE.md)'yi şablon alarak) sıfırdan kur. Domain seçerken: birikim değeri olan, zamanla derinleşeceğin, en az 5-10 kaynağa ulaşabileceğin bir konu seç — yoksa pattern'in compounding avantajı görünmez.

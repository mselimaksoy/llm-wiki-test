# Çalışma Metodu: Retrieval Nasıl Oluyor?

Bu dokümanın amacı: "LLM bu wiki'den nasıl bilgi çekiyor, klasik RAG'den farkı ne?" sorusunu netleştirmek.

## Burada klasik RAG **yok**

Karpathy'nin pattern'inde embedding-tabanlı similarity search **yapılmıyor**:
- Vector DB yok
- Chunking yok
- Cosine similarity yok
- "En yakın 5 chunk'ı getir" mantığı yok

Bunun yerine **agent'ın dosya sistemi tool'larını kullanması** var. Bu çok farklı bir paradigma.

## Agent bir sorguyu nasıl cevaplıyor? (agentic retrieval)

Sen agent'a sordun: *"Köpük neden önemli?"*

Agent'ın kafasının içindeki süreç (tool-using bir LLM'in standart döngüsü):

1. **`Read wiki/index.md`** → kataloğu görür. "Köpük entity'si var, Demleme Yöntemi concept'i var, Makbuliyet Kriterleri synthesis'i var."
2. **`Read wiki/entities/Köpük.md`** → sayfayı okur. İçinde `[[Cezve]]`, `[[Demleme Yöntemi]]` link'leri görür.
3. **Karar verir**: "Daha fazla context'e ihtiyacım var mı?" Varsa link'leri takip eder: **`Read wiki/concepts/Demleme Yöntemi.md`**.
4. Yeterli bilgi toplandı → cevabı sentezler.

Bu **agentic retrieval** — model her adımda "hangi dosyayı okumalıyım" diye karar verir. Klasik RAG'de retrieval işi LLM'den **önce** yapılır ve sabittir. Burada retrieval LLM'in **kendisi** tarafından, **iteratif** olarak yapılır.

## Evet, her tool çağrısı bir LLM invocation

Pratikteki gerçek:

- Tek bir query → 3-8 dosya okuma turu → 3-8 model çağrısı. (Claude/GPT'nin tool-use modunda tek conversation içinde peş peşe gider, ama her tool turn'ü yeni bir invocation'dır.)
- Wiki büyüdükçe `index.md` büyür → her sorguda baştan tüm index modele token olarak gider. 500 sayfalık bir wiki'de index 20-30K token olur.
- Cevap üretimi 10-30 saniye sürebilir. Bir Pinecone sorgusu 50ms.

## Neden yine de işe yarıyor?

Çünkü işin doğası değişti. Klasik RAG'in iyi olduğu şey: **"100 milyon dokümandan en alakalı 5'ini bul"**. Karpathy'nin pattern'inin iyi olduğu şey: **"100-500 dosyalık, sen ve LLM tarafından titizce düzenlenmiş bir bilgi tabanı içinde gez"**.

İkincisi şuna güveniyor:
1. **Wiki küçük** — modelin context window'una index sığıyor (Claude 200K, GPT-4 128K token).
2. **Wiki zaten organize** — sayfa isimleri anlamlı, link'ler iyi, başlıklar net. Retrieval'a gerek kalmadan "Köpük'ü oku" yeterli bir komut.
3. **Link'ler retrieval'ın yerine geçiyor** — Köpük sayfasındaki `[[Cezve]]` link'i, embedding-similarity'ye gerek bırakmadan modele "bu da alakalı" sinyalini veriyor.

Yani **wiki'nin yapısı = retrieval index'inin kendisi**. Embedding'lerle yapay olarak bağlantı bulmaya çalışmıyorsun; bağlantılar zaten yazılı halde duruyor.

## Ölçek sınırı

Bu pattern ~100-500 kaynak / birkaç bin sayfa civarına kadar saf haliyle çalışır. Onun ötesinde:

1. **index.md context'e sığmaz** olur.
2. Agent **doğru sayfayı bulamaz** çünkü çok seçenek var.

Karpathy'nin `llm-wiki.md` dosyasında bahsettiği çözüm tam burada devreye giriyor:

> "At some point you may want to build small tools that help the LLM operate on the wiki more efficiently. A search engine over the wiki pages is the most obvious one..."

**qmd** (Tobi Lütke'nin aracı) gibi araçlar yardımcı: wiki içinde **lokal BM25 + vector search** yapıyor, agent CLI olarak çağırıyor. Yani RAG geri geliyor — ama bu sefer **wiki üzerinde** RAG, raw belgeler üzerinde değil. İki katmanlı sistem:

- **Katman 1 (raw → wiki)**: LLM bir kez okur, distile eder, organize eder. Pahalı ama bir kere yapılır.
- **Katman 2 (wiki → cevap)**: Sorguda BM25/vector ile *iyi düzenlenmiş wiki sayfaları* getirilir. Çok daha küçük corpus, çok daha kaliteli içerik.

Bu, klasik RAG'in en büyük zaafını çözüyor: **klasik RAG'de retrieval kalitesi raw belgelerin kalitesi kadar kötü**. Karpathy'nin pattern'inde retrieval, LLM'in zaten düzelttiği bir corpus üzerinde yapılıyor.

## Maliyet kıyaslaması (somut, 200 kaynaklık alan)

| Sistem | Sorgu başına |
|---|---|
| Klasik RAG (Pinecone vb.) | 1 embedding + 1 vector search + 1 LLM call (~3K token in / 1K out) |
| Karpathy pattern (saf) | 5-8 LLM call (her biri index + sayfa okuma, toplam ~30K token in / 2K out) |
| Karpathy + qmd (hibrit) | 1 BM25 + 2-3 LLM call (~10K token in / 1K out) |

Karpathy pattern'i sorgu başına **3-5x daha pahalı**. Ama:
- Ingest aşamasında bir kez ödediğin distilasyon maliyeti, klasik RAG'de **her sorguda** tekrar ediyor (LLM her seferinde 5 raw chunk'tan sıfırdan sentez yapıyor).
- Cevap kalitesi çok daha yüksek çünkü model zaten organize, sentezlenmiş içerik okuyor.

## Zihin haritası

```
Klasik RAG:
  raw → [vector DB] → query'de retrieval → LLM cevap üretir
  ↑ her query'de aynı işi tekrar yapar

Karpathy pattern:
  raw → LLM (ingest) → wiki  ←  LLM (query, agentic dosya okuma) → cevap
                       ↑
                  bir kez yapılır,
                  kalıcı artefakt
```

## Pattern'in zarafeti: yeni altyapı icat edilmiyor

Hepsi "generic coding agent" ile oluyor. Çünkü agent'ın yaptığı tek şey: **`Read`, `Write`, `Edit`, `Grep`** tool'larını kullanmak. Bu tool'lar koddaki dosyalar için de markdown wiki için de aynı şekilde çalışıyor. Yeni bir SaaS ürünü, custom UI, vector DB altyapısı yok — mevcut coding agent altyapısını wiki'ye uygulamak.

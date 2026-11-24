# Kişisel ve Ev İçi Harcama Analizi (Python & SQLite)

![Python](https://img.shields.io/badge/Python-3.13-3776AB?style=for-the-badge&logo=python&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-3-003B57?style=for-the-badge&logo=sqlite&logoColor=white)
![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white)

## Proje Özeti

**Problem:** Ortak yaşam alanlarında veya kişisel bütçe yönetiminde yapılan harcamaların dağınık şekilde tutulması; kategori bazlı maliyet dağılımının görülmesini, belirlenen bütçe sınırlarının takibini ve olağan dışı harcamaların fark edilmesini zorlaştırır.

**Çözüm:** Bu proje, Python'ın yerleşik `sqlite3` modülü kullanılarak ilişkisel bir yerel veritabanı (`harcama_analizi.db`) üzerinde harcama verilerini saklar, SQL sorguları ile toplulaştırma analizleri üretir ve Python tarafında bütçe eşiği kontrolü, yüzdelik pay hesabı ile alt sorgu tabanlı anomali tespiti gerçekleştirir.

---

## Veritabanı Şeması

Veritabanı üç temel tablodan oluşmaktadır:

### `Users`
| Sütun Adı | Veri Tipi | Kısıtlama / Açıklama |
| :--- | :--- | :--- |
| `ID` | `INTEGER` | `PRIMARY KEY` — Kullanıcı benzersiz kimliği |
| `name` | `VARCHAR` | Kullanıcı adı (Örn: `Esmanur`, `Edanur`) |

### `Categorıes`
| Sütun Adı | Veri Tipi | Kısıtlama / Açıklama |
| :--- | :--- | :--- |
| `ID` | `INTEGER` | `PRIMARY KEY` — Kategori benzersiz kimliği |
| `NAME` | `TEXT` | Harcama kategorisi (`YEMEK`, `EĞLENCE`, `FATURA`, `MARKET`, `ULAŞIM`) |

### `Expenses`
| Sütun Adı | Veri Tipi | Kısıtlama / Açıklama |
| :--- | :--- | :--- |
| `id` | `INTEGER` | `PRIMARY KEY` — Harcama kayıt kimliği |
| `user_id` | `INTEGER` | Harcamayı yapan kullanıcının kimliği (`Users.ID` referansı) |
| `category_id` | `INTEGER` | Harcama kategorisinin kimliği (`Categorıes.ID` referansı) |
| `amount` | `REAL` | Harcama tutarı (TL) |
| `date` | `TEXT` | Harcama tarihi (`YYYY-MM-DD` formatında) |

**Mantıksal İlişki:** `Expenses` tablosu, `user_id` sütunu üzerinden `Users` tablosuna ve `category_id` sütunu üzerinden `Categorıes` tablosuna **bire-çok (1:N)** mantıksal ilişki ile bağlanır. Bir kullanıcı birden fazla harcama kaydına sahip olabilir; aynı şekilde bir kategori altında birden fazla harcama yer alabilir. Mevcut yapıda bu ilişkiler uygulama düzeyinde yönetilmektedir.

---

## Öne Çıkan Özellikler ve Analizler

### 1. CRUD İşlemleri
- **Create:** `Users`, `Categorıes` ve `Expenses` tablolarının oluşturulması ve örnek kayıtların `INSERT INTO` ile eklenmesi.
- **Read:** Tüm harcamaların listelenmesi ve `WHERE user_ID = 1` koşulu ile kullanıcı bazlı filtrelenmesi.
- **Update:** `ID = 1` numaralı harcama tutarının `250.0 TL`'den `600.0 TL`'ye güncellenmesi.
- **Delete:** `ID = 6` numaralı kaydın veritabanından silinmesi.

### 2. SQL Toplulaştırma (Aggregation) Sorguları
- **Kategori Bazlı Toplam:** `GROUP BY category_ID` ve `ORDER BY SUM(amount) DESC` ile en yüksek harcama yapılan kategorilerin sıralanması.
- **Genel Ortalama:** `AVG(amount)` fonksiyonu ile işlem başına ortalama harcama tutarının hesaplanması.
- **Günlük Dağılım:** `GROUP BY date` ile tarih bazlı günlük toplam harcamaların çıkarılması.
- **Kullanıcı Bazlı Toplam:** `GROUP BY user_ID` ile her kullanıcının toplam harcama miktarının hesaplanması.

### 3. Bütçe Aşımı Uyarı Mekanizması
SQL sorgusundan dönen kullanıcı toplamları, Python tarafında tanımlanan `700.0 TL` bütçe eşiği ile karşılaştırılır. Sınırı aşan kullanıcılar için uyarı, sınırın altında kalanlar için bütçeye uygunluk bildirimi üretilir.

### 4. Toplam Harcama İçindeki Yüzdelik Pay
Evdeki genel toplam harcama hesaplanarak her kullanıcının toplam gider içindeki oranı tek ondalık basamağa (`round(yuzde, 1)`) yuvarlanmış şekilde raporlanır.

### 5. Alt Sorgu (Subquery) ile Anomali Tespiti
Test amacıyla eklenen `15000.0 TL` tutarındaki yüksek harcama, genel harcama ortalamasının 2 katını aşan kayıtları filtreleyen iç içe SQL sorgusu ile tespit edilir:

```sql
SELECT * FROM Expenses
WHERE amount > (SELECT AVG(amount) * 2 FROM Expenses);
```

---

## Örnek Çıktılar

### Bütçe Kontrolü ve Uyarı Çıktısı
```text
Esmanur adlı kullanıcının toplam harcaması 825.0 TL'dir.
----------------------------------------
⚠️ Dikkat Esmanur, 700.0 TL olan bütçeyi aştın!
----------------------------------------
Edanur adlı kullanıcının toplam harcaması 640.9 TL'dir.
----------------------------------------
✅ Tebrikler Edanur, bütçeye uygun harcadın.
```

### Kullanıcı Bazlı Yüzdelik Harcama Dağılımı
```text
Evdeki Genel Toplam Harcama: 1465.9 TL
----------------------------------------
Esmanur, evdeki toplam harcamaların %56.3 'ini gerçekleştirdi.
Edanur, evdeki toplam harcamaların %43.7 'ini gerçekleştirdi.
```

### Anormal Harcama Tespiti
```text
🚨 DİKKAT: Anormal Harcamalar Tespit Edildi! 🚨
----------------------------------------
Şüpheli İşlem Paketi: (6, 1, 1, 15000.0, '2026-07-17')
```

---

## Kurulum ve Çalıştırma

Proje, Python'ın standart kütüphanesinde yer alan `sqlite3` modülünü kullandığı için ek bir veritabanı paketi gerektirmez.

### Gereksinimler
- Python 3.8 veya üzeri
- Jupyter Notebook veya JupyterLab

### Adımlar

1. Depoyu yerel bilgisayarınıza klonlayın:
   ```bash
   git clone https://github.com/esmanuryavuz908/sql-harcama-analizi.git
   cd sql-harcama-analizi
   ```

2. Jupyter Notebook yüklü değilse kurun:
   ```bash
   pip install notebook
   ```

3. Not defterini başlatın:
   ```bash
   jupyter notebook
   ```

4. Açılan tarayıcı penceresinden ilgili `.ipynb` dosyasını seçin ve hücreleri sırayla çalıştırın. Çalıştırma sırasında `harcama_analizi.db` dosyası proje dizininde otomatik olarak oluşturulacaktır.

---

## Geliştirme Planı (Roadmap)

- [ ] **İlişkisel Bütünlük ve İsimlendirme Standardizasyonu:** `Categorıes` tablo adının ASCII uyumlu (`Categories`) hale getirilmesi ve `Expenses` tablosuna veritabanı düzeyinde `FOREIGN KEY` kısıtlamalarının eklenmesi.
- [ ] **SQL `JOIN` Entegrasyonu:** Python tarafındaki statik kullanıcı sözlüğü (`{1: "Esmanur", 2: "Edanur"}`) yerine, kullanıcı ve kategori adlarını doğrudan veritabanından çeken `INNER JOIN` sorgularına geçilmesi.
- [ ] **Veri Görselleştirme:** Sorgu sonuçlarının `Pandas` DataFrame'e aktarılarak `Matplotlib` veya `Seaborn` ile kategori dağılımı (pasta/çubuk grafik) ve zaman serisi (çizgi grafik) olarak görselleştirilmesi.
- [ ] **Etkileşimli Arayüz:** Harcama ekleme, bütçe limiti belirleme ve canlı anomali takibi için `Streamlit` tabanlı bir web arayüzü veya komut satırı (CLI) aracı geliştirilmesi.

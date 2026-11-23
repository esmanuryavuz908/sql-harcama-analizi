# Python & SQLite Harcama Analizi ve Anormallik Tespiti

Python `sqlite3` kütüphanesi kullanılarak geliştirilmiş; ilişkisel veritabanı kurulumu, CRUD işlemleri, bütçe takibi ve SQL alt sorgusu (subquery) ile anormal harcama tespiti içeren veri analizi projesi.

## Veritabanı Şeması

Proje, `harcama_analizi.db` üzerinde üç temel tablo ile çalışır:
- **`Users`**: Kullanıcı bilgileri (`ID`, `name`)
- **`Categories`**: Harcama kategorileri (`ID`, `name` — Yemek, Eğlence, Fatura, Market, Ulaşım)
- **`Expenses`**: Harcama kayıtları (`ID`, `user_id`, `category_id`, `amount`, `date`)

## Yapılan Analizler ve Özellikler

- **SQL Toplulaştırma Sorguları:** Kategori, tarih ve kullanıcı bazında toplam harcamaların (`SUM`, `GROUP BY`, `ORDER BY`) ve genel ortalama harcamanın (`AVG`) hesaplanması.
- **Bütçe Kontrolü ve Pay Analizi:** Belirlenen bütçe sınırına (700 TL) göre kullanıcı bazlı aşım uyarıları ve ev içi toplam harcamadaki yüzdesel dağılımın raporlanması.
- **CRUD İşlemleri:** Harcama kayıtlarının eklenmesi (`INSERT`), güncellenmesi (`UPDATE`) ve silinmesi (`DELETE`).
- **Anormallik Tespiti:** SQL alt sorgusu kullanılarak genel ortalamanın 2 katını aşan olağan dışı harcamaların otomatik olarak yakalanması.

## Kullanılan Teknolojiler

- Python 3
- SQLite (`sqlite3`)
- Jupyter Notebook

## Çalıştırma

1. Depoyu bilgisayarınıza klonlayın veya `harcama_analizi.ipynb` dosyasını indirin.
2. Dosyayı VS Code veya Jupyter Notebook üzerinden açıp hücreleri sırayla çalıştırın (`harcama_analizi.db` dosyası otomatik oluşturulacaktır).

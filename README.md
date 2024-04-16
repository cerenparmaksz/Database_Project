# Database_Project

# OtoBayi Projesi

Bu proje, bir otomobil bayisinin veritabanı işlemlerini içeren bir SQL scriptini ve bu script ile oluşturulan bazı görünümleri içermektedir.

## Veritabanı Tanımı

Veritabanı adı: OtoBayi

Veritabanı şeması aşağıdaki gibidir:

- **tbl_Araba:** Araba verilerini saklayan tablo.
- **tbl_Departman:** Departman bilgilerini saklayan tablo.
- **tbl_DonanimOzellik:** Araç donanım özelliklerini saklayan tablo.
- **tbl_ikinciEl:** İkinci el araba bilgilerini saklayan tablo.
- **tbl_Musteri:** Müşteri bilgilerini saklayan tablo.
- **tbl_Personel:** Bayi personel bilgilerini saklayan tablo.
- **tbl_Servis:** Servis bilgilerini saklayan tablo.
- **tbl_Siparis:** Sipariş bilgilerini saklayan tablo.
- **tbl_YakitTuru:** Yakıt türü bilgilerini saklayan tablo.
- **tbl_YedekParca:** Yedek parça bilgilerini saklayan tablo.

## Oluşturulan Görünümler

### ArabaOzellikleri
- Arabaların bütün özelliklerini, ikinci el durumunu ve donanım özelliklerini içeren bir görünüm.
- Sorgu:

```sql
create View ArabaOzellikleri
As
select fiyat, renk, marka, cikisYili, model, vitesOzellik, motorGuc, motorHacim, cekisTur, kasaTip, ad as 'yakıt türü', hasarDurum, kmBilgi 
from tbl_Araba araba 
inner join tbl_DonanimOzellik donanim on araba.arabaID = donanim.donanimID 
inner join tbl_ikinciEl ikinciel on araba.arabaID = ikinciel.ID 
inner join tbl_YakitTuru yakittur on donanim.yakitTurID = yakittur.yakitID
```

## Personel Özellikleri 

Bu SQL görünümü, personellerin tüm bilgilerini ve hangi müşterilere hizmet verdiklerini içeren bir görünüm sağlar. Görünüm, personellerin isim, soyisim, doğum tarihi, işe giriş tarihi, telefon numarası, adres, maaş gibi kişisel ve işle ilgili bilgilerini sunar. Ayrıca her personelin hangi müşterilere hizmet verdiğini gösteren bir alan içerir.

Görünümü kullanmak için aşağıdaki SQL sorgusunu kullanabilirsiniz:

```sql
CREATE VIEW PersonelOzellikleri AS
SELECT 
    personel.isim, 
    personel.soyisim, 
    personel.dogumTarihi, 
    personel.iseGirisTarihi, 
    personel.telefonNo, 
    personel.adres, 
    personel.maas, 
    musteri.isim AS 'ilgilendiği müsteri' 
FROM 
    tbl_Personel personel 
INNER JOIN 
    tbl_Departman departman ON personel.departmanID = departman.departmanID 
INNER JOIN 
    tbl_Musteri musteri ON personel.personelID = musteri.ilgiliPersonelID;
```
## Trigger Tanımları

### Araba Silme Triggerı

Bir araba silme işlemi gerçekleştirilirken, bu arabanın daha önce satılmadığı kontrol edilerek silinmesi engellenmektedir.

```sql
create trigger arabaSilme
on tbl_Araba
instead of delete
As 
Begin
    if Exists(
        select * from tbl_Araba TA 
        inner join tbl_Musteri TM ON TA.arabaID=TM.satilanArabaID
        where satilanArabaID in (Select arabaID from deleted)
    )
    BEGIN
        RAISERROR('Araba satışı yapılmamış araba silinemez.',16,1)
        ROLLBACK
    END
END
```

### Personel Silme Triggerı

Bir personel silme işlemi yapılırken, silinen personelin yönetim kurulunda olup olmadığı kontrol edilerek silme işlemi gerçekleştirilmemektedir.

```sql
create trigger personelSilme
on tbl_Personel
instead of delete
As 
Begin
    if Exists(
        select * from tbl_Personel TP 
        inner join tbl_Departman TD on TP.departmanID=TD.departmanID
        where personelID in (Select personelID from deleted) AND TD.ad='YÖNETİM KURULU'
    )
    BEGIN
        RAISERROR('YÖNETİM KURULUNDAKİ PERSONEL SİLİNEMEZ.',16,1)
        ROLLBACK
    END
END



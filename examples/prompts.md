# Örnek prompt'lar

Hepsini kopyalayıp Claude Code'a yapıştırabilirsin. `<...>` içindeki kısımları kendi projene göre değiştir.

---

**1. Tek bir agent'a iş vermek: yeni endpoint.**

```text
backend-dev agent'ını kullanarak <kaynak> için bir GET /<kaynak>/:id endpoint'i ekle. Mevcut route ve hata yönetimi stiline uy.
```

**2. Yazılmış koda test eklemek.**

```text
test-writer agent'ını kullanarak <dosya yolu> içindeki fonksiyonlar için test yaz. Kenar durumlar ve hata senaryoları dahil olsun, sonra testleri çalıştır.
```

**3. Son değişiklikleri incelettirmek.**

```text
code-reviewer agent'ını kullanarak son değişiklikleri (git diff) incele. Bulguları kritik, uyarı ve öneri olarak sırala, her biri için dosya ve satır ver.
```

**4. Bir hatanın kök nedenini bulmak.**

```text
debugger agent'ını kullanarak şu hatayı çöz: <hata mesajı veya stack trace>. Kök nedeni bul, en küçük düzeltmeyi yap ve çalıştığını kanıtla.
```

**5. Dört agent'ı paralel çalıştırmak (bağımsız işler).**

```text
Şu dört işi paralel olarak, ayrı agent'larla yap: backend-dev <A modülüne> yeni bir endpoint eklesin, test-writer <B modülü> için test yazsın, code-reviewer <C modülünü> incelesin, debugger <hata mesajı> hatasını araştırsın. Hepsi bitince sonuçları tek bir özet olarak ver.
```

**6. Zincirleme çalıştırmak (yaz, test et, incele, gerekirse düzelt).**

```text
Sırayla ilerle: önce backend-dev ile <feature açıklaması> feature'ını yaz, sonra test-writer ile testlerini ekle, sonra code-reviewer ile incele. Kritik veya uyarı seviyesinde bulgu çıkarsa debugger ile düzelt ve testleri tekrar çalıştır.
```

**7. Pull request öncesi son kontrol.**

```text
code-reviewer ile bu branch'teki tüm değişiklikleri incele, ardından test-writer ile test kapsamında eksik kalan yerleri bul ve eksik testleri yaz.
```

**8. Başarısız testleri düzeltmek.**

```text
Test suite'ini çalıştır. Başarısız test varsa debugger agent'ı ile her birinin kök nedenini bul ve düzelt. Hangi testlerin neden kaldığını özetle.
```

**9. Güvenlik odaklı inceleme.**

```text
code-reviewer agent'ını kullanarak <klasör> altındaki kodu sadece güvenlik açısından incele: girdi doğrulama, yetkilendirme, secret sızıntısı ve injection riskleri. Bulguları öncelik sırasıyla ver.
```

**10. Agent teams: aynı hatayı rakip hipotezlerle araştırmak (deneysel, README'deki ayarı aç).**

```text
<hata açıklaması> hatasını araştırmak için debugger agent tipini kullanarak 3 teammate oluştur. Her biri farklı bir hipotezi incelesin ve birbirlerinin teorilerini çürütmeye çalışsınlar. Ortaya çıkan sonucu özetle.
```

BAŞLA

  # 1. KULLANICI GİRİŞİ
  YAZ "Lütfen kullanıcı adınızı giriniz:"
  KULLANICI_ADI ← GİR()
  YAZ "Şifrenizi giriniz:"
  SIFRE ← GİR()

  EĞER KULLANICI_ADI = DOĞRU_AD VE SIFRE = DOĞRU_SIFRE İSE
      YAZ "Giriş başarılı, hoş geldiniz!"
  DEĞİLSE
      YAZ "Hatalı giriş! Lütfen tekrar deneyiniz."
      DUR
  BİTTİ

  # 2. ÜRÜN GEZİNME VE SEÇME
  TEKRAR
      YAZ "Ürün kategorilerini seçiniz: (Elektronik / Giyim / Kitap / Ev / Çıkış)"
      KATEGORİ ← GİR()
      EĞER KATEGORİ = "Çıkış" İSE
          ÇIK
      BİTTİ

      YAZ KATEGORİ + " kategorisindeki ürünler listeleniyor..."
      YAZ "Bir ürün seçiniz:"
      ÜRÜN ← GİR()

      # 3. STOK KONTROLÜ
      STOK ← STOK_KONTROL_ET(ÜRÜN)
      EĞER STOK <= 0 İSE
          YAZ "Üzgünüz, bu ürün stokta yok."
      DEĞİLSE
          YAZ "Kaç adet almak istiyorsunuz?"
          ADET ← GİR()
          EĞER ADET > STOK İSE
              YAZ "Yetersiz stok. Maksimum " + STOK + " adet alınabilir."
          DEĞİLSE
              SEPETE_EKLE(ÜRÜN, ADET)
              YAZ "Ürün sepete eklendi."
          BİTTİ
      BİTTİ

      YAZ "Başka ürün eklemek istiyor musunuz? (E/H)"
      DEVAM ← GİR()
  DEVAM = "E" OLDUĞU SÜRECE TEKRAR ET

  # 4. SEPETİ GÖRÜNTÜLEME VE DÜZENLEME
  YAZ "Sepetinizdeki ürünler:"
  SEPETİ_GÖSTER()

  YAZ "Sepeti düzenlemek ister misiniz? (E/H)"
  DÜZENLE ← GİR()
  EĞER DÜZENLE = "E" İSE
      SEPETİ_DÜZENLE()
  BİTTİ

  # 5. İNDİRİM KODU UYGULAMA
  YAZ "İndirim kodunuz var mı? (E/H)"
  KOD_CEVAP ← GİR()
  EĞER KOD_CEVAP = "E" İSE
      YAZ "İndirim kodunu giriniz:"
      İNDİRİM_KODU ← GİR()
      EĞER KOD_GEÇERLİ(İNDİRİM_KODU) İSE
          İNDİRİM_UYGULA(İNDİRİM_KODU)
          YAZ "İndirim başarıyla uygulandı."
      DEĞİLSE
          YAZ "Geçersiz veya süresi dolmuş kod!"
      BİTTİ
  BİTTİ

  # 6. MİNİMUM SİPARİŞ TUTARI KONTROLÜ
  TOPLAM ← SEPET_TOPLAMI()
  EĞER TOPLAM < 50 İSE
      YAZ "Minimum alışveriş tutarı 50 TL olmalıdır."
      DUR
  BİTTİ

  # 7. KARGO ÜCRETİ HESAPLAMA
  EĞER TOPLAM >= 200 İSE
      KARGO_UCRETİ ← 0
      YAZ "200 TL üzeri alışverişlerde kargo ücretsiz!"
  DEĞİLSE
      KARGO_UCRETİ ← 20
      YAZ "Kargo ücreti: 20 TL"
  BİTTİ

  GENEL_TOPLAM ← TOPLAM + KARGO_UCRETİ
  YAZ "Ödenecek toplam tutar: " + GENEL_TOPLAM + " TL"

  # 8. ÖDEME YÖNTEMİ SEÇİMİ
  YAZ "Ödeme yönteminizi seçiniz: (Kredi Kartı / Havale / Kapıda Ödeme)"
  ÖDEME ← GİR()

  EĞER ÖDEME = "Kredi Kartı" İSE
      YAZ "Kart bilgilerini giriniz:"
      KART_NO ← GİR()
      SKT ← GİR()
      CVV ← GİR()
      EĞER KART_DOĞRULA(KART_NO, SKT, CVV) İSE
          YAZ "Ödeme işlemi gerçekleştiriliyor..."
          ÖDEME_DURUMU ← ÖDEME_ONAYLA(GENEL_TOPLAM)
      DEĞİLSE
          YAZ "Kart bilgileri hatalı!"
          DUR
      BİTTİ
  DEĞİLSE EĞER ÖDEME = "Havale" İSE
      YAZ "Lütfen belirtilen IBAN’a ödeme yapınız."
      ÖDEME_DURUMU ← BEKLEMEDE
  DEĞİLSE EĞER ÖDEME = "Kapıda Ödeme" İSE
      YAZ "Kapıda ödeme seçildi."
      ÖDEME_DURUMU ← ONAYLANDI
  DEĞİLSE
      YAZ "Geçersiz ödeme yöntemi."
      DUR
  BİTTİ

  # 9. SİPARİŞ ONAYI
  EĞER ÖDEME_DURUMU = ONAYLANDI VEYA BEKLEMEDE İSE
      SİPARİŞ_OLUŞTUR(SEPET, KULLANICI_ADI, GENEL_TOPLAM, ÖDEME_DURUMU)
      YAZ "Siparişiniz başarıyla oluşturuldu!"
      EPOSTA_GÖNDER(KULLANICI_ADI)
  DEĞİLSE
      YAZ "Ödeme başarısız. Lütfen tekrar deneyiniz."
  BİTTİ

  YAZ "Alışverişiniz için teşekkür ederiz!"
DUR

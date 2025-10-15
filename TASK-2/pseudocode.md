BAŞLA

  YAZ "Kullanıcı girişi yapınız:"
  GİRİLEN_KULLANICI ← GİR()
  GİRİLEN_SIFRE ← GİR()
  
  EĞER GİRİLEN_KULLANICI = GEÇERLİ_KULLANICI VE GİRİLEN_SIFRE = DOĞRU_SIFRE İSE
      YAZ "Giriş başarılı."
  DEĞİLSE
      YAZ "Giriş hatalı! Sistemden çıkılıyor."
      DUR
  BİTTİ

  TEKRAR
      YAZ "Ürün kategorileri: Elektronik, Giyim, Kitap, Ev..."
      YAZ "Bir kategori seçiniz:"
      KATEGORİ ← GİR()

      YAZ KATEGORİ + " kategorisindeki ürünler listeleniyor..."
      YAZ "Bir ürün seçiniz (ya da 'Çık' yazınız):"
      ÜRÜN ← GİR()

      EĞER ÜRÜN = "Çık" İSE
          ÇIK
      BİTTİ

      YAZ "Seçilen ürün stokta var mı?"
      STOK ← KONTROL_ET(ÜRÜN)

      EĞER STOK = 0 İSE
          YAZ "Ürün stokta yok."
      DEĞİLSE
          SEPETE_EKLE(ÜRÜN)
          YAZ "Ürün sepete eklendi."
      BİTTİ

      YAZ "Sepeti görüntülemek ister misiniz? (E/H)"
      CEVAP ← GİR()

      EĞER CEVAP = "E" İSE
          SEPETİ_GÖSTER()
          YAZ "Sepeti düzenlemek ister misiniz? (E/H)"
          DÜZENLE ← GİR()
          EĞER DÜZENLE = "E" İSE
              SEPETİ_DÜZENLE()
          BİTTİ
      BİTTİ

      YAZ "Başka kategoriye bakmak ister misiniz? (E/H)"
      DEVAM ← GİR()
  DEVAM = "H" OLANA KADAR

  # Ödeme Aşaması
  YAZ "İndirim kodunuz var mı? (E/H)"
  KOD_CEVAP ← GİR()
  EĞER KOD_CEVAP = "E" İSE
      İNDİRİM_KODU ← GİR()
      KODU_UYGULA(İNDİRİM_KODU)
  BİTTİ

  TOPLAM ← SEPET_TOPLAMI()

  EĞER TOPLAM < 50 İSE
      YAZ "Minimum alışveriş tutarı 50 TL olmalıdır."
      DUR
  BİTTİ

  EĞER TOPLAM >= 200 İSE
      KARGO_UCRETİ ← 0
  DEĞİLSE
      KARGO_UCRETİ ← 20
  BİTTİ

  YAZ "Ödeme yöntemi seçiniz (Kredi Kartı / Havale):"
  ÖDEME ← GİR()

  EĞER ÖDEME = "Kredi Kartı" VEYA "Havale" İSE
      SİPARİŞ_ONAYLA()
      YAZ "Siparişiniz onaylandı!"
  DEĞİLSE
      YAZ "Geçersiz ödeme yöntemi."
  BİTTİ

DUR

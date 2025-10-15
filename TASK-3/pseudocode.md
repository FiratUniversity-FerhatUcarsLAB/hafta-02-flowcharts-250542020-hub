BAŞLA

  TEKRAR
      YAZ "------------------------------------------"
      YAZ "  HASTANE RANDEVU VE TAHLİL SİSTEMİ  "
      YAZ "------------------------------------------"

      # 1. HASTA KİMLİK DOĞRULAMA
      YAZ "Lütfen TC Kimlik Numaranızı giriniz:"
      TC ← GİR()

      EĞER TC_UZUNLUK(TC) != 11 İSE
          YAZ "Geçersiz TC numarası! Lütfen 11 haneli doğru bir numara giriniz."
          DEVAM_ET
      BİTTİ

      # 2. ANA MENÜ
      YAZ "Lütfen yapmak istediğiniz işlemi seçiniz:"
      YAZ "1 - Randevu Al"
      YAZ "2 - Tahlil Sonucu Gör"
      YAZ "3 - Çıkış"
      SECIM ← GİR()

      # 3. RANDEVU MODÜLÜ
      EĞER SECIM = 1 İSE
          YAZ "------------------------------------------"
          YAZ "             RANDEVU MODÜLÜ"
          YAZ "------------------------------------------"

          YAZ "Randevu almak istediğiniz polikliniği seçiniz:"
          POLIKLINIK ← GİR()

          YAZ POLIKLINIK + " polikliniğindeki doktorlar listeleniyor..."
          DOKTOR_LISTESINI_GÖSTER(POLIKLINIK)

          YAZ "Bir doktor seçiniz:"
          DOKTOR ← GİR()

          TEKRAR
              YAZ "Uygun randevu saatleri:"
              SAATLERI_GÖSTER(DOKTOR)

              YAZ "Randevu saatinizi seçiniz:"
              SAAT ← GİR()

              EĞER SAAT_MUSAIT(DOKTOR, SAAT) İSE
                  RANDEVU_OLUSTUR(TC, DOKTOR, SAAT)
                  YAZ "Randevunuz başarıyla oluşturuldu."
                  YAZ "Randevu Bilgileri:"
                  YAZ "Poliklinik: " + POLIKLINIK
                  YAZ "Doktor: " + DOKTOR
                  YAZ "Saat: " + SAAT
                  SMS_GÖNDER(TC, DOKTOR, SAAT)
                  YAZ "Bilgilendirme SMS’i gönderildi."
                  ÇIK
              DEĞİLSE
                  YAZ "Seçilen saat dolu. Lütfen başka bir saat deneyiniz."
              BİTTİ
          SONSUZA_KADAR_TEKRAR

      # 4. TAHLİL MODÜLÜ
      DEĞİLSE EĞER SECIM = 2 İSE
          YAZ "------------------------------------------"
          YAZ "             TAHLİL MODÜLÜ"
          YAZ "------------------------------------------"

          EĞER TAHLIL_VAR_MI(TC) = HAYIR İSE
              YAZ "Sistemde kayıtlı bir tahlil bulunamadı."
              DEVAM_ET
          BİTTİ

          EĞER TAHLIL_HAZIR_MI(TC) = EVET İSE
              YAZ "Tahlil sonuçları hazır!"
              TAHLIL_SONUCU_GÖSTER(TC)

              YAZ "Sonuçları PDF olarak indirmek ister misiniz? (E/H)"
              CEVAP ← GİR()

              EĞER CEVAP = "E" İSE
                  PDF_OLUSTUR(TC)
                  YAZ "Tahlil sonuçları PDF olarak indirildi."
              DEĞİLSE
                  YAZ "PDF indirilmeyecek. Sonuçları sadece ekranda görüntülüyorsunuz."
              BİTTİ

          DEĞİLSE
              YAZ "Tahlil sonuçları henüz hazır değil. Lütfen daha sonra tekrar deneyiniz."
          BİTTİ

      # 5. ÇIKIŞ
      DEĞİLSE EĞER SECIM = 3 İSE
          YAZ "Sistemden çıkılıyor..."
          ÇIK

      # 6. GEÇERSİZ SEÇİM DURUMU
      DEĞİLSE
          YAZ "Geçersiz seçim! Lütfen menüden 1, 2 veya 3 giriniz."
      BİTTİ

      # 7. DÖNGÜ - BAŞKA İŞLEM
      YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
      DEVAM ← GİR()

  DEVAM = "E" OLDUĞU SÜRECE TEKRAR ET

  YAZ "------------------------------------------"
  YAZ "Sistemden çıkış yapıldı. İyi günler dileriz."
  YAZ "------------------------------------------"

DUR

BAŞLA

  # --- ÖĞRENCİ GİRİŞİ ---
  YAZ "Öğrenci numaranızı giriniz:"
  OGR_NO ← GİR()
  YAZ "Şifrenizi giriniz:"
  SIFRE ← GİR()

  EĞER NOT KIMLIK_DOGRULA(OGR_NO, SIFRE) İSE
      YAZ "Giriş başarılı. Hoşgeldiniz, Öğrenci " + OGR_NO
  DEĞİLSE
      YAZ "Giriş başarısız. Lütfen bilgilerinizi kontrol edin."
      DUR
  BİTTİ

  # Başlangıç verileri
  SEÇİLEN_DERSLER ← boş liste
  TOPLAM_KREDI ← 0

  # --- DERS LİSTESİNİ GÖRÜNTÜLEME VE SEÇİM DÖNGÜSÜ ---
  TEKRAR
      YAZ "Ders listesini görüntülemek için 'L', ders eklemek için 'E', ders çıkarmak için 'C', bitirmek için 'B' giriniz:"
      ISLEM ← GİR()

      EĞER ISLEM = "L" İSE
          DERS_LISTESI_GOSTER()
      DEĞİLSE EĞER ISLEM = "E" İSE
          YAZ "Eklemek istediğiniz ders kodunu giriniz:"
          DERS_KODU ← GİR()
          DERS ← DERS_BUL(DERS_KODU)
          
          EĞER DERS = YOK İSE
              YAZ "Geçersiz ders kodu!"
              DEVAM_ET
          BİTTİ

          # --- 1) KONTENJAN KONTROLÜ ---
          EĞER DERS.KONTENJAN_DOLU_MU() İSE
              YAZ "Bu dersin kontenjanı dolu."
              DEVAM_ET
          BİTTİ

          # --- 2) ÖNKOŞUL KONTROLÜ ---
          EĞER DERS.ONKOSUL_VAR_MI() İSE
              GEREKENLER ← DERS.ONKOSULLERI()
              EĞER ÖĞRENCİ_ONKOŞUL_TAMAMLAMIŞ_MI(OGR_NO, GEREKENLER) = HAYIR İSE
                  YAZ "Bu dersi almak için önkoşul dersleri tamamlamanız gerekir: " + GEREKENLER
                  DEVAM_ET
              BİTTİ
          BİTTİ

          # --- 3) ZAMAN ÇAKIŞMASI KONTROLÜ ---
          EĞER ZAMAN_CAKISMA_VAR_MI(SEÇİLEN_DERSLER, DERS) İSE
              YAZ "Seçilen ders, mevcut kayıtlarınızla zaman çakışıyor."
              DEVAM_ET
          BİTTİ

          # --- 4) KREDİ LİMİTİ KONTROLÜ ---
          YENI_TOPLAM ← TOPLAM_KREDI + DERS.KREDI
          EĞER YENI_TOPLAM > 35 İSE
              YAZ "Kredi limiti aşılıyor. Mevcut kredi: " + TOPLAM_KREDI + ", ders kredisi: " + DERS.KREDI
              DEVAM_ET
          BİTTİ

          # --- 5) DANIŞMAN ONAYI KONTROLÜ ---
          GPA ← ÖĞRENCİ_GPA(OGR_NO)
          EĞER GPA < 2.5 VE DANIŞMAN_ONAYI_GEREKLI_MI(DERS) = EVET İSE
              YAZ "Bu dersi alabilmek için danışman onayı gerekmektedir. Danışman onayı alındı mı? (E/H)"
              ONAY ← GİR()
              EĞER ONAY != "E" İSE
                  YAZ "Danışman onayı alınamadı. Ders eklenemedi."
                  DEVAM_ET
              BİTTİ
          BİTTİ

          # Tüm kontroller geçildiyse dersi ekle
          SEÇİLEN_DERSLER.EKLE(DERS)
          TOPLAM_KREDI ← YENI_TOPLAM
          YAZ "Ders başarıyla eklendi. Güncel toplam kredi: " + TOPLAM_KREDI

      DEĞİLSE EĞER ISLEM = "C" İSE
          YAZ "Çıkarmak istediğiniz ders kodunu giriniz:"
          DERS_KODU ← GİR()
          EĞER SEÇİLEN_DERSLER.İçeriyorMu(DERS_KODU) İSE
              DERS ← SEÇİLEN_DERSLER.Çıkar(DERS_KODU)
              TOPLAM_KREDI ← TOPLAM_KREDI - DERS.KREDI
              YAZ "Ders çıkarıldı. Güncel toplam kredi: " + TOPLAM_KREDI
          DEĞİLSE
              YAZ "Sepetinizde böyle bir ders yok."
          BİTTİ

      DEĞİLSE EĞER ISLEM = "B" İSE
          # Kayıt özetine geç
          ÇIK
      DEĞİLSE
          YAZ "Geçersiz işlem seçimi. Lütfen tekrar deneyiniz."
      BİTTİ

  SONSUZA_KADAR_TEKRAR

  # --- KAYIT ÖZETİ VE ONAY ---
  YAZ "Seçilen dersleriniz:"
  DERS_ÖZETİ_GÖSTER(SEÇİLEN_DERSLER)
  YAZ "Toplam kredi: " + TOPLAM_KREDI

  YAZ "Kayıtları onaylıyor musunuz? (E/H)"
  ONAY ← GİR()

  EĞER ONAY = "E" İSE
      # Son bir tutarlılık kontrolü: kontenjan ve zaman çakışması tekrar kontrolü, idempotent işlem
      HER DERS İÇİN D IN SEÇİLEN_DERSLER YAP
          EĞER D.KONTENJAN_DOLU_MU() İSE
              YAZ "Üzgünüz, işlem sırasında " + D.KODU + " dersinin kontenjanı doldu. Kayıt yapılamadı."
              DEVAM_ET
          BİTTİ
          EĞER ZAMAN_CAKISMA_VAR_MI(SEÇİLEN_DERSLER, D) İSE
              YAZ "İşlem sırasında zaman çakışması tespit edildi: " + D.KODU
              DEVAM_ET
          BİTTİ
      BITIR

      # Tüm dersleri üniversite kayıt sistemine ekle
      HER DERS İÇİN D IN SEÇİLEN_DERSLER YAP
          KAYIT_EKLE(OGR_NO, D)
          D.KONTENJAN_AZALT(1)
      BITIR

      YAZ "Ders kaydınız başarıyla tamamlandı."
  DEĞİLSE
      YAZ "Kayıt onaylanmadı. İşlem iptal edildi."
  BİTTİ

  YAZ "Sistemden çıkış yapılıyor. İyi günler."
DUR

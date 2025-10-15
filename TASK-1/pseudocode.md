250542020 
Ahmet Enes Altun

BAŞLA

  YAZ "Kartınızı takınız."
  KART_TAKILDI ← DOĞRU

  PIN_DENEME ← 0
  MAKS_DENEME ← 3

  TEKRAR
      YAZ "Lütfen PIN kodunuzu giriniz:"
      GİRİLEN_PIN ← GİR()
      
      EĞER GİRİLEN_PIN = DOĞRU_PIN İSE
          YAZ "PIN doğru."
          ÇIK
      DEĞİLSE
          PIN_DENEME ← PIN_DENEME + 1
          YAZ "Hatalı PIN. Kalan deneme hakkı: " + (MAKS_DENEME - PIN_DENEME)
      BİTTİ
  PIN_DENEME = MAKS_DENEME OLANA KADAR

  EĞER PIN_DENEME = MAKS_DENEME İSE
      YAZ "Kart bloke oldu."
      KARTI_İADE_ET()
      DUR
  BİTTİ

  TEKRAR
      YAZ "Bakiyeniz: " + BAKİYE
      YAZ "Çekmek istediğiniz tutarı giriniz:"
      TUTAR ← GİR()

      EĞER TUTAR % 20 ≠ 0 İSE
          YAZ "Tutar 20 TL'nin katı olmalıdır."
      DEĞİLSE EĞER TUTAR > BAKİYE İSE
          YAZ "Yetersiz bakiye."
      DEĞİLSE EĞER TUTAR > GÜNLÜK_LİMİT İSE
          YAZ "Günlük limit aşıldı."
      DEĞİLSE
          BAKİYE ← BAKİYE - TUTAR
          GÜNLÜK_LİMİT ← GÜNLÜK_LİMİT - TUTAR
          PARA_VER(TUTAR)
          FİŞ_YAZDIR()
          YAZ "Lütfen paranızı alınız."
      BİTTİ

      YAZ "Başka işlem yapmak istiyor musunuz? (E/H)"
      CEVAP ← GİR()
  CEVAP = "H" OLANA KADAR

  KARTI_İADE_ET()
  YAZ "İyi günler dileriz."

DUR

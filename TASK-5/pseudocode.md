# ==============================
# AKILLI EV GÜVENLİK SİSTEMİ - TÜRKÇE PSEUDOCODE
# 7/24 çalışan ana döngü / sensör okuma / alarm & bildirim süreçleri
# ==============================

# Başlangıç / Durum Değişkenleri
SİSTEM_AKTİF ← DOĞRU                # Sistem aktif/pasif durumu
ALARM_AKTİF ← YANLIŞ                # Mevcut alarm durumu
ALARM_SEVİYESİ ← 0                  # 0 = yok, 1 = düşük, 2 = orta, 3 = yüksek
SON_BİLDİRİ_ZAMANI ← 0              # Bildirimlerin limitlenmesi için zaman damgası
BİLDİRİ_ARALIĞI ← 60                # Saniye; aynı tür bildirimlerin aralığı
EVSAHİBİ_EVDE ← EVSAHİBİ_DURUMU_KONTROL_ET()

# Sensör ve eşik ayarları
SENSÖRLER ← [HAREKET_SENSÖRÜ, KAPI_PENCERE_SENSÖRÜ, KAMERALAR, DİĞER_SENSÖRLER]
HAREKET_SENSÖR_AYARI ← "orta"
KAPI_PENCERE_KRİTİK ← DOĞRU
YANLIŞ_ALARM_TEYİT_SAYISI ← 2

# Yardımcı fonksiyonlar
fonksiyon BİLDİRİ_GÖNDER(türü, mesaj):
    # tür ∈ {SMS, UYGULAMA, E-POSTA}
    çağrı ilgili servise(türü, mesaj)
    DÖN

fonksiyon KAMERA_AKTİF_ET(durum, kayıtSüresi):
    eğer durum = DOĞRU ise
        KAMERA.BAŞLAT_KAYIT(kayıtSüresi)
        FOTOĞRAF_AL()
    aksi halde
        KAMERA.DURDUR()
    DÖN

fonksiyon TEHDİT_DEĞERLENDİR(hareket, kapıAçılma, diğer):
    eğer kapıAçılma = DOĞRU ise
        DÖN 3   # yüksek
    eğer hareket = DOĞRU ve diğer = DOĞRU ise
        DÖN 3
    eğer hareket = DOĞRU ise
        DÖN 2   # orta
    eğer diğer = DOĞRU ise
        DÖN 1   # düşük
    DÖN 0

fonksiyon YANLIŞ_ALARM_MI(evdeMi, sonManuelSıfırlama):
    eğer evdeMi = DOĞRU ise
        DÖN DOĞRU
    eğer sonManuelSıfırlama < 120 saniye ise
        DÖN DOĞRU
    DÖN YANLIŞ

fonksiyon BİLDİRİ_SINIRLA(mesaj):
    şimdi ← ŞİMDİ()
    eğer şimdi - SON_BİLDİRİ_ZAMANI ≥ BİLDİRİ_ARALIĞI ise
        BİLDİRİ_GÖNDER("SMS", mesaj)
        BİLDİRİ_GÖNDER("UYGULAMA", mesaj)
        BİLDİRİ_GÖNDER("E-POSTA", mesaj)
        SON_BİLDİRİ_ZAMANI ← şimdi
    aksi halde
        BİLDİRİ_GÖNDER("UYGULAMA", mesaj)
    DÖN

fonksiyon ALARM_YÖNET(alarms):
    eğer alarms = 1 ise
        KAMERA_AKTİF_ET(DOĞRU, 30)
        BİLDİRİ_SINIRLA("Düşük seviye tehdit algılandı.")
    eğer alarms = 2 ise
        KAMERA_AKTİF_ET(DOĞRU, 120)
        BİLDİRİ_SINIRLA("Orta seviye tehdit! Kontrol ediniz.")
    eğer alarms = 3 ise
        KAMERA_AKTİF_ET(DOĞRU, 300)
        BİLDİRİ_SINIRLA("Yüksek seviye tehdit! Yetkililere bildiriliyor.")
        POLİS_BİLDİR("Yüksek alarm: Konum ... , Zaman ...")
    DÖN

# ==============================
# ANA DÖNGÜ (sürekli)
# ==============================
DO WHILE DOĞRU ve SİSTEM_AKTİF:
    # Sensörleri oku
    sensör_verileri ← BOŞ_SÖZLÜK()
    FOR her s IN SENSÖRLER:
        sensör_verileri[s] ← s.OKU()

    # Kritik sensörler
    hareket_algılandı ← sensör_verileri[HAREKET_SENSÖRÜ]
    kapı_pencere_açıldı ← sensör_verileri[KAPI_PENCERE_SENSÖRÜ]
    diğer_tehlike ← diğer_sensörleri_kontrol_et(sensör_verileri)

    # Yanlış alarm kontrolü
    EVSAHİBİ_EVDE ← EVSAHİBİ_DURUMU_KONTROL_ET()
    sonManuelSıfırlama ← SON_MANUEL_SIFIRLAMA_KONTROL_ET()

    eğer YANLIŞ_ALARM_MI(EVSAHİBİ_EVDE, sonManuelSıfırlama) = DOĞRU ise
        teyit ← 0
        i ← 0
        while i < YANLIŞ_ALARM_TEYİT_SAYISI:
            BEKLE(2)
            tekrar_hareket ← HAREKET_SENSÖRÜ.OKU()
            tekrar_kapı ← KAPI_PENCERE_SENSÖRÜ.OKU()
            eğer tekrar_hareket = DOĞRU veya tekrar_kapı = DOĞRU ise
                teyit ← teyit + 1
            i ← i + 1
        if teyit < YANLIŞ_ALARM_TEYİT_SAYISI:
            ALARM_AKTİF ← YANLIŞ
            ALARM_SEVİYESİ ← 0
            LOG("Yanlış alarm önlendi.")
            CONTINUE

    # Tehdit seviyesini belirle
    yeni_alarm_seviyesi ← TEHDİT_DEĞERLENDİR(hareket_algılandı, kapı_pencere_açıldı, diğer_tehlike)

    # Alarm durumu
    eğer yeni_alarm_seviyesi > 0:
        ALARM_AKTİF ← DOĞRU
        eğer yeni_alarm_seviyesi > ALARM_SEVİYESİ:
            ALARM_SEVİYESİ ← yeni_alarm_seviyesi
            LOG("Alarm seviyesi güncellendi: " + ALARM_SEVİYESİ)
            ALARM_YÖNET(ALARM_SEVİYESİ)
        aksi halde
            LOG("Tehdit algılandı, mevcut seviye: " + ALARM_SEVİYESİ)
            eğer ALARM_SEVİYESİ = 0:
                ALARM_SEVİYESİ ← yeni_alarm_seviyesi
                ALARM_YÖNET(ALARM_SEVİYESİ)

        # Alarm aktifken izleme döngüsü
        while ALARM_AKTİF = DOĞRU:
            BEKLE(5)
            hareket_algılandı ← HAREKET_SENSÖRÜ.OKU()
            kapı_pencere_açıldı ← KAPI_PENCERE_SENSÖRÜ.OKU()
            diğer_tehlike ← diğer_sensörleri_kontrol_et(sensör_verileri)

            if hareket_algılandı = YANLIŞ ve kapı_pencere_açıldı = YANLIŞ ve diğer_tehlike = YANLIŞ:
                temiz_onay ← 0
                j ← 0
                while j < 3:
                    BEKLE(2)
                    if HAREKET_SENSÖRÜ.OKU() = YANLIŞ ve KAPI_PENCERE_SENSÖRÜ.OKU() = YANLIŞ:
                        temiz_onay ← temiz_onay + 1
                    j ← j + 1
                if temiz_onay = 3:
                    ALARM_AKTİF ← YANLIŞ
                    ALARM_SEVİYESİ ← 0
                    KAMERA_AKTİF_ET(YANLIŞ, 0)
                    BİLDİRİ_SINIRLA("Alarm durumu sona erdi (otomatik).")
                    LOG("Alarm otomatik sıfırlandı.")
                    BREAK

            if ALARM_SIFIRLA_KOMUTU_ALINDI() = DOĞRU:
                ALARM_AKTİF ← YANLIŞ
                ALARM_SEVİYESİ ← 0
                KAMERA_AKTİF_ET(YANLIŞ, 0)
                BİLDİRİ_SINIRLA("Alarm manuel olarak sıfırlandı.")
                LOG("Alarm manuel sıfırlandı.")
                BREAK

    aksi halde:
        ALARM_AKTİF ← YANLIŞ
        ALARM_SEVİYESİ ← 0
        if SİSTEM_SAĞLIK_KONTROLÜ_HATA():
            LOG("Sistem sağlık kontrolü başarısız.")
            BİLDİRİ_SINIRLA("Sistem sağlık kontrolünde problem tespit edildi.")
        BEKLE(2)

    LOG_DURUMU_PERİYODİK_KAYDET()
END DO

# Sistem kapatma
if SİSTEM_AKTİF = YANLIŞ:
    KAMERA_AKTİF_ET(YANLIŞ, 0)
    LOG("Sistem devre dışı bırakıldı.")
    BİLDİRİ_GÖNDER("UYGULAMA", "G

BASLA: AKILLI_EV_GÜVENLİK_SİSTEMİ

// -----------------------------------------------------
// I. SİSTEM BAŞLANGICI VE KURULUM
// -----------------------------------------------------

BASLA: SİSTEM_KURULUMU
    Sistem_Durumu = "AKTIF" // Güvenlik sistemi çalışır durumda
    Siren_Durumu = "KAPALI"
    Bildirim_Adresleri = VERITABANINDAN_AL("Yetkili_Kullanıcılar_Eposta_SMS")
    EKRANA_YAZ("Akıllı Ev Güvenlik Sistemi Başlatıldı. 7/24 İzleme Aktif.")
BITIR: SİSTEM_KURULUMU

// -----------------------------------------------------
// II. ANA GÜVENLİK DÖNGÜSÜ (7/24 Çalışma)
// -----------------------------------------------------

BASLA: ANA_GÜVENLİK_DÖNGÜSÜ
    
    // Sürekli Çalışma Döngüsü
    DONGU: Sistem_Durumu == "AKTIF" IKEN
        
        // 1. Sensör Okuma ve Veri Toplama
        BASLA: SENSÖR_OKUMA
            // Her döngüde tüm sensörlerden (kapı, pencere, hareket, duman vb.) veri al
            Sensör_Verisi = SENSÖR_VERİLERİNİ_TOPLA()
            SICAKLIK = Sensör_Verisi.OrtamSicakligi
            KAPI_DURUMU = Sensör_Verisi.KapiAcik
            HAREKET_ALGILANDI = Sensör_Verisi.HareketAlgilama
        BITIR: SENSÖR_OKUMA

        // 2. Tehdit Algılama ve Kontroller
        BASLA: TEHDIT_ALGILAMA
            Tehdit_Seviyesi = "GÜVENLİ" // Varsayılan durum

            // KONTROL NOKTASI 1: İzinsiz Giriş (Kapı/Pencere)
            EGER KAPI_DURUMU == "AÇIK" VE Sistem_Durumu == "AKTIF" VE ZAMAN_KONTROL(GECE_SAATLERI) ISE
                Tehdit_Seviyesi = "YÜKSEK_GIRIS"
            
            // KONTROL NOKTASI 2: Hareket Algılama
            ELSE_EGER HAREKET_ALGILANDI == EVET VE Sistem_Durumu == "AKTIF" ISE
                Tehdit_Seviyesi = "ORTA_HAREKET"
            
            // KONTROL NOKTASI 3: Yangın/Duman Algılama (Ek Kontrol)
            ELSE_EGER SICAKLIK >= 60 VEYA SENSÖR_VERISI.DumanAlgilama == EVET ISE
                 Tehdit_Seviyesi = "YANGIN_ACİL"
            BITIR
        BITIR: TEHDIT_ALGILAMA

        // 3. Alarm ve Bildirim Eylemleri
        BASLA: EYLEM_KARARLARI
            
            EGER Tehdit_Seviyesi == "YÜKSEK_GIRIS" VEYA Tehdit_Seviyesi == "YANGIN_ACİL" ISE
                
                // KONTROL NOKTASI 4: Siren Aktivasyonu
                EGER Siren_Durumu == "KAPALI" ISE
                    Siren_DURUMU = "ACIK"
                    EKRANA_YAZ("KIRMIZI ALARM! Siren devreye alındı.")
                    SIREN_AC()
                BITIR
                
                // KONTROL NOKTASI 5: Bildirim Gönderimi
                EGER SON_BILDIRIM_ZAMANI(Bildirim_Adresleri) > 5_DAKIKA_ONCE ISE
                    // Spam yapmamak için bildirim sıklığını kontrol et
                    BILDIRIM_GONDER(Bildirim_Adresleri, "ACİL TEHLİKE: İzinsiz giriş algılandı!", "YÜKSEK")
                    SON_BILDIRIM_ZAMANI_GUNCELLE()
                BITIR
                
            ELSE_EGER Tehdit_Seviyesi == "ORTA_HAREKET" ISE
                // Sadece bildirim, alarm beklemede
                BILDIRIM_GONDER(Bildirim_Adresleri, "UYARI: Evde hareket algılandı.", "ORTA")
                
            ELSE // Tehdit_Seviyesi == "GÜVENLİ"
                EGER Siren_Durumu == "ACIK" ISE
                    // Siren durumu manuel olarak KAPALI yapılana kadar çalar.
                    EKRANA_YAZ("Alarm aktif.")
                ELSE
                    EKRANA_YAZ("Sistem güvenli. Beklemede.")
                BITIR
            BITIR
        BITIR: EYLEM_KARARLARI

        // Bir sonraki kontrol için kısa bekleme süresi
        BEKLE(5_SANIYE) 
        
    BITIR: DONGU
    
    // Sistem manuel olarak devre dışı bırakıldığında döngüden çıkar
    EKRANA_YAZ("Güvenlik Sistemi Devre Dışı Bırakıldı.")

BITIR: ANA_GÜVENLİK_DÖNGÜSÜ

GIT: SISTEM_KAPAT

BASLA: SISTEM_KAPAT
    EKRANA_YAZ("Akıllı Ev Güvenlik Sistemi Kapatılıyor.")
BITIR: SISTEM_KAPAT

BITIR: AKILLI_EV_GÜVENLİK_SİSTEMİ

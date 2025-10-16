BASLA: HASTANE_MERKEZ_SISTEMI_ALGORITMASI

// -----------------------------------------------------
// I. MERKEZI KIMLIK DOGRULAMA
// -----------------------------------------------------

BASLA: MERKEZI_KIMLIK_DOGRULAMA
    DONGU: Dogrulama_Basarili_Olana_Kadar
        TC_Kimlik_No = KULLANICIDAN_AL("TC Kimlik Numarası:")
        Dogum_Tarihi = KULLANICIDAN_AL("Doğum Tarihi:")
        
        // KONTROL NOKTASI 1: Kimlik Bilgileri Kontrolü
        EGER KIMLIK_BILGILERI_KONTROL_ET(TC_Kimlik_No, Dogum_Tarihi) ISE
            Kullanici_ID = TC_Kimlik_No
            EKRANA_YAZ("Kimlik doğrulama başarılı.")
            DONGU_KIR
        ELSE
            HATA_SAYACI = HATA_SAYACI + 1
            EKRANA_YAZ("Hatalı bilgi. Lütfen tekrar deneyin.")
            EGER HATA_SAYACI >= 3 ISE
                EKRANA_YAZ("Çok sayıda başarısız deneme. Sistemden çıkılıyor.")
                GIT: SISTEM_KAPAT
            BITIR
        BITIR
    BITIR: DONGU
BITIR: MERKEZI_KIMLIK_DOGRULAMA

// -----------------------------------------------------
// II. ANA MENU VE MODUL SECIMI
// -----------------------------------------------------

BASLA: ANA_MENU_SECIM
    DONGU: Kullanici_Cikis_Yapana_Kadar
        EKRANA_GOSTER("\n--- HASTANE ANA MENÜ ---")
        EKRANA_GOSTER("1: Randevu Alma Sistemi")
        EKRANA_GOSTER("2: Tahlil Sonucu Görüntüleme")
        EKRANA_GOSTER("3: Çıkış")
        Menu_Secimi = KULLANICIDAN_AL("Lütfen yapmak istediğiniz işlemi seçiniz:")
        
        // KONTROL NOKTASI 2: Menü Seçimi Yönlendirmesi
        EGER Menu_Secimi == "1" ISE
            CALISTIR: RANDEVU_ALMA_MODULU(Kullanici_ID)
        ELSE_EGER Menu_Secimi == "2" ISE
            CALISTIR: TAHLIL_GORUNTULEME_MODULU(Kullanici_ID)
        ELSE_EGER Menu_Secimi == "3" ISE
            DONGU_KIR // Çıkış
        ELSE
            EKRANA_YAZ("Geçersiz seçim. Lütfen 1, 2 veya 3 giriniz.")
        BITIR
    BITIR: DONGU
BITIR: ANA_MENU_SECIM

// -----------------------------------------------------
// III. MODÜLLER (Ayrı fonksiyonlar olarak çağrılır)
// -----------------------------------------------------

// Not: Bu kısım, daha önce yazılan iki ayrı pseudocode'un içeriğini temsil eder.

// BASLA: RANDEVU_ALMA_MODULU(Kullanici_ID)
//     ... (Poliklinik Seçimi, Doktor Seçimi, Saat Görüntüleme, Onay, SMS adımları buraya gelir) ...
// BITIR: RANDEVU_ALMA_MODULU

// BASLA: TAHLIL_GORUNTULEME_MODULU(Kullanici_ID)
//     ... (Tahlil Listesi, Sonuç Kontrolü, Görüntüleme, PDF İndirme adımları buraya gelir) ...
// BITIR: TAHLIL_GORUNTULEME_MODULU


// -----------------------------------------------------
// IV. SISTEM KAPANISI
// -----------------------------------------------------

BASLA: SISTEM_KAPAT
    EKRANA_YAZ("Sistemden çıkış yapılıyor. Bizi tercih ettiğiniz için teşekkürler.")
BITIR: SISTEM_KAPAT

GIT: SISTEM_KAPAT

BITIR: HASTANE_MERKEZ_SISTEMI_ALGORITMASI

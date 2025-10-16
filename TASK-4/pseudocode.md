BASLA: UNIVERSITE_DERS_KAYIT_SISTEMI

// -----------------------------------------------------
// I. KIMLIK DOGRULAMA VE GIRIS
// -----------------------------------------------------

BASLA: OGRENCI_GIRISI
    DONGU: Giris_Basarili_Olana_Kadar
        Ogrenci_No = KULLANICIDAN_AL("Öğrenci Numarası:")
        Sifre = KULLANICIDAN_AL("Şifre:")
        
        // KONTROL NOKTASI A: Kullanıcı Bilgileri
        EGER VERITABANI_KONTROL(Ogrenci_No, Sifre) ISE
            // KONTROL NOKTASI B: Harç/Maddi Yükümlülük
            EGER HARC_BORCU_VAR_MI(Ogrenci_No) ISE
                EKRANA_YAZ("Kaydınız için önce harç/öğrenim ücreti borcunuzu ödemeniz gerekmektedir.")
                GIT: SISTEM_KAPAT
            ELSE
                Kullanici_Oturumu = AC
                EKRANA_YAZ("Giriş başarılı.")
                DONGU_KIR
            BITIR
        ELSE
            EKRANA_YAZ("Hatalı kullanıcı adı veya şifre.")
        BITIR
    BITIR: DONGU
BITIR: OGRENCI_GIRISI

// -----------------------------------------------------
// II. DERS LISTESI GORUNTULEME VE SECIM
// -----------------------------------------------------

BASLA: DERS_SECIMI_VE_KONTROLLER
    Zorunlu_Dersler = VERITABANINDAN_AL("Zorunlu Dersler", Ogrenci_No)
    Secmeli_Dersler = VERITABANINDAN_AL("Seçmeli Havuzları", Ogrenci_No)
    Ders_Sepeti = BOS_LISTE
    Toplam_Kredi = 0
    
    EKRANA_GOSTER(Zorunlu_Dersler + Secmeli_Dersler)
    
    DONGU: Yeni_Ders_Secildigi_Sürece
        Secilen_Ders_Kodu = KULLANICIDAN_AL("Seçmek istediğiniz ders kodu:")
        Secilen_Sube = KULLANICIDAN_AL("Seçmek istediğiniz şube:")
        
        Ders_Bilgisi = DERS_VERITABANINDAN_AL(Secilen_Ders_Kodu, Secilen_Sube)
        
        EGER Ders_Bilgisi BOS_DEGIL_ISE
            
            // KONTROL NOKTASI 1: Önkoşul Kontrolü
            EGER ONKOSUL_KONTROL(Ogrenci_No, Secilen_Ders_Kodu) ISE
            
                // KONTROL NOKTASI 2: Kontenjan Kontrolü
                EGER Ders_Bilgisi.Kontenjan > Ders_Bilgisi.Kayitli_Ogrenci_Sayisi ISE
                
                    // KONTROL NOKTASI 3: Kredi Limiti Kontrolü (Maksimum)
                    EGER (Toplam_Kredi + Ders_Bilgisi.Kredi) <= MAKSIMUM_KREDI_LIMITI(Ogrenci_No) ISE
                        
                        // KONTROL NOKTASI 4: Zaman Çakışması Kontrolü
                        EGER ZAMAN_CAKISMASI_VAR_MI(Ders_Sepeti, Ders_Bilgisi) ISE
                            EKRANA_YAZ("HATA: Seçtiğiniz ders, sepetinizdeki başka bir ders ile çakışmaktadır.")
                        ELSE
                            // Tüm Kontroller Başarılı
                            DERS_SEPETE_EKLE(Ders_Sepeti, Ders_Bilgisi)
                            Toplam_Kredi = Toplam_Kredi + Ders_Bilgisi.Kredi
                            EKRANA_YAZ(Secilen_Ders_Kodu + " başarıyla sepete eklendi.")
                        BITIR // Zaman Çakışması Kontrolü
                        
                    ELSE
                        EKRANA_YAZ("HATA: Maksimum kredi limitini (" + MAKSIMUM_KREDI_LIMITI(Ogrenci_No) + " kredi) aşıyorsunuz.")
                    BITIR // Kredi Limiti Kontrolü
                    
                ELSE
                    EKRANA_YAZ("HATA: Seçilen dersin/şubenin kontenjanı dolmuştur.")
                BITIR // Kontenjan Kontrolü
                
            ELSE
                EKRANA_YAZ("HATA: Seçilen dersin önkoşulunu sağlamıyorsunuz (Eksik/başarısız ders).")
            BITIR // Önkoşul Kontrolü
            
        ELSE
            EKRANA_YAZ("HATA: Geçersiz ders kodu veya şube.")
        BITIR
        
        EGER KULLANICIDAN_AL("Başka ders seçmek ister misiniz? (H/E)") == "H" ISE
            DONGU_KIR
        BITIR
    BITIR: DONGU
BITIR: DERS_SECIMI_VE_KONTROLLER

// -----------------------------------------------------
// III. KAYIT KESINLEŞTİRME VE ONAY
// -----------------------------------------------------

BASLA: KAYIT_KESINLESTIRME
    
    // KONTROL NOKTASI 5: Kredi Limiti Kontrolü (Minimum)
    EGER Toplam_Kredi < MINIMUM_KREDI_LIMITI(Ogrenci_No) ISE
        EKRANA_YAZ("HATA: Kaydınızı kesinleştirmek için minimum kredi limitini (" + MINIMUM_KREDI_LIMITI(Ogrenci_No) + " kredi) doldurmanız gerekmektedir.")
        GIT: DERS_SECIMI_VE_KONTROLLER
    BITIR
    
    EKRANA_GOSTER("Seçilen Dersler: " + Ders_Sepeti)
    Kullanici_Onayi = KULLANICIDAN_AL("Seçimlerinizi onaylıyor musunuz? (E/H)")
    
    EGER Kullanici_Onayi == "E" ISE
        KAYDI_GECICI_KAYDET(Ogrenci_No, Ders_Sepeti, "BEKLEMEDE")
        
        // KONTROL NOKTASI 6: Danışman Onayına Gönderme
        EKRANA_YAZ("Ders kaydınız, danışman onayı için gönderilmiştir.")
        
        // Arka Planda Danışman Onay Süreci Başlatılır
        SONUC = DANISMAN_ONAYI_BEKLE(Ogrenci_No)
        
        EGER SONUC == "ONAYLANDI" ISE
            KAYDI_KESINLESTIR(Ogrenci_No)
            EKRANA_YAZ("Ders kaydınız başarıyla tamamlanmıştır. Kayıt formunuz oluşturulmuştur.")
            PDF_FORMU_OLUSTUR(Ogrenci_No)
        ELSE_EGER SONUC == "REDDEDILDI" ISE
            EKRANA_YAZ("Ders kaydınız danışman tarafından REDDEDİLMİŞTİR. Lütfen danışmanınızla görüşerek düzeltin.")
            GIT: DERS_SECIMI_VE_KONTROLLER
        BITIR
        
    ELSE
        EKRANA_YAZ("Ders kayıt işlemi iptal edildi.")
        GIT: DERS_SECIMI_VE_KONTROLLER
    BITIR
BITIR: KAYIT_KESINLESTIRME

// -----------------------------------------------------
// IV. SISTEM KAPANISI
// -----------------------------------------------------

BASLA: SISTEM_KAPAT
    EKRANA_YAZ("Sistemden çıkış yapılıyor.")
BITIR: SISTEM_KAPAT

GIT: SISTEM_KAPAT

BITIR: UNIVERSITE_DERS_KAYIT_SISTEMI

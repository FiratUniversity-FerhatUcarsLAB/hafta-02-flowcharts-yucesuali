BASLA: E-TICARET_SEPET_ODEME_SISTEMI

// -----------------------------------------------------
// I. KULLANICI GIRISI VE URUN SECIMI
// -----------------------------------------------------

BASLA: KULLANICI_GIRISI_KONTROLU
    EKRANA_YAZ("Lütfen giriş yapın veya misafir olarak devam edin.")
    Kullanici_Secimi = KULLANICIDAN_AL()
    
    EGER Kullanici_Secimi == "Giris_Yap" ISE
        Kullanici_Adi = KULLANICIDAN_AL("Kullanıcı Adı:")
        Sifre = KULLANICIDAN_AL("Şifre:")
        
        EGER VERITABANI_KONTROL(Kullanici_Adi, Sifre) ISE
            Kullanici_Oturumu = AC
            EKRANA_YAZ("Giriş Başarılı. Hoş geldiniz!")
        ELSE
            EKRANA_YAZ("Hatalı kullanıcı adı veya şifre.")
            GIT: KULLANICI_GIRISI_KONTROLU // Tekrar dene
        BITIR
    ELSE_EGER Kullanici_Secimi == "Misafir_Devam" ISE
        Kullanici_Oturumu = MISAFIR
        EKRANA_YAZ("Misafir olarak devam ediliyor.")
    ELSE
        GIT: KULLANICI_GIRISI_KONTROLU
    BITIR
BITIR: KULLANICI_GIRISI_KONTROLU

BASLA: URUN_EKLEME_VE_STOK_KONTROLU
    DONGU: Kullanıcı_Alışverişe_Devam_Ediyor_Iken
        Urun_ID = KULLANICIDAN_AL("Sepete eklenecek ürün ID'sini girin:")
        Adet = KULLANICIDAN_AL("Eklenecek adeti girin:")
        
        // KONTROL NOKTASI 1: Anlık Stok Kontrolü
        EGER STOK_VERITABANI_KONTROL(Urun_ID) >= Adet ISE
            SEPETE_EKLE(Urun_ID, Adet)
            EKRANA_YAZ(Urun_ID + " ürünü sepetinize eklendi.")
        ELSE
            EKRANA_YAZ("Üzgünüz, stokta yeterli ürün yok. Mevcut stok: " + STOK_VERITABANI_KONTROL(Urun_ID))
        BITIR
        
        Secim = KULLANICIDAN_AL("Başka ürün eklemek ister misiniz? (E/H)")
        EGER Secim == "H" ISE
            DONGU_KIR
        BITIR
    BITIR: DONGU
BITIR: URUN_EKLEME_VE_STOK_KONTROLU

// -----------------------------------------------------
// II. SEPET VE INDIRIM UYGULAMA
// -----------------------------------------------------

BASLA: SEPET_OZET_VE_INDIM
    TOPLAM_TUTAR = SEPET_TUTARINI_HESAPLA()
    EKRANA_YAZ("Sepet Toplamı: " + TOPLAM_TUTAR)
    
    Indirim_Kodu = KULLANICIDAN_AL("İndirim kodunuz varsa giriniz (Yoksa 'H'):")
    
    EGER Indirim_Kodu != "H" ISE
        // KONTROL NOKTASI 2: İndirim Kodu Geçerliliği
        EGER INDIRIM_KODU_GECERLI_MI(Indirim_Kodu, TOPLAM_TUTAR, SEPET_ICERIGI) ISE
            Indirim_Miktari = INDIRIMI_UYGULA(TOPLAM_TUTAR, Indirim_Kodu)
            TOPLAM_TUTAR = TOPLAM_TUTAR - Indirim_Miktari
            EKRANA_YAZ("İndirim Uygulandı! Yeni Toplam: " + TOPLAM_TUTAR)
        ELSE
            EKRANA_YAZ("Geçersiz veya kullanılamaz indirim kodu.")
        BITIR
    BITIR
BITIR: SEPET_OZET_VE_INDIM

// -----------------------------------------------------
// III. KARGO HESAPLAMA VE ADRES
// -----------------------------------------------------

BASLA: ADRES_VE_KARGO_HESAPLAMA
    EGER Kullanici_Oturumu == MISAFIR ISE
        Gonderim_Adresi = KULLANICIDAN_ADRES_AL()
    ELSE
        Gonderim_Adresi = KULLANICIDAN_MEVCUT_ADRES_SEC()
    BITIR

    // KONTROL NOKTASI 3: Kargo Ücreti ve Seçenekleri
    Kargo_Ucreti = KARGO_UCRETI_HESAPLA(Gonderim_Adresi, TOPLAM_TUTAR)
    EKRANA_YAZ("Kargo Ücreti: " + Kargo_Ucreti)
    
    EGER Kargo_Ucreti == 0 ISE
        EKRANA_YAZ("Kargo ücretsiz!")
    BITIR
    
    NIHAI_ODENECEK_TUTAR = TOPLAM_TUTAR + Kargo_Ucreti
    EKRANA_YAZ("Ödenecek Nihai Tutar: " + NIHAI_ODENECEK_TUTAR)
BITIR: ADRES_VE_KARGO_HESAPLAMA

// -----------------------------------------------------
// IV. ODEME ISLEMI VE ONAY
// -----------------------------------------------------

BASLA: ODEME_ASAMASI
    EKRANA_YAZ("Ödeme Yöntemini Seçiniz (1: Kart, 2: Havale/EFT, 3: Kapıda Ödeme):")
    Odeme_Yontemi = KULLANICIDAN_AL()
    
    EGER Odeme_Yontemi == "1" ISE // Kredi/Banka Kartı
        Kart_Bilgileri = KULLANICIDAN_KART_BILGISI_AL()
        
        // KONTROL NOKTASI 4: Sanal POS İşlemi ve 3D Secure
        Odeme_Sonucu = SANAL_POS_ISLEMI_BASLAT(Kart_Bilgileri, NIHAI_ODENECEK_TUTAR)
        
        EGER Odeme_Sonucu == "BASARILI" ISE
            Islem_Durumu = "ODENDI"
        ELSE
            Islem_Durumu = "ODEME_HATASI"
        BITIR
        
    ELSE_EGER Odeme_Yontemi == "2" ISE // Havale/EFT
        EKRANA_YAZ("Havale/EFT bilgileri gösterildi. Ödeme bekleniyor.")
        Islem_Durumu = "ODEME_BEKLIYOR"
        
    ELSE_EGER Odeme_Yontemi == "3" ISE // Kapıda Ödeme
        // KONTROL NOKTASI 5: Kapıda Ödeme Kısıtlaması
        EGER Kapida_Odeme_Mevcut_Mu(Gonderim_Adresi) ISE
            Islem_Durumu = "KAPIDA_ODEME"
        ELSE
            EKRANA_YAZ("Bölgenize kapıda ödeme yapılamamaktadır. Lütfen farklı bir yöntem seçin.")
            GIT: ODEME_ASAMASI
        BITIR
        
    ELSE
        EKRANA_YAZ("Geçersiz ödeme yöntemi.")
        GIT: ODEME_ASAMASI
    BITIR
BITIR: ODEME_ASAMASI

// -----------------------------------------------------
// V. SIPARIS KAYIT VE SONLANDIRMA
// -----------------------------------------------------

BASLA: SIPARIS_KAYDI_VE_STOK_GUNCELLEME
    EGER Islem_Durumu == "ODENDI" VEYA Islem_Durumu == "KAPIDA_ODEME" VEYA Islem_Durumu == "ODEME_BEKLIYOR" ISE
        
        // KONTROL NOKTASI 6: Stokları Rezerve Etme/Düşme
        STOK_MIKTARLARINI_GUNCELLE(SEPET_ICERIGI, "-", Kullanici_ID)
        
        Siparis_ID = VERITABANINA_SIPARISI_KAYDET(
            Kullanici_ID, 
            SEPET_ICERIGI, 
            NIHAI_ODENECEK_TUTAR, 
            Islem_Durumu, 
            Gonderim_Adresi)
        
        // KONTROL NOKTASI 7: Müşteri Bilgilendirme
        EPOSTA_GONDER(Kullanici_Eposta, "Sipariş Onayı #" + Siparis_ID)
        
        EKRANA_YAZ("Siparişiniz başarıyla alındı! Sipariş No: " + Siparis_ID)
        
    ELSE_EGER Islem_Durumu == "ODEME_HATASI" ISE
        EKRANA_YAZ("Ödeme işlemi başarısız oldu. Lütfen bilgilerinizi kontrol edin.")
    BITIR
BITIR: SIPARIS_KAYDI_VE_STOK_GUNCELLEME

BITIR: E-TICARET_SEPET_ODEME_SISTEMI

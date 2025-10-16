BASLA

    SABITLER
        MAKS_DENEME = 3
        GUNLUK_LIMIT = 5000  // Örnek günlük limit
        MIN_CEKIM_KATI = 20

    DEGISKENLER
        DOGRU_PIN = "1234"      // Örnek doğru PIN
        HESAP_BAKIYESI = 15000  // Örnek bakiye
        GUNLUK_CEKILEN_TOPLAM = 0
        GIRIS_PIN = ""
        PIN_DENEME_SAYISI = 0
        CEKILECEK_TUTAR = 0
        ISLEM_TEKRAR = 'E'

    DONGU (ISLEM_TEKRAR == 'E' VEYA ISLEM_TEKRAR == 'e')

        // 1. PIN DOĞRULAMA (3 HAK)
        PIN_DONGUSU_DEVAM = DOGRU
        DONGU (PIN_DONGUSU_DEVAM == DOGRU)
            YAZ "Lütfen PIN kodunuzu girin:"
            OKU GIRIS_PIN

            EGER (GIRIS_PIN == DOGRU_PIN) ISE
                PIN_DONGUSU_DEVAM = YANLIS
                YAZ "PIN Doğrulama Başarılı."
            DEGILSE
                PIN_DENEME_SAYISI = PIN_DENEME_SAYISI + 1
                YAZ "Hatalı PIN. Kalan hak: ", (MAKS_DENEME - PIN_DENEME_SAYISI)

                EGER (PIN_DENEME_SAYISI >= MAKS_DENEME) ISE
                    YAZ "PIN blokesi. Sistem kapatılıyor."
                    GIT BITIR_ISLEM
                BITIR_EGER
            BITIR_EGER
        BITIR_DONGU

        // 2. PARA ÇEKME İŞLEMİ
        ISLEM_DONGUSU_DEVAM = DOGRU
        DONGU (ISLEM_DONGUSU_DEVAM == DOGRU)
            YAZ "Çekmek istediğiniz tutarı giriniz ( ", MIN_CEKIM_KATI, " TL ve katları, Günlük Limit: ", GUNLUK_LIMIT, " TL):"
            OKU CEKILECEK_TUTAR

            // 3. TUTAR KONTROLÜ (20 TL KATLARI)
            EGER (CEKILECEK_TUTAR MOD MIN_CEKIM_KATI == 0) ISE

                // 4. GÜNLÜK LİMİT KONTROLÜ
                EGER ((GUNLUK_CEKILEN_TOPLAM + CEKILECEK_TUTAR) <= GUNLUK_LIMIT) ISE

                    // 5. BAKİYE KONTROLÜ
                    EGER (CEKILECEK_TUTAR <= HESAP_BAKIYESI) ISE
                        
                        // İŞLEM BAŞARILI
                        HESAP_BAKIYESI = HESAP_BAKIYESI - CEKILECEK_TUTAR
                        GUNLUK_CEKILEN_TOPLAM = GUNLUK_CEKILEN_TOPLAM + CEKILECEK_TUTAR
                        YAZ "İşlem başarılı. Lütfen paranızı alınız: ", CEKILECEK_TUTAR, " TL"
                        YAZ "Kalan Bakiye: ", HESAP_BAKIYESI, " TL"
                        ISLEM_DONGUSU_DEVAM = YANLIS // İşlemi bitir

                    DEGILSE
                        YAZ "Yetersiz bakiye. Mevcut bakiye: ", HESAP_BAKIYESI, " TL"
                    BITIR_EGER

                DEGILSE
                    YAZ "Günlük para çekme limitinizi aşıyorsunuz."
                    YAZ "Bugün çekilen toplam: ", GUNLUK_CEKILEN_TOPLAM, " TL. Kalan limit: ", (GUNLUK_LIMIT - GUNLUK_CEKILEN_TOPLAM), " TL"
                BITIR_EGER

            DEGILSE
                YAZ "Hatalı tutar. Lütfen ", MIN_CEKIM_KATI, " TL'nin katlarında bir tutar giriniz."
            BITIR_EGER

        BITIR_DONGU

        // 6. İŞLEM TEKRARI SEÇENEĞİ
        YAZ "Başka bir işlem yapmak ister misiniz? (E/H)"
        OKU ISLEM_TEKRAR

    BITIR_DONGU

    GIT BITIR_ISLEM

    ETIKET BITIR_ISLEM
    YAZ "İşlemleriniz için teşekkür ederiz. İyi günler dileriz."

BITIR

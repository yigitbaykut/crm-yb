# Başvuru Hattı

Staj, rol ve araştırma başvurularını takip eden kişisel bir CRM. Gönderilmiş
maillerin hangisine cevap geldiğini, hangisinin kaç gündür sessiz olduğunu ve
hangi sürecin nerede takıldığını tek ekranda gösterir.

Yayınlanmış sayfa: https://claude.ai/code/artifact/339565ca-ad16-4bc2-aa80-8cd67651e6f8

## Nasıl çalışıyor

`index.html` tek dosyalık bir uygulama. Claude Artifact olarak yayınlandığında
üç runtime yeteneği kullanıyor:

| Yetenek  | Ne için |
|----------|---------|
| `db`     | Başvurular kalıcı olarak saklanır, her cihazdan aynı veri görünür |
| `mcp`    | Gmail bağlayıcısı üzerinden `search_threads` ile gönderilmiş mailler taranır |
| `sample` | Alan adından türetilen ham kurum adlarını Claude'a düzelttirmek için |

Üçü de isteğe bağlı: yetenek yoksa `claude.use()` `null` döner ve sayfa o
özelliği gizleyip geri kalanıyla çalışmaya devam eder.

## Gmail taraması

"Gmail'i tara" düğmesi kayıtlı sorguyu çalıştırır (en fazla 4 sayfa, 200 yazışma)
ve her yazışma için:

- `SENT` etiketi taşıyan ilk mesajı başvurunun kendisi sayar,
- `SENT` taşımayan mesaj varsa **cevap geldi** olarak işaretler,
- gönderen `mailer-daemon` / `postmaster` ise **adres bulunamadı** olarak işaretler,
- alıcının alan adı `.edu` / `.ac` ise türü **akademi**, değilse **şirket** yapar.

Kayıt kimliği Gmail thread id'sidir; aynı yazışma ikinci taramada tekrar
eklenmez, yalnızca cevap ve son hareket bilgisi tazelenir. Elle girilen kurum,
rol, durum ve notlara tarama dokunmaz.

Başvuru olmayan mailler "Başvuru değil" ile `meta/ignored` belgesine yazılır ve
sonraki taramalarda bir daha çıkmaz.

## Veri düzeni

```
applications/<gmail-thread-id | m…>   bir başvuru
meta/config                           arama sorgusu, son tarama zamanı
meta/ignored                          "başvuru değil" denen thread id'leri
```

Durumlar: `gonderildi`, `cevap`, `gorusme`, `case`, `teklif`, `red`, `kapandi`.
Bir kayıt `gonderildi` durumundayken son hareketin üzerinden 10 gün geçtiyse
sessiz sayılır ve özet şeridinde uyarı olarak görünür.

## Geliştirme

Tek dosya, derleme adımı yok. `index.html` düzenlenip Artifact olarak aynı URL'e
yeniden yayınlanır. Dosyada `<!doctype>`, `<html>`, `<head>` ve `<body>`
etiketleri bilerek yoktur; yayınlanırken Artifact iskeleti sarmalar.

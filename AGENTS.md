# RoomBook · Codex çalışma kuralları

Bu repo, ANEW RoomBook eğitimindeki uygulama reposudur. Çalışmayı Türkçe açıklayabilir,
ancak kod, API sözleşmeleri, domain terimleri ve test adları İngilizce olmalıdır.

## Çalışma sırası

1. Önce `PLAYBOOK.md` içindeki ilgili adımı oku.
2. Bir özellik için önce `docs/specs/` altında yazılı ve kabul kriterleri olan bir spec oluştur.
3. Spec onaylanmadan uygulama koduna geçme.
4. Planı dosyalara ve testlere bağla; plan dışı dosya değişikliği yapma.
5. Uygulamadan sonra format/lint, unit test ve entegrasyon testlerini çalıştır.
6. Test çıktısını ve değişen dosyaları incelemeden “tamamlandı” deme.

## Kalite kapıları

- Zaman aralıklarında çakışma kuralı: `[start, end)`; `10:00–11:00` ile `11:00–12:00`
  çakışmaz.
- Geçersiz aralıklar (`start >= end`) reddedilir.
- Hata cevapları sabit, makinece okunabilir bir şemaya sahip olmalıdır.
- Conflict cevabı mümkünse önerilen boş slotları ve çakışan rezervasyonu içermelidir.
- Her davranış için anlamlı bir test ve testin gerçekten çalıştığını gösteren çıktı gerekir.
- Gizli bilgi, token veya yerel makineye özel yol commit edilmez.

## Codex davranışı

Codex kod yazmadan önce kısa bir plan ve etkilenecek dosyaları belirtir. Belirsiz bir ürün
kararını varsaymak yerine seçenekleri ve etkisini sorar. Mevcut testleri silmez veya gevşetmez.
Bir kalite kapısı başarısızsa bunu gizlemez; hatayı düzeltir ya da açıkça raporlar.

## Teslim kanıtı

Her teslimde şunları raporla:

- çalıştırılan komutlar ve sonuçları,
- eklenen veya değişen testler,
- bilinçli olarak kapsam dışında bırakılan noktalar,
- `git diff --stat` ve çalışma ağacının durumu.


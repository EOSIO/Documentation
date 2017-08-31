## EOS.IO Yazılımı Yol Haritası

Bu doküman geliştirme planının çerçevesini kuşbakışı çizer ve versiyon 1.0'a doğru ilerleme kaydedildikçe güncellenecektir. Bilinmelidir ki, bu yol haritası sadece blok zinciri yazılımı için geçerli olup Faz 1'in tamamlanması ile birlikte kendi takımlarına ve kendilerine özgü yol haritalarına sahip olacak olan cüzdanlar, blok tarayıcılar gibi araç ve yardımcı yazılımlar için geçerli değildir.

***Bu dökümanda bulunan her şey taslak halindedir ve her an değiştirilebilir ve sadece bilgi amaçlı sunulmaktadır. block.one bu yol haritası içeriğindeki bilgilerin doğruluğunu garanti etmemektedir ve bilgiler açıktan veya ima yoluyla hiçbir beyanı veya garantiyi içermeksizin "olduğu gibi" sunulmuştur.***

# Faz 1 - Asgari Uygun Test Ortamı - Yaz 2017

Bu fazın amacı geliştiricilerin EOS.IO üzerinde uygulamaları oluşturmaya başlamaları ve test etmeleri için gereksinim duyacakları API'leri (Uygulama Programlama Arayüzü) belirlemektir. Geliştiricilerin uygulamalarını test etmeye başlayabilmesi için aşağıdakilerin tamamlanmış olması gerekir:

### Bağımsız Düğüm (Dan ve Nathan)

Bağımsız bir düğüm bir API sunarken bir test blok zinciri çalıştırır ve bloklar üretir. Bu düğümün herhangi bir P2P ağ kodu ile ilgilenmesi gerekmez.

### Doğal Sözleşmeler (Nathan)

EOS.IO yazılımı bir dizi doğal sözleşmeye sahiptir. Bunlar blok zincirinin çekirdek işlemlerini yöneten kontratlardır ve Web Assembly Arabirimi dışında bulunurlar. Bu kontratlar şunlardır:

1. @eos - EOS token transferlerini yönetir
2. @pay algoritması - kilitli EOS'ları, oylamaları ve Yapımcı Seçimlerini yönetir
3. @sistem - izinleri, mesajları ve sözleşme kod güncellemelerini yönetir

### Sanal Makine API'si (Dan)

WebAssembly (WASM) ve WASM'a derlenen sözleşmeler blok zincirine belirlenmiş bir arayüz ile bağlanmalıdır. Geliştiricilerin EOS üzerinde gerçekten uygulamalar oluşturmaya başlaması bu API'ye ve onun nispeten stabil olmasına bağlıdır.

### RPC arabirimi (Arhag, Nathan)

HTTP arayüzü üzerinden basit bir JSON RPC aracılığıyla geliştiricilerin hareketleri yayınlamasına ve uygulama durumunu sorgulamalarına olanak tanıyan mimari sağlanacaktır. Bu, test uygulamalarının yayınlanması ve etkileşimi için hayati önem taşımaktadır.

### Komut Satırı Araçları (Arhag)

Komut satırı araçları RPC arabirimininin geliştirici derleme ortamları ile entegrasyonuna olanak sağlar.

### Temel Geliştirici Belgeleri (Josh)

Geliştiricilere EOS.IO blok zincirleri üzerinde nasıl geliştirme yapmaya başlayacaklarını öğreten dökümanlardır. Bunlar WASM API, RPC Arabirimi ve Komut Satırı Araçları dökümanlarını içerir.

# Faz 2 - Asgari Uygun Test Ağı - Sonbahar 2017

Faz 1'deki her şey sadece geliştiricinin kendi kodunu çalıştıran güvenilir bir ortam olduğunu varsayar. Bir test ağı kurulmasından önce birçok ilave özelliğin gerçekleştirilmesi ve test edilmesi gereklidir.

### P2P Ağ Kodu (Phil)

Bu, iki bağımsız düğüm arasındaki blok zinciri durumunu senkronize etmekten sorumlu bir eklentidir.

### WASM Sanitasyonu ve CPU düzeyinde güvenlik (Brian)

Kayan nokta işlemleri ve sonsuz döngüler gibi deterministik olmayan davranışları denetlemek için WASM kodunun sanitize edilmesi gerekir.

### Kaynak Kullanımı Takibi ve Oran Sınırlaması (Arhag)

Kötüye kullanımı engellemek amacıyla kaynak izleme ve kullanım oranı takibi kullanıcıları EOS pay algoritmasına göre sınırlandırır.

### Başlangıç İmport Testi (DappHub)

EOS Token Dağıtımı sürecinde veri çıkartmak ve bir başlangıç konfigürasyon dosyası oluşturmak için araçların geliştirilmesi gereklidir. Bu sayede Token Dağıtımına katılan herkesin bir miktar başlangıç test EOS'u (TEOS) edinmesi sağlanabilir.

### Blok Zincirleri Arası İletişim (Nathan)

Bu özellik aktarımlara ait Merkle hash işleminin doğru yürüdüğünün teyidini içerir.

# Faz 3 - Test Etme ve Güvenlik Denetimleri - Kış 2017, İlkbahar 2018

Bu faz sırasında platform güvenlik sorunları ve hataların tespiti odakta tutularak yoğun bir teste tabi tutulacaktır. Faz 3 sonunda versiyon 1.0 etiketi verilecektir.

### Örnek Uygulamaların Geliştirilmesi

Örnek uygulamalar platformun gerçek geliştiriciler tarafından ihtiyaç duyulan özelliklere sahip olduğunu kanıtlaması açısından için kritik öneme sahiptir.

### Başarılı Ağ Saldırıları için Ödüller

Versiyon 1.0'ın istikrarlı olmasını garanti etmek için ağa spam, sanal makine açıkları, bug çökertmeleri ve deterministik olmayan davranışlar ile saldırılar gerekli olacaktır ve süreçte yoğun olarak yer alacaktır. 

### Dil Desteği

WASM: C++, Rust, vb. dillerine derlenecek ilave diller için destek ekleme.

### Dökümantasyon ve Kılavuzlar

# Faz 4 - Paralel Optimizasyon - Yaz / Sonbahar 2018

Stabil bir 1.0 ürünü yayınlandıktan sonra, paralel yürütme kodunun optimizasyonu çalışmalarına yöneleceğiz.

# Faz 5 - Küme Uygulaması - Gelecek
# RUNBOOK — Çalıştırma Sırası

Bu dosya, projeyi hangi sırayla çalıştıracağını adım adım anlatır.

Kural: **bir adım FAIL verirse sonraki adıma geçme.** Her kapı, kendisinden
sonraki adımın dayandığı bir varsayımı doğruluyor.

Tüm komutlar proje kökünden (bu deponun klonlandığı dizin) çalışır.

---

## Özet tablo

| # | Komut | Süre (tahmini) | Üretir |
|---|---|---|---|
| 0 | ortam kurulumu | 5–15 dk | `.venv/` |
| 1 | `python -m scripts.verify_vault` | < 5 sn | — (kapı) |
| 2 | `python -m pytest tests -q` | 1–3 dk | — (kapı) |
| 3 | `python -m scripts.verify_solver` | 5–15 dk | — (kapı) |
| 4 | `python -m scripts.generate_data` | 3–8 dk | `results/raw/*.npz` |
| 5 | `python -m pytest tests -q` (tekrar) | 1–3 dk | — (kapı) |
| 6 | `python -m scripts.verify_model` | 5–15 dk | — (kapı) |
| 7 | `python -m scripts.run_all --profile quick` | 40–70 dk | `results/runs/*` |
| 8 | `python -m scripts.make_report` | < 1 dk | `results/benchmark.*` |
| 9 | `python -m scripts.run_all --profile full` | 4–7 saat | `results/runs/*` |
| 10 | `python -m scripts.make_report` | < 1 dk | `results/benchmark.*` |

Adım 7+8, boru hattının uçtan uca çalıştığını ucuza kanıtlar. Rapora girecek
sayılar adım 9+10'dan gelir. Numaralandırma `CALISTIRMA_SIRASI.txt` ile ve
script çıktılarındaki `step N clear` mesajlarıyla birebir aynıdır.

---

## 0 — Ortam kurulumu

`numpy`, `scipy`, `torch` sistemde kurulu değil. GPU: RTX 4050 Laptop (6 GB),
sürücü 610.74 → CUDA 12.4 build.

```bash
uv venv --python 3.11
```

Ardından sanal ortamı etkinleştir (PowerShell):

```bash
.venv\Scripts\Activate.ps1
```

PyTorch'u CUDA build olarak kur (bu tek satır ~2.5 GB indirir):

```bash
uv pip install torch --index-url https://download.pytorch.org/whl/cu124
```

Kalan bağımlılıklar:

```bash
uv pip install -r requirements.txt
```

**Geçme ölçütü:** aşağıdaki komut `True` yazmalı.

```bash
python -c "import torch; print(torch.cuda.is_available())"
```

`False` çıkarsa CPU'da da çalışır ama adım 9 birkaç kat uzar. `configs/base.yaml`
içinde `device: cpu` yap.

---

## 1 — Vault doğrulaması

```bash
python -m scripts.verify_vault
```

**Ne yapar.** `data/asm1.json` dosyasının SHA-256'sını vault'un kendi
`Audit Report.md` dosyasındaki denetlenmiş değerle karşılaştırır. Ardından 25
parametrenin ve 8×14 stokiyometri matrisinin her hücresini vault'un ürettiği
Markdown görünümleriyle tek tek karşılaştırır. Son olarak `nu @ C` süreklilik
kalıntısını yeniden hesaplar.

**Geçme ölçütü.** Çıkışın sonunda `PASS - steps 1 and 2 clear`. Fark sayısı 0,
maksimum kalıntı `5.5511151231257827e-17`, tolerans `1e-15`.

**FAIL olursa.** Vault dosyaları değişmiş demektir. `asm1_cl-pinn/` altındaki
hiçbir şeyi elle düzeltme; `tools/build_asm1_vault.py` ile yeniden üret ve
`Audit Report.md`'yi güncelle. Bu kapı geçmeden hiçbir sayı güvenilir değil.

---

## 2 — Birim testleri

```bash
python -m pytest tests -q
```

**Ne yapar.** Vault sözleşmesi, kinetik ifadeler, tesis geometrisi, influent
üreteci, sensör modeli, curriculum zamanlayıcı, kayıp terimleri ve sızıntı
disiplini.

`tests/test_leakage.py` iki katmanlı. **Statik katman** veri gerektirmez ve her
zaman koşar: kaynağı tokenize edip `src/train/` ve `src/models/` altındaki *tüm*
modüllerde `truth_reactor`/`truth_y` erişiminin tam olarak `[0]` olduğunu
doğrular, ayrıca `src/` genelinde ground truth'a dokunan her dosyanın
sınıflandırılmış olmasını şart koşar (yeni bir dosya sessizce eklenemez).
**Çalışma-zamanı katmanı** veri ister; bu adımda `skip` verir — normal, adım
5'te gerçekten koşar.

**Geçme ölçütü.** Hepsi `passed` veya `skipped`; hiç `failed` yok.

**Dikkat çeken testler:**
- `test_physics_residual_vanishes_on_the_true_dynamics` — fizik terimi doğru
  kurulmuşsa, gerçek dinamik beslendiğinde kalıntı sıfır olmalı. Bu test
  geçmiyorsa PINN'in öğrendiği her şey yanlış olur.
- `test_budget_parity_between_curriculum_and_baseline` — CL ile CL'siz koşular
  aynı toplam adım bütçesini alıyor mu.
- `test_physics_weight_is_never_zero` — tam PINN garantisi.

---

## 3 — Çözücü doğrulama kapısı

```bash
python -m scripts.verify_solver
```

**Ne yapar.** 200 günlük ısınmayı koşar (100 gün 20 °C'de yetmiyor;
`WARMUP_DAYS = 200`), sonra altı kapı:

| Kapı | Kontrol | Tolerans |
|---|---|---|
| 3a | BDF vs Radau vs LSODA | reaktör + çözünenler 1e-6, çöktürücü katıları 1e-4 |
| 3b | rtol 1e-8 → 1e-10 → 1e-12 yakınsaması | reaktör + çözünenler 1e-6, çöktürücü katıları 1e-4 |
| 3c | reaksiyon kapalı + geri devir kapalı ↔ matris üsteli (`scipy.linalg.expm`) | 1e-6 |
| 3d | izleyici kütle korunumu: `S_I` (reaksiyon açık) / `X_I` (reaksiyon kapalı) / `X_I` (reaksiyon açık, eq. 46 yaklaşımı) | 1e-8 / 1e-8 / 5e-2 |
| 3e | ısınma sonrası `‖dy/dt‖/‖y‖` | 1e-6 |
| 3f | farklı başlangıç tohumu → aynı denge noktası | 1e-6 |

Çöktürücü katılarının ayrı (gevşek) toleransı, Takacs modelinin parçalı-sürekli
RHS'inin belgelenmiş bir özelliğidir; `X_I` reaksiyon-açık sınırı ise BSM1
eq. 46 yaklaşımının ölçülen büyüklüğüdür, çözücü hatası değildir.

BSM1 Tablo 6 karşılaştırması **kasten yok**: bu proje 20 °C vault
parametreleriyle çalışır, BSM1 Tablo 6 ise 15 °C setiyle üretilmiştir. 3c ve 3d,
sıcaklıktan tamamen bağımsız, `solve_ivp` ile hiç kod paylaşmayan referanslardır.

**Geçme ölçütü.** `PASS - step 3 clear`.

**FAIL olursa.**
- 3a/3b fail → toleransları sıkılaştır (`SolverSettings`), sistem beklenenden
  daha stiff olabilir.
- 3c fail → `src/asm1/plant.py` içindeki taşınım (transport) terimlerinde hata var.
- 3d fail → çöktürücü kütle dengesinde hata var; `_settler_streams` ve eq. 34–44
  uygulamasına bak.
- 3e fail → 200 gün yetmemiş; `WARMUP_DAYS` artır.

---

## 4 — Sentetik veri üretimi

```bash
python -m scripts.generate_data
```

**Ne yapar.** Tek bir 200 günlük ısınmadan üç senaryo türetir, sonra her birini
dört gürültü seviyesinde sensör veri setine çevirir.

**Üretir** (`results/raw/` altına):

```
sim_constant.npz  sim_dry.npz  sim_rain.npz        ground truth
obs_constant_sigma0p00 / 0p05 / 0p10 / 0p15 .npz   curriculum aşama 1
obs_dry_sigma0p00 / 0p05 / 0p10 / 0p15 .npz        eğitim + holdout
obs_rain_sigma0p00 / 0p05 / 0p10 / 0p15 .npz       dağılım kayması testi
manifest.json                                       provenance
```

**Geçme ölçütü.** `PASS - steps 4 and 5 complete`. Ekranda ayrıca şunları
kontrol et:

- **Reaktör COD/N kapanışı** `1e-6`'dan küçük olmalı — script bunu **kapılar**.
  Büyükse çöktürücü kütle dengesinde sızıntı var.
- **Tesis geneli kapanış** sadece **raporlanır**, kapılanmaz: BSM1 eq. 46
  çöktürücü yaklaşımı yüzünden N tarafında ~1e-3 görülebilir. Beklenen
  davranıştır, hata değil.
- **Debi ortalaması** 18446'ya çok yakın; aralık BSM1 Şekil 3'ün
  10 000–32 000 aralığına yakın (birebir değil — ortalama ve max/min oranı
  tam sabitlenmiştir, mutlak uçlar yaklaşıktır, bu kasıtlı).
- **Kırpma oranı** (`clipped … % of samples`). Gürültü çarpımsal
  (`z·(1+eps)`); kırpma için `eps < -1` gerekir, σ=0.15'te bile beklenen değer
  ~0'dır (6.7σ'lık olay). Belirgin şekilde sıfırdan büyükse anormaldir —
  raporda belirt.

---

## 5 — Testleri tekrar koş

```bash
python -m pytest tests -q
```

**Ne yapar.** `tests/test_leakage.py` çalışma-zamanı katmanı artık `skip`
yerine gerçekten koşar: t > 0 sonrası tüm ground truth NaN yapılır ve kayıpların
(ileri kayıp **ve** geri gradyanlar, her curriculum aşamasında, her iki
mimaride) hâlâ sonlu kaldığı doğrulanır. Skip'in sessizce geçmediğinden emin
olmak istersen strict modda koş — veri eksikse `skip` yerine `fail` verir:

```bash
ASM1_STRICT_TESTS=1 python -m pytest tests -q
```

**Geçme ölçütü.** Hiç `failed` yok; önceki `skipped` testler artık `passed`.

---

## 6 — Model doğrulaması

```bash
python -m scripts.verify_model
```

**Ne yapar.** CPU'da float64 ile küçük deneme modelleri eğitip dört kapı kontrol
eder:

| Kapı | Kontrol |
|---|---|
| 6a | autograd türevi ↔ merkezî sonlu fark (< 1e-4) |
| 6b | forward-mode JVP ↔ reverse-mode VJP (< 1e-6) |
| 6c | fizik kalıntısının gradyanı sıfır değil **ve** fizik açıkken kalıntı daha küçük |
| 6d | t > 0 sonrası tüm ground truth NaN yapıldığında kayıp hâlâ sonlu |

6c, "tam PINN" iddiasının kanıtıdır: fizik terimi grafiğe gerçekten bağlı ve
sonucu gerçekten değiştiriyor. 6d, soft-sensor kurulumunun dürüstlüğünün
kanıtıdır: ölçülmeyen 11 bileşenin ground truth'u kayba hiç girmiyor.

**Geçme ölçütü.** `PASS - step 6 clear`.

**FAIL olursa.**
- 6a/6b fail → `configs/base.yaml` içinde `pinn.derivative_mode: reverse` yap ve
  tekrar dene; forward-mode kurulumu bu torch sürümünde parametre grafiğini
  taşımıyor olabilir.
- 6c fail → `src/models/losses.py` içindeki `physics_residual` ile `run.py`
  içindeki toplama arasında bağlantı kopmuş.
- 6d fail → eğitim yolunda ölçülmeyen durumdan besleniyorsun; `test_leakage.py`
  hangi satır olduğunu söyler.

---

## 7 — Hızlı süpürme (boru hattı kontrolü)

```bash
python -m scripts.run_all --profile quick
```

Önce ne koşacağını görmek istersen:

```bash
python -m scripts.run_all --profile quick --list
```

**Ne yapar.** 16 koşu (4 model × 4 gürültü), her biri 4000 adım. Amaç sonuç
üretmek değil — her kombinasyonun çökmeden sonuna kadar gittiğini görmek.

**Üretir.** `results/runs/<model>_sigma<xx>/` altına `checkpoint.pt`,
`history.json`, `predictions.npz`, `summary.json`, `config.yaml`.
Ayrıca `results/runs/sweep_index.json`.

**Geçme ölçütü.** `16 succeeded, 0 failed`.

Yarıda kesersen kaldığın yerden devam:

```bash
python -m scripts.run_all --profile quick --resume
```

**FAIL olursa.** İlgili koşunun klasöründe `error.txt` var. En olası iki neden:
GPU belleği (`collocation_points` düşür) ve `dtype: float32` ile sayısal taşma
(`float64` dene, yavaşlar).

---

## 8 — Ara rapor

```bash
python -m scripts.make_report
```

`results/benchmark.md` dosyasını aç. Bu noktada sayılar **anlamlı değil**
(4000 adım az), ama tabloların dolduğunu, Track A / Track B ayrımının
göründüğünü ve grafiklerin üretildiğini doğrula. Boş hücre varsa o koşu
başarısız olmuştur.

---

## 9 — Tam süpürme (raporlanacak koşular)

```bash
python -m scripts.run_all --profile full
```

**Süre.** 16 koşu, RTX 4050'de tahmini 4–7 saat. Kesilirse `--resume` ile devam.

Gece boyu bırakacaksan Windows'un uyku moduna geçmediğinden emin ol.

Tek tek koşmak istersen:

```bash
python -m scripts.run_all --profile full --models cl_pinn --noise 0.10
```

**Geçme ölçütü.** `16 succeeded, 0 failed`.

---

## 10 — Nihai rapor

```bash
python -m scripts.make_report
```

**Üretir.**

```
results/benchmark.csv            model × gürültü × değerlendirme kümesi
results/benchmark.md             Track A / Track B tabloları
results/benchmark_detail.json    bileşen bazında tüm metrikler
results/figures/loss_curves.png
results/figures/noise_robustness.png
```

**Nasıl okunacak.**

- **Track A** (ölçülen bileşenler: `S_O`, `S_NH`, `S_NO`) — dört modelin
  adil karşılaştırması. LSTM'lerin burada rekabetçi olması beklenir.
- **Track B** (ölçülmeyen 11 bileşen) — asıl sonuç. `lstm` ve `cl_lstm` burada
  hiçbir eğitim sinyaline sahip değil; sayıları başlangıç ölçeğini yansıtır,
  bir uydurma değil. Rapor bunu dipnotla belirtiyor.
- **`holdout`** satırları eğitimde görülmeyen son 2 güne, **`rain`** satırları
  hiç görülmemiş yağmur senaryosuna ait. Fizik teriminin değeri en çok burada
  görünür.
- **Gürültü sütunları** 0.00 → 0.15 boyunca hangi modelin ne kadar bozulduğunu
  gösterir.

---

## Sık kullanılacak yardımcılar

Tek bir koşuyu doğrudan çalıştır:

```bash
python -m src.train.run results/runs/cl_pinn_sigma0p10/config.yaml
```

Sadece testleri koş:

```bash
python -m pytest tests -q
```

Veriyi tek senaryo için yeniden üret:

```bash
python -m scripts.generate_data --scenarios dry
```

---

## Sıfırdan başlamak

`results/` klasörünü silmek her şeyi baştan üretilebilir hale getirir; vault ve
`asm1.xlsx` hiçbir adımda değiştirilmez, hepsi salt okunur kaynaktır.

```bash
rm -rf results/raw results/runs results/figures results/benchmark.*
```

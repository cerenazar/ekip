# claude-ekip

Claude'dan kendine bir yazılım ekibi kur.

Bu repo, Claude Code için hazır bir subagent ekibi içeriyor: kod yazan, test yazan, kod inceleyen ve hata ayıklayan dört agent. Klasörü projene kopyala, Claude Code işi uygun agent'a kendisi dağıtsın. [@cerenbuilds](https://instagram.com/cerenbuilds) reel'inin devamıdır.

## Kurulum

Önce repoyu indir:

```bash
git clone https://github.com/cerenazar/ekip.git claude-ekip
```

**Sadece bir projede kullanmak için** (projenin kök klasöründen çalıştır):

```bash
# macOS / Linux
mkdir -p .claude/agents && cp claude-ekip/.claude/agents/*.md .claude/agents/
```

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force .claude\agents
Copy-Item claude-ekip\.claude\agents\*.md .claude\agents\
```

**Tüm projelerinde kullanmak için** (kullanıcı seviyesi):

```bash
# macOS / Linux
mkdir -p ~/.claude/agents && cp claude-ekip/.claude/agents/*.md ~/.claude/agents/
```

```powershell
# Windows (PowerShell)
New-Item -ItemType Directory -Force $HOME\.claude\agents
Copy-Item claude-ekip\.claude\agents\*.md $HOME\.claude\agents\
```

Claude Code açıksa `/agents` komutuyla agent'ların yüklendiğini görebilirsin. Aynı isimde iki agent varsa proje seviyesindeki (`.claude/agents/`) kullanıcı seviyesindekini (`~/.claude/agents/`) ezer.

## Ekip

| Agent | Görevi | Araçları | Neden bu araçlar |
|---|---|---|---|
| `backend-dev` | Feature ve endpoint yazar, mevcut stile uyar | Read, Grep, Glob, Edit, Write, Bash | Önce kodu okuyup anlaması, sonra dosya yazıp build/test komutu çalıştırması gerekiyor |
| `test-writer` | Test yazar ve çalıştırır | Read, Grep, Glob, Edit, Write, Bash | Test dosyası oluşturur, framework'ü tespit eder, testleri çalıştırır |
| `code-reviewer` | Değişiklikleri inceler, kodu değiştirmez | Read, Grep, Glob, Bash | Düzenleme aracı yok. Bash sadece `git diff` gibi okuma komutları için |
| `debugger` | Kök nedeni bulur, en küçük düzeltmeyi yapar | Read, Grep, Glob, Edit, Bash | Hatayı yeniden üretmek için komut çalıştırır, düzeltmek için mevcut dosyayı düzenler. Yeni dosya oluşturmasına gerek yok |

Model tercihleri:

- `backend-dev`, `test-writer`, `code-reviewer`: `sonnet`. Hafif ve hızlı, günlük işler için yeterli.
- `debugger`: `inherit`. Hata ayıklama karmaşık akıl yürütme ister; ana oturumda hangi modeli kullanıyorsan onu kullanır.

Beğenmezsen agent dosyasındaki `model:` satırını değiştir (`sonnet`, `opus`, `haiku`, `inherit` veya tam bir model ID'si).

> Not: `code-reviewer` için Bash araç seviyesinde sadece `git diff` ile sınırlanamıyor. Agent'ın talimatı sadece okuma komutları çalıştırmasını söylüyor, ama sert bir kısıt değil. Daha sıkı bir sınır istersen Bash'i listeden çıkarıp diff'i prompt'ta kendin ver.

## Nasıl çalışır?

Ana Claude bir orkestra şefi gibi davranır. İsteğini alır, her agent'ın `description` alanına bakar ve işi en uygun olana verir. Her agent kendi context'inde çalışır, işi bitince sonucu ana konuşmaya özet olarak döner.

İsmiyle çağırabilirsin:

```text
code-reviewer agent'ını kullanarak son değişiklikleri incele.
```

veya @-mention ile:

```text
@agent-code-reviewer auth değişikliklerine bak
```

- **Bağımsız işler paralel çalışır.** Birbirini beklemeyen işleri aynı anda ver.
- **Bağımlı işler zincirlenir.** Bir agent'ın çıktısı bir sonrakinin girdisiyse sırayla çalıştır: önce yaz, sonra test et, sonra incele.

## Örnek prompt'lar

**Tek agent:**

```text
backend-dev agent'ını kullanarak kullanıcı profili için GET /users/:id endpoint'i ekle.
```

**Dört agent paralel:**

```text
Şu işleri paralel ve ayrı agent'larla yap: backend-dev siparişler modülüne endpoint eklesin, test-writer ödeme modülü için test yazsın, code-reviewer auth modülünü incelesin, debugger "TypeError: cannot read properties of undefined" hatasını araştırsın.
```

**Zincirleme:**

```text
Sırayla ilerle: backend-dev ile şifre sıfırlama feature'ını yaz, test-writer ile testlerini ekle, code-reviewer ile incele. Kritik veya uyarı seviyesinde bulgu çıkarsa debugger ile düzelt ve testleri tekrar çalıştır.
```

Daha fazlası için [`examples/prompts.md`](examples/prompts.md).

## Uçtan uca senaryo: bir feature'ı ekiple çıkarmak

Hedef: "Kullanıcılar favori ürün ekleyebilsin" feature'ı.

1. **Yaz.** `backend-dev agent'ını kullanarak kullanıcıların ürünü favorilere ekleyip çıkarabildiği endpoint'leri yaz.` Agent önce route, model ve servis dosyalarını okur, sonra mevcut stile uygun kodu yazar.
2. **Test et.** `test-writer agent'ını kullanarak az önce yazılan favori endpoint'leri için test ekle.` Projenin mevcut framework'ünü kullanır, kenar durumları (olmayan ürün, aynı ürünü iki kez ekleme) kapsar ve testleri çalıştırır.
3. **İncele.** `code-reviewer agent'ı ile değişiklikleri incele.` Bulguları kritik, uyarı ve öneri olarak, dosya ve satır numarasıyla alırsın.
4. **Düzelt.** Kritik veya uyarı varsa: `debugger agent'ı ile şu bulguyu düzelt: <bulgu>.` Düzeltmeyi yapar ve testlerin geçtiğini gösterir.
5. **Son kontrol.** Gerekirse 3. adımı tekrar et, sonra commit'le.

Hepsini tek prompt'ta da isteyebilirsin; bunun için zincirleme örneğine bak.

## İpuçları

- **En az yetki.** Her agent'a sadece işi için gereken araçları ver. Reviewer'ın dosya düzenleyememesi bir eksik değil, bilinçli bir güvence.
- **Her agent'ın kendi context'i var.** Ana konuşmayı şişirmezler. Ama hepsi senin kullanım limitinden yer, yani dört agent paralel çalışırsa limitin daha hızlı tükenir.
- **Description'ı net yaz.** Ana Claude işi kime vereceğine buna bakarak karar verir. Belirsiz bir description yanlış agent'a, ya da hiç agent'a gitmemesine yol açar. "Use proactively" ifadesi otomatik delege etmeyi teşvik eder.
- **Kendi agent'ını ekle.** `.claude/agents/` altına yeni bir `.md` dosyası koy, `name` ve `description` zorunlu alanlar.
- **Description'ları kısa tut.** Çok sayıda agent'ın description'ı toplamda çok uzarsa Claude Code başlangıçta uyarı verir.

## İleri seviye: agent teams (deneysel)

Subagent'lar ana Claude'a sonuç döner, birbirleriyle konuşmaz. **Agent teams** ise birden fazla Claude Code oturumunun bir ekip olarak çalışmasıdır:

| | Subagent | Agent teams |
|---|---|---|
| İletişim | Sonucu çağırana döner | Üyeler birbirine doğrudan mesaj atar |
| Koordinasyon | Ana agent yönetir | Ortak görev listesi ve mesajlaşma |
| Token kullanımı | Daha düşük | Daha yüksek, her üye ayrı bir Claude instance'ı |
| Ne zaman | Sonucu önemseyen odaklı işler | Tartışma ve iş birliği isteyen karmaşık işler |

Agent teams **deneysel** ve varsayılan olarak **kapalı**. Açmak için `settings.json` dosyana şunu ekle:

```json
{
  "env": {
    "CLAUDE_CODE_EXPERIMENTAL_AGENT_TEAMS": "1"
  }
}
```

Bu repodaki agent tanımlarını ekip üyesi olarak da kullanabilirsin. Üye oluştururken agent tipini ismiyle ver; tanımdaki `tools` ve `model` uygulanır, gövde (talimatlar) üyenin sistem prompt'una eklenir:

```text
Auth modülünü paralel incelemek için 3 teammate oluştur: biri code-reviewer agent tipini kullanarak güvenliğe, biri code-reviewer tipiyle performansa baksın, biri test-writer tipiyle test kapsamını kontrol etsin. Bulgularını birbirleriyle paylaşsınlar, sonunda özet ver.
```

Bilmen gerekenler:

- **3-5 üyeyle başla.** Dokümantasyonun önerisi bu. Çok üye, çok token ve koordinasyon yükü demek.
- **Önce inceleme ve araştırmayla dene.** Kod yazmayan işler (PR inceleme, hata araştırma) daha az sorun çıkarır.
- **Aynı dosyayı iki üyeye verme.** Üst üste yazma sorunu çıkar.
- **Açıkken davranış değişir.** Agent teams açıkken Claude'un isim vererek çağırdığı bir subagent, üye (teammate) olarak başlayabilir. Normal subagent davranışına dönmek için değeri `"0"` yap.
- **Sınırlar:** oturum devam ettirmede (`/resume`) üyeler geri gelmez, bir oturumda tek ekip olur, üyeler kendi ekip üyesini başlatamaz.

Güncel ayrıntılar için resmi dokümantasyona bak: [Subagents](https://code.claude.com/docs/en/sub-agents) · [Agent teams](https://code.claude.com/docs/en/agent-teams).

## Kapanış

Daha fazlası [@cerenbuilds](https://instagram.com/cerenbuilds)'te.

Bir hata görürsen, bir agent'ı geliştirdiysen veya yeni bir agent önermek istersen issue ya da pull request aç, ya da Instagram'dan yaz. Geri bildirimler hoş karşılanır.

MIT lisansı. Bkz. [LICENSE](LICENSE).

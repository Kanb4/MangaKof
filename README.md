# MangaKof

Um leitor de mangá TUI (interface de terminal) usando a API do MangaDex, escrito em Kof.

## Requisitos

- **Kof** 0.2.6+ — um compilador de linguagem de programação multi-target
- **JDK** 21+ (incluso na distribuição Kof)
- **MPV** — visualizador de imagens (instale via gerenciador de pacotes ou mpv.io)
- **curl** — já incluso no Windows 10+ e na maioria das distros Linux/macOS

## Instalação (Windows)

### 1. Instalar o Kof

**Opção A — Instalador oficial (recomendado):**
```powershell
# Baixar o instalador do GitHub Releases:
# https://github.com/KofLang/Kof4j/releases/latest
# Extrair e rodar:
kof install %USERPROFILE%\.kof
```
Isso instala o Kof em `%USERPROFILE%\.kof` e adiciona ao PATH automaticamente.

**Opção B — Scoop (gerenciador de pacotes):**
```powershell
scoop bucket add kof https://github.com/KofLang/scoop-bucket
scoop install kof
```

**Opção C — Chocolatey:**
```powershell
choco install kof
```

**Verificar:**
```powershell
kof version
# Deve mostrar algo como: kof 0.2.6
```

### 2. Instalar o MPV

**Opção A — Scoop (recomendado):**
```powershell
scoop install mpv
```

**Opção B — Chocolatey:**
```powershell
choco install mpv
```

**Opção C — Instalador manual:**
1. Baixar de https://mpv.io/installation/
2. Instalar e **marcar "Add to PATH"**

**Verificar:**
```powershell
mpv --version
```

### 3. Verificar curl

Já incluso no Windows 10+:
```powershell
curl --version
```

### 4. Clonar e rodar o MangaKof

```powershell
git clone https://github.com/Kanb4/MangaKof.git
cd MangaKof
kof run src\main.kf
```

---

### Verificação rápida (tudo junto)

```powershell
kof version && mpv --version && curl --version
# Se os 3 mostrarem versão, está tudo pronto!
```

---

### Dica: adicionar ao PATH permanentemente (se necessário)

```powershell
# Exemplo se instalou manualmente:
$env:PATH += ";C:\kof\kof-0.2.6\bin;C:\Program Files\mpv"
# Para permanente (requer admin):
# [Environment]::SetEnvironmentVariable("PATH", $env:PATH + ";C:\kof\kof-0.2.6\bin;C:\Program Files\mpv", "User")
```

---

## Instalação (Linux / macOS)

```bash
# Instalar Kof (requer JDK 21+ e Maven)
mvn clean package -DskipTests -DskipNative
cp kof-cli/target/kof-*.jar ~/.local/lib/kof/kof.jar

# Instalar MPV
sudo apt install mpv       # Debian/Ubuntu
brew install mpv           # macOS

# curl vem pré-instalado na maioria dos sistemas
```

## Como Executar

```bash
# Do diretório do projeto (requer kof no PATH)
kof run src\main.kf
```

### Modo Teste (entrada em arquivo)

Crie um `input.txt` com os comandos:
```
Kimagure Orange Road
1
en
1
sair
```

```bash
kof run src\main.kf < input.txt
```

## Uso

```
Digite o nome do manga (ou 'sair' para sair):
> Kimagure Orange Road
Buscando: Kimagure Orange Road
[1] Kimagure Orange★Road | Volumes: 18 | Status: completed | Lang: ja
[2] Kimagure Orange Road (Official Colored) | Volumes: 19 | Status: completed | Lang: ja
Digite o numero do manga para ver capitulos (ou 'sair'):
> 1
Digite o idioma (en=ingles, pt-br=portugues, es=espanhol, etc.):
> en
Carregando capitulos (en)...
  [1] Vol ? - Cap 1: Red Straw Hat
  ...
Total: 20 capitulos em en
Digite o numero do capitulo para baixar (ou 0 para voltar):
> 1
Baixando capitulo 1...
  Baixado pagina 1/x1-38d38d48...jpg
  ...
Download concluido: 44 paginas.
Abrindo no MPV...
```

## Arquitetura de Download

1. **Chamadas API**: `http.get(url, "User-Agent: MangaKof-TUI/1.0")` — retorna String (JSON raw)
2. **Download de imagens**: `process.run("curl", "-s", "-L", "-m", "30", "-H", "User-Agent: ...", "-H", "Referer: https://mangadex.org/", "-H", "Accept: image/webp,...", "-o", path, url)` — binariamente seguro via curl
3. **Validação**: `readFile(path)` retorna `null` para arquivos binários (imagens) ou String para texto (erros HTML). `Path(path).size()` verifica se o arquivo foi realmente criado (size > 0). Se `readFile` retorna não-null e contém `<html` ou `<!doctype`, o CDN retornou uma página de erro.
4. **Correção BaseURL**: `.replace("\\/", "/")` desfaz escape JSON `\/` da resposta da API MangaDex (`http.get` retorna strings raw sem parsear JSON)
5. **MPV**: `process.run("mpv", "--loop-playlist=inf", "manga_chap_N/playlist.m3u")` — abre todas as páginas como playlist infinita via arquivo M3U

## Organização de Arquivos

Cada capítulo baixado é salvo em sua própria pasta:

```
manga_chap_1/
├── mangakof_p01.jpg
├── mangakof_p02.jpg
├── mangakof_p03.jpg
└── playlist.m3u       ← playlist M3U auto-gerada
```

O MPV abre todas as imagens da playlist com loop infinito, permitindo navegar entre todas as páginas.

## Controles do MPV

| Tecla | Ação |
|-------|------|
| → (seta direita) | Próxima página |
| ← (seta esquerda) | Página anterior |
| Espaço | Pausar/despausar |
| f | Alternar tela cheia |
| q / ESC | Fechar MPV |
## Licença

GPLv3

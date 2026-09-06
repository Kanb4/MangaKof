# MangaKof

Um leitor de mangá TUI (interface de terminal) usando a API do MangaDex, escrito em Kof.

## Requisitos

- **Kof** 0.2.6+ — um compilador de linguagem de programação multi-target
- **JDK** 21+ (incluso na distribuição Kof)
- **MPV** — visualizador de imagens (instale via gerenciador de pacotes ou mpv.io)
- **curl** — já incluso no Windows 10+ e na maioria das distros Linux/macOS

## Instalação (Windows)

```bash
# Kof instalado em C:\kof\kof-0.2.6
# MPV instalado em C:\Program Files\GoAnime\bin
```

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
# Do diretório do projeto
java -jar <caminho-do-kof>/lib/kof.jar run src/main.kf
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
java -jar <caminho-do-kof>/lib/kof.jar run src/main.kf < input.txt
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

## Limitações Conhecidas

- **Bloqueio Cloudflare CDN**: O MangaDex CDN (`*.mangadex.network`) é protegido por Cloudflare e pode retornar páginas HTML em vez de imagens dependendo da região/IP. O app detecta isso e mostra `ERRO: CDN retornou HTML em vez de imagem`.
- **Contorno Cloudflare**: O app inclui headers `Referer: https://mangadex.org/` e `Accept: image/webp,...` nos requests curl para reduzir desafios do Cloudflare, mas isso não é garantido em todas as regiões.
- **Windows**: curl disponível via `C:\Windows\System32\curl.exe`
- **Linux/macOS**: curl vem pré-instalado (instale via `apt`, `brew`, etc.)

## Status

- ✅ Busca de mangás via API MangaDex
- ✅ Exibição de resultados (título, volumes, status, idioma)
- ✅ Listagem de capítulos
- ✅ Download de capítulos (imagens salvas como `mangakof_p*.jpg` em pastas separadas por capítulo)
- ✅ Abertura no MPV com todas as imagens em playlist infinita
- ✅ Banner ASCII "MangaKof"
- ✅ Interativo (loop de busca contínua)
- ✅ Validação de entrada (números inválidos não travam o programa)
- ✅ Validação de download (detecta HTML do CDN/Cloudflare)
- ✅ Correção de escape JSON `\/` da API MangaDex
- ✅ Headers de contorno Cloudflare (Referer + Accept)
- ✅ Pastas por capítulo (`manga_chap_<N>/`)

## Dependências

```json
"kof.web", "kof.io", "kof.config", "kof.process"
```

## Licença

GPLv3
# MangaKof - Agent Instructions

## Project Setup
- Kof compiler: `C:\kof\kof-0.2.6\jdk\bin\java.exe -jar C:\kof\kof-0.2.6\lib\kof.jar` (JVM 21)
- Run command: `java -jar C:\kof\kof-0.2.6\lib\kof.jar run src\main.kf`
- Must run from the project directory (`C:\Users\Kanba\Desktop\manga-tui`)
- kof.json dependencies: `kof.web`, `kof.io`, `kof.config`, `kof.process`

## Run Commands
```bash
# Interactive
cmd /c '"C:\kof\kof-0.2.6\jdk\bin\java.exe" -jar C:\kof\kof-0.2.6\lib\kof.jar run src\main.kf'

# Piped input (test mode)
cmd /c '"C:\kof\kof-0.2.6\jdk\bin\java.exe" -jar C:\kof\kof-0.2.6\lib\kof.jar run src\main.kf < input.txt'
```

## Known Compiler Bugs (kof 0.2.5-beta)
1. **SEM024**: Calling a user-defined function that returns a value causes `COMP002: frame crash`
   - Workaround: Only use void functions (no return values), or inline all logic in main()
2. **`.length()` on search strings**: Using `.length()` as a method call on a computed string causes issues
   - Workaround: Hardcode offsets (e.g., `"id":"` is 8 chars, so use `+ 8`)
3. **`process.spawn(cmd, args...)`**: Separate-args form causes `NoClassDefFoundError: kof/process/Result`
   - Workaround: Use `process.run(cmd, args...)` (blocks until completion) or `process.spawn("cmd arg1 arg2")` single-string form
   - Note: `process.spawn` with a single string does NOT execute as a shell command — it tries to find an executable with that exact name. Use `process.run` with separate args instead.

## Download Architecture (CORRENT)
1. **API calls**: `http.get(url, "User-Agent: MangaKof-TUI/1.0")` — returns String (text/JSON)
2. **Image downloads**: `process.run("curl", "-s", "-L", "-m", "30", "-H", "User-Agent: ...", "-H", "Referer: https://mangadex.org/", "-H", "Accept: image/webp,...", "-o", path, url)` — binary-safe via curl
3. **Validation**: `readFile(path)` returns null for binary files (images) or a String for text (HTML errors). `Path(path).size()` checks if file was created (size > 0). If non-null + contains `<html` or `<!doctype`, the CDN returned an error page.
4. **MPV**: `process.run("mpv", "--loop-playlist=inf", "manga_chap_N/playlist.m3u")` — cross-platform; M3U playlist with all pages in chapter folder
5. **BaseURL**: `.replace("\\/", "/")` — unescapes JSON `\/` sequences from MangaDex API response (`http.get` returns raw strings without JSON parsing)

## CDN Issue (known limitation)
- MangaDex CDN (`*.mangadex.network`) is behind Cloudflare and may return HTML challenge pages
  instead of images depending on the region/server/IP
- Workaround: Use a VPN or ensure the request comes from a non-blocked region
- The code now detects this and reports `ERRO: CDN retornou HTML em vez de imagem`

## Fixes Applied
1. **ID extraction**: Changed `indexOf("\"id\":\"", titleKeyPos - 200)` → `lastIndexOf("\"id\":\"", titleKeyPos)` 
   to correctly find the manga's own ID instead of tag IDs
2. **Binary download**: Replaced `http.get` (returns String, corrupts binary) with `process.run("curl", ...)`
   which writes raw bytes directly to file
3. **User-Agent**: Added to all `http.get` calls
4. **Validation**: Added download validation using `readFile` to detect HTML responses
 5. **MPV command**: Fixed hardcoded `.jpg` extension — now tracks actual `firstImage` filename
 6. **baseUrl escape fix**: `.replace("\\/", "/")` on baseUrl to unescape JSON `\/` sequences from MangaDex API response (URLs DO use forward slashes in JSON `\/` escape form; `http.get` returns raw strings without parsing JSON escapes)
 7. **Download validation**: Added `Path(savePath).size()` check — `readFile` returns `null` for both binary files AND non-existent files, so size check distinguishes success from failed curl downloads
 8. **Removed**: Debug prints (test_image.png cleanup)
 9. **MPV playlist**: Changed `--loop-file` (single image) to `--loop-playlist=inf` with M3U playlist file containing all downloaded pages — cross-platform (no `cmd /c` dependency)
 10. **Cloudflare bypass**: Added `Referer: https://mangadex.org/` and `Accept: image/webp,...` headers to curl requests — reduces HTML challenge page returns from CDN
 11. **Per-chapter folders**: Each chapter downloads to `manga_chap_<N>/` directory, all images opened via M3U playlist + `--loop-playlist=inf`

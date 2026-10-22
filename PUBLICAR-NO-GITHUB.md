# Publicar o TraceDraw no seu GitHub (versão web)

O TraceDraw já vem com tudo para rodar no navegador: o mesmo programa em
Rust é compilado para WebAssembly pelo GitHub Actions e publicado no
GitHub Pages. Você não precisa instalar nada no seu computador.

Resultado final: `https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/`

## Passo a passo

1. **Crie o repositório.** No GitHub: New repository, nome à sua escolha
   (por exemplo `tracedraw`), **Public**. O GitHub Pages é gratuito para
   repositórios públicos; em repositório privado exige plano pago. Não
   marque "Add a README", o projeto já tem um.

2. **Envie os arquivos.** Na pasta do projeto:

   ```sh
   git init -b main
   git add .
   git commit -m "TraceDraw: versão revisada"
   git remote add origin https://github.com/SEU-USUARIO/NOME-DO-REPOSITORIO.git
   git push -u origin main
   ```

   O ramo precisa se chamar `main`: é nele que o workflow publica.
   (O projeto pede `Signed-off-by` nos commits para contribuições ao
   projeto original; no seu repositório isso é opcional.)

3. **Ligue o Pages.** No repositório: Settings > Pages > Build and
   deployment > Source: **GitHub Actions**.

4. **Espere o workflow.** Aba Actions > workflow `build`. Ele roda os
   testes, compila a versão web e publica. Na primeira vez leva de 15 a
   40 minutos (compilação do zero). Os instaladores para Windows, macOS
   e Linux são compilados em paralelo e aparecem em Releases > Nightly;
   você pode ignorá-los.

5. **Abra o endereço** mostrado no job `pages` (ou em Settings > Pages).
   A primeira carga baixa alguns MB de WebAssembly; depois o navegador
   guarda em cache. Precisa de um navegador com WebGL 2.

Cada `git push` no `main` publica de novo.

## Se algo falhar

- **O job `pages` falha com "environment protection" ou "not enabled":**
  confirme o passo 3 e, em Settings > Environments > `github-pages`,
  permita o ramo `main`. Depois use Actions > build > Re-run failed jobs.
- **O job `test` ou `web` falha ao compilar:** as correções de segurança
  deste pacote foram escritas sem poder compilar no ambiente em que
  foram feitas. Se aparecer um erro de compilação, copie as linhas do
  erro do log do Actions e envie para correção; as mudanças estão
  listadas em `SECURITY.md`.
- **Página em branco e erro de "Content Security Policy" no console
  (F12):** apague a linha `<meta http-equiv="Content-Security-Policy"
  ...>` de `web/index.html` e publique de novo, ou ajuste a política
  (o `index.html` explica cada parte). Se você editar o script
  `<script type="module">` do `index.html`, atualize o hash `sha256-...`
  da política.
- **"Abrir index.html direto do disco" não funciona:** navegadores
  recusam módulos em `file://`. Teste com um servidor local (abaixo).

## Testar no seu computador (opcional)

```sh
rustup target add wasm32-unknown-unknown
cargo install wasm-bindgen-cli --version 0.2.129   # igual ao Cargo.lock
./scripts/web/build.sh
python3 -m http.server -d site 8080                # abra http://localhost:8080
```

Antes de publicar, rode também `cargo test --workspace` (os testes do
aplicativo não rodam no workflow).

## Apontar os links para o seu repositório (opcional)

O programa guarda o endereço do projeto original (`igtoth/tracedraw`) em
alguns lugares: a Ajuda (F1) abre as páginas do site original, e a
verificação de atualizações do aplicativo de desktop consulta as versões
do projeto original. Isso não afeta a versão web. Se quiser trocar:

| Arquivo | O que trocar |
|---|---|
| `Cargo.toml` | `repository` e `homepage` |
| `apps/tracedraw/src/help.rs` | `BASE` (e os dois testes com o endereço) |
| `apps/tracedraw/src/updates.rs` | `LATEST_URL`, `RELEASES_PAGE` |
| `apps/tracedraw/src/ui/menus.rs`, `ui/dialogs.rs` | endereços de discussões, issues e `CDR_ISSUE` |
| `apps/tracedraw/src/github.rs` | o dono em uma linha de teste |
| `web/index.html` | os três links da tela de carregamento |
| `apps/tracedraw/src/web_page.rs` | os mesmos três links, no teste da página |
| `PRIVACY.md`, `README.md` | endereços |

Troque sempre o código e o teste correspondente juntos.

## Licença

O projeto é MIT ou Apache-2.0. Ao publicar uma cópia, mantenha
`LICENSE-MIT`, `ATTRIBUTION.md` e o aviso de copyright de Ighor Toth e
dos colaboradores. O arquivo `LICENSE-APACHE` deste pacote contém apenas
uma linha apontando para o texto oficial; se quiser distribuir sob a
Apache-2.0, coloque nele o texto completo de
https://www.apache.org/licenses/LICENSE-2.0.txt.

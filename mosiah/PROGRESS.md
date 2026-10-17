# Progresso da investigação

## Última execução

17/10/2026 — leitura dos manifests `.deps.json` e execução do `rea setup --client codex --dry-run --json` para REA 6.5.0 em staging local. Alvo: `C:\exocad-DentalCAD3.3-SR1-2026-01-16`.

## Resultados confirmados

- Sistema identificado como Windows 10 Home Single Language 25H2 x64, build 26200.9711; PowerShell 5.1, Node.js v24.20.0 e runtime .NET 8.0.21. WMI negou acesso; política do PowerShell bloqueia a execução direta de `.ps1` e não foi alterada.
- O pacote contém quatro árvores principais: DentalCADApp (10.533 arquivos), DentalDB (1.813), exoprint (432) e LibraryManager (373). `CAD-Data` contém 1.021 itens, mas seu conteúdo não foi inspecionado nem exportado por poder incluir dados clínicos.
- Catálogo de software reproduzível: 13.151 arquivos, 2.928.133.977 bytes, 13.151 SHA-256 válidos, zero erros; 543 PEs (158 x64, 385 x86), 482 identificados como assemblies gerenciados. A passada independente em Node coincidiu com a captura PowerShell em caminhos, tamanhos, hashes, máquina PE e detecção de managed assemblies.
- Quatro SBOMs CycloneDX declaram 892 componentes, mas não incluem arestas `dependencies`.
- Nova leitura de 52 manifests `.deps.json` (DentalCADApp 5, DentalDB 44, exoprint 1, LibraryManager 2): 3.669 ocorrências de bibliotecas e 11.782 declarações de dependência; todas resolveram a uma chave presente no target correspondente, sem erros de parse. No manifest principal, DentalCADApp declara 13 dependências diretas do tipo `project`; DentalDB 24; TransferTo3DPrinter 4; libraryManager 1. Isso representa declarações de implantação, não carregamento observado em runtime.
- Validação independente PowerShell (`ConvertFrom-Json`) reproduziu as contagens: 52 manifests também presentes no catálogo Node, 11.782/11.782 arestas com chave exata no target e zero erros. Ver `artifacts/validacao-deps-json.json`.
- Evidências reprodutíveis dos manifests estão em `artifacts/deps-json/`; script `scripts/analisar-deps-json.mjs`. `docs/02-catalogo-arquivos.md`, `docs/04-analise-estatica.md`, `docs/06-arquitetura.md` e o diagrama de dependências foram ampliados com os limites dessa evidência.
- A consulta ao registry e a resolução de `latest` retornaram `EACCES`; o cache continha o tarball 6.5.0, recuperado e instalado offline em `artifacts/rea-cache/runtime` com scripts de lifecycle ignorados e opcionais omitidos. SHA-256: `531717a06e92ca651aab52691190f916fac044272c01f875e22c5e293218e699`; `npm ls` confirmou `rea-agents@6.5.0` (146 pacotes adicionados). `doctor --client codex --json` confirmou skill v42 alinhada/207 ferramentas, inscrição Codex ausente e ambiente geral não saudável por providers opcionais ausentes e autoteste Jadx bloqueado por Win32 5.
- `setup --client codex --dry-run --json` saiu com sucesso e estado `planned`: uma atualização prevista em `C:\Users\labor\.codex\config.toml`, preservando outras configurações e usando backup `C:\Users\labor\.codex\config.toml.rea.backup`; servidor proposto executa o `rea.mjs mcp` pelo Node instalado. `appliedActions` vazio. Nenhuma configuração foi alterada. O destino está fora da raiz gravável `C:\Users\labor\Downloads\mosiah`, portanto a conexão não pode ser aplicada por esta sessão.
- Validação do plano: exatamente uma ação `configure_client:codex` (`operation=update`), versão instalada conferida por `npm ls` (`6.5.0`), estado de registro `missing`, zero ações aplicadas. O recibo está em `artifacts/rea-cache/installation-and-plan.json`; contagens correntes da validação de entregáveis estão em `artifacts/validacao-entregaveis.json`. Os três scripts Node do catálogo passaram `node --check`.
- Inspeção não destrutiva do CLI `rea capabilities --json`: reporta `rea-artifact-graph`, `rea-dotnet-static` e `sqlite` disponíveis; Ghidra/IDA não configurados e Hopper/macOS utilities indisponíveis no host; nenhum target nem provider de análise selecionado. Resposta integral: `artifacts/rea-cache/capabilities-codex.json`. A captura documenta capacidades declaradas e efeitos; não foi usada para decodificar arquivos Exocad.
- `rea mcp doctor --json` terminou com `healthy=true`: servidor stdio 6.5.0, protocolo 2025-11-25, 207/207 tools, 6/6 prompts sem diferenças e chamada `binary_session` sem target bem-sucedida. Evidência: `artifacts/rea-cache/mcp-doctor-codex.json`. Isso prova o servidor MCP local, não a conexão do Codex nem análise do Exocad.
- Busca segura de nomes de possíveis avisos legais nas quatro árvores de software (sem ler conteúdo) encontrou 130 documentos: 128 avisos de terceiros e 2 cópias de `FreePalmerLicense.txt`; nenhuma EULA Exocad identificável por esse filtro. Isso não exclui termos com outro nome ou embutidos; registro em `artifacts/licensing-notice-scan.json`.
- Recatalogação Node atual do alvo concluiu 13.151 arquivos / 2.928.133.977 bytes com zero erros; comparação por caminho/tamanho/SHA-256 com a referência deu 0 added, 0 removed, 0 changed. Evidências novas: `artifacts/catalogo-recheck/` e `artifacts/validacao-catalogo-recheck.json`; CAD-Data ficou excluída.
- Consulta passiva pontual por nomes conhecidos em processos e serviços: 0 resultados às 18:29:29Z; evidência limitada em `artifacts/passive-process-check.json`. Não houve inicialização do Exocad nem captura de runtime.
- Novo verificador independente `scripts/validar-evidencias.mjs` e recibo `artifacts/verificacao-integrada.json`; duas execuções PASS, 47/47 hashes conferidos, recheck do catálogo, 52 manifests/11.782 edges, REA MCP e scripts Node validados. Escopo explicitamente limitado aos artefatos do workspace; não acessa o alvo nem dados de pacientes.
- Consulta às fontes oficiais exocad acrescentou a EULA de venda Dental de 2019 e a página de distribuidores do Brasil. A EULA define a relação via distribuidor, escopo comercial e restrição de reverse engineering sujeita à exceção §69e; distribuidores brasileiros empacotam software customizado. Isso reforça a relevância do contrato de compra/upgrade, mas não identifica o aplicável a esta instalação. Nenhuma análise de código foi iniciada.
- Extraído o certificado embutido em quatro EXEs principais: todos declaram `CN=exocad GmbH`, mesmo emissor DigiCert e validade declarada 2024-09-17 a 2027-09-16 UTC; SHA-256 dos arquivos confere com o catálogo. Evidências: `artifacts/certificados-assinatura-pe.json`, reprodutor `scripts/extrair-certificados-pe.ps1`. Não foi validado digest, cadeia, revogação nem publisher authenticity.
- Verificação local WinVerifyTrust dos mesmos quatro EXEs, com UI silenciada, revogação desligada e `WTD_CACHE_ONLY_URL_RETRIEVAL`: todos `0x00000000`; nenhum retrieval de revogação pela rede. Evidência `artifacts/validacao-authenticode-offline.json`, reprodutor `scripts/verificar-authenticode-offline.ps1`. Revogação online e frescor de timestamp não estabelecidos.
- Varredura WinVerifyTrust dos 543 PE: hashes coincidem com o catálogo em 543/543. Sob `WTD_LIFETIME_SIGNING_FLAG`, 211 passaram, 312 expiraram e 20 DLLs não têm assinatura extraível. Rechecagem sem esse flag fez 312/312 expirados retornarem sucesso; sem rede nem revogação em ambas. Evidências `artifacts/validacao-authenticode-catalogo.json` e `artifacts/comparacao-authenticode-timestamp.json`; scripts correspondentes em `scripts/`. Verificador integrado cruzou cada um dos 543 hashes com o catálogo.
- Parser PowerShell local analisou os seis scripts `.ps1` (catalogador, comparador e quatro de assinatura): zero erros; resultado `artifacts/validacao-powershell-scripts.json`. Isto é validação sintática, não execução dos scripts.
- Uma tentativa PowerShell de excluir `CAD-Data` por expressão regular falhou por sequência de escape inválida (`\C`); não alterou arquivos e não produziu inventário de nomes dessa árvore. A análise foi refeita usando exclusivamente as quatro raízes de software explicitamente permitidas.
- A validação anterior dos entregáveis foi atualizada após inclusão das evidências REA; ver `artifacts/validacao-entregaveis.json` para contagens atuais e hashes recalculados.

## Limitações e próxima ação

- Para conectar REA: aplicar o plano apresentado em uma sessão que tenha permissão de escrita em `C:\Users\labor\.codex\config.toml`, depois reiniciar/reconectar o Codex e verificar as ferramentas anunciadas. Não instalar engines adicionais sem necessidade.
- A EULA/território/distribuidor aplicável à instalação não foi identificado. Decompilação/disassembly e observação dinâmica continuam pendentes dessa confirmação; nenhuma proteção/licença foi investigada para contorno.
- Não executar fluxos dinâmicos nem abrir exemplos de projeto até confirmação de dataset sintético/autorizado e condições seguras. Os arquivos de relatório não equivalem a prova de comportamento em runtime.

## Atualizacao de verificacao - 2026-10-17

- Reexecutei `setup --client codex --dry-run --json` usando o pacote REA 6.5.0 preparado localmente. Continua planejada uma unica inscricao MCP em `C:\Users\labor\.codex\config.toml`, com backup `.rea.backup`; `appliedActions` esta vazio. `npx.cmd -y rea-agents@latest setup --dry-run` nao acessou o npm (`EACCES`), portanto `latest` nao foi confirmado nem usado.
- Consultei o `--help` real do CLI 6.5.0 para `inspect-pe-resources`; nao iniciei analise de recursos do Exocad porque a autorizacao contratual aplicavel a engenharia reversa segue desconhecida.
- A primeira execucao do verificador integrado encontrou quatro hashes desatualizados no manifesto apos edicoes anteriores: `RELATORIO_FINAL.md`, `docs/04-analise-estatica.md`, `docs/08-validacao.md` e `scripts/validar-evidencias.mjs`. Recalculei os hashes dos 56 artefatos e repeti a verificacao: `PASS`, 56/56 hashes, catalogo inalterado com 13.151 arquivos, 543 PEs correlacionados, 52 manifests/11.782 arestas, quatro scripts Node aprovados na verificacao sintatica e seis scripts PowerShell sem erros de parse.
- Proxima acao: confirmar os termos/autorizacoes aplicaveis e usar uma sessao com permissao para gravar a configuracao Codex fora deste workspace. Ate la, analise de codigo e dinamica permanecem suspensas; prosseguir com validacao documental e de evidencias locais.

- Recataloguei agora as quatro arvores de software: 13.151/13.151 arquivos, 2.928.133.977 bytes, zero erros; comparacao por caminho/tamanho/SHA-256 com a referencia: zero adicionados, removidos ou alterados. `CAD-Data` ficou excluida. Evidencia atualizada em `artifacts/catalogo-recheck/` e `artifacts/validacao-catalogo-recheck.json`.

## Runtimeconfig e framework declarado - 2026-10-17

- Analisei os 18 `*.runtimeconfig.json` das quatro arvores de software com `scripts/analisar-runtimeconfig.mjs`. Todos foram parseados; cada um declara `net8.0`, `Microsoft.NETCore.App` 8.0.0, `Microsoft.WindowsDesktop.App` 8.0.0, `RollForward=LatestMinor` e `System.Runtime.Serialization.EnableUnsafeBinaryFormatterSerialization=true`. Cinco tambem declaram ASP.NET Core 8.0.0.
- Correlacionei o SHA-256 de cada runtimeconfig com o catalogo. Nenhum executavel foi carregado; `CAD-Data` e dados de perfil nao foram acessados. A configuracao nao prova que `BinaryFormatter` seja chamado nem que haja desserializacao de entrada nao confiavel. A documentacao Microsoft alerta que a API e insegura; registrei o limite e a recomendacao de confirmacao ao fornecedor em `docs/04-analise-estatica.md`.
- Evidencia e reprodutor: `artifacts/runtimeconfig-summary.json`, `scripts/analisar-runtimeconfig.mjs`. Na verificacao daquela etapa, o manifesto validou 58/58 hashes; depois a inclusao da analise estrutural de configuracoes ampliou o conjunto para 60 artefatos.

## Estrutura de configuracao distribuida - 2026-10-17

- Selecionei 46 arquivos do software: 30 `.config`, 14 XML `defaultsettings`/`uploadsettings` e dois `appsettings.json`. O parser local @xmldom/xmldom 0.9.12 interpretou 42 XML e dois JSON, sem erros; dois `.config` do plugin `webviewExport` continuam sem formato identificado.
- O resumo registra 382 nomes de elementos XML, 47 nomes de atributos e 17 caminhos/tipos de chaves JSON, sem valores escalares. Nomes relacionados a DentalShare, FTP, proxy, ScanImport e logging aparecem nos esquemas; a presença não confirma valores ou integrações ativas. Os SHA-256 dos 46 arquivos foram reconciliados com o catálogo.
- Evidência e reprodutor: `artifacts/configuration-structure-summary.json`, `scripts/analisar-configuracoes-estrutura.mjs`. A análise leu apenas as quatro árvores de software catalogadas, sem `CAD-Data` ou configurações do perfil.
- Verificação integrada final: `PASS`, 60/60 hashes, 46 arquivos de configuração correlacionados, 44 parseados, dois formatos desconhecidos, sem erros; seis scripts Node (incluindo o verificador) e seis scripts PowerShell passaram suas verificações sintáticas.

## Verificação de malhas STL/OFF - 2026-10-17

- Analisei 172 meshes distribuídos em `DentalCADApp` e comparei seus hashes/tamanhos com o catálogo: 172/172 coincidem, sem erros.
- As 95 STL passaram verificações limitadas de layout: 86 binárias pelo comprimento `84 + 50 × facetCount` e nove ASCII por contagem consistente de `facet`, `outer loop`, `vertex`, `endloop` e `endfacet`; total agregado 1.554.401 facets. Coordenadas e comentário/header não foram armazenados.
- As 77 OFF exibem `OFF BINARY`. A documentação do fabricante confirma suporte ao tipo binário, mas não fornece nessa página o layout do payload; não decodifiquei a estrutura. A referência Geomview consultada cobre OFF ASCII, que não foi assumido equivalente.
- Reprodutor/evidência: `scripts/analisar-malhas-publicas.mjs`, `artifacts/mesh-format-summary.json`. O parser não toca `CAD-Data`; próximo passo para OFF binário requer especificação pública aplicável ou autorização para decodificação.
- Verificação integrada final após incluir a análise de malhas: `PASS`, 62/62 hashes, 172/172 meshes correlacionados ao catálogo, seis scripts de análise e o verificador passaram `node --check`; seis scripts PowerShell passaram o parser.

## Atualização da validação STL - 2026-10-17

- Refinamento do parser: layout binário STL permanece marcado estruturalmente válido se o comprimento 84 + 50 × facetCount confere. Resultado: 95/95 layouts válidos, 94 arquivos com todos os campos float finitos e um arquivo com 90 campos não finitos, todos em normais (0 em vértices), registrado em Warnings, sem classificar como corrupção. Hash e contagem de campos foram verificados sem registrar valores/coordenadas.
- Resumo regenerado em `artifacts/mesh-format-summary.json`; conferência integrada PASS, 62/62 hashes, sete scripts Node aprovados em `node --check` e seis scripts PowerShell sem erros de parse.

## Confirmacao da entrada do grafo REA - 2026-10-17

- Consultei `rea project-managed-application-graph --help` no CLI 6.5.0 local. O comando exige `<inputJson>` e descreve o insumo como managed static Evidence autenticada. O grafo `.deps.json` produzido nesta investigacao e evidencia de declaracoes de deploy, nao foi produzido pelo fluxo REA de analise estatica gerenciada, e nao presumi compatibilidade entre os esquemas. Nao executei o comando contra Exocad nem analisei binarios nesta etapa. Reproducao: `artifacts/rea-cache/runtime/node_modules/rea-agents/scripts/rea.mjs project-managed-application-graph --help`; resumo em `docs/03-plano-rea.md`.
- O achado de 90 floats nao finitos foi reconciliado com a evidencia por arquivo: os 90 pertencem ao campo normal e zero a vertices. O verificador exige arquivo, tipo de aviso e distribuicao exatos.
- Validacao final: PASS, 62/62 hashes, zero divergencias no manifesto; sete scripts Node passam `node --check`, seis scripts PowerShell sem erros de parse.

## Busca textual de termos de licenca - 2026-10-17

- Ampliei a busca anterior por nome: o novo script leu 388 arquivos textuais catalogados (.txt/.md/.rtf/.html/.htm, ate 10 MiB), confirmou hash/tamanho de cada candidato e guardou somente caminhos, hashes e categorias de frases. Foram 82 matches, todos em OpenSource.dll-LICENSES ou FreePalmerLicense.txt; zero erros e zero matches fora desses locais. PDFs, XML/JSON, recursos e binarios ficaram excluidos. Isso nao localiza nem exclui a EULA do produto.
- Evidencias/reprodutor: `artifacts/licensing-content-scan.json`, `scripts/analisar-avisos-legais.mjs`; descricao em `docs/03-plano-rea.md`, `docs/08-validacao.md` e `RELATORIO_FINAL.md`.

- Conferi disponibilidade de `pdftotext`, `mutool` e `qpdf` no PATH: nenhum foi encontrado. O catalogo das quatro arvores de software contem zero PDF; nao foi necessario instalar extrator. XML/JSON, recursos internos e binarios seguem fora da busca textual.

## Duplicatas por hash SHA-256 - 2026-10-17

- Agrupei os hashes existentes no catálogo das quatro árvores (sem reabrir arquivos da instalação): 13.151 entradas, 8.995 hashes distintos, 1.603 grupos duplicados, 5.759 caminhos e 4.156 cópias extras. 389 grupos atravessam mais de uma árvore; nenhum grupo apresenta tamanhos diferentes.
- Reprodutor/evidência: `scripts/analisar-duplicatas-catalogo.mjs` e `artifacts/duplicate-hash-summary.json`. Acrescentei validação caminho/hash/tamanho e nota de limite em `docs/04-analise-estatica.md` e `docs/06-arquitetura.md`.
- Validação integrada após inclusão dos dois artefatos: PASS, 66/66 hashes registrados; nove scripts Node incluindo o verificador passam `node --check`, e seis scripts PowerShell não têm erros de parse. O manifesto e a cobertura de caminhos foram reconciliados.

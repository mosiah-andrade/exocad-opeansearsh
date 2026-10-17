# Relatório técnico — Exocad (parcial)

**Estado:** investigação parcial em 17/10/2026. Este relatório não afirma conclusão de análise de código ou de comportamento em execução. Nenhum arquivo na instalação foi modificado.

## Resumo executivo

O alvo indicado contém uma instalação identificada por metadados como linha Exocad DentalCAD 3.3 SR1, com executável principal FileVersion `3.3.9512.15246`; nomes dos SBOMs também carregam `3.3-SR1`, `net8` e data 2026-01-16. O pacote contém quatro árvores principais (`DentalCADApp`, `DentalDB`, `exoprint`, `LibraryManager`) e uma árvore `CAD-Data` grande com dados potencialmente clínicos. O catálogo detalhado e hashes cobrem os quatro componentes de software, não os dados de caso. Foram produzidos artefatos de inventário e documentação de evidências.

Uma leitura de configuração distribuída encontrou `System.Runtime.Serialization.EnableUnsafeBinaryFormatterSerialization=true` em 18 runtimeconfigs. É uma opção estática, não prova uso de `BinaryFormatter` nem vulnerabilidade; a Microsoft, porém, recomenda não usar a API por seus riscos de segurança ([guia oficial](https://learn.microsoft.com/dotnet/standard/serialization/binaryformatter-security-guide)). A recomendação é confirmar com o fornecedor se existe uso e quais dados podem chegar a essa API.

Uma análise estrutural, com valores redigidos, parseou 44 arquivos distribuídos de configuração: 42 XML e dois JSON; outros dois `.config` permanecem sem formato identificado. Foram observados nomes de campos para DentalShare/FTP, proxy, scanner/importação e logging. Isso identifica superfícies configuráveis no pacote, sem estabelecer valores, credenciais ou integração ativa.

O CLI REA 6.5.0 está instalado em staging no workspace, recuperado do CACache npm local; `doctor` e `setup --client codex --dry-run --json` foram executados. O dry-run planejou uma única atualização de `C:\Users\labor\.codex\config.toml` para registrar o MCP, com backup `C:\Users\labor\.codex\config.toml.rea.backup`; `appliedActions` ficou vazio. Portanto REA ainda não está conectado. O destino está fora da raiz gravável desta sessão. O dist-tag `latest` não pôde ser resolvido devido a `EACCES`, logo não se afirma que 6.5.0 seja a versão latest. Não foi feita engenharia reversa de código, execução do Exocad, captura de rede ou leitura de casos. A documentação pública do fabricante contém restrições relevantes, mas não foi possível identificar o contrato específico desta instalação. As próximas etapas que decodificam código ou observam execução dependem de autorização aplicável e de dados de teste sem informações de pacientes.

## Escopo, autorização e privacidade

- Alvo de software: `C:\exocad-DentalCAD3.3-SR1-2026-01-16`.
- Somente leitura no alvo; saídas em `artifacts/`, `docs/` e `scripts/` deste workspace.
- `CAD-Data` e AppData do usuário têm projetos DICOM/CAD, DB/journals, logs e arquivos `.key`. Foram usados apenas totais agregados; nenhum conteúdo, nome de caso, basename de log/chave ou hash de dado clínico/licença foi exportado.
- Nenhum arquivo local foi transmitido a serviço externo. A consulta npm transmitiu somente uma requisição genérica para resolver o pacote `rea-agents`, sem conteúdo do Exocad.
- Os termos públicos do fabricante variam por produto/território/distribuidor e não identificam a licença instalada.
- Uma busca apenas por nomes de arquivos de termos/licenças nas quatro árvores de software encontrou 128 notices de terceiros e duas cópias de `FreePalmerLicense.txt`; não identificou EULA do produto. O escopo limitado não prova ausência de termos com outros nomes ou incorporados em outros artefatos. Ver `artifacts/licensing-notice-scan.json`.
- A busca textual local de 388 documentos (TXT/MD/RTF/HTML/HTM, ate 10 MiB) teve 82 matches, todos em notices de terceiros ou FreePalmer; nenhum fora desses caminhos. o catalogo nao contem PDFs; XML/JSON, recursos e binarios ficaram fora, portanto a EULA aplicavel segue nao identificada. Ver `artifacts/licensing-content-scan.json` e `scripts/analisar-avisos-legais.mjs`.

## Ambiente analisado

Windows 10 Home Single Language, 25H2, build 26200.9711, 64-bit; PowerShell 5.1; host .NET 8.0.21 x64. WMI negou acesso, mas versão/arquitetura foram lidas de APIs locais e Registro. Node.js v24.20.0 e npm/npx 11.19.0 estão instalados; `npx.ps1` foi bloqueado por política, e `npx.cmd` não acessou o registry. `python.exe` alias foi negado; `py`, git e ferramentas de análise binária listadas em `docs/01-inventario.md` não foram localizados no PATH.

## Versão e componentes

`DentalCADApp.exe` declara FileVersion/ProductVersion `3.3.9512.15246`; `DentalDB.exe` declara `3.3.9512.15701`; exoprint e LibraryManager declaram FileVersion na linha build 9512. Quatro nomes CycloneDX incluem `3.3-SR1-64bit-net8-2026-01-16`. A entrada de desinstalação localizada é `exocadFramework 2025-10-24` (1.3.25.4), distinta do nome do pacote analisado.

Árvores do pacote: DentalCADApp (10.533 arquivos/2.525.952.542 bytes), DentalDB (1.813/210.532.633), exoprint (432/98.476.490), LibraryManager (373/93.172.312) e CAD-Data (1.021/4.794.178.979; não detalhado). O catalogador analisou 13.151 arquivos/2.928.133.977 bytes dos quatro componentes de software; 543 PEs, 24 EXEs e 519 DLLs. O inventário separado `exocadFramework` contém 9 arquivos/343.105.624 bytes, incluindo instaladores Microsoft .NET/ASP.NET/Windows Desktop 8.0.21, WebView2 e VC++.

## Metodologia e análise estática

PowerShell/.NET para a primeira captura, `FileVersionInfo`, `AssemblyName.GetAssemblyName` (somente metadata) e leitura do campo Machine PE. Em seguida, ferramentas Node.js sem dependências externas repetiram a enumeração/hashes e analisaram manifests `.deps.json`; PowerShell voltou a parsear estes últimos de forma independente. JSON CycloneDX local foi lido com `ConvertFrom-Json`. Não foram carregados/executados binários durante a análise estática.

Quatro SBOMs CycloneDX 1.6, atribuídos pelo próprio manifesto a SCALIBR, listam 892 entradas de componentes agregadas. Nenhum deles inclui arestas `dependencies`; portanto não estabelece dependências entre módulos. O SBOM guardado em DentalDB tem componente raiz `CopyAndOpen` v1.0, discrepante do nome de arquivo `DentalDB-generic...`.

Adicionalmente, 52 manifests `.deps.json` foram analisados sem carregar assemblies: declaram 3.669 ocorrências de bibliotecas e 11.782 arestas, todas resolvidas por chave exata dentro do target de cada manifest. O inventário contempla 5 manifests de DentalCADApp, 44 de DentalDB, 1 de exoprint e 2 de LibraryManager. Nos quatro manifests dos executáveis principais, há respectivamente 13, 24, 4 e 1 dependências diretas classificadas como `project`. Isso documenta referências de build/deploy, não carregamento observado em execução. Detalhes e CSVs em `docs/04-analise-estatica.md` e `artifacts/deps-json/`.

Foram observados 158 PEs x64 e 385 PEs x86. `AssemblyName` identificou 482 assemblies gerenciados. `Machine=x86` não foi tratado como prova de que o componente exige processo x86; CLR flags não foram decodificadas. Uma leitura adicional dos 18 runtimeconfigs confirmou `net8.0`, `Microsoft.NETCore.App` 8.0.0, `Microsoft.WindowsDesktop.App` 8.0.0 e `LatestMinor` em todos, com ASP.NET Core 8.0.0 em cinco. Todos declaram `System.Runtime.Serialization.EnableUnsafeBinaryFormatterSerialization=true`. Isso não prova chamada a `BinaryFormatter`, desserialização de entrada não confiável ou vulnerabilidade; a Microsoft recomenda não usar a API por ser insegura ([guia de segurança](https://learn.microsoft.com/dotnet/standard/serialization/binaryformatter-security-guide)). A seleção efetiva de runtime não foi observada. Em quatro EXEs principais, certificado embutido declara sujeito exocad GmbH e emissor DigiCert; em todos os 543 PE os hashes ainda coincidem com o catálogo. WinVerifyTrust offline, com lifetime restrito, aprovou 211, retornou `TRUST_E_EXPIRED` em 312 e `TRUST_E_NOSIGNATURE` em 20 DLLs. Repetidos sem lifetime flag, os 312 expirados passaram sob política local comum; revogação online e atualidade de timestamp continuam não verificadas. Imports/exports, recursos PE, símbolos, strings, cross references, pseudocódigo e comunicações entre processos não foram investigados.

## Análise dinâmica

Não executada. Nenhuma aplicação foi iniciada, nenhum monitor foi instalado e nenhum pacote/log de AppData foi aberto. Uma consulta por nomes de processo/serviço correspondentes retornou zero correspondências no instante da coleta, sem cobrir nomes alternativos.

## Arquitetura reconstruída

Modelo combina componentes físicos inventariados com fluxo descrito pela documentação pública: DentalDB cria/guarda pasta de projeto, ação Design inicia DentalCAD com job `.dentalProject`; DentalCAD salva cena `.dentalCAD` e pode gerar `_cad.stl`/`.constructionInfo`, depois encaminhar a ferramenta CAM. DentalDB também documenta integrações configuráveis com scanner, CAM, FTP e dentalshare. Essas relações são documentadas para o produto, mas não foram observadas em execução nesta instalação; ativação/entitlement permanecem desconhecidos. Ver `docs/06-arquitetura.md`, `docs/diagramas/dependencias.mmd` e `docs/diagramas/fluxos.mmd`.

Os arquivos distribuídos incluem nomes de campos para DentalShare, FTP, proxy e importadores ScanImport; esses nomes não provam que as integrações estejam configuradas. Valores foram omitidos para evitar exposição de possíveis segredos.

## Dados e interoperabilidade

O catálogo do aplicativo contém 95 `.stl`, 77 `.off` e 8 `.dentalproject`; não contém `.dcm`, `.obj` ou `.ply`. Contagem agregada de CAD-Data encontrou `.dcm`, `.obj`, `.ply`, `.stl`, `.dentalProject` e `.dentalCAD`. Isso prova presença de nomes/extensões, não estrutura dos casos. A documentação oficial do DentalCAD declara suporte de entrada para malhas STL/OFF/PLY/OBJ; Add/Remove Mesh lista STL/OFF ASCII ou binário, OBJ ASCII, PLY e “open system .DCM mesh”. A especificação oficial `.dentalProject` documenta convenções de nomes/arquivos e a Performance Specification recomenda OFF. Documentação de produto também descreve exportação `_cad.stl`/`.constructionInfo`, cena `.dentalCAD`, STL/PDF/HTML e modelos OBJ. O DICOM Viewer é módulo opcional para visualização CT/CBCT DICOM, não para diagnóstico; presença local de executável/DLL não prova que o módulo esteja licenciado/habilitado. Nenhum formato local de caso foi aberto ou testado. Fontes detalhadas em `docs/07-formatos-interoperabilidade.md`.

Nas assets distribuídas, 95 STL passaram checagens estruturais limitadas (86 binárias e 9 ASCII; 1.554.401 facets contados) e hashes/tamanhos concordaram com o catálogo. As 77 OFF exibem `OFF BINARY`, mas seus payloads não foram interpretados. A documentação oficial lista STL/OFF ASCII e binário, enquanto a referência geomview consultada descreve OFF ASCII; portanto não há base usada aqui para inferir a estrutura da variante OFF binária nem seu carregamento local ([exocad Add/Remove Mesh](https://wiki.exocad.com/wiki/index.php/Add/Remove_Mesh), [Library of Congress STL Binary](https://www.loc.gov/preservation/digital/formats/fdd/fdd000505.shtml), [Geomview Manual](https://geomview.sourceforge.io/docs/geomview.pdf)).

## Hipóteses e validação

- **Fortemente sustentada:** o diretório contém componentes Exocad build 3.3.9512.x e o pacote é rotulado 3.3 SR1 nos SBOMs.
- **Confirmada como contagem/assinatura:** 13.151 arquivos de software catalogados; hashes completos e sem erro; `CAD-Data` ausente do catálogo.
- **Reproduzida independentemente:** o catálogo Node.js coincidiu com o PowerShell em 13.151 caminhos, tamanhos, SHA-256, 543 máquinas PE e 482 classificações gerenciadas; zero diferenças, zero erros, mesmos 1.603 grupos de hash.
- **Confirmada como conteúdo declarado pelo SBOM:** 892 componentes em quatro SBOMs CycloneDX; dependências entre eles não declaradas.
- **Confirmada como declarações de manifests .NET, não como runtime:** 52 `.deps.json`, 11.782 arestas resolvidas dentro dos targets; zero erros de leitura/parse. A contagem de arestas não representa módulos efetivamente carregados.
- **Confirmada como configuração distribuída:** 18 runtimeconfigs declaram .NET 8.0.0 e Windows Desktop 8.0.0; cinco também ASP.NET Core 8.0.0; todos definem `EnableUnsafeBinaryFormatterSerialization=true`. Uso da API e dados desserializados não foram observados.
- **Confirmada como estrutura de configuração distribuída:** 44 de 46 arquivos selecionados foram parseados sem registrar valores; dois `.config` do plugin `webviewExport` continuam desconhecidos. Nomes de campos relacionados a DentalShare/FTP, proxy, scanner/importação e logging foram observados; estado ativo não confirmado.
- **Fortemente sustentada pela documentação de produto, não confirmada em runtime local:** DentalDB inicia DentalCAD com job `.dentalProject`; scene `.dentalCAD`, saída STL/`.constructionInfo` e encaminhamento CAM.
- **Não confirmada:** quais processos efetivamente chamam quais módulos nesta instalação, comunicação de rede e sequência de execução.
- **Confirmada pela documentação pública, não por teste nesta build:** formatos de entrada/mesh listados acima, convenções do `.dentalProject` e exportação STL/3D PDF/HTML; licenciamento/habilitação local desconhecidos.
- **Confirmada estruturalmente para assets empacotadas:** 95/95 STL passam o parser público limitado; 77/77 OFF exibem assinatura `OFF BINARY`; payload binário OFF e uso em runtime não confirmados.
- **Não confirmada:** licença e autorização que regem esta cópia.
- **Validado:** autocomparação do CSV consigo mesmo produziu 0 adicionados/0 removidos/0 alterados. Ver `artifacts/validacao-autocomparacao.json`.

## Índice de artefatos

- `PLAN.md`, `PROGRESS.md` — estados, critérios e última execução.
- `docs/01-inventario.md` a `docs/08-validacao.md` — evidências e limitações por fase.
- `docs/diagramas/dependencias.mmd`, `docs/diagramas/fluxos.mmd`.
- `artifacts/catalogo-exocad.csv`, `artifacts/catalogo-exocad.json` — 13.151 itens com SHA-256 e metadados de versão; exclui CAD-Data.
- `artifacts/catalogo-exocad-node/` — segunda catalogação independente em Node, com caminhos/hashes idênticos.
- `artifacts/catalogo-recheck/`, `artifacts/validacao-catalogo-recheck.json` — recatalogação atual de 13.151 arquivos e comparação com a referência; zero diferenças por caminho/tamanho/SHA-256.
- `artifacts/framework-exocad.csv` — 9 itens de runtime/framework, com SHA-256.
- `artifacts/duplicate-hash-summary.json`, `scripts/analisar-duplicatas-catalogo.mjs` — 1.603 grupos com hash repetido no catálogo; 4.156 cópias além da primeira; script opera apenas sobre o catálogo.
- `artifacts/sbom-summary.csv`, `artifacts/sbom-components.csv` — resumos de SBOM e 892 componentes declarados.
- `artifacts/deps-json/` — resumo, 52 manifests declarados e 11.782 arestas dos `.deps.json`.
- `scripts/analisar-runtimeconfig.mjs`, `artifacts/runtimeconfig-summary.json` — análise de metadados dos 18 runtimeconfigs; configurações declaradas e SHA-256 correlacionados ao catálogo, sem dados de perfil ou casos.
- `scripts/analisar-configuracoes-estrutura.mjs`, `artifacts/configuration-structure-summary.json` — estrutura redigida de 46 arquivos; 44 parseados, dois formatos desconhecidos, sem valores escalares.
- `scripts/analisar-malhas-publicas.mjs`, `artifacts/mesh-format-summary.json` — verificação limitada de 95 STL e classificação de assinatura de 77 OFF; sem coordenadas ou payload OFF.
- `scripts/validar-evidencias.mjs` — valida integridade, contagens de framework, configuração estática relevante e hashes cruzados dos runtimeconfigs.
- `artifacts/rea-cache/` — metadados/tarball REA 6.5.0 em staging, resultado de `npm ls`, `doctor` e plano JSON do setup dry-run; nenhum registro MCP foi aplicado.
- `artifacts/rea-cache/capabilities-codex.json` — inventário reportado localmente por `rea capabilities --json`; lista providers/capacidades declaradas e efeitos sem analisar binários Exocad.
- `artifacts/rea-cache/mcp-doctor-codex.json` — teste MCP local bem-sucedido: identidade 6.5.0, 207 ferramentas, 6 prompts e request-flow `binary_session`; sem conexão ao Codex.
- `artifacts/licensing-notice-scan.json` — resultado agregado da busca somente por nomes de documentos de licença nas árvores de software, sem ler conteúdo de licenças de terceiros.
- `artifacts/licensing-content-scan.json`, `scripts/analisar-avisos-legais.mjs` — busca categorizada em 388 documentos textuais catalogados; sem trechos e sem conclusão sobre contrato aplicável.
- `artifacts/passive-process-check.json` — consulta pontual de nomes de processos/serviços conhecidos; não é observação dinâmica do comportamento.
- `artifacts/certificados-assinatura-pe.json` — metadados do certificado embutido em quatro EXEs principais, correlacionados por SHA-256; sem validação Authenticode.
- `artifacts/validacao-authenticode-offline.json` — quatro verificações WinVerifyTrust retornaram sucesso contra política local, cache-only, sem revogação pela rede; reprodutor `scripts/verificar-authenticode-offline.ps1`.
- `artifacts/validacao-authenticode-catalogo.json`, `artifacts/comparacao-authenticode-timestamp.json` — status WinVerifyTrust dos 543 PE; 543/543 hashes concordam com o catálogo; rechecagem dos 312 `TRUST_E_EXPIRED` sem lifetime flag retornou sucesso.
- `scripts/verificar-authenticode-catalogo.ps1`, `scripts/comparar-authenticode-timestamp.ps1` — reprodutores da varredura PE completa e da comparação offline de flags.
- `artifacts/validacao-powershell-scripts.json` — parse estático dos seis scripts PowerShell; zero erros.
- `scripts/validar-evidencias.mjs`, `artifacts/verificacao-integrada.json` — reprodutor e recibo de validação integrada local (62 SHA-256 e checagens cruzadas), sem leitura do alvo.
- `artifacts/validacao-autocomparacao.json` — resultado idêntico/sem mudanças.
- `artifacts/validacao-catalogos-independentes.json`, `artifacts/validacao-catalogos-independentes-node.json`, `artifacts/validacao-autocomparacao-node.json` — provas de comparação cruzada e autoconsistência.
- `artifacts/validacao-entregaveis.json` — checagem de contagens, SHA-256, parsing de scripts e existência de artefatos.
- `scripts/catalogar-exocad.mjs`, `scripts/comparar-inventarios.mjs` — reprodutores executáveis via Node; `.ps1` equivalentes também preservados.
- `scripts/validar-evidencias.mjs` — verifica integridade e consistência das evidências atuais: `node scripts/validar-evidencias.mjs`.
- `artifacts/validacao-powershell-scripts.json` — parse de seis scripts PowerShell, zero erros; valida sintaxe, não comportamento.
- `scripts/analisar-deps-json.mjs` — extrai grafo declarado de `.deps.json` sem carregar código.

## Matriz de rastreabilidade

| Conclusão | Evidência |
|---|---|
| Windows 10 Home Single Language 25H2 x64, build 26200.9711 | `docs/01-inventario.md`, leitura de Environment/Registro; WMI negado |
| Build principal 3.3.9512.15246 | Linha `DentalCADApp/bin/DentalCADApp.exe` em `artifacts/catalogo-exocad.csv` e hash registrado em `docs/04-analise-estatica.md` |
| Catálogo cobre só 4 árvores de software, não CAD-Data | Campos `Components`/`Excluded` de `artifacts/catalogo-exocad.json` |
| Catálogo reproduzível independente | `artifacts/validacao-catalogos-independentes.json` e `artifacts/validacao-catalogos-independentes-node.json` |
| Arquivos com SHA-256 repetido no catálogo | `artifacts/duplicate-hash-summary.json`; 1.603 grupos, 5.759 caminhos, 4.156 cópias extras, 389 grupos entre componentes (246 abrangem as quatro árvores, 84 DentalCADApp/DentalDB), tamanhos consistentes |
| Instalação de software permaneceu igual à referência no recatalogamento de 2026-10-17 | `artifacts/validacao-catalogo-recheck.json`; 13.151/13.151, added/removed/changed = 0 |
| 543 PEs; machine x64/x86; .NET 8 | `artifacts/catalogo-exocad.csv`; runtimeconfig em `docs/04-analise-estatica.md` |
| Certificado embutido declara exocad nos quatro EXEs principais (sem validar trust) | `artifacts/certificados-assinatura-pe.json`; SHA-256 de cada binário correlacionado ao catálogo |
| Authenticode validado para quatro EXEs contra política local, sem consulta de revogação online | `artifacts/validacao-authenticode-offline.json`; quatro `0x00000000`, SHA-256 correlacionados ao catálogo; sem claim de freshness |
| Hashes concordam para todos os 543 PE; efeitos dos flags de timestamp registrados | `artifacts/validacao-authenticode-catalogo.json` (543/543 hashes; 211 pass, 312 expired, 20 sem assinatura) e `artifacts/comparacao-authenticode-timestamp.json` (312/312 pass sem lifetime flag) |
| Distribuição de sujeitos dos certificados dos PE, sem atribuir autoria lógica | `artifacts/verificacao-integrada.json`; 220 exocad, 195 Microsoft, 82 .NET/Microsoft, 26 terceiros, 20 sem certificado extraível |
| 892 componentes declarados, sem arestas de dependência | `artifacts/sbom-summary.csv`, `artifacts/sbom-components.csv` e SBOMs originais catalogados |
| Dependências .NET declaradas; sem confirmação de runtime | `artifacts/deps-json/deps-manifests-summary.csv`, `deps-edges.csv`, `deps-analysis-summary.json` |
| Frameworks e opção de compatibilidade de serialização declarados em runtimeconfig | `artifacts/runtimeconfig-summary.json`; 18/18 hashes reconciliados com `artifacts/catalogo-exocad.json`; uso efetivo não observado |
| Nomes de campos de configuração para integrações, sem valores ou estado ativo | `artifacts/configuration-structure-summary.json`; 46/46 hashes/itens catalogados, 44 estruturas parseadas, dois formatos desconhecidos |
| STL/OFF distribuídos; estrutura OFF binária não decodificada | `artifacts/mesh-format-summary.json`; 172/172 hashes correspondem ao catálogo, 95 STL passaram checks estruturais limitados; 94/95 têm floats finitos e uma apresenta 90 campos não finitos, todos em normais (0 em vértices); 77 `OFF BINARY` classificados por assinatura |
| Presença de formatos por extensão, sem suporte funcional provado | Contagens em `docs/07-formatos-interoperabilidade.md`; CAD-Data não foi detalhado |
| REA CLI/MCP 6.5.0 verificados localmente; Codex não conectado | `artifacts/rea-cache/npm-ls.json`, `mcp-doctor-codex.json`, `installation-and-plan.json`, `doctor-codex.json`, `setup-dry-run-codex.json`; MCP local íntegro, inscrição Codex planejada mas `appliedActions` vazio |
| Capacidades REA reportadas localmente; nenhuma análise Exocad pelo REA | `artifacts/rea-cache/capabilities-codex.json`; `analysis_provider_binding=null`, nenhum target aberto |
| Busca limitada não encontrou EULA do produto pelo nome | `artifacts/licensing-notice-scan.json`; 128 avisos terceiros, 2 cópias FreePalmer; filtro explicitamente limitado |
| Busca textual limitada não encontrou termos fora de licenças de terceiros | `artifacts/licensing-content-scan.json`; 388 documentos, 82 matches, zero fora de notices/FreePalmer; não identifica contrato nem cobre XML/JSON/recursos/binários |
| EULA pública Dental 2019 é potencialmente relevante, mas não identificada como contrato desta cópia | `docs/03-plano-rea.md`, `docs/08-validacao.md`; EULA de venda define distribuidor/usuário comercial e cláusula 3.5/§69e; relação contratual do alvo desconhecida |
| Sem análise dinâmica | `docs/05-analise-dinamica.md`; nenhuma execução iniciada |
| Sem correspondência em consulta passiva por nomes conhecidos | `artifacts/passive-process-check.json`; 0 processos/serviços no instante observado, sem inferência de ausência geral |

## Limitações legais e técnicas

Além dos termos da exocad America e do webshop, o fabricante publica uma EULA de venda para licenças Dental, documentalmente datada de março de 2019. Ela define a EULA como parte da venda entre distribuidor e usuário final, restringe o escopo declarado a usuários comerciais e a cláusula 3.5 limita decompilação/desmontagem/engenharia reversa, ressalvada a §69e da lei alemã. A página oficial brasileira informa que distribuidores empacotam o software em sistemas customizados. Não foi possível identificar produto/bundle, upgrade, território, distribuidor ou Sales Agreement desta cópia instalada; esses documentos públicos não provam sua aplicabilidade. Por isso, análise de código permanece suspensa até confirmar o contrato e a autorização relevantes; isto não é conclusão jurídica sobre o usuário. Fontes oficiais: [EULA exocad GmbH para venda de licenças Dental (2019)](https://bcontent.exocad.com/secure_area/exocad_GmbH_EULA_DentalCAD-en.pdf), [revendedores oficiais no Brasil](https://exocad.com/resellers/Brazil), [EULA exocad America](https://bcontent.exocad.com/content/T_and_C/exocad_America_-_Non-Medical_Device_Software_Perpetual_EULA.pdf), [termos do webshop](https://shop.exocad.com/us_en/agb).

REA 6.5.0 foi instalado em staging do workspace a partir de tarball npm já presente no CACache, sem executar scripts de lifecycle nem instalar dependências opcionais. SHA-256 do tarball: `531717a06e92ca651aab52691190f916fac044272c01f875e22c5e293218e699`; npm registrou integridade SHA-512. O `doctor` identifica skill v42 alinhada e 207 ferramentas no catálogo, mas também ausência de registro Codex, providers opcionais ausentes e falha Win32 5 no autoteste Windows do Jadx. Dry-run propõe adicionar o MCP com comando Node local e preservar configuração existente, sem variáveis adicionais; nenhuma ação foi aplicada. `latest` não foi resolvido, pois registry retornou `EACCES`. A configuração alvo é externa à raiz gravável, portanto REA permanece desconectado até executar o plano em contexto que possa gravar esse arquivo. A execução do catalogador PowerShell segue bloqueada pela política efetiva; a versão Node é executável sem alterar essa política. Catálogo, metadados, SBOM, diagrama e análise dinâmica ainda estão parciais.

## Próximas ações

1. Confirmar qual contrato/distribuidor/território rege a instalação e se autoriza engenharia reversa para a finalidade pretendida.
2. Confirmar que existe dataset sintético/autorizado antes de dinâmica ou análise de formatos de projetos.
3. Aplicar o plano REA já apresentado em um ambiente que possa gravar `C:\Users\labor\.codex\config.toml`; reiniciar/reconectar Codex e conferir as ferramentas MCP anunciadas.
4. Se autorizado: usar REA para metadados PE/.NET e relacionamentos e atualizar a arquitetura com IDs Evidence; não instalar providers extras sem necessidade.
5. Testar apenas interfaces oficiais com dados sintéticos autorizados e concluir o relatório somente quando as perguntas do plano estiverem respondidas ou explicitamente limitadas.

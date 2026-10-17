# Plano de investigação do Exocad

Alvo indicado: `C:\exocad-DentalCAD3.3-SR1-2026-01-16`.

Princípios: somente leitura no alvo; nenhuma alteração na instalação; saída e scripts em `docs/`, `scripts/` e `artifacts/`; conteúdo de `CAD-Data` é potencialmente clínico e não será copiado, aberto ou detalhado no catálogo enquanto não houver base clara de que é dado sintético/de teste. Licenciamento, autenticação e proteções ficam fora do escopo de bypass.

| Fase | Critério | Estado |
|---|---|---|
| 1. Reconhecimento do ambiente | OS, permissões observáveis, árvore do alvo, processos/serviços e ferramentas registrados | Concluído parcialmente |
| 2. Catálogo e integridade | Inventário reproduzível com caminho relativo, tamanho, data, identificação, metadados e SHA-256; dados potencialmente clínicos protegidos | Concluído parcialmente (hashes agrupados: 1.603 grupos com duplicidade em 13.151 arquivos; cópias exatas por SHA-256 no catálogo identificadas) |
| 3. Estratégia REA | Capacidades confirmadas em instruções da skill/ferramenta; plano por formato e limitações | Concluído parcialmente (CLI/MCP 6.5.0 e entrada do grafo gerenciado conferidos; licença da instalação ainda não identificada) |
| 4. Análise estática | PE/.NET/configuração/formato analisados por mecanismos disponíveis; observação separada de inferência | Concluído parcialmente (metadados PE/.NET, SBOMs, 52 manifests `.deps.json`, 18 runtimeconfigs, 46 estruturas de configuração, certificados e Authenticode offline em 543 PE; imports/código não analisados) |
| 5. Análise dinâmica | Só se houver autorização/condições seguras; sem iniciar ou perturbar processos por conta própria | Pendente |
| 6. Arquitetura | Componentes e relações apoiados por evidência; diagramas Mermaid | Concluído parcialmente |
| 7. Formatos/interoperabilidade | Tipos efetivamente observados e interfaces documentadas sem abrir conteúdo clínico | Concluído parcialmente (95 STL passaram checks estruturais limitados; 94/95 com campos float finitos, uma com 90 campos não finitos registrados como aviso; 77 OFF identificados como `OFF BINARY`, payload não decodificado) |
| 8. Validação | Hipóteses com método, evidência e confiança | Concluído parcialmente |
| 9. Reprodução | Script documentado e comparação/hash não destrutivos | Concluído parcialmente (catálogo independente, grafo `.deps.json`, parsers de runtimeconfig/configuração/malhas e verificador integrado reproduzíveis; PASS atual) |
| 10. Relatório final | Relatório com limites e rastreabilidade; somente afirmações verificadas | Concluído parcialmente |

## Decisões/autorização pendentes

- REA 6.5.0 está instalado em staging local após recuperação do cache npm. `doctor` e `setup --client codex --dry-run --json` foram executados; o plano concreto adiciona a inscrição MCP em `C:\Users\labor\.codex\config.toml`, com backup `.rea.backup`, sem ações aplicadas. A alteração permanece pendente: o destino está fora da raiz gravável da sessão. Requer ambiente com permissão de escrita nesse arquivo; aprovação conversacional isolada não altera o limite do sandbox. O registry não resolveu `latest` (`EACCES`), logo a versão usada não é declarada como latest.
- A pasta `CAD-Data` contém arquivos `.dcm`, `.dentalCAD`, `.dentalProject`, malhas e projetos individuais; é conteúdo potencialmente clínico. Só farei contagem agregada e não registrarei nomes ou hashes desses arquivos.
- A autorização/licença do software não pode ser inferida pelo simples acesso ao diretório. A análise estática não destrutiva continua limitada a metadados e conteúdo distribuído localmente; nenhuma ativação ou proteção será investigada para contorno.
- Há uma licença pública do fabricante com restrições de engenharia reversa, mas a cópia instalada pode ter contrato/território/distribuidor diferente. Análise de código (decompilação/disassembly/extrair conteúdo interno) e observação dinâmica de comportamento ficam pendentes de confirmação da autorização aplicável.

## Atualizacao local - 2026-10-17

- Dry-run do REA 6.5.0 repetido localmente: uma unica acao de registro Codex planejada; nenhuma acao aplicada. A tentativa de resolver `rea-agents@latest` falhou com `EACCES` no registry npm.
- Verificacao integrada de artefatos repetida: 62/62 hashes, catalogo PASS com 13.151 arquivos, 52 manifests/11.782 arestas, 18 runtimeconfigs, 46 estruturas de configuracao e 172 meshes classificados; sem erros de sintaxe nos seis scripts de analise Node e no verificador, nem nos seis scripts PowerShell.
- Analise de codigo, observacao dinamica e gravacao de `C:\Users\labor\.codex\config.toml` continuam pendentes das autorizacoes/limites descritos acima.

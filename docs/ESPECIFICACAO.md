# Especificação da API inicial - Pessoa 6 - Vlad Tairovit

Versão 0.1.0-inicial. 12/10/2026. GrowUp | Accenture.

## 1. Limite de responsabilidade

Decisão explícita do usuário: Pessoa 6 não é responsável por Qiskit nem por ChromaDB.
É responsável pela interface de comunicação com os serviços da equipe, contratos JSON,
validações, falhas, OpenAPI, fluxo e metodologia de cálculo. Nenhuma funcionalidade além
do escopo inicial deve ser incluída sem nova solicitação.

As responsabilidades de armazenamento vetorial, embeddings, curadoria, busca semântica,
análise/otimização de circuitos e geração com IA continuam nos serviços externos.
servicos.py é o ponto de conexão; atualmente fornece somente uma simulação em memória.

Esta entrega preserva os requisitos da Pessoa 6. Não declara concluídos os requisitos
funcionais do sistema inteiro, que dependem das outras frentes.

## 2. Especificação e endpoints

Base local: http://127.0.0.1:8000. Formato normal: application/json.
Upload: multipart/form-data. OpenAPI 3.1.0 em openapi.json e /openapi.json.
Documentação interativa: /docs. Seis operações em cinco caminhos.

| Método | Rota | Entrada | Saída | Sucesso |
|---|---|---|---|---|
| GET | /health | Nenhuma | status, mode | 200 |
| POST | /query | input | response, sources, status, request_id, mode | 200 |
| POST | /search | query | chunks, citations, status, request_id, mode | 200 |
| POST | /documents | file no formulário | status, id, filename, mode | 201 |
| GET | /documents | Nenhuma | documents, mode | 200 |
| POST | /evaluate | case_id, evaluator, ratings, falha crítica opcional | notas, média, decisão, id e data | 201 |

As primeiras cinco operações são as do quadro recebido. /evaluate é o único acréscimo,
necessário para demonstrar o cálculo e registrar avaliações exigidos na US-09 e no documento
da Pessoa 6. O quadro não proíbe endpoints adicionais para cumprir esse requisito.

## 3. Contratos de entrada e saída

input: texto obrigatório, 1..30.000 caracteres. Pode conter pergunta, código ou circuito
representado como texto. A API confere o formato; não executa nem valida tecnicamente Qiskit.
query: texto obrigatório, 1..2.000 caracteres. Campos desconhecidos são rejeitados.

file: documento textual UTF-8, .txt/.md/.py/.qasm, não vazio e até 256 KiB. A API valida
transporte/formato e encaminha bytes e nome ao serviço documental. Não faz ingestão real,
chunking, embedding ou gravação Chroma. PDF/OCR não foram definidos como formatos obrigatórios
pelas fontes e não são implementados nesta versão.

Cada fonte tem document_id, chunk_id, source e content. O serviço externo deverá fornecer
esses identificadores, mapeando seus dados internos. Não é necessário expor os embeddings.
A modelagem interna da collection qiskit_knowledge_base continua com a Pessoa 5.

/query e /search retornam status ok ou insufficient_context. Listas vazias indicam ausência
de contexto; o demo orienta enviar material relacionado, sem inventar resposta técnica.
A resposta demonstrativa contém SIMULAÇÃO e mode=demo. A API não atribui confiança numérica.

A futura integração deve devolver os mesmos contratos. Falta de campos ou tipos incompatíveis
na resposta de serviço produz 502. O modo integrated está reservado no schema, mas nenhuma
implementação real desse modo está incluída nesta entrega inicial.

## 4. Ponto de conexão

| Método assíncrono de servicos.py | Responsabilidade da equipe externa | Retorno contratado |
|---|---|---|
| consultar(texto) | Agente chama RAG, Qiskit quando necessário e LLM | ResultadoConsulta |
| buscar(query) | Recuperar contexto na base vetorial | ResultadoBusca |
| adicionar_documento(filename, content) | Preparar/indexar documento | Documento com id e filename |
| listar_documentos() | Consultar o catálogo de documentos | Lista de Documento |

Os modelos estão em contratos.py. Falha de dependência deve ser traduzida em FalhaServico
com status 502 ou 503, sem devolver segredos ao usuário. Todas as chamadas têm limite de
10 segundos na API. A implementação real deverá respeitar cancelamento e timeouts próprios.
URL, credenciais, protocolos e modelos só serão implementados quando fornecidos pela equipe
e autorizados pelo usuário. Não há um endpoint comercial presumido.

O simulador armazena arquivos em memória, procura palavras com pelo menos quatro caracteres,
retorna até três correspondências e exibe até 600 caracteres do documento. É deliberadamente
simples: não ranqueia relevância semântica, não otimiza código e não gera texto por IA.
Os dados simulados somem ao reiniciar. Use um único processo Uvicorn para a demonstração.

## 5. Erros e fluxo

Todo erro tem request_id, code, message e fields. O header X-Request-ID permite correlacionar
a chamada com a resposta. Os detalhes de validação indicam campos, sem repetir entrada sensível.

| HTTP | Situação | Ação esperada |
|---|---|---|
| 400 | Requisição/formulário malformado | Corrigir o envio |
| 404 / 405 | Rota inexistente / método incorreto | Conferir rota e método |
| 413 | Arquivo maior que 256 KiB | Reduzir o arquivo |
| 415 | Content-Type ou extensão inadequados | Usar JSON ou multipart conforme a rota |
| 422 | Dados vazios, campos inválidos, encoding ou notas incorretas | Corrigir a entrada |
| 500 | Falha interna ou gravação da avaliação | Não assumir sucesso; verificar operação |
| 502 | Erro ou resposta incompatível do serviço externo | Rever contrato da integração |
| 503 | Serviço externo indisponível | Verificar a dependência |
| 504 | Serviço excedeu 10 segundos | Verificar o processamento externo |

Insuficiência de contexto é resultado 200, não falha HTTP. Não há autenticação nesta versão
local nem promessas de uso público seguro. A API escuta 127.0.0.1. O limite de arquivo é
validado após o parser multipart; não representa uma cota global de requisições.

Fluxo: usuário envia -> API valida -> chama serviço -> confere retorno -> devolve JSON.
Na avaliação, recebe notas -> valida -> calcula -> grava registro -> retorna resultado.
O diagrama completo está em SEQUENCIA.md, incluindo o que pertence às outras frentes.

## 6. Metodologia de cálculo

A fonte de backlog define cinco critérios, notas 0..5, média mínima 4 e revisão obrigatória
em falhas críticas. Mantidos: pertinencia, coerencia, aderencia, clareza e falhas.

| Critério | Questão a observar |
|---|---|
| Pertinência | O contexto é relevante para a entrada? |
| Coerência | A resposta é tecnicamente consistente? |
| Aderência | As afirmações se sustentam nas fontes? |
| Clareza | A explicação é organizada e compreensível? |
| Falhas | Limitações e incertezas são tratadas adequadamente? |

Escala operacional: 0 não atende; 1 muito insuficiente; 2 insuficiente; 3 parcialmente adequado;
4 adequado; 5 plenamente adequado. Essas âncoras e pesos iguais são propostas operacionais.
Cada nota é inteira e exige evidência textual de 5..1.000 caracteres. Cinco critérios sem repetir.

Média = (nota1 + nota2 + nota3 + nota4 + nota5) / 5.
Aprovação: média >=4 e nenhuma falha crítica. Abaixo de 4: reprovado. Com falha crítica:
revisao_obrigatoria, independentemente da média. Não há score extra em porcentagem.

O avaliador pode informar critical_failure=true com critical_evidence obrigatório.
Mantém-se como proposta operacional a detecção automática de nota zero em coerência,
aderência ou falhas. A API registra a origem automática na evidência quando aplicável.

Exemplo: [4,4,4,4,4] -> média 4 -> aprovado, sem falha crítica. [3,4,4,4,4] -> 3,8 -> reprovado.
[5,0,5,5,5] -> média 4, mas revisão obrigatória. Os exemplos não são avaliações reais da IA.

Registro: avaliacoes.jsonl, uma linha JSON por avaliação, com id UUID, data UTC, versão da
rubrica inicial-1.0, case_id, avaliador, notas/evidências, média, falha crítica e decisão.
case_id é definido pelo avaliador; não existe consulta de análise em banco para verificar o caso.
O avaliador deve conservar entrada, resposta e fontes do caso para sustentar suas evidências.
Não incluir segredos nesse arquivo. Ele não é enviado automaticamente a serviços externos.

O cálculo é reproduzível; a atribuição de notas exige julgamento. Para avaliar de verdade,
fixar caso, corpus/modelos e condições, registrar a saída, aplicar a rubrica com evidências e
revisar divergências. Falhas HTTP entram nos testes de robustez, não em notas inventadas.
Aprovar uma avaliação não significa homologar todo o produto.

## 7. Rastreabilidade dos requisitos da Pessoa 6

| Entregável exigido | Onde está | Situação |
|---|---|---|
| Especificação da API | Este documento | Entregue |
| Endpoints e métodos | main.py, tabela e OpenAPI | Entregue |
| Requests/responses e validação | contratos.py e exemplos | Entregue |
| OpenAPI 3.1/JSON | openapi.json | Entregue e validado |
| Falhas 4xx/5xx | Handlers e testes | Entregue |
| Diagrama de sequência | SEQUENCIA.md e PDF | Entregue |
| Metodologia de cálculo | /evaluate e seção 6 | Implementada; decisões operacionais identificadas |
| Demonstração e validação | README, testes e evidências | Executada com serviços simulados |

US-01 e US-02: contrato de consulta, validação e erros. US-09: avaliação e registro.
US-03/04: interfaces documentais/busca mantidas; indexação e recuperação semântica são externas.
US-05/06: análise/otimização Qiskit externas. US-07/08: contrato suporta resposta e fontes;
qualidade, justificativas e limites reais são responsabilidade do gerador e avaliadores.
US-10: insuficiência de contexto explícita; detecção de inconsistências técnicas é externa.
Não há declaração de que a simulação cumpre os requisitos funcionais dos serviços ausentes.

As imagens divergem entre cinco/seis épicos e repetem US-02 na tabela resumida. Esta versão
não redefine o backlog global nem renumera histórias. Usa as cinco operações explícitas e
os critérios de avaliação claros; a equipe deve resolver as divergências de organização.

## 8. O que foi simplificado

Retirados desta versão: clientes Chroma/LLM, ingestão real, SQLite, histórico de análises,
rotas duplicadas /v1, filtros, paginação, múltiplos modos configuráveis, chave de acesso e
compatibilidade com avaliações antigas. Nenhum desses detalhes era necessário para a
interface inicial solicitada da Pessoa 6. O registro de avaliações foi mantido em arquivo
para cumprir o requisito de guardar os resultados.

As rotas /documents permaneceram: recebimento/encaminhamento pertence à API, enquanto
organização e persistência reais pertencem à equipe da base. Uma funcionalidade do projeto
que depende de outro responsável não foi implementada sob uma nova responsabilidade sua.

## 9. Fontes

- Documento_Descritivo_Projeto_GrowUp_Accenture.pdf: fluxo e divisão das frentes.
- Modelagem Banco RAG.pdf: responsabilidades da base vetorial e rastreabilidade dos chunks.
- DocDescritivaPessoa6APIFINAL(1).pdf: oito entregáveis da Pessoa 6 e metodologia.
- Duas imagens de backlog de 11/10/2026, recebidas em 12/10/2026: endpoints e escala 0..5.
- Definição expressa do usuário em 12/10/2026: somente sua parte, sem ChromaDB/Qiskit, evolução por solicitação.

Referências técnicas já consultadas: https://fastapi.tiangolo.com/ ;
https://fastapi.tiangolo.com/tutorial/request-forms-and-files/ ; https://spec.openapis.org/oas/v3.1.0 .

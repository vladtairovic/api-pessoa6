# API inicial - Pessoa 6

**Esta é a versão simples, separada da entrega anterior.**
Seu trabalho é a API: receber, validar, encaminhar e devolver dados, documentar erros e
calcular/registrar avaliações. Você não implementa ChromaDB nem Qiskit.

Não acrescente recursos sem solicitação expressa do responsável pela Pessoa 6.
Versão: 0.1.0-inicial. Data: 12/10/2026 (Brasil).

## O que existe

Três arquivos principais, seis operações HTTP e uma documentação OpenAPI 3.1 em JSON.
FastAPI continua sendo o framework. Não é preciso configurar ChromaDB, Qiskit, LLM,
chave de IA, Docker ou banco de dados para testar.

| Arquivo | Responsabilidade |
|---|---|
| main.py | Rotas, validação HTTP, erros, timeout e cálculo/registro das avaliações |
| contratos.py | Campos de entrada e saída, limites e tipos |
| servicos.py | Único ponto de conexão com os serviços da equipe; hoje contém um simulador |
| openapi.json | Especificação formal da API |
| docs/ESPECIFICACAO.md | Requisitos, contratos, erros, avaliação e limites de responsabilidade |
| docs/SEQUENCIA.md | Diagrama de sequência editável |
| docs/Guia_API_Inicial.pdf | Guia resumido da entrega |
| test_api.py | Testes da API, cálculo e falhas |
| exemplos/ | JSONs prontos e documento de demonstração |

## Executar no Windows

1. Extraia o ZIP e abra a pasta `api_pessoa6_inicial`.
2. Confirme que `main.py` e `requirements.txt` estão nessa pasta.
3. Na barra de endereço do Explorador, digite `powershell` e pressione Enter.
4. Confira o Python (validado com 3.12; requer 3.11+):

```powershell
py --version
```

5. Crie o ambiente e instale os pacotes:

```powershell
py -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
```

6. Ligue a API:

```powershell
.\.venv\Scripts\python.exe -m uvicorn main:app --host 127.0.0.1 --port 8000
```

7. Mantenha o terminal aberto e acesse http://127.0.0.1:8000/docs.
8. Para desligar, volte ao terminal e pressione Ctrl+C.

Não precisa ativar o ambiente virtual nem criar `.env`. Não execute os comandos da versão
anterior: esta usa `main:app`. Instalar dependências requer internet. O Swagger também usa
arquivos visuais externos; os endpoints locais funcionam sem internet após a instalação.

No Linux/macOS, substitua `py` por `python3` e `.\.venv\Scripts\python.exe` por `.venv/bin/python`.

## Testar por etapas no Swagger

Em cada operação: clique na rota, em **Try it out**, preencha e clique em **Execute**.

1. GET /health: deve retornar 200, status=ok e mode=demo.
2. POST /documents: selecione `exemplos/documento.txt` em file. Deve retornar 201 e um id.
3. GET /documents: deve listar o documento enviado.
4. POST /search: envie `{"query":"Qiskit"}`. Deve retornar trechos e referências simulados.
5. POST /query: envie `{"input":"Quero consultar o material sobre Qiskit."}`.
   A resposta tem fontes, request_id e mode=demo. O texto informa explicitamente SIMULAÇÃO.
6. POST /query com `{"input":""}`: deve retornar 422, demonstrando a validação.
7. POST /search com `{"query":"astronomiaextraterrestre"}`: sem material correspondente,
   deve retornar 200 com status=insufficient_context e listas vazias.
8. POST /evaluate: copie o conteúdo de `exemplos/avaliacao.json`. Deve retornar média 4,
   approved=true, result=aprovado e registrar a avaliação.

**No primeiro uso não há documentos carregados. Envie o arquivo antes de consultar.**
Os documentos do simulador ficam só na memória e desaparecem ao reiniciar. Não são
indexados em um banco vetorial. A comparação de palavras é apenas uma simulação do serviço.

## Avaliação

Notas inteiras 0..5 para pertinência, coerência, aderência, clareza e falhas. Média = soma / 5.
Média >=4 sem falha crítica aprova; abaixo de 4 reprova; falha crítica obriga revisão.
As notas vêm do avaliador, não de uma IA automática. Os exemplos são fictícios.
O case_id identifica o caso definido pelo avaliador; não é um identificador de análise em banco.

Cada avaliação é gravada em `avaliacoes.jsonl`, criado automaticamente junto de main.py.
É um arquivo textual com um JSON por linha. Ele permanece após reiniciar a API.
Abra-o no editor para consultar os registros. Não há banco de dados ou rota extra de histórico.
A falha de escrita retorna 500: nesse caso a API não confirma que a avaliação foi registrada.

## Testes automatizados (opcionais para executar a API)

Na pasta do projeto:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-testes.txt
.\.venv\Scripts\python.exe -m pytest -q
```

Para atualizar a especificação após uma alteração autorizada:

```powershell
.\.venv\Scripts\python.exe exportar_openapi.py
```

## O que ficou fora desta versão

Não há implementação de Qiskit, ChromaDB, embeddings, LLM, indexação real, SQLite, login,
chave de API, rotas antigas /v1, paginação, filtros avançados ou configuração de serviços reais.
Esses itens não são requisitos da camada inicial definidos para sua responsabilidade.
As cinco operações do quadro foram preservadas e a avaliação exigida continua funcionando.
Os requisitos do projeto inteiro que dependem de IA/banco seguem com seus responsáveis.

A futura conexão será feita em servicos.py, com os endereços e contratos fornecidos pela equipe,
quando você pedir essa inclusão. Nesta versão o ponto de integração existe; não há conexão real ativa.

## Problemas comuns

- Arquivo não encontrado: use `Get-ChildItem`. Precisa aparecer `main.py` e `requirements.txt`.
- py não encontrado: teste `python --version` e use `python` na criação do ambiente.
- Falta fastapi: instale requirements.txt usando o executável da .venv.
- Navegador não abre a API: confirme que o terminal com Uvicorn continua aberto.
- Porta ocupada: troque 8000 por 8001 no comando e no endereço do navegador.
- Busca vazia: envie o documento e use uma palavra com ao menos quatro caracteres presente nele.
- Upload recusado: use .txt/.md/.py/.qasm em UTF-8, não vazio, até 256 KiB.

A versão anterior permanece independente. Esta versão inicial não lê seus bancos nem importa
as avaliações antigas. Use somente esta pasta para aprender e apresentar a parte simplificada.

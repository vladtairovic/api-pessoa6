# Diagrama de sequência - API inicial

O serviço externo representa trabalho das outras frentes. A implementação inicial usa
ServicosDemo no lugar desse serviço. Não há chamada real a RAG, ChromaDB, Qiskit ou LLM.

```mermaid
sequenceDiagram
    actor U as Usuário
    participant A as API Pessoa 6
    participant S as Serviço da equipe
    U->>A: POST /query com input
    A->>A: Validar formato e limites
    alt Entrada inválida
        A-->>U: 422 com campos inválidos
    else Entrada válida
        A->>S: consultar(input)
        Note over S: Agente, RAG, ChromaDB, Qiskit e LLM ficam fora da API
        alt Serviço retorna resultado
            S-->>A: response, sources e status
            A->>A: Validar contrato de saída
            A-->>U: 200, resultado e request_id
        else Serviço indisponível ou lento
            S-->>A: Falha
            A-->>U: 502, 503 ou 504
        end
    end
```

```mermaid
sequenceDiagram
    actor U as Usuário
    participant A as API Pessoa 6
    participant S as Serviço documental
    U->>A: POST /documents com file
    A->>A: Conferir tamanho, extensão e UTF-8
    A->>S: adicionar_documento(nome, bytes)
    Note over S: Indexação e banco reais são responsabilidade externa
    S-->>A: id e filename
    A-->>U: 201 com status
    U->>A: GET /documents
    A->>S: listar_documentos()
    S-->>A: Lista
    A-->>U: 200 com documents
```

```mermaid
sequenceDiagram
    actor V as Avaliador
    participant A as API Pessoa 6
    participant R as Arquivo de registros
    V->>A: POST /evaluate com notas e evidências
    A->>A: Validar cinco critérios e escala 0..5
    A->>A: Calcular média e verificar falha crítica
    A->>R: Acrescentar uma linha JSON
    R-->>A: Gravação concluída
    A-->>V: 201 com média e decisão
```

/search usa buscar(query) no serviço externo. Sem contexto, retorna 200 com listas vazias
e status insufficient_context. Erro inesperado retorna 500 sanitizado. Falha de gravação
da avaliação não retorna sucesso.

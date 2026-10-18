# Validação da API inicial

Executada em 12/10/2026 (Brasil), Python 3.12. Resultado: **30 testes aprovados**, zero falhas.
O único aviso é de depreciação de uma dependência do TestClient; não é falha de execução.

Verificados: fluxo upload/listagem/busca/consulta; ausência de contexto; campos obrigatórios,
limites e campos extras; arquivos vazios/binários/encoding inválido; cálculo de notas 0..5;
limite de aprovação 4; falha crítica com média alta; evidência crítica obrigatória; critérios
duplicados; gravação JSONL; 404/405/415/422; respostas externas 502/503; timeout 504; erro interno
500 sanitizado; OpenAPI 3.1 formalmente válido e igual ao arquivo exportado.

Também foi iniciado Uvicorn em loopback e executado um fluxo HTTP real pelas seis operações.
Evidências: resultado_testes.txt, teste_http.txt e exemplo_respostas.json.
Os UUIDs dos exemplos são somente evidências da execução; não representam registros disponíveis
em uma API nova. A avaliação de exemplo usa notas fictícias, não mede qualidade da IA.

Os testes de falhas usam serviços substitutos controlados. Não foram usados ChromaDB, Qiskit,
LLM, embeddings reais ou infraestrutura da equipe. A validação é da camada da Pessoa 6.
Não foram feitos benchmark de circuitos, teste de carga, auditoria de segurança ou validação
semântica da IA. Esses itens não são anunciados como concluídos.

Para reproduzir:

```powershell
.\.venv\Scripts\python.exe -m pip install -r requirements-testes.txt
.\.venv\Scripts\python.exe -m pytest -q
```

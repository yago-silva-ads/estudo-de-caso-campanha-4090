# Estudo de Caso: Engenharia de Dados e Automação em Campanha Eleitoral (2026)

**Autor:** Yago Santos Silva  
**Data:** Outubro de 2026  
**Tecnologias:** Python, SQLite, Node.js, Meta Graph API, GitHub Actions, Azure (Blob, Static Website), n8n, LLMs (Groq).

---

## 📌 O Desafio
Campanhas digitais geram um volume massivo de interações em curtos períodos de tempo. O desafio era construir, com orçamento quase zero e sendo o único desenvolvedor do projeto, uma infraestrutura capaz de capturar, classificar e visualizar comentários públicos das páginas oficiais do candidato (Facebook e Instagram). 

O sistema precisava gerar indicadores agregados confiáveis de mais de **850 mil comentários** para a tomada de decisão da equipe, tudo isso em conformidade com a LGPD e prazos inegociáveis.

---

## 🏗️ Evolução da Arquitetura

### Fase 1: MVP Low-Code
A primeira versão utilizou **n8n** para consumir a Graph API da Meta, classificar sentimentos via LLM gratuito (Groq) e persistir os dados em planilhas (CRM). Para interações ativas, fluxos automatizados curtiam e geravam respostas.
* **Problema de Escala:** O limite de rate limit da API de IA gerava erros `429 Too Many Requests`, e o armazenamento em planilhas tornou-se instável com o pico de 30 mil comentários em um único dia.

### Fase 2: Pipeline Robusto em Python
Para suportar o tráfego, a arquitetura foi refatorada:
```text
Meta Graph API → pipeline.py → SQLite (comment_id PK) → Azure Blob (canônico)
                                   ↓
                     Classificação Determinística (Motor de Regras)
                                   ↓
              Painel Web QG publicado (Azure Static Website) com validação SHA-256
```
* **Idempotência:** Uso de `comment_id` e `post_id` como chaves. Reprocessar o mesmo lote não duplica dados.
* **Resiliência:** Backoff exponencial implementado para falhas de rede.
* **Processamento Batch:** Agendamento automatizado via GitHub Actions executando scripts a cada 15 minutos.

---

## ⚡ IA vs Motor de Regras: Escalando para 1.000 req/s
Depender de um LLM gratuito limitou a aplicação. A solução de engenharia foi substituir o LLM por um **motor de classificação determinística** (Regex, listas e pontuações de peso). 
* **Resultado:** Desempenho saltou para o processamento de **~1.166 comentários por segundo** rodando localmente sem latência de rede externa, garantindo maior precisão na classificação de "Apoio" versus "Ataque/Crítica".

---

## 📊 Resultados e Métricas Alcançadas

* **Total de Comentários Processados:** 788.763 (Snapshot final: 852.924)
* **Duplicatas:** 0 (Deduplicação garantida em banco)
* **Performance do classificador:** +1.000 comentários/s sem custos adicionais de nuvem.

### Lições Aprendidas na Engenharia de Dados:
1. **Unidades importam:** Separar estritamente métricas de "comentários de apoio" de "pessoas únicas cadastradas".
2. **Ausência não é zero:** Falhas de coleta (como indisponibilidade da API) foram mapeadas como `N/D` no dashboard para não quebrar a confiabilidade da curva de engajamento do cliente.
3. **Low-code acelera, mas tem um teto de escala:** A transição para Python/SQLite foi obrigatória para garantir persistência real e estabilidade.

---
*Este é um repositório de estudo de caso. Códigos-fonte fechados e dados sensíveis de cidadãos (PII) foram omitidos para cumprimento da Lei Geral de Proteção de Dados (LGPD).*

# Supernova Colheita

**Assistente virtual com inteligência artificial e recuperação de informações (RAG básico) para agricultura familiar**

Projeto desenvolvido para a **Global Solution FIAP 2026/2**.

## Objetivo

O Supernova Colheita é um protótipo que auxilia agricultores familiares a consultar informações introdutórias sobre o **Programa Nacional de Alimentação Escolar (PNAE)**, incluindo participação em chamadas públicas, documentação e aquisição de alimentos da agricultura familiar.

O sistema responde em português brasileiro e procura limitar as respostas aos conteúdos recuperados de uma base de conhecimento. Quando não encontra conteúdo suficientemente relevante, informa que não dispõe daquela informação.

## Tecnologias utilizadas

- **Python** e **Google Colab** para desenvolvimento e execução.
- **Google Gemini**, por meio da biblioteca `google-genai`, para geração de respostas.
- **scikit-learn** para vetorização **TF-IDF** e cálculo da **similaridade do cosseno**.
- **JSON** para registrar perguntas, respostas e resultados dos testes.
- **pypdf** instalado como dependência, embora o protótipo atual não faça extração de PDFs.

## Como funciona

O projeto implementa um fluxo simples de *Retrieval-Augmented Generation* (RAG):

1. Uma pergunta é recebida pelo assistente.
2. O texto da pergunta é transformado em vetor TF-IDF.
3. A similaridade do cosseno identifica até dois trechos da base de conhecimento relacionados à pergunta.
4. São considerados relevantes apenas os trechos com similaridade **igual ou superior a 0,20**.
5. Se não houver trechos relevantes, o sistema responde que não encontrou informações na base.
6. Se houver trechos relevantes, eles são enviados como contexto ao Gemini, acompanhado de instruções para não inventar exigências ou informações.
7. A resposta informa os títulos dos trechos recuperados e o modelo utilizado.

O código também tenta modelos alternativos quando ocorrem erros de cliente ou servidor na chamada à API.

**Importante:** a busca utiliza correspondência lexical TF-IDF, não embeddings semânticos nem banco de dados vetorial. A base é pequena e foi escrita manualmente para demonstrar o funcionamento do RAG.

## Base de conhecimento

A versão atual contém **três textos demonstrativos**, organizados em uma lista Python:

| Título | Assunto |
| --- | --- |
| Programa Nacional de Alimentação Escolar | Apresentação geral do PNAE |
| Agricultura Familiar e PNAE | Percentual mínimo de 45% a partir de 2026 |
| Documentação para Participação | Documentação, projeto de venda, exigências sanitárias e chamada pública |

Os textos são cadastrados diretamente no notebook. **Não existe, nesta versão, coleta automática, importação de PDFs ou rastreabilidade por trecho até documentos oficiais.**

### Referências institucionais cadastradas

- [FNDE — Programa Nacional de Alimentação Escolar (PNAE)](https://www.gov.br/fnde/pt-br/acesso-a-informacao/acoes-e-programas/programas/pnae)
- [FNDE — Portal institucional](https://www.gov.br/fnde/pt-br)

Esses links aparecem como referências para consulta. **O código não baixa nem extrai o conteúdo dessas páginas**, e os títulos dos textos recuperados não equivalem a citações verificadas de documentos oficiais. Para decisões reais sobre fornecimento às escolas, consulte a legislação vigente, o edital da chamada pública e as orientações do FNDE.

## Como executar no Google Colab

1. Abra o arquivo `Supernova_Colheita_IA.ipynb` no [Google Colab](https://colab.research.google.com/). Caso o arquivo ainda esteja com outro nome, use o nome correspondente ao arquivo publicado no repositório.
2. No painel **Secrets** (ícone de chave), cadastre uma chave de API Gemini com o nome **`GEMINI_API_KEY_3`**, que é o identificador utilizado pelo notebook atual, e permita o acesso ao notebook.
3. Execute as células na ordem em que aparecem, começando pela instalação das bibliotecas:

   ```python
   !pip -q install -U google-genai pypdf scikit-learn
   ```

4. A célula de configuração lê a chave sem incluí-la diretamente no código:

   ```python
   from google.colab import userdata
   from google import genai

   GEMINI_API_KEY = userdata.get("GEMINI_API_KEY_3")
   client = genai.Client(api_key=GEMINI_API_KEY)
   ```

5. Depois da configuração, execute a criação da base, a vetorização, a definição das funções de busca e de resposta e os testes.
6. O notebook gera o arquivo `relatorio_testes.json` com as respostas e a avaliação automática.

**Ajuste necessário no notebook antes de executar tudo do zero:** na seção **5. Implementação da Busca por Similaridade**, existe um bloco de teste que chama `perguntar_supernova(pergunta)` antes da definição dessa função na seção 7. Remova **somente esse bloco de teste** ou mova sua execução para depois da seção 7. A definição da função `buscar_documentos` deve ser mantida. Sem esse ajuste, uma sessão limpa pode apresentar `NameError`.

A execução requer internet, uma chave Gemini válida e disponibilidade/quota da API. A lista de modelos configurada no notebook pode precisar de atualização conforme o acesso da conta e a disponibilidade do provedor.

## Testes e resultados

O notebook inclui três cenários:

| Cenário | Resultado registrado |
| --- | --- |
| Pergunta sobre documentos necessários para vender às escolas | APROVADO |
| Pergunta sobre o percentual mínimo da agricultura familiar em 2026 | APROVADO |
| Pergunta fora do tema: Copa do Mundo de 2002 | APROVADO |

**Resultado registrado: 3 de 3 testes aprovados (100%).**

O resultado foi obtido com **verificações automáticas por palavras-chave**, não com avaliação humana de precisão jurídica, abrangência ou confiabilidade factual. O conjunto de três perguntas é limitado e não demonstra desempenho em escala. O arquivo `relatorio_testes.json` contém as perguntas, respostas geradas e classificações.

## Limitações e próximos passos

- A base atual tem apenas três textos curtos, redigidos manualmente; é necessário substituí-los ou complementá-los por documentos oficiais verificados.
- Os links do FNDE estão registrados para referência, mas não são processados pelo sistema.
- O limite de similaridade de 0,20 foi escolhido para o protótipo; ele pode rejeitar perguntas válidas ou aceitar trechos pouco pertinentes.
- A geração depende da disponibilidade e das cotas da API Gemini.
- As respostas são informativas e **não substituem a leitura do edital, a legislação vigente nem a orientação dos órgãos responsáveis**.
- Para evolução do projeto: ampliar a base documental, adicionar identificação precisa das fontes, avaliar mais perguntas e registrar métricas de recuperação e qualidade das respostas.

## Segurança

A chave da API deve permanecer nos **Secrets do Colab**, nunca escrita diretamente no notebook, no README ou em arquivos versionados. Antes de publicar, revise também as saídas das células para garantir que não contenham credenciais ou outros dados sensíveis.

## Arquivos do repositório

```text
Supernova-Colheita/
├── README.md
├── Supernova_Colheita_IA.ipynb
└── relatorio_testes.json
```

O nome do arquivo `.ipynb` no exemplo é uma sugestão de padronização; ele deve coincidir com o arquivo realmente enviado ao GitHub.

## Equipe

RM565802 - Kaique Mascarenhas dos Santos
RM563982 - Felipe Augusto Lopes Ferreira

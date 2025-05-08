# Nota Técnica – Construção do Dataset Unificado **ICJ + Educação** (2010‑2023)

> Versão 1.0 — Abril 2025\
> Elaborado por: Tiago Ribeiro dos Anjos

---

## 1. Objetivo

Esta Nota Técnica documenta integralmente o processo de extração, tratamento e integração de quatro bases públicas — financiamento do CNPq (Iniciação Científica Júnior) e estatísticas educacionais do INEP —, resultando na tabela ``. O arquivo consolida indicadores de fomento à ciência, matrículas da educação básica, matrículas do ensino superior e formação docente **por município e ano** (2010‑2023).

A estrutura segue o modelo de Notas Técnicas do *Painel de Fomento em Ciência, Tecnologia e Inovação* do CNPq citeturn1file0, garantindo transparência metodológica e reprodutibilidade.

---

## 2. Fontes de Dados

| Sigla    | Arquivo                                              | Origem / Descrição                                                                                        | Período   |
| -------- | ---------------------------------------------------- | --------------------------------------------------------------------------------------------------------- | --------- |
| **ICJ**  | `ICJ_Municipio_Ano_2010_2023.xlsx`                   | Painel CNPq – bolsas ICJ agregadas por município‑ano (processo descrito na Seção 3.2)                     | 2010‑2023 |
| **EB**   | `MATRICULAS_EB_ENSF_ENSM_ENSMT_2010_2023_LIMPO.xlsx` | *Sinopse Estatística da Educação Básica* (INEP) – matrículas dos anos finais do EF, EM geral e EM técnico | 2010‑2023 |
| **SUP**  | `MATRICULA_ENS_SUP_2010_a_2023_LIMPO.xlsx`           | *Sinopse do Ensino Superior* (INEP) – matrículas de graduação e pós‑graduação lato/stricto sensu          | 2010‑2023 |
| **FORM** | `FORMACAO_PROF_EB_2010_2023_LIMPO.xlsx`              | *Sinopse Estatística da Educação Básica* – formação dos docentes em exercício                             | 2010‑2023 |

Todas as planilhas foram obtidas em formato `.xlsx` diretamente nos portais oficiais (downloads em 02–04 / 2025).

---

## 3. Metodologia

### 3.1. Ambiente de trabalho

- Google Colab (Python 3.10, Pandas 2.2)
- Repositório GitHub **Agregação\_ICJ\_2010\_2023** para versionamento.

### 3.2. Tratamento da base ICJ (fomento CNPq)

1. **Leitura** do arquivo `ICJ_CNPQ_2010_2023_LIMPO.xlsx` (77 965 linhas).
2. **Agrupamento** pelas dimensões `Regiao_Geografica, Unidade_Federacao, Nome_Municipio, Codigo_IBGE, Ano, Sigla_UF, Pais` ⇒ criação da tabela‑fato **ICJ\_Municipio\_Ano\_2010\_2023.xlsx** contendo:
   - `Qtd_Bolsistas` (contagem de linhas);
   - médias e somatórios de `Valor_Rs`, `Valor_USS`, `Benef_Modal/Ano`, `Bolsa/Ano`, `Auxílio/Ano`.
3. **Validação pontual** (conferência manual para Lorena/SP 2010, etc.).

### 3.3. Tratamento das bases INEP

- **MATRÍCULAS EB** – filtros nas variáveis de interesse e padronização do texto de UF/município;
- **ENSINO SUPERIOR** – seleção de indicadores de graduação e pós‑graduação;
- **FORMAÇÃO DOCENTE** – extração de totais por rede e nível de formação.

Todas foram previamente “limpas” (sufixo `_LIMPO`) em Colabs específicos, seguindo a mesma padronização de cabeçalhos e acentuação.

### 3.4. Harmonização de chaves

- `` (7 dígitos) + `` ⇒ chave primária;
- Divergência `Unidade_da_Federacao` (INEP) → renomeada para `Unidade_Federacao`.

### 3.5. União das bases

```python
merged = (
    icj
      .merge(eb,   on=["Codigo_IBGE","Ano"], how="outer")
      .merge(form, on=["Codigo_IBGE","Ano"], how="outer")
      .merge(sup,  on=["Codigo_IBGE","Ano"], how="outer")
)
```

- Utilizado **outer join** para não perder registros exclusivos de alguma fonte.
- Dimensões duplicadas foram conciliadas com `combine_first()`.

### 3.6. Validação final

- Contagem de linhas = **62 481** (município‑ano distintos).
- Amostragem aleatória de 30 municípios comparada às planilhas originais.

---

## 4. Dicionário de Dados (colunas finais)

### 4.1 Dimensões

| Coluna              | Definição                                                         |
| ------------------- | ----------------------------------------------------------------- |
| `Regiao_Geografica` | Macro‑região brasileira onde se localiza a instituição/município. |
| `Unidade_Federacao` | Nome completo da UF conforme IBGE (ex.: *São Paulo*).             |
| `Nome_Municipio`    | Topônimo oficial segundo IBGE (grafia 2023).                      |
| `Codigo_IBGE`       | Código de 7 dígitos (município + dígito verificador).             |
| `Ano`               | Ano‑base da observação (2010 – 2023).                             |
| `Sigla_UF`          | Sigla da UF.                                                      |
| `Pais`              | País de localização da instituição (majoritariamente “Brasil”).   |

### 4.2 Métricas de Fomento (ICJ)

Sufixos `` (média) e `` (soma) em R\$ correntes e US\$.\
Definições idênticas às da Nota Técnica oficial do Painel CNPq citeturn1file0.

| Coluna              | Descrição                                                                       |
| ------------------- | ------------------------------------------------------------------------------- |
| `Qtd_Bolsistas`     | Número de bolsistas ICJ no município‑ano.                                       |
| `Valor_Rs_*`        | Total/média de recursos pagos em reais.                                         |
| `Valor_USS_*`       | Total/média convertidos para dólares à época do pagamento.                      |
| `Benef_Modal/Ano_*` | Quantidade de bolsas‑ano (conceito ver Seção “Bolsa‑ano” da Nota Técnica CNPq). |
| `Bolsa/Ano_*`       | Valor financeiro associado às bolsas‑ano.                                       |
| `Auxílio/Ano_*`     | Valor financeiro de auxílios vinculados a ICJ.                                  |

### 4.3 Indicadores de Educação Básica (EB)

| Coluna                 | Definição                                              |
| ---------------------- | ------------------------------------------------------ |
| `Ens_Fund_Anos_Finais` | Matrículas 6º – 9º ano do EF (rede pública + privada). |
| `Ensino_Medio`         | Matrículas do ensino médio regular.                    |
| `Ensino_Medio_Tecnico` | Matrículas do ensino médio integrado/concomitante.     |

### 4.4 Indicadores de Formação Docente (FORM)

| Coluna                                                | Definição                                          |
| ----------------------------------------------------- | -------------------------------------------------- |
| `Graduação_Total`                                     | Docentes com nível superior completo.              |
| `Com_Especialização`, `Com_Mestrado`, `Com_Doutorado` | Quantidade de docentes com a respectiva titulação. |

### 4.5 Indicadores de Ensino Superior (SUP)

| Coluna                                                        | Definição                                                       |
| ------------------------------------------------------------- | --------------------------------------------------------------- |
| `Total_Geral`                                                 | Matrículas totais de graduação.                 |
| `Total_Publica`, `Federal`, `Estadual`, `Municipal`           | Matrículas por esfera administrativa.                           |
| `Total_Privada`, `Com Fins Lucrativos`, `Sem Fins Lucrativos` | Matrículas na rede privada, discriminando finalidade econômica. |

---

## 5. Limitações

1. Conversão cambial do CNPq já vem calculada na base original; não foi aplicado deflator.

---

## 6. Recomendações de Uso

- Para análises **per capita**, sugerimos integrar dados populacionais (IBGE Estimativas).
- Utilize as colunas "+\_sum" quando o interesse for impacto financeiro total; utilize "+\_mean" para comparações de valores médios por bolsista.
- Valores ausentes (`NaN`) indicam inexistência de registro no ano ou falta de informação na fonte.

---

## 7. Atualização e Reprodutibilidade

- **Script Colab:** `aggregate_icj_education_2010_2023.ipynb` disponível no repositório.
- Atualizações anuais requerem apenas substituir as planilhas‑fonte e reexecutar o notebook.

---

## 8. Contato

Dúvidas ou sugestões:\
*Tiago Ribeiro dos Anjos* — [tiago.r.anjos@ufscar.br](mailto\:tiago.r.anjos@ufscar.br)\
Repositório GitHub: [https://github.com/tiagoanjos/Agregacao\_ICJ\_2010\_2023](https://github.com/tiagoanjos/Agregacao_ICJ_2010_2023)


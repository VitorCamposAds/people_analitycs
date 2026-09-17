# 🧑‍💼 People Analytics — Turnover, Desempenho e Absenteísmo

> **Dashboard analítico em Power BI** para exploração de indicadores de gestão de pessoas: turnover demissional, absenteísmo, desempenho profissional e percepção dos funcionários sobre a organização.  
> 🎓 *Projeto profissional adaptado para fins metodológicos e educacionais. Dados fictícios e medidas ajustadas à realidade deste dataset demonstrativo.*

---

## 📌 Visão geral

Este projeto integra dados de Recursos Humanos em um modelo analítico no **Power BI**, permitindo explorar indicadores por:

- Período
- Departamento
- Cargo
- Unidade
- Centro de custo
- Motivo de desligamento
- Tipos de ocorrência de absenteísmo

A solução foi construída com:

- 🧹 Preparação e transformação de dados no **Power Query**
- 🧠 Modelagem semântica no **Power BI**
- 🧮 Medidas analíticas em **DAX**
- 📅 Calendário para análises mensais, anuais e comparações temporais
- 📊 Dashboards interativos com filtros e indicadores-chave

---

## 🎯 Objetivos

Os objetivos específicos do projeto são:

- Descrever a evolução dos principais indicadores de gestão de pessoas
- Calcular **turnover**, **headcount**, **absenteísmo** e **contratações malsucedidas**
- Comparar os indicadores de **2020** com os resultados de **2019**
- Analisar desligamentos por período, departamento, cargo, centro de custo e motivo
- Avaliar o desempenho profissional e a percepção sobre a organização
- Cruzar informações de desligamento, desempenho e absenteísmo
- Identificar possíveis desligamentos de profissionais com desempenho elevado
- Disponibilizar os resultados em dashboards interativos

---

## 📊 Principais resultados

| Indicador | Resultado |
|---|---:|
| Funcionários distintos analisados | **859** |
| Turnover demissional em 2020 | **29,7%** |
| Turnover demissional em 2019 | **28,8%** |
| Desligamentos em 2020 | **86** |
| Desligamentos em 2019 | **84** |
| Absenteísmo em 2020 | **1,2%** |
| Desempenho profissional médio | **7,1 / 10** |
| Desempenho organizacional médio | **5,2 / 10** |
| Contratações malsucedidas em 2020 | **16** |

> A análise identificou desligamentos envolvendo profissionais com desempenho elevado e baixo absenteísmo. Portanto, os dados não sustentam a interpretação de que todos os desligamentos sejam explicados exclusivamente por baixo desempenho e alto absenteísmo.

---

## 🧮 Indicadores e regras de negócio

### 👥 Funcionários distintos

Contagem distinta dos funcionários registrados na dimensão `dFuncionarios`:

```DAX
Funcionários analisados =
DISTINCTCOUNT(
    dFuncionarios[Funcionário]
)
```

### ⭐ Desempenho profissional

Média das notas registradas nas avaliações profissionais, em escala de 0 a 10:

```DAX
Desempenho Profissional =
AVERAGE(
    'fAvaliaçãoDesempenho'[Nota]
)
```

### 🏢 Desempenho organizacional

Média das notas relacionadas à percepção sobre a organização:

```DAX
Desempenho Empresa =
AVERAGE(
    fDesempenhoEmpresa[Nota]
)
```

### 🕒 Absenteísmo

As horas de absenteísmo são compostas por atestados, atrasos e faltas não justificadas.

```DAX
Horas Absenteismo =
[Faltas Não Justificadas]
    + [Atrasos]
    + [Atestados]

(%) Absenteismo =
DIVIDE(
    [Horas Absenteismo],
    [Horas Normais]
)
```

Eventos utilizados no modelo:

| Tipo de ocorrência | Código do evento |
|---|---:|
| Horas normais | 1 e 100 |
| Atestados | 14 e 113 |
| Atrasos | 2457 |
| Faltas não justificadas | 3 |

### 🚪 Demissões

As demissões são identificadas pelo código de situação `7` e pela data de afastamento:

```DAX
Demissões =
CALCULATE(
    COUNTROWS(fContrato),
    fContrato[Cód Situação] = 7,
    USERELATIONSHIP(
        fContrato[Data Afastamento],
        dCalendario[Data]
    )
)
```

### 📈 Headcount

O headcount é calculado como o saldo acumulado de contratações menos demissões até a maior data disponível no contexto filtrado:

```DAX
Headcount =
VAR DataAtual =
    MAX(dCalendario[Data])
RETURN
    CALCULATE(
        [Contratações] - [Demissões],
        FILTER(
            ALL(dCalendario[Data]),
            dCalendario[Data] <= DataAtual
        )
    )
```

> **Observação:** como as medidas de contratações e demissões usam `COUNTROWS(fContrato)`, o headcount representa um saldo de registros de contrato. Ele não deve ser interpretado automaticamente como uma contagem distinta de pessoas.

### 🔁 Turnover

O turnover é calculado pela relação entre demissões e o headcount de referência anterior:

```DAX
Turnover =
DIVIDE(
    [Demissões],
    [(LM) Headcount],
    0
)
```

A medida de referência utiliza `PREVIOUSDAY`. Apesar do nome `(LM) Headcount`, ela representa a data imediatamente anterior ao contexto analisado e não necessariamente o mês anterior em todas as granularidades.

### ❌ Contratações malsucedidas

São consideradas malsucedidas as contratações com situação `7` e desligamento em até 60 dias após a admissão:

```DAX
Más Contratações =
CALCULATE(
    [Contratações],
    FILTER(
        fContrato,
        fContrato[Cód Situação] = 7
            && DATEDIFF(
                fContrato[Data Admissão],
                fContrato[Data Afastamento],
                DAY
            ) <= 60
    )
)
```

---

## 🗃️ Modelo de dados

O modelo utiliza tabelas de fatos e dimensões relacionadas por funcionários, datas e demais chaves organizacionais.

### Principais tabelas

- `fContrato`: admissões, contratos e desligamentos
- `fFichaFinanceira`: horas, eventos e valores financeiros relacionados ao absenteísmo
- `fAvaliaçãoDesempenho`: notas de desempenho profissional
- `fDesempenhoEmpresa`: avaliações da percepção sobre a organização
- `dFuncionarios`: dimensão de funcionários
- `dCalendario`: calendário utilizado nas análises temporais
- Dimensões organizacionais: departamento, cargo, unidade e centro de custo

---

## 🗂️ Estrutura do repositório

```text
people-analytics-powerbi/
├── README.md
├── people-analytics-turnover.pbix
├── artigo-cientifico.pdf
└── relatorio-empresarial.pdf
```

---

## ▶️ Como visualizar o projeto

1. Instale o [Power BI Desktop](https://www.microsoft.com/power-platform/products/power-bi/desktop)
2. Clone este repositório:

   ```bash
   git clone https://github.com/SEU-USUARIO/NOME-DO-REPOSITORIO.git
   ```

3. Abra o arquivo `people-analytics-turnover.pbix`
4. Caso o Power BI solicite atualização das fontes, ajuste os caminhos ou conexões de dados
5. Atualize os dados somente se possuir autorização para utilizá-los
6. Navegue pelas páginas do relatório e utilize os filtros para explorar os indicadores

---

## ⚠️ Limitações

- A análise contempla apenas os anos de **2019** e **2020**
- O estudo é **descritivo e exploratório**
- Os resultados **não demonstram relações de causa e efeito**
- A qualidade dos indicadores depende da completude e consistência dos registros administrativos
- Avaliações de desempenho podem conter subjetividade e diferenças entre avaliadores
- As medidas de headcount, admissões e demissões são baseadas em registros de contrato
- O dashboard não substitui entrevistas de desligamento, pesquisas de clima ou análises qualitativas

---

## 🛠️ Tecnologias utilizadas

- Microsoft Power BI Desktop
- Power Query / linguagem M
- DAX
- Modelagem dimensional e calendário analítico
- Git e GitHub para versionamento e documentação

---

## 👤 Autor

**Vitor Campos Moura Costa**  
📧 [vitorcamposmouracosta@gmail.com](mailto:vitorcamposmouracosta@gmail.com)

---

## 📄 Licença

Este projeto está licenciado sob a [MIT License](LICENSE).

---

## 🧭 Aviso metodológico

> Este dashboard foi desenvolvido para apoiar a exploração de indicadores de People Analytics. Os resultados devem ser interpretados em conjunto com o contexto organizacional, a qualidade dos dados e outras fontes de informação. O Power BI organiza evidências; não deve ser utilizado isoladamente para tomar decisões sobre pessoas.

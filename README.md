# PROJETO 3 - FICHA TÉCNICA

# Ficha Técnica: Projeto Análise de Dados 3

## Título do Projeto: Risco Relativo

---

## Objetivo

O banco “Super Cajá” está com uma proposta de automação do processo de análise de crédito utilizando técnicas de análise de dados para melhorar a eficiência, precisão e rapidez na avaliação das solicitações de crédito. O objetivo da análise é desenvolver um score de crédito a partir de uma análise de dados e avaliação do risco relativo que possa classificar os solicitantes em diferentes categorias de risco com base em sua probabilidade de inadimplência. Essa classificação permitirá ao banco tomar decisões informadas sobre quem conceder crédito, reduzindo assim o risco de empréstimos não reembolsáveis. Além disso, a integração da métrica existente de pagamentos em atraso fortalecerá a capacidade do modelo de identificar riscos, contribuindo assim para a solidez financeira e eficiência operacional do banco.

---

## Equipe

Débora Vasconcellos

---

## Ferramentas e Tecnologias

- Big Query
- Google Colab
- Looker Studio

Linguagens: SQL e Python

---

## Processamento e Análises

---

### Limpeza dos dados

***Tabela “user_info”***:
7199 valores nulos na variável “last_month_salary”,
943 valores nulos na variável “number_dependents”,

Devido ao número de nulos da variável  “last_month_salary” corresponder a 20% do banco total e nossa análise ser focada no entendimento do que configura um mal pagador, optei por verificar quantos casos correspondiam aos clientes que já apresentavam uma flag para inadimplência. 

Para isso, criei uma tabela chamada “user_info_flag” e identifiquei que 130 dos 7199 casos possuíam uma flag. Dessa forma, por não comprometer de forma significativa na para a construção futura dos modelos, optei por substituir os nulos pelo valor da mediana (5400).

Assim como, substitui o valor nulo da variável “number_dependents” pelo valor de sua mediana (0). Para ambos os casos a escolha foi pela mediana pela sensibilidade maior apresentada pela média.

***Tabela “user_info_flag”:***
Variáveis com nulos corrigidos receberam os seguintes novos nomes:
”number_last_month_salary_corrigido”

“number_dependents_corrigido”

Com relação a variável ”number_last_month_salary_corrigido”, alterei através do safe_cast seu tipo de float para integer. 

***Tabela “loans_outstanding”:***

Na variável “loan_type” observei que havia mais de uma grafia para a mesma categoria:

| Linha | loan_type |
| --- | --- |
| 1 | real estate |
| 2 | REAL ESTATE |
| 3 | Real Estate |
| 4 | other |
| 5 | OTHER |
| 6 | Other |
| 7 | others |

Optei por padronizar as categorias, estabelecendo “Real Estate” e “Other” como padrão.

Tabela users_loans_full

Unificação de todas as tabelas principais através da chave “user_id”,

425 valores nulos nas variáveis “total_loans”, “real_estate_loans”, “other_loans”. Optei por não utilizar os casos já que ter empréstimos ativos era parte importante da análise de risco de crédito. Mantive a tabela com os casos e criei uma nova sem eles chamada users_loans_clean

---

### Análise do Risco Relativo e Novas Variáveis

### Variáveis Gerais

***Tabela “loans_outstanding” para “Tabela total_loans”***

Para a unificação das tabelas através de user_id, transformei as informações da tabela “loans_outstanding” para correspondência com as demais. Criando as variáveis:

***“total_loans”***: somatório de todos os empréstimos realizados por user_id.

***“real_estate_loans”***: somatório de todos os empréstimos realizados nessa modalidade.

***“other_loans”***: somatório de todos os empréstimos realizados nessa modalidade.

***Tabela “users_loan_clean” no Looker***

***”*dependents_have”:** transformação da variável “number_dependents_corrigida” em categórica com Sim ou Não. 

**“default_flag_name”**: variável “default_flag” em uma versão nominal de “Sim” ou “Não”

---

### Risco Relativo

Para construir um score de identificação automatizada de clientes, ajudando na identificação de possíveis clientes inadimplentes. Observamos algumas variáveis e suas possibilidades de apresentar maior ou menor risco relativo. Para isso, dividi em percentis (20%) as seguintes variáveis:

- Age (Idade)
- Number Last Month Salary (Salário do Último Mês)
- Debt Ratio (Taxa de Endividamento)
- Using Lines Not Secured Personal Assets (Usando Linhas de Crédito Sem Garantias de Bens Pessoais)
- Total Loans (Total de Empréstimos Ativos)

E dividi em dois grupos (binário) as seguintes variáveis:

- Has Dependents (Tem Dependentes) * Variável original “number dependents” apresentava um formato quantitativo
- More 90 Days Overdue (Mais de 90 dias de Atraso no Pagamento de Empréstimo)
- Number Times Delayed Payment Loan 30 59 Days (Atraso de Pagamento do Empréstimo por 30-59 Dias)
- Number times delayed payment loan 60 89 days (Atraso de Pagamento do Empréstimo por 60-89 Dias)

O cálculo do Risco Relativo foi realizado com cada uma das variáveis em comparação com a variável default_flag que contabilizava aqueles que haviam de fato ocorrido em inadimplência. Com relação aos resultados segue abaixo o comportamento de cada variável:

- **Age (Idade):**

| Linha | age_percentile | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 7115 | 221 | 0.031061138439915658 | 1.7765273311897165 |
| 2 | 2 | 7115 | 172 | 0.024174279690794172 | 1.3826366559485626 |
| 3 | 3 | 7115 | 136 | 0.019114546732255912 | 1.0932475884244479 |
| 4 | 4 | 7115 | 59 | 0.0082923401264933475 | 0.47427652733119285 |
| 5 | 5 | 7115 | 34 | 0.0047786366830639415 | 0.27331189710610992 |

Através dos resultados identificamos que os dois primeiros percentis apresentam maior risco, os grupos são compostos pelo intervalo de idades de:  21-48 anos. Corroborando com a hipótese de que pessoas mais jovens apresentam maior risco de inadimplência. Agrupamos os dois percentis com maior risco relativo na categoria Alto Risco e as demais categorias em Baixo Risco. 
Em formato dummie: **age_risk**
Em formato categórica: **age_risk_flag** (Apenas no Looker)

- **Number Last Month Salary (Salário do Último Mês):**

| Linha | last_month_salary_percentile | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 7115 | 201 | 0.028250175685172161 | 1.6157556270096518 |
| 2 | 2 | 7115 | 157 | 0.022066057624736526 | 1.2620578778135128 |
| 3 | 3 | 7115 | 99 | 0.013914265635980314 | 0.79581993569132081 |
| 4 | 4 | 7115 | 117 | 0.016444132115249484 | 0.94051446945338035 |
| 5 | 5 | 7115 | 48 | 0.0067463106113843955 | 0.3858520900321 |

Observamos que os dois primeiros grupos apresentam maior risco, portanto indivíduos com salários entre 0-5400 se encontram em uma posição de maior vulnerabilidade financeira e consequente maior risco de inadimplência. Até o quarto percentil os salários ficam em torno de 5400 e 8330, o que torna os percentis com intervalos de valores curtos em alguns grupos e mais concentrados nos extremos. 

Agrupamos os dois percentis com maior risco relativo na categoria Alto Risco e as demais categorias em Baixo Risco. 
Em formato dummie: **salary_risk**
Em formato categórica: **salary_risk_flag** (Apenas no Looker)

- **Debt Ratio (Taxa de Endividamento)**

| Linha | debt_ratio_percentile | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 7115 | 83 | 0.011665495432185535 | 0.6672025723472701 |
| 2 | 2 | 7115 | 123 | 0.017287420941672529 | 0.98874598070739961 |
| 3 | 3 | 7115 | 113 | 0.015881939564300781 | 0.90836012861736726 |
| 4 | 4 | 7115 | 199 | 0.027969079409697749 | 1.5996784565916418 |
| 5 | 5 | 7115 | 104 | 0.014617006324666218 | 0.83601286173633 |

---

Observamos que o quarto percentil apresentou maior risco de inadimplência, com uma taxa de endividamento entre (0.30-0.47), e selecionamos o grupo como de maior risco de inadimplência.

Agrupamos  os demais como Baixo Risco e o quarto percentil como Alto Risco.

Em formato dummie: **debt_ratio_risk**
Em formato categórica: **debt_ratio_risk_flag** (Apenas no Looker)

- **Using Lines Not Secured Personal Assets (Usando Linhas de Crédito Sem Garantias de Bens Pessoais)**

| Linha | lines_not_secured_percentile | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 7115 | 8 | 0.0011243851018973982 | 0.064308681672025872 |
| 2 | 2 | 7115 | 0 | 0.0 | 0.0 |
| 3 | 3 | 7115 | 4 | 0.000562192550948699 | 0.032154340836012929 |
| 4 | 4 | 7115 | 52 | 0.00730850316233311 | 0.41800643086816941 |
| 5 | 5 | 7115 | 558 | 0.078425860857343488 | 4.48553054662380 |

Observamos que o último grupo, ou seja aqueles com maior uso do limite de crédito, apresentam maior risco de inadimplência. 

Indicamos o grupo com maior risco relativo na categoria Alto Risco e as demais categorias em Baixo Risco. 
Em formato dummie: **lines_not_secured_risk**
Em formato categórica: **lines_not_secured_risk_flag** (Apenas no Looker)

- **Total Loans (Total de Empréstimos Ativos)**

| Linha | total_loans_percentile | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 1 | 7115 | 208 | 0.029234012649332407 | 1.6720257234726759 |
| 2 | 2 | 7115 | 160 | 0.022487702037947956 | 1.2861736334405169 |
| 3 | 3 | 7115 | 87 | 0.01222768798313423 | 0.69935691318328275 |
| 4 | 4 | 7115 | 82 | 0.011524947294448366 | 0.65916398713826718 |
| 5 | 5 | 7115 | 85 | 0.011946591707659856 | 0.68327974276527 |

Observamos que os dois primeiros percentis apresentam maior risco de inadimplência. Se encontrando no intervalo de 1-7 empréstimos, o que refuta a hipótese de que pessoas com mais empréstimos apresentariam maior risco de inadimplência. 

Agrupamos os dois percentis com maior risco relativo na categoria Alto Risco e as demais categorias em Baixo Risco. 
Em formato dummie: **total_loans_risk**
Em formato categórica: **total_loans_risk_flag** (Apenas no Looker)

- **Has Dependents (Tem Dependentes)**

| 1 | 0 | 21541 | 302 | 0.014019776240657424 | 0.80185456553278089 |
| --- | --- | --- | --- | --- | --- |
| 2 | 1 | 14034 | 320 | 0.022801767136953081 | 1.30413644034904 |

Apesar do cálculo demonstrar que pessoas com dependentes apresentam maior risco de inadimplência, optei por não seguir com essa variável por entender que a presença ou não de filhos/dependentes não deveria estar sendo posta como parte da análise de crédito do sujeito analisado. 

- **More 90 Days Overdue (Mais de 90 dias de Atraso no Pagamento de Empréstimo)**

| Linha | has_more_90_days_overdue | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 0 | 33799 | 56 | 0.0016568537530696173 | 0.0947629779187329 |
| 2 | 1 | 1776 | 566 | 0.31869369369369283 | 18.227537223718915 |

Transformamos a variável em dicotômico para 0 (sem atrasos de +90dias) e 1 (teve algum atraso de +90). Observamos que o risco relativo daqueles que já atrasaram 90 dias é muito alto. Corroborando com a hipótese de que esse grupo apresentaria maior risco de inadimplência. 

Em formato dummie: **has_more_90_days_overdue**
Em formato categórica: **has_more_90_days_flag** (Apenas no Looker)

- **Number Times Delayed Payment Loan 30 59 Days (Atraso de Pagamento do Empréstimo por 30-59 Dias)**

| Linha | has_30_59_days | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 0 | 29858 | 63 | 0.00210998727309262 | 0.12067973832840877 |
| 2 | 1 | 5717 | 559 | 0.097778555186286628 | 5.5923988758073326 |

Apesar do grupo de pessoas que já atrasaram entre 30-59 dias apresentar um risco relativo significativo, devido a colinearidade entre as variáveis de dias de atraso optei por utilizar apenas a variável de +90 dias de atraso para a construção do score.

- **Number times delayed payment loan 60 89 days (Atraso de Pagamento do Empréstimo por 60-89 Dias)**

| Linha | has_60_89_days | total_clients | total_default | default_rate | relative_risk |
| --- | --- | --- | --- | --- | --- |
| 1 | 0 | 33797 | 143 | 0.0042311447761635484 | 0.24199835275244186 |
| 2 | 1 | 1778 | 479 | 0.26940382452193462 | 15.408426137247368 |
|  |  |  |  |  |  |

Apesar do grupo de pessoas que já atrasaram entre 60-89 dias apresentar um risco relativo significativo, devido a colinearidade entre as variáveis de dias de atraso optei por utilizar apenas a variável de +90 dias de atraso para a construção do score.

---

### **Score Risco - Auxílio na Identificação de Inadimplentes**

Após calcular o risco relativo, realizamos várias combinações para construção de um score com a pontuação obtida em cada variável de risco relativo. 
Dessa forma, utilizamos as variáveis:

- Age (Idade)
- Number Last Month Salary (Salário do Último Mês)
- Using Lines Not Secured Personal Assets (Usando Linhas de Crédito Sem Garantias de Bens Pessoais)
- Total Loans (Total de Empréstimos Ativos)
- More 90 Days Overdue (Mais de 90 dias de Atraso no Pagamento de Empréstimo)

Optei pelas variáveis entendendo que seus grupos representavam comportamentos que poderiam provocar maiores chances de inadimplência, apresentando resultados de maior risco para a relação com o banco. Optei por não utilizar gênero, nem a presença de dependentes, por entender que são questões que podem levar a ações discriminatórias. 

Assim, com um somatório de 0 a 5 pontos,  criei a variável **score_default**. Onde classifiquei que aqueles que apresentam 3 ou mais pontos devem apresentar um Score de Alto Risco, enquanto aqueles com menor pontuação apresentam um Baixo Risco. (Essas informações se encontram de forma categórica no Looker Studio através da variável **score_flag**) 

O resultado desse Score através da Matriz de Confusão apresentou o seguinte resultado:

| Linha | true_positive | false_positive | false_negative | true_negative | accuracy | precision | recall | f1_score |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 518 | 6204 | 104 | 28749 | 0.82268446943078 | 0.077060398690865811 | 0.83279742765273312 | 0.14106753812636166 |

Devido ao forte desbalanceamento dos dados (34.953 adimplentes para 622 inadimplentes), o modelo do score apresentou um certo rigor em sua alta identificação de falsos positivos. Necessitando maiores ajustes que podem ser contornados com o uso de técnicas de balanceamento e uso de Machine Learning para identificação automatizada. 

Sobre os dados, observamos com a técnica de undersamling, ou seja, diminuindo a amostra de adimplentes para 10%. Obtemos os seguintes resultados:

| Linha | true_positive | false_positive | false_negative | true_negative | accuracy | precision | recall | f1_score |
| --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 518 | 624 | 104 | 2984 | 0.82789598108747042 | 0.45359019264448336 | 0.83279742765273312 | 0.58730158730158732 |

Apesar da melhora significativa, ainda percebi que o modelo poderia ter outras mudanças.

Então realizei a regressão logística que apresentou o melhor resultado usando as cinco variáveis escolhidas, determinando o limiar para 0.6 e utilizando a técnica de balanceamento oversampling SMOTE:

| Linha | true_positive | false_positive | false_negative | true_negative | accuracy | precision | recall | f1_score | roc AUC | log loss |
| --- | --- | --- | --- | --- | --- | --- | --- | --- | --- | --- |
| 1 | 189 | 375 | 24 | 10085 | 0.9626 | 0.3351 | 0.8873 | 0.4865 | 0.9801 | 0.1757 |

Diante dos resultados, entendo que o Score atual auxilia o banco nesse processo de automação e pode ser um sinalizador interessante para facilitar as atividades realizadas pelos funcionários. Entretanto, ainda cabe maior aperfeiçoamento para a diminuição de erros e necessita do funcionário humano para refletir mais profundamente sobre cada grupo de risco ao qual o cliente solicitante do empréstimo se encontra identificado.

---

## Resultados e Conclusões

O banco Super Cajá busca identificar com maior facilidade clientes que podem apresentar maior risco de inadimplência e tomar decisões mais rápidas para a concessão de créditos. 

Observando os dados gerais do banco, percebemos que dos 35.425 presentes na base de dados, apenas 622 realmente apresentaram inadimplência (correspondendo a menos de 2% do total de clientes). São 305,3 mil empréstimos ativos sendo 12% (36,6mil) segurados por imóveis.

Em uma visão geral dos clientes, podemos observar que:
60% se identificam como Homens e 40% mulheres.

60,6% possuem dependentes. 

O público é composto por uma maioria de Adultos mais maduros, com uma média de 53 anos. 32,65% possuem entre 51-64 anos. Apenas 6,85% possuem 30 anos ou menos. 

O salário médio gira em torno de $6k. Porém, há uma grande variância nos dados tendo o mínimo de $0 e o máximo de 2 mi. 

Quando observamos os inadimplentes, percebemos que a média salarial fica em $5k e o intervalo de mínimo e máximo passa de $0 para $28k. A idade média cai para 45 anos. Não há muita diferença entre quem tem e não tem dependentes os dados se encontrando em praticamente 50% para cada lado. A maioria dos empréstimos dos Inadimplentes se encontram na categoria “Outros”. 

Quando analisamos os grupos de risco percebemos que de fato as menores faixa etárias apresentam maior risco de inadimplência. Quando olhamos para os dados gerais percebemos que as pessoas mais jovens também possuem menores salários (outro ponto que aumenta o risco de inadimplência) e tem um menor histórico de empréstimos ativos (outro indicador de risco). Dessa forma, a idade isolada não é um identificador suficiente, já que está suscetível a outras dinâmicas de risco que intensificam seu risco.

Aqueles com maior número de empréstimos já realizados com o banco apresentam um menor risco relativo. Ter realizado esse tipo de compromisso ao longo do tempo demonstra que o cliente tem entendimento das responsabilidades inerentes ao contrato. Entretanto, essa variável não pode ser vista de forma isolada já que um cliente com excelente histórico pode passar por adversidades e ter por exemplo perda salarial.

A perda salarial ou salários menores apresentam um maior risco para inadimplência, através da análise de risco relativo ficou evidente que a vulnerabilidade financeira é um fator de atenção. 

Ainda observando os riscos de inadimplência, percebemos que o uso excessivo de linhas de crédito não seguradas por bens pessoais também ressoam de forma alarmante para o risco de inadimplência. O comportamento já pode indicar uma dificuldade financeira que deve chamar a atenção para a concessão de novos créditos.

Além disso, atrasos acima de 90 dias são um grande alerta vermelho para o risco de inadimplência. A falta de compromisso com os prazos, chamam a atenção para um comportamento desistente em manutenção com o contrato estabelecido. 

Também observamos a variável de taxa de endividamento, mas por acrescentar pouco ou nada na análise de construção do score de crédito, optamos por retirá-la.

Dessa forma construímos o Score de Crédito a partir da análise dos grupos de maior risco nas variáveis: idade, último mês de salário, total de empréstimos, uso de linhas de crédito não seguradas por bens e ter atrasado pagamento por mais de 90 dias. O score de 0-5 interpreta que aqueles que possuam 3 ou mais pontos já apresentam um risco de inadimplência.  Essa pontuação possui uma acurácia de 82%,  utilizando as mesmas cinco variáveis podemos chegar a 96% de acurácia através do modelo de regressão logística. 

Devido à raridade de clientes que realmente são inadimplentes, podemos entender que a maioria das pessoas conseguem gerir com constância os seus pagamentos. Entretanto, a raridade do evento me fez optar por um recall alto em detrimento de uma precisão mais alta. Dessa forma, o modelo atual identifica mais pessoas de alto risco que não necessariamente se tornarão inadimplentes. Por isso é importante traçar estratégias para os clientes que se encontram como Alto Risco para além da negação do crédito. Cruzando o Score Risco com outras informações qualitativas e quantitativas do cliente, podemos traçar estratégias como:

- Diminuir a margem de disponibilidade de crédito para empréstimo;
- Permitir apenas formas de crédito que possam ser asseguradas com bens;
- Cruzar as informações com outras instituições como SPC e Serasa;
- Atrelar contratos de seguro aos empréstimos concedidos para os grupos de maior risco.

Além disso, para os casos não identificados pelo score e que venham a  ser inadimplentes podemos desenvolver estratégias de facilitação dos pagamentos, restrição a novas linhas de crédito, entre outras possibilidades.

Sendo assim, concluímos que o uso das variáveis escolhidas é funcional para encontrar possíveis inadimplentes, mas é necessário contínuo aperfeiçoamento do modelo. Levando sempre em consideração mudanças econômicas, políticas e sociais que ocorram ao longo do tempo. Os sujeitos se encontram em constante interação em um mundo que está sempre se transformando, dessa forma não podemos aplicar o modelo de forma atemporal e não contextual. 

Fontes:
[Regressão Logística e suas Aplicações (ufma.br)](https://monografias.ufma.br/jspui/bitstream/123456789/3572/1/LEANDRO-GONZALEZ.pdf)

https://www.serasa.com.br/credito/blog/analise-de-credito-o-que-e-para-que-serve-e-como-e-feita/

https://seer.pucgoias.edu.br/index.php/estudos/article/download/4020/2323

---

## Limitações/Próximos Passos

Diante de resultados insatisfatórios na precisão do modelo, entendo que o Score necessita de maiores aperfeiçoamentos. O processo avaliativo necessita de constante checagem e análise dos grupos, inclusive, necessitando de abarcar mais variáveis que não estavam disponíveis no banco como valores dos empréstimos obtidos anteriormente, histórico com datas dos contratos e seus atrasos ao longo do tempo de relação do cliente com o banco. 

Devemos ter atenção com processos de avaliação de risco dos clientes para não cairmos em armadilhas discriminatórias que reforçam estereótipos e violentam grupos sociais. Além disso, categorizações preconceituosas acabam por causar atritos na relação da empresa com os colaboradores e clientes. Dessa forma, entendo que o processo avaliativo deve estar em constante discussão, reformulação e treinamento das tecnologias empregadas visando a segurança, proteção e cuidado com a instituição e todas as pessoas envolvidas em seu pleno funcionamento. 

Próximos Passos:

- Encaminhar os dados para equipe de programação para discutirmos modelos de consulta e checagem dos riscos de inadimplência através das variáveis analisadas.
- Aperfeiçoar o banco de dados para uma coleta mais sofisticada do histórico dos clientes com o banco, principalmente os tipos de empréstimo, valores e pagamentos e atrasos ao longo do tempo em que o cliente se encontra sob os serviços prestados pelo banco.
- Contínuo processo de aperfeiçoamento do modelo, reavaliando as variáveis escolhidas, acrescentando ou retirando pontos de acordo com os resultados obtidos.
- Acompanhamento contínuo das mudanças econômicas, políticas e sociais que afetam a sociedade e, consequentemente, a vida dos clientes. Com isso, há uma atualização nos comportamentos que podem afetar diretamente a capacidade de adimplência.

---

## Links de interesse

**Google Colab:** 

https://colab.research.google.com/drive/1sq4mo7AufIGMQ25qDJdFFfaLhkFpFomD?usp=sharing

**Dashboard**: 

https://lookerstudio.google.com/reporting/92b9935f-57ea-4e85-adb6-8e3c3ec9a093

**Slide**:

[https://docs.google.com/presentation/d/1sh7M2NngSlIgJmzXLxv0I2HEjD5-QmX0m-FMJjoye5M/edit?usp=sharing](https://docs.google.com/presentation/d/1sh7M2NngSlIgJmzXLxv0I2HEjD5-QmX0m-FMJjoye5M/edit?usp=sharing)

**Video**:

https://www.loom.com/share/67f3dc4f790842bb944341bc192531c2?sid=55560959-7159-495f-b3f9-26d0f16db40d

---

---
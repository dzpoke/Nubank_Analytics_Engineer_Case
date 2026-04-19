# Nubank Analytics Engineer Case

#### Dividir e conquistar!

### Enunciado do Problema, pontos chaves

-  solução tanto para um público de negócios quanto para um público técnico, e que as pessoas podem acessar 
- deve ser a mais autoexplicativa possível
-  Novos produtos às vezes não estão relacionados a transações entre pessoas (peer-to-peer) e alguns deles podem estar disponíveis apenas em determinados países.

---

### Principais Entregáveis

> A proposta de modificação do modelo de dados, levando em consideração a introdução, deve conter:

- Representação visual das mudanças propostas 
- Análise de trade-offs das mudanças propostas ou do que foi mantido igual

---

## Transfer_ins, Transfer_outs e Pix_movements

- transfer_ins: transferências não-PIX recebidas por uma conta (dinheiro entrando)

- transfer_outs: transferências não-PIX realizadas a partir de uma conta (dinheiro saindo)

- pix_movements: transferências **recebidas ou enviadas** por uma conta utilizando o PIX

Part 1.

As tabelas **transfer_ins e transfer_outs** são redundantes e só é possível saber que se trata de entradas ou saídas únicamente pelo nome das tabelas, porém, não é possível saber que tipo de transferencia se trata, ou se é entrada ou saída de dinheiro.

Part 2.

A tabela **pix_movements** tem praticamente a mesma estrutura das tabelas mencionadas anteriormente na **part 1**, só que ela carrega uma melhoria que é a coluna **in_or_out** que torna possível identificar se a transação é uma entrada ou saída de dinheiro.

> Solução proposta
>
> Consolidação das três entidades e nomear a nova matriz como **financial_transactions**, a nova tabela vai receber a melhoria feita na antiga **pix_movements** que é a coluna **in _or_our**, para identificar se é entrada ou saida de dinheiro.

![alt text](/img/image-3.png)


### Accounts

- status agora so pode receber tres valores possiveis **active, inactive e blocked**, isso vai evitar a entrada
de tipos de situacoes de contas inexistentes. As demais informacoes foram mantidas levando em consideracao que sao informacoes basicas e essenciais para o cliente poder utilizar a conta bancaria nubank.

![alt text](/img/image-11.png)


### Customers


Part 1.

- Coluna **cpf** renomeada para **identification_document**
e tipo alterado para varchar, para evitar problema de integridade com zero a esquerda.

- Coluna **country_name** e **customer_city** removidas porque não se enquadram no contexto atual da matriz que visa dados diretos e simples para a identificação do cliente.


![alt text](/img/image-5.png)

#### City, State e Country

As três matrizes foram excluídas porque trazem  complexidade de entendimento sobre o modelo e não são
relevamentes para o enriquecimento de informações.

> No lugar delas vamos dar origem a **customers_address** que tem como objetivo registrar o que foi proposto nas três matrizes anteriormente, só que de maneira mais completa e enriquecedora a nivel de informacoes do cliente, trazendo agora o **Country, Zip_code, State, City e Neighborhood** de onde o cliente realizou a transacao.


#### d_time

Essa matriz foi totalmente remodelada levando como inspiração uma tabela de d_calendario que eu tenho construida no Power Bi.

> A ideia é ter uma nova matriz onde vamos ter granularidades maiores para poder analisar em varios cenarios diferentes, isso vai desde de a data da transacao ate a semana do ano da transacao realizada pelo cliente.

![alt text](/img/image-7.png)


# Holidays

> Adicionei uma tabela auxiliar chamada **holidays** que tem como objetivo registrar se a data da transacao ocorreu em um feriado, essa matriz tem o **country_hol, name_hol** do feriado em questao e para fechar a coluna **abount_hol** é um campo que vai ter um breve descricao do se trata o feriado assim, os usuarios nao precisam ir perquisar cada feriado que tiverem duvida na web.


A versão 0.1 da modelagem onde os problemas principais de redundancia foram resolvidos ficaram dessa forma.

![alt text](/img/image-8.png)

---

### Problema 02

> Tenha em mente que os novos produtos às vezes não estão relacionados atransações entre pessoas (peer-to-peer) — por exemplo: seguro de vida, crédito,recompensas e outros produtos — e alguns deles podem estar disponíveis apenas em determinados países.

> Solução proposta

Part 1

Criei a matriz **actions_type** que tem como objetivo registrar todos os tipos de produtos e transacoes disponiveis na instituicao bancaria.

As colunas **country_off** e **city_off** vai registrar em quais paises e cidades determinado produto esta indisponivel

A criacao dessa tabela torna necessario alteracoes na tabela **financial_transactions**

- Adicao da **product_id** tornando possivel identificar qual foi o tipo de transacao realizada pelos clietes.

- Adicao das colunas  **unavailable_at_tra** e  **canceled_at_tra**, se o tipo de acao realizada pelo cliente nao estiver disponivel em seu pais a mesma vai ser cancelada e registrada como indisponivel.

- 

Depois disso o resultado final da modelagem foi esse aqui

![alt text](/img/image-9.png)
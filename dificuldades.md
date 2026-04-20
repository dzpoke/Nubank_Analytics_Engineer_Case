
## Dúvidas no decorrer do projeto

### trade-offs ?

Trade-off é uma expressão em inglês que significa "equilíbrio" ou "troca compensatória". No mundo da tecnologia e de dados, ela é usada quando você precisa escolher entre duas opções e, ao ganhar algo em uma, inevitavelmente perde (ou abre mão de algo) na outra.

Basicamente, é a análise do: "O que eu ganho e o que eu perco com essa decisão?"

### uuid ?

UUID (Universally Unique Identifier) é um identificador de 128 bits usado para garantir que uma informação seja única em todo o mundo, sem a necessidade de um coordenador central (como um banco de dados que gera números sequenciais).

Mesmo com essa explicação eu ainda não consegui entender direito como isso aqui funciona.

### city, state e country

Não consegui compreender porque na tabela customersa a coluna customer_city foi criada levando em consideração a cidade.

### Tipos de dados

Eu fiquei bem bugado quanto aos tipos de dados nas tabelas, na minha cabeça eu poderia mudar, mas como tinha o envolvimento de demais paises e comportamentos diferentes para cada um deles eu preferi deixar a maior parte dos dados com VARCHAR levando em consideração que existe uma regra de backend para cada pais.
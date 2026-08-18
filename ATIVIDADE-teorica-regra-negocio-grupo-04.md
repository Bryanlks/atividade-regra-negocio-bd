# Atividade Teórica: Regra de Negócio no BD versus na Aplicação

**Aluno(s):** Ryan Torres, Pedro Lucas Santana, Gerson Adriano, João Andre, Nickolas Silva
**Turma:** Banco de Dados 2026  
**Data:** 18/08/2026  
**Repositório Git:** https://github.com/Bryanlks/atividade-regra-negocio-bd

##Resumo Executivo

As regras de negócio são condições que determinam como um sistema deve funcionar de acordo com as necessidades de uma organização. Em um sistema de vendas, por exemplo, podem existir regras como impedir a venda de produtos sem estoque, exigir idade mínima para cadastro ou garantir que um CPF pertença a apenas um cliente.

Uma questão importante de arquitetura é decidir onde essas regras devem ser implementadas: no banco de dados, na aplicação ou em uma combinação dos dois.

A posição defendida neste trabalho é que não existe uma opção vencedora absoluta. O banco de dados deve ser responsável principalmente pelas regras de integridade e consistência dos dados, enquanto a aplicação deve concentrar as regras de processo, comportamento e negócio que sejam complexas ou mudem frequentemente.

Essa abordagem evita depender exclusivamente da aplicação para garantir a integridade dos dados, mas também evita transformar o banco em um local onde toda a lógica do sistema fique concentrada. Dessa maneira, cada camada exerce uma responsabilidade adequada, proporcionando maior segurança, manutenção e flexibilidade.

##1. Desenvolvimento Teórico
###1.1 O que é regra de negócio?

Uma regra de negócio é uma condição, política ou restrição que determina como uma organização realiza determinada atividade ou como um sistema deve se comportar para representar corretamente essas atividades.

Por exemplo, em um sistema de vendas:

- um produto não pode ser vendido quando não há estoque;

- o CPF de um cliente deve ser único;

- um cliente deve possuir idade mínima para determinado cadastro;

- um pedido pode possuir determinados estados, como aberto, pago e cancelado;

- determinados produtos podem possuir descontos específicos;

- uma venda acima de determinado valor pode exigir uma autorização.

As regras podem ser classificadas de diferentes maneiras.

Regras de integridade

São regras diretamente relacionadas à validade e consistência dos dados.

Exemplos:

- CPF não pode ser duplicado;
- produto de um pedido deve existir;
- preço não pode ser negativo;
- determinado campo não pode ser nulo.

Esse tipo de regra possui forte relação com o banco de dados, podendo ser implementado através de NOT NULL, UNIQUE, CHECK, PRIMARY KEY e FOREIGN KEY.

O PostgreSQL disponibiliza esses mecanismos justamente para impedir que dados que violem determinadas restrições sejam armazenados.

#### Regras de processo

Determinam como uma operação deve acontecer.

Exemplo:

> Um pedido só pode ser enviado depois que o pagamento for confirmado.

Essa regra normalmente pertence à lógica da aplicação, pois envolve um fluxo de negócio.

#### Regras de cálculo

Determinam como determinados valores devem ser calculados.

Exemplo:

> Clientes VIP recebem 15% de desconto, exceto em produtos promocionais.

Esse tipo de regra geralmente é mais adequado à camada de negócio da aplicação, principalmente quando pode sofrer alterações frequentes.

Regras de autorização

Determinam quem pode realizar determinada operação.

Exemplo:

> Somente usuários com permissão de gerente podem cancelar um pedido já pago.

Embora o banco possa participar da proteção dos dados, a decisão de negócio normalmente deve ser coordenada pela aplicação e pelo sistema de autorização.

### 1.2 Regras no banco de dados

O banco de dados possui mecanismos próprios para proteger a integridade das informações.

Entre os principais estão:

- `CHECK`;
- `NOT NULL`;
- `UNIQUE`;
- `PRIMARY KEY`;
- `FOREIGN KEY`;
- triggers;
- procedures/functions;
- transações.

A documentação do PostgreSQL classifica entre suas constraints mecanismos como `CHECK`, `NOT NULL`, `PRIMARY KEY`, `UNIQUE` e `FOREIGN KEY`.

#### CHECK

Permite determinar uma condição que o valor armazenado precisa satisfazer.

Exemplo:

sql

CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) CHECK (preco > 0)
);

Nesse caso, o banco impede que seja armazenado um produto com preço menor ou igual a zero.

UNIQUE

Garante que determinado valor ou combinação de valores não se repita.

Exemplo:

CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(11) UNIQUE NOT NULL
);

Assim, dois clientes não podem possuir o mesmo CPF.

FOREIGN KEY

Garante a integridade referencial entre tabelas.

Exemplo:

CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER NOT NULL,
    FOREIGN KEY (cliente_id) REFERENCES clientes(id)
);

Nesse caso, um pedido não pode apontar para um cliente que não exista na tabela clientes.

Triggers

Uma trigger é um mecanismo que executa determinada função automaticamente em resposta a eventos no banco, como INSERT, UPDATE ou DELETE.

Pode ser utilizada quando uma regra não pode ser expressa adequadamente através de uma constraint simples.

Entretanto, triggers devem ser utilizadas com cuidado. Quando muitas regras são colocadas nesse mecanismo, pode ficar mais difícil descobrir por que determinada alteração no banco produziu determinado comportamento.

Stored procedures e funções

Procedures e funções permitem executar lógica diretamente no banco.

Podem ser úteis quando uma operação precisa ser realizada de maneira controlada e centralizada no banco.

Porém, colocar grande parte da lógica de negócio em SQL pode aumentar o acoplamento com determinado SGBD e dificultar a manutenção ou migração da aplicação.

Martin Fowler destaca justamente que existe uma discussão arquitetural sobre colocar lógica de domínio no SQL ou na aplicação, envolvendo questões como desempenho, modificabilidade, testabilidade e portabilidade.

Transações e ACID

As transações permitem agrupar operações de banco para que elas sejam tratadas como uma unidade lógica.

O modelo ACID está relacionado às propriedades de:

Atomicidade: a transação é executada integralmente ou não é aplicada;
Consistência: as operações devem preservar as regras de integridade;
Isolamento: transações concorrentes não devem produzir resultados inconsistentes;
Durabilidade: alterações confirmadas devem permanecer armazenadas.

O PostgreSQL oferece diferentes níveis de isolamento de transações. O nível Serializable, por exemplo, fornece a garantia mais forte de isolamento e procura fazer com que transações concorrentes produzam um resultado equivalente à execução serial.

Vantagens das regras no banco

- 1. Proteção centralizada dos dados

Se várias aplicações utilizam o mesmo banco, uma UNIQUE, por exemplo, será aplicada independentemente de qual aplicação tentou inserir o registro.

- 2. Maior garantia de integridade

A aplicação pode possuir um erro de programação, mas uma constraint continua sendo aplicada pelo banco.

- 3. Controle sobre concorrência

O banco possui mecanismos de transação e isolamento capazes de lidar com operações concorrentes.

- 4. Redução de duplicação para regras de integridade

Uma regra como "CPF deve ser único" não precisa ser implementada separadamente em cada aplicação que acessa o banco.

Desvantagens das regras no banco

- 1. Maior acoplamento ao SGBD

Uma solução que depende muito de funcionalidades específicas do PostgreSQL pode ser mais difícil de migrar para outro banco.

- 2. Manutenção mais difícil quando há lógica complexa

Triggers e procedures podem tornar o comportamento do sistema menos evidente para desenvolvedores que esperam encontrar a lógica na aplicação.

- 3. Testabilidade pode ser mais trabalhosa

Regras espalhadas em triggers, procedures e constraints podem exigir uma estratégia específica de testes.

- 4. Mudanças de negócio podem exigir alterações no banco

Se uma regra muda constantemente, manter essa lógica dentro do banco pode tornar a evolução mais trabalhosa.

### 1.3 Regras na aplicação

Na aplicação, as regras de negócio normalmente ficam em uma camada de serviço ou domínio.

Uma arquitetura comum separa:

Interface / Controller
        ↓
Camada de Serviço / Domínio
        ↓
Camada de Persistência
        ↓
Banco de Dados

Essa separação permite que a lógica de negócio fique organizada independentemente da interface utilizada pelo usuário. Martin Fowler descreve o padrão Service Layer como uma camada que define as operações disponíveis para as interfaces da aplicação e coordena a lógica e as transações.

Exemplo:

function criarPedido(cliente, produto, quantidade) {


    if (cliente.idade < 18) {
        throw new Error("Cliente deve possuir pelo menos 18 anos.");
    }


    if (produto.estoque < quantidade) {
        throw new Error("Estoque insuficiente.");
    }


    if (cliente.possuiPedidoAberto) {
        throw new Error("Cliente já possui um pedido em aberto.");
    }


    // criação do pedido
}

Nesse caso, a aplicação está verificando diversas regras antes de executar a operação.

Vantagens das regras na aplicação

- 1. Maior flexibilidade

Alterações nas regras podem ser realizadas no código sem necessariamente modificar a estrutura do banco.

- 2. Melhor organização para regras complexas

Regras que envolvem vários passos, cálculos ou decisões podem ser mais fáceis de compreender em uma linguagem de programação.

- 3. Facilidade de testes automatizados

É possível criar testes unitários para diferentes cenários da regra de negócio.

- 4. Melhor integração com outras partes do sistema

A aplicação pode combinar informações vindas do banco, APIs externas, serviços de pagamento e outros componentes.

Desvantagens das regras na aplicação

- 1. Risco de inconsistência entre diferentes aplicações

Imagine que um banco seja utilizado por:

Aplicação Web
Aplicativo Mobile
Sistema Administrativo
API de Parceiros

Se cada aplicação implementar a mesma regra de maneira diferente, podem surgir inconsistências.

- 2. O banco pode receber dados sem passar pela regra

Se alguém executar diretamente:

INSERT INTO clientes (...)

a regra existente apenas na aplicação pode ser ignorada.

- 3. Duplicação

Uma mesma regra pode acabar sendo implementada em vários serviços.

- 4. Problemas de concorrência

Uma simples validação na aplicação pode não ser suficiente quando duas requisições acontecem simultaneamente.

Por isso, determinadas garantias devem ser realizadas pelo próprio banco.

### 1.4 Comparativo BD x Aplicação
Critério	                        Banco de Dados	                                            Aplicação
Integridade dos dados	            Excelente	                                                Depende da aplicação
Unicidade	                        Excelente com UNIQUE	                                    Precisa de controle adicional
Integridade referencial	            Excelente com FOREIGN KEY	                                Mais trabalhosa
Regras complexas	                Pode ficar difícil de manter	                            Mais adequada
Regras que mudam frequentemente	    Menos conveniente	                                        Mais conveniente
Concorrência	                    Forte suporte através de transações	                        Depende do uso correto do banco
Testes unitários	                Mais trabalhosos	                                        Mais simples
Portabilidade	                    Pode diminuir com recursos específicos do SGBD	            Geralmente maior
Centralização	                    Alta	                                                    Pode ser baixa em sistemas distribuídos
Experiência do usuário	            Limitada	                                                Excelente para mensagens e fluxos
Segurança da integridade	        Muito alta	                                                Depende do acesso ao banco
Desempenho	                        Pode ser excelente para validações próximas aos dados	    Pode evitar operações desnecessárias, dependendo do caso

Portanto, não se trata simplesmente de escolher "banco" ou "aplicação". O mais importante é determinar qual camada possui a responsabilidade mais adequada para cada tipo de regra.

### 1.5 Análise crítica: qual a melhor opção?

A análise dos dois modelos mostra que não existe uma solução universal.

A melhor abordagem é uma combinação das duas camadas, evitando, entretanto, que a mesma regra seja implementada de maneira independente e divergente em vários lugares.

Uma divisão razoável seria:

BANCO DE DADOS
│
├── PRIMARY KEY
├── FOREIGN KEY
├── UNIQUE
├── NOT NULL
├── CHECK
└── Integridade/concordância dos dados
          
          ↓


APLICAÇÃO
│
├── Fluxos de negócio
├── Cálculos
├── Descontos
├── Autorizações
├── Regras complexas
└── Processos que mudam frequentemente
Sistema acessado por múltiplas aplicações

Nesse cenário, o banco deve assumir uma responsabilidade maior pela integridade.

Por exemplo, se existem:

aplicativo mobile;
site;
sistema interno;
API externa;

todos podem tentar cadastrar clientes.

Se a regra "CPF deve ser único" existir somente no código do site, o aplicativo mobile ou outra integração poderá violá-la.

Por isso:

cpf VARCHAR(11) UNIQUE NOT NULL

é uma garantia mais forte.

A aplicação ainda pode verificar antecipadamente se o CPF existe para oferecer uma mensagem amigável ao usuário, mas a garantia definitiva deve estar no banco.

Dados sensíveis ou exigências legais/fiscais

Quando a consistência dos dados é crítica, não é recomendável depender exclusivamente da aplicação.

Restrições como:

unicidade;
integridade referencial;
campos obrigatórios;
valores válidos;

devem ser protegidas no banco.

Isso cria uma segunda barreira contra erros de programação e acessos que não passem pela aplicação.

Regras que mudam frequentemente

Imagine:

"Durante o mês de dezembro, clientes VIP recebem 20% de desconto."

Depois:

"A partir de janeiro, o desconto será de 10%."

Esse tipo de regra é mais adequado à aplicação, pois faz parte do comportamento comercial e pode sofrer alterações frequentes.

Também é possível parametrizar algumas dessas regras em tabelas de configuração, evitando alterar o código a cada mudança.

Protótipos e equipes pequenas

Em um protótipo, pode ser tentador colocar toda a lógica na aplicação pela velocidade de desenvolvimento.

Isso pode ser aceitável em determinadas situações, desde que as regras fundamentais de integridade não sejam negligenciadas.

Mesmo em um projeto pequeno, utilizar:

PRIMARY KEY
UNIQUE
NOT NULL
FOREIGN KEY
CHECK

costuma ser uma decisão de baixo custo e alto benefício.


## 2. Exemplos e Casos

### 2.1 Exemplo em PostgreSQL: regra no banco

Considere um sistema de vendas.

As regras são:

Todo cliente deve possuir CPF.
O CPF deve ser único.
Todo pedido deve estar associado a um cliente existente.
O preço de um produto deve ser positivo.
A quantidade de estoque não pode ser negativa.

Podemos representar essas regras da seguinte maneira:

CREATE TABLE clientes (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    cpf VARCHAR(11) NOT NULL UNIQUE
);


CREATE TABLE produtos (
    id SERIAL PRIMARY KEY,
    nome VARCHAR(100) NOT NULL,
    preco NUMERIC(10,2) NOT NULL CHECK (preco > 0),
    estoque INTEGER NOT NULL CHECK (estoque >= 0)
);


CREATE TABLE pedidos (
    id SERIAL PRIMARY KEY,
    cliente_id INTEGER NOT NULL,
    status VARCHAR(20) NOT NULL DEFAULT 'ABERTO',


    FOREIGN KEY (cliente_id)
        REFERENCES clientes(id),


    CHECK (status IN ('ABERTO', 'PAGO', 'CANCELADO'))
);

Nesse exemplo:

PRIMARY KEY identifica cada registro;
NOT NULL impede valores ausentes;
UNIQUE impede CPFs duplicados;
FOREIGN KEY garante a existência do cliente;
CHECK restringe valores inválidos.

Essas são exatamente as situações para as quais as constraints do PostgreSQL são destinadas.

### 2.2 Regra de negócio na aplicação

Agora considere a seguinte regra:

Um cliente não pode criar um novo pedido enquanto possuir outro pedido aberto.

Essa regra pode envolver uma consulta e uma decisão de negócio.

Um pseudocódigo seria:

function criarPedido(clienteId) {


    const pedidoAberto = buscarPedidoAberto(clienteId);


    if (pedidoAberto) {
        throw new Error(
            "O cliente já possui um pedido em aberto."
        );
    }


    return criarNovoPedido(clienteId);
}

A aplicação pode ser responsável por essa decisão porque ela faz parte do fluxo de negócio.

Entretanto, se essa regra for realmente uma garantia de integridade que precisa ser respeitada sob concorrência, simplesmente consultar antes de inserir pode não ser suficiente.

Imagine duas requisições simultâneas:

Requisição A → verifica → nenhum pedido aberto
Requisição B → verifica → nenhum pedido aberto


Requisição A → cria pedido
Requisição B → cria pedido

As duas podem passar pela validação antes que qualquer uma delas perceba a alteração da outra.

Nesse tipo de situação, a implementação precisa considerar mecanismos de concorrência e transação do banco. O PostgreSQL documenta diferentes níveis de isolamento justamente para controlar efeitos de transações concorrentes.

### 2.3 Caso realista: sistema de vendas

Considere uma empresa que possui:

                ┌── Site
                │
Cliente ────────┼── Aplicativo
                │
                ├── Sistema interno
                │
                └── API
                       │
                       ↓
                  PostgreSQL

A empresa possui a regra:

CPF deve ser único.

Se essa regra estiver somente no site, o aplicativo poderá cadastrar o mesmo CPF.

Uma solução melhor é:

Aplicação

Verificar previamente:

if (cpfJaExiste(cpf)) {
    return "CPF já cadastrado";
}

Isso melhora a experiência do usuário.

Banco

Manter:

cpf VARCHAR(11) UNIQUE NOT NULL

Isso garante a integridade mesmo que outra aplicação tente inserir o mesmo CPF.

Portanto, as duas camadas podem trabalhar juntas, mas possuem responsabilidades diferentes:

Aplicação
→ valida antecipadamente
→ informa o usuário
→ controla o fluxo


Banco
→ garante a integridade
→ impede o dado inválido

Essa separação também está alinhada à ideia de separar a lógica de domínio da camada de acesso aos dados. Fowler descreve arquiteturas em camadas nas quais apresentação, lógica de domínio e acesso a dados possuem responsabilidades distintas.

## 3. Referências
PostgreSQL Global Development Group. PostgreSQL Documentation — Constraints. Disponível em: PostgreSQL Documentation — Constraints. Acesso em: 18 ago. 2026.
PostgreSQL Global Development Group. PostgreSQL Documentation — Transaction Isolation. Disponível em: PostgreSQL Documentation — Transaction Isolation. Acesso em: 18 ago. 2026.
PostgreSQL Global Development Group. PostgreSQL Documentation — Transaction Processing. Disponível em: PostgreSQL Documentation — Transaction Processing. Acesso em: 18 ago. 2026.
FOWLER, Martin. Domain Logic and SQL. Disponível em: Martin Fowler — Domain Logic and SQL. Acesso em: 18 ago. 2026.
FOWLER, Martin. Service Layer. Disponível em: Martin Fowler — Service Layer. Acesso em: 18 ago. 2026.
FOWLER, Martin. Patterns of Enterprise Application Architecture. Addison-Wesley, 2002.

## 4. Conclusões

A análise realizada demonstra que não é adequado considerar banco de dados e aplicação como alternativas mutuamente exclusivas.

O banco de dados possui uma responsabilidade fundamental na proteção da integridade dos dados. Regras como unicidade, obrigatoriedade, integridade referencial e limites simples de valores são exemplos que devem ser protegidos por constraints sempre que possível.

Por outro lado, a aplicação é mais adequada para regras relacionadas ao comportamento do sistema, especialmente quando envolvem fluxos, cálculos complexos, integrações externas, autorização e políticas comerciais que podem sofrer alterações frequentes.

Uma conclusão importante é que a aplicação não deve ser considerada a única responsável pela consistência dos dados. Se uma regra fundamental existir somente no código, qualquer outra aplicação ou processo que acesse o banco poderá potencialmente violá-la.

Da mesma forma, colocar toda a lógica de negócio no banco também não é uma solução ideal. Isso pode aumentar o acoplamento com o SGBD e dificultar a manutenção de regras complexas.

Assim, a posição do grupo é a favor de uma arquitetura híbrida, na qual cada camada possui responsabilidades bem definidas:

O banco deve garantir a integridade dos dados; a aplicação deve coordenar e implementar a lógica de negócio complexa.

Por exemplo, no sistema de vendas:

CPF único
        ↓
Banco de dados → UNIQUE


Cliente existente
        ↓
Banco de dados → FOREIGN KEY


Preço válido
        ↓
Banco de dados → CHECK


Desconto para cliente VIP
        ↓
Aplicação → Regra de negócio


Fluxo de pagamento
        ↓
Aplicação → Serviço de negócio


Concorrência e consistência
        ↓
Banco + Transações


Experiência e mensagens ao usuário
        ↓
Aplicação

Portanto, a melhor arquitetura não consiste em escolher exclusivamente entre banco ou aplicação, mas em atribuir cada responsabilidade à camada mais adequada, evitando tanto a fragilidade de depender somente da aplicação quanto o excesso de lógica dentro do banco.

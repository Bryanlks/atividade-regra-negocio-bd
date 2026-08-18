# Atividade Teórica: Regra de Negócio no BD versus na Aplicação

**Aluno(s):** Ryan Torres, Pedro Lucas Santana, Gerson Adriano, João Andre, Nickolas Silva
**Turma:** Banco de Dados 2026  
**Data:** 18/08/2026  
**Repositório Git:** https://github.com/Bryanlks/atividade-regra-negocio-bd

Resumo Executivo

As regras de negócio são condições que determinam como um sistema deve funcionar de acordo com as necessidades de uma organização. Em um sistema de vendas, por exemplo, podem existir regras como impedir a venda de produtos sem estoque, exigir idade mínima para cadastro ou garantir que um CPF pertença a apenas um cliente.

Uma questão importante de arquitetura é decidir onde essas regras devem ser implementadas: no banco de dados, na aplicação ou em uma combinação dos dois.

A posição defendida neste trabalho é que não existe uma opção vencedora absoluta. O banco de dados deve ser responsável principalmente pelas regras de integridade e consistência dos dados, enquanto a aplicação deve concentrar as regras de processo, comportamento e negócio que sejam complexas ou mudem frequentemente.

Essa abordagem evita depender exclusivamente da aplicação para garantir a integridade dos dados, mas também evita transformar o banco em um local onde toda a lógica do sistema fique concentrada. Dessa maneira, cada camada exerce uma responsabilidade adequada, proporcionando maior segurança, manutenção e flexibilidade.

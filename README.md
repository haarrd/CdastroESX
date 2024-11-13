Este projeto é um **sistema de cadastro** simples desenvolvido com a biblioteca `tkinter` para a interface gráfica e `sqlite3` para o banco de dados. O objetivo principal é gerenciar registros de pacientes, permitindo que o usuário adicione, pesquise e delete informações de maneira intuitiva. Vou destacar os principais recursos e funcionalidades do sistema:

### Funcionalidades:
1. **Cadastro de Pacientes**:
   - O sistema permite o registro de pacientes através de três campos principais: nome, prontuário e endereço.
   - O prontuário precisa estar dentro de um intervalo específico (1000-1300), garantindo a validade dos dados inseridos.
   - Caso o prontuário já exista no banco de dados, o sistema emite uma mensagem de erro, evitando duplicidade de registros.

2. **Pesquisa**:
   - O sistema possui uma funcionalidade de pesquisa que permite buscar registros de pacientes com base em seu nome, prontuário ou endereço.
   - A pesquisa é realizada com base no texto digitado, e os resultados são exibidos em uma tabela.

3. **Exclusão de Registros**:
   - O usuário pode excluir registros específicos, com confirmação de ação para evitar exclusões acidentais.

4. **Interface Gráfica**:
   - A interface gráfica é construída com `tkinter`, proporcionando uma experiência de uso amigável.
   - Os campos de entrada e os botões têm uma formatação clara e os dados são apresentados em uma tabela (`Treeview`).
   - Há também um ajuste dinâmico da largura das colunas da tabela quando a janela é redimensionada.

5. **Banco de Dados**:
   - O banco de dados `SQLite` armazena as informações dos pacientes de forma persistente.
   - A tabela no banco de dados é criada automaticamente na primeira execução do programa, se não existir, com três colunas: `nome`, `prontuario`, e `endereco`.

### Propósito:
Este sistema serve como uma **ferramenta de gerenciamento de registros** em um cenário como o de clínicas, hospitais ou qualquer outro tipo de instituição que precise controlar informações de pacientes. Ele é bastante simples, mas eficaz para gerenciar dados essenciais, oferecendo ao usuário uma maneira fácil de adicionar, consultar e remover registros. A interface amigável e a estrutura de banco de dados garantem a eficiência do sistema para essas tarefas.

Esse tipo de solução pode ser útil para profissionais da área da saúde, ou pequenas clínicas, onde a organização das informações dos pacientes pode ser feita de forma prática, sem a necessidade de sistemas mais complexos ou caros. 

### Pontos de Melhoria:
- **Validação de Dados**: Pode-se expandir a validação de dados, por exemplo, checando o formato do endereço ou adicionando outros tipos de validação para campos como o nome.
- **Segurança**: Para uma aplicação real, seria importante implementar mecanismos de segurança, como a proteção de dados sensíveis e o controle de acessos.
- **Recursos Avançados**: O sistema pode ser melhorado com funcionalidades como a edição de registros, exportação de dados para arquivos (como CSV ou PDF), ou integração com sistemas mais complexos.

No geral, é um excelente projeto de aprendizado para quem está começando a trabalhar com Python e desenvolvimento de aplicações de gerenciamento de dados!

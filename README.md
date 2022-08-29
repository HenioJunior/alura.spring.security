## Notas

### Criação da classe SecurityConfigurations

- Criação do método de autorização: SecurityFilterChain filterChain(HttpSecurity http) - NOVA IMPLEMENTAÇÃO
- Utilizar as anotações de classe: @EnableWebSecurity @Configuration
- Esta classe estende WebSecurityConfigurerAdapter - DEPRECATED

### Liberando os endpoints

- Criação do método `configure(HttpSecurity http)` <--- Configurações de autorização
- os métodos são chamados apartir do parâmetro http

### Restrigindo o acesso aos endpoints privados(Autenticação)

#### Implementando a logica de autenticação

 - Implementação da Interface UserDetails apartir da classe Usuario;

 - Criação da classe Perfil(Role) e implementacao da interface GrantedAuthority para o spring saber
    que esta é a classe das Roles. Esta interface só tem um método: getAuthority();

 - Na classe Usuario, criar a cardinalidade @ManyToMany no atributo perfis
    Um usuario pode ter vários perfis e vice versa
    Por padrão na Jpa todo o relacionamento toMany, se eu carregar do banco de dados\n
    
    o usuario ele nao carrega a lista de Perfis porque é LAZY
    Para isso o fetch precisa ser EAGER --> @ManyToMany(fetch = FetchType.EAGER)




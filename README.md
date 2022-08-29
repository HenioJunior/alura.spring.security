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
    No Jpa, todo o relacionamento toMany tem o LAZY como fetch padrão.
    ex: Se eu carregar do banco de dados o usuario ele nao carrega a lista de Perfis. Para isso o fetch precisa ser EAGER --> @ManyToMany(fetch = FetchType.EAGER)

#### Ensinando o Spring: Como logar no sistema?

- Criação da classe responsável pela lógica de autenticação --> security/AutenticacaoService

    Esta classe será gerenciada pelo Spring anotação --> @Service
    
- implementar a interface UserDetailsService
        implementação do método loadUserByUsername()
        ##Explicação: No form Login, qdo clicar no botão "Sign in", o Spring vai saber que esta <br/>
        é a classe de Autenticação(AutenticacaoService) e vai chamar o método loadUserByUserame()

- Criação do UsuarioRepository para fazer a pesquisa no banco e injetar na classe AutenticacaoService
        Criação da assinatura do método -> `Optional<Usuario> findByEmail(String email);` <br />
         <br /> 
         O controller de autenticação é do próprio Spring
          <br />

### Em SecurityConfigurations

- injetar a AutenticacaoService;
- Criação do método `void configure(AuthenticationManagerBuilder auth)`;
- chamar o método `auth.userDetailsService()`;
- chamada do método `.passwordEncoder(new BCryptPasswordEncoder())` qual o algoritmo de geração de hash para a senha;

 





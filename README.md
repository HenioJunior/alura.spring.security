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

### Gerando token com JWT

#### Configurando autenticação Stateless

- Instalação da dependencia `<groupId>io.jsonwebtoken</groupId>`
- Em SecurityConfigurations `void configure(HttpSecurity http)`
    inclusão do método `.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);` Com isso, o projeto não criará session, pois iremos utilizar token;
    Perdemos o Controller nativo do Spring --> Necessário a criação;

### Criação do `AutenticacaoController`

- Criação do método `ResponseEntity<?> autenticar(@RequestBody @Valid LoginForm form)`
    O objetivo deste método é pegar o login e a senha, autenticar no sistema(verificar no banco) e estando tudo ok eu gero o token;

- Criação da classe DTO `LoginForm`LoginForm
- Liberar o caminho `/auth` em SecurityConfigurations

- Injetar o `AuthenticationManager`, pois como não estamos mais utilizando o formulário do Spring, faremos essa chamada para a autenticação manualmente;
Só que esta classe não vem configurada para a injeção de dependência.

Precisamos ir em `SecurityConfigurations`
A classe `WebSecurityConfigurerAdapter` tem um método que sabe criar o `AuthenticationManager`;

Vamos sobreescrever o método `protected AuthenticationManager authenticationManager()`;
Precisamos anotar com `@Bean` Com isso o Spring sabe que este método devolve o AuthenticationManager e com isso conseguimos injetar no Controller;

- Voltando ao método `ResponseEntity<?> autenticar(@RequestBody @Valid LoginForm form)`

Preciso de um objeto do tipo `UsernamePasswordAuthenticationToken` que vai receber dadosLogin convertidos pelo método `form.converter()`, pois precisamos dos dados de login. Não posso passar o LoginForm e nem o login/senha solta...

Vamos chamar um método do tipo `Authentication authManager.authenticate(dadosLogin)`

Quando o Spring chamar a linha acima, o Spring vai ler as configurações e sabe que é para chamar o `AutenticacaoService` que chama o Repository para consultar o dados de login no banco

Implemento um tratamento para o caso de Exception

Com isso a autenticação esta implementada













 





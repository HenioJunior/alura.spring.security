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

- Em uma API Rest, não é uma boa prática utilizar autenticação com o uso de session;
- Uma das maneiras de fazer autenticação stateless é utilizando tokens JWT (Json Web Token);

- Para utilizar JWT na API, devemos adicionar a dependência da biblioteca jjwt no arquivo pom.xml do projeto --> `<groupId>io.jsonwebtoken</groupId>`

- Para configurar a autenticação stateless no Spring Security, devemos utilizar o método 
`.sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS);` localizado em SecurityConfigurations `void configure(HttpSecurity http)`
   
- Perdemos o Controller nativo do Spring. Com isso, é necessário a criação do AutenticacaoController;

### Para disparar manualmente o processo de autenticação no Spring Security, devemos utilizar a classe AuthenticationManager;

Para poder injetar o AuthenticationManager no controller, devemos criar um método anotado com @Bean, na classe SecurityConfigurations, que retorna uma chamada ao método super.authenticationManager();



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


### Criação do Token

Antes de devolver um ok `return ResponseEntity.ok().build();` preciso gerar um token;

Para criar o token JWT, devemos utilizar a classe Jwts da biblioteca `jjwt`;

Criação do método `tokenService.gerarToken(authentication);` na classe `TokenService` que será injetada no Controller;

#### Na classe TokenService;

Preciso injetar os atributos expiration e secret que configurei em `application.properties` com a anotação `@Value`para serem utilizados durante a criação do token;

- No método `gerarToken(authentication)`

O parâmetro authentication tem o método `getPrincipal()` para recuperar o usuário que esta logado. Ele devolve um Object e com isso, é necessário o cast para Usuario;

- Vamos utilizar a api do jjwt para a geração do token;

`Jwts.builder()`é um método utilizado para construir o token;
a propriedade `compact()` transforma para uma String;

### Devolvendo o token para o cliente

- Criação do `TokenDto`;
- `No ResponseEntity.ok(new TokenDto(token, "Bearer"))`
devolverei um objeto(token) no corpo da resposta;
junto com o token preciso informar o tipo de autenticação(Bearer);

- Bearer é um dos mecanismos de autenticação utilizados no protocolo HTTP, tal como o Basic e o Digest;

Para enviar o token JWT na requisição, é necessário adicionar o cabeçalho Authorization, passando como valor Bearer token;

### Recuperando o token do header Authorization

#### Chegou a requisição com o token -> recuperar/validar o token -> autenticar o usuario antes de prosseguir com a requisição

- Interceptar a requisição com o uso de filtro;

Para criar um filtro no Spring, devemos criar uma classe(AutenticacaoViaTokenFilter) que herda da classe OncePerRequestFilter;

Implementação do método abstrato `protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain)`;

1º passo recuperar o token JWT da requisição no filter, criar um método privado `recuperarToken(HttpServletRequest request)`;

- Devemos chamar o método `request.getHeader("Authorization");` e guardar na variavel `String token`;

- Precisamos verificar se o cabeçalho esta correto`if (token == null || token.isEmpty() || !token.startsWith("Bearer "))`;

- Caso o token esteja ok `return token.substring(7, token.length());`
substring é para capturar o token sem o `Bearer `;







Para habilitar o filtro no Spring Security, devemos chamar o método and().addFilterBefore(new AutenticacaoViaTokenFilter(), UsernamePasswordAuthenticationFilter.class);

Para indicar ao Spring Security que o cliente está autenticado, devemos utilizar a classe SecurityContextHolder, chamando o método SecurityContextHolder.getContext().setAuthentication(authentication).
























 





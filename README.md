## Notas

### Criação da classe SecurityConfigurations

- Criação do método de autorização: SecurityFilterChain filterChain(HttpSecurity http) - NOVA IMPLEMENTAÇÃO
- Utilizar as anotações de classe: @EnableWebSecurity @Configuration
- Esta classe estende WebSecurityConfigurerAdapter - DEPRECATED

### Liberando os endpoints

- Criação do método `configure(HttpSecurity http)` <--- Configurações de autorização
- os métodos são chamados apartir do parâmetro http




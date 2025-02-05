# Spring Security Architecture Example

#### Dependencies 

```xml
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-api</artifactId>
    <version>0.11.5</version>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-impl</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>io.jsonwebtoken</groupId>
    <artifactId>jjwt-jackson</artifactId>
    <version>0.11.5</version>
    <scope>runtime</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

- Application property file

```yml
security:
  jwt:
    secret-key: 5367566B59703373367639792F423F4528482B4D6251655468576D5A71347437
    expiration-time: 864_000_000
```

#### User model

```java
@Table(name = "users")
@Entity
public class User implements UserDetails {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    private String username;
    private String password;

    @Email
    @Column(unique = true)
    private String mail;

    private String idTag;
    private String companyName;
    private String role;

    @Column(name = "user_role", nullable = true)
    @Enumerated(EnumType.STRING)
    private Role userRole;

    @OneToOne(cascade = CascadeType.ALL)
    @JoinColumn(name = "address_id", referencedColumnName = "id")
    private Address address;

    private ZonedDateTime createdAt;
    private ZonedDateTime updatedAt;
    private ZonedDateTime deletedAt;

    @PrePersist
    private void onCreate(){
        final ZonedDateTime now = ZonedDateTime.now();
        this.createdAt = now;
        this.updatedAt = now;
    }

    @PostUpdate
    private void onUpdate(){
        this.updatedAt = ZonedDateTime.now();
    }


    @Override
    public String getUsername() {
        return mail;
    }


    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(userRole);
    }

    @Override
    public boolean isAccountNonExpired() {
        return true;
    }

    @Override
    public boolean isAccountNonLocked() {
        return true;
    }

    @Override
    public boolean isCredentialsNonExpired() {
        return true;
    }

    @Override
    public boolean isEnabled() {
        return true;
    }
}
```

```
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(userRole);
    }
```
used for authorities

#### Services

- JWT Service 

```java
@Service
@Slf4j
public class JwtService {

    @Value("${security.jwt.secret-key}")
    private String secretKey;

    @Value("${security.jwt.expiration-time}")
    private long jwtExpiration;


    public String extractUsername(String token) {
        return extractClaim(token, Claims::getSubject);
    }

    public <T> T extractClaim(String token, Function<Claims, T> claimsResolver) {
        final Claims claims = extractAllClaims(token);
        return claimsResolver.apply(claims);
    }

    public String generateToken(UserDetails userDetails) {
        return generateToken(new HashMap<>(), userDetails);
    }

    public String generateToken(Map<String, Object> extraClaims, UserDetails userDetails) {
        return buildToken(extraClaims, userDetails, jwtExpiration);
    }

    public long getExpirationTime() {
        return jwtExpiration;
    }

    private String buildToken(Map<String, Object> extraClaims, UserDetails userDetails, long expiration) {
        return Jwts.builder()
                .setClaims(extraClaims)
                .setSubject(userDetails.getUsername())
                .setIssuedAt(new Date(System.currentTimeMillis()))
                .setExpiration(new Date(System.currentTimeMillis() + 1000 * 60 * 30))
                .signWith(getSignInKey(), SignatureAlgorithm.HS256)
                .compact();
    }

    public boolean isTokenValid(String token, UserDetails userDetails) {
        final String username = extractUsername(token);
        return (username.equals(userDetails.getUsername())) && !isTokenExpired(token);
    }

    private boolean isTokenExpired(String token) {
        return extractExpiration(token).before(new Date());
    }

    private Date extractExpiration(String token) {
        return extractClaim(token, Claims::getExpiration);
    }

    private Claims extractAllClaims(String token) {
        return Jwts
                .parserBuilder()
                .setSigningKey(getSignInKey())
                .build()
                .parseClaimsJws(token)
                .getBody();
    }

    private Key getSignInKey() {
        byte[] keyBytes = Decoders.BASE64.decode(secretKey);
        return Keys.hmacShaKeyFor(keyBytes);
    }

}
```


- UserDetailService 

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class UserServiceImpl implements UserService, UserDetailsService {


    private final UserRepository userRepository;

    @Override
    public User findUserWithIdTag(String idTag) {
        return null; // from UserService
    }

    @Override
    public User findUserWithId(String userId) {
        return null; // from UserService
    }

    @Override
    public boolean validateUserWithIdTag(String idTag) {
        return false; // from UserService
    }

    public User saveUser(@NonNull User user){
        return userRepository.save(user);
    }


    public User findUserByEmail(@NonNull String email){
        return userRepository.findByMail(email).orElseThrow(RuntimeException::new);
    }

    public boolean existsByEmail(@NonNull String email){
        return userRepository.existsUserByMail(email);
    }

    @Override // from UserDetailService
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        final Optional<User> user = userRepository.findByMail(username);
        User user1 = user.get();
        UserDetails details = new org.springframework.security.core.userdetails.User(user1.getMail(), user1.getPassword(), user1.getAuthorities());
        return details;
    }

}
```

loadByUsername method
```java
@Override // from UserDetailService
public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
    final Optional<User> user = userRepository.findByMail(username);
    User user1 = user.get();
    UserDetails details = new org.springframework.security.core.userdetails.User(user1.getMail(), user1.getPassword(), user1.getAuthorities());
    return details;
}
```
this method should return user details with username(can be email, name etc), password and authorities

- JWT Authentication Filter -> once per request filter

public class JwtAuthenticationFilter extends OncePerRequestFilter

```java

@Component
@Slf4j
@RequiredArgsConstructor
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserServiceImpl userService;

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException{
        final String authHeader = request.getHeader("Authorization");
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            log.error("Invalid token!");
            filterChain.doFilter(request, response);
            return;
        }

        final String jwt = authHeader.substring(7);
        final String userEmail = jwtService.extractUsername(jwt);

        Authentication authentication = SecurityContextHolder
                .getContext()
                .getAuthentication();
        if (userEmail != null && authentication == null) {
            UserDetails userDetails = userService.loadUserByUsername(userEmail);
            log.info("User details are found, email is not null and auth is null");


            if (jwtService.isTokenValid(jwt, userDetails)) {
                log.info("Token valid...!!!!");
                UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
                        userDetails,
                        null,
                        userDetails.getAuthorities()
                );

                authenticationToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder
                        .getContext()
                        .setAuthentication(authenticationToken);

            }
        }

        filterChain.doFilter(request, response);
    }

}
```

- Security Config

```java
Configuration
@EnableWebSecurity
public class SecurityConfig{

    private final UserServiceImpl userServiceImpl;
    private final JwtAuthenticationFilter jwtAuthenticationFilter;
    private final LogoutHandlerService logoutHandlerService;

    public SecurityConfig(UserServiceImpl userServiceImpl, JwtAuthenticationFilter jwtAuthenticationFilter, LogoutHandlerService logoutHandlerService) {
        this.userServiceImpl = userServiceImpl;
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
        this.logoutHandlerService = logoutHandlerService;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
                .csrf(csrf -> csrf.disable()) // Disable CSRF for stateless APIs
                .authorizeHttpRequests(auth -> auth
                        .requestMatchers("/api/auth/login", "/api/auth/register").permitAll()
                        .requestMatchers("/api/auth/test2").hasAuthority("ADMIN")
                        .requestMatchers("/api/auth/test").hasAuthority("USER")
                        .anyRequest().authenticated() // Protect all other endpoints
                )
                .sessionManagement(sess -> sess
                        .sessionCreationPolicy(SessionCreationPolicy.STATELESS) // No sessions
                )
                .authenticationProvider(authenticationProvider()) // Custom authentication provider
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class); // Add JWT filter

        return http.build();
    }

/*
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return http.csrf(AbstractHttpConfigurer::disable) // Disable CSRF for JWT
//                .authorizeRequests(request -> request
//                        .requestMatchers("/api/auth/register", "/api/auth/login")
//                        .permitAll())
                .authorizeHttpRequests(c -> c.requestMatchers("/api/auth/register", "api/auth/login").permitAll())
                .sessionManagement(httpSecuritySessionManagementConfigurer -> httpSecuritySessionManagementConfigurer.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authenticationProvider(authenticationProvider())
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .build();
    }
    */

/*
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception{
        return http.csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(x ->
                        x.requestMatchers(
//                                "/api/auth/test/**",
//                                "/api/**",
                                "/api/auth/register/**",
                                "/api/auth/login/**",
                                "/api/v1/auth/**",
                                "/v2/api-docs",
                                "/v3/api-docs",
                                "/v3/api-docs/**",
                                "/swagger-resources",
                                "/swagger-resources/**",
                                "/configuration/ui",
                                "/configuration/security",
                                "/swagger-ui/**",
                                "/webjars/**",
                                "/api/swagger-ui.html"
                        ).permitAll()
                )
//                .authorizeHttpRequests(x ->
//                        x.requestMatchers("/auth/user").authenticated()
//                                .requestMatchers("/auth/admin").hasRole("ADMIN")
//                )
                .sessionManagement(x -> x.sessionCreationPolicy(SessionCreationPolicy.STATELESS))
                .authenticationProvider(authenticationProvider())
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
//                .logout(x -> {
//                    x.logoutUrl("/auth/logout");
//                    x.addLogoutHandler(logoutHandlerService);
//                    x.logoutSuccessHandler(((request, response, authentication) -> SecurityContextHolder.clearContext()));
//                })
                .build();
    }
*/

    @Bean
    public AuthenticationProvider authenticationProvider() {
        final DaoAuthenticationProvider authenticationProvider = new DaoAuthenticationProvider();
        authenticationProvider.setUserDetailsService(userServiceImpl);
        authenticationProvider.setPasswordEncoder(passwordEncoder());
        return authenticationProvider;
    }

    @Bean
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration) throws Exception {
        return configuration.getAuthenticationManager();
    }

    @Bean
    public PasswordEncoder passwordEncoder() {
        return new BCryptPasswordEncoder();
    }

}
```


```java
.authorizeHttpRequests(auth -> auth
        .requestMatchers("/api/auth/login", "/api/auth/register").permitAll()
        .requestMatchers("/api/auth/test2").hasAuthority("ADMIN")
        .requestMatchers("/api/auth/test").hasAuthority("USER")
        .anyRequest().authenticated() // Protect all other endpoints
)
```

authorizing requests and making authority for endpoints

```java
//                .logout(x -> {
//                    x.logoutUrl("/auth/logout");
//                    x.addLogoutHandler(logoutHandlerService);
//                    x.logoutSuccessHandler(((request, response, authentication) -> SecurityContextHolder.clearContext()));
//                })
```
can be used for logout handling

**Example logout handler**

```java
@Service
@Slf4j
public class LogoutHandlerService implements LogoutHandler {

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {
        // todo -> will be implemented
    }

}
```



**Authentication Service**

```java

@Service
@RequiredArgsConstructor
@Slf4j
public class AuthenticationService {

    private final AuthenticationManager authenticationManager;
    private final UserServiceImpl userService;
    private final JwtService jwtService;
    private final PasswordEncoder passwordEncoder;

    public LoginResponse login(AuthRequest authRequest) {
        Authentication authentication = authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(authRequest.getMail(), authRequest.getPassword()));
        if (authentication.isAuthenticated()) {
            final User user = userService.findUserByEmail(authRequest.getMail());
            final String stringToken = jwtService.generateToken(user);

            final UsernamePasswordAuthenticationToken authenticationToken = new UsernamePasswordAuthenticationToken(
                    authRequest.getMail(),
                    authRequest.getPassword()
            );

            authenticationManager.authenticate(authenticationToken);

            return LoginResponse.builder()
                    .mail(user.getMail())
                    .token(stringToken)
                    .build();
        }
        log.info("Invalid username {} " + authRequest.getMail());
        throw new UsernameNotFoundException("invalid username {}" + authRequest.getMail());
    }


    public LoginResponse register(UserCreateRequest request){

        // check user if user already created
        if (userService.existsByEmail(request.getMail())){
            final AuthRequest authRequest = new AuthRequest(request.getMail(), request.getPassword());
            log.info("User is already registered");
            return login(authRequest);
        }

        final User user = new User();
        user.setMail(request.getMail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setUserRole(Role.ADMIN);

        userService.saveUser(user);

        final AuthRequest newUser = new AuthRequest(request.getMail(), request.getPassword());
        return login(newUser);
    }

}
```
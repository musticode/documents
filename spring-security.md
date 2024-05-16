# SpringBoot Security Implementation


## Architecture

- When the request from the user accesses our application, AuthenticationFilter takes the username and password from the request and an object is created.
- Using the created Authentication Object comes to Authentication Manager after the filter. The created object contains Granted Authority, Roles and Principal information. Authentication Manager is an interface and the authentication method is run. Authentication Manager is an interface and sends data to the Authentication Provider.
- It informs the Authentication Provider which type of verification will be performed in authentication transactions.
- Authentication Provider has different providers such as LdapAuthenticationProvider, CasAuthenticationProvider, DaoAuthenticationProvider. Authentication Provider selects the most suitable provider.
- The Authentication Provider calls the User Details Service and retrieves the user information for the corresponding user. Using the service, it retrieves the User Object corresponding to the username.
- In the User Details Service, it accesses the corresponding in-memory or database or from whatever sources it needs to access by looking at information such as the loadUserByUsername method (whether the account is locked or active, whether the credentials have expired) and brings the incoming user information it finds and if the correct user is used. If found and returns the object of the found user. This service includes Password Encoder, which verifies the user password. Password Encoder is the interface that tells the user password to be encoded and decrypted.
- Security Context contains information about the authenticated user. We can use this user information throughout our application thanks to SecurityContextHolder.
- When an unauthenticated user is encountered, an Authentication Exception is thrown.
- The authenticated user is kept in the Security Context within our application. When the user who has logged in later requests access again, Security Context decides whether to log in or not. If the user information is available in the Security Context, there will be no need to log in again.


## Implementation Details

dependencies:
**pom.xml**

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-security</artifactId>
</dependency>
```

### Configuration Classes

**SecurityConfig**

```java
@EnableWebSecurity
@Configuration
public class SecurityConfig {

    private final UserServiceImpl userService;
    private final CustomLogoutHandler logoutHandler;
    private final JwtAuthenticationFilter jwtAuthenticationFilter;

    public SecurityConfig(UserServiceImpl userService, CustomLogoutHandler logoutHandler, JwtAuthenticationFilter jwtAuthenticationFilter) {
        this.userService = userService;
        this.logoutHandler = logoutHandler;
        this.jwtAuthenticationFilter = jwtAuthenticationFilter;
    }

    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        return  http
                .csrf(AbstractHttpConfigurer::disable)
                .authorizeHttpRequests(req ->
                        req.requestMatchers("api/auth/login/**", "api/auth/register/**").permitAll()
                                .requestMatchers("/admin_only/**").hasAuthority("ADMIN")
                                .anyRequest()
                                .authenticated()
                )
                .userDetailsService(userService)
                .sessionManagement(session ->
                        session.sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                )
                .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class)
                .exceptionHandling(e ->
                        e.accessDeniedHandler((request, response, accessDeniedException) ->
                                        response.setStatus(403)
                                )
                                .authenticationEntryPoint(new HttpStatusEntryPoint(HttpStatus.UNAUTHORIZED))
                )
                .logout(logout ->
                        logout.logoutUrl("/logout")
                                .addLogoutHandler(logoutHandler)
                                .logoutSuccessHandler(((request, response, authentication) -> SecurityContextHolder.clearContext()))
                )
                .build();

    }

    @Bean
    public PasswordEncoder passwordEncoder(){
        return new BCryptPasswordEncoder();
    }

    @Bean(name = BeanIds.AUTHENTICATION_MANAGER)
    public AuthenticationManager authenticationManager(AuthenticationConfiguration configuration){
        try {
            return configuration.getAuthenticationManager();
        } catch (Exception e) {
            throw new RuntimeException(e);
        }
    }

}
```

**CustomLogoutHandler**

```java
@Configuration
public class CustomLogoutHandler implements LogoutHandler {

    private final TokenService tokenService;

    public CustomLogoutHandler(TokenService tokenService) {
        this.tokenService = tokenService;
    }

    @Override
    public void logout(HttpServletRequest request, HttpServletResponse response, Authentication authentication) {

        String authHeader = request.getHeader("Authorization");
        if (authentication != null || !authHeader.startsWith("Bearer ")){
            return;
        }

        String token = authHeader.substring(7);
        Token storedToken = tokenService.findToken(token);
        if (storedToken != null){
            storedToken.setLoggedOut(true);
            tokenService.saveToken(storedToken);
        }

    }
}
```

**JwtAuthenticationFilter**

```java
@Service
@Slf4j
public class JwtAuthenticationFilter extends OncePerRequestFilter {

    private final JwtService jwtService;
    private final UserServiceImpl userService;

    public JwtAuthenticationFilter(JwtService jwtService, UserServiceImpl userService) {
        this.jwtService = jwtService;
        this.userService = userService;
    }


    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {

        String authHeader = request.getHeader("Authorization");

        if(authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request,response);
            return;
        }

        String token = authHeader.substring(7);
        String username = jwtService.extractUserName(token);

        if(username != null && SecurityContextHolder.getContext().getAuthentication() == null) {

            UserDetails userDetails = userService.loadUserByUsername(username);


            if(jwtService.isValid(token, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities()
                );

                authToken.setDetails(
                        new WebAuthenticationDetailsSource().buildDetails(request)
                );

                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        filterChain.doFilter(request, response);

    }
}
```


### Service Classess

**JwtService**
```java
@Service
@Slf4j
public class JwtService {

    private final String SECRET_KEY = "9a4f2c8d3b7a1e6f45c8a0b3f267d8b1d4e6f3c8a9d2b5f8e3a9c8b5f6v8a3d9";
    private final TokenRepository tokenRepository;

    public JwtService(TokenRepository tokenRepository) {
        this.tokenRepository = tokenRepository;
    }

    public String extractUserName(String token){
        return extractClaim(token, Claims::getSubject);
    }

    public boolean isValid(String token, UserDetails user){
        String username = extractUserName(token);
        boolean validToken = tokenRepository
                .findByToken(token)
                .map(t -> !t.isLoggedOut())
                .orElse(false);

        log.info("Valid token : {}", validToken);

        return (username.equals(user.getUsername())) && !isTokenExpired(token) && validToken;

    }

    private boolean isTokenExpired(String token){
        return extractExpiration(token).before(new Date());
    }

    public Date extractExpiration(String token){
        return extractClaim(token, Claims::getExpiration);
    }

    public <T> T extractClaim(String token, Function<Claims, T> resolver){
        Claims claims = extRactAllClaims(token);
        return resolver.apply(claims);
    }

    private Claims extRactAllClaims(String token){
        return Jwts.parser()
                .verifyWith(getSignInKey())
                .build()
                .parseSignedClaims(token)
                .getPayload();
    }


    public String generateToken(User user){
        String token = Jwts
                .builder()
                .subject(user.getUsername())
                .issuedAt(new Date(System.currentTimeMillis()))
                .expiration(new Date(System.currentTimeMillis() + 24*60*60*1000))
                .signWith(getSignInKey())
                .compact();

        return token;
    }

    private SecretKey getSignInKey(){
        byte [] keyBytes = Decoders.BASE64URL.decode(SECRET_KEY);
        return Keys.hmacShaKeyFor(keyBytes);
    }

}
```

**AuthenticationService**

```java
@Service
@Slf4j
public class AuthenticationServiceImpl implements AuthenticationService {


    private final UserRepository userRepository;
    private final TokenRepository tokenRepository;
    private final TokenService tokenService;
    private final JwtService jwtService;
    private final AuthenticationManager authenticationManager;
    private final PasswordEncoder passwordEncoder;
    private final UserServiceImpl userService;
    private final StringRedisTemplate stringRedisTemplate;
    private final RedisTemplate<String, Object> redisTemplate;
    private final CacheTokenService cacheTokenService;
    private final RedisMessagePublisher redisMessagePublisher;

    public AuthenticationServiceImpl(UserRepository userRepository, TokenRepository tokenRepository, TokenService tokenService, JwtService jwtService, AuthenticationManager authenticationManager, PasswordEncoder passwordEncoder, UserServiceImpl userService, StringRedisTemplate stringRedisTemplate, RedisTemplate<String, Object> redisTemplate, CacheTokenService cacheTokenService, RedisMessagePublisher redisMessagePublisher/*, RedisTemplate<String, Object> redisTemplate*/) {
        this.userRepository = userRepository;
        this.tokenRepository = tokenRepository;
        this.tokenService = tokenService;
        this.jwtService = jwtService;
        this.authenticationManager = authenticationManager;
        this.passwordEncoder = passwordEncoder;
        this.userService = userService;
//        this.redisTemplate = redisTemplate;
        this.stringRedisTemplate = stringRedisTemplate;
        this.redisTemplate = redisTemplate;
        this.cacheTokenService = cacheTokenService;
        this.redisMessagePublisher = redisMessagePublisher;
    }


    @Override
    public AuthenticationResponse register(UserCreateRequest request) {

        if (userRepository.findByUsername(request.getUsername()).isPresent()){
            throw new UserAlreadyExistsException("User is already exists");
        }

        User user = new User();
        user.setUsername(request.getUsername());
        user.setFirstName(request.getFirstName());
        user.setLastName(request.getLastName());
        user.setEmail(request.getEmail());
        user.setPassword(passwordEncoder.encode(request.getPassword()));
        user.setRole(Role.USER);
        userRepository.save(user);

        String jwt = jwtService.generateToken(user);
        tokenService.saveUserToken(jwt, user);

        return AuthenticationResponse.builder()
                .message("User is created")
                .token(jwt)
                .build();
    }

    @Override
    public AuthenticationResponse authenticate(LoginRequest request) {
//        /**
//         * username üzerinden user var mı diye bak
//         * user varsa cache de jwt var mı diye bak
//         * cache varsa ordan al
//         * cache'de yoksa yeni token yarat, cache'e at
//         * */
//
        String  jwt = "";
        User foundUser = userService.findUserWithUsername(request.getUsername());
        if (foundUser.isCredentialsNonExpired()){
            final String cachedToken = cacheTokenService.getCachedToken(request.getUsername());

            if (cachedToken != null){
                authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword()));
                jwt = cacheTokenService.getCachedToken(request.getUsername());
                log.info("Cached token : {}", jwt);
            }else {
                authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword()));
                jwt = jwtService.generateToken(foundUser);
                cacheTokenService.cacheToken(request.getUsername(), jwt);
            }
        }



//        final String cachedToken = getCachedToken(request.getUsername());
//        if (cachedToken.isEmpty()){
//            cacheToken(request.getUsername(), jwtService.generateToken(user));
//        }


        /*
        // authentication without cache

        authenticationManager.authenticate(new UsernamePasswordAuthenticationToken(request.getUsername(), request.getPassword()));
        User user = userService.findUserWithUsername(request.getUsername());
        String jwt = jwtService.generateToken(user);
        tokenService.revokeAllTokenByUser(user);
        tokenService.saveUserToken(jwt, user);
         */



//        cacheToken(request.getUsername(), jwt);
//        String cached = getCachedToken(request.getUsername());
//        System.out.println("caccccheeddd  "  + cached);
//
//
//        saveData(request.getUsername(), jwt);
//        log.info("tstttt : {}", getData(request.getUsername()));

        redisMessagePublisher.publish("test message");
        
        messageList.forEach(System.out::println);
        return AuthenticationResponse.builder()
                .token(jwt)
                .message("User is logged in")
                .build();

    }

    @Override
    public String logout() {
        //User user = userService.findUserWithUsername("testUser");
        List<Token> tokens = user.getTokens();
        tokens.forEach(token -> {
            if (!token.isLoggedOut()){
                token.setLoggedOut(true);
            }
        });

        cacheTokenService.removeCachedToken(user.getUsername());

        return null;
    }

}
```
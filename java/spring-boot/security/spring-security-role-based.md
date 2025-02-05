# Spring security role based authority

- ex 1
```java
@Service
public class CustomUserDetailsService implements UserDetailsService {

    @Autowired
    private UserRepository userRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        User user = userRepository.findByUsername(username);
        if (user == null) {
            throw new UsernameNotFoundException("User not found");
        }

        Set<GrantedAuthority> grantedAuthorities = new HashSet<>();
        user.getRoles().forEach(role -> {
            grantedAuthorities.add(new SimpleGrantedAuthority(role.getName()));
            role.getPermissions().forEach(permission -> {
                grantedAuthorities.add(new SimpleGrantedAuthority(permission.getName()));
            });
        });

        return new org.springframework.security.core.userdetails.User(user.getUsername(), user.getPassword(), grantedAuthorities);
    }
}
```

> https://medium.com/@Mohd_Aamir_17/implementing-role-based-access-control-rbac-in-java-987bcc393739


* ex 2

user model : 

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
```java
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(userRole);
    }
```
- Bu kısımda userRole parametresini granted authority şeklinde vermek gerekiyor. Bunu verdikten sonra hasAuthority ile security filter içinde bu api'a istek atanların authority'sini bekleyebiliriz.

```java
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
```



```java
        final Optional<User> user = userRepository.findByMail(username);
        User user1 = user.get();
        UserDetails details = new org.springframework.security.core.userdetails.User(user1.getMail(), user1.getPassword(), user1.getAuthorities());
```
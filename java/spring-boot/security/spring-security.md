# Spring Security


Spring Security’nin Mimarisinin kuş bakışı görüntüsü gördüğünüz gibidir.

- Kullanıcıdan gelen istek uygulamamıza erişmede AuthenticationFilter istekten kullanıcı adını ve parolayı alır ve bir nesne oluşturulmaktadır.

- Oluşturulan Authentication Nesnesinin kullanılması, filtrenin sonrasında Authentication Manager‘a gelir. Oluşturulan nesne içerisinde Granted Authority, Roles, Principal bilgileri bulunmaktadır. Authentication Manager bir interfacedir ve kimlik doğrulama metodu çalıştırılmaktadır. Authentication Manager bir interface olup, Authentication Provider‘a gönderir.

- Kimlik doğrulama işlemlerinde hangi tipte bir doğrulama işleminin yapılacağını Authentication Provider‘a bildirir.

- Authentication Provider‘ın LdapAuthenticationProvider, CasAuthenticationProvider, DaoAuthenticationProvider gibi farklı providerler mevcuttur. En uygun providerı Authentication Provider seçer.

- Authentication Provider, User Details Service‘i çağırır ve kullanıcı bilgilerini karşılık gelen kullanıcıyı bulur getirir. hizmetini kullanarak, kullanıcı adına karşılık gelen Kullanıcı Nesnesini getirir.

- User Details Service içerisinde loadUserByUsername metodu içerisinde (hesap kiliti mi veya etkin mi, kimlik bilgileri süresi dolup dolmadığı)  gibi bilgilere bakarak karşılık gelen in-memory ya da veritabanı ya da hangi kaynaklardan erişmesi gerekiyorsa erişir ve gelen kullanıcı bilgilerini bulduğu bilgilerini getirir ve eğer doğru kullanıcı bulunduysa ve bulunan kullanıcının nesnesini döndürmektedir. Bu servis içerisinde kullanıcı parolasını doğrulayan Password Encoder bulunmaktadır. Password Encoder kullanıcı parolasının kodlanması ve şifresinin çözülmesi gerektiğini söyleyen arabirimdir.

- Security Context, kimliği doğrulanmış kullanıcı hakkında bilgileri içermektedir. Bu kullanıcı bilgileri SecurityContextHolder sayesinde tüm uygulamamız içerisinde kullanabiliriz.

- Doğrulanmamış kullanıcı ile karşılaşıldığında ise Authentication Exception fırlatılır.

- Doğrulanmış kullanıcı uygulamamız içerisinde Security Context içerisinde tutulmaktadır. Sonradan giriş yapmış olan kullanıcı tekrar erişim istediğinde login olup olmaması kararını Security Context vermektedir. Kullanıcı bilgileri Security Context içerisinde mevcutsa tekrar login olmasına gerek kalmayacaktır.

EN <br>

- When the request from the user accesses our application, AuthenticationFilter takes the username and password from the request and an object is created.

Using the created Authentication Object comes to Authentication Manager after the filter. The created object contains Granted Authority, Roles and Principal information. Authentication Manager is an interface and the authentication method is run. Authentication Manager is an interface and sends data to the Authentication Provider.

It informs the Authentication Provider which type of verification will be performed in authentication transactions.

Authentication Provider has different providers such as LdapAuthenticationProvider, CasAuthenticationProvider, DaoAuthenticationProvider. Authentication Provider selects the most suitable provider.

The Authentication Provider calls the User Details Service and retrieves the user information for the corresponding user. Using the service, it fetches the User Object corresponding to the username.

In the User Details Service, it accesses the corresponding in-memory or database or from whatever sources it needs to access by looking at information such as the loadUserByUsername method (whether the account is locked or active, whether the credentials have expired) and brings the incoming user information it finds and if the correct user is used. If found and returns the object of the found user. This service includes Password Encoder, which verifies the user password. Password Encoder is the interface that tells the user password to be encoded and decrypted.

Security Context contains information about the authenticated user. We can use this user information throughout our application thanks to SecurityContextHolder.

When an unauthenticated user is encountered, an Authentication Exception is thrown.

The authenticated user is kept in the Security Context within our application. When the user who has logged in later requests access again, Security Context decides whether to log in or not. If the user information is available in the Security Context, there will be no need to log in again.
# Adapter Design Pattern

- Adapter iki uyumsuz interface'i beraber kullanmamızı sağlar
- Daha önce yazılmış olan kodları düzenlemek zorunda kalmayız, önceki kodun düzgün çalıştığını varsayıyoruz.
  
```java
public interface Crypt {
    void encrypt(String text);
    void decrypt(String text);
}
```

A class

```java
public class CryptA implements Crypt {

    public void encrypt(String text) {
        System.out.println("#CryptA#encrypt()");
    }

    public void decrypt(String text) {
        System.out.println("#CryptA#decrypt()");
    }
}
```

B class 

```java
public class CryptB implements Crypt {

    public void encrypt(String text) {
        System.out.println("#CryptB#encrypt()");
    }

    public void decrypt(String text) {
        System.out.println("#CryptB#decrypt()");
    }
}
```



```java
public class CodeX {

    public void textToCode(String text) {
        System.out.println("#CodeX#textToCode()");
    }

    public void codeToText(String text) {
        System.out.println("#CodeX#codeToText()");
    }

}
```

Adapter class

```java
public class CodeXAdapter implements Crypt{
    private CodeX codeX;
    
    public CodeXAdapter(CodeX codeX) {
        this.codeX = codeX;
    }

    // override
    public void encrypt(String text) {
        codeX.textToCode(text);
    }

    // override
    public void decrypt(String text) {
        codeX.codeToText(text);
    }

}
```

Main class

```java

public class Test {
    public static void main(String[] args) {

        Crypt crypt = new CryptA();
        crypt.encrypt("Test1");
        crypt.decrypt("Test2");

        System.out.println("-------------");

        crypt = new CryptB();
        crypt.encrypt("Test3");
        crypt.decrypt("Test4");

        System.out.println("-------------");

        CodeX codeX = new CodeX();

        crypt = new CodeXAdapter(codeX);
        crypt.encrypt("Test5");
        crypt.decrypt("Test6");
    }
}
```
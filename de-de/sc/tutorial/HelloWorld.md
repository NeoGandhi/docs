# Smart Contract Beispiel - HelloWorld

```c#
public class HelloWorld : SmartContract
{
    public static void Main()
    {
        Storage.Put(Storage.CurrentContext, "Hello", "World");
    }
}
```

Die Storage Klasse ist eine Static Klasse die den Privaten Contract Speicher manipuliert. Die 'Storage.Put()' Methode erlaubt es Ihnen Daten im Privaten Speicher im key-value format zu speichern. 

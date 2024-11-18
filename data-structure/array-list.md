# Array List


 **önemli not**
 ```java
     ArrayList<String> cars = new ArrayList<String>();
    cars.add("Volvo");
    cars.add("BMW");
    cars.add("Ford");
    cars.add("Mazda");
    for (int i = 0; i < cars.size(); i++) {
      System.out.println(cars.get(i));
    }
 ```
 for ile listeyi dönerken cars.size() almamak gerekiyor, her iterasyonda tekrardan listenin boyutunu hesaplamaya çalışır.


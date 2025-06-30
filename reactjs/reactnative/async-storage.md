# Async Storage

for jsx :

- <https://reactnative.dev/docs/asyncstorage>

for tsx :

- [https://medium.com/@can.gel/react-nativede-asyncstorage-kullanımı-2d85f5c387bd](https://medium.com/@can.gel/react-nativede-asyncstorage-kullan%C4%B1m%C4%B1-2d85f5c387bd)

## TSX Configuration

**installation :**

- npm install @react-native-async-storage/async-storage\`

import :

- import AsyncStorage from '@react-native-async-storage/async-storage'

```javascript
import { View, Text, Button } from "react-native";
import React from "react";
import AsyncStorage from "@react-native-async-storage/async-storage";

const App = () => {
  const saveUsername = async (username: string) => {
    try {
      await AsyncStorage.setItem("username", username);
      console.log("Kullanıcı adı kaydedildi:", username);
    } catch (error) {
      console.error("Kullanıcı adını kaydetme hatası:", error);
    }
  };

  return (
    <View>
      <Button title="Kaydet" onPress={() => saveUsername("Can")} />
    </View>
  );
};

export default App;
```

# ReactJS Backend Integration


src/App.js

```javascript
import React, { useState, useEffect } from 'react';
import logo from './logo.svg';
import './App.css';

// React backend API endpoint and API token
// REPLACE WITH YOURS
const API_ENDPOINT = 'https://reactback-nyly.api.codehooks.io/dev/hello';
const API_KEY = 'a4679c85-b4c8-49fb-b8ac-63230b269dd7';

function App() {
  // Application state variables
  const [visits, setVisits] = useState(null);
  const [message, setMessage] = useState(null);

  useEffect(()=>{
    // Call Codehooks backend API
    const fetchData = async () => {
      const response = await fetch(API_ENDPOINT, {
        method: "GET",
        headers: { "x-apikey": API_KEY }
      });
      const data = await response.json();
      // Change application state and reload
      setMessage(data.message);
      setVisits(data.visits);
    }
    fetchData();
  },[])

  return (
    <div className="App">
        <header className="App-header">
          <img src={logo} className="App-logo" alt="logo" />
          <h2>
            React backend with Codehooks.io
          </h2>
          <h2 style={{height: '50px'}} className="heading">
            {message || ''}
          </h2> 
          <p>
            Visitors: {visits || '---'}
          </p>          
        </header>
      </div>
  );
}

export default App;
```

- `useEffect()` : useEffect de tıpkı useState gibi React da bir hooks fonksiyonudur. Bu fonksiyon da component’in mount, update veya unmount durumlarında işlemleri gerçekleştirmek için kullanılır. Yani useEffect React component’inin yaşam döngüsü boyunca belirli işlemleri takip etmek ve gerçekleştirmek için kullanılır.
- Kafalar karışmasın, örneğin bir component’in mount edilmesi durumunda API’den veri çekilmesi, state veya props güncellenmesi durumunda tekrar API’den veri çekilmesi, component’in unmount edilmesi durumunda aboneliklerin iptal edilmesi gibi işlemler gerçekleştirebiliriz. Aşağıda useEffect örneğini görebilirsiniz.
```javascript
 useEffect(() => {
    console.log("lorem");
  });

  useEffect(() => {
    console.log(`lorem`);
  }, []);
```
  - Eğer React sınıf yaşam döngülerini (lifecycle) biliyorsanız, useEffect Hook’unu componentDidMount, componentDidUpdate, ve componentWillUnmount yaşam döngüsü methodlarının birleşimi olarak düşünebilirsiniz.


kaynak : https://medium.com/kodcular/react-hook-usestate-ve-useeffect-nedir-c50bf73f7a12
# Reusing authentication state


Web apps use cookie-based or token-based authentication, where authenticated state is stored as cookies. Playwright provides apiRequestContext.storageState() method that can be used to retrieve storage state from an authenticated context and then create new contexts with that state.

Storage state is interchangeable between BrowserContext and APIRequestContext. You can use it to log in via API calls and then create a new context with cookies already there. The following code snippet retrieves state from an authenticated APIRequestContext and creates a new BrowserContext with that state.

```javascript
const requestContext = await request.newContext({
  httpCredentials: {
    username: 'user',
    password: 'passwd'
  }
});
await requestContext.get(`https://api.example.com/login`);
// Save storage state into the file.
await requestContext.storageState({ path: 'state.json' });

// Create a new context with the saved storage state.
const context = await browser.newContext({ storageState: 'state.json' });
```
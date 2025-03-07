# Create React App

> In case you are with create-react-app, and only need local and production environment go with Adding Custom Environment Variables build-in feature. Otherwise, a useful alternative is env-cmd.

Install env-cmd, as a development dependency:

`npm i -D env-cmd`

Add .env file (at project root, same for all environments):

`REACT_APP_API_ENDPOINT = https://default.example.com`

Add .env.qa file:

`REACT_APP_API_ENDPOINT = https://qa.example.com`

Add .env.staging file:

`REACT_APP_API_ENDPOINT = https://stage.example.com`

Add .env.production file:

`REACT_APP_API_ENDPOINT = https://production.example.com`

Update package.json

```json
{
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "build:qa": "env-cmd -f .env.qa npm run-script build",
    "build:staging": "env-cmd -f .env.staging npm run-script build",
  }
}
```

- Note that default start and build create-react-app commands are using .env and .env.production respectively, and have no need of env-cmd.

Ready to use in our code

And, we also can use env-cmd for start command at development environment.

[Multiple env file](https://javascript.plainenglish.io/multiple-environments-in-create-react-app-619bd3dc1c61)



## Example 2

```
REACT_APP_API_END_POINT==http://my-dev-url.com
//.dev.env

REACT_APP_API_END_POINT==http://my-qa-url.com
//.qa.env

REACT_APP_API_END_POINT==http://my-prod-url.com
//.prod.env
```

package.json file 

```json
“scripts”: {“start”: “react-scripts start”,
“build”: “react-scripts build”,
“test”: “react-scripts test”,
“eject”: “react-scripts eject”,
“start:dev”: “env-cmd -f ./environments/.dev.env react-scripts start”,
“build:dev”: “env-cmd -f ./environments/.dev.env npm run-script build”,
“start:qa”: “env-cmd -f ./environments/.qa.env react-scripts start”,
“build:qa”: “env-cmd -f ./environments/.qa.env npm run-script build”,
“start:prod”: “env-cmd -f ./environments/.prod.env react-scripts start”,
“build:prod”: “env-cmd -f ./environments/.prod.env npm run-script build”
},
```

running application : 

`npm run start:dev`


## Vite Bundle

Configuring env files for staging, development and other phases is quite easy.

create .env.staging and .env.production and modify your package.json file in this way.

```
 "dev": "vite --mode staging",
 "production": "vite --mode production",
 "build": "vite build --mode production",
```

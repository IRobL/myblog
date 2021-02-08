---
layout: post
title:  "Java, React, MySql and Docker Compose!"
date:   2021-02-01 12:00:00 -0500
categories: software programming devops fullstack
---

Oh man, doing posts in winter is awful, but here goes.  I'm practicing with Java and React lately, and am coming up with some pretty interesting strategies to containerize things so local development isn't flaky.  So here's a step by step guide on starting a new full-stack git repo.


###### Folder Structure

The folder structure is pretty sweet, at the root of the project is a folder named after your project where the docker-compose.yml, and `README.md` file exist along with folders for each of the services involved in the app (a java folder, a react folder, and a db folder).

```
project-name
└─┐
  ├── java-api
  │
  ├── react-project-name
  │
  └── liquibase
```

Having a structure like this makes it very easy to begin local development, simply run `docker-compose build` and `docker-compose up` and all of your microservices do their thing and your'e ready to begin developing and watching your changes take place live over the services running in docker on your local machine.


###### Bootstrap Steps

0. Scaffold out the base folder (readme, git repo, docker-compose.yml)
0. Scaffold out the java-api folder
0. Scaffold out the react folder
0. Scaffold out the liquibase folder
0. Drop in the docker compose file


## Root Folder

Contents:

- Add `docker-compose.yml`
- Add `.gitignore`
- Add `.env` for secrets
- Add `README.md`


Here's my do-it-all `docker-compose.yml` file for mysql, java, react setups meant for placement in the root `project-name` folder.

(`docker-compose.yml`)
```
version: '3'

services:
  java:
    build: ./java-api
    image: 'registry.njax.org/project-name/java-api'
    env_file: ./.env
    environment:
      EMPTY: 'true'
    ports:
      - '8080:8080'
    depends_on:
      - 'db'
    volumes:
      - './java-api:/app'
    entrypoint: [ /entrypoint.sh, development_server ]

  react:
    build: ./react-project-name
    image: 'registry.njax.org/project-name/java-api'
		stdin_open: true
    tty: true
    env_file: ./.env
    environment:
      EMPTY: 'true'
    ports:
      - '3000:3000'
    volumes:
      - ./react-project-name:/app

  db:
    image: mysql:5.7
    restart: "no"
    ports:
      - "30409:3306"
    env_file:
      - ./.env
    volumes:
      - my-db-project-name:/var/lib/mysql

volumes:
  my-db-project-name:
```

(`.gitignore`)
```
.env  # Make sure you're hiding the sensitive files here
**/node_modules
```

(`.env`)
```
export MYSQL_USER='root'
export MYSQL_ROOT_PASSWORD='YOUR DESIRED MYSQLPASS'
export MYSQL_HOST='127.0.0.1'
export MYSQL_PORT='30409'

export REACT_APP_API_URL='127.0.0.1'
```

Good `README.md` files describe the project, the steps to boot it up in development mode, and may also include fun badges or diagrams.  Remember you can link to readme files located within your microservices to go into great depths about those systems.


## Java

- Head to [Spring Initializr](https://start.spring.io/) and generate the base project with `org.springframework.boot:spring-boot-starter-web` and `org.springframework.boot:spring-boot-starter-data-jpa`
- Add the `Dockerfile`
- Make sure to add in these snippets to the `build.gradle` file
- Configure


(`Dockerfile`)
```
FROM gradle:6.5.1-jdk11 AS build

EXPOSE 8080

# Create space for development mode builds/ serves shared via volumes
RUN mkdir /app

# Conduct build (for production mode containers)
COPY --chown=gradle:gradle build.gradle /home/gradle/src/
COPY --chown=gradle:gradle settings.gradle /home/gradle/src/
COPY --chown=gradle:gradle gradle /home/gradle/src/
COPY --chown=gradle:gradle src /home/gradle/src/src

WORKDIR /home/gradle/src
RUN gradle build --no-daemon
RUN cp /home/gradle/src/build/libs/app.jar /app.jar

COPY entrypoint.sh /entrypoint.sh
ENTRYPOINT [ "/entrypoint.sh", "deployment" ]
```

(`build.gradle`)
```
.
.
.
// enables automatic app restarting based on class changes
configurations {
  developmentOnly
  runtimeClasspath {
    extendsFrom developmentOnly
  }
}

// Force name of jar to be the same across all projects
jar     { archiveName 'app.jar' }
bootJar { archiveName 'app.jar' }
```

To build it, run `docker build . -t testbuild`.  Hopefully it built fine and you can test that it runs properly with `docker run -it testbuild`.


## React

- Use `npx create-react-app react-project-name` to scaffold the base files
- Add the common dependencies (react-router, redux,)
- Add debugging information to the root page

```
npx create-react-app react-project-name
cd react-project-name

# Add dependencies
npm install --save redux @reduxjs/toolkit react-router-dom typescript react-bootstrap react-toastify
```

Then overwrite your ReactDOM section in `index.js` with this snippet for react router and redux.

(`src/index.js`)
```
.
.
.
import { BrowserRouter as Router } from "react-router-dom";
import { Provider } from "react-redux";
import store from "./app/store";

ReactDOM.render(
  <React.StrictMode>
    <Provider store={store}>
      <Router>
        <App />
      </Router>
    </Provider>
  </React.StrictMode>,
  document.getElementById('root')
);
.
.
.
```

You'll need to create `src/app/store.js` for things to work now.

```
mkdir src/app
echo '
import { configureStore } from "@reduxjs/toolkit";
// import counterReducer from "../features/counter/counterSlice";

export default configureStore({
  reducer: {
    empty: "empty",
  },
});
' > src/app/store.js
```

Finally we get to the routing.  Include toastify here because it's better than `alert`.

```
echo '
import React from "react";
import { Route, Switch } from "react-router-dom";
import { ToastContainer } from "react-toastify";
import "react-toastify/dist/ReactToastify.css";

import Header from "./features/static/Header";
import PageNotFound from "./features/static/PageNotFound";
import HomePage from "./features/static/HomePage";
import AboutPage from "./features/static/AboutPage";

function App() {
  return (
    <div className="container-fluid">
      <Header />
      <Switch>
        <Route exact path="/" component={HomePage} />
        <Route path="/about" component={AboutPage} />
        <Route component={PageNotFound} />
      </Switch>
      <ToastContainer autoClose={3000} hideProgressBar />
    </div>
  );
}

export default App;
' > src/App.js
```

Now you have `6` new files to create before things will work again: Header, PageNotFound, Spinner, HomePage, AboutPage, and Store.  These files are all really simple.


###### Header.js

```
mkdir -p src/features/static

echo '
import React, {useEffect} from "react";

import { NavLink } from "react-router-dom";
import { connect } from "react-redux";
import propTypes from "prop-types";
// import { bindActionCreators } from "redux";

const Header = ({currentUserName, actions, ...props}) => {
  const activeStyle = { color: "#F15B2A" };


  useEffect(() => {
    console.log("I used header effects");
  });

  return (
    <nav>
      <NavLink to="/" activeStyle={activeStyle} exact>
        Home
      </NavLink>
      {" | "}
      <NavLink to="/about" activeStyle={activeStyle}>
        About
      </NavLink>
    </nav>
  );
};

Header.propTypes = {
  actions: propTypes.object.isRequired,
};

// Redux will magically call this when our state.users object changes following
// an action being sent to a reducer modifieing state.users
function mapStateToProps(state) {
  return {
    // currentUserName: state.login.currentUserName,
  };
}

// this fancy method gets installed into the components props for you per the export line below
function mapDispatchToProps(dispatch) {
  return {
    actions: {
      // setCurrentUsername: bindActionCreators(loginSliceActions.setCurrentUsername, dispatch),
      // logout: bindActionCreators(loginSliceActions.logout, dispatch),
    },
  };
}

// export default UsersPage;
export default connect(mapStateToProps, mapDispatchToProps)(Header);
' > src/features/static/Header.js
```


###### PageNotFound.js

```
echo '
import React from "react";

const PageNotFound = () => <h1>Oops! Page not found</h1>;

export default PageNotFound;
' > src/features/static/PageNotFound.js
```


###### Spinner.js

```
mkdir src/common

echo '
import React from "react";
// import "./Spinner.css";

const Spinner = () => {
  return <div className="loader">Loading...</div>;
};

export default Spinner;
' > src/common/Spinner.js
```


###### HomePage.js

```
echo '
import React from "react";

const HomePage = () => {
  return (
    <>
      <h1>Welcome to Your App</h1>

      <div className="debug">
        <div>API_URL: {process.env.REACT_APP_API_URL} </div>
      </div>
    </>
  );
};

export default HomePage;
' > src/features/static/HomePage.js
```


###### AboutPage.js

```
echo '
import React from "react";

const AboutPage = () => {
  return (
    <>
      <h1>About</h1>
      <div>I am the web frontend for project-name! I am written in React!</div>
    </>
  );
};

export default AboutPage;
' > src/features/static/AboutPage.js
```




###### Liquibase

Now that we have a Java backend, and a very vanilla React frontend, we can put together our liquibase folder project.  It's composition is as such:

```
liquibase
└─┐
  ├── Dockerfile
  │
  ├── entrypoint.sh
  │
  └── liquibase.properties
  │
  └── changelogs/
      └─┐
        ├── changelog.xml
        │
        ├── tables/
        │    └─┐
        |      ├─ changelog-ref-data.xml
        |      └─ users/users-0001.sql
        |
        └── data/
             └─┐
               ├─ changelog-ref-data.xml
               └─ users/users-0001.sql
```



## TODO: Pass through the services making them actually do stuff

I sadly didn't have the time to finish this posting... maybe I can get to it next weekend????  :(


###### Java - JPA Repository


###### React - Janked together CRUD scaffolding





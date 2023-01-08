---
date: "2020-03-05"
title: "Handling React Routes with Apache and Express.js"
tags: ["javascript", "react", "apache", "expressjs"]
author: "masoudkf"
description: "As far as web servers are concerned, React routes are fake routes. They do not follow the traditional file/folder hierarchy. They're just different components being mounted/unmounted by React at different times. Selling these fake routes as actual routes to Apache and Express.js is what this post is all about."
---

Without changing the configurations of Apache or Express.js, we will not be able to serve React routes as expected. Here's why: imagine that you have a React application with two pages, **Index** and **About**. You probably have an `App.js` file similar to this:

```javascript
import React from 'react';
import Home from './pages/Home';
import About from './pages/About';

import {
	BrowserRouter as Router,
	Switch,
	Route
} from 'react-router-dom'

class App extends React.Component {
   render() {
      return (
				<Router>
					<Switch>
						<Route path="/" component={Home} exact />
						<Route path="/about/" component={About} />
					</Switch>
				</Router>
			)
   }
}

export default App;
```

This tells React to **mount** the `Home` component for the main page (`/`) and the `About` component for the about page (`/about`). So, when you go to `/about`, it seems that you're now on a new page, where you're actually not. The `/about` page is just another component; just like the `/` page. React is handling everything behind the scene; you actually just have one `index.html` file that loads your React code to manage routing.

That means for the routing to be working correctly, React should always be present; otherwise, the routing will break, since there's no one to mount the correct component based on the current path. And who's responsible for loading your React code: the `index.html` file and no one else.

So, to wrap it up: for the React routing to work, React code should always be present, which means that the `index.html` file should always be present, which means that your web server should always be serving the `index.html` file regardless of the path in the address bar. The path is `yourwebsite.com/about`? Your web server should serve the `index.html` file. The path is `yourwebsite.com/batman`? Your web server should serve the `index.html` file. The path is ... You get the idea. No matter where you want to go inside your website, your web server should (almost) always bring up the `index.html` file that loads your React code. Period.

However, that's not the default behavior of web servers. For instance, when you go to `yourwebsite.com/about`, the Apache web server will normally look for an `index.html` file inside your `/about` folder. But that's not right. There's no `/about` folder inside our React application. The Apache web server should instead serve the `index.html` file from your root directory. That's also the case with other web application frameworks like Express.js. The rest of this tutorial is about how we can change this default behavior to serve our React routes correctly. Let's start with Apache.


## Configuring Apache

First, go to the root folder of your website. If you are using the default configurations that come with installing Apache, it's `/var/www/html/`. Otherwise, it's the path of your virtual host, like `/var/www/example.com/`. Create a file with the name `.htaccess` (or edit the existing file) using this command&mdash;I assume that you're most probably using a Linux distribution to launch your website; so `vi` should be available:

```bash
sudo vi /var/www/html/.htaccess
```

Once it's open, put the following script on the top:

```bash
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteCond %{REQUEST_FILENAME} !-l
  RewriteRule . /index.html [L]
</IfModule>
```

Don't worry too much if it seems complicated to you; it does to pretty much everyone. But just to clarify a little bit, let's break it down. The first line tells Apache that here we're trying to **rewrite** some rules and the second line turns on the `Rewrite` engine. The third line says that for the following rule (line 4), we should use the prefix `/` in front of every path. Therefore, on line 4, `index.html` becomes `/index.html` which is the correct relative path. Line 4 basically says that if the user looks for `/index.html`, don't do anything; just give the user the file they want&mdash;remember that the `/index.html` file is the file that the server should mount (almost) all the time. Lines 5-8 introduce a condition block. For each `RewriteCond`, if the condition is met, Apache will continue processing the next line. Finally, if all the three conditions are met, Apache will execute Line 8: `RewriteRule . /index.html [L]`.

But what are these conditions anyway? They're basically checking if the file or folder being asked by the user exists or not&mdash;that's the reason behind all those *almost*s so far. There are times when we don't want the server to hand the `/index.html` file to the user; images, for instance. If there is an image in the path `/static/images/elephant.jpg`, and somewhere inside our code we refer to that image, our server should be able to catch it and give it to us. It shouldn't serve the `/index.html` file this time, since the file does in fact exist. So, if the file or folder doesn't exist, it means that it's a route to be handled by React. Therefore, the server should again serve the `/index.html` file instead. That's what `RewriteRule . /index.html [L]` does. The `[L]` flag at the end simply means to stop going further (to the next line). It doesn't make much difference at the end of this example, since there's no line after that. But the first `[L]` flag is necessary to avoid the conditions. Hope it all makes sense to you. If it doesn't, don't worry about it at the moment. Just copy and paste it as it is.


There are still two little steps to take before Apache can serve our React app correctly. First, we should allow the `.htaccess` file we just made to apply those modifications. To do that, we should edit our Apache config file. Again, if you haven't touched Apache before, it means that all the default configurations are still in place and your config file is located at `/etc/apache2/sites-available/000-default.conf`. If you're using virtual hosts, it's probably named after your domain. For instance: `/etc/apache2/sites-available/mywebsite.com.conf`. Use the following command to open it:

```bash
sudo vim /etc/apache2/sites-available/000-default.conf
```

Once it's open, add the following script at the end of the file, just before the closing `</VirtualHost>` tag:

```apache
<Directory /var/www/html/>
		AllowOverride All
</Directory>
```

The whole file should look like this:

```apache
<VirtualHost *:80>
	ServerAdmin webmaster@localhost
	DocumentRoot /var/www/html

	ErrorLog ${APACHE_LOG_DIR}/error.log
	CustomLog ${APACHE_LOG_DIR}/access.log combined

	<Directory /var/www/html/>
		AllowOverride All
	</Directory>
</VirtualHost>
```

This allows Apache to override any configuration it needs to apply our settings in the `.htaccess` file. Note that if you're not using the default virtual host, you want to specify a different path rather than `/var/www/html/`. Also, I deleted all the comments in the file to save some space; yours should also include the comments (the lines with the `#` sign at the beginning).

We're almost there. The last thing we need to do is to enable `mode_rewrite` and restart Apache. To do that, run the following command:

```bash
sudo a2enmod rewrite
sudo systemctl restart apache2
```

And that's all. Now you can go to your website and try different React routes. You should be able to see that it's working as it's supposed to. Great!

Let's see how we can achieve the same thing with Express.js.

## Configuring Express.js
Compared to Apache, the pain of configuring Express.js is nothing. There are just two steps.

- Tell Express.js where to find your static files; that is your `index.html` file, `.js` files, etc.
- Tell Express.js to serve the `index.html` file if the file or path does not exist


### Serving static files with Express.js
In order to serve your static files, Express.js needs to know where they are located; put differently, it needs to know the root folder of all your static files. Just add this  line after calling the `express()` function:

```javascript
var app = express()

//add this line after the above line
app.use(express.static(path.join(__dirname, '/public')))
```

Here, I assumed that your static files are located inside the `/public` folder. You can easily change it to something else, of course, based on your application.

#### Telling Express.js to serve the `index.html` file (almost) always
By now, you completely understand the importance of this step&mdash;we already did the same thing for Apache. To do this in Express.js, you only need to add just three lines (you can also make it one line) at the end of your Express.js main file, just before the line where you listen to a specific port:

```javascript
//add these 3 lines just before the app.listen line
app.get("*", (req, res) => {
   res.sendFile(path.join(__dirname, "/public/index.html"))
})

app.listen(port, () => console.log(`Example app listening on port ${port}!`))
```

It's important to add this line at the end of the file. The reason is that once Express.js matches a request to a particular route, it won't continue to consider the remaining routes. So, if you remember from the Apache part, we need our server to serve everything that exists and only show the `index.html` when there is no matched file, folder, or route. For instance, let's assume that in your Express.js app, you also have a few other endpoints, like the following:

```javascript
app.get('/post/:id', cors(), async (req, res) => {
	//stuff
})
```

If you don't put the code snippet for serving the `index.html` after these lines, what happens is that Express.js won't serve these endpoints anymore because our code snippet basically matches all `get` requests. So, it's important to have the code at the end.
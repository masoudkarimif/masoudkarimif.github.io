---
date: "2023-02-25"
title: "Netlify Gotchas When Deploying a React Application"
tags: ["react", "netlify", "devops"]
author: "masoudkf"
description: "There are at least two gotchas when it comes to deploying a React applications built with `create-react-app` on Netlify. One's related to the deployment, and the other to routing."
---

[Netlify](https://netlify.com) is a cool platform that lets you deploy your React applications (among other types of applications) for free--assuming your website doesn't use a lot of resources, such as bandwidth. You can check out Netlify plans [here](https://www.netlify.com/pricing/).

But regardless of what plan you're going with, you'll be in for a surprise when you want to deploy a React app with routing on Netlify.

## Gotcha #1: Deployment
Although Netlify figures out the build command when you connect your React GitHub repo, it may not work right out of the box. If you used the `create-react-app` application to start your React project, the default build command is `react-scripts build`. This will work perfectly on your system when you run `npm run build`. However, it won't work on Netlify. As Netlify builds the application inside a CI (Continuous Integration) provider, `create-react-app` doesn't let the application be built unless there's an environment variable named `CI` that points to the build command. 

The fix is easy. You need to open up the `package.json` inside the root directory of your React application, find the build command, and prefix it with `CI=`. Here's how the command looks like at first:

`package.json`
```json
  ...
  "scripts": {
    ...
    "build": "react-scripts build",
    ...
  }
  ...
```

Here's how it should look like after the change:

`package.json`
```json
  ...
  "scripts": {
    ...
    "build": "CI= react-scripts build",
    ...
  }
  ...
```

---

## Gotcha #2: Front-end Routing
If you're using a front-end router in your React application, such as `react-router-dom`, the routing will not work as expected after deploying on Netlify. For example, if you have a page `/about` that loads an About component, the address will result in a 404 Page Not Found after deploying on Netlify. To understand the reason, you need to know how web servers work in general when you try to visit an address.

If you're trying to visit a file, such as `/about.html`, web servers look for a file with the same name and show it to you, assuming it exists. However, when you're trying a URL without an extension, such as `/about`, web servers look for a **folder** named `about` and show you the `index.html` (could be also `index.php`, etc.) file inside the folder, assuming it exists.

Now, let's revisit our problem. We try the URL `/about` on Netlify and the web server looks for an `index.html` file inside an `about` folder. However, we don't have an `about` folder; that's why the server returns a 404 response. But what do we have in our React application anyway?

The only HTML file we have in our React application is an `index.html` file inside the `public` folder (will be moved to the `build` folder after the build). That's the file that loads the React library, which in turn loads the router library. Everything, including all the navigations, happens inside the `index.html` file: a single file; hence the name Single Page Application or SPA.

So, now, you should be able to see where this is going. If we had a way to always serve the `index.html` file no matter which page users request, our problem would be solved. Fortunately, Netlify provides a way to alter the default behaviour of its web servers. The secret lies in a special file named `_redirects`. If this file exists inside the `public` folder of our React application, Netlify changes the routing behaviour according to the rules it finds in the file. Here's how the syntax works in a nutshell:

```bash
path(s)-requested path-to-be-served status-code
```

The first parameter is the path(s) requested by the user. To match all paths on your website, put `/*`. The second parameter is the path or file you want to be served if the first parameter is a match. Here, we want to serve the `/index.html` file (the only HTML file we have). The last parameter is the HTTP status code we want to return to the user. We choose `200` as that's the successful HTTP status code. 

Here's how it should look like:

`public/_redirects`
```bash
/* /index.html 200
```

And that's it. Push your changes to your repo and Netlify will re-deploy your application with the latest changes. Both problems should be fixed now.


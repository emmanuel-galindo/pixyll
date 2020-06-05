---
published: true
title: Avoid adding credit card in heroku to get MongoDB
layout: post
description: Heroku will request your credit card when trying to spin up a MongoDB instance, but you can still connect to MongoDB own cloud Atlas, using their free tier
---


## Connecting MongoDB from Heroku free tier

Heroku out of the box provide a simple way to spin MongoDB instances, and automatically configures it to your running application:

``` shell
$ heroku addons:create mongolab
```

A good tutorial on this topic is [https://devcenter.heroku.com/articles/mean-apps-restful-api#provision-a-mongodb-database](https://devcenter.heroku.com/articles/mean-apps-restful-api#provision-a-mongodb-database)

However, when running the command in the free tier, it will ask you validate a credit card in their site to verify your account:

``` shell
$ heroku addons:create mongolab
Creating mongolab on ⬢ radiant-lowlands-14472... !
 ▸    Please verify your account to install this add-on plan (please enter a credit card) For more information, see https://devcenter.heroku.com/categories/billing Verify now
 ▸    at https://heroku.com/verify
``` 

### Configuring MongoDB

Luckily, MongoDB has Atlas offering a free tier as well and we could connect to it from Heroku following these steps:

1. Register into MongoDB site
2. Select Shared Cluster offering for the free tier
3. Select the region to deploy the cluster. It might be a good idea to have in consideration the region your heroku is running to match it.
4. MongoDB will take several minutos to provision the cluster
5. Once provisioned it will look like below, and click Connect:

![mongodb.png](/images/mongodb.png)

6. On whitelist, it asks for the IP ranges to allow connectivity from the remote client, in this case Heroku. Not sure if Heroku publish this, but you could select 0.0.0.0/0. This will enable connectivity from any source.
7. For the user, fill your desired username and password
8. Move to the next page by clicking "Choose a connection method" 
9. Select "Connect your application"
10. Copy the URL and Close the dialog

You'll have something like
```
mongodb+srv://user:<password>@cluster0-cmmbk.mongodb.net/<dbname>?retryWrites=true&w=majority
```

Replace password, and dbname use "test":
```
mongodb+srv://user:password@cluster0-cmmbk.mongodb.net/test?retryWrites=true&w=majority
```

### Configuring Heroku to connect to MongoDB

> If you get below output when executing an heroku command:
> ```
>  ›   Error: Missing required flag:
>  ›     -a, --app APP  app to run command against
>  ›   See more help with --help
> ````
> Execute the command with the app name as part of the heroku command with -a modifier. For example:
> ```
> $ heroku apps
> == user@email.com Apps
> radiant-lowlands-14321
> $ heroku ps -a radiant-lowlands-14321  
> ```


Now the DB is ready to receive connections. Go to your local heroku app folder and generate this config env var:

```shell
$ heroku config:set MONGODB_URI="mongodb+srv://user:password@cluster0-cmmbk.mongodb.net/test?retryWrites=true&w=majority"
```

If you need a simple unit test, use this server.js:
```javascript
const db = process.env.MONGODB_URI;

const connectDB = async () => {
  try {
    await mongoose.connect(db, {
      useUnifiedTopology: true,
      useNewUrlParser: true
    });
    console.log("MongoDB is Connected...");
  } catch (err) {
    console.error(err.message);
    process.exit(1);
  }
};
```

Commit and push it as usual and in the logs of the app should read:
```
2020-06-05T00:10:30.017396+00:00 app[web.1]: Database connection ready
```


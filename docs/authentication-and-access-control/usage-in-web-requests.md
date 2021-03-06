# Usage in Web Requests

FoalTS uses sessions and the `Authenticate` hook to authenticate users accross several requests.

## The `Authenticate` Hook

It should decorate the `AppController`. If the user has already logged in in a previous request, then it will be available in the `context` with which the controller methods are called.

```ts
@Authenticate(User)
export class AppController {
  @Get('/foo')
  foo(ctx: Context) {
    // ctx.user is defined.
  }
}
```

## How to Log a User In or Out

If you want to attach an authenticated user to the current session, proceed as follows:

```typescript
logIn(ctx, user); // The logIn utility is in the @foal/core package.
```

To log out a user, use the `logOut` utility:
```typescript
logOut(ctx);
```

> Note: The `LoginController` presented below does these steps for you.

## Limiting Access to Logged-In Users

### The LoginRequired Hook

When no user is authenticated, the `LoginRequired(options?: { redirect?: string })` hook:
- returns a `401 Unauthorized` if `options.redirect` is undefined,
- redirects the page to the given path if `options.redirect` is defined.

### Access Control in Simples Scenarios

In simple scenarios where authorization is managed with an `isAdmin` property or static roles, you can use these hooks:

```typescript
import { Context, Hook } from '@foal/core';

function AdminRequired() {
  return Hook(ctx => {
    if (!ctx.user) {
      return new HttpResponseUnauthorized();
    }
    if (!ctx.user.isAdmin) {
      return new HttpResponseForbidden();
    }
  })
}
```

```typescript
import { Context, Hook } from '@foal/core';

function RoleRequired(role: string) {
  return Hook(ctx => {
    if (!ctx.user) {
      return new HttpResponseUnauthorized();
    }
    if (!ctx.user.roles.includes(role)) {
      return new HttpResponseForbidden();
    }
  })
}
```

### The PermissionRequired Hook

The `PermissionRequired(perm: string, options: { redirect?: string } = {})` hook:
- returns a `401 Unauthorized` if no user is authenticated and `options.redirect` is undefined,
- redirects the page to the given path if no user is authenticated and  `options.redirect` is defined,
- returns a `403 Forbidden` if the user does not have the given permission. The `perm` argument is the `codeName` of the permission.

## The LoginController

> *This LoginController is experimental*.

The `LoginController` is a handy controller to log a user in or out using one or several authentication mechanisms.

To log a user out using this controller, simply request GET /logout. When the logout succeeds it returns an `HttpResponseNoContent` if `redirect` is undefined or an `HttpResponseRedirect` if it is defined.

<!--
When the authentication succeeds it returns an `HttpResponseNoContent` if `successRedirect` is undefined or an `HttpResponseRedirect` if it is defined.

When the authentication fails it returns an `HttpResponseUnauthorized` if `failureRedirect` is undefined or an `HttpResponseRedirect` if it is defined.
-->

### The MVC Approach

Create a `User` with an email and a password and create an `Authenticator` that extends the `EmailAuthenticator` class.

```sh
foal g controller auth --register
> Login
```

Replace the content with:
```typescript
import { emailSchema, Get, LoginController, render, strategy } from '@foal/core';
​​
import { Authenticator } from '../services/authenticator.service';

export class AuthController extends LoginController {
  strategies = [
    strategy('login-with-email', Authenticator, emailSchema),
  ];

  redirect = {
    failure: '/login', // optional
    logout: '/login', // optional
    success: '/home', // optional
  };

  @Get('/login')
  renderLogin(ctx) {
    return render('./templates/login.html', { csrfToken: ctx.request.csrfToken() }, __dirname);
  }
}
```

Create a file named `login.html` inside `controllers/templates` with the following content:
```html
<html>
  <head></head>
  <body>
    <form action="/auth/login-with-email" method="POST">
      <input style="display: none" name="_csrf" value="<%= csrfToken %>">
      <input type="email" name="email">
      <input type="password" name="password">
      <button type="submit">Log in</button>
    </form>
  </body>
</html>
```

### The API Approach

### The Strategies

- `EmailAuthenticator` and `emailSchema`

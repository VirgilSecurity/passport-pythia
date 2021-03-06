> This README is for @virgilsecurity/passport-pythia v1.0.0. Check the [v0.1.x branch](https://github.com/VirgilSecurity/virgil-passport-pythia/tree/v0.1.x) for an old version.

# @virgilsecurity/passport-pythia
[![npm](https://img.shields.io/npm/v/@virgilsecurity/passport-pythia.svg)](https://www.npmjs.com/package/@virgilsecurity/passport-pythia)
[![Build Status](https://img.shields.io/travis/VirgilSecurity/virgil-passport-pythia.svg)](https://travis-ci.org/VirgilSecurity/virgil-passport-pythia)
[![GitHub license](https://img.shields.io/badge/license-BSD%203--Clause-blue.svg)](https://github.com/VirgilSecurity/virgil-passport-pythia/blob/master/LICENSE)

[Passport](http://www.passportjs.org/) strategy for authenticating with the Virgil [Pythia PRF](https://eprint.iacr.org/2015/644.pdf) service.

This module lets you authenticate using a username and password while protecting the passwords cryptographically using the Pythia PRF service. We'll refer to passwords protected with the Pythia PRF service as [Breach-Proof Password](https://developer.virgilsecurity.com/docs/go/use-cases/v1/breach-proof-password).

By plugging into Passport, Breach-Proof Password support can be easily and unobtrusively
integrated into any application or framework that supports
[Connect](http://www.senchalabs.org/connect/)-style middleware, including
[Express](http://expressjs.com/).

## Pre-requisites

* Create a free [Virgil Security](https://dashboard.virgilsecurity.com/) account.
* Create a **Breach-Proof Password Storage** app in the Virgil Security [Dashboard](https://dashboard.virgilsecurity.com/apps/new).
* Create an **API Key** in the Virgil Security [Dashboard](https://dashboard.virgilsecurity.com/api-keys).

## Install

```sh
npm install @virgilsecurity/passport-pythia
```

This module depends on `virgil-pythia` module to be installed to be able to communicate with the Virgil Pythia PRF service and perform the cryptographic operations necessary to verify the passwords.

```sh
npm install virgil-pythia
```

You also need to install `@virgilsecurity/pythia-crypto` and `virgil-crypto`, unless plan to use custom crypto implementations.

```sh
npm install @virgilsecurity/pythia-crypto virgil-crypto
```

## Usage

### Configure strategy

The strategy requires two parameters. The first is an instance of `Pythia` class from the `virgil-pythia` module. The second is a `getAuthenticationParams` callback, which is responsible for retrieving the breach-proof password parameters of the user making the request. It accepts the `request` object and a callback to be called with an error as a first argument, if any, and the breach-proof password parameters as the second argument.

```javascript
passport.use(new PythiaStrategy(
    virgilPythia,
    (request, cb) => {
        User.findOne({ username: request.body.username }, (err, user) => {
            if (err) return cb(err);
            if (!user) return cb(new Error('Invalid username'));
            cb(null, {
                user,
                password: request.body.password,
                salt: user.bppSalt,
                deblindedPassword: user.bppDeblindedPassword,
                version: user.bppVersion
            });
        });
    }
));
```

### Authenticate Requests

Use `passport.authenticate()`, specifying the `'pythia'` strategy, to authenticate requests.
For example, as route middleware in an Express application:

```javascript
app.post(
  '/sign-in',
  passport.authenticate('pythia', {
    successRedirect: '/profile',
    failureRedirect: '/sign-in',
  }),
);
```

## Examples

Developers using the [Express](http://expressjs.com/) web framework can refer to an [example](./example) as a starting point for their own web applications.

## Tests

To run this example on your computer, clone this repository and install dependencies.

```sh
git clone https://github.com/VirgilSecurity/virgil-passport-pythia.git
cd passport-pythia
npm install
```

Create a new file named `.env` with the contents of `.env.example`

```sh
cp .env.example .env
```

Open the `.env` file in a text editor and replace the values starting with `[YOUR_VIRGIL_...` with the corresponding values from your Virgil Dashboard.

Run the tests.

```
npm test
```

## License

This library is released under the [BSD 3-Clause License](LICENSE).

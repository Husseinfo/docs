---
title: Post Change Password Extensibility Point
description: Learn how hooks can be used with the Post Change Password extensibility point, which is available for database connections.
beta: true
toc: true
topics:
    - hooks
    - extensibility-points
    - post-change-password
contentType:
    - how-to
useCase: extensibility-hooks
v2: true
---

# Post Change Password Extensibility Point

At the Post Change Password extensibility point, Hooks allow custom actions to be executed after a successful user password change, whether initiated by a user for their own password or by a tenant administrator for another user's password. For example, you may send an email to a user to notify them that their password has been changed.

The Hook added to this extensibility point executes asynchronously with the rest of the Auth0 pipeline, and its outcome does not affect the Auth0 transaction.

The Post Change Password extensibility point is available for [Database Connections](/connections/database).

To learn about other extensibility points, see [Extensibility Points](/hooks/extensibility-points).

To learn how to create Hooks, see [Create New Hooks](/hooks/create).

## Starter code and parameters

After you've created a new Post Change Password Hook, open up the Hook and edit it using the Webtask Editor embedded in the Dashboard. 

The parameters listed in the comment at the top of the code indicate the Auth0 objects and their parameters that can be passed into and used by the Hook's function.

```js
/**
@param {object} user - The affected user
@param {string} user.id - user id
@param {string} user.username - user name
@param {string} user.email - email
@param {string} user.last_password_reset - exact date/time the user's password was changed
@param {object} context - Auth0 connection and other context info
@param {object} context.connection - information about the Auth0 connection
@param {object} context.connection.id - connection id
@param {object} context.connection.name - connection name
@param {object} context.connection.tenant - connection tenant
@param {object} context.webtask - webtask context
@param {function} cb - function (error)
**/
module.exports = function (user, context, cb) {
  // Perform any asynchronous actions, e.g. send notification to Slack.
  cb();
};
```

The callback function `cb` at the end of the sample code is used to signal completion and must not be omitted.

### Response

The Post Change Password Hook ignores any response object. If an error is returned a tenant log entry is created, but this does not affect the Auth0 transaction.

## Testing Hooks

<%= include('../_includes/_test_runner_save_warning') %>

You can test Hooks using the Runner. The runner simulates a call to the Hook with the appropriate user information body/payload. The following is the sample body that populates the Runner by default (these are the same objects/parameters detailed in the comment at the top of the sample Hook code):

```json
{
  "user": {
    "id": "abc123",
    "username": "user1",
    "email": "user1@foo.com",
    "last_password_reset": "2019-02-27T14:14:29.206Z"
  },
  "context": {
    "connection": {
      "id": "con_xxxxxxxxxxxxxxxx",
      "name": "Username-Password-Authentication",
      "tenant": "my-tenant"
    }
  }
}
```

## Example: Use SendGrid to send a password change notification email

```js
module.exports = function (user, context, cb) {

  const request = require('request');

  // https://sendgrid.api-docs.io/v3.0/mail-send
  request.post({
    url: 'https://api.sendgrid.com/v3/mail/send',
    headers: {
      'Authorization': 'Bearer YOUR_API_KEY'
    },
    json: {
      personalizations: [{
        to: [{
          email: user.email
        }]
      }],
      from: {
        email: 'admin@example.com'
      },
      subject: 'Your password was changed',
      content: [{
        type: 'text/plain',
        value: 'The password for your ' + context.connection.name + ' account ' + user.email + ' was recently changed.'
      }]
    }
  }, function (err, resp, body) {
    if (err || resp.statusCode !== 202) {
      return cb(err || new Error(body.errors[0].message));
    }

    cb();
  });
};
```
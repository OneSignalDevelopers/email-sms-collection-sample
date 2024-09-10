# Email and SMS Collection

Collecting an email or SMS from a frontend website for subscribing to a newsletter or coupon code.

## Guide

Exposing the REST API key publicaly is dangerous and not advised. Fortunately, our API allows you to create a user and a subscription without using the API key when identity verification is not enabled.

```javascript
fetch(createUserAPI, {
    method: "POST",
    mode: "no-cors",
    body: JSON.stringify({
      subscriptions: [
        {
          type: "Email",
          token: "iamnotwill@onesignal.com",
          enabled: true,
        },
      ],
    }),
  }),
```

Pass an array of Subscription objects to the Create User API to create a user who owns those subscriptions.

```javascript
const payload = {
  subscriptions: [
    {
      type: "Email",
      token: "iamnotwill@onesignal.com",
      enabled: true,
    },
    {
      type: "SMS",
      token: "+12345678901",
      enabled: false,
    },
  ]
}
```

* Set the subscription's `type` to Email or SMS.
* The `token` must be valid for the given `type`, e.g., email subscriptions must use a valid email address.
* Conditionally enable the subscription upon creation with `enabled`.

```javascript
fetch(createUserAPI, {
    method: "POST",
    mode: "no-cors",
    body: JSON.stringify({
      subscriptions: [
        {
          type: "Email",
          token: "iamnotwill@onesignal.com",
          enabled: true,
        },
      ],
    }),
  }),
```

### No Access to Backend

```javascript
const OoneSignalAppID = "<APP_ID>";
const createUserAPI = `https://api.onesignal.com/apps/${OoneSignalAppID}/users`;
const [error, _opaqueResult] = await safeTry(() =>
  fetch(createUserAPI, {
    method: "POST",
    mode: "no-cors", // this is key if the customer doesn't control their server'
    body: JSON.stringify({
      subscriptions: [
        {
          type: "Email",
          token: "iamnotwill@onesignal.com",
          enabled: true,
        },
      ],
    }),
  }),
);

if (error) {
  console.error("Failed to create user");
} else {
  // response is not useful because of no-cors mode
  console.log("User created successfully");
}
```

### Access to Backend

If you have the ability to modify your server's CORS headers, allow our API `https://api.onesignal.com`.

```javascript
const [error, result] = await safeTry(() =>
  fetch(createUserAPI, {
    method: "POST",
    body: JSON.stringify({
      subscriptions: [
        {
          type: "Email",
          token: "iamnotwill@onesignal.com",
          enabled: true,
        },
      ],
    }),
  }),
);

if (error) {
  console.error("Failed to create user");
} else {
  const { identity, subscriptions } = await result.json();
  console.log("User created successfully");
  console.log(`OneSignal ID: ${identity["onesignal_id"]}`);
  console.log(`Subscription ID: ${subscriptions[0].id}`);
}
```

### Misc

Utility code to make consuming our API result slightly nicer

```javascript
const safeTry = async (fn) => {
  try {
    const result = await fn();
    return [null, result];
  } catch (e) {
    if (e instanceof Error) {
      console.error(e.message);
    } else if (typeof e === "string") {
      console.error(e);
    } else {
      console.error("An error occurred");
    }

    return [e, null];
  }
};
```

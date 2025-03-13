---
layout: post
title: "Using Basic Auth in Pipedream"
tags:
  - Pipedream
  - HTTP
  - Authentication
  - Basic Auth
---

# Overview
I've been making some UIs in Pipedream and I couldn't find a clear way to implement basic authentication in Pipedream workflows.
It might be obvious to some but it wasn't to me. It was pretty obvious to me after I figured it out.

So I hope this helps you!

# How to Set Up Basic Authentication in Pipedream

Create a New Workflow and choose New HTTP / Webhook Requests as your trigger.
Make sure to configure the trigger to return a custom response from your workflow for the HTTP Response.

Add a new code step as your first step in the workflow and add the following code:

```javascript
export default defineComponent({
  async run({ steps, $ }) {
    // Get the Authorization header from the incoming request
    const authHeader = steps.trigger.event.headers.authorization;

    // Check if the header exists and starts with 'Basic '
    if (authHeader == undefined || !authHeader.startsWith('Basic ')) {
      await $.respond({
        status: 401,
        headers: { 'WWW-Authenticate': 'Basic realm="Restricted Area"' },
        body: 'Unauthorized',
      });
      $.flow.exit('Authentication required');
      return;
    }

    // Extract and decode the base64 credentials
    const base64Credentials = authHeader.split(' ')[1];
    const credentials = Buffer.from(base64Credentials, 'base64').toString('utf-8');
    const [username, password] = credentials.split(':');

    // Verify the username and password (customize these values)
    const expectedUsername = 'user';
    const expectedPassword = 'password';

    if (username === expectedUsername && password === expectedPassword) {
      console.log('Authentication successful');
    } else {
      await $.respond({
        status: 401,
        headers: { 'WWW-Authenticate': 'Basic realm="Restricted Area"' },
        body: 'Unauthorized',
      });
      $.flow.exit('Authentication failed');
    }
  },
});
```

## Firebase

A work in progress to contains a full cheatsheet of all the features of firebase in one and simple place
You are welcome to do PR

* [Security Rules](#security-rules)
* [Cloud Functions](#cloud-functions)
* [Firestore](#firestore)

## Security Rules
TODO: There is a great video on the firebase channel about this


## Cloud Functions
TODO: https://firebase.google.com/docs/functions/http-events

##### Import the admin/functions and set a region of the functions
```
import * as Functions from "firebase-functions";
import * as admin from "firebase-admin";

const region = "europe-west1";
admin.initializeApp();
const functions = Functions.region(region);
```

##### Simple HTTP Trigger
```
export const onExample = functions.https.onRequest(request, response) =>
{
    const query = request.query; // Query params
    const body = request.body; // POST/PUT body

    return response.json({data: {success: true}});
});
```

##### Authorized User HTTP Trigger
```
function getDecodedIdToken(request, response): Promise<admin.auth.DecodedIdToken>
{
    const tokenId = request.get("Authorization").split("Bearer ")[1];

    return admin.auth().verifyIdToken(tokenId).catch((err) => response.status(401).send(err));
}

export const onUserRequest = functions.https.onRequest(request, response) =>
{
    return getDecodedIdToken(request, response).then(decoded =>
    {
        console.log("User UID:", decoded.uid);
    });
});
```

##### Using exsiting Express apps
```
const express = require("express");
const cors = require("cors");

const app = express();

// Automatically allow cross-origin requests
app.use(cors({ origin: true }));

// Add middleware to authenticate requests
app.use(myMiddleware);

// build multiple CRUD interfaces:
app.get("/:id", (req, res) => res.send(Widgets.getById(req.params.id)));
app.post("/", (req, res) => res.send(Widgets.create()));
app.put("/:id", (req, res) => res.send(Widgets.update(req.params.id, req.body)));
app.delete("/:id", (req, res) => res.send(Widgets.delete(req.params.id)));
app.get("/", (req, res) => res.send(Widgets.list()));

// Expose Express API as a single Cloud Function:
exports.widgets = functions.https.onRequest(app);
```
[source](https://firebase.google.com/docs/functions/http-events#using_existing_express_apps)

## Firestore

##### Get all the documents of a collection

```
admin.firestore()
        .collection("items")
        .get()
        .then(snapshot =>
        {
            const list = [];

            snapshot.docs.forEach(doc =>
            {
                list.push({
                    id: doc.id,
                    ...doc.data()
                });
            });

            console.log("items:", list);
        });
```

##### Simple transaction

```
admin.firestore().runTransaction(t =>
{
    return admin.firestore()
        .collection("users")
        .doc(user.uid)
        .collection("items")
        .add({
            name: "Test"
        })
        .then(doc =>
        {
            const itemId = doc.id;

            // Use the ref
            t.set(doc.ref, {name: "Test updated " + itemId});
        });
});
```